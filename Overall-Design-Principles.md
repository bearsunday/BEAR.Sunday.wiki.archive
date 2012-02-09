## Concept

### BEAR Key Concept
* Intuitiveness 直感性
* Drivability　操縦性
* Separation of concern 関心の分離
* Zen ミニマリズム

### Sunday New Concept
* Everything is a resource.　全てはリソース
* Agility and quality.　アジリティとクオリティ
* REST meets CQRS. RESTとCQRSの融合
* Object Framework (DI + AOP)

### Function / Goal
* MVC friendly resource oriented architecture (extends HMVC) / HMVCを拡張したリソース指向
* Multiple application resource / マルチアプリリソースの使用　
* UA-Sniffing optimization / マルチUAの最適化
* push / pull 双方向のリソース
* CQRS  / (クエリーとコマンドでのデータソースの分離。時間ではなくメソッドトリガーのCache機構)
* orbited / websocket (model/viewの変化が即view/modelに反映されるovserbed MVC)
* clean modern PHP code (SRP,SoC,LoD,OCP,Tell don't ask...) / test coverage
* HATEOAS (Hypermedia as the Engine of Application State) optional support
* pjax
* framework as CMS

### Policy / Principle
* No Overengineering / オーバーエンジニアリングの否定
* Easy learning / 低学習コスト
* Component connection over component itself / コンポーネントの接続に注目
* Independency over Interdependency / 相互依存よりも独立することを好む
* Prefer standard technology / 中立的な標準技術を好む。
* Prefer simpleness over complexity / 簡素を好む。
* Prefer DSL / DSLを好む。
* Pluggable, flexibility / プラガブル、柔軟性
* Minimaze lock-in / ロックインの最小化
* Not only for just users, but also real developers. / ユーザーのためだけでなく開発者のためのもの。
* Self contained library / セルフコンテインドライブラリ
* Be professional, nothing personal. / 好き嫌いで語らない。

##Technology

### General

#### be clean code

* 粗結合、階層構造。SRP重視
* 継承より合成を好む。
* 遅延結合
* Fatになるのを嫌う。
* POPO
* スタティックコールの抑制
* 共通ベースクラスを持たない。
* 共通ベース例外を用意しない。SPL例外や継承した例外を好む。
* メソッドの引数は少なく。基本は4つ以下。
* タイプヒンティングは実クラスではなく、インターフェイスか抽象クラスを用いる。
* setter/getterの抑制（値ではなくてデータ構造を隠蔽する）
* 構造を持つスカラーデータのセットはDTOにまとめる((int)x, (int)y => class Position{public $x; public $y})
* DTOとオブジェクトの明確な区別
* DTOはpublicプロパティを持ちLoD違反を避ける。
* プロパティ固定名にフレームワークの機能を持たせない。
* メソッド固定名にフレームワークの機能を持たせない。
* グローバルdefineを持たない。
* include_pathの依存を持たない
* アプリケーション、フレームワーク共にメソッド内でのインスタンス生成
* フレームワークが持つグローバルな設定ファイル、レジストリはない
* 依存は全てモジュールでバインドする
* 実行モードを持たない。エントリーポイントでdev/prodを切り替える (クラス内でif ($debug)がない。必要ならクラスを入れ替える）

### as a framework

* 共通ファクトリー(DI)を持つ。
* newの不使用。シングルトンなど、オブジェクトの生成の機能を各クラスが提供しない。コンテナが提供する。
* ユニファイドコンストラクタ以外のコンストラクタも可能に。
* オブジェクトのコンストラクトをアプリケーションキャッシュ可能に。
* PEARパッケージ・ディストリビューション
* 機能を実装によって実現することを躊躇し、組み合わせによって実現することを好む。
* 十分な機能を持つライブラリがあれば依存PHPのバージョンが低くても利用を躊躇しない。(=PEAR)。エレーレベルの問題はエラーハンドラーで解決する。
* PHP5.2用BEARとの混在（リソースの独立性）
* 設定はファイルよりコードを好む。

### DI

* Guiceインスパイア。[Aura.Di](http://auraphp.github.com/Aura.Di/)をfork
* [JSR-330](https://docs.google.com/View?id=djppsvp_31hmdxfpgb&pli=1)(Guice)スタイルの@InjectアノテーションによるDI
* プロパティ/セッター/コンストラクトインジェクション
* サービスロケータとサービスプロバイダ
* タイプヒンティングインジェクション / ネームインジェクション
* namedパラメーターによるコンストラクト引数 (Aura.Di)
* 遅延引数 (Aura.Di)
* クラス継承に伴うコンストラクタ引数、インスタンス管理定義の継承
* アノテーションによるインスタンス管理定義を加える(@PostConstruct, @PreDestroy等)
* アノテーションによるサービスプロバイダの指定
* コンテナもインジェクトされるクリーンDI
* XMLを使わないバインディング(Guice / Aura.Di)

### アノテーション
* リソース以外にも原則どのクラス、メソッドにもアノテーション可能に
* アノテーション情報はすべてキャッシュ
* アノテーションの動的定義
* アプリケーションアノテーション
* Doctrine\Common\Annotationsを使用

### リソース(ROA)

* 従来のリソース(M)だけでなくビュー(V)、ページ(C)もリソースにし、view, pageのURIスキーマを与える
* ROA４つの原則 URI, 統一インターフェイス, ステートレス, (リンク)、これを満たしてリソースにする。VもCも可能。
* リソースパラメータはuri, values, optionsをpublicプロパティに持つDTOに
* Pullリソース（制御の反転）。不足情報は制御を反転して尋ねられる (例ユーザーIDが分からないユーザーページviewが、ページ、リソースの順に尋ねる）
* リソースパラメータプロバイダ
* パラメータはリクエストメソッドを持つ
* リソースが被リクエスト処理を行わない。リクエストアダプターに依頼する。
* リンクはHTMLのAタグアナロジーを用いてtargetを指定できる。"self"だと入れ替わり、文字列を指定するとそのリンクされたリソースが付加される（Satudayのリンクと同じ）
* ページリソースとroリソースの構造のリクエストを受けてレスポンスを返すという基本的に同じ。
* ページリソースとroリソースの違いは境界（バウンダリー）、ページリソースはHTTPを境界にもち、roリソースは純粋PHPの世界。
* 多言語、例えばJavaのroリソースをphpのページリソースが利用　(Thrift)
* ローカルサービスリソース、ローカルホストリソース、リモートリソース、を区別しない
* 全てのリソースは非Web(CLI等）から利用可能
* リンク重視
* リソースリフレクションとそのメソッド(info)
* メッセージキューリソース
* リソースサーバー(PHP5.4)
* リソースルーター
* URIテンプレート
* Hypertext-driven REST API

* リソースはcode, header, bodyの他にdocument(or representation?)=表現のプロパティを保持する(?=viewリソースとコンフリクト)

### ページリソース
* onInitの代わりにonGet - Side Effect Free
* フォームはHTTP-METHOD-OVERIDEを使いGET/POST/PUT/DELETEと全てのHTTPメソッドを使用
* onActionの代わりにonGet/onPost/onPutに。マルチフォームではonPostEntryなど
* リンクメソッドを持つ
* リンクメソッドは他のページへのリンクの手続きと、viewリソースへのリンクの二種類がある。
* ページリソースはroリソースの集合体。HTML表現はviewリソースにリンクすることで行う。リンク先を変更することで他表現に。つまりどう表現するかをページ自身は持たずクライアントの操作（リンク）で決定する。
* リソースのセットオプションに"polling"を持つ。これはビューで一定間隔でajaxでリソースをpullする仕組み
* リソースのセットオプションに"realtime"を持つ。これはorbited/またはwebsocketでソケット接続されたリソースがリアルタイム更新される仕組み。速報などのbroadcastにも使える。モデルの変化が即ビューに通知されるSmallTalk的真MVC
* pageリソースは基本はroリソースの集合体。値のget/setでなく、接続を指向する。「ユーザービューに対してユーザーリソースと友達リソースを接続する」と言う風に。ユーザーの値はroリソースの責務、表現はviewリソースの責務。pageリソースは値には無関心。
* FormはPEAR::HTML_QuickForm2またはzf2\FormまたはAura.Formデフォルト
* 値の操作ではなくリソースの関係性を記述する

### Apiリソース
* pullリソース強化
* パラメータプロバイダ
* notify
* URIスキーマ再検討
* updateはidentifyをoptionsで
* CQRS
* メソッドエイリアス
* データグリッド
* リソースのJSレンダーオプション(pjax)

### viewリソース
* リモートview
* create / updateメソッドのサポートも
* メタファイルの活用
* メタファイルにリソースリンクを記述
* Smarty3デフォルト

### アノテーション
* アプリケーション作成のアノテーション
* アノテーションクラス
* Doctrine/Commons/Annotations使用

### CQRS (Command Query Responsibility Segregation)
* readと非readの関心の相違によるレポジトリの分離
* 時間でなくメソッドによるキャッシュ
* 分散化（クラウド対応）
* リソースに実装

### 高速化
* アノテーションとクラス生成メタ情報はすべてAPCキャッシュ
* DTO以外は基本的にミュータブルオブジェクトでステートレスリクエスト
* 原則的にコンストラクトはアプリケーションを通じて共通にし依存オブジェクトを内包したものをserizlize/unserializeしてnewを使わない
* CQRSで原則クエリーはキャッシュを読むのみ

## 規約・規則
* コーディング規約はPEAR/Zend/Solar/Auraを踏襲。ただしprivate/protectedでアンダースコアはprefixしない。
* ファイル配置や命名規則はAura/zf2準拠

## 採用ライブラリ
* Doctrine.Common (Annotatin, Cache)
* Doctrine.DBAL / ( zf2.db, Aura.sql0) 
* Smarty3 ? Twig ? Haanga ? Mustache ?
* Aura.* (Aura.Router, Aura.Web)
* Symfony.Component (Loader)
* Log - Guzzle interface(Monolog, Zend_Log)
* Guzzle
* Cache - Guzzle interface(Doctrine.Common.Cache, Zend_Cache)
* zf2/*, zf1/*
* PEAR
* PHPUnit
** Orbited / WebSocket

## PHP5.3+ / 5.4+
* アプリケーションnamespaceでローカルサービス以外のリソースのリクエスト
* リソースtrait
* "buit-in" HATEOASサーバー 

## 開発
* TDD / GitHub / Jenkins
* Resource Driven Development (RDD)
* リソースのデータグリッド表示
* リソースURIルーターによるモックリソース
* aceエディター更なる活用
* 編集可能gridリソース
* 専用phpcs ruleset (BEAR/ruleset.xml)

## Challenge / Dream / Possibility / Consideration
* プログラマレスプロジェクトの可能性
* オープンリソースリポジトリ (Resource API)
* エラーのソーシャライズ
* リソースのリモートAPI公開
* スキーマ・オープンマーケット
* CMSのプラグイン利用
* MVC JSフレームワーク Backbone.jsやangular.jsとの連携

###エラー
* エラーのダウンロード
* エラー記録のDB化
* エラーの共有
* エラーの追跡可能性向上