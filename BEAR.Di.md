status[編集中]

# Overview

BEAR.DiはGoogleのDI (Dependency Injection: 依存性注入) フレームワーク[Guice](http://code.google.com/p/google-guice/)にインスパイアされたライブラリです。実装は[Aura.Di](http://auraphp.github.com/Aura.Di/)をベースに、ライフサイクルの管理や[JSR-330](https://docs.google.com/View?id=djppsvp_31hmdxfpgb&pli=1)([和訳](https://docs.google.com/View?id=djppsvp_31hmdxfpgb&pli=1))の@Injectアノテーションの一部機能を付加し依存性の注入を伴うオブジェクトの生成と管理を行います。インジェクションのバインディングはGuiceと同じように設定ファイルではなくアノテーションを使ったコードで行います。

## Motivation

###何故DIか

PHP5.2+のフレームワークではごく一部のフレームワークがDIを採用するのみでしたが、PHP5.3+系のフレームワークではメジャーなほぼ全てがDI、あるいはDI技術が解決しようとする依存性の問題に対しての取り組みがあります。プログラムに柔軟性と保守性を与え、テストを容易にする技術としてPHPでも普及しつつあると考えます。

### Saturdayでは
Saturdayでは変更可能な初期化メソッドを(=インジェクター)から依存オブジェクトをDependecy Pullすることとサービスロケータを使用することで利用オブジェクトとの生成（装クラス、インジェクター、コンストラクタパラメータ）をコントロールしていました。理解も利用もしやすいものですが、一方、１）コンストラクタのパラメータを継承できない。２）利用コードがコンテナクラスを直接利用している。３）パラメータのレイジーロードが出来ない、４）レジストリ（コンテナ）はグローバルで異なるアプリケーションのサービスが共用できないなどの問題がありました。

### 課題
設定より記述、複雑で全てを一律的に網羅する設計より、簡素で比較的大くをでカバーできるアプローチがPHPらしく良いのではないかと考えます。

## Aura.Di

ベースとするDIの比較検討したのはzf2-DiとAura.Diです。その中でAura.Diはよりコンパクトでベースに使いやすいのではないかと考えました。Aura.Diをそのまま採用していのはアノテーションによるDIととライフサイクルの機能を入れたいためです。PHPでアノテーションによるDi

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

他にはキャッシュ機能。

## Sample

### Aura.Diサンプル
    $di = require '/path/to/Aura.Di/scripts/instance.php';
    // lazy loading
    $di->set('database', function() {
        return new \Example\Package\Database('localhost', 'user', 'passwd');
    });
    // Class Constructor Params
    $di->params['Example\Package\Database'] = array(
        'hostname' => 'localhost',
        'username' => 'user',
        'password' => 'passwd',
    );
    $di->set('database', function() use ($di) {
        return $di->newInstance('Example\Package\Database');
    });
Factory

	class ModelFactory
	{
	    protected $forge;
    
	    public function __construct(Forge $forge)
	    {
	        $this->forge = $forge;
	    }
    
	    public function newInstance($model_name)
	    {
	        $class = 'Example\Package\Model' . ucfirst($model_name);
	        return $this->forge->newInstance($class);
	    }
	}

オブジェクトの生成を行う$forgeも注入されて生成を行います。（全ての依存オブジェクトが注入されます。）

### BEAR.Diサンプル案

#### 対象クラスのアノテーション例

*  @Injectでインジェクトが必要なメソッドやプロパティをアノテートします。

    	class MockDefinitionClass
    	{
	    /**
	     * @PostConstruct
	     */
	    public function onInit()
	    {
	    }

	    /**
	     * @PreDestoroy
	     */
	    public function onEnd()
	    {
	    }

	    /**
	     * @Inject
	     *
	     * @param Db $db
	     */
	    public function setDb(Db $db)
	    {
	        $this->db = $db;
	    }

	    /**
	     * @Inject
	     * @Named("user_db")
	     *
	     * @param Db $db
	     *
	     */
	    public function setUserDb(Db $db)
	    {
	        $this->db = $db;
	    }
    
	    /**
	     * @Inject
	     * @Named("user=admin_user,db=production_db")
	     *
	     */
	    public function setDouble(User $user, Db $db)
	    {
	    }


#### 注入コード

A案 Guiceスタイル。インターフェイスと実装クラスをバインドするモジュールをAbstractModuleクラスを継承して作成します。

    // Linked Binding
    // インターフェイス/抽象クラスを具象クラスにバインドします
    // ex) TransactionLogInterfaceインターフェイスをDatabaseTransactionLog実装クラスにバインドします
    class BillingModule extends AbstractModule {
        protected function configure() {
            $this->bind('TransactionLogInterface')->to('DatabaseTransactionLog');
        }
    }
    // Binding annotation
    // 同じインターフェイスで違うクラスにバインドする時に名前を付けます
    // ex) TransactionLogInterfaceインターフェイスのうち@Named('Db')アノテーションがついているものをDatabaseTransactionLog実装クラスに
    class BillingModule extends AbstractModule {
        protected function configure() {
            $this->bind('TransactionLogInterface')->annotatedWith('Db')->to('DatabaseTransactionLog');
        }
    }
    // Instance Bindings
    // TransactionLogInterfaceインターエフィスに生成したインスタンスをバインドします。
    // ex )TransactionLogInterfaceインターフェイスにDatabaseTransactionLogクラスインスタンスをバインド
    class BillingModule extends AbstractModule {
        protected function configure() {
            $instance = new DatabaseTransactionLog;
            $this->bind('TransactionLogInterface')->annotatedWith('Db')->toInantance($instance);
        }
    }
    
    //@Provides Methods
    // オブジェクトの生成にコードが必要なときに@Injectを使います。このメソッドはモジュール内で定義して@Providesアノテーションをつけ、必要な型のインスタンスを返します。
    class BillingModule extends AbstractModule {
        protected function configure() {
        }
    
        /**
         * @Provide
         */
        protected function provideTransactionLog()
        {
            $transactionLog = $this->di->newInstance('TransactionLog');
            return $transactionLog;
        }
    }
    
    // @Providesメソッドが複雑になってきたら、Provider専用クラスを持つことを検討します。ProviderクラスではProviderインターフェイスを実装しインスタンスを返します。
    
    /**
     * Provider interface
     */
    interface provider
    {
        public function get();
    }
    
    /**
     * DatabaseTransactionLogオブジェクトをprovideするクラス
     */
    class DatabaseTransactionLogProvider implements provider
    {
        /**
         * @Inject
         */
        public function __construct(Di $di, Connection $connection)
        {
            $this->di = $di;
            $this->connection = $connection;
        }
    
        public function get()
        {
            $transactionLog = $this->di->newInstance('DatabaseTransactionLog');
            $transactionLog->setConnection($this->connection);
            return $transactionLog;
        }
    }
    
    // Provider Bindings
    // toProviderメソッドでバインドします。
    class BillingModule extends AbstractModule {
        protected function configure() {
            $this->bind('TransactionLogInterface')->toProvider('DatabaseTransactionLogProvider');
        }
    }
    
    // Just-in-time Bindings
    
    // インターフェイスのアノテーションで特定実装クラスを指定します。
    // ex) CreditCardProcessorインターフェイスをCreditCardクラスにバインドします
    /**
     * @ImplementedBy("PayPalCreditCardProcessor")
     */
    interface CreditCardProcessor {
        public function charge(CreditCard $creditCard);
    }
    
    // インターフェイスのアノテーションでProviderクラスを指定します。
    /**
     *  @ProvidedBy("DatabaseTransactionLogProvider")
     */
    interface TransactionLog {
      public function logConnectException(UnreachableException $e);
      public function logChargeResult(ChargeResult $result);
    }

    // Scopeの指定
    bind('TransactionLog').to('InMemoryTransactionLog').in('Singleton');

B案　Diクラスにbindメソッドを持たせる方法。Moduleクラス不使用。

	$di = new Manager(new Forge(new Config));

	// Linked Binding
	$di->bind('Sample\Db')->to('Test\Db');
	$di->bind('Sample\Db')->in("Smple\Di\Model")->to('Test\Db');

	// Binding annotation
	$di->bind('Sample\Db')->annotatedWith('Test\Db')->to('Test\User\Db');

	// Instance Bindings
	$di->bind('Sample\Db')->annotatedWith('Test\Db')->toInstance('HelloWorld');
	$di->bind('Sample\Db')->annotatedWith('Test\Db')->toInstance(10);

	// Provider Bindings
	$provideDb = function (Di $di){
		return $di->newInstance('Test\Db');
	}
	$di->bind('Sample\Db')->toProvider($provideDb);

### 課題

1. バインディングはA案かB案か。A案 pros:各モジュールは別クラスで独立性がある、Guiceのコードがサンプルになる cons:やや煩雑。Aura.Diからは遠ざかる。　B案 pros:Aura.Diの延長で使える。記述がシンプル cons:定義がグローバル。

1. @Injectでバインド無指定の場合挙動をどうするか - 解決案：クラスヒントを実装クラスかインターフェイスか抽象クラスか判断。実装クラスならそのクラスを生成、その他ならパスから実装クラスを類推してそのクラスを生成する。または、そのタイプヒントをサービスキーとしてコンテナから取り出す。または単に例外をthrowする。

1. コンテナに登録されてるサービス名を直接する方法は不要か - 解決案：例えばサービスを指定するときにSpringがするように@Injectではなく@Resourceにするか？ => 現在のコンパクトな仕様から不必要にアノテーションを増やすような気がする。@Namedのように補助的に@Serviceという名前をつける？またはサポートしない？

### 低コスト化

高速化のためにアプリケーションを通じてコンストラクタのパラメータが変更されないものはインジェクト済みオブジェクトをAPCキャッシュします。この場合はアノテーションやインジェクトコストが０になります。@InjectedCacheアノテーション。



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
