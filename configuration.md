# Cấu hình

- [Giới thiệu](#introduction)
- [Biến môi trường](#environment-configuration)
    - [Nhận về giá trị biến môi trường](#retrieving-environment-configuration)
    - [Xác định môi trường hiện tại](#determining-the-current-environment)
- [Nhận về biến config](#accessing-configuration-values)
- [Caching các biến config](#configuration-caching)
- [Chế độ bảo trì](#maintenance-mode)

<a name="introduction"></a>
## Giới thiệu

Tất cả các file config cho Laravel framework được lưu trữ trong thư mục `config`. Các biến config đều đã được tài liệu hoá bằng comment, vì vậy hãy xem qua chúng và làm quen với chúng.

<a name="environment-configuration"></a>
## Biến môi trường

Nó rất hữu dụng cho những giá trị config khác nhau ở trên nơi mà ứng dụng của bạn đang chạy. Ví dụ, bạn có thể muốn dùng một loại cache khác ở local, hơn là dùng cache đang được dùng ở production server của bạn.

Để làm nó dễ dàng hơn, Laravel sử dụng thư viện PHP [DotEnv](https://github.com/vlucas/phpdotenv) được tạo bởi Vance Lucas. Trong thư mục project gốc của bạn có chứa một file `.env.example`. Nếu  bạn cài đặt project thông qua Laravel installer hoặc Composer. thì file đó sẽ tự động sửa thành tên `.env`. Nếu không thì, bạn nên sửa file đó bằng tay.

File `.env` của bạn không nên được commit vào trong source code của bạn, bởi vì mỗi người phát triển, và server họ đang dùng cho web application của bạn có thể yêu cầu những biến môi trường khác nhau. Hơn nữa, Nó cũng là một rủi ro bảo mật trong trường hợp có kẻ xâm nhập nào đó có quyền truy cập vào source code của bạn, thì tất cả các thông tin đều sẽ bị lộ.

Nếu bạn đang phát triển cùng với một team, bạn cũng có thể thêm một file `.env.example` vào trong project của bạn, bằng cách thêm cái giá trị ví dụ vào file `.env.example`, thì các người phát triển khác sẽ hiểu rõ ràng hơn các biến môi trường cần được cài đặt để chạy application của bạn, Bạn cũng có thể tạo một file `.env.testing`. File này sẽ bị đè vào file `.env` khi chạy PHPUnit để test hoặc khi chạy lệnh Artisan với lựa chọn là `--env=testing`.

> {tip} Tất cả các biến trong file `.env` có thể bị ghi đè bởi biến môi trường bên ngoài như là biến môi trường server hoặc system.

<a name="retrieving-environment-configuration"></a>
### Nhận về giá trị biến môi trường

Tất cả các biến môi trường sẽ đều được load vào `$_ENV` PHP super-global khi mà ứng dụng của bạn nhân được một request. Tuy nhiên, bạn cũng có thể dùng hàm `env` trong helper để nhận về các biến môi trường. Trong thực tế, nếu bạn nhìn vào file config của Laravel, bạn sẽ nhận thấy rằng có một số config đang dùng helper ở trên:

    'debug' => env('APP_DEBUG', false),

Parameter thứ hai được truyền vào hàm `env` là giá trị mặc định. Giá trị đó sẽ được trả về nếu như biến môi trường bạn muốn trả về không tồn tại.

<a name="determining-the-current-environment"></a>
### Xác định môi trường hiện tại

Môi trường hiện tại có thể được xác định thông biến `APP_ENV` trong file `.env` của bạn. Bạn có thể lấy giá trị đó thông qua hàm `environment` trong [facade](/docs/{{version}}/facades) `App`:

    $environment = App::environment();

Bạn cũng có thể truyền vào hàm `environment` tên của một môi trường nào đó để kiểm tra nó có đúng hay không. Hàm đó sẽ trả về `true` nếu tên môi trường trùng với tên môi trường đang được định nghĩa trong file `.env` hiện tại.

    if (App::environment('local')) {
        // Môi trường hiện tại là local
    }

    if (App::environment(['local', 'staging'])) {
        // Môi trường hiện tại có thể là local hoặc staging
    }

> {tip} Môi trường hiện tại của application của bạn có thể bị ghi đè bởi một biến môi trường `APP_ENV` khác ở mức độ server. Nó sẽ hữu ích khi mà bạn cần chia sẻ một application cho config ở môi trường khác, vì vậy bạn có cài đặt một host với môi trường bạn mong muốn trong config server của bạn.

<a name="accessing-configuration-values"></a>
## Nhận về biến config

Bạn có thể dễ dàng gọi biến mà bạn đã config bằng hàm helper `config` từ mọi nơi trong application của bạn. Giá trị config có thể được gọi thông qua dấu "chấm", nó sẽ chứa tên file config và tên biến mà bạn muốn nhận về. Và bạn cũng có thể cài đặt một giá trị mặc định, nếu giá trị config đó không tồn tại:

    $value = config('app.timezone');

Để set giá trị config khi đang chạy, thì bạn có thể set một array vào hàm helper `config`:

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## Caching các biến config

Để tăng tốc độ application của bạn, bạn nên dùng cache để cache lại tất cả các file config vào một file duy nhất bằng cách dùng lệnh Artisan `config:cache`. Nó sẽ nối tất các biến đã được config cho application vào một file duy nhất và nó sẽ load nhanh hơn bởi framework.

Bạn nên chạy lệnh `php artisan config:cache` cho mỗi lần chạy build production. Lệnh này không nên chạy ở dưới local bởi cái các giá trị config thường xuyên bị thay đổi trong quá trình phát triển.

> {note} Nếu bạn chạy lệnh `config:cache` trong quá trình phát triển, thì bạn nên chắc chắn rằng bạn chỉ gọi được hàm `env` trong file config. Khi các config đã lưu vào cache thì file `.env` sẽ không được load và vì thế mọi thứ bạn gọi đến hàm `env` này sẽ trả về giá trị null.

<a name="maintenance-mode"></a>
## Chế độ bảo trì

Khi mà application của ban đang trong chế độ bảo trì, thì một giao diện bảo trì sẽ được hiển thị cho tất cả các request tới application của bạn. Nó sẽ làm bạn dễ dàng vô hiệu hoá application của bạn trong khi bạn đang cập nhật hoặc đang thực hiện bảo trì. Việc kiểm tra aplication của bạn có ở trong chế độ bảo trì không sẽ được mặc định thêm vào middleware. Và nếu như application đang ở trong chế độ bảo trì thì một `MaintenanceModeException` sẽ quăng ra với mã lỗi là 503.

Để bật chế độ bảo trì, hãy chạy lệnh Artisan `down`:

    php artisan down

Bạn cũng có thể cho thêm một `message` và một `retry` vào trong lệnh `down`. Giá trị `message` sẽ là giá trị được dùng để hiển thị message trong giao diện bảo trì, trong khi giá trị `retry` sẽ được set vào giá trị `Retry-After` của HTTP header:

    php artisan down --message="Upgrading Database" --retry=60

Để tắt chế độ bảo trì, hãy dùng lệnh `up`:

    php artisan up

> {tip} Bạn cũng có sửa đổi giao diện bảo trì mặc định cảu Laravel bằng cách tạo thêm giao diện tuỳ chỉnh của bạn vào thư mục có đường dẫn như sau: `resources/views/errors/503.blade.php`.

#### Chế độ bảo trì và Queues

Mỗi khi application của bạn trong chế độ bảo trì, thì sẽ không có một [queued jobs](/docs/{{version}}/queues) được chạy. Job sẽ được chạy bình thường cho đến khi nào bạn tắt chế dộ bảo trì.

#### Lựa chọn thay thế chế độ bảo trì

Vì chế độ bảo trì sẽ yêu cầu application của ban sẽ ngường hoạt động một khoảng thời gian, nên có một lựa chọn thay thế là [Envoyer](https://envoyer.io) để giúp bạn vừa có thể nâng cấp application của bạn vừa không khiến application của bạn ngừng hoạt động.
