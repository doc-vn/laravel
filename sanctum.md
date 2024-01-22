# Laravel Sanctum

- [Giới thiệu](#introduction)
    - [Nó hoạt động như thế nào](#how-it-works)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
    - [Ghi đè model mặc định](#overriding-default-models)
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
## Giới thiệu

[Laravel Sanctum](https://github.com/laravel/sanctum) cung cấp một hệ thống xác thực nhẹ cho các SPAs (các ứng dụng single page), ứng dụng di động và các API đơn giản dựa trên token. Sanctum cho phép mỗi người dùng ứng dụng của bạn tạo ra nhiều API token cho tài khoản của họ. Các token này có thể được cấp các quyền / phạm vi cụ thể cho các hành động mà token được phép thực hiện.

<a name="how-it-works"></a>
### Nó hoạt động như thế nào

Laravel Sanctum tồn tại để giải quyết hai vấn đề chính. Hãy thảo luận từng vấn đề trước khi tìm hiểu sâu hơn về library.

<a name="how-it-works-api-tokens"></a>
#### API Tokens

Đầu tiên, Sanctum là một package đơn giản bạn có thể dùng để phát hành API token cho người dùng của bạn mà không có sự phức tạp của OAuth. Tính năng này được lấy cảm hứng từ GitHub và các ứng dụng khác phát hành "personal access token". Ví dụ: hãy tưởng tượng trong "cài đặt tài khoản" của ứng dụng của bạn có một màn hình để người dùng có thể tạo các API token cho tài khoản của họ. Bạn có thể sử dụng Sanctum để tạo và quản lý các token đó. Những token này thường có thời gian hết hạn rất dài (tính bằng năm), nhưng người dùng có thể thu hồi các API token đó bất cứ lúc nào.

Laravel Sanctum cung cấp tính năng này bằng cách lưu trữ các API token của người dùng vào trong một bảng cơ sở dữ liệu và xác thực các HTTP request thông qua header `Authorization`, header này phải chứa các API token hợp lệ.

<a name="how-it-works-spa-authentication"></a>
#### SPA Authentication

Thứ hai, Sanctum cũng cung cấp một cách đơn giản để xác thực các ứng dụng single page (SPAs) cần giao tiếp với một API được hỗ trợ bởi Laravel. Các ứng dụng SPAs này có thể tồn tại trong cùng một repository như một ứng dụng Laravel của bạn hoặc có thể là một repository hoàn toàn khác riêng biệt, chẳng hạn như một SPA được tạo bằng Vue CLI hoặc một ứng dụng tạo từ Next.js.

Đối với tính năng này, Sanctum không sử dụng bất kỳ loại token nào. Thay vào đó, Sanctum sử dụng các service xác thực session dựa trên cookie được tích hợp sẵn trong Laravel. Thông thường, Sanctum sử dụng authentication guard `web` của Laravel để thực hiện việc này. Điều này cung cấp các lợi ích về bảo vệ CSRF, xác thực session, cũng như bảo vệ chống rò rỉ thông tin xác thực thông qua XSS.

Sanctum sẽ chỉ cố gắng xác thực bằng cookie khi request bắt nguồn từ frontend SPA của chính bạn. Khi Sanctum kiểm tra một request HTTP đến, trước tiên nó sẽ kiểm tra cookie authentication và nếu không có cookie nào thì Sanctum sẽ kiểm tra header `Authorization` để tìm API token hợp lệ.

> {tip} Sẽ hoàn toàn tốt nếu chỉ sử dụng Sanctum để xác thực các API token hoặc là xác thực SPA. Nếu bạn sử dụng Sanctum không có nghĩa là bạn bị bắt buộc phải sử dụng cả hai tính năng mà nó cung cấp, bạn có thể sử dụng một trong hai.

<a name="installation"></a>
## Cài đặt

> {tip} Phiên bản mới nhất của Laravel đã chứa Laravel Sanctum. Tuy nhiên, nếu file `composer.json` của ứng dụng của bạn không chứa `laravel/sanctum`, bạn có thể làm theo hướng dẫn cài đặt bên dưới.

Bạn có thể cài đặt Laravel Sanctum thông qua Composer package manager:

    composer require laravel/sanctum

Tiếp theo, bạn nên export cấu hình Sanctum và các file migration bằng lệnh Artisan `vendor:publish`. File cấu hình `sanctum` sẽ được lưu trong thư mục `config` của application:

    php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

Cuối cùng, bạn nên chạy migration cơ sở dữ liệu của bạn. Sanctum sẽ tạo ra một bảng cơ sở dữ liệu để lưu trữ các API token:

    php artisan migrate

Tiếp theo, nếu bạn muốn sử dụng Sanctum để xác thực một SPA, bạn nên thêm middleware của Sanctum vào group middleware `api` trong file `app/Http/Kernel.php` của application:

    'api' => [
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

<a name="migration-customization"></a>
#### Migration Customization

Nếu bạn không sử dụng migration mặc định của Sanctum, bạn nên gọi phương thức `Sanctum::ignoreMigrations` trong phương thức `register` của `App\Providers\AppServiceProvider` của bạn. Bạn có thể export các migration mặc định bằng cách chạy lệnh sau: `php artisan vendor:publish --tag=sanctum-migrations`

<a name="configuration"></a>
## Cấu hình

<a name="overriding-default-models"></a>
### Ghi đè model mặc định

Mặc dù không bắt buộc, nhưng bạn có thể thoải mái extend model `PersonalAccessToken` được Sanctum sử dụng bên trong:

    use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

    class PersonalAccessToken extends SanctumPersonalAccessToken
    {
        // ...
    }

Sau đó, bạn có thể hướng dẫn Sanctum sử dụng model tùy chỉnh của bạn thông qua phương thức `usePersonalAccessTokenModel` do Sanctum cung cấp. Thông thường, bạn nên gọi phương thức này trong phương thức `boot` của một trong những service provider cho ứng dụng của bạn:

    use App\Models\Sanctum\PersonalAccessToken;
    use Laravel\Sanctum\Sanctum;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
    }

<a name="api-token-authentication"></a>
## API Token Authentication

> {tip} Bạn không nên sử dụng API token để xác thực các ứng dụng SPA của riêng bạn. Thay vào đó, hãy sử dụng [chức năng xác thực SPA](#spa-authentication) được tích hợp sẵn của Sanctum.

<a name="issuing-api-tokens"></a>
### Phát hành API Token

Sanctum cho phép bạn phát hành các API token hoặc các personal access token có thể được sử dụng để xác thực các API request đến application của bạn. Khi thực hiện các request bằng API token, token đó phải được chứa trong một header `Authorization` như là một token `Bearer`.

Để bắt đầu phát hành token cho người dùng, model User của bạn nên sử dụng trait `Laravel\Sanctum\HasApiTokens`:

    use Laravel\Sanctum\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }

Để phát hành một token, bạn có thể sử dụng phương thức `createToken`. Phương thức `createToken` sẽ trả về một instance `Laravel\Sanctum\NewAccessToken`. API token này sẽ được hash bằng cách sử dụng hàm hash SHA-256 trước khi được lưu vào trong cơ sở dữ liệu của bạn, nhưng bạn có thể truy cập vào giá trị thật của token này bằng cách sử dụng thuộc tính `plainTextToken` của instance `NewAccessToken`. Bạn nên hiển thị giá trị thật này cho người dùng ngay sau khi token được tạo:

    use Illuminate\Http\Request;

    Route::post('/tokens/create', function (Request $request) {
        $token = $request->user()->createToken($request->token_name);

        return ['token' => $token->plainTextToken];
    });

Bạn có thể truy cập vào tất cả các token của người dùng bằng cách sử dụng quan hệ Eloquent `tokens` được cung cấp bởi trait `HasApiTokens`:

    foreach ($user->tokens as $token) {
        //
    }

<a name="token-abilities"></a>
### Quyền của token

Sanctum cho phép bạn gán các token vào các "quyền". Mục đích của các "quyền" tương tự như "scope" của OAuth. Bạn có thể truyền một mảng quyền làm tham số thứ hai cho phương thức `createToken`:

    return $user->createToken('token-name', ['server:update'])->plainTextToken;

Khi xử lý một request được Sanctum xác thực, bạn có thể xác định xem token đó có một quyền nhất định hay không bằng cách sử dụng phương thức `tokenCan`:

    if ($user->tokenCan('server:update')) {
        //
    }

<a name="token-ability-middleware"></a>
#### Token Ability Middleware

Sanctum cũng chứa hai middleware có thể được sử dụng để xác minh request đến là đã được xác thực bằng một token mà đã được cấp một quyền nhất định. Để bắt đầu, hãy thêm middleware sau vào thuộc tính `$routeMiddleware` của file `app/Http/Kernel.php` của ứng dụng của bạn:

    'abilities' => \Laravel\Sanctum\Http\Middleware\CheckAbilities::class,
    'ability' => \Laravel\Sanctum\Http\Middleware\CheckForAnyAbility::class,

Middleware `abilities` có thể được gán cho một route để xác minh xem token của request đến có tất cả các quyền đã được liệt kê hay không:

    Route::get('/orders', function () {
        // Token has both "check-status" and "place-orders" abilities...
    })->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);

Middleware `ability` có thể được chỉ định cho một route để xác minh xem token của request đến có *ít nhất một* trong số các quyền đã được liệt kê hay không:

    Route::get('/orders', function () {
        // Token has the "check-status" or "place-orders" ability...
    })->middleware(['auth:sanctum', 'ability:check-status,place-orders']);

<a name="first-party-ui-initiated-requests"></a>
#### First-Party UI Initiated Requests

Để thuận tiện, phương thức `tokenCan` sẽ luôn trả về `true` nếu request cần được xác thực đến từ một SPA của bạn và bạn đang sử dụng [xác thực SPA](#spa-authentication) được tích hợp sẵn của Sanctum.

Tuy nhiên, điều này không nhất thiết có nghĩa là ứng dụng của bạn phải cho phép người dùng thực hiện hành động. Thông thường, [chính sách authorization](/docs/{{version}}/authorization#creating-policies) của ứng dụng của bạn sẽ xác định xem token có được cấp quyền để thực hiện các quyền này hay không cũng như kiểm tra xem bản thân instance người dùng có được phép hay không để thực hiện hành động.

Ví dụ: nếu chúng ta hãy tưởng tượng một ứng dụng quản lý máy chủ, điều này có thể có nghĩa là kiểm tra token đó có được phép cập nhật máy chủ **và** máy chủ đó có thuộc về người dùng đó hay không:

```php
return $request->user()->id === $server->user_id &&
       $request->user()->tokenCan('server:update')
```

Lúc đầu, việc cho phép gọi phương thức `tokenCan` và luôn trả về `true` cho các request do fontend của bạn khởi tạo có vẻ lạ; tuy nhiên, sẽ thật thuận tiện khi có thể luôn giả định rằng token API có sẵn và luôn có thể được kiểm tra thông qua phương thức `tokenCan`. Bằng cách thực hiện cách tiếp cận này, bạn luôn có thể gọi phương thức `tokenCan` trong chính sách authorization của ứng dụng của bạn mà không phải lo lắng về việc liệu request đó có được kích hoạt từ fontend của ứng dụng của bạn hay là đang được thực hiện bởi một trong những người dùng bên thứ ba của API của bạn.

<a name="protecting-routes"></a>
### Bảo vệ route

Để bảo vệ các route sao cho tất cả các request phải được xác thực, bạn nên gắn guard `sanctum` vào các route  bảo vệ của bạn trong file route `routes/web.php` và `routes/api.php`. Guard này sẽ đảm bảo rằng các request sẽ được xác thực bằng cookie hoặc chứa một header token API hợp lệ nếu request đến từ bên thứ ba.

Bạn có thể thắc mắc tại sao chúng tôi khuyên bạn nên xác thực các route trong file `routes/web.php` của ứng dụng bằng cách sử dụng guard `sanctum`. Hãy nhớ rằng, trước tiên Sanctum sẽ cố gắng xác thực các request đến bằng cách sử dụng cookie xác thực session thông thường của Laravel. Nếu cookie đó không xuất hiện thì Sanctum sẽ cố gắng xác thực request bằng cách sử dụng token trong header `Authorization` của request. Ngoài ra, việc xác thực tất cả request bằng Sanctum sẽ đảm bảo rằng chúng ta luôn có thể gọi phương thức `tokenCan` trên instance người dùng hiện đã được xác thực:

    use Illuminate\Http\Request;

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="revoking-tokens"></a>
### Thu hồi token

Bạn có thể "thu hồi" token bằng cách xóa chúng ra khỏi cơ sở dữ liệu của bạn bằng cách sử dụng quan hệ `tokens` được cung cấp bởi trait `Laravel\Sanctum\HasApiTokens`:

    // Revoke all tokens...
    $user->tokens()->delete();

    // Revoke the token that was used to authenticate the current request...
    $request->user()->currentAccessToken()->delete();

    // Revoke a specific token...
    $user->tokens()->where('id', $tokenId)->delete();

<a name="spa-authentication"></a>
## SPA Authentication

Sanctum cũng cung cấp một phương thức đơn giản để xác thực các ứng dụng single page (SPAs) cần giao tiếp với một API được hỗ trợ bởi Laravel. Các ứng dụng SPAs này có thể tồn tại trong cùng một repository như một ứng dụng Laravel của bạn hoặc có thể là một repository hoàn toàn khác riêng biệt.

Đối với tính năng này, Sanctum không sử dụng bất kỳ loại token nào. Thay vào đó, Sanctum sử dụng các service xác thực session dựa trên cookie được tích hợp sẵn trong Laravel. Cách xác thực này cung cấp các lợi ích về bảo vệ CSRF, xác thực session, cũng như bảo vệ chống rò rỉ thông tin xác thực thông qua XSS.

> {note} Để xác thực, SPA và API của bạn phải chia sẻ cùng một tên miền. Tuy nhiên, chúng có thể được set trên các subdomain khác nhau. Additionally, you should ensure that you send the `Accept: application/json` header with your request.


<a name="spa-configuration"></a>
### Cấu hình

<a name="configuring-your-first-party-domains"></a>
#### Configuring Your First-Party Domains

Đầu tiên, bạn nên cấu hình các tên miền mà SPA của bạn sẽ thực hiện request từ đó. Bạn có thể cấu hình các tên miền này bằng cách sử dụng tùy chọn cấu hình `stateful` trong file cấu hình `sanctum` của bạn. Cài đặt cấu hình này sẽ xác định xem tên miền nào sẽ duy trì "trạng thái" xác thực bằng cách sử dụng session cookie Laravel khi tạo request tới API của bạn.

> {note} Nếu bạn đang truy cập ứng dụng của bạn thông qua URL có cổng (`127.0.0.1:8000`), bạn nên đảm bảo là bạn đã cấu hình cả số cổng với tên miền.

<a name="sanctum-middleware"></a>
#### Sanctum Middleware

Tiếp theo, bạn nên thêm middleware của Sanctum vào group middleware `api` trong file `app/Http/Kernel.php` của bạn. Middleware này sẽ chịu trách nhiệm đảm bảo rằng các request đến từ các SPA của bạn có thể được xác thực bằng session cookie của Laravel, trong khi vẫn cho phép các request từ bên thứ ba hoặc ứng dụng di động xác thực bằng cách sử dụng API token:

    'api' => [
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

<a name="cors-and-cookies"></a>
#### CORS & Cookies

Nếu bạn gặp sự cố khi xác thực ứng dụng của bạn từ một SPA chạy trên một subdomain riêng biệt, có thể bạn đã cấu hình sai cài đặt CORS (Cross-Origin Resource Sharing) hoặc session cookie của bạn.

Bạn nên đảm bảo là cấu hình CORS của ứng dụng của bạn đang trả về header `Access-Control-Allow-Credentials` có giá trị là `True`. Nó có thể hoàn thành bằng cách set tùy chọn `supports_credentials` trong file cấu hình `config/cors.php` của ứng dụng thành `true`.

Ngoài ra, bạn cũng nên thêm tùy chọn `withCredentials` trên instance global `axios` của application của bạn. Thông thường, điều này sẽ được thực hiện trong file `resources/js/bootstrap.js` của bạn. Nếu bạn không sử dụng Axios để thực hiện các request HTTP từ fontend của bạn, bạn nên thực hiện cấu hình tương đương trên HTTP client của riêng bạn:

    axios.defaults.withCredentials = true;

Cuối cùng, bạn nên đảm bảo cấu hình session cookie của têm miền trong ứng dụng hỗ trợ tất cả các subdomain của tên miền gốc. Bạn có thể hoàn thành việc này bằng cách set thêm tiền tố dấu `.` đứng trước tên miền bằng trong file cấu hình `config/session.php` của application của bạn:

    'domain' => '.domain.com',

<a name="spa-authenticating"></a>
### Authenticating

<a name="csrf-protection"></a>
#### CSRF Protection

Để xác thực SPA của bạn, trước tiên, trang đăng nhập của SPA của bạn phải thực hiện một request đến route `/sanctum/csrf-cookie` để khởi tạo bảo vệ CSRF cho ứng dụng:

    axios.get('/sanctum/csrf-cookie').then(response => {
        // Login...
    });

Trong request này, Laravel sẽ set cookie `XSRF-TOKEN` chứa token CSRF hiện tại. Token này sau đó sẽ được truyền vào trong header `X-XSRF-TOKEN` trong các request tiếp theo, đối với các thư viện HTTP client như Axios và Angular HttpClient sẽ tự động thực hiện điều này cho bạn. Nếu thư viện JavaScript HTTP của bạn không set giá trị này cho bạn, bạn sẽ cần phải tự set header `X-XSRF-TOKEN` để khớp với giá trị của cookie `XSRF-TOKEN` được set theo route này.

<a name="logging-in"></a>
#### Logging In

Khi bảo vệ CSRF đã được khởi tạo, bạn nên thực hiện một request `POST` đến route `/login` của Laravel application của bạn. Route `/login` này có thể [được làm theo cách thủ công](/docs/{{version}}/authentication#authenticating-users) hoặc sử dụng package xác thực không có giao diện như [Laravel Fortify](/docs/{{version}}/fortify).

Nếu request đăng nhập thành công, bạn sẽ được xác thực và các request tiếp theo đối với các route của application của bạn sẽ tự động xác thực thông qua session cookie, cái mà backend Laravel application đã cung cấp cho client của bạn. Ngoài ra, vì ứng dụng của bạn đã tạo ra request tới route `/sanctum/csrf-cookie` nên các request tiếp theo sẽ tự động nhận được bảo vệ CSRF miễn là client JavaScript HTTP của bạn gửi giá trị của cookie `XSRF-TOKEN` vào trong header `X-XSRF-TOKEN`.

Tất nhiên, nếu session người dùng của bạn hết hạn do không hoạt động, thì các request tiếp theo tới ứng dụng Laravel có thể nhận được response lỗi HTTP 401 hoặc 419. Trong trường hợp này, bạn nên chuyển hướng người dùng đến trang đăng nhập SPA của bạn.

> {note} Bạn có thể tự do thoải mái viết bất kỳ endpoint `/login` nào của riêng bạn; tuy nhiên, bạn nên đảm bảo rằng nó xác thực người dùng bằng cách sử dụng tiêu chuẩn [dịch vụ xác thực dựa trên session mà Laravel cung cấp](/docs/{{version}}/authentication#authenticating-users). Thông thường, điều này có nghĩa là sử dụng guard authentication `web`.

<a name="protecting-spa-routes"></a>
### Bảo vệ route

Để bảo vệ các route sao cho tất cả các request phải được xác thực, bạn nên gắn guard `sanctum` vào các route API của bạn trong file `routes/api.php`. Guard này sẽ đảm bảo rằng các request sẽ được xác thực trạng thái từ SPA của bạn hoặc chứa một header API token hợp lệ nếu request đến từ bên thứ ba:

    use Illuminate\Http\Request;

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="authorizing-private-broadcast-channels"></a>
### Authorizing Private Broadcast Channels

Nếu SPA của bạn cần xác thực với [các channel private / presence broadcast](/docs/{{version}}/broadcasting#authorizing-channels), bạn nên gọi phương thức `Broadcast::routes` trong file `routes/api.php` của bạn:

    Broadcast::routes(['middleware' => ['auth:sanctum']]);

Tiếp theo, để các authorization request của Pusher thành công, bạn sẽ cần phải cung cấp một tùy chỉnh `authorizer` của Pusher khi khởi tạo [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation). Điều này cho phép ứng dụng của bạn cấu hình Pusher để sử dụng một instance `axios` được [cấu hình đúng cho các request cross-domain](#cors-and-cookies):

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

Bạn cũng có thể sử dụng token Sanctum để xác thực các request của ứng dụng di động đối với API của bạn. Quy trình xác thực các request ứng dụng di động tương tự như xác thực các request API của bên thứ ba; tuy nhiên, có những khác biệt nhỏ về cách bạn sẽ phát hành ra các API token.

<a name="issuing-mobile-api-tokens"></a>
### Phát hành API Token

Để bắt đầu, hãy tạo một route chấp nhận email hoặc tên người dùng, mật khẩu và tên thiết bị của người dùng, sau đó kiểm tra các thông tin đăng nhập đó để lấy token Sanctum mới. "Tên thiết bị" được cung cấp cho route này nhằm mục đích cung cấp thông tin và có thể là bất kỳ giá trị nào bạn muốn. Nói chung, giá trị tên thiết bị phải là tên mà người dùng có thể nhận ra, chẳng hạn như "iPhone 12 của Nuno".

Thông thường, bạn sẽ tạo một request tới route token từ màn hình "đăng nhập" ứng dụng di động của bạn. Route sẽ trả về một token Sanctum thật để có thể được lưu trên thiết bị di động và được sử dụng để thực hiện thêm các request API sau đó:

    use App\Models\User;
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

Khi ứng dụng di động sử dụng token để thực hiện một request API đối với application của bạn, ứng dụng đó sẽ truyền token vào trong header `Authorization` dưới dạng một token `Bearer`.

> {mẹo} Khi phát hành token cho ứng dụng di động, bạn cũng có thể tự do chỉ định [các quyền cho token](#token-abilities).

<a name="protecting-mobile-api-routes"></a>
### Bảo vệ route

Như đã được ghi ở trước đó, bạn có thể bảo vệ các route để tất cả các request đến phải được xác thực bằng cách gắn guard `sanctum` vào các route.

    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="revoking-mobile-api-tokens"></a>
### Thu hồi token

Để cho phép người dùng thu hồi token API đã cấp cho thiết bị di động, bạn có thể liệt kê chúng theo tên, cùng với nút "thu hồi", trong phần "cài đặt tài khoản" trong giao diện web người dùng ứng dụng. Khi người dùng nhấp vào nút "thu hồi", bạn có thể xóa token ra khỏi cơ sở dữ liệu. Hãy nhớ rằng, bạn có thể truy cập vào token API của người dùng thông qua quan hệ `tokens` được cung cấp sẵn bởi trait `Laravel\Sanctum\HasApiTokens`:

    // Revoke all tokens...
    $user->tokens()->delete();

    // Revoke a specific token...
    $user->tokens()->where('id', $tokenId)->delete();

<a name="testing"></a>
## Testing

Trong khi testing, phương thức `Sanctum::actingAs` có thể được sử dụng để xác thực một người dùng và chỉ định quyền nào sẽ được cấp cho token của họ:

    use App\Models\User;
    use Laravel\Sanctum\Sanctum;

    public function test_task_list_can_be_retrieved()
    {
        Sanctum::actingAs(
            User::factory()->create(),
            ['view-tasks']
        );

        $response = $this->get('/api/task');

        $response->assertOk();
    }

Nếu bạn muốn cấp tất cả các quyền cho một token, bạn nên thêm dấu `*` vào trong danh sách các quyền được cung cấp cho phương thức `actingAs`:

    Sanctum::actingAs(
        User::factory()->create(),
        ['*']
    );
