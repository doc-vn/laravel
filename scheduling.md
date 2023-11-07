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
- [Chạy Scheduler](#running-the-scheduler)
    - [Chạy Scheduler local](#running-the-scheduler-locally)
- [Task Output](#task-output)
- [Task Hook](#task-hooks)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Trong quá khứ, có thể bạn đã viết một cấu hình Cron cho mỗi task schedule mà bạn cần để server của bạn chạy. Tuy nhiên, điều này có thể nhanh chóng trở thành một vấn đề lớn, bởi vì task schedule của bạn không có trong source code nên bạn phải SSH vào server để xem từng mục cron hiện có của bạn hoặc thêm các Cron vào bằng một cách thủ công.

Lệnh schedule của Laravel cung cấp một cách tiếp cận mới để quản lý các task schedule trên máy chủ của bạn. Scheduler cho phép bạn định nghĩa một cách đơn giản và rõ ràng các lệnh đó trong chính Laravel application của bạn. Khi sử dụng schedule, chỉ cần một cron duy nhất trên server của bạn. Các task schedule của bạn sẽ được định nghĩa trong phương thức `schedule` ở file `app/Console/Kernel.php`. Để giúp bạn bắt đầu, một ví dụ mẫu đơn giản đã được định nghĩa sẵn trong phương thức đó.

<a name="defining-schedules"></a>
## Định nghĩa Schedule

Bạn có thể định nghĩa tất cả các task đã được schedule của bạn trong phương thức `schedule` của class `App\Console\Kernel` của application của bạn. Để bắt đầu, chúng ta hãy xem một ví dụ về schedule cho một task. Trong ví dụ này, chúng ta sẽ schedule một closure được gọi mỗi ngày vào lúc nửa đêm. Trong closure, chúng ta sẽ thực hiện một truy vấn vào cơ sở dữ liệu để xóa bảng:

    <?php

    namespace App\Console;

    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
    use Illuminate\Support\Facades\DB;

    class Kernel extends ConsoleKernel
    {
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

Ngoài việc tạo schedule bằng closures, bạn cũng có thể schedule cho [các đối tượng invokable](http://php.net/manual/en/language.oop5.magic.php#object.invoke). Đối tượng invokable là các class PHP đơn giản có chứa phương thức `__invoke`:

    $schedule->call(new DeleteRecentUsers)->daily();

Nếu bạn muốn xem tổng quan về các scheduled task của bạn và lần tiếp theo chúng được chạy, bạn có thể sử dụng lệnh Artisan `schedule:list`:

```nothing
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### Các lệnh Artisan Schedule

Ngoài việc tạo schedule cho closures, bạn cũng có thể tạo schedule cho [Lệnh Artisan](/docs/{{version}}/artisan) và các lệnh của hệ điều hành. Ví dụ, bạn có thể sử dụng phương thức `command` để tạo schedule cho lệnh Artisan bằng cách sử dụng tên hoặc class của command đó.

Khi lên lịch cho các lệnh Artisan bằng cách sử dụng tên class của lệnh, bạn có thể truyền thêm một mảng các tham số command-line cần được cung cấp cho lệnh khi nó được gọi:

    use App\Console\Commands\SendEmailsCommand;

    $schedule->command('emails:send Taylor --force')->daily();

    $schedule->command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();

<a name="scheduling-queued-jobs"></a>
### Schedule Queued Job

Phương thức `job` có thể được sử dụng để tạo schedule cho một [queued job](/docs/{{version}}/queues). Phương thức này cung cấp một cách thuận tiện để tạo schedule queued job mà không cần phải sử dụng phương thức `call` để định nghĩa closure cho queue job:

    use App\Jobs\Heartbeat;

    $schedule->job(new Heartbeat)->everyFiveMinutes();

Tùy chọn tham số thứ hai và thứ ba có thể được cung cấp cho phương thức `job` để chỉ định tên queue và kết nối của queue sẽ được sử dụng để queue job:

    use App\Jobs\Heartbeat;

    // Dispatch the job to the "heartbeats" queue on the "sqs" connection...
    $schedule->job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### Lệnh Shell Schedule

Phương thức `exec` có thể được sử dụng để ra lệnh cho hệ điều hành:

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### Tuỳ chọn tần suất Schedule

Chúng ta đã xem một số ví dụ về cách mà bạn có thể cấu hình task để chạy theo các khoảng thời gian nhất định. Tuy nhiên, có nhiều tần suất task schedule khác mà bạn có thể cấu hình cho một task:

Method  | Description
------------- | -------------
`->cron('* * * * *');`  |  Chạy task theo một tùy chỉnh cron schedule
`->everyMinute();`  |  Chạy task mỗi phút
`->everyTwoMinutes();`  |  Chạy task hai phút một lần
`->everyThreeMinutes();`  |  Chạy task ba phút một lần
`->everyFourMinutes();`  |  Chạy task bốn phút một lần
`->everyFiveMinutes();`  |  Chạy task năm phút một lần
`->everyTenMinutes();`  |   Chạy task mười phút một lần
`->everyFifteenMinutes();`  |   Chạy task mười lăm phút một lần
`->everyThirtyMinutes();`  |   Chạy task ba mươi phút một lần
`->hourly();`  |   Chạy task một giờ một lần
`->hourlyAt(17);`  |  Chạy task một giờ một lần vào phút thứ 17
`->everyTwoHours();`  |  Chạy task hai giờ một lần
`->everyThreeHours();`  |  Chạy task ba giờ một lần
`->everyFourHours();`  |  Chạy task bốn giờ một lần
`->everySixHours();`  | Chạy task sáu giờ một lần
`->daily();`  |  Chạy task hàng ngày
`->dailyAt('13:00');`  |  Chạy task hàng ngày vào lúc 13:00
`->twiceDaily(1, 13);`  | Chạy task hàng ngày vào lúc 1:00 và 13:00
`->weekly();`  |  Chạy task hàng tuần vào chủ nhật lúc 00:00
`->weeklyOn(1, '8:00');`  |  Chạy task hàng tuần vào thứ hai lúc 8:00
`->monthly();`  | Chạy task vào ngày đầu tiên của tháng lúc 00:00
`->monthlyOn(4, '15:00');`  |  Chạy task hàng ngày vào ngày thứ 4 của tháng và vào lúc 15:00
`->twiceMonthly(1, 16, '13:00');`  |  Chạy task hàng tháng vào ngày 1 và ngày 16 lúc 13h00
`->lastDayOfMonth('15:00');` | Chạy task vào ngày cuối dùng của tháng lúc 15:00
`->quarterly();` |  Chạy task vào ngày đầu tiên của quý lúc 00:00
`->yearly();`  | Chạy task vào ngày đầu tiên của năm lúc 00:00
`->yearlyOn(6, 1, '17:00');`  |  Chạy task hàng năm vào ngày 1 tháng 6 lúc 17:00
`->timezone('America/New_York');` | Set timezone cho task

Các phương thức này có thể được kết hợp thêm các ràng buộc để tạo ra các schedule có thể được điều chỉnh tốt hơn, ví dụ như chỉ chạy vào một số ngày nhất định trong tuần. Ví dụ, bạn có thể schedule một lệnh chạy vào thứ hai hàng tuần thì bạn có thể làm như sau:

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

Dưới đây là một danh sách các ràng buộc schedule có thể được thêm:

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
`->days(array\|mixed);`  |  Giới hạn task chỉ chạy vào ngày cụ thể
`->between($startTime, $endTime);`  | Giới hạn task chỉ chạy vào giữa thời gian start và end
`->unlessBetween($startTime, $endTime);`  |  Giới hạn task không chạy vào giữa thời gian start và end
`->when(Closure);`  |  Giới hạn task chỉ chạy trên một điều kiện đúng
`->environments($env);`  |  Giới hạn task trong các môi trường cụ thể

<a name="day-constraints"></a>
#### Day Constraints

Phương thức `days` có thể được sử dụng để giới hạn việc thực hiện một task trong những ngày cụ thể có trong một tuần. Ví dụ: bạn có thể tạo lịch chạy một lệnh mỗi giờ vào ngày chủ nhật và thứ tư:

    $schedule->command('emails:send')
                    ->hourly()
                    ->days([0, 3]);

Ngoài ra, bạn có thể sử dụng các hằng số có sẵn trên class `Illuminate\Console\Scheduling\Schedule` khi định nghĩa ngày mà một task sẽ chạy:

    use Illuminate\Console\Scheduling\Schedule;

    $schedule->command('emails:send')
                    ->hourly()
                    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);

<a name="between-time-constraints"></a>
#### Between Time Constraints

Phương thức `between` có thể được sử dụng để giới hạn việc thực thi của một task dựa vào một khoảng thời gian có trong ngày:

    $schedule->command('emails:send')
                        ->hourly()
                        ->between('7:00', '22:00');

Tương tự, phương thức `unlessBetween` có thể được sử dụng để chạy ngược lại việc thực thi của một task trong một khoảng thời gian:

    $schedule->command('emails:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

<a name="truth-test-constraints"></a>
#### Truth Test Constraints

Phương thức `when` có thể được sử dụng để hạn chế việc thực hiện một task dựa trên kết quả của một điều kiện nhất định. Nói cách khác, nếu closure đã cho trả về giá trị `true`, tác vụ sẽ thực thi, miễn là không có điều kiện ràng buộc nào khác ngăn task đó chạy:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

Phương thức `skip` có thể được xem là ngược lại với phương thức `when`. Nếu phương thức `skip` trả về giá trị `true`, scheduled task đó sẽ không được thực thi:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

Khi sử dụng kết hợp nhiều phương thức `when`, lệnh đã được schedule sẽ chỉ được thực thi nếu tất cả các điều kiện `when` đều trả về giá trị `true`.

<a name="environment-constraints"></a>
#### Environment Constraints

Phương thức `environments` có thể được sử dụng để chỉ thực thi các task trong các môi trường đã cho (được định nghĩa bởi [biến môi trường](/docs/{{version}}/configuration#environment-configuration) `APP_ENV` ):

    $schedule->command('emails:send')
                ->daily()
                ->environments(['staging', 'production']);

<a name="timezones"></a>
### Timezones

Sử dụng phương thức `timezone`, bạn có thể chỉ định thời gian của một task schedule sẽ được sử dụng trong một timezone nhất định:

    $schedule->command('report:generate')
             ->timezone('America/New_York')
             ->at('2:00')

Nếu bạn đang muốn chỉ định liên tục một múi giờ cho tất cả các task schedule của bạn, bạn có thể muốn định nghĩa phương thức `scheduleTimezone` trong class `App\Console\Kernel` của bạn. Phương thức này sẽ trả về múi giờ mặc định sẽ được chỉ định cho tất cả các task schedule:

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

> {note} Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng driver cache `database`, `memcached` `dynamodb`, hoặc `redis` làm driver cache mặc định của ứng dụng của bạn. Ngoài ra, tất cả các server phải được giao tiếp với cùng một server cache trung tâm.

Nếu schedule của ứng dụng của bạn đang chạy trên nhiều server, bạn có thể giới hạn schedule job chỉ được chạy trên một server duy nhất. Ví dụ: giả sử bạn đang có một task schedule là tạo một báo cáo vào mỗi tối thứ Sáu. Nếu schedule của bạn đang chạy trên ba server worker, thì task schedule sẽ được chạy trên cả ba server và tạo báo cáo ba lần. Không tốt!

Để yêu cầu task chỉ được chạy trên một server, hãy sử dụng phương thức `onOneServer` khi bạn định nghĩa task schedule. Server đầu tiên nhận được task sẽ dùng một atomic lock trên job đó để ngăn các server khác chạy cùng một task vào cùng một thời điểm:

    $schedule->command('report:generate')
                    ->fridays()
                    ->at('17:00')
                    ->onOneServer();

<a name="background-tasks"></a>
### Background Tasks

Mặc định, nhiều task được schedule vào cùng một thời gian sẽ phải chạy theo tuần tự dựa trên thứ tự mà chúng được định nghĩa trong phương thức `schedule` của bạn. Nếu bạn có những task cần phải chạy dài, thì điều này có thể khiến các task tiếp theo sẽ chạy muộn hơn so với dự kiến. Nếu bạn muốn chạy các task trong background để tất cả chúng có thể được chạy đồng thời, thì bạn có thể sử dụng phương thức `runInBackground`:

    $schedule->command('analytics:report')
             ->daily()
             ->runInBackground();

> {note} Phương thức `runInBackground` chỉ có thể được sử dụng khi task được tạo thông qua phương thức `command` và `exec`.

<a name="maintenance-mode"></a>
### Chế độ bảo trì

Các scheduled task của application của bạn sẽ không được chạy khi application của bạn ở [trong chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode), vì chúng tôi không muốn các task của bạn gây trở ngại với bất kỳ bảo trì nào mà bạn đang thực hiện trên server chưa được hoàn thành. Tuy nhiên, nếu bạn muốn task chạy ngay cả trong chế độ bảo trì, bạn có thể gọi phương thức `evenInMaintenanceMode` khi định nghĩa task:

    $schedule->command('emails:send')->evenInMaintenanceMode();

<a name="running-the-scheduler"></a>
## Chạy Scheduler

Hiện tại, chúng ta đã học cách định nghĩa các scheduled task, hãy thảo luận về cách thực sự chạy chúng trên máy chủ của chúng ta. Lệnh `schedule:run` Artisan sẽ đánh giá tất cả các scheduled task của bạn và xác định xem chúng có cần chạy hay không dựa trên thời gian hiện tại của máy chủ.

Vì vậy, khi sử dụng scheduler của Laravel, chúng ta chỉ cần thêm một mục cấu hình cron duy nhất vào máy chủ để chạy lệnh `schedule:run` mỗi phút. Nếu bạn không biết cách thêm các mục cron vào máy chủ của bạn, hãy cân nhắc sử dụng một dịch vụ như [Laravel Forge](https://forge.laravel.com) để có thể quản lý các mục cron cho bạn:

    * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1

<a name="running-the-scheduler-locally"></a>
## Chạy Scheduler local

Thông thường, bạn sẽ cần không thêm mục cron của scheduler vào máy phát triển local của bạn. Thay vào đó, bạn có thể sử dụng lệnh Artisan `schedule:work`. Lệnh này sẽ chạy ở trên giao diện và gọi scheduler mỗi phút cho đến khi bạn kết thúc lệnh:

    php artisan schedule:work

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

    $schedule->command('report:generate')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('taylor@example.com');

Nếu bạn muốn chỉ gửi email nếu scheduled Artisan hoặc system command kết thúc với exit code khác 0, hãy sử dụng phương thức `emailOutputOnFailure`:

    $schedule->command('report:generate')
             ->daily()
             ->emailOutputOnFailure('taylor@example.com');

> {note} Các phương thức `emailOutputTo`, `emailOutputOnFailure`, `sendOutputTo` và `appendOutputTo` sẽ chỉ được dùng với phương thức `command` và phương thức `exec`.

<a name="task-hooks"></a>
## Task Hook

Sử dụng các phương thức `before` và `after`, bạn có thể khai báo các code mà sẽ được thực thi trước và sau khi scheduled task được chạy:

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // The task is about to execute...
             })
             ->after(function () {
                 // The task has executed...
             });

Phương thức `onSuccess` và `onFailure` cho phép bạn chỉ định một đoạn code cụ thể sẽ được thực thi nếu scheduled task chạy thành công hoặc bị thất bại. Một thất bại sẽ xảy ra khi lệnh Artisan hoặc system command kết thúc với exit code khác 0:

    $schedule->command('emails:send')
             ->daily()
             ->onSuccess(function () {
                 // The task succeeded...
             })
             ->onFailure(function () {
                 // The task failed...
             });

Nếu lệnh của bạn trả về output, bạn có thể truy cập vào output trong hook `after`, `onSuccess` hoặc `onFailure` bằng cách khai báo một instance `Illuminate\Support\Stringable` làm tham số `$output` trong khi định nghĩa closure của hook:

    use Illuminate\Support\Stringable;

    $schedule->command('emails:send')
             ->daily()
             ->onSuccess(function (Stringable $output) {
                 // The task succeeded...
             })
             ->onFailure(function (Stringable $output) {
                 // The task failed...
             });

<a name="pinging-urls"></a>
#### Pinging URLs

Sử dụng các phương thức `pingBefore` và `thenPing`, schedule có thể tự động ping đến một URL đã cho trước hoặc sau khi task được chạy. Phương thức này sẽ hữu ích để thông báo cho một dịch vụ bên ngoài, chẳng hạn như [Laravel Envoyer](https://envoyer.io), rằng scheduled task của bạn đang bắt đầu hoặc đã kết thúc chạy:

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

Các phương thức `pingBeforeIf` và `thenPingIf` chỉ có thể được sử dụng để ping đến một URL nếu một điều kiện đưa vào là `true`:

    $schedule->command('emails:send')
             ->daily()
             ->pingBeforeIf($condition, $url)
             ->thenPingIf($condition, $url);

Phương thức `pingOnSuccess` và `pingOnFailure` có thể được sử dụng để ping đến một URL nhất định nếu task chạy thành công hoặc bị thất bại. Một thất bại sẽ xảy ra khi lệnh Artisan hoặc system command kết thúc với exit code khác 0:

    $schedule->command('emails:send')
             ->daily()
             ->pingOnSuccess($successUrl)
             ->pingOnFailure($failureUrl);

Tất cả các phương thức ping sẽ đều cần thư viện Guzzle HTTP. Guzzle thường được cài đặt mặc định trong tất cả các dự án Laravel mới, tuy nhiên, bạn có thể cài đặt Guzzle vào dự án của bạn theo cách thủ công bằng cách sử dụng Composer package manager nếu nó vô tình bị xóa:

    composer require guzzlehttp/guzzle

<a name="events"></a>
## Events

Nếu cần, bạn có thể listen [events](/docs/{{version}}/events) do scheduler gửi đi. Thông thường, ánh xạ giữa event và listener sẽ được định nghĩa trong class `App\Providers\EventServiceProvider` của ứng dụng của bạn:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Console\Events\ScheduledTaskStarting' => [
            'App\Listeners\LogScheduledTaskStarting',
        ],

        'Illuminate\Console\Events\ScheduledTaskFinished' => [
            'App\Listeners\LogScheduledTaskFinished',
        ],

        'Illuminate\Console\Events\ScheduledBackgroundTaskFinished' => [
            'App\Listeners\LogScheduledBackgroundTaskFinished',
        ],

        'Illuminate\Console\Events\ScheduledTaskSkipped' => [
            'App\Listeners\LogScheduledTaskSkipped',
        ],

        'Illuminate\Console\Events\ScheduledTaskFailed' => [
            'App\Listeners\LogScheduledTaskFailed',
        ],
    ];
