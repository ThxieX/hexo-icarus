---
title: 蓝桥杯练习 - 十六进制转八进制
date: 2014-11-15
categories:
  - 算法
tags:
  - 蓝桥杯
  - Archived
excerpt: "将十六进制正整数转换为八进制数，利用十六进制转二进制再转八进制的短除法原理实现大数处理。"
aiSummary: "本文介绍蓝桥杯竞赛中十六进制转八进制问题的解法。核心思路是利用进制转换的短除法原理：十六进制转八进制可通过先转二进制（每一位十六进制扩展为四位二进制）、再每三位二进制合并为一位八进制来实现。关键在于处理大数（每行十六进制数长度不超过 100000），需使用字符串而非整数类型进行输入输出。代码使用 Java 实现，通过字符串操作完成进制转换。"
---


### 问题描述：

> 给定n个十六进制正整数，输出它们对应的八进制数。


### 输入格式：

> 输入的第一行为一个正整数n （1<=n<=10）。
> 接下来n行，每行一个由0~9、大写字母A~F组成的字符串，表示要转换的十六进制正整数，每个十六进制数长度不超过100000。

### 输出格式：

> 输出n行，每行为输入对应的八进制正整数。

<!-- more -->

### 实现：

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;

public class Main {
    public static void main(String[] args) throws Exception {
    	BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    	int len = Integer.parseInt(br.readLine());
    	String[] arr = new String[len];
    	for(int i=0; i<arr.length; ++i){
    		arr[i] = br.readLine();
    	}
    	
    	for(int i=0; i<arr.length; ++i){
    		transform1(arr[i]);
    	}
    }
    
    //16-->2
    public static void transform1(String str){
    	char[] c = new char[str.length()];
    	c = str.toCharArray();
    	StringBuffer sb = new StringBuffer();
    	
    	for(int i=0; i<str.length(); ++i){
    		switch(c[i]){
    		case '0':
    			sb.append("0000");
    			break;
    		case '1':
    			sb.append("0001");
    			break;
    		case '2':
    			sb.append("0010");
    			break;
    		case '3':
    			sb.append("0011");
    			break;
    		case '4':
    			sb.append("0100");
    			break;
    		case '5':
    			sb.append("0101");
    			break;
    		case '6':
    			sb.append("0110");
    			break;
    		case '7':
    			sb.append("0111");
    			break;
    		case '8':
    			sb.append("1000");
    			break;
    		case '9':
    			sb.append("1001");
    			break;
    		case 'A':
    			sb.append("1010");
    			break;
    		case 'B':
    			sb.append("1011");
    			break;
    		case 'C':
    			sb.append("1100");
    			break;
    		case 'D':
    			sb.append("1101");
    			break;
    		case 'E':
    			sb.append("1110");
    			break;
    		case 'F':
    			sb.append("1111");
    			break;
    		
    		}
    	}

    	transform2(sb);
    }
    
    
    //2-->8
    public static void transform2(StringBuffer sb){
    	int len = sb.length();
    	if(len%3 == 0){
    		if("000".equals(sb.substring(0, 3)))
    			sb.delete(0, 3);
    	}else if(len%3 == 1){
    		if("0".equals(sb.substring(0, 1)))
    			sb.delete(0, 1);
    		else
    			sb.insert(0, "00");
    	}else if(len%3 == 2){
    		if("00".equals(sb.substring(0, 2)))
    			sb.delete(0, 2);
    		else
    			sb.insert(0, "0");
    	}
    	
    	StringBuffer sb2 = new StringBuffer();
    	int len2 = sb.length()/3;
    	for(int i=0; i<len2; ++i){
    		if("000".equals(sb.substring(3*i, 3*i+3)))
    			sb2.append("0");
    		else if("001".equals(sb.substring(3*i, 3*i+3)))
    			sb2.append("1");
    		else if("010".equals(sb.substring(3*i, 3*i+3)))
    			sb2.append("2");
    		else if("011".equals(sb.substring(3*i, 3*i+3)))
    			sb2.append("3");
    		else if("100".equals(sb.substring(3*i, 3*i+3)))
    			sb2.append("4");
    		else if("101".equals(sb.substring(3*i, 3*i+3)))
    			sb2.append("5");
    		else if("110".equals(sb.substring(3*i, 3*i+3)))
    			sb2.append("6");
    		else if("111".equals(sb.substring(3*i, 3*i+3)))
    			sb2.append("7");
    	}
    	
    	System.out.println(sb2);
    }
}
```

> 输入的十六进制数不会有前导0，比如012A。
> 输出的八进制数也不能有前导0。

### 样例输入：

> 2
> 39
> 123ABC

### 样例输出：

> 71
> 4435274

### 提示：

> 先将十六进制数转换成某进制数，再由某进制数转换成八进制。