# Laravel Horizon

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cấu hình](#configuration)
    - [Authorization vào bảng điều khiển](#dashboard-authorization)
    - [Max Job Attempts](#max-job-attempts)
    - [Job Timeout](#job-timeout)
    - [Job Backoff](#job-backoff)
    - [Silenced Jobs](#silenced-jobs)
- [Balancing Strategies](#balancing-strategies)
    - [Auto Balancing](#auto-balancing)
    - [Simple Balancing](#simple-balancing)
    - [No Balancing](#no-balancing)
- [Cập nhật Horizon](#upgrading)
- [Chạy Horizon](#running-horizon)
    - [Deploy Horizon](#deploying-horizon)
- [Tags](#tags)
- [Thông báo](#notifications)
- [Số liệu](#metrics)
- [Xoá job thất bại](#deleting-failed-jobs)
- [Xoá job từ queue](#clearing-jobs-from-queues)

<a name="introduction"></a>
## Giới thiệu

> [!NOTE]
> Trước khi tìm hiểu về Laravel Horizon, bạn nên tìm hiểu qua [queue services](/docs/{{version}}/queues) trong Laravel. Horizon hỗ trợ queue của Laravel với các tính năng mới nên có thể gây nhầm lẫn nếu bạn chưa quen với các tính năng cơ bản của queue do Laravel cung cấp.

[Laravel Horizon](https://github.com/laravel/horizon) cung cấp một bảng điều khiển đẹp mắt và cấu hình code-driven cho [queue Redis](/docs/{{version}}/queues) được hỗ trợ bởi Laravel. Horizon cho phép bạn dễ dàng theo dõi các số liệu chính của hệ thống queue của bạn như job được thông qua, thời gian chạy hoặc job bị thất bại.

Khi sử dụng Horizon, tất cả các cấu hình queue worker của bạn được lưu trong một file cấu hình đơn giản. Bằng cách định nghĩa cấu hình worker của ứng dụng của bạn trong file được version controll, bạn có thể dễ dàng mở rộng quy mô hoặc sửa chữa queue worker của ứng dụng khi triển khai ứng dụng của bạn.

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## Cài đặt

> [!WARNING]
> Laravel Horizon yêu cầu bạn sử dụng [Redis](https://redis.io) để hỗ trợ queue của bạn. Vì thế, bạn nên đảm bảo rằng queue connection của bạn đã được set thành `redis` trong file cấu hình `config/queue.php` trong apllication của bạn. Hiện tại, horizon không tương thích với Redis Cluster.

Bạn có thể sử dụng Composer để cài đặt Horizon vào project Laravel của bạn:

```shell
composer require laravel/horizon
```

Sau khi cài đặt Horizon, hãy export asset của nó bằng lệnh Artisan `horizon:install`:

```shell
php artisan horizon:install
```

<a name="configuration"></a>
### Cấu hình

Sau khi export asset của Horizon xong, file cấu hình của nó sẽ được lưu tại `config/horizon.php`. File cấu hình này cho phép bạn cài đặt các tùy chọn cho queue worker cho application của bạn. Mỗi tùy chọn cài đặt này đều có chứa phần mô tả về mục đích của nó, vì vậy bạn hãy đọc kỹ file này.

> [!WARNING]
> Horizon sử dụng kết nối Redis có tên là `horizon` trong nội bộ. Tên kết nối Redis này được đặt tên trước và không được gán cho bất kỳ kết nối Redis nào khác trong file cấu hình `database.php` hoặc làm giá trị của tùy chọn `use` trong file cấu hình `horizon.php`.

<a name="environments"></a>
#### Environments

Sau khi cài đặt, tùy chọn cấu hình Horizon chính mà bạn nên xem là tùy chọn cấu hình `environments`. Tùy chọn cấu hình này là một mảng các môi trường mà ứng dụng của bạn có thể chạy trên đó và ngoài ra, nó còn định nghĩa các tùy chọn worker process cho từng loại môi trường đó. Mặc định, tùy chọn này chứa môi trường `production` và `local`. Tuy nhiên, bạn có thể thoải mái thêm nhiều môi trường hơn nếu cần:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],

    'local' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

Bạn cũng có thể định nghĩa một wildcard (`*`) cho môi trường, wildcard này sẽ được sử dụng khi không tìm thấy môi trường nào phù hợp:

```php
'environments' => [
    // ...

    '*' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

Khi bạn khởi động Horizon, nó sẽ sử dụng các tùy chọn cấu hình worker process tương ứng với môi trường mà ứng dụng của bạn được chạy. Thông thường, môi trường được xác định bằng giá trị của [biến môi trường](/docs/{{version}}/configuration#determining-the-current-environment) `APP_ENV`. Ví dụ: môi trường Horizon mặc định`local` được cấu hình để bắt đầu với ba worker process và tự động cân bằng số lượng worker process được chỉ định cho mỗi queue. Môi trường `production` mặc định sẽ được cấu hình để bắt đầu tối đa 10 worker process và tự động cân bằng số lượng worker process được chỉ định cho mỗi queue.

> [!WARNING]
> Bạn nên đảm bảo tùy chọn `environments` trong file cấu hình `horizon` chứa các mục cho mỗi [environment](/docs/{{version}}/configuration#environment-configuration) mà bạn định chạy trên Horizon.

<a name="supervisors"></a>
#### Supervisors

Như bạn có thể thấy trong file cấu hình mặc định của Horizon, mỗi môi trường có thể chứa một hoặc nhiều "supervisors". Mặc định, file cấu hình định nghĩa supervisor này là `supervisor-1`; tuy nhiên, bạn có thể tự do đặt tên cho supervisor của bạn bất cứ điều gì mà bạn muốn. Mỗi supervisor về cơ bản chịu trách nhiệm "giám sát" một nhóm worker process và đảm nhiệm việc cân bằng các worker process trên các queue.

Bạn có thể thêm các supervisor vào một môi trường nhất định nếu bạn muốn định nghĩa một nhóm các worker process mới sẽ được chạy trong môi trường đó. Bạn có thể chọn thực hiện việc này nếu bạn muốn định nghĩa một chiến lược cân bằng mới hoặc một số lượng worker process nhất định cho một queue mà được ứng dụng của bạn sử dụng.

<a name="maintenance-mode"></a>
#### Maintenance Mode

Trong khi ứng dụng của bạn đang ở [chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode), các queued job sẽ không được Horizon xử lý trừ khi có tùy chọn `force` của supervisor được định nghĩa là `true` trong file cấu hình Horizon:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'force' => true,
        ],
    ],
],
```

<a name="default-values"></a>
#### Default Values

Trong file cấu hình mặc định của Horizon, bạn có thể thấy tùy chọn cấu hình `defaults`. Tùy chọn cấu hình này chỉ định các giá trị mặc định cho [supervisor](#supervisors) trong ứng dụng của bạn. Các giá trị cấu hình mặc định của supervisor sẽ được hợp nhất vào một cấu hình của supervisor cho từng môi trường cụ thể, cho phép bạn tránh lặp lại không cần thiết khi định nghĩa supervisor của bạn.

<a name="dashboard-authorization"></a>
### Authorization vào bảng điều khiển

Bảng điều khiển Horizon có thể được truy cập thông qua route `/horizon`. Mặc định, bạn sẽ chỉ có thể truy cập trang tổng quan này trong môi trường `local`. Tuy nhiên, trong file `app/Providers/HorizonServiceProvider.php` của bạn, có một định nghĩa [gate authorization](/docs/{{version}}/authorization#gates). Gate authorization này sẽ kiểm soát quyền truy cập vào Horizon trong các môi trường **không phải là local**. Bạn có thể thoải mái sửa gate này nếu cần để hạn chế quyền truy cập vào các cài đặt Horizon của bạn:

```php
/**
 * Register the Horizon gate.
 *
 * This gate determines who can access Horizon in non-local environments.
 */
protected function gate(): void
{
    Gate::define('viewHorizon', function (User $user) {
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

<a name="alternative-authentication-strategies"></a>
#### Alternative Authentication Strategies

Hãy nhớ rằng Laravel sẽ tự động đưa người dùng đã xác thực vào gate closure. Nếu ứng dụng của bạn đang cung cấp bảo mật cho Horizon thông qua một phương thức khác, chẳng hạn như hạn chế IP, thì người dùng Horizon của bạn có thể không cần "đăng nhập". Do đó, bạn sẽ cần phải thay đổi format `function (User $user)` của closure ở trên thành `function (User $user = null)` để yêu cầu Laravel không yêu cầu xác thực.

<a name="max-job-attempts"></a>
### Max Job Attempts

> [!NOTE]
> Trước khi tinh chỉnh các lựa chọn này, bạn hãy đảm bảo là bạn đã biết các [queue services](/docs/{{version}}/queues#max-job-attempts-and-timeout) mặc định của Laravel và khái niệm 'attempts'.

Bạn có thể định nghĩa số lần thử tối đa mà một job có thể thực hiện trong cấu hình của supervisor:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'tries' => 10,
        ],
    ],
],
```

> [!NOTE]
> Tùy chọn này tương tự như tùy chọn `--tries` khi sử dụng lệnh Artisan để xử lý các queue.

Việc điều chỉnh tùy chọn `tries` là rất quan trọng khi sử dụng các middleware như `WithoutOverlapping` hoặc `RateLimited` vì chúng sẽ thực hiện các lần thử. Để xử lý việc này, hãy điều chỉnh giá trị cấu hình `tries` ở cấp độ supervisor hoặc bằng cách định nghĩa thuộc tính `$tries` trong class job.

Nếu bạn không thiết lập tùy chọn `tries`, Horizon sẽ mặc định là thử một lần duy nhất, trừ khi class job có định nghĩa thuộc tính `$tries`, thuộc tính này sẽ được ưu tiên hơn cấu hình của Horizon.

Việc thiết lập `tries` hoặc `$tries` bằng 0 sẽ cho phép thực hiện số lần thử không giới hạn, điều này rất lý tưởng khi số lượng lần thử là không xác định. Để ngăn chặn các lỗi xảy ra vô hạn, bạn có thể giới hạn số lượng exception được phép xảy ra bằng cách thiết lập thuộc tính `$maxExceptions` trong class job.

<a name="job-timeout"></a>
### Job Timeout

Tương tự, bạn có thể thiết lập giá trị `timeout` ở cấp độ supervisor, giá trị này xác định số giây mà một worker process có thể chạy một job trước khi nó bị buộc phải dừng lại. Sau khi bị dừng, job sẽ được thử lại hoặc bị đánh dấu là thất bại, tùy thuộc vào cấu hình queue của bạn:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...¨
            'timeout' => 60,
        ],
    ],
],
```

> [!WARNING]
Khi sử dụng chiến lược balance `auto`, Horizon sẽ coi các worker đang trong quá trình xử lý là bị "giữ" và sẽ buộc chúng dừng lại sau khi hết thời gian timeout của Horizon trong quá trình scale down. Luôn đảm bảo rằng thời gian timeout của Horizon luôn lớn hơn bất kỳ thời gian timeout nào ở cấp độ job, nếu không các job có thể bị chấm dứt khi đang thực hiện. Ngoài ra, giá trị `timeout` nên luôn ngắn hơn ít nhất vài giây so với giá trị `retry_after` được định nghĩa trong file cấu hình `config/queue.php` của bạn. Nếu không, các job có thể bị xử lý hai lần.

<a name="job-backoff"></a>
### Job Backoff

Bạn có thể định nghĩa giá trị `backoff` ở cấp độ supervisor để xác định thời gian Horizon nên đợi trước khi thử lại một job gặp phải exception:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'backoff' => 10,
        ],
    ],
],
```

Bạn cũng có thể cấu hình "exponential" backoff bằng cách sử dụng một mảng cho giá trị `backoff`. Trong ví dụ này, thời gian chờ thử lại sẽ là 1 giây cho lần thử lại đầu tiên, 5 giây cho lần thử lại thứ hai, 10 giây cho lần thử lại thứ ba và 10 giây cho các lần thử lại tiếp theo nếu vẫn còn số lần thử:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'backoff' => [1, 5, 10],
        ],
    ],
],
```

<a name="silenced-jobs"></a>
### Silenced Jobs

Thỉnh thoảng, bạn có thể không quan tâm đến việc xem một số job nhất định được gửi bởi ứng dụng của bạn hoặc các package của bên thứ ba. Thay vì những job này sẽ chiếm không gian trong danh sách "job đã hoàn thành" của bạn, bạn có thể tắt chúng. Để bắt đầu, hãy thêm tên class của job vào tùy chọn cấu hình `silenced` trong file cấu hình `horizon` của ứng dụng của bạn:

```php
'silenced' => [
    App\Jobs\ProcessPodcast::class,
],
```

Ngoài việc tắt một class job riêng lẻ, Horizon cũng hỗ trợ tắt các job dựa trên [tag](#tags). Điều này có thể hữu ích nếu bạn muốn ẩn nhiều job có chung một tag:

```php
'silenced_tags' => [
    'notifications'
],
```

Ngoài ra, job mà bạn muốn tắt có thể implement interface `Laravel\Horizon\Contracts\Silenced`. Nếu một job mà đã implement interface này, nó sẽ tự động ở chế độ im lặng, ngay cả khi nó không có trong mảng cấu hình `silenced`:

```php
use Laravel\Horizon\Contracts\Silenced;

class ProcessPodcast implements ShouldQueue, Silenced
{
    use Queueable;

    // ...
}
```

<a name="balancing-strategies"></a>
## Balancing Strategies

Mỗi supervisor có thể xử lý một hoặc nhiều queue, nhưng không giống như hệ thống queue mặc định của Laravel, Horizon cho phép bạn lựa chọn một trong ba chiến lược cân bằng worker: `auto`, `simple`, và `false`.

<a name="auto-balancing"></a>
### Auto Balancing

Chiến lược `auto`, là chiến lược mặc định, sẽ điều chỉnh số lượng worker process cho mỗi queue dựa trên khối lượng công việc hiện tại của queue đó. Ví dụ: nếu queue `notifications` của bạn có 1.000 job đang chờ xử lý trong khi queue `default` của bạn đang trống, Horizon sẽ phân bổ thêm worker cho queue `notifications` cho đến khi queue này hết.

Khi sử dụng chiến lược `auto`, bạn cũng có thể cấu hình các tùy chọn `minProcesses` và `maxProcesses`:

<div class="content-list" markdown="1">

- `minProcesses` sẽ định nghĩa số lượng worker process tối thiểu cho mỗi queue. Giá trị này phải lớn hơn hoặc bằng 1.
- `maxProcesses` sẽ định nghĩa tổng số lượng worker process tối đa mà Horizon có thể mở rộng trên tất cả các queue. Giá trị này thường phải lớn hơn số lượng queue nhân với giá trị `minProcesses`. Để ngăn supervisor tạo ra bất kỳ process nào, bạn có thể set giá trị này bằng 0.

</div>

Ví dụ, bạn có thể cấu hình Horizon để duy trì ít nhất một process cho mỗi queue và mở rộng lên tổng cộng tối đa 10 worker process:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default', 'notifications'],
            'balance' => 'auto',
            'autoScalingStrategy' => 'time',
            'minProcesses' => 1,
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],
],
```

Tùy chọn cấu hình `autoScalingStrategy` sẽ xác định cách Horizon sẽ phân bổ thêm các worker process cho các queue. Bạn có thể chọn giữa hai chiến lược:

<div class="content-list" markdown="1">

- Chiến lược `time` sẽ phân bổ worker dựa trên tổng thời gian ước tính cần thiết để xử lý hết queue.
- Chiến lược `size` sẽ phân bổ worker dựa trên tổng số lượng job có trong queue.

</div>

Các giá trị cấu hình `balanceMaxShift` và `balanceCooldown` sẽ xác định tốc độ Horizon sẽ điều chỉnh scale như thế nào để đáp ứng nhu cầu worker. Trong ví dụ trên, tối đa một process mới sẽ được tạo ra hoặc bị hủy bỏ sau mỗi ba giây. Bạn có thể tự do tinh chỉnh các giá trị này khi cần thiết dựa trên nhu cầu của ứng dụng.

<a name="auto-queue-priorities"></a>
#### Queue Priorities and Auto Balancing

Khi sử dụng chiến lược balance `auto`, Horizon không áp đặt mức độ ưu tiên giữa các queue. Thứ tự của các queue trong cấu hình của supervisor không ảnh hưởng đến cách các worker process được phân bổ. Thay vào đó, Horizon dựa vào `autoScalingStrategy` đã chọn để phân bổ động các worker process dựa trên tải của queue.

Ví dụ, trong cấu hình sau, queue `high` không được ưu tiên hơn queue `default`, mặc dù nó xuất hiện đầu tiên trong danh sách:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['high', 'default'],
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

Nếu bạn cần set độ ưu tiên giữa các queue, bạn có thể định nghĩa nhiều supervisor và phân bổ tài nguyên xử lý một cách rõ ràng:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default'],
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
        'supervisor-2' => [
            // ...
            'queue' => ['images'],
            'minProcesses' => 1,
            'maxProcesses' => 1,
        ],
    ],
],
```

Trong ví dụ này, queue `default` có thể mở rộng lên đến 10 process, trong khi queue `images` bị giới hạn ở một process. Cấu hình này đảm bảo rằng các queue của bạn có thể mở rộng một cách độc lập.

> [!NOTE]
> Khi gửi các job tiêu tốn nhiều tài nguyên, thỉnh thoảng cách tốt nhất là chỉ định chúng vào một queue riêng biệt với giá trị `maxProcesses` được giới hạn. Nếu không, các job này có thể tiêu thụ quá nhiều tài nguyên CPU và làm hệ thống của bạn bị quá tải.

<a name="simple-balancing"></a>
### Simple Balancing

Chiến lược `simple` sẽ phân bổ các worker process đồng đều giữa các queue được chỉ định. Với chiến lược này, Horizon không tự động scale số lượng worker process. Thay vào đó, nó sử dụng một số lượng process cố định:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default', 'notifications'],
            'balance' => 'simple',
            'processes' => 10,
        ],
    ],
],
```

Trong ví dụ trên, Horizon sẽ chỉ định 5 process cho mỗi queue và chia đều tổng số 10 process.

Nếu bạn muốn kiểm soát số lượng worker process được chỉ định cho từng queue theo một cách riêng biệt, bạn có thể định nghĩa nhiều supervisor:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default'],
            'balance' => 'simple',
            'processes' => 10,
        ],
        'supervisor-notifications' => [
            // ...
            'queue' => ['notifications'],
            'balance' => 'simple',
            'processes' => 2,
        ],
    ],
],
```

Với cấu hình này, Horizon sẽ chỉ định 10 process cho queue `default` và 2 process cho queue `notifications`.

<a name="no-balancing"></a>
### No Balancing

Khi tùy chọn `balance` được thiết lập thành `false`, Horizon sẽ xử lý các queue theo đúng thứ tự mà chúng được liệt kê, tương tự như hệ thống queue mặc định của Laravel. Tuy nhiên, nó vẫn sẽ điều chỉnh số lượng worker process nếu các job bắt đầu tích tụ:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default', 'notifications'],
            'balance' => false,
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

Trong ví dụ trên, các job trong queue `default` sẽ luôn được ưu tiên hơn các job trong queue `notifications`. Ví dụ, nếu có 1.000 job trong queue `default` và chỉ có 10 job trong queue `notifications`, Horizon sẽ xử lý hết toàn bộ các job có trong queue `default` trước khi xử lý bất kỳ job nào có trong queue `notifications`.

Bạn có thể kiểm soát khả năng scale các worker process của Horizon bằng cách sử dụng các tùy chọn `minProcesses` và `maxProcesses`:

<div class="content-list" markdown="1">

- `minProcesses` xác định số lượng worker process tối thiểu có trong tổng số worker process. Giá trị này phải lớn hơn hoặc bằng 1.
- `maxProcesses` xác định tổng số lượng worker process tối đa mà Horizon có thể scale lên.

</div>

<a name="upgrading"></a>
#### Cập nhật Horizon

Khi nâng cấp lên phiên bản mới của Horizon, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/horizon/blob/master/UPGRADE.md).

<a name="running-horizon"></a>
## Chạy Horizon

Khi bạn đã cài đặt worker và supervisor của bạn vào trong file cấu hình `config/horizon.php` trong application của bạn, bạn có thể bắt đầu Horizon bằng cách chạy lệnh Artisan `horizon`. Lệnh này sẽ chạy tất cả các worker processes đã được cài đặt cho môi trường hiện tại:

```shell
php artisan horizon
```

Bạn có thể dừng process của Horizon và bảo nó tiếp tục xử lý các job bằng cách sử dụng các lệnh Artisan `horizon:pause` và `horizon:continue`:

```shell
php artisan horizon:pause

php artisan horizon:continue
```

Bạn cũng có thể tạm dừng và tiếp tục [supervisor](#supervisors) của Horizon cụ thể bằng cách sử dụng các lệnh Artisan `horizon:pause-supervisor` và `horizon:continue-supervisor`:

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1
```

Bạn có thể kiểm tra trạng thái hiện tại của process Horizon bằng cách sử dụng lệnh Artisan `horizon:status`:

```shell
php artisan horizon:status
```

Bạn có thể kiểm tra trạng thái hiện tại của một [supervisor](#supervisors) Horizon bằng cách sử dụng lệnh Artisan `horizon:supervisor-status`:

```shell
php artisan horizon:supervisor-status supervisor-1
```

Bạn có thể huỷ process Horizon bằng lệnh Artisan `horizon:terminate`. Tất cả các job đang được xử lý sẽ được hoàn thành và sau đó Horizon sẽ dừng thực hiện:

```shell
php artisan horizon:terminate
```

<a name="automatically-restarting-horizon"></a>
#### Automatically Restarting Horizon

Trong quá trình phát triển ở local, bạn có thể chạy lệnh `horizon:listen`. Khi sử dụng lệnh `horizon:listen`, bạn không cần phải khởi động lại Horizon một cách thủ công mỗi khi muốn load lại code đã cập nhật của mình. Trước khi sử dụng tính năng này, bạn nên đảm bảo là [Node](https://nodejs.org) đã được cài đặt trong môi trường phát triển local của bạn. Ngoài ra, bạn cũng nên cài đặt thư viện theo dõi file [Chokidar](https://github.com/paulmillr/chokidar) vào trong dự án của bạn:

```shell
npm install --save-dev chokidar
```

Sau khi Chokidar được cài đặt, bạn có thể bắt đầu Horizon bằng lệnh `horizon:listen`:

```shell
php artisan horizon:listen
```

Khi chạy trong Docker hoặc Vagrant, bạn nên sử dụng tùy chọn `--poll`:

```shell
php artisan horizon:listen --poll
```

Bạn có thể cấu hình các thư mục và file sẽ được theo dõi bằng cách sử dụng tùy chọn cấu hình `watch` trong file cấu hình `config/horizon.php` của ứng dụng:

```php
'watch' => [
    'app',
    'bootstrap',
    'config',
    'database',
    'public/**/*.php',
    'resources/**/*.php',
    'routes',
    'composer.lock',
    '.env',
],
```

<a name="deploying-horizon"></a>
### Deploy Horizon

Khi bạn đã sẵn sàng deploy Horizon tới máy chủ thực tế của ứng dụng của bạn, bạn nên cài đặt một process giám sát để theo dõi lệnh `php artisan horizon` và khởi động lại nếu nó bị thoát bất ngờ.

Trong quá trình deploy ứng dụng của bạn, bạn nên bảo process Horizon dừng lại để nó có thể được khởi động lại bởi process giám sát của bạn và nhận được các thay đổi của code của bạn.

```shell
php artisan horizon:terminate
```

<a name="installing-supervisor"></a>
#### Installing Supervisor

Supervisor là một process giám sát cho hệ điều hành Linux và sẽ tự động khởi động lại các process `horizon` nếu process đó bị ngừng chạy. Để cài đặt Supervisor trên Ubuntu, bạn có thể sử dụng lệnh sau. Nếu không sử dụng Ubuntu, bạn có thể cài đặt Supervisor bằng package manager của hệ điều hành:

```shell
sudo apt-get install supervisor
```

> [!NOTE]
> Nếu bạn không muốn tự cấu hình Supervisor, hãy xem xét việc sử dụng [Laravel Cloud](https://cloud.laravel.com), công cụ này có thể quản lý các process background cho ứng dụng Laravel của bạn.

<a name="supervisor-configuration"></a>
#### Supervisor Configuration

Các file cấu hình Supervisor thường được lưu trong thư mục `/etc/supervisor/conf.d` của server của bạn. Trong thư mục này, bạn có thể tạo bất kỳ file cấu hình nào để hướng dẫn supervisor cách giám sát các process của bạn. Ví dụ: hãy tạo một file `horizon.conf` để khởi động và theo dõi process `horizon`:

```ini
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

Khi định nghĩa cấu hình supervisor, bạn phải đảm bảo rằng giá trị của `stopwaitsecs` sẽ lớn hơn số giây mà job chạy lâu nhất của bạn sử dụng. Nếu không, supervisor có thể hủy job đó trước khi nó được xử lý xong.

> [!WARNING]
> Bạn nên chắc chắn rằng giá trị của `stopwaitsecs` sẽ luôn lớn hơn số giây lâu nhất mà job của bạn đang chạy. Nếu không, Supervisor có thể kết thúc job đó trước khi nó được xử lý xong.

<a name="starting-supervisor"></a>
#### Starting Supervisor

Khi file cấu hình đã được tạo xong, bạn có thể cập nhật cấu hình của Supervisor và bắt đầu các process giám sát bằng các lệnh sau:

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

> [!NOTE]
> Để biết thêm thông tin về cách chạy Supervisor, hãy tham khảo [tài liệu về Supervisor](http://supervisord.org/index.html).

<a name="tags"></a>
## Tags

Horizon cho phép bạn gán các "tags" cho các job, bao gồm cả mailables, event broadcast, notification và queued event listener. Trong thực tế, Horizon sẽ tự động gắn tag cho tất cả các job tùy thuộc vào các model Eloquent được gắn vào job. Ví dụ, hãy xem job sau:

```php
<?php

namespace App\Jobs;

use App\Models\Video;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class RenderVideo implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Video $video,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // ...
    }
}
```

Nếu job này được queue với một instance `App\Models\Video` có thuộc tính `id` là `1`, thì nó sẽ tự động nhận tag là `App\Models\Video:1`. Điều này là do Horizon sẽ tìm kiếm các thuộc tính của job xem có model Eloquent nào không. Nếu có một model Eloquent được tìm thấy, thì Horizon sẽ gắn tag job bằng cách sử dụng tên class model và khóa của model đó:

```php
use App\Jobs\RenderVideo;
use App\Models\Video;

$video = Video::find(1);

RenderVideo::dispatch($video);
```

<a name="manually-tagging-jobs"></a>
#### Manually Tagging Jobs

Nếu bạn muốn tự định nghĩa tag cho một trong các đối tượng queueable của bạn, bạn có thể định nghĩa một phương thức `tags` trên class:

```php
class RenderVideo implements ShouldQueue
{
    /**
     * Get the tags that should be assigned to the job.
     *
     * @return array<int, string>
     */
    public function tags(): array
    {
        return ['render', 'video:'.$this->video->id];
    }
}
```

<a name="manually-tagging-event-listeners"></a>
#### Manually Tagging Event Listeners

Khi lấy ra các tag cho queued event listener, Horizon sẽ tự động truyền instance event tới phương thức `tags`, cho phép bạn thêm dữ liệu event vào các tag:

```php
class SendRenderNotifications implements ShouldQueue
{
    /**
     * Get the tags that should be assigned to the listener.
     *
     * @return array<int, string>
     */
    public function tags(VideoRendered $event): array
    {
        return ['video:'.$event->video->id];
    }
}
```

<a name="notifications"></a>
## Thông báo

> [!WARNING]
> Khi cấu hình Horizon để gửi thông báo như Slack hoặc SMS, thì bạn cũng nên xem lại [các yêu cầu cần thiết của channel mà bạn muốn xử dụng](/docs/{{version}}/notifications).

Nếu bạn muốn nhận được thông báo khi một trong các queue của bạn có thời gian chờ quá lâu, bạn có thể sử dụng các phương thức `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, và `Horizon::routeSmsNotificationsTo`. Bạn có thể gọi các phương thức này từ `App\Providers\HorizonServiceProvider`:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    parent::boot();

    Horizon::routeSmsNotificationsTo('15556667777');
    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
}
```

<a name="configuring-notification-wait-time-thresholds"></a>
#### Configuring Notification Wait Time Thresholds

Bạn có thể cài đặt số giây thì sẽ được coi là "chờ lâu" trong file cấu hình `config/horizon.php` trong application của bạn. Tùy chọn cấu hình `waits` trong file này cho phép bạn kiểm soát ngưỡng chờ cho mỗi connection và queue. Bất kỳ sự kết hợp nào giữa connection và queue mà không định nghĩa trước giá trị này sẽ mặc định ở ngưỡng là 60 giây:

```php
'waits' => [
    'redis:critical' => 30,
    'redis:default' => 60,
    'redis:batch' => 120,
],
```

Việc thiết lập ngưỡng của một queue về `0` sẽ vô hiệu hóa các thông báo chờ lâu cho queue đó.

<a name="metrics"></a>
## Số liệu

Horizon có chứa một bảng điều khiển cung cấp các thông tin về số liệu job, thời gian chờ và lưu lượng của queue. Để hiển thị bảng điều khiển này, bạn nên cài đặt lệnh Artisan `snapshot` của Horizon chạy năm phút một lần trong file `routes/console.php` của application của bạn:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

Nếu bạn muốn xóa tất cả các số liệu, bạn có thể sử dụng lệnh Artisan `horizon:clear-metrics`:

```shell
php artisan horizon:clear-metrics
```

<a name="deleting-failed-jobs"></a>
## Xoá job thất bại

Nếu bạn muốn xóa một job thất bại, bạn có thể sử dụng lệnh `horizon:forget`. Lệnh `horizon:forget` sẽ chấp nhận một ID hoặc một UUID của job bị thất bại làm tham số duy nhất của nó:

```shell
php artisan horizon:forget 5
```

Nếu bạn muốn xóa tất cả các job bị thất bại, bạn có thể cung cấp tùy chọn `--all` cho lệnh `horizon:forget`:

```shell
php artisan horizon:forget --all
```

<a name="clearing-jobs-from-queues"></a>
## Xoá job từ queue

Nếu bạn muốn xóa tất cả các job ra khỏi queue mặc định của ứng dụng, bạn có thể làm việc đó bằng cách sử dụng lệnh Artisan `horizon:clear`:

```shell
php artisan horizon:clear
```

Bạn cũng có thể cung cấp tùy chọn `queue` để xóa job ra khỏi một queue cụ thể:

```shell
php artisan horizon:clear --queue=emails
```
