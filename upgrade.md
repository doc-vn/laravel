# Upgrade Guide

- [Nâng cấp đến 8.0 từ 7.x](#upgrade-8.0)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Model Factories](#model-factories)
- [Queue `retryAfter` Method](#queue-retry-after-method)
- [Queue `timeoutAt` Property](#queue-timeout-at-property)
- [Queue `allOnQueue` and `allOnConnection`](#queue-allOnQueue-allOnConnection)
- [Pagination Defaults](#pagination-defaults)
- [Seeder & Factory Namespaces](#seeder-factory-namespaces)

</div>

<a name="medium-impact-changes"></a>
## Những thay đổi có tác động trung bình

<div class="content-list" markdown="1">

- [PHP 7.3.0 Required](#php-7.3.0-required)
- [Failed Jobs Table Batch Support](#failed-jobs-table-batch-support)
- [Maintenance Mode Updates](#maintenance-mode-updates)
- [The `php artisan down --message` Option](#artisan-down-message)
- [The `assertExactJson` Method](#assert-exact-json-method)

</div>

<a name="upgrade-8.0"></a>
## Nâng cấp đến 8.0 từ 7.x

<a name="estimated-upgrade-time-15-minutes"></a>
#### Estimated Upgrade Time: 15 Minutes

> {note} Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này mới có thể thực sự ảnh hưởng đến application của bạn.

<a name="php-7.3.0-required"></a>
### PHP 7.3.0 Required

**Likelihood Of Impact: Medium**

Phiên bản PHP tối thiểu bây giờ là 7.3.0.

<a name="updating-dependencies"></a>
### Updating Dependencies

Cập nhật các library sau trong file `composer.json` của bạn:

<div class="content-list" markdown="1">

- `guzzlehttp/guzzle` to `^7.0.1`
- `facade/ignition` to `^2.3.6`
- `laravel/framework` to `^8.0`
- `laravel/ui` to `^3.0`
- `nunomaduro/collision` to `^5.0`
- `phpunit/phpunit` to `^9.0`

</div>

Các package sau đây có các bản phát hành mới để hỗ trợ Laravel 8. Nếu bạn có dùng các package này, hãy đọc qua các hướng dẫn nâng cấp của chúng trước khi nâng cấp Laravel 8:

<div class="content-list" markdown="1">

- [Horizon v5.0](https://github.com/laravel/horizon/blob/master/UPGRADE.md)
- [Passport v10.0](https://github.com/laravel/passport/blob/master/UPGRADE.md)
- [Socialite v5.0](https://github.com/laravel/socialite/blob/master/UPGRADE.md)
- [Telescope v4.0](https://github.com/laravel/telescope/blob/master/UPGRADE.md)

</div>

Ngoài ra, Laravel installer đã được cập nhật để hỗ trợ `composer create-project` và Laravel Jetstream. Mọi installer cũ hơn phiên bản 4.0 sẽ ngừng hoạt động sau tháng 10 năm 2020. Bạn nên nâng cấp installer global của bạn lên phiên bản `^4.0` càng sớm càng tốt.

Cuối cùng, kiểm tra bất kỳ package bên thứ 3 nào mà được ứng dụng của bạn sử dụng và xác nhận rằng bạn đang sử dụng phiên bản phù hợp của package để hỗ trợ Laravel 8.

<a name="collections"></a>
### Collections

<a name="the-isset-method"></a>
#### The `isset` Method

**Likelihood Of Impact: Low**

Để nhất quán với hành vi PHP thông thường, phương thức `offsetExists` của `Illuminate\Support\Collection` đã được cập nhật để sử dụng `isset` thay vì `array_key_exists`. Điều này có thể được thể hiện trong khi xử lý các item trong collection mà có giá trị `null`:

    $collection = collect([null]);

    // Laravel 7.x - true
    isset($collection[0]);

    // Laravel 8.x - false
    isset($collection[0]);

<a name="database"></a>
### Database

<a name="seeder-factory-namespaces"></a>
#### Seeder & Factory Namespaces

**Likelihood Of Impact: High**

Seeder và factory đã được đổi namespace. Để phù hợp với những thay đổi này, hãy thêm namespace `Database\Seeders` vào các class seeder của bạn. Ngoài ra, thư mục `database/seeds` trước đây nên được đổi thành tên `database/seeders`:

    <?php

    namespace Database\Seeders;

    use App\Models\User;
    use Illuminate\Database\Seeder;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Seed the application's database.
         *
         * @return void
         */
        public function run()
        {
            ...
        }
    }

Nếu bạn đang chọn sử dụng package `laravel/legacy-factories`, thì không cần phải thay đổi các class factory của bạn. Tuy nhiên, nếu bạn đang nâng cấp các factory của bạn, thì bạn nên thêm namespace `Database\Factories` này vào trong các class của bạn.

Tiếp theo, trong file `composer.json` của bạn, hãy xóa block `classmap` ra khỏi phần `autoload` và thêm ánh xạ thư mục namespace của class mới:

    "autoload": {
        "psr-4": {
            "App\\": "app/",
            "Database\\Factories\\": "database/factories/",
            "Database\\Seeders\\": "database/seeders/"
        }
    },

<a name="eloquent"></a>
### Eloquent

<a name="model-factories"></a>
#### Model Factories

**Likelihood Of Impact: High**

Tính năng [model factory](/docs/{{version}}/database-testing#defining-model-factories) của Laravel đã được viết lại hoàn toàn để hỗ trợ các class và không tương thích với các factory theo kiểu Laravel 7.x. Tuy nhiên, để đơn giản hóa quá trình nâng cấp, một package `laravel/legacy-factories` mới đã được tạo để tiếp tục sử dụng các factory hiện có của bạn với Laravel 8.x. Bạn có thể cài đặt package này thông qua Composer:

    composer require laravel/legacy-factories

<a name="the-castable-interface"></a>
#### The `Castable` Interface

**Likelihood Of Impact: Low**

Phương thức `castUsing` của interface `Castable` đã được cập nhật để chấp nhận một mảng tham số. Nếu bạn đang implement interface này, bạn nên cập nhật cách implement của bạn cho phù hợp:

    public static function castUsing(array $arguments);

<a name="increment-decrement-events"></a>
#### Increment / Decrement Events

**Likelihood Of Impact: Low**

Giờ đây, các event model liên quan đến các sự kiện "cập nhật" hoặc "lưu" sẽ được gửi đi khi chạy các phương thức `increment` hoặc `decrement` trên các instances model Eloquent.

<a name="events"></a>
### Events

<a name="the-event-service-provider-class"></a>
#### The `EventServiceProvider` Class

**Likelihood Of Impact: Low**

Nếu class `App\Providers\EventServiceProvider` của bạn chứa phương thức `register`, thì bạn nên đảm bảo rằng bạn đã gọi `parent::register` ở đầu phương thức này. Nếu không gọi, thì các event trong ứng dụng của bạn sẽ không được đăng ký.

<a name="the-dispatcher-contract"></a>
#### The `Dispatcher` Contract

**Likelihood Of Impact: Low**

Phương thức `listener` của contract `Illuminate\Contracts\Events\Dispatcher` đã được cập nhật để làm cho thuộc tính `$listener` trở thành tùy chọn. Thay đổi này được thực hiện để hỗ trợ tự động phát hiện các loại event được xử lý thông qua tham chiếu. Nếu bạn đang implement interface này, thì bạn nên cập nhật cách implement của bạn cho phù hợp:

    public function listen($events, $listener = null);

<a name="framework"></a>
### Framework

<a name="maintenance-mode-updates"></a>
#### Maintenance Mode Updates

**Likelihood Of Impact: Optional**

Chức năng [chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode) của Laravel đã được cải tiến trong Laravel 8.x. Hiện đã hỗ trợ tạo trước template chế độ bảo trì và loại bỏ khả năng người dùng gặp phải lỗi khi ứng dụng đang trong chế độ bảo trì. Tuy nhiên, để hỗ trợ điều này, các dòng code sau phải được thêm vào file `public/index.php` của bạn. Những dòng này phải được đặt ngay dưới định nghĩa hằng số `LARAVEL_START`:

    define('LARAVEL_START', microtime(true));

    if (file_exists($maintenance = __DIR__.'/../storage/framework/maintenance.php')) {
        require $maintenance;
    }

<a name="artisan-down-message"></a>
#### The `php artisan down --message` Option

**Likelihood Of Impact: Medium**

Tùy chọn `--message` của lệnh `php artisan down` đã bị xóa. Để thay thế, hãy xem xét [tạo trước view chế độ bảo trì của bạn](/docs/{{version}}/configuration#maintenance-mode) với message mà bạn chọn.

<a name="php-artisan-serve-no-reload-option"></a>
#### The `php artisan serve --no-reload` Option

**Likelihood Of Impact: Low**

Tùy chọn `--no-reload` đã được thêm vào lệnh `php artisan serve`. Điều này sẽ hướng dẫn máy chủ tích hợp sẵn trong php không load lại máy chủ khi phát hiện thấy có thay đổi file môi trường. Tùy chọn này chủ yếu hữu ích khi chạy test Laravel Dusk trong môi trường CI.

<a name="manager-app-property"></a>
#### Manager `$app` Property

**Likelihood Of Impact: Low**

Thuộc tính `$app` không được dùng của class `Illuminate\Support\Manager` đã bị xóa. Nếu bạn đang dùng thuộc tính này, thì bạn nên sử dụng thuộc tính `$container` thay thế.

<a name="the-elixir-helper"></a>
#### The `elixir` Helper

**Likelihood Of Impact: Low**

Helper `elixir` không được dùng đã bị xóa. Các ứng dụng vẫn sử dụng phương thức này được khuyến khích nâng cấp lên [Laravel Mix](https://github.com/JeffreyWay/laravel-mix).

<a name="mail"></a>
### Mail

<a name="the-sendnow-method"></a>
#### The `sendNow` Method

**Likelihood Of Impact: Low**

Phương thức `sendNow` không được dùng đã bị xóa. Thay vào đó, hãy sử dụng phương thức `send`.

<a name="pagination"></a>
### Pagination

<a name="pagination-defaults"></a>
#### Pagination Defaults

**Likelihood Of Impact: High**

Bây giờ, paginator mặc định sẽ sử dụng [framework CSS Tailwind](https://tailwindcss.com) để tạo css. Để tiếp tục sử dụng Bootstrap, bạn nên thêm code sau vào phương thức `boot` của `AppServiceProvider` trong ứng dụng của bạn:

    use Illuminate\Pagination\Paginator;

    Paginator::useBootstrap();

<a name="queue"></a>
### Queue

<a name="queue-retry-after-method"></a>
#### The `retryAfter` Method

**Likelihood Of Impact: High**

Để nhất quán với các tính năng khác của Laravel, phương thức `retryAfter` và thuộc tính `retryAfter` của các queued job, mailer, notification và listener đã được đổi tên thành `backoff`. Bạn nên cập nhật tên của phương thức và thuộc tính này trong các class có liên quan trong ứng dụng của bạn.

<a name="queue-timeout-at-property"></a>
#### The `timeoutAt` Property

**Likelihood Of Impact: High**

Thuộc tính `timeoutAt` của queued job, notification và listener đã được đổi tên thành `retryUntil`. Bạn nên cập nhật tên của thuộc tính này trong các class có liên quan trong ứng dụng của bạn.

<a name="queue-allOnQueue-allOnConnection"></a>
#### The `allOnQueue()` / `allOnConnection()` Methods

**Likelihood Of Impact: High**

Để nhất quán với các phương thức gửi khác, các phương thức `allOnQueue()` và `allOnConnection()` được sử dụng cùng với chuỗi job đã bị xóa. Thay vào đó, bạn có thể sử dụng các phương thức `onQueue()` và `onConnection()`. Những phương thức này nên được gọi trước khi gọi phương thức `dispatch`:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->onConnection('redis')->onQueue('podcasts')->dispatch();

Lưu ý rằng thay đổi này chỉ ảnh hưởng đến các đoạn code sử dụng phương thức `withChain`. Phương thức `allOnQueue()` và `allOnConnection()` vẫn khả dụng khi sử dụng helper global `dispatch()`.

<a name="failed-jobs-table-batch-support"></a>
#### Failed Jobs Table Batch Support

**Likelihood Of Impact: Optional**

Nếu bạn định sử dụng chức năng [job batching](/docs/{{version}}/queues#job-batching) của Laravel 8.x, bảng cơ sở dữ liệu `failed_jobs` của bạn sẽ cần phải được cập nhật. Đầu tiên, một cột `uuid` mới sẽ được thêm vào bảng của bạn:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('failed_jobs', function (Blueprint $table) {
        $table->string('uuid')->after('id')->nullable()->unique();
    });

Tiếp theo, tùy chọn cấu hình `failed.driver` trong file cấu hình `queue` của bạn phải được cập nhật thành `database-uuids`.

Ngoài ra, bạn có thể muốn tạo UUID cho các job bị thất bại của bạn:

    DB::table('failed_jobs')->whereNull('uuid')->cursor()->each(function ($job) {
        DB::table('failed_jobs')
            ->where('id', $job->id)
            ->update(['uuid' => (string) Illuminate\Support\Str::uuid()]);
    });

<a name="routing"></a>
### Routing

<a name="automatic-controller-namespace-prefixing"></a>
#### Automatic Controller Namespace Prefixing

**Likelihood Of Impact: Optional**

Trong các bản phát hành trước của Laravel, class `RouteServiceProvider` có chứa thuộc tính `$namespace` với giá trị là `App\Http\Controllers`. Giá trị của thuộc tính này sẽ được sử dụng để làm tiền tố thêm vào các khai báo route của controller và tạo các URL route của controller, chẳng hạn như khi gọi helper `action`.

Trong Laravel 8, mặc định, thuộc tính này được set thành `null`. Điều này cho phép các khai báo route controller của bạn sử dụng cú pháp PHP tiêu chuẩn, cung cấp hỗ trợ tốt hơn cho việc nhảy vào class controller trong nhiều IDE:

    use App\Http\Controllers\UserController;

    // Using PHP callable syntax...
    Route::get('/users', [UserController::class, 'index']);

    // Using string syntax...
    Route::get('/users', 'App\Http\Controllers\UserController@index');

Trong nhiều các trường hợp, điều này sẽ không ảnh hưởng đến các ứng dụng đang được nâng cấp vì `RouteServiceProvider` của bạn sẽ vẫn chứa thuộc tính `$namespace` với giá trị trước đó của nó. Tuy nhiên, nếu bạn đang nâng cấp ứng dụng của bạn bằng cách tạo một project Laravel hoàn toàn mới, bạn có thể coi đây là một thay đổi nghiêm trọng.

Nếu bạn muốn tiếp tục sử dụng route controller có tiền tố tự động thêm như các phiên bản trước, bạn chỉ cần set giá trị của thuộc tính `$namespace` trong `RouteServiceProvider` của bạn và cập nhật đăng ký route trong phương thức `boot` để sử dụng thuộc tính `$namespace`:

    class RouteServiceProvider extends ServiceProvider
    {
        /**
         * The path to the "home" route for your application.
         *
         * This is used by Laravel authentication to redirect users after login.
         *
         * @var string
         */
        public const HOME = '/home';

        /**
         * If specified, this namespace is automatically applied to your controller routes.
         *
         * In addition, it is set as the URL generator's root namespace.
         *
         * @var string
         */
        protected $namespace = 'App\Http\Controllers';

        /**
         * Define your route model bindings, pattern filters, etc.
         *
         * @return void
         */
        public function boot()
        {
            $this->configureRateLimiting();

            $this->routes(function () {
                Route::middleware('web')
                    ->namespace($this->namespace)
                    ->group(base_path('routes/web.php'));

                Route::prefix('api')
                    ->middleware('api')
                    ->namespace($this->namespace)
                    ->group(base_path('routes/api.php'));
            });
        }

        /**
         * Configure the rate limiters for the application.
         *
         * @return void
         */
        protected function configureRateLimiting()
        {
            RateLimiter::for('api', function (Request $request) {
                return Limit::perMinute(60)->by(optional($request->user())->id ?: $request->ip());
            });
        }
    }

<a name="scheduling"></a>
### Scheduling

<a name="the-cron-expression-library"></a>
#### The `cron-expression` Library

**Likelihood Of Impact: Low**

Library `dragonmantank/cron-expression` của Laravel đã được cập nhật từ `2.x` lên `3.x`. Điều này sẽ không gây ra bất kỳ thay đổi nghiêm trọng nào trong ứng dụng của bạn trừ khi bạn đang tương tác trực tiếp với thư viện `cron-expression`. Nếu bạn đang tương tác trực tiếp với thư viện này, vui lòng xem lại [nhật ký thay đổi](https://github.com/dragonmantank/cron-expression/blob/master/CHANGELOG.md).

<a name="session"></a>
### Session

<a name="the-session-contract"></a>
#### The `Session` Contract

**Likelihood Of Impact: Low**

Contract `Illuminate\Contracts\Session\Session` đã nhận được một phương thức `pull` mới. Nếu bạn đang implement contract này, bạn nên cập nhật cách implement của bạn cho phù hợp:

    /**
     * Get the value of a given key and then forget it.
     *
     * @param  string  $key
     * @param  mixed  $default
     * @return mixed
     */
    public function pull($key, $default = null);

<a name="testing"></a>
### Testing

<a name="decode-response-json-method"></a>
#### The `decodeResponseJson` Method

**Likelihood Of Impact: Low**

Phương thức `decodeResponseJson` thuộc class `Illuminate\Testing\TestResponse` sẽ không còn chấp nhận bất kỳ tham số nào. Thay vào đó, hãy cân nhắc sử dụng phương thức `json` để thay thế.

<a name="assert-exact-json-method"></a>
#### The `assertExactJson` Method

**Likelihood Of Impact: Medium**

Phương thức `assertExactJson` hiện yêu cầu các khóa numeric của mảng được so sánh phải khớp và có cùng thứ tự. Nếu bạn muốn so sánh JSON với một mảng mà không cần mảng đó có khóa numeric phải có cùng thứ tự, bạn có thể sử dụng phương thức `assertSimilarJson` thay thế.

<a name="validation"></a>
### Validation

<a name="database-rule-connections"></a>
### Database Rule Connections

**Likelihood Of Impact: Low**

Các quy tắc `unique` và `exists` bây giờ sẽ ưu tiên tên kết nối đã chỉ định (được truy cập thông qua phương thức `getConnectionName` của model) của các model Eloquent khi thực hiện truy vấn.

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [Công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/7.x...8.x) và chọn bản cập nhật nào quan trọng với bạn.
