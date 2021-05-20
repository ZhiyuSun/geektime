# apache shiro-介绍

shiro securityManager
所有的操作都可以借助这个来

realm

授权也叫访问控制

主体subject
资源
权限

apache shiro权限拦截

preHandle
postHandle

shiro会话管理

权限缓存Cache Manager

AuthRealm extends AuthorizingRealm

授权AuthorizationInfo
认证AuthenticationInfo

授权，组装SimpleAuthorizationInfo

ShiroConfiguration
shiro配置类
增加了两个方法：AuthRealm，SecurityManager
ShiroFilterFactoryBean
AuthorizationAttributeSourceAdvisor
DefaultAdvisorAutoProxyCreator

写接口验证

shiroFilterFactoryBean里面配各种权限

cachemanager

shiro的配置比spring security简单，学习成本低，只要在filter里配置，