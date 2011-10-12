# Hello World Prototype

## Overview

Hello World Prototypeとは全体構成の理解に役立てるための最小構成のアプリです。

※案として示しているだけで実装はまだ存在しません。
### Table of contets
webアプリの最小限の構成として以下のものがあります。

* htdocs/ bootstrapスクリプト
* ページリソース
* アプリケーションリソース
* ビューリソース


## htdocs/ bootstrapスクリプト

※　

https://github.com/koriym/BEAR.Sunday/blob/master/docs/sampleApps/HelloWorld/htdocs/helloworld.php

最も重要な部分はこの部分です。

    $ro = $resource->get($params->setUri('page://self/helloWorld'))->link('html://self/helloWorld')->getResourceObject();
    $ro->output('http');

リソースクライアントがページリソースをGETリクエストし、その結果をビューリソースにリンクし結果をリソースオブジェクトとして取得しています。次に取得したリソースオブジェクトをHTTP出力しています。

（このhtdocs/下のスクリプトは、フレームワークが行うべきディスパッチャーやbootstrapの処理を含んでいます。ユーザーが毎回記述する内容ではありません。）

ビューが独立してリソースになりページに出力の責任がなくなりました。出力は呼び出されたページが決定するのではなく、呼び出すリソースクライアントが決定します。ページリソースの主な仕事は与えられたwebコンテキスト（$_GETや$_COOKIE, $_SESSION）に従ってアプリケーションリソースをリクエストし、自らのリソースbodyに保持することです。

そのリソースをhtml://スキームを持つビューリソースにリンクするとHTTPのcode/header/bodyを持ったリソースオブジェクト（変数）になります。それを例にあげてるようにoutput('http')で出力するとHTMLページになります。

ページリソースリクエストは１つに制限されません。複数のページリソースリクエストを行い、比較合成して一つの出力として出力することも可能です。

## ページリソース

### 1.最小限のHelloWorld
https://github.com/koriym/BEAR.Sunday/blob/master/docs/sampleApps/HelloWorld/Page/HelloWorld.php

引数もなくリソースも利用しないページリソースがメッセージをsetするだけのアプリケーションです。

ページにはリクエストに対する内部処理を記述します。ここのHelloWorldページリソースはページリソースには引数が渡されず、HTTPのGETリクエストでonGet()というメソッドが呼ばれます。setで自らのbodyに値をセットしています。Saturdayと違い出力は記述しません。

### 2.アプリケーションリソースを利用するHelloWorld

https://github.com/koriym/BEAR.Sunday/blob/master/docs/sampleApps/HelloWorld/Page/Resource.php

コンストラクタでリソースをリクエストするためのリソースリクエストクライアントクラスと、そのリクエストパラメーターオブジェクトを取得するためのparamsプロバイダーを受け取っています。

コンストラクタを@Injectとアノテートすることで、必要なオブジェクトが渡されます。Sundayではクラスの内側でnewでクラスを生成したり、外部のファクトリークラスを直接指定してインスタンス生成を依頼したりする事はあまりないでしょう。※　内部で生成する必要ができた時はインジェクターやファクトリー(Forge)を@Injectでインジェクトして利用します。

    /**
     * Constructor
     *
     * @param Resource $resource
     * @param Provider $params
     *
     * @Inject
     * @Named("params=params")
     */
    public function __construct(Resource $resource, Provider $params)
    {
        $this->resource = $resource;
        $this->params = $params;
    }

タイプヒントはインターフェイスまたは抽象クラスを使います。実装でなくインターフェイスに依存するようにします。

Providerインターフェイスは特別なインターフェイスで、引数なしのget()メソッド一つだけのメソッドを持つマイクロファクトリーです。利用は簡単です。get()するとインスタンスが入手できます。メソッド内で複数のインスタンスが必要なクラスはこのように取得します。（Providerインターフェイスを持ったマイクロファクトリーがどのようにインスタンスを生成してるかに利用メソッドは興味がありません。）

    public function onGet()
    {
        $params = $this->params->get()->setUri('app://self/HelloWorld');
        $this->resource->get($params)->set('message');
    }

メソッド名がHTTPリクエストに応じたGET/PUT/POST/DELETEと変更になります。１つのメソッドに複数のモードがあるときは（例えば一覧表示GETとアイテムGET、ログインPOSTとリマインダーPOST）を持つときは、メソッド名＋アクション　onPostLogin, onPostReminder のようになります。）

onGet内ではアプリケーションリソースのURIを指定したリクエストパラメターでGETリクエストを行い、結果をmessageという名前でセットしているように記述します。

※実際にはすぐにこのリクエストは行われず、onGet終了後にまとめて実行されて遅延結合されます。ビューリソースが必要としないリクエスト(Put/Post/Delete）であれば画面出力終了後、あるいはPHP実行終了後に実行する事も可能です。（"足跡"のupdateでユーザーの画面表示を待たせる必要はありません）、あるいはその結果を誰も必要としてなければ、結局リクエストは行われません。（ビューで使わないのだからDBヘのクエリーが無駄になりますよね）これは今のSaturdayでも実装されてる遅延読み込みの機能ですが、CQRSパターンで拡張され更なる柔軟性とも持つ予定（野望）です。

## アプリケーションリソース

    class HelloWorld extends ResourceObject
    {
        public function onGet()
        {
            return 'Hello World !';
        }
    }

アプリケーションリソース（旧Roリソース）はcrudの代わりにGET/POST/PUT/DELETEとメソッド名が変更になります。ページと同じように必要なサービスオブジェクトは@Injectアノテーションで取得します。

詳細未定ですが、検討しているのは以下のものです。

 * アノテーションによるデフォルト指定 (@Cache, @Template, @Aspect　... ）
 * URIテンプレート
 * URIリライト
 * 動的ポイントカットによるアスペクト指向
 * ネームドパラメーター（引数を複数もち、変数名=値で指定）

## ビューリソース

ビューもリソースとして扱います。例えば以下のテンプレート、

	<html>
	<head><title>Hello World</title></head>
	<body>
	{$greeting}
	</body>
	</html>

以下のようなfunctionと同じようなものとして考えられないでしょうか。

    public function onGet($message)
    {
        return "
	<html>
	<head><title>Hello World</title></head>
	<body>
	{$greeting}
	</body>
	</html>";
    }

つまりこのビューリソースをリクエストは以下のようなURIで表せます。
    
    html://self/helloWold?greeting=こんにちは


ビューのテストも容易になるでしょう。リソースにすることで、キャッシュ、他アプリ、リモートの利用、リンク等の機能が使える事も期待できます。

## Conclusion

疑似リクエストURIでHelloWorldをもう一度振り返ります。HelloWorldリソースをHTML出力するHelloWorldページです。

このHelloWorlページはWebブラウザから以下のリクエストを受けます。

    GET /HelloWorld.php HTTP/1.0

これを受けてページリソースはアプリケーションリソースに対して以下のリクエストをします。

    GET app://self/HelloWorld

このリクエストを受けたApp\Resource\HelloWorldクラスのonGetメソッドは結果を返し、受け取ったページはmessageという名前でsetしビューリソースにリンクします。

最終的にビューリソースに対して以下のリクエストが行われます。

    GET html://self/HelloWorld?greeting=こんにちは

こういうbodyをもったリソースオブジェクトが得られるはずです。

    <html>
      <head><title>Hello World</title></head>
      <body>
        こんにちは
      </body>
    </html>

リクエストに問題がなかったので、ビューリソースは適切なHTTPコード(200)やheader(Content-Type: text/html)がセットしてあるRoを返してくれるでしょう。

最後にHTTP出力します。

    ->output('http');

このサンプルアプリでは、HTTPリクエスト、ページ、アプリケーションリソース、ビューリスース、とコンポーネント間全てのリクエストがURIで表現できました。各リソースがアドレス可能性、統一インターフェイス、ステートレス、等のリソース指向アーキテクチャの特徴を持っていてそれぞれがリソースとして機能しています。
