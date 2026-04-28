# Task Scheduling

- [Giới thiệu](#introduction)
- [Định nghĩa Schedule](#defining-schedules)
    - [Các lệnh Artisan Schedule](#scheduling-artisan-commands)
    - [Schedule Queued Job](#scheduling-queued-jobs)
    - [Lệnh Shell Schedule](#scheduling-shell-commands)
    - [Tùy chọn tần suất Schedule](#schedule-frequency-options)
    - [Timezones](#timezones)
    - [Ngăn task chồng nhau](#preventing-task-overlaps)
    - [Chạy task trên một server](#running-tasks-on-one-server)
    - [Background Tasks](#background-tasks)
    - [Chế độ bảo trì](#maintenance-mode)
    - [Tạm dừng Scheduled Tasks](#pausing-scheduled-tasks)
    - [Schedule Groups](#schedule-groups)
- [Chạy Scheduler](#running-the-scheduler)
    - [Scheduled Task theo giây](#sub-minute-scheduled-tasks)
    - [Chạy Scheduler local](#running-the-scheduler-locally)
- [Task Output](#task-output)
- [Task Hook](#task-hooks)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Trong quá khứ, có thể bạn đã viết một cấu hình Cron cho mỗi task schedule mà bạn cần để server của bạn chạy. Tuy nhiên, điều này có thể nhanh chóng trở thành một vấn đề lớn, bởi vì task schedule của bạn không có trong source code nên bạn phải SSH vào server để xem từng mục cron hiện có của bạn hoặc thêm các Cron vào bằng một cách thủ công.

Lệnh schedule của Laravel cung cấp một cách tiếp cận mới để quản lý các task schedule trên máy chủ của bạn. Scheduler cho phép bạn định nghĩa một cách đơn giản và rõ ràng các lệnh đó trong chính Laravel application của bạn. Khi sử dụng schedule, chỉ cần một cron duy nhất trên server của bạn. Các task schedule của bạn sẽ được định nghĩa trong file `routes/console.php` của ứng dụng.

<a name="defining-schedules"></a>
## Định nghĩa Schedule

Bạn có thể định nghĩa tất cả các task đã được schedule của bạn trong file `routes/console.php` của application của bạn. Để bắt đầu, chúng ta hãy xem một ví dụ về schedule cho một task. Trong ví dụ này, chúng ta sẽ schedule một closure được gọi mỗi ngày vào lúc nửa đêm. Trong closure, chúng ta sẽ thực hiện một truy vấn vào cơ sở dữ liệu để xóa bảng:

```php
<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

Ngoài việc tạo schedule bằng closures, bạn cũng có thể schedule cho [các đối tượng invokable](http://php.net/manual/en/language.oop5.magic.php#object.invoke). Đối tượng invokable là các class PHP đơn giản có chứa phương thức `__invoke`:

```php
Schedule::call(new DeleteRecentUsers)->daily();
```

Nếu bạn muốn dành riêng file `routes/console.php` của bạn chỉ để định nghĩa các lệnh, bạn có thể sử dụng phương thức `withSchedule` trong file `bootstrap/app.php` của ứng dụng để định nghĩa các task được schedule của bạn. Phương thức này chấp nhận một closure nhận một instance của scheduler:

```php
use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->call(new DeleteRecentUsers)->daily();
})
```

Nếu bạn muốn xem tổng quan về các scheduled task của bạn và lần tiếp theo chúng được chạy, bạn có thể sử dụng lệnh Artisan `schedule:list`:

```shell
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### Các lệnh Artisan Schedule

Ngoài việc tạo schedule cho closures, bạn cũng có thể tạo schedule cho [Lệnh Artisan](/docs/{{version}}/artisan) và các lệnh của hệ điều hành. Ví dụ, bạn có thể sử dụng phương thức `command` để tạo schedule cho lệnh Artisan bằng cách sử dụng tên hoặc class của command đó.

Khi lên lịch cho các lệnh Artisan bằng cách sử dụng tên class của lệnh, bạn có thể truyền thêm một mảng các tham số command-line cần được cung cấp cho lệnh khi nó được gọi:

```php
use App\Console\Commands\SendEmailsCommand;
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send Taylor --force')->daily();

Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

<a name="scheduling-artisan-closure-commands"></a>
#### Scheduling Artisan Closure Commands

Nếu bạn muốn schedule một lệnh Artisan đang được định nghĩa bởi một closure, bạn có thể nối thêm các phương thức liên quan đến việc tạo schedule sau định nghĩa của lệnh:

```php
Artisan::command('delete:recent-users', function () {
    DB::table('recent_users')->delete();
})->purpose('Delete recent users')->daily();
```

Nếu bạn muốn truyền tham số vào lệnh closure, bạn có thể cung cấp chúng cho phương thức `schedule`:

```php
Artisan::command('emails:send {user} {--force}', function ($user) {
    // ...
})->purpose('Send emails to the specified user')->schedule(['Taylor', '--force'])->daily();
```

<a name="scheduling-queued-jobs"></a>
### Schedule Queued Job

Phương thức `job` có thể được sử dụng để tạo schedule cho một [queued job](/docs/{{version}}/queues). Phương thức này cung cấp một cách thuận tiện để tạo schedule queued job mà không cần phải sử dụng phương thức `call` để định nghĩa closure cho queue job:

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new Heartbeat)->everyFiveMinutes();
```

Tùy chọn tham số thứ hai và thứ ba có thể được cung cấp cho phương thức `job` để chỉ định tên queue và kết nối của queue sẽ được sử dụng để queue job:

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

// Dispatch the job to the "heartbeats" queue on the "sqs" connection...
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

<a name="scheduling-shell-commands"></a>
### Lệnh Shell Schedule

Phương thức `exec` có thể được sử dụng để ra lệnh cho hệ điều hành:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::exec('node /home/forge/script.js')->daily();
```

<a name="schedule-frequency-options"></a>
### Tùy chọn tần suất Schedule

Chúng ta đã xem một số ví dụ về cách mà bạn có thể cấu hình task để chạy theo các khoảng thời gian nhất định. Tuy nhiên, có nhiều tần suất task schedule khác mà bạn có thể cấu hình cho một task:

<div class="overflow-auto">

| Method                             | Description                                                      |
| ---------------------------------- | ---------------------------------------------------------------- |
| `->cron('* * * * *');`             | Chạy task theo một tùy chỉnh cron schedule.                      |
| `->everySecond();`                 | Chạy task mỗi giây.                                              |
| `->everyTwoSeconds();`             | Chạy task mỗi hai giây.                                          |
| `->everyFiveSeconds();`            | Chạy task mỗi năm giây.                                          |
| `->everyTenSeconds();`             | Chạy task mỗi mười giây.                                         |
| `->everyFifteenSeconds();`         | Chạy task mỗi 15 giây.                                           |
| `->everyTwentySeconds();`          | Chạy task mỗi 20 giây.                                           |
| `->everyThirtySeconds();`          | Chạy task mỗi 30 giây.                                           |
| `->everyMinute();`                 | Chạy task mỗi phút.                                              |
| `->everyTwoMinutes();`             | Chạy task hai phút một lần.                                      |
| `->everyThreeMinutes();`           | Chạy task ba phút một lần.                                       |
| `->everyFourMinutes();`            | Chạy task bốn phút một lần.                                      |
| `->everyFiveMinutes();`            | Chạy task năm phút một lần.                                      |
| `->everyTenMinutes();`             | Chạy task mười phút một lần.                                     |
| `->everyFifteenMinutes();`         | Chạy task mười lăm phút một lần.                                 |
| `->everyThirtyMinutes();`          | Chạy task ba mươi phút một lần.                                  |
| `->hourly();`                      | Chạy task một giờ một lần.                                       |
| `->hourlyAt(17);`                  | Chạy task một giờ một lần vào phút thứ 17.                       |
| `->everyOddHour($minutes = 0);`    | Chạy task vào giờ lẻ.                                            |
| `->everyTwoHours($minutes = 0);`   | Chạy task hai giờ một lần.                                       |
| `->everyThreeHours($minutes = 0);` | Chạy task ba giờ một lần.                                        |
| `->everyFourHours($minutes = 0);`  | Chạy task bốn giờ một lần.                                       |
| `->everySixHours($minutes = 0);`   | Chạy task sáu giờ một lần.                                       |
| `->daily();`                       | Chạy task hàng ngày.                                             |
| `->dailyAt('13:00');`              | Chạy task hàng ngày vào lúc 13:00.                               |
| `->twiceDaily(1, 13);`             | Chạy task hàng ngày vào lúc 1:00 và 13:00.                       |
| `->twiceDailyAt(1, 13, 15);`       | Chạy task hàng ngày vào lúc 1:15 và 13:15.                       |
| `->daysOfMonth([1, 10, 20]);`      | Chạy task vào các ngày cụ thể trong tháng.                       |
| `->weekly();`                      | Chạy task hàng tuần vào chủ nhật lúc 00:00.                      |
| `->weeklyOn(1, '8:00');`           | Chạy task hàng tuần vào thứ hai lúc 8:00.                        |
| `->monthly();`                     | Chạy task vào ngày đầu tiên của tháng lúc 00:00.                 |
| `->monthlyOn(4, '15:00');`         | Chạy task hàng ngày vào ngày thứ 4 của tháng và vào lúc 15:00.   |
| `->twiceMonthly(1, 16, '13:00');`  | Chạy task hàng tháng vào ngày 1 và ngày 16 lúc 13h00.            |
| `->lastDayOfMonth('15:00');`       | Chạy task vào ngày cuối dùng của tháng lúc 15:00.                |
| `->quarterly();`                   | Chạy task vào ngày đầu tiên của quý lúc 00:00.                   |
| `->quarterlyOn(4, '14:00');`       | Chạy task mỗi quý vào ngày 4 lúc 14:00.                          |
| `->yearly();`                      | Chạy task vào ngày đầu tiên của năm lúc 00:00.                   |
| `->yearlyOn(6, 1, '17:00');`       | Chạy task hàng năm vào ngày 1 tháng 6 lúc 17:00.                 |
| `->timezone('America/New_York');`  | Set timezone cho task.                                           |

</div>

Các phương thức này có thể được kết hợp thêm các ràng buộc để tạo ra các schedule có thể được điều chỉnh tốt hơn, ví dụ như chỉ chạy vào một số ngày nhất định trong tuần. Ví dụ, bạn có thể schedule một lệnh chạy vào thứ hai hàng tuần thì bạn có thể làm như sau:

```php
use Illuminate\Support\Facades\Schedule;

// Run once per week on Monday at 1 PM...
Schedule::call(function () {
    // ...
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
Schedule::command('foo')
    ->weekdays()
    ->hourly()
    ->timezone('America/Chicago')
    ->between('8:00', '17:00');
```

Dưới đây là một danh sách các ràng buộc schedule có thể được thêm:

<div class="overflow-auto">

| Method                                   | Description                                                |
| ---------------------------------------- | ---------------------------------------------------------- |
| `->weekdays();`                          | Giới hạn task chỉ chạy vào các ngày trong tuần.            |
| `->weekends();`                          | Giới hạn task chỉ chạy vào cuối tuần.                      |
| `->sundays();`                           | Giới hạn task chỉ chạy vào chủ nhật.                       |
| `->mondays();`                           | Giới hạn task chỉ chạy vào thứ hai.                        |
| `->tuesdays();`                          | Giới hạn task chỉ chạy vào thứ ba.                         |
| `->wednesdays();`                        | Giới hạn task chỉ chạy vào thứ tư.                         |
| `->thursdays();`                         | Giới hạn task chỉ chạy vào thứ năm.                        |
| `->fridays();`                           | Giới hạn task chỉ chạy vào thứ sáu.                        |
| `->saturdays();`                         | Giới hạn task chỉ chạy vào thứ bảy.                        |
| `->days(array\|mixed);`                  | Giới hạn task chỉ chạy vào ngày cụ thể.                    |
| `->between($startTime, $endTime);`       | Giới hạn task chỉ chạy vào giữa thời gian start và end.    |
| `->unlessBetween($startTime, $endTime);` | Giới hạn task không chạy vào giữa thời gian start và end.  |
| `->when(Closure);`                       | Giới hạn task chỉ chạy trên một điều kiện đúng.            |
| `->environments($env);`                  | Giới hạn task trong các môi trường cụ thể.                 |

</div>

<a name="day-constraints"></a>
#### Day Constraints

Phương thức `days` có thể được sử dụng để giới hạn việc thực hiện một task trong những ngày cụ thể có trong một tuần. Ví dụ: bạn có thể tạo lịch chạy một lệnh mỗi giờ vào ngày chủ nhật và thứ tư:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->hourly()
    ->days([0, 3]);
```

Ngoài ra, bạn có thể sử dụng các hằng số có sẵn trên class `Illuminate\Console\Scheduling\Schedule` khi định nghĩa ngày mà một task sẽ chạy:

```php
use Illuminate\Support\Facades;
use Illuminate\Console\Scheduling\Schedule;

Facades\Schedule::command('emails:send')
    ->hourly()
    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);
```

<a name="between-time-constraints"></a>
#### Between Time Constraints

Phương thức `between` có thể được sử dụng để giới hạn việc thực thi của một task dựa vào một khoảng thời gian có trong ngày:

```php
Schedule::command('emails:send')
    ->hourly()
    ->between('7:00', '22:00');
```

Tương tự, phương thức `unlessBetween` có thể được sử dụng để chạy ngược lại việc thực thi của một task trong một khoảng thời gian:

```php
Schedule::command('emails:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
```

<a name="truth-test-constraints"></a>
#### Truth Test Constraints

Phương thức `when` có thể được sử dụng để hạn chế việc thực hiện một task dựa trên kết quả của một điều kiện nhất định. Nói cách khác, nếu closure đã cho trả về giá trị `true`, tác vụ sẽ thực thi, miễn là không có điều kiện ràng buộc nào khác ngăn task đó chạy:

```php
Schedule::command('emails:send')->daily()->when(function () {
    return true;
});
```

Phương thức `skip` có thể được xem là ngược lại với phương thức `when`. Nếu phương thức `skip` trả về giá trị `true`, scheduled task đó sẽ không được thực thi:

```php
Schedule::command('emails:send')->daily()->skip(function () {
    return true;
});
```

Khi sử dụng kết hợp nhiều phương thức `when`, lệnh đã được schedule sẽ chỉ được thực thi nếu tất cả các điều kiện `when` đều trả về giá trị `true`.

<a name="environment-constraints"></a>
#### Environment Constraints

Phương thức `environments` có thể được sử dụng để chỉ thực thi các task trong các môi trường đã cho (được định nghĩa bởi [biến môi trường](/docs/{{version}}/configuration#environment-configuration) `APP_ENV` ):

```php
Schedule::command('emails:send')
    ->daily()
    ->environments(['staging', 'production']);
```

<a name="timezones"></a>
### Timezones

Sử dụng phương thức `timezone`, bạn có thể chỉ định thời gian của một task schedule sẽ được sử dụng trong một timezone nhất định:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00')
```

Nếu bạn đang muốn chỉ định liên tục một múi giờ cho tất cả các task schedule của bạn, bạn có thể chỉ định timezone nào sẽ được gán cho tất cả các schedule bằng cách định nghĩa tùy chọn `schedule_timezone` trong file cấu hình `app` của ứng dụng của bạn:

```php
'timezone' => 'UTC',

'schedule_timezone' => 'America/Chicago',
```

> [!WARNING]
> Hãy nhớ rằng một số timezone sử dụng quy ước giờ mùa hè. Khi các thay đổi về quy ước giờ mùa hè xảy ra, schedule task của bạn có thể chạy hai lần hoặc thậm chí là hoàn toàn không chạy. Vì lý do này, chúng tôi khuyên bạn nên tránh tạo schedule timezone khi có thể.

<a name="preventing-task-overlaps"></a>
### Ngăn task chồng nhau

Mặc định, các task đã được schedule sẽ được chạy, ngay cả khi instance trước đó của task vẫn đang được chạy. Để ngăn điều này xảy ra, bạn có thể sử dụng phương thức `withoutOverlapping`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->withoutOverlapping();
```

Ở ví dụ trên, [Lệnh Artisan](/docs/{{version}}/artisan) `emails:send` sẽ được chạy mỗi phút nếu nó chưa được chạy. Phương thức `withoutOverlapping` đặc biệt hữu ích nếu bạn có một task phức tạp cần nhiều thời gian để thực hiện, và bạn không thể dự đoán chính xác task đó sẽ mất bao nhiêu thời gian để hoàn thành.

Nếu cần, bạn có thể chỉ định số phút mà sau khi task được thực hiện thì khóa "chống lặp" hết hạn. Mặc định, khóa này sẽ hết hạn sau 24 giờ:

```php
Schedule::command('emails:send')->withoutOverlapping(10);
```

Ở hậu trường, phương thức `withoutOverlapping` sẽ sử dụng [cache](/docs/{{version}}/cache) của ứng dụng của bạn để lấy khóa. Nếu cần, bạn có thể xóa các khóa cache này bằng lệnh Artisan `schedule:clear-cache`. Điều này thường chỉ cần thiết nếu một task bị kẹt do sự cố bất ngờ từ máy chủ.

<a name="running-tasks-on-one-server"></a>
### Chạy task trên một server

> [!WARNING]
> Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng driver cache `database`, `memcached` `dynamodb`, hoặc `redis` làm driver cache mặc định của ứng dụng của bạn. Ngoài ra, tất cả các server phải được giao tiếp với cùng một server cache trung tâm.

Nếu schedule của ứng dụng của bạn đang chạy trên nhiều server, bạn có thể giới hạn schedule job chỉ được chạy trên một server duy nhất. Ví dụ: giả sử bạn đang có một task schedule là tạo một báo cáo vào mỗi tối thứ Sáu. Nếu schedule của bạn đang chạy trên ba server worker, thì task schedule sẽ được chạy trên cả ba server và tạo báo cáo ba lần. Không tốt!

Để yêu cầu task chỉ được chạy trên một server, hãy sử dụng phương thức `onOneServer` khi bạn định nghĩa task schedule. Server đầu tiên nhận được task sẽ dùng một atomic lock trên job đó để ngăn các server khác chạy cùng một task vào cùng một thời điểm:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```

Bạn có thể sử dụng phương thức `useCache` để tùy chỉnh cache store sẽ được scheduler sử dụng để lấy các atomic lock cần thiết cho các task chạy trên một server duy nhất:

```php
Schedule::useCache('database');
```

<a name="naming-unique-jobs"></a>
#### Naming Single Server Jobs

Thỉnh thoảng bạn có thể cần schedule cùng một job nhưng gửi đi với các tham số khác nhau, trong khi vẫn hướng dẫn Laravel chạy hoán vị từng job trên một máy chủ duy nhất. Để thực hiện điều này, bạn có thể gán cho mỗi định nghĩa schedule một tên duy nhất thông qua phương thức `name`:

```php
Schedule::job(new CheckUptime('https://laravel.com'))
    ->name('check_uptime:laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();

Schedule::job(new CheckUptime('https://vapor.laravel.com'))
    ->name('check_uptime:vapor.laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();
```

Tương tự như vậy, các scheduled closure cũng phải được đặt tên nếu chúng có dự định chạy trên một máy chủ:

```php
Schedule::call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

<a name="background-tasks"></a>
### Background Tasks

Mặc định, nhiều task được schedule vào cùng một thời gian sẽ phải chạy theo tuần tự dựa trên thứ tự mà chúng được định nghĩa trong phương thức `schedule` của bạn. Nếu bạn có những task cần phải chạy dài, thì điều này có thể khiến các task tiếp theo sẽ chạy muộn hơn so với dự kiến. Nếu bạn muốn chạy các task trong background để tất cả chúng có thể được chạy đồng thời, thì bạn có thể sử dụng phương thức `runInBackground`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('analytics:report')
    ->daily()
    ->runInBackground();
```

> [!WARNING]
> Phương thức `runInBackground` chỉ có thể được sử dụng khi task được tạo thông qua phương thức `command` và `exec`.

<a name="maintenance-mode"></a>
### Chế độ bảo trì

Các scheduled task của application của bạn sẽ không được chạy khi application của bạn ở [trong chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode), vì chúng tôi không muốn các task của bạn gây trở ngại với bất kỳ bảo trì nào mà bạn đang thực hiện trên server chưa được hoàn thành. Tuy nhiên, nếu bạn muốn task chạy ngay cả trong chế độ bảo trì, bạn có thể gọi phương thức `evenInMaintenanceMode` khi định nghĩa task:

```php
Schedule::command('emails:send')->evenInMaintenanceMode();
```

<a name="pausing-scheduled-tasks"></a>
### Tạm dừng Scheduled Tasks

Bạn có thể tạm dừng việc xử lý các scheduled task mà không cần thay đổi source code của bạn bằng cách sử dụng lệnh Artisan `schedule:pause`:

```shell
php artisan schedule:pause
```

Trong khi scheduler đang bị tạm dừng, sẽ không có task nào được thực thi. Bạn có thể tiếp tục việc xử lý các task bằng cách sử dụng lệnh `schedule:continue`:

```shell
php artisan schedule:continue
```

Nếu một task vẫn cần được chạy trong khi scheduler đang bị tạm dừng, bạn có thể đánh dấu nó bằng phương thức `evenWhenPaused`:

```php
Schedule::command('emails:send')->evenWhenPaused();
```

<a name="schedule-groups"></a>
### Schedule Groups

Khi định nghĩa nhiều scheduled task với các cấu hình tương tự, bạn có thể sử dụng tính năng nhóm task của Laravel để tránh lặp lại các cài đặt giống nhau cho mỗi task. Việc nhóm các task này giúp đơn giản hóa code của bạn và đảm bảo tính nhất quán giữa các task liên quan.

Để tạo một nhóm các scheduled task, hãy gọi các phương thức cấu hình task mà bạn mong muốn, sau đó là phương thức `group`. Phương thức `group` chấp nhận một closure chịu trách nhiệm định nghĩa các task chia sẻ cấu hình đã chỉ định:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::daily()
    ->onOneServer()
    ->timezone('America/New_York')
    ->group(function () {
        Schedule::command('emails:send --force');
        Schedule::command('emails:prune');
    });
```

<a name="running-the-scheduler"></a>
## Chạy Scheduler

Hiện tại, chúng ta đã học cách định nghĩa các scheduled task, hãy thảo luận về cách thực sự chạy chúng trên máy chủ của chúng ta. Lệnh `schedule:run` Artisan sẽ đánh giá tất cả các scheduled task của bạn và xác định xem chúng có cần chạy hay không dựa trên thời gian hiện tại của máy chủ.

Vì vậy, khi sử dụng scheduler của Laravel, chúng ta chỉ cần thêm một mục cấu hình cron duy nhất vào máy chủ để chạy lệnh `schedule:run` mỗi phút. Nếu bạn không biết cách thêm các mục cron vào máy chủ của bạn, hãy cân nhắc sử dụng một nền tảng quản lý chẳng hạn như [Laravel Cloud](https://cloud.laravel.com), nơi có thể quản lý việc thực thi các task đã được schedule cho bạn:

```shell
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

<a name="sub-minute-scheduled-tasks"></a>
### Scheduled Task theo giây

Trên hầu hết các hệ điều hành, cron job bị giới hạn bởi số lần chạy tối đa trong mỗi phút. Tuy nhiên, scheduler của Laravel cho phép bạn schedule các tác vụ chạy trong các khoảng thời gian thường xuyên hơn, thậm chí là một lần mỗi giây:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

Khi các tác vụ chạy theo giây được định nghĩa trong ứng dụng của bạn, lệnh `schedule:run` sẽ tiếp tục chạy cho đến khi hết số phút hiện tại thay vì thoát ngay lập tức. Điều này cho phép command gọi tất cả các tác vụ chạy theo giây trong suốt số phút.

Vì các tác vụ chạy theo giây có thể mất nhiều thời gian chạy hơn dự kiến và ​​có thể làm chậm quá trình thực hiện các tác vụ chạy theo giây sau đó, nên chúng tôi khuyến khích tất cả các tác vụ chạy theo giây nên được gửi đến các queued job hoặc background command để xử lý các tác vụ đó:

```php
use App\Jobs\DeleteRecentUsers;

Schedule::job(new DeleteRecentUsers)->everyTenSeconds();

Schedule::command('users:delete')->everyTenSeconds()->runInBackground();
```

<a name="interrupting-sub-minute-tasks"></a>
#### Ngắt quãng các tác vụ theo giây

Vì lệnh `schedule:run` sẽ chạy trong toàn bộ phút khi các tác vụ chạy theo giây được định nghĩa, thỉnh thoảng bạn có thể cần ngắt lệnh khi triển khai ứng dụng của bạn. Nếu không, một instance của lệnh `schedule:run` đang chạy sẽ tiếp tục sử dụng code đã triển khai trước đó của ứng dụng cho đến khi số phút hiện tại kết thúc.

Để ngắt các lệnh gọi `schedule:run` đang chạy, bạn có thể thêm lệnh `schedule:interrupt` vào script deploy của ứng dụng. Lệnh này sẽ được gọi sau khi ứng dụng của bạn hoàn tất quá trình deploy:

```shell
php artisan schedule:interrupt
```

<a name="running-the-scheduler-locally"></a>
## Chạy Scheduler local

Thông thường, bạn sẽ không cần phải thêm mục cron của scheduler vào máy chủ phát triển local của bạn. Thay vào đó, bạn có thể sử dụng lệnh Artisan `schedule:work`. Lệnh này sẽ chạy trên giao diện người dùng và gọi scheduler mỗi phút cho đến khi bạn kết thúc lệnh. Khi các task chạy dưới một phút được định nghĩa, scheduler sẽ tiếp tục chạy theo mỗi phút để xử lý các task đó:

```shell
php artisan schedule:work
```

<a name="task-output"></a>
## Task Output

Laravel schedule cung cấp một số phương thức thuận tiện để làm việc với output được tạo ra bởi schedule task. Đầu tiên, bằng cách sử dụng phương thức `sendOutputTo`, bạn có thể gửi output tới một file để kiểm tra sau khi chạy:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo($filePath);
```

Nếu bạn muốn thêm output vào một file đã có, bạn có thể sử dụng phương thức `appendOutputTo`:

```php
Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo($filePath);
```

Sử dụng phương thức `emailOutputTo`, bạn có thể gửi email output đến một địa chỉ email mà bạn đã chọn. Trước khi gửi email output của một task, bạn nên cấu hình [e-mail services](/docs/{{version}}/mail) của Laravel:

```php
Schedule::command('report:generate')
    ->daily()
    ->sendOutputTo($filePath)
    ->emailOutputTo('taylor@example.com');
```

Nếu bạn muốn chỉ gửi email nếu scheduled Artisan hoặc system command kết thúc với exit code khác 0, hãy sử dụng phương thức `emailOutputOnFailure`:

```php
Schedule::command('report:generate')
    ->daily()
    ->emailOutputOnFailure('taylor@example.com');
```

> [!WARNING]
> Các phương thức `emailOutputTo`, `emailOutputOnFailure`, `sendOutputTo` và `appendOutputTo` sẽ chỉ được dùng với phương thức `command` và phương thức `exec`.

<a name="task-hooks"></a>
## Task Hook

Sử dụng các phương thức `before` và `after`, bạn có thể khai báo các code mà sẽ được thực thi trước và sau khi scheduled task được chạy:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->before(function () {
        // The task is about to execute...
    })
    ->after(function () {
        // The task has executed...
    });
```

Phương thức `onSuccess` và `onFailure` cho phép bạn chỉ định một đoạn code cụ thể sẽ được thực thi nếu scheduled task chạy thành công hoặc bị thất bại. Một thất bại sẽ xảy ra khi lệnh Artisan hoặc system command kết thúc với exit code khác 0:

```php
Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function () {
        // The task succeeded...
    })
    ->onFailure(function () {
        // The task failed...
    });
```

Nếu lệnh của bạn trả về output, bạn có thể truy cập vào output trong hook `after`, `onSuccess` hoặc `onFailure` bằng cách khai báo một instance `Illuminate\Support\Stringable` làm tham số `$output` trong khi định nghĩa closure của hook:

```php
use Illuminate\Support\Stringable;

Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function (Stringable $output) {
        // The task succeeded...
    })
    ->onFailure(function (Stringable $output) {
        // The task failed...
    });
```

<a name="pinging-urls"></a>
#### Pinging URLs

Sử dụng các phương thức `pingBefore` và `thenPing`, schedule có thể tự động ping đến một URL đã cho trước hoặc sau khi task được chạy. Phương thức này sẽ hữu ích để thông báo cho một dịch vụ bên ngoài, chẳng hạn như [Laravel Envoyer](https://envoyer.io), rằng scheduled task của bạn đang bắt đầu hoặc đã kết thúc chạy:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBefore($url)
    ->thenPing($url);
```

Phương thức `pingOnSuccess` và `pingOnFailure` có thể được sử dụng để ping đến một URL nhất định nếu task chạy thành công hoặc bị thất bại. Một thất bại sẽ xảy ra khi lệnh Artisan hoặc system command kết thúc với exit code khác 0:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccess($successUrl)
    ->pingOnFailure($failureUrl);
```

Các phương thức `pingBeforeIf`,`thenPingIf`,`pingOnSuccessIf`, và `pingOnFailureIf` chỉ có thể được sử dụng để ping đến một URL nếu một điều kiện đưa vào là `true`:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBeforeIf($condition, $url)
    ->thenPingIf($condition, $url);

Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccessIf($condition, $successUrl)
    ->pingOnFailureIf($condition, $failureUrl);
```

<a name="events"></a>
## Events

Laravel gửi đi nhiều [event](/docs/{{version}}/events) khác nhau trong quá trình scheduling. Bạn có thể [định nghĩa các listener](/docs/{{version}}/events) cho bất kỳ event nào sau đây:

<div class="overflow-auto">

| Event Name                                                  |
| ----------------------------------------------------------- |
| `Illuminate\Console\Events\ScheduledTaskStarting`           |
| `Illuminate\Console\Events\ScheduledTaskFinished`           |
| `Illuminate\Console\Events\ScheduledBackgroundTaskFinished` |
| `Illuminate\Console\Events\ScheduledTaskSkipped`            |
| `Illuminate\Console\Events\ScheduledTaskFailed`             |

</div>
