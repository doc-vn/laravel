# API Authentication (Passport)

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Frontend Quickstart](#frontend-quickstart)
    - [Deploy Passport](#deploying-passport)
- [Cấu hình](#configuration)
    - [Thời gian sống token](#token-lifetimes)
- [Phát hành access token](#issuing-access-tokens)
    - [Quản lý client](#managing-clients)
    - [Request token](#requesting-tokens)
    - [Refresh token](#refreshing-tokens)
- [Token password grant](#password-grant-tokens)
    - [Tạo một password grant client](#creating-a-password-grant-client)
    - [Request token](#requesting-password-grant-tokens)
    - [Request all scope](#requesting-all-scopes)
- [Token với grant ẩn](#implicit-grant-tokens)
- [Token thông tin client grant](#client-credentials-grant-tokens)
- [Access token cá nhân](#personal-access-tokens)
    - [Tạo một access client cá nhân](#creating-a-personal-access-client)
    - [Quản lý access token cá nhân](#managing-personal-access-tokens)
- [Bảo vệ route](#protecting-routes)
    - [Thông qua middleware](#via-middleware)
    - [Pass access token](#passing-the-access-token)
- [Token scope](#token-scopes)
    - [Định nghĩa scope](#defining-scopes)
    - [Gán scope đến token](#assigning-scopes-to-tokens)
    - [Kiểm tra scope](#checking-scopes)
- [Sử dụng API của bạn với JavaScript](#consuming-your-api-with-javascript)
- [Event](#events)
- [Test](#testing)

<a name="introduction"></a>
## Giới thiệu

Laravel đã giúp bạn dễ dàng thực hiện authentication thông qua các form đăng nhập truyền thống, nhưng còn API thì sao? API thường sử dụng token để authenticate người dùng và không duy trì trạng thái session giữa các request. Laravel giúp authenticate API dễ dàng bằng cách sử dụng Laravel Passport, cung cấp implementation hoàn toàn OAuth2 server cho application Laravel của bạn trong vài phút. Passport được xây dựng trên top [League OAuth2 server](https://github.com/thephpleague/oauth2-server) được duy trì bởi Alex Bilbie.

> {note} Tài liệu này giả định rằng bạn đã biết OAuth2. Nếu bạn không biết gì về OAuth2, hãy xem xét việc tự học với các thuật ngữ và tính năng chung của OAuth2 trước khi tiếp tục.

<a name="installation"></a>
## Cài đặt

Để bắt đầu, hãy cài đặt Passport thông qua Composer package manager:

    composer require laravel/passport=~4.0

Passport service provider sẽ đăng ký thư mục database migration của nó với framework, vì vậy bạn nên migrate cơ sở dữ liệu của bạn sau khi đăng ký provider. Việc migrate của Passport sẽ tạo các table mà application của bạn cần để lưu trữ client và access token:

    php artisan migrate

> {note} Nếu bạn không sử dụng migration mặc định của Passport, bạn nên gọi phương thức `Passport::ignoreMigrations` trong phương thức `register` của `AppServiceProvider` của bạn. Bạn có thể export các migration mặc định bằng cách sử dụng `php artisan vendor:publish --tag=passport-migrations`.

Tiếp theo, bạn nên chạy lệnh `passport:install`. Lệnh này sẽ tạo các key mã hóa cần thiết để tạo secure access token. Ngoài ra, lệnh sẽ tạo "personal access" và "password grant" client sẽ được sử dụng để tạo access token:

    php artisan passport:install

Sau khi chạy lệnh này, hãy thêm trait `Laravel\Passport\HasApiTokens` vào model `App\User` của bạn. Trait này sẽ cung cấp một vài phương thức helper cho model của bạn, cho phép bạn kiểm tra token và scope của người dùng đã được authenticate:

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

Tiếp theo, bạn nên gọi phương thức `Passport::routes` trong phương thức `boot` của `AuthServiceProvider` của bạn. Phương thức này sẽ đăng ký các route cần thiết để phát hành access token và thu hồi access token, client và access token cá nhân:

    <?php

    namespace App\Providers;

    use Laravel\Passport\Passport;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

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

Cuối cùng, trong file cấu hình `config/auth.php` của bạn, bạn nên set tùy chọn `driver` của `api` authentication guard thành `passport`. Điều này sẽ hướng dẫn application của bạn sử dụng `TokenGuard` của Passport khi authenticate các incoming request API:

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

<a name="frontend-quickstart"></a>
### Frontend Quickstart

> {note} Để sử dụng các component Passport Vue, bạn phải sử dụng JavaScript framework [Vue](https://vuejs.org). Các component này cũng sử dụngBootstrap CSS framework. Tuy nhiên, ngay cả khi bạn không sử dụng các công cụ này, các component đóng vai trò là tài liệu tham khảo có giá trị cho việc triển khai frontend của chính bạn.

Passport ship cùng một JSON API mà bạn có thể sử dụng để cho phép người dùng của bạn tạo client và access token cá nhân. Tuy nhiên, nó có thể mất thời gian để code một frontend để tương tác với các API này. Vì vậy, Passport cũng chứa các component [Vue](https://vuejs.org) được xây dựng sẵn mà bạn có thể sử dụng làm ví dụ triển khai hoặc điểm bắt đầu cho việc triển khai của riêng bạn.

Để publish component Passport Vue, hãy sử dụng lệnh Artisan `vendor:publish`:

    php artisan vendor:publish --tag=passport-components

Các thành phần được publish sẽ được đặt trong thư mục `resources/assets/js/components`của bạn. Khi các thành phần đã được publish, bạn nên đăng ký chúng vào trong file `resources/assets/js/app.js`:

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue')
    );

    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue')
    );

    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue')
    );

Sau khi đăng ký các component, hãy chạy `npm run dev` để biên dịch lại assets của bạn. Khi bạn đã biên dịch lại assets của bạn, bạn có thể drop các component vào một trong các template của application để bắt đầu tạo client và access token cá nhân:

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="deploying-passport"></a>
### Deploying Passport

Khi deploy Passport đến server production của bạn cho lần đầu tiên, bạn có thể sẽ cần chạy lệnh `passport:keys`. Lệnh này tạo các key mã hóa Passport cần để tạo access token. Các key được tạo thường không nên lưu trong source control:

    php artisan passport:keys

<a name="configuration"></a>
## Cấu hình

<a name="token-lifetimes"></a>
### Thời gian sống token

Mặc định, Passport phát hành các access token tồn tại lâu dài mà không bao giờ cần phải refresh. Nếu bạn muốn cấu hình vòng đời token ngắn hơn, bạn có thể sử dụng các phương thức `tokensExpireIn` và `refreshTokensExpireIn`. Các phương thức này phải được gọi từ phương thức `boot` của `AuthServiceProvider`:

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
    }

<a name="issuing-access-tokens"></a>
## Phát hành token truy cập

Sử dụng OAuth2 với authorization code là cách mà hầu hết các developer quen thuộc với OAuth2. Khi sử dụng authorization code, một client application sẽ chuyển hướng một người dùng đến server của bạn, nơi họ sẽ chấp nhận hoặc từ chối request cấp access token cho client.

<a name="managing-clients"></a>
### Quản lý client

Đầu tiên, các developer cần xây dựng các application cần tương tác với API của application của bạn, họ sẽ cần phải đăng ký application của họ với application của bạn bằng cách tạo "client". Thông thường, việc này gồm việc cung cấp tên application của họ và URL mà application của bạn có thể chuyển hướng đến sau khi người dùng chấp thuận request ủy quyền của họ.

#### Lệnh `passport:client`

Cách đơn giản nhất để tạo một client là sử dụng lệnh Artisan `passport:client`. Lệnh này có thể được sử dụng để tạo các client của riêng bạn để test chức năng OAuth2 của bạn. Khi bạn chạy lệnh `client`, Passport sẽ nhắc bạn để biết thêm thông tin về client của bạn và sẽ cung cấp cho bạn với client ID và secret:

    php artisan passport:client

#### JSON API

Vì người dùng của bạn sẽ không thể sử dụng lệnh `client`, Passport cung cấp một JSON API mà bạn có thể sử dụng để tạo client. Điều này giúp bạn tránh những rắc rối khi phải tự viết controller để tạo, cập nhật và xóa client.

Tuy nhiên, bạn sẽ cần pair JSON API của Passport với frontend của riêng bạn để cung cấp bảng điều khiển cho người dùng của bạn để quản lý client của họ. Dưới đây, chúng ta sẽ review tất cả các API endpoint để quản lý client. Để thuận tiện, chúng ta sẽ sử dụng [Axios](https://github.com/mzabriskie/axios) để thể hiện việc thực hiện các HTTP request đến các endpoint.

> {tip} Nếu bạn không muốn tự mình thực hiện toàn bộ quản lý client, bạn có thể sử dụng [frontend quickstart](#frontend-quickstart) để có một frontend đầy đủ chức năng trong vài phút.

#### `GET /oauth/clients`

Route này trả về tất cả các client cho người dùng đã được authenticate. Điều này chủ yếu hữu ích để liệt kê tất cả các client của người dùng để họ có thể chỉnh sửa hoặc xóa chúng:

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

Route này được sử dụng để tạo client mới. Nó đòi hỏi hai phần dữ liệu: `name` của client và một URL `redirect`. URL `redirect` là nơi người dùng sẽ được chuyển hướng đến sau khi chấp nhận hoặc từ chối một request cho authorization.

Khi một client đã được tạo, nó sẽ được cấp một client ID và client secret. Các giá trị này sẽ được sử dụng khi yêu cầu access token từ application của bạn. Route tạo client sẽ trả về instance client mới:

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

Route này được sử dụng để cập nhật client. Nó đòi hỏi hai phần dữ liệu:`name` của client và một URL `redirect`. URL `redirect` là nơi người dùng sẽ được chuyển hướng đến sau khi chấp nhận hoặc từ chối một request cho authorization. Route sẽ trả về instance client đã được cập nhật:

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

Khi một client đã được tạo, các developer có thể sử dụng client ID và secret của họ để yêu cầu authorization code và access token từ application của bạn. Đầu tiên, application của bên thứ ba sẽ tạo một yêu cầu chuyển hướng đến route `/oauth/authorize` của application của bạn như sau:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Hãy nhớ rằng, route `/oauth/authorize` đã được định nghĩa bởi phương thức `Passport::routes`. Bạn không cần phải tự định nghĩa route này.

#### Approving The Request

Khi nhận được authorization request, Passport sẽ tự động hiển thị một template cho người dùng cho phép họ phê duyệt hoặc từ chối authorization request. Nếu họ chấp thuận request, họ sẽ được chuyển hướng trở lại `redirect_uri` sẽ được chỉ định bởi application của bên thứ ba. `redirect_uri` phải khớp với URL `redirect` được chỉ định khi client được tạo.

Nếu bạn muốn tùy chỉnh màn hình phê duyệt authorization, bạn có thể publish các view của Passport bằng cách sử dụng lệnh Artisan `vendor:publish`. Các view được publish sẽ được đặt trong `resources/views/vendor/passport`:

    php artisan vendor:publish --tag=passport-views

#### Converting Authorization Codes To Access Tokens

Nếu người dùng chấp thuận authorization request, họ sẽ được chuyển hướng trở lại application của bên thứ ba. Sau đó, bên thứ ba sẽ đưa ra request `POST` cho application của bạn để yêu cầu access token. Yêu cầu phải chứa authorization code được cấp bởi application của bạn khi người dùng chấp thuận authorization request. Trong ví dụ này, chúng ta sẽ sử dụng thư viện Guzzle HTTP để thực hiện request `POST`:

    Route::get('/callback', function (Request $request) {
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

Route `/oauth/token` này sẽ trả về một JSON response có chứa các thuộc tính `access_token`, `refresh_token` và `expires_in`. Thuộc tính `expires_in` chứa số giây cho đến khi access token hết hạn.

> {tip} Giống như route `/oauth/authorize`, route `/oauth/token` được định nghĩa cho bạn bằng phương thức`Passport::routes`. Không cần phải tự định nghĩa cho route này.

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

Route `/oauth/token` này sẽ trả về một JSON response có chứa các thuộc tính `access_token`, `refresh_token` và `expires_in`. Thuộc tính `expires_in` chứa số giây cho đến khi access token hết hạn.

<a name="password-grant-tokens"></a>
## Token password grant

OAuth2 password grant cho phép các client bên thứ nhất khác của bạn, chẳng hạn như một mobile application, có được access token bằng địa chỉ email hoặc tên người dùng và mật khẩu. Điều này cho phép bạn phát hành access token một cách an toàn cho các client bên thứ nhất mà không yêu cầu người dùng của bạn thực hiện toàn bộ luồng chuyển hướng OAuth2 authorization code.

<a name="creating-a-password-grant-client"></a>
### Tạo một password grant client

Trước khi application của bạn có thể phát hành token thông qua password grant, bạn sẽ cần tạo một client password grant. Bạn có thể làm điều này bằng cách sử dụng lệnh `passport:client` với tùy chọn `--password`. Nếu bạn đã chạy lệnh `passport:install`, bạn không cần chạy lệnh này:

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### Request token

Khi bạn đã tạo một client password grant, bạn có thể yêu cầu access token bằng cách đưa ra request `POST` cho route `/oauth/token` với địa chỉ email và mật khẩu của người dùng. Hãy nhớ rằng, route này đã được đăng ký bằng phương thức `Passport::routes` nên không cần tự định nghĩa. Nếu yêu cầu thành công, bạn sẽ nhận được một `access_token` và `refresh_token` trong JSON response từ server:

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

> {tip} Hãy nhớ rằng, access token sẽ mặc định được tồn tại mãi mãi. Tuy nhiên, bạn có thể thoải mái [cấu hình maximum vòng đời access token của bạn](#configuration) nếu cần.

<a name="requesting-all-scopes"></a>
### Requesting All Scopes

Khi sử dụng password grant, bạn có thể muốn ủy quyền token cho tất cả các scope được application của bạn hỗ trợ. Bạn có thể làm điều này bằng cách yêu cầu scope `*`. Nếu bạn yêu cầu scope `*`, phương thức `can` trên instance token sẽ luôn trả về `true`. Scope này chỉ có thể được gán cho token được cấp bằng cách sử dụng `password` grant:

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

<a name="implicit-grant-tokens"></a>
## Token với grant ẩn

Grant ẩn tương tự như authorization code grant; tuy nhiên, token được trả về cho client mà không trao đổi authorization code. Grant này được sử dụng phổ biến nhất cho các application JavaScript hoặc di động nơi thông tin đăng nhập của client không thể được lưu trữ an toàn. Để kích hoạt grant, hãy gọi phương thức `enableImplicitGrant` trong `AuthServiceProvider` của bạn:

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

Khi một grant đã được bật, developer có thể sử dụng client ID của họ để yêu cầu access token từ application của bạn. Ứng dụng của developer sẽ tạo một yêu cầu chuyển hướng đến route `/oauth/authorize` của application của bạn như sau:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Hãy nhớ rằng, route `/oauth/authorize` đã được định nghĩa bởi phương thức `Passport::routes`. Bạn không cần phải tự định nghĩa route này.

<a name="client-credentials-grant-tokens"></a>
## Token thông tin client grant

thông tin client grant phù hợp cho authentication machine-to-machine. Ví dụ: bạn có thể sử dụng grant này trong một scheduled job đang thực hiện công việc bảo trì qua API. Để sử dụng phương thức này, trước tiên bạn cần thêm middleware mới vào `$routeMiddleware` trong `app/Http/Kernel.php`:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

Sau đó gắn middleware này vào một route:

    Route::get('/user', function(Request $request) {
        ...
    })->middleware('client');

Để lấy một token, hãy tạo một request đến `oauth/token` endpoint:

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

Đôi khi, người dùng của bạn có thể muốn phát hành access token cho chính họ mà không cần thông qua luồng chuyển hướng authorization code thông thường. Cho phép người dùng phát hành token cho chính họ thông qua giao diện người dùng của application của bạn có thể hữu ích cho phép người dùng thử nghiệm API của bạn hoặc có thể phục vụ như một cách tiếp cận đơn giản hơn để phát hành access token nói chung.

> {note} Access token cá nhân luôn tồn tại lâu dài. Vòng đời của chúng không bị sửa  khi sử dụng các phương thức `tokensExpireIn` hoặc` refreshTokensExpireIn`.

<a name="creating-a-personal-access-client"></a>
### Creating A Personal Access Client

Trước khi application của bạn có thể phát hành personal access token, bạn sẽ cần tạo personal access client. Bạn có thể làm điều này bằng cách sử dụng lệnh `passport:client` với tùy chọn `--personal`. Nếu bạn đã chạy lệnh `passport:install`, bạn không cần chạy lệnh này:

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### Managing Personal Access Tokens

Khi bạn đã tạo một personal access client, bạn có thể phát hành token cho một người dùng nhất định bằng cách sử dụng phương thức `createToken` trên instance model `User`. Phương thức `createToken` chấp nhận tên của token làm tham số đầu tiên và một mảng tùy chọn [scopes](#token-scopes) làm tham số thứ hai:

    $user = App\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport cũng chứa một JSON API để quản lý personal access token. Bạn có thể kết hợp điều này với frontend của riêng bạn để cung cấp cho người dùng bảng điều khiển để quản lý personal access token. Dưới đây, chúng ta sẽ review tất cả các API endpoint để quản lý personal access token. Để thuận tiện, chúng ta sẽ sử dụng [Axios](https://github.com/mzabriskie/axios) để thể hiện việc thực hiện các HTTP request đến các endpoint.

> {tip} Nếu bạn không muốn tự triển khai frontend cho personal access token của bạn, bạn có thể sử dụng [frontend quickstart](#frontend-quickstart) để có một frontend đầy đủ chức năng trong vài phút.

#### `GET /oauth/scopes`

Route này trả về tất cả [scopes](#token-scopes) được định nghĩa cho application của bạn. Bạn có thể sử dụng route này để liệt kê scope mà người dùng có thể chỉ định cho một personal access token:

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

Route này trả về tất cả các personal access token mà người dùng đã authenticate, đã tạo. Điều này chủ yếu hữu ích để liệt kê tất cả các token của người dùng để họ có thể chỉnh sửa hoặc xóa chúng:

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

Route này tạo personal access token mới. Nó đòi hỏi hai phần dữ liệu: `name` của token và `scopes` cần được gán cho token:

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

Passport có chứa [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) sẽ validate access token theo incoming request. Khi bạn đã cấu hình `api` guard để sử dụng `passport` driver, bạn chỉ cần chỉ định `auth:api` middleware trên bất kỳ route nào require một access token hợp lệ:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### Pass access token

Khi gọi các route được bảo vệ bởi Passport, API bên thứ ba của application của bạn nên chỉ định access token của họ dưới dạng một `Bearer` token trong header `Authorization` trong request của họ. Ví dụ: khi sử dụng thư viện Guzzle HTTP:

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## Token scope

<a name="defining-scopes"></a>
### Định nghĩa scope

Scope cho phép API client của bạn yêu cầu một nhóm quyền cụ thể khi request authorization để truy cập vào tài khoản. Ví dụ: nếu bạn đang xây dựng một application thương mại điện tử, không phải tất cả API bên thứ ba sẽ cần khả năng đặt hàng. Thay vào đó, bạn có thể cho phép bên thứ ba chỉ request authorization truy cập trạng thái giao hàng. Nói cách khác, scope cho phép người dùng application của bạn giới hạn các hành động mà application của bên thứ ba có thể thực hiện thay mặt họ.

Bạn có thể định nghĩa scope của API của mình bằng phương thức `Passport::tokensCan` trong phương thức `boot` của `AuthServiceProvider`. Phương thức `tokensCan` chấp nhận một loạt các tên scope và mô tả scope. Mô tả scope có thể là bất cứ điều gì bạn muốn và sẽ được hiển thị cho người dùng trên màn hình phê duyệt authorization:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### Gán scope đến token

#### When Requesting Authorization Codes

Khi yêu cầuaccess token bằng cách sử dụng authorization code grant, bên thứ ba nên chỉ định scope mong muốn của họ là tham số chuỗi truy vấn `scope`. Tham số `scope` phải là một danh sách scope đã được phân tách bằng dấu cách:

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

Nếu bạn đang phát hành personal access token bằng cách sử dụng phương thức `createToken` củamodel `User`, bạn có thể pass một mảng scop mong muốn làm tham số thứ hai cho phương thức:
If you are issuing personal access tokens using the `User` model's `createToken` method, you may pass the array of desired scopes as the second argument to the method:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Kiểm tra scope

Passport có chứa hai middleware có thể được sử dụng để xác minh rằng incoming request đã được authenticate với một tokeno đã được grant một scope nhất định. Để bắt đầu, hãy thêm middleware sau vào thuộc tính `$routeMiddleware` của file `app/Http/Kernel.php` của bạn:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### Check For All Scopes

Middleware `scopes` có thể được chỉ định cho một route để xác minh rằng access token của incoming request có *tất cả* trong scope được liệt kê:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware('scopes:check-status,place-orders');

#### Check For Any Scopes

Middleware `scope` có thể được chỉ định cho một routeđể xác minh rằng access token của incoming request có *ít nhất một* trong scope được liệt kê:

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware('scope:check-status,place-orders');

#### Kiểm tra scope On A Token Instance

Khi một request được authenticate bằng access token đã vào application của bạn, bạn vẫn có thể kiểm tra xem token có scope nhất định hay không bằng cách sử dụng phương thức `tokenCan` trên instance `User` đã được xác thực:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## Sử dụng API của bạn với JavaScript

Khi xây dựng một API, nó có thể cực kỳ hữu ích để có thể sử dụng API của riêng bạn từ application JavaScript của bạn. Cách tiếp cận này để phát triển API cho phép application của bạn sử dụng cùng API mà bạn đang chia sẻ với mọi người. API tương tự có thể được sử dụng bởi application web, application di động, application của bên thứ ba và bất kỳ SDK nào bạn có thể publish trên các trình quản lý package khác nhau.

Thông thường, nếu bạn muốn sử dụng API từ application JavaScript của mình, bạn cần  tự gửi access token đến application và chuyển nó với mỗi request đến application của bạn. Tuy nhiên, Passport có chứa một middleware có thể xử lý việc này cho bạn. Tất cả những gì bạn cần làm là thêm middleware `CreateFreshApiToken` vào middleware group `web` của bạn:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

Passport middleware này sẽ gán một cookie `laravel_token` vào các response gửi đi của bạn. Cookie này chứa JWT đã được mã hóa mà Passport sẽ sử dụng để xác thực các API request từ application JavaScript của bạn. Bây giờ, bạn có thể thực hiện các yêu cầu đối với API của application mà không cần pass một access token:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

Khi sử dụng phương thức xác thực này, mặc định Laravel JavaScript scaffolding sẽ hướng dẫn Axios luôn gửi các header `X-CSRF-TOKEN` và `X-Requested-With`. Tuy nhiên, bạn nên chắc chắn đã thêm CSRF token của mình vào một [HTML meta tag](/docs/{{version}}/csrf#csrf-x-csrf-token):

    window.axios.defaults.headers.common = {
        'X-Requested-With': 'XMLHttpRequest',
    };

> {note} Nếu bạn đang sử dụng một JavaScript framework khác, bạn nên đảm bảo rằng nó được cấu hình để gửi các header `X-CSRF-TOKEN` và `X-Requested-With` với mọi request gửi đi.

<a name="events"></a>
## Event

Passport tạo event khi phát hành access token và refresh token. Bạn có thể sử dụng các event này để bỏ bớt hoặc thu hồi các access token khác trong cơ sở dữ liệu của bạn. Bạn có thể gán listener vào các event này trong `EventServiceProvider`:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Laravel\Passport\Event\AccessTokenCreated' => [
        'App\Listeners\RevokeOldTokens',
    ],

    'Laravel\Passport\Event\RefreshTokenCreated' => [
        'App\Listeners\PruneOldTokens',
    ],
];
```

<a name="testing"></a>
## Test

Phương thức `actingAs` của Passport có thể được sử dụng để chỉ định người dùng đã được xác thực hiện tại cũng như scope của nó. Tham số đầu tiên được đưa vào cho phương thức `actingAs` là instance user và tham số thứ hai là một mảng scope nên được grant cho token của người dùng:

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(200);
    }
