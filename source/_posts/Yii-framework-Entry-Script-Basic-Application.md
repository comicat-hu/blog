---
title: Yii framework - Entry Script & Basic Application
date: 2017-10-03 11:41:19
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - Yii1.x
---
在Yii建出專案資料夾的最上層有個index.php當作應用程式的進入點。

## index.php

這裡面大概做了幾件事:

* 定義一些常數(設定值) [define vs const](https://stackoverflow.com/questions/2447791/define-vs-const)
* require`\framework\yii.php`
* `Yii::createWebApplication($config)->run();`，傳入設定檔並建立一個WebApplication，然後啟動

<!--more-->

## framework\yii.php

這個檔案只做兩件事，

* 判斷`Class YiiBase` 是否存在，不存在就載入`YiiBase.php`

```PHP
if(!class_exists('YiiBase', false))
    require(dirname(__FILE__).'/YiiBase.php');
```

* 定義`Class Yii extends YiiBase`

## framework\YiiBase.php

* 定義了Yii的一些常數預設值
* 定義`Class YiiBase`
* auto load Yii core classes
* require `base\interfaces.php`

不建議更動及呼叫這個核心檔的東西，使用寫在yii.php的`Class Yii`比較好。

createWebApplication()就是在這時候被載入的。

* Yii::createWebApplication()

  接收設定檔，並回傳呼叫`createApplication('CWebApplication', $config)`的結果

* Yii::createApplication()

  接收要啟用的應用及設定檔，並且回傳依據名稱建立出來對應的應用物件，這裡會建立出一個CWebApplication物件。

--

## framework\web\CWebApplication.php

* 定義`Class CWebApplication extends CApplication`

## framework\base\CApplication.php

* 定義`Class CApplication extends CModule`

CWebApplication物件使用了這裡的\_\_construct()建立，並且從這裡繼承了run()方法。

## Application life cycle

CWebApplication建立(construct)時大概會發生一些事

* 呼叫`preinit()`(從CModule繼承而來)，預先初始化一些模組
* 呼叫`initSystemHandlers()`，初始化啟用yii的例外錯誤處理
* 呼叫`registerCoreComponents()`，註冊核心組件
    + `coreMessages => CPhpMessageSource` : 提供Yii的核心訊息翻譯，預設是`en_us`
    + `db => CDbConnection` : 提供Database連線，使用時需要設定DSN(Data Source Name)
    + `messages => CPhpMessageSource` : 提供Yii的應用程式訊息翻譯
    + `errorHandler => CErrorHandler` : 處理程式中未處理的PHP例外錯誤處理，依據錯誤類型調用適當的view顯示
    + `securityManager => CSecurityManager` : 提供資安相關功能，ex. hash, crypto...等
    + `CStatePersister => statePersister` : 提供全域的狀態維持功能
    + `urlManager => CUrlManager` : 提供URL相關功能
    + `request => CHttpRequest` : 提供http request相關功能
    + `format => CFormatter` : 提供數據顯示格式化功能
* 呼叫`configure($config)`(從CModule繼承而來)，載入剛剛傳進來的config
* 呼叫`attachBehaviors($this->behaviors)`(從CComponent繼承而來)
* 呼叫`preloadComponents`(從CModule繼承而來)
* 呼叫`init()`(從CModule繼承而來並在CWebApplication中有被覆寫)

建立後立即呼叫run()

* 等待觸發`onBeginRequest`事件
* 呼叫`processRequest()`，處理request、create controller、run controller
* 等待處發`onEndRequest`事件

CApplication中定義了`abstract public function processRequest();`，內容由各子類別自己去實作。

另外參考:
<http://www.cnblogs.com/JosephLiu/archive/2011/12/19/2292852.html>

## CApplication::\_\_construct()

{% asset_img yii_CApplication_construct.PNG yii_CApplication_construct %}

這裡的$this是指CWebApplication物件

* 134行: 設定應用程式物件，這樣在應用環境中使用`Yii::app()`就可以取得該CWebApplication實例
* 137行: require config
* 139~145行: 設定basePath
* 146~159行: 設定了路徑的別名以利使用
* 161行: yii預留的方法，預設是沒有內容的，需要的話可以自行覆寫
* 163行: 初始化yii的Exception handler，預設應該是啟用的
* 164行: 註冊必須的核心模組，registerCoreComponents()最後其實呼叫到了繼承自CModule的setComponent()，將component的實體存入\_components中 (???not sure)
* 166行: 將設定值轉成應用程式實體的屬性，configure()方法繼承自CModule，可以看到他其實是這樣設的`$this->$key=$value;`，在存取設定不存在的屬性時，會分別用到CModule中的\_\_get()及CComponent中的\_\_set()
* 167行: attach application behaviors，大概是個註冊事件集合的概念
* 168行: 將configure中從main.php中載入的preload component，存入\_components中，完成初始化
* 170行: 呼叫init()，這個方法在CWebApplication中有被覆寫。先跑父層的init()，並且呼叫getRequest('request')。getRequest()使用到CModule中的getComponent()，取得對應的request component實體並回傳。
