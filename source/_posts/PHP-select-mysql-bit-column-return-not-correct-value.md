---
title: PHP select mysql bit column return not correct value
date: 2019-05-25 10:59:56
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - MySQL
---

最近在一個PHP5.3的專案中遇到存取MySQL bit型態欄位取值總是拿到0的狀況，

不要問我為什麼還在PHP5.3，嘛~ 萬惡歷史淵源嘛~ 進入正題~

## 狀況摘要

經查發現，

取bit=1，php這邊得到

```TEXT
string(1)"\001"
```

取bit=0，php這邊得到

```TEXT
string(1)"\000"
```

<!--more-->

以嘗試過用這幾種設定都無法解決

* `$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);`
* `ord()`

## 原因

php mysql driver版本的問題。

**舊版的php預設使用libmysqlclient做為驅動，php5.3起支援mysqlnd，但編譯時需要指定，php5.4後預設使用mysqlnd**，

**libmysqlclient無法取得mysql中的原始型態，在弱型別的處理中bit被不正確的強轉成string**，而造成此bug，

使用mysqlnd則沒有這個問題，bit型態將會順利的以string 0/1回傳，若設定`PDO::ATTR_EMULATE_PREPARES`，則以int 0/1回傳，

目前在insert時並沒有遇到這個問題，都是select時，回傳值不正確。

## 解決辦法

* 環境面的解法: 升級mysql driver重新編譯php環境或換主機

* DB面的解法: 避免使用bit型態，使用tinyint(1)，根據文件這兩者占用的空間是差不多的

* 程式面的暫時解: 特別針對select bit形態欄位時+0，`SELECT (bit_column+0) AS bit_column`

## 如何判斷環境使用何種mysql driver

通常phpinfo(php -i)沒有查閱到mysqlnd這個字眼時，使用的就是舊的libmysqlclient

以下提供一段網路上找的的程式檢查，他是透過檢查mysqlnd特有的function來判斷使用的哪種driver

```PHP
$hasMySQL = false;
$hasMySQLi = false;
$withMySQLnd = false;
$sentence='';

if (function_exists('mysql_connect')) {
    $hasMySQL = true;
    $sentence.= "(Deprecated) MySQL <b>is installed</b> ";
} else
    $sentence.= "(Deprecated) MySQL <b>is not</b> installed ";

if (function_exists('mysqli_connect')) {
    $hasMySQLi = true;
    $sentence.= "and the new (improved) MySQL <b>is installed</b>. ";
} else
    $sentence.= "and the new (improved) MySQL <b>is not installed</b>. ";

if (function_exists('mysqli_fetch_all')) {
    $withMySQLnd = true;
    $sentence.= "This server is using MySQLnd as the driver.";
} else
    $sentence.= "This server is using libmysqlclient as the driver.";

echo $sentence;
```

## 參考資料

​* <https://stackoverflow.com/questions/10540483/pdostatement-mysql-inserting-value-0-into-a-bit1-field-results-in-1-written/10542145>
​* <https://stackoverflow.com/questions/20079320/php-pdo-mysql-how-do-i-return-integer-and-numeric-columns-from-mysql-as-int>
​* <https://stackoverflow.com/questions/15106985/mysql-select-bit1-shows-as-string3>
​* <https://www.junorz.com/archives/683.html>
​* <http://www.q2zy.com/php-pdo%E9%A9%B1%E5%8A%A8-mysqlnd%E6%8A%98%E8%85%BE%E8%AE%B0/>
​* <https://www.php.net/manual/zh/mysqlinfo.library.choosing.php>
