# Resetting Passwords

- [Giới thiệu](#introduction)
    - [Chuẩn bị Model](#model-preparation)
    - [Chuẩn bị Database](#database-preparation)
    - [Cấu hình Trusted Hosts](#configuring-trusted-hosts)
- [Routing](#routing)
    - [Requesting The Password Reset Link](#requesting-the-password-reset-link)
    - [Resetting The Password](#resetting-the-password)
- [Xoá Token hết hạn](#deleting-expired-tokens)
- [Tuỳ chỉnh](#password-customization)

<a name="introduction"></a>
## Giới thiệu

Hầu hết các ứng dụng web đều cung cấp một cách nào đó để người dùng reset lại mật khẩu của họ. Thay vì buộc bạn phải làm lại việc này cho mọi ứng dụng mà bạn tạo ra, Laravel cung cấp một service thuận tiện để gửi link reset mật khẩu và reset lại mật khẩu một cách an toàn.

> [!NOTE]
> Bạn muốn bắt đầu nhanh chóng? Hãy cài đặt [starter kit](/docs/{{version}}/starter-kits) của Laravel trong ứng dụng mới của bạn. Bộ khởi đầu của Laravel sẽ đảm nhiệm việc xây dựng toàn bộ hệ thống xác thực cho bạn, bao gồm cả việc reset mật khẩu.

<a name="model-preparation"></a>
### Chuẩn bị Model

Trước khi sử dụng các tính năng reset mật khẩu của Laravel, model `App\Models\User` của ứng dụng của bạn phải sử dụng trait `Illuminate\Notifications\Notifiable`. Thông thường, mặc định trait này đã được đưa vào model `App\Models\User` khi tạo ứng dụng Laravel mới.

Tiếp theo, hãy chú ý model `App\Models\User` của bạn phải được implement từ contract `Illuminate\Contracts\Auth\CanResetPassword`. Model `App\Models\User` đi kèm với framework Laravel đã implement interface này và sử dụng trait `Illuminate\Auth\Passwords\CanResetPassword` để chứa các phương thức để implement interface đó.

<a name="database-preparation"></a>
### Chuẩn bị Database

Một bảng phải được tạo để lưu trữ các mã token reset của application của bạn. Mặc định, việc migration cho bảng này đã có sẵn trong Laravel application, vì vậy bạn chỉ cần migrate cơ sở dữ liệu của bạn để tạo bảng này:

```shell
php artisan migrate
```

<a name="configuring-trusted-hosts"></a>
### Cấu hình Trusted Hosts

Mặc định, Laravel sẽ respond tất cả các request mà nó nhận được bất kể nội dung của header `Host` của request HTTP đó là gì. Ngoài ra, giá trị của header `Host` sẽ được sử dụng khi tạo URL cho ứng dụng của bạn trong khi web request.

Thông thường, bạn nên cấu hình máy chủ web của bạn, chẳng hạn như Nginx hoặc Apache, để chỉ gửi request đến ứng dụng giống với một host name nhất định. Tuy nhiên, nếu bạn không có khả năng tùy chỉnh trực tiếp máy chủ web của bạn và cần hướng dẫn Laravel chỉ phản hồi với một số host name nhất định, bạn có thể làm như vậy bằng cách bật middleware `App\Http\Middleware\TrustHosts` trong ứng dụng của bạn. Điều này đặc biệt quan trọng khi ứng dụng của bạn cung cấp chức năng set lại mật khẩu.

Để tìm hiểu thêm về middleware này, vui lòng tham khảo [tài liệu về middleware `TrustHosts`](/docs/{{version}}/requests#configuring-trusted-hosts).

<a name="routing"></a>
## Routing

Để triển khai đúng cách hỗ trợ cho phép người dùng set lại mật khẩu, chúng ta sẽ cần định nghĩa một số route. Đầu tiên, chúng ta sẽ cần một cặp route để xử lý việc cho phép người dùng yêu cầu link set lại mật khẩu thông qua địa chỉ email của họ. Thứ hai, chúng ta sẽ cần một cặp route để xử lý việc set lại mật khẩu sau khi người dùng truy cập vào link set lại mật khẩu được gửi thông qua email của họ và hoàn thành form set lại mật khẩu.

<a name="requesting-the-password-reset-link"></a>
### Requesting The Password Reset Link

<a name="the-password-reset-link-request-form"></a>
#### The Password Reset Link Request Form

Đầu tiên, chúng ta sẽ định nghĩa các route cần thiết để yêu cầu link set lại mật khẩu. Để bắt đầu, chúng ta sẽ định nghĩa một route trả về view với form yêu cầu link set lại mật khẩu:

    Route::get('/forgot-password', function () {
        return view('auth.forgot-password');
    })->middleware('guest')->name('password.request');

View mà được route này trả về phải có form chứa field `email`, field này sẽ cho phép người dùng yêu cầu link set lại mật khẩu cho một địa chỉ email nhất định.

<a name="password-reset-link-handling-the-form-submission"></a>
#### Handling The Form Submission

Tiếp theo, chúng ta sẽ định nghĩa một route để xử lý request form từ view "quên mật khẩu". Route này sẽ chịu trách nhiệm xác thực địa chỉ email và gửi yêu cầu set lại mật khẩu đến người dùng tương ứng:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Password;

    Route::post('/forgot-password', function (Request $request) {
        $request->validate(['email' => 'required|email']);

        $status = Password::sendResetLink(
            $request->only('email')
        );

        return $status === Password::RESET_LINK_SENT
                    ? back()->with(['status' => __($status)])
                    : back()->withErrors(['email' => __($status)]);
    })->middleware('guest')->name('password.email');

Trước khi tiếp tục, chúng ta hãy xem xét route này chi tiết hơn. Đầu tiên, thuộc tính `email` của request sẽ được validate. Tiếp theo, chúng ta sẽ sử dụng "password broker" có sẵn của Laravel (thông qua facade `Password`) để gửi link set lại mật khẩu cho người dùng. Password broker sẽ đảm nhiệm việc lấy ra người dùng theo một field nhất định (trong trường hợp này là địa chỉ email) và gửi cho người dùng link set lại mật khẩu thông qua [hệ thống notification](/docs/{{version}}/notifications).

Phương thức `sendResetLink` sẽ trả về một biến "trạng thái". Trạng thái này có thể được chuyển sang ngôn ngữ khác bằng cách sử dụng helper [localization](/docs/{{version}}/localization) của Laravel để hiển thị thông báo cho người dùng về trạng thái yêu cầu của họ. Việc chuyển ngôn ngữ này được xác định bởi file ngôn ngữ `lang/{lang}/passwords.php` trong ứng dụng của bạn. Các mục cho các giá trị có thể có của biến trạng thái sẽ nằm sẵn trong file ngôn ngữ `passwords`.

> [!NOTE]
> Mặc định, Laravel application không chứa thư mục `lang`. Nếu bạn muốn tùy chỉnh các file ngôn ngữ của Laravel, bạn có thể export chúng thông qua lệnh Artisan `lang:publish`.

Bạn có thể thắc mắc làm thế nào mà Laravel biết cách lấy ra bản ghi người dùng từ cơ sở dữ liệu ứng dụng của bạn khi gọi phương thức `sendResetLink` của facade `Password`. Password broker của Laravel sẽ sử dụng "user providers" của hệ thống authentication của bạn để lấy ra các bản ghi trong cơ sở dữ liệu. User provider mà được password broker sử dụng sẽ được cấu hình trong mảng cấu hình `passwords` của file cấu hình `config/auth.php` của bạn. Để tìm hiểu thêm về cách viết user provider tùy chỉnh, hãy tham khảo [tài liệu authentication](/docs/{{version}}/authentication#adding-custom-user-providers).

> [!NOTE]
> Khi bạn muốn tự làm chức năng set lại mật khẩu này, thì bạn phải tự định nghĩa nội dung của các view và route của nó. Nếu bạn muốn một bộ gồm tất cả logic về xác minh và xác thực cần thiết, hãy xem [starter kit của Laravel](/docs/{{version}}/starter-kits).

<a name="resetting-the-password"></a>
### Resetting The Password

<a name="the-password-reset-form"></a>
#### The Password Reset Form

Tiếp theo, chúng ta sẽ định nghĩa các route cần thiết để set lại mật khẩu khi người dùng nhấn vào link set lại mật khẩu đã được gửi qua email cho họ và cung cấp một mật khẩu mới. Trước tiên, hãy định nghĩa route sẽ hiển thị form set lại mật khẩu mà được hiển thị khi người dùng nhấn vào link set lại mật khẩu. Route này sẽ nhận vào tham số `token` mà chúng ta sẽ sử dụng sau này để xác minh yêu cầu set lại mật khẩu:

    Route::get('/reset-password/{token}', function (string $token) {
        return view('auth.reset-password', ['token' => $token]);
    })->middleware('guest')->name('password.reset');

View được route này trả về sẽ hiển thị một form chứa các field `email`, field `password`, field `password_confirmation` và field `token` hidden, field này phải chứa giá trị của `$token` mà đã được nhận bởi route của chúng ta.

<a name="password-reset-handling-the-form-submission"></a>
#### Handling The Form Submission

Tất nhiên, chúng ta sẽ cần định nghĩa một route để xử lý việc gửi form set lại mật khẩu. Route này sẽ chịu trách nhiệm xác thực request đến và cập nhật mật khẩu của người dùng trong cơ sở dữ liệu:

    use App\Models\User;
    use Illuminate\Auth\Events\PasswordReset;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Facades\Password;
    use Illuminate\Support\Str;

    Route::post('/reset-password', function (Request $request) {
        $request->validate([
            'token' => 'required',
            'email' => 'required|email',
            'password' => 'required|min:8|confirmed',
        ]);

        $status = Password::reset(
            $request->only('email', 'password', 'password_confirmation', 'token'),
            function (User $user, string $password) {
                $user->forceFill([
                    'password' => Hash::make($password)
                ])->setRememberToken(Str::random(60));

                $user->save();

                event(new PasswordReset($user));
            }
        );

        return $status === Password::PASSWORD_RESET
                    ? redirect()->route('login')->with('status', __($status))
                    : back()->withErrors(['email' => [__($status)]]);
    })->middleware('guest')->name('password.update');

Trước khi tiếp tục, chúng ta hãy xem xét route này một cách chi tiết hơn. Đầu tiên, các thuộc tính `token`, `email` và `password` của request sẽ được validate. Tiếp theo, chúng ta sẽ sử dụng "password broker" có sẵn của Laravel (thông qua facade `Password`) để validate thông tin xác thực của request đặt lại mật khẩu.

Nếu token, địa chỉ email và mật khẩu được cung cấp cho password broker là hợp lệ, thì closure mà được truyền cho phương thức `reset` sẽ được gọi. Trong closure này sẽ nhận vào một instance người dùng và mật khẩu được nhập từ form set lại mật khẩu, sau đó chúng ta có thể cập nhật mật khẩu của người dùng trong cơ sở dữ liệu.

Phương thức `reset` sẽ trả về một biến "trạng thái". Trạng thái này có thể được chuyển sang ngôn ngữ khác bằng cách sử dụng helper [localization](/docs/{{version}}/localization) của Laravel để hiển thị thông báo cho người dùng về trạng thái yêu cầu của họ. Việc chuyển ngôn ngữ này được xác định bởi file ngôn ngữ `lang/{lang}/passwords.php` trong ứng dụng của bạn. Các mục cho các giá trị có thể có của biến trạng thái sẽ nằm sẵn trong file ngôn ngữ `passwords`. Nếu ứng dụng của bạn không chứa thư mục `lang`, bạn có thể tạo ra thư mục đó bằng lệnh Artisan `lang:publish`.

Trước khi tiếp tục, bạn có thể thắc mắc làm thế nào mà Laravel biết cách lấy ra bản ghi người dùng từ cơ sở dữ liệu ứng dụng của bạn khi gọi phương thức `reset` của facade `Password`. Password broker của Laravel sẽ sử dụng "user providers" của hệ thống authentication của bạn để lấy ra các bản ghi trong cơ sở dữ liệu. User provider mà được password broker sử dụng sẽ được cấu hình trong mảng cấu hình `passwords` của file cấu hình `config/auth.php` của bạn. Để tìm hiểu thêm về cách viết user provider tùy chỉnh, hãy tham khảo [tài liệu authentication](/docs/{{version}}/authentication#adding-custom-user-providers).

<a name="deleting-expired-tokens"></a>
## Xoá Token hết hạn

Token reset password đã hết hạn sẽ vẫn còn nằm trong cơ sở dữ liệu của bạn. Tuy nhiên, bạn có thể dễ dàng xóa các bản ghi này bằng lệnh Artisan `auth:clear-resets`:

```shell
php artisan auth:clear-resets
```

Nếu bạn muốn tự động hóa quy trình này, hãy cân nhắc thêm lệnh vào [scheduler](/docs/{{version}}/scheduling) trong ứng dụng của bạn:

    $schedule->command('auth:clear-resets')->everyFifteenMinutes();

<a name="password-customization"></a>
## Customization

<a name="reset-link-customization"></a>
#### Reset Link Customization

Bạn có thể tùy chỉnh URL link set lại mật khẩu bằng phương thức `createUrlUsing` do class notification `ResetPassword` cung cấp. Phương thức này chấp nhận một closure nhận vào một instance người dùng đang nhận thông báo cũng như một token set lại mật khẩu. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của service provider `App\Providers\AuthServiceProvider`:

    use App\Models\User;
    use Illuminate\Auth\Notifications\ResetPassword;

    /**
     * Register any authentication / authorization services.
     */
    public function boot(): void
    {
        ResetPassword::createUrlUsing(function (User $user, string $token) {
            return 'https://example.com/reset-password?token='.$token;
        });
    }

<a name="reset-email-customization"></a>
#### Reset Email Customization

Bạn có thể dễ dàng sửa class notification được sử dụng để gửi link reset mật khẩu cho người dùng. Để bắt đầu, hãy ghi đè phương thức `sendPasswordResetNotification` trong model `App\Models\User` của bạn. Trong phương thức này, bạn có thể gửi thông báo bằng bất kỳ [class notification](/docs/{{version}}/notifications) do chính bạn tạo. `$token` set lại mật khẩu là tham số đầu tiên mà phương thức nhận vào. Bạn có thể sử dụng `$token` này để tạo URL set lại mật khẩu mà bạn chọn và gửi thông báo cho người dùng:

    use App\Notifications\ResetPasswordNotification;

    /**
     * Send a password reset notification to the user.
     *
     * @param  string  $token
     */
    public function sendPasswordResetNotification($token): void
    {
        $url = 'https://example.com/reset-password?token='.$token;

        $this->notify(new ResetPasswordNotification($url));
    }
