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
    
* 上面说到的 `bootstrapper` 是什么?
    * `HttpKernel`, `ConsoleKernel` 初始化的时候的一些初始化操作
