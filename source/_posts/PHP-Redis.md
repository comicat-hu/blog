---
title: PHP - Redis
date: 2017-11-23 10:56:26
tags:
  - PHP
  - Storage
---
Redis是一個BSD開源的key-value快取資料庫，可以保持資料的持久度，有支援多種的資料結構可以儲存

windows的執行檔可以從這裡找到

<https://github.com/MicrosoftArchive/redis/releases>

解壓縮之後可以使用指令`redis-server redis.windows.conf`來啟動一個本機的redis server，預設起在port 6379，沒有連線密碼

## 常用指令

* `redis-cli`是操作的client interface
* `redis-cli -h 127.0.0.1` 連接到本機的redis，預設連port 6379，也可以連接到別的host，如果沒有設密碼驗證的話
* `get`, `set`指令可以用來存放或讀取資料，當你打出指令的時候，redis-cli會很貼心的提示後面的參數要放什麼
* `info keyspace`指令可以列出全部的redis db和其中的key數量
* `select`可以切換到不同的index，如果沒有select預設會使用index 0
* `keys *`列出所有key，星號可以用其他的pattern代替

## 使用PHP存取redis

當然，會需要一個redis extension

### 連線

```PHP
$redis = new Redis();

$redis->connect(host, port);
```

### select db

```PHP
$redis->select(index);
```

### get something

```PHP
$result = $redis->get(key);
```

### set something

```PHP
$redis->set(key, data, expire);
```

### get keys

```PHP
$keys = $redis->keys('*');
```

## 參考資料

* <http://www.runoob.com/redis/redis-tutorial.html>
* <https://github.com/phpredis/phpredis>
