---
title: 中位数问题（对顶堆的应用）
date: 2022-06-30 15:49:03
tags:
- Algorithm
categories: 
- Algorithm
description: 关于新建可以get中位数的数据结构，和滑动窗口中的中位数问题，均用两个堆实现。
top_img: 
cover: /image/covers/duiding.png
---

# Leetcode295. 数据流的中位数

* 设计一个支持以下两种操作的数据结构：
  - void addNum(int num) - 从数据流中添加一个整数到数据结构中。
  - double findMedian() - 返回目前所有元素的中位数。
* 设计思路：维护两个堆：左侧大顶堆，用于维护比中位数小的元素；右侧小顶堆，用于维护比中位数大的元素；平均值：当两堆元素个数相同，则中位数为堆顶元素平均值；不同时为左侧大顶堆的堆顶值。插入元素时，两堆元素同样多则插入到左侧大顶堆（先插入右侧，弹出堆顶再插入左侧），不同时左侧多，插入到左侧弹出堆顶再插入到右侧。
* **注，less是大顶堆，greater是小顶堆！！！**

```c++
class MedianFinder {
public:
    // 维护大顶堆、小顶堆
    priority_queue<int, vector<int>, less<int>> p1;
    priority_queue<int, vector<int>, greater<int>> p2;

    MedianFinder() {
        
    }

    void addNum(int num) {
        if (p1.empty()) p1.push(num);
        else {
            if (p1.size() == p2.size()) {
                p2.push(num);
                int x = p2.top();
                p2.pop();
                p1.push(x);
            }
            else {
                p1.push(num);
                int x = p1.top();
                p1.pop();
                p2.push(x);
            }
        }
    }

    double findMedian() {
        if (p1.size() == p2.size()) return (p1.top() + p2.top()) * 1.0 / 2;
        else return p1.top() * 1.0;
    }
};
```

# Leetcode480. 滑动窗口中位数

- 给你一个数组 nums，有一个长度为 k 的窗口从最左端滑动到最右端。窗口中有 k 个数，每次窗口向右移动 1 位。你的任务是找出每次窗口移动后得到的新窗口中元素的中位数，并输出由它们组成的数组。
- 思路：对顶堆+延迟删除

- 具体讲解见[滑动窗口中位数 - 滑动窗口中位数 - 力扣（LeetCode）](https://leetcode.cn/problems/sliding-window-median/solution/hua-dong-chuang-kou-zhong-wei-shu-by-lee-7ai6/)

```c++
class DualHeap {
private:
    // 大根堆，维护较小的一半元素
    priority_queue<int> small;
    // 小根堆，维护较大的一半元素
    priority_queue<int, vector<int>, greater<int>> large;
    // 哈希表，记录「延迟删除」的元素，key 为元素，value 为需要删除的次数
    unordered_map<int, int> delayed;

    int k;
    // small 和 large 当前包含的元素个数，需要扣除被「延迟删除」的元素
    int smallSize, largeSize;

public:
    DualHeap(int _k): k(_k), smallSize(0), largeSize(0) {}

private:
    // 不断地弹出 heap 的堆顶元素，并且更新哈希表
    template<typename T>
    void prune(T& heap) {
        while (!heap.empty()) {
            int num = heap.top();
            if (delayed.count(num)) {
                --delayed[num];
                if (!delayed[num]) {
                    delayed.erase(num);
                }
                heap.pop();
            }
            else {
                break;
            }
        }
    }

    // 调整 small 和 large 中的元素个数，使得二者的元素个数满足要求
    void makeBalance() {
        if (smallSize > largeSize + 1) {
            // small 比 large 元素多 2 个
            large.push(small.top());
            small.pop();
            --smallSize;
            ++largeSize;
            // small 堆顶元素被移除，需要进行 prune
            prune(small);
        }
        else if (smallSize < largeSize) {
            // large 比 small 元素多 1 个
            small.push(large.top());
            large.pop();
            ++smallSize;
            --largeSize;
            // large 堆顶元素被移除，需要进行 prune
            prune(large);
        }
    }

public:
    void insert(int num) {
        if (small.empty() || num <= small.top()) {
            small.push(num);
            ++smallSize;
        }
        else {
            large.push(num);
            ++largeSize;
        }
        makeBalance();
    }

    void erase(int num) {
        ++delayed[num];
        if (num <= small.top()) {
            --smallSize;
            if (num == small.top()) {
                prune(small);
            }
        }
        else {
            --largeSize;
            if (num == large.top()) {
                prune(large);
            }
        }
        makeBalance();
    }

    double getMedian() {
        return k & 1 ? small.top() : ((double)small.top() + large.top()) / 2;
    }
};

class Solution {
public:
    vector<double> medianSlidingWindow(vector<int>& nums, int k) {
        DualHeap dh(k);
        for (int i = 0; i < k; ++i) {
            dh.insert(nums[i]);
        }
        vector<double> ans = {dh.getMedian()};
        for (int i = k; i < nums.size(); ++i) {
            dh.insert(nums[i]);
            dh.erase(nums[i - k]);
            ans.push_back(dh.getMedian());
        }
        return ans;
    }
};
```

