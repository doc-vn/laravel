# Authentication

- [Giới thiệu](#introduction)
    - [Database Considerations](#introduction-database-considerations)
- [Authentication Quickstart](#authentication-quickstart)
    - [Routing](#included-routing)
    - [Views](#included-views)
    - [Authenticating](#included-authenticating)
    - [Lấy user đã được authenticate](#retrieving-the-authenticated-user)
    - [Bảo vệ route](#protecting-routes)
    - [Login Throttling](#login-throttling)
- [Authenticate user thủ công](#authenticating-users)
    - [Nhớ tài khoản user](#remembering-users)
    - [Các phương thức authentication khác](#other-authentication-methods)
- [HTTP Basic Authentication](#http-basic-authentication)
    - [Stateless HTTP Basic Authentication](#stateless-http-basic-authentication)
- [Authentication bằng bên thứ ba](https://github.com/laravel/socialite)
- [Thêm tuỳ biến guard](#adding-custom-guards)
- [Thêm tuỳ biến user provider](#adding-custom-user-providers)
    - [User Provider Contract](#the-user-provider-contract)
    - [Authenticatable Contract](#the-authenticatable-contract)
- [Event](#events)

<a name="introduction"></a>
## Giới thiệu

> {tip} **Bạn muốn bắt đầu nhanh?** Chỉ cần chạy `php artisan make:auth` và `php artisan migrate` trong một application Laravel mới. Sau đó, điều hướng trình duyệt của bạn đến `http://your-app.dev/register` hoặc bất kỳ URL nào khác được gán cho application của bạn. Hai lệnh này sẽ đảm nhiệm việc tạo toàn bộ hệ thống authentication của bạn!

Laravel làm cho việc implementing authentication rất đơn giản. Trong thực tế, mặc định hầu hết mọi thứ được cấu hình cho bạn. File cấu hình authentication được đặt tại `config/auth.php`, chứa một số tùy chọn cùng theo document để điều chỉnh hành vi của các service authentication.

Về cốt lõi, các cơ sở authentication của Laravel được tạo thành từ "guards" và "providers". Guards sẽ định nghĩa cách người dùng được authentication cho mỗi request. Ví dụ, Laravel ship có một guard `session` duy trì trạng thái bằng cách sử session storage phiên và các cookie.

Provider sẽ định nghĩa cách người dùng được lấy ra từ storage của bạn. Laravel ships có hỗ trợ lấy ra người dùng bằng Eloquent và database query builder. Tuy nhiên, bạn có thể thoải mái định nghĩa thêm các provider khi cần thiết cho application của bạn.

Đừng lo lắng nếu bây giờ tất cả những điều này nghe có vẻ khó hiểu! Nhiều application sẽ không bao giờ cần sửa file cấu hình authentication mặc định.

<a name="introduction-database-considerations"></a>
### Database Considerations

Mặc định, Laravel có chứa một [Eloquent model](/docs/{{version}}/eloquent)) `App\User` trong thư mục `app` của bạn. Model này có thể được sử dụng mặc định với driver Eloquent authentication. Nếu application của bạn không sử dụng Eloquent, bạn có thể sử dụng driver `database` authentication sử dụng query builder của Laravel.

Khi xây dựng cơ sở dữ liệu cho model `App\User`, bạn hãy đảm bảo cột mật khẩu có độ dài ít nhất 60 ký tự. Với độ dài cột là 255 ký tự sẽ là một lựa chọn tốt.

Ngoài ra, bạn nên định nghĩa thêm bảng `users` (hoặc tương đương) của bạn có chứa cột `remember_token` có thể rỗng, gồm 100 ký tự. Cột này sẽ được sử dụng để lưu trữ token cho người dùng chọn "remember me" khi đăng nhập vào application của bạn.

<a name="authentication-quickstart"></a>
## Authentication Quickstart

Laravel ships có sẵn một số controller authentication được xây dựng từ trước, được đặt trong namespace `App\Http\Controllers\Auth`. `RegisterController` sẽ xử lý đăng ký người dùng mới, `LoginController` sẽ xử lý authentication, `ForgotPasswordController` sẽ xử lý các liên kết email để đăng ký lại mật khẩu và `ResetPasswordController` sẽ chứa logic để reset lại mật khẩu. Mỗi controller này đều sử dụng một trait để thêm các phương thức cần thiết của chúng. Đối với nhiều application, bạn sẽ không cần thiết phải sửa đổi các controller này.

<a name="included-routing"></a>
### Routing

Laravel cung cấp một cách nhanh chóng để hỗ trợ tất cả các route và view mà bạn cần để authentication bằng một lệnh đơn giản:

    php artisan make:auth

Lệnh này nên được sử dụng trên các application mới và sẽ cài đặt một layout view, view đăng ký và đăng nhập, cũng như các route cho tất cả việc authentication. Một `HomeController` cũng sẽ được tạo để xử lý các request đăng nhập để vào dashboard application của bạn.

<a name="included-views"></a>
### Views

Như đã đề cập trong phần trước, lệnh `php artisan make:auth` sẽ tạo ra tất cả các view bạn cần để authentication và đặt chúng trong thư mục `resources/views/auth`.

Lệnh `make:auth` cũng sẽ tạo một thư mục `resources/views/layouts` chứa layout cơ bản cho application của bạn. Tất cả các view này sử dụng framework CSS Bootstrap, nhưng bạn có thể tùy chỉnh chúng theo cách bạn muốn.

<a name="included-authenticating"></a>
### Authenticating

Bây giờ bạn đã có route và view được setup cho các controller authentication đi kèm, bạn đã sẵn sàng để đăng ký và authentication người dùng mới cho application của bạn! Bạn có thể truy cập application của bạn trong trình duyệt vì controller authentication đã chứa logic (thông qua các trait của chúng) để authentication những người dùng đã tồn tại và lưu trữ người dùng mới vào trong cơ sở dữ liệu.

#### Tuỳ chỉnh path

Khi người dùng được authenticate thành công, họ sẽ được redirect đến URI `/home`. Bạn có thể tùy chỉnh vị trí redirect đến sau authenticate bằng cách định nghĩa một thuộc tính `redirectTo` trên `LoginController`, `RegisterController` và `ResetPasswordController`:

    protected $redirectTo = '/';

Nếu redirect path cần một logic tạo được tuỳ chỉnh, bạn có thể định nghĩa phương thức `redirectTo` thay vì thuộc tính `redirectTo`:

    protected function redirectTo()
    {
        return '/path';
    }

> {tip} Phương thức `redirectTo` sẽ được ưu tiên hơn thuộc tính `redirectTo`.

#### Tuỳ chỉnh username

Mặc định, Laravel sử dụng field `email` để authentication. Nếu bạn muốn tùy chỉnh điều này, bạn có thể định nghĩa một phương thức `username` trên `LoginController`:

    public function username()
    {
        return 'username';
    }

#### Tuỳ chỉnh guard

Bạn cũng có thể tùy chỉnh "guard" được sử dụng để authenticate và đăng ký người dùng. Để bắt đầu, hãy định nghĩa một phương thức `guard` trên `LoginController`, `RegisterController` và `ResetPasswordController` của bạn. Phương thức sẽ trả về một instance guard:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Tuỳ chỉnh validation và storage

Để sửa các field của form được require khi người dùng mới đăng ký với application của bạn hoặc để tùy chỉnh cách người dùng mới được lưu trữ vào cơ sở dữ liệu của bạn, bạn có thể sửa đổi class `RegisterController`. Class này chịu trách nhiệm validate và tạo người dùng mới cho application của bạn.

Phương thức `validator` của `RegisterController` chứa các quy tắc validation cho người dùng mới của application. Bạn có thể thoải mái sửa phương thức này theo ý bạn.

Phương thức `create` của `RegisterController` chịu trách nhiệm tạo các record `App\User` mới trong cơ sở dữ liệu của bạn bằng cách sử dụng [Eloquent ORM](/docs/{{version}}/eloquent). Bạn có thể thoái mái sửa phương thức này theo nhu cầu của cơ sở dữ liệu của bạn.
The `create` method of the `RegisterController` is responsible for creating new `App\User` records in your database using the [Eloquent ORM](/docs/{{version}}/eloquent). You are free to modify this method according to the needs of your database.

<a name="retrieving-the-authenticated-user"></a>
### Lấy user đã được authenticate

Bạn có thể truy cập người dùng đã được authenticate thông qua facade `Auth`:

    use Illuminate\Support\Facades\Auth;

    // Get the currently authenticated user...
    $user = Auth::user();

    // Get the currently authenticated user's ID...
    $id = Auth::id();

Ngoài ra, khi người dùng đã được authenticate, bạn có thể truy cập người dùng được authenticate thông qua một instance `Illuminate\Http\Request`. Hãy nhớ rằng, khai báo kiểu class thì sẽ tự động được injecte vào các phương thức controller của bạn:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }

#### Xác định người dùng hiện tại đã được authenticate hay chưa

Để xác định xem người dùng đã đăng nhập vào application của bạn hay chưa, bạn có thể sử dụng phương thức `check` trên facade `Auth`, nó sẽ trả về` true` nếu người dùng đã được authenticate:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} Mặc dù có thể xác định xem người dùng đã được authenticate hay chưa bằng phương thức `check`, thông thường bạn sẽ sử dụng một middleware để xác minh rằng người dùng phải được authenticate trước khi cho phép người dùng truy cập vào các route / controller nhất định. Để tìm hiểu thêm về điều này, hãy xem tài liệu về [protecting routes](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Bảo vệ route

[Route middleware](/docs/{{version}}/middleware) có thể được sử dụng để chỉ cho phép người dùng đã được authenticate truy cập vào một route đã cho.  Laravel ship với middleware `auth`, được định nghĩa tại `Illuminate\Auth\Middleware\Authenticate`. Vì middleware này đã được đăng ký trong HTTP kernel của bạn, tất cả những gì bạn cần làm là gắn middleware này vào một định nghĩa route:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');

Tất nhiên, nếu bạn đang sử dụng [controllers](/docs/{{version}}/controllers), bạn có thể gọi phương thức `middleware` từ hàm khởi tạo của controller thay vì gắn trực tiếp vào định nghĩa route:

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Chỉ định một guard

Khi gắn middleware `auth` vào một route, bạn cũng có thể chỉ định guard nào sẽ được sử dụng để authenticate người dùng. Guard được chỉ định phải tương ứng với một trong các key trong mảng `guards` của file cấu hình `auth.php` của bạn:

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### Login Throttling

Nếu bạn đang sử dụng class `LoginController` tích hợp của Laravel, trait `Illuminate\Foundation\Auth\ThrottlesLogins` sẽ được thêm vào trong controller của bạn. Mặc định, người dùng sẽ không thể đăng nhập trong một phút nếu họ không cung cấp thông tin authenticate chính xác sau vài lần thử. Throttling là unique cho username / địa chỉ e-mail và địa chỉ IP của họ.

<a name="authenticating-users"></a>
## Authenticate user thủ công

Tất nhiên, bạn không bắt buộc phải sử dụng các controller authentication đi kèm với Laravel. Nếu bạn chọn xóa các controller này, bạn sẽ cần quản lý authentication người dùng bằng cách sử dụng trực tiếp các lớp authentication Laravel. Đừng lo lắng, nó rất là dễ dàng!

Chúng ta sẽ truy cập các service authentication của Laravel thông qua [facade](/docs/{{version}}/facades) `Auth`, vì vậy chúng ta cần đảm bảo import facade `Auth` ở đầu class. Tiếp theo, hãy kiểm tra bằng phương thức `attempt`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

Phương thức `attempt` chấp nhận một mảng các cặp key / value làm tham số đầu tiên của nó. Các giá trị trong mảng sẽ được sử dụng để tìm người dùng trong bảng cơ sở dữ liệu của bạn. Vì thế, trong ví dụ trên, người dùng sẽ được lấy ra theo giá trị của cột `email`. Nếu người dùng được tìm thấy, hashed password được lưu trữ trong cơ sở dữ liệu sẽ được so sánh với giá trị `password` đã được pass cho phương thức thông qua mảng. Bạn không cần phải hash mật khẩu mà người dùng đã nhập làm giá trị `password`, vì framework sẽ tự động hash giá trị trước khi so sánh nó với mật khẩu được hash trong cơ sở dữ liệu. Nếu hai mật khẩu hash khớp thì một session authenticate sẽ được bắt đầu cho người dùng này.

Phương thức `attempt` sẽ trả về `true` nếu authentication thành công. Nếu không, `false` sẽ được trả về.

Phương thức `intended` trên redirector sẽ redirect người dùng đến URL mà họ đang truy cập trước khi bị chặn bởi middleware authentication. Một URI dự phòng có thể được cung cấp cho phương thức này trong trường hợp điểm đến không có sẵn.

#### Chỉ định thêm điều kiện bổ sung

Nếu bạn muốn, bạn cũng có thể thêm các điều kiện bổ sung vào authentication query bên cạnh e-mail và mật khẩu của người dùng. Ví dụ: chúng ta có thể xác minh rằng người dùng được đánh dấu là "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} Trong các ví dụ này, `email` không phải là một tùy chọn bắt buộc, nó chỉ được sử dụng làm ví dụ. Bạn nên sử dụng bất kỳ tên cột nào tương ứng với "username" trong cơ sở dữ liệu của bạn.

#### Truy cập vào instance guard cụ thể

ạn có thể chỉ định instance guard mà bạn muốn sử dụng bằng cách sử dụng phương thức `guard` trên facade `Auth`. Điều này cho phép bạn quản lý authentication cho các phần riêng biệt của application bằng các model hoặc bảng user hoàn toàn riêng biệt.
The guard name passed to the `guard` method should correspond to one of the guards configured in your `auth.php` configuration file:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Logging Out

Để đăng xuất người dùng ra khỏi application của bạn, bạn có thể sử dụng phương thức `logout` trên facade `Auth`. Điều này sẽ xóa thông tin authentication trong session của người dùng:

    Auth::logout();

<a name="remembering-users"></a>
### Nhớ tài khoản user

Nếu bạn muốn cung cấp chức năng "remember me" trong application của bạn, bạn có thể pass một giá trị boolean làm tham số thứ hai cho phương thức `attempt`, nó sẽ giữ cho người dùng được authenticate vô thời hạn hoặc cho đến khi họ đăng xuất. Tất nhiên, bảng `users` của bạn phải bao gồm cột `remember_token`, chuỗi này sẽ được sử dụng để lưu trữ mã token "remember me".

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} Nếu bạn đang sử dụng `LoginController` được cung cấp cùng với Laravel, logic của chức năng "remember" người dùng đã được triển khai bởi các trait được sử dụng bởi controller.

Nếu bạn đang "remembering" người dùng, bạn có thể sử dụng phương thức `viaRemember` để xác định xem người dùng có được authenticate bằng cookie "remember me" hay không:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Các phương thức authentication khác

#### Authenticate với một instance user

Nếu bạn cần đăng nhập một instance user hiện có vào application của mình, bạn có thể gọi phương thức `login` với instance user. Đối tượng đã cho phải là một implementation của [contract](/docs/{{version}}/contracts) `Illuminate\Contracts\Auth\Authenticatable`. Tất nhiên, model `App\User` đi kèm với Laravel đã implement interface này:

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

Tất nhiên, bạn có thể chỉ định instance guard bạn muốn sử dụng:

    Auth::guard('admin')->login($user);

#### Authenticate một user qua ID

Để đăng nhập người dùng vào application bằng ID của họ, bạn có thể sử dụng phương thức `loginUsingId`. Phương thức này chấp nhận khóa chính của người dùng mà bạn muốn authenticate:

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### Authenticate A User Once

Bạn có thể sử dụng phương thức `once` để đăng nhập người dùng vào application cho một request. Không có session hoặc cookie nào được sử dụng, điều đó có nghĩa là phương thức này có thể hữu ích khi xây dựng API:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) cung cấp một cách nhanh chóng để authenticate người dùng application của bạn mà không cần thiết lập trang"login" chuyên dụng. Để bắt đầu, hãy gắn [middleware](/docs/{{version}}/middleware) `auth.basic` vào route của bạn. Middleware `auth.basic` đã được thêm cùng framework Laravel, vì vậy bạn không cần định nghĩa nó:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

Khi middleware đã được gắn vào route, bạn sẽ tự động được nhắc về thông tin đăng nhập khi truy cập route trong trình duyệt của bạn. Theo mặc định, middleware `auth.basic` sẽ sử dụng cột `email` trong user record làm "username".

#### A Note On FastCGI

Nếu bạn đang sử dụng PHP FastCGI,  HTTP Basic authentication có thể không hoạt động chính xác. Các dòng lệnh sau nên được thêm vào file `.htaccess` của bạn:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication

Bạn cũng có thể sử dụng HTTP Basic Authentication mà không cần set cookie định danh người dùng trong session, điều này đặc biệt hữu ích cho xác thực API. Để làm như vậy, [định nghĩa một middleware](/docs/{{version}}/middleware) gọi phương thức `onceBasic`. Nếu không có exception nào được tạo bởi phương thức `onceBasic`, request có thể được chuyển tiếp vào application:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            Auth::onceBasic();
            return $next($request);
        }

    }

Tiếp theo, [đăng ký route middleware](/docs/{{version}}/middleware#registering-middleware) và gắn nó vào một route:

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="adding-custom-guards"></a>
## Thêm tuỳ biến guard

Bạn có thể định nghĩa các guard authentication của riêng bạn bằng cách sử dụng phương thức `extend` trên facade `Auth`. Bạn nên gọi tới `extend` trong một [service provider](/docs/{{version}}/providers). Vì Laravel đã ship với một `AuthServiceProvider`, chúng ta có thể đặt code vào trong provider đó:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

Như bạn có thể thấy trong ví dụ trên, hàm callback được pass cho phương thức `extend` sẽ trả về một implementation của `Illuminate\Contracts\Auth\Guard`. Interface này chứa một vài phương thức bạn sẽ cần implement để định nghĩa một guard tùy chỉnh. Khi guard tùy chỉnh của bạn đã được định nghĩa, bạn có thể sử dụng guard này trong cấu hình `guards` của file cấu hình` auth.php` của bạn:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## Thêm tuỳ biến user provider

Nếu bạn không sử dụng cơ sở dữ liệu quan hệ truyền thống để lưu trữ người dùng của bạn, bạn sẽ cần mở rộng Laravel với user provider authentication của riêng bạn. Chúng ta sẽ sử dụng phương thức `provider` trên facade `Auth` để định nghĩa user provider tùy chỉnh:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function ($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

Sau khi bạn đã đăng ký provider bằng phương thức `provider`, bạn có thể chuyển sang user provider mới trong file cấu hình` auth.php` của bạn. Đầu tiên, xác định một `provider` sử dụng driver mới của bạn:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

Cuối cùng, bạn có thể sử dụng provider này trong cấu hình `guards` của bạn:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### User Provider Contract

Việc implementation `Illuminate\Contracts\Auth\UserProvider` chỉ chịu trách nhiệm tìm nạp một implementation `Illuminate\Contracts\Auth\Authenticatable` trong một hệ thống lưu trữ, như MySQL, Riak, vv... Hai interface này cho phép các cơ chế Laravel authentication tiếp tục hoạt động bất kể dữ liệu người dùng được lưu trữ như thế nào hoặc loại class nào được sử dụng để thể hiện nó.

Chúng ta hãy xem contract `Illuminate\Contracts\Auth\UserProvider`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

Hàm `retrieveById` thường nhận một khóa đại diện cho người dùng, chẳng hạn như ID tự động tăng từ cơ sở dữ liệu MySQL. Implementation `Authenticatable` khớp với ID sẽ được lấy và trả về bởi phương thức.

Hàm `retrieveByToken` lấy một người dùng bằng unique `$identifier` của họ và "remember me" `$token`, được lưu trữ trong một field `remember_token`. Giống như với phương thức trước đó, việc implementation `Authenticatable` sẽ được trả về.

Phương thức `updateRememberToken` cập nhật field `remember_token` với `$token` mới của `$user`. Mã token mới có thể là mã token mới, được chỉ định khi thử đăng nhập "remember me" thành công hoặc khi người dùng đăng xuất.

Phương thức `retrieveByCredentials` nhận được mảng thông tin đăng nhập được pass cho phương thức `Auth::attempt` khi đăng nhập vào một application. Phương thức sau đó sẽ "truy vấn" bộ lưu trữ bên dưới cho người dùng khớp với các thông tin đăng nhập đó. Thông thường, phương thức này sẽ chạy một truy vấn với điều kiện "where" trên `$credentials['username']`. Phương thức sau đó sẽ trả về một implementation `Authenticatable`. **Phương thức này không nên thực hiện bất kỳ authentication hoặc validation mật khẩu nào.**

Phương thức `validateCredentials` nên so sánh `$user` đã cho với `$credentials` để authenticate người dùng. Ví dụ, phương thức này có thể nên sử dụng `Hash::check` để so sánh giá trị của `$user->getAuthPassword()` với giá trị của `$credentials['password']`. Phương thức này sẽ trả về `true` hoặc` false` cho biết mật khẩu có hợp lệ hay không.

<a name="the-authenticatable-contract"></a>
### Authenticatable Contract

Bây giờ chúng ta đã khám phá từng phương thức trên `UserProvider`, chúng ta hãy xem contract `Authenticatable`. Hãy nhớ rằng, provider nên trả về các implementation của interface này từ các phương thức `retrieveById` và `retrieveByCredentials`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

Interface này đơn giản. Phương thức `getAuthIdentifierName` sẽ trả về tên của field "primary key" của người dùng và phương thức `getAuthIdentifier` sẽ trả về "primary key" của người dùng. Trong một back-end MySQL, một lần nữa, đây sẽ là khóa chính tăng tự động. `getAuthPassword` sẽ trả về mật khẩu đã được hash của người dùng. Interface này cho phép hệ thống authentication hoạt động với bất kỳ class User nào, bất kể ORM hay layer trừu tượng lưu trữ nào bạn đang sử dụng. Mặc định, Laravel có chứa một class `User` trong thư mục `app` implement interface này, vì vậy bạn có thể tham khảo class này để biết ví dụ implementation.

<a name="events"></a>
## Event

Laravel tăng nhiều [events](/docs/{{version}}/events) trong quá trình authentication. Bạn có thể gắn listener vào các event này trong `EventServiceProvider` của bạn:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Event\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Event\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],

        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];
