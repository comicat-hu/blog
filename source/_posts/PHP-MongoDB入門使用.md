---
title: PHP-MongoDB入門使用
date: 2017-09-30 15:02:37
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - Storage
  - MongoDB
---

需要安裝PHP mongo extension

安裝參考: <http://www.w3big.com/zh-TW/mongodb/mongodb-install-php-driver.html>

<!--more-->

## connect mongo

預設會連接到`mongodb://localhost:27017`

```PHP
$mongo = new MongoClient();
```

## use db

db不存在會自動產生一個新的，建立一個叫myDB的資料庫

```PHP
$db = $mongo->myDB;
```

## show dbs

取得所有db的資訊與統計(Array)，空db不會列出來

```PHP
$dbs = $mongo->listDBs();
```

{% asset_img mongo_listDBs.PNG mongo_listDBs %}

最後還有一個totalSize標示出總大小

## create collection

建立一個叫users的集合

```PHP
$collection = $db->createCollection('users');
```

## show collections

取得所有$db中的集合資訊(Array)

```PHP
$db->listCollections();
```

## insert data

在集合中插入一筆資料，$data是一個Array

```PHP
$collection->insert($data);
```

## find data

取得一個MongoCursor Object，
必須使用迴圈讀取

```PHP
$cursor = $collection->find();
foreach ($cursor as $data) {
    print_r($data);
}
```

## update data

把所有id = 1的資料的age更新成25

```PHP
$collection->update(array('id' => 1), array('$set' => array('age' => 25)));
```

## remove data

```PHP
$collection->remove();
```

## insert() vs save()

在mongo shell中直接輸入指令不加括號可以看到執行的函數內容。

insert()是直接插入資料，效率較好

save()會先判斷是否有重複，如果資料存在就呼叫update()，反之insert()
