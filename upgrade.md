# Upgrade Guide

- [Nâng cấp đến 5.8.0 từ 5.7](#upgrade-5.8.0)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Cache TTL In Seconds](#cache-ttl-in-seconds)
- [Cache Lock Safety Improvements](#cache-lock-safety-improvements)
- [Environment Variable Parsing](#environment-variable-parsing)
- [Markdown File Directory Change](#markdown-file-directory-change)
- [Nexmo / Slack Notification Channels](#nexmo-slack-notification-channels)
- [New Default Password Length](#new-default-password-length)

</div>

<a name="medium-impact-changes"></a>
## Thay đổi có tác động trung bình

<div class="content-list" markdown="1">

- [Container Generators & Tagged Services](#container-generators)
- [SQLite Version Constraints](#sqlite)
- [Prefer String And Array Classes Over Helpers](#string-and-array-helpers)
- [Deferred Service Providers](#deferred-service-providers)
- [PSR-16 Conformity](#psr-16-conformity)
- [Model Names Ending With Irregular Plurals](#model-names-ending-with-irregular-plurals)
- [Custom Pivot Models With Incrementing IDs](#custom-pivot-models-with-incrementing-ids)
- [Pheanstalk 4.0](#pheanstalk-4)
- [Carbon 2.0](#carbon-2.0)

</div>

<a name="upgrade-5.8.0"></a>
## Nâng cấp đến 5.8.0 từ 5.7

#### Estimated Upgrade Time: 1 Hour

> {note} Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này có thể thực sự ảnh hưởng đến application của bạn.

<a name="updating-dependencies"></a>
### Updating Dependencies

Cập nhật library `laravel/framework` của bạn thành `5.8.*` trong file `composer.json` của bạn.

Tiếp theo, kiểm tra bất kỳ package bên thứ 3 nào mà được ứng dụng của bạn sử dụng và xác nhận rằng bạn đang sử dụng phiên bản phù hợp của package để hỗ trợ Laravel 5.8.

<a name="the-application-contract"></a>
### The `Application` Contract

#### The `environment` Method

**Likelihood Of Impact: Very Low**

Định dạng phương thức `environment` của contract `Illuminate\Contracts\Foundation\Application` [đã bị thay đổi](https://github.com/laravel/framework/pull/26296). Nếu bạn đang implement contract này trong ứng dụng của bạn, bạn nên cập nhật định dạng mới của phương thức đó:

    /**
     * Get or check the current application environment.
     *
     * @param  string|array  $environments
     * @return string|bool
     */
    public function environment(...$environments);

#### Added Methods

**Likelihood Of Impact: Very Low**

Các phương thức `bootstrapPath`, `configPath`, `databasePath`, `environmentPath`, `resourcePath`, `storagePath`, `resolveProvider`, `bootstrapWith`, `configurationIsCached`, `detectEnvironment`, `environmentFile`, `environmentFilePath`, `getCachedConfigPath`, `getCachedRoutesPath`, `getLocale`, `getNamespace`, `getProviders`, `hasBeenBootstrapped`, `loadDeferredProviders`, `loadEnvironmentFrom`, `routesAreCached`, `setLocale`, `shouldSkipMiddleware` và `terminate` [đã được thêm vào contract `Illuminate\Contracts\Foundation\Application`](https://github.com/laravel/framework/pull/26477).

Đây là trường hợp rất khó xảy ra, nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

<a name="authentication"></a>
### Authentication

#### Password Reset Notification Route Parameter

**Likelihood Of Impact: Low**

Khi người dùng yêu cầu một link để reset mật khẩu của họ, Laravel sẽ tạo một URL bằng cách sử dụng helper `route` để tạo URL cho route có tên là `password.reset`. Khi sử dụng Laravel 5.7, token được truyền tới helper `route` mà không có tên cụ thể, như sau:

    route('password.reset', $token);

Khi sử dụng Laravel 5.8, token được truyền tới helper `route` dưới dạng một tham số cụ thể:

    route('password.reset', ['token' => $token]);

Do đó, nếu bạn đang định nghĩa route `password.reset` của riêng bạn, bạn nên đảm bảo rằng nó có chứa tham số `{token}` trong URI của nó.

<a name="new-default-password-length"></a>
#### New Default Password Length

**Likelihood Of Impact: High**

Độ dài bắt buộc của mật khẩu khi chọn hoặc reset một mật khẩu đã được [thay đổi thành tám ký tự](https://github.com/laravel/framework/pull/25957). Bạn nên cập nhật bất kỳ quy tắc hoặc logic validation nào trong ứng dụng của bạn để phù hợp với độ dài mặc định tám ký tự này.

Nếu bạn cần giữ nguyên độ dài sáu ký tự trước đó hoặc một độ dài khác, bạn có thể extend class `Illuminate\Auth\Passwords\PasswordBroker` và ghi đè phương thức `validatePasswordWithDefaults` bằng logic của bạn.

<a name="cache"></a>
### Cache

<a name="cache-ttl-in-seconds"></a>
#### TTL in seconds

**Likelihood Of Impact: Very High**

Để cho phép thời gian hết hạn chi tiết hơn khi lưu trữ các item và tuân thủ theo tiêu chuẩn bộ nhớ cache PSR-16, thời gian tồn tại của item trong bộ nhớ cache đã thay đổi từ phút thành giây. Các phương thức `put`, `putMany`, `add`, `remember` và `setDefaultCacheTime` của class `Illuminate\Cache\Repository` và các class extend của nó, cũng như phương thức `put` của mỗi cache store đã được cập nhật với thay đổi này. Hãy xem [PR này](https://github.com/laravel/framework/pull/27276) để biết thêm thông tin.

Nếu bạn đang truyền một số nguyên cho bất kỳ phương thức nào trong số này, thì bạn nên cập nhật code của bạn để đảm bảo rằng bạn đang truyền số giây mà bạn muốn item vẫn còn tồn tại trong bộ nhớ cache. Ngoài ra, bạn có thể truyền một instance `DateTime` cho biết khi nào item sẽ hết hạn:

    // Laravel 5.7 - Store item for 30 minutes...
    Cache::put('foo', 'bar', 30);

    // Laravel 5.8 - Store item for 30 seconds...
    Cache::put('foo', 'bar', 30);

    // Laravel 5.7 / 5.8 - Store item for 30 seconds...
    Cache::put('foo', 'bar', now()->addSeconds(30));

> {tip} Thay đổi này làm cho hệ thống bộ cache của Laravel hoàn toàn tuân thủ theo [tiêu chuẩn thư viện bộ nhớ cache PSR-16](https://www.php-fig.org/psr/psr-16/).

<a name="psr-16-conformity"></a>
#### PSR-16 Conformity

**Likelihood Of Impact: Medium**

Ngoài [giá trị trả về được thay đổi ở đây](#the-repository-and-store-contracts), tham số TTL của phương thức `put`, `putMany` và `add` trong class `Illuminate\Cache\Repository` đã được cập nhật để phù hợp hơn với thông số kỹ thuật của PSR-16. Hành vi mới cung cấp một giá trị mặc định là `null`, do đó, khi bạn gọi phương thức mà không truyền vào TTL, thì sẽ dẫn đến việc lưu vĩnh viễn item đó vào trong bộ nhớ cache. Ngoài ra, nếu bạn lưu các item trong bộ nhớ cache mà có giá trị TTL từ 0 trở xuống, thì item đó sẽ xóa khỏi bộ nhớ đệm. Xem [PR này](https://github.com/laravel/framework/pull/27217) để biết thêm thông tin.

Event `KeyWritten` [cũng đã được cập nhật](https://github.com/laravel/framework/pull/27265) với những thay đổi này.

<a name="cache-lock-safety-improvements"></a>
#### Lock Safety Improvements

**Likelihood Of Impact: High**

Trong Laravel 5.7 và các phiên bản trước của Laravel, tính năng "atomic lock" được cung cấp bởi một số driver cache có thể có hành vi mà bạn không mong muốn đó là việc giải phóng sớm các khóa đang lock.

Ví dụ: **Client A** có được khóa `foo` với thời hạn 10 giây. **Client A** mất 20 giây để hoàn thành công việc của nó. Khóa được hệ thống bộ nhớ cache tự động giải phóng sau 10 giây kể từ thời gian xử lý của **Client A**. **Client B** có được khóa `foo`. **Client A** sau 10 giây cũng hoàn thành công việc của nó và giải phóng tiếp khóa `foo`, và vô tình giải phóng luôn khóa mà **Client B** đang giữ. Và do đó, một **Client C** khác có thể nhận được khóa.

Để giảm thiểu những trường hợp như thế này, các khóa hiện được tạo bằng một "scope token" cho phép framework đảm bảo rằng trong các trường hợp bình thường, chỉ có chủ sở hữu của khóa mới có thể giải phóng được khóa.

Nếu bạn đang sử dụng phương thức `Cache::lock()->get(Closure)` để tương tác với các khóa, thì không cần thay đổi gì:

    Cache::lock('foo', 10)->get(function () {
        // Lock will be released safely automatically...
    });

Tuy nhiên, nếu bạn đang gọi `Cache::lock()->release()` theo cách thủ công, thì bạn phải cập nhật code của bạn để duy trì một instance của khóa. Sau đó, sau khi thực hiện xong tác vụ của bạn, bạn có thể gọi phương thức `release` trên **cùng một instance của khóa**. Ví dụ:

    if (($lock = Cache::lock('foo', 10))->get()) {
        // Perform task...

        $lock->release();
    }

Thỉnh thoảng, bạn có thể muốn có được một khóa trong một process và giải phóng nó trong một process khác. Ví dụ: bạn có thể muốn có được khóa trong một web request và muốn mở khóa khi kết thúc một queued job được kích hoạt bởi chính request đó. Trong trường hợp này, bạn nên truyền vào một "owner token" trong scope của khóa cho queued job để job có thể khởi tạo lại khóa đó bằng cách sử dụng token được truyền vào:

    // Within Controller...
    $podcast = Podcast::find(1);

    if (($lock = Cache::lock('foo', 120))->get()) {
        ProcessPodcast::dispatch($podcast, $lock->owner());
    }

    // Within ProcessPodcast Job...
    Cache::restoreLock('foo', $this->owner)->release();

Nếu bạn muốn giải phóng khóa mà bỏ qua owner hiện tại của khoá, bạn có thể sử dụng phương thức `forceRelease`:

    Cache::lock('foo')->forceRelease();

<a name="the-repository-and-store-contracts"></a>
#### The `Repository` and `Store` Contracts

**Likelihood Of Impact: Very Low**

Để tuân thủ hoàn toàn tiêu chuẩn `PSR-16`, các giá trị được trả về của phương thức `put` và `forever` của contract `Illuminate\Contracts\Cache\Repository` và các giá trị được trả về của `put`, `putMany` và phương thức `forever` của contract `Illuminate\Contracts\Cache\Store` [đã được thay đổi](https://github.com/laravel/framework/pull/26726) từ `void` thành `bool`.

<a name="carbon-2.0"></a>
### Carbon 2.0

**Likelihood Of Impact: Medium**

Laravel hiện hỗ trợ cả Carbon 1 và Carbon 2; do đó, nếu Composer không phát hiện thấy các vấn đề về tương thích với bất kỳ package nào trong project của bạn, nó sẽ cố gắng thử nâng cấp lên Carbon 2.0. Vui lòng xem lại [hướng dẫn nâng cấp cho Carbon 2.0](https://carbon.nesbot.com/docs/#api-carbon-2).

<a name="collections"></a>
### Collections

#### The `add` Method

**Likelihood Of Impact: Very Low**

Phương thức `add` [đã được di chuyển](https://github.com/laravel/framework/pull/27082) từ class Eloquent collection sang class base collection. Nếu bạn đang mở rộng `Illuminate\Support\Collection` và class mở rộng của bạn có phương thức `add`, thì hãy đảm bảo là định dạng của phương thức này khớp với class cha của nó:

    public function add($item);

#### The `firstWhere` Method

**Likelihood Of Impact: Very Low**

Định dạng của phương thức `firstWhere` [đã được thay đổi](https://github.com/laravel/framework/pull/26261) để khớp với định dạng của phương thức `where`. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật lại định dạng phương thức để khớp với class cha của nó:

    /**
     * Get the first item by the given key value pair.
     *
     * @param  string  $key
     * @param  mixed  $operator
     * @param  mixed  $value
     * @return mixed
     */
    public function firstWhere($key, $operator = null, $value = null);

<a name="console"></a>
### Console

#### The `Kernel` Contract

**Likelihood Of Impact: Very Low**

Phương thức `terminate` [đã được thêm vào contract `Illuminate\Contracts\Console\Kernel`](https://github.com/laravel/framework/pull/26393). Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

<a name="container"></a>
### Container

<a name="container-generators"></a>
#### Generators & Tagged Services

**Likelihood Of Impact: Medium**

Phương thức `tagged` của container bây giờ sẽ sử dụng PHP generator để khởi tạo lazy-instantiate cho các service với một tag được cho. Điều này giúp cải thiện hiệu suất nếu bạn không sử dụng mọi service được gắn thẻ.

Bởi vì sự thay đổi này, nên phương thức `tagged` bây giờ sẽ trả về một `iterable` thay vì một `array`. Nếu bạn đang khai báo giá trị trả về của phương thức này, thì bạn nên đảm bảo là khai báo của bạn được thay đổi thành `iterable`.

Ngoài ra, không còn có thể truy cập trực tiếp vào một tagged service bằng giá trị array offset, chẳng hạn như `$container->tagged('foo')[0]`.

#### The `resolve` Method

**Likelihood Of Impact: Very Low**

Phương thức `resolve` [bây giờ sẽ chấp nhận](https://github.com/laravel/framework/pull/27066) một tham số boolean mới cho biết liệu các event(resolve callback) có nên được chạy / thực thi trong quá trình khởi tạo một đối tượng hay không. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật lại định dạng phương thức để khớp với class cha của nó:

#### The `addContextualBinding` Method

**Likelihood Of Impact: Very Low**

Phương thức `addContextualBinding` [đã được thêm vào contract `Illuminate\Contracts\Container\Container`](https://github.com/laravel/framework/pull/26551). Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

#### The `tagged` Method

**Likelihood Of Impact: Low**

Định dạng của phương thức `tagged` [đã được thay đổi](https://github.com/laravel/framework/pull/26953) và bây giờ, nó trả về một `iterable` thay vì một `array`. Nếu bạn đã khai báo trong code của bạn một tham số nào đó nhận vào giá trị trả về của phương thức này là `array`, thì bạn nên sửa khai báo đó thành `iterable`.

#### The `flush` Method

**Likelihood Of Impact: Very Low**

Phương thức `flush` [đã được thêm vào contract `Illuminate\Contracts\Container\Container`](https://github.com/laravel/framework/pull/26477). Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

<a name="database"></a>
### Database

#### Unquoted MySQL JSON Values

**Likelihood Of Impact: Low**

Query builder bây giờ sẽ trả về các giá trị JSON chưa được trích dẫn khi sử dụng MySQL và MariaDB. Hành vi này phù hợp với các cơ sở dữ liệu được hỗ trợ khác:

    $value = DB::table('users')->value('options->language');

    dump($value);

    // Laravel 5.7...
    '"en"'

    // Laravel 5.8...
    'en'

Do đó, toán tử `->>` không còn được hỗ trợ.

<a name="sqlite"></a>
#### SQLite

**Likelihood Of Impact: Medium**

Kể từ phiên bản Laravel 5.8, [phiên bản SQLite cũ nhất được hỗ trợ](https://github.com/laravel/framework/pull/25995) là SQLite 3.7.11. Nếu bạn đang sử dụng phiên bản SQLite cũ hơn, thì bạn nên cập nhật phiên bản mới (khuyến cáo nên sử dụng SQLite 3.8.8+).

#### Migrations & `bigIncrements`

**Likelihood Of Impact: None**

[Kể từ phiên bản Laravel 5.8](https://github.com/laravel/framework/pull/26472), mặc định, migration sẽ sử dụng phương thức `bigIncrements` trên các cột ID. Trước đây, các cột ID được tạo bằng phương thức `increments`.

Điều này sẽ không ảnh hưởng đến bất kỳ code nào đang hiện có trong project của bạn; tuy nhiên, hãy lưu ý rằng các cột khóa ngoại phải cùng loại với cột ID. Do đó, một cột được tạo bằng phương thức `increments` không thể tham chiếu đến một cột được tạo bằng phương thức `bigIncrements`.

<a name="eloquent"></a>
### Eloquent

<a name="model-names-ending-with-irregular-plurals"></a>
#### Model Names Ending With Irregular Plurals

**Likelihood Of Impact: Medium**

Kể từ phiên bản Laravel 5.8, các tên model có nhiều từ mà kết thúc của nó bằng một từ số nhiều bất quy tắc, thì [bây giờ sẽ được số nhiều hoá một cách chính xác](https://github.com/laravel/framework/pull/26421).

    // Laravel 5.7...
    App\Feedback.php -> feedback (correctly pluralized)
    App\UserFeedback.php -> user_feedbacks (incorrectly pluralized)

    // Laravel 5.8
    App\Feedback.php -> feedback (correctly pluralized)
    App\UserFeedback.php -> user_feedback (correctly pluralized)

Nếu bạn có một model đã được số nhiều hóa không chính xác, bạn có thể tiếp tục sử dụng tên bảng cũ bằng cách định nghĩa thuộc tính `$table` trên model của bạn:

    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user_feedbacks';

<a name="custom-pivot-models-with-incrementing-ids"></a>
#### Custom Pivot Models With Incrementing IDs

Nếu bạn đã định nghĩa một quan hệ nhiều-nhiều sử dụng model pivot tùy chỉnh và model pivot đó có khóa chính tự động tăng, bạn nên đảm bảo là class model pivot tùy chỉnh của bạn đã định nghĩa thuộc tính `incrementing` là `true `.

    /**
     * Indicates if the IDs are auto-incrementing.
     *
     * @var bool
     */
    public $incrementing = true;

#### The `loadCount` Method

**Likelihood Of Impact: Low**

Phương thức `loadCount` đã được thêm vào class base `Illuminate\Database\Eloquent\Model`. Nếu ứng dụng của bạn cũng định nghĩa một phương thức `loadCount`, nó có thể conflict với định nghĩa của Eloquent.

#### The `originalIsEquivalent` Method

**Likelihood Of Impact: Very Low**

Phương thức `originalIsEquivalent` của trait `Illuminate\Database\Eloquent\Concerns\HasAttributes` [đã được thay đổi](https://github.com/laravel/framework/pull/26391) từ `protected` thành `public`.

#### Automatic Soft-Deleted Casting Of `deleted_at` Property

**Likelihood Of Impact: Low**

Thuộc tính `delete_at` [bây giờ sẽ được tự động truyền](https://github.com/laravel/framework/pull/26985) sang một instance `Carbon` khi model Eloquent của bạn sử dụng trait `Illuminate\Database\Eloquent\SoftDeletes`. Bạn có thể ghi đè hành vi này bằng cách tạo một accessor tùy chỉnh của bạn cho thuộc tính đó hoặc là thêm thủ công vào thuộc tính `casts`:

    protected $casts = ['deleted_at' => 'string'];

#### BelongsTo `getForeignKey` & `getOwnerKey` Methods

**Likelihood Of Impact: Low**

Các phương thức `getForeignKey`, `getQualifiedForeignKey` và `getOwnerKey` của quan hệ` BelongsTo` đã được đổi tên thành `getForeignKeyName`, `getQualifiedForeignKeyName` và `getOwnerKeyName`, tên phương thức mới này làm cho tên phương thức nhất quán với các quan hệ khác do Laravel cung cấp .

<a name="environment-variable-parsing"></a>
### Environment Variable Parsing

**Likelihood Of Impact: High**

Package [phpdotenv](https://github.com/vlucas/phpdotenv) được sử dụng để phân tích cú pháp file `.env` đã phát hành một phiên bản mới, có thể ảnh hưởng đến kết quả trả về từ helper `env`. Cụ thể, ký tự `#` trong một giá trị sẽ không được trích dẫn, thì bây giờ sẽ được coi là một comment thay vì là một phần của giá trị:

Hành vi trước đây:

    ENV_VALUE=foo#bar

    env('ENV_VALUE'); // foo#bar

Hành vi mới:

    ENV_VALUE=foo#bar
    env('ENV_VALUE'); // foo

Để duy trì hành vi trước đó, bạn có thể set các giá trị môi trường trong dấu nháy:

    ENV_VALUE="foo#bar"

    env('ENV_VALUE'); // foo#bar

Để biết thêm thông tin, vui lòng tham khảo [hướng dẫn nâng cấp của phpdotenv](https://github.com/vlucas/phpdotenv/blob/master/UPGRADING.md).

<a name="events"></a>
### Events

#### The `fire` Method

**Likelihood Of Impact: Low**

Phương thức `fire` (không được dùng trong Laravel 5.4) của class `Illuminate\Events\Dispatcher` [đã bị xóa](https://github.com/laravel/framework/pull/26392).
Thay vào đó, bạn có thể sử dụng phương thức `dispatch`.

<a name="exception-handling"></a>
### Exception Handling

#### The `ExceptionHandler` Contract

**Likelihood Of Impact: Low**

Phương thức `shouldReport` [đã được thêm vào contract `Illuminate\Contracts\Debug\ExceptionHandler`](https://github.com/laravel/framework/pull/26193). Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

#### The `renderHttpException` Method

**Likelihood Of Impact: Low**

Định dạng của phương thức `renderHttpException` trong class `Illuminate\Foundation\Exceptions\Handler` [đã được thay đổi](https://github.com/laravel/framework/pull/25975). Nếu bạn đang ghi đè phương thức này trong exception handler của bạn, bạn nên cập nhật định dạng mới của phương thức để khớp với class cha của nó:

    /**
     * Render the given HttpException.
     *
     * @param  \Symfony\Component\HttpKernel\Exception\HttpExceptionInterface  $e
     * @return \Symfony\Component\HttpFoundation\Response
     */
    protected function renderHttpException(HttpExceptionInterface $e);

<a name="mail"></a>
### Mail

<a name="markdown-file-directory-change"></a>
<a name="markdown-file-directory-change"></a>
### Markdown File Directory Change

**Likelihood Of Impact: High**

Nếu bạn đã export các component mail Markdown của Laravel bằng lệnh `vendor:publish`, bạn nên đổi tên thư mục `/resources/views/vendor/mail/markdown` thành `/resources/views/vendor/mail/text`.

Ngoài ra, phương thức `markdownComponentPaths` [đã được đổi tên](https://github.com/laravel/framework/pull/26938) thành `textComponentPaths`. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật lại định dạng phương thức để khớp với class cha của nó:

#### Method Signature Changes In The `PendingMail` Class

**Likelihood Of Impact: Very Low**

Các phương thức `send`, `sendNow`, `queue`, `later` và `fill` của class `Illuminate\Mail\PendingMail` [đã được thay đổi](https://github.com/laravel/framework/pull/26790) để chấp nhận một instance `Illuminate\Contracts\Mail\Mailable` thay vì một `lluminate\Mail\Mailable`. Nếu bạn đang ghi đè các phương thức này, bạn nên cập nhật lại định dạng phương thức để khớp với class cha của nó:

<a name="queue"></a>
### Queue

<a name="pheanstalk-4"></a>
#### Pheanstalk 4.0

**Likelihood Of Impact: Medium**

Laravel 5.8 cung cấp hỗ trợ cho bản phát hành `~4.0` của thư viện Pheanstalk queue. Nếu bạn đang sử dụng thư viện Pheanstalk trong ứng dụng của bạn, vui lòng nâng cấp thư viện của bạn lên bản phát hành `~4.0` thông qua Composer.

#### The `Job` Contract

**Likelihood Of Impact: Very Low**

Phương thức `isReleased`, `hasFailed` và `markAsFailed` [đã được thêm vào contract `Illuminate\Contracts\Queue\Job`](https://github.com/laravel/framework/pull/26908). Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

#### The `Job::failed` & `FailingJob` Class

**Likelihood Of Impact: Very Low**

Khi một queued job bị thất bại trong Laravel 5.7, queue worker sẽ thực thi phương thức `FailingJob::handle`. Trong Laravel 5.8, logic có trong class `FailingJob` đã được chuyển sang phương thức` fail` trên chính class job đó. Do đó, một phương thức `fail` đã được thêm vào contract `Illuminate\Contracts\Queue\Job`.

Class `Illuminate\Queue\Jobs\Job` sẽ chứa việc implementation của phương thức `fail` và bạn sẽ không cần thay đổi code của bạn. Tuy nhiên, nếu bạn đang xây dựng một queue driver tùy chỉnh sử dụng class job **không** extend class job do Laravel cung cấp, thì bạn nên implement phương thức `fail` theo cách thủ công trong class job tùy chỉnh của bạn. Bạn có thể tham khảo class job của Laravel như là một implement mẫu.

Thay đổi này cho phép queue driver tùy chỉnh có nhiều quyền kiểm soát hơn đối với quá trình xóa job.

#### Redis Blocking Pop

**Likelihood Of Impact: Very Low**

Sử dụng tính năng "blocking pop" của Redis queue driver hiện đã an toàn. Trước đây, có một khả năng nhỏ là queued job có thể bị mất nếu server hoặc worker của Redis gặp sự cố vào cùng thời điểm với lúc job được lấy ra. Để làm cho chức năng blocking pop được an toàn, thì một danh sách Redis mới với hậu tố là `:notify` được tạo cho mỗi queue của Laravel.

<a name="requests"></a>
### Requests

#### The `TransformsRequest` Middleware

**Likelihood Of Impact: Low**

Phương thức `transform` của middleware `Illuminate\Foundation\Http\Middleware\TransformsRequest` bây giờ sẽ nhận vào được tên "đầy đủ" của một khóa input trong request khi input là một mảng:

    'employee' => [
        'name' => 'Taylor Otwell',
    ],

    /**
     * Transform the given value.
     *
     * @param  string  $key
     * @param  mixed  $value
     * @return mixed
     */
    protected function transform($key, $value)
    {
        dump($key); // 'employee.name' (Laravel 5.8)
        dump($key); // 'name' (Laravel 5.7)
    }

<a name="routing"></a>
### Routing

#### The `UrlGenerator` Contract

**Likelihood Of Impact: Very Low**

Phương thức `previous` [đã được thêm vào contract `Illuminate\Contracts\Routing\UrlGenerator`](https://github.com/laravel/framework/pull/25616). Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

#### The `cachedSchema` Property Of `Illuminate\Routing\UrlGenerator`

**Likelihood Of Impact: Very Low**

Tên thuộc tính `$cachedSchema` (không được dùng trong Laravel `5.7`) của class `Illuminate\Routing\UrlGenerator` [đã được đổi thành](https://github.com/laravel/framework/pull/26728) `$cachedScheme`.

<a name="sessions"></a>
### Sessions

#### The `StartSession` Middleware

**Likelihood Of Impact: Very Low**

Logic session persistence đã được [chuyển từ phương thức `terminate()` sang phương thức `handle()`](https://github.com/laravel/framework/pull/26410). Nếu bạn đang ghi đè một hoặc cả hai phương thức này, bạn nên cập nhật lại chúng để phản ánh những thay đổi này.

<a name="support"></a>
### Support

<a name="string-and-array-helpers"></a>
#### Prefer String And Array Classes Over Helpers

**Likelihood Of Impact: Medium**

Tất cả các global helper `array_*` và `str_*` [sẽ không được dùng nữa](https://github.com/laravel/framework/pull/26898). Bạn nên sử dụng trực tiếp các phương thức từ class `Illuminate\Support\Arr` và `Illuminate\Support\Str`.

Tác động của thay đổi này được đánh dấu là `medium` vì những helper này đã được chuyển sang package [laravel/helpers](https://github.com/laravel/helpers), cung cấp một lớp tương thích cho tất cả các phương thức global helper trong xử lý mảng và chuỗi.

Nếu bạn chọn cập nhật các view trong ứng dụng Laravel của bạn để sử dụng các phương thức dựa trên class, bạn nên xóa các view đã được biên dịch của bạn để có thể vẫn sử dụng được global helper:

    php artisan view:clear

<a name="deferred-service-providers"></a>
#### Deferred Service Providers

**Likelihood Of Impact: Medium**

Thuộc tính boolean `defer` trong service provider đã được sử dụng để cho biết liệu provider có bị hoãn lại hay không [đã không còn được sử dụng](https://github.com/laravel/framework/pull/27067). Để đánh dấu một service provider là trì hoãn, provider đó nên implement contract `Illuminate\Contracts\Support\DeferrableProvider`.

#### Read-Only `env` Helper

**Likelihood Of Impact: Low**

Trước đây, helper `env` có thể lấy ra được các giá trị của biến môi trường mà đã được thay đổi trong lúc runtime. Trong Laravel 5.8, helper `env` xử lý các biến môi trường sẽ là cố định. Nếu bạn muốn thay đổi một biến môi trường trong lúc runtime, hãy xem xét sử dụng giá trị config, nó có thể được lấy ra được bằng helper `config`:

Hành vi trước đây:

    dump(env('APP_ENV')); // local

    putenv('APP_ENV=staging');

    dump(env('APP_ENV')); // staging

Hành vi mới:

    dump(env('APP_ENV')); // local

    putenv('APP_ENV=staging');

    dump(env('APP_ENV')); // local

<a name="testing"></a>
### Testing

#### The `setUp` & `tearDown` Methods

Các phương thức `setUp` và `tearDown` bây giờ sẽ bắt buộc kiểu trả về void:

    protected function setUp(): void
    protected function tearDown(): void

#### PHPUnit 8

**Likelihood Of Impact: Optional**

Mặc định, Laravel 5.8 sử dụng PHPUnit 7. Tuy nhiên, bạn có thể tùy chọn nâng cấp lên phiên bản PHPUnit 8, nó yêu cầu PHP > = 7.2. Ngoài ra, vui lòng đọc qua toàn bộ danh sách các thay đổi có trong [thông báo phát hành PHPUnit 8](https://phpunit.de/announcements/phpunit-8.html).

<a name="validation"></a>
### Validation

#### The `Validator` Contract

**Likelihood Of Impact: Very Low**

Phương thức `validated` [đã được thêm vào contract `Illuminate\Contracts\Validation\Validator`](https://github.com/laravel/framework/pull/26419):

    /**
     * Get the attributes and values that were validated.
     *
     * @return array
     */
    public function validated();

Phương thức `addContextualBinding` [đã được thêm vào contract `Illuminate\Contracts\Container\Container`](https://github.com/laravel/framework/pull/26551). Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

#### The `ValidatesAttributes` Trait

**Likelihood Of Impact: Very Low**

Các phương thức `parseTable`, `getQueryColumn` và `requireParameterCount` của trait `Illuminate\Validation\Concerns\ValidatesAttributes` đã được thay đổi từ `protected` thành` public`.

#### The `DatabasePresenceVerifier` Class

**Likelihood Of Impact: Very Low**

Phương thức `table` trong class `Illuminate\Validation\DatabasePresenceVerifier` đã được thay đổi từ `protected` thành` public`.

#### The `Validator` Class

**Likelihood Of Impact: Very Low**

Phương thức `getPresenceVerifierFor` trong class `Illuminate\Validation\Validator` [đã được thay đổi](https://github.com/laravel/framework/pull/26717) từ `protected` thành` public`.

#### Email Validation

**Likelihood Of Impact: Very Low**

Quy tắc validation email bây giờ sẽ kiểm tra xem email có tuân thủ theo tiêu chuẩn [RFC6530](https://tools.ietf.org/html/rfc6530) hay không, điều này làm cho logic validation nhất quán với logic được SwiftMailer sử dụng. Trong Laravel `5.7`, quy tắc `email` chỉ kiểm tra rằng email tuân thủ tiêu chuẩn [RFC822](https://tools.ietf.org/html/rfc822).

Do đó, khi sử dụng Laravel 5.8, những email trước đây bị coi là không hợp lệ thì bây giờ sẽ được coi là hợp lệ (ví dụ như mail: `hej@bär.se`). Nói chung, đây nên được coi là một bản sửa lỗi; tuy nhiên, nó được liệt kê là một thay đổi nghiêm trọng cần thận trọng. [Vui lòng cho chúng tôi biết nếu bạn gặp phải bất kỳ vấn đề nào xung quanh thay đổi này](https://github.com/laravel/framework/pull/26503).

<a name="view"></a>
### View

#### The `getData` Method

**Likelihood Of Impact: Very Low**

Phương thức `getData` [đã được thêm vào contract `Illuminate\Contracts\View\View`](https://github.com/laravel/framework/pull/26754). Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

<a name="notifications"></a>
### Notifications

<a name="nexmo-slack-notification-channels"></a>
#### Nexmo / Slack Notification Channels

**Likelihood Of Impact: High**

Các channel Nexmo và Slack Notification đã được lấy ra thành các package độc lập. Để sử dụng các channel này trong ứng dụng của bạn, hãy thêm các package sau bằng Composer:

    composer require laravel/nexmo-notification-channel
    composer require laravel/slack-notification-channel

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [Công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/5.7...5.8) và chọn bản cập nhật nào quan trọng với bạn.
