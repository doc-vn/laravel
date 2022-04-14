# Upgrade Guide

- [Nâng cấp đến 6.0 từ 5.8](#upgrade-6.0)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Authorized Resources & `viewAny`](#authorized-resources)
- [String & Array Helpers](#helpers)

</div>

<a name="medium-impact-changes"></a>
## Những thay đổi có tác động trung bình

<div class="content-list" markdown="1">

- [Carbon 1.x No Longer Supported](#carbon-support)
- [Redis Default Client](#redis-default-client)
- [Database `Capsule::table` Method](#capsule-table)
- [Eloquent Arrayable & `toArray`](#eloquent-to-array)
- [Eloquent `BelongsTo::update` Method](#belongs-to-update)
- [Eloquent Primary Key Types](#eloquent-primary-key-type)
- [Localization `Lang::trans` and `Lang::transChoice` Methods](#trans-and-trans-choice)
- [Localization `Lang::getFromJson` Method](#get-from-json)
- [Queue Retry Limit](#queue-retry-limit)
- [Resend Email Verification Route](#email-verification-route)
- [Email Verification Route Change](#email-verification-route-change)
- [The `Input` Facade](#the-input-facade)

</div>

<a name="upgrade-6.0"></a>
## Nâng cấp đến 6.0 từ 5.8

#### Estimated Upgrade Time: One Hour

> {note} Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này có thể thực sự ảnh hưởng đến application của bạn.

### PHP 7.2 Required

**Likelihood Of Impact: Medium**

PHP 7.1 sẽ không còn được phát triển kể từ tháng 12 năm 2019. Do đó, Laravel 6.0 sẽ yêu cầu PHP 7.2 trở lên.

<a name="updating-dependencies"></a>
### Updating Dependencies

Cập nhật phần library `laravel/framework` của bạn thành `^6.0` trong file `composer.json`. Nếu bạn cài đặt passport, hãy cập nhật library `laravel/passport` của bạn thành `^9.3.2` trong file `composer.json`.

Tiếp theo, kiểm tra bất kỳ package bên thứ 3 nào mà được ứng dụng của bạn sử dụng và xác nhận rằng bạn đang sử dụng phiên bản phù hợp của package để hỗ trợ Laravel 6.

### Authorization

<a name="authorized-resources"></a>
#### Authorized Resources & `viewAny`

**Likelihood Of Impact: High**

Các authorization policy được gắn với controller sử dụng phương thức `authorizeResource` bây giờ sẽ định nghĩa một phương thức `viewAny`, phương thức này sẽ được gọi khi người dùng truy cập vào phương thức `index` của controller. Nếu không định nghĩa, các lệnh gọi đến phương thức `index` của controller sẽ bị từ chối với lỗi là unauthorized.

#### Authorization Responses

**Likelihood Of Impact: Low**

Định dạng của phương thức khởi tạo của class `Illuminate\Auth\Access\Response` đã được thay đổi. Bạn nên cập nhật code của bạn cho phù hợp với định dạng mới. Nếu bạn không tự tạo ra các authorization response và chỉ sử dụng các phương thức `allow` và `deny` trong các policy của bạn, thì bạn không cần thay đổi gì thêm:

    /**
     * Create a new response.
     *
     * @param  bool  $allowed
     * @param  string  $message
     * @param  mixed  $code
     * @return void
     */
    public function __construct($allowed, $message = '', $code = null)

#### Returning "Deny" Responses

**Likelihood Of Impact: Low**

Trong các bản phát hành trước của Laravel, bạn không cần phải trả về giá trị của phương thức `deny` từ các phương thức policy của bạn vì một ngoại lệ sẽ được đưa ra ngay lập tức. Tuy nhiên, theo tài liệu Laravel, bây giờ bạn phải trả về giá trị của phương thức `deny` từ các policy của bạn:

    public function update(User $user, Post $post)
    {
        if (! $user->role->isEditor()) {
            return $this->deny("You must be an editor to edit this post.")
        }

        return $user->id === $post->user_id;
    }

<a name="auth-access-gate-contract"></a>
#### The `Illuminate\Contracts\Auth\Access\Gate` Contract

**Likelihood Of Impact: Low**

Contract `Illuminate\Contracts\Auth\Access\Gate` đã nhận được một phương thức `inspect` mới. Nếu bạn đang implement interface này, bạn nên thêm các phương thức này vào trong quá trình implement của bạn.

### Carbon

<a name="carbon-support"></a>
#### Carbon 1.x No Longer Supported

**Likelihood Of Impact: Medium**

Carbon 1.x [sẽ không còn được hỗ trợ](https://github.com/laravel/framework/pull/28683) vì nó sắp hết thời gian bảo trì. Nên vui lòng nâng cấp ứng dụng của bạn lên Carbon 2.0.

### Configuration

#### The `AWS_REGION` Environment Variable

**Likelihood Of Impact: Optional**

Nếu bạn có ý định sử dụng [Laravel Vapor](https://vapor.laravel.com), bạn nên cập nhật tất cả các chỗ dùng biến `AWS_REGION` trong thư mục `config` của bạn thành `AWS_DEFAULT_REGION`. Ngoài ra, bạn nên cập nhật tên biến môi trường này trong file `.env` của bạn.

<a name="redis-default-client"></a>
#### Redis Default Client

**Likelihood Of Impact: Medium**

Redis client mặc định đã thay đổi từ `predis` thành `phpredis`. Để tiếp tục sử dụng `predis`, hãy đảm bảo cấu hình `redis.client` của bạn được set thành `predis` trong file cấu hình `config/database.php` của bạn.

<a name="dynamodb-cache-store"></a>
#### DynamoDB Cache Store

**Likelihood Of Impact: Optional**

Nếu bạn có ý định sử dụng [Laravel Vapor](https://vapor.laravel.com), bạn nên cập nhật file `config/cache.php` của bạn chứa thêm `dynamodb` store.

    <?php
    return [
        ...
        'stores' => [
            ...
            'dynamodb' => [
                'driver' => 'dynamodb',
                'key' => env('AWS_ACCESS_KEY_ID'),
                'secret' => env('AWS_SECRET_ACCESS_KEY'),
                'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
                'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
                'endpoint' => env('DYNAMODB_ENDPOINT'),
            ],
        ],
        ...
    ];

<a name="sqs-environment-variables"></a>
#### SQS Environment Variables

**Likelihood Of Impact: Optional**

Nếu bạn có ý định sử dụng [Laravel Vapor](https://vapor.laravel.com), bạn nên cập nhật file `config/queue.php` của bạn để chứa thêm các biến môi trường cho kết nối `sqs`.

    <?php
    return [
        ...
        'connections' => [
            ...
            'sqs' => [
                'driver' => 'sqs',
                'key' => env('AWS_ACCESS_KEY_ID'),
                'secret' => env('AWS_SECRET_ACCESS_KEY'),
                'prefix' => env('SQS_PREFIX', 'https://sqs.us-east-1.amazonaws.com/your-account-id'),
                'queue' => env('SQS_QUEUE', 'your-queue-name'),
                'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
            ],
        ],
        ...
    ];

### Database

<a name="capsule-table"></a>
#### The Capsule `table` Method

**Likelihood Of Impact: Medium**

> {note} Thay đổi này chỉ áp dụng cho các ứng dụng chỉ đang sử dụng library `illuminate/database` không phải toàn bộ Laravel framework.

Định dạng của phương thức `table` trong class `Illuminate\Database\Capsule\Manager` đã được cập nhật để chấp nhận một bí danh bảng làm tham số thứ hai của nó. Nếu bạn đang sử dụng library `illuminate/database`, bạn nên cập nhật bất kỳ phương thức nào gọi tới phương thức này để phù hợp với định dạng mới:

    /**
     * Get a fluent query builder instance.
     *
     * @param  \Closure|\Illuminate\Database\Query\Builder|string  $table
     * @param  string|null  $as
     * @param  string|null  $connection
     * @return \Illuminate\Database\Query\Builder
     */
    public static function table($table, $as = null, $connection = null)

#### The `cursor` Method

**Likelihood Of Impact: Low**

Phương thức `cursor` bây giờ sẽ trả về một instance của `Illuminate\Support\LazyCollection` thay vì một `Generator`. `LazyCollection` có thể được lặp giống như một generator:

    $users = App\User::cursor();

    foreach ($users as $user) {
        //
    }

<a name="eloquent"></a>
### Eloquent

<a name="belongs-to-update"></a>
#### The `BelongsTo::update` Method

**Likelihood Of Impact: Medium**

Để nhất quán, phương thức `update` của quan hệ `BelongsTo` bây giờ sẽ hoạt động như là một truy vấn cập nhật đặc biệt, có nghĩa là nó không cung cấp khả năng mass assignment hoặc kích hoạt các event Eloquent. Điều này làm cho quan hệ sẽ nhất quán với các phương thức `update` có trên tất cả các loại quan hệ khác.

Nếu bạn muốn cập nhật một model trong một quan hệ `BelongsTo` với một model khác mà vẫn nhận được các event hoặc các bảo vệ mass assignment, thì bạn nên gọi phương thức `update` trên chính model đó:

    // Ad-hoc query... no mass assignment protection or events...
    $post->user()->update(['foo' => 'bar']);

    // Model update... provides mass assignment protection and events...
    $post->user->update(['foo' => 'bar']);

<a name="eloquent-to-array"></a>
#### Arrayable & `toArray`

**Likelihood Of Impact: Medium**

Phương thức `toArray` của model Eloquent bây giờ sẽ cast kiểu bất kỳ thuộc tính nào được implement `Illuminate\Contracts\Support\Arrayable` vào một mảng.

<a name="eloquent-primary-key-type"></a>
#### Declaration Of Primary Key Type

**Likelihood Of Impact: Medium**

Laravel 6.0 đã nhận được những [tối ưu hóa về hiệu suất](https://github.com/laravel/framework/pull/28153) cho các loại khóa integer. Nếu bạn đang sử dụng một string làm khóa chính của một model, bạn nên khai báo loại của khóa đó bằng cách sử dụng thuộc tính `$keyType` trên model của bạn:

    /**
     * The "type" of the primary key ID.
     *
     * @var string
     */
    protected $keyType = 'string';

### Email Verification

<a name="email-verification-route"></a>
#### Resend Verification Route HTTP Method

**Likelihood Of Impact: Medium**

Để ngăn chặn các cuộc tấn công CSRF có thể xảy ra, route `email/resend` mà được đăng ký khi sử dụng chức năng xác minh email mà được đi kèm trong Laravel đã được cập nhật từ route `GET` thành route `POST`. Do đó, bạn sẽ cần phải cập nhật giao diện người dùng của bạn để gửi đến server loại request thích hợp. Ví dụ: nếu bạn đang sử dụng template scaffolding mẫu của xác minh email của Laravel:

    {{ __('Before proceeding, please check your email for a verification link.') }}
    {{ __('If you did not receive the email') }},

    <form class="d-inline" method="POST" action="{{ route('verification.resend') }}">
        @csrf

        <button type="submit" class="btn btn-link p-0 m-0 align-baseline">
            {{ __('click here to request another') }}
        </button>.
    </form>

<a name="mustverifyemail-contract"></a>
#### The `MustVerifyEmail` Contract

**Likelihood Of Impact: Low**

Phương thức `getEmailForVerification` mới đã được thêm vào contract `Illuminate\Contracts\Auth\MustVerifyEmail`. Nếu bạn đang implement contract này theo cách thủ công, bạn nên implement thêm phương thức này. Phương thức này sẽ trả về địa chỉ email được liên kết với đối tượng. Nếu model `App\User` của bạn đang sử dụng trait `Illuminate\Auth\MustVerifyEmail`, thì bạn không cần thay đổi gì vì trait này đã implement sẵn phương thức này cho bạn.

<a name="email-verification-route-change"></a>
#### Email Verification Route Change

**Likelihood Of Impact: Medium**

Đường dẫn đến route để xác minh email đã được thay đổi từ `/email/verify/{id}` thành `/email/verify/{id}/{hash}`. Bất kỳ email xác minh nào được gửi trước khi nâng cấp lên bản Laravel 6.x sẽ không còn được phù hợp và sẽ hiển thị trang lỗi 404. Nếu bạn muốn, bạn có thể định nghĩa một route giống với đường dẫn URL xác minh email cũ và hiển thị một thông báo cung cấp thông tin cho người dùng của bạn biết và yêu cầu họ xác minh lại địa chỉ email của họ một lần nữa.

<a name="helpers"></a>
### Helpers

#### String & Array Helpers Package

**Likelihood Of Impact: High**

Tất cả các helper `str_` và `array_` đã được chuyển sang package Composer `laravel/helpers` mới và bị xóa ra khỏi framework. Nếu bạn muốn, bạn có thể cập nhật tất cả các phương thức gọi tới các helper này sang sử dụng các phương thức trong class `Illuminate\Support\Str` và class `Illuminate\Support\Arr`. Ngoài ra, bạn có thể thêm package `laravel/helpers` vào ứng dụng của bạn để sử dụng tiếp các helper này:

    composer require laravel/helpers

Nếu bạn chọn cập nhật các view trong ứng dụng của bạn sử dụng các phương thức dựa trên các class, thì bạn nên xóa tất cả các view đã được biên dịch vì chúng có thể vẫn đang sử dụng global helper:

    php artisan view:clear

### Localization

<a name="trans-and-trans-choice"></a>
#### The `Lang::trans` & `Lang::transChoice` Methods

**Likelihood Of Impact: Medium**

Phương thức `Lang::trans` và phương thức `Lang::transChoice` của translator đã được đổi tên thành  `Lang::get` và `Lang::choice`.

Ngoài ra, nếu bạn đang implement contract `Illuminate\Contracts\Translation\Translator` theo cách thủ công, thì bạn nên cập nhật phương thức `trans` và `transChoice` của phần implement thành `get` và `choice`.

<a name="get-from-json"></a>
#### The `Lang::getFromJson` Method

**Likelihood Of Impact: Medium**

Các phương thức `Lang::get` và phương thức `Lang::getFromJson` đã được hợp nhất. Các phương thức gọi đến phương thức `Lang::getFromJson` nên được cập nhật lại để gọi đến phương thức `Lang::get`.

> {note} Bạn nên chạy lệnh Artisan `php artisan view:clear` để tránh các lỗi Blade liên quan đến việc xoá `Lang::transChoice`, `Lang::trans`, và `Lang::getFromJson`.

### Mail

#### Mandrill & SparkPost Drivers Removed

**Likelihood Of Impact: Low**

Các mail driver `mandrill` và `sparkpost` đã bị xóa. Nếu bạn muốn tiếp tục sử dụng một trong hai driver này, chúng tôi khuyến khích bạn nên lựa chọn một package do cộng đồng phát triển để cung cấp các driver này.

### Notifications

#### Nexmo Routing Removed

**Likelihood Of Impact: Low**

Một phần còn tồn tại của channel thông báo Nexmo đã bị loại bỏ khỏi phần core của framework. Nếu bạn đang dựa vào việc route thông báo Nexmo, thì bạn nên tự implement phương thức `routeNotificationForNexmo` trên notifiable entity của bạn [như được mô tả trong tài liệu này](/docs/{{version}}/notifications#routing-sms-notifications).

### Password Reset

#### Password Validation

**Likelihood Of Impact: Low**

`PasswordBroker` không còn hạn chế hoặc validate mật khẩu. Validation mật khẩu đã được xử lý bởi class `ResetPasswordController`, làm cho việc validate của broker trở nên thừa thãi và không thể tùy chỉnh. Nếu bạn đang sử dụng `PasswordBroker` (hoặc facade `Password`) bên ngoài class `ResetPasswordController` được đi kèm với Laravel, thì bạn nên validate tất cả các mật khẩu trước khi truyền chúng cho broker.

### Queues

<a name="queue-retry-limit"></a>
#### Queue Retry Limit

**Likelihood Of Impact: Medium**

Trong các bản phát hành trước của Laravel, lệnh `php artisan queue:work` sẽ thử lại các job không giới hạn. Bắt đầu với Laravel 6.0, lệnh này bây giờ sẽ thử lại một job theo mặc định. Nếu bạn muốn các job phải được thử lại không giới hạn, bạn có thể truyền `0` đến tùy chọn `--tries`:

    php artisan queue:work --tries=0

Ngoài ra, hãy đảm bảo rằng cơ sở dữ liệu ứng dụng của bạn có chứa một bảng `failed_jobs`. Bạn có thể tạo một migration cho bảng này bằng cách sử dụng lệnh Artisan `queue:failed-table`:

    php artisan queue:failed-table

### Requests

<a name="the-input-facade"></a>
#### The `Input` Facade

**Likelihood Of Impact: Medium**

Facade `Input` đã bị xóa, facade này chủ yếu là bản sao của facade `Request`. Nếu bạn đang sử dụng phương thức `Input::get`, bây giờ bạn sẽ cần gọi đến phương thức `Request::input`. Tất cả các phương thức khác gọi đến facade `Input` có thể được cập nhật đơn giản thành sử dụng facade `Request`.

### Scheduling

#### The `between` Method

**Likelihood Of Impact: Low**

Trong các bản phát hành trước của Laravel, phương thức `between` của schedule biểu hiện hành vi khó hiểu qua các ranh giới giữa các ngày. Ví dụ:

    $schedule->command('list')->between('23:00', '4:00');

Đối với hầu hết người dùng, hành vi mong đợi của phương thức này là chạy lệnh `list` mỗi phút trong tất cả các phút mà trong khoảng thời gian từ 23:00 đến 4:00. Tuy nhiên, trong các bản phát hành trước của Laravel, schedule sẽ chạy lệnh `list` mỗi phút trong khoảng thời gian từ 4:00 đến 23:00, về cơ bản là chạy ngược thời gian. Trong Laravel 6.0, hành vi này đã được sửa.

### Storage

<a name="rackspace-storage-driver"></a>
#### Rackspace Storage Driver Removed

**Likelihood Of Impact: Low**

Driver lưu trữ `rackspace` đã bị xóa. Nếu bạn muốn tiếp tục sử dụng Rackspace với tư cách là một provider cung cấp dịch vụ lưu trữ, chúng tôi khuyến khích bạn nên lựa chọn một package do cộng đồng phát triển để cung cấp driver này.

### URL Generation

#### Route URL Generation & Extra Parameters

Trong các bản phát hành trước của Laravel, việc truyền các tham số mảng cho các phương thức helper `route` hoặc `URL::route` đôi khi sử dụng nhầm các tham số này làm giá trị URI khi tạo các URL cho các route, ngay cả khi giá trị của tham số này không có khóa phù hợp trong đường dẫn route. Bắt đầu từ Laravel 6.0, những giá trị này sẽ được gắn vào chuỗi truy vấn. Ví dụ, hãy xem xét route sau:

    Route::get('/profile/{location?}', function ($location = null) {
        //
    })->name('profile');

    // Laravel 5.8: http://example.com/profile/active
    echo route('profile', ['status' => 'active']);

    // Laravel 6.0: http://example.com/profile?status=active
    echo route('profile', ['status' => 'active']);

Helper `action` và phương thức `URL::action` cũng bị ảnh hưởng bởi sự thay đổi này:

    Route::get('/profile/{id?}', 'ProfileController@show');

    // Laravel 5.8: http://example.com/profile/1
    echo action('ProfileController@show', ['profile' => 1]);

    // Laravel 6.0: http://example.com/profile?profile=1
    echo action('ProfileController@show', ['profile' => 1]);

### Validation

#### FormRequest `validationData` Method

**Likelihood Of Impact: Low**

Phương thức `validationData` của request form đã được thay đổi từ `protected` thành `public`. Nếu bạn đang ghi đè phương thức này trong quá trình implement của bạn, bạn nên cập nhật thành `public`.

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [Công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/5.8...6.x) và chọn bản cập nhật nào quan trọng với bạn.
