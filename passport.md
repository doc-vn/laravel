# Laravel Passport

- [Giới thiệu](#introduction)
- [Cập nhật Passport](#upgrading)
- [Cài đặt](#installation)
    - [Frontend Quickstart](#frontend-quickstart)
    - [Deploy Passport](#deploying-passport)
- [Cấu hình](#configuration)
    - [Thời gian sống token](#token-lifetimes)
    - [Ghi đè các model mặc định](#overriding-default-models)
- [Phát hành access token](#issuing-access-tokens)
    - [Quản lý client](#managing-clients)
    - [Request token](#requesting-tokens)
    - [Refresh token](#refreshing-tokens)
    - [Lọc token](#purging-tokens)
- [Authorization Code Grant với PKCE](#code-grant-pkce)
    - [Tạo client](#creating-a-auth-pkce-grant-client)
    - [Request token](#requesting-auth-pkce-grant-tokens)
- [Token password grant](#password-grant-tokens)
    - [Tạo một password grant client](#creating-a-password-grant-client)
    - [Request token](#requesting-password-grant-tokens)
    - [Request all scope](#requesting-all-scopes)
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

Laravel đã giúp bạn dễ dàng thực hiện authentication thông qua các form đăng nhập truyền thống, nhưng còn API thì sao? API thường sử dụng token để authenticate người dùng và không duy trì trạng thái session giữa các request. Laravel giúp authenticate API dễ dàng bằng cách sử dụng Laravel Passport, cung cấp một implementation OAuth2 server đầy đủ cho application Laravel của bạn trong vài phút. Passport được xây dựng trên top của [League OAuth2 server](https://github.com/thephpleague/oauth2-server) được duy trì bởi Andy Millington và Simon Hamp.

> {note} Tài liệu này giả định rằng bạn đã biết OAuth2. Nếu bạn chưa biết về OAuth2, hãy xem xét việc tự học với các [thuật ngữ](https://oauth2.thephpleague.com/terminology/) và tính năng chung của OAuth2 trước khi tiếp tục.

<a name="upgrading"></a>
## Cập nhật Passport

Khi nâng cấp lên phiên bản mới của Passport, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/passport/blob/master/UPGRADE.md).

<a name="installation"></a>
## Cài đặt

Để bắt đầu, hãy cài đặt Passport thông qua Composer package manager:

    composer require laravel/passport

Passport service provider sẽ đăng ký thư mục database migration của riêng nó với framework, nên vì thế bạn nên migrate cơ sở dữ liệu của bạn sau khi cài đặt xong package. Việc migrate của Passport sẽ tạo ra các table mà application của bạn cần để lưu trữ client và access token:

    php artisan migrate

Tiếp theo, bạn nên chạy lệnh `passport:install`. Lệnh này sẽ tạo các key mã hóa cần thiết để tạo secure access token. Ngoài ra, lệnh này cũng sẽ tạo các "personal access" và các "password grant" client được sử dụng để tạo access token:

    php artisan passport:install

Sau khi chạy lệnh này, hãy thêm trait `Laravel\Passport\HasApiTokens` vào model `App\User` của bạn. Trait này sẽ cung cấp một vài phương thức helper cho model của bạn, cho phép bạn kiểm tra token và phạm vi của người dùng đã được authenticate:

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

Tiếp theo, bạn nên gọi phương thức `Passport::routes` vào trong phương thức `boot` của `AuthServiceProvider` của bạn. Phương thức này sẽ đăng ký các route cần thiết để phát hành các access token và thu hồi các access token, client và các access token cá nhân:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;
    use Laravel\Passport\Passport;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            'App\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * Register any authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Passport::routes();
        }
    }

Cuối cùng, trong file cấu hình `config/auth.php` của bạn, bạn nên set tùy chọn `driver` của `api` authentication guard thành `passport`. Điều này sẽ hướng dẫn application của bạn sử dụng `TokenGuard` của Passport khi authenticate các request API:

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

### Tuỳ biến migration

Nếu bạn không sử dụng migration mặc định của Passport, bạn cần gọi phương thức `Passport::ignoreMigrations` trong phương thức `register` của `AppServiceProvider` của bạn. Bạn có thể export các migration mặc định này bằng cách sử dụng `php artisan vendor:publish --tag=passport-migrations`.

Mặc định, Passport sẽ sử dụng một cột số nguyên để lưu trữ `user_id`. Nếu ứng dụng của bạn sử dụng một loại cột khác để xác định người dùng (ví dụ như: UUID), thì bạn cần sửa các migration mặc định của Passport sau khi export chúng.

<a name="frontend-quickstart"></a>
### Frontend Quickstart

> {note} Để sử dụng các component Passport Vue, bạn phải sử dụng JavaScript [Vue](https://vuejs.org) framework. Các component này cũng sử dụng Bootstrap CSS framework. Tuy nhiên, ngay cả khi bạn không sử dụng các công cụ này, thì các component cũng đóng vai trò là tài liệu tham khảo có giá trị cho việc triển khai frontend của chính bạn.

Passport đi kèm với một số JSON API mà bạn có thể sử dụng để cho phép người dùng tạo client và access token cá nhân. Tuy nhiên, bạn có thể mất thời gian để tạo một frontend có thể tương tác với các API này. Vì vậy, Passport cũng chứa các component [Vue](https://vuejs.org) được xây dựng sẵn mà bạn có thể sử dụng làm ví dụ để triển khai hoặc là điểm bắt đầu cho việc triển khai của bạn.

Để publish component Passport Vue, hãy sử dụng lệnh Artisan `vendor:publish`:

    php artisan vendor:publish --tag=passport-components

Các thành phần được publish sẽ được lưu trong thư mục `resources/js/components` của bạn. Khi các thành phần đã được publish, bạn nên đăng ký chúng vào trong file `resources/js/app.js`:

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue').default
    );

    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue').default
    );

    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue').default
    );

> {note} Trước Laravel v5.7.19, việc thêm `.default` khi đăng ký các component sẽ dẫn đến một lỗi trong console. Để giải thích cho thay đổi này bạn có thể được tìm thấy trong [release note Laravel Mix v4.0.0](https://github.com/JeffreyWay/laravel-mix/releases/tag/v4.0.0).

Sau khi đăng ký các component, hãy chạy `npm run dev` để biên dịch lại các assets của bạn. Khi bạn đã biên dịch lại các assets của bạn, bạn có thể dùng các component này vào một trong các template của application để bắt đầu tạo client và access token cá nhân:

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="deploying-passport"></a>
### Deploying Passport

Khi deploy Passport lần đầu đến server production của bạn, bạn có thể sẽ cần chạy lệnh `passport:keys`. Lệnh này sẽ tạo các key mã hóa Passport cần, để tạo access token. Các key được tạo thường không nên được lưu trữ trong source code control:

    php artisan passport:keys

Nếu cần, bạn có thể định nghĩa đường dẫn nơi mà các khóa của Passport sẽ được load từ đó. Bạn có thể sử dụng phương thức `Passport::loadKeysFrom` để thực hiện việc này:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::loadKeysFrom('/secret-keys/oauth');
    }

Ngoài ra, bạn có thể export file cấu hình của Passport bằng cách sử dụng `php artisan vendor:publish --tag=passport-config`, sau đó sẽ cung cấp tùy chọn để load các khóa mã hóa từ các biến môi trường của bạn:

    PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    <private key here>
    -----END RSA PRIVATE KEY-----"

    PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
    <public key here>
    -----END PUBLIC KEY-----"

<a name="configuration"></a>
## Cấu hình

<a name="token-lifetimes"></a>
### Thời gian sống token

Mặc định, Passport phát hành các access token tồn tại lâu dài có thời hạn một năm. Nếu bạn muốn cấu hình vòng đời token dài hoặc ngắn hơn, bạn có thể sử dụng các phương thức `tokensExpireIn`, `refreshTokensExpireIn`, và `personalAccessTokensExpireIn`. Các phương thức này phải được gọi từ phương thức `boot` của `AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(now()->addDays(15));

        Passport::refreshTokensExpireIn(now()->addDays(30));

        Passport::personalAccessTokensExpireIn(now()->addMonths(6));
    }

<a name="overriding-default-models"></a>
### Ghi đè các model mặc định

Bạn có thể thoải mái mở rộng các model được sử dụng trong nội bộ bằng Passport:

    use Laravel\Passport\Client as PassportClient;

    class Client extends PassportClient
    {
        // ...
    }

Sau đó, bạn có thể hướng dẫn Passport sử dụng các model tùy biến này thông qua class `Passport`:

    use App\Models\Passport\AuthCode;
    use App\Models\Passport\Client;
    use App\Models\Passport\PersonalAccessClient;
    use App\Models\Passport\Token;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::useTokenModel(Token::class);
        Passport::useClientModel(Client::class);
        Passport::useAuthCodeModel(AuthCode::class);
        Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
    }

<a name="issuing-access-tokens"></a>
## Phát hành token truy cập

Sử dụng OAuth2 với authorization code là cách mà hầu hết các nhà phát triển quen thuộc với OAuth2. Khi sử dụng authorization code, một client application sẽ chuyển hướng người dùng đến server của bạn, nơi mà người dùng sẽ chấp nhận hoặc từ chối request cấp access token cho client.

<a name="managing-clients"></a>
### Quản lý client

Đầu tiên, các nhà phát triển cần xây dựng các ứng dụng của họ, mà cần tương tác với API application của bạn, họ sẽ cần phải đăng ký ứng dụng của họ với application của bạn bằng cách tạo "client". Thông thường, việc này bao gồm việc cung cấp tên ứng dụng và URL mà application của bạn cần chuyển hướng người dùng đến sau khi người dùng đã chấp thuận request ủy quyền.

#### Lệnh `passport:client`

Cách đơn giản nhất để tạo một client là sử dụng lệnh Artisan `passport:client`. Lệnh này có thể được sử dụng để tạo các client của riêng bạn để test các chức năng OAuth2. Khi bạn chạy lệnh `client`, Passport sẽ hỏi bạn cho biết thêm thông tin về client của bạn và trả về cho bạn một client ID và một secret:

    php artisan passport:client

**Redirect URLs**

Nếu bạn muốn lập một danh sách whitelist gồm nhiều URL chuyển hướng cho client của bạn, bạn có thể chỉ định chúng bằng cách sử dụng một danh sách được phân cách bằng dấu phẩy khi nhập URL bằng lệnh `passport:client`:

    http://example.com/callback,http://examplefoo.com/callback

> {note} Bất kỳ URL nào mà chứa dấu phẩy đều phải được encoded.

#### JSON API

Vì người dùng của bạn sẽ không thể sử dụng lệnh `client`, nên Passport cũng cung cấp một JSON API mà bạn có thể sử dụng để tạo client. Điều này giúp bạn tránh những rắc rối khi phải tự viết controller để tạo, cập nhật và xóa client.

Tuy nhiên, bạn sẽ cần kết nối JSON API của Passport với frontend của bạn để cung cấp một bảng điều khiển cho người dùng biết và quản lý các client của họ. Dưới đây, chúng ta sẽ xem xét tất cả các API endpoint để quản lý client. Để thuận tiện, chúng ta sẽ sử dụng [Axios](https://github.com/axios/axios) để thực hiện các HTTP request đến các endpoint.

JSON API được bảo vệ bởi middleware `web` và `auth`; do đó, nó chỉ có thể được gọi từ ứng dụng của bạn. Nó không thể được gọi từ một nguồn ở bên ngoài nào khác.

> {tip} Nếu bạn không muốn tự mình thực hiện toàn bộ công việc quản lý client, bạn có thể sử dụng [frontend quickstart](#frontend-quickstart) để có một frontend đầy đủ các chức năng trong vài phút.

#### `GET /oauth/clients`

Route này sẽ trả về tất cả các client cho người dùng đã được authenticate. Điều này chủ yếu hữu ích để liệt kê tất cả các client của người dùng để họ có thể chỉnh sửa hoặc xóa chúng:

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

Route này được sử dụng để tạo client mới. Nó đòi hỏi hai phần dữ liệu: một là `name` của client và một URL `redirect`. URL `redirect` là nơi người dùng sẽ được chuyển hướng đến sau khi chấp nhận hoặc từ chối một request cho authorization.

Sau Khi một client đã được tạo, nó sẽ được cũng cấp cho một client ID và một client secret. Các giá trị này sẽ được sử dụng khi yêu cầu access token từ application của bạn. Route tạo client sẽ trả về instance client mới:

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

#### `PUT /oauth/clients/{client-id}`

Route này được sử dụng để cập nhật client. Nó đòi hỏi hai phần dữ liệu: một là `name` của client và một URL `redirect`. URL `redirect` là nơi người dùng sẽ được chuyển hướng đến sau khi chấp nhận hoặc từ chối một request cho authorization. Route sẽ trả về instance client đã được cập nhật:

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

#### `DELETE /oauth/clients/{client-id}`

Route này được sử dụng để xóa client:

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### Request token

#### Redirecting For Authorization

Khi một client đã được tạo, các developer có thể sử dụng client ID và secret được trả về để yêu cầu authorization code và access token từ application của bạn. Đầu tiên, application của bên thứ ba sẽ tạo một yêu cầu chuyển hướng đến route `/oauth/authorize` của application của bạn như sau:

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Hãy nhớ rằng, route `/oauth/authorize` đã được định nghĩa bởi phương thức `Passport::routes`. Bạn không cần phải tự định nghĩa route này.

#### Approving The Request

Khi nhận được authorization request, Passport sẽ tự động hiển thị một template cho người dùng để họ có thể chấp nhận hoặc từ chối authorization request. Nếu họ chấp nhận request, họ sẽ được chuyển hướng trở lại `redirect_uri` sẽ được chỉ định bởi application của bên thứ ba. `redirect_uri` phải khớp với URL `redirect` được chỉ định khi client được tạo.

Nếu bạn muốn tùy chỉnh màn hình phê duyệt authorization, bạn có thể publish các view của Passport bằng cách sử dụng lệnh Artisan `vendor:publish`. Các view được publish sẽ được lưu trong `resources/views/vendor/passport`:

    php artisan vendor:publish --tag=passport-views

Thỉnh thoảng bạn có thể muốn bỏ qua các lời nhắc cấp quyền, chẳng hạn như khi cấp quyền cho client bên thứ nhất. Bạn có thể thực hiện điều này bằng cách [extending the `Client` model](#overriding-default-models) và định nghĩa phương thức `skipsAuthorization`. Nếu `skipsAuthorization` trả về `true` thì ứng dụng client sẽ được chấp thuận và người dùng sẽ được chuyển hướng trở lại về `redirect_uri` ngay lập tức:

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

#### Converting Authorization Codes To Access Tokens

Nếu người dùng chấp nhận authorization request, họ sẽ được chuyển hướng trở lại application của bên thứ ba. Sau đó, đầu tiên, bên thứ ba sẽ kiểm tra tham số `state` với giá trị đã được lưu trữ trước khi chuyển hướng. Nếu tham số state trùng khớp với giá trị đã được lưu, nó sẽ đưa ra một request `POST` cho application của bạn để yêu cầu access token. Yêu cầu phải chứa authorization code được cấp bởi application của bạn khi người dùng chấp nhận authorization request. Trong ví dụ này, chúng ta sẽ sử dụng thư viện Guzzle HTTP để thực hiện request `POST`:

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $http = new GuzzleHttp\Client;

        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

Route `/oauth/token` này sẽ trả về một JSON response có chứa các thuộc tính `access_token`, `refresh_token` và `expires_in`. Thuộc tính `expires_in` sẽ chứa số giây cho đến khi access token hết hạn.

> {tip} Giống như route `/oauth/authorize`, route `/oauth/token` đã được định nghĩa cho bạn bằng phương thức `Passport::routes`. Bạn không cần phải tự định nghĩa route này. Mặc định, route này được điều tiết bằng cách sử dụng cài đặt của middleware `ThrottleRequests`.

<a name="refreshing-tokens"></a>
### Refresh token

Nếu application của bạn phát hành access token ngắn hạn, người dùng sẽ cần phải refresh access token của họ thông qua refresh token được cung cấp cho họ khi access token được phát hành. Trong ví dụ này, chúng ta sẽ sử dụng thư viện Guzzle HTTP để refresh token:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

Route `/oauth/token` này sẽ trả về một JSON response có chứa các thuộc tính `access_token`, `refresh_token` và `expires_in`. Thuộc tính `expires_in` sẽ chứa số giây cho đến khi access token hết hạn.

<a name="purging-tokens"></a>
### Lọc token

Khi token bị thu hồi hoặc bị hết hạn, bạn có thể muốn xóa chúng ra khỏi cơ sở dữ liệu. Passport có kèm theo một lệnh có thể thực hiện việc này cho bạn:

    # Purge revoked and expired tokens and auth codes...
    php artisan passport:purge

    # Only purge revoked tokens and auth codes...
    php artisan passport:purge --revoked

    # Only purge expired tokens and auth codes...
    php artisan passport:purge --expired

Bạn cũng có thể cấu hình một [scheduled job](/docs/{{version}}/scheduling) trong class `Kernel` trong thư mục console để tự động lọc token của bạn theo một schedule:

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

Trước khi ứng dụng của bạn có thể phát hành token thông qua authorization code grant với PKCE, bạn sẽ cần tạo một ứng dụng client hỗ trợ PKCE. Bạn có thể thực hiện việc này bằng lệnh `passport:client` với tùy chọn `--public`:

    php artisan passport:client --public

<a name="requesting-auth-pkce-grant-tokens"></a>
### Request token

#### Code Verifier & Code Challenge

Vì authorization grant này không cung cấp một client secret, nên các nhà phát triển sẽ cần phải tạo ra một code verifier và một code challenge để yêu cầu token.

Code verifier phải là một chuỗi ngẫu nhiên từ 43 đến 128 ký tự chứa các chữ cái, số và `"-"`, `"."`, `"_"`, `"~"`, như được định nghĩa trong [tài liệu RFC 7636 đặc điểm kỹ thuật](https://tools.ietf.org/html/rfc7636).

Code challenge phải là một chuỗi được mã hóa Base64 với URL và các ký tự an toàn cho tên file. Các ký tự ở cuối dấu `'='` phải được loại bỏ và không được có dấu ngắt dòng, khoảng trắng hoặc các ký tự bổ sung khác.

    $encoded = base64_encode(hash('sha256', $code_verifier, true));

    $codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');

#### Redirecting For Authorization

Sau khi một ứng dụng client đã được tạo xong, bạn có thể sử dụng ID của ứng dụng client đó và code verifier và code challenge đã tạo để yêu cầu một authorization code và một access token từ ứng dụng của bạn. Đầu tiên, ứng dụng đang sử dụng phải thực hiện một request chuyển hướng đến route `/oauth/authorize` của ứng dụng của bạn:

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $request->session()->put('code_verifier', $code_verifier = Str::random(128));

        $codeChallenge = strtr(rtrim(
            base64_encode(hash('sha256', $code_verifier, true))
        , '='), '+/', '-_');

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            'code_challenge' => $codeChallenge,
            'code_challenge_method' => 'S256',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### Converting Authorization Codes To Access Tokens

Nếu người dùng chấp thuận yêu cầu authorization, họ sẽ được chuyển hướng trở lại ứng dụng mà họ đang sử dụng. Người dùng api của bạn nên xác thực thông số `state` so với giá trị đã được lưu trữ trước khi được chuyển hướng, như trong Authorization Code Grant tiêu chuẩn.

Nếu thông số state khớp, Người dùng api của bạn nên đưa ra một request `POST` cho ứng dụng của bạn để yêu cầu một access token. Yêu cầu này phải chứa authorization code do ứng dụng của bạn cấp khi người dùng chấp thuận yêu cầu authorization cùng với code verifier đã được tạo ra ban đầu:

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        $codeVerifier = $request->session()->pull('code_verifier');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = (new GuzzleHttp\Client)->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'redirect_uri' => 'http://example.com/callback',
                'code_verifier' => $codeVerifier,
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

<a name="password-grant-tokens"></a>
## Token password grant

OAuth2 password grant cho phép các client bên thứ nhất, chẳng hạn như một application mobile trong tổ chức của bạn, có được access token bằng địa chỉ email hoặc tên người dùng và mật khẩu của họ. Điều này cho phép bạn phát hành access token một cách an toàn cho client bên thứ nhất mà không yêu cầu người dùng của bạn thực hiện toàn bộ các luồng chuyển hướng OAuth2 authorization code.

<a name="creating-a-password-grant-client"></a>
### Tạo một password grant client

Trước khi application của bạn có thể phát hành token thông qua password grant, bạn sẽ cần phải tạo một password grant client. Bạn có thể làm điều này bằng cách sử dụng lệnh `passport:client` với tùy chọn `--password`. Nếu bạn đã chạy lệnh `passport:install`, thì bạn không cần phải chạy lệnh này:

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### Request token

Khi bạn đã tạo một password grant client, bạn có thể yêu cầu access token bằng cách đưa ra một request `POST` cho route `/oauth/token` với địa chỉ email và mật khẩu của người dùng. Hãy nhớ rằng, route này đã được đăng ký bằng phương thức `Passport::routes` nên bạn không cần phải định nghĩa lại chúng. Nếu yêu cầu thành công, bạn sẽ nhận được một `access_token` và một `refresh_token` trong JSON response từ server:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

> {tip} Hãy nhớ rằng, access token sẽ mặc định là tồn tại mãi mãi. Tuy nhiên, bạn có thể thoải mái [cấu hình maximum vòng đời access token của bạn](#configuration) nếu cần.

<a name="requesting-all-scopes"></a>
### Yêu cầu tất cả scope

Khi sử dụng password grant hoặc chứng chỉ client grant, bạn có thể muốn ủy quyền token cho tất cả các scope được application của bạn hỗ trợ. Bạn có thể làm điều này bằng cách yêu cầu scope `*`. Nếu bạn yêu cầu scope là `*`, thì phương thức `can` trên instance token sẽ luôn trả về `true`. Scope này chỉ có thể được gán cho những token mà được cấp bằng `password` hoặc `client_credentials` grant`:

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="customizing-the-username-field"></a>
### Tuỳ biến field username

Khi xác thực bằng password grant, Passport sẽ sử dụng thuộc tính `email` của model của bạn làm "username". Tuy nhiên, bạn có thể tùy chỉnh hành động này bằng cách định nghĩa phương thức `findForPassport` trên model của bạn:

    <?php

    namespace App;

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
         * @return \App\User
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

    namespace App;

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

Grant ẩn tương tự như authorization code grant; tuy nhiên, token được trả về cho client mà không cần thông qua authorization code. Grant này được sử dụng phổ biến nhất cho các application JavaScript hoặc mobile application nơi mà thông tin đăng nhập của client không thể được lưu trữ an toàn. Để kích hoạt grant, hãy gọi phương thức `enableImplicitGrant` trong `AuthServiceProvider` của bạn:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }

Khi grant này đã được bật, nhà phát triển có thể sử dụng client ID của chính họ để yêu cầu access token từ application của bạn. Ứng dụng của nhà phát triển sẽ tạo một yêu cầu chuyển hướng đến route `/oauth/authorize` của application của bạn như sau:

   Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
            'state' => $state,
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Hãy nhớ rằng, route `/oauth/authorize` đã được định nghĩa bởi phương thức `Passport::routes`. Bạn không cần phải định nghĩa route này.

<a name="client-credentials-grant-tokens"></a>
## Token chứng chỉ client grant

Chứng chỉ client grant thích hợp cho việc authentication machine-to-machine. Ví dụ: bạn có thể sử dụng grant này trong một scheduled job đang thực hiện công việc bảo trì qua API.

Trước khi ứng dụng của bạn có thể phát hành mã token thông qua chứng chỉ client grant, bạn sẽ cần tạo một client chứng chỉ client grant. Bạn có thể thực hiện việc này bằng cách sử dụng tùy chọn `--client` trong lệnh `passport:client`:

    php artisan passport:client --client

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

### Retrieving Tokens

Để lấy một token của một loại grant này, hãy tạo một request đến `oauth/token` endpoint:

    $guzzle = new GuzzleHttp\Client;

    $response = $guzzle->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'client_credentials',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => 'your-scope',
        ],
    ]);

    return json_decode((string) $response->getBody(), true)['access_token'];

<a name="personal-access-tokens"></a>
## Personal Access Tokens

Đôi khi, người dùng của bạn có thể muốn phát hành access token cho chính họ mà không cần thông qua luồng chuyển hướng authorization code thông thường. Việc cho phép người dùng phát hành token cho chính họ thông qua giao diện người dùng của application của bạn có thể hữu ích khi cho phép người dùng thử nghiệm API của bạn hoặc có thể dùng như một cách tiếp cận đơn giản hơn khi phát hành access token nói chung.

<a name="creating-a-personal-access-client"></a>
### Tạo một Personal Access Client

Trước khi application của bạn có thể phát hành một personal access token, bạn sẽ cần tạo một personal access client. Bạn có thể làm điều này bằng cách sử dụng lệnh `passport:client` với tùy chọn `--personal`. Nếu bạn đã chạy lệnh `passport:install`, bạn không cần chạy lệnh này:

    php artisan passport:client --personal

Nếu bạn đã định nghĩa một personal access client, bạn có thể hướng dẫn Passport sử dụng client đó bằng phương thức `personalAccessClientId`. Thông thường, phương thức này sẽ được gọi từ phương thức `boot` trong `AuthServiceProvider` của bạn:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::personalAccessClientId('client-id');
    }

<a name="managing-personal-access-tokens"></a>
### Quản lý Personal Access Tokens

Khi bạn đã tạo một personal access client, bạn có thể phát hành token cho một người dùng bằng cách sử dụng phương thức `createToken` trên instance model `User` đó. Phương thức `createToken` chấp nhận tên của token làm tham số đầu tiên và một mảng tùy chọn [scopes](#token-scopes) làm tham số thứ hai:

    $user = App\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport cũng chứa một JSON API để quản lý personal access token. Bạn có thể kết hợp api này với frontend của riêng bạn để cung cấp cho người dùng bảng điều khiển để quản lý personal access token của họ. Dưới đây, chúng ta sẽ xem qua tất cả các API endpoint để quản lý personal access token. Để thuận tiện, chúng ta sẽ sử dụng [Axios](https://github.com/mzabriskie/axios) để thực hiện các HTTP request.

JSON API được bảo vệ bởi middleware `web` và `auth`; do đó, nó chỉ có thể được gọi từ ứng dụng của bạn. Nó không thể được gọi từ một nguồn ở bên ngoài nào khác.

> {tip} Nếu bạn không muốn tự phát triển frontend riêng của bạn, bạn có thể sử dụng [frontend quickstart](#frontend-quickstart) để  có thể có được một frontend đầy đủ chức năng trong vài phút.

#### `GET /oauth/scopes`

Route này trả về tất cả [scopes](#token-scopes) được định nghĩa cho application của bạn. Bạn có thể sử dụng route này để liệt kê scope mà người dùng có thể gán cho một personal access token:

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

Route này trả về tất cả các personal access token mà người dùng hiện tại đã tạo. Điều này sẽ hữu ích khi liệt kê tất cả các token của người dùng để họ có thể chỉnh sửa hoặc xóa chúng:

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

Route này sẽ tạo personal access token mới. Nó đòi hỏi hai phần dữ liệu: một là `name` của token và một là `scopes` cần được gán cho token:

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

#### `DELETE /oauth/personal-access-tokens/{token-id}`

Route này có thể được sử dụng để xóa personal access token:

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## Bảo vệ route

<a name="via-middleware"></a>
### Thông qua middleware

Passport có chứa một [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) sẽ kiểm tra access token khi có request đến. Sau khi bạn đã cấu hình xong guard `api` sử dụng `passport` driver, bạn chỉ cần cài đặt middleware `auth:api` vào bất kỳ route nào mà bạn cần một access token hợp lệ:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### Pass access token

Khi gọi các route mà được bảo vệ bởi Passport, thì API bên thứ ba của application của bạn nên cài đặt access token của họ dưới dạng một `Bearer` token trong header `Authorization` trong request của họ. Ví dụ: khi sử dụng thư viện Guzzle HTTP:

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## Token scope

Scope cho phép API client của bạn yêu cầu một nhóm quyền cụ thể khi request authorization để truy cập vào tài khoản. Ví dụ: nếu bạn đang xây dựng một application thương mại điện tử, không phải tất cả API bên thứ ba nào cũng sẽ cần khả năng đặt hàng. Thay vào đó, bạn có thể cho phép bên thứ ba chỉ request authorization truy cập vào được trạng thái giao hàng. Nói cách khác, scope cho phép người dùng application của bạn giới hạn các hành động mà application của bên thứ ba có thể thực hiện.

<a name="defining-scopes"></a>
### Định nghĩa scope

Bạn có thể định nghĩa scope của API bằng phương thức `Passport::tokensCan` trong phương thức `boot` của `AuthServiceProvider`. Phương thức `tokensCan` chấp nhận một loạt các tên scope và mô tả của nó. Mô tả scope có thể là bất cứ điều gì bạn muốn và sẽ được hiển thị cho người dùng trên màn hình phê duyệt authorization:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="default-scope"></a>
### Scope mặc định

Nếu một client không yêu cầu bất kỳ scope nào, bạn có thể cấu hình Passport server của bạn để gắn một scope mặc định vào mã token bằng phương thức `setDefaultScope`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` trong `AuthServiceProvider`:

    use Laravel\Passport\Passport;

    Passport::setDefaultScope([
        'check-status',
        'place-orders',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### Gán scope đến token

#### When Requesting Authorization Codes

Khi yêu cầu access token bằng cách sử dụng authorization code grant, thì bên thứ ba nên chỉ định scope mà họ mong muốn bằng tham số chuỗi truy vấn `scope`. Tham số `scope` phải là một danh sách scope đã được phân tách bằng dấu cách:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### When Issuing Personal Access Tokens

Nếu bạn đang phát hành personal access token bằng cách sử dụng phương thức `createToken` của model `User`, bạn có thể truyền một mảng scope mong muốn làm tham số thứ hai cho phương thức:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Kiểm tra scope

Passport có chứa hai middleware có thể được sử dụng để xác minh xem request đến đã được authenticate với một token mà đã được cấp với một scope hay chưa. Để bắt đầu, hãy thêm middleware sau vào thuộc tính `$routeMiddleware` trong file `app/Http/Kernel.php` của bạn:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### Check For All Scopes

Middleware `scopes` có thể được chỉ định cho một route để xác minh xem access token của request đến đã có *tất cả* những scope đã được liệt kê hay chưa:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);

#### Check For Any Scopes

Middleware `scope` có thể được chỉ định cho một route để xác minh xem access token của request đến đã có *ít nhất một* trong những scope đã được liệt kê hay chưa:

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);

#### Kiểm tra scope On A Token Instance

Khi một request được authenticate bằng access token đã vào đến application của bạn, bạn vẫn có thể kiểm tra xem token đã có scope hay chưa bằng cách sử dụng phương thức `tokenCan` trên instance `User` đã được xác thực:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

#### Các phương thức scope khác

Phương thức `scopeIds` sẽ trả về một mảng gồm tất cả các ID và tên đã được định nghĩa:

    Laravel\Passport\Passport::scopeIds();

Phương thức `scopes` sẽ trả về một mảng gồm tất cả các scope đã được định nghĩa dưới dạng các instance của `Laravel\Passport\Scope`:

    Laravel\Passport\Passport::scopes();

Phương thức `scopesFor` sẽ trả về một mảng các instance `Laravel\Passport\Scope` mà khớp với các ID và tên đã cho:

    Laravel\Passport\Passport::scopesFor(['place-orders', 'check-status']);

Bạn có thể kiểm tra xem một scope nhất định đã được định nghĩa hay chưa bằng cách sử dụng phương thức `hasScope`:

    Laravel\Passport\Passport::hasScope('place-orders');

<a name="consuming-your-api-with-javascript"></a>
## Sử dụng API của bạn với JavaScript

Khi xây dựng một API, nó có thể rất hữu ích khi sử dụng API của riêng bạn từ application JavaScript. Cách tiếp cận này cho phép application của bạn sử dụng cùng API mà bạn đang chia sẻ với mọi người. API tương tự cũng có thể được sử dụng bởi application web, application di động, application của bên thứ ba hoặc bất kỳ SDK nào bạn có thể publish trên các trình quản lý package khác nhau.

Thông thường, nếu bạn muốn sử dụng API từ application JavaScript của bạn, bạn cần phải tự gửi access token đến application và truyền nó theo mỗi request đến application của bạn. Tuy nhiên, Passport có chứa một middleware có thể xử lý việc này cho bạn. Tất cả những gì bạn cần làm là thêm một middleware `CreateFreshApiToken` vào middleware group `web` trong file `app/Http/Kernel.php` của bạn:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

> {note} Bạn nên đảm bảo rằng middleware `CreateFreshApiToken` sẽ được khai báo cuối cùng trong stack middleware của bạn.

Passport middleware này sẽ gán một cookie `laravel_token` vào các response gửi về cho bạn. Cookie này chứa JWT đã được mã hóa mà Passport sẽ sử dụng để xác thực các API request từ application JavaScript của bạn. Bây giờ, bạn có thể thực hiện các request đối với API của application mà không cần phải truyền một access token:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

#### Tùy biến tên cookie

Nếu cần, bạn có thể tùy biến tên cookie `laravel_token` bằng phương thức `Passport::cookie`. Thông thường, phương thức này sẽ được gọi từ phương thức `boot` trong `AuthServiceProvider` của bạn:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::cookie('custom_name');
    }

#### CSRF Protection

Khi sử dụng phương thức xác thực này, bạn sẽ cần đảm bảo một header CSRF token hợp lệ đã được chứa trong các request của bạn. Mặc định, Laravel JavaScript scaffolding đã chứa một instance Axios, instance này sẽ tự động sử dụng giá trị cookie `XSRF-TOKEN` được mã hóa để gửi header `X-XSRF-TOKEN` trên các request có cùng origin.

> {tip} Nếu bạn chọn gửi header `X-CSRF-TOKEN` thay vì `X-XSRF-TOKEN`, bạn sẽ cần sử dụng một token chưa được mã hóa do `csrf_token()` cung cấp.

<a name="events"></a>
## Event

Passport sẽ tạo ra các event mỗi khi phát hành một access token và một refresh token. Bạn có thể sử dụng các event này để bỏ bớt hoặc thu hồi các access token khác trong cơ sở dữ liệu của bạn. Bạn có thể gán listener vào các event này trong `EventServiceProvider`:

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

    use App\User;
    use Laravel\Passport\Passport;

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(201);
    }

Phương thức `actingAsClient` của Passport có thể được sử dụng để chỉ định những client hiện đang được xác thực cũng như scope của nó. Tham số đầu tiên được cung cấp cho phương thức `actingAsClient` là instance client và tham số thứ hai là một mảng scope sẽ được cấp cho token của client đó:

    use Laravel\Passport\Client;
    use Laravel\Passport\Passport;

    public function testGetOrders()
    {
        Passport::actingAsClient(
            factory(Client::class)->create(),
            ['check-status']
        );

        $response = $this->get('/api/orders');

        $response->assertStatus(200);
    }
