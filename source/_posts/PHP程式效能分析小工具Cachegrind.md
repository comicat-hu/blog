---
title: PHP程式效能分析小工具Cachegrind
date: 2019-10-20 15:31:54
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
---
借助XDebug Profiler記錄callstack，再用Cachegrind工具圖形化顯示，讓我們可以快速簡單地瞭解程式碼效能瓶頸，在追蹤程式碼時也十分好用。

{% asset_img qcachegring介面參考.png qcachegring介面參考 %}

<!--more-->

## 設定xdebug profiler

首先需要在php.ini中啟用XDebug，並且填入以下設定

```
;預設不啟用
xdebug.profiler_enable=0
;可帶XDEBUG_PROFILE=1參數透過GET或POST觸發
xdebug.profiler_enable_trigger=1
;檔名格式
xdebug.profiler_output_name=callgrind.%H_%t.out
;存放的資料夾，需要先開好並給它存取權限
xdebug.profiler_output_dir="D:\xdebug_profiler_output\"
```

## 產生並載入記錄檔

接著只要實際存取一下網頁，並且帶上`XDEBUG_PROFILE=1`參數，每個請求就會產生一筆記錄檔案在指定的位置了。

用Cachegrind載入後，就會自動顯示如首圖了。

耗時顯示可切換成百分比或1/1000000秒。

p.s ubuntu上的叫KCachegrind，windows上的叫QCachegrind

## 參考資料

* https://xdebug.org/docs/profiler
* https://ithelp.ithome.com.tw/articles/10194661
* https://blog.xuite.net/chingwei/blog/32217722-%E3%80%90%E7%B3%BB%E7%B5%B1%E3%80%91Profiling+PHP+with+Xdebug+%26+KCachegrind