---
title: Yii framework - Simple Workflow
date: 2017-10-16 17:42:18
tags:
  - PHP
  - Yii1.x
---
{% asset_img yii_workflow.png yii_workflow %}

參考上圖，簡化一個最簡單的Yii workflow

## index.php

整個網站的進入點，主要就是

* require YiiBase核心
* `$app = Yii::createWebApplication($config)`，建立一個`CWebApplication`物件
* `$app->run()`啟動它

## Application

這裡主要是run()中的processRequest()

* `$route = $this->getUrlManager()->parseUrl($this->getRequest());`，建立CHttpRequest物件(request)，建立CUrlManager物件(urlManager)，並且將request傳入urlManager的parseUrl()去處理，拿到controllerID/actionID
* `$this->runController($route);`，根據傳入的route去建立及啟動對應的controller和action

## Controller、Filters、Actions

根據對應的controllerID/actionID分別進入各controller中後

* 先根據filters()中的return值來呼叫對應的過濾器做存取權限、參數驗證...等，可使用`+`,`-`來設定需要使用filter的action
* filter都通過之後啟動action，會先進入符合actionID的方法，如果沒有會再actions()中找外部的action

## Views

在action中render()要輸出的內容，這時候render會引入layout，並將輸出內容放入$content變數中，包含在layout中echo
