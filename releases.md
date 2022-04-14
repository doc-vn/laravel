# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 6](#laravel-6)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Laravel và các package khác của nó tuân theo [Phiên bản Semantic](https://semver.org). Các phiên bản được phát hành chính thức của framework được phát hành sáu tháng một lần (tháng 2 và tháng 8), trong khi các bản phát hành nhỏ hơn và các bản sửa lỗi có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ và các bản sửa lỗi sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `^6.0`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với các bản phát hành hỗ trợ dài hạn, chẳng hạn như Laravel 6, các bản sửa lỗi được cung cấp trong 2 năm và các bản sửa lỗi bảo mật được cung cấp trong 3 năm. Những bản phát hành này cung cấp các hỗ trợ và bảo trì dài nhất. Đối với các bản phát hành bình thường, các bản sửa lỗi được cung cấp trong 6 tháng và các bản sửa lỗi bảo mật được cung cấp trong 1 năm. Đối với tất cả các thư viện, bao gồm cả Lumen, chỉ bản phát hành mới nhất mới nhận được các bản sửa lỗi. Ngoài ra, hãy xem các phiên bản cơ sở dữ liệu [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

| Version | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- |
| 5.5 (LTS) | ngày 30 tháng 8 năm 2017 | ngày 30 tháng 8 năm 2019 | ngày 30 tháng 8 năm 2020 |
| 5.6 | ngày 7 tháng 2 năm 2018 | ngày 7 tháng 8 năm 2018 | ngày 7 tháng 2 năm 2019 |
| 5.7 | ngày 4 tháng 9 năm 2018 | ngày 4 tháng 3 năm 2019 | ngày 4 tháng 9 năm 2019 |
| 5.8 | ngày 26 tháng 2 năm 2019 | ngày 26 tháng 8 năm 2019 | ngày 26 tháng 2 năm 2020 |
| 6 (LTS) | ngày 3 tháng 9 năm 2019 | ngày 3 tháng 9 năm 2021 | ngày 3 tháng 9 năm 2022 |

<a name="laravel-6"></a>
## Laravel 6

Laravel 6 (LTS) sẽ tiếp tục những cải tiến được thực hiện trong Laravel 5.8 bằng cách giới thiệu phiên bản semantic, khả năng tương thích với [Laravel Vapor](https://vapor.laravel.com), cải thiện response authorization, job middleware, lazy collection, cải tiến về subquery, tách các frontend scaffolding thành một package Composer `laravel/ui`, và một loạt các bản sửa lỗi và cải tiến khả năng sử dụng khác.

### Semantic Versioning

Package Laravel framework (`laravel/framework`) bây giờ sẽ tuân theo tiêu chuẩn [phiên bản semantic](https://semver.org/). Điều này làm cho framework nhất quán với các package khác của Laravel đã tuân thủ theo tiêu chuẩn phiên bản này. Chu kỳ phát hành phiên bản của Laravel sẽ không bị thay đổi.

### Laravel Vapor Compatibility

_Laravel Vapor được xây dựng bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel 6 cung cấp khả năng tương thích với [Laravel Vapor](https://vapor.laravel.com), một nền tảng triển khai tự động mở rộng cho Laravel mà không cần quản lý server . Vapor tổng hợp sự phức tạp của việc quản lý các ứng dụng Laravel trên AWS Lambda, cũng như giao tiếp các ứng dụng đó với hàng đợi SQS, cơ sở dữ liệu, Redis cluster, network, CloudFront CDN, vv...

### Improved Exceptions Via Ignition

Laravel 6 sẽ đi kèm với [Ignition](https://github.com/facade/ignition), một trang hiển thi chi tiết về exception mã nguồn mở được tạo bởi Freek Van der Herten và Marcel Pociot. Ignition cung cấp nhiều lợi ích hơn so với các bản phát hành trước, chẳng hạn như cải tiến file lỗi Blade và xử lý số dòng, các giải pháp có thể chạy được cho các sự cố thường gặp, chỉnh sửa code, chia sẻ exception và một UX được cải tiến.

### Improved Authorization Responses

_Improved Authorization Responses đã được thực hiện bởi [Gary Green](https://github.com/garygreen)_.

Trong các bản phát hành trước của Laravel, rất khó để lấy ra và hiển thị các thông báo authorization tùy chỉnh cho end user. Điều này khiến end user khó lý giải chính xác lý do tại sao một request lại bị từ chối. Trong Laravel 6, việc này giờ đây dễ dàng hơn nhiều bằng cách sử dụng các thông báo response authorization và phương thức `Gate::inspect` mới. Ví dụ: cho phương thức policy sau:

    /**
     * Determine if the user can view the given flight.
     *
     * @param  \App\User  $user
     * @param  \App\Flight  $flight
     * @return mixed
     */
    public function view(User $user, Flight $flight)
    {
        return $this->deny('Explanation of denial.');
    }

Có thể dễ dàng lấy ra response và thông báo của policy authorization bằng cách sử dụng phương thức `Gate::inspect`:

    $response = Gate::inspect('view', $flight);

    if ($response->allowed()) {
        // User is authorized to view the flight...
    }

    if ($response->denied()) {
        echo $response->message();
    }

Ngoài ra, các thông báo tùy chỉnh này sẽ được tự động trả về giao diện người dùng của bạn khi sử dụng các phương thức helper như `$this->authorize` hoặc `Gate::authorize` từ các route hoặc controller của bạn.

### Job Middleware

_Job middleware were implemented by [Taylor Otwell](https://github.com/taylorotwell)_.

Job middleware cho phép bạn custom logic toàn bộ việc chạy các queued job, giảm việc viết code trong các job đó. Ví dụ: trong các bản phát hành trước của Laravel, bạn có thể đã bao bọc logic của phương thức `handle` của một job trong một lệnh callback có giới hạn tốc độ:

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
            info('Lock obtained...');

            // Handle job...
        }, function () {
            // Could not obtain lock...

            return $this->release(5);
        });
    }

Trong Laravel 6, logic này có thể được mở rộng thành một job middleware, cho phép bạn giữ cho phương thức `handle` job của bạn không có bất kỳ trách nhiệm nào về giới hạn tốc độ:

    <?php

    namespace App\Jobs\Middleware;

    use Illuminate\Support\Facades\Redis;

    class RateLimited
    {
        /**
         * Process the queued job.
         *
         * @param  mixed  $job
         * @param  callable  $next
         * @return mixed
         */
        public function handle($job, $next)
        {
            Redis::throttle('key')
                    ->block(0)->allow(1)->every(5)
                    ->then(function () use ($job, $next) {
                        // Lock obtained...

                        $next($job);
                    }, function () use ($job) {
                        // Could not obtain lock...

                        $job->release(5);
                    });
        }
    }

Sau khi tạo xong job middleware, chúng ta có thể được gắn chúng vào một job bằng cách trả lại chúng từ phương thức `middleware` của job:

    use App\Jobs\Middleware\RateLimited;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array
     */
    public function middleware()
    {
        return [new RateLimited];
    }

### Lazy Collections

_Lazy collection được thực hiện bởi [Joseph Silber](https://github.com/JosephSilber)_.

Nhiều nhà phát triển đã thích thú với [các phương thức collection](https://laravel.com/docs/collections) mạnh mẽ của Laravel. Để bổ sung cho class `Collection` vốn đã mạnh mẽ, class `LazyCollection` sử dụng [generators](https://www.php.net/manual/en/language.generators.overview.php) của PHP để cho phép bạn làm việc với bộ dữ liệu rất lớn trong khi vẫn giữ mức sử dụng bộ nhớ thấp.

Ví dụ: hãy tưởng tượng ứng dụng của bạn cần xử lý file log nhiều gigabyte trong khi tận dụng các phương thức của Laravel để phân tích cú pháp log. Thay vì đọc toàn bộ file vào bộ nhớ cùng một lúc, lazy collection có thể được sử dụng để chỉ giữ một phần nhỏ của file vào trong bộ nhớ tại một thời điểm nhất định:

    use App\LogEntry;
    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    })
    ->chunk(4)
    ->map(function ($lines) {
        return LogEntry::fromLines($lines);
    })
    ->each(function (LogEntry $logEntry) {
        // Process the log entry...
    });

Hoặc, hãy tưởng tượng bạn cần lặp 10.000 model Eloquent. Khi sử dụng collection truyền thống của Laravel, tất cả 10.000 model Eloquent sẽ được load vào trong bộ nhớ cùng một lúc:

    $users = App\User::all()->filter(function ($user) {
        return $user->id > 500;
    });

Tuy nhiên, trong laravel 6, phương thức `cursor` của query builder sẽ trả về một instance `LazyCollection`. Điều này cho phép bạn vẫn chạy một truy vấn duy nhất đối với cơ sở dữ liệu nhưng cũng chỉ giữ một model Eloquent được load vào trong bộ nhớ tại một thời điểm. Trong ví dụ này, lệnh callback `filter` không được thực thi cho đến khi chúng ta thực sự lặp từng user, cho phép giảm đáng kể mức sử dụng bộ nhớ:

    $users = App\User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

### Eloquent Subquery Enhancements

_Các cải tiến Eloquent subquery đã được thực hiện bởi [Jonathan Reinink](https://github.com/reinink)_.

Laravel 6 giới thiệu một số nâng cấp và cải tiến mới để hỗ trợ subquery cơ sở dữ liệu. Ví dụ, hãy tưởng tượng rằng chúng ta có một bảng các chuyến bay `destinations` và một bảng `flights` đến các destination. Bảng `flights` chứa một cột `arrived_at` cho biết thời điểm chuyến bay đến dest
Sử dụng chức năng select của subquery mới trong Laravel 6, chúng ta có thể lấy ra tất cả các `destinations` và tên của chuyến bay đã đến điểm đến đó gần đây nhất chỉ bằng một câu lệnh truy vấn duy nhất:

    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
    ])->get();

Ngoài ra, hàm `orderBy` của query builder cũng hỗ trợ các subquery. Chúng ta có thể sử dụng chức năng này để sắp xếp tất cả các điểm đến dựa trên thời điểm chuyến bay cuối cùng đến điểm đến đó. Một lần nữa, điều này có thể được thực hiện chỉ trong một truy vấn duy nhất đối với cơ sở dữ liệu:

    return Destination::orderByDesc(
        Flight::select('arrived_at')
            ->whereColumn('destination_id', 'destinations.id')
            ->orderBy('arrived_at', 'desc')
            ->limit(1)
    )->get();

### Laravel UI

Scaffolding frontend thường được cung cấp với các bản phát hành trước của Laravel đã được mở rộng thành một package Composer `laravel/ui`. Điều này cho phép các UI scaffolding được phát triển và tạo ra các phiên bản riêng biệt với framework chính. Do sự thay đổi này, mặc định, không có code của Bootstrap hoặc Vue nào xuất hiện trong scaffolding framework và lệnh `make:auth` cũng đã được mở rộng từ framework.

Để khôi phục lại scaffolding của Vue hoặc Bootstrap có trước đó trong các bản phát hành trước của Laravel, bạn có thể cài đặt package `laravel/ui` và sử dụng lệnh Artisan `ui` để cài đặt scaffolding frontend:

    composer require laravel/ui "^1.0" --dev

    php artisan ui vue --auth
