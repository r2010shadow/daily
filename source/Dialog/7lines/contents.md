[我们是这样崩的 (转载)](https://mp.weixin.qq.com/s/nGtC5lBX_Iaj57HIdXq3Qg)

Original 来还债的 [哔哩哔哩技术] *2022-07-12 12:00* *Posted on 上海*

收录于合集

\#B站48个

\#SRE5个

**至暗时刻**

2021年7月13日22:52，SRE收到大量服务和域名的接入层不可用报警，客服侧开始收到大量用户反馈B站无法使用，同时内部同学也反馈B站无法打开，甚至APP首页也无法打开。基于报警内容，SRE第一时间怀疑机房、网络、四层LB、七层SLB等基础设施出现问题，紧急发起语音会议，拉各团队相关人员开始紧急处理（为了方便理解，下述事故处理过程做了部分简化）。

**初因定位**

**22:55** 远程在家的相关同学登陆VPN后，无法登陆内网鉴权系统（B站内部系统有统一鉴权，需要先获取登录态后才可登陆其他内部系统），导致无法打开内部系统，无法及时查看监控、日志来定位问题。

**22:57** 在公司Oncall的SRE同学（无需VPN和再次登录内网鉴权系统）发现在线业务主机房七层SLB（基于OpenResty构建） CPU 100%，无法处理用户请求，其他基础设施反馈未出问题，此时已确认是接入层七层SLB故障，排除SLB以下的业务层问题。

**23:07** 远程在家的同学紧急联系负责VPN和内网鉴权系统的同学后，了解可通过绿色通道登录到内网系统。

**23:17** 相关同学通过绿色通道陆续登录到内网系统，开始协助处理问题，此时处理事故的核心同学（七层SLB、四层LB、CDN）全部到位。

**故障止损**

**23:20** SLB运维分析发现在故障时流量有突发，怀疑SLB因流量过载不可用。因主机房SLB承载全部在线业务，先Reload SLB未恢复后尝试拒绝用户流量冷重启SLB，冷重启后CPU依然100%，未恢复。

**23:22** 从用户反馈来看，多活机房服务也不可用。SLB运维分析发现多活机房SLB请求大量超时，但CPU未过载，准备重启多活机房SLB先尝试止损。

**23:23** 此时内部群里同学反馈主站服务已恢复，观察多活机房SLB监控，请求超时数量大大降低，业务成功率恢复到50%以上。此时做了多活的业务核心功能基本恢复正常，如APP推荐、APP播放、评论&弹幕拉取、动态、追番、影视等。非多活服务暂未恢复。

**23:25 - 23:55** 未恢复的业务暂无其他立即有效的止损预案，此时尝试恢复主机房的SLB。

- 我们通过Perf发现SLB CPU热点集中在Lua函数上，怀疑跟最近上线的Lua代码有关，开始尝试回滚最近上线的Lua代码。
- 近期SLB配合安全同学上线了自研Lua版本的WAF，怀疑CPU热点跟此有关，尝试去掉WAF后重启SLB，SLB未恢复。
- SLB两周前优化了Nginx在balance_by_lua阶段的重试逻辑，避免请求重试时请求到上一次的不可用节点，此处有一个最多10次的循环逻辑，怀疑此处有性能热点，尝试回滚后重启SLB，未恢复。
- SLB一周前上线灰度了对 HTTP2 协议的支持，尝试去掉 H2 协议相关的配置并重启SLB，未恢复。

 **新建源站SLB**

**00:00** SLB运维尝试回滚相关配置依旧无法恢复SLB后，决定重建一组全新的SLB集群，让CDN把故障业务公网流量调度过来，通过流量隔离观察业务能否恢复。

**00:20** SLB新集群初始化完成，开始配置四层LB和公网IP。

**01:00** SLB新集群初始化和测试全部完成，CDN开始切量。SLB运维继续排查CPU 100%的问题，切量由业务SRE同学协助。

**01:18** 直播业务流量切换到SLB新集群，直播业务恢复正常。

**01:40** 主站、电商、漫画、支付等核心业务陆续切换到SLB新集群，业务恢复。

**01:50** 此时在线业务基本全部恢复。

 **恢复SLB**

**01:00** SLB新集群搭建完成后，在给业务切量止损的同时，SLB运维开始继续分析CPU 100%的原因。

**01:10 - 01:27** 使用Lua 程序分析工具跑出一份详细的火焰图数据并加以分析，发现 CPU 热点明显集中在对 lua-resty-balancer 模块的调用中，从 SLB 流量入口逻辑一直分析到底层模块调用，发现该模块内有多个函数可能存在热点。

**01:28 - 01:38** 选择一台SLB节点，在可能存在热点的函数内添加 debug 日志，并重启观察这些热点函数的执行结果。

**01:39 - 01:58** 在分析 debug 日志后，发现 lua-resty-balancer模块中的 _gcd 函数在某次执行后返回了一个预期外的值：nan，同时发现了触发诱因的条件：某个容器IP的weight=0。

**01:59 - 02:06** 怀疑是该 _gcd 函数触发了 jit 编译器的某个 bug，运行出错陷入死循环导致SLB CPU 100%，临时解决方案：全局关闭 jit 编译。

**02:07** SLB运维修改SLB 集群的配置，关闭 jit 编译并分批重启进程，SLB CPU 全部恢复正常，可正常处理请求。同时保留了一份异常现场下的进程core文件，留作后续分析使用。

**02:31 - 03:50** SLB运维修改其他SLB集群的配置，临时关闭 jit 编译，规避风险。

**根因定位**

**11:40** 在线下环境成功复现出该 bug，同时发现SLB 即使关闭 jit 编译也仍然存在该问题。此时我们也进一步定位到此问题发生的诱因：在服务的某种特殊发布模式中，会出现容器实例权重为0的情况。

**12:30** 经过内部讨论，我们认为该问题并未彻底解决，SLB 仍然存在极大风险，为了避免问题的再次产生，最终决定：平台禁止此发布模式；SLB 先忽略注册中心返回的权重，强制指定权重。

**13:24** 发布平台禁止此发布模式。

**14:06** SLB 修改Lua代码忽略注册中心返回的权重。

**14:30** SLB 在UAT环境发版升级，并多次验证节点权重符合预期，此问题不再产生。

**15:00 - 20:00** 生产所有 SLB 集群逐渐灰度并全量升级完成。

**原因说明**

 **背景**

B站在19年9月份从Tengine迁移到了OpenResty，基于其丰富的Lua能力开发了一个服务发现模块，从我们自研的注册中心同步服务注册信息到Nginx共享内存中，SLB在请求转发时，通过Lua从共享内存中选择节点处理请求，用到了OpenResty的lua-resty-balancer模块。到发生故障时已稳定运行快两年时间。

在故障发生的前两个月，有业务提出想通过服务在注册中心的权重变更来实现SLB的动态调权，从而实现更精细的灰度能力。SLB团队评估了此需求后认为可以支持，开发完成后灰度上线。

 **诱因**

- 在某种发布模式中，应用的实例权重会短暂的调整为0，此时注册中心返回给SLB的权重是字符串类型的"0"。此发布模式只有生产环境会用到，同时使用的频率极低，在SLB前期灰度过程中未触发此问题。
- SLB 在balance_by_lua阶段，会将共享内存中保存的服务IP、Port、Weight 作为参数传给lua-resty-balancer模块用于选择upstream server，在节点 weight = "0" 时，balancer 模块中的 _gcd 函数收到的入参 b 可能为 "0"。

 **根因**



![Image](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754Q3huYZhu2vdTAWBnwDGXmfrARHAoLUf5nYGxdiamicOBbqKP1DBElfRZELAyowSyf9RBGMWnFkCTvg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



- Lua 是动态类型语言，常用习惯里变量不需要定义类型，只需要为变量赋值即可。
- Lua在对一个数字字符串进行算术操作时，会尝试将这个数字字符串转成一个数字。
- 在 Lua 语言中，如果执行数学运算 n % 0，则结果会变为 nan（Not A Number）。
- _gcd函数对入参没有做类型校验，允许参数b传入："0"。同时因为"0" != 0，所以此函数第一次执行后返回是 _gcd("0",nan)。如果传入的是int 0，则会触发[ if b == 0 ]分支逻辑判断，不会死循环。
- _gcd("0",nan)函数再次执行时返回值是 _gcd(nan,nan)，然后Nginx worker开始陷入死循环，进程 CPU 100%。

**问题分析**



\1. 为何故障刚发生时无法登陆内网后台？

事后复盘发现，用户在登录内网鉴权系统时，鉴权系统会跳转到多个域名下种登录的Cookie，其中一个域名是由故障的SLB代理的，受SLB故障影响当时此域名无法处理请求，导致用户登录失败。流程如下：

![Image](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754Q3huYZhu2vdTAWBnwDGXmfcTIicrXh48JOBTKaRqVdzRHiaNdAaISdn2vA6r1bqGhl4RwgwAkgCniaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

事后我们梳理了办公网系统的访问链路，跟用户链路隔离开，办公网链路不再依赖用户访问链路。



\2. 为何多活SLB在故障开始阶段也不可用？

多活SLB在故障时因CDN流量回源重试和用户重试，流量突增4倍以上，连接数突增100倍到1000W级别，导致这组SLB过载。后因流量下降和重启，逐渐恢复。此SLB集群日常晚高峰CPU使用率30%左右，剩余Buffer不足两倍。如果多活SLB容量充足，理论上可承载住突发流量， 多活业务可立即恢复正常。此处也可以看到，在发生机房级别故障时，多活是业务容灾止损最快的方案，这也是故障后我们重点投入治理的一个方向。

![Image](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\3. 为何在回滚SLB变更无效后才选择新建源站切量，而不是并行？

我们的SLB团队规模较小，当时只有一位平台开发和一位组件运维。在出现故障时，虽有其他同学协助，但SLB组件的核心变更需要组件运维同学执行或review，所以无法并行。

\4. 为何新建源站切流耗时这么久？

我们的公网架构如下：

![Image](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754Q3huYZhu2vdTAWBnwDGXmfGySPicz0hG0VdPCdYk0bQqUaUqbiaPwMGmIRErD3kdRNBHc3oZg9E67A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

此处涉及三个团队：

- SLB团队：选择SLB机器、SLB机器初始化、SLB配置初始化
- 四层LB团队：SLB四层LB公网IP配置
- CDN团队：CDN更新回源公网IP、CDN切量



SLB的预案中只演练过SLB机器初始化、配置初始化，但和四层LB公网IP配置、CDN之间的协作并没有做过全链路演练，元信息在平台之间也没有联动，比如四层LB的Real Server信息提供、公网运营商线路、CDN回源IP的更新等。所以一次完整的新建源站耗时非常久。在事故后这一块的联动和自动化也是我们的重点优化方向，目前一次新集群创建、初始化、四层LB公网IP配置已经能优化到5分钟以内。

\5. 后续根因定位后证明关闭jit编译并没有解决问题，那当晚故障的SLB是如何恢复的？

当晚已定位到诱因是某个容器IP的weight="0"。此应用在1:45时发布完成，weight="0"的诱因已消除。所以后续关闭jit虽然无效，但因为诱因消失，所以重启SLB后恢复正常。

如果当时诱因未消失，SLB关闭jit编译后未恢复，基于定位到的诱因信息：某个容器IP的weight=0，也能定位到此服务和其发布模式，快速定位根因。

**优化改进**

此事故不管是技术侧还是管理侧都有很多优化改进。此处我们只列举当时制定的技术侧核心优化改进方向。

 **1. 多活建设**

在23:23时，做了多活的业务核心功能基本恢复正常，如APP推荐、APP播放、评论&弹幕拉取、动态、追番、影视等。故障时直播业务也做了多活，但当晚没及时恢复的原因是：直播移动端首页接口虽然实现了多活，但没配置多机房调度。导致在主机房SLB不可用时直播APP首页一直打不开，非常可惜。通过这次事故，我们发现了多活架构存在的一些严重问题：

**多活基架能力不足**

- 机房与业务多活定位关系混乱。

- CDN多机房流量调度不支持用户属性固定路由和分片。

- 业务多活架构不支持写，写功能当时未恢复。

- 部分存储组件多活同步和切换能力不足，无法实现多活。

  

**业务多活元信息缺乏平台管理**

- 哪个业务做了多活？

- 业务是什么类型的多活，同城双活还是异地单元化？

- 业务哪些URL规则支持多活，目前多活流量调度策略是什么？

- 上述信息当时只能用文档临时维护，没有平台统一管理和编排。

  

**多活切量容灾能力薄弱**

- 多活切量依赖CDN同学执行，其他人员无权限，效率低。

- 无切量管理平台，整个切量过程不可视。

- 接入层、存储层切量分离，切量不可编排。

- 无业务多活元信息，切量准确率和容灾效果差。

  

我们之前的多活切量经常是这么一个场景：业务A故障了，要切量到多活机房。SRE跟研发沟通后确认要切域名A+URL A，告知CDN运维。CDN运维切量后研发发现还有个URL没切，再重复一遍上面的流程，所以导致效率极低，容灾效果也很差。 

所以我们多活建设的主要方向：

**多活基架能力建设**

- 优化多活基础组件的支持能力，如数据层同步组件优化、接入层支持基于用户分片，让业务的多活接入成本更低。

- 重新梳理各机房在多活架构下的定位，梳理Czone、Gzone、Rzone业务域。

- 推动不支持多活的核心业务和已实现多活但架构不规范的业务改造优化。

  

**多活管控能力提升**

- 统一管控所有多活业务的元信息、路由规则，联动其他平台，成为多活的元数据中心。
- 支持多活接入层规则编排、数据层编排、预案编排、流量编排等，接入流程实现自动化和可视化。
- 抽象多活切量能力，对接CDN、存储等组件，实现一键全链路切量，提升效率和准确率。
- 支持多活切量时的前置能力预检，切量中风险巡检和核心指标的可观测。

 **2. SLB治理**

**架构治理**

- 故障前一个机房内一套SLB统一对外提供代理服务，导致故障域无法隔离。后续SLB需按业务部门拆分集群，核心业务部门独立SLB集群和公网IP。

- 跟CDN团队、四层LB&网络团队一起讨论确定SLB集群和公网IP隔离的管理方案。

- 明确SLB能力边界，非SLB必备能力，统一下沉到API Gateway，SLB组件和平台均不再支持，如动态权重的灰度能力。

  

**运维能力**

- SLB管理平台实现Lua代码版本化管理，平台支持版本升级和快速回滚。

- SLB节点的环境和配置初始化托管到平台，联动四层LB的API，在SLB平台上实现四层LB申请、公网IP申请、节点上线等操作，做到全流程初始化5分钟以内。

- SLB作为核心服务中的核心，在目前没有弹性扩容的能力下，30%的使用率较高，需要扩容把CPU降低到15%左右。

- 优化CDN回源超时时间，降低SLB在极端故障场景下连接数。同时对连接数做极限性能压测。

  

**自研能力**

- 运维团队做项目有个弊端，开发完成自测没问题后就开始灰度上线，没有专业的测试团队介入。此组件太过核心，需要引入基础组件测试团队，对SLB输入参数做完整的异常测试。
- 跟社区一起，Review使用到的OpenResty核心开源库源代码，消除其他风险。基于Lua已有特性和缺陷，提升我们Lua代码的鲁棒性，比如变量类型判断、强制转换等。
- 招专业做LB的人。我们选择基于Lua开发是因为Lua简单易上手，社区有类似成功案例。团队并没有资深做Nginx组件开发的同学，也没有做C/C++开发的同学。

 **3. 故障演练**

本次事故中，业务多活流量调度、新建源站速度、CDN切量速度&回源超时机制均不符合预期。所以后续要探索机房级别的故障演练方案：

- 模拟CDN回源单机房故障，跟业务研发和测试一起，通过双端上的业务真实表现来验收多活业务的容灾效果，提前优化业务多活不符合预期的隐患。
- 灰度特定用户流量到演练的CDN节点，在CDN节点模拟源站故障，观察CDN和源站的容灾效果。
- 模拟单机房故障，通过多活管控平台，演练业务的多活切量止损预案。

 **4. 应急响应**

B站一直没有NOC/技术支持团队，在出现紧急事故时，故障响应、故障通报、故障协同都是由负责故障处理的SRE同学来承担。如果是普通事故还好，如果是重大事故，信息同步根本来不及。所以事故的应急响应机制必须优化：

- 优化故障响应制度，明确故障中故障指挥官、故障处理人的职责，分担故障处理人的压力。

- 事故发生时，故障处理人第一时间找backup作为故障指挥官，负责故障通报和故障协同。在团队里强制执行，让大家养成习惯。

- 建设易用的故障通告平台，负责故障摘要信息录入和故障中进展同步。

  

本次故障的诱因是某个服务使用了一种特殊的发布模式触发。我们的事件分析平台目前只提供了面向应用的事件查询能力，缺少面向用户、面向平台、面向组件的事件分析能力：

- 跟监控团队协作，建设平台控制面事件上报能力，推动更多核心平台接入。
- SLB建设面向底层引擎的数据面事件变更上报和查询能力，比如服务注册信息变更时某个应用的IP更新、weight变化事件可在平台查询。
- 扩展事件查询分析能力，除面向应用外，建设面向不同用户、不同团队、不同平台的事件查询分析能力，协助快速定位故障诱因。

**总结**

此次事故发生时，B站挂了迅速登上全网热搜，作为技术人员，身上的压力可想而知。事故已经发生，我们能做的就是深刻反思，吸取教训，总结经验，砥砺前行。

此篇作为“713事故”系列之第一篇，向大家简要介绍了故障产生的诱因、根因、处理过程、优化改进。后续文章会详细介绍“713事故”后我们是如何执行优化落地的，敬请期待。

最后，想说一句：多活的高可用容灾架构确实生效了。
