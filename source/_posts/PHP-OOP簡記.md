---
title: PHP OOP簡記
date: 2017-09-22 10:33:49
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - OOP
---

* OOP三大用法簡介
* $this , self, parent
* \_\_construct() 和 \_\_destruct()
* 物件複製
* abstract 和 interface
* trait
* namespace , use , as 用法
* spl_autoload_register()

<!--more-->

另外找到還不錯的系列文章

* [iT邦幫忙: PHP物件導向](http://ithelp.ithome.com.tw/search?search=PHP%E7%89%A9%E4%BB%B6%E5%B0%8E%E5%90%91&tab=article)
* <https://old-oomusou.goodjack.tw/php/php-namespace/>
* <http://ithelp.ithome.com.tw/users/20000108/ironman/690?page=1>

## OOP 三大用法簡介

* 封裝(Encapsulation) : 將功能相近的變數(屬性)、函式(方法)使用類別(class)包裝，使用public、private、protected關鍵字宣告，區分變數及函式的存取權限。
    + public    : 開放物件本身及外部程式使用
    + private   : 限定物件本身使用(通常私有成員會在命名前加上_區別)
    + protected : 限定物件本身及繼承類別使用

* 繼承(Inheritance) : 類別可以使用extends來繼承父類別的屬性及方法。(PHP不能多重繼承)

```PHP
class Transportation
{
    public $cost = 100;
    private $_hasDriver = true;

    public function run()
    {
        echo 'Go!';
    }
}

class Car extends Transportation
{
}

$myCar = new Car();

$myCar->run(); // Go!
```

繼承後子類別擁有父類別的所有屬性和方法，但可以被重載(overloading)

* 多型(Polymorphism) : 基於同一個class或interface可以去定義不同的class並實作

## $this , self, parent

* $this在物件實體化時指向物件主體，使用`->`
* self指向類別本身，通常會用在存取靜態成員，使用`::`
* parent指向父類別，存取父類別的成員，使用`::`

參考: <http://www.howzhi.com/group/php/discuss/10022>

## \_\_construct() 和 \_\_destruct()

* \_\_construct()中定義物件實體化時要做的事，基本原則會用來預設屬性值或預執行函式
* \_\_destruct()用於定義物件銷毀前做的事，一般在程式結束前物件會被自動銷毀

```PHP
class Demo
{
    public $hello = 'Hello World' . "\n";

    public function __construct()
    {
        echo 'run __construct()' . "\n";
    }

    public function __destruct()
    {
        echo 'run __destruct()' . "\n";
    }

    public function hello()
    {
        echo $this->hello;
    }
}

$demo = new Demo();
$demo->hello();
```

Output:

```text
run __construct()
Hello World
run __destruct()
```

使用同類別名稱也可以定義建構式，但PHP5後通常不會這麼使用也不建議

```PHP
class Demo
{
    public function Demo()
    {
    }
}
```

**這類由雙底線開頭的方法，稱為magic method，系統會在特定的時機調用這些方法。**
可以參考:

* [PHP的語言特性 : magic methods](http://ithelp.ithome.com.tw/articles/10135522)
* [PHP手冊 magic methods](http://www.php.net/manual/en/language.oop5.magic.php)

## 物件複製

先上類別定義

```PHP
class A
{
    public $count = 1;
    public function __clone()
    {
        $this->count ++;
    }
}
```

* 方式1，直接assign(reference)

```PHP
$a = new A();

$b = $a;

var_dump($a);
var_dump($b);

var_dump($a === $b);
```

Output:

```text
class A#1 (1) {
  public $count =>
  int(1)
}

class A#1 (1) {
  public $count =>
  int(1)
}

bool(true)
```

這時$b完全參照$a，而且兩邊會一起變動，如果`$b->count = 99`改了count值，`$a->count`也等於99

* 方式2，使用clone(shallow copy)

```PHP
$a = new A();

$b = clone $a;

var_dump($a);
var_dump($b);

var_dump($a === $b);
```

Output:

```text
class A#1 (1) {
  public $count =>
  int(1)
}

class A#2 (1) {
  public $count =>
  int(2)
}

bool(false)
```

結果有些不一樣了，當使用clone複製完後，系統會自動去執行`__clone()`這個magic method，而且修改count值並不會改變另一邊

不過當物件的屬性值是參考或是物件時，仍然會是以參考的形式複製

我們把class宣告改一下

```PHP
class A
{
    public $count = 1;
    public $obj;

    public function __construct()
    {
        $this->obj = new B();
    }

    public function __clone()
    {
        $this->count ++;
    }
}

class B
{
    public $say = 'yes';
}


$a = new A();

$b = clone $a;

var_dump($a);
var_dump($b);

$b->obj->say = 'no';

var_dump($a);
var_dump($b);
```

Output:

```text
class A#1 (2) {
  public $count =>
  int(1)
  public $obj =>
  class B#2 (1) {
    public $say =>
    string(3) "yes"
  }
}

class A#3 (2) {
  public $count =>
  int(2)
  public $obj =>
  class B#2 (1) {
    public $say =>
    string(3) "yes"
  }
}

class A#1 (2) {
  public $count =>
  int(1)
  public $obj =>
  class B#2 (1) {
    public $say =>
    string(2) "no"
  }
}

class A#3 (2) {
  public $count =>
  int(2)
  public $obj =>
  class B#2 (1) {
    public $say =>
    string(2) "no"
  }
}
```

可以發現我們改了`$b->obj->say`，連$a那邊的也被改了

### 解決辦法

這時我們只要在`__clone()`中加上`$this->obj = clone $this->obj;`，把該物件另外再複製一份，就可以獨立兩邊的值了。

## abstract 和 interface

先上程式碼

```PHP
interface Action
{
    public function run();
    public function fast();
}

abstract class Animal implements Action
{
    public function run()
    {
        $this->fast();
    }
}

class Dog extends Animal
{
    public function fast()
    {
        echo 'very fast.';
    }
}

class Cat extends Animal
{
    public function fast()
    {
        echo 'very slow.';
    }
}

$myDog = new Dog();
$myCat = new Cat();

$myDog->run(); // very fast.
$myCat->run(); // very slow.
```

* interface(介面): 定義必須被實作的方法。上面我們宣告了Action介面，並定義實現Action的所有類別都必須實作run()跟fast()兩個方法。
* abstract class(抽象類別): 定義一個不可被實體化的類別，表示一種型態、一個上層的分類概念。這裡我們宣告了一個Animal抽象類別，並且用implements關鍵字表示要依照Action介面來實現。

  這邊問題來了，但在Animal中我們並沒有依照Action實作fast()方法，而為什麼不會報錯呢?

  是因為我們宣告成abstract，所以在這裡Animal並不會被實作，也不能被實作，自然也不用去實現，它就是個抽象虛擬的定義...(覺得很難表達...)

  當然也可以由每種動物各自去實現Action，但是這樣就沒有層級的概念了，而且在這邊每種動物的run()方法是一樣的。

* extends(繼承): 這裡我們宣告Dog和Cat類別，都繼承Animal抽象類別，包含實現Action的方法和run()方法都繼承下來了，而因為這邊是實際的類別，所以必須依照interface Action的定義去實作出還沒被實作的fast()方法。

最後我們實體化出$myDog和$myCat物件，並且呼叫其中的run()方法。

output: `very fast.very slow.`

## trait

(特徵)

PHP不能多重繼承，但是PHP5.4後加入了`trait`可以用來達成類似多重繼承的效果。

trait使用方式跟class很像。

延續上面那個程式，

這邊我們宣告一個trait Play，裡面有個run()方法。

```PHP
trait Play
{
    public function run()
    {
        echo 'run and play with people';
    }
}
```

然後我們在class Cat中使用它`use Play;`，這時Cat類別已經同時繼承了Animal又擁有了Play的內容，

屬性覆蓋的順序為: `類別 -> trait -> 父類別`

這時`$myCat->run();`的結果會變成`run and play with people`，因為在Cat類別中我們沒有實作run()方法，所以這時候呼叫會先採用Play中的run()。

也許trait的方法在每個類別中又需要不一樣，那也可以宣告成abstract抽象方法，讓每個類別去實作不同的方法。

參考: [逐步提昇PHP技術能力 - PHP的語言特性: Traits](http://ithelp.ithome.com.tw/articles/10133226)

## namespace , use , as 用法

* namespace 主要可以用於解決class, interface, function, constant在不同套件重複名稱的問題，將其階層式的管理起來。
* use關鍵字可用來使用需要的namespace
* as關鍵字可用來為namespace指定別名

```PHP
namespace Namespace1;

class Boy
{
    public function __construct($name)
    {
        $this->name = $name;
    }
    public function go()
    {
        echo 'Go NS1! ' . $this->name . "!!\n";
    }
}
```

```PHP
namespace Namespace2;

class Boy
{
    public function __construct($name)
    {
        $this->name = $name;
    }
    public function go()
    {
        echo 'Go NS2! ' . $this->name . "!!\n";
    }
}
```

然後我們把資料夾檔案整理成這樣
{% asset_img namespace_level.PNG namespace_level %}

## spl_autoload_register()

在use_namespace.php中使用`spl_autoload_register()`自動去載入需要的class，

PHP事實上會將類別名稱套用上完整的命名空間，

所以我們在`apl_auto_register()`中的`$class`實際上會拿到`Namespace1\Boy`和`Namespace2\Boy`

```PHP
spl_autoload_register(
    function ($class) {
        include_once 'vendors\\' . $class . '.php';
    }
);

use Namespace1 as NS1;
use Namespace2 as NS2;

$boy1 = new NS1\Boy('Comi');
$boy2 = new NS2\Boy('Comi');

$boy1->go(); // Go NS1! Comi!!
$boy2->go(); // Go NS2! Comi!!
```

## 範例程式碼參考

<https://github.com/comicat-hu/php-oop-test>
