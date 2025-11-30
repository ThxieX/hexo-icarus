---
title: 蓝桥杯练习 - 报时助手
date: 2014-11-12
categories:
  - 算法
tags:
  - 蓝桥杯
  - Archived
excerpt: "将时间的小时和分钟转换为英文读法，如 3:00 读作 'three o'clock'，5:30 读作 'five thirty'。"
aiSummary: "本文介绍时间转英文读法的实现思路，核心逻辑是分别处理小时和分钟，当分钟为0时加 'o'clock'，否则直接拼接英文数字。代码使用两个字符串数组分别存储0-20和20、30、40、50的英文表示，通过整除和取模运算组合出最终读法。"
---




------

### 问题描述：

> 给定当前的时间，请用英文的读法将它读出来。



>
> 时间用时h和分m表示，在英文的读法中，读一个时间的方法是：
>
> 如果m为0，则将时读出来，然后加上“o’clock”，如3:00读作“three o’clock”。
> 如果m不为0，则将时读出来，然后将分读出来，如5:30读作“five thirty”。
>
> 时和分的读法使用的是英文数字的读法，其中0~20读作：
> 0:zero, 1: one, 2:two, 3:three, 4:four, 5:five, 6:six, 7:seven, 8:eight, 9:nine, 10:ten, 11:eleven, 12:twelve, 13:thirteen, 14:fourteen, 15:fifteen, 16:sixteen, 17:seventeen, 18:eighteen, 19:nineteen, 20:twenty。
> 30读作thirty，40读作forty，50读作fifty。
>
> 按上面的规则21:54读作”twenty one fifty four”，9:07读作”nine seven”，0:15读作”zero fifteen”。

<!-- more -->

### 输入格式：

> 输入包含两个非负整数h和m，表示时间的时和分。非零的数字前没有前导0。h小于24，m小于60。

### 输出格式：

> 输出时间时刻的英文。

### 样例输入：

> 0 15

### 样例输出：

> zero fifteen

### 实现：

```java
import java.util.Scanner;

public class Timer {
	
	public static void main(String[] args){
		System.out.println("Please input hour min:");
		Scanner scanner = new Scanner(System.in);
		int hour = 0;
		int min = 0;
		while(scanner.hasNext()){
			hour = scanner.nextInt();
			min = scanner.nextInt();
			
			String[] time1 = {
				"zero","one","two","three","four","five","six","seven",
				"eight","nine", "ten", "eleven", "twelve", "thirteen", 
				"fourteen", "fifteen", "sixteen", "seventeen", 
				"eighteen", "nineteen","twenty"
			};
			String[] time2 = {"twenty","thirty","forty","fifty"};
			
			if(hour < 20){
				System.out.print(time1[hour]);
				if(min == 0){
					System.out.print(" " + "o'clock");
				}else if(min < 20){
					System.out.print(" " + time1[min]);
				}else{
					System.out.print(" " + time2[min/10 - 2] + " " + time1[min%10]);
				}
			}else{
				System.out.print(time2[0] + " " + time1[hour-20]);
				if(min == 0){
					System.out.print("o'clock");
				}else if(min < 20){
					System.out.print(" " + time1[min]);
				}else{
					System.out.print(" " + time2[min/10 - 2] + " " + time1[min%10]);
				}
			}	
		}
	}
}
```