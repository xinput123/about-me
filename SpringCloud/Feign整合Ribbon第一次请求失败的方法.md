在Spring Cloud中，Feign和Ribbon在整合了Hystrix后，可能会出现首次调用失败。

### 1、造成该问题的原因？
Hystrix默认的超时时间是1秒，如果超过这个时间尚未响应，将会进入fallback代码。而首次请求往往会比较慢（**因为Spring的懒加载机制，要实例化一些类**），这个响应时间可能就大于1秒了。知道原因后，我们来总结一下解决放你。
解决方案有三种，以feign为例。
> - 1、方法一
> hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 5000 该配置是让Hystrix的超时时间改为5秒

> - 2、方法二
> hystrix.command.default.execution.timeout.enabled: false 该配置，用于禁用Hystrix的超时时间

> - 3、方法三
> feign.hystrix.enabled: false 该配置，用于索性禁用feign的hystrix。该做法除非一些特殊场景，不推荐使用。