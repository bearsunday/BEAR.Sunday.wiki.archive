## BEAR.Sunday デザインメモ

##Theme
* Everything is a resource.
* +agility +quality with ROA

## Function / Goal
* マルチアプリリソースの使用　(namespace)
* マルチUAの最適化
* push / pull 双方向のリソース、制御の反転。
* pjaxサポート
* CQRSによるデータソースの分離。時間ではなくメソッドトリガーのCache機構
* orbited / websocket サポート。model/viewの変化が即view/modelに反映される真MVC
* framework as CMS
* modern clean code

##Key Concept
* Intuitiveness 直感性
* Drivability　操縦性
* Separation of concern 関心の分離
* Minimalism 禅

## 基本・方針
* ×オーバーエンジニアリング
* コンポーネントの接続に注目
* Clean PHP Code (SRP,SoC,LoD,OCP,Tell don't ask,継承より合成,ボーイスカウトルール, information hiding...)
* 最新最良より定番で広く使われてる標準技術を好む。しかし邪魔はしない。
* 簡素を好む
* DSLを好む 
* プラッガブルコンポーネント 
* 低学習コスト
* 高い応用性
* ロングタームサポート
* (self-contained library)
* (not only on the rail, but also off the beaten track)

## 実践
* スタティックコールの抑制
* setter/getterの抑制（値ではなくてデータ構造を隠蔽する）
* 共通ベースクラスを持たない
* 共通ベース例外を用意しない
* 汎用(SPL)例外使用(zf2/aura)
* 共通ファクトリー(DI)を持つ
* POPO
* ユニファイドコンストラクタ以外のコンストラクタも可能に
* DTOとオブジェクトの明確な区別
* DTOはpublicプロパティを持つ (LoD違反を避ける意味ももつ)
* PHP5.2用BEARとの混在（リソースの独立性）
* コンストラクトをアプリケーションキャッシュ可能に
* PEAR/zf/zf2使用
* PEARパッケージ

##基本アーキテクチャ

## DI

* [JSR-330](https://docs.google.com/View?id=djppsvp_31hmdxfpgb&pli=1)スタイルの@InjectアノテーションによるDI
* プロパティ/セッター/コンストラクト/Pullインジェクション
* サービスロケータとサービスプロバイダ
* Aura.Diをfork
* namedパラメーターによるコンストラクト引数 (Aura.Di)
* 遅延引数 (Aura.Di)
* クラス継承に伴うコンストラクタ引数、インスタンス管理定義の継承
* アノテーションによるインスタンス管理定義を加える(@PostConstruct, @PreDestroy等)
* アノテーションによるサービスプロバイダの指定
* 簡単DI

## アノテーション
* リソース以外にも原則どのクラス、メソッドにもアノテーション可能に
* アノテーション情報はすべてキャッシュ
* 動的なアノテーションも
* アプリケーションアノテーション

## リソース(ROA)

* リソース(M)、ビュー(V)、ページ(C)すべてをリソースにし、ro, view, pageのURIスキーマを与える
* リソースはcode, header, bodyの他にdocument(or representation?)=表現のプロパティを保持する
* リソースパラメータはuri, values, optionsをpublicプロパティに持つDTOに
* Pullリソース（制御の反転）
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

### ページリソース
* onInitの代わりにonRead
* フォームはHTTP-METHOD-OVERIDEを使いGET/POST/PUT/DELETEと全てのHTTPメソッドを使用
* onActionの代わりにonGet/onPost/onPutに。マルチフォームではonPostEntryなど
* リンクメソッドを持つ
* リンクメソッドは他のページへのリンクの手続きと、viewリソースへのリンクの二種類がある。
* ページリソースはroリソースの集合体。HTML表現はviewリソースにリンクすることで行う。リンク先を変更することで他表現に。つまりどう表現するかをページ自身は持たずクライアントの操作（リンク）で決定する。
* リソースのセットオプションに"polling"を持つ。これはビューで一定間隔でajaxでリソースをpullする仕組み
* リソースのセットオプションに"realtime"を持つ。これはorbited/またはwebsocketでソケット接続されたリソースがリアルタイム更新される仕組み。速報などのbroadcastにも使える。モデルの変化が即ビューに通知されるSmallTalk的真MVC
* pageリソースは基本はroリソースの集合体。値のget/setでなく、接続を好む。「ユーザービューに対してユーザーリソースと友達リソースを接続する」と言う風に。ユーザーの値はroリソースの責務、表現はviewリソースの責務。pageリソースは値には無関心。
* FormはPEAR::HTML_QuickForm2またはzf2\FormまたはAura.Formデフォルト
* 値の操作ではなくリソースの関係性を記述する

### roリソース
* pullリソース強化
* パラメータプロバイダ
* notify
* URIスキーマ再検討
* updateはidentifyをoptionsで
* CQRS
* メソッドエイリアス
* グリッド
* リソースのJSレンダーオプション(pjax)

### viewリソース
* リモートview
* create / updateメソッドのサポートも
* メタファイルの活用
* メタファイルにリソースリンクを記述
* Smarty3デフォルト

## アノテーション
* コールクラス使用
* 原則どのクラスでも利用可能に
* アプリケーション作成のアノテーション

## CQRS (Command Query Responsibility Segregation)
* readと非readの関心の相違によるレポジトリの分離
* 時間でなくメソッドによるキャッシュ
* 分散化（クラウド対応）
* リソースに実装

## 高速化
* アノテーションとクラス生成メタ情報はすべてAPCキャッシュ
* DTO以外は基本的にミュータブルオブジェクトでステートレスリクエスト
* 原則的にコンストラクトはアプリケーションを通じて共通にし依存オブジェクトを内包したものをserizlize/unserializeしてnewを使わない
* CQRSで原則クエリーはキャッシュを読むのみ

### 規約・規則
* コーディング規約はPEAR/Zendを踏襲。ただしprivate/protectedでアンダースコアはprefixしない。
* ファイル配置や命名規則はzf2準拠

## 採用ライブラリ
* PEAR
* Aura.Di, Aura.*
* Doctorine DBAL
* Smarty3
* Orbited / WebSocket
* zf2/*
* zf1/*

## PHP5.3+ / 5.4+
* アプリケーションnamespaceでローカルサービス以外のリソースのリクエスト
* リソースtrait

##エラー
* エラーのダウンロード
* エラー記録のDB化
* エラーの共有
* エラーの追跡可能性向上

## 開発
* TDD / GitHub / Jenkins
* Resource Driven Development (RDD)
* リソースのデータグリッド表示
* リソースURIルーターによるモックリソース
* aceエディター更なる活用
* 編集可能gridリソース
* 専用phpcs ruleset (BEAR/ruleset.xml)

## Challenge / Dream
* プログラマレスプロジェクトの可能性
* オープンリソースリポジトリ (Resource API)
* エラーのソーシャライズ