こんにちは、と言う
==================

この節では、アプリケーションに「こんにちは」という新しいページを作成する方法を説明します。
この目的を達するために、[アクション](structure-controllers.md#creating-actions) と [ビュー](structure-views.md) を作成します。

* アプリケーションは、このページへのリクエストをそのアクションに送付します。
* 次にそのアクションが「こんにちは」という言葉をエンドユーザに示すビューを表示します。

このチュートリアルを通じて、三つのことを学びます。

1. リクエストに応える [アクション](structure-controllers.md) を作成する方法
2. レスポンスのコンテントを作成する [ビュー](structure-views.md) を作成する方法
3. アプリケーションがリクエストを [アクション](structure-controllers.md#creating-actions) に送付する仕組み


アクションを作成する <span id="creating-action"></span>
--------------------

「こんにちは」のタスクのために、リクエストから `message` パラメータを読んで、そのメッセージをユーザに表示して返す `say` [アクション](structure-controllers.md#creating-actions) を作ります。
リクエストが `message` パラメータを提供しなかった場合は、アクションはデフォルト値として "こんにちは" というメッセージを表示するものとします。

> Info|情報: [アクション](structure-controllers.md#creating-actions) は、エンドユーザが直接に参照して実行できるオブジェクトです。
  アクションは [コントローラ](structure-controllers.md) によってグループ化されます。
  アクションの実行結果が、エンドユーザが受け取るレスポンスです。

アクションは [コントローラ](structure-controllers.md) の中で宣言されなければなりません。
話を簡単にするために、`say` アクションを既存の `SiteController` の中で宣言しましょう。
このコントローラは `controllers/SiteController.php` というクラスファイルの中で定義されています。
次のようにして、新しいアクションが始まります。

```php
<?php

namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    // ... 既存のコード ...

    public function actionSay($message = 'こんにちは')
    {
        return $this->render('say', ['message' => $message]);
    }
}
```

上記のコードでは、`SiteController` クラスの中で、`say` アクションが `actionSay` という名前のメソッドとして定義されています。
Yii はコントローラクラスの中で、アクションのメソッドとアクションでないメソッドを区別するために、`action` という接頭辞を使います。
`action` という接頭辞の後に続く名前がアクション ID にマップされます。

アクションを命名するについては、Yii がアクション ID をどのように取り扱うかを知っていなければなりません。
アクション ID は常に小文字で参照されます。
アクション ID が複数の単語を必要とするときは、単語がダッシュ (-) で連結されます (例えば、`create-comment`)。
アクションメソッドの名前は、アクション ID からダッシュを全て削除し、各単語の先頭の文字を大文字にした結果に `action` という接頭辞を付けたものになります。
例えば、アクション ID `create-comment` に対応するアクションメソッド名は `actionCreateComment` となります。

私たちの例では、アクションメソッドは `$message` というパラメータを取り、そのデフォルト値は `"こんにちは"` です
(PHP で関数やメソッドの引数にデフォルト値を設定するのと全く同じ方法です)。
アプリケーションがリクエストを受け取って、当該リクエストの処理を `say` アクションが担当すべきであると決定した場合は、リクエストの中に見つかった同じ名前のパラメータの値をこの `$message` パラメータに代入します。
言い換えれば、もしリクエストの中に `"さようなら"` という値の `message` パラメータが入っていれば、アクションの `$message` 変数にその値が割り当てられます。

アクションメソッドの中では、[[yii\web\Controller::render()|render()]] が呼ばれて `say` と言う名前の [ビュー](structure-views.md) ファイルがレンダリングされます。
`message` パラメータも同時にビューに渡され、そこで使用されます。
レンダリング結果はアクションメソッドによって返されます。
返された結果はアプリケーションによって受け取られ、ブラウザ上でエンドユーザに (完全な HTML ページの一部として) 表示されます。


ビューを作成する <span id="creating-view"></span>
----------------

[ビュー](structure-views.md) は、レスポンスのコンテントを生成するために書かれるスクリプトです。
「こんにちは」のタスクのためには、アクションメソッドから受け取った `message` パラメータを出力する `say` ビューを作成します。

```php
<?php
use yii\helpers\Html;
?>
<?= Html::encode($message) ?>
```

`say` ビューは `views/site/say.php` というファイルに保存しなければなりません。
アクションの中で [[yii\web\Controller::render()|render()]] メソッドが呼ばれるとき、`render()` メソッドは `views/ControllerID/ViewName.php` という名前の PHP ファイルを探します。

上記のコードで `message` パラメータが出力される前に  [[yii\helpers\Html::encode()|HTML-エンコード]] されていることに注意してください。
パラメータはエンドユーザから来るものであり、悪意のある JavaScript コードを埋め込まれて [クロスサイトスクリプティング (XSS) 攻撃](http://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0) に使われうるものですから、脆弱性を防止するためにこうすることが必要です。

当然ながら、`say` ビューにはもっと多くのコンテントを入れても構いません。
コンテントには、HTML タグ、平文テキスト、さらには PHP 文を含めることが出来ます。
実際、`say` ビューは [[yii\web\Controller::render()|render()]] メソッドによって実行される PHP スクリプトであるに過ぎません。
ビュースクリプトによって出力されたコンテントはレスポンス結果としてアプリケーションに返されます。
そしてアプリケーションがこの結果をエンドユーザに対して出力します。


試してみる <span id="trying-it-out"></span>
----------

アクションとビューを作成したら、下記の URL で新しいページにアクセスすることが出来ます。

```
http://hostname/index.php?r=site/say&message=Hello+World
```

![Hello World](images/start-hello-world.png)

この URL は、結果として、"Hello World" を表示するページになります。
このページはアプリケーションの他のページと同じヘッダとフッタを共有しています。

URL から `message` パラメータを省略すると、"こんにちは" を表示するページを見ることになるでしょう。
これは、`message` が `actionSay()` メソッドにパラメータとして渡されるものであり、それが省略された場合には、デフォルト値である `"こんにちは"` が代りに使われるからです。

> Info|情報: 新しいページは他のページと同じヘッダとフッタを共有していますが、それは [[yii\web\Controller::render()|render()]] メソッドが `say` ビューの結果をいわゆる [レイアウト](structure-views.md#layouts) に自動的に埋め込むからです。
レイアウトは、この場合、`views/layouts/main.php` にあります。

上記の URL の `r` パラメータについては、さらに説明が必要でしょう。
これは [ルート](runtime-routing.md)、すなわち、アクションを指し示すアプリケーションを通じて一意な ID を表します。
ルートの書式は `ControllerID/ActionID` です。
アプリケーションはリクエストを受け取ると、このパラメータ `r` をチェックし、`ControllerID` の部分を使って、このリクエストを処理するためにどのコントローラクラスのインスタンスを作成すべきかを決定します。
そして、コントローラは `ActionID` の部分を使って、実際の仕事をするためにどのアクションを呼び出すべきかを決定します。
この例で言えば、`site/say` というルートは、`SiteController` コントローラクラスと `say` アクションとして解決されます。
結果として、`SiteController::actionSay()` メソッドがリクエストを処理するために呼び出されます。


> Info|情報: アクションと同じく、コントローラもまたアプリケーションの中で一意に定義される ID を持ちます。
  コントローラ ID も、アクション ID と同じ命名規則を使います。
  コントローラクラスの名前は、コントローラ ID からダッシュを削除し、各単語の最初の文字を大文字にし、結果として出来る文字列に `Controller` という接尾辞を追加したものとなります。
  例えば、`post-comment` というコントローラ ID に対応するコントローラクラスの名前は `PostCommentController` です。


まとめ <span id="summary"></span>
------

この節では、MVC デザインパターンのうちのコントローラとビューの部分に触れました。
特定のリクエストを処理するためのアクションをコントローラの一部として作成しました。
また、レスポンスのコンテントを作成するためのビューも作成しました。
この単純な例においては、使用される唯一のデータが `message` パラメータであったため、モデルは関係していません。

また、Yii におけるルートについても学びました。ルートはユーザのリクエストとコントローラのアクションとの橋渡しとして働くものです。

次の節では、モデルを作成する方法を学びます。そして、HTML フォームを含むページを追加します。