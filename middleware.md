# Middleware

- [Giới thiệu](#introduction)
- [Định nghĩa Middleware](#defining-middleware)
- [Đăng ký Middleware](#registering-middleware)
    - [Global Middleware](#global-middleware)
    - [Gán Middleware vào Routes](#assigning-middleware-to-routes)
    - [Middleware Groups](#middleware-groups)
    - [Middleware Aliases](#middleware-aliases)
    - [Sắp xếp Middleware](#sorting-middleware)
- [Middleware Parameters](#middleware-parameters)
- [Middleware kết thúc](#terminable-middleware)

<a name="introduction"></a>
## Giới thiệu

Middleware cung cấp một cơ chế thuận tiện để xem xét và lọc các request HTTP vào ứng dụng của bạn. Ví dụ: Laravel có chứa một middleware để xác minh người dùng vào ứng dụng của bạn đã được xác thực hay chưa. Nếu người dùng chưa được xác thực, middleware sẽ chuyển hướng người dùng đến màn hình login của application của bạn. Và, nếu người dùng đã được xác thực, middleware sẽ cho phép request đó tiếp tục vào ứng dụng.

Bạn có thể muốn viết thêm các middleware khác để thực hiện các nhiệm vụ khác, ngoài việc xác thực. Ví dụ, một middleware logging có thể log tất cả các request đến application của bạn. Laravel có sẵn nhiều middleware, bao gồm middleware để xác thực và bảo vệ CSRF; tuy nhiên, tất cả các middleware do người dùng định nghĩa thường nằm trong thư mục `app/Http/Middleware` của ứng dụng của bạn.

<a name="defining-middleware"></a>
## Định nghĩa Middleware

Để tạo một middleware mới, hãy dùng lệnh Artisan `make:middleware`:

```shell
php artisan make:middleware EnsureTokenIsValid
```

Lệnh này sẽ lưu một class `EnsureTokenIsValid` mới vào trong thư mục `app/Http/Middleware` của bạn. Trong middleware này, chúng ta sẽ chỉ cho phép truy cập vào route nếu input `token` trùng với một giá trị cụ thể. Và nếu không, chúng ta sẽ chuyển hướng người dùng trở lại URI `/home`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

Như bạn có thể thấy, nếu `token` không trùng với một secret token, thì middleware sẽ trả về một chuyển hướng HTTP cho client; nếu ngược lại, request sẽ được tiếp tục vào ứng dụng. Để request tiếp tục vào ứng dụng, bạn nên gọi một callback là `$next` cùng với `$request`.

Tốt nhất là bạn hãy hình dung middleware như là các "layers" mà các HTTP request phải vượt qua trước khi chúng đến được với ứng dụng của bạn. Mỗi layer có thể kiểm tra request và thậm chí từ chối nó hoàn toàn.

> [!NOTE]
> Tất cả các middleware đều được resolve thông qua [service container](/docs/{{version}}/container), vì vậy bạn có thể khai báo bất kỳ phụ thuộc nào mà bạn cần trong phương thức khởi tạo của middleware.

<a name="middleware-and-responses"></a>
#### Middleware và Responses

Tất nhiên, middleware có thể thực hiện các tác vụ trước hoặc sau khi truyền request vào sâu hơn trong ứng dụng. Ví dụ: middleware ở dưới đây sẽ thực hiện một số tác vụ **trước** khi request được ứng dụng xử lý:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class BeforeMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // Perform action

        return $next($request);
    }
}
```

Tuy nhiên, middleware này sẽ thực hiện nhiệm vụ của mình **sau** khi request được ứng dụng xử lý:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AfterMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```

<a name="registering-middleware"></a>
## Đăng ký Middleware

<a name="global-middleware"></a>
### Global Middleware

Nếu bạn muốn một middleware chạy trong mỗi request HTTP đến application của bạn, bạn có thể thêm nó vào global middleware stack trong file `bootstrap/app.php` của ứng dụng:

```php
use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware) {
     $middleware->append(EnsureTokenIsValid::class);
})
```

Đối tượng `$middleware` được cung cấp cho closure `withMiddleware` là một instance của `Illuminate\Foundation\Configuration\Middleware` và chịu trách nhiệm quản lý middleware được gán cho các route của ứng dụng. Phương thức `append` sẽ thêm middleware vào cuối danh sách global middleware. Nếu bạn muốn thêm middleware vào đầu danh sách, hãy sử dụng phương thức `prepend`.

<a name="manually-managing-laravels-default-global-middleware"></a>
#### Manually Managing Laravel's Default Global Middleware

Nếu bạn muốn tự quản lý stack middleware global của Laravel, bạn có thể cung cấp stack middleware global mặc định của Laravel cho phương thức `use`. Sau đó, bạn có thể điều chỉnh stack middleware mặc định nếu cần:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->use([
        \Illuminate\Foundation\Http\Middleware\InvokeDeferredCallbacks::class,
        // \Illuminate\Http\Middleware\TrustHosts::class,
        \Illuminate\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Http\Middleware\ValidatePostSize::class,
        \Illuminate\Foundation\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ]);
})
```

<a name="assigning-middleware-to-routes"></a>
### Gán Middleware với Routes

Nếu bạn muốn gán một middleware cho một route cụ thể, bạn có thể gọi phương thức `middleware` khi định nghĩa route:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```

Bạn có thể gán nhiều middleware cho một route bằng cách truyền một mảng tên middleware cho phương thức `middleware`:

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

<a name="excluding-middleware"></a>
#### Excluding Middleware

Khi gán một middleware cho một nhóm các route, đôi khi bạn có thể cần ngăn middleware này được chạy cho một route cụ thể trong nhóm. Bạn có thể thực hiện việc này bằng phương thức `withoutMiddleware`:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });

    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

You may also exclude a given set of middleware from an entire [group](/docs/{{version}}/routing#route-groups) of route definitions:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

Phương thức `withoutMiddleware` sẽ chỉ có thể xóa middleware route và không áp dụng được cho [global middleware](#global-middleware).

<a name="middleware-groups"></a>
### Middleware Groups

Thỉnh thoảng bạn cũng có thể muốn group nhiều middleware dưới một tên để dễ dàng gán chúng vào route. Bạn có thể hoàn thành điều này bằng cách sử dụng phương thức `appendToGroup` trong file `bootstrap/app.php` của ứng dụng:

```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;

->withMiddleware(function (Middleware $middleware) {
    $middleware->appendToGroup('group-name', [
        First::class,
        Second::class,
    ]);

    $middleware->prependToGroup('group-name', [
        First::class,
        Second::class,
    ]);
})
```

Các group middleware có thể được gán cho các route và action của controller bằng cách sử dụng cùng cú pháp như các middleware đơn lẻ:

```php
Route::get('/', function () {
    // ...
})->middleware('group-name');

Route::middleware(['group-name'])->group(function () {
    // ...
});
```

<a name="laravels-default-middleware-groups"></a>
#### Laravel's Default Middleware Groups

Laravel đã định nghĩa trước các group middleware `web` và `api`, chứa các middleware phổ biến mà bạn có thể muốn áp dụng cho các web hoặc route API của bạn. Hãy nhớ rằng, Laravel sẽ tự động áp dụng các group middleware này cho các file `routes/web.php` và `routes/api.php`:

<div class="overflow-auto">

| The `web` Middleware Group                                |
| --------------------------------------------------------- |
| `Illuminate\Cookie\Middleware\EncryptCookies`             |
| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` |
| `Illuminate\Session\Middleware\StartSession`              |
| `Illuminate\View\Middleware\ShareErrorsFromSession`       |
| `Illuminate\Foundation\Http\Middleware\PreventRequestForgery` |
| `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` |
| `Illuminate\Routing\Middleware\SubstituteBindings`        |

</div>

<div class="overflow-auto">

| The `api` Middleware Group                         |
| -------------------------------------------------- |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

</div>

Nếu bạn muốn thêm hoặc thêm middleware vào đầu các group này, bạn có thể sử dụng các phương thức `web` và `api` trong file `bootstrap/app.php` của ứng dụng. Các phương thức `web` và `api` là các lựa chọn thay thế tiện lợi cho phương thức `appendToGroup`:

```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

Bạn thậm chí có thể thay một middleware có trong middleware group mặc định của Laravel bằng một middleware tùy chỉnh của riêng bạn:

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

Bạn cũng có thể xóa một middleware ra khỏi group:

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

<a name="manually-managing-laravels-default-middleware-groups"></a>
#### Manually Managing Laravel's Default Middleware Groups

Nếu bạn muốn tự quản lý tất cả các middleware có trong các group middleware `web` và `api` mặc định của Laravel, bạn có thể định nghĩa lại toàn bộ các group này. Ví dụ dưới đây sẽ định nghĩa các group middleware `web` và `api` với các middleware mặc định của chúng, cho phép bạn tùy chỉnh khi cần thiết:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestForgery::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
    ]);

    $middleware->group('api', [
        // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        // 'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ]);
})
```

> [!NOTE]
> Mặc định, group middleware `web` và `api` sẽ được tự động áp dụng cho các file `routes/web.php` và `routes/api.php` tương ứng trong ứng dụng của bạn bởi `bootstrap/app.php` file.

<a name="middleware-aliases"></a>
### Middleware Aliases

Bạn có thể gán các alias cho middleware trong file `bootstrap/app.php` của ứng dụng. Các alias middleware cho phép bạn định nghĩa một tên ngắn gọn hơn cho một class middleware nhất định, điều này đặc biệt hữu ích đối với các middleware có tên class dài:

```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

Sau khi đã định nghĩa alias middleware trong file `bootstrap/app.php` của ứng dụng, bạn có thể sử dụng alias khi gán middleware cho route:

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

Để thuận tiện, một số middleware mặc định của Laravel đã được gán alias. Ví dụ, middleware `auth` là một alias cho middleware `Illuminate\Auth\Middleware\Authenticate`. Dưới đây là danh sách các alias middleware mặc định:

<div class="overflow-auto">

| Alias              | Middleware                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------- |
| `auth`             | `Illuminate\Auth\Middleware\Authenticate`                                                                     |
| `auth.basic`       | `Illuminate\Auth\Middleware\AuthenticateWithBasicAuth`                                                        |
| `auth.session`     | `Illuminate\Session\Middleware\AuthenticateSession`                                                           |
| `cache.headers`    | `Illuminate\Http\Middleware\SetCacheHeaders`                                                                  |
| `can`              | `Illuminate\Auth\Middleware\Authorize`                                                                        |
| `guest`            | `Illuminate\Auth\Middleware\RedirectIfAuthenticated`                                                          |
| `password.confirm` | `Illuminate\Auth\Middleware\RequirePassword`                                                                  |
| `precognitive`     | `Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests`                                            |
| `signed`           | `Illuminate\Routing\Middleware\ValidateSignature`                                                             |
| `subscribed`       | `\Spark\Http\Middleware\VerifyBillableIsSubscribed`                                                           |
| `throttle`         | `Illuminate\Routing\Middleware\ThrottleRequests` or `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` |
| `verified`         | `Illuminate\Auth\Middleware\EnsureEmailIsVerified`                                                            |

</div>

<a name="sorting-middleware"></a>
### Sắp xếp Middleware

Hiếm khi, bạn cần middleware của bạn thực thi theo một thứ tự cụ thể nhưng lại không thể sắp xếp thứ tự của chúng khi chúng được chỉ định cho route. Trong những tình huống như thế này, bạn có thể chỉ định mức độ ưu tiên middleware của bạn bằng cách sử dụng phương thức `priority` trong file `bootstrap/app.php` của ứng dụng:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestForgery::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

<a name="middleware-parameters"></a>
## Middleware Parameters

Middleware cũng có thể nhận vào thêm các tham số bổ sung. Ví dụ: nếu ứng dụng của bạn cần xác minh rằng người dùng đang được xác thực phải có một "role" nhất định thì mới thực hiện được một hành động, vì vậy bạn có thể tạo một middleware `EnsureUserHasRole` nhận thêm tên role làm tham số bổ sung.

Các tham số middleware bổ sung sẽ được truyền đến middleware sau tham số `$next`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }
}
```

Các tham số middleware có thể được định nghĩa khi tạo route bằng cách tách tên của middleware và tham số với một dấu `:`:

```php
use App\Http\Middleware\EnsureUserHasRole;

Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor');
```

Nhiều tham số có thể được phân tách bằng dấu phẩy:

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor,publisher');
```

<a name="terminable-middleware"></a>
## Middleware kết thúc

Đôi khi, một middleware có thể cần thực hiện một số công việc sau khi response HTTP được gửi về broswer. Nếu bạn định nghĩa một phương thức `terminate` trong middleware và web server của bạn đang sử dụng FastCGI, thì phương thức `terminate` sẽ tự động được gọi sau khi response đã được gửi về trình duyệt:

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    /**
     * Handle tasks after the response has been sent to the browser.
     */
    public function terminate(Request $request, Response $response): void
    {
        // ...
    }
}
```

Phương thức `terminate` sẽ nhận vào cả request và response. Khi bạn đã định nghĩa một middleware terminate, bạn nên thêm nó vào danh sách route hoặc global middleware trong file `bootstrap/app.php` của ứng dụng.

Khi gọi phương thức `terminate` trong middleware của bạn, Laravel sẽ resolve một instance mới của middleware từ [service container](/docs/{{version}}/container). Nếu bạn muốn sử dụng lại cùng một instance middleware khi các phương thức `handle` và `terminate` được gọi, hãy đăng ký middleware với container bằng phương thức `singleton` của container. Thông thường, điều này nên được thực hiện trong phương thức `register` của `AppServiceProvider` của bạn:

```php
use App\Http\Middleware\TerminatingMiddleware;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```
