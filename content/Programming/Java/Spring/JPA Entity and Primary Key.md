
# Spring Data JPA：Entity 與主鍵設定

以下以 Spring Boot 3 為準（Jakarta Persistence 3.x、Hibernate 6），import 皆為 `jakarta.persistence.*`。

**Entity**
```Java
import java.util.UUID;
import jakarta.persistance.Entity;
import jakarta.persistance.GeneratedValue;
import jakarta.persistance.GenerationType;
import jakarta.persistance.Id;
import jakarta.persistance.Table;
import jakarta.persistance.Column;
```
映射到Table的Java Class, 由JPA Provider(ex. Hibernate)管理 life cycle.
```Java
@Entity
@Table(name = 'users', 
		uniqueConstraints = @UniqueConstraint(columnNames = "email"))
public class User{

	@Id
	@GeneratedValue(strategy = GenerationType.UUID)
	private UUID id;
	
	@Column(name = "user_name", nullable = false, length = 50)
	private String name;
	
	@Column(name = "email", nullable = false)
	private String email;
	
	protected User(){} //JPA 要求無參constructor
}

```

- 類別必須有 public 或 protected 的無參建構子，因為 provider 用反射建立實例。
- 類別與持久化欄位不可為 `final`，否則 Hibernate 無法產生 lazy loading 用的 proxy 子類別。
- 每個 Entity 必須有且只有一個主鍵定義，可以是 `@Id`、`@EmbeddedId` 或 `@IdClass



**單一主鍵：`@Id` + `@GeneratedValue`**

主鍵型別建議用包裝型別（`Long`、`Integer`、`UUID`），不要用 `long`。原因見第 5 節的 `isNew()` 判斷。

**GenerationType** 

| 策略         | 機制                  | 適用 DB                                                    | 特性                                                                                        |
| ---------- | ------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `IDENTITY` | 依賴欄位 auto-increment | MySQL、MariaDB、SQL Server、PostgreSQL（`serial`/`identity`） | INSERT 後才取得 ID，因此 Hibernate 會停用 JDBC batch insert                                         |
| `SEQUENCE` | 從 DB sequence 取值    | PostgreSQL、Oracle、H2                                     | 可在 INSERT 前取得 ID，支援 batch；搭配 `allocationSize`減少 round-trip                                |
| `TABLE`    | 用一張表模擬 sequence     | 任意                                                       | 需要 row lock，效能差，一般不用                                                                      |
| `AUTO`     | 由 provider 決定       | —                                                        | Hibernate 6 預設選 `SEQUENCE`；DB 不支援 sequence 時退回 table 模擬（MySQL 會出現 `hibernate_sequence` 表） |
| `UUID`     | 應用端產生 UUID          | 任意                                                       | JPA 3.1 新增，不需 DB 參與                                                                       |

**SEQUENCE 的完整設定**

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "order_seq")
@SequenceGenerator(name = "order_seq", sequenceName = "order_id_seq", allocationSize = 50)
private Long id;
```

- `allocationSize = 50` 代表 Hibernate 每次向 DB 取一個 sequence 值後，在記憶體中分配 50 個 ID（pooled optimizer）。DB sequence 的 `INCREMENT BY` 必須與 `allocationSize` 一致，否則會產生重複鍵或跳號。
- JPA 預設的 `allocationSize` 是 50，而手動建立的 sequence 通常是 `INCREMENT BY 1`，兩者不一致是常見的錯誤來源。

**UUID 主鍵**

```java
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private UUID id;
```


**複合主鍵**

複合主鍵類別必須符合三個條件：public、implements `Serializable`、正確覆寫 `equals()` / `hashCode()`。JPA 需要後者來做 persistence context 的一級快取比對。

### 方式 A：`@EmbeddedId` + `@Embeddable`（較常用）

```java
@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
    // 無參建構子、equals、hashCode
}

@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
    private int quantity;
}

public interface OrderItemRepository extends JpaRepository<OrderItem, OrderItemId> {}
```

使用時以 `item.getId().getOrderId()` 存取鍵欄位。JPQL 寫法為 `where o.id.orderId = :x`，衍生查詢方法命名為 `findByIdOrderId(...)`。

### 方式 B：`@IdClass`

```java
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
    // equals、hashCode
}

@Entity
@IdClass(OrderItemId.class)
public class OrderItem {
    @Id private Long orderId;
    @Id private Long productId;
    private int quantity;
}
```

Entity 內的 `@Id` 欄位名稱與型別必須與 IdClass 完全對應。優點是欄位直接位於 Entity 上，查詢可寫成 `findByOrderId`。

兩者的差異在於：`@EmbeddedId` 把主鍵視為一個值物件，型別封裝較完整；`@IdClass` 則讓欄位保持扁平，查詢較直觀。Repository 的第二個泛型參數在兩種方式下都是 ID 類別。

## 4. 衍生主鍵：`@MapsId`

當主鍵同時是外鍵時使用 `@MapsId`，例如一對一共用主鍵，或複合鍵中含 FK：

```java
@Entity
public class UserProfile {
    @Id
    private Long id;          // 值取自 user.id

    @OneToOne
    @MapsId
    @JoinColumn(name = "id")
    private User user;
}
```

在複合鍵中可寫 `@MapsId("orderId")`，將關聯映射到 `@EmbeddedId` 內的指定屬性，避免同一欄位重複映射。

## 5. `save()` 與主鍵的關係：`isNew()` 判斷

`SimpleJpaRepository.save()` 的邏輯如下：

```java
if (entityInformation.isNew(entity)) em.persist(entity);
else return em.merge(entity);
```

預設的 `isNew()` 判斷分兩種情況。有 `@Version` 欄位時，看 version 是否為 null。沒有 `@Version` 時，看 ID 是否為 null；若 ID 是 primitive 型別，則看是否為 0。

這帶來一個常見問題：主鍵若由應用端手動指定（沒有 `@GeneratedValue`，例如業務代碼或自行產生的 UUID），`save()` 會判定為非新物件而呼叫 `merge()`，因此先發出一次 SELECT 再 INSERT。解法是實作 `Persistable<ID>`：

```java
@Entity
public class Product implements Persistable<String> {
    @Id private String code;

    @Transient private boolean isNew = true;

    @Override public String getId() { return code; }
    @Override public boolean isNew() { return isNew; }

    @PostLoad @PostPersist
    void markNotNew() { this.isNew = false; }
}
```

## 6. `equals()` / `hashCode()` 實作建議

若 Entity 使用 DB 產生的 ID，persist 前 ID 為 null。這時若以 ID 計算 `hashCode()`，物件放入 `HashSet` 後，hash 值會在 persist 時改變，導致集合內找不到該物件。常見做法有三種：

1. 有不可變的業務鍵時，以業務鍵實作，可搭配 Hibernate 的 `@NaturalId`。
2. 以 ID 判斷相等，`hashCode()` 則回傳固定值，例如 `getClass().hashCode()`。
3. 使用應用端產生的 UUID 主鍵，建構時即賦值。

Lombok 的 `@Data` 與 `@EqualsAndHashCode` 會把所有欄位（包含 lazy 關聯）納入計算，可能觸發 lazy loading 或雙向關聯的無窮遞迴，不建議用在 Entity 上。