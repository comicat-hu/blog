---
title: PHP5 Codecademy課程學習
date: 2017-08-29 15:54:06
tags:
  - PHP
---
Learn Url: [https://www.codecademy.com/learn/php](https://www.codecademy.com/learn/php)

<!--more-->

適逢Codecademy整理課程，這版的課程使用的是比較舊的php版本(5.3.10-1ubuntu3.1)，而且Codecademy也已經計畫更新並在主頁下架該課程首頁，不過新版的似乎還沒上架，所以這個url可能會404。

目前還是可以直接連進課程內容練習[Introduction to PHP](https://www.codecademy.com/courses/web-beginner-en-StaFQ/0/1) (已失效)

p.s. Codecademy比較偏向語法快速上手，但是對於php的介紹比較少。

建議php初學者可以閱讀官方文件，先瞭解一些基本的php特性，對於基本語法的介紹也更臻完整。
PHP文件: [http://php.net/docs.php](http://php.net/docs.php)

邊看文件邊整理的一些PHP入門需知(持續更新): [php-note](https://github.com/comicat-hu/php-note/blob/master/php.md)

## PHP CodeSniffer

可以幫助你檢查程式碼是否有符合PSR的規範。

使用XAMPP安裝PHP CodeSniffer套件

因為使用XAMPP裝的PHP已經有pear套件了，所以就不用再裝。
打開XAMPP shell輸入指令
`pear install PHP_CodeSniffer`
安裝完成畫面
{% asset_img phpcs_install.PNG phpcs install %}

在VSCode中可以安裝phpcs套件來引入PHP CodeSniffer使用。
一些設定在phpcs的說明頁中可以查詢。

## php-cs-fixer

修正程式碼符合PSR-2
[PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer)

我這邊是直接載phar檔。
vscode中安裝php-cs-fixer extension，設定php-cs-fixer.executablePath指到剛載下來的檔案。
F1 search : php-cs-fixer: fix this file 使用

## 課程重點摘要

### Ch1 Introduction to PHP

* PHP程式碼撰寫在`<?php ?>`中，並且附檔名須為.php
* echo 可以顯示字串在螢幕上，字串連接使用 '.'，ex: `"Hello" . "World!"`
* 變數使用$開頭命名(這是用法不是風格規範)，程是句尾加;

### Ch2 Conditionals and Control Flow

* 這邊的內容不太照PSR規範來寫

正常的PSR寫法應該為:

```PHP
if ($expr1) {
    // if body
} elseif ($expr2) {
    // elseif body
} else {
    // else body;
}
```

* elseif 和 else if 的差異: [php elseif](http://php.net/manual/zh/control-structures.elseif.php)
  使用`else if` 在 : endif 的寫法下會導致解析錯誤，花括號寫法則沒有差異。

### Ch3 Control Flow: Switch

* switch的case一定要加break;
* case可以多重使用。

### Ch4 Arrays

* php5.4版後，array可以有兩種宣告方式，`$arr = array();` 或 `$arr = [];`，不過在這個課程只支援前者。
* 使用`[]`或`{}`都可以存取array的值，`unset()`可以用來清除特定值或整個變數。

### Ch5 For Loops in PHP

* 使用`foreach`存取陣列

```PHP
foreach ($langs as $lang) {
    echo "<li>$lang</li>";
}
```

### Ch6 While Loops in PHP

* 記得`do while`的結尾要加;

### Ch7 Functions, Part I

* `strlen($str)` 回傳$str字串長度
* `substr($str, $begin, $length)` 回傳從$str字串的$begin位置取$length長度的子字串
* `strtoupper($str)` 轉大寫，`strtolower($str)` 轉小寫
* `strpos($str1, $str2)` 回傳$str2在$str1中的位置(int), 如果沒有找到回傳bool(false)
* `round($num [, $p])` 回傳$num四捨五入到小數點第$p位的值($p值預設為0)
* `rand()` 隨機回傳一個0 ~ RAND_MAX的整數，`rand($min, $max)` 隨機回傳一個$min ~ $max的整數(PHP4.2+ 不需要再使用種子亂數，已經自動內含了)
* `array_push($arr, $v1 [, $v2......])` 將$v1及之後的參數值push進$arr中，回傳完成後的陣列長度
* `count($arr)` 回傳$arr陣列長度(註1)
* `sort($arr)` 將$arr升冪排序，`rsort($arr)` 將$arr降冪排序
* `join($str, $arr)` 使用$str1連接$arr每個值，回傳字串
* `explode($str1, $str2)` 使用$str1當斷點分割$str2，回傳陣列
* `str_split($str [, $length])` 將$str分成每個$length長度，預設為1，回傳陣列

### Ch8 Functions, Part II

* 自訂function傳入及傳出參數

### Ch9 Objects in PHP

* 使用`class`定義類別
* 使用`function __construct()` 定義建構子
* 使用`new`創建物件
* 使用`->`存取物件屬性及方法
* 使用`$this`定義物件自身的屬性值

### Ch10 Object-Oriented PHP

* `property_exists($obj, $str)` 檢查$obj物件中的屬性是否存在
* `method_exists($obj, $str)` 檢查$obj物件中的方法是否存在
* `extends`可以讓子類別繼承父類別的所有屬性和方法(不支援多重繼承)
* 父類別中使用`final`宣告的方法不能被覆寫
* 在類別定義中`parent::`可以用來呼叫父層的屬性或方法
* 在類別定義外使用 `ClassName::` 可以用來呼叫類別中的靜態變數、常數、靜態方法(issue1)

### Ch11 Advanced Arrays

* 使用key-value方式存取array
* 使用`foreach($arr as $value)`
* 使用`foreach($arr as $key => $value)`

#### issue1

雖然class中的function沒有宣告成static的話，理當是不能在new instance前被呼叫
但是不知道為什麼可以...，官方手冊是說這種用法會給warning，
不過PHP7之後這種呼叫方式會逐漸被棄用移除。
[PHP OOP static](http://php.net/manual/en/language.oop5.static.php)

#### 註1

在PHP4.2之後，count()函數有新增一個mode參數`count($arr, mode)`

mode參數不填寫預設為0，表示不遞回計算個數，意指只計算第一層的元素個數

```PHP
$arr = [
    [1,2],
    3,
    4,
    [5,6,7,[8]],
];

echo count($arr); // 4
```

mode參數設成1或COUNT_RECURSIVE，表示遞迴計算元素個數，意指加總所有層的元素個數

```PHP
$arr = [
    [1,2],
    3,
    4,
    [5,6,7,[8]],
];

echo count($arr, 1); // 11
```

$arr在第一層的有4個元素，
$arr在第二層的有6個元素，
$arr在第三層的有1個元素，

故4 + 6 + 1 = 11

### echo vs print() vs print_r()

* 比較表

| echo | print() | print_r()
---------|---------|----------|---------
 型態 | 語句 | 函數 | 函數
 接受參數 | 多個 | 一個 | 多個
 參數型態 | string, number | string, number | mixed
 回傳值 | 無 | int(1) | bool(true/false)

* print_r() 如果加上第二個參數true，則不會輸出而是return函數處理完的值。

### isset() vs empty() vs is_null()

先上手冊說明
[isset()官方說明](http://php.net/manual/en/function.isset.php)
[empty()官方說明](http://php.net/manual/en/function.empty.php)
[is_null()官方說明](http://php.net/manual/en/function.is-null.php)

* `isset()`: 檢查變數是否存在且不為null (可帶多個參數)
* `empty()`: 檢查變數值是否為空
* `is_null()`: 檢查變數值是否為null

#### 實驗結果

(PHP 5.6.31)

為統一輸出避免顯示被自動轉型，所以我用var_dump()來處理輸出

輸出執行下方程式碼

```PHP
echo var_dump($var);
echo var_dump(isset($var));
echo var_dump(empty($var));
echo var_dump(is_null($var));
```

* 不宣告$var
    + $var出現未定義變數警告訊息，並回傳NULL
    + isset()回傳bool(false)
    + empty()回傳bool(true)
    + is_null()出現未定義變數警告訊息，並回傳bool(true)
* 宣告$var = null;
    + $var輸出得到NULL
    + isset()回傳bool(false)
    + empty()回傳bool(true)
    + is_null()回傳bool(true)
* 宣告$var = 0;
    + $var輸出得到int(0)
    + isset()回傳bool(true)
    + empty()回傳bool(true)
    + is_null()回傳bool(false)
* 宣告$var = "0";
    + $var輸出得到string(1) "0"
    + isset()回傳bool(true)
    + empty()回傳bool(true)
    + is_null()回傳bool(false)
* 宣告$var = "";
    + $var輸出得到string(0) ""
    + isset()回傳bool(true)
    + empty()回傳bool(true)
    + is_null()回傳bool(false)
* 宣告$var = 1;
    + $var輸出得到int(1)
    + isset()回傳bool(true)
    + empty()回傳bool(false)
    + is_null()回傳bool(false)

#### 結論

* `isset()`只要變數存在不為null都會回傳true
* `empty()`會把null, 0, "0", 空字串, 空陣列...等空集合的值都視為空值，回傳true
* `is_null()`只在變數不存在或值為null時，回傳true

### 參考

[php.ini重要設定](http://ithelp.ithome.com.tw/articles/10189840)
