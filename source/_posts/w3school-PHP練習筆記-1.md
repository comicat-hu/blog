---
title: w3school PHP練習筆記-1
date: 2017-09-02 18:20:05
tags:
  - PHP
  - 學習筆記
---
[https://www.w3schools.com/php/php_forms.asp](https://www.w3schools.com/php/php_forms.asp)

* PHP Forms
* PHP Advanced
* MySQL Database
* PHP - AJAX

* PHP Forms - 全
* PHP Advanced
    + PHP Arrays Multi
    + PHP Date and Time
    + PHP Include
    + PHP File Handling
    + PHP File Open/Read
    + PHP File Create/Write
    + PHP File Upload

## include vs require

include與require同是引入檔案

實驗在include及require不存在的檔案之後echo "Hello"

* 使用include引入時，如果檔案不存在或錯誤發生，會出現警告訊息，但程式仍然會繼續執行。

  圖中可以看到下面的Hello仍然有輸出
  {% asset_img include_file_error.PNG include file error %}

* 使用require引入時，如果檔案不存在或錯誤發生，會出現錯誤訊息，程式會立即中斷執行。

  錯誤之後就沒有輸出了
  {% asset_img require_file_error.PNG require file error %}

* `include_once` 及 `require_once`同以上效果，多加了能避免重複引用的狀況。

## PHP Forms重點摘要

### Form

* Form Handling
    + `$_POST`和`$_GET`可以取得form submit的資料，型態是array
* Form Validation
    + `$_SERVER["REQUEST_METHOD"]`可以取得request method，型態是string大寫
    + `$_SERVER["PHP_SELF"]`可以取得目前動作的檔案路徑
    + 使用`htmlspecialchars()`將輸入的html特殊字轉換，避免注入攻擊
        - `&` 轉成 `&amp;`
        - `"` 轉成 `&quot;`
        - `'` 轉成 `&#039;`
        - `<` 轉成 `&lt;`
        - `>` 轉成 `&gt;`
    + 使用`trim()`去除不必要的空白字元，預設會去除頭尾空白字元
        - 不加第二個參數，預設還會去除字串中的" ", "\t", "\n", "\r", "\0", "\x0B"
    + 使用`stripslashes()`去除反斜線
* Form Required & Validation E-mail and URL
    + 檢查必要欄位(如果使用HTML設定屬性required只能限制input有值，如果輸入一堆空白字元仍然會通過檢查)
    + 用上面Validation中的函數處理完輸入資料後，使用`empty()`檢查不得為空值。
    + 使用`preg_match()` 用regular expression檢查輸入(測試前輸入參數都會先轉string) [php preg_match()](http://php.net/manual/en/function.preg-match.php)
    + 使用`filter_var()` 用內建的特殊規則檢查輸入，這邊範例是用`FILTER_VALIDATE_EMAIL`(驗證email) [php filter_var()](http://php.net/manual/en/function.filter-var.php)
        關於empty()一些問題以及比較其他檢查函數可以參考[isset-vs-empty-vs-is_null](https://github.com/comicat-hu/php-note/blob/master/php.md#isset-vs-empty-vs-is_null)
* Form Complete
    + 表單送出之後，input欄位中的值會不見，依需求可以將表單的值再回填

### 用以下方式處理迴圈

```PHP
foreach ($variable as $key => &$value) {
    //code...
}
```

如果在迴圈結束後沒有`unset($value)`會發生記憶體位置指向不正確，
而導致之後取值可能發生問題，var_dump()結果會看到&記號，標示出該型態為一個指向(類似指標的概念??

如果非必要也許直接用明確的陣列存取方式會穩定一些。

### PHP變數範圍

PHP的變數作用範圍(Scope)通常都只有小區域，尤其是當一個變數需要跨function內外處理的時候更需要注意。
一般變數宣告後，可以在自身、include和require的檔案中作用，但不包括function內部。

* function內外的變數是分開的

```PHP
$a = 5;
$b = 7;

function add()
{
    $a ++;
    echo $a + $b; // null
}

add();
var_dump($a, $b); // int(5) int(7)
```

輸出會出現警告訊息，因為$a和$b在add()內未定義值，於是結果會是null。

* 傳值進去function

```PHP
$a = 5;
$b = 7;

function add($a, $b)
{
    $a ++;
    echo $a + $b; // 13
}

add($a, $b);
var_dump($a, $b); //int(5) int(7)
```

這時我們定義兩個參數$a, $b給add()，並且把外面的$a, $b透過參數傳進add()內，得到$a + $b結果為13，但外頭的$a, $b仍然維持原值。

* 傳參考進去function

```PHP
$a = 5;
$b = 7;

function add(&$a, &$b)
{
    $a ++;
    echo $a + $b; // 13
}

add($a, $b);
var_dump($a, $b); //int(6) int(7)
```

這時add()內的對於變數的操作，則會直接影響到外面(因為參考同一個變數操作)，外面的$a, $b值被更新。

* 使用global宣告

```PHP
$a = 5;
$b = 7;

function add()
{
    global $a, $b;
    $a ++;
    echo $a + $b; // 13
}

add();
var_dump($a, $b); //int(6) int(7)
```

global關鍵字表示這裡宣告的變數將會參考外部全域的變數，是直接指向外部變數去操作，所以內外連動。

* 使用預設的$GLOBALS陣列來存取

```PHP
$a = 5;
$b = 7;

function addGlobal()
{
    $GLOBALS['a'] ++;
    echo $GLOBALS['a'] + $GLOBALS['b']; // 13
}

addGlobal();
var_dump($a, $b); //int(6) int(7)
```

$GLOBALS裡存放了許多預設的全域變數，包括用global宣告過的變數，而使用$GLOBALS['a']也會自動與$a做連動，不管是存取更動了哪一方的值，都相當於修改了兩邊。(等同於上面global用法)

* static宣告

使用static靜態宣告的變數，將會在程式執行前就分配記憶體空間，並且在程式執行結束後才釋放，並且不會因為多次呼叫而重置內容。

```PHP
function add()
{
    static $a = 0;
    $a ++;
    echo $a;
}
add(); // 1
add(); // 2
```

### PHP Advanced 重點摘要

* Array Multi
    + 多維陣列的存取
    + 把多維陣列分類進表格及清單中
  {% asset_img array_multi.PNG array multiple %}

* Date and Time
    + `date()`取得伺服器當前日期，`time()`取得伺服器當前時間，預設為GMT+0，第一個參數都接格式，第二個參數接timestamp
    + `date_default_timezone_set("Asia/Taipei")` 將時區設定為台北時間。(也可以到php.ini中直接更改date.timezone的設定)
    + 時間日期格式[php date](http://php.net/manual/zh/function.date.php)
    + `mktime(時,分,秒,日,月,年)` 自定義時間
    + `strtotime($str)` 把$str字串轉成timestamp，$str大致上可以接受英文格式的日期時間，+-符號等組成 [php strtotime()](http://php.net/manual/zh/function.strtotime.php)
    + 使用strtotime()的第二個參數來計算相對時間區間

* File Handling
    + 用`readfile()`讀取txt檔案，顯示檔案內文並回傳字數

* File Open/Read
    + 使用`@fopen(url, "r") or die("Error!");`開啟檔案。
        - `fopen()`成功會回傳一個resource，錯誤的話回傳bool(false)
        - `@`符號是一個錯誤運算符，它能隱藏系統出現的錯誤訊息，它只能被加在表達式前面(有拿到回傳值)
        - `die()`函數用於退出程式，並且輸出填入的錯誤字串
    + `fread(file resource, filesize(url));` 讀取檔案，回傳檔案內容(string)，第二個參數指定要讀取的長度，這邊直接用檔案長度為參數
    + `fclose(file resource);` 關閉檔案釋放資源。
    + `fgets(file resource);` 單行讀取
    + `feof();` 回傳是否檔案結尾
    + `fgetc();` 單一個字元讀取

* File Create/Write
    + `fwrite(file resource, string)` 寫入檔案，第三個參數可以帶入最大寫入長度

[@ or die()用法解釋](http://www.webkaka.com/tutorial/php/2013/060736/)

#### 關於fopen() mode參數設定

覆寫意指，清空原來的內容並從起始位置開始寫入。(一開檔就先清除)

* r  : 唯讀，指標指向檔案起始位置(從頭覆蓋)
* w  : 唯寫，指標指向檔案起始位置(覆寫)，如果檔案不存在會新增一個
* a  : 唯寫，指標指像檔案結束位置(新增)，如果檔案不存在會新增一個
* x  : 唯寫，直接新增檔案，如果檔案已經存在回傳false並有錯誤訊息
* r+ : 可讀可寫，指標指向檔案起始位置(從頭覆蓋)
* w+ : 可讀可寫，指標指向檔案起始位置(覆寫)，如果檔案不存在會新增一個
* a+ : 可讀可寫，指標指像檔案結束位置(新增)，如果檔案不存在會新增一個
* x+ : 可讀可寫，直接新增檔案，如果檔案已經存在回傳false並有錯誤訊息

### File

* File Upload
    + 確認php.ini中的設定file_uploads=On (upload_max_filesize可以限制上傳檔案大小，同時post_max_size必須大於upload_max_filesize，memory_limit > post_max_size)
    + 設定form method="post"，設定enctype="multipart/form-data"
    + 設定input type="file"，預設會出現如圖中紅框內的樣式，選擇檔案會跳出檔案對話窗
    {% asset_img input_type_file.PNG input_type_file %}
    + 上傳之後的檔案資訊可以透過`$_FILES["input name"]`取得(裡面會包含name, type, tmp_name, error, size)
    {% asset_img upload_file_info.PNG upload_file_info %}
    + basename(path)函數可用於取得路徑中檔案的名稱
    + pathinfo(path, option) option不填預設是回傳array("dirname" => "", "basename" => "", "extension" => "")
    + option = PATHINFO_DIRNAME，回傳string(dirname)
    + option = PATHINFO_BASENAME，回傳string(basename)
    + option = PATHINFO_EXTENSION，回傳string(extension name)
    + 上傳的檔案會被放到xampp/tmp資料夾下，並且有一個系統產生的tmp_name
    + 可以透過move_uploaded_file(tmp_name, target path) 來移動，target path要是一個完整包含檔案名稱的路徑
    + file_exists(path) 檢查檔案是否存在
    + getimagesize(path)可以取得關於圖片的一些資訊，如果不是回傳false
    {% asset_img get_image_size.PNG get_image_size %}
    + 0 存放寬度
    + 1 存放高度
    + 2 存放圖片類型(1 = GIF，2 = JPG，3 = PNG，4 = SWF，5 = PSD，6 = BMP，7 = TIFF(intel byte order)，8 = TIFF(motorola byte order)，9 = JPC，10 = JP2，11 = JPX，12 = JB2，13 = SWC，14 = IFF，15 = WBMP，16 = XBM)
    + 3 存放大小資訊(string)
    + bits 存放顏色的位元數
    + channels 存放圖片的通道值，RGB圖檔是3
    + mime 存放圖片的MIME

#### 關於html form enctype屬性

這個屬性在method="post"時才有效，用於指定送出表單時對資料的編碼格式。

enctype有以下三種設定值:

* `application/x-www-form-urlencoded` : 預設值，會將空白編碼為"+"，特殊字元轉換為ASCII HEX
* `multipart/form-data` : 不進行編碼，通常用於form上傳檔案時使用
* `text/plain` : 只有空白會編碼成"+"
