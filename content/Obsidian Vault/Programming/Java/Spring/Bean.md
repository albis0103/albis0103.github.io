The object managed by  Spring IoC (Inversion of Control) container instantiation, assembly and lifecycle.

Bean Difinition
1. XML config
```Java
<bean id="userService" class="com.example.UserService">
```
2. annotation
```Java
@Component
public class UserService{
@Service
@Repository
@Controller
}
```
3. Configuration
```Java
@Configuration
public class AppConfig{
	@Bean
	public UserService userService(){
		return new UserService();
	}
}
```

Feature
- Bean Scope
	- `Singleton`(Default): Container only unique instantiation
		- `@Service //@Scope("singleton")`
	- `prototyope`: new instantiation for each used
		- `@Component`
		- `@Scope("prototype")`
	- `request`, `session`, `application`: web environment
- life cycle
![[Pasted image 20260913102341.png]]
	1. Instantiation（出生）
	2. set Property（出生登記）
	3. BeanNameAware`setBeanName()`(取名字)
	4. BeanFactoryAware`setBeanFactory()`（選學校）
	5. BeanPostProcess`postProcessBeforeInitialization()`（報名）
	6. Initializing Bean `afterPropertySet()`(入學登記)
	7. 自定義init方法（努力學習）
	8. BeanPostProcess`postProcessAfterInitialization()`（畢業）
	9. Using Bean（工作）
	10. DisposableBean `destroy()`（死了）
	11. 自定義 `destroy()`（埋了）
