# Error Handling

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Xử lý exception](#the-exception-handler)
    - [Reporting Exceptions](#reporting-exceptions)
    - [Chặn exceptions theo loại](#ignoring-exceptions-by-type)
    - [Rendering Exceptions](#rendering-exceptions)
    - [Reportable và Renderable Exceptions](#renderable-exceptions)
- [HTTP Exceptions](#http-exceptions)
    - [Tuỳ biến page HTTP Error](#custom-http-error-pages)

<a name="introduction"></a>
## Giới thiệu

Khi bạn bắt đầu một dự án mới, các xử lý lỗi và các ngoại lệ đã được cấu hình sẵn cho bạn. Class `App\Exceptions\Handler` là nơi mà tất cả các ngoại lệ sẽ được đưa ra trong application, tại đó các ngoại lệ sẽ được log và sau đó được hiển thị cho người dùng. Chúng ta sẽ đi sâu hơn vào class này trong các phần còn lại của tài liệu.

<a name="configuration"></a>
## Cấu hình

Tùy chọn `debug` trong file cấu hình `config/app.php` của bạn sẽ định nghĩa lượng thông tin về lỗi sẽ được hiển thị cho người dùng. Mặc định, tùy chọn này được khai báo từ giá trị biến môi trường `APP_DEBUG` được lưu trữ trong file `.env` của bạn.

Trong quá trình phát triển ở local, thì bạn nên lưu biến môi trường `APP_DEBUG` thành `true`. **Nếu môi trường chạy product, thì bạn nên lưu giá trị này là `false`. Nếu giá trị được lưu thành `true` trong môi trường product, bạn có thể có nguy cơ lộ các giá trị cấu hình nhạy cảm cho người dùng application.**

<a name="the-exception-handler"></a>
## Xử lý exception

<a name="reporting-exceptions"></a>
### Reporting Exceptions

Tất cả các ngoại lệ sẽ được xử lý bởi class `App\Exceptions\Handler`. Class này chứa một phương thức `register` là nơi bạn có thể đăng ký các exception reporting tùy biến của bạn và các rendering callback. Chúng ta sẽ xem xét chi tiết từng khái niệm này. Exception reporting sẽ được sử dụng để ghi lại log hoặc gửi ngoại lệ đến một dịch vụ bên ngoài như là [Flare](https://flareapp.io), [Bugsnag](https://bugsnag.com) hoặc [Sentry](https://github.com/getsentry/sentry-laravel). Mặc định, ngoại lệ sẽ được log lại trên cấu hình [ghi log](/docs/{{version}}/logging) của bạn. Tuy nhiên, bạn có thể thoải mái log ngoại lệ theo cách bạn muốn.

Ví dụ: nếu bạn cần report các loại ngoại lệ khác nhau theo những cách khác nhau, bạn có thể sử dụng phương thức `reportable` để đăng ký một closure sẽ được thực thi khi cần report một ngoại lệ nhất định. Laravel sẽ phân loại ngoại lệ của closure report bằng cách kiểm tra khai báo của closure:

    use App\Exceptions\InvalidOrderException;

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->reportable(function (InvalidOrderException $e) {
            //
        });
    }

Khi bạn đăng ký custom exception reporting callback bằng phương thức `reportable` xong, Laravel sẽ vẫn ghi log ngoại lệ bằng cách sử dụng cấu hình ghi log mặc định cho ứng dụng. Nếu bạn muốn dừng việc ghi log mặc định đối với ngoại lệ, bạn có thể sử dụng phương thức `stop` khi định nghĩa reporting callback của bạn hoặc trả về `false` từ trong callback:

    $this->reportable(function (InvalidOrderException $e) {
        //
    })->stop();

    $this->reportable(function (InvalidOrderException $e) {
        return false;
    });

> {tip} Để tùy chỉnh exception reporting cho một exception nhất định, bạn cũng có thể sử dụng [reportable exceptions](/docs/{{version}}/errors#renderable-exceptions)

<a name="global-log-context"></a>
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

<a name="exception-log-context"></a>
#### Exception Log Context

Mặc dù việc thêm thông tin vào mọi thông báo log có thể hữu ích, nhưng đôi khi có một số ngoại lệ cụ thể mới có thể có những thông tin mà bạn muốn đưa vào trong log của bạn. Bằng cách định nghĩa phương thức `context` trên một trong các ngoại lệ tùy chỉnh của ứng dụng, bạn có thể chỉ định bất kỳ dữ liệu nào liên quan đến ngoại lệ đó sẽ được thêm vào trong log của ngoại lệ:

    <?php

    namespace App\Exceptions;

    use Exception;

    class InvalidOrderException extends Exception
    {
        // ...

        /**
         * Get the exception's context information.
         *
         * @return array
         */
        public function context()
        {
            return ['order_id' => $this->orderId];
        }
    }

<a name="the-report-helper"></a>
#### Helper `report`

Thỉnh thoảng bạn có thể cần report một ngoại lệ nhưng vẫn tiếp tục chạy request hiện tại. Hàm helper `report` cho phép bạn nhanh chóng report một ngoại lệ thông qua exception handler mà không cần tạo trên trang lỗi cho người dùng:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Throwable $e) {
            report($e);

            return false;
        }
    }

<a name="ignoring-exceptions-by-type"></a>
### Chặn exceptions theo loại

Khi xây dựng ứng dụng của bạn, sẽ có một số loại ngoại lệ mà bạn chỉ muốn bỏ qua và không muốn report. Exception handler của ứng dụng của bạn có chứa một thuộc tính `$dontReport`là một mảng trống được khởi tạo sẵn. Bất kỳ class nào mà bạn thêm vào thuộc tính này sẽ không được report; tuy nhiên, chúng vẫn có thể có logic rendering riêng:

    use App\Exceptions\InvalidOrderException;

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        InvalidOrderException::class,
    ];

> {tip} Hậu trường, Laravel đã bỏ qua một số loại lỗi cho bạn, chẳng hạn như các trường hợp ngoại lệ do lỗi 404 HTTP "không tìm thấy" hoặc lỗi HTTP response 419 được tạo bởi do CSRF token không hợp lệ.

<a name="rendering-exceptions"></a>
### Rendering Exceptions

Mặc định, Laravel exception handler sẽ chuyển một ngoại lệ thành một response HTTP cho bạn. Tuy nhiên, bạn có thể tự do đăng ký một custom rendering closure cho các exception của một loại nhất định. Bạn có thể thực hiện điều này thông qua phương thức `renderable` trong exception handler của bạn.

Closure được truyền cho phương thức `renderable` sẽ phải trả về một instance của `Illuminate\Http\Response`, có thể được tạo thông qua helper `response`. Laravel sẽ phân loại ngoại lệ mà closure render bằng cách kiểm tra khai báo của closure:

    use App\Exceptions\InvalidOrderException;

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (InvalidOrderException $e, $request) {
            return response()->view('errors.invalid-order', [], 500);
        });
    }

Bạn cũng có thể sử dụng phương thức `renderable` để ghi đè các hành động rendering cho các ngoại lệ được tích hợp sẵn trong Laravel hoặc Symfony, chẳng hạn như `NotFoundHttpException`. Nếu closure được cung cấp cho phương thức `renderable` không trả về giá trị, rendering ngoại lệ mặc định của Laravel sẽ được sử dụng:

    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (NotFoundHttpException $e, $request) {
            if ($request->is('api/*')) {
                return response()->json([
                    'message' => 'Record not found.'
                ], 404);
            }
        });
    }

<a name="renderable-exceptions"></a>
### Reportable và Renderable Exceptions

Thay vì cách kiểm tra các loại của ngoại lệ như trong các phương thức `register` của class `App\Exceptions\Handler`, bạn có thể định nghĩa các phương thức `report` và `render` trực tiếp trên các ngoại lệ tùy biến của bạn. Khi các phương thức này đã tồn tại, chúng sẽ được gọi tự động bởi framework:

    <?php

    namespace App\Exceptions;

    use Exception;

    class InvalidOrderException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return bool|null
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

Nếu ngoại lệ của bạn được extend từ một ngoại lệ đã có sẵn renderable, chẳng hạn như ngoại lệ được tích hợp sẵn của Laravel hoặc Symfony, bạn có thể trả về `false` từ phương thức `render` của ngoại lệ để hiển thị HTTP response mặc định của ngoại lệ:

    /**
     * Render the exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function render($request)
    {
        // Determine if the exception needs custom rendering...

        return false;
    }

Nếu ngoại lệ của bạn chứa logic reporting tùy chỉnh mà chỉ cần thiết khi một số điều kiện nhất định được đáp ứng, bạn có thể cần hướng dẫn Laravel báo cáo ngoại lệ này bằng cách sử dụng cấu hình xử lý ngoại lệ mặc định. Để thực hiện điều này, bạn có thể trả về `false` từ phương thức `report` của ngoại lệ:

    /**
     * Report the exception.
     *
     * @return bool|null
     */
    public function report()
    {
        // Determine if the exception needs custom reporting...

        return false;
    }

> {tip} Bạn có thể khai báo bất kỳ phụ thuộc nào bắt buộc của phương thức `report` và chúng sẽ tự động được tích hợp vào trong phương thức bởi [service container](/docs/{{version}}/container).

<a name="http-exceptions"></a>
## HTTP Exceptions

Một số ngoại lệ mô tả mã lỗi HTTP từ server. Ví dụ: nó có thể là lỗi "page not found" error (404), "unauthorized error" (401) hoặc thậm chí là lỗi do nhà phát triển tạo ra error (500). Để tạo ra các response như vậy từ bất kỳ đâu trong application của bạn, bạn có thể sử dụng helper `abort`:

    abort(404);

<a name="custom-http-error-pages"></a>
### Tuỳ biến page HTTP Error

Laravel giúp dễ dàng tuỳ biến các trang error có HTTP status code khác nhau. Ví dụ: nếu bạn muốn tùy biến trang erorr có HTTP status code 404, hãy tạo một file view template `resources/views/errors/404.blade.php`. File view sẽ được hiển thị cho tất cả các erorr 404 do application của bạn tạo ra. Các view trong thư mục này phải được đặt tên khớp với HTTP status code tương ứng. Một instance `Symfony\Component\HttpKernel\Exception\HttpException` sẽ được đưa ra bởi hàm `abort` và sẽ được chuyển đến view như là một biến `$exception`:

    <h2>{{ $exception->getMessage() }}</h2>

Bạn có thể export các trang template lỗi mặc định của Laravel bằng lệnh Artisan `vendor:publish`. Khi các template này đã được export, bạn có thể tùy chỉnh chúng theo ý thích của bạn:

    php artisan vendor:publish --tag=laravel-errors
