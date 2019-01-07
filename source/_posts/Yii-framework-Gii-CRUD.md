---
title: Yii framework - Gii CRUD
date: 2017-10-02 18:06:01
tags:
  - PHP
  - Yii1.x
---
在使用yiic webapp建立好的網站雛形上，操作Gii web code generator生成CRUD程式。

## 設定資料庫連線資訊

yii framework使用PDO操作資料庫，

在`protected\config\database.php`中設定資料庫連線資訊，預設幫你連到sqlite，這邊我修改成連接到現有的MySQL

<!--more-->

```PHP
return array(
    'connectionString' => 'mysql:host=localhost;dbname=mydb',
    'emulatePrepare' => true,
    'username' => '*****',
    'password' => '*****',
    'charset' => 'utf8',
);
```

## Enable Gii

要使用Gii，先到protected\config\main.php中啟用modules gii(拿掉註解)，並且設個password

連上`/index.php?r=gii`，輸入密碼後就可以進入gii 介面
{% asset_img yii_gii.PNG yii_gii %}

## Generate Model

進入Model Generator，如果連線設定正確的話可以順利進入，反之會有exception畫面，

建立一個table-myguest的model，預覽執行後會在`protected\models`生成一個Myguests.php
{% asset_img yii_gii_model_generator.PNG yii_gii_model_generator %}

## Generate View、Controller

進入CRUD Generator，輸入Model Class name，自動新建下列檔案
{% asset_img yii_gii_crud_generator.PNG yii_gii_crud_generator %}

完成後連上`/index.php?r=myguests`就可以看到讀取的資料結果

{% asset_img yii_read.PNG yii_read %}

點選manage或進入`/index.php?r=site/login`，登入之後可到後台進行CUD操作

預設有"demo/demo", "admin/admin"兩組帳號密碼可以使用，在`\protected\components\UserIdentity.php`可以看到一些使用者驗證設定，但這邊先不要修改，因為admin這個帳號跟很多隻檔案連動。

登入成功後到`index.php?r=myguests/admin`，可以看到所有資料並且有操作的功能(操作功能會受限於MySQL登入帳號的權限，如果權限不夠操作後會有alert錯誤)
{% asset_img yii_manage.PNG yii_manage %}

* 新增單筆資料(Create)

{% asset_img yii_manage_create.PNG yii_manage_create %}

* 點選放大鏡查看單筆資料(Read)

{% asset_img yii_manage_view.PNG yii_manage_view %}

* 點選鉛筆修改單筆資料(Update)

{% asset_img yii_manage_update.PNG yii_manage_update %}

* 點選刪除單筆資料(Delete)，會跳Alert詢問是否刪除
