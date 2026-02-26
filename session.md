# HTTP Session

- [Giới thiệu](#introduction)
    - [Cấu hình](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Tương tác với session](#interacting-with-the-session)
    - [Lấy dữ liệu](#retrieving-data)
    - [Lưu dữ liệu](#storing-data)
    - [Flash dữ liệu](#flash-data)
    - [Xoá dữ liệu](#deleting-data)
    - [Tạo Session ID](#regenerating-the-session-id)
- [Session Cache](#session-cache)
- [Chặn session](#session-blocking)
- [Thêm tuỳ chỉnh Session Drivers](#adding-custom-session-drivers)
    - [Implementing Driver](#implementing-the-driver)
    - [Đăng ký Driver](#registering-the-driver)

<a name="introduction"></a>
## Giới thiệu

Vì các HTTP driven sẽ không lưu trạng thái của người dùng qua mỗi request, nên các session sẽ cung cấp một cách dễ dàng để lưu trữ thông tin về người dùng qua mỗi request. Thông tin người dùng sẽ thường được lưu trong một store hoặc một backend ổn định có thể được truy cập từ các request tiếp theo.

Laravel có nhiều session backend khác nhau và có thể được truy cập thông qua một API thống nhất, rõ ràng. Laravel hỗ trợ cho các backend phổ biến như [Memcached](https://memcached.org), [Redis](https://redis.io) và có cả database.

<a name="configuration"></a>
### Cấu hình

File cấu hình cho session của application của bạn sẽ được lưu tại `config/session.php`. Bạn hãy xem qua các tùy chọn có sẵn cho bạn trong file này. Mặc định, Laravel sẽ cấu hình sử dụng session driver là `database`.

Tham số `driver` sẽ khai báo nơi mà dữ liệu của session sẽ được lưu trữ cho mỗi request. Laravel cung cấp cho bạn nhiều driver khác nhau:

<div class="content-list" markdown="1">

- `file` - sessions được lưu ở file `storage/framework/sessions`.
- `cookie` - sessions được lưu trữ trong cookie và đã được mã hóa.
- `database` - sessions được lưu trữ trong cơ sở dữ liệu.
- `memcached` / `redis` - sessions được lưu trữ ở trong những cache base này.
- `dynamodb` - sessions được lưu trữ ở AWS DynamoDB.
- `array` - sessions được lưu trữ trong một mảng PHP và sẽ không được duy trì.

</div>

> [!NOTE]
> Array driver sẽ được sử dụng là driver chính trong các [testing](/docs/{{version}}/testing) để ngăn việc dữ liệu được lưu trữ trong session.

<a name="driver-prerequisites"></a>
### Điều kiện dùng Driver

<a name="database"></a>
#### Database

Khi sử dụng session driver `database`, bạn sẽ cần đảm bảo là bạn có một bảng cơ sở dữ liệu để chứa thông tin session. Thông thường, bảng này đã có trong file [migration cơ sở dữ liệu](/docs/{{version}}/migrations) mặc định `0001_01_01_000000_create_users_table.php` của Laravel; tuy nhiên, nếu bất kỳ lý do gì bạn không có bảng `sessions` đó, bạn có thể sử dụng lệnh Artisan `make:session-table` để tạo migration này:

```shell
php artisan make:session-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Trước khi sử dụng session Redis cùng với Laravel, bạn sẽ cần phải cài đặt extension của PHP thông qua PECL hoặc cài đặt package `predis/predis` (~1.0) thông qua Composer. Để biết thêm thông tin về cách cấu hình Redis, hãy tham khảo [tài liệu Redis](/docs/{{version}}/redis#configuration) của Laravel.

> [!NOTE]
> Biến môi trường `SESSION_CONNECTION`, hoặc tùy chọn `connection` trong file cấu hình `session.php`, có thể được sử dụng để chỉ định kết nối Redis sẽ được sử dụng để lưu trữ session.

<a name="interacting-with-the-session"></a>
## Tương tác với session

<a name="retrieving-data"></a>
### Lấy dữ liệu

Có hai cách chính để truy cập vào dữ liệu session trong Laravel: global helper `session` và thông qua một instance `Request`. Đầu tiên, chúng ta hãy xem xét việc truy cập session thông qua một instance `Request`, nó có thể được khai báo dưới dạng kiểu trong một route closure hoặc một phương thức controller. Hãy nhớ rằng, các phụ thuộc của phương thức controller được tự động đưa vào thông qua [service container](/docs/{{version}}/container) của Laravel:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(Request $request, string $id): View
    {
        $value = $request->session()->get('key');

        // ...

        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

Khi bạn muốn lấy một item từ session, bạn có thể truyền vào một giá trị mặc định làm tham số thứ hai cho phương thức `get`. Giá trị mặc định này sẽ được trả về nếu key bạn muốn lấy không tồn tại trong session. Nếu bạn truyền vào một closure làm giá trị mặc định cho phương thức `get` nếu key được yêu cầu không tồn tại, thì closure sẽ được thực thi và kết quả của nó sẽ được trả về:

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

<a name="the-global-session-helper"></a>
#### Global helper session

Bạn cũng có thể sử dụng qua hàm PHP global `session` để lấy và lưu trữ dữ liệu vào trong session. Khi helper `session` được gọi với một tham số là key, nó sẽ trả về giá trị của key đó trong session. Khi helper được gọi với một loạt các cặp key / value, thì các giá trị đó sẽ được lưu trữ vào trong session:

```php
Route::get('/home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

> [!NOTE]
> Có rất ít sự khác biệt giữa việc sử dụng session thông qua instance request HTTP và sử dụng thông qua global helper `session`. Cả hai phương thức đều có thể [test](/docs/{{version}}/testing) thông qua phương thức `assertSessionHas` có sẵn trong tất cả các test case của bạn.

<a name="retrieving-all-session-data"></a>
#### Lấy tất cả dữ liệu trong session

Nếu bạn muốn lấy tất cả dữ liệu trong session, bạn có thể sử dụng phương thức `all`:

```php
$data = $request->session()->all();
```

<a name="retrieving-a-portion-of-the-session-data"></a>
#### Retrieving a Portion of the Session Data

Các phương thức `only` và `except` có thể được sử dụng để lấy ra một tập con của dữ liệu session:

```php
$data = $request->session()->only(['username', 'email']);

$data = $request->session()->except(['username', 'email']);
```

<a name="determining-if-an-item-exists-in-the-session"></a>
#### Xác định một item có tồn tại trong session hay không

Để xác định xem một item có trong session hay không, bạn có thể sử dụng phương thức `has`. Phương thức `has` sẽ trả về `true` nếu item đó tồn tại và khác giá trị `null`:

```php
if ($request->session()->has('users')) {
    // ...
}
```

Để xác định xem một item có trong session hay không, ngay cả khi giá trị của nó là `null`, thì bạn có thể sử dụng phương thức `exists`:

```php
if ($request->session()->exists('users')) {
    // ...
}
```

Để xác định xem một item có tồn tại trong session hay không, bạn có thể sử dụng phương thức `missing`. Phương thức `missing` sẽ trả về `true` nếu item đó không có tồn tại trong session:

```php
if ($request->session()->missing('users')) {
    // ...
}
```

<a name="storing-data"></a>
### Lưu dữ liệu

Để lưu dữ liệu vào trong session, bạn có thể sử dụng phương thức `put` của một instance request hoặc một global helper `session`:

```php
// Via a request instance...
$request->session()->put('key', 'value');

// Via the global "session" helper...
session(['key' => 'value']);
```

<a name="pushing-to-array-session-values"></a>
#### Push một giá trị mới vào một mảng value trong session

Phương thức `push` có thể được sử dụng để đẩy một giá trị mới vào trong một mảng value session. Ví dụ: nếu key `user.teams` chứa một mảng gồm các tên team, bạn có thể đẩy thêm một giá trị mới vào mảng đó như sau:

```php
$request->session()->push('user.teams', 'developers');
```

<a name="retrieving-deleting-an-item"></a>
#### Lấy và xoá một item

Phương thức `pull` sẽ lấy ra và xóa đi một item ra khỏi session chỉ với một câu lệnh:

```php
$value = $request->session()->pull('key', 'default');
```

<a name="incrementing-and-decrementing-session-values"></a>
#### Incrementing & Decrementing Session Values

Nếu dữ liệu session của bạn chứa một số nguyên và bạn muốn tăng hoặc giảm số nguyên đó, bạn có thể sử dụng các phương thức `increment` và `decrement`:

```php
$request->session()->increment('count');

$request->session()->increment('count', $incrementBy = 2);

$request->session()->decrement('count');

$request->session()->decrement('count', $decrementBy = 2);
```

<a name="flash-data"></a>
### Flash dữ liệu

Thỉnh thoảng bạn muốn lưu trữ các item trong session cho đến request kế tiếp. Bạn có thể làm như vậy bằng cách sử dụng phương thức `flash`. Dữ liệu được lưu trữ trong session sử dụng phương thức này sẽ được lưu trữ ngay lập tức và trong request tiếp theo. Sau request tiếp theo, dữ liệu đã flash sẽ bị xóa. Dữ liệu flash chủ yếu hữu ích khi dùng cho các thông báo trạng thái ngắn:

```php
$request->session()->flash('status', 'Task was successful!');
```

Nếu bạn cần lưu dữ liệu flash của mình cho một số request, bạn có thể sử dụng phương thức `reflash`, phương thức này sẽ lưu tất cả dữ liệu đã được flash thêm một request nữa. Nếu bạn chỉ cần lưu một dữ liệu flash cụ thể nào đó, bạn có thể sử dụng phương thức `keep`:

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

Để dữ liệu flash của bạn chỉ được dành cho request hiện tại, bạn có thể sử dụng phương thức `now`:

```php
$request->session()->now('status', 'Task was successful!');
```

<a name="deleting-data"></a>
### Xoá dữ liệu

Phương thức `forget` sẽ xóa một phần dữ liệu ra khỏi session. Nếu bạn muốn xóa tất cả dữ liệu ra khỏi session, bạn có thể sử dụng phương thức `flush`:

```php
// Forget a single key...
$request->session()->forget('name');

// Forget multiple keys...
$request->session()->forget(['name', 'status']);

$request->session()->flush();
```

<a name="regenerating-the-session-id"></a>
### Tạo Session ID

Việc tạo lại session ID thường được thực hiện để ngăn kẻ xấu khai thác lỗ hỏng bảo mật [session fixation](https://owasp.org/www-community/attacks/Session_fixation) trên application của bạn.

 Nếu bạn đang sử dụng một trong các [bộ khởi động ứng dụng](/docs/{{version}}/starter-kits) của Laravel hoặc [Laravel Fortify](/docs/{{version}}/fortify), thì Laravel sẽ tự động tạo lại session ID trong khi xác thực; tuy nhiên, nếu bạn cần tự tạo lại session ID, bạn có thể sử dụng phương thức `regenerate`:

```php
$request->session()->regenerate();
```

Nếu bạn cần tạo lại ID session và xóa tất cả các dữ liệu ra khỏi session hiện tại trong một câu lệnh, bạn có thể sử dụng phương thức `invalidate`:

```php
$request->session()->invalidate();
```

<a name="session-cache"></a>
## Session Cache

Session cache của Laravel cung cấp một cách thuận tiện để lưu cache dữ liệu có phạm vi trong một session của người dùng riêng biệt. Không giống như global cache của ứng dụng, dữ liệu session cache được tự động tách biệt theo từng session và được xoá khi session hết hạn hoặc bị hủy. Session cache hỗ trợ tất cả các [phương thức cache quen thuộc của Laravel](/docs/{{version}}/cache) như `get`, `put`, `remember`, `forget`, và nhiều phương thức khác, nhưng có phạm vi trong session hiện tại.

Session cache hoàn hảo để lưu trữ các dữ liệu tạm thời, dành riêng cho người dùng mà bạn muốn duy trì qua nhiều request trong cùng một session, nhưng không cần lưu trữ vĩnh viễn. Điều này bao gồm những thứ như dữ liệu form, các phép tính tạm thời, phản hồi API hoặc bất kỳ dữ liệu ngắn hạn nào khác được gắn liền với session của một người dùng cụ thể.

Bạn có thể truy cập vào session cache thông qua phương thức `cache` trên session:

```php
$discount = $request->session()->cache()->get('discount');

$request->session()->cache()->put(
    'discount', 10, now()->plus(minutes: 5)
);
```

Để biết thêm thông tin về các phương thức cache của Laravel, hãy tham khảo [tài liệu cache](/docs/{{version}}/cache).

<a name="session-blocking"></a>
## Chặn session

> [!WARNING]
> Để sử dụng tính năng chặn session, ứng dụng của bạn phải sử dụng một driver cache mà hỗ trợ [atomic locks](/docs/{{version}}/cache#atomic-locks). Hiện tại, những driver cache đó là các driver `memcached`, `dynamicodb`, `redis`, `mongodb` (bao gồm cả package `mongodb/laravel-mongodb` official), `database`, `file`, và `array` . Ngoài ra, bạn không thể sử dụng driver session `cookie`.

Mặc định, Laravel cho phép các request sử dụng cùng một session để chạy đồng thời. Vì vậy, ví dụ: nếu bạn sử dụng thư viện JavaScript HTTP để thực hiện hai request HTTP tới ứng dụng của bạn cùng một lúc, thì cả hai sẽ thực thi đồng thời. Đối với nhiều ứng dụng, đây không phải là vấn đề; tuy nhiên, mất dữ liệu session cũng có thể xảy ra trong một phần hiếm các ứng dụng khi thực hiện request đồng thời đến hai điểm khác nhau trong cùng một ứng dụng, mà cả hai điểm đó đều có cùng chức năng ghi dữ liệu vào session.

Để giảm thiểu điều này, Laravel cung cấp chức năng cho phép bạn giới hạn các request đồng thời cho một session nhất định. Để bắt đầu, bạn có thể chỉ cần kết hợp thêm phương thức `block` vào định nghĩa route của bạn. Trong ví dụ này, một request đến điểm `/profile` sẽ nhận được một session lock. Trong khi lock này mà đang được giữ, thì bất kỳ request nào khác đến điểm `/profile` hoặc điểm `/order` mà có cùng ID session thì sẽ đợi request đến trước đó kết thúc rồi mới đến request tiếp theo tiếp tục được thực thi:

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);
```

Phương thức `block` chấp nhận hai tham số tùy chọn. Tham số đầu tiên được phương thức `block` chấp nhận là số giây tối đa mà session lock sẽ được giữ trước khi nó được giải phóng. Tất nhiên, nếu request kết thúc trước thời điểm này, thì lock này sẽ được giải phóng sớm hơn.

Tham số thứ hai được phương thức `block` chấp nhận là số giây mà một request sẽ phải đợi trong khi cố gắng lấy một session lock. Một `Illuminate\Contracts\Cache\LockTimeoutException` sẽ được đưa ra nếu request không thể lấy được session lock trong một số giây nhất định.

Nếu cả hai tham số này đều không được truyền vào, thì lock sẽ được giữ trong thời gian tối đa là 10 giây và các request khác sẽ đợi tối đa 10 giây trong khi cố gắng lấy lock:

```php
Route::post('/profile', function () {
    // ...
})->block();
```

<a name="adding-custom-session-drivers"></a>
## Thêm tuỳ chỉnh Session Drivers

<a name="implementing-the-driver"></a>
#### Implementing Driver

Nếu không có driver session nào phù hợp với nhu cầu ứng dụng của bạn, Laravel sẽ giúp bạn viết trình xử lý phiên của riêng mình. Driver session tùy chỉnh của bạn sẽ cần phải implement từ một `SessionHandlerInterface` trong PHP. Interface này chỉ chứa một vài phương thức đơn giản mà bạn cần implement. Một implementation MongoDB đơn giản sẽ trông giống như sau:

```php
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

Vì Laravel không cung cấp một thư mục mặc định để lưu các extension cho bạn. Bạn có thể tự do lưu extension của bạn vào bất kỳ nơi nào mà bạn thích. Trong ví dụ này, chúng tôi đã tạo một thư mục `Extensions` để chứa `MongoSessionHandler`.

Vì mục đích của những phương thức này là không dễ hiểu, đây là một overview về mục đích của mỗi phương thức:

<div class="content-list" markdown="1">

- Phương thức `open` thường được sử dụng trong các hệ thống lưu trữ session dựa trên file. Vì Laravel đã định nghĩa driver session `file`, nên bạn sẽ hiếm khi cần phải lưu bất cứ thứ gì vào trong phương thức này. Bạn chỉ cần để trống phương thức này.
- Phương thức `close`, giống như phương thức` open`, thường có thể bị bỏ qua. Đối với hầu hết các driver, nó là không cần thiết.
- Phương thức `read` sẽ trả về string của dữ liệu session được liên kết với `$sessionId` đã cho. Bạn sẽ không cần thực hiện bất kỳ việc chuyển đổi hoặc encoding nào khác khi truy xuất hoặc lưu trữ dữ liệu session vào trong driver của bạn, vì Laravel sẽ thực hiện việc chuyển đổi cho bạn.
- Phương thức `write` sẽ viết chuỗi `$data` đã cho liên kết với một `$sessionId` vào trong một số hệ thống lưu trữ, chẳng hạn như MongoDB, hoặc hệ thống lưu trữ khác mà bạn lựa chọn. Một lần nữa, bạn không nên thực hiện bất kỳ chuyển đổi nào - Laravel sẽ xử lý điều đó cho bạn.
- Phương thức `destroy` sẽ xóa dữ liệu được liên kết với `$sessionId` ra khỏi bộ lưu trữ.
- Phương thức `gc` sẽ hủy tất cả dữ liệu session cũ hơn so với `$lifetime` đã cho, đó là UNIX timestamp. Đối với các hệ thống tự hết hạn như Memcached và Redis, phương thức này có thể bị bỏ trống.

</div>

<a name="registering-the-driver"></a>
#### Đăng ký Driver

Khi driver của bạn đã được thực hiện xong, bạn đã sẵn sàng để đăng ký nó với Laravel. Để thêm driver vào backend session của Laravel, bạn có thể sử dụng phương thức `extend` được cung cấp bởi [facade](/docs/{{version}}/facades) `Session`. Bạn nên gọi phương thức `extend` từ phương thức `boot` của một [service provider](/docs/{{version}}/providers). Bạn có thể làm điều này từ `AppServiceProvider` hoặc tạo một provider mới hoàn toàn:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Session::extend('mongo', function (Application $app) {
            // Return an implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
        });
    }
}
```

Khi driver session đã được đăng ký, bạn có thể chỉ định driver `mongo` làm driver session của ứng dụng bằng cách sử dụng biến môi trường `SESSION_DRIVER` hoặc trong file cấu hình `config/session.php` của ứng dụng.
