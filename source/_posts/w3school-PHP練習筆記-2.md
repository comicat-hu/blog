---
title: w3school-PHP練習筆記-2
date: 2017-09-18 10:08:04
tags:
  - PHP
---
* PHP Cookies
* PHP Sessions
* PHP Filters
* PHP Filters Advanced
* PHP Error Handling
* PHP Exception

<!--more-->

## PHP Cookies

[php setcookie](http://php.net/manual/zh/function.setcookie.php)

* setcookie(name, value, expire, path, domain, secure, httponly);
    + name: cookie名稱，必填
    + value: cookie值，可以透過`$_COOKIE['name']`取得
    + expire: 該筆cookie消失時間(timestamp)，設0的話瀏覽器關閉才消失
    + path: 該筆cookie的有效路徑
    + domain: 該筆cookie的有效網域
    + secure: 只能在https使用(true/false)
    + httponly: 只能透過http訪問(true/false)
* 必須在產生輸出前設定(包含html標記前)，如果在此之前就產生輸出，setcookie()就會失敗，return bool(false)

## PHP Session

* 使用前必須執呼叫`session_start();`，PHP會自動在cookie中建立一串PHPSESSID的字串
  {% asset_img PHPSESSID.PNG PHPSESSID %}
* 使用`$_SESSION`來直接存取session
* `session_start()`裡面可以帶入陣列參數:
    + `'cookie_lifetime' => 86400` 覆蓋cookie的消失時間設定
    + `'read_and_close'  => true` 讀取後立即關閉session

## PHP Filters

* 用來驗證資料(Validate)或清除資料中不合法的數據(Sanitize)
* 輸出`filter_list()` 可以列出所有PHP支援的filters
* `filter_var($str, filter)`
* [PHP Sanitize filters](http://php.net/manual/en/filter.filters.sanitize.php)

## PHP Filter Advanced

* `filter_var()`的第三個參數可以填入一些設定參數

**當然，不要太相信filter的結果，[PHP filter_var](http://php.net/manual/en/function.filter-var.php)底下也已經有很多人提供奇怪的測資，而被驗證合法的。**

## PHP Error Handling

* 用`die()`函數，輸出錯誤訊息並且停止腳本
* 自訂錯誤訊息

```PHP
function myError($errno, $errstr)
{
    echo "<b>Error: </b> [$errno] $errstr";
}

set_error_handler('myError'); // 設定錯誤訊息函式
```

* `trigger_error($errMsg);` 直接觸發錯誤訊息

* mail log 嘗試去php.ini和sendmail.ini設定了smtp=ssl://smtp.gmail.com, smtp_port, openssl等設定，執行完畢沒有錯誤訊息, 但沒收到信, 有空再研究。[參考](https://stackoverflow.com/questions/21836282/php-function-mail-isnt-working)

## PHP Exception

* `throw new Exception()` 拋出新的例外狀況
* 自訂一個例外狀況: 從Exception繼承class下來改寫

```PHP
/**
 * Self exception class
 */
class MyException extends Exception
{
    /**
      * Process error message
      *
      * @return string $errorMsg Error message
     **/
    public function errorMessage()
    {
        $errorMsg = 'Error on line' . $this->getline() . ' in ' .
                    $this->getFile() . ':<b>' . $this->getMessage() . '</b>';
        return $errorMsg;
    }
}
```

* 調用自訂的例外狀況，並傳入錯誤訊息

```PHP
try {
    throw new MyException('OOOOOOOOOOOOOOPS!');
} catch (MyException $e) {
    echo $e->errorMessage();
}
```

{% asset_img exception_example.PNG exception_example %}
(上面是原生的，下面是使用myException的輸出結果)
