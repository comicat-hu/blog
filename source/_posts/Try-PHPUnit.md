---
title: Try PHPUnit
date: 2017-09-19 14:12:53
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - PHPUnit
  - Unit-Test
---
[PHPUnit](https://phpunit.de/index.html)

## 安裝新的PHPUnit

xampp 安裝後裡面有附PHPUnit(3.7.21)，

如果path設定沒錯的話，在終端機下`phpunit --version`可以查詢版號。

1. 到​[https://phpunit.de/index.html](https://phpunit.de/index.html)下載，這邊我們下載PHPUnit5.7版(因為至少有支援到我們使用的PHP5.6)
2. 載下來後是一個phpunit-5.7.21.phar檔案，把它丟到xampp/php資料夾下。
3. 修改`xampp/php/phpunit.bat`的`:RUN "%PHPBIN%"`後的路徑檔名，改成剛剛載下來的檔案名(phpunit-5.7.21.phar)
4. `phpunit --version` 這時就可以看到新的PHPUnit版號了

{% asset_img phpunit_version.PNG phpunit version %}

<!--more-->

## 簡單的測試範例

* 在lib_test.php撰寫test case

```PHP
<?php

use PHPUnit\Framework\TestCase;

require_once 'lib.php';

class InputTest extends TestCase
{
    public function testNormal()
    {
        $this->assertEquals('a', convertInput('a'));
        $this->assertEquals('abc', convertInput('abc'));
        $this->assertEquals('aaa', convertInput('aaa'));
        $this->assertEquals('0', convertInput('0'));
    }

    public function testSpace()
    {
        $this->assertEquals('', convertInput(' '));
        $this->assertEquals('', convertInput('   '));
        $this->assertEquals('', convertInput("\0"));
    }
}
```

* lib.php的內容是一個簡單的轉換資料的function

```PHP
<?php

function convertInput($data)
{
    $data = trim($data); // Remove more space
    $data = stripcslashes($data); // Remove "\"
    $data = htmlspecialchars($data); // HTML special chars encode
    return $data;
}
```

* 執行phpunit lib_test.php結果

  {% asset_img phpunit_ok.PNG phpunit ok %}

## 在vscode執行phpunit

* 安裝`PHPUnit` extension
* 設定`"phpunit.exePath": "C:\\xampp\\php\\phpunit.bat"`
* 使用方式可參考[vscode-phpunit](https://github.com/elonmallin/vscode-phpunit)

執行狀況如下圖:

紅線上方為測試全部通過

紅線下方有一個案例失敗

{% asset_img phpunit_vscode.PNG phpunit_vscode %}

## composer install phpunit

[installation.composer](https://phpunit.de/manual/current/en/installation.html#installation.composer)

### 安裝composer

`composer --version`確認可以執行查看版本

我裝在全域路徑(windows的路徑是預設在`C:\Users\使用者名稱\AppData\Roaming\Composer`)，composer會自動把套件裝去那
`composer global require phpunit/phpunit ^5.7`

記得把這個指令集路徑`C:\Users\使用者名稱\AppData\Roaming\Composer\vendor\bin`加入到環境變數中

可以透過composer來管理這些套件，使用pear方式安裝的方法很多都已經不可行了，套件官方可能也不維護該頻道

### 備註

phpunit官方建議安裝的`php-invoker`需要相依PHP extension `pcntl`，

但這個extension根據網路上眾多說法是完全不能在windows環境使用，

所以composer在安裝時檢查會失敗。

如果仍要安裝可以加上`--ignore-platform-reqs`，讓composer忽略檢查條件

參考: <https://github.com/composer/composer/issues/4584>

## test private/protected function

<http://php.net/manual/en/class.reflectionclass.php>
<https://stackoverflow.com/questions/249664/best-practices-to-test-protected-methods-with-phpunit>

```PHP
/**
 * 取得 SomeClass 這隻class的特定方法，設定存取權限
 * @param string $name 方法的名稱
 * @return ReflectionMethod
 */
private function getMethod($name) {
    $class = new \ReflectionClass('SomeClass');
    $method = $class->getMethod($name);
    $method->setAccessible(true);
    return $method;
}

/**
 * 呼叫權限受限的方法時被執行
 * @param string $methodName 自動填入方法名稱
 * @param array  $args 自動填入呼叫方法的參數
 * @return mixed 該方法的執行結果
 */
public function __call($methodName, $args)
{
    $method = $this->getMethod($methodName);
    $result = $method->invokeArgs($this->api, $args);
    return $result;
}
```

範例:

當我`$this->getCache()`時，因為getCache在ApplyAnalysisToJob中是private方法，

所以存取被拒絕，進入`__call()`，啟動getMethod()，透過`ReflectionClass`回傳可存取的`ReflectionMethod`物件，

用`ReflectionMethod::invokeArgs()`帶入參數，執行該實體方法，得到結果。

## code coverage

原程式的測試覆蓋率

在`phpunit.xml`裡的filter區塊加入白名單設定，並且設定`addUncoveredFilesFromWhitelist="true"`

將要產生覆蓋率報告的資料夾或檔案引入

```xml
<whitelist addUncoveredFilesFromWhitelist="true">
    <directory suffix=".php">path\to\dir</directory>
    <file>path\to\file</file>
</whitelist>
```

這段設定表示加入一個資料夾(內符合suffix=".php"的檔案)，和兩個單檔，所以產出的報告會包含這些檔案的執行覆蓋率

透過這份覆蓋率報告可以得知，可能會有測試案例不滿足function中每行程式的情況，會有if, for沒有跑到之類的，表示測試案例不夠完整
