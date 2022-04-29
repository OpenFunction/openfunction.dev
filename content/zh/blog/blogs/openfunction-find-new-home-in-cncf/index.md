---
title: 'OpenFunction 成为 CNCF 沙箱项目，使 Serverless 函数与应用运行更简单'
linkTitle: "OpenFunction 成为 CNCF 沙箱项目，使 Serverless 函数与应用运行更简单" 
date: 2022-04-29
weight: 100
---

![](https://pek3b.qingstor.com/kubesphere-community/images/202204291642608.png)

2022 年 4 月 27 日，青云科技容器团队开源的函数即服务（FaaS: Function-as-a-Service）项目 [OpenFunction](https://github.com/OpenFunction/OpenFunction) 顺利通过了云原生计算基金会 CNCF 技术监督委员会（TOC）的投票，正式进入 CNCF 沙箱（Sandbox）托管。这就意味着 OpenFunction 得到了云原生开源社区的认可，同时通过进入 Sandbox 可以进一步保障项目的中立性，开发者以及合作伙伴等都可以参与项目建设，共同打造新一代开源函数计算平台。

![](https://pek3b.qingstor.com/kubesphere-community/images/202204281428250.png)

**这已经是青云科技容器团队发起的第三个进入 CNCF 的项目了**。早在 2021 年 7 月份青云科技就将 [Fluent Operator](https://github.com/fluent/fluent-operator) 项目捐给 Fluent 社区成为 CNCF 子项目，大大降低了 Fluent Bit 和 Fluentd 用户的使用门槛；同年 11 月份，负载均衡器插件 [OpenELB](https://github.com/kubesphere/openelb/) 加入 CNCF Sandbox，帮助私有化环境更便捷地对外暴露服务。

目前可以在 [CNCF Landscape 的 Serverless 版块](https://landscape.cncf.io/serverless)中找到 OpenFunction 项目。

![](https://pek3b.qingstor.com/kubesphere-community/images/202204281513459.jpeg)

## 项目介绍

OpenFunction 是一个现代化的函数即服务（FaaS: Function-as-a-Service）项目，它能够帮助开发者专注于他们的业务逻辑，而不必担心底层运行环境和基础设施。用户只需提交一段代码，就可以生成事件驱动的、动态伸缩的 Serverless 工作负载。

OpenFunction 引入了很多非常优秀的开源技术栈，包括 Knative、Tekton、Shipwright、Dapr、KEDA 等，这些技术栈为打造新一代开源函数计算平台提供了无限可能：

- Shipwright 可以在函数构建的过程中让用户自由选择和切换镜像构建的工具，并对其进行抽象，提供了统一的 API；
- Knative 提供了优秀的同步函数运行时，具有强大的自动伸缩能力；
- KEDA 可以基于更多类型的指标来自动伸缩，更加灵活；
- Dapr 可以将不同应用的通用能力进行抽象，减轻开发分布式应用的工作量。

![](https://pek3b.qingstor.com/kubesphere-community/images/202204281447199.png)

从架构图上可以看出，OpenFunction 包含了 4 个核心组件：

- **Function** : 用户和函数打交道(函数的增删改查)的唯一入口，包含了函数的 Build 和 Serving 阶段的全部定义；
- **Build** : 用户创建 Function 后，Function 生成相应的 Builder，用户无须手动创建。 Builder 通过 Shipwright 选择不同的镜像构建工具 Cloud Native Buildpacks, buildah, BuildKit, Kaniko, 并在 Tekton 的控制下将应用构建为容器镜像；
- **Serving** : 用户创建 Function 后，Function 生成相应的 Serving，用户无须手动创建。Serving CRD 创建后，函数将被部署到不同的运行时中，可以选择同步运行时或异步运行时。同步运行时可以通过 Knative Serving 或者 KEDA-HTTP (开发中)来支持，异步运行时通过 Dapr+KEDA 来支持。
- **Events** : 对于事件驱动型函数来说，需要提供事件管理的能力。由于 Knative Eventing 过于复杂，所以我们研发了一个新的事件管理框架叫 **OpenFunction Events**。

目前 OpenFunction 已经正式发布了 0.6.0 版本，与上一个版本相比，新增了许多值得关注的功能，包括函数插件、函数的分布式跟踪、控制自动缩放、HTTP 函数触发异步函数等。同时，异步运行时定义也被重构了。核心 API 也已经从 `v1alpha1` 升级到 `v1beta1`。值得一提的是，OpenFunction 团队还与 [Apache SkyWalking](https://skywalking.apache.org/) 社区合作，增加了 FaaS 平台对函数可观测性的支持，现在大家可以直接在 SkyWalking UI 上通过图表来可视化 Serverless 函数的依赖关系并追踪函数的调用。

![](https://pek3b.qingstor.com/kubesphere-community/images/202204281510987.png)

详情可参考这篇文章：[OpenFunction 0.6.0 发布: FaaS 可观测性、HTTP 同步函数能力增强及更多特性](https://openfunction.dev/zh/blog/2022/03/25/openfunction-0.6.0-%E5%8F%91%E5%B8%83-faas-%E5%8F%AF%E8%A7%82%E6%B5%8B%E6%80%A7http-%E5%90%8C%E6%AD%A5%E5%87%BD%E6%95%B0%E8%83%BD%E5%8A%9B%E5%A2%9E%E5%BC%BA%E5%8F%8A%E6%9B%B4%E5%A4%9A%E7%89%B9%E6%80%A7/)。

关于 OpenFunction 的实际使用案例可以参考下面两篇文章：

+ [以 Serverless 的方式实现 Kubernetes 日志告警](https://mp.weixin.qq.com/s/EZWYqtXJ7Cj-Yd7Fro6uyA)
+ [基于 OpenFunction 构建 FaaS 化的数据归档系统](https://mp.weixin.qq.com/s/f5qL-cc8zNDkf5hF_VA2ug)

## 社区生态

OpenFunction 自 2020 年 12 月开源并提交了第一个 commit，到 2021 年 5 月发布第一个 Release，仅一年多的时间就发布了 6 个大版本，吸引了 **24** 个 Contributors 和 **480+** GitHub Stars，并且已经被**驭势科技**、**中国联通**、**全象低代码平台**等多个企业、组织和平台采用。截止目前参与贡献的企业和组织有：**KubeSphere**、**驭势科技**、**Apache SkyWalking**、**SAP**、**中国联通**、**全象云**，在此感谢每一位参与贡献的社区小伙伴对 OpenFunction 的支持和帮助，同时也欢迎更多的开发者和用户参与体验和贡献 OpenFunction。

除此之外，OpenFunction 团队还受邀参加了一些上游社区的例会并向大家介绍过 OpenFunction 项目及其典型用例，包括 [CNCF 的 TAG-runtime 会议](https://youtu.be/qDH_LbagrVA?t=821)和 [Dapr 社区会议](https://youtu.be/S9e3ol7JCDA?t=183)。尤其 Dapr 社区对 OpenFunction 青睐有加，Dapr 联合创始人非常看好项目的发展前景，感兴趣的同学可以观看视频进一步了解。OpenFunction 社区也正在和 Dapr 社区及 Quarkus 社区合作以实现将 Java 代码编译为 Native 程序在 Quarkus 环境中运行，可大幅降低 Java 程序的资源占用并极大地提升性能。接下来，OpenFunction 项目发起人霍秉杰还将在 5 月 10 日举办的 Apache SkyWalking 峰会介绍其与 Apache SkyWalking 在函数可观测性领域的联合方案。

[![](https://pek3b.qingstor.com/kubesphere-community/images/202204281713635.jpg)](https://twitter.com/yaronschneider/status/1506669219911331840)

[![](https://pek3b.qingstor.com/kubesphere-community/images/202204291134555.png)](https://twitter.com/mfussell/status/1506674826114609154)

## 未来规划

得益于 CNCF 为项目提供了开源和中立的背书，OpenFunction 也将真正变成一个由 100% 社区驱动的开源项目。接下来，OpenFunction 将开发与实现如下功能，欢迎给社区提交需求与反馈：

- 支持更多语言的异步函数框架包括 Nodejs, Python, Java 和 .NET；
- 支持将 Java 函数编译成 Native 程序运行在 Quarkus 环境中；
- 使用 KEDA 的 http-add-on 作为 Knative Serving 之外同步函数运行时的又一个选择；
- 支持 OpenTelemetry 生态作为 SkyWalking 之外的另一个函数 Tracing 的方案；
- 增加 OpenFunction 控制台；
- 实现 Serverless 工作流；
- 对在边缘运行的函数有更好的支持；
- 预研基于 Pool 的冷启动优化方案；
- 使用 WebAssembly 作为更加轻量的运行时，结合 Rust 函数来加速冷启动速度。

## 持续开源开放

未来 KubeSphere 团队将继续保持开源、开放的理念，持续作为 OpenFunction 项目的参与方之一，推动国内和国际开源组织的生态建设，将 OpenFunction 社区培育成一个开放中立的开源社区与生态，与更多的函数计算平台及上下游生态伙伴进行深度合作，欢迎大家关注、使用 OpenFunction  以及参与社区贡献。

+ ⚙️ GitHub：https://github.com/openfunction/openfunction
+ 🔗 官网：https://openfunction.dev/
+ 🙋 社群：微信搜索 kubesphere 加好友即可邀请您进群（或扫描下方二维码）

<center><img style="width: 200px" src="https://pek3b.qingstor.com/kubesphere-community/images/202204291640428.png"></center>

---

最后附上 OpenFunction 项目重磅参与者与关注者对 OpenFunction 的寄语：

### 吴晟

> Apache SkyWalking 创始人

我很高兴和兴奋看到 OpenFunction 顺利加入 CNCF，作为一个仅一年多的年轻项目，这是一个项目从原型走向稳定，多元和成熟过程中的重要里程碑。作为 Apache SkyWalking 的一员，我有幸参加了 SkyWalking v9 迭代过程中与 OpenFunction 的集成。开放，平等，中立的开源合作模式，让人印象深刻。我们双方会在 Serverless 的可观测性上，进行紧密深入的合作，包括更多语言集成，日志的集成，平台的性能集成等等。祝贺 OpenFunction 成功加入沙箱孵化，期待项目更上一层楼。Enjoy your CNCF journey.

### 张海立

> 驭势科技云平台研发总监

驭势科技 UISEE 是中国领先的自动驾驶公司，OpenFunction 帮助我们找到了一种基于 FaaS / Serverless 的业务服务快速定制方案，我们已将它用于解决跨公私有云的、针对不同存储中间件的数据处理和落盘问题（参见 [此案例](https://openfunction.dev/zh/blog/2021/12/31/%E5%9F%BA%E4%BA%8E-openfunction-%E6%9E%84%E5%BB%BA-faas-%E5%8C%96%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BD%92%E6%A1%A3%E7%B3%BB%E7%BB%9F/)）。期待有更多社区伙伴参与到 OpenFunction 的功能建设中，一起探索更多应用场景，提升研发效能！

### 张善友

> 深圳市友浩达科技有限公司CTO

OpenFunction 加入 CNCF 对我来说是一个额外的惊喜。我是最近一个月才成为 OpenFunction 的贡献者，我在最近 2 年实践 Dapr 的项目实战经验，让我深信基于 Dapr 的 OpenFunction 是一个非常有前景的 FaaS 项目。我现在负责建设 OpenFunction 的 .NET 支持框架开发工作，期待有更多的社区伙伴参与到 OpenFunction 的功能建设。

### 蔡礼泽

> SAP, OpenFunction 早期用户

我从去年关注到OpenFunction，当时被它的技术选型所吸引，非常的前沿，让我想到了许多的可能性。之后一直关注着项目的技术走向还有社区发展，还有参与贡献。一个优秀的项目离不开社区的支持，OpenFunction的维护者非常专业与热情。优秀的技术设计加上专业的社区，我相信OpenFunction会在云原生领域大放异彩。