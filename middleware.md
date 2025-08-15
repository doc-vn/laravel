# Middleware

- [Giới thiệu](#introduction)
- [Định nghĩa Middleware](#defining-middleware)
- [Đăng ký Middleware](#registering-middleware)
    - [Global Middleware](#global-middleware)
    - [Gán Middleware vào Routes](#assigning-middleware-to-routes)
    - [Middleware Groups](#middleware-groups)
    - [Sắp xếp Middleware](#sorting-middleware)
- [Middleware Parameters](#middleware-parameters)
- [Middleware kết thúc](#terminable-middleware)

<a name="introduction"></a>
## Giới thiệu

Middleware cung cấp một cơ chế thuận tiện để xem xét và lọc các request HTTP vào ứng dụng của bạn. Ví dụ: Laravel có chứa một middleware để xác minh người dùng vào ứng dụng của bạn đã được xác thực hay chưa. Nếu người dùng chưa được xác thực, middleware sẽ chuyển hướng người dùng đến màn hình login của application của bạn. Và, nếu người dùng đã được xác thực, middleware sẽ cho phép request đó tiếp tục vào ứng dụng.

Bạn có thể muốn viết thêm các middleware khác để thực hiện các nhiệm vụ khác, ngoài việc xác thực. Ví dụ, một middleware logging có thể log tất cả các request đến application của bạn. Có một số middleware đã có sẵn trong framework Laravel, bao gồm cả middleware để xác thực và bảo vệ CSRF. Tất cả các middleware này đều nằm trong thư mục `app/Http/Middleware`.

<a name="defining-middleware"></a>
## Định nghĩa Middleware

Để tạo một middleware mới, hãy dùng lệnh Artisan `make:middleware`:

```shell
php artisan make:middleware EnsureTokenIsValid
```

Lệnh này sẽ lưu một class `EnsureTokenIsValid` mới vào trong thư mục `app/Http/Middleware` của bạn. Trong middleware này, chúng ta sẽ chỉ cho phép truy cập vào route nếu input `token` trùng với một giá trị cụ thể. Và nếu không, chúng ta sẽ chuyển hướng người dùng trở lại URI `home`:

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
                return redirect('home');
            }

            return $next($request);
        }
    }

Như bạn có thể thấy, nếu `token` không trùng với một secret token, thì middleware sẽ trả về một chuyển hướng HTTP cho client; nếu ngược lại, request sẽ được tiếp tục vào ứng dụng. Để request tiếp tục vào ứng dụng, bạn nên gọi một callback là `$next` cùng với `$request`.

Tốt nhất là bạn hãy hình dung middleware như là các "layers" mà các HTTP request phải vượt qua trước khi chúng đến được với ứng dụng của bạn. Mỗi layer có thể kiểm tra request và thậm chí từ chối nó hoàn toàn.

> [!NOTE]
> Tất cả các middleware đều được resolve thông qua [service container](/docs/{{version}}/container), vì vậy bạn có thể khai báo bất kỳ phụ thuộc nào mà bạn cần trong phương thức khởi tạo của middleware.

<a name="middleware-and-responses"></a>
#### Middleware và Responses

Tất nhiên, middleware có thể thực hiện các tác vụ trước hoặc sau khi truyền request vào sâu hơn trong ứng dụng. Ví dụ: middleware ở dưới đây sẽ thực hiện một số tác vụ **trước** khi request được ứng dụng xử lý:

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

Tuy nhiên, middleware này sẽ thực hiện nhiệm vụ của mình **sau** khi request được ứng dụng xử lý:

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

<a name="registering-middleware"></a>
## Đăng ký Middleware

<a name="global-middleware"></a>
### Global Middleware

Nếu bạn muốn một middleware chạy trong mỗi request HTTP đến application của bạn, hãy liệt kê class middleware đó trong thuộc tính `$middleware` của class `app/Http/Kernel.php`.

<a name="assigning-middleware-to-routes"></a>
### Gán Middleware với Routes

Nếu bạn muốn gán một middleware cho một route cụ thể, bạn có thể gọi phương thức `middleware` khi định nghĩa route:

    use App\Http\Middleware\Authenticate;

    Route::get('/profile', function () {
        // ...
    })->middleware(Authenticate::class);

Bạn có thể gán nhiều middleware cho một route bằng cách truyền một mảng tên middleware cho phương thức `middleware`:

    Route::get('/', function () {
        // ...
    })->middleware([First::class, Second::class]);

Để thuận tiện, bạn có thể alias danh cho middleware trong file `app/Http/Kernel.php` của application của bạn. Mặc định, thuộc tính `$middlewareAliases` của class này sẽ chứa sẵn một danh sách middleware đi kèm với Laravel. Bạn có thể thêm middleware của bạn vào danh sách này và gán cho nó một alias mà bạn chọn:

    // Within App\Http\Kernel class...

    protected $middlewareAliases = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];

Khi alias của middleware đã được định nghĩa trong HTTP kernel, bạn có thể sử dụng alias đó khi gán một middleware cho một route:

    Route::get('/profile', function () {
        // ...
    })->middleware('auth');

<a name="excluding-middleware"></a>
#### Excluding Middleware

Khi gán một middleware cho một nhóm các route, đôi khi bạn có thể cần ngăn middleware này được chạy cho một route cụ thể trong nhóm. Bạn có thể thực hiện việc này bằng phương thức `withoutMiddleware`:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::middleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/', function () {
            // ...
        });

        Route::get('/profile', function () {
            // ...
        })->withoutMiddleware([EnsureTokenIsValid::class]);
    });

You may also exclude a given set of middleware from an entire [group](/docs/{{version}}/routing#route-groups) of route definitions:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/profile', function () {
            // ...
        });
    });

Phương thức `withoutMiddleware` sẽ chỉ có thể xóa middleware route và không áp dụng được cho [global middleware](#global-middleware).

<a name="middleware-groups"></a>
### Middleware Groups

Thỉnh thoảng bạn cũng có thể muốn group nhiều middleware dưới một tên để dễ dàng gán chúng vào route. Bạn có thể hoàn thành điều này bằng cách sử dụng thuộc tính `$middlewareGroups` trong class HTTP kernel của bạn.

Laravel đã định nghĩa trước các group middleware `web` và `api`, chứa các middleware phổ biến mà bạn có thể muốn áp dụng cho các web hoặc route API của bạn. Hãy nhớ rằng, các group middleware này được service provider `App\Providers\RouteServiceProvider` trong ứng dụng của bạn tự động áp dụng cho các route có trong các file route `web` và `api` của bạn:

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];

Các group middleware có thể được gán cho một route hoặc một controller action bằng cách sử dụng cùng một cú pháp như middleware riêng lẻ. Một lần nữa, các group middleware giúp thuận tiện hơn khi gán nhiều middleware cho một route cùng một lúc:

    Route::get('/', function () {
        // ...
    })->middleware('web');

    Route::middleware(['web'])->group(function () {
        // ...
    });

> [!NOTE]
> Mặc định, group middleware `web` và `api` sẽ được tự động áp dụng cho các file `routes/web.php` và `routes/api.php` tương ứng trong ứng dụng của bạn bởi `App\Providers\RouteServiceProvider`.

<a name="sorting-middleware"></a>
### Sắp xếp Middleware

Hiếm khi, bạn cần middleware của bạn thực thi theo một thứ tự cụ thể nhưng lại không thể sắp xếp thứ tự của chúng khi chúng được chỉ định cho route. Trong trường hợp này, bạn có thể chỉ định mức độ ưu tiên middleware của bạn bằng cách sử dụng thuộc tính `$middlewarePriority` trong file `app/Http/Kernel.php` của bạn. Mặc định, thuộc tính này có thể không tồn tại trong HTTP kernel. Nếu nó không tồn tại, bạn có thể copy định nghĩa của nó ở bên dưới:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var string[]
     */
    protected $middlewarePriority = [
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Contracts\Session\Middleware\AuthenticatesSessions::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];

<a name="middleware-parameters"></a>
## Middleware Parameters

Middleware cũng có thể nhận vào thêm các tham số bổ sung. Ví dụ: nếu ứng dụng của bạn cần xác minh rằng người dùng đang được xác thực phải có một "role" nhất định thì mới thực hiện được một hành động, vì vậy bạn có thể tạo một middleware `EnsureUserHasRole` nhận thêm tên role làm tham số bổ sung.

Các tham số middleware bổ sung sẽ được truyền đến middleware sau tham số `$next`:

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

Các tham số middleware có thể được định nghĩa khi tạo route bằng cách tách tên của middleware và tham số với một dấu `:`:

   Route::put('/post/{id}', function (string $id) {
        // ...
    })->middleware('role:editor');

Nhiều tham số có thể được phân tách bằng dấu phẩy:

    Route::put('/post/{id}', function (string $id) {
        // ...
    })->middleware('role:editor,publisher');

<a name="terminable-middleware"></a>
## Middleware kết thúc

Đôi khi, một middleware có thể cần thực hiện một số công việc sau khi response HTTP được gửi về broswer. Nếu bạn định nghĩa một phương thức `terminate` trong middleware và web server của bạn đang sử dụng FastCGI, thì phương thức `terminate` sẽ tự động được gọi sau khi response đã được gửi về trình duyệt:

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

Phương thức `terminate` sẽ nhận vào cả request và response. Khi bạn đã định nghĩa một middleware terminate, bạn nên thêm nó vào danh sách route hoặc global middleware trong file `app/Http/Kernel.php`.

Khi gọi phương thức `terminate` trong middleware của bạn, Laravel sẽ resolve một instance mới của middleware từ [service container](/docs/{{version}}/container). Nếu bạn muốn sử dụng lại cùng một instance middleware khi các phương thức `handle` và `terminate` được gọi, hãy đăng ký middleware với container bằng phương thức `singleton` của container. Thông thường, điều này nên được thực hiện trong phương thức `register` của `AppServiceProvider` của bạn:

    use App\Http\Middleware\TerminatingMiddleware;

    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(TerminatingMiddleware::class);
    }
