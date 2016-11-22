# laravel 学习笔记  version：5.2
## 生命周期：
		 从`index.php`->加载`bootstrap/app.php`后创建服务容器->分发至内核`http内核或者console内核`
		内核配置错误处理、日志、检测环境、中间件、服务提供者->路由`router.php`->`控制器`
## 服务提供者：
		register 只用与绑定服务容易
		boot 用于事件监听 composer绑定试图，并可以访问可加重已注册的所有服务（改方法会在所有访问提供者注册完成后才被调用）
## 注册服务提供者：
所有服务提供者都是通过配置文件config/app.php中进行注册，该文件包含了一个列出所有服务提供者名字的providers数组，默认情况下，其中列出了所有核心服务提供者，这些服务提供者启动Laravel核心组件，比如邮件、队列、缓存等等。

要注册你自己的服务提供者，只需要将其追加到该数组中即可：	
```
'providers' => [
     // 其它服务提供者
     App\Providers\AppServiceProvider::class,
],
```	
## 懒加载服务提供者：
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