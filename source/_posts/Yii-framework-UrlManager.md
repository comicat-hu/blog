---
title: Yii framework - UrlManager
date: 2017-10-20 18:24:35
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - Yii1.x
---
UrlManager主要用來解析URL route，有兩種格式可以設定

* get: 用query string指定route(/?r=controllerID/actionID)
* path: 在URL中指定route(/controllerID/actionID)

<!--more-->

在config中可以設定，沒有指定的話get是預設格式

```PHP
"components" => array(
    'urlManager' => array(
        'urlFormat' => 'path',
        'rules' => array(
            // 格式化規則
        ),
    ),
),
```

## CWebApplication::processRequest()

`$route = $this->getUrlManager()->parseUrl($this->getRequest());`

## CApplication::getUrlManager()

`return $this->getComponent('urlManager');`

在getComponent()，如果在應用程式中urlManager沒有被建立，這時會createComponent，

這邊插一下createComponent($config)，這邊傳入的config其實是從建立CWebApplication物件時configure()後的\_\_componenetConfig中取出的對應內容(`__componentConfig['urlManager']`)，這個陣列其中就包含了該id對應的class以及我們在config中設定的urlFormat, showScript, rules...等等。

createComponent()在YiiBase中定義，進去之後會依照class的名稱去import(Yii的import)檔案，new出對應的物件，並且foreach從傳入的config直接`$object->$key=$value;`(直接改物件的public參數或是call \_\_set())

最後回傳物件。

接下來會有個init()的過程(call CUrlManager::init())，

```PHP
parent::init();
$this->processRules();
```

CUrlManager的父類別是CApplicationComponent，在這裡的init()有兩件事

* attachBehavoirs (預設behaviors是空陣列)
* set `_initialized` = true (有個public getIsInitialized()方法可以調用)

## processRules()

* rules為設定或是urlFormat = 'get'，直接return
* 先檢查cache中是否有已解析的route，有就回傳它
* 再來才是foreach解析每條rule

```PHP
foreach($this->rules as $pattern => $route)
    $this->_rules[] = $this->createUrlRule($route,$pattern);
```

createUrlRule()中會先判斷使否有指定class，沒有的話會使用預設的CUrlRule(`$this->urlRuleClass`)，然後依照傳入的route, pattern回傳CUrlRule實體。(CUrlRule定義在CUrlManager.php中)

* 將解析好的route存入cache

## rule

rules可以直接以`'pattern' => 'route'`的格式定義，或是`'pattern' => ['route']`陣列中選填一些參數，urlSuffix, caseSensitive, defaultParams, matchValue

rules中可以使用`<ParamName>`來包含一個參數，並且可以使用`:`加入regex，格式是這樣`<ParamName:ParamRegex>`，且該參數會直接套用到GET定義的屬性中。

e.g.:

* 將該controller指定為該位置的參數，並且要求符合文數字格式([A-Za-z0-9_])

`<controller:\w+>`

* 拿之前的計算範例來改，math路徑下要求符合一個數字a，任意四種action，一個數字b。符合該規則的route會對應到相應的四種math/actionID，a,b 會直接被帶進GET對應的參數。

```PHP
'math/<a:\d+><action:(add|div|mul|sub)><b:\d+>' => 'math/<action>'
```

## 使用.htaccess隱藏index.php

一般我們在url路徑中需要包含index.php(尤其是在urlFormat = path時)，但是這樣非常麻煩而且不友善

所以我們可以在index.php入口的同層路徑中加入.htaccess檔案

確認httpd.conf有`LoadModule rewrite_module modules/mod_rewrite.so`

確認httpd.conf須設定AllowOverride ALL，表示允許該檔案的設定可以被覆寫(要開對地方，xampp的話是在`<Directory "C:/xampp/htdocs">`的那個設定裡)

```Text
# 啟用FollowSymLinks
Options +FollowSymLinks
IndexIgnore */*

# 啟用rewrite功能
RewriteEngine on

# 請求的檔案 資料夾都不存在
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d

# forward到index.php這隻檔案
RewriteRule . index.php
```

## 參考資料

* <http://www.yiichina.com/doc/guide/1.1/topics.url>
* <http://blog.xiayf.cn/2014/11/12/read-yii-code-2/>
