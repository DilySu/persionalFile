# Feign 声明式服务调用

## 一、简介

Sping Cloud 组件中的轻量级 RESTful 的 HTTP 服务客户端，实现了负载均衡和 Rest 调用的开源框架，封装了 Ribbon 和 RestTemplate，实现了 WebService 的面向接口编程，进一步降低了系统的耦合度。

Feign 内置了 Ribbon，用来做客户端负载均衡调用注册中心的服务

Feign 本身不支持 Spring MVC 注解，为了方便使用，Spring Cloud 孵化了 OpenFeign

Feign 是一种声明式、模板化的 HTTP 客户端（仅在消费者服务使用）

Feign 的支持和注解参考 spring.io 官方文档

Feign 使用方式： 使用 Feign 的注解定义接口，调用这个接口就可以调用注册中心的服务

## 二、入门实例

### 服务消费者

#### 1、添加 Feign 依赖

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 2、使用注解声明要调用的接口

```java
// 调用 oa 模块
@FeignClient("oa")
public interface IOAFeign {

    // 调用的 oa 模块的 接口
   @RequestMapping("/oa/meeting/getList")
   public BaseResponse<?> getData(@RequestParam String name);
    
   @RequestMapping("/product/create")
   public Map createProduction(Object production);
}
```

#### 3、调用接口

```java
@Service
public class PermissionService{
    
    // 调用 OA 模块 接口
    @Autowired
    private IOAFeign iogFeign;
    
    public BaseResponse test(){
        // 通过 Feign 调用 OA 模块的接口 
       return ioaFeign.getData("111")
    }
    
    // 对象传参 
    public Map test3(Object production){
        // 通过 Feign 调用 OA 模块的接口 
       return ioaFeign.createProduction(production)
    }
}
```

#### 4、application 启用注解 @EnableFeignClients

```java
@EnableFeignClients
@SpringBootApplication
public class WorkflowApplication {
   public static void main(String[] args) {
      SpringApplication.run(WorkflowApplication.class, args);
   }
}
```

## 三、负载均衡

### 1、全局

pom.xml

```xml
<!--        ribbon 点对点直连 和 局部负载均衡 不可用-->
<dependency>
    <groupId>com.netflix.ribbon</groupId>
    <artifactId>ribbon-loadbalancer</artifactId>
    <version>2.7.18</version>
    <scope>compile</scope>
</dependency>
```

Application.java

```java
// 启动类 或者 配置类（configuration）中注入，这里可选择需要的负载均衡策略进行注入
@Bean
public RandomRule randomRule(){
    return new RandomRule();
}
```

### 2、局部

pom.xml

```xml
<!--        ribbon 点对点直连 和 局部负载均衡，配置文件里写必须用这个-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```

application.yml

```yml
# 负载均衡 局部策略
provider:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

## 四、性能优化

### 1、Gzip 压缩

####  简介

- **介绍：**Gzip 是一种数据格式，采用 deflate 算法压缩数据；gzip 是一种流行的文件压缩算法，应用十分广泛，尤其是在 linux 平台

- **能力：**Gzip 压缩纯文件时，效果十分显著，大约可减少 70% 以上的文件大小

- **作用：**网络数据经过压缩后实际上降低了网络传输的字节数，做明显的好处就是可以加快网页加载的速度，网页加载速度加快的好处不言而喻，除了节省流量，改善用户的浏览体验外，另一个潜在好处就是 Gzip 与 搜索引擎抓取工具有更好的关系，例如 Google 可通过 读取 gzip 文件 比 普通手工抓取更快的索引网页。

- **HTTP 关于 压缩传输的规定**

  > - 客户端向服务器的请求中带有：Accept-Encoding：gzip，deflate 字段，向服务器表示客户端支持的压缩格式（gzip 或 deflate），如果不发该消息头，服务端默认不压缩
  > - 服务端接收到请求头，发现请求头还有 Accept-Encoding 字段，并支持该类型压缩，就会对响应报文进行压缩，并携带 Content-Encoding 消息头，表面响应报文时根据该格式压缩
  > - 客户端接收到响应后，先判断有无Content-Encoding 消息头，如果有，按该格式解压报文，否则按正常报文处理

#### 实例

修改 Application.yml 文件

- #### 全局

  对客户端的请求及 Consumer 对 Provider 的请求和响应都实现 Gzip 压缩

```yml
server:
  port: 6061                  # 端口
  # 全局开启压缩
  compression:
    enabled: true
    # 配置压缩支持的 MIME TYPE
    mime-types: application/json,application/xml,text/html,text/xml,text/plain
```

- #### 局部

  只配置 Consumer 通过 Feign 到 Provider 的请求与响应的 Gzip 压缩

```yml
# 局部 通过 Feign 到 Provider 的请求 进行 Gzip 压缩
feign:
  compression:
    request:
      min-request-size: 512  # 配置压缩数据大小的最小阈值，默认 2048
      mime-types: text/xml,application/xml,application/json  # 配置压缩文件支持的 MIME TYPE
      enabled: true                # 请求是否开启 Gzip 压缩
    response:
      enabled: true                # 响应是否开启 Gzip 压缩
```

### 2、HTTP 连接池

#### 简介

- **原理：**

  > 两个服务器 建立 HTTP 连接 需要 3 次握手、4 次握手，对于比较小的 HTTP 消息来说 开销很大。
  >
  > 采用连接池，可以节省大量握手时间，大大提升吞吐量

- **方案：**

  httpClient

#### 实例

pom.xml	

```xml
  <!--        Feign 使用 httpClient -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

application.yml

```yml
feign:
  httpclient:
    enabled: true    # 开始 httpClient
```

### 3、状态查看

Consumer 服务添加 logback.xml 日志文件，内容如下（logback 日志的输出级别需要是DEBUG级别）

resource 下 logback.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration scan="true" scanPeriod="10 seconds">
    <!--    日志上下文名称-->
    <contextName>my_logback</contextName>
    <!--    name 是变量名称，value 是变量定义的值，通过定义的值会被插入到 Logger 中，定义变量后 可用${}来使用变量-->
    <property name="log.path" value="${catalina.base}/consumer-eureka-feign/logs"/>
    <!--    彩色日志-->
    <!--    彩色日志依赖的渲染类-->
    <conversionRule conversionWord="clr"
                    converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex"
                    converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <!--    彩色日志格式-->
<!--    <property name="CONSOLE_LOG_PATTERN"-->
<!--              value="${CONSOLE_LOG_PATTERN}：-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint}"/>-->
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN}：-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${CONSOLE_LOG_PATTERN:-%wEx}}" />
    <!--    文件日志格式-->
    <property name="FAIL_LOG_PATTERN"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>

    <!--    输出到控制台-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!--        此日志 appender 是为开发使用, 只配置最低级别, 控制台输出的日志级别是大于或等于次级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>DEBUG</level>
        </filter>
        <encoder>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <!--    输出到文件-->
    <!--    时间滚动输出 level 为 debug 的日志-->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--       记录日志文件路径及文件名-->
        <file>${log.path}/log_debug.log</file>
        <!--        日志文件输出格式-->
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!--        日志归档-->
        <fileNamePattern>${log.path}/debug/log-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <timeBaseFileNamingAndTriggerPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            <maxFileSize>100MB</maxFileSize>
        </timeBaseFileNamingAndTriggerPolicy>
        <!--        日志保留天数-->
        <maxHistory>15</maxHistory>
    </rollingPolicy>
    <!--    此日志只记录 debug 级别-->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>DEBUG</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>

    <!--   时间滚动输出 level 为 info 日志 到文件-->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--       记录日志文件路径及文件名-->
        <file>${log.path}/log_info.log</file>
        <!--        日志文件输出格式-->
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <!--    日志记录器的滚动策略，按日期，按大小记录-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!--        日志归档-->
        <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <timeBaseFileNamingAndTriggerPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            <maxFileSize>100MB</maxFileSize>
        </timeBaseFileNamingAndTriggerPolicy>
        <!--        日志保留天数-->
        <maxHistory>15</maxHistory>
    </rollingPolicy>
    <!--    此日志只记录 info 级别-->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>INFO</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>

    <!--   时间滚动输出 level 为 warn 日志 到文件-->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--       记录日志文件路径及文件名-->
        <file>${log.path}/log_warn.log</file>
        <!--        日志文件输出格式-->
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <!--    日志记录器的滚动策略，按日期，按大小记录-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!--        日志归档-->
        <fileNamePattern>${log.path}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <timeBaseFileNamingAndTriggerPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            <maxFileSize>100MB</maxFileSize>
        </timeBaseFileNamingAndTriggerPolicy>
        <!--        日志保留天数-->
        <maxHistory>15</maxHistory>
    </rollingPolicy>
    <!--    此日志只记录 info 级别-->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>WARN</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>

    <!--   时间滚动输出 level 为 error 日志 到文件-->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--       记录日志文件路径及文件名-->
        <file>${log.path}/log_error.log</file>
        <!--        日志文件输出格式-->
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <!--    日志记录器的滚动策略，按日期，按大小记录-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!--        日志归档-->
        <fileNamePattern>${log.path}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <timeBaseFileNamingAndTriggerPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            <maxFileSize>100MB</maxFileSize>
        </timeBaseFileNamingAndTriggerPolicy>
        <!--        日志保留天数-->
        <maxHistory>15</maxHistory>
    </rollingPolicy>
    <!--    此日志只记录 info 级别-->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>ERROR</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>

    <!--    对类似于 com.example.Logback 开头的 Logger, 输出级别设置为 warn, 并且只输出到控制台-->
    <!--    这个 Logger 没有指定 appender, 它会继承 root 节点中定义的 appender-->
    <!--    <Logger name="com.example.Logback" level="warn"></Logger>-->

    <!--    通过 LoggerFactory.getLogger("myLog") 可以获取到这个 Logger-->
    <!--    由于这个 Logger 自动继承了 root 的 appender, root 中已经有了 stdout 的 appender了 自己这边引入了 stdout 的 appender-->
    <!--    如果没有设置 additivity=“false”, 会导致一条日志在控制台输出两次的情况-->
    <!--    additivity 表示不要使用 rootLogger 配置的 appender 进行输出-->
    <logger name="myLog" level="INFO" adddivity="false">
        <appender-ref ref="CONSOLE"/>
    </logger>
    <!--日志输出级别及方式-->
    <root level="DEBUG">
        <appender-re ref="CONSOLE"/>
        <appender-re ref="DEBUG_FILE"/>
        <appender-re ref="INFO_FILE"/>
        <appender-re ref="WARN_FILE"/>
        <appender-re ref="ERROR_FILE"/>
    </root>
</configuration>
```

#### 全局

启动类 或者 配置类中加入

```java
// feign 中的 Logger
@Bean
public Logger.Level getLog(){
    return Logger.Level.FULL;
}
```

####  局部

application.xml 中

```yml
#  日志局部定义
feign:
  client:
    config:
      provider: # 需要调用的服务
        loggerLevel: FULL
```

### 4、请求超时

Feign 的负载均衡底层使用 Ribbon，这里的请求超时其实就是配置 Ribbon

分布式系统中，服务压力比较大的情况下，可能处理服务的过程需要花费一定的时间，而默认情况下的请求超时的配置是 1s，所以需要调整该配置，延长请求超时时间。

#### 全局

```yml
# 配置请求超时时间
feign:
  client:
    config:
      # 全局 配置请求超时时间
      default:
        connectTimeout: 1000  # 请求连接超时时间 默认为 1s
        readTimeout: 1000     # 请求处理的超时时间
```

#### 局部

```yml
feign:
  client:
    config:
      # 局部 配置请求超时时间
      provider: # 服务名
        OkToRetryOnAllOperations: true   # 对所有请求都进行重试
        MaxAutoRetries: 2                # 对当前实例的重复次数
        MaxAutoRetriesNextServer: 0      # 切换实例的重复次数
        ConnectTimeOut: 3000             # 请求连接超时时间 默认为 1s
        ReadTimeOut: 3000                # 请求处理的超时时间# 局部 配置 请求请求超时
```