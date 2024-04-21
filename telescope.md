# Laravel Telescope

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Chỉ cài đặt trên local](#local-only-installation)
    - [Cấu hình](#configuration)
    - [Bỏ bớt Data](#data-pruning)
    - [Dashboard Authorization](#dashboard-authorization)
- [Cập nhật Telescope](#upgrading-telescope)
- [Filtering](#filtering)
    - [Entries](#filtering-entries)
    - [Batches](#filtering-batches)
- [Tagging](#tagging)
- [Available Watchers](#available-watchers)
    - [Batch Watcher](#batch-watcher)
    - [Cache Watcher](#cache-watcher)
    - [Command Watcher](#command-watcher)
    - [Dump Watcher](#dump-watcher)
    - [Event Watcher](#event-watcher)
    - [Exception Watcher](#exception-watcher)
    - [Gate Watcher](#gate-watcher)
    - [HTTP Client Watcher](#http-client-watcher)
    - [Job Watcher](#job-watcher)
    - [Log Watcher](#log-watcher)
    - [Mail Watcher](#mail-watcher)
    - [Model Watcher](#model-watcher)
    - [Notification Watcher](#notification-watcher)
    - [Query Watcher](#query-watcher)
    - [Redis Watcher](#redis-watcher)
    - [Request Watcher](#request-watcher)
    - [Schedule Watcher](#schedule-watcher)
    - [View Watcher](#view-watcher)
- [Displaying User Avatars](#displaying-user-avatars)

<a name="introduction"></a>
## Giới thiệu

[Laravel Telescope](https://github.com/laravel/telescope) là một người bạn đồng hành tuyệt vời với môi trường phát triển Laravel local của bạn. Telescope sẽ cung cấp các thông tin chi tiết về các request đi đến ứng dụng của bạn, ngoại lệ, log, truy vấn cơ sở dữ liệu, queued job, mail, thông báo, cache, task schedule, dump các biến, và hơn thế nữa.

<img src="https://laravel.com/img/docs/telescope-example.png">

<a name="installation"></a>
## Cài đặt

Bạn có thể sử dụng Composer package manager để cài đặt Telescope vào project Laravel của bạn:

```shell
composer require laravel/telescope
```

Sau khi cài đặt Telescope, hãy export nội dung của nó bằng lệnh Artisan `telescope:install`. Sau khi cài đặt Telescope xong, bạn cũng nên chạy lệnh `migrate` để tạo ra các bảng cần thiết để lưu trữ dữ liệu của Telescope:

```shell
php artisan telescope:install

php artisan migrate
```

<a name="migration-customization"></a>
#### Migration Customization

Nếu bạn không định sử dụng các migration mặc định của Telescope, bạn nên gọi phương thức `Telescope::ignoreMigrations` trong phương thức `register` của class `App\Providers\AppServiceProvider` trong ứng dụng của bạn. Bạn có thể export các migration mặc định bằng lệnh sau: `php artisan vendor:publish --tag=telescope-migrations`

<a name="local-only-installation"></a>
### Chỉ cài đặt trên local

Nếu bạn chỉ định sử dụng Telescope để hỗ trợ quá trình phát triển local của bạn, bạn có thể thêm cài đặt Telescope bằng flag `--dev`:

```shell
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

Sau khi chạy `telescope:install`, bạn nên xóa đăng ký service provider `TelescopeServiceProvider` ra khỏi file cấu hình `config/app.php` của application của bạn. Thay vào đó, hãy tự đăng ký service provider của Telescope vào trong phương thức `register` của class `App\Providers\AppServiceProvider`. Chúng tôi sẽ đảm bảo môi trường hiện tại là `local` trước khi đăng ký provider:

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        if ($this->app->environment('local')) {
            $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
            $this->app->register(TelescopeServiceProvider::class);
        }
    }

Cuối cùng, bạn cũng nên ngăn package Telescope [tự động đăng ký](/docs/{{version}}/packages#package-discovery) bằng cách thêm code sau vào file `composer.json` của bạn:

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "laravel/telescope"
        ]
    }
},
```

<a name="configuration"></a>
### Cấu hình

Sau khi export nội dung của Telescope, file cấu hình chính của Telescope sẽ được lưu tại `config/telescope.php`. File cấu hình này cho phép bạn cấu hình các [tùy chọn theo dõi](#available-watchers) của bạn. Mỗi tùy chọn cấu hình lại chứa phần mô tả về mục đích của nó, vì vậy hãy chắc chắn là bạn đã xem kỹ file này.

Nếu muốn, bạn có thể tắt hoàn toàn việc thu thập dữ liệu của Telescope bằng cách sử dụng tùy chọn cấu hình `enabled`:

    'enabled' => env('TELESCOPE_ENABLED', true),

<a name="data-pruning"></a>
### Bỏ bớt Data

Nếu không bỏ bớt data, thì bảng `telescope_entries` có thể bị tăng các bản ghi một cách nhanh chóng. Để giảm thiểu điều này, bạn nên lập một [lịch](/docs/{{version}}/scheduling) để chạy lệnh Artisan `telescope:prune` mỗi ngày:

    $schedule->command('telescope:prune')->daily();

Mặc định, tất cả các dữ liệu cũ hơn 24 giờ sẽ bị lược bỏ. Bạn cũng có thể sử dụng tùy chọn `hours` khi gọi lệnh để định nghĩa thời gian lưu trữ dữ liệu của Telescope. Ví dụ: lệnh sau sẽ xóa tất cả các bản ghi được tạo từ 48 giờ trước:

    $schedule->command('telescope:prune --hours=48')->daily();

<a name="dashboard-authorization"></a>
### Dashboard Authorization

Trang tổng quan của Telescope có thể truy cập tại route `/telescope`. Mặc định, bạn sẽ chỉ có thể truy cập được trang tổng quan này trong môi trường `local`. Trong file `app/Providers/TelescopeServiceProvider.php` của bạn, sẽ có một định nghĩa [authorization gate](/docs/{{version}}/authorization#gates). Authorization gate này sẽ kiểm soát quyền truy cập vào Telescope trong các môi trường **không phải là local**. Bạn có thể thoải mái sửa gate này nếu cần để hạn chế quyền truy cập vào cài đặt telescope của bạn:

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

> **Warning**
> Bạn nên đảm bảo là bạn đã thay đổi biến môi trường `APP_ENV` thành `production` trong môi trường production của bạn. Nếu không, các cài đặt Telescope của bạn sẽ bị công khai trên môi trường internet.

<a name="upgrading-telescope"></a>
## Cập nhật Telescope

Khi nâng cấp lên phiên bản mới của Telescope, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/telescope/blob/master/UPGRADE.md).

Ngoài ra, khi bạn nâng cấp lên bất kỳ phiên bản Telescope mới nào, bạn nên export lại assets của Telescope:

```shell
php artisan telescope:publish
```

Để giữ cập nhật các file asset và tránh các sự cố trong tương lai, bạn có thể thêm một lệnh `vendor:publish --tag=laravel-assets` vào trong tập lệnh `post-update-cmd` trong file `composer.json` của bạn:

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}
```

<a name="filtering"></a>
## Filtering

<a name="filtering-entries"></a>
### Entries

Bạn có thể lọc dữ liệu được Telescope ghi lại thông qua lệnh closure `filter` đã được định nghĩa trong class `App\Providers\TelescopeServiceProvider` của bạn. Mặc định, lệnh closure này sẽ ghi lại tất cả các dữ liệu trong môi trường `local` và các ngoại lệ, các job bị thất bại, các task schedule và dữ liệu có các thẻ được giám sát trong tất cả các môi trường khác:

    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filter(function (IncomingEntry $entry) {
            if ($this->app->environment('local')) {
                return true;
            }

            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->isSlowQuery() ||
                $entry->hasMonitoredTag();
        });
    }

<a name="filtering-batches"></a>
### Batches

Trong khi lệnh closure `filter` dùng để lọc dữ liệu cho các mục riêng lẻ, thì bạn có thể sử dụng phương thức `filterBatch` để đăng ký một lệnh closure để lọc tất cả dữ liệu cho một request hoặc một lệnh console. Nếu lệnh closure này trả về giá trị `true`, thì tất cả các mục sẽ được ghi lại bởi Telescope:

    use Illuminate\Support\Collection;
    use Laravel\Telescope\Telescope;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filterBatch(function (Collection $entries) {
            if ($this->app->environment('local')) {
                return true;
            }

            return $entries->contains(function ($entry) {
                return $entry->isReportableException() ||
                    $entry->isFailedJob() ||
                    $entry->isScheduledTask() ||
                    $entry->isSlowQuery() ||
                    $entry->hasMonitoredTag();
                });
        });
    }

<a name="tagging"></a>
## Tagging

Telescope cho phép bạn tìm kiếm các entry theo "tag". Thông thường, các tag là các tên class của model Eloquent hoặc ID người dùng đã được xác thực mà Telescope tự động thêm vào các entry. Đôi khi, bạn có thể muốn đính kèm thêm các tag tùy chỉnh của bạn vào các entry. Để thực hiện điều này, bạn có thể sử dụng phương thức `Telescope::tag`. Phương thức `tag` chấp nhận một lệnh closure sẽ trả về một mảng tag. Các tag được closure trả về sẽ được merge với bất kỳ tag nào khác được Telescope tự động gắn vào entry. Thông thường, bạn nên gọi phương thức `tag` trong phương thức `register` của class `App\Providers\TelescopeServiceProvider` của bạn:

    use Laravel\Telescope\IncomingEntry;
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
            return $entry->type === 'request'
                        ? ['status:'.$entry->content['response_status']]
                        : [];
        });
     }

<a name="available-watchers"></a>
## Available Watchers

Telescope "watcher" sẽ thu thập dữ liệu ứng dụng của bạn khi một request hoặc một lệnh console được thực thi. Bạn có thể tùy chỉnh danh sách watcher mà bạn muốn bật trong file cấu hình `config/telescope.php` của bạn:

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

<a name="batch-watcher"></a>
### Batch Watcher

Batch watcher sẽ ghi lại thông tin về queued [batche](/docs/{{version}}/queues#job-batching), bao gồm cả thông tin về job và connection.

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

Dump watcher sẽ ghi lại và hiển thị các dump dữ liệu của bạn trong Telescope. Khi sử dụng Laravel, các biến có thể được dump bằng cách sử dụng hàm global `dump`. Tab dump watcher phải được mở trong trình duyệt để quá trình được ghi lại, nếu không dump watcher sẽ bỏ qua.

<a name="event-watcher"></a>
### Event Watcher

Event watcher sẽ ghi lại payload, listener và broadcast data cho bất kỳ [event](/docs/{{version}}/events) nào được ứng dụng của bạn gửi đi. Các event nội bộ của framework Laravel bị bỏ qua bởi Event watcher.

<a name="exception-watcher"></a>
### Exception Watcher

Exception watcher sẽ ghi lại dữ liệu và message lỗi cho bất kỳ exception report nào được ứng dụng của bạn đưa vào.

<a name="gate-watcher"></a>
### Gate Watcher

Gate watcher sẽ ghi lại dữ liệu và kết quả check của [gate và policy](/docs/{{version}}/authorization) bởi ứng dụng của bạn. Nếu bạn muốn watcher bỏ qua một số kiểm tra nhất định, bạn có thể chỉ định những kiểm tra này trong tùy chọn `ignore_abilities` của file `config/telescope.php` của bạn:

    'watchers' => [
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => ['viewNova'],
        ],
        ...
    ],

<a name="http-client-watcher"></a>
### HTTP Client Watcher

HTTP client watcher sẽ ghi lại [HTTP client requests](/docs/{{version}}/http-client) do ứng dụng của bạn thực hiện.

<a name="job-watcher"></a>
### Job Watcher

Job watcher sẽ ghi lại dữ liệu và trạng thái của bất kỳ [job](/docs/{{version}}/queues) nào mà ứng dụng của bạn gửi đi.

<a name="log-watcher"></a>
### Log Watcher

Log watcher sẽ ghi lại dữ liệu [log](/docs/{{version}}/logging) cho bất kỳ log nào được viết bởi ứng dụng của bạn.

<a name="mail-watcher"></a>
### Mail Watcher

Mail watcher cho phép bạn xem trước trong trình duyệt các [email](/docs/{{version}}/mail) đã được gửi bởi application của bạn cùng với dữ liệu của chúng. Bạn cũng có thể tải email xuống dưới dạng file `.eml`.

<a name="model-watcher"></a>
### Model Watcher

Model watcher sẽ ghi lại những thay đổi của model bất cứ khi nào một [model event](/docs/{{version}}/eloquent#events) Eloquent được gửi đi. Bạn có thể chỉ định các event nào của model sẽ được ghi lại thông qua tùy chọn `events` của watcher:

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
        ],
        ...
    ],

Nếu bạn muốn ghi lại số lượng model được tái tạo lại trong một request nhất định, hãy bật tùy chọn `hydrations`:

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
            'hydrations' => true,
        ],
        ...
    ],

<a name="notification-watcher"></a>
### Notification Watcher

Notification watcher sẽ ghi lại tất cả các [thông báo](/docs/{{version}}/notifications) do ứng dụng của bạn gửi đi. Nếu thông báo đó kích hoạt một email và bạn đã enable mail watcher, thì email đó cũng sẽ có thể xem trên màn hình mail watcher.

<a name="query-watcher"></a>
### Query Watcher

Query watcher sẽ ghi lại các raw SQL, binding và thời gian thực thi cho tất cả các truy vấn được ứng dụng của bạn chạy. Watcher cũng gắn thẻ vào bất kỳ truy vấn nào mà chậm hơn 100 milliseconds là `slow`. Bạn có thể tùy chỉnh ngưỡng mà truy vấn được coi là chậm bằng cách sử dụng tùy chọn `slow` của watcher:

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 50,
        ],
        ...
    ],

<a name="redis-watcher"></a>
### Redis Watcher

Redis watcher sẽ ghi lại tất cả các lệnh [Redis](/docs/{{version}}/redis) được thực thi bởi ứng dụng của bạn. Nếu bạn đang sử dụng Redis để lưu vào cache, thì các cache command cũng sẽ được Redis Watcher ghi lại.

<a name="request-watcher"></a>
### Request Watcher

Request watcher sẽ ghi lại dữ liệu request, header, session và response được liên kết với bất kỳ request nào được ứng dụng của bạn xử lý. Bạn có thể giới hạn dữ liệu response được ghi lại thông qua tùy chọn `size_limit` (tính bằng kilobytes):

    'watchers' => [
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
        ],
        ...
    ],

<a name="schedule-watcher"></a>
### Schedule Watcher

Schedule watcher sẽ ghi lại các lệnh và output của bất kỳ [task schedule](/docs/{{version}}/scheduling) nào mà do ứng dụng của bạn chạy.

<a name="view-watcher"></a>
### View Watcher

View watcher sẽ ghi lại tên, đường dẫn, dữ liệu và "composers" của [view](/docs/{{version}}/views) được sử dụng khi tạo view.

<a name="displaying-user-avatars"></a>
## Displaying User Avatars

Trang tổng quan của Telescope sẽ hiển thị ảnh đại diện của người dùng cho những người dùng đã xác thực khi một mục nào đó được lưu. Mặc định, Telescope sẽ lấy ảnh đại diện bằng dịch vụ Gravatar web. Tuy nhiên, bạn có thể tùy chỉnh URL hình đại diện bằng cách đăng ký một lệnh callback trong class `App\Providers\TelescopeServiceProvider` của bạn. Lệnh callback này sẽ nhận vào một ID và một địa chỉ email của người dùng và sẽ trả về URL hình ảnh đại diện của người dùng:

    use App\Models\User;
    use Laravel\Telescope\Telescope;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        // ...

        Telescope::avatar(function ($id, $email) {
            return '/avatars/'.User::find($id)->avatar_path;
        });
    }
