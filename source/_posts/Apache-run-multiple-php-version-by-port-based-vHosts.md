---
title: Apache run multiple php version by port-based vHosts
date: 2019-09-08 01:15:44
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - Apache
---

這篇記錄一下在windows 10 上設定Apache2.4的vHosts，透過不同的port來跑多版本的PHP。

像這樣:

* localhost:8056 (php5.6)
* localhost:8072 (php7.2)
* localhost:8073 (php7.3)

<!--more-->

## 資料夾配置

這裡列一下我的資料夾配置，後面也會依這邊的路徑來配置Apache、PHP設定。

```text
- D:/web
    |_ /Apache24
    |_ /php
        |_ /56
        |_ /72
        |_ /73
```

## 下載Apache2.4

下載ApacheHaus (Apache 2.4.x OpenSSL 1.1.1 VC15)，如果需要啟用TLSv1.3需要使用含OpenSSL 1.1.1以上的版本。

* 解壓縮Apache24資料夾
* 設定conf/httpd.conf
    - SRVROOT
    - Listen PORT (這邊我改成8080)
* 手動啟動bin/httpd測試，應可連上localhost:8080和localhost:443，顯示index.html預設頁面
* 註冊服務 httpd -k install
* 啟動服務 httpd -k start
* 重啟服務 httpd -k restart
* httpd -S

## 下載PHP

### 下載php (x64 TS版)

* <https://windows.php.net/download/>
* <https://windows.php.net/downloads/releases/archives/>  (停止維護的在這裡)
* 分別解壓縮到php/73、php/72、php/56 (php7需要VC15、php5需要VC11)
* 複製php-development.ini到php.ini

### 建立預設的php環境

* 在apache conf/httpd.conf加入以下內容，重啟apache

    ```ini
        DirectoryIndex index.php index.html
    ```

    ```ini
        LoadModule php7_module "D:/web/php/73/php7apache2_4.dll"
        AddType application/x-httpd-php .php
        PHPIniDir "D:/web/php/73/"
    ```

* 在htdocs建立phpinfo.php，連到localhost:8080，應該可以順利解析並且顯示phpinfo，並且看到版本為7.3

    ```php
        <?php phpinfo(); ?>
    ```

### 啟用fcgi

* 下載mod_fcgid.so (<https://httpd.apache.org/download.cgi>)
* 解壓縮到apache的modules目錄
* httpd.conf加入設定

    ```ini
        AddHandler fcgid-script .fcgi .php

        FcgidInitialEnv PHPRC "D:/web/php/73"
        FcgidWrapper "D:/web/php/73/php-cgi.exe" .php
    ```

### 啟用vHosts

* 拿掉註解 `Include conf/extra/httpd-vhosts.conf`
* 修改httpd-vhosts.conf

    ```ini
        Listen 8056
        Listen 8072
        Listen 8073

        <Directory "${SRVROOT}/htdocs">
            Options -Indexes +ExecCGI
            AllowOverride All
            Require all granted
        </Directory>

        <VirtualHost _default_:8056>
            ErrorLog "logs/php56-error.log"
            CustomLog "logs/php56-access.log" common

            FcgidInitialEnv PHPRC "D:/web/php/56"
            FcgidWrapper "D:/web/php/56/php-cgi.exe" .php
        </VirtualHost>

        <VirtualHost _default_:8072>
            ErrorLog "logs/php72-error.log"
            CustomLog "logs/php72-access.log" common

            FcgidInitialEnv PHPRC "D:/web/php/72"
            FcgidWrapper "D:/web/php/72/php-cgi.exe" .php
        </VirtualHost>

        <VirtualHost _default_:8073>
            ErrorLog "logs/php73-error.log"
            CustomLog "logs/php73-access.log" common

            FcgidInitialEnv PHPRC "D:/web/php/73"
            FcgidWrapper "D:/web/php/73/php-cgi.exe" .php
        </VirtualHost>
    ```

過程中如果設定檔有誤可能會持續遭遇ConnectionRefuse或403 Forbidden，

還有注意apache22跟apache24有些指令不同，例如資料夾權限在apache22的指令是`Allow from all`，在apache24則是`Require all granted`

## 參考資料

* <https://github.com/pniaps/win-apache-php/blob/master/Apache-2.4-win64/conf/httpd.conf>
* <https://github.com/pniaps/win-apache-php/blob/master/Apache-2.4-win64/conf/extra/httpd-vhosts.conf>
* <http://www.osyum.com/article/show/287/>
* <https://shazi.info/ubuntu-16-04-%E5%AE%89%E8%A3%9D-apache2-mod_fcgid-mpm_worker-%E8%B7%91-php-7-x/>
* <https://recwe.com/article/25>
