# 简介

<figure><img src=".gitbook/assets/kuja.png" alt="Kuja Logo" style="zoom:50%;"><figcaption></figcaption></figure>

`Kuja`（`库加`）是面向中小规模系统的、通用型业务和底层框架。

其目标是简化脚手架和系统框架的搭建，尽可能开发者专注在业务领域上，从而提高开发和维护效率。

跟 Spring 一样，Kuja 既提供开箱即用的功能满足基础需要，又能以插件的方式进行定制开发 —— 而不需要对 Kuja 的源码进行修改（和重新编译打包）。



## 快速入门

* [面向 Spring](./quickstart/spring.md)



## 功能清单

主要功能清单如下，具体参照：[功能清单](./develop/features.md "mention")。

### 底层框架级别

| 分类     | 功能                                                         |
| -------- | ------------------------------------------------------------ |
| 单体功能 | <ul class="contains-task-list"><li>统一错误处理（定义、抛出、捕获和处理）</li><li>返回结果（定义和补全）</li><li>业务上下文（生命周期管理和传递）</li></ul> |
| 集群功能 | <ul class="contains-task-list"><li>全局配置 </li><li>服务注册和发现 </li><li>服务调用 </li><li>消息通知</li></ul> |
| 可观性   | <ul class="contains-task-list"><li>度量（Metrics） </li><li>跟踪（Tracing） </li><li>日志（Logging）</li></ul> |

### 中间件级别

| 分类   | 功能                                                                                                                                                                                                                                        |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 服务封装 | <ul class="contains-task-list"><li>ID 生成</li><li>短信服务</li><li>对象存储服务</li><li>User-Agent 解析</li><li>IP-Geo 解析</li></ul> |

### 应用功能级别

| 分类     | 功能                                                                                                                                                                                                                            |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DevOps | <ul class="contains-task-list"><li>综合运维平台</li><li>网关控制台</li></ul>                                                                                                               |
| 业务功能   | <ul class="contains-task-list"><li>账户和鉴权体系</li><li>站内通知</li><li>操作记录</li><li>导出管理</li><li>通用数据管理</li></ul> |



## 适用场景

如上所述，Kuja 的目标是提供面向业务的通用型框架，而不是提供一套完整的业务系统。

因此，相比需要快速见效的场景，Kuja 更适合于以下场景：

* 在中等规模、有一定技术要求的场景中，提供业务系统的底层支撑
* 在业务迭代发展或者希望做基类复用的场景中，作为多业务的基础和迭代框架
* 单纯作为研究和分析使用



## 基本原则

Kuja 开发中，遵循以下的基本原则。

* 整合高质量组件做抽象和集成，而不是重新发明轮子
* 对通用业务做抽象封装，既提供开箱即用的实现，也支持以插件的方式进行定制
* 开发单点突破并做透，努力做到小而美
* 侧重但不局限于 Spring 框架，以次要优先级兼容 JarkataEE、MicroProfile、Quarkus 等



## 欢迎加入

### Bug、反馈和建议

* [Github Issues](https://github.com/wingsweaver/kuja/issues)
* [码云 Issues](https://gitee.com/wingsweaver/kuja/issues)

### 贡献代码

Kuja 内置了以下的静态检查，贡献代码时请遵守。`人人为我，我为人人`

* CheckStyle 具体规则参照 Kuja-CheckStyle 规约 。\
  除了命名规约等，也注意文件、函数的复杂度不要超标（行数、嵌套层数、散度、圈复杂度等）。
* Alibaba P3C 使用默认的 [阿里巴巴 Java 编码规约](https://github.com/alibaba/p3c)。
* JavaDoc 请按照 Java 文档规范标明方法说明、参数说明、返回值说明等信息。\
  Kuja 打包过程中，会把 JavaDoc 的 warning 当成错误来处理。
  

详细的编码规约，请参照：[Kuja 编码规约](./develop/coding-rules.md)。  
