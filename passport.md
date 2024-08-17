# Laravel Passport

- [Giới thiệu](#introduction)
    - [Passport hay Sanctum?](#passport-or-sanctum)
- [Cài đặt](#installation)
    - [Deploy Passport](#deploying-passport)
    - [Migration Customization](#migration-customization)
    - [Cập nhật Passport](#upgrading-passport)
- [Cấu hình](#configuration)
    - [Client Secret Hashing](#client-secret-hashing)
    - [Thời gian sống token](#token-lifetimes)
    - [Ghi đè các model mặc định](#overriding-default-models)
    - [Ghi đè routes](#overriding-routes)
- [Phát hành access token](#issuing-access-tokens)
    - [Quản lý client](#managing-clients)
    - [Request token](#requesting-tokens)
    - [Refresh token](#refreshing-tokens)
    - [Thu hồi Tokens](#revoking-tokens)
    - [Lọc token](#purging-tokens)
- [Authorization Code Grant với PKCE](#code-grant-pkce)
    - [Tạo client](#creating-a-auth-pkce-grant-client)
    - [Request token](#requesting-auth-pkce-grant-tokens)
- [Token password grant](#password-grant-tokens)
    - [Tạo một password grant client](#creating-a-password-grant-client)
    - [Request token](#requesting-password-grant-tokens)
    - [Request all scope](#requesting-all-scopes)
    - [Tuỳ biến User Provider](#customizing-the-user-provider)
    - [Tuỳ biến field username](#customizing-the-username-field)
    - [Tuỳ biến Password Validation](#customizing-the-password-validation)
- [Token với grant ẩn](#implicit-grant-tokens)
- [Token chứng chỉ client grant](#client-credentials-grant-tokens)
- [Access token cá nhân](#personal-access-tokens)
    - [Tạo một access client cá nhân](#creating-a-personal-access-client)
    - [Quản lý access token cá nhân](#managing-personal-access-tokens)
- [Bảo vệ route](#protecting-routes)
    - [Thông qua middleware](#via-middleware)
    - [Pass access token](#passing-the-access-token)
- [Token scope](#token-scopes)
    - [Định nghĩa scope](#defining-scopes)
    - [Scope mặc định](#default-scope)
    - [Gán scope đến token](#assigning-scopes-to-tokens)
    - [Kiểm tra scope](#checking-scopes)
- [Sử dụng API của bạn với JavaScript](#consuming-your-api-with-javascript)
- [Event](#events)
- [Test](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Passport](https://github.com/laravel/passport) cung cấp một implementation OAuth2 server đầy đủ cho application Laravel của bạn trong vài phút. Passport được xây dựng trên top của [League OAuth2 server](https://github.com/thephpleague/oauth2-server) được duy trì bởi Andy Millington và Simon Hamp.

> **Warning**
> Tài liệu này giả định rằng bạn đã biết OAuth2. Nếu bạn chưa biết về OAuth2, hãy xem xét việc tự học với các [thuật ngữ](https://oauth2.thephpleague.com/terminology/) và tính năng chung của OAuth2 trước khi tiếp tục.

<a name="passport-or-sanctum"></a>
### Passport hay Sanctum?

Trước khi bắt đầu, bạn có thể muốn xem xét xem ứng dụng của bạn sẽ được phục vụ tốt hơn bởi Laravel Passport hay [Laravel Sanctum](/docs/{{version}}/sanctum). Nếu ứng dụng của bạn thực sự cần hỗ trợ OAuth2 thì bạn nên sử dụng Laravel Passport.

Tuy nhiên, nếu bạn đang làm xác thực cho một ứng dụng single-page, mobile application hoặc phát hành API token, bạn nên sử dụng [Laravel Sanctum](/docs/{{version}}/sanctum). Laravel Sanctum không hỗ trợ OAuth2; tuy nhiên, nó cung cấp trải nghiệm phát triển xác thực API đơn giản hơn nhiều.

<a name="installation"></a>
## Cài đặt

Để bắt đầu, hãy cài đặt Passport thông qua Composer package manager:

```shell
composer require laravel/passport
```

[Service provider](/docs/{{version}}/providers) của Passport sẽ đăng ký thư mục database migration của riêng nó với framework, nên vì thế bạn nên migrate cơ sở dữ liệu của bạn sau khi cài đặt xong package. Việc migrate của Passport sẽ tạo ra các table mà application của bạn cần để lưu trữ OAuth2 client và access token:

```shell
php artisan migrate
```

Tiếp theo, bạn nên chạy lệnh Artisan `passport:install`. Lệnh này sẽ tạo các key mã hóa cần thiết để tạo secure access token. Ngoài ra, lệnh này cũng sẽ tạo các "personal access" và các "password grant" client được sử dụng để tạo access token:

```shell
php artisan passport:install
```

> **Note**
> Nếu bạn muốn sử dụng UUID làm khóa chính của model Passport `Client` thay vì các integer tự động tăng, vui lòng cài đặt Passport với [tùy chọn `uuids`](#client-uuids).

Sau khi chạy lệnh `passport:install`, hãy thêm trait `Laravel\Passport\HasApiTokens` vào model `App\User` của bạn. Trait này sẽ cung cấp một vài phương thức helper cho model của bạn, cho phép bạn kiểm tra token và phạm vi của người dùng đã được authenticate. Nếu model của bạn đã sử dụng trait `Laravel\Sanctum\HasApiTokens`, bạn có thể xóa trait đó đi:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }

Cuối cùng, trong file cấu hình `config/auth.php` của application của bạn, bạn nên định nghĩa một guard xác thực `api` và thiết lập tùy chọn `driver` thành `passport`. Điều này sẽ hướng dẫn application của bạn sử dụng `TokenGuard` của Passport khi authenticate các request API:

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

<a name="client-uuids"></a>
#### Client UUIDs

Bạn cũng có thể chạy lệnh `passport:install` với tùy chọn `--uuids`. Tuỳ chọn này sẽ hướng dẫn Passport là bạn muốn sử dụng UUID làm giá trị khóa chính của model Passport `Client` thay vì một integer tự động tăng. Sau khi chạy lệnh `passport:install` với tùy chọn `--uuids`, bạn cũng sẽ được nhận được các hướng dẫn bổ sung về cách tắt tính năng migration mặc định của Passport:

```shell
php artisan passport:install --uuids
```

<a name="deploying-passport"></a>
### Deploying Passport

Khi deploy Passport lần đầu đến server application của bạn, bạn có thể sẽ cần chạy lệnh `passport:keys`. Lệnh này sẽ tạo các key mã hóa Passport cần, để tạo access token. Các key được tạo thường không nên được lưu trữ trong source code control:

```shell
php artisan passport:keys
```

Nếu cần, bạn có thể định nghĩa đường dẫn nơi mà các khóa của Passport sẽ được load từ đó. Bạn có thể sử dụng phương thức `Passport::loadKeysFrom` để thực hiện việc này. Thông thường, phương thức này phải được gọi từ phương thức `boot` của class `App\Providers\AuthServiceProvider` trong ứng dụng của bạn:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
    }

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

<a name="migration-customization"></a>
### Migration Customization

Nếu bạn không muốn sử dụng migration mặc định của Passport, bạn nên gọi phương thức `Passport::ignoreMigrations` trong phương thức `register` của class `App\Providers\AppServiceProvider` của bạn. Bạn có thể export các migration mặc định này bằng cách sử dụng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag=passport-migrations
```

<a name="upgrading-passport"></a>
### Cập nhật Passport

Khi nâng cấp lên phiên bản mới của Passport, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/passport/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Cấu hình

<a name="client-secret-hashing"></a>
### Client Secret Hashing

Nếu bạn muốn hash các client secret khi lưu vào trong cơ sở dữ liệu của bạn, bạn nên gọi phương thức `Passport::hashClientSecrets` trong phương thức `boot` của class `App\Providers\AuthServiceProvider`:

    use Laravel\Passport\Passport;

    Passport::hashClientSecrets();

Sau khi bạn đã cài đặt xong, tất cả các client secret của bạn sẽ chỉ có thể hiển thị cho người dùng ngay sau khi họ tạo ra. Vì giá trị chính xác của client secret này sẽ không bao giờ được lưu vào trong cơ sở dữ liệu, nên bạn sẽ không thể khôi phục lại giá trị của secret nếu nó bị mất.

<a name="token-lifetimes"></a>
### Thời gian sống token

Mặc định, Passport phát hành các access token tồn tại lâu dài có thời hạn một năm. Nếu bạn muốn cấu hình vòng đời token dài hoặc ngắn hơn, bạn có thể sử dụng các phương thức `tokensExpireIn`, `refreshTokensExpireIn`, và `personalAccessTokensExpireIn`. Các phương thức này phải được gọi từ phương thức `boot` của class `App\Providers\AuthServiceProvider` của application:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::tokensExpireIn(now()->addDays(15));
        Passport::refreshTokensExpireIn(now()->addDays(30));
        Passport::personalAccessTokensExpireIn(now()->addMonths(6));
    }

> **Warning**
> Các cột `expires_at` trong bảng cơ sở dữ liệu Passport sẽ ở chế độ chỉ-đọc và chỉ dành cho mục đích hiển thị. Khi phát hành token, Passport sẽ lưu trữ thông tin hết hạn vào trong các token đó và mã hóa chúng. Nếu bạn muốn làm mất hiệu lực token, bạn nên [thu hồi nó](#revoking-tokens).

<a name="overriding-default-models"></a>
### Ghi đè các model mặc định

Bạn có thể thoải mái mở rộng các model được sử dụng trong nội bộ Passport bằng cách định nghĩa model của riêng bạn và extend model Passport tương ứng:

    use Laravel\Passport\Client as PassportClient;

    class Client extends PassportClient
    {
        // ...
    }

Sau khi định nghĩa xong model của bạn, bạn có thể hướng dẫn Passport sử dụng các model tùy biến này thông qua `Laravel\Passport\Passport` class. Thông thường, bạn nên thông báo cho Passport biết về các model tùy chỉnh của bạn trong phương thức `boot` của class `App\Providers\AuthServiceProvider` trong ứng dụng của bạn:

    use App\Models\Passport\AuthCode;
    use App\Models\Passport\Client;
    use App\Models\Passport\PersonalAccessClient;
    use App\Models\Passport\RefreshToken;
    use App\Models\Passport\Token;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::useTokenModel(Token::class);
        Passport::useRefreshTokenModel(RefreshToken::class);
        Passport::useAuthCodeModel(AuthCode::class);
        Passport::useClientModel(Client::class);
        Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
    }

<a name="overriding-routes"></a>
### Ghi đè routes

Thỉnh thoảng bạn có thể muốn tùy chỉnh các route được định nghĩa bởi Passport. Để thực hiện điều này, trước tiên bạn cần bỏ qua các route được Passport đăng ký bằng cách thêm `Passport::ignoreRoutes` vào phương thức `register` của `AppServiceProvider` của ứng dụng:

    use Laravel\Passport\Passport;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Passport::ignoreRoutes();
    }

Sau đó, bạn có thể copy các route được Passport định nghĩa trong [file route](https://github.com/laravel/passport/blob/11.x/routes/web.php) vào file `routes/web.php` của ứng dụng và sửa chúng theo ý thích của bạn:

    Route::group([
        'as' => 'passport.',
        'prefix' => config('passport.path', 'oauth'),
        'namespace' => 'Laravel\Passport\Http\Controllers',
    ], function () {
        // Passport routes...
    });

<a name="issuing-access-tokens"></a>
## Phát hành token truy cập

Sử dụng OAuth2 thông qua authorization code là cách mà hầu hết các nhà phát triển quen thuộc với OAuth2. Khi sử dụng authorization code, một client application sẽ chuyển hướng người dùng đến server của bạn, nơi mà người dùng sẽ chấp nhận hoặc từ chối request cấp access token cho client.

<a name="managing-clients"></a>
### Quản lý client

Đầu tiên, các nhà phát triển cần xây dựng các ứng dụng của họ, mà cần tương tác với API application của bạn, họ sẽ cần phải đăng ký ứng dụng của họ với application của bạn bằng cách tạo "client". Thông thường, việc này bao gồm việc cung cấp tên ứng dụng và URL mà application của bạn cần chuyển hướng người dùng đến sau khi người dùng đã chấp thuận request ủy quyền.

<a name="the-passportclient-command"></a>
#### Lệnh `passport:client`

Cách đơn giản nhất để tạo một client là sử dụng lệnh Artisan `passport:client`. Lệnh này có thể được sử dụng để tạo các client của riêng bạn để test các chức năng OAuth2. Khi bạn chạy lệnh `client`, Passport sẽ hỏi bạn cho biết thêm thông tin về client của bạn và trả về cho bạn một client ID và một secret:

```shell
php artisan passport:client
```

**Redirect URLs**

Nếu bạn muốn lập một danh sách cho phép nhiều URL chuyển hướng cho client của bạn, bạn có thể chỉ định chúng bằng cách sử dụng một danh sách được phân cách bằng dấu phẩy khi nhập URL bằng lệnh `passport:client`. Bất kỳ URL nào chứa dấu phẩy đều phải được encode URL:

```shell
http://example.com/callback,http://examplefoo.com/callback
```

<a name="clients-json-api"></a>
#### JSON API

Vì người dùng application của bạn sẽ không thể sử dụng lệnh `client`, nên Passport cũng cung cấp một JSON API mà bạn có thể sử dụng để tạo client. Điều này giúp bạn tránh những rắc rối khi phải tự viết controller để tạo, cập nhật và xóa client.

Tuy nhiên, bạn sẽ cần kết nối JSON API của Passport với frontend của bạn để cung cấp một bảng điều khiển cho người dùng biết và quản lý các client của họ. Dưới đây, chúng ta sẽ xem xét tất cả các API endpoint để quản lý client. Để thuận tiện, chúng ta sẽ sử dụng [Axios](https://github.com/axios/axios) để thực hiện các HTTP request đến các endpoint.

JSON API được bảo vệ bởi middleware `web` và `auth`; do đó, nó chỉ có thể được gọi từ ứng dụng của bạn. Nó không thể được gọi từ một nguồn ở bên ngoài nào khác.

<a name="get-oauthclients"></a>
#### `GET /oauth/clients`

Route này sẽ trả về tất cả các client cho người dùng đã được authenticate. Điều này chủ yếu hữu ích để liệt kê tất cả các client của người dùng để họ có thể chỉnh sửa hoặc xóa chúng:

```js
axios.get('/oauth/clients')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthclients"></a>
#### `POST /oauth/clients`

Route này được sử dụng để tạo client mới. Nó đòi hỏi hai phần dữ liệu: một là `name` của client và một URL `redirect`. URL `redirect` là nơi người dùng sẽ được chuyển hướng đến sau khi chấp nhận hoặc từ chối một request cho authorization.

Sau Khi một client đã được tạo, nó sẽ được cũng cấp cho một client ID và một client secret. Các giá trị này sẽ được sử dụng khi yêu cầu access token từ application của bạn. Route tạo client sẽ trả về instance client mới:

```js
const data = {
    name: 'Client Name',
    redirect: 'http://example.com/callback'
};

axios.post('/oauth/clients', data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="put-oauthclientsclient-id"></a>
#### `PUT /oauth/clients/{client-id}`

Route này được sử dụng để cập nhật client. Nó đòi hỏi hai phần dữ liệu: một là `name` của client và một URL `redirect`. URL `redirect` là nơi người dùng sẽ được chuyển hướng đến sau khi chấp nhận hoặc từ chối một request cho authorization. Route sẽ trả về instance client đã được cập nhật:

```js
const data = {
    name: 'New Client Name',
    redirect: 'http://example.com/callback'
};

axios.put('/oauth/clients/' + clientId, data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="delete-oauthclientsclient-id"></a>
#### `DELETE /oauth/clients/{client-id}`

Route này được sử dụng để xóa client:

```js
axios.delete('/oauth/clients/' + clientId)
    .then(response => {
        //
    });
```

<a name="requesting-tokens"></a>
### Request token

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### Redirecting For Authorization

Khi một client đã được tạo, các developer có thể sử dụng client ID và secret được trả về để yêu cầu authorization code và access token từ application của bạn. Đầu tiên, application của bên thứ ba sẽ tạo một yêu cầu chuyển hướng đến route `/oauth/authorize` của application của bạn như sau:

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

Tham số `prompt` có thể được sử dụng để chỉ định loại xác thực của ứng dụng Passport.

Nếu giá trị `prompt` là `none`, Passport sẽ luôn đưa ra lỗi xác thực nếu người dùng chưa được xác thực với ứng dụng Passport. Nếu giá trị là `consent`, Passport sẽ luôn hiển thị màn hình chấp nhận authorization, ngay cả khi tất cả các scope đã được cấp quyền trước đó cho ứng dụng sử dụng. Khi giá trị là `login`, ứng dụng Passport sẽ luôn nhắc người dùng phải đăng nhập lại vào ứng dụng, ngay cả khi họ đã tồn tại trong session.

Nếu không cung cấp giá trị `prompt`, người dùng sẽ chỉ được nhắc là cấp quyền nếu trước đó họ chưa cấp quyền truy cập cho ứng dụng đang sử dụng với các scope được yêu cầu.

> **Note**
> Hãy nhớ rằng route `/oauth/authorize` đã được Passport định nghĩa. Bạn không cần phải tự định nghĩa route này nữa.

<a name="approving-the-request"></a>
#### Approving The Request

Khi nhận được authorization request, Passport sẽ tự động phản hồi dựa trên giá trị của tham số `prompt` (nếu có) và có thể hiển thị một template cho người dùng để họ có thể chấp nhận hoặc từ chối authorization request. Nếu họ chấp nhận request, họ sẽ được chuyển hướng trở lại `redirect_uri` sẽ được chỉ định bởi application của bên thứ ba. `redirect_uri` phải khớp với URL `redirect` được chỉ định khi client được tạo.

Nếu bạn muốn tùy chỉnh màn hình phê duyệt authorization, bạn có thể publish các view của Passport bằng cách sử dụng lệnh Artisan `vendor:publish`. Các view được publish sẽ được lưu trong thư mục `resources/views/vendor/passport`:

```shell
php artisan vendor:publish --tag=passport-views
```

Thỉnh thoảng bạn có thể muốn bỏ qua các lời nhắc cấp quyền, chẳng hạn như khi cấp quyền cho client bên thứ nhất. Bạn có thể thực hiện điều này bằng cách [extend model `Client`](#overriding-default-models) và định nghĩa phương thức `skipsAuthorization`. Nếu `skipsAuthorization` trả về `true` thì ứng dụng client sẽ được chấp thuận và người dùng sẽ được chuyển hướng trở lại về `redirect_uri` ngay lập tức, trừ khi ứng dụng đang sử dụng đó đã thiết lập tham số `prompt` khi chuyển hướng để xác thực:

    <?php

    namespace App\Models\Passport;

    use Laravel\Passport\Client as BaseClient;

    class Client extends BaseClient
    {
        /**
         * Determine if the client should skip the authorization prompt.
         *
         * @return bool
         */
        public function skipsAuthorization()
        {
            return $this->firstParty();
        }
    }

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### Converting Authorization Codes To Access Tokens

Nếu người dùng chấp nhận authorization request, họ sẽ được chuyển hướng trở lại application của bên thứ ba. Sau đó, đầu tiên, bên thứ ba sẽ kiểm tra tham số `state` với giá trị đã được lưu trữ trước khi chuyển hướng. Nếu tham số state trùng khớp với giá trị đã được lưu, nó sẽ đưa ra một request `POST` cho application của bạn để yêu cầu access token. Yêu cầu phải chứa authorization code được cấp bởi application của bạn khi người dùng chấp nhận authorization request. Trong ví dụ này, chúng ta sẽ sử dụng thư viện Guzzle HTTP để thực hiện request `POST`:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code' => $request->code,
        ]);

        return $response->json();
    });

Route `/oauth/token` này sẽ trả về một JSON response có chứa các thuộc tính `access_token`, `refresh_token` và `expires_in`. Thuộc tính `expires_in` sẽ chứa số giây cho đến khi access token hết hạn.

> **Note**
> Giống như route `/oauth/authorize`, route `/oauth/token` đã được định nghĩa cho bạn bằng Passport. Bạn không cần phải tự định nghĩa route này.

<a name="tokens-json-api"></a>
#### JSON API

Passport cũng chứa một JSON API để quản lý các access token đã được ủy quyền. Bạn có thể ghép API này với giao diện người dùng của riêng bạn để cung cấp cho người dùng một trang tổng thể để quản lý access token. Để thuận tiện, chúng ta sẽ sử dụng [Axios](https://github.com/mzabriskie/axios) để demo việc thực hiện các HTTP request tới các endpoint. JSON API này được bảo vệ bởi middleware `web` và `auth`; do đó, nó chỉ có thể được gọi từ ứng dụng của bạn.

<a name="get-oauthtokens"></a>
#### `GET /oauth/tokens`

Route này sẽ trả về tất cả các access token đã được ủy quyền mà người dùng đã tạo. Điều này chủ yếu hữu ích cho việc hiển thị tất cả các token của người dùng để họ có thể thu hồi chúng:

```js
axios.get('/oauth/tokens')
    .then(response => {
        console.log(response.data);
    });
```

<a name="delete-oauthtokenstoken-id"></a>
#### `DELETE /oauth/tokens/{token-id}`

Route này có thể được sử dụng để thu hồi một access token đã được ủy quyền và các refresh token liên quan của chúng:

```js
axios.delete('/oauth/tokens/' + tokenId);
```

<a name="refreshing-tokens"></a>
### Refresh token

Nếu application của bạn phát hành access token ngắn hạn, người dùng sẽ cần phải refresh access token của họ thông qua refresh token được cung cấp cho họ khi access token được phát hành:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ]);

    return $response->json();

Route `/oauth/token` này sẽ trả về một JSON response có chứa các thuộc tính `access_token`, `refresh_token` và `expires_in`. Thuộc tính `expires_in` sẽ chứa số giây cho đến khi access token hết hạn.

<a name="revoking-tokens"></a>
### Thu hồi Tokens

Bạn có thể thu hồi một token cách sử dụng phương thức `revokeAccessToken` trên `Laravel\Passport\TokenRepository`. Bạn có thể thu hồi các refresh của một token phương thức `revokeRefreshTokensByAccessTokenId` trên `Laravel\Passport\RefreshTokenRepository`. Các class này có thể được resolve bằng cách sử dụng [service container](/docs/{{version}}/container):

    use Laravel\Passport\TokenRepository;
    use Laravel\Passport\RefreshTokenRepository;

    $tokenRepository = app(TokenRepository::class);
    $refreshTokenRepository = app(RefreshTokenRepository::class);

    // Revoke an access token...
    $tokenRepository->revokeAccessToken($tokenId);

    // Revoke all of the token's refresh tokens...
    $refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);

<a name="purging-tokens"></a>
### Lọc token

Khi token bị thu hồi hoặc bị hết hạn, bạn có thể muốn xóa chúng ra khỏi cơ sở dữ liệu. Passport có kèm theo một lệnh Artisan `passport:purge` có thể thực hiện việc này cho bạn:

```shell
# Purge revoked and expired tokens and auth codes...
php artisan passport:purge

# Only purge tokens expired for more than 6 hours...
php artisan passport:purge --hours=6

# Only purge revoked tokens and auth codes...
php artisan passport:purge --revoked

# Only purge expired tokens and auth codes...
php artisan passport:purge --expired
```

Bạn cũng có thể cấu hình một [scheduled job](/docs/{{version}}/scheduling) trong class `App\Console\Kernel` của application của bạn để tự động lọc token của bạn theo một schedule:

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('passport:purge')->hourly();
    }

<a name="code-grant-pkce"></a>
## Authorization Code Grant với PKCE

Việc Authorization Code grant với "Proof Key for Code Exchange" (PKCE) là một cách an toàn để xác thực các trang web hoặc các ứng dụng truy cập vào API của bạn. Grant này sẽ được sử dụng khi bạn không thể đảm bảo rằng client secret sẽ được lưu trữ một cách an toàn hoặc cũng có thể là để giảm thiểu nguy cơ bị kẻ tấn công chặn authorization code. Sự kết hợp giữa một "code verifier" và một "code challenge" sẽ thay thế client secret khi trao đổi authorization code để lấy một access token.

<a name="creating-a-auth-pkce-grant-client"></a>
### Tạo client

Trước khi ứng dụng của bạn có thể phát hành token thông qua authorization code grant với PKCE, bạn sẽ cần tạo một ứng dụng client hỗ trợ PKCE. Bạn có thể thực hiện việc này bằng lệnh Artisan `passport:client` với tùy chọn `--public`:

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### Request token

<a name="code-verifier-code-challenge"></a>
#### Code Verifier & Code Challenge

Vì authorization grant này không cung cấp một client secret, nên các nhà phát triển sẽ cần phải tạo ra một code verifier và một code challenge để yêu cầu token.

Code verifier phải là một chuỗi ngẫu nhiên từ 43 đến 128 ký tự chứa các chữ cái, số, và các ký tự `"-"`, `"."`, `"_"`, `"~"`, như được định nghĩa trong [tài liệu RFC 7636 đặc điểm kỹ thuật](https://tools.ietf.org/html/rfc7636).

Code challenge phải là một chuỗi được mã hóa Base64 với URL và các ký tự an toàn cho tên file. Các ký tự ở cuối dấu `'='` phải được loại bỏ và không được có dấu ngắt dòng, khoảng trắng hoặc các ký tự bổ sung khác.

    $encoded = base64_encode(hash('sha256', $code_verifier, true));

    $codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### Redirecting For Authorization

Sau khi một ứng dụng client đã được tạo xong, bạn có thể sử dụng ID của ứng dụng client đó và code verifier và code challenge đã tạo để yêu cầu một authorization code và một access token từ ứng dụng của bạn. Đầu tiên, ứng dụng đang sử dụng phải thực hiện một request chuyển hướng đến route `/oauth/authorize` của ứng dụng của bạn:

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $request->session()->put(
            'code_verifier', $code_verifier = Str::random(128)
        );

        $codeChallenge = strtr(rtrim(
            base64_encode(hash('sha256', $code_verifier, true))
        , '='), '+/', '-_');

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            'code_challenge' => $codeChallenge,
            'code_challenge_method' => 'S256',
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### Converting Authorization Codes To Access Tokens

Nếu người dùng chấp thuận yêu cầu authorization, họ sẽ được chuyển hướng trở lại ứng dụng mà họ đang sử dụng. Người dùng api của bạn nên xác thực thông số `state` so với giá trị đã được lưu trữ trước khi được chuyển hướng, như trong Authorization Code Grant tiêu chuẩn.

Nếu thông số state khớp, Người dùng api của bạn nên đưa ra một request `POST` cho ứng dụng của bạn để yêu cầu một access token. Yêu cầu này phải chứa authorization code do ứng dụng của bạn cấp khi người dùng chấp thuận yêu cầu authorization cùng với code verifier đã được tạo ra ban đầu:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        $codeVerifier = $request->session()->pull('code_verifier');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code_verifier' => $codeVerifier,
            'code' => $request->code,
        ]);

        return $response->json();
    });

<a name="password-grant-tokens"></a>
## Token password grant

> **Warning**
> Chúng tôi khuyên bạn không nên sử dụng password grant token nữa. Thay vào đó, bạn nên chọn [loại grant mà được OAuth2 Server đề xuất](https://oauth2.thephpleague.com/authorization-server/which-grant/).

OAuth2 password grant cho phép các client bên thứ nhất, chẳng hạn như một application mobile trong tổ chức của bạn, có được access token bằng địa chỉ email hoặc tên người dùng và mật khẩu của họ. Điều này cho phép bạn phát hành access token một cách an toàn cho client bên thứ nhất mà không yêu cầu người dùng của bạn thực hiện toàn bộ các luồng chuyển hướng OAuth2 authorization code.

<a name="creating-a-password-grant-client"></a>
### Tạo một password grant client

Trước khi application của bạn có thể phát hành token thông qua password grant, bạn sẽ cần phải tạo một password grant client. Bạn có thể làm điều này bằng cách sử dụng lệnh Artisan `passport:client` với tùy chọn `--password`. **Nếu bạn đã chạy lệnh `passport:install`, thì bạn không cần phải chạy lệnh này:**

```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### Request token

Khi bạn đã tạo một password grant client, bạn có thể yêu cầu access token bằng cách đưa ra một request `POST` cho route `/oauth/token` với địa chỉ email và mật khẩu của người dùng. Hãy nhớ rằng, route này đã được đăng ký bằng Passport nên bạn không cần phải định nghĩa lại chúng. Nếu yêu cầu thành công, bạn sẽ nhận được một `access_token` và một `refresh_token` trong JSON response từ server:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ]);

    return $response->json();

> **Note**
> Hãy nhớ rằng, access token sẽ mặc định là tồn tại mãi mãi. Tuy nhiên, bạn có thể thoải mái [cấu hình maximum vòng đời access token của bạn](#configuration) nếu cần.

<a name="requesting-all-scopes"></a>
### Yêu cầu tất cả scope

Khi sử dụng password grant hoặc chứng chỉ client grant, bạn có thể muốn ủy quyền token cho tất cả các scope được application của bạn hỗ trợ. Bạn có thể làm điều này bằng cách yêu cầu scope `*`. Nếu bạn yêu cầu scope là `*`, thì phương thức `can` trên instance token sẽ luôn trả về `true`. Scope này chỉ có thể được gán cho những token mà được cấp bằng `password` hoặc `client_credentials` grant`:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '*',
    ]);

<a name="customizing-the-user-provider"></a>
### Tuỳ biến User Provider

Nếu ứng dụng của bạn sử dụng nhiều hơn một [user provider để xác thực](/docs/{{version}}/authentication#introduction), bạn có thể chỉ định user provider nào sẽ được password grant client sử dụng bằng cách cung cấp thêm một tùy chọn `--provider` khi tạo client thông qua lệnh `artisan passport:client --password`. Tên user provider phải khớp với tên một user provider đã được định nghĩa trong file cấu hình `config/auth.php` của application của bạn. Sau đó, bạn có thể [bảo vệ route của bạn thông qua middleware](#via-middleware) để đảm bảo rằng chỉ những người dùng từ user provider được chỉ định mới được cấp quyền.

<a name="customizing-the-username-field"></a>
### Tuỳ biến field username

Khi xác thực bằng password grant, Passport sẽ sử dụng thuộc tính `email` của model authenticatable của bạn làm "username". Tuy nhiên, bạn có thể tùy chỉnh hành động này bằng cách định nghĩa phương thức `findForPassport` trên model của bạn:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * Find the user instance for the given username.
         *
         * @param  string  $username
         * @return \App\Models\User
         */
        public function findForPassport($username)
        {
            return $this->where('username', $username)->first();
        }
    }

<a name="customizing-the-password-validation"></a>
### Tuỳ biến Password Validation

Khi xác thực bằng password grant, Passport sẽ sử dụng thuộc tính `password` trong model của bạn để xác thực mật khẩu đã cho. Nếu model của bạn không có thuộc tính `password` hoặc bạn muốn tùy chỉnh logic xác thực password, bạn có thể định nghĩa phương thức `validateForPassportPasswordGrant` trong model của bạn:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Support\Facades\Hash;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * Validate the password of the user for the Passport password grant.
         *
         * @param  string $password
         * @return bool
         */
        public function validateForPassportPasswordGrant($password)
        {
            return Hash::check($password, $this->password);
        }
    }

<a name="implicit-grant-tokens"></a>
## Token với grant ẩn

> **Warning**
> Chúng tôi khuyên bạn không nên sử dụng implicit grant token nữa. Thay vào đó, bạn nên chọn [loại grant mà được OAuth2 Server đề xuất](https://oauth2.thephpleague.com/authorization-server/which-grant/).

Grant ẩn tương tự như authorization code grant; tuy nhiên, token được trả về cho client mà không cần thông qua authorization code. Grant này được sử dụng phổ biến nhất cho các application JavaScript hoặc mobile application nơi mà thông tin đăng nhập của client không thể được lưu trữ an toàn. Để kích hoạt grant, hãy gọi phương thức `enableImplicitGrant` trong the `boot` method of your application's `App\Providers\AuthServiceProvider` class: phương thức `boot` của lớp `App\Providers\AuthServiceProvider` trong ứng dụng của bạn:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::enableImplicitGrant();
    }

Khi grant này đã được bật, nhà phát triển có thể sử dụng client ID của chính họ để yêu cầu access token từ application của bạn. Ứng dụng của nhà phát triển sẽ tạo một yêu cầu chuyển hướng đến route `/oauth/authorize` của application của bạn như sau:

    use Illuminate\Http\Request;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'token',
            'scope' => '',
            'state' => $state,
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

> **Note**
> Hãy nhớ rằng, route `/oauth/authorize` đã được định nghĩa bởi Passport. Bạn không cần phải định nghĩa route này.

<a name="client-credentials-grant-tokens"></a>
## Token chứng chỉ client grant

Chứng chỉ client grant thích hợp cho việc authentication machine-to-machine. Ví dụ: bạn có thể sử dụng grant này trong một scheduled job đang thực hiện công việc bảo trì qua API.

Trước khi ứng dụng của bạn có thể phát hành mã token thông qua chứng chỉ client grant, bạn sẽ cần tạo một client chứng chỉ client grant. Bạn có thể thực hiện việc này bằng cách sử dụng tùy chọn `--client` trong lệnh Artisan `passport:client`:

```shell
php artisan passport:client --client
```

Tiếp theo, để sử dụng loại grant này, bạn cần thêm middleware `CheckClientCredentials` vào thuộc tính `$routeMiddleware` trong file `app/Http/Kernel.php` của bạn:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

Sau đó gắn middleware này vào một route:

    Route::get('/orders', function(Request $request) {
        ...
    })->middleware('client');

Để hạn chế quyền truy cập vào route đối với một số scope cụ thể, bạn có thể cung cấp một danh sách các scope yêu cầu bắt buộc khi gắn middleware `client` vào route, bạn có thể được phân chia các scope này bằng dấu phẩy:

    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client:check-status,your-scope');

<a name="retrieving-tokens"></a>
### Retrieving Tokens

Để lấy một token của một loại grant này, hãy tạo một request đến `oauth/token` endpoint:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'client_credentials',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => 'your-scope',
    ]);

    return $response->json()['access_token'];

<a name="personal-access-tokens"></a>
## Personal Access Tokens

Đôi khi, người dùng của bạn có thể muốn phát hành access token cho chính họ mà không cần thông qua luồng chuyển hướng authorization code thông thường. Việc cho phép người dùng phát hành token cho chính họ thông qua giao diện người dùng của application của bạn có thể hữu ích khi cho phép người dùng thử nghiệm API của bạn hoặc có thể dùng như một cách tiếp cận đơn giản hơn khi phát hành access token nói chung.

> **Note**
> Nếu ứng dụng của bạn sử dụng Passport chủ yếu là để cấp các mã personal access token, thì hãy cân nhắc sử dụng [Laravel Sanctum](/docs/{{version}}/sanctum), đây là thư viện gọn nhẹ của Laravel để cấp mã API access token.

<a name="creating-a-personal-access-client"></a>
### Tạo một Personal Access Client

Trước khi application của bạn có thể phát hành một personal access token, bạn sẽ cần tạo một personal access client. Bạn có thể làm điều này bằng cách chạy lệnh Artisan `passport:client` với tùy chọn `--personal`. Nếu bạn đã chạy lệnh `passport:install`, bạn không cần chạy lệnh này:

```shell
php artisan passport:client --personal
```

Sau khi tạo personal access client của bạn, hãy set một giá ID của client và một giá trị secret vào trong file `.env` của ứng dụng của bạn:

```ini
PASSPORT_PERSONAL_ACCESS_CLIENT_ID="client-id-value"
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET="unhashed-client-secret-value"
```

<a name="managing-personal-access-tokens"></a>
### Quản lý Personal Access Tokens

Khi bạn đã tạo một personal access client, bạn có thể phát hành token cho một người dùng bằng cách sử dụng phương thức `createToken` trên instance model `User` đó. Phương thức `createToken` chấp nhận tên của token làm tham số đầu tiên và một mảng tùy chọn [scopes](#token-scopes) làm tham số thứ hai:

    use App\Models\User;

    $user = User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="personal-access-tokens-json-api"></a>
#### JSON API

Passport cũng chứa một JSON API để quản lý personal access token. Bạn có thể kết hợp api này với frontend của riêng bạn để cung cấp cho người dùng bảng điều khiển để quản lý personal access token của họ. Dưới đây, chúng ta sẽ xem qua tất cả các API endpoint để quản lý personal access token. Để thuận tiện, chúng ta sẽ sử dụng [Axios](https://github.com/mzabriskie/axios) để thực hiện các HTTP request.

JSON API được bảo vệ bởi middleware `web` và `auth`; do đó, nó chỉ có thể được gọi từ ứng dụng của bạn. Nó không thể được gọi từ một nguồn ở bên ngoài nào khác.

<a name="get-oauthscopes"></a>
#### `GET /oauth/scopes`

Route này trả về tất cả [scopes](#token-scopes) được định nghĩa cho application của bạn. Bạn có thể sử dụng route này để liệt kê scope mà người dùng có thể gán cho một personal access token:

```js
axios.get('/oauth/scopes')
    .then(response => {
        console.log(response.data);
    });
```

<a name="get-oauthpersonal-access-tokens"></a>
#### `GET /oauth/personal-access-tokens`

Route này trả về tất cả các personal access token mà người dùng hiện tại đã tạo. Điều này sẽ hữu ích khi liệt kê tất cả các token của người dùng để họ có thể chỉnh sửa hoặc huỷ bỏ chúng:

```js
axios.get('/oauth/personal-access-tokens')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthpersonal-access-tokens"></a>
#### `POST /oauth/personal-access-tokens`

Route này sẽ tạo personal access token mới. Nó đòi hỏi hai phần dữ liệu: một là `name` của token và một là `scopes` cần được gán cho token:

```js
const data = {
    name: 'Token Name',
    scopes: []
};

axios.post('/oauth/personal-access-tokens', data)
    .then(response => {
        console.log(response.data.accessToken);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="delete-oauthpersonal-access-tokenstoken-id"></a>
#### `DELETE /oauth/personal-access-tokens/{token-id}`

Route này có thể được sử dụng để huỷ bỏ personal access token:

```js
axios.delete('/oauth/personal-access-tokens/' + tokenId);
```

<a name="protecting-routes"></a>
## Bảo vệ route

<a name="via-middleware"></a>
### Thông qua middleware

Passport có chứa một [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) sẽ kiểm tra access token khi có request đến. Sau khi bạn đã cấu hình xong guard `api` sử dụng `passport` driver, bạn chỉ cần cài đặt middleware `auth:api` vào bất kỳ route nào mà yêu cầu một access token hợp lệ:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

> **Warning**
> Nếu bạn đang sử dụng [client credentials grant](#client-credentials-grant-tokens), bạn nên sử dụng [middleware `client`](#client-credentials-grant-tokens) để bảo vệ các route của bạn thay vì middleware `auth:api`.

<a name="multiple-authentication-guards"></a>
#### Multiple Authentication Guards

Nếu ứng dụng của bạn xác thực các loại người dùng khác nhau mà dùng các model Eloquent khác nhau, bạn có thể sẽ cần định nghĩa một cấu hình guard cho từng loại user providers trong ứng dụng của bạn. Điều này cho phép bạn bảo vệ các request dành cho các user providers cụ thể. Ví dụ: cho cấu hình guard của file cấu hình `config/auth.php` sau:

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],

Route sau sẽ sử dụng guard `api-customers`, sử dụng user provider `customers`, để xác thực các request đến:

    Route::get('/customer', function () {
        //
    })->middleware('auth:api-customers');

> **Note**
> Để biết thêm thông tin về cách sử dụng nhiều user provider cùng với Passport, vui lòng tham khảo thêm [tài liệu về password grant](#customizing-the-user-provider).

<a name="passing-the-access-token"></a>
### Pass access token

Khi gọi các route mà được bảo vệ bởi Passport, thì API bên thứ ba của application của bạn nên cài đặt access token của họ dưới dạng một `Bearer` token trong header `Authorization` trong request của họ. Ví dụ: khi sử dụng thư viện Guzzle HTTP:

    use Illuminate\Support\Facades\Http;

    $response = Http::withHeaders([
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ])->get('https://passport-app.test/api/user');

    return $response->json();

<a name="token-scopes"></a>
## Token scope

Scope cho phép API client của bạn yêu cầu một nhóm quyền cụ thể khi request authorization để truy cập vào tài khoản. Ví dụ: nếu bạn đang xây dựng một application thương mại điện tử, không phải tất cả API bên thứ ba nào cũng sẽ cần khả năng đặt hàng. Thay vào đó, bạn có thể cho phép bên thứ ba chỉ request authorization truy cập vào được trạng thái giao hàng. Nói cách khác, scope cho phép người dùng application của bạn giới hạn các hành động mà application của bên thứ ba có thể thực hiện.

<a name="defining-scopes"></a>
### Định nghĩa scope

Bạn có thể định nghĩa scope của API bằng phương thức `Passport::tokensCan` trong phương thức `boot` của class `App\Providers\AuthServiceProvider` của application. Phương thức `tokensCan` chấp nhận một loạt các tên scope và mô tả của nó. Mô tả scope có thể là bất cứ điều gì bạn muốn và sẽ được hiển thị cho người dùng trên màn hình phê duyệt authorization:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::tokensCan([
            'place-orders' => 'Place orders',
            'check-status' => 'Check order status',
        ]);
    }

<a name="default-scope"></a>
### Scope mặc định

Nếu một client không yêu cầu bất kỳ scope nào, bạn có thể cấu hình Passport server của bạn để gắn một scope(s) mặc định vào mã token bằng phương thức `setDefaultScope`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` trong class `App\Providers\AuthServiceProvider` của application:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

    Passport::setDefaultScope([
        'check-status',
        'place-orders',
    ]);

> **Note**
> Scope mặc định của Passport sẽ không được áp dụng cho personal access token do người dùng tạo ra.

<a name="assigning-scopes-to-tokens"></a>
### Gán scope đến token

<a name="when-requesting-authorization-codes"></a>
#### When Requesting Authorization Codes

Khi yêu cầu access token bằng cách sử dụng authorization code grant, thì bên thứ ba nên chỉ định scope mà họ mong muốn bằng tham số chuỗi truy vấn `scope`. Tham số `scope` phải là một danh sách scope đã được phân tách bằng dấu cách:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

<a name="when-issuing-personal-access-tokens"></a>
#### When Issuing Personal Access Tokens

Nếu bạn đang phát hành personal access token bằng cách sử dụng phương thức `createToken` của model `App\Models\User`, bạn có thể truyền một mảng scope mong muốn làm tham số thứ hai cho phương thức:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Kiểm tra scope

Passport có chứa hai middleware có thể được sử dụng để xác minh xem request đến đã được authenticate với một token mà đã được cấp với một scope hay chưa. Để bắt đầu, hãy thêm middleware sau vào thuộc tính `$routeMiddleware` trong file `app/Http/Kernel.php` của bạn:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

<a name="check-for-all-scopes"></a>
#### Check For All Scopes

Middleware `scopes` có thể được chỉ định cho một route để xác minh xem access token của một request đến application đã có tất cả những scope đã được liệt kê hay chưa:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);

<a name="check-for-any-scopes"></a>
#### Check For Any Scopes

Middleware `scope` có thể được chỉ định cho một route để xác minh xem access token của request đến đã có *ít nhất một* trong những scope đã được liệt kê hay chưa:

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);

<a name="checking-scopes-on-a-token-instance"></a>
#### Kiểm tra scope On A Token Instance

Khi một request được authenticate bằng access token đã vào đến application của bạn, bạn vẫn có thể kiểm tra xem token đã có scope hay chưa bằng cách sử dụng phương thức `tokenCan` trên instance `App\Models\User` đã được xác thực:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="additional-scope-methods"></a>
#### Các phương thức scope khác

Phương thức `scopeIds` sẽ trả về một mảng gồm tất cả các ID và tên đã được định nghĩa:

   use Laravel\Passport\Passport;

    Passport::scopeIds();

Phương thức `scopes` sẽ trả về một mảng gồm tất cả các scope đã được định nghĩa dưới dạng các instance của `Laravel\Passport\Scope`:

    Passport::scopes();

Phương thức `scopesFor` sẽ trả về một mảng các instance `Laravel\Passport\Scope` mà khớp với các ID và tên đã cho:

    Passport::scopesFor(['place-orders', 'check-status']);

Bạn có thể kiểm tra xem một scope nhất định đã được định nghĩa hay chưa bằng cách sử dụng phương thức `hasScope`:

    Passport::hasScope('place-orders');

<a name="consuming-your-api-with-javascript"></a>
## Sử dụng API của bạn với JavaScript

Khi xây dựng một API, nó có thể rất hữu ích khi sử dụng API của riêng bạn từ application JavaScript. Cách tiếp cận này cho phép application của bạn sử dụng cùng API mà bạn đang chia sẻ với mọi người. API tương tự cũng có thể được sử dụng bởi application web, application di động, application của bên thứ ba hoặc bất kỳ SDK nào bạn có thể publish trên các trình quản lý package khác nhau.

Thông thường, nếu bạn muốn sử dụng API từ application JavaScript của bạn, bạn cần phải tự gửi access token đến application và truyền nó theo mỗi request đến application của bạn. Tuy nhiên, Passport có chứa một middleware có thể xử lý việc này cho bạn. Tất cả những gì bạn cần làm là thêm một middleware `CreateFreshApiToken` vào middleware group `web` trong file `app/Http/Kernel.php` của bạn:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

> **Warning**
> Bạn nên đảm bảo rằng middleware `CreateFreshApiToken` sẽ được khai báo cuối cùng trong stack middleware của bạn.

Passport middleware này sẽ gán một cookie `laravel_token` vào các response gửi về cho bạn. Cookie này chứa JWT đã được mã hóa mà Passport sẽ sử dụng để xác thực các API request từ application JavaScript của bạn. JWT có thời gian tồn tại bằng với giá trị cấu hình `session.lifetime` của bạn. Bây giờ, vì trình duyệt sẽ tự động gửi cookie này cho tất cả các request tiếp theo, nên bạn có thể thực hiện các request đối với API của application mà không cần phải truyền một access token:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

<a name="customizing-the-cookie-name"></a>
#### Tùy biến tên cookie

Nếu cần, bạn có thể tùy biến tên cookie `laravel_token` bằng phương thức `Passport::cookie`. Thông thường, phương thức này sẽ được gọi từ phương thức `boot` trong class `App\Providers\AuthServiceProvider` của application của bạn:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::cookie('custom_name');
    }

<a name="csrf-protection"></a>
#### CSRF Protection

Khi sử dụng phương thức xác thực này, bạn sẽ cần đảm bảo một header CSRF token hợp lệ đã được chứa trong các request của bạn. Mặc định, Laravel JavaScript scaffolding đã chứa một instance Axios, instance này sẽ tự động sử dụng giá trị cookie `XSRF-TOKEN` được mã hóa để gửi một header `X-XSRF-TOKEN` cho các request có cùng origin.

> **Note**
> Nếu bạn chọn gửi header `X-CSRF-TOKEN` thay vì `X-XSRF-TOKEN`, bạn sẽ cần sử dụng một token chưa được mã hóa do `csrf_token()` cung cấp.

<a name="events"></a>
## Event

Passport sẽ tạo ra các event mỗi khi phát hành một access token và một refresh token. Bạn có thể sử dụng các event này để bỏ bớt hoặc thu hồi các access token khác trong cơ sở dữ liệu của bạn. Nếu bạn muốn, bạn có thể gán một listener vào các event này trong class `App\Providers\EventServiceProvider` của application của bạn:

    /**
        * The event listener mappings for the application.
        *
        * @var array
        */
    protected $listen = [
        'Laravel\Passport\Events\AccessTokenCreated' => [
            'App\Listeners\RevokeOldTokens',
        ],

        'Laravel\Passport\Events\RefreshTokenCreated' => [
            'App\Listeners\PruneOldTokens',
        ],
    ];

<a name="testing"></a>
## Test

Phương thức `actingAs` của Passport có thể được sử dụng để chỉ định một người dùng với scope của họ. Tham số đầu tiên được đưa vào cho phương thức `actingAs` là instance user và tham số thứ hai là một mảng scope được cấp cho token đó của người dùng:

    use App\Models\User;
    use Laravel\Passport\Passport;

    public function test_servers_can_be_created()
    {
        Passport::actingAs(
            User::factory()->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(201);
    }

Phương thức `actingAsClient` của Passport có thể được sử dụng để chỉ định những client hiện đang được xác thực cũng như scope của nó. Tham số đầu tiên được cung cấp cho phương thức `actingAsClient` là instance client và tham số thứ hai là một mảng scope sẽ được cấp cho token của client đó:

    use Laravel\Passport\Client;
    use Laravel\Passport\Passport;

    public function test_orders_can_be_retrieved()
    {
        Passport::actingAsClient(
            Client::factory()->create(),
            ['check-status']
        );

        $response = $this->get('/api/orders');

        $response->assertStatus(200);
    }
