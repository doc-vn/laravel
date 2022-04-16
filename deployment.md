# Deployment

- [Giới thiệu](#introduction)
- [Cấu hình server](#server-configuration)
    - [Nginx](#nginx)
- [Tối ưu](#optimization)
    - [Tối ưu autoloader](#autoloader-optimization)
    - [Tối ưu load config](#optimizing-configuration-loading)
    - [Tối ưu load route](#optimizing-route-loading)
    - [Tối ưu View Loading](#optimizing-view-loading)
- [Deploy cùng Forge và Vapor](#deploying-with-forge-or-vapor)

<a name="introduction"></a>
## Giới thiệu

Khi bạn đã sẵn sàng deploy application Laravel của bạn vào production, có một số điều quan trọng mà bạn có thể làm để đảm bảo application của bạn chạy hiệu quả nhất có thể. Trong tài liệu này, chúng tôi sẽ đề cập đến một số điểm khởi đầu tuyệt vời để đảm bảo application Laravel của bạn được deploy đúng cách.

<a name="server-configuration"></a>
## Cấu hình server

<a name="nginx"></a>
### Nginx

Nếu bạn đang deploy application của bạn đến một server đang chạy Nginx, bạn có thể sử dụng file cấu hình sau đây để làm điểm bắt đầu cho cấu hình web server của bạn. Nhiều khả năng, file này sẽ cần được tùy chỉnh tùy thuộc vào cấu hình server của bạn. Nếu bạn muốn được hỗ trợ trong việc quản lý server, hãy cân nhắc sử dụng một dịch vụ như [Laravel Forge](https://forge.laravel.com):

    server {
        listen 80;
        server_name example.com;
        root /srv/example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>
## Tối ưu

<a name="autoloader-optimization"></a>
### Tối ưu autoloader

Khi deploy application vào production, hãy chắc chắn là bạn đã tối ưu hoá class autoloader map của Composer, để Composer có thể nhanh chóng tìm thấy file thích hợp cho một class:

    composer install --optimize-autoloader --no-dev

> {tip} Ngoài việc tối ưu autoloader, bạn cũng nên chắc chắn là luôn có file `composer.lock` trong project source code của bạn. Các library trong project của bạn có thể cài đặt nhanh hơn khi mà có file `composer.lock` này.

<a name="optimizing-configuration-loading"></a>
### Tối ưu load config

Khi deploy application vào production, bạn cũng nên đảm bảo là bạn đã chạy lệnh Artisan `config:cache` trong quá trình deploy:

    php artisan config:cache

Lệnh này sẽ nối tất cả các file config của Laravel thành một file và được lưu vào trong bộ nhớ cache, giúp giảm đáng kể số lượng trao đổi giữa framework với filesystem khi tải các value config của bạn.

> {note} Nếu bạn chạy lệnh `config:cache` trong quá trình deploy, bạn nên đảm bảo là bạn chỉ gọi hàm `env` từ trong các file cấu hình của bạn. Khi các file cấu hình đã được lưu vào trong bộ nhớ cache, thì file `.env` sẽ không được load và tất cả các code được gọi đến hàm `env` sẽ đều trả về `null`.

<a name="optimizing-route-loading"></a>
### Tối ưu load route

Nếu bạn đang build một application lớn với nhiều route, bạn nên đảm bảo rằng bạn đã chạy lệnh Artisan `route:cache` trong quá trình deploy của bạn:

    php artisan route:cache

Lệnh này sẽ giảm tất cả các đăng ký route của bạn vào trong một phương thức duy nhất và lưu trong một file ở cache, nó giúp cải thiện hiệu suất của việc đăng ký route khi đăng ký hàng trăm route.

> {note} Vì chức nằng này sẽ dùng hàm mã hoá chuỗi của PHP, nên bạn chỉ có thể cache được các loại route cho apllication theo loại route cơ bản. PHP sẽ không thể mã hoá được các hàm callback.

<a name="optimizing-view-loading"></a>
### Tối ưu View Loading

Khi deploy ứng dụng của bạn vào production, bạn nên đảm bảo rằng bạn đã chạy lệnh Artisan `view:cache` trong quá trình deploy của bạn:

    php artisan view:cache

Lệnh này biên dịch tất cả các view Blade của bạn để chúng không cần phải biên dịch mỗi khi có request đến, cải thiện hiệu suất cho mỗi request trả về một view.

<a name="deploying-with-forge-or-vapor"></a>
## Deploy cùng Forge và Vapor

Nếu bạn chưa sẵn sàng để quản lý cấu hình server của bạn hoặc không thoải mái với cấu hình các dịch vụ khác nhau cần thiết để chạy ứng dụng Laravel, [Laravel Forge](https://forge.laravel.com) là một điều thay thế tuyệt vời.

Laravel Forge có thể tạo server trên các nhà cung cấp khác nhau như DigitalOcean, Linode, AWS, v.v. Ngoài ra, Forge có thể cài đặt và quản lý tất cả các công cụ cần thiết để xây dựng các ứng dụng Laravel, như Nginx, MySQL, Redis, Memcached, Beanstalk,...

#### Laravel Vapor

Nếu bạn muốn một nền tảng deploy hoàn toàn không có server, và tự động lớn dần theo thời gian, hãy xem xét sử dụng [Laravel Vapor](https://vapor.laravel.com). Laravel Vapor là một nền tảng deploy không có server cho Laravel, và được cung cấp bởi AWS. Chạy ứng dụng Laravel của bạn trên Vapor và yêu thích sự đơn giản có thể mở rộng đến vô tận của serverless. Laravel Vapor được những người tạo ra Laravel tinh chỉnh để hoạt động liền mạch với framework, do đó bạn có thể tiếp tục viết các ứng dụng Laravel của bạn giống hệt với những điều bạn đã từng làm.
