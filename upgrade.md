# Upgrade Guide

- [Nâng cấp đến 10.0 từ 9.x](#upgrade-10.0)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Updating Dependencies](#updating-dependencies)
- [Updating Minimum Stability](#updating-minimum-stability)

</div>

<a name="medium-impact-changes"></a>
## Những thay đổi có tác động trung bình

<div class="content-list" markdown="1">

- [Database Expressions](#database-expressions)
- [Model "Dates" Property](#model-dates-property)
- [Monolog 3](#monolog-3)
- [Redis Cache Tags](#redis-cache-tags)
- [Service Mocking](#service-mocking)
- [The Language Directory](#language-directory)

</div>

<a name="low-impact-changes"></a>
## Những thay đổi có tác động thấp

<div class="content-list" markdown="1">

- [Closure Validation Rule Messages](#closure-validation-rule-messages)
- [Form Request `after` Method](#form-request-after-method)
- [Public Path Binding](#public-path-binding)
- [Query Exception Constructor](#query-exception-constructor)
- [Rate Limiter Return Values](#rate-limiter-return-values)
- [The `Redirect::home` Method](#redirect-home)
- [The `Bus::dispatchNow` Method](#dispatch-now)
- [The `registerPolicies` Method](#register-policies)
- [ULID Columns](#ulid-columns)

</div>

<a name="upgrade-10.0"></a>
## Nâng cấp đến 10.0 từ 9.x

<a name="estimated-upgrade-time-30-minutes"></a>
#### Estimated Upgrade Time: 30 Minutes

> [!NOTE]
> Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này mới có thể thực sự ảnh hưởng đến application của bạn. Bạn muốn tiết kiệm thời gian? Bạn có thể sử dụng [Laravel Shift](https://laravelshift.com/) để giúp tự động hóa việc nâng cấp ứng dụng của bạn.

<a name="updating-dependencies"></a>
### Updating Dependencies

**Likelihood Of Impact: High**

#### PHP 8.1.0 Required

Laravel hiện yêu cầu PHP 8.1.0 trở lên.

#### Composer 2.2.0 Required

Laravel hiện yêu cầu [Composer](https://getcomposer.org) 2.2.0 trở lên.

#### Composer Dependencies

Bạn nên cập nhật các library sau vào file `composer.json` của ứng dụng:

<div class="content-list" markdown="1">

- `laravel/framework` to `^10.0`
- `laravel/sanctum` to `^3.2`
- `doctrine/dbal` to `^3.0`
- `spatie/laravel-ignition` to `^2.0`
- `laravel/passport` to `^11.0` ([Upgrade Guide](https://github.com/laravel/passport/blob/11.x/UPGRADE.md))

</div>

Nếu bạn đang nâng cấp lên Sanctum 3.x từ phiên bản phát hành 2.x, vui lòng tham khảo [hướng dẫn nâng cấp Sanctum](https://github.com/laravel/sanctum/blob/3.x/UPGRADE.md).

Ngoài ra, nếu bạn đang muốn sử dụng [PHPUnit 10](https://phpunit.de/announcements/phpunit-10.html), thì bạn nên xóa thuộc tính `processUncoveredFiles` ra khỏi phần `<coverage>` trong file cấu hình `phpunit.xml` của ứng dụng. Sau đó, hãy cập nhật các library sau trong file `composer.json` của ứng dụng:

<div class="content-list" markdown="1">

- `nunomaduro/collision` to `^7.0`
- `phpunit/phpunit` to `^10.0`

</div>

Cuối cùng, hãy kiểm tra tất cả các package của bên thứ ba mà ứng dụng của bạn sử dụng và xác nhận rằng bạn đang sử dụng phiên bản phù hợp để hỗ trợ Laravel 10.

<a name="updating-minimum-stability"></a>
#### Minimum Stability

Bạn nên cập nhật cài đặt `minimum-stability` trong file `composer.json` của ứng dụng thành `stable`. Hoặc, nếu giá trị mặc định của `minimum-stability` là `stable`, bạn cũng có thể xóa cài đặt này ra khỏi file `composer.json` của ứng dụng:

```json
"minimum-stability": "stable",
```

### Application

<a name="public-path-binding"></a>
#### Public Path Binding

**Likelihood Of Impact: Low**

Nếu ứng dụng của bạn đang tùy chỉnh "public path" bằng cách liên kết `path.public` vào trong container, thay vào đó bạn nên cập nhật code của bạn gọi phương thức `usePublicPath` do đối tượng `Illuminate\Foundation\Application` cung cấp:

```php
app()->usePublicPath(__DIR__.'/public');
```

### Authorization

<a name="register-policies"></a>
### The `registerPolicies` Method

**Likelihood Of Impact: Low**

Phương thức `registerPolicies` của `AuthServiceProvider` bây giờ đã được framework gọi mặc định. Vì vậy, bạn có thể xóa code gọi đến phương thức này trong phương thức `boot` của `AuthServiceProvider` trong ứng dụng của bạn.

### Cache

<a name="redis-cache-tags"></a>
#### Redis Cache Tags

**Likelihood Of Impact: Medium**

Việc sử dụng `Cache::tags()` chỉ được khuyến cáo cho các ứng dụng sử dụng Memcached. Nếu bạn đang sử dụng Redis làm driver cache cho ứng dụng, bạn nên cân nhắc chuyển sang Memcached hoặc sử dụng một giải pháp thay thế.

### Database

<a name="database-expressions"></a>
#### Database Expressions

**Likelihood Of Impact: Medium**

Database "expressions" (thường được tạo thông qua `DB::raw`) đã được viết lại trong Laravel 10.x để cung cấp thêm chức năng trong tương lai. Điều quan trọng, giá trị chuỗi raw của expression bây giờ phải được lấy ra thông qua phương thức `getValue(Grammar $grammar)` của expression. Việc ép kiểu expression thành chuỗi bằng `(string)` sẽ không còn được hỗ trợ nữa.

**Thông thường, điều này không ảnh hưởng đến các ứng dụng của bạn**; tuy nhiên, nếu ứng dụng của bạn đang ép kiểu thủ công các database expression liệu thành chuỗi thông qua cách sử dụng `(string)` hoặc gọi trực tiếp phương thức `__toString` trong biểu thức, thì bạn nên cập nhật code của bạn gọi đến phương thức `getValue` để thay thế:

```php
use Illuminate\Support\Facades\DB;

$expression = DB::raw('select 1');

$string = $expression->getValue(DB::connection()->getQueryGrammar());
```

<a name="query-exception-constructor"></a>
#### Query Exception Constructor

**Likelihood Of Impact: Very Low**

Hàm constructor của `Illuminate\Database\QueryException` bây giờ sẽ chấp nhận tên kết nối làm tham số đầu tiên. Nếu ứng dụng của bạn đang đưa ra exception này, bạn nên điều chỉnh code cho phù hợp.

<a name="ulid-columns"></a>
#### ULID Columns

**Likelihood Of Impact: Low**

Khi các migration gọi phương thức `ulid` mà không có bất kỳ tham số nào, thì cột sẽ được đặt tên là `ulid`. Trong các phiên bản Laravel trước, việc gọi phương thức này mà không có bất kỳ tham số nào đã tạo ra một cột được đặt tên sai là `uuid`:

    $table->ulid();

Để chỉ định rõ ràng tên cột khi gọi phương thức `ulid`, bạn có thể truyền tên cột đó cho phương thức:

    $table->ulid('ulid');

### Eloquent

<a name="model-dates-property"></a>
#### Model "Dates" Property

**Likelihood Of Impact: Medium**

Thuộc tính `$dates` của model Eloquent đã bị loại bỏ trước đó, đã bị xóa. Ứng dụng của bạn bây giờ sẽ sử dụng thuộc tính `$casts`:

```php
protected $casts = [
    'deployed_at' => 'datetime',
];
```

### Localization

<a name="language-directory"></a>
#### The Language Directory

**Likelihood Of Impact: None**

Mặc dù không liên quan đến các ứng dụng hiện có, mặc định, framework Laravel đã không còn chứa thư mục `lang` . Thay vào đó, khi viết các ứng dụng Laravel mới, bạn có thể tạo ra thư mục đó bằng lệnh Artisan `lang:publish`:

```shell
php artisan lang:publish
```

### Logging

<a name="monolog-3"></a>
#### Monolog 3

**Likelihood Of Impact: Medium**

Library Monolog của Laravel đã được cập nhật lên Monolog 3.x. Nếu bạn đang tương tác trực tiếp với Monolog trong ứng dụng của bạn, bạn nên xem qua [hướng dẫn nâng cấp](https://github.com/Seldaek/monolog/blob/main/UPGRADE.md) của Monolog.

Nếu bạn đang sử dụng dịch vụ ghi log của bên thứ ba như BugSnag hoặc Rollbar, bạn có thể cần phải nâng cấp các package của bên thứ ba này lên phiên bản hỗ trợ Monolog 3.x và Laravel 10.x.

### Queues

<a name="dispatch-now"></a>
#### The `Bus::dispatchNow` Method

**Likelihood Of Impact: Low**

Các phương thức `Bus::dispatchNow` và `dispatch_now` đã bị loại bỏ trước đó, đã bị xóa. Thay vào đó, ứng dụng của bạn nên sử dụng các phương thức `Bus::dispatchSync` và `dispatch_sync` tương ứng.

<a name="dispatch-return"></a>
#### The `dispatch()` Helper Return Value

**Likelihood Of Impact: Low**

Việc gọi `dispatch` với một class không implement `Illuminate\Contracts\Queue` trước đây sẽ trả về kết quả của phương thức `handle` của class đó. Tuy nhiên, bây giờ phương thức này sẽ trả về một instance `Illuminate\Foundation\Bus\PendingBatch`. Bạn có thể sử dụng `dispatch_sync()` để thực hiện hành vi trước đó.

### Routing

<a name="middleware-aliases"></a>
#### Middleware Aliases

**Likelihood Of Impact: Optional**

Trong các ứng dụng Laravel mới, thuộc tính `$routeMiddleware` của class `App\Http\Kernel` đã được đổi tên thành `$middlewareAliases` để phản ánh rõ hơn mục đích của nó. Bạn có thể đổi tên thuộc tính này trong các ứng dụng hiện có của bạn; tuy nhiên, việc này không bắt buộc.

<a name="rate-limiter-return-values"></a>
#### Rate Limiter Return Values

**Likelihood Of Impact: Low**

Khi gọi phương thức `RateLimiter::attempt`, giá trị được trả về bởi closure cũng sẽ được phương thức trả về giá trị đó. Nếu không có giá trị nào hoặc `null` được trả về, thì phương thức `attempt` sẽ trả về `true`:

```php
$value = RateLimiter::attempt('key', 10, fn () => ['example'], 1);

$value; // ['example']
```

<a name="redirect-home"></a>
#### The `Redirect::home` Method

**Likelihood Of Impact: Very Low**

Phương thức `Redirect::home` đã bị loại bỏ trước đó, đã bị xóa. Thay vào đó, ứng dụng của bạn nên redirect đến một named route:

```php
return Redirect::route('home');
```

### Testing

<a name="service-mocking"></a>
#### Service Mocking

**Likelihood Of Impact: Medium**

Trait `MocksApplicationServices` đã bị loại bỏ trước đó, đã bị xóa ra khỏi framework. Trait này cung cấp các phương thức kiểm tra như `expectsEvents`, `expectsJobs` và `expectsNotifications`.

Nếu ứng dụng của bạn sử dụng các phương thức này, chúng tôi khuyên bạn nên chuyển sang sử dụng `Event::fake`, `Bus::fake` và `Notification::fake` tương ứng. Bạn có thể tìm hiểu thêm về cách làm giả thông qua fake trong tài liệu tương ứng cho từng loại mà bạn đang cố gắng làm giả.

### Validation

<a name="closure-validation-rule-messages"></a>
#### Closure Validation Rule Messages

**Likelihood Of Impact: Very Low**

Khi viết các quy tắc validation tùy chỉnh dựa trên closure, việc gọi hàm `$fail` nhiều lần sẽ làm thêm các thông báo vào một mảng thay vì ghi đè lên thông báo trước đó. Thông thường, điều này sẽ không ảnh hưởng đến ứng dụng của bạn.

Ngoài ra, hàm callback `$fail` bây giờ sẽ trả về một đối tượng. Nếu trước đó bạn đã khai báo kiểu trả về của hàm closure validation, điều này có thể yêu cầu bạn cập nhật lại khai báo kiểu:

```php
public function rules()
{
    'name' => [
        function ($attribute, $value, $fail) {
            $fail('validation.translation.key')->translate();
        },
    ],
}
```

<a name="validation-messages-and-closure-rules"></a>
#### Validation Messages and Closure Rules

**Likelihood Of Impact: Very Low**

Trước đây, bạn có thể gán thông báo lỗi cho một khóa khác bằng cách cung cấp một mảng cho hàm callback `$fail` được đưa vào trong Closure trong quy tắc validation. Tuy nhiên, bây giờ bạn nên cung cấp khóa làm tham số đầu tiên và thông báo lỗi làm tham số thứ hai:

```php
Validator::make([
    'foo' => 'string',
    'bar' => [function ($attribute, $value, $fail) {
        $fail('foo', 'Something went wrong!');
    }],
]);
```

<a name="form-request-after-method"></a>
#### Form Request After Method

**Likelihood Of Impact: Very Low**

Trong các form request, phương thức `after` hiện đã được [Laravel sử dụng](https://github.com/laravel/framework/pull/46757). Nếu form request của bạn định nghĩa thêm phương thức `after`, phương thức này nên được đổi tên hoặc sửa để sử dụng tính năng "after validation" mới trong form request Laravel.

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến.

Bạn có thể dễ dàng xem các thay đổi đó bằng [công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/9.x...10.x) và chọn bản cập nhật nào quan trọng với bạn. Tuy nhiên, nhiều thay đổi được hiển thị bởi công cụ so sánh GitHub là do chúng tôi áp dụng các kiểu dữ liệu PHP native. Những thay đổi này tương thích với các phiên bản trước đó và việc áp dụng chúng trong quá trình migration sang Laravel 10 là không bắt buộc.
