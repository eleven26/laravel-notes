### Laravel Facades 加载原理

* 框架里面注册 Facades 的地方
    * `\Illuminate\Foundation\Http\Kernel::$bootstrappers`
    * `\Illuminate\Foundation\Console\Kernel::$bootstrappers`

* 框架启动的时候会启动上面两个属性里面预设的类, 里面包含了 `\Illuminate\Foundation\Bootstrap\RegisterFacades`
    ```php
    /**
     * Bootstrap the given application.
     *
     * @param  \Illuminate\Contracts\Foundation\Application  $app
     * @return void
     */
    public function bootstrap(Application $app)
    {
        Facade::clearResolvedInstances();

        Facade::setFacadeApplication($app);

        AliasLoader::getInstance(array_merge(
            $app->make('config')->get('app.aliases', []),
            $app->make(PackageManifest::class)->aliases()
        ))->register();
    }
    ```

* `register` 方法里面调用了 `prependToLoaderStack`, 在 `prependToLoaderStack` 方法里面, 为所有 `alias` 定义了(通过了 `spl_autoload_register`)一个自动加载的方法

    ```php
    /**
     * Register the loader on the auto-loader stack.
     *
     * @return void
     */
    public function register()
    {
        if (!$this->registered) {
            $this->prependToLoaderStack();

            $this->registered = true;
        }
    }

    /**
     * Prepend the load method to the auto-loader stack.
     *
     * @return void
     */
    protected function prependToLoaderStack()
    {
        spl_autoload_register([$this, 'load'], true, true);
    }
    ```
    
* `load` 方法如下
    ```php
    /**
     * Load a class alias if it is registered.
     *
     * @param  string $alias
     * @return bool|null
     */
    public function load($alias)
    {
        if (static::$facadeNamespace && strpos($alias, static::$facadeNamespace) === 0) {
            $this->loadFacade($alias);

            return true;
        }

        if (isset($this->aliases[$alias])) {
            return class_alias($this->aliases[$alias], $alias);
        }
    }
    ```

我们可以看到, 在 `load` 方法里面, 使用了 `class_alias` 为 `alias` 定义了别名, 因此在我们使用 `\Log::info()` 之类的 `Facades` 调用的时候, 

实际上加载的是 `class_alias` 关联的完整的 `Illuminate\Support\Facades\Log`, 注册自动加载方法的时候, 第三个参数设置为 `true`,

因此, 自动加载的时候会先找出完整的命名空间(使用 `class_alias` 定义 `class` 别名), 然后在后续的自动加载中就可以成功加载 `Facades` 对应的类了。

* 注意: 上面 `load` 方法的返回值没有任何意义, `return` 在里面的意义是中断函数流程
    * [`https://gist.github.com/cebe/6206940`](https://gist.github.com/cebe/6206940)
    * [`https://github.com/php/php-src/blob/1d6a136051051e4e25f89a280ca92fde2da04463/ext/spl/php_spl.c#L411`](https://github.com/php/php-src/blob/1d6a136051051e4e25f89a280ca92fde2da04463/ext/spl/php_spl.c#L411)

* 另外: lumen 默认不使用 `Facade`, 需要使用的话, 需要在 `app.php` 加上 `$app->withFacades();`