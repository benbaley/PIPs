---
PIP:  11
Topic: 零出块惩罚规则优化
Author: luowei
Status: Final
Type: Requirement
Description: 对零出块的节点只锁定一定期限不解除质押
Created: 2020-07-16

---

# PIP-11：零出块惩罚规则优化

## 背景

对于诚实的节点，会经常遇到由于网络原因或者其它非恶意原因导致节点错过出块窗口，而系统对零出块的行为不仅扣除部分质押金还解除节点的质押，这样导致很多非恶意节点积累的大量委托关系失效，导致受损严重。

## 目的

保留节点的委托关系，不让委托关系失效，这样无论对于接受委托多的节点还是少的节点都是一样的规则，更公平。

## 内容

对于零出块惩罚新增一个治理参数：`ZeroProduceFreezeDuration`

**参数意义：** 表示节点零出块被处罚后，需要被锁定的时间，单位是结算周期数

**参数默认值：** 20（锁定二十个结算周期后恢复正常状态）

**参数范围：** 1<= x <`UnStakeFreezeDuration`（解质押的冻结期）

新的零出块惩罚将优化以下逻辑：

- 删除备选节点资格（不参与101候选人的选举）
- 扣除一定处罚金（默认250个区块奖励）
- 当前结算周期原计划用于分红的委托奖励，将退回给节点自己
- 从当前结算周期的101候选人列表中移除
- 增加一条锁定期的信息，节点将被锁定`ZeroProduceFreezeDuration`个结算周期
- 在锁定期间节点状态处于无效状态
- 锁定期到期后节点状态将恢复为正常状态，并加入备选节点列表

**特殊点：**

- 当节点已经零出块惩罚了，不连续进行零出块处罚（在零出块惩罚时节点的当前状态为零出块时不再次进行处罚）
- 如果节点先被零出块惩罚，接着（还处于锁定期）又双签处罚，节点的质押金会被扣除两次，并且先执行零出块的锁定，等待零出块的锁定期到期后，再自动执行双签的锁定并解除质押
- 如果节点先被双签处罚，再被零出块处罚，节点的质押金会被扣除两次，但只执行双签的锁定并解除质押



#### 零出块惩罚的参数默认值调整：

零出块处罚目前由以下2个参数决定：

- 判定期限M（共识周期数）：从节点首次零出块开始，在M个共识周期内未恢复出块，则开始判断是否要处罚节点
- 零出块次数N：  在M个共识周期内，超过N次零出块，则处罚改节点

**说明：**
M值越大，意味着对节点的容忍时间越久， 从网络中移出一个坏节点的时间越长，系统越不稳定
M值可以抽象为系统给节点“改过”的时间窗口，从系统稳定性角度，M越小越好，从节点角度，M越大越好

**修改：**

M值默认：30

N值默认：1
