## 一、简介

当微服务中的某个子服务，发生异常服务器宕机，其他服务在进行时不能正常访问而一直占用资源导致正常的服务也发生资源不能释放而崩溃，这时为了不造成整个微服务群瘫痪，进行的保护机制 就叫做**熔断**，是一种**降级**策略

> 熔断的目的：保护微服务集群

## 二、作用

- 对第三方访问的延迟和故障进行保护和控制
- 防止复杂分布式系统中的雪崩效应（级联故障）
- 快速失败，快速恢复
- 回退，尽可能优雅的降级
- 启用近实时监控、报警和操作控制

## 三、核心概念

### 3.1 熔断目的

应对雪崩效应，快速失败，快速恢复

### 3.2 降级目的

保证整体系统的高可用性

## 四、实例

### 4.1 基于Hytrix

- pom.xml

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  </dependency>
  ```

- application

  ```java
  @EnableHystrix // 开启熔断
  @SpringBootApplication
  public class HystrixApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(HystrixApplication.class, args);
      }
  
  }
  ```

#### 4.1.1 熔断触发降级

```java
/**
 * Date: 2022-09-16 星期五
 * Time: 9:49
 * Author: Dily_Su
 * Remark:
 */
@RestController
@RequestMapping("/hystrix")
public class HystrixController {


    @Autowired
    private RestTemplate restTemplate;


    /**
     * @param num 参数
     * @return 字符串
     */
    @HystrixCommand(
            commandProperties = {
                    @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
                    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "5"),  // 默认20, 最小产生5次请求
                    @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "5000"), // 熔断时间, 该时间内快速失败
                    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"), // 10s内失败率达到50%触发熔断
            }, // 10s 内最少 5 个请求, 如果失败率大于 50% 则触发熔断
            fallbackMethod = "fallback"
    )  // 服务调用失败时，触发熔断进行降级

    @GetMapping("/test")
    public String test(@RequestParam Integer num) {
        if (num % 2 == 0) {
            return "访问正常";
        }

        List<?> list = restTemplate.getForObject("http://localhost:7070/product/list", List.class);
        assert list != null;
        return list.toString();
    }

    /**
     * 熔断时触发降级
     *
     * @param num 参数
     * @return 字符串
     */
    private String fallback(Integer num) {
        // 降级操作
        return "系统繁忙";
    }
}
```

#### 4.1.2 超时触发降级

```java
/**
 * Date: 2022-09-16 星期五
 * Time: 9:49
 * Author: Dily_Su
 * Remark:
 */
@RestController
@RequestMapping("/hystrix")
public class HystrixController {


    @Autowired
    private RestTemplate restTemplate;

    @HystrixCommand(
            commandProperties = {
                    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "500")
            },
            fallbackMethod = "timeout"
    )
    @GetMapping("/timeout")
    public String timeoutTest(){
        return restTemplate.getForObject("http://localhost:7070/product/list", String.class);
    }

    /**
     * 超时时触发降级
     *
     * @return 字符串
     */
    private String timeout() {
        // 降级操作
        return "请求超时";
    }
}
```

#### 4.1.3 资源隔离触发降级

平台隔离、业务隔离、部署隔离

线程池隔离、信号量隔离

### 4.2 基于OpenFeign

- pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

- application

```JAVA
@SpringBootApplication
@ComponentScan(basePackages = {  
        "com.study.provider.service",  // feign 包目录
        "com.study.hystrix"  // 当前模块启动目录
})
@EnableFeignClients(basePackages = "com.study.provider.service")  // feign 目录
public class HystrixApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixApplication.class, args);
    }

}
```

- HystrixFeignController

```java
/**
 * Date: 2022-09-19 星期一
 * Time: 9:58
 * Author: Dily_Su
 * Remark: Feign 远程调用熔断
 */
@RestController
@RequestMapping("/hystrixFeign")
public class HystrixFeignController {

    @Qualifier("com.study.provider.service.ProductionService")
    @Autowired
    ProductionService productService;  // feign Client 

    @GetMapping("test")
    public String test(){
        return productService.getProduction(1).toString();   // 远程调用
    }

}
```

- feign Client 

```java
/**
 * Date: 2022-06-22 星期三
 * Time: 15:06
 * Author: Dily_Su
 * Remark: openFeign 远程服务
 */
@FeignClient(value = "provider", fallback = ProductionFallback.class)   // fallback 用于熔断
public interface ProductionService {
    @RequestMapping("/product/getProduction")
    Object getProduction(@RequestParam Integer id);
}
```

- ProductionFallback

```java
/**
 * Date: 2022-09-19 星期一
 * Time: 14:36
 * Author: Dily_Su
 * Remark: ProductionService 的熔断降级操作
 */
@Component
public class ProductionFallback implements ProductionService {
    @Override
    public Object getProduction(Integer id) {
        return "请求失败";
    }

    @Override
    public Map createProduction(Object production) {
        return new HashMap<>();
    }

    @Override
    public Object selectByProduct(Object product) {
        return "请求失败";
    }
}
```







