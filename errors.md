# Errors & Logging

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Chi tiết về error](#error-detail)
    - [Lưu trữ log](#log-storage)
    - [Cấp độ log](#log-severity-levels)
    - [Tuỳ biến cấu hình Monolog](#custom-monolog-configuration)
- [Xử lý exception](#the-exception-handler)
    - [Phương thức report](#report-method)
    - [Phương thức render](#render-method)
    - [Reportable và Renderable Exceptions](#renderable-exceptions)
- [HTTP Exceptions](#http-exceptions)
    - [Tuỳ biến page HTTP Error](#custom-http-error-pages)
- [Logging](#logging)

<a name="introduction"></a>
## Giới thiệu

Khi bạn bắt đầu một dự án Laravel mới, các xử lý lỗi và các ngoại lệ đã được cấu hình sẵn cho bạn. Class `App\Exceptions\Handler` là nơi tất cả các ngoại lệ sẽ được gọi tới bởi application của bạn, tại đó các ngoại lệ sẽ được log và sau đó được hiển thị trở lại cho người dùng. Chúng ta sẽ đi sâu hơn vào class này trong suốt tài liệu này.

Để log, Laravel sử dụng thư viện [Monolog](https://github.com/Seldaek/monolog), cung cấp và hỗ trợ nhiều xử lý log mạnh mẽ. Laravel đã cấu hình sẵn một số xử lý này cho bạn, cho phép bạn chọn giữa log ra một file duy nhất, hoặc chuyển file log thành theo ngày hoặc ghi thông tin lỗi vào log system.

<a name="configuration"></a>
## Cấu hình

<a name="error-detail"></a>
### Chi tiết về error

Tùy chọn `debug` trong file cấu hình `config/app.php` của bạn sẽ định nghĩa lượng thông tin về lỗi thực sự được hiển thị cho người dùng. Mặc định, tùy chọn này được khai báo từ giá trị của biến môi trường `APP_DEBUG`, được lưu trữ trong file `.env` của bạn.

Cho môi trường phát triển ở local, bạn nên lưu biến môi trường `APP_DEBUG` thành `true`. Trong môi trường chạy product của bạn, giá trị này phải luôn là `false`. Nếu giá trị được lưu thành `true` trong môi trường product, bạn có thể có nguy cơ lộ các giá trị cấu hình nhạy cảm cho người dùng application.

<a name="log-storage"></a>
### Lưu trữ log

Mặc định, Laravel hỗ trợ ghi log vào `single` file, `daily` file, `syslog` và `errorlog`. Để cấu hình cơ chế lưu trữ nào sẽ Laravel sử dụng, bạn nên sửa đổi tùy chọn `log` trong file cấu hình `config/app.php` của bạn. Ví dụ: nếu bạn muốn sử dụng các file log daily thay vì một file single, bạn nên lưu giá trị `log` trong file cấu hình `app` của bạn thành `daily`:

    'log' => 'daily'

#### Maximum của file daily log

Khi sử dụng chế độ log `daily`, Laravel mặc định sẽ chỉ lưu lại file log trong năm ngày. Nếu bạn muốn điều chỉnh số lượng file được lưu lại, bạn có thể thêm giá trị cấu hình `log_max_files` vào file cấu hình `app` của bạn:

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### Cấp độ log

Khi sử dụng Monolog, thông điệp log có thể có các mức độ nghiêm trọng khác nhau. Mặc định, Laravel ghi tất cả các cấp độ log vào bộ lưu trữ. Tuy nhiên, trong môi trường product của bạn, bạn có thể muốn cấu hình mức độ nghiêm trọng tối thiểu cần được ghi lại bằng cách thêm tùy chọn `log_level` vào file cấu hình `app.php` của bạn.

Khi tùy chọn này đã được cấu hình, Laravel sẽ chỉ ghi lại tất cả các cấp độ lớn hơn hoặc bằng với mức độ nghiêm trọng đã được định nghĩa. Ví dụ: nếu `log_level` của bạn là `error` thì các log sẽ log ra các thông báo của **error**, **critical**, **alert**, và **emergency**:

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} Monolog có thể nhận ra các mức độ nghiêm trọng như sau - từ mức độ nghiêm trọng thấp nhất đến mức độ nghiêm trọng cao nhất: `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.

<a name="custom-monolog-configuration"></a>
### Tuỳ biến cấu hình Monolog

Nếu bạn muốn có toàn quyền kiểm soát cách Monolog được cấu hình trong application của bạn, bạn có thể sử dụng phương thức `configureMonologUsing` của application. Bạn nên gọi đến phương thức này trong file `bootstrap/app.php` ngay trước khi biến `$app` được trả về bởi file đó:

    $app->configureMonologUsing(function ($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

#### Tuỳ biến tên channel

Theo mặc định, Monolog được khởi tạo với tên trùng với tên môi trường hiện tại, chẳng hạn như `production` hoặc `local`. Để thay đổi giá trị này, hãy thêm tùy chọn `log_channel` vào file cấu hình `app.php` của bạn:

    'log_channel' => env('APP_LOG_CHANNEL', 'my-app-name'),

<a name="the-exception-handler"></a>
## Xử lý exception

<a name="report-method"></a>
### Phương thức report

Tất cả các ngoại lệ sẽ được xử lý bởi class `App\Exceptions\Handler`. Class này chứa hai phương thức: `report` và `render`. Chúng ta sẽ tìm hiểu từng phương thức một cách chi tiết. Phương thức `report` được sử dụng để ghi lại log ngoại lệ hoặc gửi chúng đến một dịch vụ bên ngoài như [Bugsnag](https://bugsnag.com) hoặc [Sentry](https://github.com/getsentry/sentry-laravel). Mặc định, phương thức `report` sẽ truyền ngoại lệ cho một class cơ sở, nơi mà ngoại lệ sẽ được log lại. Tuy nhiên, bạn có thể thoải mái log ngoại lệ theo cách bạn muốn.

Ví dụ, nếu bạn cần report các loại ngoại lệ khác nhau theo các cách khác nhau, bạn có thể sử dụng toán tử so sánh `instanceof` của PHP:

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

#### Helper `report`

Thỉnh thoảng bạn có thể cần thông báo một ngoại lệ nhưng vẫn tiếp tục chạy request hiện tại. Hàm helper `report` cho phép bạn nhanh chóng thông báo một ngoại lệ bằng cách sử dụng phương thức `report` trong class `App\Exceptions\Handler` mà không hiển thị lỗi trên trang:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Exception $e) {
            report($e);

            return false;
        }
    }

#### Chặn exceptions theo loại

Thuộc tính `$dontReport` trong class `App\Exceptions\Handler` chứa một mảng các loại của ngoại lệ sẽ không được log lại. Ví dụ: các ngoại lệ do lỗi 404, cũng như một số ngoại lệ khác, không được ghi vào file log của bạn. Bạn có thể thêm các loại ngoại lệ khác vào mảng này nếu cần:

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### Phương thức render

Phương thức `render` chịu trách nhiệm chuyển đổi một ngoại lệ thành một response HTTP cần được gửi về cho trình duyệt. Theo mặc định, ngoại lệ sẽ được truyền cho một lớp cơ sở để tạo ra response cho bạn. Tuy nhiên, bạn có thể tự do kiểm tra loại ngoại lệ hoặc trả về response tùy biến của riêng bạn:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="renderable-exceptions"></a>
### Reportable và Renderable Exceptions

Thay vì cách kiểm tra các loại của ngoại lệ như trong các phương thức `report` và `render` của class `App\Exceptions\Handler`, bạn có thể định nghĩa các phương thức `report` và `render` trực tiếp trên ngoại lệ tùy biến của bạn. Khi các phương thức này tồn tại, chúng sẽ được tự động gọi bởi framework:

    <?php

    namespace App\Exceptions;

    use Exception;

    class RenderException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * Render the exception into an HTTP response.
         *
         * @param  \Illuminate\Http\Request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }

<a name="http-exceptions"></a>
## HTTP Exceptions

Một số ngoại lệ mô tả mã lỗi HTTP từ server. Ví dụ: nó có thể là lỗi "page not found" error (404), "unauthorized error" (401) hoặc thậm chí là lỗi do nhà phát triển tạo ra error 500. Để tạo ra các response như vậy từ bất kỳ đâu trong application của bạn, bạn có thể sử dụng helper `abort`:

    abort(404);

Helper `abort` sẽ ngay lập tức đưa ra một ngoại lệ mà được xử lý bởi class `App\Exceptions\Handler`. Ngoài ra, bạn cũng có thể cung cấp response text:

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### Tuỳ biến page HTTP Error

Laravel giúp dễ dàng tuỳ biến các trang erorr cho các HTTP status code khác nhau. Ví dụ: nếu bạn muốn tùy biến trang erorr cho HTTP status code 404, hãy tạo một file `resources/views/errors/404.blade.php`. File sẽ được hiển thị cho tất cả các erorr 404 do application của bạn tạo ra. Các view trong thư mục này phải được đặt tên để khớp với HTTP status code mà chúng tương ứng. Một instance `HttpException` sẽ được đưa ra bởi hàm `abort` và sẽ được chuyển đến view như là một biến `$exception`:

    <h2>{{ $exception->getMessage() }}</h2>

<a name="logging"></a>
## Logging

Laravel cung cấp một lớp trừu tượng đơn giản trên thư viện [Monolog](https://github.com/seldaek/monolog) mạnh mẽ. Theo mặc định, Laravel được cấu hình để tạo file log cho application của bạn trong thư mục `storage/logs`. Bạn có thể viết thông tin vào log bằng cách sử dụng [facade](/docs/{{version}}/facades) `Log`:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Logger cung cấp tám cấp độ ghi log được định nghĩa trong [RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** và **debug**.

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### Context data

Một mảng dữ liệu context data cũng có thể được truyền vào trong các phương thức log. Dữ liệu context data này sẽ được định dạng và hiển thị với thông điệp log:

    Log::info('User failed to login.', ['id' => $user->id]);

#### Truy cập vào istance Monolog

Monolog có nhiều xử lý bổ sung mà có thể bạn cần sử dụng để log. Nếu cần, bạn có thể truy cập vào instance Monolog đang được sử dụng bởi Laravel:

    $monolog = Log::getMonolog();
