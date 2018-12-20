---
title: graphql在去哪儿dsp系统中的尝试
date: 2017-11-11
categories: work
tags: graphql
---

## 背景
去哪儿dsp广告投放系统，是自己17年下半年开发维护的项目之一，采用的技术栈是egg + antdesign. 最初从0开始搭建，到现在基本稳定大概有一个多月的时间，中间还经历了一次大的迭代改版。这里mark下心得。

dsp目的主要是为了更好的帮助产品运营在市场营销方面的工作。支持在线管理和组装去哪儿对外的广告投放计划，相关素材模板，竞价花费等。业务性较强，它有两个很明显的业务特点：

投放计划和素材模板存在多对多的关系
一个用户关联着多个计划和素材

最初采用后端java提供的restful风格的接口，由于涉及到复杂的嵌套查询，基本是由后端负责封装逻辑，内部查询多次，对外暴露给前端一个接口。
后来再想这种场景下前端有没有解决方案，就接触了graphql。（其实前面说的java逻辑某种程度和graphql的工作很像）

## 尝试

由于已经有现成的restful接口，这次实践的目的主要是在node层将已有的restful接口包装为graphql

实现上主要有两个部分：graphql数据定义和对外提供的graphql查询服务。
主要基于 apollo-server-koa, apollo-server-core, graphql-tools 实现

### 数据定义
graphql数据定义目录结构如下：
![graphql数据定义](//happy-alex.github.io/images/job/dsp/dsp_1.png)

现在有3个模块：用户模块auth，推广计划模块plan，素材模板模块material。
每个模块都有自己的connector，model，resolver, schema文件，最后由graphql目录下的schema文件合并各个模块的schema定义，通过graphql-tools 提供的 makeExecutableSchema 方法统一导出。

比如auth模块的简单定义：
![auth模块的schema](//happy-alex.github.io/images/job/dsp/dsp_2.png)

### graphql查询

graphql查询主要是将restful的接口转换为graphql风格，这里使用了 apollo-server-core 提供的 runHttpQuery 来完成

runHttpQuery 核心代码
![runHttpQuery核心代码](//happy-alex.github.io/images/job/dsp/dsp_runhttpquery.png)

其中的context由合并各个模块的model而来
![runHttpQuery核心代码](//happy-alex.github.io/images/job/dsp/dsp_runhttpquery2.png)

graphql的一次查询过程，将restful转为graphql
![graphql一次查询](//happy-alex.github.io/images/images/job/dsp/dsp_restful_graphql.png)

最终实现的graphiql效果：

![dsp的graphql查询界面](https://img-blog.csdnimg.cn/20181206221207909.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RoengyNjVib2Jv,size_16,color_FFFFFF,t_70)


## 总结

### graphql好在哪儿
1. 一次查询可以请求复杂的数据
2. 静态类型，类型文档可以详细查询接口字段
3. 处理多版本的情况，客户端的自行决定请求内容

### 潜在问题
1. 查询通过递归取值完成的，存在性能问题：在一次http请求中，根据要查询的schema结构递归的调用各项资源的resolver，最后返回结果
2. 安全，http缓存： Ddos攻击，过大查询反复请求，graphql操作经常变化，怎么做http缓存
3. 查询schema写起来过于繁琐
4. 如何推动后端改造：graphql主要是前端收益，如何说服后端支持

### graphql vs restful

1. 数据格式：restful是千人一面，多个终端设备的请求结果可能都是相同的；而graphql千人千面，支持前端按需获取，可针对不同环境返回不同内容
2. API调用：restful访问每个资源，其都是一个endpoint；而graphql对外只暴露一个endpoint,差别只在请求体不同
3. 复杂数据处理：复杂数据嵌套查询的场景，restful可能需要多次来返请求；而graphql只有一次，可减少网络开销

### 后记
BFF + Graphql的场景实践越来越多

适合：客户端对数据有定制化需求或后端微服务较多，微服务小而美只专注于自己的领域，需要有一层能综合这些服务

不适合：客户端数据单一，无需定制化；后端服务简单，或本身已经对外提供了综合服务，不存在多个微服务的场景