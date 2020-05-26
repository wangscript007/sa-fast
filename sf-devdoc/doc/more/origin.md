# sa-fast 来历

许多人好奇，sa-fast是如何诞生的，以及sa系列的框架是如何一步步走到今天的 

---

<!-- ### 造轮子阶段 -->

### 蒙昧时代

x年前，因为对原生`jdbc`重复代码的憎恨，我比着网上的各种博客，封装一个工具类，也就是我们常常见到的`SqlHelper.java`，它以静态方法封装了常见的数据读写步骤，让我彻底告别了对`Connection`、`ResultSet`等对象的操作，大大的简化了增删改查所需要的代码

这个东西一直被我视为神器，直到我遇见了更加智能的国民框架 `MyBatis`，让我第一次明白一个框架可以做的事情原来有这么多，我当即从我的项目删掉了`SqlHelper.java`，引入`MyBatis` 的jar包，开始了更加愉快的增删改查

但是随着时间推移，`MyBatis`的种种弊端，也逐渐暴露开来，比如集成前各种繁琐的配置，以及`sql`必须强行写在各种`xxxMapper.xml`的文件中，都让我逐渐觉得`mybatis`的设计理念与我追求的`短平快`目标不符

`sql`与`java代码`的强行分离，固然有他的好处，因为它让我们写`长sql`时不必再进行各种字符串拼接，让sql清晰明了，方便定位bug，但是像`MyBatis`这样自断后路式的分离，总让我感觉java代码与sql之间有着一条难以融合的界限，失去了对sql的掌控力

在`MyBatis`中，你必须遵循它的规则：该你写的`sql`，你一条也少不了。

这让我觉得对代码架构的优化，很快就触碰到了`MyBatis`的天花板

这时，我又怀念起了我的 `SqlHelper.java`，我怀念它对`sql`的掌控力，但同时又不想放弃`MyBatis`的种种强大特性，鱼与熊掌难以兼得，这时候我的脑海中突然冒出一个念头，为什么不能把它们两个的优点合二为一呢？


### SqlFly

说干就干，我将`MyBatis`的种种特性，写在纸上，然后在百度上一遍又一遍的搜索着它们的实现原理，然后将这些特性逐一加到了`SqlHelper`里。

这是我第一次接触`java`语言的各种高级特性，什么`反射`、`泛型`、`IO流`、`动态代理`、`自定义注解`等等，一次又一次颠覆我对`java`的认知

终于在我废寝忘食一周的之后，这个`SqlHelper.java`写到了一千多行，它具备了一个ORM框架应该具有的大多数功能，比`MyBatis`更强大的点在于，它可以根据实体类的字段，自动完成数据的插入、更新、删除、查询映射等功能，除了在一些边际情况下，它可能不是那么灵活。

既然它已经变得这么牛逼，当然不可以继续叫做`SqlHelper`这么土味十足的名字，于是在我苦思冥想两天之后，它有了一个崭新的名字——`SqlFly`

就这样，`SqlFly`的第一个版本，诞生了。

此后的一段岁月，我不断的添砖加瓦，让`SqlFly`变的更强大，更智能，它使得我一度认为，我有了干翻`MyBatis`的资本（现在想想真是年轻），直到遇到了神器：`MyBatis-Plus`。

论自动化生产sql，`MyBatis-Plus`可谓登峰造极，天下第一

虽然在某些情况下，它也有相应的局限性，比如：`联表查询`，以及自动化生产sql再次带来了对sql的失控感————但不可否认，它是一个极其优秀的框架

这一刻，我想通了一点，即使我把`SqlFly`优化到极点，也不过是另一版本的`MyBatis-Plus`，那我为何还要自己造轮子，不如直接使用`MyBatis-Plus`来的痛快

于是我改变了思路，毅然删除了所有有关自动化生产sql的代码，将sql的掌控力重新还给框架使用者，而框架本身，只负责好sql的执行，与结果集的各种映射。

于是又经过一段时间的升级迭代，`SqlFly`走向了另一个极端，即：对结果集的各种强大方便的映射。并且为了解决重复代码的问题，它还自带了一个小巧的代码生成器，可以自动化生成一些增删改查。

但是当我准备向社区推广它时，我又遇到了另一神器：`Spring Jdbc`

论数据集的映射能力，`Spring Jdbc`强于`SqlFly`，而且其本身就属于`Spring`全家桶系列，可以与`Spring`无缝集成，且在自动化生产sql方面，`Spring`也有另一大神器：`JPA`

这时候我决定不再闭门造车，开始学习社区各种ORM框架，有`点点点`就能写出sql的`jooq`，也有上古神器`hibernate`，等等等等

这些都让我明白了一个事实，java的ORM领域，已经百家争鸣，不缺新框架了。

且由于java在`多行字符串`功能上的弱势（虽然有不少插件可以实现它），所有的ORM框架（除了MyBatis）都有个绕不过去的门槛，就是长sql需要用到难看的字符串拼接（java14的新特性`文本块`有望解决这个难题，拭目以待）

于是对`SqlFly`的更新，我不再着重于功能增加，而是尽力简化了API调用方式，以及完善了开发文档，便把它放在了GitHub上，以供学习：[文档](https://sqlfly.dev33.cn/)



### sa-token 

起初做登录验证，使用的是`session会话`，即：在用户登录后，将User对象放在`session`上，在接下来的请求中，我们检查其`session`上是否挂载了对应的`User`对象，以验证其是否登录

但由于session的一些机制，其不少弱点暴露出来，比如：难以主动过期对方session、难以让session在分布式应用中相互关联，等等等等

这时候，`shiro`、`spring security`等框架的出现，一定程度上解决了这些问题，但也在更深层次的场景下，表现乏力，比如：`小程序、APP等前后台分离场景`、`踢人下线`、`多账号体系验证`等等等等 

虽然这些问题，社区也都给出了一定的解决方案，比如：集成redis完成分布式扩展，重写其`读取token`方法使其可以应用在前后台分离场景，等等等等，大多数难题，都有其应对措施

凡此种种，一定程度上证明了其框架的生态活跃，但是也侧面说明了一个问题：框架本身并不给力，因此才需要在实际应用场景中给它打各种补丁。

一次两次还好，但是如果每一个功能点，都需要打对应的补丁才能实现，那我还用你这个框架干嘛？

带着这个疑问，我将对`shiro`的各种补丁，分离开来，再加上一定的修补，于是：

`sa-token`诞生了。

登录验证、权限验证、自定义session会话、踢人下线、集成redis、无cookie模式、模拟他人账号、多账号体系验证、注解式鉴权、Spring集成...

零配置开箱即用，覆盖所有应用场景，你所需要的功能，`sa-token`都有，详见：[文档](http://sa-token.dev33.cn/)


### sa-admin

起初写后台管理页面，用过`easy-ui`、`hui-admin`、试过各式各样的模板，也用过`layui-admin`，以及什么`vue-element-admin`、`iview-admin`、`ant-design-pro`等等

但都感觉不太合适，为了真正的拥有一个用起来顺手的后台模板，我便趁着五一假期，亲自写了一个，即：[sa-admin](http://sa-admin.dev33.cn/)

为了满足熟悉脚手架的同学，它还有对应的vue单页版本：[sa-vue-admin](http://sa-vue-admin.dev33.cn/)


### sa-doc

sa-doc 是一个基于markdown语法的接口文档编写工具

在此不做赘述，详见：[文档](http://sa-doc.dev33.cn/)


### 整合

造了n多轮子，是时候做一个整合了

于是，`sa-fast`诞生了

它包括的组件有：sa-token、sa-admin、sa-doc

以及一个项目基架（集成各种常用功能），以及一个代码生成器






<br><br><br><br><br><br><br><br><br><br><br><br>

