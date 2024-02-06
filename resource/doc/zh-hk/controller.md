＃ 控制器

建立控制器文件 `app/controller/FooController.php`。

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

當訪問 `http://127.0.0.1:8787/foo` 時，頁面返回 `hello index`。

當訪問 `http://127.0.0.1:8787/foo/hello` 時，頁面返回 `hello webman`。

當然你可以通過路由配置來更改路由規則，參見[路由](route.md)。

> **提示**
> 如果出現404無法訪問，請打開`config/app.php`，將`controller_suffix`設置為`Controller`，並重啟。

## 控制器後綴
webman從1.3版本開始，支援在`config/app.php`設置控制器後綴，如果`config/app.php`裡`controller_suffix`設置為空`''`，則控制器類似如下

`app\controller\Foo.php`。

```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

強烈建議將控制器後綴設置為`Controller`，這樣能能避免控制器與模型類名衝突，同時增加安全性。

## 說明
 - 框架會自動向控制器傳遞`support\Request` 對象，通過它可以獲取用戶輸入數據(get post header cookie等數據)，參見[請求](request.md)
 - 控制器裡可以返回數字、字符串或者`support\Response` 對象，但是不能返回其它類型的數據。
 - `support\Response` 對象可以通過`response()` `json()` `xml()` `jsonp()` `redirect()`等助手函數創建。

## 控制器生命週期

當`config/app.php`裡`controller_reuse`為`false`時，每個請求都會初始化一次對應的控制器實例，請求結束後控制器實例銷毀，這與傳統框架運行機制相同。

當`config/app.php`裡`controller_reuse`為`true`時，所有請求將復用控制器實例，也就是控制器實例一旦創建便常駐內存，所有請求復用。

> **注意**
> 關閉控制器復用需要webman>=1.4.0，也即是說在1.4.0之前控制器默認是所有請求復用的，無法更改。

> **注意**
> 開啟控制器復用時，請求不應該更改控制器的任何屬性，因為這些更改將影響後續請求，例如

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    protected $model;
    
    public function update(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->update();
        return response('ok');
    }
    
    public function delete(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->delete();
        return response('ok');
    }
    
    protected function getModel($id)
    {
        // 该方法將在第一次請求 update?id=1 後會保留下 model
        // 如果再次請求 delete?id=2 時，會刪除 1 的數據
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **提示**
> 在控制器`__construct()`構造函數中return數據不會有任何效果，例如

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // 構造函數中return數據沒有任何效果，瀏覽器不會收到此響應
        return response('hello'); 
    }
}
```

## 控制器不復用與復用區別
區別如下

#### 不復用控制器
每個請求都會重新new一個新的控制器實例，請求結束後釋放該實例，並回收內存。不復用控制器和傳統框架一樣，符合大部分開發者習慣。由於控制器反復的創建銷毀，所以性能會比復用控制器稍差(helloworld壓測性能差10%左右，帶業務可以基本忽略)

#### 復用控制器
復用的話一個進程只new一次控制器，請求結束後不釋放這個控制器實例，當前進程的後續請求會復用這個實例。復用控制器性能更好，但是不符合大部分開發者習慣。

#### 以下情況不能使用控制器復用

當請求會改變控制器的屬性時，不能開啟控制器復用，因為這些屬性的更改會影響後續請求。

有些開發者喜歡在控制器構造函數`__construct()`里針對每個請求做一些初始化，這時候就不能復用控制器，因為當前進程構造函數只會調用一次，並不是每個請求都會調用。