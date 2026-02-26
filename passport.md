# Laravel Passport

- [Giới thiệu](#introduction)
    - [Passport hay Sanctum?](#passport-or-sanctum)
- [Cài đặt](#installation)
    - [Deploy Passport](#deploying-passport)
    - [Cập nhật Passport](#upgrading-passport)
- [Cấu hình](#configuration)
    - [Thời gian sống token](#token-lifetimes)
    - [Ghi đè các model mặc định](#overriding-default-models)
    - [Ghi đè routes](#overriding-routes)
- [Authorization Code Grant](#authorization-code-grant)
    - [Quản lý client](#managing-clients)
    - [Request token](#requesting-tokens)
    - [Quản lý token](#managing-tokens)
    - [Refresh token](#refreshing-tokens)
    - [Thu hồi token](#revoking-tokens)
    - [Lọc token](#purging-tokens)
- [Authorization Code Grant với PKCE](#code-grant-pkce)
    - [Tạo client](#creating-a-auth-pkce-grant-client)
    - [Request token](#requesting-auth-pkce-grant-tokens)
- [Device Authorization Grant](#device-authorization-grant)
    - [Tạo một Device Code Grant Client](#creating-a-device-authorization-grant-client)
    - [Request token](#requesting-device-authorization-grant-tokens)
- [Password Grant](#password-grant)
    - [Tạo một password grant client](#creating-a-password-grant-client)
    - [Request token](#requesting-password-grant-tokens)
    - [Request all scope](#requesting-all-scopes)
    - [Tuỳ biến User Provider](#customizing-the-user-provider)
    - [Tuỳ biến field username](#customizing-the-username-field)
    - [Tuỳ biến Password Validation](#customizing-the-password-validation)
- [Grant ẩn](#implicit-grant)
- [Chứng chỉ client grant](#client-credentials-grant)
- [Access token cá nhân](#personal-access-tokens)
    - [Tạo một access client cá nhân](#creating-a-personal-access-client)
    - [Tuỳ chỉnh User Provider](#customizing-the-user-provider-for-pat)
    - [Quản lý access token cá nhân](#managing-personal-access-tokens)
- [Bảo vệ route](#protecting-routes)
    - [Thông qua middleware](#via-middleware)
    - [Pass access token](#passing-the-access-token)
- [Token scope](#token-scopes)
    - [Định nghĩa scope](#defining-scopes)
    - [Scope mặc định](#default-scope)
    - [Gán scope đến token](#assigning-scopes-to-tokens)
    - [Kiểm tra scope](#checking-scopes)
- [SPA Authentication](#spa-authentication)
- [Event](#events)
- [Test](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Passport](https://github.com/laravel/passport) cung cấp một implementation OAuth2 server đầy đủ cho application Laravel của bạn trong vài phút. Passport được xây dựng trên top của [League OAuth2 server](https://github.com/thephpleague/oauth2-server) được duy trì bởi Andy Millington và Simon Hamp.

> [!NOTE]
> Tài liệu này giả định rằng bạn đã biết OAuth2. Nếu bạn chưa biết về OAuth2, hãy xem xét việc tự học với các [thuật ngữ](https://oauth2.thephpleague.com/terminology/) và tính năng chung của OAuth2 trước khi tiếp tục.

<a name="passport-or-sanctum"></a>
### Passport hay Sanctum?

Trước khi bắt đầu, bạn có thể muốn xem xét xem ứng dụng của bạn sẽ được phục vụ tốt hơn bởi Laravel Passport hay [Laravel Sanctum](/docs/{{version}}/sanctum). Nếu ứng dụng của bạn thực sự cần hỗ trợ OAuth2 thì bạn nên sử dụng Laravel Passport.

Tuy nhiên, nếu bạn đang làm xác thực cho một ứng dụng single-page, mobile application hoặc phát hành API token, bạn nên sử dụng [Laravel Sanctum](/docs/{{version}}/sanctum). Laravel Sanctum không hỗ trợ OAuth2; tuy nhiên, nó cung cấp trải nghiệm phát triển xác thực API đơn giản hơn nhiều.

<a name="installation"></a>
## Cài đặt

Bạn có thể cài đặt Laravel Passport thông qua lệnh `install:api` của Artisan:

```shell
php artisan install:api --passport
```

Lệnh này sẽ publish và chạy các migration cơ sở dữ liệu cần thiết để tạo ra các bảng mà ứng dụng của bạn cần để lưu các client OAuth2 và access token. Lệnh này cũng sẽ tạo các khóa encryption cần thiết để tạo các access token an toàn.

Sau khi chạy lệnh `install:api`, hãy thêm trait `Laravel\Passport\HasApiTokens` và interface `Laravel\Passport\Contracts\OAuthenticatable` vào model `App\Models\User` của bạn. Trait này sẽ cung cấp một số phương thức helper cho model của bạn, cho phép bạn kiểm tra token và scope của người dùng đã được xác thực:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Cuối cùng, trong file cấu hình `config/auth.php` của application của bạn, bạn nên định nghĩa một guard xác thực `api` và thiết lập tùy chọn `driver` thành `passport`. Điều này sẽ hướng dẫn application của bạn sử dụng `TokenGuard` của Passport khi authenticate các request API:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

<a name="deploying-passport"></a>
### Deploying Passport

Khi deploy Passport lần đầu đến server application của bạn, bạn có thể sẽ cần chạy lệnh `passport:keys`. Lệnh này sẽ tạo các key mã hóa Passport cần, để tạo access token. Các key được tạo thường không nên được lưu trữ trong source code control:

```shell
php artisan passport:keys
```

Nếu cần, bạn có thể định nghĩa đường dẫn nơi mà các khóa của Passport sẽ được load từ đó. Bạn có thể sử dụng phương thức `Passport::loadKeysFrom` để thực hiện việc này. Thông thường, phương thức này phải được gọi từ phương thức `boot` của class `App\Providers\AppServiceProvider` trong ứng dụng của bạn:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
}
```

<a name="loading-keys-from-the-environment"></a>
#### Loading Keys From The Environment

Ngoài ra, bạn có thể export file cấu hình của Passport bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag=passport-config
```

Sau khi file cấu hình được export, bạn có thể load khóa mã hóa của ứng dụng bằng cách định nghĩa chúng dưới dạng biến môi trường:

```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

<a name="upgrading-passport"></a>
### Cập nhật Passport

Khi nâng cấp lên phiên bản mới của Passport, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/passport/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Cấu hình

<a name="token-lifetimes"></a>
### Thời gian sống token

Mặc định, Passport phát hành các access token tồn tại lâu dài có thời hạn một năm. Nếu bạn muốn cấu hình vòng đời token dài hoặc ngắn hơn, bạn có thể sử dụng các phương thức `tokensExpireIn`, `refreshTokensExpireIn`, và `personalAccessTokensExpireIn`. Các phương thức này phải được gọi từ phương thức `boot` của class `App\Providers\AppServiceProvider` của application:

```php
use Carbon\CarbonInterval;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensExpireIn(CarbonInterval::days(15));
    Passport::refreshTokensExpireIn(CarbonInterval::days(30));
    Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));
}
```

> [!WARNING]
> Các cột `expires_at` trong bảng cơ sở dữ liệu Passport sẽ ở chế độ chỉ-đọc và chỉ dành cho mục đích hiển thị. Khi phát hành token, Passport sẽ lưu trữ thông tin hết hạn vào trong các token đó và mã hóa chúng. Nếu bạn muốn làm mất hiệu lực token, bạn nên [thu hồi nó](#revoking-tokens).

<a name="overriding-default-models"></a>
### Ghi đè các model mặc định

Bạn có thể thoải mái mở rộng các model được sử dụng trong nội bộ Passport bằng cách định nghĩa model của riêng bạn và extend model Passport tương ứng:

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

Sau khi định nghĩa xong model của bạn, bạn có thể hướng dẫn Passport sử dụng các model tùy biến này thông qua `Laravel\Passport\Passport` class. Thông thường, bạn nên thông báo cho Passport biết về các model tùy chỉnh của bạn trong phương thức `boot` của class `App\Providers\AppServiceProvider` trong ứng dụng của bạn:

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\DeviceCode;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::useTokenModel(Token::class);
    Passport::useRefreshTokenModel(RefreshToken::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::useClientModel(Client::class);
    Passport::useDeviceCodeModel(DeviceCode::class);
}
```

<a name="overriding-routes"></a>
### Ghi đè routes

Thỉnh thoảng bạn có thể muốn tùy chỉnh các route được định nghĩa bởi Passport. Để thực hiện điều này, trước tiên bạn cần bỏ qua các route được Passport đăng ký bằng cách thêm `Passport::ignoreRoutes` vào phương thức `register` của `AppServiceProvider` của ứng dụng:

```php
use Laravel\Passport\Passport;

/**
 * Register any application services.
 */
public function register(): void
{
    Passport::ignoreRoutes();
}
```

Sau đó, bạn có thể copy các route được Passport định nghĩa trong [file route](https://github.com/laravel/passport/blob/master/routes/web.php) vào file `routes/web.php` của ứng dụng và sửa chúng theo ý thích của bạn:

```php
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Passport routes...
});
```

<a name="authorization-code-grant"></a>
## Authorization Code Grant

Sử dụng OAuth2 thông qua authorization code là cách mà hầu hết các nhà phát triển quen thuộc với OAuth2. Khi sử dụng authorization code, một client application sẽ chuyển hướng người dùng đến server của bạn, nơi mà người dùng sẽ chấp nhận hoặc từ chối request cấp access token cho client.

Để bắt đầu, chúng ta cần hướng dẫn Passport cách mà Passport sẽ trả về view "authorization" của chúng ta.

Tất cả logic render của view authorization có thể được tùy chỉnh bằng cách sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Passport\Passport`. Thông thường, bạn nên gọi phương thức này từ trong phương thức `boot` của class `App\Providers\AppServiceProvider` trong ứng dụng của bạn:

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // By providing a view name...
    Passport::authorizationView('auth.oauth.authorize');

    // By providing a closure...
    Passport::authorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );
}
```

Passport sẽ tự động định nghĩa route `/oauth/authorize` để trả về view này. Template `auth.oauth.authorize` của bạn nên chứa một form gửi một POST request đến route `passport.authorizations.approve` để chấp nhận cấp quyền và một form gửi một DELETE request đến route `passport.authorizations.deny` để từ chối cấp quyền. Các route `passport.authorizations.approve` và `passport.authorizations.deny` sẽ yêu cầu các field `state`, `client_id`, và `auth_token`.

<a name="managing-clients"></a>
### Quản lý client

Các nhà phát triển cần xây dựng các ứng dụng của họ, mà cần tương tác với API application của bạn, họ sẽ cần phải đăng ký ứng dụng của họ với application của bạn bằng cách tạo "client". Thông thường, việc này bao gồm việc cung cấp tên ứng dụng và URI mà application của bạn cần chuyển hướng người dùng đến sau khi người dùng đã chấp thuận request ủy quyền.

<a name="managing-first-party-clients"></a>
#### First-Party Clients

Cách đơn giản nhất để tạo một client là sử dụng lệnh Artisan `passport:client`. Lệnh này có thể được sử dụng cho các client bên thứ nhất hoặc test các chức năng OAuth2. Khi bạn chạy lệnh `passport:client`, Passport sẽ hỏi bạn cho biết thêm thông tin về client của bạn và trả về cho bạn một client ID và một secret:

```shell
php artisan passport:client
```

Nếu bạn muốn lập một danh sách cho phép nhiều URL chuyển hướng cho client của bạn, bạn có thể chỉ định chúng bằng cách sử dụng một danh sách được phân cách bằng dấu phẩy khi nhập URL bằng lệnh `passport:client`. Bất kỳ URL nào chứa dấu phẩy đều phải được encode URL:

```shell
https://third-party-app.com/callback,https://example.com/oauth/redirect
```

<a name="managing-third-party-clients"></a>
#### Third-Party Clients

Vì người dùng application của bạn sẽ không thể sử dụng lệnh `passport:client`, bạn có thể sử dụng phương thức `createAuthorizationCodeGrantClient` của class `Laravel\Passport\ClientRepository` để đăng ký một client cho một người dùng cụ thể:

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

// Creating an OAuth app client that belongs to the given user...
$client = app(ClientRepository::class)->createAuthorizationCodeGrantClient(
    user: $user,
    name: 'Example App',
    redirectUris: ['https://third-party-app.com/callback'],
    confidential: false,
    enableDeviceFlow: true
);

// Retrieving all the OAuth app clients that belong to the user...
$clients = $user->oauthApps()->get();
```

Phương thức `createAuthorizationCodeGrantClient` sẽ trả về một instance của `Laravel\Passport\Client`. Bạn có thể hiển thị `$client->id` như là client ID và `$client->plainSecret` như là client secret cho người dùng.

<a name="requesting-tokens"></a>
### Request token

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### Redirecting For Authorization

Khi một client đã được tạo, các developer có thể sử dụng client ID và secret được trả về để yêu cầu authorization code và access token từ application của bạn. Đầu tiên, application của bên thứ ba sẽ tạo một yêu cầu chuyển hướng đến route `/oauth/authorize` của application của bạn như sau:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

Tham số `prompt` có thể được sử dụng để chỉ định loại xác thực của ứng dụng Passport.

Nếu giá trị `prompt` là `none`, Passport sẽ luôn đưa ra lỗi xác thực nếu người dùng chưa được xác thực với ứng dụng Passport. Nếu giá trị là `consent`, Passport sẽ luôn hiển thị màn hình chấp nhận authorization, ngay cả khi tất cả các scope đã được cấp quyền trước đó cho ứng dụng sử dụng. Khi giá trị là `login`, ứng dụng Passport sẽ luôn nhắc người dùng phải đăng nhập lại vào ứng dụng, ngay cả khi họ đã tồn tại trong session.

Nếu không cung cấp giá trị `prompt`, người dùng sẽ chỉ được nhắc là cấp quyền nếu trước đó họ chưa cấp quyền truy cập cho ứng dụng đang sử dụng với các scope được yêu cầu.

> [!NOTE]
> Hãy nhớ rằng route `/oauth/authorize` đã được Passport định nghĩa. Bạn không cần phải tự định nghĩa route này nữa.

<a name="approving-the-request"></a>
#### Approving The Request

Khi nhận được authorization request, Passport sẽ tự động phản hồi dựa trên giá trị của tham số `prompt` (nếu có) và có thể hiển thị một template cho người dùng để họ có thể chấp nhận hoặc từ chối authorization request. Nếu họ chấp nhận request, họ sẽ được chuyển hướng trở lại `redirect_uri` sẽ được chỉ định bởi application của bên thứ ba. `redirect_uri` phải khớp với URL `redirect` được chỉ định khi client được tạo.

Thỉnh thoảng bạn có thể muốn bỏ qua các lời nhắc cấp quyền, chẳng hạn như khi cấp quyền cho client bên thứ nhất. Bạn có thể thực hiện điều này bằng cách [extend model `Client`](#overriding-default-models) và định nghĩa phương thức `skipsAuthorization`. Nếu `skipsAuthorization` trả về `true` thì ứng dụng client sẽ được chấp thuận và người dùng sẽ được chuyển hướng trở lại về `redirect_uri` ngay lập tức, trừ khi ứng dụng đang sử dụng đó đã thiết lập tham số `prompt` khi chuyển hướng để xác thực:

```php
<?php

namespace App\Models\Passport;

use Illuminate\Contracts\Auth\Authenticatable;
use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * Determine if the client should skip the authorization prompt.
     *
     * @param  \Laravel\Passport\Scope[]  $scopes
     */
    public function skipsAuthorization(Authenticatable $user, array $scopes): bool
    {
        return $this->firstParty();
    }
}
```

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### Converting Authorization Codes To Access Tokens

Nếu người dùng chấp nhận authorization request, họ sẽ được chuyển hướng trở lại application của bên thứ ba. Sau đó, đầu tiên, bên thứ ba sẽ kiểm tra tham số `state` với giá trị đã được lưu trữ trước khi chuyển hướng. Nếu tham số state trùng khớp với giá trị đã được lưu, nó sẽ đưa ra một request `POST` cho application của bạn để yêu cầu access token. Yêu cầu phải chứa authorization code được cấp bởi application của bạn khi người dùng chấp nhận authorization request. Trong ví dụ này, chúng ta sẽ sử dụng thư viện Guzzle HTTP để thực hiện request `POST`:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class,
        'Invalid state value.'
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

Route `/oauth/token` này sẽ trả về một JSON response có chứa các thuộc tính `access_token`, `refresh_token` và `expires_in`. Thuộc tính `expires_in` sẽ chứa số giây cho đến khi access token hết hạn.

> [!NOTE]
> Giống như route `/oauth/authorize`, route `/oauth/token` đã được định nghĩa cho bạn bằng Passport. Bạn không cần phải tự định nghĩa route này.

<a name="managing-tokens"></a>
### Managing Tokens

Bạn có thể lấy ra các token đã được ủy quyền của người dùng bằng cách sử dụng phương thức `tokens` của trait `Laravel\Passport\HasApiTokens`. Ví dụ, điều này có thể được sử dụng để cung cấp cho người dùng của bạn một dashboard để theo dõi các kết nối của họ với các ứng dụng bên thứ ba:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Retrieving all of the valid tokens for the user...
$tokens = $user->tokens()
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get();

// Retrieving all the user's connections to third-party OAuth app clients...
$connections = $tokens->load('client')
    ->reject(fn (Token $token) => $token->client->firstParty())
    ->groupBy('client_id')
    ->map(fn (Collection $tokens) => [
        'client' => $tokens->first()->client,
        'scopes' => $tokens->pluck('scopes')->flatten()->unique()->values()->all(),
        'tokens_count' => $tokens->count(),
    ])
    ->values();
```

<a name="refreshing-tokens"></a>
### Refresh token

Nếu application của bạn phát hành access token ngắn hạn, người dùng sẽ cần phải refresh access token của họ thông qua refresh token được cung cấp cho họ khi access token được phát hành:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

Route `/oauth/token` này sẽ trả về một JSON response có chứa các thuộc tính `access_token`, `refresh_token` và `expires_in`. Thuộc tính `expires_in` sẽ chứa số giây cho đến khi access token hết hạn.

<a name="revoking-tokens"></a>
### Thu hồi Tokens

Bạn có thể thu hồi một token cách sử dụng phương thức `revoke` trên model `Laravel\Passport\Token`. Bạn có thể thu hồi các refresh của một token bằng cách sử dụng phương thức `revoke` trên model `Laravel\Passport\RefreshToken`.

```php
use Laravel\Passport\Passport;
use Laravel\Passport\Token;

$token = Passport::token()->find($tokenId);

// Revoke an access token...
$token->revoke();

// Revoke the token's refresh token...
$token->refreshToken?->revoke();

// Revoke all of the user's tokens...
User::find($userId)->tokens()->each(function (Token $token) {
    $token->revoke();
    $token->refreshToken?->revoke();
});
```

<a name="purging-tokens"></a>
### Lọc token

Khi token bị thu hồi hoặc bị hết hạn, bạn có thể muốn xóa chúng ra khỏi cơ sở dữ liệu. Passport có kèm theo một lệnh Artisan `passport:purge` có thể thực hiện việc này cho bạn:

```shell
# Purge revoked and expired tokens, auth codes, and device codes...
php artisan passport:purge

# Only purge tokens expired for more than 6 hours...
php artisan passport:purge --hours=6

# Only purge revoked tokens, auth codes, and device codes...
php artisan passport:purge --revoked

# Only purge expired tokens, auth codes, and device codes...
php artisan passport:purge --expired
```

Bạn cũng có thể cấu hình một [scheduled job](/docs/{{version}}/scheduling) trong file `routes/console.php` của application của bạn để tự động lọc token của bạn theo một schedule:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('passport:purge')->hourly();
```

<a name="code-grant-pkce"></a>
## Authorization Code Grant với PKCE

Việc Authorization Code grant với "Proof Key for Code Exchange" (PKCE) là một cách an toàn để xác thực các trang web hoặc các ứng dụng mobile truy cập vào API của bạn. Grant này sẽ được sử dụng khi bạn không thể đảm bảo rằng client secret sẽ được lưu trữ một cách an toàn hoặc cũng có thể là để giảm thiểu nguy cơ bị kẻ tấn công chặn authorization code. Sự kết hợp giữa một "code verifier" và một "code challenge" sẽ thay thế client secret khi trao đổi authorization code để lấy một access token.

<a name="creating-a-auth-pkce-grant-client"></a>
### Tạo client

Trước khi ứng dụng của bạn có thể phát hành token thông qua authorization code grant với PKCE, bạn sẽ cần tạo một ứng dụng client hỗ trợ PKCE. Bạn có thể thực hiện việc này bằng lệnh Artisan `passport:client` với tùy chọn `--public`:

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### Request token

<a name="code-verifier-code-challenge"></a>
#### Code Verifier và Code Challenge

Vì authorization grant này không cung cấp một client secret, nên các nhà phát triển sẽ cần phải tạo ra một code verifier và một code challenge để yêu cầu token.

Code verifier phải là một chuỗi ngẫu nhiên từ 43 đến 128 ký tự chứa các chữ cái, số, và các ký tự `"-"`, `"."`, `"_"`, `"~"`, như được định nghĩa trong [tài liệu RFC 7636 đặc điểm kỹ thuật](https://tools.ietf.org/html/rfc7636).

Code challenge phải là một chuỗi được mã hóa Base64 với URL và các ký tự an toàn cho tên file. Các ký tự ở cuối dấu `'='` phải được loại bỏ và không được có dấu ngắt dòng, khoảng trắng hoặc các ký tự bổ sung khác.

```php
$encoded = base64_encode(hash('sha256', $codeVerifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### Redirecting For Authorization

Sau khi một ứng dụng client đã được tạo xong, bạn có thể sử dụng ID của ứng dụng client đó và code verifier và code challenge đã tạo để yêu cầu một authorization code và một access token từ ứng dụng của bạn. Đầu tiên, ứng dụng đang sử dụng phải thực hiện một request chuyển hướng đến route `/oauth/authorize` của ứng dụng của bạn:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put(
        'code_verifier', $codeVerifier = Str::random(128)
    );

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $codeVerifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### Converting Authorization Codes To Access Tokens

Nếu người dùng chấp thuận yêu cầu authorization, họ sẽ được chuyển hướng trở lại ứng dụng mà họ đang sử dụng. Người dùng api của bạn nên xác thực thông số `state` so với giá trị đã được lưu trữ trước khi được chuyển hướng, như trong Authorization Code Grant tiêu chuẩn.

Nếu thông số state khớp, Người dùng api của bạn nên đưa ra một request `POST` cho ứng dụng của bạn để yêu cầu một access token. Yêu cầu này phải chứa authorization code do ứng dụng của bạn cấp khi người dùng chấp thuận yêu cầu authorization cùng với code verifier đã được tạo ra ban đầu:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    $codeVerifier = $request->session()->pull('code_verifier');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code_verifier' => $codeVerifier,
        'code' => $request->code,
    ]);

    return $response->json();
});
```

<a name="device-authorization-grant"></a>
## Device Authorization Grant

Cấp phép authorization device OAuth2 cho phép các thiết bị không có trình duyệt hoặc bị hạn chế nhập liệu, chẳng hạn như TV hoặc máy chơi game console, sẽ lấy một access token bằng cách trao đổi một "device code". Khi sử dụng device flow, device client sẽ hướng dẫn người dùng sử dụng một thiết bị thứ hai, chẳng hạn như máy tính hoặc điện thoại thông minh có kết nối với server của bạn, nơi mà họ sẽ nhập "user code" được cung cấp để chấp nhận hoặc từ chối yêu cầu truy cập.

Để bắt đầu, chúng ta cần hướng dẫn Passport trả về các view "user code" và "authorization".

Tất cả logic hiển thị của view authorization có thể được tùy chỉnh bằng các phương thức tương ứng thông qua class `Laravel\Passport\Passport`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` trong class `App\Providers\AppServiceProvider` của ứng dụng.

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // By providing a view name...
    Passport::deviceUserCodeView('auth.oauth.device.user-code');
    Passport::deviceAuthorizationView('auth.oauth.device.authorize');

    // By providing a closure...
    Passport::deviceUserCodeView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/UserCode')
    );

    Passport::deviceAuthorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );

    // ...
}
```

Passport sẽ tự động định nghĩa các route để trả về các view này. Template `auth.oauth.device.user-code` của bạn nên chứa một form thực hiện một request GET đến route `passport.device.authorizations.authorize`. Route `passport.device.authorizations.authorize` này sẽ yêu cầu một tham số query `user_code`.

Template `auth.oauth.device.authorize` của bạn nên chứa một form thực hiện một request POST đến route `passport.device.authorizations.approve` để chấp nhận request cấp quyền và một form thực hiện một request DELETE đến route `passport.device.authorizations.deny` để từ chối request cấp quyền. Các route `passport.device.authorizations.approve` và `passport.device.authorizations.deny` này sẽ yêu cầu các field `state`, `client_id`, và `auth_token`.

<a name="creating-a-device-authorization-grant-client"></a>
### Creating a Device Authorization Grant Client

Trước khi ứng dụng của bạn có thể cấp phát token thông qua device authorization grant, bạn sẽ cần tạo một client hỗ trợ device flow. Bạn có thể thực hiện việc này bằng lệnh Artisan `passport:client` với tùy chọn `--device`. Lệnh này sẽ tạo ra một client first-party hỗ trợ device flow và cung cấp cho bạn một client ID cùng client secret:

```shell
php artisan passport:client --device
```

Ngoài ra, bạn có thể sử dụng phương thức `createDeviceAuthorizationGrantClient` trong class `ClientRepository` để đăng ký một client third-party cho một người dùng cụ thể:

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

$client = app(ClientRepository::class)->createDeviceAuthorizationGrantClient(
    user: $user,
    name: 'Example Device',
    confidential: false,
);
```

<a name="requesting-device-authorization-grant-tokens"></a>
### Requesting Tokens

<a name="device-code"></a>
#### Requesting a Device Code

Khi một client đã được tạo, các nhà phát triển có thể sử dụng client ID của họ để yêu cầu device code từ ứng dụng của bạn. Đầu tiên, thiết bị yêu cầu nên thực hiện một request `POST` tới route `/oauth/device/code` của ứng dụng để yêu cầu device code:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [
    'client_id' => 'your-client-id',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

Việc này sẽ trả về một JSON response chứa các thuộc tính `device_code`, `user_code`, `verification_uri`, `interval`, và `expires_in`. Thuộc tính `expires_in` sẽ chứa thời gian device code sẽ bị hết hạn. Thuộc tính `interval` chứa số giây mà thiết bị yêu cầu nên đợi giữa các request khi thực hiện polling tới route `/oauth/token` để tránh các lỗi rate limit.

> [!NOTE]
> Hãy nhớ rằng, route `/oauth/device/code` đã được Passport định nghĩa. Bạn không cần phải tự định nghĩa route này.

<a name="user-code"></a>
#### Displaying the Verification URI and User Code

Khi đã nhận được device code, thiết bị yêu cầu nên hướng dẫn người dùng sử dụng một thiết bị khác, truy cập vào `verification_uri` được cung cấp và nhập `user_code` để chấp thuận yêu cầu cấp quyền.

<a name="polling-token-request"></a>
#### Polling Token Request

Vì người dùng sẽ sử dụng một thiết bị khác để cấp quyền (hoặc từ chối) truy cập, thiết bị yêu cầu nên thực hiện polling tới route `/oauth/token` của ứng dụng để xác định khi nào người dùng đã phản hồi yêu cầu. Thiết bị yêu cầu nên sử dụng giá trị `interval` polling tối thiểu được cung cấp trong JSON response khi yêu cầu device code để tránh các lỗi rate limit:

```php
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Sleep;

$interval = 5;

do {
    Sleep::for($interval)->seconds();

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret', // Required for confidential clients only...
        'device_code' => 'the-device-code',
    ]);

    if ($response->json('error') === 'slow_down') {
        $interval += 5;
    }
} while (in_array($response->json('error'), ['authorization_pending', 'slow_down']));

return $response->json();
```

Nếu người dùng đã chấp nhận một yêu cầu cấp quyền, việc này sẽ trả về một JSON response chứa các thuộc tính `access_token`, `refresh_token`, và `expires_in`. Thuộc tính `expires_in` sẽ chứa thời gian access token sẽ bị hết hạn.

<a name="password-grant"></a>
## Password Grant

> [!WARNING]
> Chúng tôi khuyên bạn không nên sử dụng password grant token nữa. Thay vào đó, bạn nên chọn [loại grant mà được OAuth2 Server đề xuất](https://oauth2.thephpleague.com/authorization-server/which-grant/).

OAuth2 password grant cho phép các client bên thứ nhất, chẳng hạn như một application mobile trong tổ chức của bạn, có được access token bằng địa chỉ email hoặc tên người dùng và mật khẩu của họ. Điều này cho phép bạn phát hành access token một cách an toàn cho client bên thứ nhất mà không yêu cầu người dùng của bạn thực hiện toàn bộ các luồng chuyển hướng OAuth2 authorization code.

Để enable password grant, hãy gọi phương thức `enablePasswordGrant` trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enablePasswordGrant();
}
```

<a name="creating-a-password-grant-client"></a>
### Tạo một password grant client

Trước khi application của bạn có thể phát hành token thông qua password grant, bạn sẽ cần phải tạo một password grant client. Bạn có thể làm điều này bằng cách sử dụng lệnh Artisan `passport:client` với tùy chọn `--password`.

```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### Request token

Khi bạn đã enable grant và đã tạo một password grant client, bạn có thể yêu cầu access token bằng cách đưa ra một request `POST` cho route `/oauth/token` với địa chỉ email và mật khẩu của người dùng. Hãy nhớ rằng, route này đã được đăng ký bằng Passport nên bạn không cần phải định nghĩa lại chúng. Nếu yêu cầu thành công, bạn sẽ nhận được một `access_token` và một `refresh_token` trong JSON response từ server:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

> [!NOTE]
> Hãy nhớ rằng, access token sẽ mặc định là tồn tại mãi mãi. Tuy nhiên, bạn có thể thoải mái [cấu hình maximum vòng đời access token của bạn](#configuration) nếu cần.

<a name="requesting-all-scopes"></a>
### Yêu cầu tất cả scope

Khi sử dụng password grant hoặc chứng chỉ client grant, bạn có thể muốn ủy quyền token cho tất cả các scope được application của bạn hỗ trợ. Bạn có thể làm điều này bằng cách yêu cầu scope `*`. Nếu bạn yêu cầu scope là `*`, thì phương thức `can` trên instance token sẽ luôn trả về `true`. Scope này chỉ có thể được gán cho những token mà được cấp bằng `password` hoặc `client_credentials` grant`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => '*',
]);
```

<a name="customizing-the-user-provider"></a>
### Tuỳ biến User Provider

Nếu ứng dụng của bạn sử dụng nhiều hơn một [user provider để xác thực](/docs/{{version}}/authentication#introduction), bạn có thể chỉ định user provider nào sẽ được password grant client sử dụng bằng cách cung cấp thêm một tùy chọn `--provider` khi tạo client thông qua lệnh `artisan passport:client --password`. Tên user provider phải khớp với tên một user provider đã được định nghĩa trong file cấu hình `config/auth.php` của application của bạn. Sau đó, bạn có thể [bảo vệ route của bạn thông qua middleware](#multiple-authentication-guards) để đảm bảo rằng chỉ những người dùng từ user provider được chỉ định mới được cấp quyền.

<a name="customizing-the-username-field"></a>
### Tuỳ biến field username

Khi xác thực bằng password grant, Passport sẽ sử dụng thuộc tính `email` của model authenticatable của bạn làm "username". Tuy nhiên, bạn có thể tùy chỉnh hành động này bằng cách định nghĩa phương thức `findForPassport` trên model của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Bridge\Client;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Find the user instance for the given username.
     */
    public function findForPassport(string $username, Client $client): User
    {
        return $this->where('username', $username)->first();
    }
}
```

<a name="customizing-the-password-validation"></a>
### Tuỳ biến Password Validation

Khi xác thực bằng password grant, Passport sẽ sử dụng thuộc tính `password` trong model của bạn để xác thực mật khẩu đã cho. Nếu model của bạn không có thuộc tính `password` hoặc bạn muốn tùy chỉnh logic xác thực password, bạn có thể định nghĩa phương thức `validateForPassportPasswordGrant` trong model của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Validate the password of the user for the Passport password grant.
     */
    public function validateForPassportPasswordGrant(string $password): bool
    {
        return Hash::check($password, $this->password);
    }
}
```

<a name="implicit-grant"></a>
## Implicit Grant

> [!WARNING]
> Chúng tôi khuyên bạn không nên sử dụng implicit grant token nữa. Thay vào đó, bạn nên chọn [loại grant mà được OAuth2 Server đề xuất](https://oauth2.thephpleague.com/authorization-server/which-grant/).

Grant ẩn tương tự như authorization code grant; tuy nhiên, token được trả về cho client mà không cần thông qua authorization code. Grant này được sử dụng phổ biến nhất cho các application JavaScript hoặc mobile application nơi mà thông tin đăng nhập của client không thể được lưu trữ an toàn. Để kích hoạt grant, hãy gọi phương thức `enableImplicitGrant` trong phương thức `boot` của class `App\Providers\AppServiceProvider` trong ứng dụng của bạn:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

Trước khi ứng dụng của bạn có thể cấp phát token thông qua implicit grant, bạn sẽ cần tạo một implicit grant client. Bạn có thể thực hiện việc này bằng cách sử dụng lệnh Artisan `passport:client` với tùy chọn `--implicit`.

```shell
php artisan passport:client --implicit
```

Khi grant này đã được bật và một implicit client đã được tạo, nhà phát triển có thể sử dụng client ID của chính họ để yêu cầu access token từ application của bạn. Ứng dụng của nhà phát triển sẽ tạo một yêu cầu chuyển hướng đến route `/oauth/authorize` của application của bạn như sau:

```php
use Illuminate\Http\Request;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'token',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

> [!NOTE]
> Hãy nhớ rằng, route `/oauth/authorize` đã được định nghĩa bởi Passport. Bạn không cần phải định nghĩa route này.

<a name="client-credentials-grant"></a>
## Client Credentials Grant

Chứng chỉ client grant thích hợp cho việc authentication machine-to-machine. Ví dụ: bạn có thể sử dụng grant này trong một scheduled job đang thực hiện công việc bảo trì qua API.

Trước khi ứng dụng của bạn có thể phát hành mã token thông qua chứng chỉ client grant, bạn sẽ cần tạo một client chứng chỉ client grant. Bạn có thể thực hiện việc này bằng cách sử dụng tùy chọn `--client` trong lệnh Artisan `passport:client`:

```shell
php artisan passport:client --client
```

Tiếp theo, hãy gán middleware `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner` cho một route:

```php
use Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner;

Route::get('/orders', function (Request $request) {
    // Access token is valid and the client is resource owner...
})->middleware(EnsureClientIsResourceOwner::class);
```

Để giới hạn quyền truy cập vào route cho các scope cụ thể, bạn có thể cung cấp một danh sách các scope bắt buộc cho phương thức `using`:

```php
Route::get('/orders', function (Request $request) {
    // Access token is valid, the client is resource owner, and has both "servers:read" and "servers:create" scopes...
})->middleware(EnsureClientIsResourceOwner::using('servers:read', 'servers:create'));
```

<a name="retrieving-tokens"></a>
### Retrieving Tokens

Để lấy một token của một loại grant này, hãy tạo một request đến `oauth/token` endpoint:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope' => 'servers:read servers:create',
]);

return $response->json()['access_token'];
```

<a name="personal-access-tokens"></a>
## Personal Access Tokens

Đôi khi, người dùng của bạn có thể muốn phát hành access token cho chính họ mà không cần thông qua luồng chuyển hướng authorization code thông thường. Việc cho phép người dùng phát hành token cho chính họ thông qua giao diện người dùng của application của bạn có thể hữu ích khi cho phép người dùng thử nghiệm API của bạn hoặc có thể dùng như một cách tiếp cận đơn giản hơn khi phát hành access token nói chung.

> [!NOTE]
> Nếu ứng dụng của bạn sử dụng Passport chủ yếu là để cấp các mã personal access token, thì hãy cân nhắc sử dụng [Laravel Sanctum](/docs/{{version}}/sanctum), đây là thư viện gọn nhẹ của Laravel để cấp mã API access token.

<a name="creating-a-personal-access-client"></a>
### Tạo một Personal Access Client

Trước khi application của bạn có thể phát hành một personal access token, bạn sẽ cần tạo một personal access client. Bạn có thể làm điều này bằng cách chạy lệnh Artisan `passport:client` với tùy chọn `--personal`. Nếu bạn đã chạy lệnh `passport:install`, bạn không cần chạy lệnh này:

```shell
php artisan passport:client --personal
```

<a name="customizing-the-user-provider-for-pat"></a>
### Customizing the User Provider

Nếu ứng dụng của bạn sử dụng nhiều hơn một [user provider để xác thực](/docs/{{version}}/authentication#introduction), bạn có thể chỉ định xem user provider nào sẽ được personal access grant client sử dụng bằng cách cung cấp thêm tùy chọn `--provider` khi tạo client thông qua lệnh `artisan passport:client --personal`. Tên user provider được cung cấp phải giống với một trong các user provider hợp lệ đã được định nghĩa trong file cấu hình `config/auth.php` của ứng dụng. Sau đó, bạn có thể [bảo vệ route của bạn bằng middleware](#multiple-authentication-guards) để đảm bảo rằng chỉ những người dùng từ user provider đã được chỉ định mới được cấp quyền.

<a name="managing-personal-access-tokens"></a>
### Quản lý Personal Access Tokens

Khi bạn đã tạo một personal access client, bạn có thể phát hành token cho một người dùng bằng cách sử dụng phương thức `createToken` trên instance model `User` đó. Phương thức `createToken` chấp nhận tên của token làm tham số đầu tiên và một mảng tùy chọn [scopes](#token-scopes) làm tham số thứ hai:

```php
use App\Models\User;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Creating a token without scopes...
$token = $user->createToken('My Token')->accessToken;

// Creating a token with scopes...
$token = $user->createToken('My Token', ['user:read', 'orders:create'])->accessToken;

// Creating a token with all scopes...
$token = $user->createToken('My Token', ['*'])->accessToken;

// Retrieving all the valid personal access tokens that belong to the user...
$tokens = $user->tokens()
    ->with('client')
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get()
    ->filter(fn (Token $token) => $token->client->hasGrantType('personal_access'));
```

<a name="protecting-routes"></a>
## Bảo vệ route

<a name="via-middleware"></a>
### Thông qua middleware

Passport có chứa một [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) sẽ kiểm tra access token khi có request đến. Sau khi bạn đã cấu hình xong guard `api` sử dụng `passport` driver, bạn chỉ cần cài đặt middleware `auth:api` vào bất kỳ route nào mà yêu cầu một access token hợp lệ:

```php
Route::get('/user', function () {
    // Only API authenticated users may access this route...
})->middleware('auth:api');
```

> [!WARNING]
> Nếu bạn đang sử dụng [client credentials grant](#client-credentials-grant), bạn nên sử dụng [middleware `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner`](#client-credentials-grant) để bảo vệ các route của bạn thay vì middleware `auth:api`.

<a name="multiple-authentication-guards"></a>
#### Multiple Authentication Guards

Nếu ứng dụng của bạn xác thực các loại người dùng khác nhau mà dùng các model Eloquent khác nhau, bạn có thể sẽ cần định nghĩa một cấu hình guard cho từng loại user providers trong ứng dụng của bạn. Điều này cho phép bạn bảo vệ các request dành cho các user providers cụ thể. Ví dụ: cho cấu hình guard của file cấu hình `config/auth.php` sau:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],
],
```

Route sau sẽ sử dụng guard `api-customers`, sử dụng user provider `customers`, để xác thực các request đến:

```php
Route::get('/customer', function () {
    // ...
})->middleware('auth:api-customers');
```

> [!NOTE]
> Để biết thêm thông tin về cách sử dụng nhiều user provider cùng với Passport, vui lòng tham khảo thêm [tài liệu về personal access tokens](#customizing-the-user-provider-for-pat) và [tài liệu về password grant](#customizing-the-user-provider).

<a name="passing-the-access-token"></a>
### Pass access token

Khi gọi các route mà được bảo vệ bởi Passport, thì API bên thứ ba của application của bạn nên cài đặt access token của họ dưới dạng một `Bearer` token trong header `Authorization` trong request của họ. Ví dụ: khi sử dụng Facade `Http`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => "Bearer $accessToken",
])->get('https://passport-app.test/api/user');

return $response->json();
```

<a name="token-scopes"></a>
## Token scope

Scope cho phép API client của bạn yêu cầu một nhóm quyền cụ thể khi request authorization để truy cập vào tài khoản. Ví dụ: nếu bạn đang xây dựng một application thương mại điện tử, không phải tất cả API bên thứ ba nào cũng sẽ cần khả năng đặt hàng. Thay vào đó, bạn có thể cho phép bên thứ ba chỉ request authorization truy cập vào được trạng thái giao hàng. Nói cách khác, scope cho phép người dùng application của bạn giới hạn các hành động mà application của bên thứ ba có thể thực hiện.

<a name="defining-scopes"></a>
### Định nghĩa scope

Bạn có thể định nghĩa scope của API bằng phương thức `Passport::tokensCan` trong phương thức `boot` của class `App\Providers\AppServiceProvider` của application. Phương thức `tokensCan` chấp nhận một loạt các tên scope và mô tả của nó. Mô tả scope có thể là bất cứ điều gì bạn muốn và sẽ được hiển thị cho người dùng trên màn hình phê duyệt authorization:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensCan([
        'user:read' => 'Retrieve the user info',
        'orders:create' => 'Place orders',
        'orders:read:status' => 'Check order status',
    ]);
}
```

<a name="default-scope"></a>
### Scope mặc định

Nếu một client không yêu cầu bất kỳ scope nào, bạn có thể cấu hình Passport server của bạn để gắn một scope mặc định vào mã token bằng phương thức `defaultScopes`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` trong class `App\Providers\AppServiceProvider` của application:

```php
use Laravel\Passport\Passport;

Passport::tokensCan([
    'user:read' => 'Retrieve the user info',
    'orders:create' => 'Place orders',
    'orders:read:status' => 'Check order status',
]);

Passport::defaultScopes([
    'user:read',
    'orders:create',
]);
```

<a name="assigning-scopes-to-tokens"></a>
### Gán scope đến token

<a name="when-requesting-authorization-codes"></a>
#### When Requesting Authorization Codes

Khi yêu cầu access token bằng cách sử dụng authorization code grant, thì bên thứ ba nên chỉ định scope mà họ mong muốn bằng tham số chuỗi truy vấn `scope`. Tham số `scope` phải là một danh sách scope đã được phân tách bằng dấu cách:

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="when-issuing-personal-access-tokens"></a>
#### When Issuing Personal Access Tokens

Nếu bạn đang phát hành personal access token bằng cách sử dụng phương thức `createToken` của model `App\Models\User`, bạn có thể truyền một mảng scope mong muốn làm tham số thứ hai cho phương thức:

```php
$token = $user->createToken('My Token', ['orders:create'])->accessToken;
```

<a name="checking-scopes"></a>
### Kiểm tra scope

Passport có chứa hai middleware có thể được sử dụng để xác minh xem request đến đã được authenticate với một token đã được cấp với một scope hay chưa.

<a name="check-for-all-scopes"></a>
#### Check For All Scopes

Middleware `Laravel\Passport\Http\Middleware\CheckToken` có thể được chỉ định cho một route để xác minh xem access token của một request đến application đã có tất cả những scope đã được liệt kê hay chưa:

```php
use Laravel\Passport\Http\Middleware\CheckToken;

Route::get('/orders', function () {
    // Access token has both "orders:read" and "orders:create" scopes...
})->middleware(['auth:api', CheckToken::using('orders:read', 'orders:create')]);
```

<a name="check-for-any-scopes"></a>
#### Check For Any Scopes

Middleware `Laravel\Passport\Http\Middleware\CheckTokenForAnyScope` có thể được chỉ định cho một route để xác minh xem access token của request đến đã có *ít nhất một* trong những scope đã được liệt kê hay chưa:

```php
use Laravel\Passport\Http\Middleware\CheckTokenForAnyScope;

Route::get('/orders', function () {
    // Access token has either "orders:read" or "orders:create" scope...
})->middleware(['auth:api', CheckTokenForAnyScope::using('orders:read', 'orders:create')]);
```

<a name="checking-scopes-on-a-token-instance"></a>
#### Kiểm tra scope On A Token Instance

Khi một request được authenticate bằng access token đã vào đến application của bạn, bạn vẫn có thể kiểm tra xem token đã có scope hay chưa bằng cách sử dụng phương thức `tokenCan` trên instance `App\Models\User` đã được xác thực:

```php
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    if ($request->user()->tokenCan('orders:create')) {
        // ...
    }
});
```

<a name="additional-scope-methods"></a>
#### Các phương thức scope khác

Phương thức `scopeIds` sẽ trả về một mảng gồm tất cả các ID và tên đã được định nghĩa:

```php
use Laravel\Passport\Passport;

Passport::scopeIds();
```

Phương thức `scopes` sẽ trả về một mảng gồm tất cả các scope đã được định nghĩa dưới dạng các instance của `Laravel\Passport\Scope`:

```php
Passport::scopes();
```

Phương thức `scopesFor` sẽ trả về một mảng các instance `Laravel\Passport\Scope` mà khớp với các ID và tên đã cho:

```php
Passport::scopesFor(['user:read', 'orders:create']);
```

Bạn có thể kiểm tra xem một scope nhất định đã được định nghĩa hay chưa bằng cách sử dụng phương thức `hasScope`:

```php
Passport::hasScope('orders:create');
```

<a name="spa-authentication"></a>
## SPA Authentication

Khi xây dựng một API, nó có thể rất hữu ích khi sử dụng API của riêng bạn từ application JavaScript. Cách tiếp cận này cho phép application của bạn sử dụng cùng API mà bạn đang chia sẻ với mọi người. API tương tự cũng có thể được sử dụng bởi application web, application di động, application của bên thứ ba hoặc bất kỳ SDK nào bạn có thể publish trên các trình quản lý package khác nhau.

Thông thường, nếu bạn muốn sử dụng API từ application JavaScript của bạn, bạn cần phải tự gửi access token đến application và truyền nó theo mỗi request đến application của bạn. Tuy nhiên, Passport có chứa một middleware có thể xử lý việc này cho bạn. Tất cả những gì bạn cần làm là thêm middleware `CreateFreshApiToken` vào group middleware `web` trong file `bootstrap/app.php` của ứng dụng của bạn:

```php
use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        CreateFreshApiToken::class,
    ]);
})
```

> [!WARNING]
> Bạn nên đảm bảo rằng middleware `CreateFreshApiToken` sẽ được khai báo cuối cùng trong stack middleware của bạn.

Passport middleware này sẽ gán một cookie `laravel_token` vào các response gửi về cho bạn. Cookie này chứa JWT đã được mã hóa mà Passport sẽ sử dụng để xác thực các API request từ application JavaScript của bạn. JWT có thời gian tồn tại bằng với giá trị cấu hình `session.lifetime` của bạn. Bây giờ, vì trình duyệt sẽ tự động gửi cookie này cho tất cả các request tiếp theo, nên bạn có thể thực hiện các request đối với API của application mà không cần phải truyền một access token:

```js
axios.get('/api/user')
    .then(response => {
        console.log(response.data);
    });
```

<a name="customizing-the-cookie-name"></a>
#### Tùy biến tên cookie

Nếu cần, bạn có thể tùy biến tên cookie `laravel_token` bằng phương thức `Passport::cookie`. Thông thường, phương thức này sẽ được gọi từ phương thức `boot` trong class `App\Providers\AppServiceProvider` của application của bạn:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::cookie('custom_name');
}
```

<a name="csrf-protection"></a>
#### CSRF Protection

Khi sử dụng phương thức xác thực này, bạn sẽ cần đảm bảo một header CSRF token hợp lệ đã được chứa trong các request của bạn. Mặc định, khung Laravel JavaScript đã có sẵn một khung mẫu ứng dụng và tất cả các starter kit cũng sẽ chứa một instance [Axios](https://github.com/axios/axios), instance này sẽ tự động sử dụng giá trị cookie `XSRF-TOKEN` được mã hóa để gửi một header `X-XSRF-TOKEN` cho các request có cùng origin.

> [!NOTE]
> Nếu bạn chọn gửi header `X-CSRF-TOKEN` thay vì `X-XSRF-TOKEN`, bạn sẽ cần sử dụng một token chưa được mã hóa do `csrf_token()` cung cấp.

<a name="events"></a>
## Event

Passport sẽ tạo ra các event mỗi khi phát hành một access token và một refresh token. Bạn có thể [lắng nghe các event này](/docs/{{version}}/events) để xoá hoặc thu hồi các access token khác có trong cơ sở dữ liệu của bạn:

<div class="overflow-auto">

| Event Name                                    |
| --------------------------------------------- |
| `Laravel\Passport\Events\AccessTokenCreated`  |
| `Laravel\Passport\Events\AccessTokenRevoked`  |
| `Laravel\Passport\Events\RefreshTokenCreated` |

</div>

<a name="testing"></a>
## Test

Phương thức `actingAs` của Passport có thể được sử dụng để chỉ định một người dùng với scope của họ. Tham số đầu tiên được đưa vào cho phương thức `actingAs` là instance user và tham số thứ hai là một mảng scope được cấp cho token đó của người dùng:

```php tab=Pest
use App\Models\User;
use Laravel\Passport\Passport;

test('orders can be created', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
});
```

```php tab=PHPUnit
use App\Models\User;
use Laravel\Passport\Passport;

public function test_orders_can_be_created(): void
{
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
}
```

Phương thức `actingAsClient` của Passport có thể được sử dụng để chỉ định những client hiện đang được xác thực cũng như scope của nó. Tham số đầu tiên được cung cấp cho phương thức `actingAsClient` là instance client và tham số thứ hai là một mảng scope sẽ được cấp cho token của client đó:

```php tab=Pest
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

test('servers can be retrieved', function () {
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function test_servers_can_be_retrieved(): void
{
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
}
```
