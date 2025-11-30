---
title: 停电抓取（二）- Selenium 自动化
date: 2021-10-02
categories:
  - Backend Engineering
tags:
  - 爬虫
  - Selenium
excerpt: "通过 Selenium/Puppeteer 等 Web 自动化框架模拟浏览器抓取停电数据，绕过反爬检测的实现方法。"
aiSummary: "本文承接上文 WebScraper 方案，探讨为何需要从插件化爬虫转向程序化方案：无法完全自动化、依赖图形界面。核心思路是使用 Web 自动化框架（Puppeteer/Selenium）配合模拟浏览器，并通过 CDP 协议屏蔽模拟浏览器的特征参数以绕过反爬检测。文章详细调研了 PhantomJS、Chrome Headless、Puppeteer、Selenium 等技术方案及其区别，并重点讲解了如何隐藏 navigator.webdriver 等关键特征参数（方式一：CDP 注入 JS；方式二：stealth.min.js；方式三：禁用 AutomationControlled 参数）。最后给出 Puppeteer 和 Selenium 的实战代码示例，以及 Linux 环境下使用 Xvfb 部署无界面浏览器的方案。"
---

> 本文仅用于技术学习与研究, 内容均已做脱敏处理(相关网站已以“***”代替)

**目标**：抓取 XX 停电数据，作为一手信息流（Feed 流 / 推贴）
继续 [停电抓取（一）- WebScraper](/posts/power-outage-scraping-1) --> 自动化

<!-- more -->

## 抓取自动化方案思考


**问题 & 思考**：Web Scraper 方式抓取存在的问题

1. 暂无法完全自动化：基于真实浏览器，以插件的方式完成抓取，暂无法实现完全自动化
2. 使用平台：必须是具有图形化界面的系统



**难点 & 目标**

1. 实现自动化的方案——程序 / 编程方式灵活抓取网站内容
2. 不受限于图形化界面的操作系统——可部署于生产环境



**主要思路**

1. 解密 / 破解：瑞数安全，难度较大
2. WEB 自动化框架 / 工具 + 模拟浏览器



综合考虑，使用第 2 种方式：WEB 自动化框架 / 工具 + 模拟浏览器

- **（1）预研目前流行的框架 / 工具进行选择**

  WEB 自动化框架广泛应用于数据爬取领域，通过驱动或浏览器相关协议的形式，提供简易 API，以编程方式去灵活控制 headless / 非 headless 形态下的浏览器，作为载体

- **（2）找到模拟浏览器的特征参数（反爬参数）将其屏蔽**

  根据查询 URL 在真实浏览器和模拟浏览器中的不同响应（即模拟浏览器下 403 xxx），说明模拟浏览器的某些特征被网站反爬锁识别到



## 相关技术调研

WEB 自动化框架/工具 + 模拟浏览器  相关技术介绍



### PhantomJS

- [Github](https://github.com/ariya/phantomjs)：Scriptable Headless Browser
- 提供浏览器环境的命令行接口，类似虚拟 / 无界面浏览器，使用 WebKit 来编译解释执行 JavaScript 代码
- 会把网站加载到内存并执行页面上的 JavaScript，获取页面加载完后展示的数据



### Chrome Headless

- Chrome 的无界面形态，自版本 59 以来，谷歌发布了其 Chrome 浏览器的无头版本
- Headless：模拟浏览器的模式，一种在没有图形界面的情况下操纵和使用浏览器。可通过命令行、编程的方式进行控制，例如完成 WEB 页面测试、网页截图等...
- 相比于现代浏览器，Headless Chrome 更加方便测试 web 应用，例如获得网站的截图，做爬虫抓取信息等。
- e.g. `chrome --no-sandbox --headless --screenshot`

> 与 PhantomJS 不同的是它基于一个普通的 Chrome，而不是外部框架，使其存在更难以检测



### Puppeteer

- [Github](https://github.com/puppeteer/puppeteer)（Stars: 74.7K）：Headless Chrome Node.js API

- Chrome 开发团队在 2017 年发布的一个 Node.js 包，它提供了一个高级 API 来通过 DevTools 协议控制 Chromium 或 Chrome。功能比 PhantomJS 要强大很多

- 无需了解太多的底层 CDP 协议实现与浏览器的通信，通过提供的一系列 API，通过 CDP 控制 Chromium/Chrome 浏览器的行为

- 其它语言：[jvppeteer](https://github.com/fanyong920/jvppeteer)、pyppeteer，维护差

> 缺点：增加了复杂度



**关于 CDP**

- Chrome DevTools Protocol
- 基于 WebSocket，利用 WebSocket 实现与浏览器内核的快速数据通道
- https://chromedevtools.github.io/devtools-protocol/tot/Emulation/
- e.g. F12 开发者工具



### Selenium

- [Github](https://github.com/SeleniumHQ/selenium)（Stars: 22.1K）：A browser automation framework and ecosystem.
- Selenium / WebDriver focuses on cross-browser automation
- Selenium IDE、WebDriver

与 Puppeteer 的主要区别

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/DevToolsProtocol.png)

![image](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/SeleniumProcess.png)



## 模拟浏览器特征探测

模拟浏览器的常见特征参数 & 探测方式 & 解决方式



### 常见特征参数

加密网站常见探测方式 / 属性

**（1）UserAgent**

- 检测操作系统属性和用户浏览器属性

- 参数：`window.navigator.userAgent`

> `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36`



**（2）Plugins**

- 浏览器中的插件数量

- `navigator.plugins.length`



**（3）Language**

- 浏览器 UI 语言 / 用户首选项语言数组

- 参数：`navigator.language` 和 `navigator.languages`



**（4）WebDriver（主要）**

- 参数：`navigator.webdriver`

无论是 Puppeteer 还是 Selenium，都会被认为是 webdriver。这也是很多网站反爬的主要判断属性.



> 更多参数
>
> - [Detecting Chrome Headless](https://antoinevastel.com/bot detection/2017/08/05/detect-chrome-headless.html)
> - [翻译版本](https://mlln.cn/2019/07/05/%E5%89%8D%E7%AB%AF%E5%A6%82%E4%BD%95%E6%A3%80%E6%B5%8BChrome-Headless%E4%B8%8D%E8%A2%AB%E7%88%AC%E8%99%AB%E8%99%90/)



### 探测工具

Headless 探测工具

- https://infosimples.github.io/detect-headless/
- https://bot.sannysoft.com/



### WebDriver 隐藏方式

特征参数 WebDriver 的屏蔽方式：从 Selenium 启动的 Chrome 浏览器中移除 `window.navigator.webdriver` 的几种方法



**~~方式 0（旧）~~**

```
　option.add_experimental_option("excludeSwitches", ['enable-automation'])
```

注：79 及以后的版本失效，除非用老版本



**方式 1**

直接用 JS 代码设置该参数为 `undefined`

```js
Object.defineProperty(navigator, 'webdriver', {get: () => undefined})
```

注：这种方式只能在当前页面修改生效，而且时机是在网页加载完毕之后才加载该 JS



可以通过下面这种方式：调用 CDP Method，传入要修改的 JS 代码

- Method：[addScriptToEvaluateOnNewDocument](https://chromedevtools.github.io/devtools-protocol/tot/Page#method-addScriptToEvaluateOnNewDocument)
- 执行时机：打开模拟浏览器的网站页面，在还没有运行网站 JS 之前，执行该脚本

```python
from selenium.webdriver import Chrome

driver = Chrome('./chromedriver')
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
  "source": """
    Object.defineProperty(navigator, 'webdriver', {
      get: () => undefined
    })
  """
}
```



**方式 2**

stealth.min.js

- 来源于 Puppeteer 的扩展插件
- 工具生成 / Github 获取

使用 `stealth.min.js` 文件，同样用方式二的方式执行该 JS 脚本，其中就包含 WebDriver 的隐藏



**方式 3**

添加参数：`--disable-blink-features=AutomationControlled`

禁用启用 Blink 运行时的功能，该参数也可屏蔽，去掉 webdriver 痕迹



**使用方式建议：** 不同网站检测方法可能会不同，可以用这几种方法分别去尝试，或者组合





## 实践示例

Puppeteer 和 Selenium 抓取 xxxxx 隐藏模拟浏览器特征代码示例

### Puppeteer

依赖环境：

- Node.js
- Chrome / Chromium

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch({
        headless: false,
        args: [
            '--no-sandbox',
            '--disable-blink-features=AutomationControlled', // 方式3
        ],
        dumpio: false
    })
    const page = await browser.newPage()

    // 方式1
    // await page.evaluateOnNewDocument(() => {
    //      const newProto = navigator.__proto__;
    //      delete newProto.webdriver;
    //      navigator.__proto__ = newProto;
    //  });

    await page.on('response', async response => {
        if (response.url().indexOf(`queryOutageNoticeList`) != -1 ){
            var header = response.headers();
            if (header['content-type'] == 'application/json;charset=UTF-8') {
                var text = await response.text();
                console.log(response.url() + ", " + response.status())
                console.log(text + "\n");
            }

        }
    });

    await page.goto("http://www.xxxxx.cn/xxxxx/outageNotice/queryOutageNoticeList" +
        "?orgNo=12407&outageStartTime=2021-11-02&outageEndTime=2021-11-02" + 
        "&scope=&provinceNo=12101&typeCode=&lineName=&anHui=02"
    );
})()
```



### Selenium

> 注意事项：1. 控制抓取频率；2. 注意关闭浏览器资源

依赖环境

- [Selenium Java](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)：CDP 相关 API 只存在于 3.x beta 及 4.x 版本（建议用 4.0.0）
- [Chromedriver](http://npm.taobao.org/mirrors/chromedriver/)：要与 Chrome 版本对应

```java
// 如果配置 Path 就可以不用在这指定 driver 路径
System.setProperty("webdriver.chrome.driver", "/opt/driver/chromedriver");

ChromeOptions options = new ChromeOptions();
options.addArguments("--no-sandbox", "--disable-gpu", "--disable-dev-shm-usage"
   // "--headless",
   // "--disable-blink-features=AutomationControlled" // 方式 3
);

ChromeDriver driver = new ChromeDriver(options);
driver.executeCdpCommand("Page.addScriptToEvaluateOnNewDocument",

    // 方式 1. 屏蔽 `navigator.webdriver` 属性
    // ImmutableMap.of("source",
    //     " Object.defineProperty(navigator, 'webdriver', {get: () => undefined})"
    // )

    // 方式 2: JS 脚本 stealth 屏蔽，包含屏蔽方式 1 中的属性
    ImmutableMap.of("source",
         FileUtils.readFileToString(new File("stealth.min.js File路径"), "utf-8")
    )
);

driver.get("http://www.xxxxx.cn/xxxxx/outageNotice/queryOutageNoticeList" +
        "?orgNo=23411&outageStartTime=2021-11-22&outageEndTime=2021-11-22&scope=" +
        "&provinceNo=23101&typeCode=&lineName=&anHui=02"
);

System.out.println(driver.getPageSource());
driver.close();
```





## Linux 部署 Xvfb

Linux 环境运行 Selenium + Chromedriver：使用 Xvfb

- Xvfb 是一个在类 Unix 系统中运行在内存的显示服务器，让你可以没有连接物理显示设备就能运行图形用户界面程序（比如谷歌浏览器）。许多人用 Xvfb 运行早期版本的谷歌浏览器来做”headless”测试。
- 使用方式：`xvfb-run java -jar test.jar`



## 整体业务流程

![scraper](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/power-scraper-business-process.png)
