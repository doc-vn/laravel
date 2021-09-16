# Middleware

- [Giới thiệu](#introduction)
- [Định nghĩa Middleware](#defining-middleware)
- [Đăng ký Middleware](#registering-middleware)
    - [Global Middleware](#global-middleware)
    - [Gán Middleware vào Routes](#assigning-middleware-to-routes)
    - [Middleware Groups](#middleware-groups)
- [Middleware Parameters](#middleware-parameters)
- [Middleware kết thúc](#terminable-middleware)

<a name="introduction"></a>
## Giới thiệu

Middleware cung cấp một cơ chế thuận tiện để lọc các request HTTP vào ứng dụng của bạn. Ví dụ: Laravel có chứa một middleware để xác minh người dùng vào ứng dụng của bạn đã được xác thực hay chưa. Nếu người dùng chưa được xác thực, middleware sẽ chuyển hướng người dùng đến màn hình login. Và, nếu người dùng đã được xác thực rồi, middleware sẽ cho phép request tiếp tục vào ứng dụng.

Và dĩ nhiên, bạn có thể muốn viết thêm các middleware khác để thực hiện các nhiệm vụ khác nhau, ngoài việc xác thực. Một middleware CORS có thể chịu trách nhiệm cho việc thêm một thuộc tính header vào tất cả các response mà application của bạn gửi về client. Hoặc là một middleware logging có thể log tất cả các request đến application của bạn.

Có một số middleware đã được chứa sẵn trong framework Laravel, bao gồm middleware để xác thực và bảo vệ CSRF. Tất cả các middleware này đều nằm trong thư mục `app/Http/Middleware`.

<a name="defining-middleware"></a>
## Định nghĩa Middleware

Để tạo một middleware mới, hãy dùng lệnh Artisan `make:middleware`:

    php artisan make:middleware CheckAge

Lệnh này sẽ lưu một class `CheckAge` mới vào trong thư mục `app/Http/Middleware` của bạn. Trong middleware này, chúng ta sẽ chỉ cho phép truy cập vào route nếu `age` được nhập vào lớn hơn 200. Nếu không, chúng ta sẽ chuyển hướng người dùng trở lại URI `home`.

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

Như bạn có thể thấy, nếu `age` đã cho nhỏ hơn hoặc bằng `200`, middleware sẽ trả về một chuyển hướng HTTP cho client; nếu không, request sẽ được chuyển tiếp vào ứng dụng. Để request tiếp tục vào ứng dụng, hãy gọi một callback là `$next` cùng với `$request`.

Tốt nhất là hãy hình dung middleware như là một loạt các "layers" mà các HTTP request phải vượt qua trước khi chúng đến ứng dụng của bạn. Mỗi lớp có thể kiểm tra request và thậm chí từ chối nó hoàn toàn.

### Trước và Sau khi Middleware

Việc một middleware chạy trước hay sau một request phụ thuộc vào chính middleware. Ví dụ: middleware ở dưới đây sẽ thực hiện một số tác vụ **trước** khi request được ứng dụng xử lý:

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

Nếu bạn muốn một middleware chạy trong mỗi request HTTP đến application của bạn, hãy liệt kê class middleware trong thuộc tính `$middleware` của class `app/Http/Kernel.php` của bạn.

<a name="assigning-middleware-to-routes"></a>
### Gán Middleware với Routes

Nếu bạn muốn gán middleware cho các route cụ thể, trước tiên bạn nên gán middleware với một khoá trong file `app/Http/Kernel.php` của bạn. Mặc định, thuộc tính `$routeMiddleware` của class này chứa sẵn một danh sách middleware đi kèm với Laravel. Để thêm middleware riêng của bạn, hãy thêm nó vào danh sách này và gán cho nó một khóa bạn chọn. Ví dụ:

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

Khi middleware đã được định nghĩa trong HTTP kernel, bạn có thể sử dụng phương thức `middleware` để gán middleware cho một route:

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

Bạn cũng có thể gán nhiều middleware cho một route:

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

Khi gán middleware, bạn cũng có thể truyền tên của class middleware:

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### Middleware Groups

Thỉnh thoảng bạn có thể muốn group nhiều middleware dưới một khoá để dễ dàng gán chúng tới route. Bạn có thể làm diều đó bằng cách sử dụng thuộc tính `$middlewareGroups` trong class HTTP kernel của bạn.

Mặc định, Laravel đi kèm với các group middleware `web` và `api` có chứa middleware phổ biến mà bạn có thể muốn áp dụng cho các route API và web UI của bạn:

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

Các middleware group có thể được gán cho các route và các controller action bằng cách sử dụng cùng một cú pháp như middleware riêng lẻ. Một lần nữa, các middleware group giúp thuận tiện hơn khi gán nhiều middleware cho một route cùng một lúc:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

> {tip} Mặc định, group middleware `web` sẽ được tự động áp dụng cho file `routes/web.php` bởi `RouteServiceProvider`.

<a name="middleware-parameters"></a>
## Middleware Parameters

Middleware cũng có thể nhận thêm các tham số bổ sung. Ví dụ: nếu ứng dụng của bạn cần xác minh rằng người dùng đang được xác thực phải có một "role" nhất định thì mới thực hiện được một hành động nhất định, vì vậy bạn có thể tạo middleware `CheckRole` nhận thêm tên role làm tham số bổ sung.

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

Đôi khi, một middleware có thể cần thực hiện một số công việc sau khi response HTTP  đã được gửi về trình duyệt. Ví dụ: middleware "session" đi kèm với Laravel dùng để ghi dữ liệu session vào bộ nhớ sau khi response đã được gửi về trình duyệt. Nếu bạn định nghĩa một phương thức `terminate` trong middleware của mình, thì nó sẽ tự động được gọi sau khi response được gửi về trình duyệt.

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

Phương thức `terminate` sẽ nhận được cả request và response. Khi bạn đã định nghĩa một middleware kết thúc, bạn nên thêm nó vào danh sách route hoặc global middleware trong file `app/Http/Kernel.php`.

Khi gọi phương thức  `terminate` trong middleware của bạn, Laravel sẽ resolve một instance mới của middleware từ [service container](/docs/{{version}}/container). Nếu bạn muốn sử dụng cùng một instance middleware khi các phương thức `handle` và `terminate` được gọi, hãy đăng ký middleware với container bằng phương thức `singleton` của container.
