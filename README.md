# efewfew
![karakal.logo](http://www.karakal.com.cn/public/css/img/logo.jpg "karakal.logo")
**指纹消息队列**

----------

> 本模块功能提供中音接口程序与指纹客户端之间异步信息交互。
> 
> 


**调用流程**

---------


把使用了该项目的案例放在这里。可以放APK下载链接，或者简单放几张截图。  
（示例一开始就放出来，方便浏览者一眼就看出是不是想找的东西）

---------

**部署情况**



----------

**运行环境**


    操作系统： linux 
    开发语言： java
    运行容器： 无
    数据库： MySQL


----------

**使用框架**

   spring

----------

**接口文档**

[时光网抓取分析](docs/时光网/抓取分析.md)
[QQ音乐](docs/QQ音乐/抓取分析.md)

----------

**原理说明（可选）**

该项目定制scrapy框架实现，原理参见[Scrapy架构概览](http://scrapy-chs.readthedocs.io/zh_CN/latest/topics/architecture.html)

整体架构如下：
![scrapy架构](docs/scrapy_architecture_02.png)

在该项目中对部分组件做了定制：
增加了以下downloadmiddlewares：
1. ProxyPoolMiddleware:实现为请求增加动态代理，依赖github上一个开源的动态代理池项目[ProxyPool](https://github.com/Greyh4t/ProxyPool)，该中间件会读取如下配置：
```
PROXY_POOL_URL: 获取代理地址的代理池地址
PROXY_POOL_DEFAULT_OPEN： 对每个spider是否开启动态代理的支持，默认为False;也可在spider中设置proxy_pool_open属性单独指定该spider是否使用动态代理
```

2.RefererMiddleware：实现对某个请求的header中增加referer值，该中间件会读取如下配置：
```
REFERER:对每个spider使用的referer值；也可在spider中设置referer属性单独指定该spider使用的referer
```

3.RandomUserAgentMiddleware：实现对某个请求随机设置一个User-Agent的haader值；可在spider中设置user_agent属性指定该spider使用的User-Agent值

定制了调度器，包括以下组件：
1. 队列实现了redis与kafka两种，且redis支持三种类型的队列：先进先出队列、后进先出队列、优先级队列。且调度器增加读取spider的close_if_idle属性，表示当没有任务时是否停止spider，若为False则不会停止，一直等待任务的到来  
2. 过滤器使用redis实现，且过滤使用的url的md5值   
3. 任务序列化有两种，一种为将request对象完全序列化进队列；另一种为请求的必要参数，这种实现要依赖spider的url_template类属性，如此实现是因为大部分url很长，若使用redis队列则比较消耗内存，所以只要必要参数，这种实现会丢失一下特性，比如redirect功能   

增加了一下pipline：
1. MongoPipeline：实现结构化数据的保存，该pipline会处理包含site、type、subtype(默认为stable)、data四个字段的Item对像。site、type、subtype这三个字段会拿来生成mongodb的集合名，其取值为item_fied.py中对应类属性值。data为抓取解析到的dict类型的结构化数据，其中的key应为item_fied.py中对应类属性值。
  该pipeline会读取spider的dynamic属性，若dynamic为True，则会将data中的值作为动态数据保存，所以每个字段会对应一个表，并加入时间插入动态数据表中。然后，将data中的值更新到对应的stable表中


增加任务自动调度功能，该组件依赖APScheduler模块，实现原理为：   
扫描所有spider类，然后检查每个spider类中是否有schedule类方法与schedule_cron类属性。schedule类方法会被自动调度组件自动定时执行，且组件会传入方法的依赖，比如mongodb链接等，且该方法返回包含Request的可迭代对象。schedule_cron类属性为一个字典类型，用于配置schedule类方法何时执行，若没配置值为每天零点执行(如何配置参见APScheduler的API)。

增加自动部署组件：
1. 服务端(api.py)：使用Flask实现的api：包括启动所有可运行的spider(spider类中将is_run设置为True表示该spider可运行)进程、停止所有spider进程、从新部署所有spider接口。其中重新部署实现的步骤如下
```
接收上传的zip压缩包
删除mzk_spiders
解压上传的压缩包
删除压缩包
重启所有spider
```
2. 客户端(deploy.py):一个脚本文件，可配置servers这个list变量来设置部署到那些服务器，服务器需运行服务端的api.py.实现步骤如下：
```
修改配置文件为正式环境配置
压缩文件夹
上传压缩包
修改配置文件为本地配置文件
删除压缩文件
```

----------

**下载安装**

安装ApacheMq环境以及相关依赖

----------

**使用方法**

运行项目中的mq_main.java即可

----------

**注意事项**

spring版本使用spring3.0以上


----------

**TODO（可选）**

接下来的开发/维护计划。

----------

**License**
遵守的协议
