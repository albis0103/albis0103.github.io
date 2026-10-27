
**Common used**


**1. /Config** 

用來集中管理某個技術領域的 bean 設定，通常放在 `config` 或 `configuration` 套件下：

```
com.example.app
└── config
    ├── DataSourceConfig.java
    ├── SecurityConfig.java
    ├── WebMvcConfig.java
    ├── RedisConfig.java
    └── SwaggerConfig.java
```
ex. Redis configuration
```Java
@Configuration
public class RedisConfig{
	@Bean
	public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory){
		RedisTemplate<String, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(factory);
		template.setKeySerializer(new StringRedisSerializer());
		return template;
	}
}
```

**2. 第三方函式庫的整合點**

任何需要組裝第三方物件的地方：`DataSource`、`RestTemplate`、`WebClient`、`ObjectMapper`、`RabbitTemplate`、`OpenAPI` 設定等。

**3. 條件式/多環境 bean 設定**

搭配 `@Profile`、`@ConditionalOnProperty` 依環境切換實作：

```java
@Configuration
public class StorageConfig {
    @Bean
    @Profile("prod")
    public StorageService s3StorageService() { return new S3StorageService(); }

    @Bean
    @Profile("dev")
    public StorageService localStorageService() { return new LocalStorageService(); }
}
```

**4. Spring Boot 自動配置類別**

Spring Boot 本身大量使用這個模式（如 `DataSourceAutoConfiguration`、`WebMvcAutoConfiguration`），本質上就是官方寫好的 `@Configuration` 類別。

**5. 匯入外部設定（`@Import`）**

`@Configuration` 類別彼此可用 `@Import` 組合，常見於模組化架構或 Spring Boot Starter 開發。

## @Component 家族常見使用位置

對應到標準三層架構，位置固定：

```
com.example.app
├── controller
│   └── OrderController.java      // @Controller / @RestController
├── service
│   ├── OrderService.java          // @Service (interface)
│   └── impl
│       └── OrderServiceImpl.java  // @Service
├── repository
│   └── OrderRepository.java       // @Repository
└── component (或 util)
    ├── JwtTokenProvider.java       // @Component
    └── EmailSender.java            // @Component
```

- `@Controller` / `@RestController`：接收 HTTP 請求的入口類別
- `@Service`：業務邏輯層
- `@Repository`：資料存取層（DAO）
- 純 `@Component`：不屬於上述三層，但仍需容器管理的元件，例如：filter、interceptor、validator、事件監聽器（`@EventListener` 所在類別）、排程任務（`@Scheduled` 所在類別）、工具類但需要注入依賴的情況

## 實務判斷準則

|情境|用哪個|
|---|---|
|自己寫的業務邏輯類別，建構簡單|`@Component` 系列|
|第三方套件的物件（`DataSource`、`ObjectMapper`...）|`@Configuration` + `@Bean`|
|需要依環境/條件切換實作|`@Configuration` + `@Bean`（搭配 `@Profile`/`@Conditional`）|
|Spring Security 的 `SecurityFilterChain`、`PasswordEncoder`|`@Configuration` + `@Bean`（官方 API 就是這樣設計）|
|Controller / Service / DAO|`@Component` 系列對應的專用註解|

簡單記法：**業務程式碼用 `@Component` 系列（歸位到 controller/service/repository），基礎設施與第三方整合用 `@Configuration` + `@Bean`（集中在 config 套件）**。這也是為什麼幾乎所有 Spring Boot 專案的目錄結構都長這樣的原因。