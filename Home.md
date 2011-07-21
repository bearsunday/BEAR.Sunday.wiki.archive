## BEAR.Sunday デザインメモ

##原則

## 基本
* なるべくself-contained / なるべくライブラリ指向
* スタティックコール、無原則なsetter/getter、over engineering、にsay no
* 機能数よりも柔軟性の保持

##ソフトウエア設計
* Satudayと違って共通ベースクラス(BEAR_Base)を持たない
* ユニファイドコンストラクタ以外（ただ１つのarray引数）のコンストラクタも可能に
* DTOと振る舞いを持つ一般オブジェクトの明確な区別
* DTOはpublicプロパティを持つ (デメテルの法則を避ける意味ももつ)
* ハリウッド原則
* デメテルの法則（Law of Demetre (LoD)）または最小知識の原則](http://ja.wikipedia.org/wiki/%E3%83%87%E3%83%A1%E3%83%86%E3%83%AB%E3%81%AE%E6%B3%95%E5%89%87)

##基本アーキテクチャ

## DI
* [JSR-330](https://docs.google.com/View?id=djppsvp_31hmdxfpgb&pli=1)スタイル　(Google Guice)
* Aura.Diをfork
* アノテーションによるインスタンス管理定義を加える(@PostConstruct, @PreDestroy等)
* namedパラメーターによるコンストラクト引数 (Aura.Di)
* クラス継承に伴うコンストラクタ引数、インスタンス管理定義の継承
* @Injectアノテーションによるインジェクト
* サービスオブジェクトプロバイダ

## アノテーション
* リソース以外にも原則どのクラス、メソッドにもアノテーション可能に
* アノテーション情報はすべてキャッシュ
* 動的なアノテーションも
* アプリケーションアノテーション

## リソース(ROA)

* リソース(M)、ビュー(V)、ページ(C)すべてをリソースにし、ro, view, pageのURIスキーマを与える
* リソースはcode, header, bodyの他にdocument(or representation?)=表現のプロパティを保持する
* リソースパラメータはuri, values, optionsをpublicプロパティに持つオブジェクトに
* パラメータはリクエストメソッドを持つ
* リソースが被リクエスト処理を行わない。リクエストアダプターに以来する。
* リンクはHTMLのAタグアナロジーを用いてtargetを指定できる。"self"だと入れ替わり、文字列を指定するとそのリンクされたリソースが付加される（Satudayのリンクと同じ）
* ページリソースとroリソースの構造のリクエストを受けてレスポンスを返すという基本的に同じ。
* ページリソースとroリソースの違いは境界（バウンダリー）、ページリソースはHTTPを境界にもち、roリソースは純粋PHPの世界。
* 多言語、例えばJavaのroリソースをphpのページリソースが利用　(by Thrift)
* ローカルサービスリソース、ローカルホストリソース、リモートリソース、を区別しない
* リソースサーバー(daemon)

### ページリソース
* onInitの代わりにonRead
* フォームはHTTP-METHOD-OVERIDEを使いGET/POST/PUT/DELETEと全てのHTTPメソッドを使用
* onActionの代わりにonGetやonPostに。マルチフォームではonPostEntryなど
* リンクメソッドを持つ
* リンクメソッドは他のページへのリンクの手続きと、viewリソースへのリンクの二種類がある。
* ページリソースはroリソースの集合体。HTML表現はviewリソースにリンクすることで行う。リンク先を変更することで他表現に。つまりどう表現するかをページ自身は持たずクライアントの操作（リンク）で決定する。
* リソースのセットオプションに"polling"を持つ。これはビューで一定間隔でajaxでリソースをpullする仕組み
* リソースのセットオプションに"realtime"を持つ。これはorbited/またはwebsocketでソケット接続されたリソースがリアルタイム更新される仕組み。速報などのbroadcastにも使える。

### roリソース
* notify
* パラメータプロバイダ = pull強化
* URIスキーマ再検討
* updateはidentifyをoptionsで
* CQRS
* メソッドエイリアス
* リソースルーター
* URIテンプレート

### viewリソース
* リモートview
* create / updateメソッドのサポート
* メタファイルの活用
* メタファイルにリソースリンクを記述

## アノテーション
* コールクラス使用
* 原則どのクラスでも利用可能に
* アプリケーション作成のアノテーション

## CQRS (Command Query Responsibility Segregation)
* readと非readの関心の相違によるレポジトリの分離
* 時間でなくメソッドによるキャッシュ
* 分散化（クラウド対応）

## 高速化
* アノテーションとクラス生成メタ情報はすべてAPCキャッシュ
* DTO以外は基本的にミュータブルオブジェクトでステートレスリクエスト
* 原則的にコンストラクトはアプリケーションを通じて共通にし依存オブジェクトを内包したものをserizlize/unserializeしてnewを使わない
* CQRSで原則クエリーはキャッシュを読みこむのみ


### 規約・規則
* コーディング規約はPEAR/Zendを踏襲。ただしprivate/protectedでアンダースコアはprefixしない。
* ファイル配置や命名規則はzf2準拠

## 採用ライブラリ
* Aura.Di, Aura.*
* Doctorine DBAL
* Smarty3
* Orbited / WebSocket
* zf2/*

## PHP5.3+ / 5.4+
* アプリケーションnamespaceでローカルサービス以外のリソースのリクエスト
* リソースtrait

## 開発
* TDD / Jenkins
* GitHub
* エラーのダウンロード
* エラー記録のDB化
* エラーの共有
* エラーの追跡可能性
* リソースのデータグリッド表示
* リソースURIルーターによるモックリソース
* aceエディター更なる活用