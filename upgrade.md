# Upgrade Guide

- [Nâng cấp đến 5.7.0 từ 5.6](#upgrade-5.7.0)

<a name="upgrade-5.7.0"></a>
## Nâng cấp đến 5.7.0 từ 5.6

#### Estimated Upgrade Time: 10 - 15 Minutes

> {note} Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này có thể thực sự ảnh hưởng đến application của bạn.

### Updating Dependencies

Cập nhật library `laravel/framework` của bạn thành `5.7.*` trong file `composer.json` của bạn.

Nếu bạn đang sử dụng Laravel Passport, bạn nên cập nhật library `laravel/passport` của bạn thành `^7.0` trong file `composer.json` của bạn.

Tiếp theo, kiểm tra bất kỳ package bên thứ 3 nào mà được ứng dụng của bạn sử dụng và xác nhận rằng bạn đang sử dụng phiên bản phù hợp của package để hỗ trợ Laravel 5.7.

### Application

#### The `register` Method

**Likelihood Of Impact: Very Low**

Tham số `options` trong phương thức `register` của class `Illuminate\Foundation\Application` không được sử dụng nên đã bị xóa. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật định dạng mới cho phương thức của bạn:

    /**
     * Register a service provider with the application.
     *
     * @param  \Illuminate\Support\ServiceProvider|string  $provider
     * @param  bool   $force
     * @return \Illuminate\Support\ServiceProvider
     */
    public function register($provider, $force = false);

### Artisan

#### Scheduled Job Connection & Queues

**Likelihood Of Impact: Low**

Phương thức `$schedule->job` sẽ ưu tiên dùng các thuộc tính `queue` và `connection` trên job class nếu một connection hoặc một job không được truyền vào phương thức `job`.

Nói chung, đây nên được coi là một bản sửa lỗi; tuy nhiên, nó được liệt kê là một thay đổi nghiêm trọng cần thận trọng. [Vui lòng cho chúng tôi biết nếu bạn gặp phải bất kỳ vấn đề nào xung quanh thay đổi này](https://github.com/laravel/framework/pull/25216).

### Assets

#### Asset Directory Flattened

**Likelihood Of Impact: None**

Đối với các ứng dụng Laravel 5.7 mới, thư mục assets chứa các script và các style đã được chuyển thành thư mục `resources`. **Điều này sẽ không** ảnh hưởng đến các ứng dụng hiện có và không yêu cầu bạn phải thay đổi các ứng dụng hiện có của bạn.

Tuy nhiên, nếu bạn muốn thực hiện thay đổi này, bạn nên di chuyển tất cả các file từ thư mục `resources/assets/*` lên một cấp:

- Từ `resources/assets/js/*` thành `resources/js/*`
- Từ `resources/assets/sass/*` thành `resources/sass/*`

Sau đó, cập nhật các tham chiếu đến các thư mục đó trong file `webpack.mix.js` của bạn:

    mix.js('resources/js/app.js', 'public/js')
       .sass('resources/sass/app.scss', 'public/css');

#### `svg` Directory Added

**Likelihood Of Impact: Very High**

Một thư mục mới, `svg`, đã được thêm vào thư mục `public`. Nó chứa bốn file svg: `403.svg`, `404.svg`, `500.svg` và `503.svg`, được hiển thị trong các trang lỗi tương ứng với các lỗi của chúng.

Bạn có thể lấy các file [từ GitHub](https://github.com/laravel/laravel/tree/5.7/public/svg).

### Authentication

#### The `Authenticate` Middleware

**Likelihood Of Impact: Low**

Phương thức `authenticate` của middleware `Illuminate\Auth\Middleware\Authenticate` đã được cập nhật để chấp nhận một `$request` làm tham số đầu tiên của nó. Nếu bạn đang ghi đè phương thức này trong middleware `Authenticate` của bạn, bạn nên cập nhật định dạng mới của nó:

    /**
     * Determine if the user is logged in to any of the given guards.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  array  $guards
     * @return void
     *
     * @throws \Illuminate\Auth\AuthenticationException
     */
    protected function authenticate($request, array $guards)

#### The `ResetsPasswords` Trait

**Likelihood Of Impact: Low**

Phương thức protected `sendResetResponse` của trait `ResetsPasswords` bây giờ sẽ chấp nhận một tham số `Illuminate\Http\Request` làm tham số đầu tiên của nó. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật định dạng mới của nó:

    /**
     * Get the response for a successful password reset.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  string  $response
     * @return \Illuminate\Http\RedirectResponse|\Illuminate\Http\JsonResponse
     */
    protected function sendResetResponse(Request $request, $response)

#### The `SendsPasswordResetEmails` Trait

**Likelihood Of Impact: Low**

Phương thức protected `sendResetLinkResponse` của trait `SendsPasswordResetEmails` bây giờ sẽ chấp nhận một tham số `Illuminate\Http\Request` làm tham số đầu tiên của nó. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật định dạng mới của nó:

    /**
     * Get the response for a successful password reset link.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  string  $response
     * @return \Illuminate\Http\RedirectResponse|\Illuminate\Http\JsonResponse
     */
    protected function sendResetLinkResponse(Request $request, $response)

### Authorization

#### The `Gate` Contract

**Likelihood Of Impact: Very Low**

Phương thức `raw` đã thay đổi từ `protected` thành `public`. Ngoài ra, nó [đã được thêm vào contract `Illuminate\Contracts\Auth\Access\Gate`](https://github.com/laravel/framework/pull/25143):

    /**
     * Get the raw result from the authorization callback.
     *
     * @param  string  $ability
     * @param  array|mixed  $arguments
     * @return mixed
     */
    public function raw($ability, $arguments = []);

Nếu bạn đang implement interface này, bạn nên thêm phương thức này vào trong quá trình implement của bạn.

#### The `Login` Event

**Likelihood Of Impact: Very Low**

Phương thức `__construct` của event `Illuminate\Auth\Events\Login` có một tham số `$guard` mới:

    /**
     * Create a new event instance.
     *
     * @param  string  $guard
     * @param  \Illuminate\Contracts\Auth\Authenticatable  $user
     * @param  bool  $remember
     * @return void
     */
    public function __construct($guard, $user, $remember)

Nếu bạn đang tự gửi event này trong ứng dụng của bạn, bạn sẽ cần truyền tham số mới này vào hàm constructor của event. Ví dụ sau đây sẽ truyền guard mặc định của framework cho event Login:

    use Illuminate\Auth\Events\Login;

    event(new Login(config('auth.defaults.guard'), $user, $remember))

### Blade

#### The `or` Operator

**Likelihood Of Impact: High**

Toán tử Blade "or" đã bị loại bỏ và thay vào đó là toán tử `??` "null coalesce" trong PHP, chúng có cùng mục đích và chức năng:

    // Laravel 5.6...
    {{ $foo or 'default' }}

    // Laravel 5.7...
    {{ $foo ?? 'default' }}

### Cache

**Likelihood Of Impact: Very High**

Một thư mục `data` mới đã được thêm vào `storage/framework/cache`. Bạn nên tạo thư mục này trong ứng dụng của bạn:

    mkdir -p storage/framework/cache/data

Sau đó, thêm file [.gitignore](https://github.com/laravel/laravel/blob/76369205c8715a4a8d0d73061aa042a74fd402dc/storage/framework/cache/data/.gitignore) vào trong thư mục `data` mới được tạo:

    cp storage/framework/cache/.gitignore storage/framework/cache/data/.gitignore

Và cuối cùng, hãy đảm bảo là file [storage/framework/cache/.gitignore](https://github.com/laravel/laravel/blob/76369205c8715a4a8d0d73061aa042a74fd402dc/storage/framework/cache/.gitignore) đã được cập nhật như sau:

    *
    !data/
    !.gitignore

### Carbon

**Likelihood Of Impact: Very Low**

Các "macros" carbon bây giờ sẽ được thư viện Carbon xử lý trực tiếp thay vì thư viện mở rộng của Laravel. Chúng tôi không mong đợi điều này sẽ phá hỏng code của bạn; tuy nhiên, [vui lòng cho chúng tôi biết về bất kỳ vấn đề nào mà bạn gặp phải liên quan đến thay đổi này](https://github.com/laravel/framework/pull/23938).

### Collections

#### The `split` Method

**Likelihood Of Impact: Low**

Phương thức `split` [đã được cập nhật để luôn trả về số lượng "nhóm" mà được yêu cầu](https://github.com/laravel/framework/pull/24088), trừ khi tổng số item trong collection gốc quá ít so với số lượng được yêu cầu. Nói chung, đây nên được coi là một bản sửa lỗi; tuy nhiên, nó được liệt kê là một thay đổi nghiêm trọng cần phải thận trọng.

### Cookie

#### `Factory` Contract Method Signature

**Likelihood Of Impact: Very Low**

Định dạng của các phương thức `make` và `forever` của interface `Illuminate\Contracts\Cookie\Factory` [đã được thay đổi](https://github.com/laravel/framework/pull/23200). Nếu bạn đang implement interface này, bạn nên cập nhật các phương thức đó trong quá trình implement của bạn.

### Database

#### The `softDeletesTz` Migration Method

**Likelihood Of Impact: Low**

Phương thức `softDeletesTz` của schema table builder bây giờ sẽ chấp nhận tên cột làm tham số đầu tiên của nó, trong khi `$precision` sẽ được chuyển đến vị trí tham số thứ hai:

    /**
     * Add a "deleted at" timestampTz for the table.
     *
     * @param  string  $column
     * @param  int  $precision
     * @return \Illuminate\Support\Fluent
     */
    public function softDeletesTz($column = 'deleted_at', $precision = 0)

#### The `ConnectionInterface` Contract

**Likelihood Of Impact: Very Low**

Định dạng phương thức `select` và `selectOne` của contract `Illuminate\Database\ConnectionInterface` đã được cập nhật để phù hợp với tham số `$useReadPdo` mới:

    /**
     * Run a select statement and return a single result.
     *
     * @param  string  $query
     * @param  array   $bindings
     * @param  bool  $useReadPdo
     * @return mixed
     */
    public function selectOne($query, $bindings = [], $useReadPdo = true);

    /**
     * Run a select statement against the database.
     *
     * @param  string  $query
     * @param  array   $bindings
     * @param  bool  $useReadPdo
     * @return array
     */
    public function select($query, $bindings = [], $useReadPdo = true);

Ngoài ra, phương thức `cursor` cũng đã được thêm vào trong contract:

    /**
     * Run a select statement against the database and returns a generator.
     *
     * @param  string  $query
     * @param  array  $bindings
     * @param  bool  $useReadPdo
     * @return \Generator
     */
    public function cursor($query, $bindings = [], $useReadPdo = true);

Nếu bạn đang implement interface này, bạn nên thêm phương thức đó vào trong quá trình implement của bạn.

#### The `whereDate` Method

**Likelihood Of Impact: Low**

Phương thức `whereDate` của query builder bây giờ sẽ tự động chuyển các instance `DateTime` sang định dạng `Y-m-d`:

    // previous behaviour - SELECT * FROM `table` WHERE `created_at` > '2018-08-01 13:00:00'
    $query->whereDate('created_at', '>', Carbon::parse('2018-08-01 13:00:00'));

    // current behaviour - SELECT * FROM `table` WHERE `created_at` > '2018-08-01'
    $query->whereDate('created_at', '>', Carbon::parse('2018-08-01 13:00:00'));


#### Migration Command Output

**Likelihood Of Impact: Very Low**

Các lệnh migration core đã được [cập nhật để set output instance trên class migrator](https://github.com/laravel/framework/pull/24811). Nếu bạn đang ghi đè hoặc mở rộng các lệnh migration, bạn nên xóa các tham chiếu đến `$this->migrator->getNotes()` và sử dụng `$this->migrator->setOutput($this->output)` để thay thế.

#### SQL Server Driver Priority

**Likelihood Of Impact: Low**

Trước Laravel 5.7, mặc định, driver `PDO_DBLIB` sẽ được sử dụng làm driver SQL Server PDO. Driver này đã bị Microsoft loại bỏ. Và từ Laravel 5.7, `PDO_SQLSRV` sẽ được sử dụng làm driver mặc định nếu nó tồn tại. Ngoài ra, bạn có thể chọn sử dụng driver `PDO_ODBC`:

    'sqlsrv' => [
        // ...
        'odbc' => true,
        'odbc_datasource_name' => 'your-odbc-dsn',
    ],

Nếu cả hai driver trên đều không tồn tại, Laravel sẽ sử dụng driver `PDO_DBLIB`.

#### SQLite Foreign Keys

**Likelihood Of Impact: Medium**

SQLite không hỗ trợ xoá khóa ngoại. Vì lý do đó, việc sử dụng phương thức `dropForeign` trên table bây giờ sẽ đưa ra một ngoại lệ. Nói chung, đây nên được coi là một bản sửa lỗi; tuy nhiên, nó được liệt kê là một thay đổi nghiêm trọng cần phải thận trọng.

Nếu bạn chạy migration của bạn trên nhiều loại cơ sở dữ liệu, hãy xem xét sử dụng `DB::getDriverName()` trong migration của bạn để bỏ qua các phương thức khóa ngoại không được hỗ trợ cho SQLite.

### Debug

#### Dumper Classes

**Likelihood Of Impact: Very Low**

Các class `Illuminate\Support\Debug\Dumper` và `Illuminate\Support\Debug\HtmlDumper` đã bị xoá để sử dụng các dump variable của Symfony: `Symfony\Component\VarDumper\VarDumper` và `Symfony\Component\VarDumper\Dumper\HtmlDumper`.

### Eloquent

#### The `latest` / `oldest` Methods

**Likelihood Of Impact: Low**

Phương thức `latest` và `oldest` của Eloquent query builder đã được cập nhật để ưu tiên các cột timestamp "created at" tùy biến có thể được chỉ định trong các model Eloquent của bạn. Nói chung, đây nên được coi là một bản sửa lỗi; tuy nhiên, nó được liệt kê là một thay đổi nghiêm trọng cần phải thận trọng.

#### The `wasChanged` Method

**Likelihood Of Impact: Very Low**

Những thay đổi của model Eloquent bây giờ sẽ có sẵn trong phương thức `wasChanged` **trước khi** kích hoạt event model `updated`. Nói chung, đây nên được coi là một bản sửa lỗi; tuy nhiên, nó được liệt kê là một thay đổi nghiêm trọng cần phải thận trọng. [Vui lòng cho chúng tôi biết nếu bạn gặp bất kỳ vấn đề nào xung quanh thay đổi này](https://github.com/laravel/framework/pull/25026).

#### PostgreSQL Special Float Values

**Likelihood Of Impact: Low**

PostgreSQL hỗ trợ các giá trị float `Infinity`, `-Infinity` và `NaN`. Trước Laravel 5.7, chúng được cast thành kiểu `0` khi Eloquent cast cho các cột có loại là `float`, `double` hoặc `real`.

Kể từ Laravel 5.7, các giá trị này sẽ được chuyển sang thành các hằng số PHP tương ứng `INF`, `-INF` và `NAN`.

### Email Verification

**Likelihood Of Impact: Optional**

Nếu bạn chọn sử dụng [dịch vụ xác minh email](/docs/{{version}}/verification) mới của Laravel, bạn sẽ cần phải thêm scaffolding vào trong ứng dụng của bạn. Đầu tiên, hãy thêm `VerificationController` vào trong ứng dụng của bạn: [App\Http\Controllers\Auth\VerificationController](https://github.com/laravel/laravel/blob/5.7/app/Http/Controllers/Auth/VerificationController.php).

Bạn cũng sẽ cần sửa model `App\User` của bạn để implement contract `MustVerifyEmail`:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

Để sử dụng middleware `verified` dành cho những người dùng đã được xác minh mới có thể truy cập vào một route nhất định, bạn sẽ cần cập nhật thuộc tính `$routeMiddleware` trong file `app/Http/Kernel.php` của bạn để chứa thêm một middleware `verified` mới và một middleware `signed`:

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];

Bạn cũng sẽ cần một view để hiển thị form xác minh. View này nên được lưu tại `resources/views/auth/verify.blade.php`. Bạn có thể lấy nội dung của view này [trên GitHub](https://github.com/laravel/framework/blob/5.7/src/Illuminate/Auth/Console/stubs/make/views/auth/verify.stub).

Tiếp theo, bảng user của bạn phải chứa cột `email_verified_at` để lưu ngày giờ mà địa chỉ email được xác minh:

    $table->timestamp('email_verified_at')->nullable();

Để gửi email khi người dùng mới đăng ký, bạn nên đăng ký các event và listener sau trong class [App\Providers\EventServiceProvider](https://github.com/laravel/laravel/blob/5.7/app/Providers/EventServiceProvider.php) của bạn:

    use Illuminate\Auth\Events\Registered;
    use Illuminate\Auth\Listeners\SendEmailVerificationNotification;

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        Registered::class => [
            SendEmailVerificationNotification::class,
        ],
    ];

Cuối cùng, khi gọi phương thức `Auth::routes`, bạn cần truyền thêm tùy chọn `verify` cho phương thức đó:

    Auth::routes(['verify' => true]);

### Filesystem

#### `Filesystem` Contract Methods

**Likelihood Of Impact: Low**

Phương thức `readStream` và `writeStream` [đã được thêm vào contract `Illuminate\Contracts\Filesystem\Filesystem`](https://github.com/laravel/framework/pull/23755). Nếu bạn đang implement interface này, bạn nên thêm các phương thức đó vào quá trình implement của bạn.

### Hashing

#### `Hash::check` Method

**Likelihood Of Impact: None**

Phương thức `check` bây giờ sẽ có **tùy chọn** kiểm tra xem thuật toán hash có khớp với thuật toán đã được cấu hình hay không.

### Mail

#### Mailable Dynamic Variable Casing

**Likelihood Of Impact: Low**

Các biến được truyền vào trong các view của mail [bây giờ sẽ được xử lý, theo kiểu "camel case"](https://github.com/laravel/framework/pull/24232), điều này làm cho các biến trong mail sẽ phù hợp hơn với các biến trong view. Các biến trong mail không phải là một tính năng mà được Laravel ghi lại, nên do đó, khả năng nó ảnh hưởng đến ứng dụng của bạn là rất thấp.

#### Template Theme

**Likelihood Of Impact: Medium**

Nếu bạn đang tùy biến các theme mặc định được sử dụng trong các template mail Markdown, bạn có thể sẽ phải cần export và thực hiện lại các tùy biến của bạn. Các class button color đã được đổi tên từ 'blue', 'green' và 'red' thành "primary", "success" và 'error'.

### Queue

#### `QUEUE_DRIVER` Environment Variable

**Likelihood Of Impact: Very Low**

Biến môi trường `QUEUE_DRIVER` đã được đổi tên thành `QUEUE_CONNECTION`. Điều này sẽ không ảnh hưởng đến các ứng dụng của bạn trừ khi bạn sửa file cấu hình `config/queue.php` để giống với phiên bản Laravel 5.7.

#### `WorkCommand` Options

**Likelihood Of Impact: Very Low**

Tùy chọn `stop-when-empty` đã được thêm vào trong `WorkCommand`. Nếu bạn đang extend command này, bạn cần thêm `stop-when-empty` vào thuộc tính `$signature` trong class của bạn.

### Routing

#### The `Route::redirect` Method

**Likelihood Of Impact: High**

Phương thức `Route::redirect` bây giờ sẽ trả về một chuyển hướng với HTTP status code là `302`. Phương thức `permanentRedirect` cũng đã được thêm vào để cho phép chuyển hướng với HTTP status code là `301`.

    // Return a 302 redirect...
    Route::redirect('/foo', '/bar');

    // Return a 301 redirect...
    Route::redirect('/foo', '/bar', 301);

    // Return a 301 redirect...
    Route::permanentRedirect('/foo', '/bar');

#### The `addRoute` Method

**Likelihood Of Impact: Low**

Phương thức `addRoute` của class `Illuminate\Routing\Router` đã được đổi từ `protected` thành `public`.

### Validation

#### Nested Validation Data

**Likelihood Of Impact: Medium**

Trong các phiên bản trước của Laravel, phương thức `validate` sẽ không trả về đúng các dữ liệu cho các validation rule lồng nhau. Điều này đã được sửa trong Laravel 5.7:

    $data = Validator::make([
        'person' => [
            'name' => 'Taylor',
            'job' => 'Developer'
        ]
    ], ['person.name' => 'required'])->validate();

    dump($data);

    // Prior Behavior...
    ['person' => ['name' => 'Taylor', 'job' => 'Developer']]

    // New Behavior...
    ['person' => ['name' => 'Taylor']]

#### The `Validator` Contract

**Likelihood Of Impact: Very Low**

Phương thức `validate` [đã được thêm vào contract `Illuminate\Contracts\Validation\Validator`](https://github.com/laravel/framework/pull/25128):

    /**
     * Run the validator's rules against its data.
     *
     * @return array
     */
    public function validate();

Nếu bạn đang implement interface này, bạn nên thêm các phương thức đó vào quá trình implement của bạn.

### Testing

**Likelihood of Impact: Medium**

Laravel 5.7 giới thiệu các công cụ cải tiến kiểm tra cho các lệnh Artisan. Mặc định, output của lệnh Artisan bây giờ sẽ bị làm giả. Nếu bạn đang dựa vào phương thức `artisan` để chạy các lệnh như một phần của bài test của bạn, bạn nên sử dụng `Artisan::call` hoặc định nghĩa `public $mockConsoleOutput = false` làm một thuộc tính trong class test của bạn.

### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [Công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/5.6...5.7) và chọn bản cập nhật nào quan trọng với bạn.
