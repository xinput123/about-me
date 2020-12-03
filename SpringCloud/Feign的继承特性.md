**背景**
        在项目中经常会有这样的设计，在最外面一层只负责提供 URL地址，具体逻辑在内部实现，并且不对外暴露，这种做法让我最先想的的是dubbo。

### 一、准备：我们需要四个模块。
- 1.Eureka。
- 2.服务接口定义模块：通过Spring MVC注解定义抽象的interface服务接口。
- 3.服务接口实现模块：实现服务接口定义模块的interface，该模块作为服务提供者注册到eureka。
- 4.服务接口消费模块：服务接口定义模块的客户端实现，该模块通过注册到eureka来消费服务接口。

### 二、project层。
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.3.RELEASE</version>
        <relativePath/>
    </parent>

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.audit</groupId>
    <artifactId>audit</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>

    <modules>
        <module>audit-eureka</module>
        <module>audit-rest</module>
        <module>audit-service</module>
        <module>audit-service-user</module>
    </modules>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-cloud.version>Dalston.RELEASE</spring-cloud.version>
        <feign-httpclient.version>8.18.0</feign-httpclient.version>
        <spring-boot-starter-log4j.version>1.3.8.RELEASE</spring-boot-starter-log4j.version>
        <eureka-client.version>1.3.0.RELEASE</eureka-client.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- other module -->
            <dependency>
                <groupId>com.audit</groupId>
                <artifactId>audit-service</artifactId>
                <version>${project.version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-log4j</artifactId>
                <version>${spring-boot-starter-log4j.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

### 三、Eureka
#### 3-1、pom.xml 文件
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>audit</artifactId>
        <groupId>com.audit</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.audit</groupId>
    <artifactId>audit-eureka</artifactId>

    <properties>
        <!-- 启动类 -->
        <start-class>com.eureka.EurekaApp</start-class>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <finalName>audit-eurekaServer</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

#### 3-2、application.yml 文件
```
server:
  port: 9010

eureka:
  instance:
    hostname: 127.0.0.1
  client:
    registerWithEureka: false
    fetchRegistry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

#### 3-3、启动类
```
package com.eureka;

import org.springframework.boot.SpringApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringCloudApplication
public class EurekaApp {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApp.class, args);
    }
}
```

### 四、服务接口定义模块 audit-service
#### 4-1、pom.xml 文件
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>audit</artifactId>
        <groupId>com.audit</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.audit</groupId>
    <artifactId>audit-service</artifactId>

    <dependencies>
        <!-- feign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

</project>
```

#### 4-2、定义接口类
```
package com.server;

import com.base.param.UserParam;
import com.base.po.SelectResult;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@FeignClient(value = "${audit.services.name}")
public interface UserService {
    @RequestMapping(value = "/test",method = RequestMethod.GET)
    String test();
}
```

### 五、服务接口实现模块 audit-service-user
#### 5-1、pom.xml 文件
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>audit</artifactId>
        <groupId>com.audit</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.audit</groupId>
    <artifactId>audit-service-user</artifactId>

    <dependencies>
        <!-- other module-->
        <dependency>
            <groupId>com.audit</groupId>
            <artifactId>audit-service</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>     

        <!-- eureka -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <!-- feign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>

    </dependencies>

</project>
```

#### 5-2、application.yml 文件
```
server:
  port: 9012
  tomcat:
    uri-encoding: UTF-8

eureka:
  client:
    service-url:
      defaultZone: http://localhost:9010/eureka/ # 服务注册中心地址

spring:
  application:
    name: audit-services-user
```

#### 5-3、接口实现类
```
package com.service.impl;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserServiceImpl implements UserService{

    private final static Logger logger = LoggerFactory.getLogger(UserServiceImpl.class);

    @Override
    public String test() {
        return "adfadfadf";
    }

}
```

###### 5-4、启动类，将服务注册到eureka
```
package com.service;

import org.springframework.boot.SpringApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringCloudApplication
@EnableEurekaClient
public class UserServiceApp {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApp.class, args);
    }

}
```


### 六、服务接口消费模块 audit-rest
#### 6-1、pom.xml 文件
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>audit</artifactId>
        <groupId>com.audit</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.audit</groupId>
    <artifactId>audit-rest</artifactId>

    <properties>
        <start-class>com.rest.RestApplication</start-class>
    </properties>

    <dependencies>
        <!-- other module-->
        <dependency>
            <groupId>com.audit</groupId>
            <artifactId>audit-service</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>

        <!-- feign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <finalName>audit-rest</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>

        <resources>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
</project>
```

#### 6-2、application.yml 文件
```
server:
  port: 9011
  tomcat:
    uri-encoding: UTF-8

eureka:
  client:
    service-url:
      defaultZone: http://localhost:9010/eureka/ # 服务注册中心地址

spring:
  application:
    name: audit-rest

feign:
  compression:
    request:
      enabled: true
    response:
      enabled: true

audit:
  services:
    name: audit-services-user
```

#### 6-3、客户端，提供外部访问
```
package com.rest.user;

import com.server.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(value = "/test",method = RequestMethod.GET)
    public String test(){
        return userService.test();
    }
}
```

#### 6-4、启动类，并注册到eureka
```
package com.rest;

import org.springframework.boot.SpringApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.feign.EnableFeignClients;

@SpringCloudApplication
@EnableEurekaClient
@EnableFeignClients(basePackages={"com.server","com.rest"})
public class RestApplication {
    public static void main(String[] args) {
        SpringApplication.run(RestApplication.class, args);
    }
}
```