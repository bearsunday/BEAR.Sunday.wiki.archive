**※このページで検討したBEAR.DiはRay.Diとして実装されました。Google Codeのwiki [RayPHP](http://code.google.com/p/rayphp/) をご覧下さい。**


***


preview:2
note: パッケージ名はまだ未定です。仮にBEAR.Diとしています。

# Overview

BEAR.DiはGoogleのDI (Dependency Injection: 依存性注入) フレームワーク[Guice](http://code.google.com/p/google-guice/)にインスパイアされたアノテーションベースのDependency Injectionフレームワークです。実装は[Aura.Di](http://auraphp.github.com/Aura.Di/)をベースに、ライフサイクルの管理や[JSR-330](https://docs.google.com/View?id=djppsvp_31hmdxfpgb&pli=1)([和訳](https://docs.google.com/View?id=djppsvp_31hmdxfpgb&pli=1))の@Injectアノテーションの一部機能を付加し依存性の注入を伴うオブジェクトの生成と管理を行います。

## Motivation

###何故DIか

PHP5.2+のフレームワークではごく一部のフレームワークがDIを採用するのみでしたが、PHP5.3+系のフレームワークではメジャーなほぼ全てがDI、あるいはDI技術が解決しようとする依存性の問題に対しての取り組みがあります。プログラムに柔軟性と保守性を与え、テストを容易にする技術としてPHPでも普及しつつあると考えます。

### Saturdayでは
Saturdayでは変更可能な初期化メソッドを(=インジェクター)から依存オブジェクトをDependecy Pullすることとサービスロケータを使用することで利用オブジェクトとの生成（装クラス、インジェクター、コンストラクタパラメータ）をコントロールしていました。理解も利用もしやすいものですが、一方、以下の問題がありました。

 # コンストラクタのパラメータを継承できない。
 # 利用コードがコンテナクラスを直接利用している。
 # パラメータのレイジーロードが出来ない
 # レジストリ（コンテナ）はグローバル。異なるアプリケーションでサービスが共用できない。

## Aura.Di
### クリーン
BEAR.DiはAura.DiのConfig, Forge(ファクトリー）,Containerを利用しています。Aura.Diはクリーンでコンテナ自身の依存性も全て外部から渡されています。（内部でnewされている固定クラスがありません）。つまり全てのコンポーネントが入れ替え可能です。

InjectorはGofのデコレーターパターンで生成されます。

	$injector = new	Injector(new Container(new Forge(new Config(new Annotation))), new BillingModule);

これは「「「「アノテーションをスキャンするAnnotationクラス」を使ったConfigクラス」を使ってオブジェクトを生成するForgeクラス」を利用するコンテナクラス」を使ったInjectorクラス」の生成という意味です。依存する（必要な）オブジェクトは全て外部から渡されていてためにこのような記述になります。簡素な記述ではありませんが、基本的にbootstrap時に一度だけです。

## Function
###Aura.Diの機能
コンストラクタインジェクション、セッターインジェクション、継承が有効なコンストラクタのネームドパラメータ指定などです。ネームドパラメータは引き数を順序でなく変数名で指定します。デフォルトの値を持ち、上書きするパラメータだけを指定することができます。

###BEAR.Diでの付加機能
アノテーション

* @PostConstruct コンストラクタの後に実行される初期化メソッド。一つのみ存在可。(JSR-250)
* @PreDestory デストラクタの前に実行される終了メソッド。一つのみ存在可。(JSR-250)
* @Prototype("singleton") prototype, singleton, session、application などをライフサイクルを指定
* @Inject 依存性を注入しなくてはならないポイントを特定
* @Named 特定の実装を指定するバインディングアノテーション
* @Provide 依存オブジェクトプロバイダー


## Getting Started ##

DIを用いてオブジェクトを生成する時に、コンストラクタは（そのオブジェクトが）依存するオブジェクトを引数としてとります。オブジェクトを生成するには、また別のオブジェクトが必要です。それぞれの依存オブジェクトにはまた違うそれぞれの依存オブジェクトがあります。つまり、オブジェクトを作成する時にはオブジェクト間の関係性（オブジェクトグラフ）を明らかにする必要があります。

手作業でオブジェクト・グラフを構成するのは、過酷な労働、不具合を起こしがちな傾向、そしてテスト実施の難しさの原因となります。これに対してGuiceはオブジェクト・グラフを〔自動的に〕構成してくれます。しかしGuiceがオブジェクト・グラフを構成するために、まず必要な設定をしなくてはなりません。

このようなオブジェクトグラフを手動で行うのは大変な作業です。またできあがったものをテストするのも大変です。BEAR.Diではこの作業を自動でしてくれますが、オブジェクトグラフの作成を行うために設定をする必要があります。

説明のため、RealBillingServiceクラスからはじめましょう。このクラスはそのコンストラクタにCreditCardProcessorとTransactionLogという2つのインターフェースに適合するクラスが必要です。RealBillingServiceのコンストラクタが必要なオブジェクトがある事を@Injectアノテーションを加える事で表します。


	class RealBillingService implements BillingService
	{
	    /**
	     * @var CreditCardProcessor
	     */
	    private $processor;

	    /**
	     * @var TransactionLog
	     */
	    private $transactionLog;

	    /**
	     * @Inject
	     */
	    public function __construct(CreditCardProcessor $processor, TransactionLog $transactionLog)
		{
	        $this->processor = $processor;
	        $this->transactionLog = $transactionLog;
	    }

	    public function chargeOrder(PizzaOrder $order, CreditCard $creditCard)
		{
		...
		}
}

RealBillingServiceを作成するため、PaypalCreditCardProcessorとDatabaseTransactionLogクラスが必要です。インターフェイスとその実装をバインドします。モジュールは自然言語のようなFluentインターフェースで記述できます。モジュールはインターフェイスと実装クラスのバインドのまとまったものです。

	public class BillingModule extends AbstractModule {

	    protected void configure() {

	        /*
	         * TransactionLogインターフェイスにDatabaseTransactionLogクラスをバインドしています。
	         */
	        $this->bind('TransactionLog')->to('DatabaseTransactionLog');

	        /*
	         * 同様にCreditCardProcessorインターフェイスをPaypalCreditCardProcessorクラスにバインドしています。
	         */
	        $this->bind('CreditCardProcessor')->to('PaypalCreditCardProcessor');
	    }
	}

モジュールを作成してからインジェクターを作成します。インジェクターはオブジェクトグラフの構築マシンのようなものです。モジュールを使って、必要なものが整ったオブジェクトをきちんと組み立てます。ここではインジェクターがRealBillingServiceクラスのインスタンスを作成します。

通常モジュールはアプリケーション１つについて１つだけでしょう。通常はbootstrap部分で記述します。getInstanceメソッドでモジュールに従ったオブジェクトグラフを持つオブジェクトが取得できます。

	$injector = new	Injector(new Container(new Forge(new Config(new Annotation))), new BillingModule);
	$billingService = $injector->getInstance('BillingService');

原典：Google Guice “Getting Started”（2011/06/01 9:45取得）
参考訳：http://d.hatena.ne.jp/m12i/20110603

### 前回課題

1. バインディングはA案かB案か。A案 pros:各モジュールは別クラスで独立性がある、Guiceのコードがサンプルになる cons:やや煩雑。Aura.Diからは遠ざかる。　B案 pros:Aura.Diの延長で使える。記述がシンプル cons:定義がグローバル。

Guiceと同じスタイル。

1. @Injectでバインド無指定の場合挙動をどうするか - 解決案：クラスヒントを実装クラスかインターフェイスか抽象クラスか判断。実装クラスならそのクラスを生成、その他ならパスから実装クラスを類推してそのクラスを生成する。または、そのタイプヒントをサービスキーとしてコンテナから取り出す。または単に例外をthrowする。

JITバインディング。

1. コンテナに登録されてるサービス名を直接する方法は不要か - 解決案：例えばサービスを指定するときにSpringがするように@Injectではなく@Resourceにするか？ => 現在のコンパクトな仕様から不必要にアノテーションを増やすような気がする。@Namedのように補助的に@Serviceという名前をつける？またはサポートしない？

DIコンテナからサービスを取り出すような事はあまりなさそう。
コンテナから取得ではなくて、@Injectで注入

### 低コスト化

>高速化のためにアプリケーションを通じてコンストラクタのパラメータが変更されないものはインジェクト済みオブジェクトをAPCキャッシュします。この場合はアノテーションやインジェクトコストが０になります。@InjectedCacheアノテーション。

キャッシュの詳細は他のクラス実装の時に再検討。
キャッシュが最も効果的な「注入が終わった後」のオブジェクトキャッシュが可能であればdocやreflectionのキャッシュは不要になる。


## 参考リンク

* Aura.Di http://auraphp.github.com/Aura.Di/
* JSR-250 http://en.wikipedia.org/wiki/JSR_250
* JSR-330 https://docs.google.com/View?id=djppsvp_31hmdxfpgb
* JSR-250/330サポートのDIフレームワーク Ding http://marcelog.github.com/Ding/

### DIアンチパターン
#### [Richard Miller When Dependency Injection goes Wrong](http://miller.limethinking.co.uk/2011/05/19/when-dependency-injection-goes-wrong/)

* PoorHinting - タイプヒントを指定しよう。それもクラス名では無くインターフェイスをタイプヒントに利用しよう、さもなくばサブクラスしか注入できなくなる。
* Making the Container a Dependency - 実コンテナクラスを直接利用するのではなくて、そのコンテナクラスも注入してもらおうようにしよう。
* Tying dependency choice to class definition - インターフェイスインジェクションやアノテーションによるDIの設定は変更が容易でなく良くない。
* Springの@Requieredはバッドプラクティス、コンストラクタインジェクションにしよう＆プライベートフィールドにはやめよう http://diegacho.blogspot.com/2011/09/some-java-dependency-injection-bad.html
