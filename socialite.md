# Laravel Socialite

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Cập nhật Socialite](#upgrading-socialite)
- [Cấu hình](#configuration)
- [Authentication](#authentication)
    - [Routing](#routing)
    - [Xác thực và lưu trữ](#authentication-and-storage)
    - [Truy cập đến Scope](#access-scopes)
    - [Tham số tuỳ chọn](#optional-parameters)
- [Lấy ra thông tin User](#retrieving-user-details)

<a name="introduction"></a>
## Giới thiệu

Ngoài những cách authentication thông thường dựa trên form, Laravel cũng cung cấp thêm một số cách đơn giản, thuận tiện để authentication với các provider OAuth khác bằng cách sử dụng [Laravel Socialite](https://github.com/laravel/socialite). Socialite hiện hỗ trợ authentication với Facebook, Twitter, LinkedIn, Google, GitHub, GitLab, và Bitbucket.

> {tip} Bộ chuyển đổi cho các nền tảng này được liệt kê trong trang web [Socialite Providers](https://socialiteproviders.com/) do cộng đồng phát triển.

<a name="installation"></a>
## Cài đặt

Để bắt đầu với Socialite, hãy sử dụng Composer package manager để thêm package của nó vào library project của bạn:

    composer require laravel/socialite

<a name="upgrading-socialite"></a>
## Cập nhật Socialite

Khi nâng cấp lên phiên bản mới của Socialite, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/socialite/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng Socialite, bạn sẽ cần phải thêm thông tin các OAuth provider mà application của bạn đang muốn sử dụng. Các thông tin này phải được set trong file cấu hình `config/services.php` của application của bạn và sử dụng các key `facebook`, `twitter`, `linkedin`, `google`, `github`, `gitlab`, hoặc `bitbucket`, tùy thuộc vào provider application của bạn yêu cầu. Ví dụ:

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => 'http://example.com/callback-url',
    ],

> {tip} Nếu tùy chọn `redirect` chứa một relative path, nó sẽ tự động được resolve thành một absolute path.

<a name="authentication"></a>
## Authentication

<a name="routing"></a>
### Routing

Để authenticate người dùng bằng OAuth provider! bạn sẽ cần hai route: một là để chuyển hướng người dùng đến provider OAuth và một route khác để nhận các callback từ provider sau khi authenticate thành công. Controller mẫu ở bên dưới sẽ minh họa việc triển khai cả hai route này:

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/redirect', function () {
        return Socialite::driver('github')->redirect();
    });

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // $user->token
    });

Phương thức `redirect` được cung cấp bởi facade `Socialite` sẽ đảm nhiệm việc chuyển hướng người dùng đến provider OAuth, trong khi phương thức `user` sẽ đọc request gửi về và lấy ra thông tin của người dùng từ provider sau khi họ được authenticate.

<a name="authentication-and-storage"></a>
### Xác thực và lưu trữ

Sau khi người dùng được lấy ra từ OAuth provider, bạn có thể xác định xem người dùng đó có tồn tại trong cơ sở dữ liệu ứng dụng của bạn hay không và [xác thực người dùng](/docs/{{version}}/authentication#authenticate-a-user-instance) đó. Nếu người dùng không tồn tại trong cơ sở dữ liệu ứng dụng của bạn, thông thường bạn sẽ tạo một record mới trong cơ sở dữ liệu của bạn để đại diện cho người dùng đó:

    use App\Models\User;
    use Illuminate\Support\Facades\Auth;
    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $githubUser = Socialite::driver('github')->user();

        $user = User::where('github_id', $githubUser->id)->first();

        if ($user) {
            $user->update([
                'github_token' => $githubUser->token,
                'github_refresh_token' => $githubUser->refreshToken,
            ]);
        } else {
            $user = User::create([
                'name' => $githubUser->name,
                'email' => $githubUser->email,
                'github_id' => $githubUser->id,
                'github_token' => $githubUser->token,
                'github_refresh_token' => $githubUser->refreshToken,
            ]);
        }

        Auth::login($user);

        return redirect('/dashboard');
    });

> {tip} Để biết thêm chi tiết về những thông tin người dùng mà có sẵn từ các OAuth provider, vui lòng tham khảo tài liệu về [lấy ra chi tiết người dùng](#retrieving-user-details).

<a name="access-scopes"></a>
### Truy cập đến Scope

Trước khi chuyển hướng người dùng, bạn cũng có thể thêm các "scopes" vào các authentication request bằng phương thức `scopes`. Phương thức này sẽ merge tất cả các scope hiện tại với scope mà bạn đã cung cấp:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

Bạn có thể ghi đè tất cả các scope đã có trong authentication request bằng phương thức `setScopes`:

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="optional-parameters"></a>
### Tham số tuỳ chọn

Một số OAuth provider hỗ trợ các tham số tùy chọn trong request chuyển hướng. Để thêm bất kỳ tham số tùy chọn nào vào trong request, hãy gọi phương thức `with` với một mảng:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> {note} Khi sử dụng phương thức `with`, bạn nên cẩn thận để không truyền bất kỳ từ khóa nào đã được dùng như `state` hoặc `response_type`.

<a name="retrieving-user-details"></a>
## Lấy ra thông tin User

Sau khi người dùng được chuyển hướng trở lại route callback xác thực của bạn, bạn có thể lấy ra thông tin chi tiết của người dùng bằng phương thức `user` của Socialite. Đối tượng người dùng được trả về bởi phương thức `user` cung cấp nhiều thuộc tính và phương thức khác nhau mà bạn có thể sử dụng để lưu thông tin về người dùng vào trong cơ sở dữ liệu của bạn. Các thuộc tính và các phương thức có sẵn vẫn còn tùy thuộc vào việc OAuth provider mà bạn đang xác thực có hỗ trợ OAuth 1.0 hay OAuth 2.0 hay không:

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // OAuth 2.0 providers...
        $token = $user->token;
        $refreshToken = $user->refreshToken;
        $expiresIn = $user->expiresIn;

        // OAuth 1.0 providers...
        $token = $user->token;
        $tokenSecret = $user->tokenSecret;

        // All providers...
        $user->getId();
        $user->getNickname();
        $user->getName();
        $user->getEmail();
        $user->getAvatar();
    });

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### Retrieving User Details From A Token (OAuth2)

Nếu bạn đã có một access token hợp lệ của một người dùng, bạn có thể lấy ra thông tin chi tiết của họ bằng phương thức `userFromToken` của Socialite:

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('github')->userFromToken($token);

<a name="retrieving-user-details-from-a-token-and-secret-oauth1"></a>
#### Retrieving User Details From A Token And Secret (OAuth1)

Nếu bạn đã có một token và secret hợp lệ của người dùng, bạn có thể truy xuất thông tin chi tiết của người dùng đó bằng phương thức `userFromTokenAndSecret` của Socialite:

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);

<a name="stateless-authentication"></a>
#### Stateless Authentication

Phương thức `stateless` có thể được sử dụng để vô hiệu hóa việc xác minh trạng thái của session. Điều này hữu ích khi thêm xác thực social vào API:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')->stateless()->user();

> {note} Xác thực không trạng thái sẽ không khả dụng cho driver Twitter sử dụng OAuth 1.0 để xác thực.
