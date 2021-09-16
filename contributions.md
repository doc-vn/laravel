# Contribution Guide

- [Báo cáo Bug](#bug-reports)
- [Các kệnh phát triển chính](#core-development-discussion)
- [Branch nào?](#which-branch)
- [Lỗ hổng bảo mật](#security-vulnerabilities)
- [Coding Style](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## Báo cáo Bug

Để khuyến khích sự phát triển tích cực, Laravel rất khuyến khích tạo các pull request, không chỉ là các bug. "Báo cáo bug" cũng có thể là một pull request được gửi dưới dạng chứa một bài test thất bại.

Tuy nhiên, nếu bạn muốn tạo một bug, thì bug của bạn nên chứa một tiêu đề và một mô tả rõ ràng về bug của bạn. Bạn cũng nên mô tả càng nhiều thông tin liên quan càng tốt và một code ví dụ để tạo ra bug đó. Mục tiêu của báo cáo bug là giúp bạn dễ dàng - và những người khác - tái hiện bug và phát triển các bản sửa bug.

Hãy nhớ rằng, các báo cáo bug được tạo ra với hy vọng rằng những người khác có cùng vấn đề với bạn có thể cộng tác với nhau để cùng nhau sửa bug. Bạn đừng hy vọng rằng báo cáo bug sẽ là khởi động một quá trình sửa bug nào đó hoặc người khác sẽ nhảy vào để sửa. Tạo một báo cáo bug để giúp chính bạn và những người khác, bắt đầu một quá trình sửa bug mà bạn đã báo cáo.

Mã nguồn của Laravel được quản lý trên GitHub và có các repository cho từng dự án của Laravel:

<div class="content-list" markdown="1">
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Website](https://github.com/laravel/laravel.com)
</div>

<a name="core-development-discussion"></a>
## Các kệnh phát triển chính

Bạn có thể đề xuất các tính năng mới hoặc các cải tiến về các hành động của Laravel trong Laravel Internals [issue board](https://github.com/laravel/internals/issues). Nếu bạn đề xuất một tính năng mới, vui lòng sẵn sàng triển khai một số code cần thiết để hoàn thành tính năng này.

Kênh `#internals` của team Slack [LaraChat](https://larachat.co) sẽ thảo luận về các lỗi, tính năng mới và triển khai các tính năng hiện tại. Taylor Otwell, maintainer của Laravel, thường có mặt trong kênh này vào các ngày trong tuần từ 8 giờ sáng đến 5 giờ chiều (UTC-06:00 or America/Chicago) và xuất hiện thường xuyên trong kênh vào các thời điểm khác.

<a name="which-branch"></a>
## Branch nào?

**Tất cả** các bản sửa lỗi phải được gửi đến branch ổn định mới nhất hoặc tới các branch LTS và branch hiện tại là (5.5). Các bản sửa lỗi sẽ **không** được gửi đến branch `master` trừ khi chúng sửa các tính năng đã tồn tại trong bản phát hành sắp tới.

Các tính năng **phụ** có **tương thích** với bản phát hành Laravel hiện tại thì có thể được gửi đến branch ổn định mới nhất.

Các tính năng **chính** mới phải luôn được gửi đến branch `master`, nơi chứa bản phát hành Laravel sắp tới.

Nếu bạn không chắc chắn tính năng của bạn đủ điều kiện là chính hay phụ, vui lòng hỏi Taylor Otwell trong kênh `#internals` của nhóm Slack [LaraChat](https://larachat.co).

<a name="security-vulnerabilities"></a>
## Lỗ hổng bảo mật

Nếu bạn phát hiện ra lỗ hổng bảo mật trong Laravel, vui lòng gửi email đến Taylor Otwell theo địa chỉ <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. Tất cả các lỗ hổng bảo mật sẽ được giải quyết kịp thời.

<a name="coding-style"></a>
## Coding Style

Laravel tuân theo tiêu chuẩn coding [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) và tiêu chuẩn autoloading [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md).

<a name="phpdoc"></a>
### PHPDoc

Dưới đây là một ví dụ mẫu về Laravel documentation hợp lệ. Lưu ý rằng đằng sau thuộc tính `@param` là hai khoảng trắng, tiếp theo là kiểu của tham số, và hai khoảng trắng nữa và cuối cùng là tên biến:

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

Đừng lo lắng nếu code style của bạn không hoàn hảo! [StyleCI](https://styleci.io/) sẽ tự động merge các bản sửa style cho bạn vào repository của Laravel sau khi các pull request được merge. Điều này cho phép chúng ta tập trung vào nội dung đóng góp thay vì phải tập trung vào code style.
