status[編集中]

# Overview

BEAR.DiはAura.Diをベースに、ライフサイクルの管理やJSR-330の@Injectアノテーションの一部機能を付加したものです。依存性の注入を伴うオブジェクトの生成と管理を行います。

## Motivation

PHP5.2+のフレームワークではごく一部のフレームワークがDIを採用するのみでしたが、PHP5.3+系のフレームワークではメジャーなほぼ全てがDI、あるいはDI技術が解決しようとする依存性の問題に対しての取り組みがあります。BEAR.Diは利用コスト低減のため、Guice/Springが採用してるJSR-330のアノテーションによるインジェクトが利用できるように既存のDIライブラリを拡張します。

### Saturdayでは
Saturdayでは変更可能な初期化メソッドを(=インジェクター)から依存オブジェクトをDependecy Pullすることとサービスロケータを使用することで利用オブジェクトとの生成（装クラス、インジェクター、コンストラクタパラメータ）をコントロールしていました。理解も利用もしやすいものですが、一方、コンストラクタのパラメータを継承できない。利用コードがコンテナクラスを直接利用している。パラメータのレイジーロードが出来ない。などの問題がありました。

## Aura.Di

ベースとするDIの比較検討したのはzf2-DiとAura.Diです。その中でAura.Diはよりコンパクトでベースに使いやすいのではないかと考えました。既存のものをそのまま採用していのは@Injectアノテーションとライフサイクルの機能を入れたいためです。またJavaではGuiceやSpring@InjectはGuice(JSR-330)が持つ使い方に近いものを想定しています。

## Function
###Aura.Diの機能
コンストラクタインジェクション、セッターインジェクション、ネームドパラメータ、継承が有効なコンストラクタパラメータ指定などです。

###BEAR.Diでの付加機能
アノテーション
@PostConstruct コンストラクタの後に実行される初期化メソッド。一つのみ存在可。(JSR-250)
@PreDestory デストラクタの前に実行される終了メソッド。一つのみ存在可。(JSR-250)
@Prototype("singleton") prototype, singleton, session、application などをライフサイクルを指定
@Inject 依存性を注入しなくてはならないポイントを特定
@Named パラメータインジェクト指定
@Provide インジェクトプロバイダー

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
 @Injectでアノテートされたメソッドがsetterインジェクションになります。
 @Namedアノテーションはタイプヒントだけで決定されないサービスに対して指定をします。(Guice)

#### 注入コード

	$di = new Manager(new Forge(new Config));
	$di->newInsntance('Mock');

	// Type hint injection
	$di->bindHint('Sample\Model\Db')->to('Test\Db');
	$di->bindHint('Sample\Model\Db')->in("Smple\Di\Model")->to('Test\Db');

	// Type hint injection with @Name
	$di->bindHint('Sample\Model\Db')->named('Test\Db')->to('Test\User\Db');

	// Inject with instance
	$di->bindHint('Sample\Model\Db')->named('Test\Db')->toInstance('HelloWorld');
	$di->bindHint('Sample\Model\Db')->named('Test\Db')->toInstance(10);

	// Inject with service provider
	$provide = function ($di){
		return new Test\Db();
	}
	// Named param injection
	$di->bindParamName('db')->in("Smple\Di\Model", "SetDb")->to('Test\Db');

### 課題

1. @Injectでバインド無指定の場合挙動をどうするか - 解決案：クラスヒントを実装クラスかインターフェイスか抽象クラスか判断。実装クラスならそのクラスを生成、その他ならパスから実装クラスを類推してそのクラスを生成する。または、そのタイプヒントをサービスキーとしてコンテナから取り出す。[疑問]
1. アノテーションでインジェクトするサービス名やクラス名を指定する方法はなくても良いか - 解決案：例えばサービスを指定するときにSpringがするように@Injectではなく@Resourceにするのは現在のコンパクトな仕様から不必要にアノテーションを増やすような気がする。@Namedのように補助的に@Serviceという名前をつけるか、またはサポートしないか。


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
