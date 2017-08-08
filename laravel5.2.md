# laravel 学习笔记  version：5.2
个人学习笔记，有错误欢迎指出。

## 1 生命周期：
		 从`index.php`->加载`bootstrap/app.php`后创建服务容器->分发至内核`http内核或者console内核`
		内核配置错误处理、日志、检测环境、中间件、服务提供者->路由`router.php`->`控制器`
## 2 服务提供者：
		register 只用与绑定服务容器
		boot 用于事件监听 composer绑定视图，并可以访问可加重已注册的所有服务（改方法会在所有访问提供者注册完成后才被调用）
## 3 注册服务提供者：
所有服务提供者都是通过配置文件config/app.php中进行注册，该文件包含了一个列出所有服务提供者名字的providers数组，默认情况下，其中列出了所有核心服务提供者，这些服务提供者启动Laravel核心组件，比如邮件、队列、缓存等等。

要注册你自己的服务提供者，只需要将其追加到该数组中即可：	
```
'providers' => [
     // 其它服务提供者
     App\Providers\AppServiceProvider::class,
],
```	
### 3.1 懒加载服务提供者：
如果你的提供者仅仅只是在服务容器中注册绑定，你可以选在延迟加载该绑定直到注册绑定真的需要时再加载，延迟加载这样的一个提供者将会提升应用的性能，因为它不会在每次请求时都从文件系统加载。

想要延迟加载一个提供者，设置defer属性为true并定义一个provides方法，该方法返回该提供者注册的服务容器绑定：

```
<?php

namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider{
    /**
     * 服务提供者加是否延迟加载.
     *
     * @var bool
     */
    protected $defer = true;

    /**
     * 注册服务提供者
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton('Riak\Contracts\Connection', function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * 获取由提供者提供的服务.
     *
     * @return array
     */
    public function provides()
    {
        return ['Riak\Contracts\Connection'];
    }

}
```
Laravel 编译并保存所有延迟服务提供者提供的服务及服务提供者的类名。然后，只有当你尝试解析其中某个服务时Laravel才会加载其服务提供者。



## 4 服务容器($this->app)

几乎所有的服务容器绑定都是在服务提供者中完成。因此本章节的演示例子用到的容器都是在这种上下文环境中，如果一个类没有基于任何接口那么就没有必要将其绑定到容器。容器并不需要被告知如何构建对象，因为它会使用 PHP 的反射服务自动解析出具体的对象。

在一个服务提供者中，可以通过 $this->app 变量访问容器，然后使用 bind 方法注册一个绑定，该方法需要两个参数，第一个参数是我们想要注册的类名或接口名称，第二个参数是返回类的实例的闭包：
```
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app['HttpClient']);
});
```
注意到我们接受容器本身作为解析器的一个参数，然后我们可以使用该容器来解析我们正在构建的对象的子依赖。

### 4.1 绑定一个单例

singleton 方法绑定一个只需要解析一次的类或接口到容器，然后接下来对容器的调用将会返回同一个实例：
```
$this->app->singleton('FooBar', function ($app) {
    return new FooBar($app['SomethingElse']);
});
```
### 4.2 绑定实例

你还可以使用 instance 方法绑定一个已存在的对象实例到容器，随后对容器的调用将总是返回给定的实例：
```
$fooBar = new FooBar(new SomethingElse);

$this->app->instance('FooBar', $fooBar);
```
### 4.3 绑定接口到实现

服务容器的一个非常强大的特性是其绑定接口到实现的能力。我们假设有一个 EventPusher 接口及其 RedisEventPusher 实现，编写完该接口的 RedisEventPusher 实现后，就可以将其注册到服务容器：
```
$this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');
```
这段代码告诉容器当一个类需要 EventPusher 的实现时将会注入 RedisEventPusher，现在我们可以在构造器或者任何其它通过服务容器注入依赖的地方进行 EventPusher 接口的类型提示：
```
use App\Contracts\EventPusher;

/**
 * 创建一个新的类实例
 *
 * @param  EventPusher  $pusher
 * @return void
 */
public function __construct(EventPusher $pusher){
    $this->pusher = $pusher;
}
```
### 4.4 上下文绑定

有时侯我们可能有两个类使用同一个接口，但我们希望在每个类中注入不同实现，例如，当系统接到一个新的订单的时候，我们想要通过PubNub而不是 Pusher 发送一个事件。Laravel 定义了一个简单、平滑的方式来定义这种行为：
```
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
          ->needs('App\Contracts\EventPusher')
          ->give('App\Services\PubNubEventPusher');
```
你甚至还可以传递一个闭包到 give 方法：
```
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
    ->needs('App\Contracts\EventPusher')
    ->give(function () {
        // Resolve dependency...
    });
```    
### 4.5 绑定原始值

有时候你可能有一个获取若干注入类的类，但还需要一个注入的原始值，比如整型数据，你可以轻松使用上下文绑定来注入指定类所需要的任何值：
```
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
    ->needs('$maxOrderCount')
    ->give(10);
```
### 4.6 标签

少数情况下我们需要解析特定分类下的所有绑定，比如，也许你正在构建一个接收多个不同 Report 接口实现的报告聚合器，在注册完 Report 实现之后，可以通过 tag 方法给它们分配一个标签：
```
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```
这些服务被打上标签后，可以通过 tagged 方法来轻松解析它们：
```
$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});
```
### 4.7 服务容器的监听

服务容器在每一次解析对象时都会触发一个事件，可以使用 resolving 方法监听该事件：
```
$this->app->resolving(function ($object, $app) {
    // 容器解析所有类型对象时调用
});

$this->app->resolving(function (FooBar $fooBar, $app) {
    // 容器解析“FooBar”对象时调用
});
```
正如你所看到的，被解析的对象将会传递给回调，从而允许你在对象被传递给消费者之前为其设置额外属性。


### 4.8 关于服务提供者、服务容器、facedes三者的总结

	服务提供者提供服务容器的绑定（register）和事件监听、中间件等注册（boot）
	
	服务容器（$this->app）各种形式绑定*命名空间*到对应的*类*，单例、实例、接口、上下文绑定、原始值、标签。
	
	facedes是服务容器中 '静态'的提供接口，方便快捷,静态调用。

## 5 laravel 数据库
支持数据系统
		Mysql	
		Postgres
		SQLite
		SQL Server
### 5.1 起步
#### 读写分类数据库配置（读大于写）
	'mysql'=>[
		'read'=>[
			'host'=>'192.168.1.1',
		],
		'write'=>[
			'host'=>'192.168.1.2',
		],
		'driver'=>'mysql',
		'database'=>'database',
		'username'=>'root',
		'password'=>'',
		'charset'=>'utf8',
		'collation'=>'utf8_unicode_ci',
		'prefix'=>'',
		
	],
如果想覆盖住数据中的配置，只需要将相应的配置项放到read和write数组中即可。在本例中，读和谐共用mysql的配置，
#### 运行原生SQL查询
##### 参数绑定，避免SQL注入
	DB::select('select * from users where active = ?',[1]);
	DB::select('select * from users where id = :id',['id' => 1]);
	DB::insert('insert into users (id,name) value (?,?)', [1,'Dayle']);
	DB::update('update users set votes = 100 where name = ?',['John']);
	DB::delete('delete from users');
	DB::statement('drop table user');
	
##### 在服务提供者中注册监听查询事件
	<?php
	namespace App\Providers;
	use DB;
	use Illuminate\Support\ServiceProvider;
	
	class AppSericeProvider extends ServiceProvider {
		public function boot(){
			DB::listen(function($query)){
				//$query->sql;
				//$query->bindings;
				//$query->time;
			}
		}
		
	}
	
##### 数据库事务
	想要在一个数据库事务中运行一连串操作，可以使用DB门面的transaction方法，如果事务闭包中抛出异常，事务将会自动回滚，
	如果闭包执行成功，事务将会自动提交。使用transaction方法不需要担心手动回滚或提交：
	DB::transaction(function(){
		DB::table('users')->update(['votes'=> 1]);
		DB::table('posts')->delete();
	})
	
##### 手动提交事务
	DB::beginTransaction();
	DB::rollBack();
	DB::commit();
	
##### 使用多个数据库连接
	使用多个数据库连接的时候，可以使用DB门面的connection方法访问每个连接。参数名来至配置文件
	DB::connection('foo')->select();
	可以通过getPdo方法获取原生的PDO实例
	DB::connection()->getPdo();

### 5.2 数据库查询构建器
#### DB方法集合
 	原生查询 statement用于无返回的情况 e.g.DB::select
	select,insert,update,delete,statement
	
数据库查询构建器 e.g.DB::table()->get() 链式调用

	table 选中表
	get,first,value,lists get所有结果，first第一条，value一条的指定字段，lists 多条指定字段
	where,orWhere 条件 e.g. ('id',1) ('id','=',1) 
	whereBetween,whereNotBetween e.g. whereBetween('votes', [1, 100])
	whereIn,whereNotIn e.g. whereIn('id', [1, 2, 3])
	whereNull,whereNotNull whereNull('updated_at')
	whereExists 创建where existSQL子句，whereExists方法接收一个闭包参数，
	该闭包获取一个查询构建器实例从而允许你定义放置在“exists”子句中的查询：
	whereRaw e.g. whereRaw('orders.user_id = users.id') 等同于where orders.user_id = users.id
	select 可以用作需要的字段e.g. select('a','b as c')
	addSelect 扩展selecct的字段 e.g. addSelect('a','b as c')
	chunk 对得到的结果集进行分块处理
	count,max,min,avg,sum 聚合函数
	row 希望在查询中使用原生表达式，这些表达式将会以字符串的形式注入到查询中，所以要格外小心避免被 SQL注入
	join，leftJoin，rightJoin e.g. leftJoin('posts', 'users.id', '=', 'posts.user_id')
	
	orderBy e.g. orderBy('name', 'desc')
	groupBy e.g. groupBy('account_id')
	having  e.g. having('account_id', '>', 100)
	havingRaw e.g. havingRaw('SUM(price) > 2500')
	skip,take 跳过多少条目，取多少条记录
	insert，update e.g.insert(['email' => 'john@example.com', 'votes' => 0]);
	insertGetId e.g.insert(['email' => 'john@example.com', 'votes' => 0],'id'); 
	如果你想要从其他“序列”获取ID，可以将序列名作为第二个参数传递到insertGetId方法。
	increment,decrement e.g.increment('votes', 5); 5为步长 ；
	可以额外更新->increment('votes', 1, ['name' => 'John']);
	delete，truncate e.g.->delete(); truncate() 清空表
	
#### 高级连接语言

	DB::table('users')->
		join('表名',function($join){
			$join->on('users.id','=','表名.user_id')->orOn(...);
		})->get();
	DB::table('users')
        ->join('表名', function ($join) {
            $join->on('users.id', '=', '表名.user_id')
                 ->where('表名.user_id', '>', 5);
        })
        ->get();
       
#### 联合(union) 
几个结果集的合并 列数相同，值相似即可。用limit order的时候可以带上() union 去除相同行，unionAll不去除

	$first = DB::table('users')
            ->whereNull('first_name');

	$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
         
#### 高级where字句
	DB::table('users')->where('name','=','John')
		->orWhere(function ($query){
			$query->where('vote','>',100)
				  ->where('title','<>','Admin');
		})
		->get();
等同于
	select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
	
whereExists方法允许你编写where existSQL子句.

whereExists方法接收一个闭包参数，该闭包获取一个查询构建器实例从而允许你定义放置在“exists”子句中的查询：
	DB::table('users')->whereExists(function($query){
		$query->select(DB::raw(1))
			  ->from('orders')
			  ->whereRaw('orders.user_id = users.id);
		
	})->get();	
等同于select * from users where exists( select 1 from orders where orders.user_id = users.id)

#### 悲观锁

查询构建器还包含一些方法帮助你在select语句中实现“悲观锁”。可以在查询中使用sharedLock方法从而在运行语句时带一把”共享锁“。共享锁可以避免被选择的行被修改直到事务提交：

	DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
此外你还可以使用lockForUpdate方法。“for update”锁避免选择行被其它共享锁修改或删除：	

	DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

## Eloquent ORM

注意我们并没有告诉 Eloquent 我们的Flight模型使用哪张表。默认规则是模型类名的复数作为与其对应的表名，除非在模型类中明确指定了其它名称
```

    //关联到模型的数据表
    protected $table = 'my_flighs' ;
    //主键
    protected $primaryKey = 'id';

    //表明模型是否应该被打上时间戳,Eloquent 期望created_at和updated_at已经存在于数据表中
   
    public $timestamps = false; 
    //如果你需要自定义时间戳格式
    protected $dateFormat = 'U';
    //默认情况下，所有的 Eloquent 模型使用应用配置中的默认数据库连接，如果你想要为模型指定不同的连接，可以通过 $connection 属性来设置
    protected $connection = 'connection-name';
    //可以被批量赋值的属性.
    protected $fillable = ['name'];
    //不能被批量赋值的属性
    protected $guarded = ['price'];
```
	
    find(1),findOrFail(1)
    save(),update()区别

### 匿名的全局作用域

全局作用域允许我们为给定模型的所有查询添加条件约束。Laravel 自带的软删除功能就使用了全局作用域来从数据库中拉出所有没有被删除的模型。

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;
	use Illuminate\Database\Eloquent\Builder;

	class User extends Model{
	    /**
	     * The "booting" method of the model.
	     *
	     * @return void
	     */
	    protected static function boot()
	    {
		parent::boot();

		static::addGlobalScope('age', function(Builder $builder) {
		    $builder->where('age', '>', 200);
		});
	    }
	}

  #### 移除全局作用域
    User::withoutGlobalScope(AgeScope::class)->get();
    User::withoutGlobalScopes([FirstScope::class, SecondScope::class])->get();

### 本地作用域

本地作用域允许我们定义通用的约束集合以便在应用中复用。例如，你可能经常需要获取最受欢迎的用户，要定义这样的一个作用域，只需简单在对应Eloquent模型方法前加上一个scope前缀，作用域总是返回查询构建器：

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model{
	    public function scopeOfType($query, $type)
	    {
		return $query->where('type', $type);
	    }
	}

	$users = App\User::ofType('admin')->get();

### 事件

Eloquent模型可以触发事件，允许你在模型生命周期中的多个时间点调用如下这些方法：creating, created, updating, updated, saving, saved,deleting, deleted, restoring, restored。事件允许你在一个指定模型类每次保存或更新的时候执行代码。

一个新模型被首次保存的时候，creating和created事件会被触发。如果一个模型已经在数据库中存在并调用save方法，updating/updated事件会被触发，无论是创建还是更新，saving/saved事件都会被调用。

举个例子，我们在服务提供者中定义一个Eloquent事件监听器，在事件监听器中，我们会调用给定模型的isValid方法，如果模型无效会返回false。如果从Eloquent事件监听器中返回false则取消save/update操作：


	<?php

	namespace App\Providers;

	use App\User;
	use Illuminate\Support\ServiceProvider;

	class AppServiceProvider extends ServiceProvider{
	    /**
	     * 启动所有应用服务
	     *
	     * @return void
	     */
	    public function boot()
	    {
		User::creating(function ($user) {
		    if ( ! $user->isValid()) {
			return false;
		    }
		});
	    }

	    /**
	     * 注册服务提供者.
	     *
	     * @return void
	     */
	    public function register()
	    {
		//
	    }
	}


### Eloquent ORM关联关系

一对一，关联模型，一个use模型对应一个phone模型，需要讲方法定义于user的模型。在该方法上定义hasOne方法（定义模式一对一关系）
	<?php
	namespace App;
	use Illuminate\Database\Eloquent\Model;
	class User extends Model{
		//定义一对一模型
		public function phone(){
			return $this->hasOne('App\Phone','user_id','id');
		}
	}


hasOne第一个参数是绑定的模型，第二个和第三个是可选填，第二个是App\phone对应User模型的外键，默认为User模型拼接上id,第三个参数是User的主键

	$phone = User::find(1)->phone;

同理根据phone模型 找到User模型
在phone里声明
	public function user(){
	    return $this->belongsTo('App\User', 'user_id', 'id');
	}
第二个参数，第三个参数同上


##### 利用中间模型实现一对多
	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Country extends Model{
	    /**
	     * 获取指定国家的所有文章
	     */
	    public function posts()
	    {
		return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
	    }
	}
第一个参数，最终访问的模型，第二个为中间模型，第三个，中间模型的外键，第四个，最终模型的外键

#### 多对多的多态关联
#### 动态属性（渴求式加载） and 链式条件约束Eloquent查询
#### 查询已存在的关联关系

	// 获取所有至少有一条评论的文章...
	$posts = App\Post::has('comments')->get();

#### 渴求式查询
##### 带条件约束的渴求式加载
##### 懒惰渴求式加载
##### 多对多关联(未读)

#### Eloquent 集合

#### Eloquent 访问器 修改器

##### 定义访问器

在模型中建立一个getFooAttribute，其中Foo为字段，驼峰命名规则。
在模型访问这个属性的时候会指定调用这个方法重写值
	<?php
	namespace App;
	use Illuminate\Database\Eloquent\Model;

	class User extends Model{
		public function getFooAttribute($value){
			return ucfirst($value);
		}
	}

访问如下
	$user = App\User::find(1);
	$foo = $user->foo;

#### 定义修改器
在模型中建立修改器，定义SetFooAttribute方法。
当掉on个set的时候，会对应修改
	<?php
	namespace App;
	use Illuminate\Database\Eloquent\Model;

	class User extends Model{
		public function setFooAttribute($value){
			$this->attributes['foo'] = strtolower($value);
		}
	}
访问如下
	$user =App\User::find(1);
	$user->foo = "Sally";
#### 属性转换

模型中的$casts属性为属性字段转换到通用数据类型提供了便利方法 。$casts属性是数组格式，其键是要被转换的属性名称，其值时你想要转换的类型。目前支持的转换类型包括：integer, real, float, double, string, boolean, object，array，collection，date和datetime。
	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model{
	    /**
	     * 应该被转化为原生类型的属性
	     *
	     * @var array
	     */
	    protected $casts = [
		'is_admin' => 'boolean',
	    ];
	}
array类型转换在处理被存储为序列化 JSON 格式的字段时特别有用，例如，如果数据库有一个 TEXT 字段类型包含了序列化JSON，添加array类型转换到该属性将会在 Eloquent 模型中访问其值时自动将其反序列化为 PHP 数组：

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model{
	    /**
	     * 应该被转化为原生类型的属性
	     *
	     * @var array
	     */
	    protected $casts = [
		'options' => 'array',
	    ];
	}

##### 追加值到 JSON

有时候，需要添加数据库中没有的字段到数组中，要实现这个功能，首先要为这个值定义一个访问器：

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model{
	    /**
	     * 为用户获取管理员标识
	     *
	     * @return bool
	     */
	    public function getIsAdminAttribute()
	    {
		return $this->attributes['admin'] == 'yes';
	    }
	}
定义好访问器后，添加字段名到该模型的 appends 属性：
	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model{
	    /**
	     * 追加到模型数组表单的访问器
	     *
	     * @var array
	     */
	    protected $appends = ['is_admin'];
	}
appends数组里会去找对应的字段，刚好在服务器里做逻辑判断。追加出新字段。


## 服务
