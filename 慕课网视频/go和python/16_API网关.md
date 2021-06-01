# API网关

1. 动态路由
2. 限流管理

所以需要API网关

kong对nginx进行了进一步的封装

服务的路由
- 动态路由
- 负载均衡
服务发现
限流，熔断，降级
流量管理，黑白名单，反爬虫策略

主流API网关对比

API网关对性能要求是很高的

goku-api-gateway

kong

可以在那些上游服务之前，额外地实现一些功能
基于OpenRestr(Nginx+Lua模块)编写的高可用、易扩展

可以在konga上面配置路由、服务等等

kong——routes——services——upstreams——订单服务
一个service可以有多个route
upstream可以负载均衡，健康检查

services对接consul

kong plugin插件
里面有各种认证

