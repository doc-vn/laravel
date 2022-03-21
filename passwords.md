# Resetting Passwords

- [Giới thiệu](#introduction)
- [Các chú ý về database](#resetting-database)
- [Routing](#resetting-routing)
- [Views](#resetting-views)
- [Sau khi reset password](#after-resetting-passwords)
- [Tuỳ chỉnh](#password-customization)

<a name="introduction"></a>
## Giới thiệu

> {tip} **Nếu bạn muốn bắt đầu nhanh?** Bạn chỉ cần chạy `php artisan make:auth` trong terminal của một application Laravel mới và điều hướng trình duyệt của bạn đến địa chỉ `http://your-app.test/register` hoặc bất kỳ URL nào khác được gán cho application của bạn. Lệnh này sẽ đảm nhiệm việc hỗ trợ toàn bộ hệ thống xác thực của bạn, bao gồm cả việc reset mật khẩu!

Hầu hết các ứng dụng web đều cung cấp một cách để người dùng có thể reset mật khẩu mà họ đã quên. Thay vì buộc bạn phải thực hiện lại điều này trên mỗi ứng dụng, Laravel cung cấp các phương thức thuận tiện để gửi lời nhắc mật khẩu và thực hiện reset mật khẩu.

> {note} Trước khi sử dụng các tính năng reset mật khẩu của Laravel, user model của bạn phải sử dụng trait `Illuminate\Notifications\Notifiable`.

<a name="resetting-database"></a>
## Các chú ý về database

Để bắt đầu, hãy chú ý model `App\User` của bạn phải được implement từ contract `Illuminate\Contracts\Auth\CanResetPassword`. Model `App\User` đi kèm với framework Laravel đã implement interface này và sử dụng trait `Illuminate\Auth\Passwords\CanResetPassword` để chứa các phương thức để implement interface đó.

#### Generating The Reset Token Table Migration

Tiếp theo, một bảng phải được tạo để lưu trữ các mã token reset. Mặc định, việc migration cho bảng này đã được đi kèm trong Laravel và nằm trong thư mục `database/migrations`. Vì vậy, tất cả những gì bạn cần làm là chạy migration cho cơ sở dữ liệu của bạn:

    php artisan migrate

<a name="resetting-routing"></a>
## Routing

Laravel đã có sẵn các class `Auth\ForgotPasswordController` và `Auth\ResetPasswordController` để chứa các logic cần thiết cho việc gửi e-mail reset mật khẩu. Tất cả các route cần thiết để thực hiện việc reset mật khẩu có thể được tạo bằng lệnh Artisan `make:auth`:

    php artisan make:auth

<a name="resetting-views"></a>
## Views

Một lần nữa, Laravel sẽ tạo ra tất cả các view cần thiết để reset mật khẩu khi bạn chạy lệnh `make:auth`. Các view này được lưu trong `resources/views/auth/passwords`. Bạn có thể tùy biến chúng nếu cần thiết cho application của bạn.

<a name="after-resetting-passwords"></a>
## Sau khi reset password

Khi bạn đã định nghĩa các route và các view để reset mật khẩu của người dùng, bạn có thể truy cập các route này trong trình duyệt của bạn: `/password/reset`. `ForgotPasswordController` đi kèm với framework có chứa sẵn logic để gửi e-mail reset mật khẩu, trong khi `ResetPasswordController` chứa logic để reset mật khẩu người dùng.

Sau khi một mật khẩu đã được reset, người dùng sẽ tự động đăng nhập vào ứng dụng và được chuyển hướng đến màn hình `/home`. Bạn có thể tùy biến vị trí chuyển đến sau khi reset mật khẩu bằng cách định nghĩa thuộc tính `redirectTo` trong class `ResetPasswordController`:

    protected $redirectTo = '/dashboard';

> {note} Mặc định, token reset mật khẩu sẽ hết hạn sau một giờ. Bạn có thể thay đổi điều này thông qua tùy chọn `expire` trong file `config/auth.php` của bạn.

<a name="password-customization"></a>
## Tuỳ chỉnh

#### Authentication Guard Customization

Trong file cấu hình `auth.php` của bạn, bạn có thể cấu hình nhiều "guards" để có thể định nghĩa nhiều hành vi authentication cho nhiều loại user. Bạn có thể tùy chỉnh `ResetPasswordController` đi kèm để sử dụng bất kỳ guard nào mà bạn chọn bằng cách ghi đè phương thức `guard` trong controller đó. Phương thức này sẽ trả về một instance guard:

    use Illuminate\Support\Facades\Auth;

    /**
     * Get the guard to be used during password reset.
     *
     * @return \Illuminate\Contracts\Auth\StatefulGuard
     */
    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Password Broker Customization

Trong file cấu hình `auth.php` của bạn, bạn có thể cấu hình nhiều "brokers" mật khẩu, có thể được sử dụng để reset mật khẩu trên nhiều bảng user. Bạn có thể tùy chỉnh các class `ForgotPasswordController` và `ResetPasswordController` đi kèm để sử dụng broker mà bạn chọn, bằng cách ghi đè phương thức `broker`:

    use Illuminate\Support\Facades\Password;

    /**
     * Get the broker to be used during password reset.
     *
     * @return PasswordBroker
     */
    public function broker()
    {
        return Password::broker('name');
    }

#### Reset Email Customization

Bạn có thể dễ dàng sửa class notification được sử dụng để gửi link reset mật khẩu cho người dùng. Để bắt đầu, hãy ghi đè phương thức `sendPasswordResetNotification` trong model `User` của bạn. Trong phương thức này, bạn có thể gửi thông báo bằng bất kỳ class notification nào mà bạn chọn. `$token` reset mật khẩu là tham số đầu tiên mà phương thức nhận:

    /**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }
