---
title: q-learning
date: 2021-02-24 10:53:22
tags: 算法
mathjax: true
---

强化学习并不新鲜。

<!-- more -->

2017年AlphaZero出来之后，一堆自媒体狂吹强化学习有多么牛b，利用DQN吊打普通的AlphaGo（说来惭愧，那阵自己还关注新智元和机器之心这种公众号）。

尽管好奇，可由于课程等诸多原因，一直没有闲暇的时间关注RL，直到2021年需要在研究领域用到强化学习算法，就简单了解和实践了一下，发现这个算法的核心思想其实也蛮简单，没有各路自媒体吹的那么神奇，于是将自己的一些理解简单记录一下。

## 智能体和环境

人们总是希望创造出一种通用智能体，能够通过和环境不断的交互，在实现目标的过程当中，自己总结经验，得出与环境交互最优的方法。

这种智能体与人类的思维模式很类似，因为从婴儿牙牙学语到高中生根据高考大纲有意识培养自己的应试能力，人类一直在做的事情就是适应环境，找到在当前环境下解决问题的最优方式。

但是环境是复杂多变的，经验是抽象的，与环境的交互是有代价的。所以对于计算机来说，如何用比特流表征经验，环境和代价是一个困难的问题。

**其实到这里，**强化学习的概念就算是介绍完了，强化学习就是智能体实现和环境智能交互的方法。剩下的就是经验，环境以及最优如何被形式化地定义的问题。

总结一下，

    1. 强化学习的最终目标是使得智能体面对复杂环境能够实现某种决策方案，使得某个目标最优。

    2. 强化学习算法已知的信息只有智能体和环境交互的规则和代价。

    3. 强化学习的难点在于如何表征经验，环境，定义最终目标以及智能体更新“经验”的算法。

## 数学表示

环境和智能体整体状态的集合表示：$s \in S$，每个状态$s$都包含环境和智能体两者的状态信息。

智能体和环境交互动作的集合表示：$a \in A$

智能体采取某种动作后整体状态发生变化，变化可能不确定，用概率分布表示：$P_a(s, s^{'}) = Pr(s_{t+1} = s^{'} | s_t = s, a_t = a)$，表示时刻t智能体执行动作a后整体状态从s到$s^{'}$的概率。

智能体和环境交互后会有一个反馈：$r_a(s, s^{'})$，表示执行动作a，整体状态从s到$s^{'}$后，智能体得到的反馈R，我们假设这个R为标量，值越大动作策略越好。由于在状态s下执行动作a迁移到的状态不确定，所以$R_a(s) = \sum_{s^{'}\in S} P_a(s, s^{'})r_a(s, s^{'})$

**优化目标**的定义：首先我们定义智能体和环境的交互策略$\pi$，事实上$\pi$就是我们要优化的变量，即**不同环境状态下该如何选择动作**。在交互策略$\pi$下，当前状态的累计收益为

$$V^{\pi}(s) = E[R] = E[\sum_{t=t_0}^{\inf}\gamma^{t}r_t|s_0=s]$$

其中$\gamma$是衰减因子，最终的优化表达式为$V^{*}(s) = \max_{\pi}V^{\pi}(s)$

## q-learning

由于探索环境的过程复杂，精确求解$V^{*}(s)$似乎比较困难，有人提出了q-learning算法。

所谓q-learning就是将$\pi$用一张具有动作和状态两个维度的二维表表示，称为q。智能体处于一个状态时，衡量该状态采取不同动作得到的q值大小，以高概率选择q值大的动作和环境交互。注意q和r的区别：r只代表了当前收益，而q包含了未来的收益。每次执行过后，用如下公式更新q表，

$$Q^{new}(s_t, a_t) \leftarrow Q(s_t, a_t) + \alpha (r_a(t) + \gamma \max_{a}Q(s_{t+1}, a) - Q(s_t, a_t))$$

其中$\alpha$表示学习率，值越大更新q表速度越快，q表就是智能体和环境交互的策略。这个公式不难理解，将当前的收益和未来的收益全部统计进入q表，因为未来的收益需要递归展开，而未来的收益难以确定，这里用当前q表里的$\gamma \max_{a}Q(s_{t+1}, a)$来表征未来收益，从而简化递归过程。

## 简单例子

环境是一条长度20的赛道，终点在赛道右侧。智能体可以选择的动作为向左或向右，初始阶段智能体位于赛道最左侧，目标是达到赛道终点。

显然达到这个目标的算法非常简单，智能体直接向右走，判定走过的点是不是终点就行，但是现在智能体不会设计这样的算法，只希望尽快达到终点。假设智能体每走一步r=-0.5，达到终点r=1，我们可以令智能体在和环境交互的过程当中获得经验。

```python
def q_learning():
    q_table = build_q_table(N_STATE, ACTIONS)
    step_counter_times = []
    for episode in range(MAX_EPISODES):
        state = 0
        is_terminal = False
        step_counter = 0
        update_env(state, episode, step_counter)
        while not is_terminal:
            action = choose_action(state, q_table)
            next_state, reward = get_env_feedback(state, action)
            next_q = q_table.loc[state, action]
            if next_state == 'terminal':
                is_terminal = True
                q_target = reward
            else:
                delta = reward + GAMMA*q_table.iloc[next_state,:].max() - q_table.loc[state, action]
                # crucial step
                q_table.loc[state, action] += ALPHA*delta
            state = next_state
            is_terminal, steps = update_env(state, episode, step_counter+1)
            step_counter += 1
            if is_terminal:
                step_counter_times.append(steps)

    return q_table, step_counter_times
```

这个q表大小2*20，但是足够了。首先选择动作，获得反馈后更新q表，重复此过程直到达到终点。这为一个episode，然后反复训练这个智能体更新q表，这个q表就反映了智能体获得的“经验”。

这个例子非常简单，也方便实现，但是当我们的状态数量非常多，难以用一张表表示，这该怎么办呢？

## DQN

待续
