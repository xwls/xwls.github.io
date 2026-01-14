+++
title = 'SpringBoot 整合 Shiro 实现认证和授权'
date = 2019-12-27T13:47:04+08:00
categories = ['Java']
tags = ['java', 'spring', 'springboot', 'shiro']
summary = 'Shiro 是 Apache 软件基金会下有名的安全框架，本文记录在 SpringBoot 中如何使用 Shiro 实现用户认证及授权。'
+++

## 用户认证和授权

自定义 Realm 完成用户的认证和授权，这里使用模拟数据，规定如下：

* 用户名输入"unknown"，表示用户名不存在
* 用户 tom，密码 123456
* 用户 jerry，密码 123456

```java
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;

/**
 * 自定义 Realm 继承自 AuthenticatingRealm 完成用户认证和授权
 */
public class MyShiroRealm extends AuthorizingRealm {
    @Override
    //此方法用来完成用户认证
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //1. 将 token 转换为 UsernamePasswordToken
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        //2. 从 token 获取用户名
        String username = token.getUsername();
        //3. 从数据库根据用户名查询用户信息
        System.out.println("从数据库查询、"" + username + "\"用户。");
        //4. 根据查询结果判断用户状态
        if ("unknown".equals(username)) {
            throw new UnknownAccountException("账号不存在");
        }
        //盐值加密
//        String password = "123456";
        //5. 进行认证
        //1). principal：当前登录的用户名或者该用户名对应的实体对象
        String principal = username;
        //2). credentials：加密之后的密码
        String credentials = "";
        if ("tom".equals(username)) {
            credentials = "a7ffa5d8b1b4f5f1e8492623147bccf0";
        } else if ("jerry".equals(username)) {
            credentials = "6b2244d0a6fca5f5dc590437d3ca6781";
        }
        //3). 当前使用的 Realm 的名字，可以直接调用 getName 方法获取
        String realmName = getName();
        //credentials：加密之后的密码
        //credentialsSalt：盐值，需要保证盐值唯一
        ByteSource credentialsSalt = ByteSource.Util.bytes(username);
        return new SimpleAuthenticationInfo(principal, credentials, credentialsSalt, realmName);
    }

    //完成授权的方法
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("principalCollection = " + principalCollection);
        //1. 获取当前登录的用户
        Object username = principalCollection.getPrimaryPrincipal();
        //2. 获取当前登录用户的角色和权限
        //1). 第一种方式：去数据库查询
        //2). 第二种方式：
        //          认证成功之后，将用户的角色和权限保存在用户的实体对象中，每次授权时，从实体对象的属性获取
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        if ("tom".equals(username)) {
            simpleAuthorizationInfo.addStringPermission("book:link");
            simpleAuthorizationInfo.addStringPermission("book:save");
            simpleAuthorizationInfo.addStringPermission("book:remove");
        } else if ("jerry".equals(username)) {
            simpleAuthorizationInfo.addStringPermission("student:link");
            simpleAuthorizationInfo.addStringPermission("student:remove");
            simpleAuthorizationInfo.addStringPermission("student:edit");
        }
        //3. 构建 AuthorizationInfo 对象并返回
        return simpleAuthorizationInfo;
    }
}
```

## Shiro 相关的配置

```java
import at.pollux.thymeleaf.shiro.dialect.ShiroDialect;
import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.codec.Base64;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.CookieRememberMeManager;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;
import java.util.Map;

@Configuration
public class ShiroConfig {

    //配置密码加密方式，这里是有 MD5 盐值加密 10 次
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher() {
        //构造方法传入加密方式
        HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher("md5");
        //设置加密次数
        credentialsMatcher.setHashIterations(10);
        return credentialsMatcher;
    }

    //配置 Realm
    @Bean
    public MyShiroRealm shiroRealm(HashedCredentialsMatcher hashedCredentialsMatcher) {
        MyShiroRealm myShiroRealm = new MyShiroRealm();
        myShiroRealm.setCredentialsMatcher(hashedCredentialsMatcher);
        return myShiroRealm;
    }

    //配置 SecurityManager
    @Bean
    public SecurityManager securityManager(MyShiroRealm shiroRealm, CookieRememberMeManager cookieRememberMeManager) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRememberMeManager(cookieRememberMeManager);
        securityManager.setRealm(shiroRealm);
        return securityManager;
    }

    //配置 ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
        factoryBean.setSecurityManager(securityManager);
        factoryBean.setLoginUrl("/login");
        factoryBean.setSuccessUrl("/index");
        factoryBean.setUnauthorizedUrl("/403");
        Map<String, String> map = new LinkedHashMap<>();
        //anon：可以匿名访问，不需要认证
        map.put("/css/**", "anon");
        map.put("/img/**", "anon");
        map.put("/login", "anon");
        //注销，退出登录
        map.put("/logout", "logout");
        //authc：需要认证才能方法
        map.put("/", "authc");
//        map.put("/book","perms[book]");
//        map.put("/student","perms[student]");
        map.put("/index", "user");//记住我登录可以访问
        map.put("/**", "authc");
        factoryBean.setFilterChainDefinitionMap(map);
        return factoryBean;
    }

    //使用注解校验权限需要配置
    @Bean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }

    //使用注解校验权限需要配置
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }

    //配置 Thymeleaf 整合 Shiro，可以使用 shiro:xx 标签
    @Bean
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }

    //实现记住我功能
    @Bean
    public CookieRememberMeManager cookieRememberMeManager() {
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        //设置 cookie 的有效期为 1 个月
        cookieRememberMeManager.getCookie().setMaxAge(60 * 60 * 24 * 30);
        //设置加密密钥，密码可以自行替换，生成方式下面有介绍
        cookieRememberMeManager.setCipherKey(Base64.decode("Jt3C93kMR9D5e8QzwfsiMw=="));
        return cookieRememberMeManager;
    }

}
```

## 记住我功能密钥生成方式

```java
KeyGenerator aes = KeyGenerator.getInstance("AES");
SecretKey secretKey = aes.generateKey();
byte[] encoded = secretKey.getEncoded();
String s = Base64.encodeToString(encoded);
System.out.println(s);