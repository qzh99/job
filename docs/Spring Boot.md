# SpringBoot可以兼容老的Spring版本吗，如何做？

参考：https://blog.csdn.net/syilt/article/details/92217652

`@ImportResource` 注解导入旧配置文件（注解写在启动类）

```java
@SpringBootApplication
@ImportResource(locations = {"classpath:spring.xml"})
public class Application{
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

