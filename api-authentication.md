# API Authentication

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Database Preparation](#database-preparation)
- [Tạo token](#generating-tokens)
    - [Hashing token](#hashing-tokens)
- [Bảo vệ route](#protecting-routes)
- [Truyền token trong request](#passing-tokens-in-requests)

<a name="introduction"></a>
## Giới thiệu

Mặc định, Laravel cung cấp một giải pháp đơn giản để xác thực API thông qua một random token được chỉ định cho mỗi người dùng vào ứng dụng của bạn. Trong file cấu hình `config/auth.php` của bạn, một guard `api` đã được định nghĩa mặc định và sử dụng một driver `token`. Driver này chịu trách nhiệm kiểm tra token API trong request đến khớp với token đã được chỉ định cho người dùng trong cơ sở dữ liệu.

> **Note:** Mặc dù Laravel cung cấp một guard xác thực dựa trên token đơn giản, nhưng chúng tôi thực sự khuyên bạn rằng nên cân nhắc sử dụng [Laravel Passport](/docs/{{version}}/passport) cho các ứng dụng production, mạnh mẽ cung cấp xác thực API .

<a name="configuration"></a>
## Cấu hình

<a name="database-preparation"></a>
### Database Preparation

Trước khi sử dụng driver `token`, bạn cần phải [tạo một migration](/docs/{{version}}/migrations) để thêm cột `api_token` vào bảng `users` của bạn:

    Schema::table('users', function ($table) {
        $table->string('api_token', 80)->after('password')
                            ->unique()
                            ->nullable()
                            ->default(null);
    });

Khi migration đã được tạo, hãy chạy lệnh Artisan `migrate`.

> {tip} Nếu bạn chọn sử dụng một tên cột khác, thì hãy nhớ cập nhật lại tùy chọn cấu hình `storage_key` của API trong file cấu hình `config/auth.php`.

<a name="generating-tokens"></a>
## Tạo token

Sau khi cột `api_token` đã được thêm vào bảng `users` của bạn, bạn đã sẵn sàng gắn token API random cho mỗi người dùng đăng ký vào ứng dụng của bạn. Bạn nên chỉ định các token này khi model `User` được tạo trong quá trình đăng ký. Khi bạn sử dụng [authentication scaffolding](/docs/{{version}}/authentication#authentication-quickstart) được cung cấp bởi package `laravel/ui` thông qua Composer, bạn có thể thực hiện được điều này trong phương thức `create` của `RegisterController`:

    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    /**
     * Create a new user instance after a valid registration.
     *
     * @param  array  $data
     * @return \App\User
     */
    protected function create(array $data)
    {
        return User::forceCreate([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
            'api_token' => Str::random(80),
        ]);
    }

<a name="hashing-tokens"></a>
### Hashing token

Trong các ví dụ trên, token API sẽ được lưu vào trong cơ sở dữ liệu của bạn dưới dạng một văn bản thuần. Nếu bạn muốn hash token API của bạn bằng cách sử dụng hàm hash SHA-256, bạn có thể set tùy chọn `hash` trong cấu hình guard `api` của bạn thành `true`. Guard `api` được định nghĩa trong file cấu hình `config/auth.php` của bạn:

    'api' => [
        'driver' => 'token',
        'provider' => 'users',
        'hash' => true,
    ],

#### Generating Hashed Tokens

Khi sử dụng token API được hash, bạn không nên tạo token API của bạn trong quá trình đăng ký người dùng. Thay vào đó, bạn nên làm một trang quản lý token API  riêng vào trong ứng dụng của bạn. Trang này sẽ cho phép người dùng khởi tạo và làm mới token API của họ. Khi người dùng đưa ra các yêu cầu khởi tạo hoặc làm mới token của họ, bạn nên lưu trữ lại bản sao đã hash của token vào trong cơ sở dữ liệu và trả về bản thuần tuý của token cho giao diện người dùng để hiển thị một lần duy nhất.

Ví dụ: một phương thức controller khởi tạo và làm mới token cho một người dùng và trả về token thuần túy dưới dạng một response JSON có thể trông giống như sau:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    class ApiTokenController extends Controller
    {
        /**
         * Update the authenticated user's API token.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function update(Request $request)
        {
            $token = Str::random(80);

            $request->user()->forceFill([
                'api_token' => hash('sha256', $token),
            ])->save();

            return ['token' => $token];
        }
    }

> {tip} Vì các token API trong ví dụ trên đã có entropy, nên việc tạo "rainbow tables" để tra cứu giá trị ban đầu của token đã hash là không thể. Do đó, phương thức hash chậm như `bcrypt` là không cần thiết.

<a name="protecting-routes"></a>
## Bảo vệ route

Laravel có chứa một [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) sẽ tự động xác thực token API khi có request đến. Bạn chỉ cần chỉ định middleware `auth:api` vào trong bất kỳ route nào yêu cầu một token để truy cập:

    use Illuminate\Http\Request;

    Route::middleware('auth:api')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="passing-tokens-in-requests"></a>
## Truyền token trong request

Có một số cách để truyền token API đến ứng dụng của bạn. Chúng ta sẽ thảo luận về những cách này trong khi sử dụng thư viện Guzzle HTTP để demo cách sử dụng của chúng. Bạn có thể chọn bất kỳ cách nào trong số này dựa theo nhu cầu của ứng dụng của bạn.

#### Query String

Người dùng API của ứng dụng của bạn có thể truyền token của họ dưới dạng giá trị của một chuỗi truy vấn `api_token`:

    $response = $client->request('GET', '/api/user?api_token='.$token);

#### Request Payload

Người dùng API của ứng dụng của bạn có thể chứa token API của họ trong các tham số form của request dưới dạng một thuộc tính `api_token`:

    $response = $client->request('POST', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
        ],
        'form_params' => [
            'api_token' => $token,
        ],
    ]);

#### Bearer Token

Người dùng API của ứng dụng của bạn có thể cung cấp token API của họ dưới dạng token `Bearer` trong header `Authorization` của request:

    $response = $client->request('POST', '/api/user', [
        'headers' => [
            'Authorization' => 'Bearer '.$token,
            'Accept' => 'application/json',
        ],
    ]);
