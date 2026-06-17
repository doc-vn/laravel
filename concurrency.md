# Concurrency

- [Giới thiệu](#introduction)
- [Chạy các task đồng thời](#running-concurrent-tasks)
    - [Named Results](#named-results)
    - [Task Timeouts](#task-timeouts)
- [Hoãn các task đồng thời](#deferring-concurrent-tasks)

<a name="introduction"></a>
## Giới thiệu

Thỉnh thoảng bạn có thể cần chạy nhiều task nặng mà không phụ thuộc vào bên khác. Trong nhiều trường hợp, hiệu suất có thể được cải thiện bằng cách chạy đồng thời các task đó. Facade `Concurrency` của Laravel sẽ cung cấp một API đơn giản, thuận tiện để chạy các task đó.

<a name="how-it-works"></a>
#### How it Works

Laravel đạt được tính năng chạy đồng thời này bằng cách chuyển hoá các closure đã cho và phân phối chúng đến một lệnh Artisan CLI ẩn, lệnh này sẽ dịch lại các closure và gọi nó trong process PHP của lệnh. Sau khi closure đã chạy xong, giá trị kết quả sẽ được chuyển trở lại process cha.

Facade `Concurrency` hỗ trợ ba driver: `process` (mặc định), `fork` và `sync`.

Driver `fork` mang lại hiệu suất tốt so với driver `process` mặc định, nhưng nó chỉ có thể được sử dụng trong CLI của PHP, vì PHP không hỗ trợ fork cho các request web. Trước khi sử dụng driver `fork`, bạn cần cài đặt package `spatie/fork`:

```shell
composer require spatie/fork
```

Driver `sync` chủ yếu được dùng trong quá trình testing khi bạn muốn disable tính năng chạy đồng thời và bạn chỉ cần đơn giản là chạy các closure trong process cha.

<a name="running-concurrent-tasks"></a>
## Chạy các task đồng thời

Để chạy các task đồng thời, bạn có thể gọi phương thức `run` của facade `Concurrency`. Phương thức `run` sẽ chấp nhận một mảng các closure cần được chạy trong các process PHP con:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
]);
```

Để sử dụng một driver cụ thể, bạn có thể sử dụng phương thức `driver`:

```php
$results = Concurrency::driver('fork')->run(...);
```

Hoặc, để thay đổi driver mặc định, bạn có thể export file cấu hình `concurrency` thông qua lệnh Artisan `config:publish` và cập nhật tùy chọn `default` trong file đó:

```shell
php artisan config:publish concurrency
```

<a name="named-results"></a>
### Named Results

Nếu bạn muốn truy cập kết quả của các task đồng thời bằng tên thay vì bằng vị trí, bạn có thể truyền vào một mảng chứa các closure. Mỗi kết quả được trả về sẽ sử dụng cùng một key với closure tương ứng của nó:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

$results = Concurrency::run([
    'users' => fn () => DB::table('users')->count(),
    'orders' => fn () => DB::table('orders')->count(),
]);

$userCount = $results['users'];
$orderCount = $results['orders'];
```

<a name="task-timeouts"></a>
### Task Timeouts

Khi sử dụng driver `process` (mặc định), bạn có thể chỉ định số giây tối đa mà một task đồng thời được phép chạy trước khi nó bị dừng bằng cách truyền vào một giá trị timeout cho phương thức `run`:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
], timeout: 30);
```

Bạn cũng có thể cung cấp một instance `CarbonInterval` nếu bạn muốn định nghĩa timeout một cách trực quan hơn:

```php
use Illuminate\Support\Facades\Concurrency;

use function Illuminate\Support\seconds;

Concurrency::run([...], timeout: seconds(30));
```

<a name="deferring-concurrent-tasks"></a>
## Hoãn các task đồng thời

Nếu bạn muốn chạy một mảng các closure, nhưng không quan tâm đến kết quả trả về của các closure đó, bạn nên cân nhắc sử dụng phương thức `defer`. Khi sử dụng phương thức `defer`, các closure sẽ không được chạy ngay lập tức. Mà thay vào đó, Laravel sẽ chạy các closure này sau khi response HTTP được gửi về người dùng:

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```
