# Email Verification

- [Giới thiệu](#introduction)
- [Chuẩn bị cơ sở dữ liệu](#verification-database)
- [Routing](#verification-routing)
    - [Bảo vệ Route](#protecting-routes)
- [View](#verification-views)
- [Sau khi xác minh email](#after-verifying-emails)
- [Event](#events)

<a name="introduction"></a>
## Giới thiệu

Nhiều web application yêu cầu người dùng xác minh địa chỉ email của họ trước khi sử dụng application. Thay vì bắt bạn phải thực hiện lại điều này trên mỗi application của bạn, Laravel cung cấp các phương thức thuận tiện để gửi và xác minh các email.

### Chuẩn bị model

Để bắt đầu, hãy kiểm tra model `App\User` của bạn đã implement contract `Illuminate\Contracts\Auth\MustVerifyEmail`:

    <?php

    namespace App;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

<a name="verification-database"></a>
## Chuẩn bị cơ sở dữ liệu

#### The Email Verification Column

Tiếp theo, bảng `user` của bạn phải chứa cột `email_verified_at` để lưu ngày giờ mà địa chỉ email được xác minh. Mặc định, migration của bảng `user` tồn tại trong framework Laravel đã chứa cột này. Vì vậy, tất cả những gì bạn cần làm là chạy migration cơ sở dữ liệu của bạn:

    php artisan migrate

<a name="verification-routing"></a>
## Routing

Laravel có chứa một class `Auth\VerificationController` dành cho những logic cần thiết để gửi các link verify và cách verify email. Để đăng ký các route cần thiết cho controller này, hãy truyền một tùy chọn `verify` cho phương thức `Auth::routes`:

    Auth::routes(['verify' => true]);

<a name="protecting-routes"></a>
### Bảo vệ Route

[Route middleware](/docs/{{version}}/middleware) có thể được sử dụng để chỉ cho phép những người dùng mà đã xác minh được truy cập vào một route nhất định. Laravel cung cấp một middleware `verified`, được định nghĩa tại `Illuminate\Auth\Middleware\EnsureEmailIsVerified`. Vì middleware này đã được đăng ký trong HTTP kernel của ứng dụng của bạn, nên tất cả những gì bạn cần là gắn middleware này vào một định nghĩa route:

    Route::get('profile', function () {
        // Only verified users may enter...
    })->middleware('verified');

<a name="verification-views"></a>
## View

Để tạo ra tất cả các view cần thiết cho việc xác minh email, bạn có thể sử dụng package Composer `laravel/ui`:

    composer require laravel/ui  "^1.2" --dev

    php artisan ui vue --auth

Các view xác minh email này sẽ được lưu trong `resources/views/auth/verify.blade.php`. Bạn có thể thoải mái tùy biến các trang này nếu cần cho ứng dụng của bạn.

<a name="after-verifying-emails"></a>
## Sau khi xác minh email

Sau khi địa chỉ email được xác nhận, người dùng sẽ được tự động chuyển hướng đến vị trí `/home`. Bạn có thể tùy chỉnh vị trí chuyển hướng này bằng cách định nghĩa phương thức hoặc thuộc tính `redirectTo` trên class `VerificationController`:

    protected $redirectTo = '/dashboard';

<a name="events"></a>
## Event

Laravel sẽ gửi các [events](/docs/{{version}}/events) trong quá trình xác nhận email. Bạn có thể gắn listener vào các event này trong `EventServiceProvider` của bạn:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Event\Verified' => [
            'App\Listeners\LogVerifiedUser',
        ],
    ];
