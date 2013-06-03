# BEAR.Resource

**※このページで検討したBEAR.Resourceは実装されました。**

## 概要

BEAR.Resourceはモデルやコントローラー、ビュー等対象をリソースとして捉えそれぞれの接続にROAを適用したコンポーネント接続のためのサービスレーヤーフレームワークです。

クライアントはGET/POST/PUT/DELETE等制限されたインターフェイスを使ってのみリソースにアクセスできます。リクエストを受けたリソースはリクエストに応じて結果を返す。結果はどのような場合であってもcode/header/bodyの３つのプロパティを持ちます。（例外やエラーを含む）

## サンプルコード
クライアント側

    $user = $resource->newInstance('app://self/User');
    $resource->get->object($user)->withQuery(['id' => $id])->eager->request();
    
    $resource->get->object($user)
      ->withQuery(['id' => $id])
      ->withAnnotate(array(new Log, new Validate))
      ->page(1)->per(10)
      ->eager->request();
リソース側
    class User implementes ResourceObject
　　 {
     
    /**
     * @Template
     * @Cache("30")
     */
    public function onGet($id)
    {
     ....

メソッド呼び出しは名前付き引数（ネームドパラメーター）で行います。

## リクエストを返す

メソッドチェーンの最後は必ずrequest()メソッドで終わります。それまでのアトリビュートでeagerならリクエスト結果が、lazy（デフォルト）ならリクエストの方法を保持しているリクエストオブジェクトが返ります。


## スキーマ

 * リソースはそれぞれURIで表現されます。スキーマがデータ構造を表す。MVCのMの部分、最も基本となるリソースはAppリソースで内部APIでapp://で表されます。ページはpage://でHTTPリソースはhttp://です。
 * Sundayで作成した他のアプリケーションのページやappリソースも使用することができます。
 * デフォルトで用意されているスキーマも含め全てのスキーマのプログラムとスキーマをユーザーがバインドすることができます。

## リンク

リソースはリンクを持つことができます。リソースとリソースのリンク知識をクライアントが持つのではなく、リソースが持ちます。つまりどのモデルとモデルの接続をコントローラーが知るのではなく、モデルが知ってるということです。(HTMLでのAタグです）

## Appリソース
MVCのモデルです。ビジネスルールを記述するサービスレイヤーまたはトランザクションスクリプトとして機能します。メソッドインターセプターをアノテーションで表し、バリデーション、テンプレート適用、キャッシュなどは基本的にリソース側の機能として持ちます（クライアントで持ちません）

## ページリソース

MVCでいうとコントローラーの部分です。ページリソースの主な役割はAppリソースのリクエストです、そのリソースリクエスト方法をページリソースが保持します。通常のWeb MVCと違って、ページリソースはビューに対する知識はありません。ページリソースはAppリソースの結果、あるいはAppリソースのリクエストを自身に保持します。そのページリソースが保持しているリソースをViewがオブザーブしてエバリュエーションを行います。ページをHTMLとしてレンダリングするのか、RSSとして出力するのかは、ページ自身は決定しません。ぺージリソースをどのようなプレゼンテーションとして扱うかは、オブザーブしているビューが決定します。

## ビューリソース

ビューリソースがページリソースを参照してページが保持するリソースリクエストを行い値をテンプレートにセットします。GUI MVCのobserverパターンに近いイメージです。