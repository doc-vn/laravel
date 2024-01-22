# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
    - [Ngoại lệ](#exceptions)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 8](#laravel-8)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Laravel và các package khác của nó tuân theo [Phiên bản Semantic](https://semver.org). Các phiên bản được phát hành chính thức của framework được phát hành một năm một lần (khoảng tháng 2), trong khi các bản phát hành nhỏ hơn và các bản sửa lỗi có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ và các bản sửa lỗi sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `^8.0`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

<a name="exceptions"></a>
### Ngoại lệ

<a name="named-arguments"></a>
#### Named Arguments

Tại thời điểm này, chức năng [đặt tên cho tham số](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) của PHP không nằm trong nguyên tắc tương thích ngược của Laravel. Chúng tôi có thể đổi tên các tham số bất cứ khi nào để cải thiện codebase của Laravel. Do đó, việc sử dụng các kiểu đặt tên cho tham số khi gọi các phương thức của Laravel nên được thực hiện một cách cẩn trọng và nên hiểu rằng tên tham số có thể thay đổi trong tương lai.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với tất cả các bản phát hành chính thức, các bản sửa lỗi sẽ được cung cấp trong 18 tháng và các bản sửa lỗi bảo mật được cung cấp trong 2 năm. Đối với tất cả các thư viện, bao gồm cả Lumen, chỉ bản phát hành mới nhất mới nhận được các bản sửa lỗi. Ngoài ra, hãy xem các phiên bản cơ sở dữ liệu [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

| Version | PHP (*) | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- | --- |
| 6 (LTS) | 7.2 - 8.0 | ngày 3 tháng 9 năm 2019 | ngày 25 tháng 1 năm 2022 | ngày 6 tháng 9 năm 2022 |
| 7 | 7.2 - 8.0 | ngày 3 tháng 3 năm 2020 | ngày 6 tháng 10 năm 2020 | ngày 3 tháng 3 năm 2021 |
| 8 | 7.3 - 8.1 | ngày 8 tháng 9 năm 2020 | ngày 26 tháng 7 năm 2022 | ngày 24 tháng 1 năm 2023 |
| 9 | 8.0 - 8.1 | ngày 8 tháng 2 năm 2022 | ngày 8 tháng 8 năm 2023 | ngày 8 tháng 2 năm 2024 |
| 10 | 8.0 - 8.1 | ngày 7 tháng 2 năm 2023 | ngày 7 tháng 8 năm 2024 | ngày 8 tháng 2 năm 2025 |

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>End of life</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>Security fixes only</div>
    </div>
</div>

(*) Supported PHP versions

<a name="laravel-8"></a>
## Laravel 8

Laravel 8 tiếp tục những cải tiến được thực hiện trong Laravel 7.x bằng cách giới thiệu Laravel Jetstream, các class model factory, nén migration, job batch, cải thiện giới hạn tỷ lệ, cải tiến queue, Blade component động, pagination view bằng Tailwind, tương tác với thời gian trong các bài test, cải tiến cho `artisan serve`, cải tiến event listener, và một loạt các bản sửa lỗi và cải tiến khả năng sử dụng khác.

<a name="laravel-jetstream"></a>
### Laravel Jetstream

_Laravel Jetstream được xây dựng bởi [Taylor Otwell](https://github.com/taylorotwell)_.

[Laravel Jetstream](https://jetstream.laravel.com) cung cấp một scaffolding ứng dụng được thiết kế đẹp mắt cho Laravel. Jetstream cung cấp điểm khởi đầu hoàn hảo cho dự án tiếp theo của bạn và bao gồm đăng nhập, đăng ký, xác minh email, xác thực hai yếu tố, quản lý session, hỗ trợ API thông qua Laravel Sanctum và tuỳ chọn quản lý team. Laravel Jetstream thay thế và cải tiến dựa trên scaffolding UI xác thực cũ có sẵn trong các phiên bản trước đó của Laravel.

Jetstream được thiết kế bằng [Tailwind CSS](https://tailwindcss.com) và cho bạn lựa chọn [Livewire](https://laravel-livewire.com) hoặc [Inertia.js](https://inertiajs.com) để chạy frontend scaffolding.

<a name="models-directory"></a>
### Models Directory

Do nhu cầu quá lớn của cộng đồng, Laravel framework mặc định chứa thư mục `app/Models`. Chúng tôi hy vọng bạn sẽ thích ngôi nhà mới này cho các model Eloquent của bạn! Tất cả các lệnh tạo model liên quan đã được cập nhật để giả sử rằng các model tồn tại trong thư mục `app/Models` nếu thư mục đó tồn tại. Nếu thư mục đó không tồn tại, thì framework sẽ giả sử rằng các model của bạn đang được lưu trong thư mục `app`.

<a name="model-factory-classes"></a>
### Model Factory Classes

_Model factory classes được xây dựng bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Eloquent [model factory](/docs/{{version}}/database-testing#defining-model-factories) đã được viết lại hoàn toàn dưới dạng các factory dựa trên class và được cải tiến để ưu tiên hỗ trợ quan hệ. Ví dụ: `UserFactory` đi kèm với Laravel được viết như sau:

    <?php

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * The name of the factory's corresponding model.
         *
         * @var string
         */
        protected $model = User::class;

        /**
         * Define the model's default state.
         *
         * @return array
         */
        public function definition()
        {
            return [
                'name' => $this->faker->name(),
                'email' => $this->faker->unique()->safeEmail(),
                'email_verified_at' => now(),
                'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
                'remember_token' => Str::random(10),
            ];
        }
    }

Nhờ trait `HasFactory` mới có sẵn trên các model được tạo, nên model factory có thể được sử dụng như sau:

    use App\Models\User;

    User::factory()->count(50)->create();

Vì các model factory hiện nay là các class PHP đơn giản nên các phép biến đổi trạng thái có thể được viết dưới dạng các phương thức của class. Ngoài ra, bạn có thể thêm bất kỳ class helper nào khác vào model factory Eloquent của bạn nếu cần.

Ví dụ: model `User` của bạn có thể có trạng thái `suspended` và sửa một trong các giá trị thuộc tính mặc định của nó. Bạn có thể định nghĩa các phép biến đổi trạng thái của bạn bằng phương thức `state` của base factory. Bạn có thể đặt tên cho phương thức trạng thái của bạn bằng bất kỳ tên gì mà bạn thích. Xét cho cùng, đây cũng chỉ là một phương thức PHP điển hình:

    /**
     * Indicate that the user is suspended.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    public function suspended()
    {
        return $this->state([
            'account_status' => 'suspended',
        ]);
    }

Sau khi định nghĩa xong phương thức chuyển đổi trạng thái, chúng ta có thể sử dụng nó như sau:

    use App\Models\User;

    User::factory()->count(5)->suspended()->create();

Như đã đề cập, các model factory của Laravel 8 ưu tiên hỗ trợ cho các quan hệ. Vì vậy, giả sử model `User` của chúng ta có phương thức quan hệ `posts`, chúng ta có thể chỉ cần chạy đoạn code sau để tạo người dùng có ba bài đăng:

    $users = User::factory()
                ->hasPosts(3, [
                    'published' => false,
                ])
                ->create();

Để đơn giản hóa quá trình nâng cấp, package [laravel/legacy-factories](https://github.com/laravel/legacy-factories) đã được phát hành để cung cấp hỗ trợ cho phiên bản trước đó của các model factory trong Laravel 8.x.

Các factory đã được viết lại của Laravel chứa nhiều tính năng hơn mà chúng tôi nghĩ bạn sẽ thích. Để tìm hiểu thêm về các model factory, vui lòng tham khảo [tài liệu database testing](/docs/{{version}}/database-testing#defining-model-factories).

<a name="migration-squashing"></a>
### Migration Squashing

_Migration squashing được xây dựng bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Khi bạn xây dựng ứng dụng của bạn, bạn có thể bị tích tụ ngày càng nhiều file migration theo thời gian. Điều này có thể dẫn đến việc thư mục migration của bạn trở nên quá tải với hàng trăm file migration. Nếu bạn đang sử dụng MySQL hoặc PostgreSQL, giờ đây bạn có thể "nén" migration của bạn vào một file SQL duy nhất. Để bắt đầu, hãy chạy lệnh `schema:dump`:

    php artisan schema:dump

    // Dump the current database schema and prune all existing migrations...
    php artisan schema:dump --prune

Khi bạn chạy lệnh này, Laravel sẽ ghi ra một file "schema" vào thư mục `database/schema` trong ứng dụng của bạn. Bây giờ, khi bạn chạy migrate cơ sở dữ liệu của bạn mà chưa chạy file migration nào khác, thì Laravel sẽ chạy các câu lệnh SQL trong file schema trước tiên. Sau khi chạy xong các câu lệnh của file schema, Laravel sẽ chạy tiếp các file migrate còn lại mà không có trong schema dump.

<a name="job-batching"></a>
### Job Batching

_Job batching được xây dựng bởi [Taylor Otwell](https://github.com/taylorotwell) và [Mohamed Said](https://github.com/themsaid)_.

Tính năng job batch của Laravel cho phép bạn dễ dàng thực hiện một loạt các job và sau đó thực hiện một số hành động sau khi một loạt các job đó đã hoàn thành việc chạy.

Phương thức `batch` mới của facade `Bus` có thể được sử dụng để gửi một loạt các job. Tất nhiên, việc tạo batch chủ yếu hữu ích khi kết hợp với các lệnh callback khi hoàn thành. Vì vậy, bạn có thể sử dụng các phương thức `then`, `catch` và `final` để định nghĩa các lệnh callback khi hoàn thành cho batch. Mỗi lệnh callback này sẽ nhận vào một instance `Illuminate\Bus\Batch` khi callback được gọi:

    use App\Jobs\ProcessPodcast;
    use App\Podcast;
    use Illuminate\Bus\Batch;
    use Illuminate\Support\Facades\Bus;
    use Throwable;

    $batch = Bus::batch([
        new ProcessPodcast(Podcast::find(1)),
        new ProcessPodcast(Podcast::find(2)),
        new ProcessPodcast(Podcast::find(3)),
        new ProcessPodcast(Podcast::find(4)),
        new ProcessPodcast(Podcast::find(5)),
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->catch(function (Batch $batch, Throwable $e) {
        // First batch job failure detected...
    })->finally(function (Batch $batch) {
        // The batch has finished executing...
    })->dispatch();

    return $batch->id;

Để tìm hiểu thêm về việc job batch, vui lòng tham khảo [tài liệu về queue](/docs/{{version}}/queues#job-batching).

<a name="improved-rate-limiting"></a>
### Improved Rate Limiting

_Rate limiting được cải tiến và xây dựng bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Tính năng giới hạn tỷ lệ request của Laravel đã được tăng cường với nhiều tính linh hoạt và nhiều sức mạnh hơn, trong khi vẫn duy trì khả năng tương thích ngược với các phiên bản API middleware `throttle` trước đó.

Bộ giới hạn tỷ lệ được định nghĩa bằng phương thức `for` của facade `RateLimiter`. Phương thức `for` sẽ chấp nhận tên của giới hạn tỷ lệ và một closure trả về cấu hình tỷ lệ giới hạn sẽ được áp dụng cho các route, những cái mà sẽ được chỉ định dùng giới hạn tỷ lệ này:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Support\Facades\RateLimiter;

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });

Vì lệnh callback giới hạn tỷ lệ nhận vào được một instance HTTP request nên bạn có thể xây dựng một giới hạn tỷ lệ phù hợp dựa trên request đến hoặc số lượng người dùng được xác thực:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

Thỉnh thoảng bạn có thể muốn phân giới hạn tỷ lệ theo một số giá trị tùy ý. Ví dụ: bạn có thể muốn cho phép người dùng truy cập vào một route nhất định 100 lần mỗi phút cho mỗi địa chỉ IP. Để thực hiện điều này, bạn có thể sử dụng phương thức `by` khi xây dựng giới hạn tỷ lệ của bạn:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

Giới hạn tỷ lệ có thể được gắn vào các route hoặc một nhóm route bằng cách sử dụng `throttle` [middleware](/docs/{{version}}/middleware). Middleware throttle sẽ chấp nhận tên của giới hạn tỷ lệ mà bạn muốn gán cho route:

    Route::middleware(['throttle:uploads'])->group(function () {
        Route::post('/audio', function () {
            //
        });

        Route::post('/video', function () {
            //
        });
    });

Để tìm hiểu thêm về giới hạn tỷ lệ, vui lòng tham khảo thêm [tài liệu route](/docs/{{version}}/routing#rate-limiting).

<a name="improved-maintenance-mode"></a>
### Improved Maintenance Mode

_Các cải tiến về chế độ bảo trì được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell) với cảm hứng từ [Spatie](https://spatie.be)_.

Trong các bản phát hành trước của Laravel, tính năng chế độ bảo trì `php artisan down` có thể bypass bằng cách sử dụng một "allow list" các địa chỉ IP được phép truy cập vào ứng dụng. Tính năng này đã bị loại bỏ để thay thế bằng giải pháp "secret" và token đơn giản hơn.

Khi ở chế độ bảo trì, bạn có thể sử dụng tùy chọn `secret` để chỉ định token để bypass chế độ bảo trì:

    php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"

Sau khi set ứng dụng của bạn vào chế độ bảo trì, bạn có thể vào URL ứng dụng khớp cùng với token và Laravel sẽ cấp cookie bypass chế độ bảo trì cho trình duyệt của bạn:

    https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515

Khi truy cập vào route ẩn này, bạn sẽ được chuyển hướng đến route `/` của ứng dụng. Khi cookie đã được cấp cho trình duyệt của bạn, bạn sẽ có thể xem ứng dụng bình thường như thể nó không có ở trong chế độ bảo trì.

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### Pre-Rendering The Maintenance Mode View

Nếu bạn sử dụng lệnh `php artisan down` trong khi deploy, người dùng của bạn đôi khi vẫn có thể gặp lỗi nếu họ truy cập ứng dụng trong khi các library của Composer hoặc các thành phần cơ sở hạ tầng khác của bạn đang cập nhật. Điều này xảy ra vì một phần quan trọng của Laravel framework phải khởi động để xác định ứng dụng của bạn có đang ở trong chế độ bảo trì hay không và hiển thị view chế độ bảo trì bằng cách sử dụng công cụ tạo template.

Vì lý do này, Laravel cho phép bạn tạo trước các view khi ở trong chế độ bảo trì và sẽ được trả về ngay khi bắt đầu chu trình của request. View này sẽ được hiển thị trước khi bất kỳ library nào của ứng dụng của bạn được load. Bạn có thể tạo trước một template của bạn chọn bằng cách sử dụng tùy chọn `render` trong lệnh `down`:

    php artisan down --render="errors::503"

<a name="closure-dispatch-chain-catch"></a>
### Closure Dispatch / Chain `catch`

_Các cải tiến về catch của queue job được đóng góp bởi [Mohamed Said](https://github.com/themsaid)_.

Bằng cách sử dụng phương thức `catch` mới, giờ đây bạn có thể cung cấp một closure sẽ được chạy nếu một queued closure bị thất bại sau khi dùng hết tất cả các lần thử lại đã được cấu hình trong queue của bạn:

    use Throwable;

    dispatch(function () use ($podcast) {
        $podcast->publish();
    })->catch(function (Throwable $e) {
        // This job has failed...
    });

<a name="dynamic-blade-components"></a>
### Dynamic Blade Components

_Dynamic Blade component được xây dựng bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Thỉnh thoảng bạn có thể cần tạo ra một component nhưng không biết component nào sẽ được tạo ra cho đến khi chạy. Trong tình huống này, bạn có thể sử dụng một component `dynamic-component` được tích hợp sẵn của Laravel để hiển thị component dựa vào giá trị trong thời gian chạy hoặc biến:

    <x-dynamic-component :component="$componentName" class="mt-4" />

Để tìm hiểu thêm về các Blade component, vui lòng tham khảo [tài liệu về Blade](/docs/{{version}}/blade#components).

<a name="event-listener-improvements"></a>
### Event Listener Improvements

_Các cải tiến về event listener được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Event listener dựa trên closure hiện có thể được đăng ký bằng cách chỉ truyền closure tới phương thức `Event::listen`. Laravel sẽ kiểm tra closure để xác định loại event nào mà listener sẽ xử lý:

    use App\Events\PodcastProcessed;
    use Illuminate\Support\Facades\Event;

    Event::listen(function (PodcastProcessed $event) {
        //
    });

Ngoài ra, event listener dựa trên closure hiện có thể được đánh dấu là xử lý bằng queue bằng cách sử dụng phương thức `Illuminate\Events\queueable`:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    }));

Giống như queued job, bạn có thể sử dụng các phương thức `onConnection`, `onQueue`, và `delay` để tùy chỉnh việc thực thi queued listener:

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));

Nếu bạn muốn xử lý các lỗi nonymous queued listener, bạn có thể cung cấp một closure cho phương thức `catch` trong khi định nghĩa listener `queueable`:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // The queued listener failed...
    }));

<a name="time-testing-helpers"></a>
### Time Testing Helpers

_Các tương tác với thời gian trong các bài test được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell) với cảm hứng từ Ruby on Rails_.

Khi kiểm tra, đôi khi bạn có thể cần sửa thời gian được trả về bởi helper, chẳng hạn như `now` hoặc `Illuminate\Support\Carbon::now()`. Rất may, class kiểm tra cơ bản của Laravel đã chứa các helper cho phép bạn thao tác với thời gian hiện tại:

    public function testTimeCanBeManipulated()
    {
        // Travel into the future...
        $this->travel(5)->milliseconds();
        $this->travel(5)->seconds();
        $this->travel(5)->minutes();
        $this->travel(5)->hours();
        $this->travel(5)->days();
        $this->travel(5)->weeks();
        $this->travel(5)->years();

        // Travel into the past...
        $this->travel(-5)->hours();

        // Travel to an explicit time...
        $this->travelTo(now()->subHours(6));

        // Return back to the present time...
        $this->travelBack();
    }

<a name="artisan-serve-improvements"></a>
### Artisan `serve` Improvements

_Các cải tiến về lệnh artisan `serve` được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Lệnh Artisan `serve` đã được cải tiến với tính năng tự động load lại khi phát hiện thấy có thay đổi về biến môi trường trong file `.env` local của bạn. Trước đây, lệnh này phải được dừng và khởi động lại mới cập nhật biến môi trường.

<a name="tailwind-pagination-views"></a>
### Tailwind Pagination Views

Mặc định, paginator của Laravel đã được cập nhật để sử dụng framework [Tailwind CSS](https://tailwindcss.com). Tailwind CSS là một framework CSS cấp thấp, có khả năng tùy chỉnh cao, cung cấp cho bạn tất cả các block mà bạn cần để xây dựng các thiết kế của riêng bạn mà không có bất kỳ style khó chịu nào mà bạn phải đấu tranh để tuỳ chỉnh lại. Tất nhiên, view Bootstrap 3 và 4 vẫn có sẵn.

<a name="routing-namespace-updates"></a>
### Routing Namespace Updates

Trong các bản phát hành trước của Laravel, `RouteServiceProvider` có chứa thuộc tính `$namespace`. Giá trị của thuộc tính này sẽ tự động được thêm vào trong các tiền tố của các định nghĩa route của controller và các lệnh gọi đến phương thức helper `action` và `URL::action`. Trong Laravel 8.x, thuộc tính này mặc định là `null`. Điều này có nghĩa là Laravel sẽ không thực hiện việc tự động thêm tiền tố namespace. Do đó, trong các ứng dụng Laravel 8.x mới, định nghĩa route của controller phải được định nghĩa bằng cú pháp PHP tiêu chuẩn:

    use App\Http\Controllers\UserController;

    Route::get('/users', [UserController::class, 'index']);

Các lệnh gọi đến các phương thức liên quan đến `action` cũng phải sử dụng cùng một cú pháp:

    action([UserController::class, 'index']);

    return Redirect::action([UserController::class, 'index']);

Nếu bạn thích tiền tố route của controller theo kiểu Laravel 7.x, bạn có thể chỉ cần thêm thuộc tính `$namespace` vào `RouteServiceProvider` của ứng dụng của bạn.

> {note} Thay đổi này chỉ ảnh hưởng đến các ứng dụng Laravel 8.x mới. Các ứng dụng mà nâng cấp từ Laravel 7.x vẫn sẽ có thuộc tính `$namespace` trong `RouteServiceProvider` của chúng.
