# Error Handling

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Xử lý exception](#the-exception-handler)
    - [Reporting Exceptions](#reporting-exceptions)
    - [Mức độ log exceptions](#exception-log-levels)
    - [Chặn exceptions theo loại](#ignoring-exceptions-by-type)
    - [Rendering Exceptions](#rendering-exceptions)
    - [Reportable và Renderable Exceptions](#renderable-exceptions)
- [Throttling Reported Exceptions](#throttling-reported-exceptions)
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

Nếu bạn cần report các loại ngoại lệ khác nhau theo những cách khác nhau, bạn có thể sử dụng phương thức `reportable` để đăng ký một closure sẽ được thực thi khi cần report một ngoại lệ nhất định. Laravel sẽ xác định ngoại lệ của closure report bằng cách kiểm tra khai báo của closure:

    use App\Exceptions\InvalidOrderException;

    /**
     * Register the exception handling callbacks for the application.
     */
    public function register(): void
    {
        $this->reportable(function (InvalidOrderException $e) {
            // ...
        });
    }

Khi bạn đăng ký custom exception reporting callback bằng phương thức `reportable` xong, Laravel sẽ vẫn ghi log ngoại lệ bằng cách sử dụng cấu hình ghi log mặc định cho ứng dụng. Nếu bạn muốn dừng việc ghi log mặc định đối với ngoại lệ, bạn có thể sử dụng phương thức `stop` khi định nghĩa reporting callback của bạn hoặc trả về `false` từ trong callback:

    $this->reportable(function (InvalidOrderException $e) {
        // ...
    })->stop();

    $this->reportable(function (InvalidOrderException $e) {
        return false;
    });

> [!NOTE]
> Để tùy chỉnh exception reporting cho một exception nhất định, bạn cũng có thể sử dụng [reportable exceptions](/docs/{{version}}/errors#renderable-exceptions)

<a name="global-log-context"></a>
#### Global Log Context

Nếu có sẵn, Laravel sẽ tự động thêm ID của người dùng hiện tại vào mọi log message của ngoại lệ dưới dạng dữ liệu theo ngữ cảnh. Bạn có thể định nghĩa dữ liệu theo ngữ cảnh global của riêng bạn bằng cách định nghĩa một phương thức `context` trên class `App\Exceptions\Handler` trong ứng dụng của bạn. Thông tin này sẽ được thêm vào trong mọi log message của ngoại lệ được viết bởi ứng dụng của bạn:

    /**
     * Get the default context variables for logging.
     *
     * @return array<string, mixed>
     */
    protected function context(): array
    {
        return array_merge(parent::context(), [
            'foo' => 'bar',
        ]);
    }

<a name="exception-log-context"></a>
#### Exception Log Context

Mặc dù việc thêm thông tin vào mọi thông báo log có thể hữu ích, nhưng đôi khi có một số ngoại lệ cụ thể mới có thể có những thông tin mà bạn muốn đưa vào trong log của bạn. Bằng cách định nghĩa phương thức `context` trên một trong các ngoại lệ của ứng dụng, bạn có thể chỉ định bất kỳ dữ liệu nào liên quan đến ngoại lệ đó sẽ được thêm vào trong log của ngoại lệ:

    <?php

    namespace App\Exceptions;

    use Exception;

    class InvalidOrderException extends Exception
    {
        // ...

        /**
         * Get the exception's context information.
         *
         * @return array<string, mixed>
         */
        public function context(): array
        {
            return ['order_id' => $this->orderId];
        }
    }

<a name="the-report-helper"></a>
#### Helper `report`

Thỉnh thoảng bạn có thể cần report một ngoại lệ nhưng vẫn tiếp tục chạy request hiện tại. Hàm helper `report` cho phép bạn nhanh chóng report một ngoại lệ thông qua exception handler mà không cần tạo trên trang lỗi cho người dùng:

    public function isValid(string $value): bool
    {
        try {
            // Validate the value...
        } catch (Throwable $e) {
            report($e);

            return false;
        }
    }

<a name="deduplicating-reported-exceptions"></a>
#### Deduplicating Reported Exceptions

Nếu bạn sử dụng hàm `report` trong toàn bộ ứng dụng, đôi khi bạn có thể report cùng một loại ngoại lệ nhiều lần, và tạo ra các mục trùng nhau trong log của bạn.

Nếu bạn muốn chắc chắn rằng chỉ một instance exception được report trong một lần duy nhất, bạn có thể set thuộc tính `$withoutDuplicates` thành `true` trong class `App\Exceptions\Handler` của application:

```php
namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    /**
     * Indicates that an exception instance should only be reported once.
     *
     * @var bool
     */
    protected $withoutDuplicates = true;

    // ...
}
```

Bây giờ, khi `report` helper được gọi với cùng instance của một exception, thì chỉ lần call đầu tiên sẽ được report:

```php
$original = new RuntimeException('Whoops!');

report($original); // reported

try {
    throw $original;
} catch (Throwable $caught) {
    report($caught); // ignored
}

report($original); // ignored
report($caught); // ignored
```

<a name="exception-log-levels"></a>
### Mức độ log exceptions

Khi một message được ghi vào trong [logs](/docs/{{version}}/logging) trong ứng dụng của bạn, một message sẽ được ghi ở một [log level](/docs/{{version}}/logging#log-levels) nhất định, cho biết mức độ nghiêm trọng hoặc tầm quan trọng của message được ghi lại.

Như đã lưu ý ở trên, ngay cả khi bạn đăng ký một callback custom exception report bằng phương thức `reportable`, Laravel vẫn sẽ ghi log exception bằng cấu hình ghi log mặc định trong ứng dụng; tuy nhiên, vì cấp độ log đôi khi có thể ảnh hưởng đến các channel mà các message sẽ được ghi vào đó nên bạn có thể muốn cấu hình cấp độ log mà một số ngoại lệ nhất định được ghi vào.

Để thực hiện điều này, bạn có thể định nghĩa một thuộc tính `$levels` trong application exception handler. Thuộc tính này sẽ chứa một mảng các loại exception và cấp độ log của chúng:

    use PDOException;
    use Psr\Log\LogLevel;

    /**
     * A list of exception types with their corresponding custom log levels.
     *
     * @var array<class-string<\Throwable>, \Psr\Log\LogLevel::*>
     */
    protected $levels = [
        PDOException::class => LogLevel::CRITICAL,
    ];

<a name="ignoring-exceptions-by-type"></a>
### Chặn exceptions theo loại

Khi xây dựng ứng dụng của bạn, sẽ có một số loại ngoại lệ mà bạn sẽ muốn không bao giờ report. Để chặn những exception này, bạn hãy định nghĩa một thuộc tính `$dontReport` trong application exception handler của bạn. Bất kỳ class nào mà bạn thêm vào thuộc tính này sẽ không được report; tuy nhiên, chúng vẫn có thể có logic rendering riêng:

    use App\Exceptions\InvalidOrderException;

    /**
     * A list of the exception types that are not reported.
     *
     * @var array<int, class-string<\Throwable>>
     */
    protected $dontReport = [
        InvalidOrderException::class,
    ];

Mặc định, Laravel đã bỏ qua một số loại lỗi cho bạn, chẳng hạn như các trường hợp ngoại lệ do lỗi 404 HTTP hoặc lỗi HTTP response 419 được tạo bởi do CSRF token không hợp lệ. Nếu bạn muốn Laravel dừng việc bỏ qua một số loại exception, bạn có thể gọi phương thức `stopIgnoring` trong phương thức `register` trong exception handler của bạn:

    use Symfony\Component\HttpKernel\Exception\HttpException;

    /**
     * Register the exception handling callbacks for the application.
     */
    public function register(): void
    {
        $this->stopIgnoring(HttpException::class);

        // ...
    }

<a name="rendering-exceptions"></a>
### Rendering Exceptions

Mặc định, Laravel exception handler sẽ chuyển một ngoại lệ thành một response HTTP cho bạn. Tuy nhiên, bạn có thể tự do đăng ký một custom rendering closure cho các exception của một loại nhất định. Bạn có thể thực hiện điều này bằng cách gọi phương thức `renderable` trong exception handler của bạn.

Closure được truyền cho phương thức `renderable` sẽ phải trả về một instance của `Illuminate\Http\Response`, có thể được tạo thông qua helper `response`. Laravel sẽ xác định ngoại lệ mà closure render bằng cách kiểm tra khai báo của closure:

    use App\Exceptions\InvalidOrderException;
    use Illuminate\Http\Request;

    /**
     * Register the exception handling callbacks for the application.
     */
    public function register(): void
    {
        $this->renderable(function (InvalidOrderException $e, Request $request) {
            return response()->view('errors.invalid-order', [], 500);
        });
    }

Bạn cũng có thể sử dụng phương thức `renderable` để ghi đè các hành động rendering cho các ngoại lệ được tích hợp sẵn trong Laravel hoặc Symfony, chẳng hạn như `NotFoundHttpException`. Nếu closure được cung cấp cho phương thức `renderable` không trả về giá trị, rendering ngoại lệ mặc định của Laravel sẽ được sử dụng:

    use Illuminate\Http\Request;
    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

    /**
     * Register the exception handling callbacks for the application.
     */
    public function register(): void
    {
        $this->renderable(function (NotFoundHttpException $e, Request $request) {
            if ($request->is('api/*')) {
                return response()->json([
                    'message' => 'Record not found.'
                ], 404);
            }
        });
    }

<a name="renderable-exceptions"></a>
### Reportable và Renderable Exceptions

Thay vì định nghĩa ra một custom report và cách xử lý report đó trong phương thức `register`của exception handler của bạn, bạn có thể định nghĩa các phương thức `report` và `render` trực tiếp trên các ngoại lệ trong ứng dụng của bạn. Khi các phương thức này đã tồn tại, chúng sẽ được gọi tự động bởi framework:

    <?php

    namespace App\Exceptions;

    use Exception;
    use Illuminate\Http\Request;
    use Illuminate\Http\Response;

    class InvalidOrderException extends Exception
    {
        /**
         * Report the exception.
         */
        public function report(): void
        {
            // ...
        }

        /**
         * Render the exception into an HTTP response.
         */
        public function render(Request $request): Response
        {
            return response(/* ... */);
        }
    }

Nếu ngoại lệ của bạn được extend từ một ngoại lệ đã có sẵn renderable, chẳng hạn như ngoại lệ được tích hợp sẵn của Laravel hoặc Symfony, bạn có thể trả về `false` từ phương thức `render` của ngoại lệ để hiển thị HTTP response mặc định của ngoại lệ:

    /**
     * Render the exception into an HTTP response.
     */
    public function render(Request $request): Response|bool
    {
        if (/** Determine if the exception needs custom rendering */) {

            return response(/* ... */);
        }

        return false;
    }

Nếu ngoại lệ của bạn chứa logic reporting tùy chỉnh mà chỉ cần thiết khi một số điều kiện nhất định được đáp ứng, bạn có thể cần hướng dẫn Laravel báo cáo ngoại lệ này bằng cách sử dụng cấu hình xử lý ngoại lệ mặc định. Để thực hiện điều này, bạn có thể trả về `false` từ phương thức `report` của ngoại lệ:

    /**
     * Report the exception.
     */
    public function report(): bool
    {
        if (/** Determine if the exception needs custom reporting */) {

            // ...

            return true;
        }

        return false;
    }

> [!NOTE]
> Bạn có thể khai báo bất kỳ phụ thuộc nào bắt buộc của phương thức `report` và chúng sẽ tự động được tích hợp vào trong phương thức bởi [service container](/docs/{{version}}/container).

<a name="throttling-reported-exceptions"></a>
### Throttling Reported Exceptions

Nếu ứng dụng của bạn report ra một số lượng rất lớn các exception, bạn có thể muốn đưa ra số lượng thực sự bao nhiêu exception đã được log và đã được gửi cho hệ thống tracking error của một service bên ngoài.

Để lấy ra tỷ lệ ngẫu nhiên của một exception, bạn có thể trả về một instance `Lottery` từ phương thức `throttle` trong exception handler của bạn. Nếu class của bạn không có chứa phương thức đó, bạn chỉ đơn giản là thêm nó vào class:

```php
use Illuminate\Support\Lottery;
use Throwable;

/**
 * Throttle incoming exceptions.
 */
protected function throttle(Throwable $e): mixed
{
    return Lottery::odds(1, 1000);
}
```

Nó cũng có thể thêm điều kiện cho một loại exception cụ thể. Nếu bạn chỉ muốn lấy ra một instance mẫu của một class exception cụ thể, bạn có thể trả về một instance `Lottery` cho class đó:

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

/**
 * Throttle incoming exceptions.
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof ApiMonitoringException) {
        return Lottery::odds(1, 1000);
    }
}
```

Bạn có thể đặt một giới hạn cho một exception được log và gửi cho service error tracking của bên thứ ba bằng cách trả về một instance `Limit` thay vì một instance `Lottery`. Nó sẽ hữu dụng nếu bạn muốn bảo vệ để chống lại các ngoại lệ bị đột ngột tạo ra trong log của bạn, ví dụ, khi dịch vụ của bên thứ ba mà ứng dụng của bạn sử dụng bị ngừng hoạt động:

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

/**
 * Throttle incoming exceptions.
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof BroadcastException) {
        return Limit::perMinute(300);
    }
}
```

Mặc định, limit sẽ sử dụng class của exception làm khóa giới hạn. Bạn có thể tùy chỉnh điều này bằng cách chỉ định khóa của riêng bạn bằng phương thức `by` trên `Limit`:

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

/**
 * Throttle incoming exceptions.
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof BroadcastException) {
        return Limit::perMinute(300)->by($e->getMessage());
    }
}
```

Tất nhiên, bạn có thể trả về một mix của instance `Lottery` và instance `Limit` cho các trường hợp exception khác nhau:

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;

/**
 * Throttle incoming exceptions.
 */
protected function throttle(Throwable $e): mixed
{
    return match (true) {
        $e instanceof BroadcastException => Limit::perMinute(300),
        $e instanceof ApiMonitoringException => Lottery::odds(1, 1000),
        default => Limit::none(),
    };
}
```

<a name="http-exceptions"></a>
## HTTP Exceptions

Một số ngoại lệ mô tả mã lỗi HTTP từ server. Ví dụ: nó có thể là lỗi "page not found" error (404), "unauthorized error" (401), hoặc thậm chí là lỗi do nhà phát triển tạo ra error (500). Để tạo ra các response như vậy từ bất kỳ đâu trong application của bạn, bạn có thể sử dụng helper `abort`:

    abort(404);

<a name="custom-http-error-pages"></a>
### Tuỳ biến page HTTP Error

Laravel giúp dễ dàng tuỳ biến các trang error có HTTP status code khác nhau. Ví dụ: để tùy biến trang erorr có HTTP status code 404, hãy tạo một file view template `resources/views/errors/404.blade.php`. File view sẽ được hiển thị cho tất cả các erorr 404 do application của bạn tạo ra. Các view trong thư mục này phải được đặt tên khớp với HTTP status code tương ứng. Một instance `Symfony\Component\HttpKernel\Exception\HttpException` sẽ được đưa ra bởi hàm `abort` và sẽ được chuyển đến view như là một biến `$exception`:

    <h2>{{ $exception->getMessage() }}</h2>

Bạn có thể export các trang template lỗi mặc định của Laravel bằng lệnh Artisan `vendor:publish`. Khi các template này đã được export, bạn có thể tùy chỉnh chúng theo ý thích của bạn:

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### Fallback HTTP Error Pages

Bạn cũng có thể định nghĩa một trang lỗi "dự phòng" cho một loạt các HTTP status code nhất định. Trang này sẽ được hiển thị nếu không có trang HTTP status code nào tương ứng. Để thực hiện điều này, hãy định nghĩa một template `4xx.blade.php` và một template `5xx.blade.php` trong thư mục `resources/views/errors` của ứng dụng của bạn.
