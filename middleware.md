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

Middleware cung cấp một cơ chế thuận tiện để lọc các request HTTP vào ứng dụng của bạn. Ví dụ: Laravel có chứa một middleware để xác minh người dùng vào ứng dụng của bạn đã được xác thực hay chưa. Nếu người dùng chưa được xác thực, middleware sẽ chuyển hướng người dùng đến màn hình login. Và, nếu người dùng đã được xác thực, middleware sẽ cho phép request đó tiếp tục vào ứng dụng.

Bạn có thể muốn viết thêm các middleware khác để thực hiện các nhiệm vụ khác, ngoài việc xác thực. Một middleware CORS có thể chịu trách nhiệm cho việc thêm một thuộc tính header vào tất cả các response mà application của bạn gửi về client. Hoặc là một middleware logging có thể log tất cả các request đến application của bạn.

Có một số middleware đã có sẵn trong framework Laravel, bao gồm cả middleware để xác thực và bảo vệ CSRF. Tất cả các middleware này đều nằm trong thư mục `app/Http/Middleware`.

<a name="defining-middleware"></a>
## Định nghĩa Middleware

Để tạo một middleware mới, hãy dùng lệnh Artisan `make:middleware`:

    php artisan make:middleware CheckAge

Lệnh này sẽ lưu một class `CheckAge` mới vào trong thư mục `app/Http/Middleware` của bạn. Trong middleware này, chúng ta sẽ chỉ cho phép truy cập vào route nếu `age` nhập vào lớn hơn 200. Và nếu không, chúng ta sẽ chuyển hướng người dùng trở lại URI `home`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }

            return $next($request);
        }
    }

Như bạn có thể thấy, nếu `age` đã cho nhỏ hơn hoặc bằng `200`, middleware sẽ trả về một chuyển hướng HTTP cho client; nếu không, request sẽ được tiếp tục vào ứng dụng. Để request tiếp tục vào ứng dụng, hãy gọi một callback là `$next` cùng với `$request`.

Tốt nhất là bạn hãy hình dung middleware như là các "layers" mà các HTTP request phải vượt qua trước khi chúng đến được với ứng dụng của bạn. Mỗi layer có thể kiểm tra request và thậm chí từ chối nó hoàn toàn.

> {tip} Tất cả các middleware đều được resolve thông qua [service container](/docs/{{version}}/container), vì vậy bạn có thể khai báo bất kỳ phụ thuộc nào mà bạn cần trong phương thức khởi tạo của middleware.

### Trước và Sau khi Middleware

Việc một middleware chạy trước hay sau một request phụ thuộc vào chính middleware đó. Ví dụ: middleware ở dưới đây sẽ thực hiện một số tác vụ **trước** khi request được ứng dụng xử lý:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action

            return $next($request);
        }
    }

Tuy nhiên, middleware này sẽ thực hiện nhiệm vụ của mình **sau** khi request được ứng dụng xử lý:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
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

Nếu bạn muốn gán một middleware cho một route cụ thể, trước tiên bạn nên gán middleware đó với một khoá trong file `app/Http/Kernel.php` của bạn. Mặc định, thuộc tính `$routeMiddleware` của class này sẽ chứa sẵn một danh sách middleware đi kèm với Laravel. Để thêm middleware của bạn, hãy thêm nó vào danh sách này và gán cho nó một khóa mà bạn chọn:

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
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

Khi middleware đã được định nghĩa trong HTTP kernel, bạn có thể sử dụng phương thức `middleware` để gán middleware đó cho một route:

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

Bạn cũng có thể gán nhiều middleware cho một route:

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

Khi gán middleware, bạn cũng có thể truyền tên class của middleware:

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### Middleware Groups

Thỉnh thoảng bạn cũng có thể muốn group nhiều middleware dưới một tên để dễ dàng gán chúng với route. Bạn có thể làm diều này bằng cách sử dụng thuộc tính `$middlewareGroups` trong class HTTP kernel của bạn.

Mặc định, Laravel đã có sẵn các group middleware `web` và `api`, chứa các middleware phổ biến mà bạn có thể muốn áp dụng cho các route API hoặc web UI của bạn:

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
            'throttle:60,1',
            'auth:api',
        ],
    ];

Các group middleware có thể được gán cho một route hoặc một controller action bằng cách sử dụng cùng một cú pháp như middleware riêng lẻ. Một lần nữa, các group middleware giúp thuận tiện hơn khi gán nhiều middleware cho một route cùng một lúc:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

    Route::middleware(['web', 'subscribed'])->group(function () {
        //
    });

> {tip} Mặc định, group middleware `web` sẽ được tự động gán cho file `routes/web.php` bởi `RouteServiceProvider`.

<a name="sorting-middleware"></a>
### Sắp xếp Middleware

Hiếm khi, bạn cần middleware của bạn thực thi theo một thứ tự cụ thể nhưng lại không thể sắp xếp thứ tự của chúng khi chúng được chỉ định cho route. Trong trường hợp này, bạn có thể chỉ định mức độ ưu tiên middleware của bạn bằng cách sử dụng thuộc tính `$middlewarePriority` trong file `app/Http/Kernel.php` của bạn:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var array
     */
    protected $middlewarePriority = [
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\Authenticate::class,
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];

<a name="middleware-parameters"></a>
## Middleware Parameters

Middleware cũng có thể nhận vào thêm các tham số bổ sung. Ví dụ: nếu ứng dụng của bạn cần xác minh rằng người dùng đang được xác thực phải có một "role" nhất định thì mới thực hiện được một hành động, vì vậy bạn có thể tạo một middleware `CheckRole` nhận thêm tên role làm tham số bổ sung.

Các tham số middleware bổ sung sẽ được truyền đến middleware sau tham số `$next`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckRole
    {
        /**
         * Handle the incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

Các tham số middleware có thể được định nghĩa khi tạo route bằng cách tách tên của middleware và tham số với một dấu `:`. Nếu có nhiều tham số thì nên được phân cách bằng dấu phẩy:

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## Middleware kết thúc

Đôi khi, một middleware có thể cần thực hiện một số công việc sau khi response HTTP được gửi về broswer. Nếu bạn định nghĩa một phương thức `terminate` trong middleware và web server của bạn đang sử dụng FastCGI, thì phương thức `terminate` sẽ tự động được gọi sau khi response đã được gửi về trình duyệt:

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

Phương thức `terminate` sẽ nhận vào cả request và response. Khi bạn đã định nghĩa một middleware terminate, bạn nên thêm nó vào danh sách route hoặc global middleware trong file `app/Http/Kernel.php`.

Khi gọi phương thức `terminate` trong middleware của bạn, Laravel sẽ resolve một instance mới của middleware từ [service container](/docs/{{version}}/container). Nếu bạn muốn sử dụng lại cùng một instance middleware khi các phương thức `handle` và `terminate` được gọi, hãy đăng ký middleware với container bằng phương thức `singleton` của container. Thông thường, điều này nên được thực hiện trong phương thức `register` của `AppServiceProvider.php` của bạn:

    use App\Http\Middleware\TerminableMiddleware;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(TerminableMiddleware::class);
    }
