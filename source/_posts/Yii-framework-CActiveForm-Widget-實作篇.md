---
title: Yii framework - CActiveForm Widget(實作篇)
date: 2017-10-19 17:50:18
tags:
  - PHP
  - Yii1.x
---
這邊不會詳述關於CActiveForm(extends CWidget)的背後實作，不過可以大致說它是一個集成了各種表單常用的功能，並且封裝成一個物件的小工具，內部同樣包含了許多CHTML類別的調用

參考資料: <http://www.yiichina.com/doc/api/1.1/CActiveForm>

這邊我想直接使用CActiveForm造一個簡單的二元數字加減乘除(但要不同頁面)，

<!--more-->

先簡述一下mvc架構

* `MathController extends CController`: 這邊要負責導去不同的action，沒給action預設依然會進index
* `MathForm extends CFormModel`: 這邊建立關於傳入參數的資料模型，寫rules()驗證合法的數字格式
* views
    + 一個math layout用於整體html架構
    + 一個index view首頁內容，可以連結到各個action頁面
    + 一個math view建立CActiveForm Widget，並且可以呈現主要的輸出入結果和錯誤提示
* `MathAction extends CAction`(component): 因為每個加減乘除action的run()都一樣，所以就開了一個component來放共同方法
* action另外寫成class並extends MathAction，由MathController actions()統一處理

controllerID: math

actionID: add, sub, mul, div

## MathController extends CController

指定layouts\math_layout.php為layout輸出檔，然後將加減乘除的各個action導去不同的路徑，

這裡的application是根據我們的config設定而來的別名，指的是protected資料夾

```PHP
class MathController extends Controller
{
    public function actions()
    {
        $this->layout = 'math_layout';
        return [
            'add' => 'application.controllers.math.addAction',
            'sub' => 'application.controllers.math.subAction',
            'mul' => 'application.controllers.math.mulAction',
            'div' => 'application.controllers.math.divAction',
        ];
    }

    public function actionIndex()
    {
        $this->layout = 'math_layout';
        $this->render('index');
    }
}
```

## MathForm extends CFormModel

這裡定義a, b來存放輸入的兩個參數，並定義了驗證規則，

這個驗證規則會統一用在前端跟後端的驗證上(後面會提到)

```PHP
class MathForm extends CFormModel
{
    public $a;
    public $b;

    public function rules()
    {
        return [
            ['a, b', 'numerical'],
        ];
    }
}
```

## math view

在這裡的`$this`指的是MathController實體，或用`Yii::app()`也可以拿到

* `$this->action->id;`可以取得目前的actionID，實際上是`$this->getAction()->getId();`
* `$this->id;`可以取得目前的controllerID，實際上是`$this->getId();`

建立一個widget使用CActiveForm，啟用client驗證，設定要focus的欄位

建立兩個textField輸入框和一個submit button

```PHP
<?php
$form = $this->beginWidget('CActiveForm', array(
    'id' => 'math-form',
    'enableClientValidation' => true,
    'focus' => array($model, 'a'),
));

<div class="row submit">
    <?php echo $form->textField($model, 'a'); ?>
    <?php echo $sign;?>
    <?php echo $form->textField($model, 'b'); ?>
    <?php echo CHtml::submitButton('Submit'); ?>
</div>

<?php echo $form->error($model,'a'); ?>
<?php echo $form->error($model,'b'); ?>

<?php $this->endWidget(); ?>
```

而網頁中的一些動態功能則是會由CActiveForm去自動使用jQuery語法和yii自行定義的jQuery工具(jquery.yiiactiveform)來達成

呼叫error()讓它可以在前端驗證錯誤時丟出錯誤訊息，從網頁原始碼中可以看到它帶入了我們之前定義的要驗證numerical，一樣是透過CNumberValidator來生成驗證語法

```PHP
// because validator's allowEmpty = true
if(jQuery.trim(value)!='') {

    if(!value.match(/^\s*[-+]?[0-9]*\.?[0-9]+([eE][-+]?[0-9]+)?\s*$/)) {
        messages.push("A must be a number.");
    }
}
```

## MathAction extends CAction

這裡要寫加減乘除共同的run()方法，負責

* new MathForm model
* setAttributes()，把`$_POST['MathForm']`資料塞進model
* validate()後端驗證輸入資料
* 透過controller render塞資料給view然後呈現頁面
* 這裡的`$this`是各action實體
* 這render的內容會在math layout的`$content`塞入

```PHP
class MathAction extends CAction
{
    public function run($a = 0, $b = 0)
    {
        $show = '';
        $result = $this->calc($a, $b);
        $model = new MathForm;

        if (isset($_POST['MathForm'])) {

            $model->setAttributes($_POST['MathForm']);

            if ($model->validate()) {

                $a = $model->a;
                $b = $model->b;

                if (empty($model->a)) {
                    $a = 0;
                }
                if (empty($model->b)) {
                    $b = 0;
                }
    
                $result = $this->calc($a, $b);
            }
        }
        $sign = $this->getSign();
        $show = "$a $sign $b = $result <br>";
        return $this->getController()->render('math', ['model' => $model, 'show' => $show, 'sign' => $sign]);
    }

    public function getSign()
    {
        $signs = [
            'add' => '+',
            'div' => '/',
            'mul' => '*',
            'sub' => '-',
        ];
        $actionID = $this->controller->action->id;
        return $signs[$actionID];
    }

    public function calc($a, $b)
    {
        return 0;
    }
}
```

## 加減乘除action extends MathAction

這裡沒幹麻，主要就是各種計算不同所以要覆寫calc()方法，回傳結果

```PHP
class addAction extends MathAction
{
    public function calc($a, $b)
    {
        $result = $a + $b;
        return $result;
    }
}
```

{% asset_img math_flow.PNG math_flow %}

{% asset_img math_merge.PNG math_merge %}
