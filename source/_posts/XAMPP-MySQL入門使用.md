---
title: XAMPP-MySQL入門使用
date: 2017-09-25 14:39:09
tags:
  - MySQL
  - phpmyadmin
---

XAMPP內的MySQL是MariaDB。

## How to use

XAMPP也內建有資料庫管理工具PhpMyAdmin，啟動MySQL後，在你的主機/phpmyadmin路徑下可以開啟它。
{% asset_img phpmyadmin.PNG phpmyadmin %}

當然你也可以用shell登入，不過指令有點多，這邊就先不採用
{% asset_img mysql_shell.PNG mysql_shell %}

預設phpmyadmin會使用root直接登入，所以我們來看一點權限設定的部分

[XAMPP 5.6.15 修改 MySQL 密碼與 PhpMyAdmin 設定](http://www.mauchiuan.com/2015/12/xampp-5615-mysql-phpmyadmin.html)

* 刪除匿名登入
* 設定root密碼
* 修改phpmyadmin設定檔(config.inc.php)
* 新增操作使用者(避免使用root存取DB)

這樣我們有了root密碼和另一個使用者(寫程式時帳密就變成必填)，而在我們開啟phpmyadmin這時會自動帶入我們剛才設定的config去登入(config預設還是root)

或可以不要在config設密碼然後修改`$cfg['Servers'][$i]['auth_type'] = 'http';`，就可以在開啟phpmyadmin頁面時卡一個使用者登入了

(首頁logo下方有登出可以按)

參考文章:

* [【教學】Windows下的架站工具 – XAMPP (MYSQL篇)](https://www.future-vr.com/%E3%80%90%E6%95%99%E5%AD%B8%E3%80%91windows%E4%B8%8B%E7%9A%84%E6%9E%B6%E7%AB%99%E5%B7%A5%E5%85%B7-xampp-mysql%E7%AF%87/)
* <http://140.129.118.16/~richwang/102-2-Courses/DBA/playerinfo-102-2.html> (shell)

## MariaDB的版本資訊

{% asset_img mariadb_version.PNG mariadb_version %}

## MySQL家族版本圖

![mysql_version](http://i.imgur.com/HMxj2Zw.png)

## MySQL default engine

* MySQL 5.1之前預設是: MyISAM
* MySQL 5.5之後預設是: InnoDB

  (中間的版本好像跳掉了???)

MyISAM屬於Table-level-lock，當資料表正在異動時，會整表lock住不能讀取

InnoDB屬於Row-level-lock，當資料表正在異動時，只會lock住有異動的row，其它資料仍然有機會可以讀取

InnoDB還支援了ACID transaction功能

* Atomicity(原子性，不可分割性)
* Consistency(一致性)
* Isolation(隔離性)
* Durability(持久性)

參考: [MySQL 中，MyISAM 與 InnoDB 帶來的差異](https://blog.gslin.org/archives/2012/11/24/3034/mysql-%E4%B8%AD%EF%BC%8Cmyisam-%E8%88%87-innodb-%E5%B8%B6%E4%BE%86%E7%9A%84%E5%B7%AE%E7%95%B0/)

## START TRANSACTION(BEGIN)

(InnoDB)

預設狀況下，SQL語句都會被立即執行並且提交結果

可以`SELECT @@AUTOCOMMIT;`查詢這個參數，預設為1
{% asset_img mariadb_autocommit.PNG mariadb_autocommit %}

`START TRANSACTION;`或`BEGIN;`可以開啟一個交易區塊，表示在`COMMIT;`指令執行前的SQL執行結果都不會被立即提交出去

意思是當你在中間做新增、刪除、修改...等，在別的連線中都還不會存取到你的結果(但有一些指令會強制造成交易中斷)，

交易中`ROLLBACK;`指令會讓交易取消，並且回到交易前的狀態。

`SET AUTOCOMMIT = 0;`這時候自動提交的功能被關閉，所以即使不開啟交易也是會像上面的狀況一樣，必須等到`COMMIT;`才會將結果提交出去。

查詢autocommit參數 = 1，開啟交易區塊，並且插入一筆資料`(3, 'Jimmy')`，從另一個連線查看並沒有結果(還沒COMMIT)
{% asset_img transaction_example.PNG transaction_example %}

參考:

* <http://xyz.cinc.biz/2013/05/mysql-transaction.html>
* <https://blog.longwin.com.tw/2006/03/innodb_transaction_2006/>

## import file to MySQL

原先使用phpmyadmin操作csv匯入，但時間實在是太久了(平均一個檔30MB)

後來直接使用shell操作比較有效。

先建好db, table, 各欄位

附上匯入檔案指令(utf8編碼)

```SQL
LOAD DATA LOCAL INFILE '檔案路徑名稱' INTO TABLE 完整表格名稱 CHARACTER SET utf8 FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n';
```
