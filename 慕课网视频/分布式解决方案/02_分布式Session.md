# 分布式Session

## 传统session

session是由tomcat来管理的

第一次访问，有一个set-cookie的操作

再刷新，就有了cookie

attributes里可以看到cookie的内容

sessionid就是存放在cookie里，session离不开cookie

cookie一定要保存在域名下

session是由tomcat管理，两个应用程序，不能拿到同一个session

## spring-session

session存redis里

## jwt

既不用redis，也不用分布式session

jwt:token由加密算法计算得到

json web token
jwt.io

github上的组件，java-jwt 

token里的内容可以被解析，但是不能被篡改

敏感的内容不能放在jwt里面去

可设置过期时间

## 拦截器

token统一转换

写一个拦截器，request.setAttribute

WebMvcConfig添加拦截器
@RequestAttribute uuid

## OAuth2

spring-security-oauth2

授权数据给第三方
权限管理






