# Cài đặt

- [Cài đặt](#installation)
    - [Yêu cầu server](#server-requirements)
    - [Cài đặt Laravel](#installing-laravel)
    - [Cấu hình](#configuration)
- [Cấu hình Web Server](#web-server-configuration)
    - [Tạo URLs](#pretty-urls)

<a name="installation"></a>
## Cài đặt

> {video} Laracasts cung cấp một [dịch vụ miễn phí giới thiệu Laravel](http://laravelfromscratch.com) cho người mới bắt đầu. Nó là một nơi tốt để bạn có thể bắt đầu.

<a name="server-requirements"></a>
### Yêu cầu server

Laravel framework có nhiều yêu cầu về server. Tất cả những yêu cầu đó đều đã được máy ảo [Laravel Homestead](/docs/{{version}}/homestead) cung cấp, vì vậy chúng tôi rất khuyến khích bạn dùng Homestead làm nơi phát triển ở local.

Tuy nhiên, nếu bạn không muốn dùng Homestead, thì bạn hãy chắc chắn là server của bạn đã cài đặt những package dưới đây:

<div class="content-list" markdown="1">
- PHP >= 7.1.3
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
- Ctype PHP Extension
- JSON PHP Extension
- BCMath PHP Extension
</div>

<a name="installing-laravel"></a>
### Cài đặt Laravel

Laravel dùng [Composer](https://getcomposer.org) để quản lý các library. Nên trước khi dùng Laravel, bạn cần chắc chắn là đã cài đặt Composer vào trong máy của bạn.

#### Thông qua Laravel Installer

Đầu tiên, hãy download Laravel Installer bằng câu lệnh composer dưới đây:

    composer global require laravel/installer

Hãy chắc chắn rằng laravel installer đã được cài đặt vào trong thư mục global của composer, để bạn có thể chạy lệnh `laravel` này tại bất kỳ thư mục nào mà bạn muốn tạo project. Thư mục global của composer này sẽ tồn tại ở các vị trí khác nhau tuỳ theo hệ điều hành của bạn, nhưng dưới đây là một số vị trí cơ bản theo hệ điều hành:

<div class="content-list" markdown="1">
- macOS: `$HOME/.composer/vendor/bin`
- GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin`
</div>

Sau khi đã cài đặt xong, lệnh `laravel new` sẽ tạo một project mới tại đúng vị trí thự mục mà bạn đang chạy lệnh này, Ví dụ, khi chạy lệnh `laravel new blog` sẽ tạo a một thư mục mới với tên là `blog` mà trong đó đã cài đặt tất cả cái thứ mà laravel cần để chạy:

    laravel new blog

#### Thông qua Composer Create-Project

Hoặc, bạn cũng có thể cài đặt laravel bằng cách chạy lệnh `create-project` trong terminal của bạn:

    composer create-project --prefer-dist laravel/laravel blog "5.7.*"

#### Local Development Server

Nếu bạn đã cài đặt PHP trong local của bạn, bạn muốn dùng lệnh PHP's built-in để tạo server cho web application của bạn, bạn có thể dùng lệnh Artisan `serve`. Lệnh này sẽ tạo một server cho web application của bạn ở `http://localhost:8000`:

    php artisan serve

Bạn sẽ có nhiều lựa chọn hơn thông qua [Homestead](/docs/{{version}}/homestead) và [Valet](/docs/{{version}}/valet).

<a name="configuration"></a>
### Cấu hình

#### Thư mục public

Sau khi cài đặt Laravel, bạn cần cấu hình thư mục gốc của web trỏ vào thư mục `public`. Và file `index.php` trong thư mục này sẽ được gọi khi các request gửi đến web application của bạn.

#### Các file cấu hình

Tất cả các file cấu hình cho Laravel framework sẽ được lưu trữ tại thư mục `config`. Các cấu hình này đều đã được tài liệu hoá bằng comment, vì vậy hãy xem qua và làm quen với chúng.

#### Quyền hạn của thư mục

Sau khi cài đặt Laravel, bạn có thể cần cài đặt thêm một số quyền. Ví dụ, thư mục `storage` và thư mục `bootstrap/cache` sẽ cần quyền writable cho web application của bạn, nếu không có các quyền này Laravel sẽ không thể chạy, Và nếu bạn đang dùng máy ảo [Homestead](/docs/{{version}}/homestead), thì các quyền này đều đã được cài đặt sẵn.

#### Application Key

Tiếp theo, bạn cũng cần làm một việc sau khi đã cài đặt xong Laravel, đó là cài đặt một chuỗi random để làm application key. Nếu bạn cài đặt Laravel bằng Composer hoặc Laravel installer, thì application key có thể được tạo ra bằng cách chạy lệnh `php artisan key:generate`.

Bình thường, application key sẽ có chiều dài là 32 ký tự. Và được lưu ở trong file cài đặt môi trường `.env`, nếu bạn chưa đổi tên file `.env.example` sang `.env`, thì bạn nên làm nó ngày bây giờ. **Nếu như application key của bạn không được cài đặt, thì session của người dùng và các mã hoá data sẽ không an toàn**

#### Cấu hình thêm

Laravel gần như không yêu cần bạn cấu hình thêm. Bạn có thể thoải mái bắt đầu phát triển! Tuy nhiên, bạn cũng có thể muốn xem qua file `config/app.php` và tài liệu của nó. Nó chứa nhiều lựa chọn về `timezone` và `locale`, những lựa chọn mà có thể bạn muốn thay đổi trong application của bạn.

Bạn cũng có thể muốn cấu hình thêm nhiều phần khác của laravel, như là:

<div class="content-list" markdown="1">
- [Cache](/docs/{{version}}/cache#configuration)
- [Database](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)
</div>

<a name="web-server-configuration"></a>
## Cấu hình Web Server

<a name="pretty-urls"></a>
### Tạo URLs

#### Apache

Laravel có sẵn một file `public/.htaccess`, file này sẽ được dùng để điều khiển URLs trước khi chạy file `index.php`. Khi chạy Laravel với Apache, hãy chắc chắn rằng là bạn đã enable module `mod_rewrite` để file `.htaccess` này sẽ được máy chủ chấp nhận.

Nếu file `.htaccess` đi cùng với Laravel không hoạt động với Apache bạn đã cài, thì bạn hãy thử dòng lệnh dưới đây:

    Options +FollowSymLinks -Indexes
    RewriteEngine On

    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

Nếu bạn đang sử dụng Nginx, lệnh dưới đây nằm trong file config của webstie sẽ chuyển hướng tất cả các request đến file `index.php` của bạn.

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

Khi bạn dùng [Homestead](/docs/{{version}}/homestead) hoặc [Valet](/docs/{{version}}/valet), URLs sẽ được tự động cài đặt.
