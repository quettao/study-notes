### Hypert框架

**带来的变化**

​	从php-fpm 转向 php-cli

​	从同步阻塞转向 异步非阻塞

​	从面相 ‘请求’ 编程转向 csp 协程编程

​	从http 到 http http 1/2 tcp udp websocket 等

​	从低性能到超高性能

​	编程模式的改变也导致PHP原来的生态不能完美复用

**丰富的组件 - 服务治理**

​	基于 JSON RPC 2.0 或 gRPC 的RPC Server/Client

​	Apollo /ETCD /阿里云 ACM配置中心

​	支持以consul作为注册中心

​	基于令牌桶算法的分布式限流器

​	基于注解与AOP的熔断器和服务降级

​	Open Tracing 调用链追踪(接入了Zipkin Jaeger)

​	接入了Swoole Tracker 和 Prometheus 进行服务监控

​	Snowflake 全局ID生成器

**优势**

​	高性能、CSP协程编程

​	laravel 用户无缝转移

​	协程组件库丰富且完备

​	DI 、注解、AOP

​	可单体、可微服务、可API、可视图、可注解、可配置

​	全面实现标准的PSR协议

