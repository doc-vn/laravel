# Contribution Guide

- [Báo Bug](#bug-reports)
- [Hỗ trợ question](#support-questions)
- [Các kênh phát triển chính](#core-development-discussion)
- [Branch nào?](#which-branch)
- [Biên dịch Asset](#compiled-assets)
- [Lỗ hổng bảo mật](#security-vulnerabilities)
- [Coding Style](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)
- [Quy tắc ứng xử](#code-of-conduct)

<a name="bug-reports"></a>
## Báo Bug

Để khuyến khích cho sự phát triển, Laravel rất khuyến khích bạn tạo các pull request, không chỉ là báo bug. "Báo bug" cũng có thể là một pull request được gửi dưới dạng là một bài test thất bại.

Tuy nhiên, nếu bạn muốn tạo một bug, thì bug của bạn nên chứa một tiêu đề và một mô tả rõ ràng về bug mà bạn gặp phải. Bạn cũng nên mô tả càng nhiều thông tin liên quan đến bug càng tốt và một code ví dụ để tạo ra bug đó. Mục tiêu của báo bug là giúp bạn dễ dàng - và những người khác - tái hiện lại bug đó và phát triển các bản sửa bug.

Hãy nhớ rằng, báo bug được tạo ra với hy vọng rằng những người khác có cùng vấn đề với bạn có thể cộng tác với nhau để cùng nhau sửa bug. Bạn đừng hy vọng rằng báo bug sẽ làm khởi động một quá trình sửa bug nào đó hoặc người khác sẽ nhảy vào để sửa giúp bạn. Tạo một bug để giúp chính bạn và những người khác, bắt đầu một quá trình sửa bug mà bạn đã báo cáo. Nếu bạn muốn trợ giúp, bạn có thể trợ giúp bằng cách sửa [bất kỳ lỗi nào được liệt kê trong issue tracker của chúng tôi](https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel+-repo%3Alaravel%2Fnova-issues).

Mã nguồn của Laravel được quản lý trên GitHub và có các repository cho từng dự án của Laravel:

<div class="content-list" markdown="1">

- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Laravel Echo](https://github.com/laravel/echo)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Sanctum](https://github.com/laravel/sanctum)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravel Website](https://github.com/laravel/laravel.com-next)
- [Laravel UI](https://github.com/laravel/ui)

</div>

<a name="support-questions"></a>
## Hỗ trợ question

GitHub issue tracker của Laravel không nhằm mục đích cung cấp các trợ giúp hoặc hỗ trợ cho Laravel. Thay vào đó, hãy sử dụng một trong các kênh sau:

<div class="content-list" markdown="1">

- [GitHub Discussions](https://github.com/laravel/framework/discussions)
- [Laracasts Forums](https://laracasts.com/discuss)
- [Laravel.io Forums](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discordapp.com/invite/KxwQuKb)
- [Larachat](https://larachat.co)
- [IRC](https://webchat.freenode.net/?nick=artisan&channels=%23laravel&prompt=1)

</div>

<a name="core-development-discussion"></a>
## Các kênh phát triển chính

Bạn có thể đề xuất các tính năng mới hoặc các cải tiến về các hành động của Laravel trong Laravel Ideas [issue board](https://github.com/laravel/ideas/issues). Nếu bạn đề xuất một tính năng mới, vui lòng làm sẵn một số code cần thiết để hoàn thành tính năng này.

Kênh `#internals` của [Laravel Discord server](https://discordapp.com/invite/mPZNm7A) sẽ thảo luận về các lỗi, tính năng mới và triển khai các tính năng hiện tại. Taylor Otwell, maintainer của Laravel, thường có mặt trong kênh này vào các ngày trong tuần từ 8 giờ sáng đến 5 giờ chiều (UTC-06:00 or America/Chicago) và xuất hiện thường xuyên trong kênh vào các thời điểm khác.

<a name="which-branch"></a>
## Branch nào?

**Tất cả** các bản sửa lỗi phải được gửi đến branch ổn định mới nhất hoặc tới các [branch LTS hiện tại](/docs/{{version}}/releases#support-policy). Các bản sửa lỗi sẽ **không** được gửi đến branch `master` trừ khi chúng sửa các tính năng đã tồn tại trong bản phát hành sắp tới.

Các tính năng **phụ** có **tương thích** với bản phát hành hiện tại thì có thể được gửi đến branch ổn định mới nhất.

Các tính năng **chính** mới phải luôn được gửi đến branch `master`, nơi chứa code của các bản phát hành sắp tới.

Nếu bạn không chắc chắn tính năng của bạn là chính hay là phụ, vui lòng hỏi Taylor Otwell trong kênh `#internals` của [Laravel Discord server](https://discordapp.com/invite/mPZNm7A).

<a name="compiled-assets"></a>
## Biên dịch Asset

Nếu bạn đang gửi một thay đổi sẽ ảnh hưởng đến các file đã được biên dịch, chẳng hạn như các file ở trong `resources/sass` hoặc `resources/js` của repository `laravel/laravel`, thì đừng commit các file đã biên dịch trên. Bởi vì, trên thực tế, do kích thước của file đó quá lớn, nên chúng sẽ không thể được review bởi người quản lý. Và điều này cũng có thể bị khai thác như là một cách để đưa mã độc vào trong source code của Laravel. Để ngăn chặn điều này, tất cả các file đã biên dịch sẽ được tạo và commit bởi những người quản lý source Laravel.

<a name="security-vulnerabilities"></a>
## Lỗ hổng bảo mật

Nếu bạn phát hiện ra lỗ hổng bảo mật trong Laravel, vui lòng gửi email đến Taylor Otwell theo địa chỉ <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. Tất cả các lỗ hổng bảo mật sẽ được giải quyết kịp thời.

<a name="coding-style"></a>
## Coding Style

Laravel tuân theo tiêu chuẩn coding [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) và tiêu chuẩn autoloading [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md).

<a name="phpdoc"></a>
### PHPDoc

Dưới đây là một ví dụ mẫu về Laravel documentation hợp lệ. Lưu ý rằng đằng sau thuộc tính `@param` là hai khoảng trắng, tiếp theo là kiểu của tham số, và hai khoảng trắng và cuối cùng là tên biến:

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     *
     * @throws \Exception
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

Đừng lo lắng nếu code style của bạn không hoàn hảo! [StyleCI](https://styleci.io/) sẽ tự động merge các bản sửa style cho bạn vào repository của Laravel sau khi các pull request được merge. Điều này cho phép chúng ta tập trung vào nội dung đóng góp thay vì phải tập trung vào code style.

<a name="code-of-conduct"></a>
## Quy tắc ứng xử

Quy tắc ứng xử của Laravel có nguồn gốc từ quy tắc ứng xử của Ruby. Mọi vi phạm quy tắc ứng xử có thể được báo cáo cho Taylor Otwell (taylor@laravel.com):

<div class="content-list" markdown="1">

- Những người tham gia sẽ khoan dung với những quan điểm đối lập.
- Người tham gia phải đảm bảo rằng ngôn ngữ và hành động của họ không có tính công kích và nhận xét mang tính miệt thị cá nhân.
- Khi giải thích lời nói và hành động của người khác, người tham gia phải luôn có ý tốt.
- Hành vi được coi là quấy rối sẽ không được dung thứ.

</div>
