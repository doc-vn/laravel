# Laravel Horizon

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cấu hình](#configuration)
    - [Authorization vào bảng điều khiển](#dashboard-authorization)
- [Cập nhật Horizon](#upgrading)
- [Chạy Horizon](#running-horizon)
    - [Deploy Horizon](#deploying-horizon)
- [Tags](#tags)
- [Thông báo](#notifications)
- [Số liệu](#metrics)

<a name="introduction"></a>
## Giới thiệu

Horizon cung cấp một bảng điều khiển đẹp mắt và cấu hình code-driven cho queue Redis được hỗ trợ bởi Laravel. Horizon cho phép bạn dễ dàng theo dõi các số liệu chính của hệ thống queue của bạn như job được thông qua, thời gian chạy hoặc job bị thất bại.

Tất cả các cấu hình worker của bạn được lưu trong một file cấu hình đơn giản, cho phép lưu cấu hình của bạn vào trong source control, nơi mà toàn bộ nhóm phát triển của bạn có thể làm việc cùng nhau.

<a name="installation"></a>
## Cài đặt

> {note} Bạn nên đảm bảo rằng queue connection của bạn đã được set thành `redis` trong file cấu hình `queue` của bạn.

Bạn có thể sử dụng Composer để cài đặt Horizon vào project Laravel của bạn:

    composer require laravel/horizon ~3.0

Sau khi cài đặt Horizon, hãy export asset của nó bằng lệnh Artisan `horizon:install`:

    php artisan horizon:install

<a name="configuration"></a>
### Cấu hình

Sau khi export asset của Horizon xong, file cấu hình của nó sẽ được lưu tại `config/horizon.php`. File cấu hình này cho phép bạn cài đặt các tùy chọn cho worker của bạn và mỗi tùy chọn cài đặt này đều có chứa phần mô tả về mục đích của nó, vì vậy bạn hãy đọc kỹ file này.

> {note} Bạn nên đảm bảo là phần `environments` của file cấu hình `horizon` của bạn đã khai báo các item cho từng môi trường mà bạn định chạy Horizon.

#### Balance Options

Horizon cho phép bạn chọn từ ba chiến lược balance: `simple`, `auto`, và `false`. Chiến lược `simple` sẽ được cấu hình làm cấu hình mặc định, và nó sẽ chia đều các incoming job giữa các process:

    'balance' => 'simple',

Chiến lược `auto` sẽ điều chỉnh số lượng process worker trên mỗi queue dựa trên khối lượng job hiện tại của queue. Ví dụ: nếu queue `notifications` của bạn có 1.000 job đang chờ trong khi queue `render` của bạn thì trống không làm gì, Horizon sẽ phân bổ nhiều worker hơn vào queue `notifications` của bạn cho đến khi nó trống. Khi tùy chọn `balance` là `false`, thì mặc định hành vi của Laravel sẽ được sử dụng, nó sẽ xử lý các queue theo thứ tự mà chúng được liệt kê trong cấu hình của bạn.

Khi sử dụng chiến lược `auto`, vì bạn có thể định nghĩa các tùy chọn cấu hình `minProcesses` và `maxProcesses` để kiểm soát số lượng process tối thiểu và tối đa mà Horizon sẽ tăng hoặc giảm thành. Giá trị `minProcesses` sẽ chỉ định số lượng process tối thiểu có trong mỗi queue, trong khi giá trị `maxProcesses` sẽ chỉ định số lượng process tối đa có trong tất cả các queue:

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'tries' => 3,
            ],
        ],
    ],

#### Job Trimming

File cấu hình `horizon` sẽ cho phép bạn cấu hình thời gian tồn tại của các job gần đây và các job bị thất bại (tính bằng phút). Mặc định, các job gần đây được lưu trong một giờ trong khi các job bị thất bại được lưu trong một tuần:

    'trim' => [
        'recent' => 60,
        'failed' => 10080,
    ],

<a name="dashboard-authorization"></a>
### Authorization vào bảng điều khiển

Trong file `app/Providers/HorizonServiceProvider.php` của bạn, có một phương thức là `gate`. Gate authorization này sẽ kiểm soát quyền truy cập vào Horizon trong các môi trường **không phải là local**. Bạn có thể thoải mái sửa gate này nếu cần để hạn chế quyền truy cập vào các cài đặt Horizon của bạn:

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

> {note} Hãy nhớ rằng Laravel tự động đưa người dùng *đã xác thực* vào Gate. Nếu ứng dụng của bạn đang cung cấp bảo mật cho Horizon thông qua một phương thức khác, chẳng hạn như hạn chế IP, thì người dùng Horizon của bạn có thể không cần "đăng nhập". Do đó, bạn sẽ cần phải thay đổi `function ($user)` ở trên thành `function ($user = null)` để yêu cầu Laravel không yêu cầu xác thực.

<a name="upgrading"></a>
#### Cập nhật Horizon

Khi nâng cấp lên phiên bản mới của Horizon, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/horizon/blob/master/UPGRADE.md).

Ngoài ra, bạn nên export lại assets của Horizon:

    php artisan horizon:assets

<a name="running-horizon"></a>
## Chạy Horizon

Khi bạn đã cài đặt các worker của bạn vào trong file cấu hình `config/horizon.php`, bạn có thể bắt đầu Horizon bằng cách chạy lệnh Artisan `horizon`. Lệnh này sẽ chạy tất cả các worker đã được cài đặt của bạn:

    php artisan horizon

Bạn có thể dừng process của Horizon và bảo nó tiếp tục xử lý các job bằng cách sử dụng các lệnh Artisan `horizon:pause` và `horizon:continue`:

    php artisan horizon:pause

    php artisan horizon:continue

Bạn có thể kiểm tra trạng thái hiện tại của process Horizon bằng cách sử dụng lệnh Artisan `horizon:status`:

    php artisan horizon:status

Bạn có thể huỷ một process Horizon master trên máy của bạn bằng lệnh Artisan `horizon:terminate`. Tất cả các job mà Horizon đang xử lý sẽ được hoàn tất rồi sau đó Horizon sẽ được huỷ:

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### Deploy Horizon

Nếu bạn đang deploy Horizon đến một server thật, bạn nên cài đặt một process giám sát để theo dõi lệnh `php artisan horizon` và khởi động lại nếu nó bị thoát bất ngờ. Khi deploy code mới đến server của bạn, bạn sẽ cần phải bảo process Horizon master dừng lại để nó có thể được khởi động lại bởi process giám sát của bạn và nhận được các thay đổi của code của bạn.

#### Installing Supervisor

Supervisor là một process giám sát cho hệ điều hành Linux và sẽ tự động khởi động lại các process `horizon` nếu process đó bị thất bại. Để cài đặt Supervisor trên Ubuntu, bạn có thể sử dụng lệnh sau:

    sudo apt-get install supervisor

> {tip} Nếu bạn không muốn tự cấu hình Supervisor, hãy xem xét việc sử dụng [Laravel Forge](https://forge.laravel.com), nó sẽ tự động cài đặt và cấu hình Supervisor cho các dự án Laravel của bạn.

#### Supervisor Configuration

Các file cấu hình Supervisor thường được lưu trong thư mục `/etc/supervisor/conf.d`. Trong thư mục này, bạn có thể tạo bất kỳ file cấu hình nào để hướng dẫn supervisor cách giám sát các process của bạn. Ví dụ: hãy tạo một file `horizon.conf` để khởi động và theo dõi process `horizon`:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log
    stopwaitsecs=3600

> {note} Bạn nên chắc chắn rằng giá trị của `stopwaitsecs` sẽ luôn lớn hơn số giây lâu nhất mà job của bạn đang chạy. Nếu không, Supervisor có thể kết thúc job đó trước khi nó được xử lý xong.

#### Starting Supervisor

Khi file cấu hình đã được tạo xong, bạn có thể cập nhật cấu hình của Supervisor và bắt đầu các process bằng các lệnh sau:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start horizon

Để biết thêm thông tin về Supervisor, hãy tham khảo [tài liệu về Supervisor](http://supervisord.org/index.html).

<a name="tags"></a>
## Tags

Horizon cho phép bạn gán các “tags” cho các job, bao gồm cả mailables, event broadcast, notification và queued event listener. Trong thực tế, Horizon sẽ tự động gắn tag cho tất cả các job tùy thuộc vào các model Eloquent được gắn vào job. Ví dụ, hãy xem job sau:

    <?php

    namespace App\Jobs;

    use App\Video;
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
         * @var \App\Video
         */
        public $video;

        /**
         * Create a new job instance.
         *
         * @param  \App\Video  $video
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

Nếu job này được queue với một instance `App\Video` có `id` là `1`, thì nó sẽ tự động nhận tag là `App\Video:1`. Điều này là do Horizon sẽ kiểm tra các thuộc tính của job xem có model Eloquent nào không. Nếu có một model Eloquent được tìm thấy, thì Horizon sẽ gắn tag job bằng cách sử dụng tên class model và khóa của model đó:

    $video = App\Video::find(1);

    App\Jobs\RenderVideo::dispatch($video);

#### Manually Tagging

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

> **Lưu ý:** Khi cấu hình Horizon để gửi thông báo như Slack hoặc SMS, thì bạn cũng nên xem lại [các yêu cầu cần thiết của driver mà bạn muốn xử dụng](/docs/{{version}}/notifications).

Nếu bạn muốn nhận được thông báo khi một trong các queue của bạn có thời gian chờ quá lâu, bạn có thể sử dụng các phương thức `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, và `Horizon::routeSmsNotificationsTo`. Bạn có thể gọi các phương thức này từ `HorizonServiceProvider`:

    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    Horizon::routeSmsNotificationsTo('15556667777');

#### Configuring Notification Wait Time Thresholds

Bạn có thể cài đặt số giây thì sẽ được coi là "chờ lâu" trong file cấu hình `config/horizon.php` của bạn. Tùy chọn cấu hình `waits` trong file này cho phép bạn kiểm soát ngưỡng chờ cho mỗi connection / queue:

    'waits' => [
        'redis:default' => 60,
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
