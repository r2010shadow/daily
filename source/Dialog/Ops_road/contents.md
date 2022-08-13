

[辛酸运维路，运维工程师如何走得更好更远？ （转载）](https://mp.weixin.qq.com/s?__biz=MjM5NTk0MTM1Mw==&mid=2650652005&idx=2&sn=595463e89c304b85e218f4bcfe5a6df5&chksm=bef9c3e3898e4af5d1c9ec32b88eb6d50a79c1bb4385823410b9c3ee484af6ebc009ce17ba1c&scene=132#wechat_redirect-----------------------------------%E8%BE%9B%E9%85%B8%E8%BF%90%E7%BB%B4%E8%B7%AF%EF%BC%8C%E8%BF%90%E7%BB%B4%E5%B7%A5%E7%A8%8B%E5%B8%88%E5%A6%82%E4%BD%95%E8%B5%B0%E5%BE%97%E6%9B%B4%E5%A5%BD%E6%9B%B4%E8%BF%9C%EF%BC%9Fhttps://blog.51cto.com/lang8027/2581648)

```
**【摘要】**运维工程师之路知易行难，学的东西博而杂，难以统一集中，且有 “ 背锅侠 ” 之称，其中辛酸难以言表。笔者根据自身的经历提出几点考量，提供给大家参考，希望大家努力把自己塑造成为一名合格的、适合未来需要的运维工程师。

**【作者】****Rock，**目前担任某国内著名餐饮连锁企业运维负责人，从事过数据库、大数据和容器集群的工作，对DevOps流程和工具方面有比较深刻的理解。
```

本人 75  后，在职场摸爬滚打了二十载，初期以主机服务器和数据库起家，后期从事运维工作，现在落足在全国知名的一家连锁餐饮企业，担任运维总监的职务，负责  DevOps 平台建设工作和企业内 ToB 和 ToC  端的容器云运维。从国企到私企，从传统企业到互联网企业，再到传统企业，我深刻的感觉到技术发展的速度之快，更新迭代使人几乎处于茫然的境地，我想结合技术趋势，对于在当今时代运维人员如何学习发展，谈一下自己的心得。



## **1 、开篇前言**

在过去几十年间，技术发展的格局，发生了翻天覆地的变化。

在 90 年代时，商业化软件几乎成为了技术的全部， IOE 成为了当年每个 IT 人员最求职业高峰的不二之选。而 IOE  在那个年代也逐步统治了大部分的 IT 领域，比如 Oracle  ，原来只是数据库软件，而后通过不断的兼并，变成了前后端一体的一站式解决方案提供商。从 Java 、 JavaScript 、简易 web  开发工具 Oracle Application Express ，到中间件 weblogic ，再到数据展现工具 Obiee 、再到 ERP  领域的 Sieble 、 peoplesoft ，再到数据库的高可用 RAC 、灾备 datagurad 、复制 goldengate  ，最后索性提供一体机 Exadata ， Oracle 展现其统治 IT 上下游的雄心和实力。其实，那个时候，对 IT  人方向是比较明确的，就是学习 Oracle ，就对了，就和学习会计要考 CPA 一样，方向明确，不会迷茫。那时的我也是这么想的，想着  Oracle 可用学习一辈子，能去 Oracle 原厂工作，必定可以大展宏图。

但是这世界唯一不变的就是变化，接下来的日子里 IT 世界发生了诸多的变化。首先是，云的兴起， AWS 横空出世， Saleforce 接着跟上， Oracle 忽视了， Oracle  CEO Larry 开始时认为云无足轻重，因为云就是一堆机器用根网线上网。但事情的发展远远超出了他的意料，云的发展彻底改变了 IT  世界，源源不断的有软件 SaaS 化、 PaaS 化，到后来 AWS 的 Aurora DB 等数据库开始侵蚀 Oracle  的数据库市场份额，虽然这种侵蚀每年大约只有 0.3% 左右，但明显的 Oracle 的市场份额在慢慢下滑。而对于 IBM  来说，就是它们的服务器、小型机的订单呈下滑趋势，云端的 IaaS 使得大家不用担心服务器的庞大投资了。

其次是开源软件的广泛使用，以 google 为代表的厂商大肆在 github 上开放其软件的开源版本；对可靠性要求不高的客户，开始选择开源软件，而不去购买昂贵的 IOE  的许可证。于是开源软件开始从前端到后端开始侵蚀 IOE 的市场。每一种商业化软件，总能找到开源替代品。

再次就是分布式应用，分布式的推广不符合以 Oracle 为代表的厂商商业利益，但历史的前进是无法阻挡的，当服务器的横向线性性能扩展比纵向扩展的优势逐渐被认同， Oracle 也不得不低头，在 Oracle 12c 数据库中推出了 sharding 分片功能。但是不得不说已经晚了，各种开源版的分布式应用已经占据了大部分市场。

出于以上三点，作为技术人的我，工作和学习的轨迹发生了变化，我从一心钻研 Oracle 数据库技术，**变成了多点开花，开始了从前端到后端，从应用到数据的广度学习，为了在运维这条路上走的长远而努力。**



## **2 、运维技术之路**

其实，凭心而论，**运维技术这条路不好走。**过去不好走，因为学习的成本高，有过经历的同学一定知道，要不停的考认证，我就考过 OCM ，当初光考试费就是 7000 多人民币。现在的路更不好走，因为**开源技术发展太快了，种类繁多，不知道要学什么，容易陷入 “ 生命有限，所学无限 ” 的尴尬处境。**

大量开源的新技术出现，前端出现了 VueJS 大有取代 AngularJS 的趋势，而同时 React native 出现又大有取代 VueJS 的趋势，实现 IOS 端与  Andriod 端的统一代码；中间件家族出现了 RabbitMQ 、 RocketMQ, 使得 IT 架构中除了 Weblogic 和  Websphere 之外有了其他的选择；数据库端除了开源 MySQl ，还有 MongoDB 等 NoSQL 数据库，以及 MyCat 、  shardingJDBC ，乃至 OceanDB 、 TiDB 等众多分布式解决方案；数据分析领域出现了 Kylin 来取代 Oracle  OBIEE ， Python 语言、 Hadoop Mapreduce 取代 SPSS 等分析软件的趋势；存储领域， Ceph  和超融合系统的出现，利用高速网络和 ssd 盘的优势，将本地盘组合成虚拟存储，大有取代 EMC 的昂贵阵列的趋势。

那么如何走并走好运维工程师这条路呢？Google 给我们很好的启迪，**将来的运维工程师，目标是应该进化成 SRE 工程师，具备诸多技术。**下面我将展开阐述我的观点。

### **2.1 技术选型**

好，现在问题来了，面对如此之多的技术，学什么，怎么学，学到什么程度，是摆在我们面前的难题。我这里所提倡的是广度学习，也就是 T 字中学习法。

#### **2.1.1 深度选型 – “T” 字之” I ”**

学习中，必须首先选择好 T 字中的那个竖线，因为这是根本，将影响你今后的广度学习的众多方面。我主张首选开发，对于运维而言，就是学习 python 最为妥当，  python 被誉为是开发语言中的万能胶，因为它从前台到后头都可以有用武之地，前端 python 有自己的 Django  框架，可以编出凑合的页面，运维有运维的包体，数据分析有数据分析的包体，人工智能也能对接 tensorflow  ，几乎是“万金油”，日本已经把它列入了小学课程，把 python 学精深了是不会错的。然后，再把 mysql 或 Oracle  选一门选精深，形成 Python+Mysql 或 Python+Oracle 的 T 字“竖杠”。

Python 的迭代总的来说还是比较慢的， Python2 到 Python3 虽然变化挺大，不过目前都是 Python3 为主， Python2  已经不多见了，很好聚焦。而 MySql5.7 到 MySql8.0  大都只是增加了一些新功能，几乎没有颠覆性的改动，也比较稳定，学深学精，从时间上是可以保证的。

#### **2.1.2 广度选型 – “T” 字之 “—”**

记得曾经刚进一家公司的大数据部门的时候，几乎把我吓一跳，为什么呢？因为细排了一下，这个大数据部门搞的开源软件，多达 50 多种，而整个工程师团队不过 30  人，追求新技术本质上并没有错，但学多难精，而且还有很多的新技术如同昙花，刚开始时耀眼夺目，时间一长没有了维护了，就逐步退出了历史舞台或被取代。大家黄金的学习时间区区只有十几年，如果方向选错了，不仅时间上是种浪费，对你的 IT 之路也是一种不小的打击。

首先，在广度学习的选择上必须要遵守 “ 从众 ” 原则，也就是学的人越多，用的公司越多，越是要加入你的学习列表。那如何知道呢？Github 给我们提供了很好的依据，以开源软件 ceph 为例，如图所示

大家可以看到，目前它的 commits 数量高达 11.1 万，而 Stars 有 7.8k ，也就是点赞的有 7800 次之多，要知道 Github 是不会有 “  水军 ” 这种职业存在的（就算有也无人付工资啊），所以这个数字基本上可以反映真实情况。而且 4  小时前发生过更新，开发者更新积极。下面来对比另一个项目。

同样是分布式存储的项目， glusterfs 的 commits 数量和 stars 数量就比 Ceph 少很多，而且更新是在昨天，开发者更新不太积极。

#### **2.1.3 实势选型 – “T” 字之整合**

当今技术发展日新月异，我们一定要多看看外面，避免闭门造车，必须根据外界流行的趋势不断调整自己的方向，将 “ T” 字整合起来一起应用。

对于应用架构而言，开始时候，大家熟悉的是三段架构，前端是应用层，大都是放置 tomcat ，中间是逻辑层放置 jar 包为主的 app 和 redis 缓存，后端是数据库层。而后是 Duddo  分布式框架，再然后进化到以 SpringCloud 、 Springboot 为主的微服务框架。

对于承载架构而言，开始时候还是服务实体机或 vmware 虚机，而后 Openstack 的兴起打破了 vmware 的垄断，但是 Openstack  毕竟是个半成品，而且本身发展也有诸多问题，于是应容器化技术 docker 、 Kubernetes 技术异军突起。

而后的应用架构和承载架构整合起来了，形成了基于容器的微服务架构 istio ，所有的应用装载在容器的 pod 中、 pod 中又有 sidecar 边车来控制东西向流量，构成了服务网格（ Service Mesh ）。

当所有人认为这已经是架构演变的最终形态时，又出现了 Serverless 模式，无服务架构，这种服务架构，更为节省资源，容器在 pod 在用时才建，不用就销毁。

对应于如此变革，你不需要如何快的变，但你需要知道外面的技术形势是如何的，根据外界的形势，及时修正自己的学习方向和计划，学无止境，学习是一辈子的事，无论你的年龄，无论你的位置。

### **2.2 学习方法**

在学校学习我们是根据大纲，按照书本按部就班的来学习。但工作了要学习 IT 技术，就没这么 “ 好 ” 的条件了。

#### **2.2.1 文档学习**

首先，“好”的书本是不存在的，因为技术变化太快的。记得前两年，我“立志”学习大数据买了一本一年前的 hadoop 大数据的书，不料看了网上的资料都与书中不同，原来书中 hadoop 版本早已淘汰。

所以，我们要靠最新的资料来学习，对应开源软件最好的学习资料就是 document 文档，而文档大都是英文的，大家可以借助于翻译软件翻阅学习。

#### **2.2.2 实践学习**

光说不练总是不对的，所以，建立自己的一个实践平台是很有必要的。这里我们可以用 vmware 或 vbox 虚拟机平台，当然如果有条件，也可以用公有云上的资源，总之，搞运维的 IT 人，必须有环境进行实操。

每个文档基本上都是这个软件的最佳实践，最好就是把最佳实践都在自己的虚拟机环境中运行一遍，形成自己的初步经验。

#### **2.2.3 学以致用**

有很多时候，学过的的东西，如果不用，过段时间就会忘。所以在工作中的学习是尤为重要的，从事运维工作的 IT 人应该在平时工作时，努力将所学的东西应用于工作，比如：一个非常 routine 的工作，你就要想如何通过自己学习的 Python  把它变得自动化，这就是学以致用。这样一来有成就感，二来学过的也不容易忘。

**2.2.4 以教代学**

虽然有实验环境实践、工作上也尽量使用，但总有些软件，我们没有办法持续接触，那怎么办呢？我们可以用把所学的东西整理一下放在博客上，放在知乎上，这个有时并不是给别人看，是为了自己的学习，应该总结整理过的东西，是非常有逻辑性的，而有逻辑性的东西印象就深刻。

更进一步，可以尝试把所学的编制成 ppt 教材，尝试对着镜子讲出来，我的经验是，知识讲的过程才能发现你理解上的不足，讲过几遍后，就完全是你自己的东西了。

#### **2.2.5 不求甚解**

有很多人学习中容易犯的一个毛病就是“期望过高”，他们往往看文档前立下大志向，但很快发现有些东西难以理解，就中途放弃了。知识体系的复杂有时是前后关联的，文档并不是学校的课本，不可能循序渐进。所以正确的学习姿势应该是先通读，后精读。遇到不懂的地方，先查百度，查谷歌，尽力理解，实在不行，就要暂时放弃，去读后面的篇章，等通读完成后，精读时，再反过来解决先前不懂的东西。

### **2.3 “懒人”至上**

做运维工作，必须有懒人的意识，就是尽量“减少”自己的工作量。当然，我们这里绝对不是鼓励大家逃避工作，推卸责任；而是鼓励大家用先进的工具技术，提高工作效率，避免重复无谓的工作。

#### **2.3.1 ITIL 流程规范**

没有规矩不成方圆，有些公司的运维做的不好，不是因为技术不行，而是没有好的规范。这点我是有切身感受的。我经历过一家公司，内部的运维岗位基本上是受虐的，每天忙的不可开交，而且还要承接老板安排的即时工作，基本就是随叫随到，这样导致运维力量消耗厉害，运维工作很难开展。

正确的方法是按照 ITIL 的规范，将工作和责任分散到不同的部门，一切工作按相对固定的流程办事，建立发布流程、变更流程、事件处理流程、问题处理流程等等，这样才能有章可循。记住把工作安排好，工作量合适，也是一种能力。

#### **2.3.2 DevOps 理念和工具**

DevOps  的目的是将一个项目的发起、设计、开发、质量测试、安全检查、部署等流程完全标准化、自动化、流程化，把运维、开发、项目管理人员紧密配合，无缝衔接，最终达到端到端的应用交付。这是当前运维领域，比较流行的理念。运维人员必须对此了解，认知并接受，把自己从一个纯粹的运维，变成运维开发人员。也就是说，将来的运维人员，并不是天天如同救火队一样的，去解决问题；而是去搭建维护一个平台，来承载项目管理、持续集成持续部署、自动化质量测试等工作。



## **3、未来运维展望**

运维的未来会是如何呢？其实 DevOps 的出现已经给大家一个很好的启示，未来运维必然是在 DevOps 的基础上继续走下去。

### **3.1 人才方向 – SRE 转型**

在传统 Ops 模式上的主要问题是：过分关注如何解决那些常规问题、紧急问题，而不是找到根本原因和减少紧急事件的数量。

SRE 是 google 提出的运维工程师理念，是一套方法论体系。全称叫 Site Reliability Engineer ( 网站可靠性工程师 ) 。一个 SRE  工程师基本上需要掌握很多知识：算法，数据结构，编程能力，网络编程，分布式系统，可扩展架构，故障排除，这些也就是我在上面让大家技术选型和学习之路。

SRE 工程师，要做到以下四点：

#### **3.1.1 拥抱风险**

把风险识别出来，用 SLI/SLO 加以评估、度量、量化出来，最终达到消除风险的目的。

#### **3.1.2 质量目标**

一般可能认为没有故障就是正常，万事大吉了。SRE 要求明确定义 SLI 、 SLO ，定量分析某项服务的质量，而监控系统是 SRE  团队监控服务质量和可用性的一个主要手段。通过设立这样的指标，可量化质量，使得我们有权力 PK 业务研发，也能跟老板对话，取得更大的话语权。

#### **3.1.3 减少琐事**

SRE 理念里讲究要花 50% 左右的时间在工程研发上，剩余 50%  的时间用来做一些如资源准备、变更、部署的常规运维，以及查看和处理监控、应急事务处理、事后总结等应急处理工作。如果一个屏幕上十几个窗口，各种刷屏，但却不彻底解决问题，这时就需要用更好的方式 —— 自动化、系统化、工具化的方式 , 甚至是自治的方式去处理这些 “ 琐事 ” 。

这里对传统运维的思维也有一些挑战，因为我们日常做得最多的工作在 SRE 中是被定义为价值不高的琐事，运维不是操作， “ 运维 ” 是个工作内容，人工或是软件都可以做。

在谷歌里，会要求 SRE 有能力进行自动化工具研发，对各种技术进行研究 ，用自动化软件完成运维工作 ，并通过软件来制定、管理合理 SLI/SLO 。

#### **3.1.4 工程研发**

我个人理解的工程研发工作包括三个方面：

a 、推进产品研发改进架构，经常和研发探讨架构、集群、性能问题。

b 、引入新的运维技术，基础组件要 hold 住，比如 TSDB 、 Bosun 、 Consul 、 Zipkin 等。

c 、自研工程技术、平台、工具、系统、基础组件、框架。

### **3.2 运维方向 – AIOps 之路**

AIOps 自从 Gartner 于 2016 年提出至今已有一段时间 , ，一直是运维的方向。在顶级互联网及电信企业，已有较多落地，主要包括：精准智能告警、智能异常检测、智能趋势预测、智能化故障处理。这里拿智能化故障处理的两个功能举例。

#### **3.2.1 自愈功能**

运维体系会和人工智能相结合，利用深度学习算法，让这个系统架构有“智慧”，实现自愈功能。比如现在的百度，如何服务器中有几台宕机了，是不用立马去更好换的，系统会自动判断非正常的服务器，然后利用算法去规避这些服务器，然后，尝试通过重启和重新安装系统的方法，自动尝试去恢复这些服务器，恢复后，又能自动加入集群中。

#### **3.2.2 自动工单**

人工智能能够根据故障的特点，来自动填写工单，自动发送到相关的部门去处理解决。这个在电信等大企业中已经有相关应用。



## **4 、结束语**

运维工程师之路，其实是知易行难的，而且通常都是幕后的，其中辛酸难以为人所表，通常都是有 “ 背锅侠 ”  之称，并且，学的东西博而杂，难以统一集中。所以，我根据自身的经历提出几点考量，提供给大家借鉴，帮助大家努力把自己塑造成为一名合格的、适合未来需要的运维工程师。