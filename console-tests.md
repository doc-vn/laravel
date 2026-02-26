# Console Tests

- [Giới thiệu](#introduction)
- [Kỳ vọng thành công hay thất bai](#success-failure-expectations)
- [Kỳ vọng input và output](#input-output-expectations)
- [Console Events](#console-events)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc đơn giản hóa cách kiểm tra HTTP, Laravel cung cấp một API đơn giản để kiểm tra [các lệnh console tuỳ chỉnh](/docs/{{version}}/artisan) của application của bạn.

<a name="success-failure-expectations"></a>
## Kỳ vọng thành công hay thất bai

Để bắt đầu, hãy khám phá cách kiểm tra liên quan đến exit code của lệnh Artisan. Để thực hiện điều này, chúng ta sẽ sử dụng phương thức `artisan` để gọi một lệnh Artisan từ bài test của chúng ta. Sau đó, chúng ta sẽ sử dụng phương thức `assertExitCode` để kiểm tra xem lệnh đã chạy xong với một exit code nhất định:

```php tab=Pest
test('console command', function () {
    $this->artisan('inspire')->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('inspire')->assertExitCode(0);
}
```

Bạn có thể sử dụng phương thức `assertNotExitCode` để kiểm tra rằng lệnh đã thoát với một exit code nhất định:

```php
$this->artisan('inspire')->assertNotExitCode(1);
```

Tất nhiên, tất cả các lệnh terminal thường thoát với một status code là `0` khi chúng thành công và exit code khác 0 khi chúng thất bại. Do đó, để thuận tiện, bạn có thể sử dụng kiểm tra `assertSuccessful` và `assertFailed` để kiểm tra rằng một lệnh nhất định đã thoát với exit code thành công hay không:

```php
$this->artisan('inspire')->assertSuccessful();

$this->artisan('inspire')->assertFailed();
```

<a name="input-output-expectations"></a>
## Kỳ vọng input và output

Laravel cho phép bạn dễ dàng "mô phỏng" cách nhập của người dùng trên các cửa sổ dòng lệnh bằng phương thức `expectsQuestion`. Ngoài ra, bạn cũng có thể sử dụng các phương thức `assertExitCode` và `expectsOutput` để chỉ định các exit code và các text mà bạn mong muốn được xuất hiện trên cửa sổ dòng lệnh. Ví dụ: hãy xem lệnh console sau:

```php
Artisan::command('question', function () {
    $name = $this->ask('What is your name?');

    $language = $this->choice('Which language do you program in?', [
        'PHP',
        'Ruby',
        'Python',
    ]);

    $this->line('Your name is '.$name.' and you program in '.$language.'.');
});
```

Bạn có thể kiểm tra lệnh này bằng cách sử dụng bài test dưới đây:

```php tab=Pest
test('console command', function () {
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
}
```

Nếu bạn đang sử dụng các hàm `search` hoặc `multisearch` do [Laravel Prompts](/docs/{{version}}/prompts) cung cấp, bạn có thể sử dụng kiểm tra `expectsSearch` để mô phỏng dữ liệu input, kết quả tìm kiếm và lựa chọn của người dùng:

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->expectsSearch('What is your name?', search: 'Tay', answers: [
            'Taylor Otwell',
            'Taylor Swift',
            'Darian Taylor'
        ], answer: 'Taylor Otwell')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsSearch('What is your name?', search: 'Tay', answers: [
            'Taylor Otwell',
            'Taylor Swift',
            'Darian Taylor'
        ], answer: 'Taylor Otwell')
        ->assertExitCode(0);
}
```

Bạn cũng có thể kiểm tra lệnh console sẽ không tạo ra bất kỳ output nào bằng phương thức `doesntExpectOutput`:

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
}
```

Các phương thức `expectsOutputToContain` và `doesntExpectOutputToContain` có thể được sử dụng để đưa ra các kiểm tra đối với một phần output:

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->expectsOutputToContain('Taylor')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsOutputToContain('Taylor')
        ->assertExitCode(0);
}
```

<a name="confirmation-expectations"></a>
#### Confirmation Expectations

Khi viết một lệnh để kiểm tra một confirmation dưới dạng câu trả lời "có" hoặc "không", bạn có thể sử dụng phương thức `expectsConfirmation`:

```php
$this->artisan('module:import')
    ->expectsConfirmation('Do you really wish to run this command?', 'no')
    ->assertExitCode(1);
```

<a name="table-expectations"></a>
#### Table Expectations

Nếu lệnh của bạn hiển thị một bảng thông tin bằng cách sử dụng phương thức `table` của Artisan, thì việc viết các kỳ vọng output cho toàn bộ bảng có thể rất phức tạp. Thay vào đó, bạn có thể sử dụng phương thức `expectsTable`. Phương thức này sẽ chấp nhận header của bảng làm tham số đầu tiên và dữ liệu của bảng đó làm tham số thứ hai:

```php
$this->artisan('users:all')
    ->expectsTable([
        'ID',
        'Email',
    ], [
        [1, 'taylor@example.com'],
        [2, 'abigail@example.com'],
    ]);
```

<a name="console-events"></a>
## Console Events

Mặc định, các event `Illuminate\Console\Events\CommandStarting` và `Illuminate\Console\Events\CommandFinished` sẽ không được gửi đi khi đang chạy test cho ứng dụng của bạn. Tuy nhiên, bạn có thể kích hoạt các event này cho một class test case nhất định bằng cách thêm trait `Illuminate\Foundation\Testing\WithConsoleEvents` vào class:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\WithConsoleEvents;

pest()->use(WithConsoleEvents::class);

// ...
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithConsoleEvents;
use Tests\TestCase;

class ConsoleEventTest extends TestCase
{
    use WithConsoleEvents;

    // ...
}
```
