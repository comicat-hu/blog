---
title: w3school-PHP練習筆記-MySQL
date: 2017-09-21 18:03:35
tags:  
  - PHP
  - MySQL
  - 學習筆記
---

<https://www.w3schools.com/php/php_mysql_intro.asp>

* MySQL Database
* MySQL Connect
* MySQL Create DB
* MySQL Create Table
* MySQL Insert Data
* MySQL Get Last ID
* MySQL Insert Multiple
* MySQL Prepared
* MySQL Select Data
* MySQL Delete Data
* MySQL Update Data
* MySQL Limit Data

## SQL

SQL(Structured Query Language)，ANSI標準，可以用來操作存取資料庫系統

語法參考: 就挑自己喜歡的看吧，因為有時候不一定哪邊的解說比較好懂
<https://www.w3schools.com/sql/sql_syntax.asp>
<http://www.w3school.com.cn/sql/sql_syntax.asp>
<http://www.1keydata.com/tw/sql/sql.html>

這邊會使用XAMPP中的MySQL(MariaDB)還有PDO(PHP Data Object)來操作

## Connect

先上連接範例

```PHP
try {
    $config = include_once 'config.php';
} catch (Exception $e) {
    echo 'Missing config.php';
    exit;
}

$servername = $config['servername'];
$username = $config['username'];
$password = $config['password'];

$dbname = $config['dbname'];

$dsn = "mysql:host=$servername;dbname=$dbname;";

try {
    $db = new PDO($dsn, $username, $password);
    // set the PDO error mode to exception
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    echo "Connected successfully";
} catch (PDOException $e) {
    echo "Connection failed: " . $e->getMessage();
}
```

* 保持良好的習慣將設定檔另外獨立一個檔案
* 使用try catch攔截錯誤訊息，PDO有自己的例外處理`PDOException`可以使用
* PDO物件中必填的是`$dsn`(Data Source Name)這個參數，host填入資料庫伺服器的位置，通常還會填入`$dbname`來表示要USE db(雖然w3schools說沒填會丟例外，不過實際上並沒有，推測就像是我們連進MySQL server但沒下USE指令的狀況)
* 使用者名稱跟帳號是選填，但是我們在#3582中的設定，將資料庫設成需要帳號跟密碼才能登入，所以這邊需要填寫
* 程式結束會自動close連線，或是把PDO物件指向null，ex.`$conn = null`

## Create DB

先連線資料庫，然後

```PHP
$sql = 'CREATE DATABASE mydb'; // 建立SQL語句
$db->exec($sql); // 執行SQL語句
```

就是這麼簡單。

如果建立重複的資料庫，會收到例外訊息
`SQLSTATE[HY000]: General error: 1007 Can't create database 'mydb'; database exists`

## Use DB

建立完資料庫記得，`$db->exec('USE mydb;');`，否則不會自動切換到你建立的資料庫。

如果一開始連線就沒有指定DB，那接下來對資料表的操作都會收到例外訊息
`SQLSTATE[3D000]: Invalid catalog name: 1046 No database selected`

## Insert Data

使用SQL語句`INSERT INTO table (欄位1, 欄位2,...) VALUES(值1, 值2,...)`

## Get Last ID

`$db->lastInsertId();`: 回傳最後插入的row id或序列值，對應到SQL語句的`LAST_INSERT_ID()`

不過這在使用上會和想像得結果不太一樣就是了，

像是，交易中如果rollback了，lastInsertId不會是交易前的狀態。

## Insert Multiple

使用

* `$db->beginTransaction();` 開啟一段交易區塊
* `$db->commit();` 提交結果
* `$db->rollback();` 回溯結果，這個通常會被放在例外的catch處理中執行

## Prepared

使用Prepare可以預先解析SQL語句，在大量重複類型的句語可以省下解析時間，

節省傳輸語句的資料量

並且有效防止SQL Injection的狀況發生

SQL prepare用法可以參考:
[prepared-statement](http://www.codedata.com.tw/database/mysql-tutorial-12-prepared-statement/)

* 設定預執行語句`PREPAER statement_name FROM 'SQL語句';`
* 參數使用`?`
* 設定變數`SET @var_name = value;`或`SET @var_name := value;`，多變數可以用逗號隔開
* 使用預執行語句`EXECUTE statement_name USING @var_name;`，這時就會把變數依序帶預執行語句的`?`中

設定的變數和預執行語句都是獨立在每個連線裡的，close後都會被清除。

### 在PDO中的使用方式

首先建立一個PDO Statement: `$statement = $db->prepare()`，

預留的變數位置，可以使用`:`開頭的命名，或是直接用`?`代表。

* `$statement = $db->prepare("INSERT INTO myguests (firstname, lastname, email) VALUES (:firstname, :lastname, :email)");`
* `$statement2 = $db->prepare("INSERT INTO myguests (firstname, lastname, email) VALUES (?, ?, ?)");`

再來可以使用PDO Statement中的bindParam將參數帶進去

(使用冒號的綁定方法)

```PHP
$statement1->bindParam(':firstname', $firstname);
$statement1->bindParam(':lastname', $lastname);
$statement1->bindParam(':email', $email);
```

或

(使用問號的綁定方法，第一個變成位置index，從一開始)

```PHP
$statement2->bindParam(1, $firstname);
$statement2->bindParam(2, $lastname);
$statement2->bindParam(3, $email);
```

bindParam()函數可以選填第三個參數: data_type和第四個參數: length

ex. `$statement2->bindParam(1, $firstname, POD::PARAM_STR, 10);`，表示接受$firstname為字串並且長度限制10，並且帶入到第一個位置的問號。

最後使用PDO Statement的execute()執行它
`$statement1->execute();`

bindParam()的回傳值是bool(true|false)，表示綁成功或失敗

### 參考

<https://www.w3schools.com/php/php_mysql_prepared_statements.asp>
<http://php.net/manual/zh/pdo.prepare.php>
<http://php.net/manual/zh/pdostatement.bindparam.php>

## exec() v.s. query()

PDO有兩種常見的即時處理SQL語句的函數

* `$db->exec(SQL);` 通常用於不需要接收結果的語句，會傳回受影響的行總數(DELETE UPDATE INSERT CREATE...等)
* `$db->query(SQL);` 通常用於需要接收結果的語句(SELECT...等)

## Select Data

可以使用`query()或是prepare()+execute()`來處理查詢，
並且使用`fetch()`來獲取一筆資料

```PHP
$sql = "SELECT * FROM myguests";
$result = $db->query($sql);

$result->setFetchMode(PDO::FETCH_ASSOC);
$result->fetch();
```

或

```PHP
$statement = $db->prepare("SELECT * FROM myguests");
$statement->execute();

$statement->setFetchMode(PDO::FETCH_ASSOC);
$statement->fetch();
```

setFetchMode()可以設定讀取資料的格式，預設為`PDO::FETCH_BOTH`，拿到的資料會是索引和欄位名稱混合

```text
Array
(
    [id] => 1
    [0] => 1
    [firstname] => John
    [1] => John
    [lastname] => Doe
    [2] => Doe
    [email] => john@example.com
    [3] => john@example.com
    [reg_date] => 2017-09-21 16:42:09
    [4] => 2017-09-21 16:42:09
)
```

設定為`PDO::FETCH_ASSOC`，只會留下欄位名稱

```text
Array
(
    [id] => 1
    [firstname] => John
    [lastname] => Doe
    [email] => john@example.com
    [reg_date] => 2017-09-21 16:42:09
)
```

* fetch(): 可以讀取一筆資料，讀取完自動指向下一筆
* fetchAll(): 可以讀取全部資料
* fetchColumn(index): 可以讀取特定欄位的資料，index填入欄位的索引順序，預設讀取index = 0，讀取完自動指向下一筆

## Delete Data

```PHP
$sql = "DELETE FROM myguests WHERE id = 2";
echo $db->exec($sql);
```

## Update Data

```PHP
$sql = "UPDATE myguests SET lastname='Doeeeee' WHERE id = 1";
$statement = $db->prepare($sql);
$statement->execute();

echo $statement->rowCount(); // 影響總行數
```

## Limit Data

SQL語句

`SELECT * FROM myguests LIMIT 10 OFFSET 5` 跳過5筆後讀取10筆資料

簡寫`SELECT * FROM myguests LIMIT 5, 10` (兩個數字會倒過來)

## 練習範例code

<https://github.com/comicat-hu/php-mysql>
