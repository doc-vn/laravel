# Processes

- [Giới thiệu](#introduction)
- [Gọi Processes](#invoking-processes)
    - [Process Options](#process-options)
    - [Process Output](#process-output)
    - [Pipelines](#process-pipelines)
- [Processes bất đồng bộ](#asynchronous-processes)
    - [Process ID và Signal](#process-ids-and-signals)
    - [Output Process bất đồng bộ ](#asynchronous-process-output)
- [Processes đồng thời](#concurrent-processes)
    - [Naming Pool Processes](#naming-pool-processes)
    - [Pool Process ID và Signal](#pool-process-ids-and-signals)
- [Testing](#testing)
    - [Faking Processes](#faking-processes)
    - [Faking Process cụ thể](#faking-specific-processes)
    - [Faking chuỗi Process](#faking-process-sequences)
    - [Faking Process bất đồng bộ](#faking-asynchronous-process-lifecycles)
    - [Các hàm kiểm tra](#available-assertions)
    - [Chặn Process chưa fake chạy](#preventing-stray-processes)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một API tối giản, rõ ràng xoay quanh [component Symfony Process](https://symfony.com/doc/current/components/process.html), cho phép bạn dễ dàng gọi các process bên ngoài từ ứng dụng Laravel của bạn. Các tính năng process của Laravel tập trung vào các trường hợp sử dụng phổ biến nhất và mang lại trải nghiệm tuyệt vời cho nhà phát triển.

<a name="invoking-processes"></a>
## Gọi Processes

Để gọi một process, bạn có thể sử dụng các phương thức `run` và `start` được cung cấp bởi facade `Process`. Phương thức `run` sẽ gọi một process và chờ process đó hoàn tất việc chạy, trong khi phương thức `start` được sử dụng để chạy process bất đồng bộ. Chúng ta sẽ xem xét cả hai cách trong tài liệu này. Trước tiên, hãy cùng tìm hiểu cách gọi một process đồng bộ cơ bản và kiểm tra kết quả của nó:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

Tất nhiên, instance `Illuminate\Contracts\Process\ProcessResult` được trả về bởi phương thức `run` cung cấp nhiều phương thức hữu ích có thể được sử dụng để kiểm tra kết quả process:

```php
$result = Process::run('ls -la');

$result->successful();
$result->failed();
$result->exitCode();
$result->output();
$result->errorOutput();
```

<a name="throwing-exceptions"></a>
#### Throwing Exceptions

Nếu bạn có kết quả process và muốn đưa ra một instance của `Illuminate\Process\Exceptions\ProcessFailedException` nếu exit code lớn hơn 0 (báo hiệu sự cố), bạn có thể sử dụng các phương thức `throw` và `throwIf`. Nếu process không lỗi, instance kết quả process sẽ được trả về:

```php
$result = Process::run('ls -la')->throw();

$result = Process::run('ls -la')->throwIf($condition);
```

<a name="process-options"></a>
### Process Options

Tất nhiên, bạn có thể cần tùy chỉnh hành vi của một process trước khi gọi nó. May mắn thay, Laravel cho phép bạn tinh chỉnh nhiều tính năng của process, chẳng hạn như thư mục làm việc, thời gian chờ và biến môi trường.


<a name="working-directory-path"></a>
#### Working Directory Path

Bạn có thể sử dụng phương thức `path` để chỉ định thư mục làm việc của process. Nếu phương thức này không được gọi, process sẽ kế thừa thư mục làm việc của script PHP đang được chạy:

```php
$result = Process::path(__DIR__)->run('ls -la');
```

<a name="input"></a>
#### Input

Bạn có thể cung cấp dữ liệu input thông qua "standard input" của process bằng phương thức `input`:

```php
$result = Process::input('Hello World')->run('cat');
```

<a name="timeouts"></a>
#### Timeouts

Mặc định, các process sẽ đưa ra một instance `Illuminate\Process\Exceptions\ProcessTimedOutException` sau khi chạy hơn 60 giây. Tuy nhiên, bạn có thể tùy chỉnh hành vi này thông qua phương thức `timeout`:

```php
$result = Process::timeout(120)->run('bash import.sh');
```

Hoặc, nếu bạn muốn disable hoàn toàn thời gian chờ của process, bạn có thể gọi phương thức `forever`:

```php
$result = Process::forever()->run('bash import.sh');
```

Phương thức `idleTimeout` có thể được sử dụng để chỉ định số giây tối đa mà một process có thể chạy mà không trả về bất kỳ output nào:

```php
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

<a name="environment-variables"></a>
#### Environment Variables

Biến môi trường có thể được cung cấp cho process thông qua phương thức `env`. Process được gọi cũng sẽ kế thừa tất cả các biến môi trường do hệ thống của bạn định nghĩa:

```php
$result = Process::forever()
            ->env(['IMPORT_PATH' => __DIR__])
            ->run('bash import.sh');
```

Nếu bạn muốn xóa một biến môi trường được kế thừa ra khỏi process được gọi, bạn có thể cung cấp biến môi trường đó với giá trị `false`:

```php
$result = Process::forever()
            ->env(['LOAD_PATH' => false])
            ->run('bash import.sh');
```

<a name="tty-mode"></a>
#### TTY Mode

Phương thức `tty` có thể được sử dụng để bật chế độ TTY cho process của bạn. Chế độ TTY kết nối input và output của process với input và output của chương trình, cho phép process của bạn mở các editor như Vim hoặc Nano như một process:

```php
Process::forever()->tty()->run('vim');
```

<a name="process-output"></a>
### Process Output

Như đã thảo luận trước đó, output của process có thể được truy cập bằng cách sử dụng các phương thức `output` (stdout) và `errorOutput` (stderr) trên kết quả process:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

echo $result->output();
echo $result->errorOutput();
```

Tuy nhiên, output cũng có thể được hiển thị theo thời gian thực bằng cách truyền một closure làm tham số thứ hai cho phương thức `run`. Closure sẽ nhận hai tham số: "kiểu" của output (`stdout` hoặc `stderr`) và chính chuỗi output đó:

```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```

Laravel cũng cung cấp các phương thức `seeInOutput` và `seeInErrorOutput`, cung cấp một cách thuận tiện để xác định xem một chuỗi nhất định có nằm trong output của một process hay không:

```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

<a name="disabling-process-output"></a>
#### Disabling Process Output

Nếu process của bạn đang ghi ra một lượng lớn dữ liệu output mà bạn không quan tâm, bạn có thể tiết kiệm bộ nhớ bằng cách tắt hoàn toàn việc lấy dữ liệu output ra. Để thực hiện việc này, hãy gọi phương thức `quietly` trong khi tạo process:

```php
use Illuminate\Support\Facades\Process;

$result = Process::quietly()->run('bash import.sh');
```

<a name="process-pipelines"></a>
### Pipelines

Thỉnh thoảng bạn có thể muốn biến output của một process thành input của một process khác. Điều này thường được gọi là "pipe" (dẫn) output của một process sang một process khác. Phương thức `pipe` được cung cấp bởi các facade `Process` sẽ giúp việc này trở nên dễ dàng. Phương thức `pipe` sẽ thực thi các process đã được chuỗi hoá một cách đồng bộ và trả về kết quả của process cuối cùng trong một pipeline:

```php
use Illuminate\Process\Pipe;
use Illuminate\Support\Facades\Process;

$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
});

if ($result->successful()) {
    // ...
}
```

Nếu bạn không cần tùy chỉnh từng process tạo chuỗi, bạn có thể chỉ cần truyền một mảng chuỗi lệnh vào phương thức `pipe`:

```php
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```

Output của process có thể được hiển thị theo thời gian thực bằng cách truyền một closure làm tham số thứ hai cho phương thức `pipe`. Closure sẽ nhận hai tham số: "kiểu" của output (`stdout` hoặc `stderr`) và chính chuỗi output đó:

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
}, function (string $type, string $output) {
    echo $output;
});
```

Laravel cũng cho phép bạn gán một khóa cho từng process trong một pipeline thông qua phương thức `as`. Khóa này cũng sẽ được truyền đến closure output được cung cấp cho phương thức `pipe`, cho phép bạn xác định output là thuộc về process nào:

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->as('first')->command('cat example.txt');
    $pipe->as('second')->command('grep -i "laravel"');
})->start(function (string $type, string $output, string $key) {
    // ...
});
```

<a name="asynchronous-processes"></a>
## Processes bất đồng bộ

Trong khi phương thức `run` gọi các process theo cách đồng bộ, thì phương thức `start` có thể được sử dụng để gọi một process theo cách không đồng bộ. Điều này cho phép ứng dụng của bạn tiếp tục thực hiện các tác vụ khác trong khi một process đang chạy ở chế độ background. Sau khi process đã được gọi, bạn có thể sử dụng phương thức `running` để xác định xem process đó có còn đang chạy nữa hay là không:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // ...
}

$result = $process->wait();
```

Như bạn có thể đã nhận thấy, bạn có thể gọi phương thức `wait` để đợi cho đến khi process được thực thi xong và lấy instance kết quả của process đó:

```php
$process = Process::timeout(120)->start('bash import.sh');

// ...

$result = $process->wait();
```

<a name="process-ids-and-signals"></a>
### Process ID và Signal

Phương thức `id` có thể được sử dụng để lấy ID của process mà được hệ điều hành gán cho khi process được chạy:

```php
$process = Process::start('bash import.sh');

return $process->id();
```

Bạn có thể sử dụng phương thức `signal` để gửi một "tín hiệu" đến process đang chạy. Danh sách các hằng số tín hiệu này, bạn có thể được tìm thấy trong [tài liệu PHP](https://www.php.net/manual/en/pcntl.constants.php):

```php
$process->signal(SIGUSR2);
```

<a name="asynchronous-process-output"></a>
### Output Process bất đồng bộ

Trong khi một process bất đồng bộ đang chạy, bạn có thể truy cập vào toàn bộ output hiện tại của process này bằng các phương thức `output` và `errorOutput`; tuy nhiên, bạn cũng có thể sử dụng `latestOutput` và `latestErrorOutput` để lấy ra output cuối cùng mà lấy được:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    echo $process->latestOutput();
    echo $process->latestErrorOutput();

    sleep(1);
}
```

Giống như phương thức `run`, output cũng có thể được hiển thị theo thời gian thực từ các process bất đồng bộ bằng cách truyền một closure làm tham số thứ hai cho phương thức `start`. Closure sẽ nhận hai tham số: "kiểu" của output (`stdout` hoặc `stderr`) và chính chuỗi output đó:

```php
$process = Process::start('bash import.sh', function (string $type, string $output) {
    echo $output;
});

$result = $process->wait();
```

<a name="concurrent-processes"></a>
## Processes đồng thời

Laravel cũng giúp việc quản lý một group các process chạy đồng thời và bất đồng bộ trở nên dễ dàng, cho phép bạn dễ dàng thực hiện nhiều tác vụ cùng lúc. Để bắt đầu, hãy gọi phương thức `pool`, phương thức này chấp nhận một closure nhận vào một instance của `Illuminate\Process\Pool`.

Trong closure này, bạn có thể định nghĩa các process thuộc về group. Khi một group process được khởi động thông qua phương thức `start`, bạn có thể truy cập vào [collection](/docs/{{version}}/collections) các process đang chạy thông qua phương thức `running`:

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

$pool = Process::pool(function (Pool $pool) {
    $pool->path(__DIR__)->command('bash import-1.sh');
    $pool->path(__DIR__)->command('bash import-2.sh');
    $pool->path(__DIR__)->command('bash import-3.sh');
})->start(function (string $type, string $output, int $key) {
    // ...
});

while ($pool->running()->isNotEmpty()) {
    // ...
}

$results = $pool->wait();
```

Như bạn thấy, bạn có thể đợi tất cả các process trong group hoàn thành việc chạy và trả về kết quả của chúng thông qua phương thức `wait`. Phương thức `wait` sẽ trả về một đối tượng có thể truy cập được trong mảng, cho phép bạn truy cập vào instance kết quả của từng process trong group bằng khóa của nó:

```php
$results = $pool->wait();

echo $results[0]->output();
```

Hoặc, để thuận tiện hơn, phương thức `concurrently` có thể được sử dụng để khởi động một nhóm các process bất đồng bộ và ngay lập tức chờ trả về kết quả. Điều này có thể cung cấp cú pháp đặc biệt rõ ràng khi kết hợp với khả năng phân mảng của PHP:

```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});

echo $first->output();
```

<a name="naming-pool-processes"></a>
### Naming Pool Processes

Việc truy cập vào kết quả của nhóm process thông qua khóa mảng không mang tính biểu đạt cao; do đó, Laravel cho phép bạn gán khóa cho từng process có trong nhóm thông qua phương thức `as`. Khóa này cũng sẽ được truyền đến closure được cung cấp cho phương thức `start`, cho phép bạn xác định output sẽ thuộc về process nào:

```php
$pool = Process::pool(function (Pool $pool) {
    $pool->as('first')->command('bash import-1.sh');
    $pool->as('second')->command('bash import-2.sh');
    $pool->as('third')->command('bash import-3.sh');
})->start(function (string $type, string $output, string $key) {
    // ...
});

$results = $pool->wait();

return $results['first']->output();
```

<a name="pool-process-ids-and-signals"></a>
### Pool Process ID và Signal

Vì phương thức `running` của nhóm process trả về một collection tất cả các process được gọi trong nhóm, nên bạn có thể dễ dàng lấy ra tất cả các ID của process có trong nhóm:

```php
$processIds = $pool->running()->each->id();
```

Và, để thuận tiện, bạn có thể gọi phương thức `signal` trên một nhóm process để gửi tín hiệu đến mọi process có trong nhóm:

```php
$pool->signal(SIGUSR2);
```

<a name="testing"></a>
## Testing

Nhiều service Laravel cung cấp nhiều chức năng giúp bạn viết testcase dễ dàng và rõ ràng, và service process của Laravel cũng không ngoại lệ. Phương thức `fake` của facade `Process` cho phép bạn hướng dẫn Laravel trả về kết quả stub hoặc dummy khi các process được gọi.

<a name="faking-processes"></a>
### Faking Processes

Để khám phá khả năng fake process của Laravel, hãy tưởng tượng một route gọi đến một process:

```php
use Illuminate\Support\Facades\Process;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    Process::run('bash import.sh');

    return 'Import complete!';
});
```

Khi testing route này, chúng ta có thể hướng dẫn Laravel trả về một kết quả process thành công giả cho mỗi process được gọi bằng cách gọi phương thức `fake` trên facade `Process` mà không có tham số. Ngoài ra, chúng ta thậm chí có thể [kiểm tra](#available-assertions) một process nhất định đã được "chạy":

```php
<?php

namespace Tests\Feature;

use Illuminate\Process\PendingProcess;
use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Support\Facades\Process;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_process_is_invoked(): void
    {
        Process::fake();

        $response = $this->get('/import');

        // Simple process assertion...
        Process::assertRan('bash import.sh');

        // Or, inspecting the process configuration...
        Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
            return $process->command === 'bash import.sh' &&
                   $process->timeout === 60;
        });
    }
}
```

Như đã thảo luận ở trên, việc gọi phương thức `fake` trên facade `Process` sẽ hướng dẫn Laravel luôn trả về kết quả process là thành công và không có output. Tuy nhiên, bạn có thể dễ dàng chỉ định output và exit code cho các fake process bằng phương thức `result` của facade `Process`:

```php
Process::fake([
    '*' => Process::result(
        output: 'Test output',
        errorOutput: 'Test error output',
        exitCode: 1,
    ),
]);
```

<a name="faking-specific-processes"></a>
### Faking Process cụ thể

Như bạn có thể đã nhận thấy trong ví dụ trước, facade `Process` cho phép bạn chỉ định các kết quả fake khác nhau cho mỗi process bằng cách truyền vào một mảng các kết quả cho phương thức `fake`.

Các khóa của mảng phải đại diện cho các câu lệnh mà bạn muốn fake và kết quả tương ứng. Ký tự `*` có thể được sử dụng làm ký tự đại diện. Bất kỳ lệnh process nào chưa được fake sẽ thực hiện thực tế. Bạn có thể sử dụng phương thức `result` của facade `Process` để xây dựng kết quả stub hoặc fake cho các lệnh này:

```php
Process::fake([
    'cat *' => Process::result(
        output: 'Test "cat" output',
    ),
    'ls *' => Process::result(
        output: 'Test "ls" output',
    ),
]);
```

Nếu bạn không cần tùy chỉnh exit code hoặc lỗi output của một fake process, bạn có thể thấy thuận tiện hơn khi chỉ định kết quả của fake process dưới dạng một chuỗi đơn giản:

```php
Process::fake([
    'cat *' => 'Test "cat" output',
    'ls *' => 'Test "ls" output',
]);
```

<a name="faking-process-sequences"></a>
### Faking chuỗi Process

Nếu code của bạn đang kiểm tra việc gọi nhiều process trong cùng một lệnh, bạn có thể muốn gán một kết quả process giả khác nhau cho mỗi lần gọi process. Bạn có thể thực hiện việc này thông qua phương thức `sequence` của facade `Process`:

```php
Process::fake([
    'ls *' => Process::sequence()
                ->push(Process::result('First invocation'))
                ->push(Process::result('Second invocation')),
]);
```

<a name="faking-asynchronous-process-lifecycles"></a>
### Faking Process bất đồng bộ

Hiện tại, chúng ta chủ yếu thảo luận về việc tạo các fake process được gọi đồng bộ bằng phương thức `run`. Tuy nhiên, nếu bạn đang cố gắng kiểm tra các code mà tương tác với các fake process bất đồng bộ được gọi thông qua phương thức `start`, bạn có thể cần một phương thức phức tạp hơn để mô tả các fake process của bạn.

Ví dụ, hãy tưởng tượng route sau tương tác với một process bất đồng bộ:

```php
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    $process = Process::start('bash import.sh');

    while ($process->running()) {
        Log::info($process->latestOutput());
        Log::info($process->latestErrorOutput());
    }

    return 'Done';
});
```

Để fake đúng process này, chúng ta cần mô tả được phương thức `running` sẽ trả về `true` bao nhiêu lần. Và ngoài ra, chúng ta có thể muốn chỉ định nhiều dòng output sẽ được trả về theo trình tự. Để thực hiện điều này, chúng ta có thể sử dụng phương thức `describe` của facade `Process`:

```php
Process::fake([
    'bash import.sh' => Process::describe()
            ->output('First line of standard output')
            ->errorOutput('First line of error output')
            ->output('Second line of standard output')
            ->exitCode(0)
            ->iterations(3),
]);
```

Hãy cùng xem xét ví dụ trên. Sử dụng các phương thức `output` và `errorOutput`, chúng ta có thể chỉ định nhiều dòng output sẽ được trả về theo trình tự. Phương thức `exitCode` có thể được sử dụng để chỉ định exit code cuối cùng của fake process. Cuối cùng, phương thức `iterations` có thể được sử dụng để chỉ định số lần mà phương thức `running` sẽ trả về `true`.

<a name="available-assertions"></a>
### Các hàm kiểm tra

Như [đã thảo luận trước đó](#faking-processes), Laravel cung cấp một số các hàm kiểm tra process cho các bài test chức năng của bạn. Chúng ta sẽ thảo luận về từng hàm kiểm tra này ở bên dưới.

<a name="assert-process-ran"></a>
#### assertRan

Kiểm tra một process nhất định đã được gọi:

```php
use Illuminate\Support\Facades\Process;

Process::assertRan('ls -la');
```

Phương thức `assertRan` cũng chấp nhận một closure, sẽ nhận vào một instance của một process và một kết quả của process đó, cho phép bạn kiểm tra các tùy chọn được cấu hình cho process đó. Nếu closure này trả về `true`, kiểm tra này sẽ "pass":

```php
Process::assertRan(fn ($process, $result) =>
    $process->command === 'ls -la' &&
    $process->path === __DIR__ &&
    $process->timeout === 60
);
```

Tham số `$process` được truyền vào hàm closure `assertRan` là một instance của `Illuminate\Process\PendingProcess`, trong khi tham số `$result` là một instance của `Illuminate\Contracts\Process\ProcessResult`.

<a name="assert-process-didnt-run"></a>
#### assertDidntRun

Kiểm tra một process nhất định không được gọi:

```php
use Illuminate\Support\Facades\Process;

Process::assertDidntRun('ls -la');
```

Giống như phương thức `assertRan`, phương thức `assertDidntRun` cũng chấp nhận một closure, phương thức này sẽ nhận một instance của một process và một kết quả của process, cho phép bạn kiểm tra các tùy chọn được cấu hình cho process đó. Nếu closure này trả về `true`, kiểm tra này sẽ "thất bại":

```php
Process::assertDidntRun(fn (PendingProcess $process, ProcessResult $result) =>
    $process->command === 'ls -la'
);
```

<a name="assert-process-ran-times"></a>
#### assertRanTimes

Kiểm tra một process nhất định đã được gọi theo một số lần:

```php
use Illuminate\Support\Facades\Process;

Process::assertRanTimes('ls -la', times: 3);
```

Phương thức `assertRanTimes` cũng chấp nhận một closure, phương thức này sẽ nhận một instance của một process và một kết quả của process, cho phép bạn kiểm tra các tùy chọn được cấu hình cho process đó. Nếu closure này trả về `true` và process được gọi với số lần đã được chỉ định, thì kiểm tra sẽ "pass":

```php
Process::assertRanTimes(function (PendingProcess $process, ProcessResult $result) {
    return $process->command === 'ls -la';
}, times: 3);
```

<a name="preventing-stray-processes"></a>
### Chặn Process chưa fake chạy

Nếu bạn muốn đảm bảo rằng tất cả các process được gọi đều đã được fake kết quả trong một bài test riêng lẻ hoặc tất cả các bài test, bạn có thể gọi phương thức `preventStrayProcesses`. Sau khi gọi phương thức này, bất kỳ process nào mà không có kết quả fake tương ứng thì process đó sẽ đưa ra một ngoại lệ thay vì khởi chạy một process thực tế:

    use Illuminate\Support\Facades\Process;

    Process::preventStrayProcesses();

    Process::fake([
        'ls *' => 'Test output...',
    ]);

    // Fake response is returned...
    Process::run('ls -la');

    // An exception is thrown...
    Process::run('bash import.sh');
