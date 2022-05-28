# Error Handling

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Xử lý exception](#the-exception-handler)
    - [Phương thức report](#report-method)
    - [Phương thức render](#render-method)
    - [Reportable và Renderable Exceptions](#renderable-exceptions)
- [HTTP Exceptions](#http-exceptions)
    - [Tuỳ biến page HTTP Error](#custom-http-error-pages)

<a name="introduction"></a>
## Giới thiệu

Khi bạn bắt đầu một dự án mới, các xử lý lỗi và các ngoại lệ đã được cấu hình sẵn cho bạn. Class `App\Exceptions\Handler` là nơi mà tất cả các ngoại lệ sẽ được gọi tới trong application, tại đó các ngoại lệ sẽ được log và sau đó được hiển thị trở lại cho người dùng. Chúng ta sẽ đi sâu hơn vào class này trong các phần còn lại của tài liệu.

<a name="configuration"></a>
## Cấu hình

Tùy chọn `debug` trong file cấu hình `config/app.php` của bạn sẽ định nghĩa lượng thông tin về lỗi sẽ được hiển thị cho người dùng. Mặc định, tùy chọn này được khai báo từ giá trị biến môi trường `APP_DEBUG` được lưu trữ trong file `.env` của bạn.

Nếu môi trường phát triển ở local, thì bạn nên lưu biến môi trường `APP_DEBUG` thành `true`. Nếu môi trường chạy product, thì bạn nên lưu giá trị này là `false`. Nếu giá trị được lưu thành `true` trong môi trường product, bạn có thể có nguy cơ lộ các giá trị cấu hình nhạy cảm cho người dùng application.

<a name="the-exception-handler"></a>
## Xử lý exception

<a name="report-method"></a>
### Phương thức report

Tất cả các ngoại lệ sẽ được xử lý bởi class `App\Exceptions\Handler`. Class này chứa hai phương thức: `report` và `render`. Chúng ta sẽ tìm hiểu từng phương thức này một cách chi tiết. Phương thức `report` được sử dụng để ghi lại log hoặc gửi ngoại lệ đến một dịch vụ bên ngoài như là [Flare](https://flareapp.io), [Bugsnag](https://bugsnag.com) hoặc [Sentry](https://github.com/getsentry/sentry-laravel). Mặc định, phương thức `report` sẽ truyền ngoại lệ cho một class cơ sở, nơi mà ngoại lệ sẽ được log lại. Tuy nhiên, bạn có thể thoải mái log ngoại lệ theo cách bạn muốn.

Ví dụ, nếu bạn cần report các loại ngoại lệ khác nhau theo các cách khác nhau, bạn có thể sử dụng toán tử so sánh `instanceof` của PHP:

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Flare, Sentry, Bugsnag, etc.
     *
     * @param  \Throwable  $exception
     * @return void
     */
    public function report(Throwable $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        parent::report($exception);
    }

> {tip} Thay vì thực hiện nhiều kiểm tra `instanceof` trong phương thức `report` của bạn, hãy thử sử dụng [reportable exceptions](/docs/{{version}}/errors#renderable-exceptions)

#### Global Log Context

Nếu có sẵn, Laravel sẽ tự động thêm ID của người dùng hiện tại vào mọi log message của ngoại lệ dưới dạng dữ liệu theo ngữ cảnh. Bạn có thể định nghĩa dữ liệu theo ngữ cảnh global của riêng bạn bằng cách ghi đè phương thức `context` của class `App\Exceptions\Handler` trong ứng dụng của bạn. Thông tin này sẽ được thêm vào trong mọi log message của ngoại lệ được viết bởi ứng dụng của bạn:

    /**
     * Get the default context variables for logging.
     *
     * @return array
     */
    protected function context()
    {
        return array_merge(parent::context(), [
            'foo' => 'bar',
        ]);
    }

#### Helper `report`

Thỉnh thoảng bạn có thể cần report một ngoại lệ nhưng vẫn tiếp tục chạy request hiện tại. Hàm helper `report` cho phép bạn nhanh chóng report một ngoại lệ bằng cách sử dụng phương thức `report` trong class `App\Exceptions\Handler` mà không hiển thị lỗi đó trên trang:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Throwable $e) {
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

Phương thức `render` chịu trách nhiệm chuyển một ngoại lệ thành một response HTTP sẽ được gửi về cho trình duyệt. Mặc định, ngoại lệ sẽ được truyền cho một class cơ sở để tạo ra response cho bạn. Tuy nhiên, bạn có thể tự do kiểm tra loại ngoại lệ hoặc trả về response tùy biến của riêng bạn:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Throwable  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Throwable $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="renderable-exceptions"></a>
### Reportable và Renderable Exceptions

Thay vì cách kiểm tra các loại của ngoại lệ như trong các phương thức `report` và `render` của class `App\Exceptions\Handler`, bạn có thể định nghĩa các phương thức `report` và `render` trực tiếp trên ngoại lệ tùy biến của bạn. Khi các phương thức này đã tồn tại, chúng sẽ được gọi tự động bởi framework:

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
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }

> {tip} Bạn có thể khai báo bất kỳ phụ thuộc nào bắt buộc của phương thức `report` và chúng sẽ tự động được tích hợp vào trong phương thức bởi [service container](/docs/{{version}}/container).

<a name="http-exceptions"></a>
## HTTP Exceptions

Một số ngoại lệ mô tả mã lỗi HTTP từ server. Ví dụ: nó có thể là lỗi "page not found" error (404), "unauthorized error" (401) hoặc thậm chí là lỗi do nhà phát triển tạo ra error (500). Để tạo ra các response như vậy từ bất kỳ đâu trong application của bạn, bạn có thể sử dụng helper `abort`:

    abort(404);

Helper `abort` sẽ ngay lập tức đưa ra một ngoại lệ mà được xử lý bởi class `App\Exceptions\Handler`. Ngoài ra, bạn cũng có thể cung cấp response text:

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### Tuỳ biến page HTTP Error

Laravel giúp dễ dàng tuỳ biến các trang error có HTTP status code khác nhau. Ví dụ: nếu bạn muốn tùy biến trang erorr có HTTP status code 404, hãy tạo một file `resources/views/errors/404.blade.php`. File sẽ được hiển thị cho tất cả các erorr 404 do application của bạn tạo ra. Các view trong thư mục này phải được đặt tên khớp với HTTP status code tương ứng. Một instance `HttpException` sẽ được đưa ra bởi hàm `abort` và sẽ được chuyển đến view như là một biến `$exception`:

    <h2>{{ $exception->getMessage() }}</h2>

Bạn có thể export các trang template lỗi của Laravel bằng lệnh Artisan `vendor:publish`. Khi các template này đã được export, bạn có thể tùy chỉnh chúng theo ý thích của bạn:

    php artisan vendor:publish --tag=laravel-errors
