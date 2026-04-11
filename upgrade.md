# Upgrade Guide

- [Nâng cấp đến 13.0 từ 12.x](#upgrade-13.0)
    - [Nâng cấp dùng AI](#upgrading-using-ai)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Updating Dependencies](#updating-dependencies)
- [Updating the Laravel Installer](#updating-the-laravel-installer)
- [Request Forgery Protection](#request-forgery-protection)

</div>

<a name="medium-impact-changes"></a>
## Medium Impact Changes

<div class="content-list" markdown="1">

- [Cache `serializable_classes` Configuration](#cache-serializable_classes-configuration)
- [Database `upsert` With MySQL or MariaDB](#database-upsert-mariadb-mysql)

</div>

<a name="low-impact-changes"></a>
## Low Impact Changes

<div class="content-list" markdown="1">

- [Cache Prefixes and Session Cookie Names](#cache-prefixes-and-session-cookie-names)
- [Collection Model Serialization Restores Eager-Loaded Relations](#collection-model-serialization-restores-eager-loaded-relations)
- [`Container::call` and Nullable Class Defaults](#containercall-and-nullable-class-defaults)
- [Domain Route Registration Precedence](#domain-route-registration-precedence)
- [`JobAttempted` Event Exception Payload](#jobattempted-event-exception-payload)
- [Manager `extend` Callback Binding](#manager-extend-callback-binding)
- [MySQL `DELETE` Queries With `JOIN`, `ORDER BY`, and `LIMIT`](#mysql-delete-queries-with-join-order-by-and-limit)
- [Pagination Bootstrap View Names](#pagination-bootstrap-view-names)
- [Polymorphic Pivot Table Name Generation](#polymorphic-pivot-table-name-generation)
- [`QueueBusy` Event Property Rename](#queuebusy-event-property-rename)
- [`Str` Factories Reset Between Tests](#str-factories-reset-between-tests)

</div>

<a name="upgrade-13.0"></a>
## Nâng cấp đến 13.0 từ 12.x

#### Estimated Upgrade Time: 5 Minutes

> [!NOTE]
> Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này mới có thể thực sự ảnh hưởng đến application của bạn. Để tiết kiệm thời gian, bạn có thể sử dụng [Shift](https://laravelshift.com). Shift là một dịch vụ được phát triển bởi cộng đồng giúp bạn tự động hóa việc nâng cấp Laravel.

<a name="upgrading-using-ai"></a>
### Nâng cấp dùng AI

Bạn có thể tự động hóa việc nâng cấp này bằng cách sử dụng [Laravel Boost](https://github.com/laravel/boost). Boost là một MCP server chính thức cung cấp cho trợ lý AI của bạn các gợi ý nâng cấp có hướng dẫn — sau khi cài đặt trong bất kỳ ứng dụng Laravel 12 nào, hãy sử dụng lệnh `/upgrade-laravel-v13` trong Claude Code, Cursor, OpenCode, Gemini hoặc VS Code để bắt đầu nâng cấp lên Laravel 13. Lệnh này sẽ yêu cầu Laravel Boost `^2.0`.

<a name="updating-dependencies"></a>
### Updating Dependencies

**Likelihood Of Impact: High**

Bạn nên cập nhật các library sau vào file `composer.json` của ứng dụng:

<div class="content-list" markdown="1">

- `laravel/framework` to `^13.0`
- `laravel/boost` to `^2.0`
- `laravel/tinker` to `^3.0`
- `phpunit/phpunit` to `^12.0`
- `pestphp/pest` to `^4.0`

</div>

<a name="updating-the-laravel-installer"></a>
### Updating the Laravel Installer

Nếu bạn đang sử dụng công cụ CLI Laravel installer để tạo các ứng dụng Laravel mới, bạn nên cập nhật bản cài đặt installer của bạn để tương thích với Laravel 13.x.

Nếu bạn đã cài đặt Laravel installer thông qua `composer global require`, bạn có thể cập nhật installer bằng lệnh `composer global update`:

```shell
composer global update laravel/installer
```

Hoặc, nếu bạn đang sử dụng bản copy Laravel installer đi kèm với [Laravel Herd](https://herd.laravel.com), bạn nên cập nhật bản cài đặt Herd của bạn để lên phiên bản mới nhất.

<a name="cache"></a>
### Cache

<a name="cache-prefixes-and-session-cookie-names"></a>
#### Cache Prefixes and Session Cookie Names

**Likelihood Of Impact: Low**

Các prefix mặc định cho cache và Redis của Laravel hiện sử dụng các hậu tố nối bằng dấu gạch ngang. Ngoài ra, tên cookie session mặc định hiện cũng sử dụng `Str::snake(...)` cho tên ứng dụng.

Trong hầu hết các ứng dụng, thay đổi này sẽ không ảnh hưởng vì các file cấu hình cấp ứng dụng đã định nghĩa sẵn các giá trị này. Điều này chủ yếu ảnh hưởng đến các ứng dụng phụ thuộc vào cấu hình dự phòng ở cấp độ framework khi các giá trị cấu hình ứng dụng tương ứng không tồn tại.

Nếu ứng dụng của bạn đang dựa trên các giá trị mặc định được tạo này, thì các key cache và tên cookie session có thể thay đổi sau khi nâng cấp như sau:

```php
// Laravel <= 12.x
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_cache_';
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_database_';
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_session';

// Laravel >= 13.x
Str::slug((string) env('APP_NAME', 'laravel')).'-cache-';
Str::slug((string) env('APP_NAME', 'laravel')).'-database-';
Str::snake((string) env('APP_NAME', 'laravel')).'_session';
```

Để giữ nguyên hành vi trước đó, hãy cấu hình các biến môi trường `CACHE_PREFIX`, `REDIS_PREFIX` và `SESSION_COOKIE` trong file môi trường của bạn.

<a name="store-and-repository-contracts-touch"></a>
#### `Store` and `Repository` Contracts: `touch`

**Likelihood Of Impact: Very Low**

Các contract cache bây giờ đã chứa thêm phương thức `touch` để gia hạn thời gian TTL cho item. Nếu bạn đang duy trì các implementation lưu trữ cache tùy biến, bạn nên thêm phương thức này:

```php
// Illuminate\Contracts\Cache\Store
public function touch($key, $seconds);
```

<a name="cache-serializable_classes-configuration"></a>
#### Cache `serializable_classes` Configuration

**Likelihood Of Impact: Medium**

Cấu hình `cache` mặc định của ứng dụng sẽ chứa tùy chọn `serializable_classes` được set thành `false`. Điều này sẽ thắt chặt hành vi chuyển hoá cache để giúp ngăn chặn các cuộc tấn công nối chuỗi có sẵn (gadget chain attack) trong PHP nếu `APP_KEY` của ứng dụng bị rò rỉ. Nếu ứng dụng của bạn muốn lưu các đối tượng PHP vào trong cache, bạn nên liệt kê rõ các class nào có thể được chuyển hoá:

```php
'serializable_classes' => [
    App\Data\CachedDashboardStats::class,
    App\Support\CachedPricingSnapshot::class,
],
```

Nếu ứng dụng của bạn trước đây phụ thuộc vào việc chuyển hoá các đối tượng cache tùy ý, bạn sẽ cần chuyển đổi cách sử dụng đó sang cho phép class hoặc sang các payload cache không phải đối tượng (chẳng hạn như mảng).

<a name="container"></a>
### Container

<a name="containercall-and-nullable-class-defaults"></a>
#### `Container::call` and Nullable Class Defaults

**Likelihood Of Impact: Low**

`Container::call` bây giờ sẽ ưu tiên các giá trị mặc định của tham số class nullable khi không có liên kết nào tồn tại, giống với hành vi injection constructor được giới thiệu trong Laravel 12:

```php
$container->call(function (?Carbon $date = null) {
    return $date;
});

// Laravel <= 12.x: Carbon instance
// Laravel >= 13.x: null
```

Nếu logic injection method-call của bạn phụ thuộc vào hành vi trước đó, bạn có thể cần cập nhật lại.

<a name="contracts"></a>
### Contracts

<a name="dispatcher-contract-dispatchafterresponse"></a>
#### `Dispatcher` Contract: `dispatchAfterResponse`

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Bus\Dispatcher` hiện chứa phương thức `dispatchAfterResponse($command, $handler = null)`.

Nếu bạn đang triển khai một implementation của dispatcher này, hãy thêm phương thức đó vào class của bạn.

<a name="responsefactory-contract-eventstream"></a>
#### `ResponseFactory` Contract: `eventStream`

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Routing\ResponseFactory` hiện chứa một signature `eventStream`.

Nếu bạn đang triển khai một implementation của response factory này, hãy thêm phương thức đó vào class của bạn.

<a name="mustverifyemail-contract-markemailasunverified"></a>
#### `MustVerifyEmail` Contract: `markEmailAsUnverified`

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Auth\MustVerifyEmail` hiện chứa `markEmailAsUnverified()`.

Nếu bạn cung cấp một implementation tùy chỉnh của contract này, hãy thêm phương thức này để duy trì tính tương thích.

<a name="database"></a>
### Database

<a name="database-upsert-mariadb-mysql"></a>
#### Database `upsert` With MySQL or MariaDB

**Likelihood Of Impact: Medium**

Laravel bây giờ sẽ validate câu lệnh upsert phải cung cấp giá trị không rỗng cho tham số  `uniqueBy`, và sẽ đưa ra một `InvalidArgumentException` thay vì tạo ra SQL không hợp lệ.

Mặc dù các database driver MariaDB và MySQL sẽ bỏ qua giá trị `uniqueBy` và luôn sử dụng các index primary và unique của table để phát hiện các bản ghi có tồn tại hay không, nhưng việc validation vẫn được áp dụng. Một `InvalidArgumentException` sẽ được đưa ra nếu `uniqueBy` trống.

<a name="mysql-delete-queries-with-join-order-by-and-limit"></a>
#### MySQL `DELETE` Queries With `JOIN`, `ORDER BY`, and `LIMIT`

**Likelihood Of Impact: Low**

Laravel hiện sẽ biên dịch đầy đủ các truy vấn `DELETE ... JOIN` bao gồm `ORDER BY` và `LIMIT` cho cú pháp MySQL.

Trong các phiên bản trước, các mệnh đề `ORDER BY` / `LIMIT` có thể bị bỏ qua một cách lặng lẽ trong các câu lệnh xóa có join. Trong Laravel 13, các mệnh đề này sẽ được đưa vào câu lệnh SQL được tạo ra. Kết quả là, các database engine không hỗ trợ cú pháp này (chẳng hạn như các biến thể MySQL / MariaDB tiêu chuẩn) có thể đưa ra một `QueryException` thay vì thực hiện một lệnh xóa không kiểm soát.

<a name="eloquent"></a>
### Eloquent

<a name="model-booting-and-nested-instantiation"></a>
#### Model Booting and Nested Instantiation

**Likelihood Of Impact: Very Low**

Việc tạo một instance model mới trong khi model đó vẫn đang booting hiện sẽ không còn được phép và sẽ đưa ra `LogicException`.

Điều này ảnh hưởng đến code khởi tạo model đến từ bên trong các phương thức `boot` của model hoặc các phương thức `boot*` của trait:

```php
protected static function boot()
{
    parent::boot();

    // No longer allowed during booting...
    (new static())->getTable();
}
```

Bạn nên di chuyển logic này ra ngoài chu kỳ boot để tránh việc boot lồng nhau.

<a name="polymorphic-pivot-table-name-generation"></a>
#### Polymorphic Pivot Table Name Generation

**Likelihood Of Impact: Low**

Khi tên bảng được suy luận cho các model pivot đa hình sử dụng các class model pivot tùy biến, Laravel sẽ suy luận tên bảng trên dạng số nhiều.

Nếu ứng dụng của bạn dùng các tên suy luận ở dạng số ít cho các bảng morph pivot và sử dụng các class pivot tùy chỉnh, bạn nên định nghĩa rõ ràng tên bảng này trên model pivot của bạn.

<a name="collection-model-serialization-restores-eager-loaded-relations"></a>
#### Collection Model Serialization Restores Eager-Loaded Relations

**Likelihood Of Impact: Low**

Khi các collection model Eloquent được chuyển đổi và khôi phục (chẳng hạn như trong các queued job), các quan hệ eager-load bây giờ sẽ được khôi phục theo các model của collection đó.

Nếu code của bạn có logic về việc kiểm tra quan hệ không tồn tại sau khi khôi phục, thì bạn có thể cần điều chỉnh lại logic đó.

<a name="http-client"></a>
### HTTP Client

<a name="http-client-response-throw-and-throwif-signatures"></a>
#### HTTP Client `Response::throw` and `throwIf` Signatures

**Likelihood Of Impact: Very Low**

Các phương thức response của HTTP client bây giờ sẽ khai báo các tham số callback của chúng trong định dạng của phương thức:

```php
public function throw($callback = null);
public function throwIf($condition, $callback = null);
```

Nếu bạn đang ghi đè các phương thức này trong các class response của bạn, bạn hãy đảm bảo các định dạng phương thức của bạn tương thích với định dạng mới.

<a name="notifications"></a>
### Notifications

<a name="default-password-reset-subject"></a>
#### Default Password Reset Subject

**Likelihood Of Impact: Very Low**

Tiêu đề email mặc định khi Laravel gửi reset mật khẩu đã được thay đổi:

```text
// Laravel <= 12.x
Reset Password Notification

// Laravel >= 13.x
Reset your password
```

Nếu các bài test, kiểm tra hoặc các bản dịch của bạn phụ thuộc vào chuỗi mặc định trước đó, bạn hãy cập nhật chúng sao cho tương ứng.

<a name="queued-notifications-and-missing-models"></a>
#### Queued Notifications and Missing Models

**Likelihood Of Impact: Very Low**

Các queued notification bây giờ sẽ ưu tiên thuộc tính `#[DeleteWhenMissingModels]` và thuộc tính `$deleteWhenMissingModels` được định nghĩa trên class notification.

Trong các phiên bản trước, việc thiếu model có thể là nguyên nhân khiến các queued notification job bị thất bại thay vì bị xóa.

<a name="queue"></a>
### Queue

<a name="jobattempted-event-exception-payload"></a>
#### `JobAttempted` Event Exception Payload

**Likelihood Of Impact: Low**

Event `Illuminate\Queue\Events\JobAttempted` hiện sẽ cung cấp đối tượng exception (hoặc `null`) thông qua `$exception`, thay thế cho thuộc tính boolean `$exceptionOccurred` trước đó:

```php
// Laravel <= 12.x
$event->exceptionOccurred;

// Laravel >= 13.x
$event->exception;
```

Nếu bạn đang lắng nghe event này, thì bạn hãy cập nhật code listener của bạn sao cho tương xứng.

<a name="queuebusy-event-property-rename"></a>
#### `QueueBusy` Event Property Rename

**Likelihood Of Impact: Low**

Thuộc tính `$connection` của event `Illuminate\Queue\Events\QueueBusy` đã được đổi tên thành `$connectionName` để nhất quán với các event queue khác.

Nếu các listener của bạn tham chiếu đến `$connection`, hãy cập nhật chúng thành `$connectionName`.

<a name="queue-contract-method-additions"></a>
#### `Queue` Contract Method Additions

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Queue\Queue` bây giờ sẽ bao gồm các phương thức kiểm tra kích thước hàng đợi mà trước đây chỉ được khai báo trong comment.

Nếu bạn đang phát triển các implementation driver queue tuỳ biến cho contract này, thì bạn hãy thêm vào implementation các phương thức sau:

<div class="content-list" markdown="1">

- `pendingSize`
- `delayedSize`
- `reservedSize`
- `creationTimeOfOldestPendingJob`

</div>

<a name="routing"></a>
### Routing

<a name="domain-route-registration-precedence"></a>
#### Domain Route Registration Precedence

**Likelihood Of Impact: Low**

Các route có domain rõ ràng bây giờ sẽ được ưu tiên hơn các route không có domain trong quá trình tìm route.

Điều này cho phép các route subdomain hoạt động nhất quán ngay cả khi các route không có domain được đăng ký trước. Nếu ứng dụng của bạn phụ thuộc vào thứ tự đăng ký route, giữa các route có domain và không có domain, thì bạn hãy xem lại code của mình.

<a name="scheduling"></a>
### Scheduling

<a name="withscheduling-registration-timing"></a>
#### `withScheduling` Registration Timing

**Likelihood Of Impact: Very Low**

Các schedule được đăng ký thông qua `ApplicationBuilder::withScheduling()` bây giờ sẽ phải chờ cho đến khi `Schedule` được resolve.

Nếu ứng dụng của bạn phụ thuộc vào thời điểm đăng ký schedule trong quá trình bootstrap, bạn có thể cần phải điều chỉnh lại logic đó.

<a name="security"></a>
### Security

<a name="request-forgery-protection"></a>
#### Request Forgery Protection

**Likelihood Of Impact: High**

Middleware CSRF của Laravel đã được đổi tên từ `VerifyCsrfToken` thành `PreventRequestForgery`, và sẽ chứa thêm việc xác minh request-origin bằng cách sử dụng header `Sec-Fetch-Site`.

`VerifyCsrfToken` và `ValidateCsrfToken` tồn tại dưới dạng bí danh hiện không còn được sử dụng, nhưng các tham chiếu trực tiếp nên được cập nhật thành `PreventRequestForgery`, đặc biệt là khi bỏ qua middleware trong các bài test hoặc định nghĩa route:

```php
use Illuminate\Foundation\Http\Middleware\PreventRequestForgery;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;

// Laravel <= 12.x
->withoutMiddleware([VerifyCsrfToken::class]);

// Laravel >= 13.x
->withoutMiddleware([PreventRequestForgery::class]);
```

Middleware cấu hình API hiện cũng cung cấp phương thức `preventRequestForgery(...)`.

<a name="support"></a>
### Support

<a name="manager-extend-callback-binding"></a>
#### Manager `extend` Callback Binding

**Likelihood Of Impact: Low**

Các closure driver tùy biến được đăng ký thông qua các phương thức `extend` của manager bây giờ sẽ được liên kết với instance manager.

Nếu trước đây bạn có một đối tượng liên kết khác (chẳng hạn như một instance service provider) dưới dạng `$this` ở bên trong các callback, thì bây giờ bạn nên di chuyển các giá trị đó vào trong closure thông qua cách sử dụng `use (...)`.

<a name="str-factories-reset-between-tests"></a>
#### `Str` Factories Reset Between Tests

**Likelihood Of Impact: Low**

Laravel bây giờ sẽ reset các custom factory của `Str` trong quá trình dọn dẹp sau các bài test.

Nếu như các bài test của bạn phụ thuộc vào việc các factory UUID, ULID hoặc chuỗi ngẫu nhiên được duy trì giữa các bài test, bạn nên thiết lập chúng trong từng bài test hoặc setup hook.

<a name="jsfrom-uses-unescaped-unicode-by-default"></a>
#### `Js::from` Uses Unescaped Unicode By Default

**Likelihood Of Impact: Very Low**

Phương thức `Illuminate\Support\Js::from` bây giờ mặc định sử dụng `JSON_UNESCAPED_UNICODE`.

Nếu các bài test hoặc việc so sánh output frontend của bạn phụ thuộc vào các chuỗi Unicode được escape (ví dụ `\u00e8`), hãy cập nhật lại kết quả mà bạn mong đợi.

<a name="views"></a>
### Views

<a name="pagination-bootstrap-view-names"></a>
#### Pagination Bootstrap View Names

**Likelihood Of Impact: Low**

Các tên view phân trang mặc định cho Bootstrap 3 hiện đã rõ ràng:

```nothing
// Laravel <= 12.x
pagination::default
pagination::simple-default

// Laravel >= 13.x
pagination::bootstrap-3
pagination::simple-bootstrap-3
```

Nếu ứng dụng của bạn tham chiếu trực tiếp đến tên của các view phân trang này, bạn hãy cập nhật lại các tham chiếu đó.

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/12.x...13.x) và chọn bản cập nhật nào quan trọng với bạn.
