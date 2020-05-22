# Laravel Socialite

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
- [Routing](#routing)
- [Các tham số tuỳ chọn](#optional-parameters)
- [Truy cập đến Scope](#access-scopes)
- [Không lưu thông tin Authentication](#stateless-authentication)
- [Lấy ra thông tin User](#retrieving-user-details)

<a name="introduction"></a>
## Giới thiệu

Ngoài authentication thông thường dựa trên form, Laravel cũng cung cấp một cách đơn giản, thuận tiện để authentication với các nhà provider OAuth khác bằng cách sử dụng [Laravel Socialite](https://github.com/laravel/socialite). Socialite hiện hỗ trợ authentication với Facebook, Twitter, LinkedIn, Google, GitHub và Bitbucket.

> {tip} Adapter cho các nền tảng này được liệt kê tại trang web [Socialite Providers](https://socialiteproviders.netlify.com/) do cộng đồng phát triển.

<a name="installation"></a>
## Cài đặt

Để bắt đầu với Socialite, hãy sử dụng Composerđể thêm package vào các library của project của bạn:

    composer require laravel/socialite

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng Socialite, bạn cũng sẽ cần thêm thông tin cho các dịch vụ OAuth mà application của bạn sử dụng. Các thông tin này phải được set trong file cấu hình `config/services.php` của bạn và nên sử dụng key `facebook`, `twitter`, `linkedin`, `google`, `github` hoặc `bitbucket`, tùy thuộc vào provider application của bạn yêu cầu. Ví dụ:

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),         // Your GitHub Client ID
        'client_secret' => env('GITHUB_CLIENT_SECRET'), // Your GitHub Client Secret
        'redirect' => 'http://your-callback-url',
    ],

> {tip} Nếu tùy chọn `redirect` chứa một relative path, nó sẽ tự động được resolve thành một absolute path.

<a name="routing"></a>
## Routing

Tiếp theo, bạn đã sẵn sàng để authenticate người dùng! Bạn sẽ cần hai route: một là để chuyển hướng người dùng đến provider OAuth và một route khác để nhận callback từ provider sau khi authenticate. Chúng ta sẽ truy cập vào Socialite bằng cách sử dụng facade `Socialite`:

    <?php

    namespace App\Http\Controllers\Auth;

    use Socialite;

    class LoginController extends Controller
    {
        /**
         * Redirect the user to the GitHub authentication page.
         *
         * @return \Illuminate\Http\Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * Obtain the user information from GitHub.
         *
         * @return \Illuminate\Http\Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

Phương thức `redirect` đảm nhiệm việc gửi người dùng đến provider OAuth, trong khi phương thức `user` sẽ đọc incoming request và lấy thông tin của người dùng từ provider.

Tất nhiên là, bạn sẽ cần phải định nghĩa các route đến các phương thức của controller của bạn:

    Route::get('login/github', 'Auth\LoginController@redirectToProvider');
    Route::get('login/github/callback', 'Auth\LoginController@handleProviderCallback');

<a name="optional-parameters"></a>
## Các tham số tuỳ chọn

Một số các provider OAuth hỗ trợ các tham số tùy chọn trong request chuyển hướng. Để thêm bất kỳ tham số tùy chọn nào trong request, hãy gọi phương thức `with` với một mảng:

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> {note} Khi sử dụng phương thức `with`, hãy cẩn thận đừng pass qua bất kỳ keyword bí mật nào như `state` hoặc `answer_type`.

<a name="access-scopes"></a>
## Truy cập đến Scope

Trước khi chuyển hướng người dùng, bạn cũng có thể thêm "scopes" vào request bằng phương thức `scopes`. Phương pháp này sẽ merge tất cả các scope hiện có với scope mà bạn cung cấp:

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

Bạn có thể ghi đè tất cả các scope hiện có bằng phương thức `setScopes`:

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="stateless-authentication"></a>
## Không lưu thông tin Authentication

Phương thức `stateless` có thể được sử dụng để vô hiệu hóa xác minh trạng thái của session. Điều này hữu ích khi thêm các social authentication vào API:

    return Socialite::driver('google')->stateless()->user();

<a name="retrieving-user-details"></a>
## Lấy ra thông tin User

Khi bạn đã có một instance người dùng, bạn có thể lấy một số thông tin về người dùng:

    $user = Socialite::driver('github')->user();

    // OAuth Two Providers
    $token = $user->token;
    $refreshToken = $user->refreshToken; // not always provided
    $expiresIn = $user->expiresIn;

    // OAuth One Providers
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All Providers
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

#### Retrieving User Details From A Token (OAuth2)

Nếu bạn đã có access token hợp lệ của một người dùng, bạn có thể lấy ra thông tin chi tiết của họ bằng phương thức `userFromToken`:

    $user = Socialite::driver('github')->userFromToken($token);

#### Retrieving User Details From A Token And Secret (OAuth1)

Nếu bạn đã có một token / secret hợp lệ của người dùng, bạn có thể truy xuất thông tin chi tiết của họ bằng phương thức `userFromTokenAndSecret`:

    $user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);
