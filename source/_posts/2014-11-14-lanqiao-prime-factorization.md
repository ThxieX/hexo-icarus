---
title: 蓝桥杯练习 - 分解质因数
date: 2014-11-14 00:00:00
categories:
  - 算法
tags:
  - 蓝桥杯
  - Archived
excerpt: "求区间 [a,b] 内所有整数的质因数分解结果，如 10 = 2*5，12 = 2*2*3。"
aiSummary: "分解质因数是指将一个正整数写成若干个质数相乘的形式。本文介绍了蓝桥杯竞赛中质因数分解问题的解法：对于每个待分解的数 n，从最小的质数 2 开始尝试整除，如果能整除则记录该因数并继续用 n 除以该因数后的商重复此过程，直到商为 1。需要注意 1 不是质因数，输出时需按从小到大排列。代码实现时可以使用简单的因数枚举，时间复杂度约为 O(n*sqrt(b))。"
---


### **问题描述：**

> 求出区间[a,b]中所有整数的质因数分解。


### **输入格式：**

> 输入两个整数a，b。

### **输出格式：**

> 每行输出一个数的分解，形如k=a1*a2*a3…(a1<=a2<=a3…，k也是从小到大的)(具体可看样例)

### **样例输入：**

> 3 10

### **样例输出：**

> 3=3
> 4=2*25=56=2*3
> 7=7
> 8=2*2*2
> 9=3*310=2*5

<!-- more -->

### **提示：**

> 先筛出所有素数，然后再分解。

### **数据规模和约定：**

> 2<=a<=b<=10000

### **实现：**

#### 主函数Main
```java
public class ResolvePrimeFactor {

	public static void main(String[] args) {
		System.out.println("Please input startNum endNum:");
		Scanner scanner = new Scanner(System.in);
		int start = scanner.nextInt();
		int end = scanner.nextInt();

		for (int i = start; i <= end; ++i) {
			System.out.print(i + "=");
			fun(i);
			System.out.println();
		}

	}
}
```

#### 普通方式-循环

> 普通方式, 循环

```java
public static void fun(int n) {
	int k = 2; // --定义一个变量 k

	while (k <= n) {
		if (n % k == 0) {
			System.out.print(k);

			// 若后面还有 项, 输出"*" 后继续判断
			n = n / k;
			if (k <= n) {
				System.out.print("*");
			}
		} else {
			k++;
		}
	}
}
```

#### 递归方式 一
> 递归方法 一: (while ..) 自己写的递归， 略繁琐

```java
public static void recfun(int n) {
	int k = 2;
	while (k <= n) {
		if (n % k == 0) {
			System.out.print(k);
			if (k <= n / k) {
				System.out.print("*");
				recfun(n / k);
				return;
			}
			n = n / k;
		} else {
			k++;
		}
	}
}
```



#### 递归方式 二

> 递归方法二: (for...) 4行代码

```java
public static String recfun2(int n) {
	for (int i = 2; i < n; ++i) {
		if (n % i == 0) {
			return i + "*" + recfun2(n / i);
		}
	}
	return "" + n;
}
```

