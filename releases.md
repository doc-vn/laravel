# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 11](#laravel-11)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Laravel và các package khác của nó tuân theo [Phiên bản Semantic](https://semver.org). Các phiên bản được phát hành chính thức của framework được phát hành một năm một lần (~Q1), trong khi các bản phát hành nhỏ hơn và các bản sửa lỗi có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ và các bản sửa lỗi sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `^11.0`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

<a name="named-arguments"></a>
#### Named Arguments

[Đặt tên cho tham số](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) không nằm trong nguyên tắc tương thích ngược của Laravel. Chúng tôi có thể đổi tên các tham số bất cứ khi nào để cải thiện codebase của Laravel. Do đó, việc sử dụng các kiểu đặt tên cho tham số khi gọi các phương thức của Laravel nên được thực hiện một cách cẩn trọng và nên hiểu rằng tên tham số có thể thay đổi trong tương lai.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với tất cả các bản phát hành chính thức, các bản sửa lỗi sẽ được cung cấp trong 18 tháng và các bản sửa lỗi bảo mật được cung cấp trong 2 năm. Đối với tất cả các thư viện, bao gồm cả Lumen, chỉ bản phát hành chính thức mới nhất mới nhận được các bản sửa lỗi. Ngoài ra, hãy xem các phiên bản cơ sở dữ liệu [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

<div class="overflow-auto">

| Version | PHP (*) | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- | --- |
| 9 | 8.0 - 8.2 | ngày 8 tháng 2 năm 2022 | ngày 8 tháng 8 năm 2023 | ngày 6 tháng 2 năm 2024 |
| 10 | 8.1 - 8.3 | ngày 14 tháng 2 năm 2023 | ngày 6 tháng 8 năm 2024 | ngày 4 tháng 2 năm 2025 |
| 11 | 8.2 - 8.4 | ngày 12 tháng 3 năm 2024 | ngày 3 tháng 9 năm 2025 | ngày 12 tháng 3 năm 2026 |
| 12 | 8.2 - 8.4 | 24 tháng 2 năm 2025 | 13 tháng 8 năm 2026 | 24 tháng 2 năm 2027 |

</div>

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

<a name="laravel-11"></a>
## Laravel 11

Laravel 11 tiếp tục những cải tiến đã có trong Laravel 10.x bằng cách giới thiệu cấu trúc ứng dụng tinh giản hơn, giới hạn tỷ lệ theo từng giây, health routing, rotation encryption key, nâng cấp queue testing, transport mail [Resend](https://resend.com), tích hợp validator Prompt, các lệnh Artisan mới, và nhiều hơn nữa. Ngoài ra, Laravel Reverb, một máy chủ WebSocket chính thức của chúng tôi có khả năng mở rộng, đã được giới thiệu để cung cấp các tính năng thời gian thực mạnh mẽ cho ứng dụng của bạn.

<a name="php-8"></a>
### PHP 8.2

Laravel 11.x yêu cầu phiên bản PHP tối thiểu là 8.2.

<a name="structure"></a>
### Streamlined Application Structure

_Cấu trúc tinh giản hơn của ứng dụng Laravel được phát triển bởi [Taylor Otwell](https://github.com/taylorotwell) và [Nuno Maduro](https://github.com/nunomaduro)_.

Laravel 11 giới thiệu một cấu trúc ứng dụng tinh giản cho các ứng dụng Laravel **mới**, mà không yêu cầu bất kỳ thay đổi nào đối với các ứng dụng hiện có. Cấu trúc ứng dụng mới nhằm cung cấp một trải nghiệm tinh gọn, hiện đại hơn, trong khi vẫn giữ lại nhiều khái niệm mà các nhà phát triển Laravel đã quen thuộc. Dưới đây chúng ta sẽ thảo luận về những điểm nổi bật của cấu trúc ứng dụng mới của Laravel.

#### The Application Bootstrap File

File `bootstrap/app.php` đã được làm mới thành một file cấu hình ứng dụng theo phong cách code-first. Từ file này, giờ đây bạn có thể tùy chỉnh routing, middleware, service providers, xử lý ngoại lệ và nhiều hơn thế cho ứng dụng của bạn. File này sẽ hợp nhất các cài đặt hành vi high-level của ứng dụng mà trước đó nằm rải rác trong cấu trúc file của ứng dụng:

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

<a name="service-providers"></a>
#### Service Providers

Thay vì cấu trúc ứng dụng Laravel mặc định chứa đến năm service provider, Laravel 11 chỉ chứa một `AppServiceProvider` duy nhất. Chức năng của các service provider còn lại của framework đã được tích hợp vào `bootstrap/app.php`, được xử lý tự động bởi framework, hoặc có thể được viết vào trong `AppServiceProvider` của ứng dụng của bạn.

Ví dụ, tính năng event discovery hiện đã được enable mặc định, giúp loại bỏ phần lớn việc bạn phải đăng ký các event và listener của event. Tuy nhiên, nếu bạn cần đăng ký event một cách thủ công, bạn có thể thực hiện việc đó trong `AppServiceProvider`. Tương tự, các route model binding hoặc authorization gate mà trước đây bạn có thể đã đăng ký trong `AuthServiceProvider` cũng có thể được đăng ký trong `AppServiceProvider`.

<a name="opt-in-routing"></a>
#### Opt-in API and Broadcast Routing

Mặc định các file route `api.php` và `channels.php` không còn tồn tại nữa, vì nhiều ứng dụng không yêu cầu các file này. Thay vào đó, chúng có thể được tạo ra bằng các lệnh Artisan đơn giản:

```shell
php artisan install:api

php artisan install:broadcasting
```

<a name="middleware"></a>
#### Middleware

Trước đây, các ứng dụng Laravel mới có chứa đến chín middleware. Các middleware này thực hiện nhiều nhiệm vụ khác nhau như xác thực request, trim các chuỗi input và xác thực CSRF token.

Trong Laravel 11, các middleware này đã được chuyển vào bên trong của framework, giúp cấu trúc ứng dụng của bạn gọn nhẹ hơn. Các phương thức mới để tùy chỉnh hành vi của các middleware này đã được thêm vào framework và bạn có thể tùy chỉnh chúng từ file `bootstrap/app.php` của ứng dụng:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(
        except: ['stripe/*']
    );

    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ])
})
```

Vì tất cả middleware có thể được tùy chỉnh dễ dàng thông qua file `bootstrap/app.php` của ứng dụng, nên nhu cầu về một class HTTP "kernel" đã được loại bỏ.

<a name="scheduling"></a>
#### Scheduling

Sử dụng facade `Schedule` mới, các scheduled task có thể được định nghĩa trực tiếp vào trong file `routes/console.php` của ứng dụng, giúp loại bỏ nhu cầu về một class console "kernel":

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->daily();
```

<a name="exception-handling"></a>
#### Exception Handling

Giống như routing và middleware, việc xử lý ngoại lệ giờ đây có thể được tùy chỉnh từ file `bootstrap/app.php` của ứng dụng thay vì một class exception handler, giúp giảm số lượng file có trong một ứng dụng Laravel mới:

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReport(MissedFlightException::class);

    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

<a name="base-controller-class"></a>
#### Base `Controller` Class

Class controller base có sẵn trong các ứng dụng Laravel mới đã được đơn giản hóa. Nó không còn kế thừa class `Controller` của Laravel, và các trait `AuthorizesRequests` và `ValidatesRequests` cũng đã bị loại bỏ, vì chúng có thể được đưa vào các controller riêng của ứng dụng nếu bạn muốn:

    <?php

    namespace App\Http\Controllers;

    abstract class Controller
    {
        //
    }

<a name="application-defaults"></a>
#### Application Defaults

Mặc định, các ứng dụng Laravel mới sẽ sử dụng SQLite để lưu cơ sở dữ liệu, cũng như driver `database` cho session, cache và queue của Laravel. Điều này cho phép bạn bắt đầu xây dựng ứng dụng của bạn ngay sau khi tạo một ứng dụng Laravel mới mà không yêu cầu cài đặt gì thêm cho phần mềm hoặc tạo thêm các database migration.

Ngoài ra, theo thời gian, các driver `database` cho các service này của Laravel đã trở nên đủ ổn định để sử dụng production trong nhiều ngữ cảnh ứng dụng; do đó, chúng cung cấp một sự lựa chọn hợp lý và thống nhất cho cả ứng dụng ở môi trường local và production.

<a name="reverb"></a>
### Laravel Reverb

_Laravel Reverb được phát triển bởi [Joe Dixon](https://github.com/joedixon)_.

[Laravel Reverb](https://reverb.laravel.com) mang đến khả năng giao tiếp WebSocket thời gian thực cực nhanh và có khả năng mở rộng trực tiếp vào ứng dụng Laravel của bạn, đồng thời cung cấp khả năng tích hợp liền mạch với bộ công cụ event broadcasting hiện có của Laravel, chẳng hạn như Laravel Echo.

```shell
php artisan reverb:start
```

Ngoài ra, Reverb cũng hỗ trợ mở rộng quy mô lớn theo chiều ngang thông qua khả năng publish và subscribe của Redis, cho phép bạn phân phối lưu lượng WebSocket của bạn trên nhiều máy chủ Reverb backend, tất cả đều hỗ trợ cho một ứng dụng duy nhất có dung lượng cao.

Để biết thêm thông tin về Laravel Reverb, vui lòng tham khảo tài liệu [Reverb](/docs/{{version}}/reverb).

<a name="rate-limiting"></a>
### Per-Second Rate Limiting

_Per-second rate limiting được đóng góp bởi [Tim MacDonald](https://github.com/timacdonald)_.

Laravel hiện đã hỗ trợ rate limiting "theo giây" cho tất cả các rate limiter, bao gồm cả những rate limiter cho HTTP request và các queued job. Trước đây, các rate limiter của Laravel bị giới hạn ở mức độ "theo phút":

```php
RateLimiter::for('invoices', function (Request $request) {
    return Limit::perSecond(1);
});
```

Để biết thêm thông tin về rate limiting trong Laravel, vui lòng tham khảo tài liệu [rate limiting](/docs/{{version}}/routing#rate-limiting).

<a name="health"></a>
### Health Routing

_Health routing được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Các ứng dụng Laravel 11 mới có chứa một lệnh routing `health`, lệnh này hướng dẫn Laravel định nghĩa một url health-check đơn giản có thể được gọi bởi các dịch vụ giám sát ứng dụng của bên thứ ba hoặc các hệ thống điều phối như Kubernetes. Mặc định, route này sẽ được chạy tại `/up`:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
)
```

Khi các HTTP request được gửi đến route này, Laravel cũng sẽ kích hoạt một event `DiagnosingHealth`, cho phép bạn thực hiện thêm các health check có liên quan đến ứng dụng của bạn.

<a name="encryption"></a>
### Graceful Encryption Key Rotation

_Graceful encryption key rotation được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Vì Laravel mã hóa tất cả các cookie, bao gồm cả session cookie của ứng dụng, nên về cơ bản mọi request đến ứng dụng Laravel đều dựa vào mã hóa. Tuy nhiên, vì lý do này, việc rotation encryption key của ứng dụng sẽ khiến tất cả người dùng bị logout ra khỏi ứng dụng. Ngoài ra, việc giải mã dữ liệu đã được mã hóa bằng encryption key trước đó cũng trở nên không thể.

Laravel 11 cho phép bạn định nghĩa các encryption key trước đó của ứng dụng dưới dạng một danh sách được phân tách bằng dấu phẩy thông qua biến môi trường `APP_PREVIOUS_KEYS`.

Khi mã hóa các giá trị, Laravel sẽ luôn sử dụng encryption key "hiện tại", nằm trong biến môi trường `APP_KEY`. Và khi giải mã các giá trị, Laravel trước tiên sẽ thử với encryption key hiện tại. Nếu giải mã thất bại, thì Laravel sẽ thử với tất cả các key trước đó cho đến khi nào một trong các key có thể giải mã được giá trị.

Cách tiếp cận giải mã linh hoạt này cho phép người dùng tiếp tục sử dụng ứng dụng của bạn mà không bị gián đoạn ngay cả khi encryption key của bạn được thay đổi.

Để biết thêm thông tin về mã hóa trong Laravel, vui lòng tham khảo tài liệu [encryption](/docs/{{version}}/encryption).

<a name="automatic-password-rehashing"></a>
### Automatic Password Rehashing

_Automatic password rehashing được đóng gói bởi [Stephen Rees-Carter](https://github.com/valorin)_.

Thuật toán hashing mật khẩu mặc định của Laravel là bcrypt. "Work factor" cho các hash bcrypt có thể được điều chỉnh thông qua file cấu hình `config/hashing.php` hoặc biến môi trường `BCRYPT_ROUNDS`.

Thông thường, bcrypt work factor nên được tăng dần theo thời gian khi sức mạnh xử lý của CPU hoặc GPU tăng lên. Nếu bạn muốn tăng bcrypt work factor cho ứng dụng của bạn, Laravel giờ đây sẽ tự động rehash lại mật khẩu người dùng một cách mượt mà khi người dùng authenticate trong ứng dụng của bạn.

<a name="prompt-validation"></a>
### Prompt Validation

_Prompt validator integration được đóng gói bởi [Andrea Marco Sartori](https://github.com/cerbero90)_.

[Laravel Prompts](/docs/{{version}}/prompts) là một package PHP giúp thêm các form đẹp và thân thiện với người dùng vào các ứng dụng command-line của bạn, với các tính năng giống như trình duyệt bao gồm placeholder text và validation.

Laravel Prompts hỗ trợ input validation thông qua closures:

```php
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

Tuy nhiên, điều này có thể trở nên rườm rà khi xử lý nhiều input hoặc các kịch bản validation phức tạp. Do đó, trong Laravel 11, bạn có thể tận dụng toàn bộ sức mạnh của [validator](/docs/{{version}}/validation) của Laravel khi validate các prompt input:

```php
$name = text('What is your name?', validate: [
    'name' => 'required|min:3|max:255',
]);
```

<a name="queue-interaction-testing"></a>
### Queue Interaction Testing

_Queue interaction testing được đóng gói bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Trước đây, việc kiểm tra xem một queued job đã được release hay chưa, đã delete hay chưa hoặc bị thất bại là một việc khá rườm rà và yêu cầu phải định nghĩa các queue fake và stub tùy biến. Tuy nhiên, trong Laravel 11, bạn có thể dễ dàng kiểm tra các tương tác queue này bằng cách sử dụng phương thức `withFakeQueueInteractions`:

```php
use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
```

Để biết thêm thông tin về việc test queue job, hãy tham khảo tài liệu [queue](/docs/{{version}}/queues#testing).

<a name="new-artisan-commands"></a>
### New Artisan Commands

_Class creation Artisan commands được đóng gói bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Các lệnh Artisan mới đã được thêm vào để cho phép tạo nhanh các class, enum, interface và trait:

```shell
php artisan make:class
php artisan make:enum
php artisan make:interface
php artisan make:trait
```

<a name="model-cast-improvements"></a>
### Model Casts Improvements

_Model casts improvements được đóng gói bởi [Nuno Maduro](https://github.com/nunomaduro)_.

Laravel 11 hỗ trợ định nghĩa các cast của model bằng một phương thức thay vì là một thuộc tính. Điều này cho phép định nghĩa các cast một cách linh hoạt và mạch lạc hơn, đặc biệt là khi sử dụng các cast có tham số:

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => AsCollection::using(OptionCollection::class),
                      // AsEncryptedCollection::using(OptionCollection::class),
                      // AsEnumArrayObject::using(OptionEnum::class),
                      // AsEnumCollection::using(OptionEnum::class),
        ];
    }

Để biết thêm thông tin về cast attribute, hãy tham khảo tài liệu [Eloquent](/docs/{{version}}/eloquent-mutators#attribute-casting).

<a name="the-once-function"></a>
### The `once` Function

_The `once` helper được đóng gói bởi [Taylor Otwell](https://github.com/taylorotwell)_ và _[Nuno Maduro](https://github.com/nunomaduro)_.

Hàm helper `once` sẽ chạy một callback được cung cấp và lưu kết quả vào bộ nhớ RAM trong suốt thời gian của request. Mọi lần gọi tiếp theo đến hàm `once` với cùng một callback sẽ trả về kết quả đã được lưu trước đó:

    function random(): int
    {
        return once(function () {
            return random_int(1, 1000);
        });
    }

    random(); // 123
    random(); // 123 (cached result)
    random(); // 123 (cached result)

Để biết thêm thông tin về hàm helper `once`, hãy tham khảo tài liệu [helpers](/docs/{{version}}/helpers#method-once).

<a name="database-performance"></a>
### Improved Performance When Testing With In-Memory Databases

_Improved in-memory database testing performance được đóng gói bởi [Anders Jenbo](https://github.com/AJenbo)_

Laravel 11 mang lại sự gia tăng tốc độ đáng kể khi sử dụng cơ sở dữ liệu SQLite `:memory:` trong quá trình testing. Để đạt được điều này, Laravel hiện duy trì một tham chiếu đến đối tượng PDO của PHP và tái sử dụng nó giữa các kết nối, điều này sẽ giúp giảm một nửa tổng thời gian chạy test.

<a name="mariadb"></a>
### Improved Support for MariaDB

_Improved support for MariaDB được đóng gói bởi [Jonas Staudenmeir](https://github.com/staudenmeir) và [Julius Kiekbusch](https://github.com/Jubeki)_

Laravel 11 cũng hỗ trợ cải tiến cho MariaDB. Trong các phiên bản Laravel trước đây, bạn có thể sử dụng MariaDB thông qua driver MySQL của Laravel. Tuy nhiên, Laravel 11 hiện nay đã chứa một driver MariaDB riêng chuyên cung cấp các cấu hình mặc định tốt hơn cho hệ thống cơ sở dữ liệu này.

Để biết thêm thông tin về các driver cơ sở dữ liệu của Laravel, hãy tham khảo tài liệu [database](/docs/{{version}}/database).

<a name="inspecting-database"></a>
### Inspecting Databases and Improved Schema Operations

_Improved schema operations and database inspection được đóng gói bởi [Hafez Divandari](https://github.com/hafezdivandari)_

Laravel 11 cung cấp thêm các phương thức dành cho thao tác và kiểm tra schema cơ sở dữ liệu, bao gồm việc sửa, đổi tên và xóa cột một cách trực tiếp. Hơn nữa, các kiểu spatial nâng cao, tên schema non-default và các phương thức native schema cũng được cung cấp để thao tác với bảng, view, cột, index và foreign key:

    use Illuminate\Support\Facades\Schema;

    $tables = Schema::getTables();
    $views = Schema::getViews();
    $columns = Schema::getColumns('users');
    $indexes = Schema::getIndexes('users');
    $foreignKeys = Schema::getForeignKeys('users');
