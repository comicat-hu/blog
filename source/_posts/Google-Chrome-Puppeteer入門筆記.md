---
title: Google Chrome Puppeteer入門筆記
date: 2017-11-22 17:12:21
tags:
  - NodeJS
---
Puppeteer is a Node library which provides a high-level API to control headless Chrome over the DevTools Protocol. It can also be configured to use full (non-headless) Chrome.

官方GitHub: <https://github.com/GoogleChrome/puppeteer>

Puppeteer 最低支援到Node v6.4，但是官方提供的範例必須要使用Node v7.6以上，目前Node穩定版已經推到v8，所以我裝v8.8.1

在專案中`npm i puppeteer`就可以安裝

主要是基於chromium來模擬各種動作，

<!--more-->

以下提供幾個功能程式片段:

## 截圖

```JS
const puppeteer = require("puppeteer");

var screenshot = async(url, filename) => {
    // 啟動瀏覽器
    const browser = await puppeteer.launch({
        headless: false
    });
    // 開新分頁
    const page = await browser.newPage();
    // 連接url
    await page.goto(url);
    // 設定可視區域大小
    await page.setViewport({
        width: 1024,
        height: 768
    });
    await page.screenshot({
        path: "screenshot/" + filename
    });
    browser.close();
};

screenshot("url", "screenshot.png");
```

## 爬資料

```JS
const puppeteer = require("puppeteer");

var scrape = async(url, selector) => {

    const browser = await puppeteer.launch({
        headless: false
    });
    const page = await browser.newPage();
    await page.goto(url);
    
    const data = await page.evaluate((selector) => {
        const dom = document.querySelector(selector);
        return dom ? dom.textContent : null;
    }, selector);

    browser.close();
    return data;
};

scrape("url", "selector").then((data) => {
    console.log(data);
});
```

## Proxy

```JS
const browser = await puppeteer.launch({
    headless: false,
    args: [ '--proxy-server=HOST:PORT' ]
});
```

## 輸入表單(type)

```JS
await page.type('selector', data);
await page.click('selector_submit');
```

## 上傳檔案(Upload dialog)

```JS
let filePaths = [filePath1, filePath2, ...];
// 選取上傳檔案的元件
let uploadElement = await page.$('selector');
await uploadElement.uploadFile(...filePaths);
```

## issue

部分網頁在headless模式遇到`Error: Navigation Timeout Exceeded: 30000ms exceeded`的話，

可以試試看不載入font, css, js, image等不需要的文件

<https://github.com/GoogleChrome/puppeteer/issues/1913#issuecomment-361224733>

## 參考文件

* api-doc: <https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md>
* <https://github.com/ebidel/try-puppeteer>
