---
title: 使用docker製作多版本php-cli指令
date: 2020-09-03 17:26:04
categories:
  - 資訊技術
  - Docker
tags:
  - Docker
  - PHP
---

因為工作剛好需要經手到一個骨灰級的php專案，clone and install後第一件事就是跑測試，

但是本地端的php版本是7.4，一跑下去本來沒錯誤的測試都過不了，

想起公司大神曾玩過的docker container tools，這次就來試一下吧~

<!--more-->

## 建立php-cli image

* 首先下載使用php5.6-cli這個docker image。
```sh
docker run -it php:5.6-cli bash
```

* 進container後執行`php -v`確認版本資訊。
{% asset_img phpcli_init_version_info.png phpcli_init_version_info %}

* 我們測試使用phpunit需要安裝xdebug。
查詢PHP支援的對應xdebug版號，<https://xdebug.org/download/historical>。
這個image已經內建了pecl，`pecl install xdebug-2.5.5`，就可以安裝xdebug了 (不帶版號則會裝最新版)。
{% asset_img xdebug_install_success.png xdebug_install_success %}

* 設定php.ini，並啟用xdebug
```sh
cd '/usr/local/etc/php/'
cp  php.ini-development php.ini

# 依照xdebug安裝成功後的提供的資訊在php.ini中添加設定
echo "zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so" >> php.ini
php -v
```
正確啟用後php的版本資訊中會看到`with Xdebug v2.5.5`的字樣了。
{% asset_img xdebug_settings_success.png xdebug_settings_success %}

* 離開container，建立新的layer，並加上tag
```sh
# 使用剛剛操作的container-id
docker commit ${container-id}
# 使用剛剛commit回傳的image-id
docker tag ${image-id} php56
```

## 建立alias指令

* 呼叫container幫我們執行php
```sh
docker container run -it --rm php56 php -v
```

* 將當前目錄掛進container執行指令
```
docker container run -it --rm -v $(pwd):/source -w /source php56 ls -al
```

* 建立alias
在.bashrc或.zshrc中加入設定
```sh
alias php56="docker container run -it --rm -v \$(pwd):/source -w /source php56 php"
```
`$(pwd)`之前的`\`一定要記得，不然alias會在一登入時就將其解析成固定的路徑。


當我們執行`php56`時，實際上就是呼叫container幫我們執行php了。
{% asset_img run_php56_cli.png run_php56_cli %}
