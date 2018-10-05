---
title: Yii framework - Event
date: 2017-10-15 16:39:53
tags:
  - PHP
  - Yii1.x
---

下列資料供參考

* <http://www.cnblogs.com/JosephLiu/archive/2011/12/12/2285078.html>
* <http://www.php1.cn/article/6396.html>
* <http://yiibook.com/book/yii1.1_application_development_cookbook/chapter-1>

## Event

事件處理
**這裡的$this通常為CComponent的子類實例**

* 定義事件，raiseEvent: 啟動事件，觸發CList中所有handler

```PHP
public function onFuck($event)
{
    echo 'raiseEvent - onFuck <br>';
    $this->raiseEvent('onFuck', $event);
}
```

* 為事件加上某個event handler

```PHP
$handler = function () {
    echo 'Fuck uuu <br>';
};

$this->getEventHandlers('onFuck')->add($handler);

// detach
// $this->detachEventHandler('onFuck', $handler);
```

* 在某處觸發事件

```PHP
// hasEventHandler用來檢查event的CList(_e['onFuck'])是否有被加上event hanlder
if ($this->hasEventHandler('onFuck')) {

    // raiseEvent
    $this->onFuck(new CEvent($this));
}
```

## issue

為事件加上eventHandler有很多寫法

目前還沒瞭解差異在哪

```PHP
$this->onClick = $handler;
```

```PHP
$this->onClick->add($handler);
```

```PHP
$this->attachEventHandler('onClick', $handler);
```

```PHP
$this->getEventHandlers('onClick')->add($handler);
```

```PHP
// 加在handler list指定位置，一般預設是放在最後
$this->getEventHandlers('onClick')->insertAt(0, $handler);
```

實驗結果看起來上述所有用法都是一樣的結果

可以添加多個handler，也都可以detachEventHandler移除特定的handler

加入重複的hanlder一樣會在CList中累積增加，但detach只會移除一個

比較有趣的是最簡化的寫法`$this->onClick = $handler;`乍看之下重複assign好像會覆蓋，但其實不會，與其他效果相同

如果在某handler處理完後不想繼續後面的，可以在傳入handler的事件中設定`$event->handled = true;`

class CEvent 被放在class CComponent的檔案中，這是一個比較奇怪的點
