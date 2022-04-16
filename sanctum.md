# Laravel Sanctum

- [Giới thiêu](#introduction)
    - [Nó hoạt động như thế nào](#how-it-works)
- [Cài đặt](#installation)
- [API Token Authentication](#api-token-authentication)
    - [Phát hành API Token](#issuing-api-tokens)
    - [Quyền của token](#token-abilities)
    - [Bảo vệ route](#protecting-routes)
    - [Thu hồi token](#revoking-tokens)
- [SPA Authentication](#spa-authentication)
    - [Cấu hình](#spa-configuration)
    - [Authenticating](#spa-authenticating)
    - [Bảo vệ route](#protecting-spa-routes)
    - [Authorizing Private Broadcast Channels](#authorizing-private-broadcast-channels)
- [Mobile Application Authentication](#mobile-application-authentication)
    - [Phát hành API Token](#issuing-mobile-api-tokens)
    - [Bảo vệ route](#protecting-mobile-api-routes)
    - [Thu hồi token](#revoking-mobile-api-tokens)
- [Testing](#testing)

<a name="introduction"></a>
## Giới thiêu

Laravel Sanctum cung cấp một hệ thống xác thực nhẹ cho các SPAs (các ứng dụng single page), ứng dụng di động và các API đơn giản dựa trên token. Sanctum cho phép mỗi người dùng ứng dụng của bạn tạo ra nhiều API token cho tài khoản của họ. Các token này có thể được cấp các quyền / phạm vi cụ thể cho các hành động mà token được phép thực hiện.

<a name="how-it-works"></a>
### Nó hoạt động như thế nào

Laravel Sanctum tồn tại để giải quyết hai vấn đề chính.

#### API Tokens

Đầu tiên, đây là một package đơn giản để phát hành API token cho người dùng của bạn mà không có sự phức tạp của OAuth. Tính năng này được lấy cảm hứng từ "access token" của GitHub. Ví dụ: hãy tưởng tượng trong "cài đặt tài khoản" của ứng dụng của bạn có một màn hình để người dùng có thể tạo các API token cho tài khoản của họ. Bạn có thể sử dụng Sanctum để tạo và quản lý các token đó. Những token này thường có thời gian hết hạn rất dài (tính bằng năm), nhưng người dùng có thể thu hồi các API token đó bất cứ lúc nào.

Laravel Sanctum cung cấp tính năng này bằng cách lưu trữ các API token của người dùng vào trong một bảng cơ sở dữ liệu và xác thực các request thông qua header `Authorization`, header này phải chứa các API token hợp lệ.

#### SPA Authentication

Thứ hai, Sanctum cũng cung cấp một cách đơn giản để xác thực các ứng dụng single page (SPAs) cần giao tiếp với một API được hỗ trợ bởi Laravel. Các ứng dụng SPAs này có thể tồn tại trong cùng một repository như một ứng dụng Laravel của bạn hoặc có thể là một repository hoàn toàn khác riêng biệt, chẳng hạn như một SPA được tạo bằng Vue CLI.

Đối với tính năng này, Sanctum không sử dụng bất kỳ loại token nào. Thay vào đó, Sanctum sử dụng các service xác thực session dựa trên cookie được tích hợp sẵn trong Laravel. Điều này cung cấp các lợi ích về bảo vệ CSRF, xác thực session, cũng như bảo vệ chống rò rỉ thông tin xác thực thông qua XSS. Sanctum sẽ chỉ cố gắng xác thực bằng cookie khi request bắt nguồn từ frontend SPA của chính bạn.

> {tip} Sẽ hoàn toàn tốt nếu chỉ sử dụng Sanctum để xác thực các API token hoặc là xác thực SPA. Nếu bạn sử dụng Sanctum không có nghĩa là bạn bị bắt buộc phải sử dụng cả hai tính năng mà nó cung cấp, bạn có thể sử dụng một trong hai.

<a name="installation"></a>
## Cài đặt

Bạn có thể cài đặt Laravel Sanctum thông qua Composer:

    composer require laravel/sanctum

Tiếp theo, bạn nên export cấu hình Sanctum và các file migration bằng lệnh Artisan `vendor:publish`. File cấu hình `sanctum` sẽ được lưu trong thư mục `config` của bạn:

    php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

Cuối cùng, bạn nên chạy migration cơ sở dữ liệu của bạn. Sanctum sẽ tạo ra một bảng cơ sở dữ liệu để lưu trữ các API token:

    php artisan migrate

Tiếp theo, nếu bạn muốn sử dụng Sanctum để xác thực một SPA, bạn nên thêm middleware của Sanctum vào group middleware `api` trong file `app/Http/Kernel.php` của bạn:

    use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;

    'api' => [
        EnsureFrontendRequestsAreStateful::class,
        'throttle:60,1',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

#### Migration Customization

Nếu bạn không sử dụng migration mặc định của Sanctum, bạn nên gọi phương thức `Sanctum::ignoreMigrations` trong phương thức `register` của `AppServiceProvider` của bạn. Bạn có thể export các migration mặc định bằng cách sử dụng `php artisan vendor:publish --tag=sanctum-migrations`.

<a name="api-token-authentication"></a>
## API Token Authentication

> {tip} Bạn không nên sử dụng API token để xác thực các ứng dụng SPA của riêng bạn. Thay vào đó, hãy sử dụng [xác thực SPA](#spa-authentication) được tích hợp sẵn của Sanctum.

<a name="issuing-api-tokens"></a>
### Phát hành API Token

Sanctum cho phép bạn phát hành các API token hoặc các personal access token có thể được sử dụng để xác thực các API request. Khi thực hiện các request bằng API token, token đó phải được chứa trong một header `Authorization` như là một token `Bearer`.

Để bắt đầu phát hành token cho người dùng, model User của bạn nên sử dụng trait `HasApiTokens`:

    use Laravel\Sanctum\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

Để phát hành một token, bạn có thể sử dụng phương thức `createToken`. Phương thức `createToken` sẽ trả về một instance `Laravel\Sanctum\NewAccessToken`. API token này sẽ được hash bằng cách sử dụng hàm hash SHA-256 trước khi được lưu vào trong cơ sở dữ liệu của bạn, nhưng bạn có thể truy cập vào giá trị thật của token này bằng cách sử dụng thuộc tính `plainTextToken` của instance `NewAccessToken`. Bạn nên hiển thị giá trị thật này cho người dùng ngay sau khi token được tạo:

    $token = $user->createToken('token-name');

    return $token->plainTextToken;

Bạn có thể truy cập vào tất cả các token của người dùng bằng cách sử dụng quan hệ Eloquent `tokens` được cung cấp bởi trait `HasApiTokens`:

    foreach ($user->tokens as $token) {
        //
    }

<a name="token-abilities"></a>
### Quyền của token

Sanctum cho phép bạn gán các token vào các "quyền", tương tự như OAuth "scopes". Bạn có thể truyền một mảng quyền làm tham số thứ hai cho phương thức `createToken`:

    return $user->createToken('token-name', ['server:update'])->plainTextToken;

Khi xử lý một request được Sanctum xác thực, bạn có thể xác định xem token đó có một quyền nhất định hay không bằng cách sử dụng phương thức `tokenCan`:

    if ($user->tokenCan('server:update')) {
        //
    }

> {tip} Để thuận tiện, phương thức `tokenCan` sẽ luôn trả về `true` nếu request cần được xác thực đến từ một SPA của bạn và bạn đang sử dụng [xác thực SPA](#spa-authentication) được tích hợp sẵn của Sanctum.

<a name="protecting-routes"></a>
### Bảo vệ route

Để bảo vệ các route sao cho tất cả các request phải được xác thực, bạn nên gắn guard `sanctum` vào các route API của bạn trong file `routes/api.php`. Guard này sẽ đảm bảo rằng các request sẽ được xác thực trạng thái từ SPA của bạn hoặc chứa một header API token hợp lệ nếu request đến từ bên thứ ba:

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="revoking-tokens"></a>
### Thu hồi token

Bạn có thể "thu hồi" token bằng cách xóa chúng ra khỏi cơ sở dữ liệu của bạn bằng cách sử dụng quan hệ `tokens` được cung cấp bởi trait `HasApiTokens`:

    // Revoke all tokens...
    $user->tokens()->delete();

    // Revoke the user's current token...
    $request->user()->currentAccessToken()->delete();

    // Revoke a specific token...
    $user->tokens()->where('id', $id)->delete();

<a name="spa-authentication"></a>
## SPA Authentication

Sanctum cũng cung cấp một cách đơn giản để xác thực các ứng dụng single page (SPAs) cần giao tiếp với một API được hỗ trợ bởi Laravel. Các ứng dụng SPAs này có thể tồn tại trong cùng một repository như một ứng dụng Laravel của bạn hoặc có thể là một repository hoàn toàn khác riêng biệt, chẳng hạn như một SPA được tạo bằng Vue CLI.

Đối với tính năng này, Sanctum không sử dụng bất kỳ loại token nào. Thay vào đó, Sanctum sử dụng các service xác thực session dựa trên cookie được tích hợp sẵn trong Laravel. Điều này cung cấp các lợi ích về bảo vệ CSRF, xác thực session, cũng như bảo vệ chống rò rỉ thông tin xác thực thông qua XSS. Sanctum sẽ chỉ cố gắng xác thực bằng cookie khi request bắt nguồn từ frontend SPA của chính bạn.

> {note} Để xác thực, SPA và API của bạn phải chia sẻ cùng một tên miền. Tuy nhiên, chúng có thể được set trên các subdomain khác nhau.

<a name="spa-configuration"></a>
### Cấu hình

#### Configuring Your First-Party Domains

Đầu tiên, bạn nên cấu hình các tên miền mà SPA của bạn sẽ thực hiện request từ đó. Bạn có thể cấu hình các tên miền này bằng cách sử dụng tùy chọn cấu hình `stateful` trong file cấu hình `sanctum` của bạn. Cài đặt cấu hình này sẽ xác định xem tên miền nào sẽ duy trì "trạng thái" xác thực bằng cách sử dụng session cookie Laravel khi tạo request tới API của bạn.

> {note} Nếu bạn đang truy cập ứng dụng của bạn thông qua URL có cổng (`127.0.0.1:8000`), bạn nên đảm bảo là bạn đã cấu hình cả số cổng với tên miền.

#### Sanctum Middleware

Tiếp theo, bạn nên thêm middleware của Sanctum vào group middleware `api` trong file `app/Http/Kernel.php` của bạn. Middleware này sẽ chịu trách nhiệm đảm bảo rằng các request đến từ các SPA của bạn có thể được xác thực bằng session cookie của Laravel, trong khi vẫn cho phép các request từ bên thứ ba hoặc ứng dụng di động xác thực bằng cách sử dụng API token:

    use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;

    'api' => [
        EnsureFrontendRequestsAreStateful::class,
        'throttle:60,1',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

<a name="cors-and-cookies"></a>
#### CORS & Cookies

Nếu bạn gặp sự cố khi xác thực ứng dụng của bạn từ một SPA chạy trên một subdomain riêng biệt, có thể bạn đã cấu hình sai cài đặt CORS (Cross-Origin Resource Sharing) hoặc session cookie của bạn.

Bạn nên đảm bảo là cấu hình CORS của ứng dụng của bạn đang trả về header `Access-Control-Allow-Credentials` có giá trị là `True` bằng cách set tùy chọn `supports_credentials` trong file cấu hình `cors` của ứng dụng thành `true`.

Ngoài ra, bạn cũng nên thêm tùy chọn `withCredentials` trên instance global `axios` của bạn. Thông thường, điều này sẽ được thực hiện trong file `resources/js/bootstrap.js` của bạn:

    axios.defaults.withCredentials = true;

Cuối cùng, bạn nên đảm bảo cấu hình session cookie của têm miền trong ứng dụng hỗ trợ tất cả các subdomain của tên miền gốc. Bạn có thể thực hiện việc này bằng cách set thêm tiền tố dấu `.` đứng trước tên miền bằng trong file cấu hình `session` của bạn:

    'domain' => '.domain.com',

<a name="spa-authenticating"></a>
### Authenticating

Để xác thực SPA của bạn, trước tiên, trang đăng nhập của SPA của bạn phải thực hiện một request đến route `/sanctum/csrf-cookie` để khởi tạo bảo vệ CSRF cho ứng dụng:

    axios.get('/sanctum/csrf-cookie').then(response => {
        // Login...
    });

Trong request này, Laravel sẽ set cookie `XSRF-TOKEN` chứa token CSRF hiện tại. Token này sau đó sẽ được truyền vào trong header `X-XSRF-TOKEN` trong các request tiếp theo, mà các thư viện như Axios và Angular HttpClient sẽ tự động thực hiện cho bạn.

Khi bảo vệ CSRF đã được khởi tạo, bạn nên thực hiện một request `POST` đến route Laravel `/login`. Route `/login` này có thể được cung cấp bởi package `laravel/ui` [authentication scaffolding](/docs/{{version}}/authentication#introduction).

Nếu request đăng nhập thành công, bạn sẽ được xác thực và các request tiếp theo đối với các route API của bạn sẽ tự động xác thực thông qua session cookie mà backend của Laravel đã cung cấp cho ứng dụng của bạn.

> {tip} Bạn có thể tự do thoải mái viết bất kỳ endpoint `/login` nào của riêng bạn; tuy nhiên, bạn nên đảm bảo rằng nó xác thực người dùng bằng cách sử dụng tiêu chuẩn [dịch vụ xác thực dựa trên session mà Laravel cung cấp](/docs/{{version}}/authentication#authenticating-users).

<a name="protecting-spa-routes"></a>
### Bảo vệ route

Để bảo vệ các route sao cho tất cả các request phải được xác thực, bạn nên gắn guard `sanctum` vào các route API của bạn trong file `routes/api.php`. Guard này sẽ đảm bảo rằng các request sẽ được xác thực trạng thái từ SPA của bạn hoặc chứa một header API token hợp lệ nếu request đến từ bên thứ ba:

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="authorizing-private-broadcast-channels"></a>
### Authorizing Private Broadcast Channels

Nếu SPA của bạn cần xác thực với [các channel private / presence broadcast](/docs/{{version}}/broadcasting#authorizing-channels), bạn nên gọi phương thức `Broadcast::routes` trong file `routes/api.php` của bạn:

    Broadcast::routes(['middleware' => ['auth:sanctum']]);

Tiếp theo, để các authorization request của Pusher thành công, bạn sẽ cần phải cung cấp một tùy chỉnh `authorizer` của Pusher khi khởi tạo [Laravel Echo](/docs/{{version}}/broadcasting#installing-laravel-echo). Điều này cho phép ứng dụng của bạn cấu hình Pusher để sử dụng một instance `axios` được [cấu hình đúng cho các request cross-domain](#cors-and-cookies):

    window.Echo = new Echo({
        broadcaster: "pusher",
        cluster: process.env.MIX_PUSHER_APP_CLUSTER,
        encrypted: true,
        key: process.env.MIX_PUSHER_APP_KEY,
        authorizer: (channel, options) => {
            return {
                authorize: (socketId, callback) => {
                    axios.post('/api/broadcasting/auth', {
                        socket_id: socketId,
                        channel_name: channel.name
                    })
                    .then(response => {
                        callback(false, response.data);
                    })
                    .catch(error => {
                        callback(true, error);
                    });
                }
            };
        },
    })

<a name="mobile-application-authentication"></a>
## Mobile Application Authentication

Bạn có thể sử dụng token Sanctum để xác thực các request của ứng dụng di động đối với API của bạn. Quy trình xác thực các request ứng dụng di động tương tự như xác thực các request API của bên thứ ba; tuy nhiên, có những khác biệt nhỏ về cách bạn sẽ phát hành ra các API token.

<a name="issuing-mobile-api-tokens"></a>
### Phát hành API Token

Để bắt đầu, hãy tạo một route chấp nhận email hoặc tên người dùng, mật khẩu và tên thiết bị của người dùng, sau đó kiểm tra các thông tin đăng nhập đó để lấy token Sanctum mới. Endpoint sẽ trả về một token Sanctum thật để có thể được lưu trữ trên thiết bị di động và được sử dụng để thực hiện thêm các request API sau đó:

    use App\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Validation\ValidationException;

    Route::post('/sanctum/token', function (Request $request) {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
            'device_name' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (! $user || ! Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        return $user->createToken($request->device_name)->plainTextToken;
    });

Khi thiết bị di động sử dụng token để thực hiện một request API đối với ứng dụng của bạn, thiết bị sẽ truyền token vào trong header `Authorization` dưới dạng một token `Bearer`.

> {mẹo} Khi phát hành token cho ứng dụng di động, bạn cũng có thể tự do chỉ định [các quyền cho token](#token-abilities)

<a name="protecting-mobile-api-routes"></a>
### Bảo vệ route

Như đã được ghi ở trước đó, bạn có thể bảo vệ các route để tất cả các request đến phải được xác thực bằng cách gắn guard `sanctum` vào các route. Thông thường, bạn sẽ gắn guard này vào các route thường được định nghĩa trong file `routes/api.php` của bạn:

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="revoking-mobile-api-tokens"></a>
### Thu hồi token

Để cho phép người dùng thu hồi token API đã cấp cho thiết bị di động, bạn có thể liệt kê chúng theo tên, cùng với nút "thu hồi", trong phần "cài đặt tài khoản" trong giao diện web người dùng ứng dụng. Khi người dùng nhấp vào nút "thu hồi", bạn có thể xóa token ra khỏi cơ sở dữ liệu. Hãy nhớ rằng, bạn có thể truy cập vào token API của người dùng thông qua quan hệ `tokens` được cung cấp sẵn bởi trait `HasApiTokens`:

    // Revoke all tokens...
    $user->tokens()->delete();

    // Revoke a specific token...
    $user->tokens()->where('id', $id)->delete();

<a name="testing"></a>
## Testing

Trong khi testing, phương thức `Sanctum::actingAs` có thể được sử dụng để xác thực một người dùng và chỉ định quyền nào sẽ được cấp cho token của họ:

    use App\User;
    use Laravel\Sanctum\Sanctum;

    public function test_task_list_can_be_retrieved()
    {
        Sanctum::actingAs(
            factory(User::class)->create(),
            ['view-tasks']
        );

        $response = $this->get('/api/task');

        $response->assertOk();
    }

Nếu bạn muốn cấp tất cả các quyền cho một token, bạn nên thêm dấu `*` vào trong danh sách các quyền được cung cấp cho phương thức `actingAs`:

    Sanctum::actingAs(
        factory(User::class)->create(),
        ['*']
    );
