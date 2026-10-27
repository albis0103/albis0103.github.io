是 JPA(Java Persistance API, 通常是Hibernate)提供資料存取的抽象層。只要宣告繼承特定Java Interface, Spring自己實作CRUD



```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByEmailAndActiveTrue(String email);
}
```

泛型參數 `<T, ID>` 分別是 Entity 型別與主鍵型別。



```
Repository<T, ID>                      標記介面，無方法，供 classpath scanning 識別
 └─ CrudRepository<T, ID>              save, findById, existsById, findAll, count, delete...
     └─ ListCrudRepository<T, ID>      同上，但回傳 List 而非 Iterable（3.0+）
 └─ PagingAndSortingRepository<T, ID>  findAll(Sort), findAll(Pageable)
     └─ ListPagingAndSortingRepository
JpaRepository<T, ID>                   繼承上述 List 版本，並加入 JPA 特有操
```

 `PagingAndSortingRepository` 不再繼承 `CrudRepository`，兩者需要分別繼承；`JpaRepository` 已經同時涵蓋兩者。如果不想暴露 `delete` 這類方法，可以直接繼承 `Repository`，只宣告需要的方法（最小介面原則）。


### Design Entity and Primary Key




Spring 產生實作的流程分為四步：

1. **掃描**：Spring Boot 的 `JpaRepositoriesAutoConfiguration`（或手動加上的 `@EnableJpaRepositories`）掃描 base package 下繼承 `Repository` 的介面。
2. **產生Proxy**：`JpaRepositoryFactoryBean` 用 `JpaRepositoryFactory` 為每個介面建立 JDK dynamic proxy。
3. **委派**：代理把呼叫分派到兩個地方Query。介面中已定義的 CRUD 方法委派給預設實作 `SimpleJpaRepository<T, ID>`。自訂的查詢方法則交給 `QueryLookupStrategy` 解析成 `RepositoryQuery` 物件。
4. **套用交易**：`SimpleJpaRepository` 類別層級標註 `@Transactional(readOnly = true)`，寫入方法（`save`、`delete*`）另外覆寫為 `@Transactional`。因此即使 Service 層沒有交易，單次 Repository 呼叫也會在自己的交易中執行。

`QueryLookupStrategy` 預設為 `CREATE_IF_NOT_FOUND`：先找 `@Query` 或 Named Query，找不到才從方法名稱推導查詢。




## 查詢方式

### 1. Derived Query（方法名稱推導）

方法名稱由 `PartTree` 解析：前綴（`find…By`、`count…By`、`exists…By`、`delete…By`）加上以 `And` / `Or` 連接的屬性條件，最後轉成 JPA Criteria 查詢。

```java
List<User> findByLastNameAndAgeGreaterThan(String lastName, int age);
Optional<User> findFirstByOrderByCreatedAtDesc();
List<User> findByAddress_City(String city);          // 巢狀屬性
boolean existsByEmail(String email);
```

常用關鍵字包括 `Between`、`LessThan`、`Like`、`Containing`、`StartingWith`、`In`、`IsNull`、`True`、`IgnoreCase`、`OrderBy…Asc/Desc`、`Top/First N`。屬性名稱在啟動時就會驗證，拼錯會導致 context 啟動失敗（fail-fast）。

### 2. @Query（JPQL 或 Native SQL）

```java
@Query("select u from User u where u.status = :status and u.createdAt > :since")
List<User> findActiveSince(@Param("status") Status status,
                           @Param("since") Instant since);

@Query(value = "select * from users where email ~* ?1", nativeQuery = true)
List<User> findByEmailRegex(String regex);

@Modifying(clearAutomatically = true)
@Transactional
@Query("update User u set u.active = false where u.lastLogin < :t")
int deactivateInactive(@Param("t") Instant t);
```

更新或刪除語句必須加上 `@Modifying`。這類 bulk update 會繞過 persistence context，所以通常搭配 `clearAutomatically = true`，避免一級快取裡留下過時的 entity。

### 3. 分頁與排序

```java
Page<User> findByActiveTrue(Pageable pageable);     // 會額外執行 count 查詢
Slice<User> findByRole(Role role, Pageable p);      // 只多取一筆判斷 hasNext，無 count

repo.findByActiveTrue(PageRequest.of(0, 20, Sort.by("createdAt").descending()));
```

### 4. Projection

Projection 用來只取部分欄位，減少 SELECT 的欄位與 entity 的 hydration 成本。

```java
interface UserSummary { String getName(); String getEmail(); }   // interface-based
record UserDto(String name, String email) {}                      // class/record-based

List<UserSummary> findByActiveTrue();
<T> List<T> findByRole(Role role, Class<T> type);                 // dynamic projection
```

### 5. Specification / Query by Example

Specification 適合動態組合條件（例如搜尋表單），介面需要繼承 `JpaSpecificationExecutor<T>`：

```java
Specification<User> spec = (root, q, cb) -> cb.equal(root.get("role"), role);
repo.findAll(spec.and(otherSpec), pageable);
```

需要型別安全的複雜查詢時，另一個選擇是整合 Querydsl（`QuerydslPredicateExecutor`）。

## save() 的語意

`SimpleJpaRepository.save()` 的行為取決於 `entityInformation.isNew(entity)`：

```java
if (entityInformation.isNew(entity)) { em.persist(entity); return entity; }
else { return em.merge(entity); }
```

`isNew` 的預設判定規則是：若 entity 有 `@Version` 且非 primitive 型別，看 version 是否為 null；否則看 ID 是否為 null（primitive 型別則看是否為 0）。

這帶來兩個實務影響。第一，如果主鍵是手動指派的（非 `@GeneratedValue`），`isNew` 永遠回傳 false，於是走 `merge`，會先多一次 SELECT。解法是讓 entity 實作 `Persistable<ID>` 並自行定義 `isNew()`。第二，對 managed entity 修改欄位後，其實不必呼叫 `save`，交易 commit 時 dirty checking 會自動 flush。

## 自訂實作（Fragment）

Derived query 和 `@Query` 不足以表達的邏輯，可以用 fragment 介面加上實作類別補上：

```java
interface UserRepositoryCustom { List<User> complexSearch(Criteria c); }

class UserRepositoryCustomImpl implements UserRepositoryCustom {   // 預設後綴 Impl
    @PersistenceContext private EntityManager em;
    public List<User> complexSearch(Criteria c) { /* Criteria API / JPQL */ }
}

interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom {}
```

代理會把 `complexSearch` 分派給 `UserRepositoryCustomImpl`。

## 常見陷阱

**N+1 查詢**：延遲載入的關聯在迴圈中存取時，每筆資料都會觸發一次額外查詢。可以用 `@EntityGraph(attributePaths = {"orders"})` 或 JPQL 的 `join fetch` 在同一次查詢中載入。注意 `join fetch` 搭配分頁時，Hibernate 會在記憶體中分頁並發出 HHH90003004 警告。

**`deleteAll()` / `deleteBy…()`**：這類方法會先把所有目標 entity 載入，再逐筆 `remove`，以便觸發 lifecycle callback 與 cascade。大量刪除應改用 `deleteAllInBatch()` 或 `@Modifying` JPQL。

**`getReferenceById()`**：回傳的是 lazy proxy，不會立即查詢資料庫。ID 不存在時要到存取欄位才會拋出 `EntityNotFoundException`；若在交易外存取，則會得到 `LazyInitializationException`。

**`saveAll()` 並非真正的 batch insert**：要讓 Hibernate 合併成 JDBC batch，需要設定 `spring.jpa.properties.hibernate.jdbc.batch_size`。此外，ID 策略為 `IDENTITY` 時 Hibernate 會停用 insert batching，要改用 `SEQUENCE` 才有效。

**Open Session in View**：Spring Boot 預設開啟 `spring.jpa.open-in-view=true`，這會讓 lazy loading 延伸到 view 層，掩蓋交易邊界設計上的問題。一般建議關閉，並在 Service 層明確管理交易與抓取策略。

