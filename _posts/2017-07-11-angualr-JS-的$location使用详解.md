---
layout: post
title: "angualr JS 的$location使用详解"
date: 2017-07-11
description: "angualr JS 的$location使用详解"
tag: angularJS
---

## 前言

今天被坑了一下午，主要是使用angular JS 的 获取地址栏的参数的时候出现了问题，怎么都获取不到，上网找解决方案，各种解决方案也找了还是最终发现是自己的 config 配置里面注释了一行，奇怪的是IDEA 还没有报错，在此对 angular JS 中 $location 的使用方法进行详细的记录，以后避免踩坑，希望能看到此文的朋友能有所帮助。

### 配置 config

**这一步非常的重要，如果不配置这一步会获取不到参数。**

```js
app.config(['$locationProvider',function($locationProvider) {
            $locationProvider.html5Mode({
                enabled : true, 
                requireBase : true
            });
} ]);
```

对上面两个参数进行说明：  

- `enabled: true` ：设置为html5Mode(模式)，当为false时为Hashbang模式 
- `requireBase : true` ：是否需要加入base标签，设置为true时，需在html的head配置`<base href="" />`标签。在这个标签里面写你的项目路径，比如我使用maven 构建的使用 Tomcat 插件的项目路径是：

```xml
<!-- 配置tomcat插件 -->
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <configuration>
        <port>9102</port>
        <path>/</path>
    </configuration>
</plugin>
```

那么我的`<base href="/" />` 这么配就可以正确的解析了。

#### 下面给出简化版本的配置，一般情况下使用简化版本就够了

在页面配置位置提供者和base标签：

```javascript
<script type="text/javascript">
/** 配置位置提供者 */
app.config(function($locationProvider) {
    $locationProvider.html5Mode(true);
});
</script>
```

**简化版默认需要引入base 标签，否则会解析错误。**

### 二、基本用法

我用这段段 URL 作为测试 `http://127.0.0.1:9102/pages/test.html?name=allenlei` ，配置和上面的是一样的。结果如下，注释部分为结果。  

```javascript
1.获取绝对路径 
$location.absUrl();  
// http://127.0.0.1:9102/shop/pages/test.html?name=allenlei

2.获取主机 
$location.host(); 
// 127.0.0.1 

3.获取端口号 
$location.port(); 
// 9102 

4.获取文本传输协议 
$location.protocol(); 
// http 

5. 获取url参数 
$location.search().name 或者 $location.search()['name'] 
// allenlei 

6.获取url 
$location.url() 
// pages/test.html?name=allenlei
```

以上就是 angular JS  `$location` 的使用方法了。测试版本为 1.6.9 ，如果有任何疑问欢迎联系我，我的邮箱和微信在 [关于我](https://allenleic.github.io/about/)  可以看到。

<br/>

**原创声明**  

转载请注明: [雷聪的博客](https://allenleic.github.io) » [{{ page.title }}](https://allenleic.github.io/2017/07/{{ page.title }})
