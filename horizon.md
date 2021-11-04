# Laravel Horizon

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cấu hình](#configuration)
    - [Authentication vào bảng điều khiển](#dashboard-authentication)
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

> {note} Do sử dụng tín hiệu process không đồng bộ, nên Horizon yêu cầu PHP 7.1+.

Bạn có thể sử dụng Composer để cài đặt Horizon vào project Laravel của bạn:

    composer require laravel/horizon

Sau khi cài đặt Horizon, hãy export asset của nó bằng lệnh Artisan `vendor:publish`:

    php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"

<a name="configuration"></a>
### Cấu hình

Sau khi export asset của Horizon xong, file cấu hình của nó sẽ được lưu tại `config/horizon.php`. File cấu hình này cho phép bạn cài đặt các tùy chọn cho worker của bạn và mỗi tùy chọn cài đặt này đều có chứa phần mô tả về mục đích của nó, vì vậy bạn hãy đọc kỹ file này.

#### Balance Options

Horizon cho phép bạn chọn từ ba chiến lược balance: `simple`, `auto`, và `false`. Chiến lược `simple` là mặc định, chia đều các incoming job giữa các process:

    'balance' => 'simple',

Chiến lược `auto` sẽ điều chỉnh số lượng process worker trên mỗi queue dựa trên khối lượng job hiện tại của queue. Ví dụ: nếu queue `notifications` của bạn có 1.000 job đang chờ trong khi queue `render` của bạn thì trống không làm gì, Horizon sẽ phân bổ nhiều worker hơn vào queue `notifications` của bạn cho đến khi nó trống. Khi tùy chọn `balance` là `false`, thì mặc định hành vi của Laravel sẽ được sử dụng, nó sẽ xử lý các queue theo thứ tự mà chúng được liệt kê trong cấu hình của bạn.

<a name="dashboard-authentication"></a>
### Authentication vào bảng điều khiển

Horizon hiển thị bảng điều khiển tại `/horizon`. Mặc định, bạn sẽ chỉ có thể truy cập được vào bảng điều khiển này trong môi trường `local`. Để định nghĩa quyền truy cập cụ thể hơn cho bảng điều khiển, bạn nên sử dụng phương thức `Horizon::auth`. Phương thức `auth` chấp nhận một callback trả về kết quả `true` hoặc `false`, cho biết liệu người dùng hiện tại có thể truy cập vào bảng điều khiển Horizon hay không:

    Horizon::auth(function ($request) {
        // return true / false;
    });

<a name="running-horizon"></a>
## Chạy Horizon

Khi bạn đã cài đặt các worker của bạn vào trong file cấu hình `config/horizon.php`, bạn có thể bắt đầu Horizon bằng cách chạy lệnh Artisan `horizon`. Lệnh này sẽ chạy tất cả các worker đã được cài đặt của bạn:

    php artisan horizon

Bạn có thể dừng process của Horizon và bảo nó tiếp tục xử lý các job bằng cách sử dụng các lệnh Artisan `horizon:pause` và `horizon:continue`:

    php artisan horizon:pause

    php artisan horizon:continue

Bạn có thể huỷ một process Horizon master trên máy của bạn bằng lệnh Artisan `horizon:terminate`. Tất cả các job mà Horizon đang xử lý sẽ được hoàn tất rồi sau đó Horizon sẽ được huỷ:

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### Deploy Horizon

Nếu bạn đang deploy Horizon đến một server thật, bạn nên cài đặt một process giám sát để theo dõi lệnh `php artisan horizon` và khởi động lại nếu nó bị thoát bất ngờ. Khi deploy code mới đến server của bạn, bạn sẽ cần phải bảo process Horizon master dừng lại để nó có thể được khởi động lại bởi process giám sát của bạn và nhận được các thay đổi của code của bạn.

Bạn có thể dừng process master của Horizon trên máy của bạn bằng lệnh Artisan `horizon:terminate`. Tất cả các job mà Horizon đang xử lý sẽ được hoàn tất rồi sau đó Horizon sẽ được huỷ:

    php artisan horizon:terminate

#### Supervisor Configuration

Nếu bạn đang sử dụng process giám sát Supervisor để quản lý process `horizon` của bạn, thì file cấu hình sau nên được sử dụng:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log

> {tip} Nếu bạn cảm thấy khó khăn khi quản lý server của bạn, bạn hãy cân nhắc sử dụng [Laravel Forge](https://forge.laravel.com). Forge sẽ cung cấp các server PHP 7+ với mọi thứ bạn cần để chạy các application Laravel hiện đại, mạnh mẽ với Horizon.

<a name="tags"></a>
## Tags

Horizon cho phép bạn gán các “tags” cho các job, bao gồm cả mailables, event broadcast, notification và queued event listener. Trong thực tế, Horizon sẽ tự động gắn tag cho tất cả các job tùy thuộc vào các model Eloquent được gắn vào job. Ví dụ, hãy xem job sau:

    <?php

    namespace App\Jobs;

    use App\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

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

> **Lưu ý:** Trước khi sử dụng thông báo, bạn nên thêm package Composer `guzzlehttp/guzzle` vào trong project của bạn. Khi cấu hình Horizon để gửi thông báo như SMS, thì bạn cũng nên xem lại [các yêu cầu của driver thông báo Nexmo](https://laravel.com/docs/5.5/notifications#sms-notifications).

Nếu bạn muốn nhận được thông báo khi một trong các queue của bạn có thời gian chờ quá lâu, bạn có thể sử dụng các phương thức `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, và `Horizon::routeSmsNotificationsTo`. Bạn có thể gọi các phương thức này từ `AppServiceProvider`:

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
