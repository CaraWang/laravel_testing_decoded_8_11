# Chapter 9. Just Swap That Thang

剛開始使用 Laravel 的人，或許都對 facades 有些微不正確的認知，可能認為 Laravel 撰寫了大量的 static method，會令人聯想到難以測試的問題。

```php
Mail::send();
DB::getQueryLog();
Queue::push();
```

- [ ] static method 難以測試的例子

事實上，在 Laravel 中，利用了 [Facade Pattern](https://refactoring.guru/design-patterns/facade) 讓測試很好撰寫，同時也讓程式邏輯中享有呼叫 static method 那像的簡潔。

測試會很好撰寫的原因，是當你使用到 Laravel 中有實作 Facade 的功能時，可以很容易地做 Mock:

```php
<?php
namespace App\Services;

class JobService
{
...
    public function dispatchJob(string $queueName, Job $job): string
    {
        return Queue::connection($queueName)->push($job);
    }
}
```

上述程式對應的測試:

```php
<?php
namespace Tests\App\Services;

use App\Services\JobService;
use Tests/TestCase;

// test
class JobServiceTest extends TestCase
{
    public function testDispatchJob()
    {
        // Arrange
        $job = new JobStub;
        Queue::shouldReceive('connection')->once()->with('queue-name')->andReturnSelf();
        Queue::shouldReceive('push')->once()->with($job)->andReturn('message-id');

        // Act
        $result = (new JobService)->dispatchJob('queue-name', $job);

        // Assert
        $this->assertSame('message-id', $result);
    }
}
```

> Facade: A facade is an *object* that provides a simplified *interface* to a larger body of code.

**須注意: 單獨使用 Mockery 時，記得要在 `tearDown` 時執行: `Mockery::close()` 才不會讓 Mockery 指定的執行次數不準確:**

```php
<?php
namespace Tests\App\Services;

use Tests/TestCase;

// test
class JobServiceTest extends TestCase
{
    public function testDispatchJob()
    {
        // Arrange
        $job = new JobStub;

        // Wrong times of Queue facades be called -> `twice` should be `once`
        Queue::shouldReceive('connection')->twice()->with('queue-name')->andReturnSelf();
        Queue::shouldReceive('push')->once()->with($job)->andReturn('message-id');

        // Act
        $result = (new JobService)->dispatchJob('queue-name', $job);

        // Assert
        $this->assertSame('message-id', $result);
    }
}

```

**若使用 Laravel 5.x 則 `Mockery::close()` 已經直接被包在 TestCase 中的 [`tearDown()`](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Foundation/Testing/TestCase.php#L158) 中了。**

**若要撰寫自己的 `tearDown()` 邏輯，記得要先 `parent::tearDown()` 讓 Laravel 的 `tearDown()` 執行過。**

- [ ] trace code: laravel 的 facade 實作
