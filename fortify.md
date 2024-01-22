# Laravel Fortify

- [Giới thiệu](#introduction)
    - [Fortify là gì?](#what-is-fortify)
    - [Khi nào dùng Fortify?](#when-should-i-use-fortify)
- [Cài đặt](#installation)
    - [Fortify Service Provider](#the-fortify-service-provider)
    - [Fortify Features](#fortify-features)
    - [Disabling Views](#disabling-views)
- [Authentication](#authentication)
    - [Tuỳ biến User Authentication](#customizing-user-authentication)
    - [Tuỳ biến Authentication Pipeline](#customizing-the-authentication-pipeline)
    - [Tuỳ biến Redirect](#customizing-authentication-redirects)
- [Two Factor Authentication](#two-factor-authentication)
    - [Bật Two Factor Authentication](#enabling-two-factor-authentication)
    - [Authenticating cùng với Two Factor Authentication](#authenticating-with-two-factor-authentication)
    - [Tắt Two Factor Authentication](#disabling-two-factor-authentication)
- [Đăng ký](#registration)
    - [Tuỳ biến đăng ký](#customizing-registration)
- [Password Reset](#password-reset)
    - [Yêu cầu link password reset](#requesting-a-password-reset-link)
    - [Reset password](#resetting-the-password)
    - [Tuỳ biến password reset](#customizing-password-resets)
- [Email Verification](#email-verification)
    - [Bảo vệ route](#protecting-routes)
- [Xác nhận password](#password-confirmation)

<a name="introduction"></a>
## Giới thiệu

[Laravel Fortify](https://github.com/laravel/fortify) là một package chứa backend cho authentication của Laravel mà không chứa fontend. Fortify sẽ đăng ký các route và controller cần thiết để triển khai tất cả các tính năng xác thực của Laravel, bao gồm đăng nhập, đăng ký, set lại mật khẩu, xác minh email, vv... Sau khi cài đặt Fortify, bạn có thể chạy lệnh Artisan `route:list` để xem các route mà Fortify đã đăng ký.

Vì Fortify không cung cấp fontend, nên bạn cần nối giao diện người dùng của bạn với các route mà Fortify đã đăng ký. Chúng ta sẽ thảo luận chính xác cách thực hiện các request cho các route này trong phần còn lại của tài liệu này.

> {tip} Hãy nhớ rằng, Fortify là một package giúp bạn triển khai các chức năng xác thực của Laravel. **Bạn không bắt buộc phải sử dụng nó.** Bạn có thể tương tác với các service xác thực của Laravel bằng cách làm theo tài liệu có sẵn trong [authentication](/docs/{{version}}/authentication), [reset password](/docs/{{version}}/passwords), và [xác minh email](/docs/{{version}}/verification).

<a name="what-is-fortify"></a>
### Fortify là gì?

Như đã đề cập trước đây, Laravel Fortify là một triển khai backend xác thực cho Laravel mà không chứa giao diện người dùng. Fortify đăng ký các route và controller cần thiết để triển khai tất cả các chức năng xác thực cho Laravel, bao gồm đăng nhập, đăng ký, set lại mật khẩu, xác minh email, vv...

**Bạn không bắt buộc phải sử dụng Fortify để sử dụng các chức năng xác thực của Laravel.** Bạn luôn được tự do tương tác với các service xác thực của Laravel bằng cách làm theo tài liệu có sẵn trong [authentication](/docs/{{version}}/authentication), [reset password](/docs/{{version}}/passwords), và [xác minh email](/docs/{{version}}/verification).

Nếu bạn chưa quen với Laravel, bạn có thể muốn xem qua starter kit [Laravel Breeze](/docs/{{version}}/starter-kits) trước khi thử sử dụng Laravel Fortify. Laravel Breeze sẽ cung cấp một scaffolding xác thực cho ứng dụng của bạn bao gồm cả giao diện người dùng được tạo bằng [Tailwind CSS](https://tailwindcss.com). Không giống như Fortify, Breeze export các route và controller trực tiếp vào ứng dụng của bạn. Điều này cho phép bạn có thể nghiên cứu và làm quen với các chức năng xác thực của Laravel trước khi cho phép Laravel Fortify triển khai các chức năng này cho bạn.

Về cơ bản, Laravel Fortify sử dụng các route và controller của Laravel Breeze và cung cấp chúng dưới dạng một package không chứa giao diện người dùng. Điều này cho phép bạn vẫn nhanh chóng xây dựng việc triển khai backend của phần xác thực ứng dụng của bạn mà không bị ràng buộc với bất kỳ quan điểm cụ thể nào về giao diện người dùng.

<a name="when-should-i-use-fortify"></a>
### Khi nào dùng Fortify?

Bạn có thể tự hỏi khi nào thì thích hợp để sử dụng Laravel Fortify. Đầu tiên, nếu bạn đang sử dụng [bộ starter kit application](/docs/{{version}}/starter-kits) của Laravel, thì bạn không cần phải cài đặt Laravel Fortify vì tất cả bộ starter kit ứng dụng của Laravel đã cung cấp đầy đủ các triển khai xác thực .

Nếu bạn không sử dụng bộ starter kit và ứng dụng của bạn cần các tính năng xác thực, thì bạn có hai tùy chọn: triển khai các tính năng xác thực của ứng dụng bằng tay hoặc sử dụng Laravel Fortify để cung cấp backend triển khai cho các tính năng này.

Nếu bạn chọn cài đặt Fortify, thì giao diện người dùng của bạn sẽ cần tạo ra các request cho với các route xác thực của Fortify, và sẽ được trình bày chi tiết trong tài liệu này để xác thực và đăng ký mới cho người dùng.

Nếu bạn chọn làm thủ công với các service xác thực của Laravel thay vì sử dụng Fortify, bạn có thể làm như vậy bằng cách làm theo tài liệu có sẵn trong [authentication](/docs/{{version}}/authentication), [reset password](/docs/{{version}}/passwords), và [xác minh email](/docs/{{version}}/verification).

<a name="laravel-fortify-and-laravel-sanctum"></a>
#### Laravel Fortify & Laravel Sanctum

Có một số nhà phát triển trở nên bối rối về sự khác biệt giữa [Laravel Sanctum](/docs/{{version}}/sanctum) và Laravel Fortify. Vì hai package này giải quyết hai vấn đề khác nhau nhưng có liên quan với nhau nên Laravel Fortify và Laravel Sanctum không phải là các package cạnh tranh hoặc loại trừ lẫn nhau.

Laravel Sanctum chỉ quan tâm đến việc quản lý các token API và xác thực người dùng hiện có bằng cách sử dụng session cookie hoặc token. Sanctum không cung cấp bất kỳ route nào xử lý đăng ký người dùng, set lại mật khẩu, vv...

Nếu bạn đang thử xây dựng phần xác thực cho một ứng dụng có cung cấp API hoặc đóng vai trò như là một phần backend cho ứng dụng single-page, thì hoàn toàn có khả năng bạn sẽ sử dụng cả Laravel Fortify (để đăng ký người dùng, set lại mật khẩu, vv... ) và Laravel Sanctum (quản lý token API, xác thực session).

<a name="installation"></a>
## Cài đặt

Để bắt đầu, hãy cài đặt Fortify bằng trình quản lý package Composer:

```nothing
composer require laravel/fortify
```

Tiếp theo, export resource của Fortify bằng lệnh `vendor:publish`:

```bash
php artisan vendor:publish --provider="Laravel\Fortify\FortifyServiceProvider"
```

Lệnh này sẽ export các action của Fortify vào trong thư mục `app/Actions` của bạn, thư mục này sẽ được tạo nếu nó không tồn tại. Ngoài ra, file cấu hình của Fortify và file migration cũng sẽ được export.

Tiếp theo, bạn nên migrate cơ sở dữ liệu của bạn:

```bash
php artisan migrate
```

<a name="the-fortify-service-provider"></a>
### Fortify Service Provider

Lệnh `vendor:publish` được thảo luận ở trên cũng sẽ export class `App\Providers\FortifyServiceProvider`. Bạn nên đảm bảo là class này sẽ được đăng ký trong mảng `providers` của file cấu hình `config/app.php` của ứng dụng.

Service provider của Fortify sẽ đăng ký các action mà Fortify đã được export và hướng dẫn Fortify sử dụng các action khi các tác vụ tương ứng với các action đó được thực thi bởi Fortify.

<a name="fortify-features"></a>
### Fortify Features

File cấu hình `fortify` có chứa một mảng cấu hình `features`. Mảng này định nghĩa các route và các backend chức năng mà Fortify sẽ mặc định thực thị. Nếu bạn không sử dụng Fortify cùng với [Laravel Jetstream](https://jetstream.laravel.com), chúng tôi khuyên bạn chỉ nên bật các tính năng sau, đây là các tính năng xác thực cơ bản được cung cấp bởi hầu hết các ứng dụng Laravel:

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

<a name="disabling-views"></a>
### Disabling Views

Mặc định, Fortify sẽ định nghĩa các route nhằm return về view, chẳng hạn như màn hình đăng nhập hoặc màn hình đăng ký. Tuy nhiên, nếu bạn đang xây dựng một ứng dụng single-page dựa trên JavaScript, bạn có thể không cần thiết các route này. Vì lý do đó, bạn có thể tắt các route này bằng cách set giá trị cấu hình `views` trong file cấu hình `config/fortify.php` của ứng dụng thành `false`:

```php
'views' => false,
```

<a name="disabling-views-and-password-reset"></a>
#### Disabling Views & Password Reset

Nếu bạn chọn tắt view của Fortify và bạn muốn làm tính năng set lại mật khẩu cho ứng dụng của bạn, thì bạn vẫn nên định nghĩa route có tên là `password.reset` chịu trách nhiệm hiển thị view "set lại mật khẩu" của ứng dụng. Điều này là cần thiết vì notification `Illuminate\Auth\Notifications\ResetPassword` của Laravel sẽ tạo ra URL set lại mật khẩu thông qua route có tên `password.reset`.

<a name="authentication"></a>
## Authentication

Để bắt đầu, chúng ta cần hướng dẫn Fortify cách trả về view "đăng nhập" của chúng ta. Hãy nhớ rằng, Fortify là một thư viện xác thực không fontend. Nếu bạn muốn triển khai giao diện người dùng cho các tính năng xác thực của Laravel, bạn nên sử dụng [bộ starter kit](/docs/{{version}}/starter-kits).

Tất cả logic rendering của view xác thực có thể được tùy chỉnh bằng cách sử dụng các phương thức có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn. Fortify sẽ đảm nhiệm việc định nghĩa route `/login` trả về view này:

    use Laravel\Fortify\Fortify;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Fortify::loginView(function () {
            return view('auth.login');
        });

        // ...
    }

Template đăng nhập của bạn phải chứa một form gửi request POST tới `/login`. Route `/login` này sẽ yêu cầu một chuỗi `email` hoặc một chuỗi `username` và một `password`. Tên của trường email hoặc username phải khớp với giá trị `username` trong file cấu hình `config/fortify.php`. Ngoài ra, trường boolean `remember` có thể được cung cấp để cho biết người dùng có muốn sử dụng chức năng "remember me" do Laravel cung cấp hay không.

Nếu thử đăng nhập thành công, Fortify sẽ chuyển hướng bạn đến URI được cấu hình sẵn thông qua tùy chọn cấu hình `home` trong file cấu hình `fortify` của ứng dụng của bạn. Nếu request đăng nhập là request XHR, thì response 200 HTTP sẽ được trả về.

Nếu request không thành công, người dùng sẽ được chuyển hướng trở lại màn hình đăng nhập và các lỗi xác thực sẽ có sẵn cho bạn thông qua [biến template blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. Hoặc, trong trường hợp request là XHR, thì lỗi xác thực sẽ được trả về với response HTTP 422.

<a name="customizing-user-authentication"></a>
### Tuỳ biến User Authentication

Fortify sẽ tự động truy xuất và xác thực người dùng dựa trên các thông tin đăng nhập đã được cung cấp và bộ authentication guard được cấu hình trong ứng dụng của bạn. Tuy nhiên, thỉnh thoảng bạn có thể muốn tùy chỉnh toàn bộ về cách xác thực thông tin đăng nhập và truy xuất người dùng. Rất may, Fortify cho phép bạn dễ dàng thực hiện việc này bằng phương thức `Fortify::authenticateUsing`.

Phương thức này chấp nhận một closure sẽ nhận vào một HTTP request. Closure này sẽ chịu trách nhiệm xác thực thông tin xác thực đăng nhập được đi kèm trong request và trả về instance người dùng được đã đăng nhập. Nếu thông tin đăng nhập không hợp lệ hoặc không tìm thấy người dùng, `null` hoặc `false` sẽ được trả về bởi closure. Thông thường, phương thức này nên được gọi từ phương thức `boot` của `FortifyServiceProvider` của bạn:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::authenticateUsing(function (Request $request) {
        $user = User::where('email', $request->email)->first();

        if ($user &&
            Hash::check($request->password, $user->password)) {
            return $user;
        }
    });

    // ...
}
```

<a name="authentication-guard"></a>
#### Authentication Guard

Bạn có thể tùy chỉnh bộ authentication guard được sử dụng bởi Fortify trong file cấu hình `fortify` của ứng dụng của bạn. Tuy nhiên, bạn nên đảm bảo là guard đã được cấu hình là một implementation của `Illuminate\Contracts\Auth\StatefulGuard`. Nếu bạn đang thử sử dụng Laravel Fortify để xác thực cho một SPA, bạn nên sử dụng guard `web` mặc định của Laravel kết hợp với [Laravel Sanctum](https://laravel.com/docs/sanctum).

<a name="customizing-the-authentication-pipeline"></a>
### Tuỳ biến Authentication Pipeline

Laravel Fortify sẽ xác thực các yêu cầu đăng nhập thông qua một hệ thống các class invokable. Nếu muốn, bạn có thể định nghĩa một hệ thống tùy chỉnh gồm các class mà các request đăng nhập sẽ được dẫn qua. Mỗi class phải có một phương thức `__invoke` để nhận vào một instance `Illuminate\Http\Request` và, giống như các [middleware](/docs/{{version}}/middleware) sẽ có một biến `$next` sẽ được gọi trong đó để truyền request đến class tiếp theo trong hệ thống.

Để định nghĩa một hệ thống tùy chỉnh của bạn, bạn có thể sử dụng phương thức `Fortify::authenticateThrough`. Phương thức này chấp nhận một closure sẽ trả về mảng các class để chuyển yêu cầu đăng nhập qua. Thông thường, phương thức này nên được gọi từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của bạn.

Ví dụ bên dưới sẽ chứa một định nghĩa hệ thống mặc định mà bạn có thể sử dụng làm điểm khởi đầu khi thực hiện việc sửa đổi của bạn:

```php
use Laravel\Fortify\Actions\AttemptToAuthenticate;
use Laravel\Fortify\Actions\EnsureLoginIsNotThrottled;
use Laravel\Fortify\Actions\PrepareAuthenticatedSession;
use Laravel\Fortify\Actions\RedirectIfTwoFactorAuthenticatable;
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
            config('fortify.limiters.login') ? null : EnsureLoginIsNotThrottled::class,
            Features::enabled(Features::twoFactorAuthentication()) ? RedirectIfTwoFactorAuthenticatable::class : null,
            AttemptToAuthenticate::class,
            PrepareAuthenticatedSession::class,
    ]);
});
```

<a name="customizing-authentication-redirects"></a>
### Tuỳ biến Redirect

Nếu thử đăng nhập thành công, Fortify sẽ chuyển hướng bạn đến URI đã được cấu hình trước thông qua tùy chọn cấu hình `home` trong file cấu hình `fortify` của ứng dụng của bạn. Nếu yêu cầu đăng nhập là request XHR, response 200 HTTP sẽ được trả về. Sau khi người dùng đăng xuất khỏi ứng dụng của bạn, người dùng sẽ được chuyển hướng đến URI `/`.

Nếu bạn cần tùy chỉnh nâng cao cho các hành vi này, bạn có thể liên kết việc implementation các contract `LoginResponse` và `LogoutResponse` này vào Laravel [service container](/docs/{{version}}/container). Thông thường, điều này nên được thực hiện trong phương thức `register` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Fortify\Contracts\LogoutResponse;

/**
 * Register any application services.
 *
 * @return void
 */
public function register()
{
    $this->app->instance(LogoutResponse::class, new class implements LogoutResponse {
        public function toResponse($request)
        {
            return redirect('/');
        }
    });
}
```

<a name="two-factor-authentication"></a>
## Two Factor Authentication

Khi bật tính năng xác thực hai lớp của Fortify, người dùng sẽ bị được yêu cầu nhập mã token gồm sáu chữ số trong quá trình xác thực. Mã token này sẽ được tạo ra bằng một time-based one-time password (TOTP) có thể được lấy ra từ bất kỳ ứng dụng xác thực mobile nào tương thích với TOTP, chẳng hạn như Google Authenticator.

Trước khi bắt đầu, trước tiên bạn phải đảm bảo là model `App\Models\User` của ứng dụng của bạn đã sử dụng trait `Laravel\Fortify\TwoFactorAuthenticatable`:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use Notifiable, TwoFactorAuthenticatable;
}
 ```

Tiếp theo, bạn nên tạo thêm một màn hình trong ứng dụng của bạn, chỗ mà người dùng có thể quản lý cài đặt xác thực hai lớp của họ. Màn hình này sẽ cho phép người dùng bật hoặc tắt xác thực hai lớp, cũng như tạo lại mã khôi phục xác thực hai lớp của họ.

> Mặc định, mảng `features` của file cấu hình `fortify` sẽ hướng dẫn cài đặt xác thực hai lớp của Fortify là sẽ yêu cầu xác nhận lại mật khẩu trước khi bị sửa đổi. Do đó, ứng dụng của bạn nên tạo tính năng [xác nhận lại mật khẩu](#password-confirmation) của Fortify trước khi tiếp tục.

<a name="enabling-two-factor-authentication"></a>
### Bật Two Factor Authentication

Để bật xác thực hai lớp, ứng dụng của bạn phải gửi một request POST tới URI `/user/two-factor-authentication` do Fortify định nghĩa. Nếu request thành công, người dùng sẽ được chuyển hướng trở lại URL trước đó và biến session `status` sẽ được set thành `two-factor-authentication-enabled`. Bạn có thể kiểm tra biến session `status` này trong các template của bạn để hiển thị thông báo thành công. Nếu request là một request XHR, thì phản hồi HTTP `200` sẽ được trả về:

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm text-green-600">
        Two factor authentication has been enabled.
    </div>
@endif
```

Tiếp theo, bạn nên hiển thị mã QR xác thực hai lớp để người dùng quét vào ứng dụng xác thực của họ. Nếu bạn đang sử dụng Blade để hiển thị giao diện người dùng, thì bạn có thể lấy ra mã QR SVG này bằng phương thức `twoFactorQrCodeSvg` có sẵn trên instance người dùng:

```php
$request->user()->twoFactorQrCodeSvg();
```

Nếu bạn đang xây dựng một giao diện người dùng được hỗ trợ bởi JavaScript, thì bạn có thể tạo một request XHR GET tới URI `/user/two-factor-qr-code` để lấy ra mã QR xác thực hai lớp của người dùng. URI này sẽ trả về một đối tượng JSON chứa key `svg`.

<a name="displaying-the-recovery-codes"></a>
#### Displaying The Recovery Codes

Bạn cũng nên hiển thị một mã khôi phục cho xác thực hai lớp của người dùng. Các mã khôi phục này cho phép người dùng xác thực nếu họ mất quyền truy cập vào thiết bị di động của họ. Nếu bạn đang sử dụng Blade để hiển thị giao diện người dùng của ứng dụng, bạn có thể lấy ra mã khôi phục thông qua instance người dùng đã được xác thực:

```php
(array) $request->user()->recoveryCodes()
```

Nếu bạn đang xây dựng một giao diện người dùng được hỗ trợ bởi JavaScript, bạn có thể tạo một request XHR GET tới URI `/user/two-factor-recovery-codes`. URI này sẽ trả về một mảng JSON chứa mã khôi phục của người dùng.

Để tạo lại mã khôi phục của người dùng, ứng dụng của bạn phải tạo một request POST tới URI `/user/two-factor-recovery-codes`.

<a name="authenticating-with-two-factor-authentication"></a>
### Authenticating cùng với Two Factor Authentication

Trong quá trình xác thực, Fortify sẽ tự động chuyển hướng người dùng đến màn hình nhập mã xác thực hai lớp của ứng dụng của bạn. Tuy nhiên, nếu ứng dụng của bạn đang thực hiện một yêu cầu đăng nhập bằng XHR, thì response JSON sẽ được trả về sau lần thử xác thực thành công và sẽ chứa một đối tượng JSON có thuộc tính boolean `two_factor`. Bạn nên kiểm tra giá trị này để biết thêm rằng liệu bạn có nên chuyển hướng đến màn hình nhập xác thực hai lớp của ứng dụng hay không.

Để bắt đầu triển khai chức năng xác thực hai lớp, chúng ta cần hướng dẫn Fortify cách trả về view nhập xác thực hai lớp. Tất cả các logic tạo view xác thực của Fortify có thể được tùy chỉnh bằng cách sử dụng các phương thức có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` trong ứng dụng:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::twoFactorChallengeView(function () {
        return view('auth.two-factor-challenge');
    });

    // ...
}
```

Fortify sẽ đảm nhiệm việc định nghĩa route `/two-factor-challenge` sẽ trả về view này. Template `two-factor-challenge` của bạn phải chứa một form sẽ gửi một request POST tới URI `/two-factor-challenge`. Action `/two-factor-challenge` sẽ yêu cầu trường `code` chứa mã token TOTP hợp lệ hoặc trường `recovery_code` sẽ chứa một trong các mã khôi phục của người dùng.

Nếu thử đăng nhập thành công, Fortify sẽ chuyển hướng người dùng đến URI được cấu hình sẵn thông qua tùy chọn cấu hình `home` trong file cấu hình `fortify` của ứng dụng của bạn. Nếu request đăng nhập là request XHR, thì phản hồi HTTP 204 sẽ được trả về.

Nếu request không thành công, thì người dùng sẽ được chuyển hướng trở lại màn hình nhập xác thực hai lớp và lỗi xác thực sẽ có sẵn cho bạn thông qua [biến template blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. Hoặc, trong trường hợp request là XHR, thì lỗi xác thực sẽ được trả về với phản hồi HTTP 422.

<a name="disabling-two-factor-authentication"></a>
### Tắt Two Factor Authentication

Để tắt xác thực hai lớp, ứng dụng của bạn phải thực hiện một request DELETE tới URI `/user/two-factor-authentication`. Hãy nhớ rằng, URI xác thực hai lớp của Fortify  sẽ yêu cầu [xác nhận lại mật khẩu](#password-confirmation) trước khi được gọi.

<a name="registration"></a>
## Đăng ký

Để bắt đầu triển khai chức năng đăng ký cho ứng dụng, chúng ta cần hướng dẫn Fortify trả về view "đăng ký" của chúng ta. Hãy nhớ rằng, Fortify là một thư viện xác thực không fontend. Nếu bạn muốn triển khai giao diện người dùng cho các tính năng xác thực của Laravel, bạn nên sử dụng [bộ starter kit](/docs/{{version}}/starter-kits).

Tất cả logic tạo view của Fortify có thể được tùy chỉnh bằng cách sử dụng các phương thức có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::registerView(function () {
        return view('auth.register');
    });

    // ...
}
```

Fortify sẽ đảm nhận việc định nghĩa route `/register` sẽ trả về view này. Template `register` của bạn phải chứa một form tạo request POST tới URI `/register` do Fortify định nghĩa.

URI `/register` yêu cầu một chuỗi `name`, một chuỗi địa chỉ email hoặc tên người dùng, `password` và trường `password_confirmation`. Tên của trường email hoặc tên người dùng phải khớp với giá trị cấu hình`username` được định nghĩa trong file cấu hình `fortify` của ứng dụng của bạn.

Nếu thử đăng ký thành công, Fortify sẽ chuyển hướng người dùng đến URI được cấu hình sẵn thông qua tùy chọn cấu hình `home` trong file cấu hình `fortify` của ứng dụng của bạn. Nếu yêu cầu đăng nhập là một request XHR, thì phản hồi 200 HTTP sẽ được trả về.

Nếu request không thành công, người dùng sẽ được chuyển hướng trở lại màn hình đăng ký và các lỗi xác thực sẽ có sẵn cho bạn thông qua [biến template blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. Hoặc, trong trường hợp request là XHR, thì lỗi xác thực sẽ được trả về với response HTTP 422.

<a name="customizing-registration"></a>
### Tuỳ biến đăng ký

Quá trình xác thực và tạo người dùng có thể được tùy chỉnh bằng cách sửa action `App\Actions\Fortify\CreateNewUser` đã được tạo khi bạn cài đặt Laravel Fortify.

<a name="password-reset"></a>
## Password Reset

<a name="requesting-a-password-reset-link"></a>
### Yêu cầu link password reset

Để bắt đầu triển khai chức năng set lại mật khẩu cho ứng dụng của bạn, chúng ta sẽ cần hướng dẫn Fortify trả về view "quên mật khẩu". Hãy nhớ rằng, Fortify là một thư viện xác thực không fontend. Nếu bạn muốn triển khai giao diện người dùng cho các tính năng xác thực của Laravel, bạn nên sử dụng [bộ starter kit](/docs/{{version}}/starter-kits).

Tất cả logic tạo view của Fortify có thể được tùy chỉnh bằng cách sử dụng các phương thức có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::requestPasswordResetLinkView(function () {
        return view('auth.forgot-password');
    });

    // ...
}
```

Fortify sẽ đảm nhận việc định nghĩa route `/forgot-password` sẽ trả về view này. Template `register` của bạn phải chứa một form tạo request POST tới URI `/forgot-password`.

URI `/forgot-password` yêu cầu một trường chuỗi `email`. Tên của trường hoặc cột cơ sở dữ liệu này phải khớp với giá trị cấu hình `email` trong file cấu hình `fortify` của ứng dụng của bạn.

<a name="handling-the-password-reset-link-request-response"></a>
#### Handling The Password Reset Link Request Response

Nếu request set lại mật khẩu thành công, Fortify sẽ chuyển hướng người dùng quay trở lại URI `/forgot-password` và gửi email cho người dùng bằng link an toàn mà người dùng có thể sử dụng nó để set lại mật khẩu của họ. Nếu request là request XHR, thì phản hồi 200 HTTP sẽ được trả về.

Sau khi được chuyển hướng trở lại URI `/forgot-password` sau khi request thành công, biến session `status` có thể được sử dụng để hiển thị trạng thái của quá trình request set lại mật khẩu này. Giá trị của biến session này sẽ khớp với một trong các chuỗi được translation được định nghĩa trong [file ngôn ngữ](/docs/{{version}}/localization) `password` của ứng dụng của bạn:

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

Nếu request không thành công, người dùng sẽ được chuyển hướng trở lại màn hình request set lại mật khẩu và các lỗi xác thực sẽ có sẵn cho bạn thông qua [biến template blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. Hoặc, trong trường hợp request là XHR, thì lỗi xác thực sẽ được trả về với response HTTP 422.

<a name="resetting-the-password"></a>
### Reset password

Để hoàn thành việc triển khai chức năng set lại mật khẩu của ứng dụng, chúng ta cần hướng dẫn Fortify trả về view "set lại mật khẩu".

Tất cả logic tạo view của Fortify có thể được tùy chỉnh bằng cách sử dụng các phương thức có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::resetPasswordView(function ($request) {
        return view('auth.reset-password', ['request' => $request]);
    });

    // ...
}
```

Fortify sẽ đảm nhận việc định nghĩa route để hiển thị view này. Template `reset-password` của bạn phải chứa một form gửi một request POST tới `/reset-password`.

URI `/reset-password` sẽ yêu cầu trường chuỗi `email`, trường `password`, trường `password_confirmation` và trường hidden có tên là `token` chứa giá trị của `request()->route('token')`. Tên của trường hoặc cột cơ sở dữ liệu "email" phải khớp với giá trị cấu hình `email` được định nghĩa trong file cấu hình `fortify` của ứng dụng của bạn.

<a name="handling-the-password-reset-response"></a>
#### Handling The Password Reset Response

Nếu yêu cầu set lại mật khẩu thành công, Fortify sẽ chuyển hướng trở lại route `/login` để người dùng có thể đăng nhập bằng mật khẩu mới của họ. Ngoài ra, biến session `status` sẽ được lưu vào session để bạn có thể hiển thị trạng thái của request set lại mật khẩu thành công trên màn hình đăng nhập của bạn:

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

Nếu request là request XHR, phản hồi 200 HTTP sẽ được trả về.

Nếu request không thành công, người dùng sẽ được chuyển hướng trở lại màn hình set lại mật khẩu và các lỗi xác thực sẽ có sẵn cho bạn thông qua [biến template blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. Hoặc, trong trường hợp request là XHR, thì lỗi xác thực sẽ được trả về với response HTTP 422.

<a name="customizing-password-resets"></a>
### Tuỳ biến password reset

Quy trình set lại mật khẩu có thể được tùy chỉnh bằng cách sửa action `App\Actions\ResetUserPassword` đã được tạo ra khi bạn cài đặt Laravel Fortify.

<a name="email-verification"></a>
## Email Verification

Sau khi đăng ký, bạn có thể muốn người dùng xác minh địa chỉ email của họ trước khi họ tiếp tục truy cập ứng dụng của bạn. Để bắt đầu, hãy đảm bảo feature `emailVerification` đã được bật trong mảng `features` của file cấu hình `fortify` của bạn. Tiếp theo, bạn nên đảm bảo rằng class `App\Models\User` của bạn đã implement interface `Illuminate\Contracts\Auth\MustVerifyEmail`.

Sau khi đã hoàn thành hai bước thiết lập này, người dùng đăng ký mới sẽ nhận được email nhắc họ xác minh quyền sở hữu địa chỉ email của họ. Tuy nhiên, chúng ta cần thông báo cho Fortify cách hiển thị màn hình xác minh email cho người dùng rằng họ cần nhấp vào link xác minh trong email.

Tất cả logic tạo view của Fortify có thể được tùy chỉnh bằng cách sử dụng các phương thức có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::verifyEmailView(function () {
        return view('auth.verify-email');
    });

    // ...
}
```

Fortify sẽ đảm nhiệm việc định nghĩa route hiển thị view này khi người dùng được chuyển hướng đến URI `/email/verify` bởi middleware `verified` đã được tích hợp sẵn trong Laravel.

Template `verify-email` của bạn phải chứa một thông báo cung cấp thông tin hướng dẫn người dùng nhấn vào link xác minh email đã được gửi đến địa chỉ email của họ.

<a name="resending-email-verification-links"></a>
#### Resending Email Verification Links

Nếu muốn, bạn cũng có thể thêm một nút vào template `verify-email` của ứng dụng để kích hoạt request POST tới URI `/email/verification-notification`. Khi URI này nhận được request, một link email xác minh mới sẽ được gửi qua email cho người dùng, cho phép người dùng nhận được một link xác minh mới nếu link trước đó vô tình bị xóa hoặc bị mất.

Nếu việc gửi lại link này thành công, Fortify sẽ chuyển hướng người dùng quay lại URI `/email/verify` với biến session `status`, cho phép bạn hiển thị thông báo cho họ về hoạt động đã được thực hiện thành công. Nếu request là request XHR, phản hồi HTTP 202 sẽ được trả về:

```html
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        A new email verification link has been emailed to you!
    </div>
@endif
```

<a name="protecting-routes"></a>
### Bảo vệ route

Để chỉ định một route hoặc một nhóm route sẽ yêu cầu người dùng phải xác minh địa chỉ email của họ, bạn nên gán middleware `verified` được tích hợp sẵn trong Laravel vào route. Middleware này được đăng ký trong class `App\Http\Kernel` của ứng dụng của bạn:

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

<a name="password-confirmation"></a>
## Xác nhận password

Trong khi xây dựng ứng dụng của bạn, đôi khi bạn có thể có các action yêu cầu người dùng phải xác nhận lại mật khẩu của họ trước khi action được thực hiện. Thông thường, các route này được bảo vệ bởi middleware `password.confirm` được tích hợp sẵn trong Laravel.

Để bắt đầu triển khai chức năng xác nhậ lại mật khẩu, chúng ta cần hướng dẫn Fortify trả về view "xác nhận lại mật khẩu" của ứng dụng. Hãy nhớ rằng, Fortify là một thư viện xác thực không fontend. Nếu bạn muốn triển khai giao diện người dùng cho các tính năng xác thực của Laravel, bạn nên sử dụng [bộ starter kit](/docs/{{version}}/starter-kits).

Tất cả logic tạo view của Fortify có thể được tùy chỉnh bằng cách sử dụng các phương thức có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::confirmPasswordView(function () {
        return view('auth.confirm-password');
    });

    // ...
}
```

Fortify sẽ đảm nhận việc định nghĩa URI `/user/confirm-password` sẽ trả về view này. Template `confirm-password` của bạn phải chứa một form sẽ gửi một request POST tới URI `/user/confirm-password`.URI `/user/confirm-password` sẽ yêu cầu trường `password` chứa mật khẩu hiện tại của người dùng.

Nếu mật khẩu khớp với mật khẩu hiện tại của người dùng, Fortify sẽ chuyển hướng người dùng đến route mà họ đang cố truy cập. Nếu request là request XHR, thì phản hồi HTTP 201 sẽ được trả về.

Nếu request không thành công, người dùng sẽ được chuyển hướng trở lại màn hình xác nhận lại mật khẩu và các lỗi xác thực sẽ có sẵn cho bạn thông qua biến template blade `$errors`. Hoặc, trong trường hợp request là XHR, thì lỗi xác thực sẽ được trả về với response HTTP 422.
