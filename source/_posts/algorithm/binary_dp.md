---
title: 状态压缩dp
date: 2022-01-20 16:12:03
tags:
- Algorithm
categories: 
- Algorithm
description: 状态压缩dp是采用二进制来枚举状态进而运用动态规划求阶的方法
top_img: 
cover: /image/covers/dp.png
---

# 状态压缩dp

* 将状态用二进制位表示并进行枚举
* 例：状态dp+01背包
  * 糖果店的老板一共有 M 种口味的糖果出售。为了方便描述，我们将 M 种口味编号 1 ∼ M。小明希望能品尝到所有口味的糖果。遗憾的是老板并不单独出售糖果，而是 K 颗一包整包出售。幸好糖果包装上注明了其中 K 颗糖果的口味，所以小明可以在买之前就知道每包内的糖果口味。给定 N 包糖果，请你计算小明最少买几包，就可以品尝到所有口味的糖果。
  * 输入格式
    * 第一行包含三个整数 N、M 和 K。接下来 N 行每行 K 这整数 T₁, T₂, · · · , TK，代表一包糖果口味。
  * 输出格式
    * 一个整数表示答案。如果小明无法品尝所有口味，输出 −1。

```java
import java.util.Scanner;

public class Main {
	public static void main(String[] args){
		int n, m, k;
		Scanner scanner = new Scanner(System.in);
		n = scanner.nextInt();
		m = scanner.nextInt();
		k = scanner.nextInt();
		
		int[] dp = new int[1 << m];
		int[] a = new int[n + 1];
		
		for (int j = 0; j < (1 << m); j++) {
			dp[j] = -1;
		}
		// 初始化很重要
		for (int i = 1; i <= n; i++) {
			for (int j = 0; j < k; j++) {
				int temp = scanner.nextInt();
				a[i] = a[i] | (1 << (temp - 1));
			}
			//System.out.println(a[i]);
		}
		int count = 0;
		
		System.out.println();
		for (int i = 1; i <= n; i++) {
            // 首先每次将新增糖果口味状态赋值为1保证最小
            dp[a[i]] = 1;
			for (int j = 0; j < (1 << m); j++) {
				if (dp[j] == -1) continue;
				int to = j | a[i];
                // 如果能够凑够味道的地方之前没有，则为dp[j] + 1
				if (dp[to] == -1) dp[to] = dp[j] + 1;
                // 否则取最小值
				else dp[to] = Math.min(dp[to], dp[j] + 1);
			}
			for (int j = 0; j < 1 << m; j++) {
				if (dp[j] == -1) System.out.print("无 ");
				else System.out.print(dp[j] + " ");
			}
			System.out.println();
		}
		System.out.println(dp[(1 << m) - 1]);
	}
}

```

