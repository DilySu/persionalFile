# Eureka服务注册中心

## 一、注册中心

​      注册中心就是微服务架构中的通讯录，它记录了服务和服务地址的映射关系，在分布式架构中，服务会注册到这里，服务通过注册中心调用其他服务。

## 二、常见的注册中心

| 特性              | Eureka      | Nacos                      | Consul            | Zookeeper  |
| ----------------- | ----------- | -------------------------- | ----------------- | ---------- |
| CAP               | AP          | CP+AP                      | CP                | CP         |
| 健康检查          | Client Beat | TCP/HTTP/MYSQL/Client Beat | TCP/HTTP/gRPC/Cmd | Keep Alive |
| 雪崩保护          | 有          | 有                         | 无                | 无         |
| 自动注销实例      | 支持        | 支持                       | 不支持            | 支持       |
| 访问协议          | HTTP        | HTTP/DNS                   | HTTP/DNS          | TCP        |
| 监听支持          | 支持        | 支持                       | 支持              | 支持       |
| 多数据中心        | 支持        | 支持                       | 支持              | 不支持     |
| 跨注册中心同步    | 不支持      | 支持                       | 支持              | 不支持     |
| Spring Cloud 集成 | 支持        | 支持                       | 支持              | 支持       |

## 三、存在意义

​		注册中心主要进行服务管理和服务依赖关系管理

> - 服务注册后及时发现
> - 服务宕机后即使下线
> - 水平拓展服务
> - 服务发现后处理路由
> - 服务异常后的降级处理

## 四、角色

![注册中心角色](img/Eureka注册中心角色.jpeg)

- Eureka Server ：通过 Register、Get、Renew 等接口提供服务的注册和发现
- Server Provider（Application Service）： 服务提供方，把自身的服务实例注册到 Eureka Server 中
- Server Consumer（Application Service）：服务调用方，通过 Eureka Server 获取服务列表，消费服务

## 五、运行流程



![注册中心运转流程](img/Eureka注册中心运转流程.jpeg)

> 0：实例化服务
>
> 1：将服务注册到注册中心
>
> 2：注册中心收录服务
>
> 3：从服务中心获取服务列表
>
> 4：基于负载均衡从地址列表中选择服务地址进行服务调用
>
> 5：定期发送心跳
>
> 6：检查没有定期发送心跳的服务，并在一定时间内剔除服务地址列表

## 六、入门案例

### 1、创建Spring  Cloud  工程

### 2、创建 Eureka 服务

#####  pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <parent>
      <groupId>com.byx</groupId>
      <artifactId>scaffold-parent</artifactId>
      <version>0.0.1</version>
      <relativePath>../parent</relativePath>
   </parent>
   <artifactId>scaffold-eureka</artifactId>
   <packaging>${packaging.type}</packaging>
    
   <dependencies>
      <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
         <exclusions>
            <exclusion>
               <groupId>org.junit.vintage</groupId>
               <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
         </exclusions>
      </dependency>
   </dependencies>

   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>

</project>
```

application

```java
package com.byx.scaffold.rcentre;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class ServiceRegistrationAndDiscoveryApplication {

   public static void main(String[] args) {
      SpringApplication.run(ServiceRegistrationAndDiscoveryApplication.class, args);
   }
}
```

### 3、配置 application.yml

```yaml
spring:
  application:
    name: eureka                 # 应用名称
server:
  port: 8761                     # 端口
eureka:
  instance:
    hostname: localhost          # 主机名
    prefer-ip-address: true
  client:
    register-with-eureka: false  # 是否将自己注册到注册中心，默认 true ，集群时要为 true，相互注册
    fetch-registry: false        # 是否从注册中线获取注册信息，默认为 true
    service-url:                 # 注册中心对外暴露的注册地址
      defaultZone: http://toolKit:dily1314@localhost:8761/eureka/
```

### 4、服务间调用

#### 方式一：RestTemplate + DiscoveryClient

启动类或配置中增加 RestTemplate Bean

```java
//@EnableEurekaClient 如果配置了注册中心，则会默认开启，无需使用该注解
@SpringBootApplication
public class ConsumerApplication {

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    };

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```

Service 中调用服务

```java
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;   // 启动类 或者 Config 中 要注入bean
    @Autowired
    private DiscoveryClient discoveryClient;


    public Order getList() {
        return new Order(1, "202020202020", selectProductListWithBalancer("provider", "/product/list"));
    }


    /**
     * 跨服务调用
     * 无负载均衡
     *
     * @param service 服务名
     * @param api     接口
     * @return 结果
     */
    private List<?> selectProductList(String service, String api) {
        StringBuffer sb = new StringBuffer();

        // 获取服务方法
        List<String> services = discoveryClient.getServices();
        if (CollectionUtils.isEmpty(services))
            return null;

        // 根据服务名获取服务
        List<ServiceInstance> serviceInstances = discoveryClient.getInstances(service);
        if (CollectionUtils.isEmpty(serviceInstances))
            return null;
        ServiceInstance si = serviceInstances.get(0);
        sb.append("http://" + si.getHost() + ":" + si.getPort() + api);

        // 返回数据
        ResponseEntity<List<?>> response = restTemplate.exchange(
                sb.toString(),
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<?>>() {
                }
        );
        return response.getBody();
    }
}
```

#### 方式二：RestTemplate + LoadBalancerClient

启动类或配置中增加 RestTemplate Bean

```java
//@EnableEurekaClient 如果配置了注册中心，则会默认开启，无需使用该注解
@SpringBootApplication
public class ConsumerApplication {

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    };

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

Service 中调用服务

```java
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;   // 启动类 或者 Config 中 要注入bean
    @Autowired
    private LoadBalancerClient loadBalancerClient;


    public Order getList() {
        return new Order(1, "202020202020", selectProductListWithBalancer("provider", "/product/list"));
    }
    /**
     * 跨服务调用
     * 有负载均衡
     *
     * @param service 服务名
     * @param api     接口
     * @return 结果
     */
    private List<?> selectProductListWithBalancer(String service, String api) {
        StringBuffer sb = new StringBuffer();

        // 根据服务名 通过负载均衡 获取服务
        ServiceInstance si = loadBalancerClient.choose(service);
        if (si == null)
            return null;
        sb.append("http://" + si.getHost() + ":" + si.getPort() + api);

        // 返回数据
        ResponseEntity<List<?>> response = restTemplate.exchange(
                sb.toString(),
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<?>>() {
                }
        );
        return response.getBody();
    }
}
```

#### 方式三：RestTemplate + @LoadBalanced

启动类或配置中增加 RestTemplate Bean 加入注解 @LoadBalanced

```java
//@EnableEurekaClient 如果配置了注册中心，则会默认开启，无需使用该注解
@SpringBootApplication
public class ConsumerApplication {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    };

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}	
```



## 七、Eureka 原理

![](img\Eureka原理图.jpeg)

## 八、CAP原则

![CAP原则](img\CAP原则.jpeg)

> C: 一致性
>
> A: 高可用
>
> P: 分区容错

​		金融: CP(大型) 、AP(互联网)

#### 		取舍策略:

-   CA： 如果不要求 P（不允许分区），则 C（强一致性）和 A（高可用性）是可以保证的．放弃Ｐ意味着放弃了系统的扩展性，也就是分布式节点受限，没办法部署子节点，这是违背分布式系统设计初衷的。

- CP：如果不要求 A（不要求高可用），则 C 相当于每个请求都需要在服务之间保持强一致性，而 P （分区容错）会导致同步时间无线延长（也就是等待数据同步才能正常访问服务），一旦发生网络故障或者消息丢失等情况，就要牺牲用户的体验，等待所有数据全部一致后再让用户访问系统。设计成 CP 的系统其实不少，最典型的就是分布式数据库，如：redis，对于分布式数据库来说，数据的一致性时最基本的要求。
- AP：要求高可用并允许分区容错，则放弃一致性，一旦分区发生，节点之间可能失去联系，为了高可用，每个节点都只能使用本地数据提供服务，而这会导致全局数据不一致，典型应用就是抢购商品的场景，可能浏览的页面还显示有商品，但是下单的时候发现商品已经售完，这其实就是在A（高可用）方面保证可以正常服务，在数据一直性方面做了牺牲。

## 九、自我保护

一般情况，服务在 Eureka 注册后 每30s 发送一次心跳，Eureka 通过心跳包来判断服务是否健康，同时会定期删除超过 90s 没有发送心跳的服务。

#### 1、Eureka收不到服务的情况：

- 微服务自身原因
- 微服务与 Eureka 之间的网络故障

#### 2、自我保护模式

在运行期间，Eureka 会统计心跳失败比例，如果 15 分钟内的低于 85%，Eureka 会将这些实例保护起来，让这些实例不会过期，同时提醒一个警告

#### 3、关闭自我保护

```yaml
eureka:
  server:
    enable-self-preservation: false # true 开启自我保护， false 关闭自我保护
    eviction-interval-timer-in-ms: 60000 # 清除隔离（单位：毫秒） 默认 60 * 1000
```

## 十、优雅停服

配置优雅停服后，无需配置关闭自我保护，这里用actuator实现

#### 1、依赖

```xml
<!--        优雅停服-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### 2、配置

```yaml
# 度量指标监控与健康检查
management:
  endpoints:
    web:
      exposure:
        include: shutdown   # 开启 shutdown 端点访问
  endpoint:
    shutdown:
      enabled: true # 开启 shutdown 实现优雅停服
```

#### 4、停服

向需要停服的服务地址发送 shutdown 请求

> 服务地址：172.10.16:8080
>
> 停服请求：Post方式  172.10.16:8080/actuator/shutdown

## 十一、认证授权

#### 1、依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

#### 2、配置

##### 2.1 Eureka配置

```yaml
spring:
  # 安全认证
  security:
    user:
      name: root       # 用户名
      password: 123456 # 密码
```

##### 2.2 服务配置

```yaml
eureka:
  instance:
    hostname: localhost
  client:
    service-url:
    # 前面加用户名：密码
      defaultZone: http://root:123456@localhost:8761/eureka/
```

##### 2.3 过滤CSRF（跨域请求伪造）

###### 方式一：使 CSRF 忽略 /eureka 的所有请求

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter implements WebMvcConfigurer {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        supper.configure(http)  // 访问eureka 的控制台和/actuator 是可以做安全控制
        http.csrf().ignoringAntMatchers("/eureka/**"); //忽略eureka/**的所有请求
    }
}
```

###### 方式二： 保持密码验证的同时禁用CRSF防御机制

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter implements WebMvcConfigurer {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http // 直接disable 会把安全验证也禁用，
            .csrf().disable().authorizeRequests()
            .anyRequest()
            .authenticated()
            .and()
            .httpBasic();
    }
}
```

