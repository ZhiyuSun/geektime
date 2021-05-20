# spring security

安全拦截器
- 认证管理器
- 访问决策管理器
- 运行身份管理器

basic认证
- 明文，无状态

digest
X.509
LDAP
Form

ThreadLocal，线程内共享，线程间互斥
ThreadLocalMap 存储线程本地map

权限拦截
各种拦截过滤器

UserDetails
authorities

rbac模型

权限缓存

CachingUserDetailsService
this.userCache

EhCache

实际项目中，会尝试使用更多的cache

AcessDecisionVoter

RoleVoter

三个决策投票器
AffirmativeBased
ConsensusBased
Unaniousbased

身份认证
权限拦截
自定义决策

环境搭建及使用

常见case实现
- 只要能登录
    - inMemoryAuthentication
- 有指定的角色，每个角色有指定的权限
PreAuthorize("hasRole("ROLE_ADMIN")")

MyUserService implments UserDetailsService

PostAuthorize
PreFilter：
PostFilter
里面可以写各种各样的表达式

Spring Security
- 优点
- 缺点：
    - 配置文件多，角色被“编码”到配置文件
