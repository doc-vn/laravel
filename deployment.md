# Deployment

- [Giới thiệu](#introduction)
- [Yêu cầu server](#server-requirements)
- [Cấu hình server](#server-configuration)
    - [Nginx](#nginx)
    - [FrankenPHP](#frankenphp)
    - [Quyền thư mục](#directory-permissions)
- [Tối ưu](#optimization)
    - [Lưu cache file config](#optimizing-configuration-loading)
    - [Lưu cache event](#caching-events)
    - [Lưu cache route](#optimizing-route-loading)
    - [Lưu cache view](#optimizing-view-loading)
- [Reloading Services](#reloading-services)
- [Chế độ debug](#debug-mode)
- [Route trạng thái](#the-health-route)
- [Triển khai với Laravel Cloud và Forge](#deploying-with-cloud-or-forge)

<a name="introduction"></a>
## Giới thiệu

Khi bạn đã sẵn sàng deploy application Laravel của bạn vào production, có một số điều quan trọng mà bạn có thể làm để đảm bảo application của bạn chạy hiệu quả nhất có thể. Trong tài liệu này, chúng tôi sẽ đề cập đến một số điểm khởi đầu tuyệt vời để đảm bảo application Laravel của bạn được deploy đúng cách.

<a name="server-requirements"></a>
## Yêu cầu server

Laravel framework có một số yêu cầu về hệ thống. Bạn nên đảm bảo rằng server web của bạn có phiên bản PHP tối thiểu và các extension sau:

<div class="content-list" markdown="1">

- PHP >= 8.3
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

</div>

<a name="server-configuration"></a>
## Cấu hình server

<a name="nginx"></a>
### Nginx

Nếu bạn đang deploy application của bạn đến một server đang chạy Nginx, bạn có thể sử dụng file cấu hình sau đây để làm điểm bắt đầu cho cấu hình web server của bạn. Nhiều khả năng, file này sẽ cần được tùy chỉnh tùy thuộc vào cấu hình server của bạn. **Nếu bạn muốn được hỗ trợ trong việc quản lý server của bạn, hãy xem xét sử dụng một nền tảng được quản lý hoàn toàn như [Laravel Cloud](https://cloud.laravel.com).**

Hãy đảm bảo, giống như cấu hình bên dưới, server web của bạn sẽ hướng tất cả các request đến file `public/index.php` của ứng dụng của bạn. Bạn đừng bao giờ cố gắng di chuyển file `index.php` đến thư mục gốc của dự án của bạn, vì việc phân phát request của ứng dụng từ thư mục gốc của dự án sẽ làm lộ nhiều file cấu hình nhạy cảm lên Internet:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev/) cũng có thể được sử dụng để chạy các ứng dụng Laravel của bạn. FrankenPHP là một server ứng dụng PHP hiện đại được viết bằng ngôn ngữ Go. Để chạy một ứng dụng Laravel PHP bằng FrankenPHP, bạn chỉ cần gọi lệnh `php-server` của nó:

```shell
frankenphp php-server -r public/
```

Để tận dụng các tính năng mạnh mẽ hơn được FrankenPHP hỗ trợ, chẳng hạn như tích hợp [Laravel Octane](/docs/{{version}}/octane), HTTP/3, modern compression hoặc khả năng đóng gói các ứng dụng Laravel dưới dạng các file nhị phân độc lập, vui lòng tham khảo [tài liệu Laravel](https://frankenphp.dev/docs/laravel/) của FrankenPHP.

<a name="directory-permissions"></a>
### Quyền thư mục

Laravel sẽ cần ghi vào các thư mục `bootstrap/cache` và `storage`, vì vậy bạn phải đảm bảo process server web có quyền ghi vào các thư mục này.

<a name="optimization"></a>
## Tối ưu

Khi triển khai ứng dụng lên môi trường production, có nhiều file cần được lưu cache, như cấu hình, event, route và view. Laravel cung cấp một lệnh Artisan `optimize` tiện lợi, cho phép bạn lưu cache tất cả các file này. Lệnh này thường được gọi như một phần của quy trình deploy ứng dụng:

```shell
php artisan optimize
```

Phương thức `optimize:clear` có thể được sử dụng để xóa tất cả các file bộ nhớ cache được tạo bởi lệnh `optimize` cũng như tất cả các khóa trong driver bộ nhớ cache mặc định:

```shell
php artisan optimize:clear
```

Trong tài liệu sau đây, chúng ta sẽ thảo luận về từng lệnh tối ưu hóa chi tiết được thực hiện bởi lệnh `optimize`.

<a name="optimizing-configuration-loading"></a>
### Lưu cache file config

Khi deploy application vào production, bạn cũng nên đảm bảo là bạn đã chạy lệnh Artisan `config:cache` trong quá trình deploy:

```shell
php artisan config:cache
```

Lệnh này sẽ nối tất cả các file config của Laravel thành một file và được lưu vào trong bộ nhớ cache, giúp giảm đáng kể số lượng trao đổi giữa framework với filesystem khi tải các value config của bạn.

> [!WARNING]
> Nếu bạn chạy lệnh `config:cache` trong quá trình deploy, bạn nên đảm bảo là bạn chỉ gọi hàm `env` từ trong các file cấu hình của bạn. Khi các file cấu hình đã được lưu vào trong bộ nhớ cache, thì file `.env` sẽ không được load và tất cả các code gọi đến hàm `env` để lấy biến trong file `.env` ra sẽ đều trả về `null`.

<a name="caching-events"></a>
### Lưu cache event

Bạn nên lưu cache auto-discovered event của ứng dụng vào các mapping listener trong quá trình deploy. Điều này có thể thực hiện được bằng cách gọi lệnh Artisan `event:cache` trong quá trình deploy:

```shell
php artisan event:cache
```

<a name="optimizing-route-loading"></a>
### Lưu cache route

Nếu bạn đang build một application lớn với nhiều route, bạn nên đảm bảo rằng bạn đã chạy lệnh Artisan `route:cache` trong quá trình deploy của bạn:

```shell
php artisan route:cache
```

Lệnh này sẽ giảm tất cả các đăng ký route của bạn vào trong một phương thức duy nhất và lưu trong một file ở cache, nó giúp cải thiện hiệu suất của việc đăng ký route khi đăng ký hàng trăm route.

<a name="optimizing-view-loading"></a>
### Lưu cache view

Khi deploy ứng dụng của bạn vào production, bạn nên đảm bảo rằng bạn đã chạy lệnh Artisan `view:cache` trong quá trình deploy của bạn:

```shell
php artisan view:cache
```

Lệnh này biên dịch tất cả các view Blade của bạn để chúng không cần phải biên dịch mỗi khi có request đến, cải thiện hiệu suất cho mỗi request trả về một view.

<a name="reloading-services"></a>
## Reloading Services

> [!NOTE]
Khi deploy lên [Laravel Cloud](https://cloud.laravel.com), bạn không cần phải sử dụng lệnh `reload`, vì việc reload lại tất cả các service sẽ được xử lý tự động.

Sau khi deploy một version mới của ứng dụng, bất kỳ service nào đang chạy lâu như queue worker, Laravel Reverb, hoặc Laravel Octane đều nên được reload hoặc restart để sử dụng code mới. Laravel cung cấp một lệnh Artisan `reload` duy nhất sẽ dừng các service này:

```shell
php artisan reload
```

Nếu bạn không sử dụng [Laravel Cloud](https://cloud.laravel.com), bạn nên tự cấu hình một process monitor để có thể phát hiện khi các process của bạn bị lỗi và tự động khởi động lại chúng.

<a name="debug-mode"></a>
## Chế độ debug

Tùy chọn debug trong file cấu hình `config/app.php` của bạn sẽ xác định lượng thông tin lỗi sẽ thực sự được hiển thị cho người dùng. Mặc định, tùy chọn này được set để ưu tiên giá trị của biến môi trường `APP_DEBUG`, được lưu trong file .env trong application của bạn.

> [!WARNING]
> **Trong môi trường production của bạn, giá trị này phải luôn là `false`. Nếu biến `APP_DEBUG` được set thành `true` trong quá trình production, bạn có nguy cơ bị lộ các giá trị cấu hình nhạy cảm cho người dùng ứng dụng của bạn.**

<a name="the-health-route"></a>
## Route trạng thái

Laravel có sẵn một route kiểm tra trạng thái có thể được sử dụng để theo dõi trạng thái ứng dụng của bạn. Trong môi trường production, route này có thể được sử dụng để báo cáo trạng thái ứng dụng cho hệ thống giám sát thời gian hoạt động, bộ cân bằng tải hoặc hệ thống điều phối như Kubernetes.

Mặc định, route kiểm tra trạng thái được chạy tại `/up` và sẽ trả về response HTTP 200 nếu ứng dụng đã khởi động mà không có bất kỳ ngoại lệ nào. Nếu không, response HTTP 500 sẽ được trả về. Bạn có thể cấu hình URI cho route này trong file `bootstrap/app` của ứng dụng:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up', // [tl! remove]
    health: '/status', // [tl! add]
)
```

Khi các request HTTP được gửi đến route này, Laravel cũng sẽ gửi một event `Illuminate\Foundation\Events\DiagnosingHealth`, cho phép bạn thực hiện các kiểm tra trạng thái bổ sung liên quan đến ứng dụng. Trong [listener](/docs/{{version}}/events) của event này, bạn có thể kiểm tra trạng thái cơ sở dữ liệu hoặc bộ nhớ cache của ứng dụng. Nếu phát hiện sự cố với ứng dụng, bạn chỉ cần đưa ra một ngoại lệ từ listener.

<a name="deploying-with-cloud-or-forge"></a>
## Deploying With Laravel Cloud or Forge

<a name="laravel-cloud"></a>
#### Laravel Cloud

Nếu bạn muốn một nền tảng deploy được quản lý hoàn toàn, tự động mở rộng và được tinh chỉnh cho Laravel, hãy tham khảo [Laravel Cloud](https://cloud.laravel.com). Laravel Cloud là một nền tảng deploy mạnh mẽ dành cho Laravel, cung cấp và quản lý các dịch vụ compute, cơ sở dữ liệu, bộ nhớ cache và lưu trữ object.

Hãy chạy ứng dụng Laravel của bạn trên Cloud và trải nghiệm sự đơn giản trong việc mở rộng. Laravel Cloud được tinh chỉnh bởi chính những người tạo ra Laravel để hoạt động mượt mà với framework, giúp bạn có thể tiếp tục phát triển các ứng dụng Laravel theo đúng cách mà bạn vẫn thường làm.

<a name="laravel-forge"></a>
#### Laravel Forge

Nếu bạn muốn tự quản lý server của bạn nhưng không thoải mái với việc cấu hình các dịch vụ khác nhau cần thiết để chạy ứng dụng Laravel, [Laravel Forge](https://forge.laravel.com) là một nền tảng quản lý server VPS dành cho các ứng dụng Laravel.

Laravel Forge có thể tạo server trên các nhà cung cấp khác nhau như DigitalOcean, Linode, AWS, v.v. Ngoài ra, Forge có thể cài đặt và quản lý tất cả các công cụ cần thiết để xây dựng các ứng dụng Laravel, như Nginx, MySQL, Redis, Memcached, Beanstalk,...
