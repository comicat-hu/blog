---
title: Yii framework - Form
date: 2017-10-17 11:45:24
tags:
  - PHP
  - Yii1.x
---
使用Yii來建立一個登入表單

model建立在`protected\models`下，config需要設定import

動作流程大概是這樣:

<!--more-->

* 進入`site/login`
* 在actionLogin中create form data model，catch POST，render login page

```PHP
$model = new LoginForm;

if (isset($_POST['LoginForm'])) {

    // collection users data, use CModel setAttributes()
    $model->attributes = $_POST['LoginForm'];

    if ($model->validate()) {
        // default to '/'
        $this->redirect(Yii::app()->user->returnUrl);
    }
}

$this->render('login', ['model' => $model]);
```

* 在LoginForm model中處理資料驗證，覆寫rules()

```PHP
public function rules()
{
    return [
        // username and password are required
        // use CRequiredValidator
        ['username, password', 'required'],
        // rememberMe needs to be a boolean
        // use CBooleanValidator
        ['rememberMe', 'boolean'],
        // password needs to be authenticated
        // use method authenticate()
        ['password', 'authenticate'],
    ];
}
```

* 如果驗證都無誤，redirect到user->returnUrl，預設是'\'，可以setReturnUrl()修改

LoginForm model中有用到authenticate()，這個方法是要自己寫的，

用來驗證password的Validator

```PHP
public function authenticate($attribute, $params)
{
    $this->_identity = new UserIdentity($this->username, $this->password);

    if (!$this->_identity->authenticate()) {
      $this->addError('password', 'Invalid Username or Password!');
    }
}
```

這邊用一個自訂的class UserIdentity extends CUserIdentity，

繼承自CUserIdentity的authenticate()是需要自行覆寫的，不然預設就是丟一個例外

CUserIdentity中也有預定義一些errorCode的常數可以用

```PHP
class UserIdentity extends CUserIdentity
{
    public function authenticate()
    {

        if (strlen($this->password) < 5) {
            $this->errorCode = self::ERROR_PASSWORD_INVALID;
        } else {
            $this->errorCode = self::ERROR_NONE;
        }

        return !$this->errorCode;
    }
}
```

在view中可以使用yii helper`CHtml::beginForm()`或是widget`beginWidget('CActiveForm')`來建立表單，兩種略有不同

程式就直接參考[form.view](http://www.yiichina.com/doc/guide/1.1/form.view)

可以看到建出來的input html是用陣列的形式填充輸入資料

```HTML
<input name="LoginForm[username]" id="LoginForm_username" type="text" />
```

所以在上面action中我們才能直接使用`$_POST['LoginForm']`取得整個form data並且填進data model中。
