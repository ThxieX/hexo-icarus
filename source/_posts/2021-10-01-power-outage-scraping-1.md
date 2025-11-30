---
title: 停电抓取（一）- WebScraper
date: 2021-10-01 00:00:00
categories:
  - Backend Engineering
tags:
  - 爬虫
excerpt: "使用 WebScraper 浏览器插件抓取停电数据，无需代码的可视化爬虫方案。"
aiSummary: "本文介绍使用 WebScraper 浏览器插件抓取停电数据的方法。首先概述 WebScraper 的优缺点：优点是无需代码、基于真实浏览器可规避反爬；缺点是对复杂筛选页支持不佳、结果可能乱序、无法完全自动化。随后详细讲解 Sitemap（JSON 格式配置文件）、Selector（文本/链接/表格/元素滚动/元素点击等选择器）、分页（URL 规律变化或模拟滚动点击）以及结果导出等核心概念。最后通过实战案例，演示如何通过分析筛选参数构建 Request URL，生成 sitemap 脚本导入浏览器插件进行爬取，并上传至服务器的完整流程。"
---


> 本文仅用于技术学习与研究, 内容均已做脱敏处理(相关网站已以“***”代替)


**目标**：抓取 XX 停电数据，作为一手信息流（Feed 流 / 推贴）

<!-- more -->

## 介绍

Making web data extraction easy and accessible for everyone 

- 网站: https://www.webscraper.io/

- 数据爬取（无需代码）

- 浏览器插件



**优点**

- 简单易用，无需懂代码
- 基于真实浏览器的抓取方式，不涉及太多反爬因素制约（但也要控制频率）


**缺点**

- Web Scraper 对复杂筛选页的支持不好
- 爬取结果乱序
- 需要少量人工操作，暂无法实现完全自动化（Scheduler 为收费功能）





## 基本使用



### Sitemap

相当于配置文件，是根据操作自动生成的代码脚本文件，是 JSON 格式，支持导入导出



### Selector

- text：文字
- link：链接 + 文字，例如列表页 + 详情页信息

- table：表格
- element：一个页面多个元素内容

- element scroll down：会自动向下滚动爬取，滚动数据加载的页面（否则有些只能加载一部分）
- element click：元素点击选择，例如需要点击页码按钮或更多按钮来加载新数据的抓取

> 其它：XPath、Regex...



### 页面类型

常见的抓取数据的页面类型

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/webscrapter-pagetype.png)



### 分页

常见的分页方式和对应的抓取方式

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/webscrapter-paging.png)

常见的分页种类：

- 滚动
- 加载更多按钮

- 分页按钮



大致为分为两种方式爬取：

1. URL 有规律变化：可利用 URL 参数，URL 中增加 `[1-10:1]`
   e.g. `https://bj.58.com/danche/pn[1-10:1]`


2. URL 无规律变化
通过选择器 `element scroll`、`element click` 模拟向下滚动和点击"分页按钮"或"加载更多"来实现



### 抓取结果

查看方式

- 支持导出在线预览（Data preview），或下载为 CSV、Excel 结果文件

其它

- 抓取过程的数据结果默认保存在浏览器的 localStorage 中，结果是无序的





## 实践：抓取xxxxx数据



> 网址: http://www.xxxxx.cn/xxxxx/outageNotice/initOutageNotice



**主要难点**：

- Web scraper 对有复杂筛选项的抓取支持不好
- 无法用常规的 Selector 来选取页面元素进行抓取



思路：尝试找到筛选条件的参数，让筛选条件能反应在 URL 链接上

查询参数：

- `provinceNo`、`orgNo`：所在地区 code。e.g. 11102（北京市电力公司）、11403（海淀供电公司）

- `outageStartTime`、`outageEndTime`：停电开始、结束日期

- `typeCode`：停电类型 code

- Other... 



### 1. 构建 Request URL 

```http
GET http://www.xxxxx.cn/xxxxx/outageNotice/queryOutageNoticeList
	?orgNo=12402
    &outageStartTime=2021-10-22
    &outageEndTime=2021-10-22
    &scope=
    &provinceNo=12101
    &typeCode=
    &lineName=
    &anHui=02
```



### 2. 用 URL 去构建 sitemap 脚本

startUrl 数组：数组中每个 URL 表示 1 个地区的，需枚举出所有需要爬取的地区 URL

```json
{
    "_id": "outage",
    "startUrl":
    [
        "REQUEST_URL_1", 
        "REQUEST_URL_2", 
        "REQUEST_URL_3", 
        "..."
    ],
    "selectors":
    [
        {
            "delay": 0,
            "id": "htmls",
            "multiple": false,
            "parentSelectors":
            [
                "_root"
            ],
            "regex": "",
            "selector": "pre",
            "type": "SelectorHTML"
        }
    ]
}
```



### 3. 导入脚本 & 抓取

将 sitemap 脚本导入谷歌浏览器插件中（开发者工具），进行爬取得到结果 CSV 文件



### 4. 上传

将抓取结果文件上传至服务器



> 其中 1、2 步可以一次性生成，第 3 步需要手动完成
