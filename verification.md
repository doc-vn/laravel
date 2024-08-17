# Email Verification

- [Giới thiệu](#introduction)
    - [Chuẩn bị Model](#model-preparation)
    - [Chuẩn bị cơ sở dữ liệu](#database-preparation)
- [Routing](#verification-routing)
    - [Thông báo xác minh email](#the-email-verification-notice)
    - [Xử lý xác minh email](#the-email-verification-handler)
    - [Gửi lại email xác minh](#resending-the-verification-email)
    - [Bảo vệ Route](#protecting-routes)
- [Customization](#customization)
- [Event](#events)

<a name="introduction"></a>
## Giới thiệu

Nhiều web application yêu cầu người dùng xác minh địa chỉ email của họ trước khi sử dụng application. Thay vì bắt bạn phải triển khai lại tính năng này bằng tay cho mỗi application bạn tạo, Laravel cung cấp sẵn các service thuận tiện để gửi và xác minh các email.

> **Note**
> Bạn muốn bắt đầu nhanh không? Cài đặt một trong các [bộ công cụ khởi tạo ứng dụng của Laravel](/docs/{{version}}/starter-kits) trong một ứng dụng Laravel mới. Bộ công cụ khởi tạo sẽ đảm nhiệm việc xây dựng toàn bộ hệ thống xác thực của bạn, bao gồm cả việc xác minh email.

<a name="model-preparation"></a>
### Chuẩn bị Model

Trước khi bắt đầu, hãy kiểm tra model `App\Models\User` của bạn đã implement contract `Illuminate\Contracts\Auth\MustVerifyEmail`:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

Khi interface này đã được thêm vào model của bạn, thì những người dùng đăng ký mới sẽ được tự động gửi một email có chứa link xác minh email. Như bạn có thể thấy hãy kiểm tra `App\Providers\EventServiceProvider` của application của bạn, Laravel đã chứa sẵn một [listener](/docs/{{version}}/events) `SendEmailVerificationNotification` gắn với một event `Illuminate\Auth\Events\Registered`. Event listener này sẽ gửi link xác minh email cho người dùng.

Nếu bạn đang tự làm form đăng ký trong ứng dụng của bạn mà không sử dụng [bộ khởi tạo](/docs/{{version}}/starter-kits), thì bạn nên đảm bảo là bạn đang gửi event `Illuminate\Auth\Events\Registered` sau khi đăng ký người dùng thành công:

    use Illuminate\Auth\Events\Registered;

    event(new Registered($user));

<a name="database-preparation"></a>
### Chuẩn bị cơ sở dữ liệu

Tiếp theo, bảng `users` của bạn phải chứa cột `email_verified_at` để lưu ngày giờ mà địa chỉ email của người dùng được xác minh. Mặc định, migration của bảng `user` tồn tại trong framework Laravel đã chứa cột này. Vì vậy, tất cả những gì bạn cần làm là chạy migration cơ sở dữ liệu của bạn:

```shell
php artisan migrate
```

<a name="verification-routing"></a>
## Routing

Để thực hiện việc xác minh email đúng cách, bạn cần định nghĩa ba route. Đầu tiên, sẽ cần một route để hiển thị thông báo cho người dùng biết rằng là họ nên nhấn vào link xác minh email trong email mà Laravel đã gửi cho họ sau khi đăng ký thành công.

Thứ hai, bạn sẽ cần một route để xử lý các request được tạo khi người dùng nhấn vào link xác minh email trong email của họ.

Thứ ba, bạn sẽ cần có một route để gửi lại link xác minh nếu người dùng vô tình làm mất link xác minh đầu tiên.

<a name="the-email-verification-notice"></a>
### Thông báo xác minh email

Như đã đề cập trước đó, bạn cần định nghĩa một route sẽ trả về view hướng dẫn người dùng nhấn vào link xác minh email đã được Laravel gửi qua email cho họ sau khi đăng ký thành công. View này sẽ được hiển thị cho người dùng khi họ cố gắng truy cập các phần khác của ứng dụng mà chưa xác minh địa chỉ email của họ. Hãy nhớ rằng, link sẽ tự động được gửi qua email cho người dùng miễn là model `App\Models\User` của bạn implement interface `MustVerifyEmail`:

    Route::get('/email/verify', function () {
        return view('auth.verify-email');
    })->middleware('auth')->name('verification.notice');

Route trả về thông báo xác minh email phải được đặt tên là `verification.notice`. Điều quan trọng là route này phải được đặt tên chính xác này vì middleware `verified` [có trong Laravel](#protecting-routes) sẽ tự động chuyển hướng đến tên route này nếu người dùng chưa xác minh địa chỉ email của họ.

> **Note**
> Khi bạn tự làm chức năng xác minh email, bạn phải tự định nghĩa nội dung của view của trang thông báo xác minh. Nếu bạn muốn scaffolding bao gồm tất cả các view xác thực và xác minh cần thiết, hãy xem [bộ công cụ khởi tạo ứng dụng Laravel](/docs/{{version}}/starter-kits).

<a name="the-email-verification-handler"></a>
### Xử lý xác minh email

Tiếp theo, chúng ta cần định nghĩa một route sẽ xử lý các request được tạo khi người dùng nhấn vào link xác minh email đã được gửi qua email cho họ. Route này phải được đặt tên là `verification.verify` và được gán với middlewares `auth` và `signed`:

    use Illuminate\Foundation\Auth\EmailVerificationRequest;

    Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
        $request->fulfill();

        return redirect('/home');
    })->middleware(['auth', 'signed'])->name('verification.verify');

Trước khi tiếp tục, chúng ta hãy xem xét kỹ hơn về route này. Trước tiên, bạn sẽ nhận thấy rằng chúng tôi đang sử dụng loại request là `EmailVerificationRequest` thay vì instance `Illuminate\Http\Request`. `EmailVerificationRequest` là một [form request](/docs/{{version}}/validation#form-request-validation) có sẵn trong Laravel. Request này sẽ tự động đảm nhiệm việc xác thực các tham số `id` và `hash` của request.

Tiếp theo, chúng ta có thể tiến hành gọi trực tiếp phương thức `fulfill` trên instance request. Phương thức này sẽ gọi phương thức `markEmailAsVerified` trên người dùng đang được xác thực và gửi event `Illuminate\Auth\Events\Verified`. Mặc định, phương thức `markEmailAsVerified` có sẵn cho model `App\Models\User` thông qua class base `Illuminate\Foundation\Auth\User`. Khi địa chỉ email của người dùng đã được xác minh, bạn có thể chuyển hướng họ đến bất cứ đâu mà bạn mong muốn.

<a name="resending-the-verification-email"></a>
### Gửi lại email xác minh

Thỉnh thoảng người dùng có thể nhấn nhầm chỗ hoặc vô tình xóa email xác minh địa chỉ email. Để giải quyết vấn đề này, bạn có thể muốn định nghĩa một route cho phép người dùng yêu cầu gửi lại email xác minh. Sau đó, bạn có thể tạo request này bằng cách tạo một nút gửi form đơn giản ở trong [view thông báo xác minh](#the-email-verification-notice):

    use Illuminate\Http\Request;

    Route::post('/email/verification-notification', function (Request $request) {
        $request->user()->sendEmailVerificationNotification();

        return back()->with('message', 'Verification link sent!');
    })->middleware(['auth', 'throttle:6,1'])->name('verification.send');

<a name="protecting-routes"></a>
### Bảo vệ Route

[Route middleware](/docs/{{version}}/middleware) có thể được sử dụng để chỉ cho phép những người dùng mà đã xác minh được truy cập vào một route nhất định. Laravel cung cấp một middleware `verified` sẽ tham chiếu đến class `Illuminate\Auth\Middleware\EnsureEmailIsVerified`. Vì middleware này đã được đăng ký trong HTTP kernel của ứng dụng của bạn, nên tất cả những gì bạn cần là gắn middleware này vào một định nghĩa route. Thông thường, middleware này sẽ được gắn với middleware `auth`:

    Route::get('/profile', function () {
        // Only verified users may access this route...
    })->middleware(['auth', 'verified']);

Nếu người dùng chưa được xác minh email mà cố gắng truy cập vào route đã được chỉ định middleware này, họ sẽ tự động bị chuyển hướng đến [route có tên](/docs/{{version}}/routing#named-routes) `verification.notice`.

<a name="customization"></a>
## Customization

<a name="verification-email-customization"></a>
#### Verification Email Customization

Mặc dù thông báo xác minh email mặc định phải đáp ứng tất cản các yêu cầu của hầu hết các ứng dụng, nhưng Laravel cho phép bạn tùy chỉnh cách tạo email xác minh.

Để bắt đầu, hãy truyền một closure tới phương thức `toMailUsing` được cung cấp bởi notification `Illuminate\Auth\Notifications\VerifyEmail`. Closure sẽ nhận vào instance model notifiable đang nhận notification và một link email verification mà người dùng phải truy cập để xác minh địa chỉ email của họ. Closure sẽ trả về một instance của `Illuminate\Notifications\Messages\MailMessage`. Thông thường, bạn nên gọi phương thức `toMailUsing` từ phương thức `boot` của class `App\Providers\AuthServiceProvider` trong ứng dụng của bạn:

    use Illuminate\Auth\Notifications\VerifyEmail;
    use Illuminate\Notifications\Messages\MailMessage;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        // ...

        VerifyEmail::toMailUsing(function ($notifiable, $url) {
            return (new MailMessage)
                ->subject('Verify Email Address')
                ->line('Click the button below to verify your email address.')
                ->action('Verify Email Address', $url);
        });
    }

> **Note**
> Để tìm hiểu thêm về mail notification, vui lòng tham khảo thêm [tài liệu mail notification](/docs/{{version}}/notifications#mail-notifications).

<a name="events"></a>
## Event

Khi sử dụng [bộ khởi tạo ứng dụng của Laravel](/docs/{{version}}/starter-kits), Laravel sẽ gửi các [events](/docs/{{version}}/events) trong quá trình xác nhận email. Nếu bạn đang tự xử lý việc xác minh email cho ứng dụng của bạn, bạn có thể gửi các sự kiện này theo cách thủ công sau khi quá trình xác minh hoàn tất. Bạn có thể gắn listener vào các event này trong `EventServiceProvider` của application của bạn:

    use App\Listeners\LogVerifiedUser;
    use Illuminate\Auth\Events\Verified;

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        Verified::class => [
            LogVerifiedUser::class,
        ],
    ];
