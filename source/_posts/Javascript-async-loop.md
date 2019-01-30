---
title: Javascript async loop
date: 2019-01-30 16:27:21
categories:
  - 資訊技術
  - Javascript
tags:
  - Javascript
---

Javascript 自ES7後支援潮潮的async-await語法，解決的許多不順眼的promise同步語法，

但是`Array.prototype.forEach`是沒有支援到async function的用法哦~ (天真的踩雷

底下給了一些程式片段範例:

<!--more-->

首先為了例子方便，我封裝了一個setTimeout，這個wait(ms)傳進要等待的時間，並且執行完後回傳一個promise，等等我們會用async-await來處理它。

```JS
function wait (ms) {
    return new Promise(function (resolve, reject) {
        setTimeout(resolve, ms);
    });
}
```

## forEach

我們期望程式開始後每隔1000ms log出一個陣列值。

```JS
async function run () {
    console.log('Start');
    console.time('ExecuteTime');

    var arr = ['apple', 'ball', 'cat', 'dog', 'egg'];

    arr.forEach(async function (item) {
        console.log(item);
        await wait(1000);
    });

    console.log('End');
    console.timeEnd('ExecuteTime'); // ExecuteTime: < 5ms
}

run();
```

執行上面程式可以發現我們的wait完全沒等到，程式就結束了，總執行時間很短。

## for-of

於是我們將程式修改成下面這種寫法。

```JS
async function run () {
    console.log('Start');
    console.time('ExecuteTime');

    var arr = ['apple', 'ball', 'cat', 'dog', 'egg'];

    for (var item of arr) {
        console.log(item);
        await wait(1000);
    }

    console.log('End');
    console.timeEnd('ExecuteTime'); // ExecuteTime: 5xxx ms
}

run();
```

執行上面程式後，符合我們的期望每隔1秒輸出一個陣列值。

## How to break in forEach

NO.

在forEach裡沒辦法使用`break`，只能`return;`來跳過本次循環，有這種需求只能乖乖用for loop了。

或是參考底下連結提供的一些神奇解法。

## Reference link

* <https://blog.fundebug.com/2018/02/05/map_vs_foreach>
* <https://github.com/babel/babel/issues/909>
* <http://jser.me/2014/04/02/%E5%A6%82%E4%BD%95%E5%9C%A8Array.forEach%E7%9A%84%E5%BE%AA%E7%8E%AF%E9%87%8Cbreak.html>
* <https://stackoverflow.com/questions/2641347/short-circuit-array-foreach-like-calling-break>
