# 实现cas ticket基于redis的集群

### 目的
	克服cas单点故障，将cas认证请求分发到多台cas服务器上，降低负载。

### 实现思路：
	采用统一的ticket存取策略，所有ticket的操作都从中央缓存redis中存取。
	采用session共享，session的存取都从中央缓存redis中存取。

### 前提：
	这里只讲解如何实现cas ticket的共享，关于session的共享请移步：




- <a href="https://github.com/izerui/tomcat-redis-session-manager">https://github.com/izerui/tomcat-redis-session-manager</a>

### 实现步骤：

# 1. 进入 [![](https://jitpack.io/v/izerui/cas-server-integration-redis.svg)](https://jitpack.io/#izerui/cas-server-integration-redis)添加相关依赖

# 2. 替换默认的ticket存取策略

## cas 4.1.x 以前版本

修改文件 WEB-INF\spring-configuration\ticketRegistry.xml
```
<bean id="ticketRegistry" class="org.jasig.cas.ticket.registry.DefaultTicketRegistry" />
```
替换为
```
<bean id="ticketRegistry" class="ren.boot.cas.ticket.registry.RedisTicketRegistry">
    <constructor-arg index="0" ref="redisTemplate" />

    <!-- TGT timeout in seconds -->
    <constructor-arg index="1" value="1800" />

    <!-- ST timeout in seconds -->
    <constructor-arg index="2" value="300" />
</bean>
<!-- redis连接池 -->
<bean id="jedisConnFactory"
      class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
      p:hostName="192.168.1.89"
      p:database="0"
      p:usePool="true"/>
<bean id="redisTemplate" class="org.jasig.cas.ticket.registry.TicketRedisTemplate"
      p:connectionFactory-ref="jedisConnFactory"/>
```
> 注意： 里面的 jedisConnFactory链接信息 修改为自己的连接串，这里选择database 0 为存放cas票据的数据库

## cas 4.2.x :

在spring 上下文中声明如下即可:

```
<!-- 票据保存方式及有效期设置 -->
<alias name="redisTicketRegistry" alias="ticketRegistry" />
<bean id="redisTicketRegistry" class="ren.boot.cas.ticket.registry.RedisTicketRegistry"
    p:client-ref="ticketRedisTemplate"
    p:tgtTimeout="28800"
    p:stTimeout="10"/>
<!-- redis连接池 -->
<bean id="jedisConnFactory"
      class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
      p:hostName="localhost"
      p:database="0"
      p:usePool="true"/>
<bean id="ticketRedisTemplate" class="ren.boot.cas.ticket.registry.TicketRedisTemplate"
      p:connectionFactory-ref="jedisConnFactory"/>
```
