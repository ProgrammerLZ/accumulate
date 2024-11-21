`@Configuration`

`@Conditional`

`@ConditionalOnClass`

`@ConditionalOnMissingBean`



META-INF/spring.factories file

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

> Auto-configurations must be loaded that way *only*. Make sure that they are defined in a specific package space and that they are never the target of component scanning. Furthermore, auto-configuration classes should not enable component scanning to find additional components. Specific **@Imports** should be used instead.

`@AutoConfigureAfter`

 `@AutoConfigureBefore`

`@AutoConfigureOrder` <=> `@Order`



条件注解

- [Class Conditions](https://docs.spring.io/spring-boot/docs/2.4.4/reference/html/spring-boot-features.html#boot-features-class-conditions)
- [Bean Conditions](https://docs.spring.io/spring-boot/docs/2.4.4/reference/html/spring-boot-features.html#boot-features-bean-conditions)
- [Property Conditions](https://docs.spring.io/spring-boot/docs/2.4.4/reference/html/spring-boot-features.html#boot-features-property-conditions)
- [Resource Conditions](https://docs.spring.io/spring-boot/docs/2.4.4/reference/html/spring-boot-features.html#boot-features-resource-conditions)
- [Web Application Conditions](https://docs.spring.io/spring-boot/docs/2.4.4/reference/html/spring-boot-features.html#boot-features-web-application-conditions)
- [SpEL Expression Conditions](https://docs.spring.io/spring-boot/docs/2.4.4/reference/html/spring-boot-features.html#boot-features-spel-conditions)

