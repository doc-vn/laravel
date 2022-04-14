# Task Scheduling

- [Giới thiệu](#introduction)
- [Định nghĩa Schedule](#defining-schedules)
    - [Các lệnh Artisan Schedule](#scheduling-artisan-commands)
    - [Schedule Queued Job](#scheduling-queued-jobs)
    - [Lệnh Shell Schedule](#scheduling-shell-commands)
    - [Tuỳ chọn tần suất Schedule](#schedule-frequency-options)
    - [Timezones](#timezones)
    - [Ngăn task chồng nhau](#preventing-task-overlaps)
    - [Chạy task trên một server](#running-tasks-on-one-server)
    - [Background Tasks](#background-tasks)
    - [Chế độ bảo trì](#maintenance-mode)
- [Task Output](#task-output)
- [Task Hook](#task-hooks)

<a name="introduction"></a>
## Giới thiệu

Trong quá khứ, có thể bạn đã tạo ra Cron cho một task schedule mà bạn cần để server của bạn chạy. Tuy nhiên, điều này có thể nhanh chóng trở thành một vấn đề lớn, bởi vì task schedule của bạn không có trong source code nên bạn phải SSH vào server và thêm các Cron vào bằng một cách thủ công.

Lệnh schedule của Laravel cho phép bạn định nghĩa một cách đơn giản và rõ ràng các lệnh đó trong chính Laravel của bạn. Khi sử dụng schedule, chỉ cần một Cron duy nhất trên server của bạn. Các task schedule của bạn sẽ được định nghĩa trong phương thức `schedule` ở file `app/Console/Kernel.php`. Để giúp bạn bắt đầu, một ví dụ mẫu đơn giản đã được định nghĩa sẵn trong phương thức đó.

### Starting The Scheduler

Khi sử dụng schedule, bạn chỉ cần thêm Cron dưới đây vào server của bạn. Nếu bạn không biết cách thêm Cron vào server của bạn, hãy xem xét sử dụng một dịch vụ như [Laravel Forge](https://forge.laravel.com) để có thể quản lý Cron cho bạn:

    * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1

Cron này sẽ gọi lệnh schedule của Laravel mỗi phút. Khi lệnh `schedule:run` được thực thi, Laravel sẽ tìm các task theo schedule của bạn và chạy các task đã đến hạn.

<a name="defining-schedules"></a>
## Định nghĩa Schedule

Bạn có thể định nghĩa tất cả các task đã được schedule của bạn trong phương thức `schedule` của class `App\Console\Kernel`. Để bắt đầu, chúng ta hãy xem một ví dụ về schedule cho một task. Trong ví dụ này, chúng ta sẽ schedule một `Closure` được gọi mỗi ngày vào lúc nửa đêm. Trong `Closure`, chúng ta sẽ thực hiện một truy vấn vào cơ sở dữ liệu để xóa bảng:

    <?php

    namespace App\Console;

    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
    use Illuminate\Support\Facades\DB;

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

Ngoài việc tạo schedule bằng Closures, bạn cũng có thể sử dụng [các đối tượng invokable](http://php.net/manual/en/language.oop5.magic.php#object.invoke). Đối tượng invokable là các class PHP đơn giản có chứa phương thức `__invoke`:

    $schedule->call(new DeleteRecentUsers)->daily();

<a name="scheduling-artisan-commands"></a>
### Các lệnh Artisan Schedule

Ngoài việc tạo schedule cho Closure, bạn cũng có thể tạo schedule cho [Lệnh Artisan](/docs/{{version}}/artisan) và các lệnh của hệ điều hành. Ví dụ, bạn có thể sử dụng phương thức `command` để tạo schedule cho lệnh Artisan bằng cách sử dụng tên hoặc class của command đó:

    $schedule->command('emails:send Taylor --force')->daily();

    $schedule->command(EmailsCommand::class, ['Taylor', '--force'])->daily();

<a name="scheduling-queued-jobs"></a>
### Schedule Queued Job

Phương thức `job` có thể được sử dụng để tạo schedule cho một [queued job](/docs/{{version}}/queues). Phương thức này cung cấp một cách thuận tiện để tạo schedule job mà không cần phải sử dụng phương thức `call` để tạo Closure cho queue job:

    $schedule->job(new Heartbeat)->everyFiveMinutes();

    // Dispatch the job to the "heartbeats" queue...
    $schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### Lệnh Shell Schedule

Phương thức `exec` có thể được sử dụng để ra lệnh cho hệ điều hành:

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### Tuỳ chọn tần suất Schedule

Sẽ có nhiều schedule mà bạn có thể khai báo cho task của bạn:

Method  | Description
------------- | -------------
`->cron('* * * * *');`  |  Chạy task theo một tùy chỉnh Cron schedule
`->everyMinute();`  |  Chạy task mỗi phút
`->everyFiveMinutes();`  |  Chạy task năm phút một lần
`->everyTenMinutes();`  |   Chạy task mười phút một lần
`->everyFifteenMinutes();`  |   Chạy task mười lăm phút một lần
`->everyThirtyMinutes();`  |   Chạy task ba mươi phút một lần
`->hourly();`  |   Chạy task một giờ một lần
`->hourlyAt(17);`  |  Chạy task một giờ một lần vào phút thứ 17
`->daily();`  |  Chạy task hàng ngày
`->dailyAt('13:00');`  |  Chạy task hàng ngày vào lúc 13:00
`->twiceDaily(1, 13);`  | Chạy task hàng ngày vào lúc 1:00 và 13:00
`->weekly();`  |  Chạy task hàng tuần vào chủ nhật lúc 00:00
`->weeklyOn(1, '8:00');`  |  Chạy task hàng tuần vào thứ hai lúc 8:00
`->monthly();`  | Chạy task vào ngày đầu tiên của tháng lúc 00:00
`->monthlyOn(4, '15:00');`  |  Chạy task hàng ngày vào ngày thứ 4 của tháng và vào lúc 15:00
`->quarterly();` |  Chạy task vào ngày đầu tiên của quý lúc 00:00
`->yearly();`  | Chạy task vào ngày đầu tiên của năm lúc 00:00
`->timezone('America/New_York');` | Set timezone

Các phương thức này có thể được kết hợp thêm các ràng buộc để tạo ra các schedule có thể được điều chỉnh tốt hơn, ví dụ như chỉ chạy vào một số ngày nhất định trong tuần. Để schedule một lệnh chạy vào thứ hai hàng tuần thì bạn có thể làm như sau:

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

Dưới đây là danh sách các ràng buộc schedule có thể được thêm:

Method  | Description
------------- | -------------
`->weekdays();`  |  Giới hạn task chỉ chạy vào các ngày trong tuần
`->weekends();`  |  Giới hạn task chỉ chạy vào cuối tuần
`->sundays();`  |  Giới hạn task chỉ chạy vào chủ nhật
`->mondays();`  |  Giới hạn task chỉ chạy vào thứ hai
`->tuesdays();`  |  Giới hạn task chỉ chạy vào thứ ba
`->wednesdays();`  |  Giới hạn task chỉ chạy vào thứ tư
`->thursdays();`  |  Giới hạn task chỉ chạy vào thứ năm
`->fridays();`  |  Giới hạn task chỉ chạy vào thứ sáu
`->saturdays();`  |  Giới hạn task chỉ chạy vào thứ bảy
`->between($start, $end);`  | Giới hạn task chỉ chạy chỉ chạy vào giữa thời gian start và end
`->when(Closure);`  |  Giới hạn task chỉ chạy trên một điều kiện đúng
`->environments($env);`  |  Giới hạn task trong các môi trường cụ thể

#### Between Time Constraints

Phương thức `between` có thể được sử dụng để giới hạn việc thực thi của một task dựa vào một khoảng thời gian có trong ngày:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

Tương tự, phương thức `unlessBetween` có thể được sử dụng để chạy ngược lại việc thực thi của một task trong một khoảng thời gian:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

#### Truth Test Constraints

Phương thức `when` có thể được sử dụng để hạn chế việc thực hiện một task dựa trên kết quả của một điều kiện nhất định. Nói cách khác, nếu `Closure` đã cho trả về giá trị `true`, tác vụ sẽ thực thi, miễn là không có điều kiện ràng buộc nào khác ngăn task đó chạy:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

Phương thức `skip` có thể được xem là ngược lại với phương thức `when`. Nếu phương thức `skip` trả về giá trị `true`, scheduled task đó sẽ không được thực thi:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

Khi sử dụng kết hợp nhiều phương thức `when`, lệnh đã được schedule sẽ chỉ được thực thi nếu tất cả các điều kiện `when` đều trả về giá trị `true`.

#### Environment Constraints

Phương thức `environments` có thể được sử dụng để chỉ thực thi các task trong các môi trường đã cho:

    $schedule->command('emails:send')
                ->daily()
                ->environments(['staging', 'production']);

<a name="timezones"></a>
### Timezones

Sử dụng phương thức `timezone`, bạn có thể chỉ định thời gian của một task schedule sẽ được sử dụng trong một timezone nhất định:

    $schedule->command('report:generate')
             ->timezone('America/New_York')
             ->at('02:00')

Nếu bạn đang muốn chỉ định một múi giờ cho tất cả các task schedule của bạn, bạn có thể muốn định nghĩa phương thức `scheduleTimezone` trong file `app/Console/Kernel.php` của bạn. Phương thức này sẽ trả về múi giờ mặc định sẽ được chỉ định cho tất cả các task schedule:

    /**
     * Get the timezone that should be used by default for scheduled events.
     *
     * @return \DateTimeZone|string|null
     */
    protected function scheduleTimezone()
    {
        return 'America/Chicago';
    }

> {note} Hãy nhớ rằng một số timezone sử dụng quy ước giờ mùa hè. Khi các thay đổi về quy ước giờ mùa hè xảy ra, schedule task của bạn có thể chạy hai lần hoặc thậm chí là hoàn toàn không chạy. Vì lý do này, chúng tôi khuyên bạn nên tránh tạo schedule timezone khi có thể.

<a name="preventing-task-overlaps"></a>
### Ngăn task chồng nhau

Mặc định, các task đã được schedule sẽ được chạy, ngay cả khi instance trước đó của task vẫn đang được chạy. Để ngăn điều này xảy ra, bạn có thể sử dụng phương thức `withoutOverlapping`:

    $schedule->command('emails:send')->withoutOverlapping();

Ở ví dụ trên, [Lệnh Artisan](/docs/{{version}}/artisan) `emails:send` sẽ được chạy mỗi phút nếu nó chưa được chạy. Phương thức `withoutOverlapping` đặc biệt hữu ích nếu bạn có một task phức tạp cần nhiều thời gian để thực hiện, và bạn không thể dự đoán chính xác task đó sẽ mất bao nhiêu thời gian để hoàn thành.

Nếu cần, bạn có thể chỉ định số phút mà sau khi task được thực hiện thì khóa "chống lặp" hết hạn. Mặc định, khóa này sẽ hết hạn sau 24 giờ:

    $schedule->command('emails:send')->withoutOverlapping(10);

<a name="running-tasks-on-one-server"></a>
### Chạy task trên một server

> {note} Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng driver cache `memcached` hoặc `redis` làm driver cache mặc định của ứng dụng của bạn. Ngoài ra, tất cả các server phải được giao tiếp với cùng một server cache trung tâm.

Nếu ứng dụng của bạn đang chạy trên nhiều server, bạn có thể giới hạn schedule job chỉ được chạy trên một server duy nhất. Ví dụ: giả sử bạn đang có một task schedule là tạo một báo cáo vào mỗi tối thứ Sáu. Nếu schedule của bạn đang chạy trên ba server worker, thì task schedule sẽ được chạy trên cả ba server và tạo báo cáo ba lần. Không tốt!

Để yêu cầu task chỉ được chạy trên một server, hãy sử dụng phương thức `onOneServer` khi bạn định nghĩa task schedule. Server đầu tiên nhận được task sẽ dùng một atomic lock trên job đó để ngăn các server khác chạy cùng một task vào cùng một thời điểm:

    $schedule->command('report:generate')
                    ->fridays()
                    ->at('17:00')
                    ->onOneServer();

<a name="background-tasks"></a>
### Background Tasks

Mặc định, nhiều lệnh được schedule vào cùng một thời gian sẽ phải chạy theo tuần tự. Nếu bạn có những lệnh cần phải chạy dài, thì điều này có thể khiến các lệnh tiếp theo sẽ chạy muộn hơn so với dự kiến. Nếu bạn muốn chạy các lệnh trong background để tất cả chúng có thể được chạy đồng thời, thì bạn có thể sử dụng phương thức `runInBackground`:

    $schedule->command('analytics:report')
             ->daily()
             ->runInBackground();

> {note} Phương thức `runInBackground` chỉ có thể được sử dụng khi task được tạo thông qua phương thức `command` và `exec`.

<a name="maintenance-mode"></a>
### Chế độ bảo trì

Các scheduled task của Laravel sẽ không được chạy khi Laravel ở [trong chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode), vì chúng tôi không muốn các task của bạn gây trở ngại với bất kỳ bảo trì nào mà bạn đang thực hiện trên server chưa được hoàn thành. Tuy nhiên, nếu bạn muốn task chạy ngay cả trong chế độ bảo trì, bạn có thể sử dụng phương thức `evenInMaintenanceMode`:

    $schedule->command('emails:send')->evenInMaintenanceMode();

<a name="task-output"></a>
## Task Output

Laravel schedule cung cấp một số phương thức thuận tiện để làm việc với output được tạo ra bởi schedule task. Đầu tiên, bằng cách sử dụng phương thức `sendOutputTo`, bạn có thể gửi output tới một file để kiểm tra sau khi chạy:

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

Nếu bạn muốn thêm output vào một file đã có, bạn có thể sử dụng phương thức `appendOutputTo`:

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

Sử dụng phương thức `emailOutputTo`, bạn có thể gửi email output đến một địa chỉ email mà bạn đã chọn. Trước khi gửi email output của một task, bạn nên cấu hình [e-mail services](/docs/{{version}}/mail) của Laravel:

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

Nếu bạn muốn chỉ gửi e-mail nếu lệnh chạy không thành công, hãy sử dụng phương thức `emailOutputOnFailure`:

    $schedule->command('foo')
             ->daily()
             ->emailOutputOnFailure('foo@example.com');

> {note} Các phương thức `emailOutputTo`, `emailOutputOnFailure`, `sendOutputTo` và `appendOutputTo` sẽ chỉ được dùng với phương thức `command` và phương thức `exec`.

<a name="task-hooks"></a>
## Task Hook

Sử dụng các phương thức `before` và `after`, bạn có thể khai báo các code mà sẽ được thực thi trước và sau khi scheduled task hoàn tất:

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });

Phương thức `onSuccess` và `onFailure` cho phép bạn chỉ định một đoạn code cụ thể sẽ được thực thi nếu scheduled task chạy thành công hoặc bị thất bại:

    $schedule->command('emails:send')
             ->daily()
             ->onSuccess(function () {
                 // The task succeeded...
             })
             ->onFailure(function () {
                 // The task failed...
             });

#### Pinging URLs

Sử dụng các phương thức `pingBefore` và `thenPing`, schedule có thể tự động ping đến một URL đã cho trước hoặc sau khi task được hoàn tất. Phương thức này sẽ hữu ích để thông báo cho một dịch vụ bên ngoài, chẳng hạn như [Laravel Envoyer](https://envoyer.io), rằng scheduled task của bạn đang bắt đầu hoặc đã kết thúc:

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

Các phương thức `pingBeforeIf` và `thenPingIf` chỉ có thể được sử dụng để ping đến một URL nếu điều kiện đưa vào là `true`:

    $schedule->command('emails:send')
             ->daily()
             ->pingBeforeIf($condition, $url)
             ->thenPingIf($condition, $url);

Phương thức `pingOnSuccess` và `pingOnFailure` có thể được sử dụng để ping đến một URL nhất định nếu task chạy thành công hoặc bị thất bại:

    $schedule->command('emails:send')
             ->daily()
             ->pingOnSuccess($successUrl)
             ->pingOnFailure($failureUrl);

Tất cả các phương thức ping sẽ đều cần thư viện Guzzle HTTP. Bạn có thể thêm thư viện Guzzle này vào project của bạn bằng trình quản lý package Composer:

    composer require guzzlehttp/guzzle
