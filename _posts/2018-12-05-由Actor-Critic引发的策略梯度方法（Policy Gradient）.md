---
layout:     post
title:      由Actor-Critic引发的策略梯度方法
subtitle:   Policy Gradient
date:       2018-12-05
author:     Hybot
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - AI
    - DRL
    - Actor-Critic
---

Policy-Based方法大家都知道模型可以直接给你一个策略方法，agent按照策略方法进行行动，但是其中的数理原理细节，我之前还真没仔细研究过，现在学习了DQN和AC让我们从结果倒推方法，一起来理解策略制定对于模型的影响与区别.

先抛出结论，策略搜索方法如下图分类：

![](https://github.com/hybug/hybug.github.io/blob/master/img/20181205-pic-1.png?raw=true)


重点聚焦于无模型的策略搜索方法：
随机策略搜索“似乎”更加符合“逻辑”一点，当人类在陌生环境探索的时候，同样也是从随机动作开始的，2014年以前确实大家都在研究随机策略搜索。
确定性策略搜索，与2014年由强化学习算法大牛Silver在论文《Deterministic Policy Gradient Algorithms》提出，开始眼熟了对吧，这就是DPG，后来与DQN结合成了我们熟知的DDPG

让我们从我们现在就会的两个算法里面找一个策略搜索的影子：
DQN：我们都知道DQN为了保证agent有探索环境的能力而使用e-greedy搜索方法，当随机值小于e的时候就使用随机动作，否则就使用网络输出的action。这里面是不是有一点随机策略和确定策略的味道了？
AC：如果你像我一样，先学习的DQN再学习AC的话，你会感到疑惑：AC每次的action输出值都是随机给出的（np.randomchoice or normal.sample）这样相同的状态给出的action值都不一样，这样的网络会有效吗？
bingo，有了疑惑，就可以去找答案了，虽然学习过程中贪了时间，先看了代码，但是之前你“节约”的理解原理的时间，后面还是得还回来的

# 1.随机策略搜索

先上公式：

![](https://github.com/hybug/hybug.github.io/blob/master/img/20181205-pic-2.png?raw=true)

含义是，在状态s下，动作a符合参数为θ的概率分布，比较常用的高斯策略：就是在状态s下，采用的动作a符合均值为mu，方差为sigma的正态分布。
是不是又眼熟了，莫烦老师在AC算法的代码中就是用到的这种策略，如果你在那个时候就又疑问的话，这就是疑问的答案。
那么这样的随机策略搜索带来了什么呢？最直观的感受是，输入相同的状态，每次得到的动作也有可能不同，当然，在使用高斯策略的时候，相同的策略在相同的状态s下，采用的动作虽然不同，但是差别也不是特别大，这一点是由高斯分布的特性决定的。

## 1.1 随机策略的缺点


* **需要采样的空间大，算法效率低**

随机策略的梯度计算公式如下：

![](https://github.com/hybug/hybug.github.io/blob/master/img/20181205-pic-3.png?raw=true)

可以体验到，AC在gym各种简单的游戏中，依然需要很长时间才能收敛。原因是梯度是Q值的期望，**因为action的值有一定的随机性，所以期望的求解需要对状态分布和动作分布进行积分运算**，因此需要采样大量样本。

## 1.2 随机策略的优点


* **探索和改进同步进行**

通过探索产生大量的数据，模型从这些数据中学习到知识来改进策略

# 2.确定性策略搜索

* **算法效率高**
动作确定，梯度存在就可以直接进行求解

* **没有探索环境能力**
既然相同的状态和策略输入得到的动作永远是固定的，那么策略产生的“agent行动轨迹”也不会改变，agent就无法探索其他状态的可能性，这对于强化学习来说是致命的，RL的本质就是探索各种动作，然后在其中选择最好的解

**确定性策略既然没有探索能力，如何实现强化学习呢？**

答案是off-policy。熟悉的名词又出现了，谈到off-policy还有什么能比我们大名鼎鼎的DQN更又代表性呢。
什么叫off-policy，就是离线策略。行动策略和评估策略并不是同一个策略，其中行动策略是随机策略，而评估策略是确定性策略，这样我们的agent不但拥有了探索学习能力，也不需要通过计算耗时的积分来获得梯度更新值。

AC的确定性策略梯度如下：

![](https://github.com/hybug/hybug.github.io/blob/master/img/20181205-pic-4.png?raw=true)

少了对动作的积分，多了Q值（回报函数）对动作的导数

AC的off-policy策略梯度如下：

![](https://github.com/hybug/hybug.github.io/blob/master/img/20181205-pic-5.png?raw=true)

这就是DPG，换而言之，就是随机策略的AC+Q-Learning

# 3.DDPG

DDPG--深度确定性策略，都加上深度了大家都知道怎么做了吧。

RL变成DRL，深度网络数据输入都是要求独立同分布的，然而符合马尔可夫性的强化学习输入的数据明显非独立同分布的。DRL使用了两个trick来解决：经验回放和独立的目标网络。这里不做多余介绍。

照本宣科，DDPG也需要这两个trick，其中经验回放和DQN一模一样，不做介绍。
不一样的是DDPG如何更新目标网络，直接给出结果（推导晕了）：

![](https://github.com/hybug/hybug.github.io/blob/master/img/20181205-pic-6.png?raw=true)

我们只需要对θ和w进行更新就ok啦。

# 4.参考资料

[https://zhuanlan.zhihu.com/p/26441204](https://zhuanlan.zhihu.com/p/26441204)
[https://morvanzhou.github.io/tutorials/machine-learning/reinforcement-learning/6-1-actor-critic/](https://link.jianshu.com/?t=https%3A%2F%2Fmorvanzhou.github.io%2Ftutorials%2Fmachine-learning%2Freinforcement-learning%2F6-1-actor-critic%2F)
