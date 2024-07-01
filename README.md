### 伙伴匹配系统项目



#### 1. 项目简介



##### 1.1 介绍



帮助大家寻找志同道合的小伙伴的移动端 `H5` 网页



前端项目地址：https://github.com/Ding-Jiaxiong/xiongpao_frontend

后端项目地址：https://github.com/Ding-Jiaxiong/xiongpao_backend



##### 1.2 技术栈



###### 1.2.1 前端



1. Vue3开发框架（提高页面开发的效率）
2. Vant Ul(基于Vue的移动端组件库)(React版Zent)
3. Vite2(打包工具，快！)
4. Nginx来单机部署



###### 1.2.2 后端



1. Java编程语言+SpringBoot框架
2. SpringMVC+MyBatis+MyBatis Plus(提高开发效率)
3. MySQL数据库
4. Redis缓存
5. Swagger+Knife4接口文档





#### 2. 项目开发流程



##### 2.1 需求分析



1. 用户去添加标签，标签的分类（要有哪些标签、怎么把标签进行分类）学习方向jva/c++,工作/大学
2. 主动搜索：允许用户根据标签去搜索其他用户
    - Redis缓存
3. 组队
    - 创建队伍
    - 加入队伍
    - 根据标签查询队伍
    - 邀请其他人
4. 允许用户去修改标签
5. 推荐
    - 相似度计算算法+本地实时计算



##### 2.2 前端项目初始化



**脚手架初始化项目**



- Vue CLI：https://cli.vuejs.org/zh/



- Vite 脚手架：https://vitejs.cn/guide/



`yarn create vite`



![image-20240629095042460](./assets/image-20240629095042460.png)



我看鱼皮的版本是`2.9.2`



![image-20240629095438798](./assets/image-20240629095438798.png)



![image-20240629095601671](./assets/image-20240629095601671.png)



换成`npm`



![image-20240629095712854](./assets/image-20240629095712854.png)



ok



![image-20240629095758317](./assets/image-20240629095758317.png)



install 一下



![image-20240629100003597](./assets/image-20240629100003597.png)



启动一下

![image-20240629100055499](./assets/image-20240629100055499.png)



![image-20240629100110132](./assets/image-20240629100110132.png)



**整合组件库Vant**



安装 `Vant`



`3` 版本：https://vant.pro/vant/v3/



![image-20240629100253380](./assets/image-20240629100253380.png)



按需引入：`npm i vite-plugin-style-import@1.4.1 -D`



![image-20240629100442717](./assets/image-20240629100442717.png)



配置插件



![image-20240629100714488](./assets/image-20240629100714488.png)



安装vant：这里要指定一下版本



`npm install vant@3.4.8`

![image-20240629101537209](./assets/image-20240629101537209.png)



引入组件



![image-20240629101101502](./assets/image-20240629101101502.png)



试试



![image-20240629101600834](./assets/image-20240629101600834.png)



OK，可用



##### 2.3 前端页面设计



从上到下依次为：

1. 导航条：展示当前页面名称

2. 主页搜索框：点击跳转到搜索页=>搜索结果页（标签筛选页）

3. 内容展示

4. tab栏：

    - 主页（推荐页+广告）

        - 搜索框

        - banner
        - 推荐信息流

    - 队伍顶

    - 用户页



##### 2.4 前端通用布局开发



![image-20240629112116742](./assets/image-20240629112116742.png)



##### 2.5 数据库表设计



###### 2.5.1 标签表



建议用标签，而不是用分类，更灵活。



**标签分类**



- 性别：男、女
- 方向：Java、C++、Go、前端
- 正在学：Spring
- 目标：考研、春招、秋招、社招、考公、竞赛（蓝桥杯）、转行、跳槽
- 段位：初级、中级、高级、王者
- 身份：小学、初中、高中、大一、大二、大三、大四、学生、待业、已就业、研一、研二、研三
- 状态：乐观、有点丧、一般、单身、已婚、有对象
- 还可以支持用户自己定义标签。



###### 表设计



- id int 主键
- 标签名varchar非空（必须唯一，唯一索引）
- 上传标签的用户 userld int(如果要根据userld查已上传标签的话，最好加上，普通索引)
- 父标签 id , parentld, int (分类)
- 是否为父标签 isParent , tinyint (0 - 不是父标签、1 - 父标签)
- 创建时间 createTime , datetime
- 更新时间 updateTime , datetime
- 是否删除 isDelete , tinyint (0、1)



###### 2.5.2 用户表



主要是怎么给用户表补充标签?

常见方案2 种：

- 直接用户表补充tags 字段，json 字符串
- 加一个关联表，记录用户和标签的关系



> 企业大项目开发中尽量减少关联查询，很影响扩展性，而且会影响查询性能



此处选择第一种，可以用缓存来提高查询效率。



这里复用了之前的用户中心项目，数据库也复用了



其实标签表就没用了



```sql
-- 用户表
create table user
(
    username     varchar(256) null comment '用户昵称',
    id           bigint auto_increment comment 'id'
        primary key,
    userAccount  varchar(256) null comment '账号',
    avatarUrl    varchar(1024) null comment '用户头像',
    gender       tinyint null comment '性别',
    userPassword varchar(512)       not null comment '密码',
    phone        varchar(128) null comment '电话',
    email        varchar(512) null comment '邮箱',
    userStatus   int      default 0 not null comment '状态 0 - 正常',
    createTime   datetime default CURRENT_TIMESTAMP null comment '创建时间',
    updateTime   datetime default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP,
    isDelete     tinyint  default 0 not null comment '是否删除',
    userRole     int      default 0 not null comment '用户角色 0 - 普通用户 1 - 管理员',
    planetCode   varchar(512) null comment '星球编号',
    tags         varchar(1024) null comment '标签 json 列表'
) comment '用户';
```



![image-20240629114139827](./assets/image-20240629114139827.png)



![image-20240629114239870](./assets/image-20240629114239870.png)



![image-20240629114722498](./assets/image-20240629114722498.png)



其他就没啥改的了



##### 2.6 标签搜索用户功能



###### 2.6.1 后端接口开发



按标签搜索用户：

1. 允许用户传入多个标签，多个标签者都存在才搜索出来`and`。
2. 允许用户传入多个标签，有任何一个标签存在就肩搜索出来`or`。



2种实现方式：

1. SQL查询：实现简单，可以通过拆分查询进一步优化
2. 内存查询：灵活，可以通过并发进一步优化



**解析JSON 字符串**



- 序列化：java对象转成json
- 反序列化：把json转为java对象



实现类：

![image-20240629120644315](./assets/image-20240629120644315.png)



![image-20240629122116601](./assets/image-20240629122116601.png)



这是SQL 的方式，下面用内存过滤的方式



用GSON 库：

```xml
        <!--    GSON库    -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.9</version>
        </dependency>
```



![image-20240629122818545](./assets/image-20240629122818545.png)



这样也能搜出来。



###### 2.6.2 接口性能分析



建议通过实际测试来分析哪种查询比较快，数据量大的时候验证效果更明显。



###### 2.6.3 搜索接口调试



改了个为空的`Bug`



###### 2.6.4 前端路由整合







之前就是写死判断，现在用Vue router：https://router.vuejs.org/zh/

安装：`npm install vue-router@4`



Vue-Router其实就是帮助你根据不同的url来展示不同的页面（组件），不用自己写`if` / `else`。

路由配置影响整个项目，所以建议单独用`config`目录、单独的配置文件去集中定义和管理。



![image-20240629123816386](./assets/image-20240629123816386.png)



引用并使用



![image-20240629124742362](./assets/image-20240629124742362.png)



###### 2.6.5 前端页面开发



搜索页面



![image-20240629142641434](./assets/image-20240629142641434.png)



个人页



![image-20240629145436237](./assets/image-20240629145436237.png)



编辑页



![image-20240629150623625](./assets/image-20240629150623625.png)



###### 2.6.6 后端Knife4j接口文档



什么是接口文档？**写接口信息的文档**。

每个接口的信息包括：

- 请求参数
- 响应参数
- 错误码
- 接口地址
- 接口名称
- 请求类型
- 请求格式
- 备注



谁用接口文档？

答：一般是后端或者负责人来提供，后端和前端都要使用。



为什么需要接口文档？

- 有个书面内容（背书或者归档），便于大家参考和查阅，便于沉淀和维护，拒绝口口相传
- 接口文档便于前端和后端开发对接，前后端联调的介质。后端=>接口文档<=前端
- 好的接口文档支特在线调试、在线测试，可以作为工具提高我们的开发测试效率



怎么放接口文档？

- 手写：比如腾讯文档、Markdown笔记
- 自动化接口文档生成：自动根据项目代码生成完整的文档或在线调试的网页。Swagger、Postman(侧重接口管理)（国外）；apifox、apipost、eolink(国产)



使用Swagger：【其实是直接上了Knife4j：在swagger 基础上做了增强】

添加依赖



```xml
        <!--    knife4j    -->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>2.0.7</version>
        </dependency>

```



配置类：



![image-20240629152715998](./assets/image-20240629152715998.png)



启动看看，访问`doc.html`

![image-20240629152850045](./assets/image-20240629152850045.png)



没问题



![image-20240629152947512](./assets/image-20240629152947512.png)



能用



##### 2.7 扩展知识



**用户信息导入及同步**

1. 把所有星球用户的信息导入
2. 把写了自我介绍的同学的标签信息导入



抓取网页内容FeHelper 浏览器插件



![image-20240629153245264](./assets/image-20240629153245264.png)



![image-20240629153341210](./assets/image-20240629153341210.png)



先把依赖装了



```xml
<!--    处理表格    -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>3.1.0</version>
</dependency>
```



![image-20240629161437700](./assets/image-20240629161437700.png)





##### 2.8 用户搜索页



###### 2.8.1 前端开发



搜索结果：



![image-20240629162647033](./assets/image-20240629162647033.png)



用户搜索结果列表展示：



![image-20240629163545877](./assets/image-20240629163545877.png)





###### 2.8.2 后端开发



![image-20240629163812026](./assets/image-20240629163812026.png)



###### 2.8.3 前后端联调



换了个banner

![image-20240629164029921](./assets/image-20240629164029921.png)



![image-20240629164045254](./assets/image-20240629164045254.png)



接口文档测试下接口



![image-20240629164317023](./assets/image-20240629164317023.png)



前端对接



用的axios：`npm install axios`



![image-20240629164406148](./assets/image-20240629164406148.png)





![image-20240629172013439](./assets/image-20240629172013439.png)



![image-20240629172025474](./assets/image-20240629172025474.png)



![image-20240629172044225](./assets/image-20240629172044225.png)



##### 2.9 用户登录功能



**Session共享后端分布式登录**



为什么需要共享？

问题：为什么服务器A登录后，请求发到服务器B,不认识该用户？



回答：原因如下：

1. 用户在A登录，所以session(用户登录信息)存在了A上

2. 结果请求B时，B没有用户信息，所以不认识。



![image-20240629173546106](./assets/image-20240629173546106.png)



解决方案：共享存储，而不是把数据放到单台服务器的内存中



![image-20240629173619958](./assets/image-20240629173619958.png)





如何共享存储？

核心思想：把数据放到同一个地方去集中管理。

1. Redis(基于内存的K/V数据库)此处选择Redis,因为用户信息读取/是否登录的判断极其频繁，Redis基于内存，读写性能很高，简单的数据单机qps5w-10w。
2. MySQL
3. 文件服务器ceph





装Redis ：【我就不开Linux 子系统了，用微软那个】https://github.com/microsoftarchive/redis/releases



算了算了，我直接跑一个容器：就不在本地装了，微软那个版本实在是太低



`docker run --name xiongpaoredis -d -p 6379:6379 redis:latest`



![image-20240629175435946](./assets/image-20240629175435946.png)



![image-20240629175457289](./assets/image-20240629175457289.png)



这里搞了个`6.2` 版本的【最新应该已经7 了，不管了，应该是docker 的仓库问题】



添加依赖：

```xml
        <!--    redis    -->
        <!--    操作    -->
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
            <version>2.6.3</version>
        </dependency>

        <!--        引入spring-session和redis的整合，使得自动将session存储到redis中：-->
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.17.5</version>
        </dependency>
```



写一下配置



![image-20240629180209629](./assets/image-20240629180209629.png)



![image-20240629180742645](./assets/image-20240629180742645.png)



挺好看的



再改一个存储配置



![image-20240629180936430](./assets/image-20240629180936430.png)



其实就完事儿了，单机到分布式

但是遇到个问题啊，启动不了了



![image-20240629183829467](./assets/image-20240629183829467.png)



卡住了，网上全是特么让我改成`none`，有毒 ...



![image-20240629184343210](./assets/image-20240629184343210.png)



最后加了一个这个依赖，解决了，蛙趣



![image-20240629184742380](./assets/image-20240629184742380.png)



可以的，一个配置就行了，之后从session 中对比数据，就是互通的了，都存到了redis 服务器里面



##### 2.10 个人信息修改功能



###### 2.10.1 后端开发



![image-20240629200153744](./assets/image-20240629200153744.png)



###### 2.10.2 前端开发



![image-20240629201243334](./assets/image-20240629201243334.png)



得先登录



##### 2.11 用户登录功能



**前端开发**



![image-20240629214519345](./assets/image-20240629214519345.png)



![image-20240629214736887](./assets/image-20240629214736887.png)



![image-20240629214748557](./assets/image-20240629214748557.png)



获取信息还是有问题，cookie 没有数据



##### 2.12 基础功能



前后端联调



axios 请求时没有携带上cookie，配置一下就行



![image-20240629221027481](./assets/image-20240629221027481.png)



再试一次【加了一个配置类】



![image-20240630104147198](./assets/image-20240630104147198.png)



把3000 和 8000 都放出来了



![image-20240630104215952](./assets/image-20240630104215952.png)



登录成功，切到个人



![image-20240630104229494](./assets/image-20240630104229494.png)



没问题



![image-20240630104251551](./assets/image-20240630104251551.png)



也拿到了信息



![image-20240630104343073](./assets/image-20240630104343073.png)



就是时间上有点问题



![image-20240630104451551](./assets/image-20240630104451551.png)



这个错误之后用组件解决，先取消了



尝试更新



![image-20240630104608063](./assets/image-20240630104608063.png)



提交



![image-20240630104618724](./assets/image-20240630104618724.png)



修改成功



![image-20240630104627830](./assets/image-20240630104627830.png)



没问题，就是改之后没跳回来，这里还把用户信息重新封装了一下



![image-20240630110240236](./assets/image-20240630110240236.png)



> 红的都是检查，代码没问题的



![image-20240630110610710](./assets/image-20240630110610710.png)



之前跳不回来，router 忘记引入了





##### 2.13 H5主页开发



**抽象通用列表组件**



自己做一下，未登录重定向【主页和队伍没做】



![image-20240630111705674](./assets/image-20240630111705674.png)



![image-20240630111715941](./assets/image-20240630111715941.png)



切到个人，未登录的话，就跳转到登录页



![image-20240630111745686](./assets/image-20240630111745686.png)



登录后切过来，展示信息



主页就是把搜索结果页搬过来了



![image-20240630112452265](./assets/image-20240630112452265.png)



接下来写接口



![image-20240630112949540](./assets/image-20240630112949540.png)



简单的先拉了一遍所有用户出来展示



![image-20240630113003251](./assets/image-20240630113003251.png)



再加几个用户吧



![image-20240630113653802](./assets/image-20240630113653802.png)



没问题，把用户列表封装成一个组件



![image-20240630115025923](./assets/image-20240630115025923.png)



OK



##### 2.14 批量导入数据



###### 2.14.1 多种方案介绍及对比



模拟100万个用户，再去查询



![image-20240630121037601](./assets/image-20240630121037601.png)



先看看插入1000条要多少时间



![image-20240630121459322](./assets/image-20240630121459322.png)



1000 条插入完成，鱼皮用了1分半，我用了两秒 ...



批量插入试试：

![image-20240630122133447](./assets/image-20240630122133447.png)



![image-20240630122153920](./assets/image-20240630122153920.png)



12s 10万条插入完成。



![image-20240630122219936](./assets/image-20240630122219936.png)



###### 2.14.2 并发编程



之前就算用了批量，但是还是同步顺序执行的，搞个并发



![image-20240630124658299](./assets/image-20240630124658299.png)



![image-20240630124628803](./assets/image-20240630124628803.png)



14s 插了50 万条，再插40万



![image-20240630124755292](./assets/image-20240630124755292.png)



11s

![image-20240630124810827](./assets/image-20240630124810827.png)



OK，现在数据里面就有100万条用户数据了



###### 2.14.3 测试及性能优化



我们现在访问一下主页



![image-20240630125131296](./assets/image-20240630125131296.png)



![image-20240630125145984](./assets/image-20240630125145984.png)



页面已经卡死了，100万条数据全部喂给前端不行



![image-20240630125247118](./assets/image-20240630125247118.png)



内存爆了



做个分页：

![image-20240630125757205](./assets/image-20240630125757205.png)





![image-20240630130102969](./assets/image-20240630130102969.png)



前端也请求一下



![image-20240630130114216](./assets/image-20240630130114216.png)



![image-20240630130301797](./assets/image-20240630130301797.png)



OK了



##### 2.15 主页性能优化



###### 2.15.1 缓存与分布式缓存



数据查询慢怎么办？



用缓存：提前把数据取出来保存好（通常保存到读写更快的介质，比如内存），就不用再查数据库了，可以更快地读写。

用定时任务：预加载缓存，定时更新缓存。

思考：多个机器都要执行同一个任务么？（可以用分布式锁解决：控制同一时间只有一台机器去执行定时任务，其他机器不用重复执行了）



缓存分类：

- 分布式缓存：
    - Redis(分布式缓存)
    - memcached(分布式)
    - Etcd(云原生架构的一个分布式存储，存储配置，扩容能力)
- 单机缓存：
    - ehcache
    - Java内存集合，如HashMap
    - Caffeine(Java内存缓存性能之王，高性能)
    - Google Guava



###### 2.15.2 Redis入门



NoSQL数据库

key-value存储系统（区别于`MySQL`,他存储的是键值对）



Redis数据结构

基本：

- String字符串类型：name:"yupi"
- List列表：names:["yupi","dogyupi","yupi"]
- Set集合：names:["yupi","dogyupi'"] (值不能重复)
- Hash哈希：nameAge:{"yupi":1,"dogyupi":2}
- Zset集合：names:{yupi-9,dogyupi-12}(适合做排行榜)

高级：

- bloomfilter(布隆过滤器，主要从大量的数据中快速过滤值，比如邮件黑名单拦截)
- geo(计算地理位置)
- hyperloglog(pv/uv)
- pub/sub(发布订阅，类似消息队列)
- BitMap(1001010101010101010101010101)



------



Java 操作Redis：

- Spring Data Redis (推荐)



Spring Data:通用的数据访问框架，定义了一组增删改查的接口【还可以操作：mysql、redis、jpa】



- Jedis
- Redission



###### 2.15.3 缓存开发和注意事项



关键点：不同用户看到的数据不同

建议格式：

systemld:moduleld:func.options(不要和别人中突)

比如：xiongpao:user:recommed:userld

>  注意！redis内存不能无限增加，一定要设置过期时间！！！



![image-20240630132305645](./assets/image-20240630132305645.png)



![image-20240630132406883](./assets/image-20240630132406883.png)



查出来第一页的 8 个，看看redis



![image-20240630132440959](./assets/image-20240630132440959.png)



![image-20240630132454190](./assets/image-20240630132454190.png)



而且这次用了1s 多，再来一次



![image-20240630132617467](./assets/image-20240630132617467.png)



时间明显变短，而且没有从数据库里面拿数据



###### 2.15.4 缓存预热设计与实现

问题：即使用了缓存，第一个用户访问还是很慢(假如第一个访客是老板，哦豁！)。



缓存预热的优点：

1. 解决上面的问题，可以让用户始终访问很快
2. 也能一定程度上保护数据库

缺点：

1. 增加开发成本（你要额外的开发、设计）
2. 预热的时机和时间如果错了，有可能导致你缓存的数据不对或者太老
3. 需要占用额外空间



怎么缓存预热？

1. 定时任务
2. 手动触发



###### 2.15.5 定时任务介绍和实现



用定时任务，每天刷新所有用户的推荐列表。

注意点：

1. 缓存预热的意义（新增少、总用户多）
2. 缓存的空间不能太大，要预留给其他缓存空间
3. 缓存数据的周期（此处每天一次）


实现一下：

1. Spring Scheduler(spring boot默认整合了，推荐这种方式)
2. Quartz(独立于Spring存在的定时任务框架)
3. XXL-Job之类的分布式任务调度平台（界面+sdk)



采用第一种方式：



![image-20240630133633921](./assets/image-20240630133633921.png)



![image-20240630142841621](./assets/image-20240630142841621.png)



在线Cron 表达式生成器：https://cron.qqe2.com/



![image-20240630142902035](./assets/image-20240630142902035.png)



到点就会执行



![image-20240630143008764](./assets/image-20240630143008764.png)



时间刚一切就执行了

![image-20240630143022146](./assets/image-20240630143022146.png)



redis 也写进去了



![image-20240630143036665](./assets/image-20240630143036665.png)



30 秒过期



###### 2.15.6 锁、分布式锁



**控制定时任务的执行**



要控制定时任务在同一时间只有1个服务器能执行。



为啥？

1. 浪费资源，想象10000台服务器同时“打鸣”
2. 脏数据，比如重复插入

怎么做？几种方案：

1. 分离定时任务程序和主程序，只在1个服务器运行定时任务。成本太大
2. 写死配置，每个服务器都执行定时任务，但是只有叩符合配置的服务器才真实执行业务逻辑，其他的直接返回。成本最低：但是我们的P可能是不固定的，把P写的太死了
3. 动态配置，配置是可以轻松的、很方便地更新的（代码无需重启），但是只有符合配置的服务器才真实执行业务逻辑。
    - 数据库
    - Redis
    - 配置中心(Nacos、Apollo、Spring Cloud Config)



问题：服务器多了、IP不可控还是很麻烦，还是要人工修改

4. 分布式锁，只有抢到锁的服务器才执行业务逻辑。坏处：增加成本；好处：不用手动配置，多少个服务器都一样。
   注意，只要是单机，就会存在单点故障。



**锁：**



有限资源的情况下，控制同一时间（段）只有某些线程（用户/服务器）访问到资源。

Java实现锁：synchronized关键字、并发包的类

但存在问题：只对单个JVM有效



**分布式锁：**



为啥需要分布式锁？

1. 有限资源的情沉下，控制同一时间（段）只有某些线程（用户/服务器）访问到资源。
2. 单个锁只对单个JVM有效



分布式锁实现的关键：



抢锁机制

怎么保证同一时间只有1个服务器抢到锁？

**核心思想**就是：先来的人先把数据改成自己的标识（服务器ip），后来的人发现标识已存在，就抢锁失败，继续等待。

等先来的人执行方法结束，把标识清空，其他的人继续抢锁。

- MySQL数据库：select for update行级锁（最简单），或者用乐观锁。
- Redis实现：内存数据库，读写速度快。支持setnx、Iua脚本，比较方便我们实现分布式锁。
- setnx:set if not exists如果不存在，则设置；只有设置成功才会返回true,否则返回false。



【注意事项】

1）用完锁要释放（腾地方）

2）锁一定要加过期时间

3）如果方法执行时间过长，锁提前过期了？



会导致问题：

1. 连锁效应：释放掉别人的锁
2. 这样还是会存在多个方法同时执行的情况

解决方案：续期

4）释放锁的时候，有可能先判断出是自己的锁，但这时锁过期了，最后还是释放了别人的锁

解决方案：Redis+lua脚本保证操作原子性

5）Redis如果是集群（而不是只有一个Redis),如果分布式锁的数据不同步怎么办？

解决方案：https://blog.csdn.net/feiying0canglang/article/details/113258494



> 拒绝自己实现



**Redisson 实现分布式锁**



Redisson是一个java操作Redis的客户端，提供了大量的分布式数据集来简化对Redis的操作和使用，可以让开发者像使用本地集合一样使用Redis,完全感知不到Redis的存在。

关键词：Java Redis客户端，分布式数据网格，实现了很多Java里支持的集合。



直接引入



![image-20240630144429156](./assets/image-20240630144429156.png)



配置类：



![image-20240630144834291](./assets/image-20240630144834291.png)



**分布式锁保证定时任务不重复执行**



![image-20240630145059461](./assets/image-20240630145059461.png)



**看门狗机制 **【redisson中提供的续期机制】



开一个监听线程，如果方法还没执行完，就帮你重置redis锁的过期时间。
原理：

1. 监听当前线程，默认过期时间是30秒，每10秒续期一次（补到30秒）
2. 如果线程挂掉（注意debug模式也会被它当成服务器宕机），则不会续期



> https://blog.csdn.net/qq_26222859/article/details/79645203





##### 2.16 组队功能



###### 2.16.1 需求分析



我要跟别人一起参加竞赛或者做项目，可以发起队伍或者加入别人的队伍



用户可以创建一个队伍，设置队伍的人数、队伍名称（标题）、描述、超时时间。

> 队长、剩余的人数聊天？公开或private或加密用户创建队伍最多5个

展示队伍列表，根据名称搜索队伍P0,信息流中不展示已过期的队伍

修改队伍信息

用户可以加入队伍（其他人、未满、未过期），允许加入多个队伍，但是要有个上限

> 是否需要队长同意？筛选审批？

用户可以退出队伍（如果队长退出，权限转移给第二早加入的用户一先来后到）

队长可以解散队伍

分享队伍=>邀请其他用户加入队伍

业务流程：

1. 生成分享链接（分享二维码）
2. 用户访问链接，可以点击加入

队伍人满后发送消息通知




###### 2.16.2 系统设计



**数据库表设计**



队伍表team

字段：

- id主键bigint(最简单、连续，放url上比较简短，但缺点是爬虫)
- name队伍名称
- description描述
- maxNum最大人数
- expireTime过期时间
- userld创建人id
- status 0-公开，1-私有，2-加密
- password密码
- createTime创建时间
- updateTime更新时间
- isDelete是否删除



```sql
-- 队伍表
create table team
(
    id          bigint auto_increment comment 'id' primary key,
    name        varchar(256)                       not null comment '队伍名称',
    description varchar(1024)                      null comment '描述',
    maxNum      int      default 1                 not null comment '最大人数',
    expireTime  datetime                           null comment '过期时间',
    userId      bigint comment '用户id（队长 id）',
    status      int      default 0                 not null comment '0 - 公开，1 - 私有，2 - 加密',
    password    varchar(512)                       null comment '密码',
    createTime  datetime default CURRENT_TIMESTAMP null comment '创建时间',
    updateTime  datetime default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP,
    isDelete    tinyint  default 0                 not null comment '是否删除'
) comment '队伍';
```





两个关系：

1. 用户加了哪些队伍？
2. 队伍有哪些用户？



两种实现方式：

1. 建立用户-队伍关系表teamld userld(便于修改，查询性能高一点，可以选择这个，不用全表遍历)【√】
2. 用户表补充已加入的队伍字段，队伍表补充已加入的用户字段（便于查询，不用写多对多的代码，可以直接根据队伍查用户、根据用户查队伍）



字段：

- id主键
- userld用户id
- teamld队伍id
- joinTime加入时间
- createTime创建时间
- updateTime更新时间
- isDelete是否删除



```sql
-- 用户队伍关系
create table user_team
(
    id         bigint auto_increment comment 'id'
        primary key,
    userId     bigint comment '用户id',
    teamId     bigint comment '队伍id',
    joinTime   datetime                           null comment '加入时间',
    createTime datetime default CURRENT_TIMESTAMP null comment '创建时间',
    updateTime datetime default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP,
    isDelete   tinyint  default 0                 not null comment '是否删除'
) comment '用户队伍关系';
```



代码生成器



![image-20240630151920622](./assets/image-20240630151920622.png)



###### 2.16.3 基础接口开发及测试



**为什么需要请求参数包装类**



1. 请求参数名称/类型和实体类不一样
2. 有一些参数用不到，如果要自动生成接口文档，会增加理解成本
3. 对个实体类映射到同一个对象



![image-20240630161628856](./assets/image-20240630161628856.png)



**为什么需要包装类**



可能有些字段需要隐藏，不返回给前端

或者有些字段某些方法是不关心的



```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```



![image-20240630163440746](./assets/image-20240630163440746.png)



![image-20240630165141551](./assets/image-20240630165141551.png)



调试完成，原始接口都能用



![image-20240701093127259](./assets/image-20240701093127259.png)



这里发现了一个很神奇的事儿



![image-20240701094036386](./assets/image-20240701094036386.png)



一样的接口，postman  能调，离谱，建议改一下



![image-20240701094123194](./assets/image-20240701094123194.png)



![image-20240701094156208](./assets/image-20240701094156208.png)



下面根据需求分析修改接口



**系统接口设计**



1. 创建队伍



用户可以创建一个队伍，设置队伍的人数、队伍名称（标题）、描述、超时时间

队长、剩余的人数聊天？公开或private或加密信息流中不展示已过期的队伍

1. 请求参数是否为空？

2. 是否登录，未登录不允许创建

3. 校验信息
    1. 队伍人数>1且<=20
    2. 队伍标题<=20
    3. 描述<=512
    4. status是否公开(int)不传默认为0（公开）
    5. 如果status是加密状态，一定要有密码，且密码<=32
    6. 超时时间>当前时间
    7. 校验用户最多创建5个队伍
4. 插入队伍信息到队伍表
5. 插入用户=>队伍关系到关系表



![image-20240630171451913](./assets/image-20240630171451913.png)



![image-20240630171506279](./assets/image-20240630171506279.png)



测试一下



![image-20240630172727720](./assets/image-20240630172727720.png)



没问题



![image-20240630172736622](./assets/image-20240630172736622.png)



> 其实这里还可以更完善，status 设置为公开了，但是还是可以设置密码，有点冲



![image-20240630172856174](./assets/image-20240630172856174.png)



没问题



2. 查询队伍列表



分页展示队伍列表，根据名称、最大人数等搜索队伍,信息流中不展示已过期的队伍。

1. 从请求参数中取出队伍名称等查询条件，如果存在则作为查询条件
2. 不展示已过期的队伍（根据过期时间筛选】
3. 可以通过某个关键词同时对名称和描述查询
4. 只有管理员才能查看加密还有非公开的房间
5. 关联查询已加入队伍的用户信息
6. 关联查询已加入队伍的用户信息（可能会很耗费性能，建议大家用自己写SQL的方式实现）



用mybatis-plus 构造查询：



![image-20240701103134328](./assets/image-20240701103134328.png)



![image-20240701103158222](./assets/image-20240701103158222.png)



没问题



3. 修改队伍信息



1. 判断请求参数是否为空
2. 查询队伍是否存在
3. 只有管理员或者队伍的创建者可以修改
4. 如果用户传入的新值和老值一致，就不用update了（可自行实现，降低数据库使用次数）
5. 如果队伍状态改为加密，必须要有密码
6. 更新成功




![image-20240630175611040](./assets/image-20240630175611040.png)



![image-20240630175834317](./assets/image-20240630175834317.png)





![image-20240630175847499](./assets/image-20240630175847499.png)



4. 用户可以加入队伍



其他人、未满、未过期，允许加入多个队伍，但是要有个上限

1. 用户最多加入5个队伍
2. 队伍必须存在，只加入未满、未过期的队伍
3. 不能加入自己的队伍，不能重复加入己加入的队伍（幂等性）
4. 禁止加入私有的队伍
5. 如果加入的队伍是加密的，必须密码匹配才可以
6. 新增队伍-用户关联信息

注意，一定要加上事务注解！！！！



![image-20240630180716813](./assets/image-20240630180716813.png)



![image-20240630180809672](./assets/image-20240630180809672.png)



当前登录海绵宝宝，用他加入一个队伍



![image-20240630180917577](./assets/image-20240630180917577.png)



![image-20240630180934205](./assets/image-20240630180934205.png)



id 为 2 的用户加入了 id 为2 的队伍



如果是加密队伍：



![image-20240630181121173](./assets/image-20240630181121173.png)



![image-20240630181140646](./assets/image-20240630181140646.png)



当前是 队伍创建者尝试加入队伍



![image-20240630181225118](./assets/image-20240630181225118.png)



![image-20240630181239023](./assets/image-20240630181239023.png)



没问题



5. 用户可以退出队伍



请求参数：队伍 id

1. 校验请求参数
2. 校验队伍是否存在
3. 校验我是否已加入队伍
4. 如果队伍只剩一人，队伍解散
5. 还有其他人
    - 如果是队长退出队伍，权限转移给第二早加入的用户一先来后到（只用取id最小的2条数据）
    - 非队长，自己退出队伍



![image-20240630181608181](./assets/image-20240630181608181.png)





id 为 1 的用户退出自己创建的一个队伍，比如 3



![image-20240630181947509](./assets/image-20240630181947509.png)



当前的3 ，1 是队长，直接用1 退出队伍 3



![image-20240630182141681](./assets/image-20240630182141681.png)



![image-20240630182158660](./assets/image-20240630182158660.png)



队伍走了，另一个人提上来当队长



![image-20240630182223471](./assets/image-20240630182223471.png)



关系已删除，如果 用户2 再退



![image-20240630182312115](./assets/image-20240630182312115.png)



![image-20240630182323009](./assets/image-20240630182323009.png)



![image-20240630182335331](./assets/image-20240630182335331.png)



队伍也解散了，没人了





6. 队长可以解散队伍



请求参数：队伍 id

业务流程：

1. 校验请求参数
2. 校验队伍是否存在

3. 校验你是不是队伍的队长
4. 移除所有加入队伍的关联信息
5. 删除队伍



![image-20240630182636844](./assets/image-20240630182636844.png)



![image-20240630182711901](./assets/image-20240630182711901.png)



用户1 直接解散队伍1



![image-20240630183650789](./assets/image-20240630183650789.png)



这里发现一个源码小bug，用equals  解决了



![image-20240630183735189](./assets/image-20240630183735189.png)



![image-20240630183744160](./assets/image-20240630183744160.png)



![image-20240630183912090](./assets/image-20240630183912090.png)



关系也一并删除了



7. 获取当前用户已加入的队伍



![image-20240701120551766](./assets/image-20240701120551766.png)



接口补充根据ID 列表查询



![image-20240701120859816](./assets/image-20240701120859816.png)



![image-20240701121154509](./assets/image-20240701121154509.png)



8. 获取当前用户创建的队伍



![image-20240701120238115](./assets/image-20240701120238115.png)



![image-20240701121142360](./assets/image-20240701121142360.png)



> 这两个接口都复用了listTeam 方法，只是新增了查询条件，没有做修改【开闭原则】



没问题，都能用



###### 2.16.4 前端多页面开发



创建队伍页面实现



![image-20240630185726911](./assets/image-20240630185726911.png)



![image-20240630185737966](./assets/image-20240630185737966.png)



添加成功



![image-20240630185754500](./assets/image-20240630185754500.png)



没问题



查询展示队伍列表



![image-20240701113017284](./assets/image-20240701113017284.png)



![image-20240701113026319](./assets/image-20240701113026319.png)





接口调用就很快



###### 2.16.5 权限控制



搜索队伍功能



![image-20240701113624394](./assets/image-20240701113624394-1719804984927-1.png)



![image-20240701113637093](./assets/image-20240701113637093.png)



![image-20240701113643638](./assets/image-20240701113643638.png)



![image-20240701113650721](./assets/image-20240701113650721.png)



没问题



更新队伍：仅创建人可见



**前端不同页面传递数据**



1. url querystring(xxx?id=1)比较适用于页面跳转
2. url (/team/:id,xxx/1)
3. hash (/team#1)
4. localStorage
5. context(全局变量，同页面或整个项目要访问公共变量)





![image-20240701115747260](./assets/image-20240701115747260.png)



![image-20240701115814045](./assets/image-20240701115814045.png)



先修改一下个人信息页



![image-20240701122056751](./assets/image-20240701122056751.png)



这样也可以直接调用接口了



我已加入的队伍列表



![image-20240701122306637](./assets/image-20240701122306637.png)



我创建的队伍列表



![image-20240701122332603](./assets/image-20240701122332603.png)



这里换一个用户效果可能更明显



![image-20240701122445869](./assets/image-20240701122445869.png)



![image-20240701122451470](./assets/image-20240701122451470.png)



海绵宝宝创建的队伍没有



![image-20240701122506756](./assets/image-20240701122506756.png)



加入有点问题



![image-20240701122852190](./assets/image-20240701122852190.png)



按理说应该一条也没有



![image-20240701123054992](./assets/image-20240701123054992.png)



看到了，当idlist 为空的时候，还是把所有数据都查出来了



![image-20240701123156219](./assets/image-20240701123156219.png)



只有真正加入队伍了，才能查出来数据



![image-20240701124413537](./assets/image-20240701124413537.png)



![image-20240701124421541](./assets/image-20240701124421541.png)





退出队伍和解散队伍 先把功能实现，其实这里页面展示还需要进行判断



![image-20240701134401046](./assets/image-20240701134401046.png)



![image-20240701134728015](./assets/image-20240701134728015.png)



![image-20240701134739974](./assets/image-20240701134739974.png)



小队5 已经没人了，所以解散了



![image-20240701134801523](./assets/image-20240701134801523.png)





关系也删除了



![image-20240701134819758](./assets/image-20240701134819758.png)



再试试直接解散



![image-20240701134837587](./assets/image-20240701134837587.png)



我直接解散了1



![image-20240701134856428](./assets/image-20240701134856428.png)



![image-20240701134909635](./assets/image-20240701134909635.png)



队伍解散，所有人退出



现在是按钮的显示问题，权限



![image-20240701135125855](./assets/image-20240701135125855.png)



海绵宝宝其实只加入了 1，todo 仅加入队伍可见 `退出队伍` 按钮



##### 2.17 随机匹配功能



###### 2.17.1 匹配算法介绍及实现



需求背景：为了帮大家更快地发现和自己兴趣相同的朋友

思考：匹配1个还是匹配多个？

答：匹配多个，并且按照匹配的相以度从高到低排序

思考：怎么匹配？（根据什么匹配）

答：标签tags



> 还可以根据user_team匹配加入相同队伍的用户

问题本质：找到有相似标签的用户
举例：

- 用户A:[Java,大一，男]
- 用户B:[Java,大二，男]
- 用户C:Python,大二，女]
- 用户D:[Java,大一，女]



**1、怎么匹配？**

1. 找到有共同标签最多的用户(TopN)
2. 共同标签越多，分数越高，越排在前面
3. 如果没有匹配的用户，随机推荐几个（降级方案）



**两种算法**

- 编辑距离算法
- 余弦相似度算法



**2、怎么对所有用户匹配，取TOP?**

直接取出所有用户，依次和当前用户计算分数，取TOPN(54秒)



优化方法

1. 切忌不要在数据量大的时候循环输出日志（取消掉日志后20秒）
2. Map存了所有的分数信息，占用内存

解决：维护一个固定长度的有序集合(sortedSet)，只保留分数最高的几个用户（时间换空间）



e.g.【3,4,5,6,7】取TOP5,id为1的用户就不用放进去了

3. 细节：剔除自己√
4. 尽量只查需要的数据：
5. 过滤掉标签为空的用户√
6. 根据部分标签取用户（前提是能区分出来哪个标签比较重要）】
7. 只查需要的数据（比如id和tags)√(7.0s)
8. 提前查？（定时任务）
9. 提前把所有用户给缓存（不适用于经常更新的数据）
10. 提前运算出来结果，缓存（针对一些重点用户，提前缓存）



> 大数据推荐机制
>
> 大数据推荐场景：比如说有几亿个商品，难道要查出来所有的商品？难道要对所有的数据计算一遍相似度？
>
> 大数据推荐流程：
>
> - 检索=>召回=>粗排=>精排=>重排序等等
> - 检索：尽可多地查符合要求的数据（比如按记录查）
> - 召回：查询可要用到的数据（不故运算）
> - 粗排：粗略排序，简单地运算（运算相对轻量）
> - 精排：精细排序，确定固定排位



###### 2.17.2 性能优化及测试



算法工具类：



![image-20240701140504050](./assets/image-20240701140504050.png)



接口



![image-20240701141111790](./assets/image-20240701141111790.png)



测试一下



![image-20240701141135487](./assets/image-20240701141135487.png)



当前登录用户为 1



![image-20240701141250251](./assets/image-20240701141250251.png)



没有除开自己



![image-20240701142134499](./assets/image-20240701142134499.png)



感觉是这里的问题



![image-20240701142227385](./assets/image-20240701142227385.png)



这里直接过去了，没判断到



![image-20240701142302592](./assets/image-20240701142302592.png)



换成equals 再试一次



![image-20240701142345024](./assets/image-20240701142345024.png)



可以了，可能是数据类型的问题，发现问题了



![image-20240701142715285](./assets/image-20240701142715285.png)



皮总源码是long ，我们生成的代码是包装





##### 2.18 项目优化



###### 2.18.1 前端优化



![image-20240701143635794](./assets/image-20240701143635794.png)



是否开启心动模式



![image-20240701143659668](./assets/image-20240701143659668.png)



没问题，不同模式调用不同接口



加入loading 效果



![image-20240701144030131](./assets/image-20240701144030131.png)





然后就是那几个按钮的显示问题



![image-20240701144417386](./assets/image-20240701144417386.png)



![image-20240701144522732](./assets/image-20240701144522732.png)



要先改下后端



###### 2.18.2 后端优化



让`/team/list` 接口返回更多信息



![image-20240701144801683](./assets/image-20240701144801683.png)



![image-20240701145013712](./assets/image-20240701145013712.png)



这样显示就正常了



![image-20240701145052256](./assets/image-20240701145052256.png)



和数据库一致



设置一下每个页面展示不同的title



![image-20240701145426885](./assets/image-20240701145426885.png)



![image-20240701145444831](./assets/image-20240701145444831.png)



没问题，舒服





##### 2.19 项目改造优化



###### 2.19.1 全局登录拦截



![image-20240701151044830](./assets/image-20240701151044830.png)



未登录的状态，不会返回任何数据



![image-20240701151155804](./assets/image-20240701151155804.png)



之前我只简单做了一下个人页这儿，现在改成全局拦截



![image-20240701151244570](./assets/image-20240701151244570.png)



![image-20240701151335664](./assets/image-20240701151335664.png)



40101 也可以跳



加入要密码的房间要输入密码



![image-20240701151703647](./assets/image-20240701151703647.png)



先做一个切换，因为加密房间不展示



把那个创建按钮改一下先



![image-20240701152035901](./assets/image-20240701152035901.png)



![image-20240701152459790](./assets/image-20240701152459790.png)



换另一个用户登录



![image-20240701152954932](./assets/image-20240701152954932.png)



加入队伍



![image-20240701153133635](./assets/image-20240701153133635.png)



![image-20240701153255545](./assets/image-20240701153255545.png)



密码错误就加入失败



![image-20240701153310186](./assets/image-20240701153310186.png)



密码正确，加入成功





![image-20240701153318442](./assets/image-20240701153318442.png)



![image-20240701153329104](./assets/image-20240701153329104.png)



没问题



> 可以做一个实时刷新



##### 2.20 免备案上线



###### 2.20.1 前端Serveless 方案



打包前端项目



![image-20240701154038324](./assets/image-20240701154038324.png)



跳过语法检测那些



![image-20240701154225955](./assets/image-20240701154225955.png)



![image-20240701154319609](./assets/image-20240701154319609.png)



上线：https://vercel.com/ 【免费】



安装：`npm install -g vercel`

登录：`vercel login`



![image-20240701155417429](./assets/image-20240701155417429.png)



![image-20240701155506648](./assets/image-20240701155506648.png)



必须先登录一下啊



![image-20240701155645917](./assets/image-20240701155645917.png)



牛逼啊卧槽



![image-20240701155709612](./assets/image-20240701155709612.png)



好像好了



![image-20240701155752273](./assets/image-20240701155752273.png)



访问一下啊：

![image-20240701161029275](./assets/image-20240701161029275.png)



不错啊，就是后端还有些问题



###### 2.20.2 后端容器托管



微信云托管



![image-20240701155949227](./assets/image-20240701155949227.png)



![image-20240701160056528](./assets/image-20240701160056528.png)



![image-20240701160104502](./assets/image-20240701160104502.png)



![image-20240701160437133](./assets/image-20240701160437133.png)



这只是一个示例



![image-20240701160519599](./assets/image-20240701160519599.png)



可以新建服务



![image-20240701160552159](./assets/image-20240701160552159.png)



![image-20240701161201817](./assets/image-20240701161201817.png)



这里就这样吧，学会了。所有环境上线就行。











































































