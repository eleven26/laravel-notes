### Laravel Application

* `Application` 里面定义了什么
    * 路径相关变量
        * `$basePath` 项目根目录路径
        * `$appPath` 项目 app 文件夹绝对路径
        * `$databasePath` 项目 database 文件夹绝对路径
        * `$environmentPath` .env 文件所在绝对路径
    * Laravel 容器生命周期相关变量
        * `$hasBeenBootstrapped` 是否已经启动, 防止项目重复加载某些资源
        * `$booted`
        * `$bootingCallbacks`
        * `$bootedCallbacks`
        * `$terminatingCallbacks`
    * ServiceProvider 相关变量
        * `$serviceProviders` 所有已经 register 的 ServiceProvider
        * `$loadedProviders` 所有已经加载的 ServiceProvider
        * `$deferredServices` 延后加载的 ServiceProvider
    * `VERSION` 版本常量, Laravel 当前版本
    * `$environmentFile` 环境配置文件名
    * `$namespace` 应用命名空间

* `__construct` 做了什么?
    * `setBasePath` 设置项目根目录路径
        * 还定义了如下路径, 如 `$this->instance('path', $this->path());`
            * `path` app 文件夹路径
            * `basePath` 项目根目录路径
            * `langPath`
            * `configPath`
            * `publicPath`
            * `storagePath`
            * `databasePath`
            * `resourcePath`
            * `bootstrapPath`
        * 上面几个路径都有同名方法, 每个方法都可以传递相对路径作为参数, 最终返回绝对路径, 如 `$this->basePath('database')` 将返回和 `database_path()` 一样的值
    * `registerBaseBindings`
        * 设置容器实例为当前对象
    * `registerBaseServiceProviders`
        * 注册 `EventServiceProvider`, Laravel 事件
        * 注册 `LogServiceProvider`, Laravel Log
        * 注册 `RoutingServiceProvider`, Laravel 路由
    * `bootstrapWith` 启动另外一些基础服务
        * `\Illuminate\Foundation\Http\Kernel::$bootstrappers`
        * `\Illuminate\Foundation\Console\Kernel::$bootstrappers`
        * 也就是说, 新版本 laravel 默认不能使用 env、config、Facades 等功能, 除非 HttpKernel 或者 ConsoleKernel 加载了
        
* env 相关方法
    * `environmentPath` 获取环境变量文件所在路径
    * `loadEnvironmentFrom` 设置环境变量文件名称
    * `environmentFile` 获取环境变量文件名称
    * `environmentFilePath` 获取环境变量文件绝对路径
    * `environment` 检查当前环境是否是某个环境
    * `isLocal` 是否是 local 环境
    * `detectEnvironment` 检测当前环境
    * `runningInConsole` 是否是命令行环境
    * `runningUnitTests` 是否正在运行单元测试

* 容器相关一些钩子
    * `afterLoadingEnvironment`
    * `beforeBootstrapping`
    * `afterBootstrapping`

* 框架相关路径设置
    * `app('path')` `path($path = '')`, `path($path = '')` 可以传递相对项目根目录的路径获取该相对路径的绝对路径
    * `app('path.base')` `base_path()`, `base_path($path = '')` 可以传递相对 `base_path` 路径的参数获取该相对路径的绝对路径
    * `app('path.lang')`
    * `app('path.config')`
    * `app('path.storage')`
    * `app('path.database')`
    * `app('path.resources')`
    * `app('path.bootstrap')`

* Service Provider 相关
    * `registerConfiguredProviders`         加载 `Providers` 的方法, 在 `HttpKernel` 或者 `ConsoleKernel` 初始化的时候调用 (对象的 `$bootstrappers` 属性)
    * `register($provider, $force = false)` 注册一个 ServiceProvider, 第二个参数传 true 可以强制重新 register
        * 在这个方法里面会实例化 `register` 的每一个 ServiceProvider
        * `ServiceProvider` 里面定义的 `bindings`, `singletons` 会绑定到容器实例
        * `register` 完成会标记为 `registered`
        * 最后如果 `ServiceProvider` 是在应用启动完毕之后才 `register` 的, 会直接调用 `ServiceProvider` 的 `boot` 方法(否则, 会在 `app()->boot()` 方法里面来调用 `ServiceProvider` 的 `boot` 方法)
    * `$deferredServices`
        * 这里面定义的 `ServiceProvider` 会在 `app()->make` 的时候才会实例化这些 `ServiceProvider`
        * 添加 `$deferredServices`: `addDeferredServices(array $services)`
        * `setDeferredServices` 替换掉原有的 `$deferredServices`
        * 是否是 `deferredService`: `isDeferredService($service)`
    * 查看当前状态下应用加载的 `ServiceProviders`
        * `getLoadedProviders`
        * `getDeferredServices`
    
* 应用生命周期相关
    * `abort`       中断应用流程, 抛出 `HttpException`
    * `terminating` 注册应用退出的回调
    * `terminate`   调用应用退出的回调
    
* 请求处理
    * `app()->handle()` 方法

* 测试相关
    * `shouldSkipMiddleware` 如果我们想在测试的时候不启用中间件, 可以使用 `app()->instance('middleware.disable', true)`, 这样, 框架就会在处理 http 请求的时候忽略掉中间件
    
* 应用套件缓存相关
    * `getCachedServicesPath` 获取缓存的 `Services` 路径
    * `getCachedPackagesPath` 获取缓存的 `Packages` 路径
    * `getCachedConfigPath` 获取缓存的 `config` 路径
    * `configurationIsCached` 判断配置文件是否被缓存了
    * `routesAreCached` 判断路由文件是否被缓存了
    * `getCachedRoutesPath` 获取缓存的 路由 路径

* 应用维护模式
    * `isDownForMaintenance` 应用是否正在维护
        * 相关命令 `php artisan down`, `php artisan up`
* 地区相关
    * `getLocale`
    * `setLocale`
    * `isLocale($locale)`

* Laravel 应用容器可用的别名列表
    * `registerCoreContainerAliases`
        * 有了这个, 我们就可以使用别名来代替完整类名, 比如 `app('cache')`, `app('files')` 等
    * 所有可用的别名列表
    ```php
    [
        'app'                  => [\Illuminate\Foundation\Application::class, \Illuminate\Contracts\Container\Container::class, \Illuminate\Contracts\Foundation\Application::class,  \Psr\Container\ContainerInterface::class],
        'auth'                 => [\Illuminate\Auth\AuthManager::class, \Illuminate\Contracts\Auth\Factory::class],
        'auth.driver'          => [\Illuminate\Contracts\Auth\Guard::class],
        'blade.compiler'       => [\Illuminate\View\Compilers\BladeCompiler::class],
        'cache'                => [\Illuminate\Cache\CacheManager::class, \Illuminate\Contracts\Cache\Factory::class],
        'cache.store'          => [\Illuminate\Cache\Repository::class, \Illuminate\Contracts\Cache\Repository::class],
        'config'               => [\Illuminate\Config\Repository::class, \Illuminate\Contracts\Config\Repository::class],
        'cookie'               => [\Illuminate\Cookie\CookieJar::class, \Illuminate\Contracts\Cookie\Factory::class, \Illuminate\Contracts\Cookie\QueueingFactory::class],
        'encrypter'            => [\Illuminate\Encryption\Encrypter::class, \Illuminate\Contracts\Encryption\Encrypter::class],
        'db'                   => [\Illuminate\Database\DatabaseManager::class],
        'db.connection'        => [\Illuminate\Database\Connection::class, \Illuminate\Database\ConnectionInterface::class],
        'events'               => [\Illuminate\Events\Dispatcher::class, \Illuminate\Contracts\Events\Dispatcher::class],
        'files'                => [\Illuminate\Filesystem\Filesystem::class],
        'filesystem'           => [\Illuminate\Filesystem\FilesystemManager::class, \Illuminate\Contracts\Filesystem\Factory::class],
        'filesystem.disk'      => [\Illuminate\Contracts\Filesystem\Filesystem::class],
        'filesystem.cloud'     => [\Illuminate\Contracts\Filesystem\Cloud::class],
        'hash'                 => [\Illuminate\Hashing\HashManager::class],
        'hash.driver'          => [\Illuminate\Contracts\Hashing\Hasher::class],
        'translator'           => [\Illuminate\Translation\Translator::class, \Illuminate\Contracts\Translation\Translator::class],
        'log'                  => [\Illuminate\Log\LogManager::class, \Psr\Log\LoggerInterface::class],
        'mailer'               => [\Illuminate\Mail\Mailer::class, \Illuminate\Contracts\Mail\Mailer::class, \Illuminate\Contracts\Mail\MailQueue::class],
        'auth.password'        => [\Illuminate\Auth\Passwords\PasswordBrokerManager::class, \Illuminate\Contracts\Auth\PasswordBrokerFactory::class],
        'auth.password.broker' => [\Illuminate\Auth\Passwords\PasswordBroker::class, \Illuminate\Contracts\Auth\PasswordBroker::class],
        'queue'                => [\Illuminate\Queue\QueueManager::class, \Illuminate\Contracts\Queue\Factory::class, \Illuminate\Contracts\Queue\Monitor::class],
        'queue.connection'     => [\Illuminate\Contracts\Queue\Queue::class],
        'queue.failer'         => [\Illuminate\Queue\Failed\FailedJobProviderInterface::class],
        'redirect'             => [\Illuminate\Routing\Redirector::class],
        'redis'                => [\Illuminate\Redis\RedisManager::class, \Illuminate\Contracts\Redis\Factory::class],
        'request'              => [\Illuminate\Http\Request::class, \Symfony\Component\HttpFoundation\Request::class],
        'router'               => [\Illuminate\Routing\Router::class, \Illuminate\Contracts\Routing\Registrar::class, \Illuminate\Contracts\Routing\BindingRegistrar::class],
        'session'              => [\Illuminate\Session\SessionManager::class],
        'session.store'        => [\Illuminate\Session\Store::class, \Illuminate\Contracts\Session\Session::class],
        'url'                  => [\Illuminate\Routing\UrlGenerator::class, \Illuminate\Contracts\Routing\UrlGenerator::class],
        'validator'            => [\Illuminate\Validation\Factory::class, \Illuminate\Contracts\Validation\Factory::class],
        'view'                 => [\Illuminate\View\Factory::class, \Illuminate\Contracts\View\Factory::class],
    ]
    ```
    * 另外有人会发现使用 `app('redis')` 这种写法可能没有语法提示, 这种情况可以使用 [barryvdh/laravel-ide-helper](https://github.com/barryvdh/laravel-ide-helper) 来生成 `.phpstorm.meta.php` 文件
    
* 上面说到的 `bootstrapper` 是什么?
    * `HttpKernel`, `ConsoleKernel` 初始化的时候的一些初始化操作
