---
title: 蓄水池抽样
date: 2022-04-25 20:33:02
tags:
- Algorithm
categories: 
- Algorithm
description: 基于leetcode-398.随机数索引的蓄水池抽样问题。
top_img: 
cover: /image/covers/xushuichi.png
---

# Leetcode 398 随机数索引

## 题目描述

给你一个可能含有 重复元素 的整数数组 nums ，请你随机输出给定的目标数字 target 的索引。你可以假设给定的数字一定存在于数组中。

实现 $Solution$ 类：

- Solution(int[] nums) 用数组 nums 初始化对象。
- int pick(int target) 从 nums 中选出一个满足 nums[i] == target 的随机索引 i 。如果存在多个有效的索引，则每个索引的返回概率应当相等。

## 示例

```c++
//输入
["Solution", "pick", "pick", "pick"]
[[[1, 2, 3, 3, 3]], [3], [1], [3]]
//输出
[null, 4, 0, 2]

//解释
Solution solution = new Solution([1, 2, 3, 3, 3]);
solution.pick(3); // 随机返回索引 2, 3 或者 4 之一。每个索引的返回概率应该相等。
solution.pick(1); // 返回 0 。因为只有 nums[0] 等于 1 。
solution.pick(3); // 随机返回索引 2, 3 或者 4 之一。每个索引的返回概率应该相等。
```

# 蓄水池抽样算法

若 `nums` 并不是在初始化时完全给出，而是持续以「流」的形式给出，且数据流的很长，不便进行预处理的话，我们只能使用蓄水池抽样的方式求解。

对于一般固定大小的数据集合采样，设其样本数量为$N$, 等概率采样$k$个样本，那我们只要通过计算机生成伪随机数就能实现等概率采样。

其间对于每一个样本$x$, 有$\textstyle\dfrac{1}{N}$的概率取或者$1-\textstyle \dfrac{1}{N}$的概率不取，无论是取了还是没取，都是一锤子的买卖，买定离手。而对于数据流采样，你不知道总共有多少麦穗N，所以来了一个样本你也无从得知应该以具体多大概率保留它或者丢弃它。这时，我们可以求助于巧妙的蓄水池抽样算法。

从算法过程可以看出，对于每一个样本来说，采样不再是一锤子买卖，在整个采样过程完全结束之前，没有人是安全的：即便是某个样本当前被采样选中了，但未来还是有可能会被踢出去，没有免死金牌。

所谓买定不离手，是说手里时刻攥着当前已采样的数据，并在后面不断换入换出。

我们规定当遇到第 $i$个样本，执行一次 $[0,i)$ 的随机操作，当随机结果为0~k时，发生概率为 $\dfrac{k}{i}$，我们将该坐标作为最新的答案候选。

当对每一个样本都进行上述操作后，容易证明每一位下标返回的概率均为 $\dfrac{k}{N}$。

对于第$i$个样本，最后被选取的概率为：

$P=\dfrac{k}{i}*\dfrac{i}{i+1}*\dots*\dfrac{N-1}{N}=\dfrac{k}{N}$

# 代码

```c++
class Solution {
    vector<int> &nums;
public:
    Solution(vector<int> &nums) : nums(nums) {}

    int pick(int target) {
        int ans;
        for (int i = 0, cnt = 0; i < nums.size(); ++i) {
            if (nums[i] == target) {
                ++cnt; // 第 cnt 次遇到 target
                if (rand() % cnt == 0) {
                    ans = i;
                }
            }
        }
        return ans;
    }
};
```

