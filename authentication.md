# Authentication

- [Giới thiệu](#introduction)
    - [Database Considerations](#introduction-database-considerations)
- [Authentication Quickstart](#authentication-quickstart)
    - [Routing](#included-routing)
    - [Views](#included-views)
    - [Authenticating](#included-authenticating)
    - [Lấy user đã được authenticate](#retrieving-the-authenticated-user)
    - [Bảo vệ route](#protecting-routes)
    - [Password Confirmation](#password-confirmation)
    - [Login Throttling](#login-throttling)
- [Authenticate user thủ công](#authenticating-users)
    - [Nhớ tài khoản user](#remembering-users)
    - [Các phương thức authentication khác](#other-authentication-methods)
- [HTTP Basic Authentication](#http-basic-authentication)
    - [Stateless HTTP Basic Authentication](#stateless-http-basic-authentication)
[Logging Out](#logging-out)
    - [Vô hiệu hoá session trên các thiết bị khác](#invalidating-sessions-on-other-devices)
- [Authentication bằng bên thứ ba](https://github.com/laravel/socialite)
- [Thêm tuỳ biến guard](#adding-custom-guards)
    - [Closure Request Guards](#closure-request-guards)
- [Thêm tuỳ biến user provider](#adding-custom-user-providers)
    - [User Provider Contract](#the-user-provider-contract)
    - [Authenticatable Contract](#the-authenticatable-contract)
- [Event](#events)

<a name="introduction"></a>
## Giới thiệu

> {tip} **Bạn có muốn bắt đầu nhanh không?** Cài đặt package `laravel/ui` (1.0) thông qua Composer và chạy `php artisan ui vue --auth` trong ứng dụng Laravel của bạn. Sau khi migration cơ sở dữ liệu của bạn xong, hãy chuyển hướng url trên trình duyệt của bạn đến `http://your-app.test/register` hoặc bất kỳ URL nào được gán đến cho application của bạn. Lệnh này sẽ đảm nhiệm việc tạo ra toàn bộ hệ thống authentication của bạn!

Laravel làm cho việc thực hiện authentication trở nên rất đơn giản. Trong thực tế, hầu hết mọi thứ đã được cài đặt sẵn cho bạn. File cấu hình authentication sẽ được đặt tại `config/auth.php`, nó sẽ chứa một số tùy chọn cùng theo document để điều chỉnh các hành vi của các service authentication.

Về cốt lõi, các cơ sở authentication của Laravel được tạo từ "guards" và "providers". Guards sẽ định nghĩa cách mà người dùng sẽ được authentication cho mỗi request. Ví dụ, Laravel có một guard `session` để duy trì trạng thái đăng nhập bằng cách sử session storage và cookie.

Provider sẽ định nghĩa cách mà người dùng được lấy ra từ database của bạn. Laravels có hỗ trợ lấy ra người dùng ra bằng Eloquent hoặc query builder. Tuy nhiên, bạn có thể thoải mái định nghĩa thêm các provider khi cần thiết cho application của bạn.

Đừng lo lắng nếu tất cả những điều này nghe có vẻ khó hiểu! Nhiều application sẽ không bao giờ cần phải sửa đến các file cấu hình authentication này.

<a name="introduction-database-considerations"></a>
### Database Considerations

Mặc định, Laravel có chứa một [Eloquent model](/docs/{{version}}/eloquent) `App\User` trong thư mục `app` của bạn. Model này sẽ được sử dụng mặc định với driver Eloquent authentication. Nhưng nếu application của bạn không muốn sử dụng Eloquent, bạn có thể sử dụng driver `database` authentication của Laravel.

Khi xây dựng cơ sở dữ liệu cho model `App\User`, bạn hãy đảm bảo rằng cột mật khẩu có độ dài ít nhất 60 ký tự. Chúng tôi khuyến khích bạn nên đặt độ dài của cột mật khẩu này là 255 ký tự.

Ngoài ra, bạn nên định nghĩa thêm vào bảng `users` (hoặc tương đương) một cột `remember_token` có thể là nullable, gồm 100 ký tự. Cột này sẽ được sử dụng để lưu trữ token của người dùng khi chọn "remember me" lúc đăng nhập vào application của bạn.

<a name="authentication-quickstart"></a>
## Authentication Quickstart

Laravels có sẵn một số controller cho authentication được xây dựng từ trước, được lưu trong namespace `App\Http\Controllers\Auth`. `RegisterController` sẽ xử lý đăng ký người dùng mới, `LoginController` sẽ xử lý authentication, `ForgotPasswordController` sẽ xử lý các liên kết email để đăng ký lại mật khẩu và `ResetPasswordController` sẽ chứa logic để reset lại mật khẩu. Mỗi controller này đều sử dụng một trait để thêm các phương thức cần thiết cho chúng. Đối với nhiều application, bạn sẽ không cần phải sửa các controller này.

<a name="included-routing"></a>
### Routing

Package `laravel/ui` của Laravel cung cấp một cách nhanh chóng để hỗ trợ tất cả các route và view mà bạn cần để authentication bằng một vài câu lệnh đơn giản sau:

    composer require laravel/ui "^1.0" --dev

    php artisan ui vue --auth

Lệnh này nên được sử dụng trên các application mới và sẽ cài đặt một số màn hình như màn hình đăng ký hoặc màn hình đăng nhập, ngoài ra lệnh này cũng cài đặt thêm một số route cho việc authentication. Một `HomeController` cũng sẽ được tạo để xử lý các request đăng nhập để vào màn hình chính của application của bạn.

> {tip} Nếu ứng dụng của bạn không cần chức năng đăng ký, bạn có thể vô hiệu hóa nó bằng cách xóa class `RegisterController` và sửa khai báo route của bạn là: `Auth::routes(['register' => false]);`.

#### Creating Applications Including Authentication

Nếu bạn đang bắt đầu một ứng dụng mới và muốn thêm authentication scaffolding, bạn có thể sử dụng lệnh `--auth` khi tạo ứng dụng của mình. Lệnh này sẽ tạo một ứng dụng mới và tất cả authentication scaffolding đã được biên dịch và cài đặt:

    laravel new blog --auth

<a name="included-views"></a>
### Views

Như đã đề cập trong phần trước, lệnh `php artisan ui vue --auth` của package `laravel/ui` sẽ tạo ra tất cả các view mà bạn cần để authentication và lưu chúng trong thư mục `resources/views/auth`.

Lệnh `ui` cũng sẽ tạo ra một thư mục `resources/views/layouts` chứa layout cơ bản cho application của bạn. Tất cả các view này sử dụng framework CSS Bootstrap, nhưng bạn có thể tùy chỉnh chúng theo cách bạn muốn.

<a name="included-authenticating"></a>
### Authenticating

Hiện tại bạn đã có các route và các view đã được setup sẵn tương ứng với các controller authentication đi kèm, bây giờ bạn đã sẵn sàng để tạo và đăng ký một người dùng mới cho application của bạn! Bạn có thể truy cập vào application của bạn trên trình duyệt vì trong controller authentication của bạn đã chứa code (trong các trait) để authentication những người dùng đã tồn tại và lưu trữ những người dùng mới vào trong cơ sở dữ liệu.

#### Tuỳ chỉnh path

Khi người dùng được authenticate thành công, họ sẽ được chuyển hướng đến URI `/home`. Bạn có thể tùy chỉnh vị trí được chuyển hướng bằng cách sử dụng hằng số `HOME` được định nghĩa trong `RouteServiceProvider` của bạn:

    public const HOME = '/home';

Nếu bạn cần tùy chỉnh mạnh mẽ hơn như response được trả về khi người dùng xác thực thành công, Laravel cung cấp một phương thức trống `authenticated(Request $request, $user)` có thể được ghi đè nếu bạn muốn:

    /**
     * The user has been authenticated.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  mixed  $user
     * @return mixed
     */
    protected function authenticated(Request $request, $user)
    {
        return response([
            //
        ]);
    }

#### Tuỳ chỉnh username

Mặc định, Laravel sử dụng field `email` để authentication. Nếu bạn muốn tùy chỉnh điều này, bạn có thể định nghĩa một phương thức `username` trong `LoginController`:

    public function username()
    {
        return 'username';
    }

#### Tuỳ chỉnh guard

Bạn cũng có thể tùy chỉnh "guard" được sử dụng để authenticate và đăng ký một người dùng mới. Để bắt đầu, hãy định nghĩa một phương thức `guard` trong `LoginController`, `RegisterController` và `ResetPasswordController` của bạn. Phương thức này sẽ trả về một instance guard:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Tuỳ chỉnh validation và storage

Khi một người dùng mới đăng ký vào application của bạn, bạn có thể sửa các field bắt buộc nhập trong form hoặc có thể bạn cũng muốn sửa cách mà data của người dùng mới được lưu trữ vào trong cơ sở dữ liệu, bạn có thể tuỳ chỉnh điều này trong class `RegisterController`. Class này chịu trách nhiệm validate và tạo người dùng mới cho application của bạn.

Phương thức `validator` của `RegisterController` chứa các rule validation cho người dùng mới. Bạn có thể thoải mái sửa phương thức này theo ý bạn muốn.

Phương thức `create` của `RegisterController` chịu trách nhiệm tạo một record `App\User` mới trong cơ sở dữ liệu của bạn bằng cách sử dụng [Eloquent ORM](/docs/{{version}}/eloquent). Bạn có thể thoái mái sửa phương thức này theo nhu cầu cơ sở dữ liệu của bạn.

<a name="retrieving-the-authenticated-user"></a>
### Lấy user đã được authenticate

Bạn có thể truy cập vào người dùng đã được authenticate thông qua facade `Auth`:

    use Illuminate\Support\Facades\Auth;

    // Get the currently authenticated user...
    $user = Auth::user();

    // Get the currently authenticated user's ID...
    $id = Auth::id();

Ngoài ra, khi người dùng đã được authenticate, bạn có thể truy cập vào thông tin của người dùng đó thông qua một instance `Illuminate\Http\Request`. Hãy nhớ rằng, khi bạn khai báo kiểu class `Request` thì sẽ tự động được inject vào trong phương thức controller của bạn:

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

Để xác định xem người dùng đã đăng nhập vào application của bạn hay chưa, bạn có thể sử dụng phương thức `check` trên facade `Auth`, nó sẽ trả về `true` nếu người dùng đã được authenticate:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} Mặc dù bạn có thể xác định xem người dùng đã được authenticate hay chưa bằng phương thức `check`, nhưng thông thường bạn nên sử dụng một middleware để yêu cầu người dùng phải được authenticate trước khi truy cập vào một route hoặc một controller cụ thể. Để tìm hiểu thêm về điều này, hãy xem tài liệu về [protecting routes](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Bảo vệ route

[Route middleware](/docs/{{version}}/middleware) có thể được sử dụng để chỉ cho phép những người dùng đã được authenticate mới có thể truy cập vào một route cụ thể. Laravel có sẵn middleware `auth`, được định nghĩa tại `Illuminate\Auth\Middleware\Authenticate`. Và vì middleware `auth` này đã được đăng ký sẵn trong HTTP kernel của bạn, nên tất cả những gì bạn cần làm là gắn middleware này vào định nghĩa route của bạn:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');

Nếu bạn đang sử dụng [controllers](/docs/{{version}}/controllers), bạn có thể gọi phương thức `middleware` từ hàm khởi tạo của controller thay vì gắn trực tiếp vào định nghĩa route:

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Chuyển hướng người dùng chưa authentication

Khi middleware `auth` phát hiện người dùng chưa được unauthorized, nó sẽ gửi về response JSON `401` hoặc, nếu request không phải là request AJAX, thì nó sẽ chuyển hướng người dùng tới [route mà đã được đặt tên là](/docs/{{version}}/routing#named-routes) `login`. Bạn có thể sửa hành động này bằng cách cập nhật phương thức `redirectTo` trong file `app/Http/Middleware/Authenticate.php` của bạn:

    /**
     * Get the path the user should be redirected to.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return string
     */
    protected function redirectTo($request)
    {
        return route('login');
    }

#### Chỉ định một guard

Khi gắn middleware `auth` vào một route, bạn cũng có thể chỉ định guard nào sẽ được sử dụng để authenticate người dùng. Guard được chỉ định sẽ phải nằm trong các key trong mảng `guards` của file cấu hình `auth.php` của bạn:

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="password-confirmation"></a>
### Password Confirmation

Thỉnh thoảng, bạn có thể yêu cầu người dùng nhập lại mật khẩu của họ trước khi truy cập vào route cụ thể trong ứng dụng của bạn. Ví dụ: bạn có thể yêu cầu điều này trước khi người dùng sửa bất kỳ cài đặt thanh toán nào trong ứng dụng của bạn.

Để thực hiện điều này, Laravel cung cấp một middleware `password.confirm`. Việc gắn middleware `password.confirm` vào một route sẽ chuyển hướng người dùng đến một màn hình nơi mà họ cần nhập lại mật khẩu của họ trước khi có thể tiếp tục:

    Route::get('/settings/security', function () {
        // Users must confirm their password before continuing...
    })->middleware(['auth', 'password.confirm']);

Sau khi người dùng nhập lại thành công mật khẩu của họ, người dùng sẽ được chuyển hướng đến route mà họ đã cố gắng truy cập vào ban đầu. Mặc định, sau khi nhập lại mật khẩu thành công, người dùng sẽ không cần phải nhập lại mật khẩu trong ba giờ đồng hồ. Bạn có thể tự do tùy chỉnh khoảng thời gian trước khi người dùng phải nhập lại mật khẩu của họ bằng cách sử dụng tùy chọn cấu hình `auth.password_timeout`.

<a name="login-throttling"></a>
### Login Throttling

Nếu bạn đang sử dụng class `LoginController` có sẵn của Laravel, thì trait `Illuminate\Foundation\Auth\ThrottlesLogins` sẽ được thêm vào trong controller của bạn. Mặc định, người dùng sẽ không thể đăng nhập trong một phút nếu họ không cung cấp đúng thông tin authenticate sau một vài lần thử. Throttling là một trường duy nhất nó sẽ gắn username hoặc địa chỉ e-mail với địa chỉ IP của họ.

<a name="authenticating-users"></a>
## Authenticate user thủ công

Chú ý là bạn không bị bắt buộc phải sử dụng các controller authentication đi kèm với Laravel. Nếu bạn chọn xóa các controller này, bạn sẽ cần quản lý authentication người dùng bằng cách sử dụng trực tiếp các lớp authentication Laravel. Đừng lo lắng, nó rất là dễ dàng!

Chúng ta sẽ truy cập các service authentication của Laravel thông qua [facade](/docs/{{version}}/facades) `Auth`, vì vậy chúng ta cần đảm bảo là đã import facade `Auth` vào đầu class. Tiếp theo, hãy kiểm tra bằng phương thức `attempt`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @param  \Illuminate\Http\Request $request
         *
         * @return Response
         */
        public function authenticate(Request $request)
        {
            $credentials = $request->only('email', 'password');

            if (Auth::attempt($credentials)) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

Phương thức `attempt` chấp nhận một mảng các cặp key và value làm tham số đầu tiên của nó. Các giá trị trong mảng sẽ được sử dụng để tìm người dùng trong bảng cơ sở dữ liệu của bạn. Vì thế, trong ví dụ trên, người dùng sẽ được lấy ra theo giá trị của cột `email`. Nếu người dùng được tìm thấy, thì trường password đã được mã hoá mà đang được lưu trữ trong cơ sở dữ liệu sẽ được so sánh với giá trị `password` đã được truyền vào. Bạn không cần phải hash mật khẩu mà người dùng đã nhập làm giá trị `password`, vì framework sẽ tự động hash giá trị đó trước khi so sánh với mật khẩu đã được hash trong cơ sở dữ liệu. Nếu hai mật khẩu hash này khớp nhau thì một session authenticate sẽ được khởi tạo cho người dùng này.

Phương thức `attempt` sẽ trả về `true` nếu authentication thành công. Và nếu không thành công thì `false` sẽ được trả về.

Phương thức `intended` trên redirector sẽ chuyển hướng người dùng đến URL mà họ đang truy cập trước khi bị chặn lại bởi middleware authentication. Một URI dự phòng có thể được cung cấp cho phương thức này trong trường hợp điểm đến là không có sẵn.

#### Chỉ định thêm điều kiện bổ sung

Nếu bạn muốn, bạn cũng có thể thêm các điều kiện bổ sung vào authentication query bên cạnh e-mail và mật khẩu của người dùng. Ví dụ: chúng ta có thể kiểm tra rằng người dùng phải là trạng thái "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} Trong các ví dụ này, `email` không phải là một trường bắt buộc, nó chỉ được sử dụng làm ví dụ. Bạn có thể sử dụng bất kỳ tên cột nào tương ứng với "username" trong cơ sở dữ liệu của bạn.

#### Truy cập vào instance guard cụ thể

Bạn có thể chỉ định instance guard mà bạn muốn sử dụng bằng cách sử dụng phương thức `guard` trên facade `Auth`. Điều này cho phép bạn quản lý authentication cho các phần riêng biệt của application bằng các model hoặc bảng user hoàn toàn riêng biệt.

Tên được truyền vào trong phương thức `guard` phải là một trong các tên mà bạn đã khởi tạo trong file cấu hình `auth.php`:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Logging Out

Để đăng xuất người dùng ra khỏi application của bạn, bạn có thể sử dụng phương thức `logout` trên facade `Auth`. Điều này sẽ xóa thông tin authentication trong session của người dùng:

    Auth::logout();

<a name="remembering-users"></a>
### Nhớ tài khoản user

Nếu bạn muốn cung cấp chức năng "remember me" trong application của bạn, bạn có thể truyền một giá trị boolean làm tham số thứ hai cho phương thức `attempt`, nó sẽ giữ cho người dùng được authenticate vô thời hạn hoặc cho đến khi họ đăng xuất. Bảng `users` của bạn phải có cột `remember_token`, chuỗi này sẽ được sử dụng để lưu trữ mã token "remember me".

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} Nếu bạn đang sử dụng `LoginController` mà được cung cấp bởi Laravel, thì logic của chức năng "remember me" này cũng đã được triển khai bởi các trait được sử dụng trong controller.

Nếu bạn đang "remembering" một người dùng, bạn có thể sử dụng phương thức `viaRemember` để xác định xem người dùng đó có được authenticate bằng "remember me" hay không:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Các phương thức authentication khác

#### Authenticate với một instance user

Nếu bạn cần đăng nhập một instance user hiện tại vào application của bạn, thì bạn có thể gọi phương thức `login` với instance user đó. Với điều kiện là đối tượng mà được truyền vào phải là một implementation của [contract](/docs/{{version}}/contracts) `Illuminate\Contracts\Auth\Authenticatable`. Model `App\User` đi kèm với Laravel đã implement interface này:

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

Bạn có thể chỉ định instance guard nào mà bạn muốn sử dụng:

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

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) cung cấp một cách nhanh chóng để authenticate người dùng vào application của bạn mà không cần thiết lập ra một trang "login" chuyên dụng. Để bắt đầu, hãy gắn [middleware](/docs/{{version}}/middleware) `auth.basic` vào route của bạn. Middleware `auth.basic` đã được thêm vào sẵn trong framework Laravel, vì vậy bạn không cần định nghĩa nó:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

Khi middleware đã được gắn vào route, bạn sẽ tự động được nhắc về thông tin đăng nhập khi truy cập route trong trình duyệt của bạn. Mặc định, middleware `auth.basic` sẽ sử dụng cột `email` trong user record làm "username".

#### A Note On FastCGI

Nếu bạn đang sử dụng PHP FastCGI, thì HTTP Basic authentication có thể không hoạt động chính xác. Các dòng lệnh sau nên được thêm vào file `.htaccess` của bạn:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication

Bạn cũng có thể sử dụng HTTP Basic Authentication mà không cần phải set cookie để định danh người dùng trong session, điều này đặc biệt hữu ích cho việc xác thực API. Để làm như vậy, hãy [định nghĩa một middleware](/docs/{{version}}/middleware) gọi phương thức `onceBasic`. Nếu không có response nào được trả về bởi phương thức `onceBasic`, request sẽ được chuyển tiếp vào application:

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
            return Auth::onceBasic() ?: $next($request);
        }

    }

Tiếp theo, [đăng ký route middleware](/docs/{{version}}/middleware#registering-middleware) đó và gắn nó vào một route:

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="logging-out"></a>
## Logging Out

Để đăng xuất người dùng ra khỏi application của bạn, bạn có thể sử dụng phương thức `logout` trên facade `Auth`. Phương thức này sẽ xóa đi các thông tin xác thực có trong session của người dùng hiện tại:

    use Illuminate\Support\Facades\Auth;

    Auth::logout();

<a name="invalidating-sessions-on-other-devices"></a>
### Vô hiệu hoá session trên các thiết bị khác

Laravel cũng cung cấp các cơ chế để vô hiệu hoá session và "đăng xuất" người dùng ra khỏi các thiết bị khác của họ mà không vô hiệu hoá session hiện tại của họ. Tính năng này thường được sử dụng khi người dùng đang thay đổi hoặc cập nhật lại mật khẩu của họ và bạn muốn làm mất hiệu lực các session trên các thiết bị khác trong khi vẫn giữ xác thực trên thiết bị hiện tại.

Trước khi bắt đầu, bạn nên đảm bảo là middleware `Illuminate\Session\Middleware\AuthenticateSession` đã được bỏ comment trong group middleware `web` trong class `app/Http/Kernel.php` của bạn:

    'web' => [
        // ...
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        // ...
    ],

Sau đó, bạn có thể sử dụng phương thức `logoutOtherDevices` trên facade `Auth`. Phương thức này sẽ yêu cầu người dùng cung cấp mật khẩu hiện tại của họ:

    use Illuminate\Support\Facades\Auth;

    Auth::logoutOtherDevices($password);

Khi phương thức `logoutOtherDevices` được gọi, thì các session khác của người dùng sẽ bị vô hiệu hoàn toàn, có nghĩa là họ sẽ bị "đăng xuất" ra khỏi tất cả các guard mà họ đã được đăng nhập trước đó.

> {note} Khi sử dụng middleware `AuthenticateSession` kết hợp với một tên route tùy chỉnh cho route `login`, bạn phải ghi đè phương thức `unauthenticated` trên exception handler của ứng dụng để chuyển hướng chính xác người dùng đến trang đăng nhập của bạn.

<a name="adding-custom-guards"></a>
## Thêm tuỳ biến guard

Bạn có thể định nghĩa các guard authentication của riêng bạn bằng cách sử dụng phương thức `extend` trên facade `Auth`. Bạn nên gọi tới phương thức `extend` trong một [service provider](/docs/{{version}}/providers). Vì Laravel đã có sẵn một `AuthServiceProvider`, nên chúng ta có thể đặt code đó vào trong provider này:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Auth;

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

Như bạn có thể thấy trong ví dụ trên, hàm callback được truyền vào trong phương thức `extend` sẽ trả về một implementation của class `Illuminate\Contracts\Auth\Guard`. Interface này sẽ chứa một vài phương thức mà bạn sẽ cần phải implement để định nghĩa một guard tùy chỉnh. Khi guard tùy chỉnh của bạn đã được định nghĩa, bạn có thể sử dụng guard này trong cấu hình `guards` của file cấu hình `auth.php` của bạn:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="closure-request-guards"></a>
### Closure Request Guards

Cách đơn giản nhất để làm một hệ thống xác thực tùy biến dựa trên HTTP request là sử dụng phương thức `Auth::viaRequest`. Phương thức này sẽ cho phép bạn nhanh chóng định nghĩa quy trình xác thực của bạn bằng một Closure duy nhất.

Để bắt đầu, hãy gọi phương thức `Auth::viaRequest` trong hàm `boot` của class `AuthServiceProvider`. Phương thức `viaRequest` sẽ chấp nhận tên của authentication driver làm tham số đầu tiên của nó. Tên này có thể là bất kỳ chuỗi nào mà mô tả guard tùy biến của bạn. Tham số thứ hai được truyền cho phương thức sẽ là một Closure nhận vào một HTTP request và sẽ trả về một instance người dùng hoặc nếu xác thực không thành công, thì sẽ là `null`:

    use App\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::viaRequest('custom-token', function ($request) {
            return User::where('token', $request->token)->first();
        });
    }

Sau khi authentication driver tùy biến của bạn đã được định nghĩa, bạn có thể sử dụng nó như là một driver trong cấu hình `guards` trong file cấu hình `auth.php` của bạn:

    'guards' => [
        'api' => [
            'driver' => 'custom-token',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## Thêm tuỳ biến user provider

Nếu bạn không sử dụng cơ sở dữ liệu quan hệ để lưu trữ thông tin người dùng của bạn, bạn sẽ cần mở rộng Laravel với một user provider authentication của riêng bạn. Chúng ta sẽ sử dụng phương thức `provider` trên facade `Auth` để định nghĩa user provider tùy chỉnh mới này:

    <?php

    namespace App\Providers;

    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Auth;

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

Sau khi bạn đã đăng ký provider bằng phương thức `provider`, bạn có thể chuyển user provider mới này vào trong file cấu hình `auth.php` của bạn. Đầu tiên, định nghĩa một `provider` mà sử dụng driver mới của bạn:

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

Các implementation từ `Illuminate\Contracts\Auth\UserProvider` sẽ chỉ chịu trách nhiệm lấy ra đối tượng đã được implementation từ `Illuminate\Contracts\Auth\Authenticatable` từ cơ sở dữ liệu, như MySQL, Riak, vv... Hai interface này cho phép các cơ chế Laravel authentication tiếp tục hoạt động bất kể dữ liệu người dùng đang được lưu trữ như thế nào hoặc bất kể loại class nào thể hiện chúng.

Chúng ta hãy xem contract `Illuminate\Contracts\Auth\UserProvider`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider
    {
        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);
    }

Hàm `retrieveById` sẽ nhận một khóa đại diện cho người dùng, chẳng hạn như ID tự động tăng trong cơ sở dữ liệu MySQL. Implementation `Authenticatable` sẽ dựa vào ID để lấy ra và trả về thông qua phương thức của nó.

Hàm `retrieveByToken` lấy ra một người dùng bằng unique `$identifier` của nó và "remember me" `$token`, được lưu trữ trong field `remember_token`. Giống như phương thức trước đó, implementation `Authenticatable` sẽ thực hiện việc trả về.

Phương thức `updateRememberToken` sẽ cập nhật field `remember_token` với `$token` mới của `$user`. Một mã token mới sẽ được chỉ định khi đăng nhập "remember me" thành công hoặc khi người dùng đăng xuất.

Phương thức `retrieveByCredentials` sẽ nhận một mảng thông tin đăng nhập được truyền vào phương thức `Auth::attempt` khi đăng nhập vào application. Phương thức này sẽ "truy vấn" bộ lưu trữ bên dưới để lấy ra người dùng khớp với các thông tin đăng nhập. Thông thường, phương thức này sẽ chạy một truy vấn với điều kiện "where" trên `$credentials['username']`. Phương thức này sẽ trả về một đối tượng đã được implementation `Authenticatable`. **Phương thức này không nên thực hiện bất kỳ hành động authentication hoặc validation mật khẩu nào.**

Phương thức `validateCredentials` sẽ so sánh `$user` đã nhận với `$credentials` của người dùng để authenticate. Ví dụ, phương thức này có thể sử dụng hàm `Hash::check` để so sánh giá trị của `$user->getAuthPassword()` với giá trị của `$credentials['password']`. Phương thức này sẽ trả về giá trị `true` hoặc `false` cho biết mật khẩu có hợp lệ hay không.

<a name="the-authenticatable-contract"></a>
### Authenticatable Contract

Sau khi chúng ta đã khám phá các phương thức trên `UserProvider`, bây giờ chúng ta hãy xem contract `Authenticatable`. Hãy nhớ rằng, provider nên trả về các đối tượng mà đã implementation của interface này từ các phương thức `retrieveById`, `retrieveByToken` và `retrieveByCredentials`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable
    {
        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();
    }

Interface này rất đơn giản. Phương thức `getAuthIdentifierName` sẽ trả về tên của field "primary key" và phương thức `getAuthIdentifier` sẽ trả về giá trị của field đó. Trong MySQL thì field đó sẽ là field khóa chính tự động tăng. `getAuthPassword` sẽ trả về mật khẩu mà đã được hash của người dùng. Interface này cho phép hệ thống authentication hoạt động với bất kỳ class User nào, bất kể nó là ORM hay layer lưu trữ trừu tượng nào mà bạn đang sử dụng. Mặc định, Laravel đã chứa một class `User` trong thư mục `app` và đã implement sẵn interface này, vì vậy bạn có thể tham khảo class này để biết thêm về các implementation này.

<a name="events"></a>
## Event

Laravel đưa ra nhiều [events](/docs/{{version}}/events) khác nhau trong quá trình authentication. Bạn có thể gắn listener vào các event này trong `EventServiceProvider` của bạn:

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
