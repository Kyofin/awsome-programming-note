# headless浏览器puppeteer快速体验

> Puppeteer is a Node library which provides a high-level API to control Chrome or Chromium over the [DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/). Puppeteer runs [headless](https://developers.google.com/web/updates/2017/04/headless-chrome) by default, but can be configured to run full (non-headless) Chrome or Chromium.

## 安装puppeteer

进入新建的pupeteer目录下运行命令安装

```shell
npm install puppeteer --ignore-scripts
```

由于墙的问题，这里要跳过下载谷歌浏览器才能安装。



## 下载谷歌浏览器驱动

#### 手动下载谷歌浏览器

```
wget https://storage.googleapis.com/chromium-browser-snapshots/Mac/656675/chrome-mac.zip
```



#### 解压后看到如下应用

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190520131221.png)



#### 使用右键进入包内容，找到命令工具

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190520131323.png)



## 实战应用

### 在pupeteer目录下新建example.js

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({executablePath:"/Users/huzekang/opt/Chromium.app/Contents/MacOS/Chromium"});
  const page = await browser.newPage();
  await page.goto('http://192.168.1.150:8087/#/designBoardArea/%E8%83%A1%E6%B3%BD%E5%BA%B7%E6%9B%B4%E6%96%B0%E7%9A%84%E7%9C%8B%E6%9D%BF?boardId=29&readOnly=1');
  await page.screenshot({path: 'example.png'});

  await browser.close();
})();
```

其中executablePath指向刚才的谷歌浏览器命令工具

goto方法的参数是目录页面



### 运行命令对指定页面截图

```
node example.js
```

成功的话，可以看到目录下多了一个example.png文件



## 官方例子

```
https://github.com/GoogleChrome/puppeteer/blob/master/examples/README.md
```

