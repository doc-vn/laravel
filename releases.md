# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 5.8](#laravel-5.8)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Cấu trúc của phiên bản của Laravel được duy trì theo quy ước như sau: `paradigm.major.minor`. Các phiên bản được phát hành chính thức của framework được phát hành sáu tháng một lần (tháng 2 và tháng 8), trong khi các bản phát hành nhỏ hơn có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `5.8.*`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

Các bản phát hành thay đổi Paradigm được phân tách qua nhiều năm và đại diện cho những thay đổi căn bản trong kiến trúc và quy ước của framework. Hiện tại, chưa có bản phát hành nào mà cần phải thay đổi Paradigm được phát triển trong hiện tại.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với các bản phát hành hỗ trợ dài hạn, chẳng hạn như Laravel 5.5, các bản sửa lỗi được cung cấp trong 2 năm và các bản sửa lỗi bảo mật được cung cấp trong 3 năm. Những bản phát hành này cung cấp các hỗ trợ và bảo trì dài nhất. Đối với các bản phát hành bình thường, các bản sửa lỗi được cung cấp trong 6 tháng và các bản sửa lỗi bảo mật được cung cấp trong 1 năm. Đối với tất cả các thư viện, bao gồm cả Lumen, chỉ bản phát hành mới nhất mới nhận được các bản sửa lỗi.

| Version | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- |
| 5.0 | ngày 4 tháng 2 năm 2015 | ngày 4 tháng 8 năm 2015 | ngày 4 tháng 2 năm 2016 |
| 5.1 (LTS) | ngày 9 tháng 6 năm 2015 | ngày 9 tháng 6 năm 2017 | ngày 9 tháng 6 năm 2018 |
| 5.2 | ngày 21 tháng 12 năm 2015 | ngày 6 tháng 12 năm 2016 | ngày 21 tháng 12 năm 2016 |
| 5.3 | ngày 23 tháng 8 năm 2016 | ngày 23 tháng 2 năm 2016 | ngày 23 tháng 8 năm 2017 |
| 5.4 | ngày 24 tháng 1 năm 2017 | ngày 24 tháng 7 năm 2017| ngày 24 tháng 1 năm 2018 |
| 5.5 (LTS) | ngày 30 tháng 8 năm 2017 | ngày 30 tháng 8 năm 2019 | ngày 30 tháng 8 năm 2020 |
| 5.6 | ngày 7 tháng 2 năm 2018 | ngày 7 tháng 8 năm 2018 | ngày 7 tháng 2 năm 2019 |
| 5.7 | ngày 4 tháng 9 năm 2018 | ngày 4 tháng 3 năm 2019 | ngày 4 tháng 9 năm 2019 |
| 5.8 | ngày 26 tháng 2 năm 2019 | ngày 26 tháng 8 năm 2019 | ngày 26 tháng 2 năm 2020 |

<a name="laravel-5.8"></a>
## Laravel 5.8

Laravel 5.8 tiếp tục những cải tiến được thực hiện trong Laravel 5.7 bằng cách giới thiệu quan hệ thông qua liên kết một, cải tiến validation email, đăng ký tự động dựa trên quy ước của các authorization policy, DynamoDB cache và session driver cải tiến cấu hình timezone cho schedule, hỗ trợ chỉ định nhiều authentication guard để broadcast channel, tuân thủ cache driver PSR-16, cải tiến lệnh `artisan serve`, hỗ trợ PHPUnit 8.0, hỗ trợ Carbon 2.0, hỗ trợ Pheanstalk 4.0 và một hàng loạt các bản sửa lỗi và cải tiến sử dụng khác.

### Eloquent `HasOneThrough` Relationship

Eloquent bây giờ sẽ cung cấp hỗ trợ cho kiểu quan hệ `hasOneThrough`. Ví dụ, hãy tưởng tượng một model nhà cung cấp `hasOne` của model tài khoản và model tài khoản có một model AccountHistory. Bạn có thể sử dụng mối quan hệ `hasOneThrough` để truy cập vào lịch sử tài khoản của nhà cung cấp thông qua model tài khoản:

    /**
     * Get the account history for the supplier.
     */
    public function accountHistory()
    {
        return $this->hasOneThrough(AccountHistory::class, Account::class);
    }

### Auto-Discovery Of Model Policies

Khi sử dụng Laravel 5.7, mỗi [authorization policy](/docs/{{version}}/authorization#creating-policies) sẽ tương ứng với mỗi model cần được đăng ký trong `AuthServiceProvider` trong ứng dụng của bạn:

    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\User' => 'App\Policies\UserPolicy',
    ];

Laravel 5.8 giới thiệu tính năng auto-discovery các policy model miễn là model và policy tuân theo các quy ước đặt tên Laravel. Cụ thể, các policy phải nằm trong thư mục `Policies` bên dưới thư mục chứa model. Ví dụ: các model có thể được đặt trong thư mục `app` trong khi các policy có thể được đặt trong thư mục `app/Policies`. Ngoài ra, tên policy cũng phải khớp với tên của model và có hậu tố là `Policy`. Vì vậy, một model `User` sẽ tương ứng với một class `UserPolicy`.

Nếu bạn muốn cung cấp logic policy discovery của riêng bạn, bạn có thể đăng ký một lệnh callback tùy biến bằng cách sử dụng phương thức `Gate::guessPolicyNamesUsing`. Thông thường, phương thức này sẽ được gọi từ `AuthServiceProvider` trong ứng dụng của bạn:

    use Illuminate\Support\Facades\Gate;

    Gate::guessPolicyNamesUsing(function ($modelClass) {
        // return policy class name...
    });

> {note} Bất kỳ policy nào được ánh xạ trong `AuthServiceProvider` cũng sẽ được ưu tiên hơn các policy khác được đăng ký tự động.

### PSR-16 Cache Compliance

Để cho phép thời gian hết hạn chi tiết hơn khi lưu trữ các item và tuân thủ theo tiêu chuẩn bộ nhớ cache PSR-16, thời gian tồn tại của item trong bộ nhớ cache đã thay đổi từ phút thành giây. Các phương thức `put`, `putMany`, `add`, `remember` và `setDefaultCacheTime` của class `Illuminate\Cache\Repository` và các class extend của nó, cũng như phương thức `put` của mỗi cache store đã được cập nhật với thay đổi này. Hãy xem [PR này](https://github.com/laravel/framework/pull/27276) để biết thêm thông tin.

Nếu bạn đang truyền một số nguyên cho bất kỳ phương thức nào trong số này, thì bạn nên cập nhật code của bạn để đảm bảo rằng bạn đang truyền số giây mà bạn muốn item vẫn còn tồn tại trong bộ nhớ cache. Ngoài ra, bạn có thể truyền một instance `DateTime` cho biết khi nào item sẽ hết hạn:

    // Laravel 5.7 - Store item for 30 minutes...
    Cache::put('foo', 'bar', 30);

    // Laravel 5.8 - Store item for 30 seconds...
    Cache::put('foo', 'bar', 30);

    // Laravel 5.7 / 5.8 - Store item for 30 seconds...
    Cache::put('foo', 'bar', now()->addSeconds(30));

### Multiple Broadcast Authentication Guards

Trong các bản phát hành trước của Laravel, các channel broadcast private và presence sẽ xác thực người dùng hiện tại thông qua authentication guard mặc định của ứng dụng. Bắt đầu từ Laravel 5.8, bạn có thể chỉ định nhiều guard tùy chỉnh khác sẽ xác thực request đến nếu cần:

    Broadcast::channel('channel', function() {
        // ...
    }, ['guards' => ['web', 'admin']])

### Token Guard Token Hashing

`token` guard của Laravel sẽ cung cấp một cách xác thực API cơ bản, hiện hỗ trợ lưu trữ token API dưới dạng hàm hash SHA-256. Điều này cung cấp khả năng bảo mật được cải thiện hơn so với việc lưu trữ token văn bản thuần. Để tìm hiểu thêm về token được hash, vui lòng xem toàn bộ [tài liệu xác thực API](/docs/{{version}}/api-authentication).

> **Note:** Mặc dù Laravel cung cấp một guard xác thực dựa trên token đơn giản, nhưng chúng tôi thực sự khuyên bạn rằng nên cân nhắc sử dụng [Laravel Passport](/docs/{{version}}/passport) cho các ứng dụng production, mạnh mẽ cung cấp xác thực API .

### Improved Email Validation

Laravel 5.8 giới thiệu các cải tiến đối với logic validation email cơ bản của validator bằng cách sử dụng package `egulias/email-validator` được sử dụng bởi SwiftMailer. Logic validation email trước đây của Laravel đôi khi coi các email hợp lệ, chẳng hạn như `example@bär.se`, là không hợp lệ.

### Default Scheduler Timezone

Laravel cho phép bạn tùy chỉnh timezone của một scheduled task bằng phương thức `timezone`:

    $schedule->command('inspire')
             ->hourly()
             ->timezone('America/Chicago');

Tuy nhiên, điều này có thể trở nên rườm rà và lặp đi lặp lại nếu bạn chỉ định cùng một timezone cho tất cả các scheduled task của bạn. Vì lý do đó, bây giờ bạn có thể định nghĩa một phương thức `scheduleTimezone` trong file `app/Console/Kernel.php` của bạn. Phương thức này sẽ trả về một timezone mặc định sẽ được chỉ định cho tất cả các scheduled task:

    /**
     * Get the timezone that should be used by default for scheduled events.
     *
     * @return \DateTimeZone|string|null
     */
    protected function scheduleTimezone()
    {
        return 'America/Chicago';
    }

### Intermediate Table / Pivot Model Events

Trong các phiên bản trước của Laravel, [event của model eloquent](/docs/{{version}}/eloquent#events) sẽ không được gửi đi khi attaching, detaching, hoặc đồng bộ bảng trung gian (model "pivot") của quan hệ nhiều-nhiều. Khi sử dụng [model bảng trung gian](/docs/{{version}}/eloquent-relationships#defining-custom-intermediate-table-models) trong Laravel 5.8, các event model sẽ được gửi đi.

### Artisan Call Improvements

Laravel cho phép bạn gọi Artisan thông qua phương thức `Artisan::call`. Trong các bản phát hành trước của Laravel, các tùy chọn của lệnh này được truyền qua một mảng làm tham số thứ hai cho phương thức:

    use Illuminate\Support\Facades\Artisan;

    Artisan::call('migrate:install', ['database' => 'foo']);

Tuy nhiên, Laravel 5.8 cho phép bạn truyền toàn bộ lệnh, bao gồm cả các tùy chọn, làm tham số string đầu tiên của phương thức:

    Artisan::call('migrate:install --database=foo');

### Mock / Spy Testing Helper Methods

Để làm giả các đối tượng trở nên thuận tiện hơn, các phương thức `mock` và `spy` mới đã được thêm vào class base test case của Laravel. Các phương thức này tự động liên kết class được làm giả vào container. Ví dụ:

    // Laravel 5.7
    $this->instance(Service::class, Mockery::mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    }));

    // Laravel 5.8
    $this->mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    });

### Eloquent Resource Key Preservation

Khi trả về một [eloquent resource collection](/docs/{{version}}/eloquent-resources) từ một route, Laravel sẽ set lại các khóa của collection để chúng có số thứ tự bắt đầu từ 0:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Khi sử dụng Laravel 5.8, bây giờ bạn có thể thêm thuộc tính `preserveKeys` vào class resource của bạn để cho biết liệu khóa của collection có được giữ nguyên hay không. Mặc định và để duy trì tính nhất quán với các bản phát hành Laravel trước đó, các khóa sẽ được set lại theo mặc định:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Indicates if the resource's collection keys should be preserved.
         *
         * @var bool
         */
        public $preserveKeys = true;
    }

Khi thuộc tính `preserveKeys` được set thành `true`, các khóa của collection sẽ được giữ nguyên:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all()->keyBy->id);
    });

### Higher Order `orWhere` Eloquent Method

Trong các bản phát hành trước của Laravel, việc kết hợp nhiều scope cho model Eloquent thông qua truy vấn `or` có thể yêu cầu sử dụng lệnh callback Closure::

    // scopePopular and scopeActive methods defined on the User model...
    $users = App\User::popular()->orWhere(function (Builder $query) {
        $query->active();
    })->get();

Laravel 5.8 giới thiệu một phương thức "higher order" là `orWhere` cho phép bạn kết hợp các scope này với nhau một cách thuận tiện mà không cần sử dụng Closures:

    $users = App\User::popular()->orWhere->active()->get();

### Artisan Serve Improvements

Trong các bản phát hành trước của Laravel, lệnh `serve` của Artisan sẽ chạy ứng dụng của bạn trên cổng `8000`. Nếu có một process lệnh `serve` khác đã được chạy trên cổng này, thì lệnh chạy ứng dụng thứ hai qua `serve` sẽ không thành công. Bắt đầu từ Laravel 5.8, `serve` sẽ quét và tìm ra các cổng khả dụng cho đến cổng `8009`, cho phép bạn chạy nhiều ứng dụng cùng một lúc.

### Blade File Mapping

Khi biên dịch các template Blade, Laravel bây giờ sẽ thêm một comment vào đầu file đã được biên dịch, comment đó sẽ chứa đường dẫn đến template Blade ban đầu.

### DynamoDB Cache / Session Drivers

Laravel 5.8 giới thiệu cache [DynamoDB](https://aws.amazon.com/dynamodb/) và session driver [DynamoDB](https://aws.amazon.com/dynamodb/). DynamoDB là một cơ sở dữ liệu NoSQL không có máy chủ do Amazon Web Services cung cấp. Bạn có thể tìm thấy cấu hình mặc định cho cache driver `dynamodb` trong [file cấu hình cache](https://github.com/laravel/laravel/blob/master/config/cache.php) của Laravel 5.8.

### Carbon 2.0 Support

Laravel 5.8 cung cấp hỗ trợ cho bản phát hành `~2.0` của thư viện Carbon.

### Pheanstalk 4.0 Support

Laravel 5.8 cung cấp hỗ trợ cho bản phát hành `~4.0` của thư viện Pheanstalk queue. Nếu bạn đang sử dụng thư viện Pheanstalk trong ứng dụng của bạn, vui lòng nâng cấp thư viện của bạn lên bản phát hành `~ 4.0` thông qua Composer.
