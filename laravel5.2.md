# laravel 学习笔记  version：5.2
个人学习笔记，有错误欢迎指出。
##1 生命周期：
		 从`index.php`->加载`bootstrap/app.php`后创建服务容器->分发至内核`http内核或者console内核`
		内核配置错误处理、日志、检测环境、中间件、服务提供者->路由`router.php`->`控制器`
##2 服务提供者：
		register 只用与绑定服务容器
		boot 用于事件监听 composer绑定视图，并可以访问可加重已注册的所有服务（改方法会在所有访问提供者注册完成后才被调用）
##3 注册服务提供者：
所有服务提供者都是通过配置文件config/app.php中进行注册，该文件包含了一个列出所有服务提供者名字的providers数组，默认情况下，其中列出了所有核心服务提供者，这些服务提供者启动Laravel核心组件，比如邮件、队列、缓存等等。

要注册你自己的服务提供者，只需要将其追加到该数组中即可：	
```
'providers' => [
     // 其它服务提供者
     App\Providers\AppServiceProvider::class,
],
```	
###3.1 懒加载服务提供者：
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



##4 服务容器($this->app)

几乎所有的服务容器绑定都是在服务提供者中完成。因此本章节的演示例子用到的容器都是在这种上下文环境中，如果一个类没有基于任何接口那么就没有必要将其绑定到容器。容器并不需要被告知如何构建对象，因为它会使用 PHP 的反射服务自动解析出具体的对象。

在一个服务提供者中，可以通过 $this->app 变量访问容器，然后使用 bind 方法注册一个绑定，该方法需要两个参数，第一个参数是我们想要注册的类名或接口名称，第二个参数是返回类的实例的闭包：
```
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app['HttpClient']);
});
```
注意到我们接受容器本身作为解析器的一个参数，然后我们可以使用该容器来解析我们正在构建的对象的子依赖。

###4.1 绑定一个单例

singleton 方法绑定一个只需要解析一次的类或接口到容器，然后接下来对容器的调用将会返回同一个实例：
```
$this->app->singleton('FooBar', function ($app) {
    return new FooBar($app['SomethingElse']);
});
```
###4.2 绑定实例

你还可以使用 instance 方法绑定一个已存在的对象实例到容器，随后对容器的调用将总是返回给定的实例：
```
$fooBar = new FooBar(new SomethingElse);

$this->app->instance('FooBar', $fooBar);
```
###4.3 绑定接口到实现

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
###4.4 上下文绑定

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
###4.5 绑定原始值

有时候你可能有一个获取若干注入类的类，但还需要一个注入的原始值，比如整型数据，你可以轻松使用上下文绑定来注入指定类所需要的任何值：
```
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
    ->needs('$maxOrderCount')
    ->give(10);
```
###4.6 标签

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
###4.7 服务容器的监听

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


###4.8 关于服务提供者、服务容器、facedes三者的总结

	服务提供者提供服务容器的绑定（register）和事件监听、中间件等注册（boot）
	
	服务容器（$this->app）各种形式绑定*命名空间*到对应的*类*，单例、实例、接口、上下文绑定、原始值、标签。
	
	facedes 为服务容器中 '静态'的提供接口，方便快捷,静态调用。

##5 laravel 数据库
	支持数据系统
		Mysql	
		Postgres
		SQLite
		SQL Server
###5.1 起步
####读写分类数据库配置（读大于写）
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
####运行原生SQL查询
	#####参数绑定，避免SQL注入
	DB::select('select * from users where active = ?',[1]);
	DB::select('select * from users where id = :id',['id' => 1]);
	DB::insert('insert into users (id,name) value (?,?)', [1,'Dayle']);
	DB::update('update users set votes = 100 where name = ?',['John']);
	DB::delete('delete from users');
	DB::statement('drop table user');
	
	#####在服务提供者中注册监听查询事件
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
	
	#####数据库事务
	想要在一个数据库事务中运行一连串操作，可以使用DB门面的transaction方法，如果事务闭包中抛出异常，事务将会自动回滚，
	如果闭包执行成功，事务将会自动提交。使用transaction方法不需要担心手动回滚或提交：
	DB::transaction(function(){
		DB::table('users')->update(['votes'=> 1]);
		DB::table('posts')->delete();
	})
	
	#####手动提交事务
	DB::beginTransaction();
	DB::rollBack();
	DB::commit();
	
	#####使用多个数据库连接
	使用多个数据库连接的时候，可以使用DB门面的connection方法访问每个连接。参数名来至配置文件
	DB::connection('foo')->select();
	可以通过getPdo方法获取原生的PDO实例
	DB::connection()->getPdo();












































