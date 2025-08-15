# Laravel Reverb

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
    - [Thông tin xác thực](#application-credentials)
    - [Allowed Origins](#allowed-origins)
    - [Thêm ứng dụng](#additional-applications)
    - [SSL](#ssl)
- [Chạy Server](#running-server)
    - [Debugging](#debugging)
    - [Restarting](#restarting)
- [Chạy Reverb trong Production](#production)
    - [Mở Files](#open-files)
    - [Event Loop](#event-loop)
    - [Web Server](#web-server)
    - [Ports](#ports)
    - [Process Management](#process-management)
    - [Scaling](#scaling)

<a name="introduction"></a>
## Giới thiệu

[Laravel Reverb](https://github.com/laravel/reverb) mang đến khả năng giao tiếp WebSocket thời gian thực cực nhanh và có khả năng mở rộng trực tiếp đến ứng dụng Laravel của bạn và cung cấp khả năng tích hợp liền mạch với bộ công cụ broadcasting event hiện có của Laravel.

<a name="installation"></a>
## Cài đặt

> [!WARNING]
> Laravel Reverb yêu cầu PHP 8.2 trở lên và Laravel 10.47 trở lên.

Bạn có thể sử dụng trình quản lý package Composer để cài đặt Reverb vào dự án Laravel của bạn:

```sh
composer require laravel/reverb
```

Sau khi package đã được cài đặt xong, bạn có thể chạy lệnh cài đặt của Reverb để export ra cấu hình, thêm các biến môi trường cần thiết của Reverb và bật broadcasting event trong ứng dụng của bạn:

```sh
php artisan reverb:install
```

<a name="configuration"></a>
## Cấu hình

Lệnh `reverb:install` sẽ tự động cấu hình Reverb bằng một tập hợp các tùy chọn mặc định. Nếu bạn muốn thực hiện bất kỳ thay đổi cấu hình nào, bạn có thể thực hiện bằng cách cập nhật các biến môi trường của Reverb hoặc cập nhật file cấu hình `config/reverb.php`.

<a name="application-credentials"></a>
### Thông tin xác thực

Để thiết lập kết nối tới Reverb, một tập hợp các thông tin xác thực "ứng dụng" Reverb phải được trao đổi giữa client và server. Các thông tin xác thực này được cấu hình trên server và được sử dụng để xác thực request từ client. Bạn có thể định nghĩa các thông tin xác thực này bằng các biến môi trường sau:

```ini
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
```

<a name="allowed-origins"></a>
### Allowed Origins

Bạn cũng có thể định nghĩa các origin mà các client request có thể xuất phát bằng cách cập nhật giá trị của giá trị cấu hình `allowed_origins` trong phần `apps` của file cấu hình `config/reverb.php`. Bất kỳ request nào từ origin không được liệt kê trong origin được phép của bạn sẽ bị từ chối. Bạn có thể cho phép tất cả các origin bằng cách sử dụng `*`:

```php
'apps' => [
    [
        'id' => 'my-app-id',
        'allowed_origins' => ['laravel.com'],
        // ...
    ]
]
```

<a name="additional-applications"></a>
### Thêm ứng dụng

Thông thường, Reverb cung cấp server WebSocket cho ứng dụng mà nó được cài đặt. Tuy nhiên, nó có thể phục vụ cho nhiều ứng dụng khác chỉ bằng một lần cài đặt Reverb.

Ví dụ, bạn có thể muốn duy trì một ứng dụng Laravel duy nhất, thông qua Reverb, cung cấp kết nối WebSocket cho nhiều ứng dụng. Điều này có thể thực hiện được bằng cách định nghĩa nhiều `apps` trong file cấu hình `config/reverb.php` của ứng dụng:

```php
'apps' => [
    [
        'app_id' => 'my-app-one',
        // ...
    ],
    [
        'app_id' => 'my-app-two',
        // ...
    ],
],
```

<a name="ssl"></a>
### SSL

Trong nhiều trường hợp, kết nối WebSocket secure có thể được xử lý bởi web server (Nginx, etc.) trước khi request đó được chuyển đến server Reverb của bạn.

Tuy nhiên, thỉnh thoảng việc server Reverb xử lý trực tiếp các kết nối secure có thể hữu ích, chẳng hạn như trong quá trình phát triển ở local. Nếu bạn đang sử dụng chức năng secure site của [Laravel Herd](https://herd.laravel.com), hoặc bạn đang sử dụng [Laravel Valet](/docs/{{version}}/valet) và đã chạy [lệnh secure](/docs/{{version}}/valet#securing-sites) trên ứng dụng của bạn, bạn có thể sử dụng chứng chỉ Herd hoặc Valet được tạo cho trang web của bạn để secure các kết nối Reverb. Để thực hiện điều này, hãy set biến môi trường `REVERB_HOST` thành hostname của trang web hoặc truyền tùy chọn hostname khi khởi động server Reverb:

```sh
php artisan reverb:start --host="0.0.0.0" --port=8080 --hostname="laravel.test"
```

Bời vì domain của Herd và Valet sẽ được chuyển về `localhost`, nên khi chạy lệnh trên, máy chủ Reverb của bạn sẽ có thể truy cập được thông qua giao thức WebSocket secure (wss) ở `wss://laravel.test:8080`.

Bạn cũng có thể tự mình chọn một chứng chỉ bằng cách định nghĩa các tùy chọn `tls` trong file cấu hình `config/reverb.php` của ứng dụng. Trong mảng các tùy chọn `tls`, bạn có thể cung cấp bất kỳ tùy chọn nào được hỗ trợ bởi [tùy chọn SSL của PHP](https://www.php.net/manual/en/context.ssl.php):

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

<a name="running-server"></a>
## Chạy Server

Server Reverb có thể được khởi động bằng lệnh Artisan `reverb:start`:

```sh
php artisan reverb:start
```

Mặc định, server Reverb sẽ được khởi động tại `0.0.0.0:8080`, giúp server này có thể truy cập được từ mọi network interface.

Nếu bạn cần chỉ định host hoặc cổng riêng, bạn có thể thực hiện điều này thông qua tùy chọn `--host` và `--port` khi khởi động server:

```sh
php artisan reverb:start --host=127.0.0.1 --port=9000
```

Ngoài ra, bạn có thể định nghĩa các biến môi trường `REVERB_SERVER_HOST` và `REVERB_SERVER_PORT` trong file cấu hình `.env` của ứng dụng.

Không nên nhầm lẫn các biến môi trường `REVERB_SERVER_HOST` và `REVERB_SERVER_PORT` với các biến `REVERB_HOST` và `REVERB_PORT`. Biến môi trường server host và server port sẽ chỉ định host và cổng để chạy server Reverb, trong khi cặp biến môi trường reverb host và reverb port sẽ hướng dẫn Laravel gửi thông điệp broadcast đến đâu. Ví dụ: trong môi trường production, bạn có thể gửi các request từ public Reverb hostname trên cổng `443` đến máy chủ Reverb đang hoạt động trên `0.0.0.0:8080`. Trong trường hợp này, các biến môi trường của bạn sẽ được định nghĩa như sau:

```ini
REVERB_SERVER_HOST=0.0.0.0
REVERB_SERVER_PORT=8080

REVERB_HOST=ws.laravel.com
REVERB_PORT=443
```

<a name="debugging"></a>
### Debugging

Để cải thiện hiệu suất, mặc định Reverb sẽ không xuất bất kỳ thông tin debug nào. Nếu bạn muốn xem luồng dữ liệu đi qua server Reverb, bạn có thể thêm tùy chọn `--debug` vào lệnh `reverb:start`:

```sh
php artisan reverb:start --debug
```

<a name="restarting"></a>
### Restarting

Vì Reverb là một tiến trình chạy lâu dài nên những thay đổi trong code của bạn sẽ không được thực hiện nếu không khởi động lại server thông qua lệnh Artisan `reverb:restart`.

Lệnh `reverb:restart` sẽ đảm bảo tất cả các kết nối sẽ được kết thúc trước khi máy chủ được dừng. Nếu bạn đang chạy Reverb với trình quản lý process như Supervisor, thì máy chủ sẽ tự động được trình quản lý process khởi động lại sau khi tất cả các kết nối đã kết thúc:

```sh
php artisan reverb:restart
```

<a name="production"></a>
## Chạy Reverb trong Production

Do bản chất hoạt động lâu dài của máy chủ WebSocket, bạn có thể cần phải thực hiện một số tối ưu hóa cho máy chủ và môi trường lưu trữ của bạn để đảm bảo máy chủ Reverb có thể xử lý hiệu quả số lượng lớn kết nối tối ưu cho các tài nguyên có sẵn trên máy chủ của bạn.

> [!NOTE]
> Nếu trang web của bạn được quản lý bởi [Laravel Forge](https://forge.laravel.com), bạn có thể tự động tối ưu máy chủ của bạn cho Reverb trực tiếp từ bảng điều khiển "Ứng dụng". Bằng cách bật tích hợp Reverb, Forge sẽ đảm bảo máy chủ của bạn sẵn sàng hoạt động, bao gồm cả cài đặt mọi extension cần thiết và tăng số lượng kết nối.

<a name="open-files"></a>
### Mở Files

Mỗi kết nối WebSocket sẽ được lưu trong bộ nhớ cho đến khi client hoặc máy chủ bị ngắt kết nối. Trong môi trường Unix và các môi trường tương tự Unix, mỗi kết nối được biểu diễn bằng một file. Tuy nhiên, thường có giới hạn về số lượng file được phép mở ở cả cấp độ hệ điều hành và cấp độ ứng dụng.

<a name="operating-system"></a>
#### Operating System

Trên hệ điều hành Unix, bạn có thể xác định số lượng file được phép mở bằng lệnh `ulimit`:

```sh
ulimit -n
```

Lệnh này sẽ hiển thị giới hạn file được mở cho các người dùng khác nhau. Bạn có thể cập nhật các giá trị này bằng cách chỉnh sửa file `/etc/security/limits.conf`. Ví dụ: cập nhật số lượng file mở tối đa lên 10000 cho người dùng `forge` sẽ như sau:

```ini
# /etc/security/limits.conf
forge        soft  nofile  10000
forge        hard  nofile  10000
```

<a name="event-loop"></a>
### Event Loop

Về cơ bản, Reverb sử dụng vòng lặp event ReactPHP để quản lý các kết nối WebSocket trên máy chủ. Mặc định, vòng lặp event này được hỗ trợ bởi `stream_select`, không yêu cầu thêm bất kỳ extension nào. Tuy nhiên, `stream_select` thường bị giới hạn ở 1.024 file được mở. Do đó, nếu bạn dự định xử lý hơn 1.000 kết nối đồng thời, bạn sẽ cần sử dụng một vòng lặp event thay thế không bị ràng buộc bởi các hạn chế tương tự.

Reverb sẽ tự động chuyển sang các vòng lặp khác được hỗ trợ bởi `ext-event`, `ext-ev` hoặc `ext-uv` khi sẵn sàng. Tất cả các extension PHP này đều có thể cài đặt qua PECL:

```sh
pecl install event
# or
pecl install ev
# or
pecl install uv
```

<a name="web-server"></a>
### Web Server

Trong nhiều các trường hợp, Reverb sẽ chạy trên một cổng mà không phải cổng web trên máy chủ của bạn. Vì vậy, để chuyển lưu lượng đến Reverb, bạn nên cấu hình một proxy. Giả sử Reverb đang chạy trên máy chủ `0.0.0.0` và cổng `8080` và máy chủ của bạn đang sử dụng là máy chủ web Nginx, một proxy có thể được định nghĩa cho máy chủ Reverb của bạn bằng cách sử dụng cấu hình Nginx site như sau:

```nginx
server {
    ...

    location / {
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";

        proxy_pass http://0.0.0.0:8080;
    }

    ...
}
```

Thông thường, máy chủ web được cấu hình để giới hạn số lượng kết nối được phép nhằm tránh quá tải máy chủ. Để tăng số lượng kết nối được phép trên máy chủ web Nginx lên 10000, các giá trị `worker_rlimit_nofile` và `worker_connections` của file `nginx.conf` sẽ cần được cập nhật:

```nginx
user forge;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
worker_rlimit_nofile 10000;

events {
  worker_connections 10000;
  multi_accept on;
}
```

Cấu hình trên cho phép tạo tối đa 10000 Nginx worker cho mỗi process. Ngoài ra, cấu hình này cũng set giới hạn file được mở của Nginx là 10000.

<a name="ports"></a>
### Ports

Các hệ điều hành dựa trên Unix thường sẽ giới hạn số lượng cổng có thể mở trên máy chủ. Bạn có thể xem phạm vi được phép mở thông qua lệnh sau:

 ```sh
 cat /proc/sys/net/ipv4/ip_local_port_range
# 32768	60999
```

Kết quả output lệnh ở trên cho thấy máy chủ có thể xử lý tối đa 28231 (60999 - 32768) kết nối vì mỗi kết nối đều yêu cầu một cổng trống. Mặc dù chúng tôi khuyến cáo sử dụng [horizontal scaling](#scaling) để tăng số lượng kết nối được phép, bạn có thể tăng số lượng cổng mở bằng cách cập nhật phạm vi cổng được phép mở trong file cấu hình `/etc/sysctl.conf` của máy chủ.

<a name="process-management"></a>
### Process Management

Trong nhiều các trường hợp, bạn nên sử dụng trình quản lý process như Supervisor để đảm bảo máy chủ Reverb sẽ luôn hoạt động. Nếu bạn đang sử dụng Supervisor để chạy Reverb, bạn nên cập nhật thiết lập `minfds` của file `supervisor.conf` trên máy chủ để đảm bảo Supervisor có thể mở các file cần thiết để xử lý kết nối đến máy chủ Reverb của bạn:

```ini
[supervisord]
...
minfds=10000
```

<a name="scaling"></a>
### Scaling

Nếu bạn cần xử lý nhiều kết nối hơn mức mà một máy chủ có thể đáp ứng, bạn có thể mở rộng máy chủ Reverb theo chiều ngang. Tận dụng khả năng publish và subscribe của Redis, Reverb có thể quản lý các kết nối trên nhiều máy chủ. Khi một trong các máy chủ Reverb của ứng dụng nhận được tin nhắn, máy chủ sẽ sử dụng Redis để publish tin nhắn đến tất cả các máy chủ khác.

Để bật tính năng mở rộng theo chiều ngang, bạn nên set biến môi trường `REVERB_SCALING_ENABLED` thành `true` trong file cấu hình `.env` của ứng dụng:

```env
REVERB_SCALING_ENABLED=true
```

Tiếp theo, bạn nên có một máy chủ Redis trung tâm mà tất cả các máy chủ Reverb sẽ giao tiếp đến. Reverb sẽ sử dụng [kết nối Redis mặc định được cấu hình cho ứng dụng của bạn](/docs/{{version}}/redis#configuration) để publish tin nhắn đến tất cả các máy chủ Reverb của bạn.

Sau khi bật tùy chọn mở rộng của Reverb và cấu hình máy chủ Redis, bạn chỉ cần gọi lệnh `reverb:start` trên tất cả các máy chủ Reverb của bạn. Các máy chủ Reverb này nên được đặt phía sau một load balancer để phân bổ đều các request giữa các máy chủ.
