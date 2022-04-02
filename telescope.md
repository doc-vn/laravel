# Laravel Telescope

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cấu hình](#configuration)
    - [Bỏ bớt Data](#data-pruning)
    - [Tuỳ chỉnh Migration](#migration-customization)
- [Dashboard Authorization](#dashboard-authorization)
- [Filtering](#filtering)
    - [Entries](#filtering-entries)
    - [Batches](#filtering-batches)
- [Tagging](#tagging)
- [Available Watchers](#available-watchers)
    - [Cache Watcher](#cache-watcher)
    - [Command Watcher](#command-watcher)
    - [Dump Watcher](#dump-watcher)
    - [Event Watcher](#event-watcher)
    - [Exception Watcher](#exception-watcher)
    - [Gate Watcher](#gate-watcher)
    - [Job Watcher](#job-watcher)
    - [Log Watcher](#log-watcher)
    - [Mail Watcher](#mail-watcher)
    - [Model Watcher](#model-watcher)
    - [Notification Watcher](#notification-watcher)
    - [Query Watcher](#query-watcher)
    - [Redis Watcher](#redis-watcher)
    - [Request Watcher](#request-watcher)
    - [Schedule Watcher](#schedule-watcher)

<a name="introduction"></a>
## Giới thiệu

Laravel Telescope là một trình gỡ lỗi cho Laravel framework. Telescope sẽ cung cấp các thông tin chi tiết về các request đi đến ứng dụng của bạn, ngoại lệ, log, truy vấn cơ sở dữ liệu, queued job, mail, thông báo, cache, task schedule, dump các biến và hơn thế nữa. Telescope sẽ là người bạn đồng hành tuyệt vời cùng với môi trường phát triển của bạn.

<p align="center">
<img src="https://laravel.com/assets/img/examples/Screen_Shot_2018-10-09_at_1.47.23_PM.png" width="600">
</p>

<a name="installation"></a>
## Cài đặt

Bạn có thể sử dụng Composer để cài đặt Telescope vào project Laravel của bạn:

    composer require laravel/telescope:^3.0

Sau khi cài đặt Telescope, hãy export nội dung của nó bằng lệnh Artisan `telescope:install`. Sau khi cài đặt Telescope xong, bạn cũng nên chạy lệnh `migrate`:

    php artisan telescope:install

    php artisan migrate

#### Cập nhật Telescope

Khi cập nhật Telescope xong, bạn nên export lại nội dung của Telescope một lần nữa:

    php artisan telescope:publish

### Cài đặt trong một môi trường cụ thể

Nếu bạn chỉ định sử dụng Telescope để hỗ trợ quá trình phát triển local của bạn, bạn có thể thêm cài đặt Telescope bằng flag `--dev`:

    composer require laravel/telescope --dev

Sau khi chạy `telescope:install`, bạn nên xóa đăng ký service provider `TelescopeServiceProvider` ra khỏi file cấu hình `app` của bạn. Thay vào đó, hãy đăng ký service provider đó trong phương thức `register` của `AppServiceProvider`:

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        if ($this->app->isLocal()) {
            $this->app->register(TelescopeServiceProvider::class);
        }
    }

<a name="migration-customization"></a>
### Tuỳ chỉnh Migration

Nếu bạn không sử dụng migration mặc định của Telescope, bạn nên gọi phương thức `Telescope::ignoreMigrations` trong phương thức `register` của `AppServiceProvider` của bạn. Bạn có thể export các migration mặc định này bằng cách sử dụng lệnh `php artisan vendor:publish --tag=telescope-migrations`.

<a name="configuration"></a>
### Cấu hình

Sau khi export nội dung của Telescope, file cấu hình chính của Telescope sẽ được lưu tại `config/telescope.php`. File cấu hình này cho phép bạn cấu hình các tùy chọn theo dõi của bạn và mỗi tùy chọn cấu hình lại chứa phần mô tả về mục đích của nó, vì vậy hãy chắc chắn là bạn đã xem kỹ file này.

Nếu muốn, bạn có thể tắt hoàn toàn việc thu thập dữ liệu của Telescope bằng cách sử dụng tùy chọn cấu hình `enabled`:

    'enabled' => env('TELESCOPE_ENABLED', true),

<a name="data-pruning"></a>
### Bỏ bớt Data

Nếu không bỏ bớt data, thì bảng `telescope_entries` có thể bị tăng các bản ghi một cách nhanh chóng. Để giảm thiểu điều này, bạn nên lập một lịch để chạy lệnh Artisan `telescope:prune` mỗi ngày:

    $schedule->command('telescope:prune')->daily();

Mặc định, tất cả các dữ liệu cũ hơn 24 giờ sẽ bị lược bỏ. Bạn cũng có thể sử dụng tùy chọn `hours` khi gọi lệnh để định nghĩa thời gian lưu trữ dữ liệu của Telescope. Ví dụ: lệnh sau sẽ xóa tất cả các bản ghi được tạo từ 48 giờ trước:

    $schedule->command('telescope:prune --hours=48')->daily();

<a name="dashboard-authorization"></a>
## Dashboard Authorization

Telescope sẽ làm lộ một trang tổng quan tại `/telescope`. Mặc định, bạn sẽ chỉ có thể truy cập được trang tổng quan này trong môi trường `local`. Trong file `app/Providers/TelescopeServiceProvider.php` của bạn, sẽ có một phương thức `gate`. Authorization gate này sẽ kiểm soát quyền truy cập vào Telescope trong các môi trường **không phải là local**. Bạn có thể thoải mái sửa gate này nếu cần để hạn chế quyền truy cập vào cài đặt telescope của bạn:

    /**
     * Register the Telescope gate.
     *
     * This gate determines who can access Telescope in non-local environments.
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewTelescope', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

<a name="filtering"></a>
## Filtering

<a name="filtering-entries"></a>
### Entries

Bạn có thể lọc dữ liệu được Telescope ghi lại thông qua lệnh callback `filter` đã được đăng ký trong `TelescopeServiceProvider` của bạn. Mặc định, lệnh callback này sẽ ghi lại tất cả các dữ liệu trong môi trường `local` và các ngoại lệ, các job bị thất bại, các task schedule và dữ liệu có các thẻ được giám sát trong tất cả các môi trường khác:

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filter(function (IncomingEntry $entry) {
            if ($this->app->isLocal()) {
                return true;
            }

            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->hasMonitoredTag();
        });
    }

<a name="filtering-batches"></a>
### Batches

Trong khi lệnh callback `filter` dùng để lọc dữ liệu cho các mục riêng lẻ, thì bạn có thể sử dụng phương thức `filterBatch` để đăng ký một lệnh callback để lọc tất cả dữ liệu cho một request hoặc một lệnh console. Nếu lệnh callback này trả về giá trị `true`, thì tất cả các mục sẽ được ghi lại bởi Telescope:

    use Illuminate\Support\Collection;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filterBatch(function (Collection $entries) {
            if ($this->app->isLocal()) {
                return true;
            }

            return $entries->contains(function ($entry) {
                return $entry->isReportableException() ||
                    $entry->isFailedJob() ||
                    $entry->isScheduledTask() ||
                    $entry->hasMonitoredTag();
                });
        });
    }

<a name="tagging"></a>
## Tagging

Telescope cho phép bạn tìm kiếm các entry theo "tag". Thông thường, các tag là các tên class của model Eloquent hoặc ID người dùng đã được xác thực mà Telescope tự động thêm vào các entry. Đôi khi, bạn có thể muốn đính kèm thêm các tag tùy chỉnh của bạn vào các entry. Để thực hiện điều này, bạn có thể sử dụng phương thức `Telescope::tag`. Phương thức `tag` chấp nhận một lệnh callback sẽ trả về một mảng tag. Các tag được callback trả về sẽ được merge với bất kỳ tag nào khác được Telescope tự động gắn vào entry. Bạn nên gọi phương thức `tag` trong `TelescopeServiceProvider` của bạn:

    use Laravel\Telescope\Telescope;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::tag(function (IncomingEntry $entry) {
            if ($entry->type === 'request') {
                return ['status:'.$entry->content['response_status']];
            }

            return [];
        });
     }

<a name="available-watchers"></a>
## Available Watchers

Telescope watcher sẽ thu thập dữ liệu ứng dụng của bạn khi một request hoặc một lệnh console được thực thi. Bạn có thể tùy chỉnh danh sách watcher mà bạn muốn bật trong file cấu hình `config/telescope.php` của bạn:

    'watchers' => [
        Watchers\CacheWatcher::class => true,
        Watchers\CommandWatcher::class => true,
        ...
    ],

Một số watcher cũng cho phép bạn cung cấp thêm các tùy chọn tùy chỉnh:

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 100,
        ],
        ...
    ],

<a name="cache-watcher"></a>
### Cache Watcher

Cache watcher sẽ ghi lại dữ liệu khi một cache key được hit, bị missed, hoặc cập nhật hoặc forgotten.

<a name="command-watcher"></a>
### Command Watcher

Command watcher sẽ ghi lại các tham số, tùy chọn, exit code và output bất cứ khi nào lệnh Artisan được thực thi. Nếu bạn muốn bỏ một số lệnh ra khỏi watcher, bạn có thể chỉ định lệnh đó trong tùy chọn `ignore` trong file `config/telescope.php` của bạn:

    'watchers' => [
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => ['key:generate'],
        ],
        ...
    ],

<a name="dump-watcher"></a>
### Dump Watcher

Dump watcher sẽ ghi lại và hiển thị các dump dữ liệu của bạn trong Telescope. Khi sử dụng Laravel, các biến có thể được dump bằng cách sử dụng hàm global `dump`. Tab dump watcher phải được mở trong trình duyệt để quá trình ghi được diễn ra, nếu không dump watcher sẽ bỏ qua.

<a name="event-watcher"></a>
### Event Watcher

Event watcher sẽ ghi lại payload, listener và broadcast data cho bất kỳ event nào được ứng dụng của bạn gửi đi. Các event nội bộ của framework Laravel bị bỏ qua bởi Event watcher.

<a name="exception-watcher"></a>
### Exception Watcher

Exception watcher sẽ ghi lại dữ liệu và message lỗi cho bất kỳ exception report nào được ứng dụng của bạn đưa vào.

<a name="gate-watcher"></a>
### Gate Watcher

Gate watcher sẽ ghi lại dữ liệu và kết quả check của gate và policy bởi ứng dụng của bạn. Nếu bạn muốn watcher bỏ qua một số kiểm tra nhất định, bạn có thể chỉ định những kiểm tra này trong tùy chọn `ignore_abilities` của file `config/telescope.php` của bạn:

    'watchers' => [
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => ['viewNova'],
        ],
        ...
    ],

<a name="job-watcher"></a>
### Job Watcher

Job watcher sẽ ghi lại dữ liệu và trạng thái của bất kỳ job nào mà ứng dụng của bạn gửi đi.

<a name="log-watcher"></a>
### Log Watcher

Log watcher sẽ ghi lại dữ liệu log cho bất kỳ log nào được viết bởi ứng dụng của bạn.

<a name="mail-watcher"></a>
### Mail Watcher

Mail watcher cho phép bạn xem trước trong trình duyệt các email cùng với dữ liệu của chúng. Bạn cũng có thể tải email xuống dưới dạng file `.eml`.

<a name="model-watcher"></a>
### Model Watcher

Model watcher sẽ ghi lại những thay đổi của model bất cứ khi nào một event Eloquent `created`, `updated`, `restored`, hoặc `deleted` được gửi đi. Bạn có thể chỉ định các event nào của model sẽ được ghi lại thông qua tùy chọn `events` của watcher:

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
        ],
        ...
    ],

<a name="notification-watcher"></a>
### Notification Watcher

Notification watcher sẽ ghi lại tất cả các thông báo do ứng dụng của bạn gửi đi. Nếu thông báo đó kích hoạt một email và bạn đã enable mail watcher, thì email đó cũng sẽ có thể xem trên màn hình mail watcher.

<a name="query-watcher"></a>
### Query Watcher

Query watcher sẽ ghi lại các raw SQL, binding và thời gian thực thi cho tất cả các truy vấn được ứng dụng của bạn chạy. Watcher cũng gắn thẻ vào bất kỳ truy vấn nào mà chậm hơn 100ms là `slow`. Bạn có thể tùy chỉnh ngưỡng mà truy vấn được coi là chậm bằng cách sử dụng tùy chọn `slow` của watcher:

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 50,
        ],
        ...
    ],

<a name="redis-watcher"></a>
### Redis Watcher

Redis watcher sẽ ghi lại tất cả các lệnh Redis được thực thi bởi ứng dụng của bạn. Nếu bạn đang sử dụng Redis để lưu vào cache, thì các cache command cũng sẽ được Redis Watcher ghi lại.

<a name="request-watcher"></a>
### Request Watcher

Request watcher sẽ ghi lại dữ liệu request, header, session và response được liên kết với bất kỳ request nào được ứng dụng của bạn xử lý. Bạn có thể giới hạn dữ liệu response thông qua tùy chọn `size_limit` (tính bằng KB):

    'watchers' => [
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
        ],
        ...
    ],

<a name="schedule-watcher"></a>
### Schedule Watcher

Schedule watcher sẽ ghi lại các lệnh và output của bất kỳ task schedule nào mà do ứng dụng của bạn chạy.
