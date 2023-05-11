---
title: "Laravelを9から10へアップグレードしてみた"
emoji: "📈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["php", "laravel"]
published: true
---

普段使っているテンプレのプロジェクトをLaravel9から10へアップグレードしたので手順を残します。

https://readouble.com/laravel/10.x/ja/upgrade.html

---

まずは`composer.json`を書き換えます。
また、今回PHPのバージョンを8.2.4にしています。

```diff json:composer.json
    "require": {
-       "php": "^8.0",
+       "php": "^8.1",
        "aws/aws-sdk-php-laravel": "^3.7",
        "davejamesmiller/laravel-breadcrumbs": "^5.3",
        "doctrine/dbal": "^3.3",
-       "fruitcake/laravel-cors": "^2.0.5",
        "goodby/csv": "^1.3",
        "guzzlehttp/guzzle": "^7.2",
        "kyslik/column-sortable": "^6.4",
-       "laravel/framework": "^9.0",
+       "laravel/framework": "^10.0",
-       "laravel/sanctum": "^2.14",
+       "laravel/sanctum": "^3.2",
        "laravel/tinker": "^2.7",
-       "laravel/ui": "^3.4",
+       "laravel/ui": "^4.2",
        "laravelcollective/html": "^6.3"
    },
    "require-dev": {
        "barryvdh/laravel-debugbar": "^3.6",
        "barryvdh/laravel-ide-helper": "^2.12",
        "fakerphp/faker": "^1.9.1",
        "laravel/sail": "^1.0.1",
        "mockery/mockery": "^1.4.4",
        "nunomaduro/collision": "^6.0",
-       "phpunit/phpunit": "^9.5.10",
+       "phpunit/phpunit": "^10.0",
-       "spatie/laravel-ignition": "^1.0",
+       "spatie/laravel-ignition": "^2.1",
        "squizlabs/php_codesniffer": "^3.6"
    },

    "config": {
        "optimize-autoloader": true,
        "preferred-install": "dist",
        "sort-packages": true,
        "platform": {
-           "php": "8.1.1"
+           "php": "8.2.4"
        }
    },

```

[fruitcake/laravel-cors](https://github.com/fruitcake/laravel-cors)はLaravel9.2以降、フレームワークに内臓されたため消しています。
このため`/app/Http/Kernel.php`を書き換えます。

```diff php:/app/Http/Kernel.php
    protected $middleware = [
        // \App\Http\Middleware\TrustHosts::class,
        \App\Http\Middleware\TrustProxies::class,
-       \Fruitcake\Cors\HandleCors::class,
+       \Illuminate\Http\Middleware\HandleCors::class,
```

`composer update`を実行しモジュールを更新します。
特にエラーなく更新でき完了。

```bash
$ php artisan -V 
Laravel Framework 10.9.0
```

---

以前Laravel6からLaravel9へのアップグレードにチャレンジしたときはかなり難儀しました。
今回は依存関係でハマらなかったのもありますが、１つバージョンを上げるだけならとても簡単でした。
