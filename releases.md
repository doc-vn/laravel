# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 5.7](#laravel-5.7)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Cấu trúc của phiên bản của Laravel được duy trì theo quy ước như sau: `paradigm.major.minor`. Các phiên bản được phát hành chính thức của framework được phát hành sáu tháng một lần (tháng 2 và tháng 8), trong khi các bản phát hành nhỏ hơn có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `5.7.*`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

Các bản phát hành thay đổi Paradigm được phân tách qua nhiều năm và đại diện cho những thay đổi căn bản trong kiến trúc và quy ước của framework. Hiện tại, chưa có bản phát hành nào mà cần phải thay đổi Paradigm được phát triển trong hiện tại.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với các bản phát hành hỗ trợ dài hạn, chẳng hạn như Laravel 5.5, các bản sửa lỗi được cung cấp trong 2 năm và các bản sửa lỗi bảo mật được cung cấp trong 3 năm. Những bản phát hành này cung cấp các hỗ trợ và bảo trì dài nhất. Đối với các bản phát hành bình thường, các bản sửa lỗi được cung cấp trong 6 tháng và các bản sửa lỗi bảo mật được cung cấp trong 1 năm. Đối với tất cả các thư viện, bao gồm cả Lumen, chỉ bản phát hành mới nhất mới nhận được các bản sửa lỗi.

| Version | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- |
| 5.0 | ngày 4 tháng 2 năm 2015 | ngày 4 tháng 8 năm 2015 | ngày 4 tháng 2 năm 2016 |
| 5.1 (LTS) | ngày 9 tháng 6 năm 2015 | ngày 9 tháng 6 năm 2017 | ngày 9 tháng 6 năm 2018 |
| 5.2 | ngày 21 tháng 12 năm 2015 | ngày 6 tháng 12 năm 2016 | ngày 21 tháng 12 năm 2016 |
| 5.3 | ngày 23 tháng 8 năm 2016 | ngày 23 tháng 2 năm 2016 | ngày 23 tháng 8 năm 2017 |
| 5.4 | ngày 24 tháng 1 năm 2017 | ngày 24 tháng 7 năm 2017| ngày 24 tháng 1 năm 2018 |
| 5.5 (LTS) | ngày 30 tháng 8 năm 2017 | ngày 30 tháng 8 năm 2019 | ngày 30 tháng 8 năm 2020 |
| 5.6 | ngày 7 tháng 2 năm 2018 | ngày 7 tháng 8 năm 2018 | ngày 7 tháng 2 năm 2019 |
| 5.7 | ngày 4 tháng 9 năm 2018 | ngày 4 tháng 3 năm 2019 | ngày 4 tháng 9 năm 2019 |

<a name="laravel-5.7"></a>
## Laravel 5.7

Laravel 5.7 sẽ tiếp tục những cải tiến mà được thực hiện trong Laravel 5.6 bằng cách giới thiệu [Laravel Nova](https://nova.laravel.com) với tùy chọn xác minh email cho authentication scaffolding, hỗ trợ guest user trong authorization gate và policy, cải tiến việc test trên console, tích hợp Symfony `dump-server`, hỗ trợ ngôn ngữ trong thông báo và một hàng loạt các bản sửa lỗi và cải tiến sử dụng khác.

### Laravel Nova

[Laravel Nova](https://nova.laravel.com) là một bảng điều khiển quản trị tốt và đẹp nhất dành cho các ứng dụng Laravel. Tính năng chính của Nova là khả năng quản lý các bản ghi cơ sở dữ liệu của bạn bằng Eloquent. Ngoài ra, Nova cung cấp các hỗ trợ cho các bộ lọc, tìm kiếm chuyên sâu, hành động, queue action, số liệu, authorization, custom tool, custom card, custom field, và nhiều hơn thế nữa.

Để tìm hiểu thêm về Laravel Nova, hãy xem [Nova website](https://nova.laravel.com).

### Email Verification

Laravel 5.7 giới thiệu thêm tùy chọn xác minh email vào authentication scaffolding trong framework. Để phù hợp với tính năng này, cột timestamp `email_verified_at` đã được thêm vào migration của bảng `users` mặc định trong framework.

Để nhắc người dùng mới phải đăng ký xác minh email của họ, model `User` phải được implement bằng interface `MustVerifyEmail`:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        // ...
    }

Sau khi model `User` đã được implement bằng interface `MustVerifyEmail`, người dùng mới sẽ nhận được một email chứa link xác minh. Khi link này đã được nhấp vào, Laravel sẽ tự động ghi lại thời gian xác minh của người dùng đó vào trong cơ sở dữ liệu và chuyển hướng người dùng đến vị trí bạn chọn.

A `verified` middleware has been added to the default application's HTTP kernel. This middleware may be attached to routes that should only allow verified users:
Mặc định, một middleware `verified` đã được thêm vào file HTTP kernel của ứng dụng. Middleware này có thể được gắn với các route chỉ cho phép những người dùng đã được xác minh mới có thể được đi qua:

    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,

> {tip} Để tìm hiểu thêm về xác minh email, hãy xem [tài liệu đầy đủ hơn về nó](/docs/{{version}}/verification).

### Guest User Gates / Policies

Trong các phiên bản trước của Laravel, các authorization gate và policy sẽ tự động trả về giá trị `false` cho những người chưa được authenticate vào ứng dụng của bạn. Tuy nhiên, bây giờ bạn có thể cho phép những người dùng đó pass qua kiểm tra authorization này bằng cách khai báo "optional" hoặc cung cấp giá trị mặc định là `null` cho tham số người dùng:

    Gate::define('update-post', function (?User $user, Post $post) {
        // ...
    });

### Symfony Dump Server

Laravel 5.7 cung cấp khả năng tích hợp với lệnh `dump-server` của Symfony thông qua [một package bởi Marcel Pociot](https://github.com/beyondcode/laravel-dump-server). Để bắt đầu, hãy chạy lệnh Artisan `dump-server`:

    php artisan dump-server

Khi máy chủ đã được khởi động, tất cả các lệnh gọi đến hàm `dump` sẽ được hiển thị trong cửa sổ console `dump-server` thay vì trong trình duyệt của bạn, cho phép bạn kiểm tra các giá trị mà không làm ảnh hưởng đến HTTP response output của bạn.

### Notification Localization

Laravel bây giờ đã cho phép bạn gửi các thông báo bằng những ngôn ngữ khác, khác với ngôn ngữ mặc định của application và thậm chí sẽ ghi nhớ ngôn ngữ này nếu thông báo được queue.

Để thực hiện điều này, class `Illuminate\Notifications\Notification` đã cung cấp một phương thức `locale` để set ngôn ngữ mà bạn mong muốn. Ứng dụng của bạn sẽ thay đổi thành ngôn ngữ này khi thông báo đang được định dạng và sau đó hoàn trả về ngôn ngữ trước đó khi quá trình định dạng hoàn tất:

    $user->notify((new InvoicePaid($invoice))->locale('es'));

Thay đổi ngôn ngữ cho nhiều thông báo cùng một lúc cũng có thể đạt được thông qua facade `Notification`:

    Notification::locale('es')->send($users, new InvoicePaid($invoice));

### Console Testing

Laravel 5.7 cho phép bạn dễ dàng "mô phỏng" một input đầu vào của người dùng cho các lệnh trên console của bạn bằng phương thức `expectsQuestion`. Ngoài ra, bạn có thể chỉ định exit code và text mà bạn muốn output cho lệnh console bằng cách sử dụng phương thức `assertExitCode` và `expectsOutput`. Ví dụ: hãy xem xét lệnh console sau:

    Artisan::command('question', function () {
        $name = $this->ask('What is your name?');

        $language = $this->choice('Which language do you program in?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Your name is '.$name.' and you program in '.$language.'.');
    });

Bạn có thể kiểm tra lệnh này bằng bài test sau sử dụng các phương thức `expectsQuestion`, `expectsOutput`, và `assertExitCode`:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function test_console_command()
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you program in?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you program in PHP.')
             ->assertExitCode(0);
    }

### URL Generator & Callable Syntax

Thay vì chỉ chấp nhận các chuỗi string, trình tạo URL của Laravel bây giờ đã chấp nhận cú pháp "callable" khi tạo URL cho các action của controller:

    action([UserController::class, 'index']);

### Paginator Links

Laravel 5.7 cho phép bạn kiểm soát số lượng link sẽ được hiển thị ở mỗi bên của "window" URL của paginator. Theo mặc định, ba link sẽ được hiển thị ở mỗi bên của paginator chính. Tuy nhiên, bạn có thể kiểm soát số này bằng phương thức `onEachSide`:

    {{ $paginator->onEachSide(5)->links() }}

### Filesystem Read / Write Streams

Tích hợp Flysystem của Laravel bây giờ cung cấp các phương thức `readStream` và `writeStream`:

    Storage::disk('s3')->writeStream(
        'remote-file.zip',
        Storage::disk('local')->readStream('local-file.zip')
    );
