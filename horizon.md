# Laravel Horizon

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cấu hình](#configuration)
    - [Balancing Strategies](#balancing-strategies)
    - [Authorization vào bảng điều khiển](#dashboard-authorization)
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

> {tip} Trước khi tìm hiểu về Laravel Horizon, bạn nên tìm hiểu qua [queue services](/docs/{{version}}/queues) trong Laravel. Horizon hỗ trợ queue của Laravel với các tính năng mới nên có thể gây nhầm lẫn nếu bạn chưa quen với các tính năng cơ bản của queue do Laravel cung cấp.

[Laravel Horizon](https://github.com/laravel/horizon) cung cấp một bảng điều khiển đẹp mắt và cấu hình code-driven cho [queue Redis](/docs/{{version}}/queues) được hỗ trợ bởi Laravel. Horizon cho phép bạn dễ dàng theo dõi các số liệu chính của hệ thống queue của bạn như job được thông qua, thời gian chạy hoặc job bị thất bại.

Khi sử dụng Horizon, tất cả các cấu hình queue worker của bạn được lưu trong một file cấu hình đơn giản. Bằng cách định nghĩa cấu hình worker của ứng dụng của bạn trong file được version controll, bạn có thể dễ dàng mở rộng quy mô hoặc sửa chữa queue worker của ứng dụng khi triển khai ứng dụng của bạn.

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## Cài đặt

> {note} Laravel Horizon yêu cầu bạn sử dụng [Redis](https://redis.io) để hỗ trợ queue của bạn. Vì thế, bạn nên đảm bảo rằng queue connection của bạn đã được set thành `redis` trong file cấu hình `config/queue.php` trong apllication của bạn.

Bạn có thể sử dụng Composer để cài đặt Horizon vào project Laravel của bạn:

    composer require laravel/horizon

Sau khi cài đặt Horizon, hãy export asset của nó bằng lệnh Artisan `horizon:install`:

    php artisan horizon:install

<a name="configuration"></a>
### Cấu hình

Sau khi export asset của Horizon xong, file cấu hình của nó sẽ được lưu tại `config/horizon.php`. File cấu hình này cho phép bạn cài đặt các tùy chọn cho queue worker cho application của bạn. Mỗi tùy chọn cài đặt này đều có chứa phần mô tả về mục đích của nó, vì vậy bạn hãy đọc kỹ file này.

> {note} Horizon sử dụng kết nối Redis có tên là `horizon` trong nội bộ. Tên kết nối Redis này được đặt tên trước và không được gán cho bất kỳ kết nối Redis nào khác trong file cấu hình `database.php` hoặc làm giá trị của tùy chọn `use` trong file cấu hình `horizon.php`.

<a name="environments"></a>
#### Environments

Sau khi cài đặt, tùy chọn cấu hình Horizon chính mà bạn nên xem là tùy chọn cấu hình `environments`. Tùy chọn cấu hình này là một mảng các môi trường mà ứng dụng của bạn có thể chạy trên đó và ngoài ra, nó còn định nghĩa các tùy chọn worker process cho từng loại môi trường đó. Mặc định, tuỳ chọn này chứa môi trường `production` và `local`. Tuy nhiên, bạn có thể thoải mái thêm nhiều môi trường hơn nếu cần:

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

Khi bạn khởi động Horizon, nó sẽ sử dụng các tùy chọn cấu hình worker process tương ứng với môi trường mà ứng dụng của bạn được chạy. Thông thường, môi trường được xác định bằng giá trị của [biến môi trường](/docs/{{version}}/configuration#determining-the-current-environment) `APP_ENV`. Ví dụ: môi trường Horizon mặc định`local` được cấu hình để bắt đầu với ba worker process và tự động cân bằng số lượng worker process được chỉ định cho mỗi queue. Môi trường `sản xuất` mặc định sẽ được cấu hình để bắt đầu tối đa 10 worker process và tự động cân bằng số lượng worker process được chỉ định cho mỗi queue.

> {note} Bạn nên đảm bảo tuỳ chọn `environments` trong file cấu hình `horizon` chứa các mục cho mỗi [environment](/docs/{{version}}/configuration#environment-configuration) mà bạn định chạy trên Horizon.

<a name="supervisors"></a>
#### Supervisors

Như bạn có thể thấy trong file cấu hình mặc định của Horizon. Mỗi môi trường có thể chứa một hoặc nhiều "supervisors". Mặc định, file cấu hình định nghĩa supervisor này là `supervisor-1`; tuy nhiên, bạn có thể tự do đặt tên cho supervisor của bạn bất cứ điều gì mà bạn muốn. Mỗi supervisor về cơ bản chịu trách nhiệm "giám sát" một nhóm worker process và đảm nhiệm việc cân bằng các worker process trên các queue.

Bạn có thể thêm các supervisor vào một môi trường nhất định nếu bạn muốn định nghĩa một nhóm các worker process mới sẽ được chạy trong môi trường đó. Bạn có thể chọn thực hiện việc này nếu bạn muốn định nghĩa một chiến lược cân bằng mới hoặc một số lượng worker process nhất định cho một queue mà được ứng dụng của bạn sử dụng.

<a name="default-values"></a>
#### Default Values

Trong file cấu hình mặc định của Horizon, bạn có thể thấy tùy chọn cấu hình `defaults`. Tùy chọn cấu hình này chỉ định các giá trị mặc định cho [supervisor](#supervisors) trong ứng dụng của bạn. Các giá trị cấu hình mặc định của supervisor sẽ được hợp nhất vào một cấu hình của supervisor cho từng môi trường cụ thể, cho phép bạn tránh lặp lại không cần thiết khi định nghĩa supervisor của bạn.

<a name="balancing-strategies"></a>
### Balancing Strategies

Horizon cho phép bạn chọn từ ba chiến lược balance: `simple`, `auto`, và `false`. Chiến lược `simple` sẽ được cấu hình làm cấu hình mặc định, và nó sẽ chia đều các incoming job giữa các process:

    'balance' => 'simple',

Chiến lược `auto` sẽ điều chỉnh số lượng process worker trên mỗi queue dựa trên khối lượng job hiện tại của queue. Ví dụ: nếu queue `notifications` của bạn có 1.000 job đang chờ trong khi queue `render` của bạn thì trống không làm gì, thì Horizon sẽ phân bổ nhiều worker hơn vào queue `notifications` của bạn cho đến khi queue đó trống.

Khi sử dụng chiến lược `auto`, vì bạn có thể định nghĩa các tùy chọn cấu hình `minProcesses` và `maxProcesses` để kiểm soát số lượng process worker tối thiểu và tối đa mà Horizon sẽ tăng hoặc giảm thành:

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
                'tries' => 3,
            ],
        ],
    ],

Các giá trị cấu hình `balanceMaxShift` và `balanceCooldown` sẽ để xác định Horizon sẽ scale như thế nào để đáp ứng nhu cầu của worker. Trong ví dụ trên, tối đa một process mới sẽ được tạo hoặc hủy sau ba giây. Bạn có thể tự do điều chỉnh các giá trị này nếu cần, dựa theo nhu cầu của ứng dụng của bạn.

Khi tùy chọn `balance` được set thành `false`, thì hành vi mặc định của Laravel sẽ được sử dụng để xử lý các queue theo thứ tự mà chúng đã được liệt kê trong cấu hình của bạn.

<a name="dashboard-authorization"></a>
### Authorization vào bảng điều khiển

Horizon hiển thị bảng điều khiển tại URI `/horizon`. Mặc định, bạn sẽ chỉ có thể truy cập trang tổng quan này trong môi trường `local`. Tuy nhiên, trong file `app/Providers/HorizonServiceProvider.php` của bạn, có một định nghĩa [gate authorization](/docs/{{version}}/authorization#gates). Gate authorization này sẽ kiểm soát quyền truy cập vào Horizon trong các môi trường **không phải là local**. Bạn có thể thoải mái sửa gate này nếu cần để hạn chế quyền truy cập vào các cài đặt Horizon của bạn:

    /**
     * Register the Horizon gate.
     *
     * This gate determines who can access Horizon in non-local environments.
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewHorizon', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

<a name="alternative-authentication-strategies"></a>
#### Alternative Authentication Strategies

Hãy nhớ rằng Laravel sẽ tự động đưa người dùng đã xác thực vào gate closure. Nếu ứng dụng của bạn đang cung cấp bảo mật cho Horizon thông qua một phương thức khác, chẳng hạn như hạn chế IP, thì người dùng Horizon của bạn có thể không cần "đăng nhập". Do đó, bạn sẽ cần phải thay đổi format `function ($user)` của closure ở trên thành `function ($user = null)` để yêu cầu Laravel không yêu cầu xác thực.

<a name="upgrading"></a>
#### Cập nhật Horizon

Khi nâng cấp lên phiên bản mới của Horizon, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/horizon/blob/master/UPGRADE.md). Ngoài ra, khi bạn nâng cấp lên bất kỳ phiên bản Horizon mới nào, bạn nên export lại assets của Horizon:

    php artisan horizon:publish

Để giữ cập nhật các file asset và tránh các sự cố trong tương lai, bạn có thể thêm lệnh `horizon:publish` vào trong tập lệnh `post-update-cmd` trong file `composer.json` trong application của bạn:

    {
        "scripts": {
            "post-update-cmd": [
                "@php artisan horizon:publish --ansi"
            ]
        }
    }

<a name="running-horizon"></a>
## Chạy Horizon

Khi bạn đã cài đặt worker và supervisor của bạn vào trong file cấu hình `config/horizon.php` trong application của bạn, bạn có thể bắt đầu Horizon bằng cách chạy lệnh Artisan `horizon`. Lệnh này sẽ chạy tất cả các worker processes đã được cài đặt cho môi trường hiện tại:

    php artisan horizon

Bạn có thể dừng process của Horizon và bảo nó tiếp tục xử lý các job bằng cách sử dụng các lệnh Artisan `horizon:pause` và `horizon:continue`:

    php artisan horizon:pause

    php artisan horizon:continue

Bạn cũng có thể tạm dừng và tiếp tục [supervisor](#supervisors) của Horizon cụ thể bằng cách sử dụng các lệnh Artisan `horizon:pause-supervisor` và `horizon:continue-supervisor`:

    php artisan horizon:pause-supervisor supervisor-1

    php artisan horizon:continue-supervisor supervisor-1

Bạn có thể kiểm tra trạng thái hiện tại của process Horizon bằng cách sử dụng lệnh Artisan `horizon:status`:

    php artisan horizon:status

Bạn có thể huỷ process Horizon bằng lệnh Artisan `horizon:terminate`. Tất cả các job đang được xử lý sẽ được hoàn thành và sau đó Horizon sẽ dừng thực hiện:

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### Deploy Horizon

Khi bạn đã sẵn sàng deploy Horizon tới máy chủ thực tế của ứng dụng của bạn, bạn nên cài đặt một process giám sát để theo dõi lệnh `php artisan horizon` và khởi động lại nếu nó bị thoát bất ngờ.

Trong quá trình deploy ứng dụng của bạn, bạn nên bảo process Horizon dừng lại để nó có thể được khởi động lại bởi process giám sát của bạn và nhận được các thay đổi của code của bạn.

    php artisan horizon:terminate

<a name="installing-supervisor"></a>
#### Installing Supervisor

Supervisor là một process giám sát cho hệ điều hành Linux và sẽ tự động khởi động lại các process `horizon` nếu process đó bị ngừng chạy. Để cài đặt Supervisor trên Ubuntu, bạn có thể sử dụng lệnh sau. Nếu không sử dụng Ubuntu, bạn có thể cài đặt Supervisor bằng package manager của hệ điều hành:

    sudo apt-get install supervisor

> {tip} Nếu bạn không muốn tự cấu hình Supervisor, hãy xem xét việc sử dụng [Laravel Forge](https://forge.laravel.com), nó sẽ tự động cài đặt và cấu hình Supervisor cho các dự án Laravel của bạn.

<a name="supervisor-configuration"></a>
#### Supervisor Configuration

Các file cấu hình Supervisor thường được lưu trong thư mục `/etc/supervisor/conf.d` của server của bạn. Trong thư mục này, bạn có thể tạo bất kỳ file cấu hình nào để hướng dẫn supervisor cách giám sát các process của bạn. Ví dụ: hãy tạo một file `horizon.conf` để khởi động và theo dõi process `horizon`:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/example.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/example.com/horizon.log
    stopwaitsecs=3600

> {note} Bạn nên chắc chắn rằng giá trị của `stopwaitsecs` sẽ luôn lớn hơn số giây lâu nhất mà job của bạn đang chạy. Nếu không, Supervisor có thể kết thúc job đó trước khi nó được xử lý xong.

<a name="starting-supervisor"></a>
#### Starting Supervisor

Khi file cấu hình đã được tạo xong, bạn có thể cập nhật cấu hình của Supervisor và bắt đầu các process giám sát bằng các lệnh sau:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start horizon

> {tip} Để biết thêm thông tin về cách chạy Supervisor, hãy tham khảo [tài liệu về Supervisor](http://supervisord.org/index.html).

<a name="tags"></a>
## Tags

Horizon cho phép bạn gán các “tags” cho các job, bao gồm cả mailables, event broadcast, notification và queued event listener. Trong thực tế, Horizon sẽ tự động gắn tag cho tất cả các job tùy thuộc vào các model Eloquent được gắn vào job. Ví dụ, hãy xem job sau:

    <?php

    namespace App\Jobs;

    use App\Models\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * The video instance.
         *
         * @var \App\Models\Video
         */
        public $video;

        /**
         * Create a new job instance.
         *
         * @param  \App\Models\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

Nếu job này được queue với một instance `App\Models\Video` có thuộc tính `id` là `1`, thì nó sẽ tự động nhận tag là `App\Models\Video:1`. Điều này là do Horizon sẽ tìm kiếm các thuộc tính của job xem có model Eloquent nào không. Nếu có một model Eloquent được tìm thấy, thì Horizon sẽ gắn tag job bằng cách sử dụng tên class model và khóa của model đó:

    use App\Jobs\RenderVideo;
    use App\Models\Video;

    $video = Video::find(1);

    RenderVideo::dispatch($video);

<a name="manually-tagging-jobs"></a>
#### Manually Tagging Jobs

Nếu bạn muốn tự định nghĩa tag cho một trong các đối tượng queueable của bạn, bạn có thể định nghĩa một phương thức `tags` trên class:

    class RenderVideo implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the job.
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>
## Thông báo

> {note} Khi cấu hình Horizon để gửi thông báo như Slack hoặc SMS, thì bạn cũng nên xem lại [các yêu cầu cần thiết của channel mà bạn muốn xử dụng](/docs/{{version}}/notifications).

Nếu bạn muốn nhận được thông báo khi một trong các queue của bạn có thời gian chờ quá lâu, bạn có thể sử dụng các phương thức `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, và `Horizon::routeSmsNotificationsTo`. Bạn có thể gọi các phương thức này từ `App\Providers\HorizonServiceProvider`:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Horizon::routeSmsNotificationsTo('15556667777');
        Horizon::routeMailNotificationsTo('example@example.com');
        Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    }

<a name="configuring-notification-wait-time-thresholds"></a>
#### Configuring Notification Wait Time Thresholds

Bạn có thể cài đặt số giây thì sẽ được coi là "chờ lâu" trong file cấu hình `config/horizon.php` trong application của bạn. Tùy chọn cấu hình `waits` trong file này cho phép bạn kiểm soát ngưỡng chờ cho mỗi connection / queue:

    'waits' => [
        'redis:default' => 60,
        'redis:critical,high' => 90,
    ],

<a name="metrics"></a>
## Số liệu

Horizon có chứa một bảng điều khiển cung cấp các thông tin về số liệu job, thời gian chờ và lưu lượng của queue. Để hiển thị bảng điều khiển này, bạn nên cài đặt lệnh Artisan `snapshot` của Horizon chạy năm phút một lần thông qua [scheduler](/docs/{{version}}/scheduling) của application của bạn:

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }

<a name="deleting-failed-jobs"></a>
## Xoá job thất bại

Nếu bạn muốn xóa một job thất bại, bạn có thể sử dụng lệnh `horizon:forget`. Lệnh `horizon:forget` sẽ chấp nhận một ID hoặc một UUID của job bị thất bại làm tham số duy nhất của nó:

    php artisan horizon:forget 5

<a name="clearing-jobs-from-queues"></a>
## Xoá job từ queue

Nếu bạn muốn xóa tất cả các job ra khỏi queue mặc định của ứng dụng, bạn có thể làm việc đó bằng cách sử dụng lệnh Artisan `horizon:clear`:

    php artisan horizon:clear

Bạn cũng có thể cung cấp tùy chọn `queue` để xóa job ra khỏi một queue cụ thể:

    php artisan horizon:clear --queue=emails
