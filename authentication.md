# Authentication

- [Giới thiệu](#introduction)
    - [Starter Kits](#starter-kits)
    - [Database Considerations](#introduction-database-considerations)
    - [Tổng quan hệ sinh thái](#ecosystem-overview)
- [Authentication Quickstart](#authentication-quickstart)
    - [Cài đặt một Starter Kit](#install-a-starter-kit)
    - [Lấy user đã được authenticate](#retrieving-the-authenticated-user)
    - [Bảo vệ route](#protecting-routes)
    - [Login Throttling](#login-throttling)
- [Authenticate user thủ công](#authenticating-users)
    - [Nhớ tài khoản user](#remembering-users)
    - [Các phương thức authentication khác](#other-authentication-methods)
- [HTTP Basic Authentication](#http-basic-authentication)
    - [Stateless HTTP Basic Authentication](#stateless-http-basic-authentication)
- [Logging Out](#logging-out)
    - [Vô hiệu hoá session trên các thiết bị khác](#invalidating-sessions-on-other-devices)
- [Xác nhận password](#password-confirmation)
    - [Cấu hình](#password-confirmation-configuration)
    - [Routing](#password-confirmation-routing)
    - [Bảo vệ route](#password-confirmation-protecting-routes)
- [Thêm tuỳ biến guard](#adding-custom-guards)
    - [Closure Request Guards](#closure-request-guards)
- [Thêm tuỳ biến user provider](#adding-custom-user-providers)
    - [User Provider Contract](#the-user-provider-contract)
    - [Authenticatable Contract](#the-authenticatable-contract)
- [Social Authentication](/docs/{{version}}/socialite)
- [Event](#events)

<a name="introduction"></a>
## Giới thiệu

Nhiều ứng dụng web cung cấp nhiều cách để người dùng của họ xác thực với ứng dụng bên thứ ba và "đăng nhập" vào ứng dụng. Việc triển khai tính năng này vào trong các ứng dụng web có thể là một điều phức tạp và tiềm ẩn nhiều rủi ro. Vì lý do này, Laravel đã cung cấp cho bạn một công cụ cần thiết để triển khai xác thực một cách nhanh chóng, an toàn và dễ dàng.

Về cốt lõi, các cơ sở authentication của Laravel được tạo từ "guards" và "providers". Guards sẽ định nghĩa cách mà người dùng sẽ được authentication cho mỗi request. Ví dụ, Laravel có một guard `session` để duy trì trạng thái đăng nhập bằng cách sử session storage và cookie.

Provider sẽ định nghĩa cách mà người dùng được lấy ra từ database của bạn. Laravel có hỗ trợ lấy ra người dùng ra bằng [Eloquent](/docs/{{version}}/eloquent) hoặc query builder. Tuy nhiên, bạn có thể thoải mái định nghĩa thêm các provider mà cần thiết cho application của bạn.

File cấu hình xác thực ứng dụng của bạn được lưu tại `config/auth.php`. File này sẽ chứa một số tùy chọn đã được giải thích trong file để điều chỉnh các hành vi của các service xác thực của Laravel.

> [!NOTE]
> Guard và provider không nên bị nhầm lẫn với các "role" và các "permission". Để tìm hiểu thêm về các cách authorize hành động của người dùng thông qua permission, vui lòng tham khảo thêm tài liệu [authorization](/docs/{{version}}/authorization).

<a name="starter-kits"></a>
### Starter Kits

Bạn muốn bắt đầu nhanh không? Cài đặt [laravel application starter kit](/docs/{{version}}/starter-kits) trong một ứng dụng Laravel mới. Sau khi migrate hết cơ sở dữ liệu của bạn, hãy điều hướng trình duyệt của bạn đến url `/register` hoặc bất kỳ URL nào mà được gán cho ứng dụng của bạn. Starter kit sẽ đảm nhiệm việc hỗ trợ hoàn toàn bộ hệ thống xác thực cho bạn!

**Ngay cả khi bạn chọn không sử dụng starter kit trong ứng dụng Laravel của bạn, việc cài đặt starter kit [laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze) có thể là một cơ hội tuyệt vời để bạn tìm hiểu về cách triển khai tất cả chức năng xác thực của Laravel trong một dự án Laravel mới.** Vì Laravel Breeze sẽ tạo các controller, route và view xác thực cho bạn, bạn có thể kiểm tra code trong các file này và tìm hiểu về cách triển khai các tính năng xác thực của Laravel.

<a name="introduction-database-considerations"></a>
### Database Considerations

Mặc định, Laravel có chứa một [Eloquent model](/docs/{{version}}/eloquent) `App\Models\User` trong thư mục `app/Models` của bạn. Model này sẽ được sử dụng mặc định với driver Eloquent authentication. Nhưng nếu application của bạn không muốn sử dụng Eloquent, bạn có thể sử dụng provider `database` authentication của Laravel.

Khi xây dựng cơ sở dữ liệu cho model `App\Models\User`, bạn hãy đảm bảo rằng cột mật khẩu có độ dài ít nhất là 60 ký tự. Dĩ nhiên, migration bảng `users` có trong một ứng dụng Laravel mới đã tạo cột mật khẩu đó vượt qua độ dài này.

Ngoài ra, bạn nên định nghĩa thêm vào bảng `users` (hoặc tương đương với bảng này) một cột là `remember_token` có độ đài là 100 ký tự và có thể nullable. Cột này sẽ được sử dụng để lưu trữ token của người dùng khi chọn chức năng "remember me" lúc đăng nhập vào application của bạn. Một lần nữa, migration bảng `users` có trong một ứng dụng Laravel mới đã chứa cột này.

<a name="ecosystem-overview"></a>
### Tổng quan hệ sinh thái

Laravel cung cấp một số package liên quan đến việc xác thực. Trước khi tiếp tục, chúng ta sẽ xem xét hệ sinh thái xác thực trong Laravel và thảo luận về mục đích của từng package đó.

Đầu tiên, hãy xem xét cách hoạt động của xác thực. Khi sử dụng trình duyệt web, người dùng sẽ cung cấp username và password của họ thông qua một form đăng nhập. Nếu những thông tin đăng nhập này là chính xác, thì ứng dụng sẽ lưu thông tin của người dùng đã được xác thực vào trong [session](/docs/{{version}}/session) của người dùng. Một cookie chứa session ID sẽ được cung cấp cho trình duyệt để sau đó các request tiếp theo đến ứng dụng có thể kết nối đến thông tin người dùng thông qua session ID. Sau khi nhận được cookie session, ứng dụng sẽ lấy dữ liệu session dựa trên ID session, lưu ý rằng thông tin xác thực mà được lưu trong session sẽ được coi là người dùng "đã xác thực".

Khi một remote service cần xác thực để truy cập API, thì cookie thường sẽ không được sử dụng để xác thực vì không có trình duyệt web nào để lưu cookie đó. Thay vào đó, remote service sẽ gửi token API tới API. Ứng dụng có thể xác thực token này dựa vào một bảng gồm các token API hợp lệ và "xác thực" request cho người dùng mà được kết nối với token API đó thực hiện.

<a name="laravels-built-in-browser-authentication-services"></a>
#### Laravel's Built-in Browser Authentication Services

Laravel có sẵn các authentication mặc định và các service session có thể được truy cập thông qua bằng các facade `Auth` và `Session`. Các chức năng này cung cấp một cách xác thực dựa trên cookie cho các request được bắt đầu từ trên trình duyệt web. Nó cung cấp các phương thức cho phép bạn xác minh thông tin đăng nhập của người dùng và xác thực người dùng. Ngoài ra, các service này sẽ tự động lưu trữ dữ liệu xác thực vào trong session của người dùng và phát hành các cookie session cho người dùng. Về cách sử dụng các service trên sẽ có trong tài liệu dưới.

**Application Starter Kits**

Như đã thảo luận ở trên, bạn có thể tương tác với các service xác thực theo cách thủ công để xây dựng lớp xác thực riêng cho ứng dụng của bạn. Tuy nhiên, để giúp bạn bắt đầu nhanh hơn, chúng tôi đã phát triển một [package miễn phí](/docs/{{version}}/starter-kits) cung cấp một nền tảng hiện đại, mạnh mẽ cho toàn bộ lớp xác thực. Các package này là [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze), [Laravel Jetstream](/docs/{{version}}/starter-kits#laravel-jetstream) và [Laravel Fortify](/docs/{{version}}/fortify).

_Laravel Breeze_ là một triển khai đơn giản, tối thiểu dành cho tất cả các tính năng xác thực của Laravel, bao gồm đăng nhập, đăng ký, đặt lại mật khẩu, xác minh email và xác nhận mật khẩu. Lớp view của Laravel Breeze sẽ chứa một [Blade templates](/docs/{{version}}/blade) đơn giản được tạo bởi [Tailwind CSS](https://tailwindcss.com). Để bắt đầu, hãy xem tài liệu về [application starter kits](/docs/{{version}}/starter-kits) của Laravel.

_Laravel Fortify_ là phần backend xác thực không chứa giao diện cho Laravel, triển khai nhiều tính năng có trong tài liệu này, bao gồm xác thực dựa trên cookie cũng như các tính năng khác như xác thực hai lớp và xác minh email. Fortify cung cấp backend xác thực cho Laravel Jetstream hoặc có thể được sử dụng độc lập khi kết hợp với [Laravel Sanctum](/docs/{{version}}/sanctum) để cung cấp xác thực cho một SPA (Single Page Application) mà cần chức năng xác thực cùng với Laravel.

_[Laravel Jetstream](https://jetstream.laravel.com)_ là một bộ công cụ khởi động ứng dụng mạnh mẽ sử dụng các service xác thực của Laravel Fortify với giao diện người dùng hiện đại, đẹp mắt được hỗ trợ bởi [Tailwind CSS](https://tailwindcss.com), [Livewire](https://livewire.laravel.com) và [Inertia](https://inertiajs.com). Laravel Jetstream hỗ trợ tùy chọn cho xác thực hai lớp, hỗ trợ nhóm, quản lý session trình duyệt, quản lý hồ sơ và tích hợp sẵn [Laravel Sanctum](/docs/{{version}}/sanctum) để cung cấp xác thực token API. Các service xác thực API của Laravel sẽ được thảo luận ở bên dưới.

<a name="laravels-api-authentication-services"></a>
#### Laravel's API Authentication Services

Laravel cung cấp hai package tùy chọn để hỗ trợ bạn quản lý token API và xác thực các request được thực hiện bằng token API: [Passport](/docs/{{version}}/passport) và [Sanctum](/docs/{{version}}/sanctum). Xin lưu ý rằng các thư viện này và thư viện xác thực dựa trên cookie được tích hợp sẵn của Laravel không loại trừ nhau. Các thư viện này chủ yếu tập trung vào xác thực token API trong khi các service xác thực tích hợp sẵn tập trung vào xác thực trên trình duyệt dựa vào cookie. Nhiều ứng dụng sẽ sử dụng cả hai service xác thực dựa trên cookie được tích hợp sẵn của Laravel và một trong các package xác thực API của Laravel ở trên.

**Passport**

Passport là provider xác thực OAuth2, cung cấp nhiều "loại grant" OAuth2 cho phép bạn phát hành nhiều loại token khác nhau. Nói chung, đây là package mạnh mẽ và phức tạp để xác thực API. Tuy nhiên, hầu hết các ứng dụng không yêu cầu các tính năng phức tạp do OAuth2 cung cấp, điều này có thể gây nhầm lẫn cho cả người dùng và nhà phát triển. Ngoài ra, trước đây các nhà phát triển đã nhầm lẫn về cách xác thực của ứng dụng SPA (Single Page Application) hoặc ứng dụng di động bằng cách sử dụng provider xác thực OAuth2 như Passport.

**Sanctum**

Để đối phó lại với sự phức tạp của OAuth2 và sự nhầm lẫn của nhà phát triển, chúng tôi đã bắt đầu xây dựng một package xác thực đơn giản hơn, hợp lý hơn, có thể xử lý cả request web từ trình duyệt web và request API thông qua token. Mục tiêu này đã được thực hiện với sự phát hành của [Laravel Sanctum](/docs/{{version}}/sanctum), nên được coi là một package xác thực được khuyên dùng và ưu tiên cho các ứng dụng sẽ cung cấp giao diện người dùng web ngoài một API hoặc sẽ được cung cấp bởi một ứng dụng Single Page (SPA) tồn tại tách biệt với ứng dụng Laravel backend hoặc các ứng dụng cung cấp phần mobile cho client.

Laravel Sanctum là package xác thực cho cả web và API, nó có thể quản lý toàn bộ quy trình xác thực cho ứng dụng của bạn. Điều này là có thể là bởi vì khi các ứng dụng được dựa vào Sanctum khi nhận được request, trước tiên, Sanctum sẽ xác định xem request đó có chứa session cookie mà đã được xác thực hay không. Sanctum thực hiện điều này bằng cách gọi các service xác thực được tích hợp sẵn của Laravel mà chúng ta đã thảo luận trước đó. Nếu request không được xác thực thông qua session cookie, Sanctum sẽ kiểm tra request về token API. Nếu có token API, Sanctum sẽ xác thực request bằng token đó. Để tìm hiểu thêm về quy trình này, vui lòng tham khảo tài liệu ["cách hoạt động"](/docs/{{version}}/sanctum#how-it-works) của Sanctum.

Laravel Sanctum là một package API mà chúng tôi đã chọn để đưa vào bộ khởi động ứng dụng [Laravel Jetstream](https://jetstream.laravel.com) vì chúng tôi tin rằng package này phù hợp nhất với phần lớn nhu cầu xác thực của ứng dụng web.

<a name="summary-choosing-your-stack"></a>
#### Summary và Choosing Your Stack

Tóm lại, nếu ứng dụng của bạn mà được truy cập thông qua trình duyệt và bạn đang xây dựng một ứng dụng Laravel monolithic, thì ứng dụng của bạn sẽ sử dụng các dịch vụ xác thực được tích hợp sẵn của Laravel.

Tiếp theo, nếu ứng dụng của bạn cung cấp các API mà sẽ được bên thứ ba sử dụng, bạn sẽ chọn lựa giữa [Passport](/docs/{{version}}/passport) hoặc [Sanctum](/docs/{{version}}/sanctum) để cung cấp xác thực token API cho ứng dụng của bạn. Nói chung, Sanctum nên được ưu tiên khi có thể vì đây là giải pháp đơn giản, hoàn chỉnh để xác thực API, xác thực SPA và xác thực di động, có hỗ trợ cho "scope" hoặc "ability".

Nếu bạn đang xây dựng một ứng dụng single page (SPA) được hỗ trợ bởi phần backend của Laravel, thì bạn nên sử dụng [Laravel Sanctum](/docs/{{version}}/sanctum). Khi sử dụng Sanctum, bạn sẽ cần [triển khai thủ công các route xác thực backend của riêng bạn](#authenticating-users) hoặc sử dụng [Laravel Fortify](/docs/{{version}}/fortify) (là một service backend xác thực không UI) cung cấp route và controller cho các chức năng như đăng ký, đặt lại mật khẩu, xác minh email, v.v.

Passport có thể được lựa chọn khi ứng dụng của bạn thực sự cần tất cả các tính năng do OAuth2 cung cấp.

Và, nếu bạn muốn bắt đầu nhanh, chúng tôi rất vui lòng giới thiệu [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze) như một cách nhanh chóng để bắt đầu một ứng dụng Laravel mới, nó sử dụng một loạt các xác thực ưa thích của chúng tôi trong các service Laravel xác thực được tích hợp sẵn và Laravel Sanctum.

<a name="authentication-quickstart"></a>
## Authentication Quickstart

> [!WARNING]
> Phần tài liệu này sẽ thảo luận về việc xác thực người dùng thông qua [bộ công cụ khởi động ứng dụng của Laravel](/docs/{{version}}/starter-kits), bao gồm cả giao diện người dùng để giúp bạn bắt đầu nhanh. Nếu bạn muốn tích hợp trực tiếp với các hệ thống xác thực có sẵn của Laravel, hãy xem tài liệu về [xác thực người dùng theo cách thủ công](#authenticating-users).

<a name="install-a-starter-kit"></a>
### Cài đặt một Starter Kit

Trước tiên, bạn nên [cài đặt bộ khởi động ứng dụng Laravel](/docs/{{version}}/starter-kits). Bộ công cụ khởi động hiện tại của chúng tôi gồm có Laravel Breeze và Laravel Jetstream sẽ cung cấp các điểm khởi đầu được tốt, đẹp mắt để kết hợp phần xác thực này vào trong ứng dụng Laravel mới của bạn.

Laravel Breeze là một triển khai tối thiểu, đơn giản cho tất cả các tính năng xác thực của Laravel, bao gồm đăng nhập, đăng ký, đặt lại mật khẩu, xác minh email và xác nhận mật khẩu. Lớp view của Laravel Breeze được tạo từ [Blade templates](/docs/{{version}}/blade) và bằng [Tailwind CSS](https://tailwindcss.com). Ngoài ra, Breeze cũng cung cấp các tùy chọn dựa trên [Livewire](https://livewire.laravel.com) hoặc [Inertia](https://inertiajs.com) với lựa chọn sử dụng Vue hoặc React.

[Laravel Jetstream](https://jetstream.laravel.com) là bộ công cụ khởi động ứng dụng mạnh mẽ, bao gồm hỗ trợ tạo nền tảng cho ứng dụng của bạn với [Livewire](https://livewire.laravel.com) hoặc [Inertia và Vue](https://inertiajs.com). Ngoài ra, Jetstream cũng có tính năng hỗ trợ tùy chọn cho xác thực hai lớp, nhóm, quản lý hồ sơ, quản lý sesion trình duyệt, hỗ trợ API qua [Laravel Sanctum](/docs/{{version}}/sanctum), xóa tài khoản, v.v.

<a name="retrieving-the-authenticated-user"></a>
### Lấy user đã được authenticate

Sau khi cài đặt bộ khởi động xác thực và cho phép người dùng đăng ký và xác thực với ứng dụng của bạn, bạn sẽ thường xuyên phải tương tác với những người dùng đã được xác thực. Khi xử lý một request đến, bạn có thể truy cập vào người dùng đã được authenticate thông qua facade `Auth`:

    use Illuminate\Support\Facades\Auth;

    // Retrieve the currently authenticated user...
    $user = Auth::user();

    // Retrieve the currently authenticated user's ID...
    $id = Auth::id();

Ngoài ra, khi người dùng đã được authenticate, bạn có thể truy cập vào thông tin của người dùng đó thông qua một instance `Illuminate\Http\Request`. Hãy nhớ rằng, khi bạn khai báo kiểu class `Request` thì sẽ tự động được inject vào trong phương thức controller của bạn. Bằng cách khai báo đối tượng `Illuminate\Http\Request`, bạn có thể có quyền truy cập vào người dùng đã được xác thực từ bất kỳ phương thức controller nào trong ứng dụng của bạn thông qua phương thức `user` của request:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class FlightController extends Controller
    {
        /**
         * Update the flight information for an existing flight.
         */
        public function update(Request $request): RedirectResponse
        {
            $user = $request->user();

            // ...

            return redirect('/flights');
        }
    }

<a name="determining-if-the-current-user-is-authenticated"></a>
#### Xác định người dùng hiện tại đã được authenticate hay chưa

Để xác định xem người dùng thực hiện request HTTP đã được xác thực hay chưa, bạn có thể sử dụng phương pháp `check` trên facade `Auth`. Phương thức này sẽ trả về `true` nếu người dùng đã được xác thực:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> [!NOTE]
> Mặc dù bạn có thể xác định xem người dùng đã được authenticate hay chưa bằng phương thức `check`, nhưng thông thường bạn nên sử dụng một middleware để yêu cầu người dùng phải được authenticate trước khi truy cập vào một route hoặc một controller cụ thể. Để tìm hiểu thêm về điều này, hãy xem tài liệu về [protecting routes](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Bảo vệ route

[Route middleware](/docs/{{version}}/middleware) có thể được sử dụng để chỉ cho phép những người dùng đã được authenticate mới có thể truy cập vào một route cụ thể. Laravel có sẵn middleware `auth`, tham chiếu tới class `Illuminate\Auth\Middleware\Authenticate`. Và vì middleware `auth` này đã được đăng ký sẵn trong HTTP kernel của ứng dụng của bạn, nên tất cả những gì bạn cần làm là gắn middleware này vào định nghĩa route của bạn:

    Route::get('/flights', function () {
        // Only authenticated users may access this route...
    })->middleware('auth');

<a name="redirecting-unauthenticated-users"></a>
#### Chuyển hướng người dùng chưa authentication

Khi middleware `auth` phát hiện người dùng chưa được xác thực, nó sẽ gửi về response JSON `401` hoặc, nếu request không phải là request AJAX, thì nó sẽ chuyển hướng người dùng tới [route mà đã được đặt tên là](/docs/{{version}}/routing#named-routes) `login`. Bạn có thể sửa hành động này bằng cách cập nhật phương thức `redirectTo` trong file `app/Http/Middleware/Authenticate.php` của ứng dụng của bạn:

    use Illuminate\Http\Request;

    /**
     * Get the path the user should be redirected to.
     */
    protected function redirectTo(Request $request): string
    {
        return route('login');
    }

<a name="specifying-a-guard"></a>
#### Chỉ định một guard

Khi gắn middleware `auth` vào một route, bạn cũng có thể chỉ định "guard" nào sẽ được sử dụng để authenticate người dùng. Guard được chỉ định sẽ phải nằm trong các key trong mảng `guards` của file cấu hình `auth.php` của bạn:

    Route::get('/flights', function () {
        // Only authenticated users may access this route...
    })->middleware('auth:admin');

<a name="login-throttling"></a>
### Login Throttling

Nếu bạn đang sử dụng Laravel Breeze hoặc Laravel Jetstream [starter kits](/docs/{{version}}/starter-kits), giới hạn tốc độ sẽ tự động được áp dụng cho các lần thử đăng nhập. Mặc định, người dùng sẽ không thể đăng nhập trong một phút nếu họ không cung cấp đúng thông tin authenticate sau một vài lần thử. Throttling là một trường duy nhất nó sẽ gắn username hoặc địa chỉ email với địa chỉ IP của họ.

> [!NOTE]
> Nếu bạn muốn giới hạn các route khác trong ứng dụng của bạn, hãy xem [tài liệu về giới hạn đó](/docs/{{version}}/routing#rate-limiting).

<a name="authenticating-users"></a>
## Authenticate user thủ công

Bạn không bắt buộc phải sử dụng scaffolding xác thực đi kèm với [bộ công cụ khởi động ứng dụng](/docs/{{version}}/starter-kits) của Laravel. Nếu bạn chọn không sử dụng scaffolding này, bạn sẽ cần phải quản lý authentication người dùng bằng cách sử dụng trực tiếp các lớp authentication Laravel. Đừng lo lắng, nó rất là dễ dàng!

Chúng ta sẽ truy cập các service authentication của Laravel thông qua [facade](/docs/{{version}}/facades) `Auth`, vì vậy chúng ta cần đảm bảo là đã import facade `Auth` vào đầu class. Tiếp theo, hãy kiểm tra bằng phương thức `attempt`. The `attempt` method is normally used to handle authentication attempts from your application's "login" form. Phương thức `attempt` thường được sử dụng để xử lý các lần xác thực từ form "login" của ứng dụng của bạn. Nếu xác thực thành công, bạn nên tạo lại [session](/docs/{{version}}/session) của người dùng để ngăn ngừa việc [sửa session](https://en.wikipedia.org/wiki/Session_fixation):

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         */
        public function authenticate(Request $request): RedirectResponse
        {
            $credentials = $request->validate([
                'email' => ['required', 'email'],
                'password' => ['required'],
            ]);

            if (Auth::attempt($credentials)) {
                $request->session()->regenerate();

                return redirect()->intended('dashboard');
            }

            return back()->withErrors([
                'email' => 'The provided credentials do not match our records.',
            ])->onlyInput('email');
        }
    }

Phương thức `attempt` chấp nhận một mảng các cặp key và value làm tham số đầu tiên của nó. Các giá trị trong mảng sẽ được sử dụng để tìm người dùng trong bảng cơ sở dữ liệu của bạn. Vì thế, trong ví dụ trên, người dùng sẽ được lấy ra theo giá trị của cột `email`. Nếu người dùng được tìm thấy, thì trường password đã được mã hoá mà đang được lưu trữ trong cơ sở dữ liệu sẽ được so sánh với giá trị `password` đã được truyền vào. Bạn không cần phải hash giá trị `password` trong request, vì framework sẽ tự động hash giá trị đó trước khi so sánh nó với mật khẩu đã được hash trong cơ sở dữ liệu. Một session đã được xác thực sẽ được cung cấp cho người dùng nếu hai mật khẩu bị hash đó khớp với nhau.

Hãy nhớ rằng, các service xác thực của Laravel sẽ truy xuất người dùng từ cơ sở dữ liệu của bạn dựa trên cấu hình guard xác thực "provider" của bạn. Mặc định trong file cấu hình `config/auth.php`, provider người dùng sẽ được chỉ định là Eloquent và nó sẽ sử dụng model `App\Models\User` khi truy xuất người dùng. Bạn có thể thay đổi các giá trị này trong file cấu hình của bạn dựa trên nhu cầu của ứng dụng.

Phương thức `attempt` sẽ trả về `true` nếu authentication thành công. Và nếu không thành công thì `false` sẽ được trả về.

Phương thức `intended` được cung cấp bởi redirector của Laravel sẽ chuyển hướng người dùng đến URL mà họ đang truy cập trước khi bị chặn lại bởi middleware authentication. Một URI dự phòng có thể được cung cấp cho phương thức này trong trường hợp điểm đến là không có sẵn.

<a name="specifying-additional-conditions"></a>
#### Chỉ định thêm điều kiện bổ sung

Nếu bạn muốn, bạn cũng có thể thêm các điều kiện query vào trong authentication query bên cạnh email và mật khẩu của người dùng. Để thực hiện điều này, chúng ta có thể chỉ cần thêm các điều kiện truy vấn vào mảng được truyền cho phương thức `attemp`. Ví dụ: chúng ta có thể kiểm tra rằng người dùng phải là trạng thái "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // Authentication was successful...
    }

Đối với các điều kiện truy vấn phức tạp, bạn có thể cung cấp một closure trong mảng thông tin xác thực của bạn. Closure này sẽ được gọi cùng với instance truy vấn, cho phép bạn tùy chỉnh truy vấn dựa trên nhu cầu của ứng dụng:

    use Illuminate\Database\Eloquent\Builder;

    if (Auth::attempt([
        'email' => $email,
        'password' => $password,
        fn (Builder $query) => $query->has('activeSubscription'),
    ])) {
        // Authentication was successful...
    }

> [!WARNING]
> Trong các ví dụ này, `email` không phải là một trường bắt buộc, nó chỉ được sử dụng làm ví dụ. Bạn có thể sử dụng bất kỳ tên cột nào tương ứng với "username" trong bảng cơ sở dữ liệu của bạn.

Phương thức `attemptWhen`, nhận vào một closure làm tham số thứ hai, có thể được sử dụng để thực hiện một kiểm tra kỹ hơn đối với người dùng trước khi thực sự xác thực người dùng. Closure này sẽ nhận vào một user và phải trả về `true` hoặc `false` để cho biết user đó có thể được xác thực hay không:

    if (Auth::attemptWhen([
        'email' => $email,
        'password' => $password,
    ], function (User $user) {
        return $user->isNotBanned();
    })) {
        // Authentication was successful...
    }

<a name="accessing-specific-guard-instances"></a>
#### Truy cập vào instance guard cụ thể

Thông qua phương thức `guard` của facade `Auth`, bạn có thể chỉ định instance guard mà bạn muốn sử dụng khi đang xác thực người dùng. Điều này cho phép bạn quản lý authentication cho các phần riêng biệt của application bằng các model hoặc bảng user hoàn toàn riêng biệt.

Tên được truyền vào trong phương thức `guard` phải là một trong các tên mà bạn đã khởi tạo trong file cấu hình `auth.php`:

    if (Auth::guard('admin')->attempt($credentials)) {
        // ...
    }

<a name="remembering-users"></a>
### Nhớ tài khoản user

Nhiều ứng dụng web cung cấp chức năng "remember me" trên form đăng nhập. Nếu bạn muốn cung cấp chức năng "remember me" trong application của bạn, bạn có thể truyền một giá trị boolean làm tham số thứ hai cho phương thức `attempt`.

Khi giá trị này là `true`, Laravel sẽ giữ cho người dùng được authenticate vô thời hạn hoặc cho đến khi họ đăng xuất. Bảng `users` của bạn phải có cột `remember_token`, chuỗi này sẽ được sử dụng để lưu trữ mã token "remember me". Bảng migration `users` đi kèm với các ứng dụng Laravel đã chứa cột này:

    use Illuminate\Support\Facades\Auth;

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

Nếu ứng dụng của bạn cung cấp chức năng "remember me", bạn có thể sử dụng phương thức `viaRemember` để xác định xem người dùng hiện đang được xác thực có được xác thực bằng cookie "remember me" hay không:

    use Illuminate\Support\Facades\Auth;

    if (Auth::viaRemember()) {
        // ...
    }

<a name="other-authentication-methods"></a>
### Các phương thức authentication khác

<a name="authenticate-a-user-instance"></a>
#### Authenticate với một instance user

Nếu bạn cần set một instance người dùng đã tồn tại làm người dùng đã được xác thực, bạn có thể truyền instance người dùng đó sang phương thức `login` của facade `Auth`. Instance người dùng nhất định phải là một implementation của `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts). Model `App\Models\User` đi kèm với Laravel đã implement interface này. Phương thức xác thực này sẽ hữu ích khi bạn đã có một instance người dùng hợp lệ, chẳng hạn như ngay sau khi người dùng đăng ký mới vào ứng dụng của bạn:

    use Illuminate\Support\Facades\Auth;

    Auth::login($user);

Bạn có thể truyền một giá trị boolean làm tham số thứ hai cho phương thức `login`. Giá trị này sẽ cho biết liệu chức năng "remember me" có được dùng tring session xác thực này hay không. Hãy nhớ rằng, điều này có nghĩa là session sẽ được xác thực vô thời hạn hoặc cho đến khi người dùng đăng xuất khỏi ứng dụng theo cách thủ công:

    Auth::login($user, $remember = true);

Nếu cần, bạn có thể chỉ định một guard xác thực trước khi gọi phương thức `login`:

    Auth::guard('admin')->login($user);

<a name="authenticate-a-user-by-id"></a>
#### Authenticate một user qua ID

Để xác thực người dùng bằng khóa chính của record lưu trong cơ sở dữ liệu, bạn có thể sử dụng phương thức `loginUsingId`. Phương thức này chấp nhận khóa chính của người dùng mà bạn muốn authenticate:

    Auth::loginUsingId(1);

Bạn có thể truyền một giá trị boolean làm tham số thứ hai cho phương thức `loginUsingId`. Giá trị này sẽ cho biết liệu chức năng "remember me" có được dùng tring session xác thực này hay không. Hãy nhớ rằng, điều này có nghĩa là session sẽ được xác thực vô thời hạn hoặc cho đến khi người dùng đăng xuất khỏi ứng dụng theo cách thủ công:

    Auth::loginUsingId(1, $remember = true);

<a name="authenticate-a-user-once"></a>
#### Authenticate A User Once

Bạn có thể sử dụng phương thức `once` để xác thực người dùng vào application cho một request. Không có session hoặc cookie nào được sử dụng khi gọi phương thức này:

    if (Auth::once($credentials)) {
        // ...
    }

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) cung cấp một cách nhanh chóng để authenticate người dùng vào application của bạn mà không cần thiết lập ra một trang "login" chuyên dụng. Để bắt đầu, hãy gắn [middleware](/docs/{{version}}/middleware) `auth.basic` vào một route. Middleware `auth.basic` đã được thêm vào sẵn trong framework Laravel, vì vậy bạn không cần định nghĩa nó:

    Route::get('/profile', function () {
        // Only authenticated users may access this route...
    })->middleware('auth.basic');

Khi middleware đã được gắn vào route, bạn sẽ tự động được nhắc về thông tin đăng nhập khi truy cập vào route này trong trình duyệt của bạn. Mặc định, middleware `auth.basic` sẽ giả sử rằng cột `email` trong bảng cơ sở dữ liệu `users` của bạn là "username" của người dùng.

<a name="a-note-on-fastcgi"></a>
#### A Note On FastCGI

Nếu bạn đang sử dụng PHP FastCGI và Apache để làm ứng dụng Laravel của bạn, thì xác thực HTTP Basic có thể không hoạt động chính xác. Để khắc phục sự cố này, các dòng sau có thể được thêm vào file `.htaccess` trong ứng dụng của bạn:

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication

Bạn cũng có thể sử dụng HTTP Basic Authentication mà không cần phải set cookie để định danh người dùng trong session. Điều này chủ yếu hữu ích nếu bạn chọn sử dụng xác thực HTTP là để xác thực các request cho API trong ứng dụng của bạn. Để hoàn thành việc này, hãy [định nghĩa một middleware](/docs/{{version}}/middleware) gọi phương thức `onceBasic`. Nếu không có response nào được trả về bởi phương thức `onceBasic`, request sẽ được chuyển tiếp vào application:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;
    use Symfony\Component\HttpFoundation\Response;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

Tiếp theo, gắn middleware vào một route:

    Route::get('/api/user', function () {
        // Only authenticated users may access this route...
    })->middleware(AuthenticateOnceWithBasicAuth::class);

<a name="logging-out"></a>
## Logging Out

Để đăng xuất người dùng ra khỏi application của bạn, bạn có thể sử dụng phương thức `logout` được cung cấp bởi facade `Auth`. Phương thức này sẽ xóa thông tin xác thực của người dùng ra khỏi session để các request tiếp theo sẽ không được xác thực nữa.

Ngoài việc gọi phương thức `logout`, bạn nên vô hiệu hóa session hiện tại của người dùng và tạo lại một [token CSRF](/docs/{{version}}/csrf) cho họ. Sau khi đăng xuất người dùng, thông thường bạn sẽ chuyển hướng người dùng đến url gốc của ứng dụng:

    use Illuminate\Http\Request;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Support\Facades\Auth;

    /**
     * Log the user out of the application.
     */
    public function logout(Request $request): RedirectResponse
    {
        Auth::logout();

        $request->session()->invalidate();

        $request->session()->regenerateToken();

        return redirect('/');
    }

<a name="invalidating-sessions-on-other-devices"></a>
### Vô hiệu hoá session trên các thiết bị khác

Laravel cũng cung cấp các cơ chế để vô hiệu hoá session và "đăng xuất" người dùng ra khỏi các thiết bị khác của họ mà không vô hiệu hoá session hiện tại của họ. Tính năng này thường được sử dụng khi người dùng đang thay đổi hoặc cập nhật lại mật khẩu của họ và bạn muốn làm mất hiệu lực các session trên các thiết bị khác trong khi vẫn giữ xác thực trên thiết bị hiện tại.

Trước khi bắt đầu, bạn nên đảm bảo là middleware `Illuminate\Session\Middleware\AuthenticateSession` đã được thêm vào các route mà sẽ thực hiện xác thực session. Thông thường, bạn nên để middleware này trên các định nghĩa của các route group để có thể áp dụng nó cho nhiều route trong ứng dụng của bạn. Mặc định, middleware `AuthenticateSession` có thể được gắn vào một route bằng cách sử dụng bí danh middleware route `auth.session` như được định nghĩa trong kernel HTTP trong ứng dụng của bạn:

    Route::middleware(['auth', 'auth.session'])->group(function () {
        Route::get('/', function () {
            // ...
        });
    });

Sau đó, bạn có thể sử dụng phương thức `logoutOtherDevices` được cung cấp bởi facade `Auth`. Phương thức này sẽ yêu cầu người dùng xác nhận mật khẩu hiện tại của họ:

    use Illuminate\Support\Facades\Auth;

    Auth::logoutOtherDevices($currentPassword);

Khi phương thức `logoutOtherDevices` được gọi, thì các session khác của người dùng sẽ bị vô hiệu hoàn toàn, có nghĩa là họ sẽ bị "đăng xuất" ra khỏi tất cả các guard mà họ đã được đăng nhập trước đó.

<a name="password-confirmation"></a>
## Xác nhận password

Trong khi xây dựng ứng dụng của bạn, đôi khi bạn có thể có các hành động mà cần yêu cầu người dùng xác nhận lại mật khẩu của họ trước khi hành động đó được thực hiện hoặc trước khi người dùng được chuyển hướng đến một chỗ nhạy cảm của ứng dụng. Laravel có chứa một middleware được tích hợp sẵn để làm cho quá trình này trở nên dễ dàng. Việc triển khai chức năng này sẽ yêu cầu bạn định nghĩa hai route: một route là để hiển thị view yêu cầu người dùng xác nhận lại mật khẩu của họ và một route khác để xác nhận rằng mật khẩu hợp lệ và chuyển hướng người dùng đến đích dự định của họ.

> [!NOTE]
>  Tài liệu sau đây sẽ thảo luận về cách tích hợp trực tiếp với chức năng xác nhận lại mật khẩu của Laravel; tuy nhiên, nếu bạn muốn bắt đầu nhanh hơn, thì [bộ công cụ khởi động ứng dụng Laravel](/docs/{{version}}/starter-kits) đã hỗ trợ nó!

<a name="password-confirmation-configuration"></a>
### Cấu hình

Sau khi đã xác nhận mật khẩu của họ, người dùng sẽ không bị yêu cầu xác nhận lại mật khẩu của họ trong ba giờ. Tuy nhiên, bạn có thể cấu hình khoảng thời gian trước khi người bị nhắc phải nhập lại mật khẩu của họ bằng cách thay đổi giá trị của giá trị cấu hình `password_timeout` trong file cấu hình `config/auth.php` trong ứng dụng của bạn.

<a name="password-confirmation-routing"></a>
### Routing

<a name="the-password-confirmation-form"></a>
#### The Password Confirmation Form

Đầu tiên, chúng ta sẽ định nghĩa một route để hiển thị view để yêu cầu người dùng xác nhận lại mật khẩu của họ:

    Route::get('/confirm-password', function () {
        return view('auth.confirm-password');
    })->middleware('auth')->name('password.confirm');

Như bạn có thể mong muốn, view mà được trả về bởi route này sẽ phải có một form chứa trường `password`. Ngoài ra, vui lòng chứa thêm một đoạn text vào trong view để giải thích cho người dùng rằng họ đang vào một chỗ được bảo vệ trong ứng dụng và phải xác nhận lại mật khẩu của họ.

<a name="confirming-the-password"></a>
#### Confirming The Password

Tiếp theo, chúng ta sẽ định nghĩa một route để xử lý request form từ view "xác nhận lại mật khẩu". Route này sẽ chịu trách nhiệm xác thực mật khẩu và chuyển hướng người dùng đến đích dự định của họ:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Facades\Redirect;

    Route::post('/confirm-password', function (Request $request) {
        if (! Hash::check($request->password, $request->user()->password)) {
            return back()->withErrors([
                'password' => ['The provided password does not match our records.']
            ]);
        }

        $request->session()->passwordConfirmed();

        return redirect()->intended();
    })->middleware(['auth', 'throttle:6,1']);

Trước khi tiếp tục, hãy xem xét route này chi tiết hơn. Đầu tiên, trường `password` trong request được xác định là thực sự khớp với mật khẩu của người dùng đang được xác thực. Nếu mật khẩu hợp lệ, chúng ta cần thông báo cho session của Laravel là người dùng đã xác nhận mật khẩu của họ. Và phương thức `passwordConfirmed` sẽ set một timestamp vào trong session của người dùng mà Laravel có thể sử dụng để xác định thời điểm người dùng sẽ phải xác nhận lại mật khẩu của họ. Cuối cùng, chúng ta có thể chuyển hướng người dùng đến đích dự định của họ.

<a name="password-confirmation-protecting-routes"></a>
### Bảo vệ route

Bạn nên đảm bảo rằng bất kỳ route nào thực hiện hành động yêu cầu xác nhận lại mật khẩu đều được gán với middleware `password.confirm`. Middleware này được có trong cài đặt mặc định của Laravel và sẽ tự động lưu trữ đích dự định của người dùng vào trong session để người dùng có thể được chuyển hướng đến vị trí đó sau khi xác nhận mật khẩu của họ. Sau khi lưu trữ đích dự định của người dùng trong session, middleware sẽ chuyển hướng người dùng đó đến [route đã được đặt tên](/docs/{{version}}/routing#named-routes) là `password.confirm` :

    Route::get('/settings', function () {
        // ...
    })->middleware(['password.confirm']);

    Route::post('/settings', function () {
        // ...
    })->middleware(['password.confirm']);

<a name="adding-custom-guards"></a>
## Thêm tuỳ biến guard

Bạn có thể định nghĩa các guard authentication của riêng bạn bằng cách sử dụng phương thức `extend` trên facade `Auth`. Bạn nên gọi tới phương thức `extend` trong một [service provider](/docs/{{version}}/providers). Vì Laravel đã có sẵn một `AuthServiceProvider`, nên chúng ta có thể đặt code đó vào trong provider này:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Auth;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         */
        public function boot(): void
        {
            Auth::extend('jwt', function (Application $app, string $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

Như bạn có thể thấy trong ví dụ trên, hàm callback được truyền vào trong phương thức `extend` sẽ trả về một implementation của class `Illuminate\Contracts\Auth\Guard`. Interface này sẽ chứa một vài phương thức mà bạn sẽ cần phải implement để định nghĩa một guard tùy chỉnh. Khi guard tùy chỉnh của bạn đã được định nghĩa, bạn có thể tham chiếu đến guard này trong cấu hình `guards` của file cấu hình `auth.php` của bạn:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="closure-request-guards"></a>
### Closure Request Guards

Cách đơn giản nhất để làm một hệ thống xác thực tùy biến dựa trên HTTP request là sử dụng phương thức `Auth::viaRequest`. Phương thức này sẽ cho phép bạn nhanh chóng định nghĩa quy trình xác thực của bạn bằng một closure duy nhất.

Để bắt đầu, hãy gọi phương thức `Auth::viaRequest` trong hàm `boot` của class `AuthServiceProvider`. Phương thức `viaRequest` sẽ chấp nhận tên của authentication driver làm tham số đầu tiên của nó. Tên này có thể là bất kỳ chuỗi nào mà mô tả guard tùy biến của bạn. Tham số thứ hai được truyền cho phương thức sẽ là một closure nhận vào một HTTP request và sẽ trả về một instance người dùng hoặc nếu xác thực không thành công, thì sẽ là `null`:

    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    /**
     * Register any application authentication / authorization services.
     */
    public function boot(): void
    {
        Auth::viaRequest('custom-token', function (Request $request) {
            return User::where('token', (string) $request->token)->first();
        });
    }

Sau khi authentication driver tùy biến của bạn đã được định nghĩa, bạn có thể cấu hình nó như là một driver trong cấu hình `guards` trong file cấu hình `auth.php` của bạn:

    'guards' => [
        'api' => [
            'driver' => 'custom-token',
        ],
    ],

Cuối cùng, bạn có thể gọi đến guard đó khi gán middleware xác thực cho một route:

    Route::middleware('auth:api')->group(function () {
        // ...
    });

<a name="adding-custom-user-providers"></a>
## Thêm tuỳ biến user provider

Nếu bạn không sử dụng cơ sở dữ liệu quan hệ để lưu trữ thông tin người dùng của bạn, bạn sẽ cần mở rộng Laravel với một user provider authentication của riêng bạn. Chúng ta sẽ sử dụng phương thức `provider` trên facade `Auth` để định nghĩa user provider tùy chỉnh mới này. User provider resolver sẽ trả về một implementation của `Illuminate\Contracts\Auth\UserProvider`:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoUserProvider;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Auth;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         */
        public function boot(): void
        {
            Auth::provider('mongo', function (Application $app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new MongoUserProvider($app->make('mongo.connection'));
            });
        }
    }

Sau khi bạn đã đăng ký provider bằng phương thức `provider`, bạn có thể chuyển user provider mới này vào trong file cấu hình `auth.php` của bạn. Đầu tiên, định nghĩa một `provider` mà sử dụng driver mới của bạn:

    'providers' => [
        'users' => [
            'driver' => 'mongo',
        ],
    ],

Cuối cùng, bạn có thể tham chiếu đến provider này trong cấu hình `guards` của bạn:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### User Provider Contract

Các implementation từ `Illuminate\Contracts\Auth\UserProvider` sẽ chịu trách nhiệm lấy ra đối tượng đã được implementation từ `Illuminate\Contracts\Auth\Authenticatable` từ cơ sở dữ liệu, như MySQL, MongoDB, vv... Hai interface này cho phép các cơ chế Laravel authentication tiếp tục hoạt động bất kể dữ liệu người dùng đang được lưu trữ như thế nào hoặc bất kể loại class nào thể hiện the authenticated user:

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

Hàm `retrieveByToken` lấy ra một người dùng bằng unique `$identifier` của nó và "remember me" `$token`, thường được lưu trữ một cột trong cơ sở dữ liệu ví dụ như cột `remember_token`. Như với phương thức trước đó, việc implementation `Authenticatable` với giá trị token phù hợp sẽ được phương thức này trả về.

Phương thức `updateRememberToken` sẽ cập nhật `remember_token` của instance với `$token` mới của `$user`. Một mã token mới sẽ được chỉ định khi đăng nhập "remember me" thành công hoặc khi người dùng đăng xuất.

Phương thức `retrieveByCredentials` sẽ nhận một mảng thông tin đăng nhập được truyền vào phương thức `Auth::attempt` khi xác thực vào application. Phương thức này sẽ "truy vấn" bộ lưu trữ bên dưới để lấy ra người dùng khớp với các thông tin đăng nhập. Thông thường, phương thức này sẽ chạy một truy vấn với điều kiện "where" tìm kiếm record người dùng mà có "username" khớp với giá trị của `$credentials['username']`. Phương thức này sẽ trả về một đối tượng đã được implementation `Authenticatable`. **Phương thức này không nên thực hiện bất kỳ hành động authentication hoặc validation mật khẩu nào.**

Phương thức `validateCredentials` sẽ so sánh `$user` đã nhận với `$credentials` của người dùng để authenticate. Ví dụ, phương thức này will typically sử dụng phương thức `Hash::check` để so sánh giá trị của `$user->getAuthPassword()` với giá trị của `$credentials['password']`. Phương thức này sẽ trả về giá trị `true` hoặc `false` cho biết mật khẩu có hợp lệ hay không.

<a name="the-authenticatable-contract"></a>
### Authenticatable Contract

Sau khi chúng ta đã khám phá các phương thức trên `UserProvider`, bây giờ chúng ta hãy xem contract `Authenticatable`. Hãy nhớ rằng, user provider nên trả về các đối tượng mà đã implementation của interface này từ các phương thức `retrieveById`, `retrieveByToken` và `retrieveByCredentials`:

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

Interface này rất đơn giản. Phương thức `getAuthIdentifierName` sẽ trả về tên của field "primary key" và phương thức `getAuthIdentifier` sẽ trả về giá trị của field đó. Khi sử dụng một back-end của MySQL, đây có thể là khóa chính tự động tăng được gán với một record người dùng. Phương thức `getAuthPassword` sẽ trả lại mật khẩu đã hash của người dùng.

Interface này cho phép hệ thống authentication hoạt động với bất kỳ class "user" nào, bất kể nó là ORM hay lớp lưu trữ trừu tượng nào mà bạn đang sử dụng. Mặc định, Laravel đã chứa một class `App\Models\User` trong thư mục `app/Models` và đã implement sẵn interface này, vì vậy bạn có thể tham khảo class này để biết thêm về các implementation này.

<a name="events"></a>
## Event

Laravel gửi nhiều [events](/docs/{{version}}/events) khác nhau trong quá trình authentication. Bạn có thể gắn listener vào các event này trong `EventServiceProvider` của bạn:

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
