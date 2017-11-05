# Chapter 8. Test Databases

若測試能夠在不接觸 DB 的情況下完成，建議保留不碰觸 DB 的測試方式。原因如下：

1. 加速 Unit Test 的執行時間
2. 所謂 Unit Test 就應該在不碰觸 DB (任何會受到環境影響的周邊設定) 的情況下完成

> I'm a strong propnent for this very thing.

- 但有幾種情形或許必須要藉由碰觸 DB 才能比較方便完成測試:

1. 替 Legacy Code 補上 Unit Test
2. 若藉由經過 DB 來完成測試的話，會讓你晚上睡得比較好

> The world won't end if you break a few rules every once in a while.

**須注意: 若時常在做決定後，結果都是得經由 DB 來完成 Unit Test，則需要好好檢視開發的架構是否與 DB 依賴關係太強。**

## Test Databases

使用 Laravel Framework 的話，其本身在執行 Unit Test 時，便會將 [環境](https://github.com/laravel/laravel/blob/v4.2.0/app/tests/TestCase.php) 改為 `testing`，因此可以很方便地區分出要在 Unit Test 階段取用的 config 設定:

- [ ] 環境設定是如何將變數傳給 `Illuminate\Foundation\Application` 告知是 `testing` 環境的?

*! 書中教學是 Laravel 4.x 的做法，執行前請檢查自己使用的 Laravel 版本*

1. 在 `config/testing` 下放入 `database.php` 設定
2. 在 `config/testing/database.php` 內放入要提供 Unit Test 讀寫的 DB 設定:

```shell
<?php
// config/testing/database.php
return [
...
    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'host' => 'localhost',
            'database' => 'TEST_DB_NAME',
            'username' => 'USERNAME',
            'password' => 'PASSWORD',
            'charset' => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix' => '',
        ],
...
    ],
...
];
```
3. 執行 Unit Test 時，`config/testing/database.php` 的設定便會取代 `config/database.php` 的設定

## Databases in Memory

使用 `sqlite` 完成需要經過 DB 的測試項目，可以大幅加速 Unit Test 的時間。

- 無論哪個版本，只需要修改 connection 內的 driver 設定:

```php
<?php

return [
...
    'connections' => [
...
        'testing' => [
            'driver'   => 'sqlite',
            'database' => ':memory:',
            'prefix'   => '',
        ],
...
    ],
...
];
```

---------------------------------------

## [課外補充] Laravel 5.x 的作法參考

*使用 Laravel 5.5 做範例*

在 [phpunit.xml](https://github.com/laravel/laravel/blob/v5.5.0/phpunit.xml) 中，便將 `APP_ENV` 設定為 `testing`

1. 參考 `APP_ENV` 在 `phpunit.xml` 中的設定方式，便可以將 database connection 設定也在 `phpunit.xml` 內完成:

```shell
<env name="DB_CONNECTION" value="testing"/>
```

2. 在 `config/database.php` 下新增如下設定:

```php
<?php
// config/database.php
return [
...
    'connections' => [
...
        'testing' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_UNIT_TEST_DATABASE', 'unit_test'),
            'username' => env('DB_UNIT_TEST_USERNAME', 'forge'),
            'password' => env('DB_UNIT_TEST_PASSWORD', ''),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'strict' => true,
            'engine' => null,
        ],
...
    ],
...
];
```

若除了使用到的 database 設定不同，則也可以在 `phpunit.xml` 內簡單設定 `DB_DATABASE` 即可:

```shell
<env name="DB_DATABASE" value="testing"/>
```

**需要注意在各個測試的 `setUp` 重新做 `migrate:refresh` 與 seed，此舉是避免各自獨立的測試會因為執行的先後順序而彼此互相影響。**

- 可以在 `tests/TestCase.php` 中加入共用的 `setUp` 與 `tearDown`

```php
<?php
// tests/TestCase.php
namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
...
    public function setUp()
    {
        parent::setUp();
        $this->artisan('migrate:refresh');
        $this->seed();
    }
...
}
```

- 其他資料: [Laracasts 相關討論](https://laracasts.com/discuss/channels/testing/how-to-specify-a-testing-database-in-laravel-5)

- [ ] 其他經驗分享

---------------------------------------
