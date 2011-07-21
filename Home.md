## BEAR.Sunday デザインメモ

##原則

## 基本
* なるべくself-contained / なるべくライブラリ指向
* 機能数よりも柔軟性の保持
* スタティックコール、無原則なsetter/getterは控える
* off the beaten truck driveability (not only on the rail.)
* Be Clean (SRP,SoC,LoD,OCP,Tell don't ask,継承より合成,ボーイスカウトルール, information hiding...)
* no over engineering

## 実践
* 共通ベースクラスを持たない
* ユニファイドコンストラクタ以外（ただ１つのarray引数）のコンストラクタも可能に
* DTOと振る舞いを持つ一般オブジェクトの明確な区別
* DTOはpublicプロパティを持つ (LoD違反を避ける意味ももつ)
* DSLを好む 
* 標準と簡素を好む

##基本アーキテクチャ

## DI

* [JSR-330](https://docs.google.com/View?id=djppsvp_31hmdxfpgb&pli=1)スタイルの@InjectアノテーションによるDI
* サービスロケータとサービスプロバイダ
* Aura.Diをfork
* アノテーションによるインスタンス管理定義を加える(@PostConstruct, @PreDestroy等)
* namedパラメーターによるコンストラクト引数 (Aura.Di)
* クラス継承に伴うコンストラクタ引数、インスタンス管理定義の継承

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
* リソースのセットオプションに"realtime"を持つ。これはorbited/またはwebsocketでソケット接続されたリソースがリアルタイム更新される仕組み。速報などのbroadcastにも使える。モデルの変化が即ビューに通知されるSmallTalk的真MVC
* pageリソースは基本はroリソースの集合体。値のget/setでなく、接続を好む。「ユーザービューに対してユーザーリソースと友達リソースを接続する」と言う風に。ユーザーの値はroリソースの責務、表現はviewリソースの責務。pageリソースは値には無関心。

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
* create / updateメソッドのサポートも
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
* CQRSで原則クエリーはキャッシュを読むのみ


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