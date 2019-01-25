---
title: Yii framework安裝
date: 2017-09-30 16:30:21
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - Yii1.x
---
因專案需求安裝Yii 1.1 (PHP > 5.1)，在windows上直接下載zip
{% asset_img yii1_1.PNG yii1_1 %}
Yii v1.1.19

Download: <http://www.yiiframework.com/download/>
中文參考: <http://www.yiichina.com/>

<!--more-->

## Step

* 解壓縮檔案到自行建立的yii資料夾，我這邊是直接把資料夾放在xampp/htdocs下
* 連上`主機/yii/requirements/`可以看到yii提供的伺服器設定檢查，包含PHP版本支援度和php.ini設定、安裝的extension...等。
  {% asset_img yii_requirements.png yii_requirements %}
* yii/framework下有yiic指令檔可以執行，可以設定系統環境變數
* 執行yiic

  {% asset_img yiic.PNG yiic %}

## 自動建立一個yii web

開啟cmd，cd到要放web的資料夾，執行`yiic webapp myweb`，自動建立一個網頁應用在myweb資料夾
{% asset_img yii_webapp.PNG yii_webapp %}

連線到`主機/myweb/`，可以看到建立好的範例網站
{% asset_img first_web.PNG first_web %}

cd 到建立的web資料夾下(index.php那層)，`yiic shell`可以進入yii cli模式，

{% asset_img yiic_shell.PNG yiic_shell %}

參考:

[MVC設計模式與Yii Framework簡介](http://newsletter.ascc.sinica.edu.tw/news/read_news.php?nid=2717)
