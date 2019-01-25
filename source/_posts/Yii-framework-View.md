---
title: Yii framework - View
date: 2017-10-13 15:57:51
categories:
  - 資訊技術
  - PHP
tags:
  - PHP
  - Yii1.x
---
View單純呈現頁面結果，呼叫`CController::render()`傳入view id就可以渲染出該頁面。

render()預設會去存取`protected/views/ControllerID`資料夾。

通常網頁中會有些固定不變的元素(HTML主結構)，這時會拆分出來放在`protected/views/layouts`下，

而render()時預設會先輸出layouts下的main.php(如果有的話)，這時你要render的內容會被預設放在`$content`這個變數內，使用`<?php echo $content; ?>`輸出它

<!--more-->

```PHP
// CController::render()

public function render($view,$data=null,$return=false)
{
    if($this->beforeRender($view))
    {
        $output=$this->renderPartial($view,$data,true);
        if(($layoutFile=$this->getLayoutFile($this->layout))!==false)
            $output=$this->renderFile($layoutFile,array('content'=>$output),true);

        $this->afterRender($view,$output);

        $output=$this->processOutput($output);

        if($return)
            return $output;
        else
            echo $output;
    }
}
```

`CWebApplication::layout`可以更改預設輸出的layout檔，預設是 = 'main'

使用renderPartial()則不會預設輸出layout

在外部action render，要先取得controller物件

`$this->getController()->render();`

或

`$this->controller->render();`

`<?php echo CHtml::encode($this->pageTitle); ?>`可以輸出網站的名稱，預設是config中設定的name加上`-`加上actionID加上controllerID(像是MyWeb - Hello Site)，但是index不會顯示actionID

render()可以帶入要傳入view變數，

直接在render()的第二個參數帶入陣列，`name`就是要在view中存取的變數名

`render('index', [name => $name]);`

in view: `<?php echo $name ?>`
