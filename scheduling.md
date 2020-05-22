# Task Scheduling

- [Giới thiệu](#introduction)
- [Định nghĩa Schedule](#defining-schedules)
    - [Các lệnh Artisan Schedule](#scheduling-artisan-commands)
    - [Schedule Queued Job](#scheduling-queued-jobs)
    - [Lệnh Shell Schedule](#scheduling-shell-commands)
    - [Tuỳ chọn tần suất Schedule](#schedule-frequency-options)
    - [Ngăn task chồng nhau](#preventing-task-overlaps)
    - [Chế độ bảo trì](#maintenance-mode)
- [Task Output](#task-output)
- [Task Hook](#task-hooks)

<a name="introduction"></a>
## Giới thiệu

Trước đây, có thể bạn đã tạo một Cron cho một task mà bạn cần để lên schedule cho server của bạn chạy. Tuy nhiên, điều này có thể nhanh chóng sẽ trở thành một vấn đề lớn, bởi vì task schedule của bạn không còn trong source control và bạn phải SSH vào server của bạn để thêm các Cron.

Lệnh schedule của Laravel cho phép bạn định nghĩa một cách đơn giản và rõ ràng lệnh schedule của bạn trong chính Laravel. Khi sử dụng schedule, chỉ cần một Cron duy nhất trên server của bạn. Task schedule của bạn sẽ được định nghĩa trong phương thức `schedule` trong file `app/Console/Kernel.php`. Để giúp bạn bắt đầu, một ví dụ đơn giản được định nghĩa sẵn trong phương thức đó.

### Starting The Scheduler

Khi sử dụng schedule, bạn chỉ cần thêm Cron dưới đây vào server của bạn. Nếu bạn không biết cách thêm các Cron vào server của bạn, hãy xem xét sử dụng một dịch vụ như [Laravel Forge](https://forge.laravel.com) để có thể quản lý các Cron cho bạn:

    * * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1

Cron này sẽ gọi lệnh schedule của Laravel mỗi phút. Khi lệnh `schedule:run` được thực thi, Laravel sẽ tìm các task theo schedule của bạn và chạy các task đã đến hạn.

<a name="defining-schedules"></a>
## Định nghĩa Schedule

Bạn có thể định nghĩa tất cả các task đã được schedule của bạn trong phương thức `schedule` của class `App\Console\Kernel`. Để bắt đầu, chúng ta hãy xem một ví dụ về schedule cho một task. Trong ví dụ này, chúng ta sẽ schedule một `Closure` được gọi mỗi ngày vào lúc nửa đêm. Trong `Closure`, chúng ta sẽ thực hiện một truy vấn cơ sở dữ liệu để xóa bảng:

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * The Artisan commands provided by your application.
         *
         * @var array
         */
        protected $commands = [
            //
        ];

        /**
         * Define the application's command schedule.
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

<a name="scheduling-artisan-commands"></a>
### Các lệnh Artisan Schedule

Ngoài việc tạo schedule cho các Closure, bạn cũng có thể tạo schedule cho [Lệnh Artisan](/docs/{{version}}/artisan) và các lệnh của hệ điều hành. Ví dụ, bạn có thể sử dụng phương thức `command` để tạo schedule cho lệnh Artisan bằng cách sử dụng tên hoặc class của lệnh:

    $schedule->command('emails:send --force')->daily();

    $schedule->command(EmailsCommand::class, ['--force'])->daily();

<a name="scheduling-queued-jobs"></a>
### Schedule Queued Job

Phương thức `job` có thể được sử dụng để tạo schedule cho một [queued job](/docs/{{version}}/queues). Phương thức này cung cấp một cách thuận tiện để tạo schedule job mà không cần sử dụng phương thức `call` để tự tạo Closure để queue job:

    $schedule->job(new Heartbeat)->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### Lệnh Shell Schedule

Phương thức `exec` có thể được sử dụng để ra lệnh cho hệ điều hành:

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### Tuỳ chọn tần suất Schedule

Tất nhiên, sẽ có nhiều schedule mà bạn có thể khai báo cho task của bạn:

Method  | Description
------------- | -------------
`->cron('* * * * * *');`  |  Chạy task theo một tùy chỉnh Cron schedule
`->everyMinute();`  |  Run task mỗi phút
`->everyFiveMinutes();`  |  Run task năm phút một lần
`->everyTenMinutes();`  |   Run task mười phút một lần
`->everyFifteenMinutes();`  |   Run task mười lăm phút một lần
`->everyThirtyMinutes();`  |   Run task ba mươi phút một lần
`->hourly();`  |   Run task một giờ một lần
`->hourlyAt(17);`  |  Run task một giờ một lần vào phút thứ 17
`->daily();`  |  Run task hàng ngày
`->dailyAt('13:00');`  |  Run task hàng ngày vào lúc 13:00
`->twiceDaily(1, 13);`  | Run task hàng ngày vào lúc 1:00 và 13:00
`->weekly();`  |  Run task hàng tuần
`->monthly();`  | Run task hàng tháng
`->monthlyOn(4, '15:00');`  |  Run task hàng ngày vào ngày thứ 4 của tháng vào lúc 15:00
`->quarterly();` |  Run task hàng quý
`->yearly();`  | Run task hàng năm
`->timezone('America/New_York');` | Set timezone

Các phương thức này có thể được kết hợp với thêm các ràng buộc để tạo ra các schedule có thể được điều chỉnh tốt hơn, chỉ chạy vào một số ngày nhất định trong tuần. Ví dụ: để schedule một lệnh chạy hàng tuần vào Thứ Hai:

    // Run once per week on Monday at 1 PM...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');

    // Run hourly from 8 AM to 5 PM on weekdays...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

Dưới đây là danh sách các ràng buộc schedule bổ sung:

Method  | Description
------------- | -------------
`->weekdays();`  |  Giới hạn task chỉ chạy vào các ngày trong tuần
`->sundays();`  |  Giới hạn task chỉ chạy vào chủ nhật
`->mondays();`  |  Giới hạn task chỉ chạy vào thứ hai
`->tuesdays();`  |  Giới hạn task chỉ chạy vào thứ ba
`->wednesdays();`  |  Giới hạn task chỉ chạy vào thứ tư
`->thursdays();`  |  Giới hạn task chỉ chạy vào thứ năm
`->fridays();`  |  Giới hạn task chỉ chạy vào thứ sáu
`->saturdays();`  |  Giới hạn task chỉ chạy vào thứ bảy
`->between($start, $end);`  | Giới hạn task chỉ chạy chỉ chạy vào giữa thời gian start và end
`->when(Closure);`  |  Giới hạn task chỉ chạy trên một điều kiện đúng

#### Between Time Constraints

Phương thức `between` có thể được sử dụng để giới hạn việc thực thi một task dựa trên một khoảng thời gian trong ngày:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

Tương tự, phương thức `unlessBetween` có thể được sử dụng để chạy ngược lại việc thực thi một task trong một khoảng thời gian:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

#### Truth Test Constraints

Phương thức `when` có thể được sử dụng để hạn chế việc thực hiện một task dựa trên kết quả của một điều kiện nhất định. Nói cách khác, nếu `Closure` đã cho trả về giá trị `true`, tác vụ sẽ thực thi, miễn là không có điều kiện ràng buộc nào khác ngăn tác vụ chạy:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

Phương thức `skip` có thể được xem là ngược của phương thức `when`. Nếu phương thức `skip` trả về `true`, scheduled task sẽ không được thực thi:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

Khi sử dụng kết hợp các phương thức `when`, lệnh đã được schedule sẽ chỉ được thực thi nếu tất cả các điều kiện `when` trả về `true`.

<a name="preventing-task-overlaps"></a>
### Ngăn task chồng nhau

Mặc định, các task đã được schedule sẽ được chạy ngay cả khi instance trước đó của task vẫn đang chạy. Để ngăn điều này xảy ra, bạn có thể sử dụng phương thức `withoutOverlapping`:

    $schedule->command('emails:send')->withoutOverlapping();

Trong ví dụ trên, [Lệnh Artisan](/docs/{{version}}/artisan) `emails:send` sẽ được chạy mỗi phút nếu nó chưa được chạy. Phương thức `withoutOverlapping` đặc biệt hữu ích nếu bạn có các task phức tạp cần nhiều thời gian để thực hiện của chúng, và bạn không thể dự đoán chính xác một task có thể sẽ mất bao nhiêu thời gian.

Nếu cần, bạn có thể chỉ định bao nhiêu phút sau khi thực hiện thì khóa "chống chồng" hết hạn. Mặc định, khóa này sẽ hết hạn sau 24 giờ:

    $schedule->command('emails:send')->withoutOverlapping(10);

<a name="maintenance-mode"></a>
### Chế độ bảo trì

Các scheduled task của Laravel sẽ không được chạy khi Laravel ở [chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode), vì chúng tôi không muốn các task của bạn gây trở ngại với bất kỳ bảo trì nào chưa được hoàn thành mà bạn có thể đang thực hiện trên server của bạn. Tuy nhiên, nếu bạn muốn một task chạy ngay cả trong chế độ bảo trì, bạn có thể sử dụng phương thức `evenInMaintenanceMode`:

    $schedule->command('emails:send')->evenInMaintenanceMode();

<a name="task-output"></a>
## Task Output

Laravel schedule cung cấp một số phương thức thuận tiện để làm việc với output được tạo bởi các scheduled tas. Đầu tiên, bằng cách sử dụng phương thức `sendOutputTo`, bạn có thể gửi output tới một file để kiểm tra sau:

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

Nếu bạn muốn nối output vào một file đã cho, bạn có thể sử dụng phương thức `appendOutputTo`:

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

Sử dụng phương thức `emailOutputTo`, bạn có thể gửi email output đến một địa chỉ email bạn chọn. Trước khi gửi email output của một task, bạn nên cấu hình [e-mail services](/docs/{{version}}/mail) của Laravel:

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> {note} Các phương thức `emailOutputTo`, `sendOutputTo` và `appendOutputTo` là chỉ được dùng với phương thức `command` và không hỗ trợ cho  phương thức `call`.

<a name="task-hooks"></a>
## Task Hook

Sử dụng các phương thức `before` và `after`, bạn có thể khai báo code sẽ được thực thi trước và sau khi scheduled task hoàn tất:

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });

#### Pinging URLs

Sử dụng các phương thức `pingBefore` và `thenPing`, schedule có thể tự động ping một URL đã cho trước hoặc sau khi một task hoàn tất. Phương thức này hữu ích để thông báo cho một dịch vụ bên ngoài, chẳng hạn như [Laravel Envoyer](https://envoyer.io), rằng scheduled task của bạn đang bắt đầu hoặc đã kết thúc:

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

Sử dụng tính năng `pingBefore($url)` hoặc `thenPing($url)` sẽ yêu cầu thư viện Guzzle HTTP. Bạn có thể thêm Guzzle vào dự án của bạn bằng trình quản lý package Composer:

    composer require guzzlehttp/guzzle
