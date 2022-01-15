# Cấu hình

- [Giới thiệu](#introduction)
- [Biến môi trường](#environment-configuration)
    - [Loại biến môi trường](#environment-variable-types)
    - [Nhận về giá trị biến môi trường](#retrieving-environment-configuration)
    - [Xác định môi trường hiện tại](#determining-the-current-environment)
- [Nhận về biến config](#accessing-configuration-values)
- [Caching các biến config](#configuration-caching)
- [Chế độ bảo trì](#maintenance-mode)

<a name="introduction"></a>
## Giới thiệu

Tất cả các file config cho Laravel framework được lưu trữ trong thư mục `config`. Các biến config đều đã được tài liệu hoá bằng comment, nên hãy xem qua và làm quen với chúng.

<a name="environment-configuration"></a>
## Biến môi trường

Nó rất hữu dụng cho những giá trị config khác nhau dựa trên môi trường, nơi mà ứng dụng của bạn đang được chạy. Ví dụ, bạn có thể muốn dùng một loại cache riêng ở local, hơn là dùng loại cache mà đang được sử dụng ở trên server production.

Để dễ dàng hơn, Laravel sử dụng thư viện PHP [DotEnv](https://github.com/vlucas/phpdotenv) được tạo bởi Vance Lucas. Trong thư mục project gốc của bạn có chứa một file `.env.example`. Nếu  bạn cài đặt project thông qua Laravel installer hoặc Composer. Thì file đó sẽ được tự động sửa thành tên `.env`. Nếu không có file đó, thì bạn nên tạo file đó.

File `.env` không nên commit vào trong source code của bạn, bởi vì mỗi người phát triển hoặc mỗi server mà đang chạy web application của bạn có thể yêu cầu những biến môi trường khác nhau. Hơn thế nữa, nó cũng là một rủi ro bảo mật trong trường hợp có kẻ nào đó xâm nhập được vào source code của bạn, thì tất cả các thông tin đều sẽ bị lộ.

Nếu bạn đang phát triển cùng với một team, bạn nên thêm file `.env.example` vào trong project của bạn, sau đó, thêm cái giá trị ví dụ vào trong file `.env.example`, các nhà phát triển tiếp theo sẽ hiểu rõ ràng hơn về các biến môi trường cần được cài đặt để chạy application của bạn, bạn cũng có thể tạo một file `.env.testing`. File này sẽ ghi đè vào file `.env` khi chạy PHPUnit để test hoặc khi chạy lệnh Artisan với lựa chọn là `--env=testing`.

> {tip} Tất cả các biến trong file `.env` có thể bị ghi đè bởi biến môi trường bên ngoài như là biến môi trường server hoặc system.

<a name="environment-variable-types"></a>
### Loại biến môi trường

Tất cả các biến trong file `.env` của bạn đều được nhận dạng là dưới dạng kiểu string, vì vậy có một số giá trị đã được tạo để cho phép bạn trả về nhiều kiểu hơn từ hàm `env()`:

`.env` Value  | `env()` Value
------------- | -------------
true | (bool) true
(true) | (bool) true
false | (bool) false
(false) | (bool) false
empty | (string) ''
(empty) | (string) ''
null | (null) null
(null) | (null) null

Nếu bạn cần định nghĩa một biến môi trường có chứa khoảng trắng, bạn có thể làm như vậy bằng cách đặt giá trị đó vào trong dấu ngoặc kép.

    APP_NAME="My Application"

<a name="retrieving-environment-configuration"></a>
### Nhận về giá trị biến môi trường

Khi ứng dụng của bạn nhận một request, thì tất cả các biến môi trường sẽ đều được load vào `$_ENV` PHP super-global. Tuy nhiên, bạn cũng có thể dùng hàm `env` trong helper để nhận về các biến môi trường. Trong thực tế, nếu bạn nhìn vào file config của Laravel, bạn sẽ nhận thấy có một số config đang dùng helper trên:

    'debug' => env('APP_DEBUG', false),

Tham số thứ hai được truyền vào trong hàm `env` là giá trị mặc định. Giá trị này sẽ được trả về nếu như biến môi trường của bạn không tồn tại.

<a name="determining-the-current-environment"></a>
### Xác định môi trường hiện tại

Môi trường hiện tại có thể được xác định thông biến `APP_ENV` trong file `.env`. Bạn có thể lấy giá trị đó thông qua hàm `environment` trong [facade](/docs/{{version}}/facades) `App`:

    $environment = App::environment();

Bạn cũng có thể truyền vào hàm `environment` tên của một môi trường để kiểm tra xem môi trường hiện tại có đúng là môi trường đó hay không. Hàm đó sẽ trả về `true` nếu tên môi trường trùng với tên môi trường đang được định nghĩa trong file `.env` hiện tại.

    if (App::environment('local')) {
        // Môi trường hiện tại là local
    }

    if (App::environment(['local', 'staging'])) {
        // Môi trường hiện tại có thể là local hoặc staging
    }

> {tip} Môi trường hiện tại của application có thể bị ghi đè bởi một biến môi trường `APP_ENV` khác ở mức độ server. Nó có thể hữu ích khi mà bạn cần chia sẻ một application cho những môi trường khác, ví dụ bạn có thể ghi đè tên host đúng với môi trường mà bạn mong muốn trong config của server bạn.

<a name="accessing-configuration-values"></a>
## Nhận về biến config

Bạn có thể dễ dàng gọi biến mà bạn đã cấu hình bằng hàm helper `config` từ mọi nơi trong application của bạn. Giá trị config có thể được gọi thông qua dấu "chấm", nó sẽ chứa tên file config và tên biến mà bạn muốn nhận về. Và bạn cũng có thể tạo một giá trị mặc định, nếu giá trị config đó không tồn tại:

    $value = config('app.timezone');

Để tạo một giá trị config khi đang chạy, bạn có thể truyền một array vào hàm helper `config`:

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## Caching các biến config

Để tăng tốc độ application của bạn, bạn nên dùng cache để cache lại tất cả các file config vào một file duy nhất bằng cách dùng lệnh Artisan `config:cache`. Nó sẽ nối tất các biến đã được config cho application của bạn vào một file duy nhất và nó sẽ load nhanh hơn bởi framework.

Bạn nên chạy lệnh `php artisan config:cache` cho mỗi lần chạy build production. Lệnh này không nên chạy ở dưới local bởi cái các giá trị config thường xuyên bị thay đổi trong quá trình phát triển.

> {note} Nếu bạn chạy lệnh `config:cache` trong quá trình phát triển, thì bạn nên chắc chắn là bạn chỉ gọi hàm `env` trong file config. Vì khi các config đã lưu vào cache thì file `.env` sẽ không được load và vì thế mọi thứ bạn gọi đến hàm `env` này sẽ trả về giá trị null.

<a name="maintenance-mode"></a>
## Chế độ bảo trì

Khi application của bạn đang trong chế độ bảo trì, thì một giao diện bảo trì sẽ được hiển thị cho tất cả các request tới application của bạn. Nó sẽ làm bạn dễ dàng vô hiệu hoá application của bạn trong khi bạn đang cập nhật hoặc đang thực hiện bảo trì. Việc kiểm tra aplication của bạn có đang ở trong chế độ bảo trì hay không sẽ được mặc định thêm vào middleware. Và nếu như application đang ở trong chế độ bảo trì thì một `MaintenanceModeException` sẽ được đưa ra với mã lỗi là 503.

Để bật chế độ bảo trì, hãy chạy lệnh Artisan `down`:

    php artisan down

Bạn cũng có thể cho thêm một `message` và một `retry` vào trong lệnh `down`. Giá trị `message` sẽ là giá trị được dùng để hiển thị message trong màn hình bảo trì, trong khi giá trị `retry` sẽ được set vào giá trị `Retry-After` của HTTP header:

    php artisan down --message="Upgrading Database" --retry=60

Ngay cả khi ở chế độ bảo trì, các địa chỉ IP hoặc các mạng cụ thể có thể được phép truy cập ứng dụng của bạn bằng cách sử dụng tùy chọn `allow`:

    php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16

Để tắt chế độ bảo trì, hãy dùng lệnh `up`:

    php artisan up

> {tip} Bạn cũng có sửa đổi màn hình bảo trì mặc định của Laravel bằng cách tạo thêm màn hình tuỳ biến của bạn vào thư mục có đường dẫn như sau: `resources/views/errors/503.blade.php`.

#### Chế độ bảo trì và Queues

Mỗi khi application của bạn vào trong chế độ bảo trì, thì sẽ không có một [queued jobs](/docs/{{version}}/queues) nào sẽ được chạy. Job sẽ được chạy bình thường cho đến khi nào bạn tắt chế độ bảo trì.

#### Lựa chọn thay thế chế độ bảo trì

Vì chế độ bảo trì sẽ yêu cầu application của ban sẽ ngường hoạt động một khoảng thời gian, nên có một lựa chọn thay thế là dùng [Envoyer](https://envoyer.io) để giúp bạn vừa có thể nâng cấp application của bạn vừa không khiến application của bạn ngừng hoạt động.
