### 1、@EnableDiscoveryClient和@EnableEurekaClient。
> - 1、@EnableDiscoveryClient注解是基于spring-cloud-commons依赖，并且在classpath中实现；
> - 2、@EnableEurekaClient注解是基于spring-cloud-netflix依赖，只能为eureka作用。
> - 3、如果你的classpath中添加了eureka，则它们的作用是一样的。