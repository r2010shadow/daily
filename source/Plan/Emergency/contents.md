# 应急预案手册

[参照ISO20000与ISO27001](#)

| 时间       | 版本  | 变更     |
| ---------- | ----- | -------- |
| 2022-08-25 | 0.0.1 | 文档初始 |
|            |       |          |
|            |       |          |

[TOC]

## 应急预案手册自述

通过预先设定目标，假设可能存在情况，布置防范措施。处理结束后，应将事件归档保存。

## 应急管理概览

具体实施各项规范时，参考流程，并结合实际情况操作。

### 权责

1. 针对应急情况，设立应急指挥部、应急小组。
2. 从范围角度，应急指挥部主外，应急小组主内。
3. 从功能角度，应急指挥部调度分配，应急小组具体实施。
4. 应急小组分为网络应急、软件应急、硬件应急、火灾水灾应急、供电应急。

## 网络应急处理规范

1. 接到用户故障处理请求，判断是个别电脑发生普通网络故障，通知运维人员处理。
2. 判断为机房网络核心软故障时，进行网络测试确定故障根源并进行修复。预计在10分钟内无法修复，应立即上报并提交处理方案。
3. 判断为机房网络核心软故障，同时预计10分钟内无法修复时，必要时通知供应商远程支援。
4. 在半小时内未能处理故障时，在继续远程支援的同时，安排人员4小时内现场处理。

## 软件应急处理规范

1. 接到用户故障处理请求，判断不是网络故障时，登录相关系统服务器查看是否为软件系统故障。如果是立即处理并通知相关软件管理员配合。
2. 如确定故障为病毒引起，先定位被感染的相关服务器并关闭其网络。马上进行病毒处理，同时在软件管理员配合下启用备用服务器。

## 硬件应急处理规范

1. 接到用户故障处理请求，判断是网络硬件故障时，登录相关网络设备，找出异常设备后尝试重启设备。如无法修复立即更换备机。
2. 判断是服务器硬件故障时，登录相关服务器，找出异常设备后尝试重启，如无法修复立即更换备机。
3. 判断15分钟内不能完全修复故障时，立即上报相关领导并提交相关处理方案。
4. 判断为硬件故障时，通知设备供应商到场处理，并详细描述故障情况。尽可能准确定位故障点，配合供应商收集日志、更换配件或备机。

## 消防应急规范

1. 在发生火灾或水灾时，确保人身安全情况下，应立即切断电源，防止设备进一步损坏。
2. 在确定火灾源头时，使用灭火器进行扑火，同时通知安全部门及相关领导。在确定火势失控时拨打119.
3. 如火势失控，在确保人身安全情况下应立即将相关设备进行撤离。撤离顺序为核心数据服务器-核心网络设备-普通设备。

## 水浸应急规范

1. 在发生水浸时，确保人身安全情况下，应立即切断电源，防止设备进一步损坏。
2. 在切断电源前提下组织人员排水，同时向领导汇报恢复系统的时间。
3. 如水浸严重，应立即将相关设备进行撤离。撤离顺序为核心数据服务器-核心网络设备-普通设备。

## 供电应急处理规范

1. 应急小组立即检查UPS运行情况，跟踪可供电时间；
2. 向供电管理部门咨询断电原因，及需要多久恢复正常供电。
3. 确认非供电部门维护导致，立即检查相关漏电开关，通知电力管理人员到场处理，告知UPS剩余供电时间。
4. 在不能及时修复情况下，根据UPS供电情况在耗尽前1小时关闭对业务不影响或较少的相关设备及服务器。
5. 在UPS耗尽前35分钟，通告并妥善关闭业务相关设备及服务器，使程序正常退出并数据存盘。

## 季度网络及供电演练

1. 测试机房网络核心主备，切断主核心，备核心自动切换。
2. 测试UPS供电，UPS放电验证，承载负载下可用时长验证。
3. 供电主干线路备用线路切换验证，切换到备用供电线路验证可用性。

## 应急清单

| 所属             | 联络人或供应商 | 电话 | 有效响应时间 |
| ---------------- | -------------- | ---- | ------------ |
| 应急指挥部       |                |      |              |
| 机房空调         |                |      |              |
| 城市电力         |                |      |              |
| UPS供电          |                |      |              |
| 防水火保障       |                |      |              |
| 温湿度、视频监控 |                |      |              |
| 机柜             |                |      |              |
| 深信服           |                |      |              |
| 交换机           |                |      |              |
| 软件报障         |                |      |              |


