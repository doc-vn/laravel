# Deployment

- [Giới thiệu](#introduction)
- [Cấu hình server](#server-configuration)
    - [Nginx](#nginx)
- [Tối ưu](#optimization)
    - [Tối ưu autoloader](#autoloader-optimization)
    - [Tối ưu load config](#optimizing-configuration-loading)
    - [Tối ưu load route](#optimizing-route-loading)
- [Deploy cùng Forge](#deploying-with-forge)

<a name="introduction"></a>
## Giới thiệu

Khi bạn đã sẵn sàng deploy application Laravel của bạn vào production, có một số điều quan trọng mà bạn có thể làm để đảm bảo application của bạn chạy hiệu quả nhất có thể. Trong tài liệu này, chúng tôi sẽ đề cập đến một số điểm khởi đầu tuyệt vời để đảm bảo application Laravel của bạn được deploy đúng cách.

<a name="server-configuration"></a>
## Cấu hình server

<a name="nginx"></a>
### Nginx

Nếu bạn đang deploy application của bạn đến một server đang chạy Nginx, bạn có thể sử dụng file cấu hình sau đây để làm điểm bắt đầu để cấu hình server web của bạn. Nhiều khả năng, file này sẽ cần được tùy chỉnh tùy thuộc vào cấu hình server của bạn. Nếu bạn muốn được hỗ trợ trong việc quản lý server của mình, hãy cân nhắc sử dụng một dịch vụ như [Laravel Forge](https://forge.laravel.com):

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
            fastcgi_index index.php;
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

Khi đang deploy application vào production, hãy chắc chắn rằng bạn đã tối ưu hoá class autoloader map của Composer, để Composer có thể nhanh chóng tìm thấy file thích hợp cho một class:

    composer install --optimize-autoloader

> {tip} Ngoài việc tối ưu autoloader, bạn cũng nên chắc chắn là luôn có file `composer.lock` trong project source code của bạn. Các library trong project của bạn có thể cài đặt nhanh hơn khi mà có file `composer.lock`.

<a name="optimizing-configuration-loading"></a>
### Tối ưu load config

Khi đang deploy application vào production, bạn nên đảm bảo rằng bạn đã chạy lệnh Artisan `config:cache` trong quá trình deploy của bạn:

    php artisan config:cache

Lệnh này sẽ kết hợp tất cả các file config của Laravel thành một file và được lưu vào trong bộ nhớ cache, giúp giảm đáng kể số lượng trao đổi giữa framework với filesystem khi tải các value config của bạn.

<a name="optimizing-route-loading"></a>
### Tối ưu load route

Nếu bạn đang build một application lớn với nhiều route, bạn nên đảm bảo rằng bạn đã chạy lệnh Artisan `route:cache` trong quá trình deploy của bạn:

    php artisan route:cache

Lệnh này sẽ giảm tất cả các đăng ký route của bạn vào trong một phương thức duy nhất và được lưu trong một file ở cache, nó giúp cải thiện hiệu suất của việc đăng ký route khi đăng ký hàng trăm route.

> {note} Vì chức nằng này sẽ dùng hàm mã hoá thành chuỗi của PHP, bạn chỉ có thể cache được các route cho apllication theo loại route cơ bản. PHP sẽ không thể mã hoá được các hàm callback.

<a name="deploying-with-forge"></a>
## Deploy cùng Forge

Nếu bạn chưa sẵn sàng để quản lý cấu hình server của riêng mình hoặc không thoải mái với cấu hình tất cả các dịch vụ khác nhau cần thiết để chạy ứng dụng Laravel, [Laravel Forge](https://forge.laravel.com) là một điều thay thế tuyệt vời.

Laravel Forge có thể tạo server trên các nhà cung cấp khác nhau như DigitalOcean, Linode, AWS, v.v. Ngoài ra, Forge có thể cài đặt và quản lý tất cả các công cụ cần thiết để xây dựng các ứng dụng Laravel, như Nginx, MySQL, Redis, Memcached, Beanstalk,...
