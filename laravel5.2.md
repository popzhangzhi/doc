# laravel 学习笔记  version：5.2
## 生命周期：
		 从```index.php```->加载`bootstrap/app.php`后创建服务容器->分发至内核`http内核或者console内核`
		内核配置错误处理、日志、检测环境、中间件、服务提供者->路由`router.php`->`控制器`
