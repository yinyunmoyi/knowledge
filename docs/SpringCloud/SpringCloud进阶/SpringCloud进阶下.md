# Hystrix

## 环境搭建

基于之前的Feign工程，为了能达到服务降级的效果，需要声明一个 ProviderFeignClient 的实现类，来实现服务调用出错时的降级方法：

```java
@Component
public class ProviderHystrixFallback implements ProviderFeignClient {
    
    @Override
    public String getInfo() {
        return "hystrix fallback getInfo ......";
    }
}
```

在 ProviderFeignClient 接口中配置该降级类：

~~~java
@FeignClient(value = "eureka-client", fallback = ProviderHystrixFallback.class)
public interface ProviderFeignClient
~~~

之后 application.properties 中加入配置：

~~~
feign.hystrix.enabled=true
~~~

最后，主启动类上标注 @EnableHystrix 或 @EnableCircuitBreaker 注解即可。（这俩都一样）

启动 eureka-server 与 eureka-client ，随后启动刚改造好的 hystrix-consumer 工程。浏览器第一次发送请求，可以正常响应；关闭 eureka-client 服务，再次发起请求，一秒延迟后浏览器响应 hystrix fallback getInfo ...... 信息，证明服务降级成功。

## @EnableHystrix

