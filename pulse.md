# Laravel Pulse

- [Giới thiệu](#introduction)
- [Cài đăt](#installation)
    - [Cấu hình](#configuration)
- [Bảng điều khiển](#dashboard)
    - [Quyền](#dashboard-authorization)
    - [Tuỳ chỉnh](#dashboard-customization)
    - [Lấy thông tin User](#dashboard-resolving-users)
    - [Cards](#dashboard-cards)
- [Ghi lại mục](#capturing-entries)
    - [Recorders](#recorders)
    - [Filtering](#filtering)
- [Hiệu suất](#performance)
    - [Dùng một database khác](#using-a-different-database)
    - [Tích hợp Redis](#ingest)
    - [Lấy mẫu](#sampling)
    - [Cắt bớt](#trimming)
    - [Xử lý Pulse Exception](#pulse-exceptions)
- [Tuỳ chỉnh Cards](#custom-cards)
    - [Card Components](#custom-card-components)
    - [Styling](#custom-card-styling)
    - [Thu thập và tổng hợp dữ liệu](#custom-card-data)

<a name="introduction"></a>
## Giới thiệu

[Laravel Pulse](https://github.com/laravel/pulse) cung cấp cho bạn một cái nhìn tổng quan về hiệu suất và mức độ sử dụng trong ứng dụng của bạn. Với Pulse, bạn có thể theo dõi các slow job và request, tìm ra người dùng hoạt động tích cực nhất, vv...

Để debug chuyên sâu hơn cho từng event, hãy xem [Laravel Telescope](/docs/{{version}}/telescope).

<a name="installation"></a>
## Cài đăt

> [!WARNING]
> Hiện tại việc lưu trữ của Pulse sẽ yêu cầu cơ sở dữ liệu MySQL hoặc PostgreSQL. Nếu bạn đang sử dụng một cơ sở dữ liệu khác, bạn sẽ cần một cơ sở dữ liệu MySQL hoặc PostgreSQL riêng cho việc lưu trữ dữ liệu Pulse của bạn.

Vì Pulse hiện đang ở giai đoạn thử nghiệm nên bạn có thể cần điều chỉnh file `composer.json` của ứng dụng để cho phép cài đặt các bản phát hành beta:

```json
"minimum-stability": "beta",
"prefer-stable": true
```

Sau đó, bạn có thể sử dụng trình quản lý package Composer để cài đặt Pulse vào dự án Laravel của bạn:

```sh
composer require laravel/pulse
```

Tiếp theo, bạn nên export ra các file cấu hình và migration Pulse bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
```

Cuối cùng, bạn chạy lệnh `migrate` để tạo ra các bảng cần thiết để lưu trữ dữ liệu của Pulse:

```shell
php artisan migrate
```

Sau khi migration cơ sở dữ liệu của Pulse hoàn tất, bạn có thể truy cập vào bảng điều khiển của Pulse thông qua route `/pulse`.

> [!NOTE]
> Nếu bạn không muốn lưu dữ liệu Pulse trong cơ sở dữ liệu chính của ứng dụng, bạn có thể [chỉ định một kết nối cơ sở dữ liệu khác](#using-a-different-database).

<a name="configuration"></a>
### Cấu hình

Nhiều tùy chọn cấu hình của Pulse có thể được kiểm soát bằng các biến môi trường. Để xem các tùy chọn có sẵn, đăng ký recorder mới hoặc cấu hình các tùy chọn nâng cao, bạn có thể export file cấu hình `config/pulse.php`:

```sh
php artisan vendor:publish --tag=pulse-config
```

<a name="dashboard"></a>
## Bảng điều khiển

<a name="dashboard-authorization"></a>
### Quyền

Bảng điều khiển của Pulse có thể được truy cập thông qua đường dẫn `/pulse`. Mặc định, bạn chỉ có thể truy cập vào bảng điều khiển này trong môi trường `local`, vì vậy bạn sẽ cần cấu hình quyền cho môi trường production của bạn bằng cách tùy chỉnh authorization gate `'viewPulse'`. Bạn có thể thực hiện việc này trong file `app/Providers/AuthServiceProvider.php` của ứng dụng:

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Register any authentication / authorization services.
 */
public function boot(): void
{
    Gate::define('viewPulse', function (User $user) {
        return $user->isAdmin();
    });

    // ...
}
```

<a name="dashboard-customization"></a>
### Tuỳ chỉnh

Các card và layout của bảng điều khiển Pulse có thể được cấu hình bằng cách export ra các view. Các view sẽ được export vào `resources/views/vendor/pulse/dashboard.blade.php`:

```sh
php artisan vendor:publish --tag=pulse-dashboard
```

Bảng điều khiển của Pulse được tạo bởi [Livewire](https://livewire.laravel.com/) và cho phép bạn tùy chỉnh các card và layout mà không cần phải xây dựng lại bất kỳ nội dung JavaScript nào.

Trong file này, component `<x-pulse>` chịu trách nhiệm hiển thị bảng điều khiển và cung cấp layout dưới dạng lưới cho các card. Nếu bạn muốn bảng điều khiển trải dài trên toàn bộ chiều rộng của màn hình, bạn có thể cung cấp thuộc tính `full-width` cho component:

```blade
<x-pulse full-width>
    ...
</x-pulse>
```

Mặc định, component `<x-pulse>` sẽ tạo dạng lưới 12 cột, nhưng bạn có thể tùy chỉnh hành động này bằng cách sử dụng thuộc tính `cols`:

```blade
<x-pulse cols="16">
    ...
</x-pulse>
```

Mỗi card sẽ chấp nhận thuộc tính `cols` và `rows` để kiểm soát khoảng cách và vị trí:

```blade
<livewire:pulse.usage cols="4" rows="2" />
```

Hầu hết các card cũng chấp nhận thuộc tính `expand` để hiển thị toàn bộ card thay vì phải scroll:

```blade
<livewire:pulse.slow-queries expand />
```

<a name="dashboard-resolving-users"></a>
### Lấy thông tin User

Đối với các card hiển thị thông tin về người dùng, chẳng hạn như card mức độ sử dụng application, Pulse sẽ chỉ ghi lại ID của người dùng. Khi bảng điều khiển được hiển thị, Pulse mới bắt đầu gọi các field `name` và `email` từ model `Authenticatable` của bạn và hiển thị ảnh đại diện bằng dịch vụ web Gravatar.

Bạn có thể tùy chỉnh các field và ảnh đại diện bằng cách gọi phương thức `Pulse::user` trong class `App\Providers\AppServiceProvider` của ứng dụng.

Phương thức `user` sẽ chấp nhận một closure nhận vào model `Authenticatable` để hiển thị và sẽ trả về một mảng chứa các thông tin `name`, `extra` và `avatar` của người dùng:

```php
use Laravel\Pulse\Facades\Pulse;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::user(fn ($user) => [
        'name' => $user->name,
        'extra' => $user->email,
        'avatar' => $user->avatar_url,
    ]);

    // ...
}
```

> [!NOTE]
> Bạn có thể hoàn toàn tùy chỉnh cách người dùng được ghi lại và lấy ra bằng cách implement contract `Laravel\Pulse\Contracts\ResolvesUsers` và liên kết nó trong [service container](/docs/{{version}}/container#binding-a-singleton) của Laravel.

<a name="dashboard-cards"></a>
### Cards

<a name="servers-card"></a>
#### Servers

Card `<livewire:pulse.servers />` sẽ hiển thị mức độ sử dụng tài nguyên của tất cả các server đang chạy lệnh `pulse:check`. Vui lòng tham khảo tài liệu liên quan đến [servers recorder](#servers-recorder) để biết thêm thông tin về báo cáo tài nguyên hệ thống.

<a name="application-usage-card"></a>
#### Application Usage

Card `<livewire:pulse.usage />` sẽ hiển thị 10 người dùng đưa ra request nhiều nhất tới ứng dụng của bạn, gửi job và tình trạng request chậm.

Nếu bạn muốn xem tất cả các số liệu sử dụng trên vào trong cùng một màn hình, bạn có thể đưa nhiều card vào với thuộc tính `type` khác nhau:

```blade
<livewire:pulse.usage type="requests" />
<livewire:pulse.usage type="slow_requests" />
<livewire:pulse.usage type="jobs" />
```

Để tìm hiểu cách tùy chỉnh cách Pulse lấy và hiển thị thông tin người dùng, hãy tham khảo tài liệu của chúng tôi về [lấy thông tin người dùng](#dashboard-resolving-users).

> [!NOTE]
> Nếu ứng dụng của bạn nhận được nhiều request hoặc gửi nhiều job, bạn có thể muốn bật chế độ [lấy mẫu](#sampling). Xem tài liệu [user requests recorder](#user-requests-recorder), [user jobs recorder](#user-jobs-recorder) và [slow jobs recorder](#slow-jobs-recorder) để biết thêm thông tin chi tiết.

<a name="exceptions-card"></a>
#### Exceptions

Card `<livewire:pulse.exceptions />` sẽ hiển thị tần suất và mức độ xảy ra gần đây của các exception trong ứng dụng của bạn. Mặc định, các exception sẽ được nhóm theo class exception và vị trí xảy ra. Xem tài liệu [exceptions recorder](#exceptions-recorder) để biết thêm thông tin chi tiết.

<a name="queues-card"></a>
#### Queues

Card `<livewire:pulse.queues />` sẽ hiển thị thông lượng của các queue trong ứng dụng của bạn, bao gồm số lượng job queued, đang được xử lý, đã được xử lý, đã được release và không thành công. Xem tài liệu [queues recorder](#queues-recorder) để biết thêm thông tin chi tiết.

<a name="slow-requests-card"></a>
#### Slow Requests

Card `<livewire:pulse.slow-requests />` sẽ hiển thị các request đến ứng dụng của bạn mà chạy quá thời gian đã cấu hình, mặc định là 1000ms. Xem tài liệu [slow requests recorder](#slow-requests-recorder) để biết thêm thông tin chi tiết.

<a name="slow-jobs-card"></a>
#### Slow Jobs

Card `<livewire:pulse.slow-jobs />` sẽ hiển thị các queued job trong ứng dụng của bạn mà chạy quá thời gian đã cấu hình, mặc định là 1000ms. Xem tài liệu [slow jobs recorder](#slow-jobs-recorder) để biết thêm thông tin chi tiết.

<a name="slow-queries-card"></a>
#### Slow Queries

Card `<livewire:pulse.slow-queries />` sẽ hiển thị các truy vấn cơ sở dữ liệu trong ứng dụng của bạn mà chạy quá thời gian đã cấu hình, mặc định là 1000ms.

Mặc định, các truy vấn chậm sẽ được nhóm dựa trên truy vấn SQL (không có ràng buộc) và vị trí xảy ra truy vấn, nhưng bạn có thể chọn không ghi lại vị trí của truy vấn nếu bạn chỉ muốn nhóm theo truy vấn SQL.

Xem tài liệu [slow queries recorder](#slow-queries-recorder) để biết thêm thông tin chi tiết.

<a name="slow-outgoing-requests-card"></a>
#### Slow Outgoing Requests

Card `<livewire:pulse.slow-outgoing-requests />` sẽ hiển thị các request gửi đi từ ứng dụng được thực hiện bằng [HTTP client](/docs/{{version}}/http-client) của Laravel mà chạy quá thời gian đã cấu hình, mặc định là 1000ms.

Mặc định, các mục sẽ được nhóm theo URL. Tuy nhiên, bạn có thể muốn chuẩn hóa hoặc nhóm các request gửi đi tương tự nhau bằng biểu thức chính quy. Xem tài liệu [slow outgoing requests recorder](#slow-outgoing-requests-recorder) để biết thêm thông tin chi tiết.

<a name="cache-card"></a>
#### Cache

Card `<livewire:pulse.cache />` sẽ hiển thị số liệu thống kê về cache hit và cache miss cho ứng dụng của bạn, cả trên global và cho từng khóa riêng lẻ.

Mặc định, các mục sẽ được nhóm theo khóa. Tuy nhiên, bạn có thể muốn chuẩn hóa hoặc nhóm các khóa tương tự nhau bằng biểu thức chính quy. Xem tài liệu [cache interactions recorder](#cache-interactions-recorder) để biết thêm thông tin chi tiết.

<a name="capturing-entries"></a>
## Ghi lại mục

Hầu hết các Pulse recorder sẽ tự động ghi lại các mục dựa trên các event framework được Laravel gửi. Tuy nhiên, [servers recorder](#servers-recorder) và một số card khác của bên thứ ba thì bạn cần phải thường xuyên lấy thông tin. Để sử dụng các card này, bạn phải chạy daemon `pulse:check` trên tất cả các server ứng dụng của bạn:

```php
php artisan pulse:check
```

> [!NOTE]
> Để duy trì process `pulse:check` chạy liên tục ở chế độ background, bạn nên sử dụng một trình giám sát process như Supervisor để đảm bảo lệnh này sẽ không bị dừng.

<a name="recorders"></a>
### Recorders

Recorder sẽ chịu trách nhiệm ghi lại các mục từ ứng dụng của bạn rồi ghi vào cơ sở dữ liệu Pulse. Recorder sẽ được đăng ký và cấu hình trong phần `recorders` của [file cấu hình Pulse](#configuration).

<a name="cache-interactions-recorder"></a>
#### Cache Interactions

Recorder `CacheInteractions` ghi lại thông tin về các lần [cache](/docs/{{version}}/cache) hit và cache miss xảy ra trong ứng dụng của bạn để hiển thị trên card [Cache](#cache-card).

Bạn có thể tùy ý điều chỉnh [tỷ lệ lấy mẫu](#sampling) và bỏ qua các key patterns.

Bạn cũng có thể cấu hình cách nhóm để các khóa tương tự nhau được nhóm thành một mục duy nhất. Ví dụ: bạn có thể muốn xóa ID ra khỏi các khóa đã lưu nếu cùng loại thông tin. Các nhóm được cấu hình bằng cách sử dụng biểu thức chính quy để "tìm và thay thế" các phần của khóa. Một ví dụ đã được thêm trong file cấu hình sau:

```php
Recorders\CacheInteractions::class => [
    // ...
    'groups' => [
        // '/:\d+/' => ':*',
    ],
],
```

Pattern nào đầu tiên mà khớp thì sẽ được sử dụng. Nếu không có pattern nào khớp, thì khóa sẽ được giữ nguyên.

<a name="exceptions-recorder"></a>
#### Exceptions

Recorder `Exceptions` sẽ ghi lại thông tin về các exception xảy ra trong ứng dụng của bạn để hiển thị trên card [Exceptions](#exceptions-card).

Bạn có thể tùy ý điều chỉnh [tỷ lệ lấy mẫu](#sampling) và các pattern exception bị bỏ qua. Bạn cũng có thể cấu hình việc ghi lại vị trí phát sinh exception. Vị trí đã ghi lại sẽ được hiển thị trên bảng điều khiển Pulse, giúp bạn theo dõi nguồn gốc của exception; tuy nhiên, nếu cùng một exception xảy ra ở nhiều vị trí, nó sẽ xuất hiện nhiều lần cho mỗi vị trí đó.

<a name="queues-recorder"></a>
#### Queues

Recorder `Queues` sẽ ghi lại thông tin về queue ứng dụng của bạn để hiển thị trên [Queues](#queues-card).

Bạn có thể tùy ý điều chỉnh [tỷ lệ lấy mẫu](#sampling) và các jobs pattern sẽ bị bỏ qua.

<a name="slow-jobs-recorder"></a>
#### Slow Jobs

Recorder `SlowJobs` sẽ ghi lại thông tin về các slow job xảy ra trong ứng dụng của bạn để hiển thị trên card [Slow Jobs](#slow-jobs-recorder).

Bạn có thể tùy ý điều chỉnh ngưỡng mà job sẽ được coi là slow job, [tỷ lệ lấy mẫu](#sampling), và các jobs pattern sẽ bị bỏ qua.

<a name="slow-outgoing-requests-recorder"></a>
#### Slow Outgoing Requests

Recorder `SlowOutgoingRequests` sẽ ghi lại thông tin về các request HTTP gửi đi được thực hiện bằng [HTTP client](/docs/{{version}}/http-client) của Laravel mà chạy quá thời gian đã cấu hình để hiển thị trên card [Slow Outgoing Requests](#slow-outgoing-requests-card).

Bạn có thể tùy ý điều chỉnh ngưỡng mà request gửi đi sẽ bị coi là chậm, [tỷ lệ lấy mẫu](#sampling) và các pattern URL sẽ bị bỏ qua.

Bạn cũng có thể cấu hình nhóm của URL để các URL tương tự nhau được nhóm thành một mục duy nhất. Ví dụ: bạn có thể muốn xóa ID ra khỏi đường dẫn URL hoặc chỉ nhóm theo tên miền. Các nhóm được cấu hình bằng cách sử dụng biểu thức chính quy để "tìm và thay thế" các phần của URL. Một số ví dụ được thêm vào trong file cấu hình sau:

```php
Recorders\OutgoingRequests::class => [
    // ...
    'groups' => [
        // '#^https://api\.github\.com/repos/.*$#' => 'api.github.com/repos/*',
        // '#^https?://([^/]*).*$#' => '\1',
        // '#/\d+#' => '/*',
    ],
],
```

Pattern nào đầu tiên mà khớp thì sẽ được sử dụng. Nếu không có pattern nào khớp, thì URL sẽ được giữ nguyên.

<a name="slow-queries-recorder"></a>
#### Slow Queries

Recorder `SlowQueries` sẽ ghi lại mọi truy vấn cơ sở dữ liệu trong ứng dụng của bạn mà chạy quá thời gian đã cấu hình để hiển thị trên card [Slow Queries](#slow-queries-card).

Bạn có thể tùy chọn điều chỉnh ngưỡng mà truy vấn sẽ bị coi là chậm, [tỷ lệ lấy mẫu](#sampling) và các pattern truy vấn sẽ bị bỏ qua. Bạn cũng có thể cấu hình việc có nên ghi lại vị trí truy vấn hay không. Vị trí đã ghi lại sẽ được hiển thị trên bảng điều khiển Pulse, giúp bạn theo dõi nguồn gốc của truy vấn; tuy nhiên, nếu cùng một truy vấn được thực hiện ở nhiều vị trí khác nhau, nó sẽ xuất hiện nhiều lần cho mỗi vị trí đó.

<a name="slow-requests-recorder"></a>
#### Slow Requests

Recorder `Requests` sẽ ghi lại thông tin về các request được gửi đến ứng dụng của bạn để hiển thị trên card [Slow Requests](#slow-requests-card) và card [Application Usage](#application-usage-card).

Bạn có thể tùy ý điều chỉnh ngưỡng mà route sẽ bị coi là bị chậm, [tỷ lệ lấy mẫu](#sampling) và các path sẽ bị bỏ qua.

<a name="servers-recorder"></a>
#### Servers

Recorder `Servers` sẽ ghi lại mức độ sử dụng CPU, bộ nhớ và kho lưu trữ của các server chạy cho ứng dụng của bạn để hiển thị trên card [Servers](#servers-card). Recorder này yêu cầu [lệnh `pulse:check`](#capturing-entries) phải được chạy trên mỗi server mà bạn muốn giám sát.

Mỗi server phải có một tên duy nhất. Mặc định, Pulse sẽ sử dụng giá trị được trả về bởi hàm `gethostname` của PHP. Nếu bạn muốn tùy chỉnh, bạn có thể set biến môi trường `PULSE_SERVER_NAME`:

```env
PULSE_SERVER_NAME=load-balancer
```

File cấu hình Pulse cũng cho phép bạn tùy chỉnh các thư mục sẽ được giám sát.

<a name="user-jobs-recorder"></a>
#### User Jobs

Recorder `UserJobs` sẽ ghi lại thông tin về user đang gửi job trong ứng dụng của bạn để hiển thị trên card [Application Usage](#application-usage-card).

Bạn có thể tùy ý điều chỉnh [tỷ lệ lấy mẫu](#sampling) và các job pattern sẽ bị bỏ qua.

<a name="user-requests-recorder"></a>
#### User Requests

Recorder `UserRequests` sẽ ghi lại thông tin về user đã gửi request đối với ứng dụng của bạn để hiển thị trên card [Application Usage](#application-usage-card).

Bạn có thể tùy ý điều chỉnh [tỷ lệ lấy mẫu](#sampling) và các job pattern sẽ bị bỏ qua.

<a name="filtering"></a>
### Filtering

Như chúng ta đã thấy, nhiều [recorder](#recorders) cung cấp nhiều khả năng, thông qua cấu hình, và "bỏ qua" các mục dựa trên giá trị của chúng, chẳng hạn như URL của request. Tuy nhiên, thỉnh thoảng việc lọc các record dựa trên các yếu tố khác, chẳng hạn ví dụ như người dùng hiện tại đang được xác thực, có thể hữu ích. Để lọc các record này, bạn có thể truyền vào một closure vào phương thức `filter` của Pulse. Thông thường, phương thức `filter` nên được gọi trong phương thức `boot` của `AppServiceProvider` trong ứng dụng của bạn:

```php
use Illuminate\Support\Facades\Auth;
use Laravel\Pulse\Entry;
use Laravel\Pulse\Facades\Pulse;
use Laravel\Pulse\Value;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::filter(function (Entry|Value $entry) {
        return Auth::user()->isNotAdmin();
    });

    // ...
}
```

<a name="performance"></a>
## Hiệu suất

Pulse được thiết kế để tích hợp vào một ứng dụng hiện có mà không yêu cầu thêm bất kỳ cơ sở hạ tầng nào. Tuy nhiên, đối với các ứng dụng có hiệu suất lưu lượng truy cập cao, có một số cách để loại bỏ bất kỳ tác động nào mà Pulse có thể gây ra đối với hiệu suất của ứng dụng của bạn.

<a name="using-a-different-database"></a>
### Dùng một database khác

Đối với các ứng dụng có lưu lượng truy cập cao, bạn có thể muốn sử dụng kết nối cơ sở dữ liệu khác cho Pulse để tránh làm ảnh hưởng đến cơ sở dữ liệu ứng dụng của bạn.

Bạn có thể tùy chỉnh [kết nối cơ sở dữ liệu](/docs/{{version}}/database#configuration) được Pulse sử dụng bằng cách thiết lập biến môi trường `PULSE_DB_CONNECTION`.

```env
PULSE_DB_CONNECTION=pulse
```

<a name="ingest"></a>
### Tích hợp Redis

> [!WARNING]
> Tích hợp Redis yêu cầu Redis 6.2 trở lên và `phpredis` hoặc `predis` là Redis client driver phải được cấu hình sẵn trong ứng dụng.

Mặc định, Pulse sẽ lưu các mục trực tiếp vào [configured database connection](#using-a-different-database) sau khi HTTP response được gửi về client hoặc một job đã được xử lý xong; tuy nhiên, bạn có thể sử dụng driver Redis của Pulse để gửi các mục đến stream Redis. Tính năng này có thể được kích hoạt bằng cách cấu hình biến môi trường `PULSE_INGEST_DRIVER`:

```
PULSE_INGEST_DRIVER=redis
```

Mặc định, Pulse sẽ sử dụng [kết nối Redis](/docs/{{version}}/redis#configuration) mặc định của bạn, nhưng bạn có thể tùy chỉnh kết nối này thông qua biến môi trường `PULSE_REDIS_CONNECTION`:

```
PULSE_REDIS_CONNECTION=pulse
```

Khi sử dụng Redis, bạn có thể sẽ cần chạy lệnh `pulse:work` để theo dõi stream và di chuyển các mục từ Redis vào các bảng cơ sở dữ liệu của Pulse.

```php
php artisan pulse:work
```

> [!NOTE]
> Để duy trì process `pulse:work` chạy liên tục ở background, bạn nên sử dụng trình giám sát process như Supervisor để đảm bảo process Pulse sẽ không bị ngừng chạy.

<a name="sampling"></a>
### Lấy mẫu

Mặc định, Pulse sẽ ghi lại mọi event liên quan xảy ra trong ứng dụng của bạn. Đối với những ứng dụng có lưu lượng truy cập cao, điều này có thể dẫn đến việc cần tổng hợp hàng triệu row cơ sở dữ liệu trong bảng điều khiển, đặc biệt là trong thời gian dài.

Thay vào đó, bạn có thể chọn bật "lấy mẫu" trên một số recorder dữ liệu Pulse. Ví dụ: set tỉ lệ lấy mẫu là `0.1` trên recorder [`User Requests`](#user-requests-recorder) thì sẽ đồng nghĩa với việc bạn chỉ ghi lại khoảng 10% các request đến ứng dụng của bạn. Trong bảng điều khiển, các giá trị sẽ được tính theo tỉ lệ trên và thêm tiền tố `~` để biểu thị rằng chúng là giá trị gần đúng.

Nhìn chung, bạn càng có nhiều mục cho một số liệu cụ thể thì bạn càng có thể set một tỉ lệ lấy mẫu thấp mà không làm giảm quá nhiều độ chính xác.

<a name="trimming"></a>
### Cắt bớt

Pulse sẽ tự động cắt bớt các mục đã lưu khi chúng không nằm trong bảng điều khiển. Việc cắt bớt diễn ra khi thu thập dữ liệu bằng hệ thống ngẫu nhiên có thể được tùy chỉnh trong [file cấu hình](#configuration) của Pulse.

<a name="pulse-exceptions"></a>
### Xử lý Pulse Exception

Nếu xảy ra exception khi thu thập dữ liệu Pulse, chẳng hạn như không thể kết nối đến cơ sở dữ liệu, Pulse sẽ âm thầm ngừng hoạt động để tránh ảnh hưởng đến ứng dụng của bạn.

Nếu bạn muốn tùy chỉnh cách xử lý các exception này, bạn có thể cung cấp một closure cho phương thức `handleExceptionsUsing`:

```php
use Laravel\Pulse\Facades\Pulse;
use Illuminate\Support\Facades\Log;

Pulse::handleExceptionsUsing(function ($e) {
    Log::debug('An exception happened in Pulse', [
        'message' => $e->getMessage(),
        'stack' => $e->getTraceAsString(),
    ]);
});
```

<a name="custom-cards"></a>
## Tuỳ chỉnh Cards

Pulse cho phép bạn tự xây dựng các card tùy chỉnh để hiển thị dữ liệu phù hợp với nhu cầu cụ thể của ứng dụng của bạn. Pulse sử dụng [Livewire](https://livewire.laravel.com), vì vậy bạn có thể muốn [xem lại tài liệu hướng dẫn](https://livewire.laravel.com/docs) trước khi xây dựng card tùy chỉnh đầu tiên của bạn.

<a name="custom-card-components"></a>
### Card Components

Việc tạo một card tùy chỉnh trong Laravel Pulse sẽ bắt đầu bằng việc extend component Livewire `Card` và định nghĩa một view tương ứng:

```php
namespace App\Livewire\Pulse;

use Laravel\Pulse\Livewire\Card;
use Livewire\Attributes\Lazy;

#[Lazy]
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers');
    }
}
```

Khi sử dụng tính năng [lazy loading](https://livewire.laravel.com/docs/lazy) của Livewire, component `Card` sẽ tự động cung cấp một biến cho các thuộc tính `cols` và `rows` được truyền cho component của bạn.

Khi viết view của card Pulse, bạn có thể tận dụng các component Blade của Pulse để có một giao diện nhất quán hơn:

```blade
<x-pulse::card :cols="$cols" :rows="$rows" :class="$class" wire:poll.5s="">
    <x-pulse::card-header name="Top Sellers">
        <x-slot:icon>
            ...
        </x-slot:icon>
    </x-pulse::card-header>

    <x-pulse::scroll :expand="$expand">
        ...
    </x-pulse::scroll>
</x-pulse::card>
```

Các biến `$cols`, `$rows`, `$class` và `$expand` nên được truyền vào các component Blade tương ứng để có thể tùy chỉnh layout card từ view bảng điều khiển. Bạn cũng có thể muốn thêm thuộc tính `wire:poll.5s=""` vào view để card tự động cập nhật.

Sau khi bạn đã định nghĩa xong component và template Livewire của bạn, thì card có thể được thêm vào [view của bảng điều khiển](#dashboard-customization):

```blade
<x-pulse>
    ...

    <livewire:pulse.top-sellers cols="4" />
</x-pulse>
```

> [!NOTE]
> Nếu card của bạn được có chứa trong một package, bạn sẽ cần phải đăng ký component với Livewire bằng phương thức `Livewire::component`.

<a name="custom-card-styling"></a>
### Styling

Nếu card của bạn cần thêm style css ngoài các class và component có trong Pulse, có một số tùy chọn để thêm CSS tùy chỉnh cho card của bạn.

<a name="custom-card-styling-vite"></a>
#### Laravel Vite Integration

Nếu card tùy chỉnh của bạn nằm trong source code của ứng dụng và bạn đang sử dụng [tích hợp Vite](/docs/{{version}}/vite) của Laravel, bạn có thể cập nhật file `vite.config.js` để chứa thêm file CSS chuyên dụng cho card của bạn:

```js
laravel({
    input: [
        'resources/css/pulse/top-sellers.css',
        // ...
    ],
}),
```

Sau đó, bạn có thể sử dụng lệnh Blade `@vite` trong [view bảng điều khiển](#dashboard-customization) của bạn, chỉ định file CSS cho card của bạn:

```blade
<x-pulse>
    @vite('resources/css/pulse/top-sellers.css')

    ...
</x-pulse>
```

<a name="custom-card-styling-css"></a>
#### CSS Files

Đối với các trường hợp sử dụng khác, bao gồm các card Pulse có trong một package, bạn có thể hướng dẫn Pulse load các thêm các stylesheet bằng cách định nghĩa phương thức `css` trên component Livewire của bạn để trả về đường dẫn đến file CSS của bạn:

```php
class TopSellers extends Card
{
    // ...

    protected function css()
    {
        return __DIR__.'/../../dist/top-sellers.css';
    }
}
```

Khi card này đã được thêm vào bảng điều khiển, Pulse sẽ tự động đưa nội dung của file này vào trong tag `<style>` nên bạn sẽ không cần phải xuất file css đó ra lại thư mục `public`.

<a name="custom-card-styling-tailwind"></a>
#### Tailwind CSS

Khi sử dụng Tailwind CSS, bạn nên tạo ra một file cấu hình Tailwind chuyên dụng để tránh load file CSS không cần thiết hoặc xung đột với các class Tailwind của Pulse:

```js
export default {
    darkMode: 'class',
    important: '#top-sellers',
    content: [
        './resources/views/livewire/pulse/top-sellers.blade.php',
    ],
    corePlugins: {
        preflight: false,
    },
};
```

Sau đó, bạn có thể chỉ định file cấu hình trong file CSS của bạn:

```css
@config "../../tailwind.top-sellers.config.js";
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Bạn cũng sẽ cần phải thêm các thuộc tính `id` hoặc `class` vào trong view card của bạn để khớp với bộ selector được chuyển đến [`important` selector strategy](https://tailwindcss.com/docs/configuration#selector-strategy) của Tailwind:

```blade
<x-pulse::card id="top-sellers" :cols="$cols" :rows="$rows" class="$class">
    ...
</x-pulse::card>
```

<a name="custom-card-data"></a>
### Thu thập và tổng hợp dữ liệu

Card tùy chỉnh có thể được lấy ra và hiển thị dữ liệu từ bất kỳ đâu; tuy nhiên, bạn có thể muốn tận dụng hệ thống ghi và tổng hợp dữ liệu mạnh mẽ và hiệu quả của Pulse.

<a name="custom-card-data-capture"></a>
#### Tuỳ chỉnh data của card

Pulse cho phép bạn ghi lại "các mục" bằng phương thức `Pulse::record`:

```php
use Laravel\Pulse\Facades\Pulse;

Pulse::record('user_sale', $user->id, $sale->amount)
    ->sum()
    ->count();
```

Tham số đầu tiên được cung cấp cho phương thức `record` là `type` cho mục mà bạn đang muốn ghi vào, trong khi tham số thứ hai là `key` xác định cách dữ liệu tổng hợp sẽ được nhóm lại. Đối với hầu hết các phương thức tổng hợp, bạn cũng cần chỉ định thêm một `value` để tổng hợp. Trong ví dụ trên, giá trị được tổng hợp là `$sale->amount`. Sau đó, bạn có thể gọi một hoặc nhiều phương thức tổng hợp (chẳng hạn như `sum`) để Pulse có thể thu thập các giá trị đã tổng hợp trước đó vào một "bucket" để tính ra hiệu quả sau này.

Các phương thức tổng hợp có sẵn:

* `avg`
* `count`
* `max`
* `min`
* `sum`

> [!NOTE]
> Khi xây dựng card package ghi lại ID người dùng hiện tại đang được xác thực, bạn nên sử dụng phương thức `Pulse::resolveAuthenticatedUserId()`, phương thức này sẽ gọi mọi [tùy chỉnh user resolver](#dashboard-resolving-users) được thực hiện cho ứng dụng.

<a name="custom-card-data-retrieval"></a>
#### Retrieving Aggregate Data

Khi extend component `Card` Livewire của Pulse, bạn có thể sử dụng phương thức `aggregate` để lấy ra các dữ liệu tổng hợp cho các khoảng thời gian đang được xem trong bảng điều khiển:

```php
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers', [
            'topSellers' => $this->aggregate('user_sale', ['sum', 'count']);
        ]);
    }
}
```

Phương thức `aggregate` sẽ trả về một collection các đối tượng `stdClass` của PHP. Mỗi đối tượng sẽ chứa thuộc tính `key` đã được ghi lại trước đó cùng với các khóa cho mỗi số liệu được yêu cầu:

```
@foreach ($topSellers as $seller)
    {{ $seller->key }}
    {{ $seller->sum }}
    {{ $seller->count }}
@endforeach
```

Pulse chủ yếu sẽ lấy dữ liệu từ các bucket được tổng hợp trước đó; do đó, các bucket được chỉ định phải đã được thu thập trước bằng phương thức `Pulse::record`. Bucket tổng hợp trước đó thường sẽ nằm ngoài khoảng thời gian bạn yêu cầu, do đó Pulse sẽ tổng hợp các mục đã tổng hợp trước đó để lấp đầy khoảng trống đó và đưa ra giá trị chính xác cho toàn bộ khoảng thời gian, mà không cần phải tổng hợp toàn bộ khoảng thời gian trên mỗi request.

Bạn cũng có thể lấy ra tổng giá trị cho một loại nhất định bằng phương thức `aggregateTotal`. Ví dụ: phương thức sau sẽ lấy ra tổng doanh số của tất cả người dùng thay vì nhóm chúng theo người dùng.

```php
$total = $this->aggregateTotal('user_sale', 'sum');
```

<a name="custom-card-displaying-users"></a>
#### Displaying Users

Khi làm việc với các tổng hợp ghi lại ID người dùng làm khóa, bạn có thể resolve ra các khóa cho bản ghi người dùng bằng phương thức `Pulse::resolveUsers`:

```php
$aggregates = $this->aggregate('user_sale', ['sum', 'count']);

$users = Pulse::resolveUsers($aggregates->pluck('key'));

return view('livewire.pulse.top-sellers', [
    'sellers' => $aggregates->map(fn ($aggregate) => (object) [
        'user' => $users->find($aggregate->key),
        'sum' => $aggregate->sum,
        'count' => $aggregate->count,
    ])
]);
```

Phương thức `find` sẽ trả về một đối tượng chứa các khóa `name`, `extra` và `avatar`, mà bạn có thể truyền trực tiếp đến component Blade `<x-pulse::user-card>`:

```blade
<x-pulse::user-card :user="{{ $seller->user }}" :stats="{{ $seller->sum }}" />
```

<a name="custom-recorders"></a>
#### Custom Recorders

Tác giả của package có thể muốn cung cấp các recorder class để cho phép người dùng cấu hình việc thu thập dữ liệu của mình.

Các recorder được đăng ký trong phần `recorders` của file cấu hình `config/pulse.php` của ứng dụng:

```php
[
    // ...
    'recorders' => [
        Acme\Recorders\Deployments::class => [
            // ...
        ],

        // ...
    ],
]
```

Recorder có thể lắng nghe các event bằng cách chỉ định thuộc tính `$listen`. Pulse sẽ tự động đăng ký trình nghe này và gọi phương thức `record` của recorder:

```php
<?php

namespace Acme\Recorders;

use Acme\Events\Deployment;
use Illuminate\Support\Facades\Config;
use Laravel\Pulse\Facades\Pulse;

class Deployments
{
    /**
     * The events to listen for.
     *
     * @var list<class-string>
     */
    public array $listen = [
        Deployment::class,
    ];

    /**
     * Record the deployment.
     */
    public function record(Deployment $event): void
    {
        $config = Config::get('pulse.recorders.'.static::class);

        Pulse::record(
            // ...
        );
    }
}
```
