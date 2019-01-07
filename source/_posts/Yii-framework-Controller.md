---
title: Yii framework - Controller
date: 2017-10-06 11:59:32
tags:
  - PHP
  - Yii1.x
---
Controller處理使用者發出並經過經過應用呼叫的request

## Controller

Controller繼承自CController，但由於可能會有很多Controller，為了提高共用性，可在`protected\components`中定義了`class Controller extends CController`，再由各種情境自行去定義`class xxxController extends Controller`並存放在`protected\controllers`下。

controller執行時會呼叫相對應的action，action會與model溝通並將結果呈現在view上。

<!--more-->

預設(沒有指定route)的action是index(actionIndex)， `CController::defaultAction = 'xxx'`可以設定預設action

名稱中的`Controller`後綴和`action`前綴都是必須的寫法規則!

```PHP
class SiteController extends Controller
{
    public function actionIndex()
    {
        echo 'Action Index!';
    }
    
    public function actionHello()
    {
        echo 'Action Hello!!';
    }
}
```

## Route

使用者可以透過設定不同的route參數來存取特定的controller和action，參數名稱預設是使用`r`(CUrlManager::routeVar設定)，參數值由controller前綴詞和action的後綴詞加上`/`所組成，所以我們要存取上方範例程式中的兩個action，預設狀況的URL大概會長的像這樣
site對應到SiteController

`index.php?r=site/index`, `index.php?r=site/hello`

如果路由長這樣`r=a/b/hello`，則會自動對應路徑`controllers/a`下的BController，並呼叫其中的actionHello()

預設大小寫視為不同，但是可以設定`CUrlManager::caseSensitive = false;`讓大小寫視為相同。

另外參考:
<http://www.cnblogs.com/JosephLiu/archive/2011/12/26/2301771.html>

## processRequest()

前篇有提到，在CWebApplication啟動了應用後呼叫run()，其中包含執行了processRequest()，這時就是在將接到的request依照r參數的不同去建立出不同的controller。

在這裡會先判斷`CWebApplication::catchAllRequest`是否有被設定，這個通常是用來導向某固定頁面時使用(像是頁面維護中之類的)，正常狀態會建立CUrlManager、ChttpRequset物件，並且透過CUrlManager中的parseUrl方法，處理httpRequest，拿到需要的參數(r)

呼叫createController()傳入r參數，開始解析(用\切)，將正確的controller id及action id區分開來，建立對應的controller

確認建立完成後，runController()，並且執行對應的action

## action參數

action函數可以直接指定傳入參數，並且會自動從`$_GET`中綁入(命名需相同)，

函數中的參數如果沒有給預設值，然後又在request找不到對應的參數時，會出現例外錯誤。

傳入參數特別宣告為array的話，會自動將傳入的值轉成陣列形式(放在index 0)

## actions()

actions()可以用來動態指定要執行的動作，也可以讓宣告的動作被重複利用，

* 將需要複用的動作獨立出來成class，繼承`CAction`，定義其中的run()方法(action要執行的內容)
* 在需要的controller中覆寫actions()，並且回傳action name與class action路徑的array-hash表
  (actions()是預定義在CController中的方法，預設回傳空陣列)

以實作一個簡易計算功能為例，像是我在`controllers\math`下定義了兩隻不同的action，一隻做加法一隻做減法

```PHP
class addAction extends CAction
{
    public function run($a, $b)
    {
        $sum = $a + $b;
        echo $sum;
    }
}
```

```PHP
class subAction extends CAction
{
    public function run($a, $b)
    {
        $diff = $a - $b;
        echo $diff;
    }
}
```

這時我在不同的controller中只要這樣寫，就可以用`控制器名稱/add&a=10&b=5`和`控制器名稱/sub&a=10&b=5`來計算10+5和10-5了

```PHP
public function actions()
{
    return [
        'add' => 'application.controllers.math.addAction',
        'sub' => 'application.controllers.math.subAction',
    ];
}
```

--

## Filter

filter可以用來設定動作執行的過濾條件，可以直接在controller中定義一個名稱為filter前綴的方法，也可以另外自己定義一個filter物件(extends CFilter)，

`$filterChain`是一個`CFilterChain`物件實體，包含了與該動作相關的filter list

* controller中的filter

```PHP
public function filterTest($filterChain)
{
    // Do something filter
    $filterChain->run(); // 呼叫下一個filter執行
}
```

* 自定義filter class (驗證URL qurey中的參數a, b都是數字)

```PHP
class MathFilter extends CFilter
{
    // preFilter()會在action執行前被呼叫
    protected function preFilter($filterChain)
    {
        $request = Yii::app()->request; // 取得request物件
        $a = $request->getQuery('a'); // 取出qurey參數值
        $b = $request->getQuery('b');

        // 回傳true表示繼續執行下個filter
        if (is_numeric($a) && is_numeric($b)) {
            return true;
        }

        // 回傳false表示停止執行下個filter
        echo 'Invalid query params.';
        return false;
    }
    // postFilter()會在action執行後被呼叫
    protected function postFilter()
    {
    }
}
```

* 在controller中使用filter，覆寫從CController繼承來的filters()方法，回傳一個陣列，包含filter name或路徑(array)，會依照回傳的陣列index的順序呼叫執行

* filter可以使用`+`(只有), `-`(除外)來表示執行為特定的action過濾，不寫預設為全部的action都會呼叫過濾

```PHP
// 對add, sub, mul, div四個action預執行controllers\filters下的MathFilter
// 再對全部的action預執行filterTest
public function filters()
{
    return [
        [
            'application.controllers.filters.MathFilter + add, sub, mul, div',
        ],
        'test',
    ];
}
```

## accessControl

在之前文章中我們使用Gii生成的table CRUD controller中，有一個`accessRules()`，這是給`filterAccessControl()`(從CController繼承)所使用的，因此我們可以覆寫`accessRules()`來定義自己的權限控制(預設是空陣列)

```PHP
public function filterAccessControl($filterChain)
{
    $filter = new CAccessControlFilter;
    $filter->setRule($this->accessRules());
    $filter->filter($filterChain);
}
```

這段是table controller中的accessRules()
{% asset_img yii_accessRules.PNG yii_accessRules %}

權限是由上而下逐條檢查的，有符合就不會繼續套用下去
`*` : 表示所有使用者
`@` : 表示以驗證的使用者
`?` : 表示匿名使用者

所以我們

* 允許所有使用者都可以執行action index和view
* 允許以驗證後的使用者才可以執行action create和update
* 只有admin可以執行action admin和delete
* 沒有套用到任何權限的使用者不允許任何操作(通常設定這個預防萬一)

當然不只可以設定actions和users參數，可以參考底下網址。

另外參考資料:

* [accessRules()](http://blog.gxxsite.com/yii-quan-xian-guan-li-accessrulesde-yong-fa/)
* [filters](http://fanli7.net/a/bianchengyuyan/PHP/20130418/341941.html)
