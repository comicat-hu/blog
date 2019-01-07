---
title: Yii framework - Memcache
date: 2017-10-26 18:35:22
tags:
  - PHP
  - Yii1.x
---
memcached是一套分散式的快取系統，但server間並不互相同步，而是由client端去做分散存取(hash計算)，存資料時hash算出要存的server，取資料時用相同的hash算法指到要讀的server

用key-value的方式儲存資料

memcache有LRU機制(Least Recently Used)，會在內部儲存空間不足時自動讓較少使用到的資料失效(expire)，讓新的資料可以覆蓋到該空間上

<!--more-->

<http://qianshangding.iteye.com/blog/2259411>

## 閱讀連結

官網: [memcached.org](http://memcached.org/)
經典名著: [memcached全面剖析.pdf](http://docs.linuxtone.org/ebooks/NOSQL/memcached/memcached%E5%85%A8%E9%9D%A2%E5%89%96%E6%9E%90.pdf)
windows版執行檔: [window-install-memcached](http://www.runoob.com/memcached/window-install-memcached.html)
PHP memcache extension api: [class.memcache.php](http://php.net/manual/en/class.memcache.php)

## Getting start

官方沒有提供windows版的memcached，所以只好找別人編好的執行檔來跑，

我用x86 1.4.4版，打開啟動memcached.exe，預設就會在localhost:11211啟動了

`memcached -help`可以查看指令，

memcached可以直接telnet登入，`telnet 127.0.0.1 11211`後，下`stats`指令可以看到一些memcache的狀態，也可以直接在上面操作存取資料

{% asset_img memcache_stats.PNG memcache_stats %}

## PHP memcache

使用PHP操作memcache需要裝memcache extension，之前使用Yii時也用工具檢查過了，就不多說，用的是3.0.8版。

直接使用PHP來操作(也可以參考文末)

```PHP
$memcache = new Memcache;
$memcache->addServer('localhost');
$memcache->set('a', 'apple');
echo $memcache->get('a'); // apple
```

addServer是將一個memcache server加入到MemcachePool，但實際上並不會連線，而是真正要執行須要連線的指令時才執行，所以該動作通常都不會出錯，回傳1

都沒有addServer到MemcachePool中就操作，會得到`No servers added to memcache connection in`警告

執行其它動作(像是get, set時)，如果memcache server沒有啟動會警告，然後是一堆亂碼我也不知道為啥..，不過錯誤代號通常會是(10060)或(10061)或`無法連線，因為目標電腦拒絕連線。`，然後似乎沒有一個比較好的api來檢測server是不是啟動中...

折衷的辦法，github找到有人是用connect()來判斷，所以稍微改一下是這樣

```PHP
$isMemcacheAvaliable = @$memcache->connect('localhost');
if (!$isMemcacheAvaliable) {
    die('Memcache Not Avaliable.');
}
```

connect()方法是直接去連某台memcache server，然後那台如果shutdown，依然會出1006x的警告，所以我們就用`@`把訊息弄掉，這時因為連接失敗所以`$isMemcacheAvaliable` false，就可以抓出failed的那台機器了

`$memcache->getVersion();`拿到的版本資訊是`1.4.4-14-g9c660c0`

在Yii裡面，如果有上述情況，頁面會直接噴掉去錯誤頁(500 server internel error)，有開debug模式會告訴你正是在urlManager嘗試要去存取cache時錯誤的。錯誤點在`CMemCache::getValue()`，使用memcache的get()那行，可以自行改善這個問題

## 在Yii config components中的配置

```PHP
'cache' => [
            'class' => 'CMemCache',
            'useMemcached' => false,
            'keyPrefix' => '',
            'hashKey' => false,
            'serializer' => false,
            'servers' => [
                [
                    'host' => 'xxxx',
                    'port' => '11211',
                    'weight' => '50',
                ],
                [
                    'host' => 'xxxx',
                    'port' => '11212',
                    'weight' => '50',
                ],
            ],
        ],
```

* useMemcached預設是關的
* servers預設是空的，會預設用localhost
* keyPrefix(key加前綴字，預設是用`Yii::app()->getId()`的結果當前綴), hashKey(key做md5), serializer這三項是yii預設開啟的，要用原始的標準可以都關掉
* serializer序列化資料
* weight可以設定該server的權重，這對memcache在做hash計算去找server時會有影響

## Yii 操作 memcache

`Yii::app()->getComponent('cache')`或`Yii::app()->cache`取得CMemCache實體(extends CCache)

`CMemCache::getMemCache()`取得Memcache實體(原始的那個，可以呼叫原生方法)

但yii在CCache中有實作get, set等方法，所以透過CMemCache實體呼叫get(), set()也可以存取資料(實際上也是去呼叫原生方法)

## flush()與delete()

memcache中可以設定expire讓資料在特定時間"失效"

flush()方法可以讓整個memcache中的資料都失效

delete()則是刪除某筆資料

兩者原理略有不同

## Important note

* 如果使用了connect疑似會造成MemcachePool被清空(?

* addServer方法的參數有這些，一定要依照格式填寫!!

```PHP
bool Memcache::addServer ( string $host [, int $port = 11211 [, bool $persistent [, int $weight [, int $timeout [, int $retry_interval [, bool $status [, callable $failure_callback [, int $timeoutms ]]]]]]]] )
```

* Yii的memcache預設值在`CMemCacheServerConfiguration`類別中(同樣放在CMemCache)

```PHP
public $host;

public $port=11211;

public $persistent=true;

public $weight=1;

public $timeout=15;

public $retryInterval=15;

public $status=true;
```

如果config有設就會用config的值取代掉，

很重要的一點是你存值時的設定都會影響到hash計算，所以取值時也要用一模一樣的設定，才能保證取的到值(如果只有連一台就沒差)，疑似連addServer的順序都會有影響，當然如果存值後增減server也可能會導致hash計算指不到正確的server!!!!!!!

## memcache getAvaliableServers 程式參考

```PHP
function getAvaliableServers($servers)
{
    $memcache = new Memcache;
    $avaliableServers = [];
    foreach ($servers as $server) {
        $isMemcacheAvaliable = @$memcache->connect($server['host'], $server['port']);
        $memcache->close();
        if ($isMemcacheAvaliable) {
            array_push($avaliableServers, $server);
        }
    }
    return $avaliableServers;
}

$avaliableServers = getAvaliableServers($servers);

if (count($avaliableServers) === 0) {
    die('Memcache Not Avaliable.');
}
```

## stats指令查看到的一些參數說明

* pid: 該程式執行的PID
* uptime: 啟動後的總執行秒數
* time: 當前的timestamp
* pointer_size: 32位元或64位元
* curr_connection: 當前的開放的連接數
* total_connection: 啟動後打開的總連接束
* cmd_get: 執行get命令的總數
* cmd_set: 執行set命令的總數
* cmd_flush: 執行flush命令的總數
* xxx_misses: 執行xxx命令未命中的總數
* xxx_hits: 執行xxx命令命中次數
* cas_badval: 使用擦拭次數(?
* bytes_read: 從網路讀取的總位元組數
* bytes_written: 傳送的總位元組數
* limit_maxbytes: 可儲存的最大位元組數
* accepting_conns: 目前接受的連接數
* threads: 執行緒數
* conn_yields: 連線主動放棄的數量
* bytes: 當前儲存資料佔的大小
* curr_items: 當前儲存的項目數量
* total_items: 啟動以來儲存的項目數量
* evictions: 被LRU算法釋放掉的項目數量
