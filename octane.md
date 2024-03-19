# Laravel Octane

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Yêu cầu Server](#server-prerequisites)
    - [RoadRunner](#roadrunner)
    - [Swoole](#swoole)
- [Chạy application của bạn](#serving-your-application)
    - [Chạy application của bạn thông qua HTTPS](#serving-your-application-via-https)
    - [Chạy application của bạn thông qua Nginx](#serving-your-application-via-nginx)
    - [Theo dõi file thay đổi](#watching-for-file-changes)
    - [Chỉ định số lượng Worker](#specifying-the-worker-count)
    - [Chỉ định số lượng request](#specifying-the-max-request-count)
    - [Reload Worker](#reloading-the-workers)
    - [Dừng Server](#stopping-the-server)
- [Tích hợp phụ thuộc và Octane](#dependency-injection-and-octane)
    - [Tích hợp Container](#container-injection)
    - [Tích hợp Request](#request-injection)
    - [Tích hợp Configuration Repository](#configuration-repository-injection)
- [Quản lý Memory Leaks](#managing-memory-leaks)
- [Đồng bộ Task](#concurrent-tasks)
- [Ticks và Intervals](#ticks-and-intervals)
- [Octane Cache](#the-octane-cache)
- [Tables](#tables)

<a name="introduction"></a>
## Giới thiệu

[Laravel Octane](https://github.com/laravel/octane) sẽ tăng cường hiệu suất ứng dụng của bạn bằng cách chạy ứng dụng của bạn bằng các máy chủ ứng dụng hiệu suất cao, bao gồm [Open Swoole](https://swoole.co.uk), [Swoole](https://github.com/swoole/swoole-src) và [RoadRunner](https://roadrunner.dev). Octane khởi động ứng dụng của bạn một lần, và lưu nó vào trong memory và sau đó trả lời các request với tốc độ siêu nhanh.

<a name="installation"></a>
## Cài đặt

Octane có thể được cài đặt thông qua Composer package manager:

```bash
composer require laravel/octane
```

Sau khi cài đặt Octane xong, bạn có thể chạy lệnh Artisan `octane:install`, để cài đặt file cấu hình của Octane vào ứng dụng của bạn:

```bash
php artisan octane:install
```

<a name="server-prerequisites"></a>
## Yêu cầu Server

> {note} Laravel Octane yêu cầu [PHP 8.0+](https://php.net/releases/).

<a name="roadrunner"></a>
### RoadRunner

[RoadRunner](https://roadrunner.dev) được xây dựng dựa trên RoadRunner binary, được xây dựng bằng ngôn ngữ Go. Khi bạn bắt đầu sử dụng một máy chủ octane mà dựa trên Roadrunner lần đầu tiên, Octane sẽ cung cấp tải xuống và cài đặt RoadRunner binary cho bạn.

<a name="roadrunner-via-laravel-sail"></a>
#### RoadRunner Via Laravel Sail

Nếu bạn có kế hoạch phát triển ứng dụng của bạn bằng [Laravel Sail](/docs/{{version}}/sail), bạn nên chạy các lệnh sau để cài đặt Octane và RoadRunner:

```bash
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner
```

Tiếp theo, bạn nên start một shell của Sail và sử dụng `rr` để lấy ra bản build cho Linux mới nhất của Roadrunner Binary:

```bash
./vendor/bin/sail shell

# Within the Sail shell...
./vendor/bin/rr get-binary
```

Sau khi cài đặt xong RoadRunner binary, bạn có thể thoát khỏi session Sail shell của bạn. Bây giờ bạn sẽ cần điều chỉnh file `supervisor.conf` được Sail sử dụng để giữ cho ứng dụng của bạn chạy. Để bắt đầu, hãy thực hiện lệnh Artisan `sail:publish`:

```bash
./vendor/bin/sail artisan sail:publish
```

Tiếp theo, hãy cập nhật lệnh `command` trong file `docker/supervisord.conf` của ứng dụng của bạn để Sail chạy ứng dụng của bạn bằng octane thay vì máy chủ phát triển của PHP:

```ini
command=/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port=8000
```

Cuối cùng, hãy đảm bảo rằng file binary `rr` có quyền chạy và build image Sail của bạn:

```bash
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

<a name="swoole"></a>
### Swoole

Nếu bạn có kế hoạch sử dụng máy chủ ứng dụng Swoole để chạy ứng dụng Laravel Octane của bạn, bạn phải cài đặt extension Swoole PHP. Thông thường, điều này có thể được thực hiện thông qua PECL:

```bash
pecl install swoole
```

<a name="swoole-via-laravel-sail"></a>
#### Swoole Via Laravel Sail

> {note} Trước khi chạy một ứng dụng Octane thông qua Sail, bạn hãy đảm bảo là bạn đã có phiên bản mới nhất của Laravel Sail và chạy `./vendor/bin/sail build --no-cache` trong thư mục gốc ứng dụng của bạn.

Ngoài ra, bạn có thể phát triển ứng dụng Octane dựa trên swoole của bạn bằng [Laravel Sail](/docs/{{version}}/sail), một môi trường phát triển dựa trên Docker chính thức cho Laravel. Mặc định, Laravel Sail có chứa nhiều extension swoole. Tuy nhiên, bạn vẫn sẽ cần điều chỉnh file `supervisor.conf` được Sail sử dụng để giữ cho ứng dụng của bạn chạy. Để bắt đầu, hãy chạy lệnh Artisan `sail:publish`:

```bash
./vendor/bin/sail artisan sail:publish
```

Tiếp theo, hãy cập nhật lệnh `command` trong file `docker/supervisord.conf` của ứng dụng của bạn để Sail chạy ứng dụng của bạn bằng octane thay vì máy chủ phát triển của PHP:

```ini
command=/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port=80
```

Cuối cùng, build image Sail của bạn:

```bash
./vendor/bin/sail build --no-cache
```

<a name="swoole-configuration"></a>
#### Swoole Configuration

Swoole có hỗ trợ thêm một vài tùy chọn cấu hình bổ sung mà bạn có thể thêm vào file cấu hình `octane` của bạn nếu cần thiết. Vì chúng hiếm khi cần phải sửa đổi, nên mặc định, các tùy chọn này sẽ không có trong file cấu hình:

```php
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
];
```

<a name="serving-your-application"></a>
## Chạy application của bạn

Máy chủ octane có thể được khởi động thông qua lệnh Artisan `octane:start`. Mặc định, lệnh này sẽ sử dụng máy chủ được chỉ định trong tùy chọn cấu hình `server` của file cấu hình `octane` của ứng dụng:

```bash
php artisan octane:start
```

Mặc định, Octane sẽ khởi động máy chủ trên cổng 8000, vì vậy bạn có thể truy cập ứng dụng của bạn ở trong trình duyệt web thông qua địa chỉ `http://localhost:8000`.

<a name="serving-your-application-via-https"></a>
### Chạy application của bạn thông qua HTTPS

Mặc định, các ứng dụng chạy qua octane sẽ tạo link với tiền tố là `http://`. Biến môi trường `OCTANE_HTTPS` được sử dụng trong file cấu hình `config/octane.php` trong ứng dụng của bạn, có thể được set thành `true` khi chạy ứng dụng của bạn thông qua HTTPS. Khi giá trị cấu hình này được set thành `true`, octane sẽ bảo Laravel tạo tất cả các link với tiền tố là `https://`:

```php
'https' => env('OCTANE_HTTPS', false),
```

<a name="serving-your-application-via-nginx"></a>
### Chạy application của bạn thông qua Nginx

> {tip} Nếu bạn chưa sẵn sàng quản lý cấu hình máy chủ của bạn hoặc không thoải mái khi cấu hình tất cả các dịch vụ khác nhau cần thiết để chạy ứng dụng Laravel Octane mạnh mẽ, hãy xem [Laravel Forge](https://forge.laravel.com).

Trong môi trường production, bạn nên chạy ứng dụng Octane của bạn đằng sau một máy chủ web truyền thống như Nginx hoặc Apache. Làm như vậy sẽ cho phép máy chủ web phân phối các nội dung tĩnh như hình ảnh và stylesheet cũng như quản lý chứng chỉ SSL của bạn.

Trong ví dụ về cấu hình Nginx ở bên dưới, Nginx sẽ phân phối các request và nội dung tĩnh của trang web tới máy chủ Octane đang chạy trên cổng 8000:

```conf
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;

    index index.php;

    charset utf-8;

    location /index.php {
        try_files /not_exists @octane;
    }

    location / {
        try_files $uri $uri/ @octane;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;

    error_page 404 /index.php;

    location @octane {
        set $suffix "";

        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }

        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
```

<a name="watching-for-file-changes"></a>
### Theo dõi file thay đổi

Vì ứng dụng của bạn đã được load vào memory khi máy chủ Octane khởi động nên mọi thay đổi đối với các file trong ứng dụng của bạn sẽ không được phản ánh khi bạn refresh trình duyệt của bạn. Ví dụ: các định nghĩa route được thêm vào trong file `routes/web.php` của bạn sẽ không được phản ánh cho đến khi máy chủ được khởi động lại. Để thuận tiện, bạn có thể sử dụng flag `--watch` để bảo Octane tự động khởi động lại máy chủ khi có bất kỳ thay đổi mới nào trong file ứng dụng của bạn:

```bash
php artisan octane:start --watch
```

Trước khi sử dụng tính năng này, bạn phải đảm bảo là [Node](https://nodejs.org) đã được cài đặt vào trong môi trường phát triển local của bạn. Ngoài ra, bạn nên cài đặt thư viện theo dõi file [Chokidar](https://github.com/paulmillr/chokidar) vào trong library dự án của bạn:

```bash
npm install --save-dev chokidar
```

Bạn có thể cấu hình các thư mục và file cần theo dõi bằng tùy chọn cấu hình `watch` trong file cấu hình `config/octane.php` trong ứng dụng của bạn.

<a name="specifying-the-worker-count"></a>
### Chỉ định số lượng Worker

Mặc định, Octane sẽ khởi động số lượng worker theo mỗi lõi CPU do máy của bạn cung cấp. Sau đó, những worker này sẽ được sử dụng để xử lý các request HTTP đến khi nó vào ứng dụng của bạn. Bạn có thể chỉ định số lượng worker mà bạn mong muốn bằng cách sử dụng tùy chọn `--workers` khi gọi lệnh `octane:start`:

```bash
php artisan octane:start --workers=4
```

Nếu bạn đang sử dụng máy chủ Swoole, bạn cũng có thể chỉ định số lượng ["worker"](#concurrent-tasks) mà bạn muốn bắt đầu:

```bash
php artisan octane:start --workers=4 --task-workers=6
```

<a name="specifying-the-max-request-count"></a>
### Chỉ định số lượng request

Để giúp ngăn chặn tràn bộ nhớ, Octane có thể khởi động lại một cách nhẹ nhàng một worker sau khi nó xử lý xong một số request nhất định. Để bảo Octane thực hiện điều này, bạn có thể sử dụng tùy chọn `--max-requests`:

```bash
php artisan octane:start --max-requests=250
```

<a name="reloading-the-workers"></a>
### Reload Worker

Bạn có thể khởi động lại các worker của máy chủ Octane một cách dễ dàng bằng cách sử dụng lệnh `octane:reload`. Thông thường, việc này nên được thực hiện sau khi deploy một code mới lên server để code mới của bạn có thể được load lại vào trong bộ nhớ và được sử dụng để chạy cho các request tiếp theo:

```bash
php artisan octane:reload
```

<a name="stopping-the-server"></a>
### Dừng Server

Bạn có thể dừng máy chủ Octane bằng lệnh Artisan `octane:stop`:

```bash
php artisan octane:stop
```

<a name="checking-the-server-status"></a>
#### Checking The Server Status

Bạn có thể kiểm tra trạng thái hiện tại của máy chủ Octane bằng lệnh Artisan `octane:status`:

```bash
php artisan octane:status
```

<a name="dependency-injection-and-octane"></a>
## Tích hợp phụ thuộc và Octane

Vì Octane chỉ khởi động ứng dụng của bạn một lần duy nhất và giữ nó ở trong bộ nhớ trong khi chạy các request nên có một số lưu ý mà bạn nên cân nhắc khi xây dựng ứng dụng của bạn. Ví dụ: phương thức `register` và `boot` của service provider trong ứng dụng của bạn sẽ chỉ được thực thi một lần khi worker khởi động. Trong các request tiếp theo, instance app tương tự sẽ được sử dụng lại.

Do đó, bạn nên đặc biệt cẩn thận khi tích hợp application service container hoặc request vào hàm khởi tạo của bất kỳ đối tượng nào. Bởi bằng cách đó, đối tượng có thể chứa version cũ của container hoặc request trong các request tiếp theo.

Octane sẽ tự động xử lý việc reset lại mọi trạng thái của framework giữa các request. Tuy nhiên, Octane không phải lúc nào cũng biết cách reset lại trạng thái chung do ứng dụng của bạn tạo ra. Do đó, bạn nên biết cách xây dựng ứng dụng của bạn thân thiện với Octane. Dưới đây, chúng ta sẽ thảo luận về một số tình huống phổ biến nhất có thể gây ra sự cố khi sử dụng Octane.

<a name="container-injection"></a>
### Tích hợp Container

Nói chung, bạn nên tránh tích hợp service container hoặc instance HTTP request vào hàm khởi tạo của bất kỳ đối tượng nào. Ví dụ: liên kết sau đây sẽ tích hợp toàn bộ service container vào một đối tượng được liên kết dưới dạng một liên kết singleton:

```php
use App\Service;

/**
 * Register any application services.
 *
 * @return void
 */
public function register()
{
    $this->app->singleton(Service::class, function ($app) {
        return new Service($app);
    });
}
```

Trong ví dụ này, nếu instance `Service` được resolve trong quá trình khởi động ứng dụng, container sẽ được tích hợp vào service và container đó sẽ được instance `Service` giữ trong các request tiếp theo. Điều này **có thể** không phải là một vấn đề lớn đối với ứng dụng của bạn; tuy nhiên, điều này có thể dẫn đến việc container sẽ bị thiếu các liên kết được thêm vào sau trong chu kỳ khởi động hoặc bởi một request tiếp theo.

Để giải quyết, bạn có thể ngừng đăng ký liên kết dưới dạng singleton hoặc bạn có thể tích hợp một closure container resolver vào service luôn resolve ra instance container hiện tại:

```php
use App\Service;
use Illuminate\Container\Container;

$this->app->bind(Service::class, function ($app) {
    return new Service($app);
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance());
});
```

Helper global `app` và phương thức `Container::getInstance()` sẽ luôn trả về phiên bản mới nhất của application container.

<a name="request-injection"></a>
### Tích hợp Request

Nói chung, bạn nên tránh tích hợp service container hoặc instance HTTP request vào hàm khởi tạo của bất kỳ đối tượng nào. Ví dụ: liên kết sau đây sẽ tích hợp toàn bộ instance request vào một đối tượng được liên kết dưới dạng một liên kết singleton:

```php
use App\Service;

/**
 * Register any application services.
 *
 * @return void
 */
public function register()
{
    $this->app->singleton(Service::class, function ($app) {
        return new Service($app['request']);
    });
}
```

Trong ví dụ này, nếu instance `Service` được resolve trong quá trình khởi động ứng dụng, HTTP request sẽ được tích hợp vào service và request tương tự đó sẽ được instance `Service` giữ trong các request tiếp theo. Do đó, tất cả dữ liệu header, input và chuỗi query cũng như tất cả dữ liệu request khác đều sẽ không chính xác.

Để giải quyết, bạn có thể ngừng đăng ký liên kết dưới dạng singleton hoặc bạn có thể tích hợp một closure request resolver vào service luôn resolve ra instance request hiện tại. Hoặc, cách tiếp cận đơn giản được khuyên dùng nhất là chỉ truyền những thông tin request mà đối tượng của bạn cần tới một trong các method của đối tượng khi chạy:

```php
use App\Service;

$this->app->bind(Service::class, function ($app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function ($app) {
    return new Service(fn () => $app['request']);
});

// Or...

$service->method($request->input('name'));
```

Helper global `request` sẽ luôn trả về request mà ứng dụng hiện đang xử lý và do đó sẽ an toàn khi sử dụng trong ứng dụng của bạn.

> {note} Bạn có thể khai báo instance `Illuminate\Http\Request` trên các phương thức controller và route closure của bạn.

<a name="configuration-repository-injection"></a>
### Tích hợp Configuration Repository

Nói chung, bạn nên tránh tích hợp instance repository vào hàm khởi tạo của bất kỳ đối tượng nào. Ví dụ: liên kết sau đây sẽ tích hợp một configuration repository vào một đối tượng được liên kết dưới dạng một liên kết singleton:

```php
use App\Service;

/**
 * Register any application services.
 *
 * @return void
 */
public function register()
{
    $this->app->singleton(Service::class, function ($app) {
        return new Service($app->make('config'));
    });
}
```

Trong ví dụ này, nếu giá trị cấu hình bị thay đổi giữa các request, thì service đó sẽ không có quyền truy cập vào các giá trị mới vì nó phụ thuộc vào instance repository ban đầu.

Để giải quyết, bạn có thể ngừng đăng ký liên kết dưới dạng singleton hoặc bạn có thể tích hợp một closure configuration repository resolver vào class:

```php
use App\Service;
use Illuminate\Container\Container;

$this->app->bind(Service::class, function ($app) {
    return new Service($app->make('config'));
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance()->make('config'));
});
```

Helper global `config` sẽ luôn trả về phiên bản mới nhất của configuration repository và do đó sẽ an toàn khi sử dụng trong ứng dụng của bạn.

<a name="managing-memory-leaks"></a>
### Quản lý Memory Leaks

Hãy nhớ rằng, Octane sẽ lưu ứng dụng của bạn vào trong bộ nhớ giữa các request; do đó, việc thêm dữ liệu vào mảng tĩnh được duy trì liên tục sẽ dẫn đến tràn bộ nhớ. Ví dụ: controller sau sẽ bị tràn bộ nhớ do mỗi request tới ứng dụng sẽ liên tục thêm dữ liệu vào mảng `$data` tĩnh:

```php
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * Handle an incoming request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return void
 */
public function index(Request $request)
{
    Service::$data[] = Str::random(10);

    // ...
}
```

Trong khi xây dựng ứng dụng của bạn, bạn nên đặc biệt cẩn thận để tránh tạo ra các loại tràn bộ nhớ này. Bạn nên giám sát việc sử dụng bộ nhớ của ứng dụng trong quá trình phát triển local để đảm bảo là bạn không gây ra lỗi tràn bộ nhớ mới nào trong ứng dụng của bạn.

<a name="concurrent-tasks"></a>
## Đồng bộ Task

> {note} Tính năng này yêu cầu [Swoole](#swoole).

Khi sử dụng Swoole, bạn có thể thực hiện các thao tác đồng bộ thông qua các tác vụ background có dung lượng nhẹ. Bạn có thể thực hiện việc này bằng phương thức `concurrently` của Octane. Bạn có thể kết hợp phương thức này với việc gán mảng PHP để lấy ra kết quả của từng thao tác:

```php
use App\Models\User;
use App\Models\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

Các tác vụ đồng bộ do Octane xử lý sẽ sử dụng các "task worker" của Swoole và chạy trong một process hoàn toàn khác với request đến. Số lượng worker có sẵn để xử lý các tác vụ đồng bộ sẽ được xác định bởi lệnh `--task-workers` trong lệnh `octane:start`:

```bash
php artisan octane:start --workers=4 --task-workers=6
```

<a name="ticks-and-intervals"></a>
## Ticks và Intervals

> {note} Tính năng này yêu cầu [Swoole](#swoole).

Khi sử dụng Swoole, bạn có thể đăng ký các thao tác "tick" sẽ được chạy sau mỗi số giây được chỉ định. Bạn có thể đăng ký callback "tick" thông qua phương thức `tick`. Tham số đầu tiên được cung cấp cho phương thức `tick` phải là một chuỗi đại diện cho tên của tick. Tham số thứ hai phải là một callable và sẽ được gọi trong khoảng thời gian đã chỉ định.

Trong ví dụ này, chúng ta sẽ đăng ký một closure được gọi sau mỗi 10 giây. Thông thường, phương thức `tick` phải được gọi trong phương thức `boot` của một trong các service provider của ứng dụng của bạn:

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10);
```

Sử dụng phương thức `immediate`, bạn có thể bảo Octane gọi ngay callback tick khi máy chủ Octane được khởi động lần đầu và cứ sau N giây sau đó:

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10)
        ->immediate();
```

<a name="the-octane-cache"></a>
## Octane Cache

> {note} Tính năng này yêu cầu [Swoole](#swoole).

Khi sử dụng Swoole, bạn có thể tận dụng driver Octane cache, driver này cung cấp tốc độ đọc và ghi lên tới 2 triệu thao tác mỗi giây. Do đó, driver cache này là sự lựa chọn tuyệt vời cho các ứng dụng cần tốc độ đọc/ghi cực cao từ layer cache của chúng.

Driver cache này được hỗ trợ bởi [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table). Tất cả dữ liệu được lưu trong cache đều có sẵn cho tất cả worker trên máy chủ. Tuy nhiên, dữ liệu được lưu trong cache sẽ bị xóa khi máy chủ khởi động lại:

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```

> {tip} Số lượng item tối đa được phép có trong cache Octane có thể được định nghĩa trong file cấu hình `octane` của ứng dụng của bạn.

<a name="cache-intervals"></a>
### Cache Intervals

Ngoài các phương thức điển hình được cung cấp bởi hệ thống cache của Laravel, driver cache Octane còn có các cache dựa trên khoảng thời gian. Các cache này được tự động refresh theo một khoảng thời gian được chỉ định và phải được đăng ký bằng phương thức `boot` của một trong các service provider trong ứng dụng của bạn. Ví dụ: cache sau sẽ được refresh cứ sau mỗi 5 giây:

```php
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5)
```

<a name="tables"></a>
## Tables

> {note} Tính năng này yêu cầu [Swoole](#swoole).

Khi sử dụng Swoole, bạn có thể định nghĩa và tương tác với [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table). Các Swoole table cung cấp hiệu suất cực cao và tất cả các worker trên máy chủ có thể truy cập vào dữ liệu trong các table này. Tuy nhiên, dữ liệu bên trong chúng sẽ bị mất khi máy chủ khởi động lại.

Các table này phải được định nghĩa trong mảng cấu hình `tables` của file cấu hình `octane` trong ứng dụng của bạn. Một table ví dụ cho phép tối đa 1000 hàng đã được cấu hình sẵn cho bạn. Kích thước tối đa của các cột có thể được cấu hình bằng cách chỉ định kích thước cột sau loại cột như bên dưới:

```php
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

Để truy cập vào một table, bạn có thể sử dụng phương thức `Octane::table`:

```php
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

> {note} Các loại cột được Swoole table hỗ trợ là: `string`, `int` và `float`.
