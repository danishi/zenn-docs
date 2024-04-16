---
title: "Laravelを10から11へアップグレードしてみた"
emoji: "📈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["php", "laravel"]
published: true
publication_name: "iret"
---

年に一度の恒例行事化しつつあります。

普段使っているテンプレのプロジェクトをLaravel10から11へアップグレードしたので手順を残します。

https://readouble.com/laravel/11.x/ja/upgrade.html

---

手順はだいたい9から10の時と同じ。

https://zenn.dev/iret/articles/laravel-upgrade-from-9-to-10

PHPのバージョンは8.2.4で実施。

`composer.json`を書き換えて、依存性のエラーが出るもののバージョンを上げたり、パッケージを差し替えます。

```diff json:composer.json
    "require": {
-       "php": "^8.1",
+       "php": "^8.2",
-       "aws/aws-sdk-php-laravel": "^3.7",
+       "aws/aws-sdk-php-laravel": "^3.9",
-       "davejamesmiller/laravel-breadcrumbs": "^5.3",
+       "diglactic/laravel-breadcrumbs": "^9.0",
        "doctrine/dbal": "^3.3",
        "goodby/csv": "^1.3",
        "guzzlehttp/guzzle": "^7.2",
        "kyslik/column-sortable": "^6.4",
-       "laravel/framework": "^10.0",
+       "laravel/framework": "^11.0",
-       "laravel/sanctum": "^3.2",
+       "laravel/sanctum": "^4.0",
        "laravel/tinker": "^2.7",
        "laravel/ui": "^4.2",
-       "laravelcollective/html": "^6.3"
+       "spatie/laravel-html": "^3.6"
    },
    "require-dev": {
-       "barryvdh/laravel-debugbar": "^3.6",
+       "barryvdh/laravel-debugbar": "^3.12",
-       "barryvdh/laravel-ide-helper": "^2.12",
+       "barryvdh/laravel-ide-helper": "^3.0",
        "fakerphp/faker": "^1.9.1",
        "laravel/sail": "^1.0.1",
        "mockery/mockery": "^1.4.4",
-       "nunomaduro/collision": "^7.5",
+       "nunomaduro/collision": "^8.1",
        "phpunit/phpunit": "^10.0",
        "spatie/laravel-ignition": "^2.1",
        "squizlabs/php_codesniffer": "^3.6"
    },
```

いくつかパッケージが差し替わっているので`config/app.php`やuseしている箇所で参照パッケージを書き換えます。

```diff php:config/app.php
-       Collective\Html\HtmlServiceProvider::class,
+       Spatie\Html\HtmlServiceProvider::class,

-       DaveJamesMiller\Breadcrumbs\BreadcrumbsServiceProvider::class,
+       Diglactic\Breadcrumbs\ServiceProvider::class,
    ],

    'aliases' => Facade::defaultAliases()->merge([
        'AWS' => Aws\Laravel\AwsFacade::class,
        'Debugbar' => Barryvdh\Debugbar\Facade::class,
-       'Form' => Collective\Html\FormFacade::class,
-       'Html' => Collective\Html\HtmlFacade::class,
-       'Breadcrumbs' => DaveJamesMiller\Breadcrumbs\Facades\Breadcrumbs::class,
+       'Form' => Spatie\Html\Facades\Html::class,
+       'Html' => Spatie\Html\Facades\Html::class,
+       'Breadcrumbs' => Diglactic\Breadcrumbs\Breadcrumbs::class,
    ])->toArray(),
```

```diff php
- use DaveJamesMiller\Breadcrumbs\Facades\Breadcrumbs;
+ use Diglactic\Breadcrumbs\Breadcrumbs;
```

`composer update`を実行しモジュールを更新、以上。

```bash
$ php artisan -V 
Laravel Framework 11.0.7
```

---

Laravel11からディレクトリ構造の簡素化が行われたとのことですが、以前の構造のままでも動きはするようでLaravel6の時代から使っているテンプレートでもほとんど変えずに動かすことができました。
後方互換性神ですね👼
