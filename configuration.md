# Cấu hình

- [Giới thiệu](#introduction)
- [Biến môi trường](#environment-configuration)
    - [Loại biến môi trường](#environment-variable-types)
    - [Nhận về giá trị biến môi trường](#retrieving-environment-configuration)
    - [Xác định môi trường hiện tại](#determining-the-current-environment)
- [Nhận về biến config](#accessing-configuration-values)
- [Caching các biến config](#configuration-caching)
- [Chế độ debug](#debug-mode)
- [Chế độ bảo trì](#maintenance-mode)

<a name="introduction"></a>
## Giới thiệu

Tất cả các file config cho Laravel framework được lưu trữ trong thư mục `config`. Các biến config đều đã được tài liệu hoá bằng comment, nên hãy xem qua và làm quen với chúng.

Các file cấu hình này cho phép bạn cấu hình những thứ như thông tin kết nối cơ sở dữ liệu, thông tin mail của bạn, cũng như nhiều giá trị cấu hình cốt lõi khác, chẳng hạn như múi giờ ứng dụng và key mã hóa.

<a name="environment-configuration"></a>
## Biến môi trường

Nó rất hữu dụng cho những giá trị config khác nhau dựa trên môi trường, nơi mà ứng dụng của bạn đang được chạy. Ví dụ, bạn có thể muốn dùng một loại cache riêng ở local, hơn là dùng loại cache mà đang được sử dụng ở trên server production.

Để dễ dàng hơn, Laravel sử dụng thư viện PHP [DotEnv](https://github.com/vlucas/phpdotenv). Trong thư mục project gốc của bạn có chứa một file `.env.example` sẽ định nghĩa nhiều biến môi trường phổ biến. Trong quá trình cài đặt Laravel, file này sẽ tự động được sao chép vào file `.env`.

File `.env` mặc định của Laravel có chứa một số giá trị cấu hình phổ biến có thể khác nhau tùy thuộc vào việc ứng dụng của bạn đang chạy ở local hay trên máy chủ web production. Các giá trị này sau đó sẽ được lấy ra từ các file cấu hình Laravel khác nhau trong thư mục `config` bằng phương thức `env` của Laravel.

Nếu bạn đang phát triển cùng với một team, bạn nên thêm file `.env.example` vào trong project của bạn, sau đó, thêm cái giá trị ví dụ vào trong file `.env.example`, các nhà phát triển tiếp theo sẽ hiểu rõ ràng hơn về các biến môi trường cần được cài đặt để chạy application của bạn.

> {tip} Tất cả các biến trong file `.env` có thể bị ghi đè bởi biến môi trường bên ngoài như là biến môi trường server hoặc system.

<a name="environment-file-security"></a>
#### Environment File Security

Your `.env` file should not be com
File `.env` của bạn không nên được commit vào source code của ứng dụng của bạn, vì mỗi nhà phát triển hoặc server của bạn sẽ sử dụng ứng dụng của bạn với mỗi một yêu cầu cấu hình môi trường khác nhau. Hơn nữa, đây sẽ là một rủi ro bảo mật trong trường hợp kẻ xâm nhập giành được quyền truy cập vào repository của bạn, vì mọi thông tin đăng nhập nhạy cảm sẽ bị lộ.

<a name="additional-environment-files"></a>
#### Additional Environment Files

Trước khi load các biến môi trường của ứng dụng của bạn, Laravel sẽ xác định xem biến môi trường `APP_ENV` có được cung cấp hay không hoặc tham số `--env` CLI có được chỉ định hay không. Nếu đã được chỉ định, Laravel sẽ thử load một file `.env.[APP_ENV]` nếu nó tồn tại. Nếu nó không tồn tại, file `.env` mặc định sẽ được load.

<a name="environment-variable-types"></a>
### Loại biến môi trường

Tất cả các biến trong file `.env` của bạn thường được nhận dạng là dưới dạng kiểu string, vì vậy có một số giá trị đã được tạo để cho phép bạn trả về nhiều kiểu hơn từ hàm `env()`:

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

Nếu bạn cần định nghĩa một biến môi trường có chứa khoảng trắng, bạn có thể làm như vậy bằng cách đặt giá trị đó vào trong dấu ngoặc kép:

    APP_NAME="My Application"

<a name="retrieving-environment-configuration"></a>
### Nhận về giá trị biến môi trường

Khi ứng dụng của bạn nhận một request, thì tất cả các biến môi trường sẽ đều được load vào `$_ENV` PHP super-global. Tuy nhiên, bạn cũng có thể dùng hàm `env` trong helper để nhận về các biến môi trường. Trong thực tế, nếu bạn nhìn vào file config của Laravel, bạn sẽ nhận thấy có nhiều config đang được dùng helper trên:

    'debug' => env('APP_DEBUG', false),

Tham số thứ hai được truyền vào trong hàm `env` là giá trị mặc định. Giá trị này sẽ được trả về nếu như biến môi trường của bạn không tồn tại.

<a name="determining-the-current-environment"></a>
### Xác định môi trường hiện tại

Môi trường hiện tại có thể được xác định thông biến `APP_ENV` trong file `.env`. Bạn có thể lấy giá trị đó thông qua hàm `environment` trong [facade](/docs/{{version}}/facades) `App`:

    use Illuminate\Support\Facades\App;

    $environment = App::environment();

Bạn cũng có thể truyền vào hàm `environment` tên của một môi trường để xác định xem môi trường hiện tại có đúng là môi trường đó hay không. Hàm đó sẽ trả về `true` nếu tên môi trường trùng với tên môi trường đang được định nghĩa trong file `.env` hiện tại.

    if (App::environment('local')) {
        // Môi trường hiện tại là local
    }

    if (App::environment(['local', 'staging'])) {
        // Môi trường hiện tại có thể là local hoặc staging
    }

> {tip} Môi trường hiện tại của application có thể bị ghi đè bởi một biến môi trường `APP_ENV` khác ở mức độ server.

<a name="accessing-configuration-values"></a>
## Nhận về biến config

Bạn có thể dễ dàng gọi biến mà bạn đã cấu hình bằng hàm helper `config` từ mọi nơi trong application của bạn. Giá trị config có thể được gọi thông qua dấu "chấm", nó sẽ chứa tên file config và tên biến mà bạn muốn nhận về. Và bạn cũng có thể tạo một giá trị mặc định, nếu giá trị config đó không tồn tại:

    $value = config('app.timezone');

    // Retrieve a default value if the configuration value does not exist...
    $value = config('app.timezone', 'Asia/Seoul');

Để tạo một giá trị config khi đang chạy, bạn có thể truyền một array vào hàm helper `config`:

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## Caching các biến config

Để tăng tốc độ application của bạn, bạn nên dùng cache để cache lại tất cả các file config vào một file duy nhất bằng cách dùng lệnh Artisan `config:cache`. Nó sẽ nối tất các biến đã được config cho application của bạn vào một file duy nhất can be quickly loaded by the framework.

Thông thường, bạn nên chạy lệnh `php artisan config:cache` như một phần của quy trình deploy production. Không nên chạy lệnh này trong quá trình phát triển local vì các tùy chọn cấu hình này sẽ thường xuyên phải thay đổi trong quá trình phát triển ứng dụng của bạn.

> {note} Nếu bạn chạy lệnh `config:cache` trong quá trình phát triển của bạn, bạn nên đảm bảo là bạn chỉ gọi hàm `env` ở trong các file cấu hình của bạn. Sau khi cấu hình đã được lưu vào bộ nhớ cache, file `.env` sẽ không được load; và do đó, hàm `env` sẽ chỉ trả về các biến môi trường ở cấp độ hệ thống hoặc bên ngoài.

<a name="debug-mode"></a>
## Chế độ debug

Tùy chọn `debug` trong file cấu hình `config/app.php` của bạn sẽ xác định lượng thông tin sẽ được hiển thị cho người dùng. Mặc định, tùy chọn này được set trong giá trị của biến môi trường `APP_DEBUG`, và được lưu trong file `.env` của bạn.

Để phát triển local, bạn nên set biến môi trường `APP_DEBUG` thành `true`. **Trong môi trường production của bạn, giá trị này phải luôn là `false`. Nếu biến này được set thành `true`, thì bạn có nguy cơ để lộ các giá trị cấu hình nhạy cảm cho người dùng của ứng dụng của bạn biết.**

<a name="maintenance-mode"></a>
## Chế độ bảo trì

Khi application của bạn đang trong chế độ bảo trì, thì một giao diện bảo trì sẽ được hiển thị cho tất cả các request tới application của bạn. Nó sẽ làm bạn dễ dàng vô hiệu hoá application của bạn trong khi bạn đang cập nhật hoặc đang thực hiện bảo trì. Việc kiểm tra aplication của bạn có đang ở trong chế độ bảo trì hay không sẽ được mặc định thêm vào middleware. Và nếu như application đang ở trong chế độ bảo trì thì một instance `Symfony\Component\HttpKernel\Exception\HttpException` sẽ được đưa ra với mã lỗi là 503.

Để bật chế độ bảo trì, hãy chạy lệnh Artisan `down`:

    php artisan down

Nếu bạn muốn thêm header HTTP `Refresh` được gửi cùng với tất cả các response khi ở trong chế độ bảo trì, thì bạn có thể cung cấp tùy chọn `refresh` khi gọi lệnh `down`. Header `Refresh` sẽ hướng dẫn trình duyệt tự động refresh trang sau số giây đã được chỉ định:

    php artisan down --refresh=15

Bạn cũng có thể cung cấp tùy chọn `retry` cho lệnh `down`, tùy chọn này sẽ được set làm giá trị của header HTTP `Retry-After`, mặc dù các trình duyệt thường bỏ qua header này:

    php artisan down --retry=60

<a name="bypassing-maintenance-mode"></a>
#### Bypassing Maintenance Mode

Ngay cả khi ở chế độ bảo trì, thì bạn có thể sử dụng tùy chọn `secret` để chỉ định một mã token để bỏ qua chế độ bảo trì:

    php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"

Sau khi set ứng dụng ở chế độ bảo trì, bạn có thể điều hướng đến URL của ứng dụng với một token và Laravel sẽ cấp cookie bỏ qua chế độ bảo trì cho trình duyệt của bạn:

    https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515

Khi truy cập vào route ẩn này, bạn sẽ được chuyển hướng đến route `/` của ứng dụng. Khi cookie đã được cấp cho trình duyệt của bạn, bạn sẽ có thể xem ứng dụng bình thường như thể nó không đang ở trong chế độ bảo trì.

> {tip} Secret trong chế độ bảo trì của bạn sẽ thường phải chứa các ký tự chữ và số và các dấu gạch ngang. Bạn nên tránh sử dụng các ký tự có ý nghĩa đặc biệt trong URL, chẳng hạn như `?`.

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### Pre-Rendering The Maintenance Mode View

Nếu bạn sử dụng lệnh `php artisan down` trong khi deploy, người dùng của bạn đôi khi vẫn có thể gặp lỗi nếu họ truy cập ứng dụng trong khi các library của Composer hoặc các thành phần cơ sở hạ tầng khác của bạn đang cập nhật. Điều này xảy ra vì một phần quan trọng của Laravel framework phải khởi động để xác định ứng dụng của bạn có đang ở trong chế độ bảo trì hay không và hiển thị view chế độ bảo trì bằng cách sử dụng công cụ tạo template.

Vì lý do này, Laravel cho phép bạn tạo trước các view khi ở trong chế độ bảo trì và sẽ được trả về ngay khi bắt đầu chu trình của request. View này sẽ được hiển thị trước khi bất kỳ library nào của ứng dụng của bạn được load. Bạn có thể tạo trước một template của bạn chọn bằng cách sử dụng tùy chọn `render` trong lệnh `down`:

    php artisan down --render="errors::503"

<a name="redirecting-maintenance-mode-requests"></a>
#### Redirecting Maintenance Mode Requests

Khi ở trong chế độ bảo trì, Laravel sẽ hiển thị view của chế độ bảo trì cho tất cả các URL ứng dụng mà người dùng cố gắng truy cập vào. Và nếu muốn, bạn có thể hướng dẫn Laravel chuyển hướng tất cả các request đến một URL cụ thể. Điều này có thể được thực hiện bằng cách sử dụng tùy chọn `redirect`. Ví dụ: bạn có thể muốn chuyển hướng tất cả các request tới `/` URI:

    php artisan down --redirect=/

<a name="disabling-maintenance-mode"></a>
#### Disabling Maintenance Mode

Để tắt chế độ bảo trì, hãy dùng lệnh `up`:

    php artisan up

> {tip} Bạn cũng có sửa đổi màn hình bảo trì mặc định của Laravel bằng cách tạo thêm màn hình tuỳ biến của bạn vào thư mục có đường dẫn như sau: `resources/views/errors/503.blade.php`.

<a name="maintenance-mode-queues"></a>
#### Chế độ bảo trì và Queues

Mỗi khi application của bạn vào trong chế độ bảo trì, thì sẽ không có một [queued jobs](/docs/{{version}}/queues) nào sẽ được chạy. Job sẽ được chạy bình thường cho đến khi nào bạn tắt chế độ bảo trì.

<a name="alternatives-to-maintenance-mode"></a>
#### Lựa chọn thay thế chế độ bảo trì

Vì chế độ bảo trì sẽ yêu cầu application của ban sẽ ngường hoạt động một khoảng thời gian, nên có một lựa chọn thay thế là dùng [Laravel Vapor](https://vapor.laravel.com) và [Envoyer](https://envoyer.io) để giúp bạn vừa có thể nâng cấp application của bạn vừa không khiến application của bạn ngừng hoạt động.
