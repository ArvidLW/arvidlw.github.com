# elasticsearch 脑裂原理与测试

### 参考：

[脑裂现象-github issues](https://github.com/LiangShang/liangshang.github.com/issues/18)

[minmum阻止脑裂原理-github issues](https://github.com/LiangShang/liangshang.github.com/issues/19)

[如何防止脑裂](http://fireinwind.iteye.com/blog/2300585)

[es原理](http://www.cnblogs.com/xyniu/p/8930313.html)

[es zenDiscover源码分析](http://www.opscoder.info/es_discover.html)

[避免es脑裂](https://www.jianshu.com/p/2ed01f0eea9c)

## 脑裂出现可能情况

- 与master通信中断，节点重新选主
  - 全gc使得master无响应
  - 网络闪断
  - 负载过大，无法响应

![es-nodes](https://cloud.githubusercontent.com/assets/4581326/16606157/2282c0c0-4368-11e6-8101-75987da0a93f.png)

![es-nodes2](https://cloud.githubusercontent.com/assets/4581326/16606239/dd8a6b0c-4368-11e6-8ca5-7110e0e84a66.png)

## 解决方式

- 角色分离，以免多个角色在同一个节点，使得大规模gc时节点使去响应
- ping通节点，超时等待，默认为3秒，可以适当增大。discovery.zen.ping_timeout:3
- discovery.zen.minimum_master_nodes:n, n的值推荐为:可用master节点数/2+1. 这样当可用master节点大于等于该值，且都认为主master节点挂了则时进行重新选举。

## 原理分析

![master职责](http://opscoder.info/static/img/es_discover3.jpg)

## MasterFaultDetection && NodesFaultDetection

- MasterFaultDetection

- NodesFaultDetection



## ZenDiscover

- ping：用于node之间发现
- unicast: 通过配置`discovery.zen.ping.unicast`对列表中的node单播；
- master election: master选举
- fault detection: 用ping的方式确定节点是否在集群里
- cluster state updates: 在master上有效，向其它节点发布集群状态信息
- no master block: 通过配置`discovery.zen.no_master_block`来决定当没有active的master时，是否拒绝某些操作；

## 脑裂测试

### 环境一

- master: 2个
- data: 2个
- client与master一起

