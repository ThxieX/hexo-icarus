---
title: 蓝桥杯练习 - 大数字的读法
date: 2014-11-13 00:00:00
categories:
  - 算法
tags:
  - 蓝桥杯
  - Archived
excerpt: "将阿拉伯数字转换为汉语拼音读法，如 1234567009 读作 '十二亿三千四百五十六万七千零九'。"
aiSummary: "本文介绍大数字中文读法的实现方案，通过按数位（十亿、亿、千万、百万、十万、万、千、百、十、个）逐位处理。代码使用数组存储输入数字，通过 switch 语句将0-9映射为对应汉字拼音，根据数位添加相应的单位，读数时需注意零的处理和连续零的省略规则。"
---


### 问题描述：

> Tom教授正在给研究生讲授一门关于基因的课程，有一件事情让他颇为头疼：一条染色体上有成千上万个碱基对，它们从0开始编号，到几百万，几千万，甚至上亿。
> 
>
> 比如说，在对学生讲解第1234567009号位置上的碱基时，光看着数字是很难准确的念出来的。
>
> 所以，他迫切地需要一个系统，然后当他输入12 3456 7009时，会给出相应的念法：
> 十二亿三千四百五十六万七千零九
>
> 用汉语拼音表示为
> shi er yi san qian si bai wu shi liu wan qi qian ling jiu
>
> 这样他只需要照着念就可以了。
>
> 你的任务是帮他设计这样一个系统：给定一个阿拉伯数字串，你帮他按照中文读写的规范转为汉语拼音字串，相邻的两个音节用一个空格符格开。
>
> 注意必须严格按照规范，比如说“10010”读作“yi wan ling yi shi”而不是“yi wan ling shi”，“100000”读作“shi wan”而不是“yi shi wan”，“2000”读作“er qian”而不是“liang qian”。

### 输入格式：

> 有一个数字串，数值大小不超过2,000,000,000。

### 输出格式：

> 是一个由小写英文字母，逗号和空格组成的字符串，表示该数的英文读法。

### 样例输入：

> 1234567009

### 样例输出：

> shi er yi san qian si bai wu shi liu wan qi qian ling jiu

<!-- more -->

### 实现：

```java
import java.util.Scanner;

public class ReadNumber {
	
	public static void main(String[] args){
		Scanner scanner = new Scanner(System.in);
		String str = scanner.next();
		int len = str.length();
		int n = len;
		int m = n;
		int[] arr = new int[n]; //该数组保存输入的数字
		StringBuffer sb = new StringBuffer(); //该缓冲区存放 数字的输出
		
		/* 将输入的字符串数字 截取 放到整数类型arr数组中 方便操作 */
		for(int i=0; i<len; ++i){
			if(i < n-1)
				arr[i] = Integer.parseInt(str.substring(i, i+1));
			else
				arr[i] = Integer.parseInt(str.substring(n-1));
		}
		
		for(int i=0; i<len; ++i){
			
			if(m == 10){ //十亿
				if(arr[i] == 1)
					sb.append("shi ");
				else
					sb.append("er shi ");
			}
			
			if(m == 9){ //亿
				if(arr[i] > 0){
					sb.append(check(arr[i]) + " yi ");
				}else{
					if(arr[i]==0){
						sb.append(" yi ");
					}
				}
			}
			
			if(m == 8){//千万
				if(arr[i] > 0){
					sb.append(check(arr[i]) + " qian ");
				}else{
					if(arr[i]==0 && arr[i+1]!=0){
						sb.append("ling ");
					}
				}
			}
			
			if(m == 7){//百万
				if(arr[i] > 0){
					sb.append(check(arr[i]) + " bai  ");
				}else{
					if(arr[i]==0 && arr[i+1]!=0){
						sb.append("ling ");
					}
				}
			}
			
			if(m == 6){//十万
				if(arr[i] > 0){
					sb.append(check(arr[i]) + " shi  ");
				}else{
					if(arr[i]==0 && arr[i+1]!=0){
						sb.append("ling ");
					}
				}
			}
			
			if(m == 5){//万    
				if(arr[i] > 0){
					sb.append(check(arr[i]) + " wan ");
				}else{
					if(arr[i]==0 && arr[i+1]!=0){
						sb.append("ling ");
					}else{
						//"注意:当千万,百万,十万位都为0且万位不为0时候 不会输出万" 例如:十三亿,二十亿
						if(arr[i-1]!=0 || arr[i-2]!=0 || arr[i-3]!=0)
							sb.append("wan ");
					}
				}
			}
			
			if(m == 4){//千
				if(arr[i] > 0){
					sb.append(check(arr[i]) + " qian ");
				}else{
					if(arr[i]==0 && arr[i+1]!=0){
						sb.append("ling ");
					}
				}
			}
			
			if(m == 3){//百
				if(arr[i] > 0){
					sb.append(check(arr[i]) + " bai ");
				}else{
					if(arr[i]==0 && arr[i+1]!=0){
						sb.append("ling ");
					}
				}
			}
			
			if(m == 2){//十
				if(arr[i] > 0){
					sb.append(check(arr[i]) + " shi ");
				}else{
					if(arr[i]==0 && arr[i+1]!=0){
						sb.append("ling ");
					}
				}
			}
			
			if(m == 1){//个
				if(arr[i] > 0){
					sb.append(check(arr[i]));
				}
			}
			
			m--;
		}//end for 
		
		System.out.println(sb);
	}
		
		
	public static String check(int i){
		
		String s = "";
		switch (i){
		case 1:
			s = "yi";
			break;
		case 2:
			s = "er";
			break;
		case 3:
			s = "san";
			break;
		case 4:
			s = "si";
			break;
		case 5:
			s = "wu";
			break;
		case 6:
			s = "liu";
			break;
		case 7:
			s = "qi";
			break;
		case 8:
			s = "ba";
			break;
		case 9:
			s = "jiu";
			break;
		
		}
		return s;
		
	}
}
```