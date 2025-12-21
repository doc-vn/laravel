# Cache

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Yêu cầu driver](#driver-prerequisites)
- [Sử dụng cache](#cache-usage)
    - [Lấy một instance cache](#obtaining-a-cache-instance)
    - [Lấy item trong cache](#retrieving-items-from-the-cache)
    - [Lưu item trong cache](#storing-items-in-the-cache)
    - [Xoá item trong cache](#removing-items-from-the-cache)
    - [Cache helper](#the-cache-helper)
- [Atomic Locks](#atomic-locks)
    - [Quản lý Locks](#managing-locks)
    - [Quản lý Locks trong Processes](#managing-locks-across-processes)
- [Thêm tuỳ biến cache driver](#adding-custom-cache-drivers)
    - [Viết driver](#writing-the-driver)
    - [Đăng ký driver](#registering-the-driver)
- [Event](#events)

<a name="introduction"></a>
## Giới thiệu

Một số task sẽ lấy hoặc xử lý dữ liệu do ứng dụng của bạn thực hiện có thể tốn nhiều CPU hoặc mất vài giây để hoàn thành. Trong trường hợp này, dữ liệu đã lấy ra được lưu vào bộ nhớ cache trong một thời gian để có thể lấy ra nhanh dữ liệu này trong các request tiếp theo cho cùng một dữ liệu. Dữ liệu được lưu trong bộ nhớ cache thường được lưu trong kho lưu dữ liệu nhanh, chẳng hạn như [Memcached](https://memcached.org) hoặc [Redis](https://redis.io).

Rất may, Laravel đã cung cấp một API hợp nhất, rõ ràng cho nhiều backend cache khác nhau, cho phép bạn tận dụng khả năng lấy dữ liệu cực nhanh của chúng và tăng tốc ứng dụng web của bạn.

<a name="configuration"></a>
## Cấu hình

Cấu hình cache của application được lưu trong file `config/cache.php`. Trong file này, các bạn có thể chỉ định cache store nào bạn muốn được sử dụng mặc định trong application của bạn. Mặc định, Laravel hỗ trợ các backend caching phổ biến như [Memcached](https://memcached.org), [Redis](https://redis.io), [DynamoDB](https://aws.amazon.com/dynamodb), và mặc định là cơ sở dữ liệu quan hệ. Ngoài ra, có sẵn driver cache dựa trên file, trong khi driver cache `array` và "null" cung cấp backend cache thuận tiện cho các automated test của bạn.

File cấu hình cache cũng chứa nhiều lựa chọn khác mà bạn có thể xem xét. Mặc định, Laravel được cấu hình để sử dụng cache driver `database`, lưu trữ các đối tượng ở dưới dạng byte, và được cache trong cơ sở dữ liệu ứng dụng của bạn.

<a name="driver-prerequisites"></a>
### Yêu cầu driver

<a name="prerequisites-database"></a>
#### Database

Khi sử dụng cache driver `database`, bạn sẽ cần một bảng cơ sở dữ liệu để chứa dữ liệu bộ nhớ cache. Thông thường, bảng này đã được chứa sẵn trong file [database migration](/docs/{{version}}/migrations) `0001_01_01_000001_create_cache_table.php` mặc định của Laravel; tuy nhiên, nếu ứng dụng của bạn chưa chứa file migration này, bạn có thể sử dụng lệnh Artisan `make:cache-table` để tạo nó:

```shell
php artisan make:cache-table

php artisan migrate
```

<a name="memcached"></a>
#### Memcached

Sử dụng driver Memcached, sẽ yêu cầu [Memcached PECL package](https://pecl.php.net/package/memcached) phải được cài đặt. Bạn có thể list tất cả các máy chủ Memcached của bạn trong file cấu hình `config/cache.php`. This file already contains a `memcached.servers` entry to get you started:

    'memcached' => [
        // ...

        'servers' => [
            [
                'host' => env('MEMCACHED_HOST', '127.0.0.1'),
                'port' => env('MEMCACHED_PORT', 11211),
                'weight' => 100,
            ],
        ],
    ],

Nếu cần, bạn có thể set tùy chọn `host` thành một đường dẫn socket UNIX. Nếu bạn làm điều này, tùy chọn `port` nên được set thành `0`:

    'memcached' => [
        // ...

        'servers' => [
            [
                'host' => '/var/run/memcached/memcached.sock',
                'port' => 0,
                'weight' => 100
            ],
        ],
    ],

<a name="redis"></a>
#### Redis

Trước khi sử dụng cache Redis với Laravel, bạn sẽ cần cài đặt extension PhpRedis của PHP thông qua PECL hoặc cài đặt package `predis/predis` (~2.0) thông qua Composer. [Laravel Sail](/docs/{{version}}/sail) đã chứa extension này. Ngoài ra, mặc định, các nền tảng triển khai Laravel chính thức như [Laravel Forge](https://forge.laravel.com) và [Laravel Vapor](https://vapor.laravel.com) cũng đã được cài đặt extension PhpRedis.

Để biết thêm thông tin về cách cấu hình Redis, hãy tham khảo [tài liệu của Laravel](/docs/{{version}}/redis#configuration).

<a name="dynamodb"></a>
#### DynamoDB

Trước khi sử dụng driver cache [DynamoDB](https://aws.amazon.com/dynamodb), bạn phải tạo một bảng DynamoDB để lưu trữ tất cả dữ liệu được lưu trong cache. Thông thường, bảng này nên được set tên là `cache`. Tuy nhiên, bạn nên set tên cho bảng dựa trên giá trị của cấu hình `stores.dynamodb.table` trong file cấu hình `cache`. Tên bảng cũng có thể được set thông qua biến môi trường `DYNAMODB_CACHE_TABLE`.

Bảng này cũng phải có một chuỗi khóa phân vùng có tên tương ứng với giá trị của mục cấu hình `stores.dynamodb.attributes.key` trong file cấu hình `cache` của ứng dụng của bạn. Mặc định, khóa phân vùng phải được set tên là `key`.

Thông thường, DynamoDB sẽ không tự động xóa các item đã hết hạn ra khỏi bảng. Do đó, bạn nên [bật thời gian tồn tại của một item (TTL)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html) trong bảng. Khi cài đặt TTL của bảng, bạn nên set tên thuộc tính TTL là `expires_at`.

Tiếp theo, hãy cài đặt AWS SDK để ứng dụng Laravel của bạn có thể giao tiếp với DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Ngoài ra, bạn cũng nên đảm bảo cung cấp các giá trị cho các tùy chọn cấu hình cache store DynamoDB. Thông thường, các tùy chọn này, chẳng hạn như `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY`, phải được định nghĩa trong file cấu hình `.env` của ứng dụng:

```php
'dynamodb' => [
    'driver' => 'dynamodb',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
    'endpoint' => env('DYNAMODB_ENDPOINT'),
],
```

<a name="mongodb"></a>
#### MongoDB

Nếu bạn đang sử dụng MongoDB, driver cache `mongodb` sẽ được cung cấp bởi package official `mongodb/laravel-mongodb` và có thể được cấu hình bằng một kết nối cơ sở dữ liệu `mongodb`. MongoDB hỗ trợ TTL index, có thể được sử dụng để tự động xóa đi các mục cache đã hết hạn.

Để biết thêm thông tin về cách cấu hình MongoDB, vui lòng tham khảo [tài liệu về Cache và Lock](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/) của MongoDB.

<a name="cache-usage"></a>
## Sử dụng cache

<a name="obtaining-a-cache-instance"></a>
### Lấy một instance cache

Để có được một instance lưu trữ cache, bạn cũng có thể sử dụng facade `Cache` để truy cập vào cache, đây là thứ mà chúng ta sẽ dùng trong suốt tài liệu này. Facade `Cache` sẽ cung cấp các quyền truy cập nhanh chóng và thuận tiện vào các class implementation cơ bản của các contract cache của Laravel:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         */
        public function index(): array
        {
            $value = Cache::get('key');

            return [
                // ...
            ];
        }
    }

<a name="accessing-multiple-cache-stores"></a>
#### Accessing Multiple Cache Stores

Sử dụng facade `Cache`, bạn có thể truy cập các cache store khác nhau thông qua phương thức `store`. Key mà được truyền vào trong phương thức `store` cũng phải tương ứng với một store mà đã được liệt kê trong mảng `store` trong file cấu hình `cache` của bạn:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes

<a name="retrieving-items-from-the-cache"></a>
### Lấy item trong cache

Phương thức `get` của facade `Cache` được sử dụng để lấy các item từ cache. Nếu item không tồn tại trong cache, giá trị `null` sẽ được trả về. Nếu bạn muốn, bạn có thể truyền vào tham số thứ hai cho phương thức `get` chỉ định giá trị mặc định mà bạn muốn trả về nếu item đó không tồn tại:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

Bạn thậm chí có thể truyền vào một closure làm giá trị mặc định. Kết quả của closure sẽ được trả về nếu item cần lấy không tồn tại trong cache. Việc có thể truyền vào một closure cho phép bạn có thể lấy ra giá trị trong cache trước khi phải lấy giá trị đó ra từ một cơ sở dữ liệu hoặc một service bên ngoài:

    $value = Cache::get('key', function () {
        return DB::table(/* ... */)->get();
    });

<a name="determining-item-existence"></a>
#### Determining Item Existence

Phương thức `has` có thể được sử dụng để xác định xem một item có tồn tại trong cache hay không. Phương thức này cũng sẽ trả về `false` nếu item có tồn tại nhưng giá trị của nó là `null`:

    if (Cache::has('key')) {
        // ...
    }

<a name="incrementing-decrementing-values"></a>
#### Incrementing / Decrementing Values

Các phương thức `increment` và `decrement` có thể được sử dụng để điều chỉnh giá trị của các item integer trong cache. Cả hai phương thức này đều chấp nhận một tham số tùy chọn thứ hai cho biết số lượng tăng hoặc giảm giá trị của item:

    // Initialize the value if it does not exist...
    Cache::add('key', 0, now()->addHours(4));

    // Increment or decrement the value...
    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

<a name="retrieve-store"></a>
#### Retrieve và Store

Thỉnh thoảng bạn cũng có thể muốn lấy ra một item từ cache và cũng muốn lưu lại một giá trị mặc định vào cache nếu item đó không tồn tại. Ví dụ: bạn có thể muốn lấy ra tất cả các người dùng từ cache, nếu trong cache chưa tồn tại dữ liệu đó, thì bạn có thể lấy chúng ra từ cơ sở dữ liệu và thêm chúng vào cache. Bạn có thể làm điều này bằng cách sử dụng phương thức `Cache::remember`:

    $value = Cache::remember('users', $seconds, function () {
        return DB::table('users')->get();
    });

Nếu item đó không tồn tại trong cache, thì closure được truyền vào trong phương thức `remember` sẽ được thực thi và kết quả của nó sẽ được lưu vào cache.

Bạn có thể sử dụng phương thức `rememberForever` để lấy một item từ cache hoặc lưu trữ nó mãi mãi nếu nó không tồn tại:

    $value = Cache::rememberForever('users', function () {
        return DB::table('users')->get();
    });

<a name="swr"></a>
#### Stale While Revalidate

Khi sử dụng phương thức `Cache::remember`, một số người dùng có thể gặp phải tình trạng phản hồi chậm nếu giá trị được lưu trong bộ nhớ cache đã hết hạn. Đối với một số loại dữ liệu nhất định, việc cho phép dữ liệu cũ được lấy ra trong khi một giá trị mới đang được lưu vào trong bộ nhớ cache có thể hữu ích, để giúp người dùng tránh được tình trạng phản hồi chậm trong khi giá trị mới đang được lưu vào trong bộ nhớ cache. Điều này thường được gọi là pattern "stale-while-revalidate", và phương thức `Cache::flexible` cung cấp một implementation cho pattern này.

Phương thức flexible sẽ chấp nhận một mảng chỉ định thời gian giá trị được lưu trong bộ nhớ cache sẽ được coi là "mới" và khi nào nó trở thành giá trị "cũ". Giá trị đầu tiên trong mảng định nghĩa số giây mà bộ nhớ cache được coi là mới, trong khi giá trị thứ hai sẽ định nghĩa thời gian nó có thể được lấy như dữ liệu cũ trước khi cần lấy mới.

Nếu một request được thực hiện trong khoảng thời gian đầu (trước giá trị đầu tiên), bộ nhớ cache sẽ được trả về ngay lập tức mà không cần tính toán lại. Nếu một request được thực hiện trong khoảng thời gian cũ (giữa hai giá trị), giá trị cũ sẽ được cung cấp cho người dùng và một [hàm chạy sau](/docs/{{version}}/helpers#deferred-functions) sẽ được đăng ký để làm mới giá trị đã được lưu trong bộ nhớ cache sau khi response được gửi đến người dùng. Nhưng nếu một request được thực hiện sau giá trị thứ hai, thì bộ nhớ cache sẽ được coi như là đã hết hạn và giá trị sẽ được tính toán lại, và điều này đó có thể dẫn đến response chậm cho người dùng:

    $value = Cache::flexible('users', [5, 10], function () {
        return DB::table('users')->get();
    });

<a name="retrieve-delete"></a>
#### Retrieve và Delete

Nếu bạn cần lấy một item từ cache và sau đó xóa item đó đi, bạn có thể sử dụng phương thức `pull`. Giống như phương thức `get`, thì `null` sẽ được trả về nếu item đó không tồn tại trong cache:

    $value = Cache::pull('key');

    $value = Cache::pull('key', 'default');

<a name="storing-items-in-the-cache"></a>
### Lưu item trong cache

Bạn có thể sử dụng phương thức `put` trên facade `Cache` để lưu trữ một item vào trong cache:

    Cache::put('key', 'value', $seconds = 10);

Nếu thời gian lưu trữ không được truyền cho phương thức `put`, thì item sẽ được lưu trữ vô thời hạn:

    Cache::put('key', 'value');

Thay vì truyền vào số giây dưới dạng integer, bạn cũng có thể truyền vào một instance `DateTime` dùng để biểu thị thời gian hết hạn mong muốn của item được lưu trong bộ nhớ cache:

    Cache::put('key', 'value', now()->addMinutes(10));

<a name="store-if-not-present"></a>
#### Store If Not Present

Phương thức `add` sẽ chỉ thêm item vào cache nếu giá trị chưa tồn tại trong cache store. Phương thức này sẽ trả về `true` nếu item đó thực sự được thêm vào cache. Nếu không, phương thức sẽ trả về `false`. The `add` method is an atomic operation:

    Cache::add('key', 'value', $seconds);

<a name="storing-items-forever"></a>
#### Storing Items Forever

Phương thức `forever` có thể được sử dụng để lưu trữ một item trong cache vĩnh viễn. Vì các item này sẽ không bao giờ hết hạn, nên chúng sẽ phải được xóa khỏi cache một cách thủ công bằng phương thức `forget`:

    Cache::forever('key', 'value');

> [!NOTE]
> Nếu bạn đang sử dụng driver Memcached, các item được lưu trữ "forever" có thể bị xóa đi khi cache đạt tới một giới hạn kích thước nhất định.

<a name="removing-items-from-the-cache"></a>
### Xoá item trong cache

Bạn có thể xóa các item khỏi cache bằng phương thức `forget`:

    Cache::forget('key');

Bạn cũng có thể xóa các item bằng cách cung cấp thời gian hết hạn là 0 hoặc là một số âm:

    Cache::put('key', 'value', 0);

    Cache::put('key', 'value', -5);

Bạn có thể xóa toàn bộ cache bằng phương thức `flush`:

    Cache::flush();

> [!WARNING]
> Khi xóa toàn bộ cache thì nó sẽ xóa tất cả các item ra khỏi cache đã cấu hình mà không tâm đến cache prefix. Hãy xem xét điều này một cách cẩn thận trước khi xóa, nếu cache đó đang được dùng để chia sẻ cho các application khác.

<a name="the-cache-helper"></a>
### The Cache Helper

Ngoài việc sử dụng facade `Cache`, bạn cũng có thể sử dụng hàm global `cache` để lấy và lưu trữ dữ liệu vào bộ nhớ cache. Khi hàm `cache` được gọi với một tham số chuỗi, nó sẽ trả về giá trị của khóa tương ứng với chuỗi đó:

    $value = cache('key');

Nếu bạn cung cấp một mảng các cặp khóa, giá trị và một khoảng thời gian hết hạn cho hàm, thì nó sẽ lưu trữ các cặp khoá, giá trị đó vào trong bộ nhớ cache với khoảng thời gian hết hạn đã được chỉ định đó:

    cache(['key' => 'value'], $seconds);

    cache(['key' => 'value'], now()->addMinutes(10));

Khi hàm `cache` được gọi mà không có bất kỳ tham số nào được truyền vào, thì nó sẽ trả về một instance của implementation `Illuminate\Contracts\Cache\Factory`, cho phép bạn gọi các phương thức caching khác:

    cache()->remember('users', $seconds, function () {
        return DB::table('users')->get();
    });

> [!NOTE]
> Khi testing tới các lệnh gọi hàm global `cache`, bạn có thể sử dụng phương thức `Cache::shouldReceive` giống như bạn đang [testing một facade](/docs/{{version}}/mocking#mocking-facades).

<a name="atomic-locks"></a>
## Atomic Locks

> [!WARNING]
> Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng cache driver `memcached`, `redis`, `dynamodb`, `database`, `file`, hoặc `array` làm cache driver mặc định của ứng dụng của bạn. Ngoài ra, tất cả các server phải được giao tiếp với cùng một server cache trung tâm.

<a name="managing-locks"></a>
### Quản lý Locks

Atomic lock cho phép thao tác với các khóa phân tán mà không cần lo lắng về việc nhiều thread cùng truy cập cùng lúc. Ví dụ: [Laravel Forge](https://forge.laravel.com) sử dụng Atomic lock để đảm bảo rằng chỉ có một tác vụ đang được thực thi trên một máy chủ tại một thời điểm. Bạn có thể tạo và quản lý các khóa bằng phương thức `Cache::lock`:

    use Illuminate\Support\Facades\Cache;

    $lock = Cache::lock('foo', 10);

    if ($lock->get()) {
        // Lock acquired for 10 seconds...

        $lock->release();
    }

Phương thức `get` cũng chấp nhận một closure. Sau khi thực thi closure xong, Laravel sẽ tự động giải phóng khóa:

    Cache::lock('foo', 10)->get(function () {
        // Lock acquired for 10 seconds and automatically released...
    });

Nếu khóa chưa sẵn sàng tại thời điểm bạn yêu cầu, bạn có thể hướng dẫn Laravel đợi trong một số giây cụ thể. Nếu không thể lấy được khóa trong thời hạn đã chỉ định, một lỗi `Illuminate\Contracts\Cache\LockTimeoutException` sẽ được đưa ra:

    use Illuminate\Contracts\Cache\LockTimeoutException;

    $lock = Cache::lock('foo', 10);

    try {
        $lock->block(5);

        // Lock acquired after waiting a maximum of 5 seconds...
    } catch (LockTimeoutException $e) {
        // Unable to acquire lock...
    } finally {
        $lock->release();
    }

Ví dụ trên có thể được đơn giản hóa bằng cách truyền một closure cho phương thức `block`. Khi một closure được truyền cho phương thức này, Laravel sẽ cố lấy khóa trong số giây đã chỉ định và sẽ tự động giải phóng khóa sau khi quá trình closure đã được thực thi:

    Cache::lock('foo', 10)->block(5, function () {
        // Lock acquired after waiting a maximum of 5 seconds...
    });

<a name="managing-locks-across-processes"></a>
### Quản lý Locks trong Processes

Thỉnh thoảng, bạn có thể muốn có được một khóa trong một process và giải phóng nó trong một process khác. Ví dụ: bạn có thể muốn có được khóa trong một web request và muốn mở khóa khi kết thúc một queued job được kích hoạt bởi chính request đó. Trong trường hợp này, bạn nên truyền vào một "owner token" trong scope của khóa cho queued job để job có thể khởi tạo lại khóa đó bằng cách sử dụng token được truyền vào.

Trong ví dụ bên dưới, chúng ta sẽ gửi một queued job nếu một khóa được lấy thành công. Ngoài ra, chúng ta sẽ truyền token chủ sở hữu của khóa đó cho queued job thông qua phương thức `owner` của khóa:

    $podcast = Podcast::find($id);

    $lock = Cache::lock('processing', 120);

    if ($lock->get()) {
        ProcessPodcast::dispatch($podcast, $lock->owner());
    }

Trong job `ProcessPodcast` của ứng dụng, chúng ta có thể khôi phục và giải phóng khóa bằng cách sử dụng token chủ sở hữu:

    Cache::restoreLock('processing', $this->owner)->release();

Nếu bạn muốn giải phóng khóa mà bỏ qua owner hiện tại của khoá, bạn có thể sử dụng phương thức `forceRelease`:

    Cache::lock('processing')->forceRelease();

<a name="adding-custom-cache-drivers"></a>
## Thêm tuỳ biến cache driver

<a name="writing-the-driver"></a>
### Viết driver

Để tạo một tùy biến cache driver, trước tiên chúng ta cần implement [contract](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Store`. Và việc implementation cache MongoDB sẽ có thể trông giống như thế này:

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys) {}
        public function put($key, $value, $seconds) {}
        public function putMany(array $values, $seconds) {}
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

Chúng ta chỉ cần implement từng phương thức này bằng một kết nối đến MongoDB. Để biết thêm về cách implement cho từng phương thức này, hãy xem `Illuminate\Cache\MemcachedStore` trong [source code của framework Laravel](https://github.com/laravel/framework). Khi việc implement của chúng ta hoàn tất, chúng ta có thể đăng ký tùy biến driver như sau by calling the `Cache` facade's `extend` method:

    Cache::extend('mongo', function (Application $app) {
        return Cache::repository(new MongoStore);
    });

> [!NOTE]
> Nếu bạn đang tự hỏi nên lưu code tùy biến cache driver ở đâu, thì bạn có thể tạo ra một namespace `Extensions` trong thư mục `app` của bạn. Tuy nhiên, hãy nhớ rằng Laravel không có cấu trúc application theo kiểu cứng nhắc và bạn có thể thoải mái tự tổ chức application của bạn theo sở thích của bạn.

<a name="registering-the-driver"></a>
### Đăng ký driver

Để đăng ký tùy biến cache driver cho Laravel, chúng ta có thể sử dụng phương thức `extend` trong facade `Cache`. Vì các service provider khác có thể cố gắng đọc các giá trị được lưu trong bộ nhớ cache trong phương thức `boot` của họ, nên chúng ta sẽ đăng ký driver tùy chỉnh của chúng ta trong một callback `booting`. Bằng cách sử dụng callback `booting`, chúng ta có thể đảm bảo rằng driver tùy chỉnh được đăng ký ngay trước khi phương thức `boot` được gọi trên các service provider khác của ứng dụng và sau khi phương thức `register` được gọi trên tất cả các service provider khác. Chúng ta sẽ đăng ký callback `booting` trong phương thức `register` của class `App\Providers\AppServiceProvider` của ứng dụng:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            $this->app->booting(function () {
                 Cache::extend('mongo', function (Application $app) {
                     return Cache::repository(new MongoStore);
                 });
             });
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            // ...
        }
    }

Tham số đầu tiên được truyền vào phương thức `extend` là tên của driver. Điều này sẽ tương ứng với option `driver` trong file cấu hình `config/cache.php`. Tham số thứ hai là một closure sẽ trả về một instance `Illuminate\Cache\Repository`. Closure cũng sẽ được truyền vào một instance [service container](/docs/{{version}}/container) `$app`.

Khi extension của bạn đã được đăng ký, hãy cập nhật biến môi trường `CACHE_STORE` hoặc tùy chọn `default` trong file cấu hình `config/cache.php` của ứng dụng của bạn thành tên của extension của bạn.

<a name="events"></a>
## Event

Để chạy một đoạn code khi bạn thao tác trên bộ nhớ cache, bạn có thể listen nhiều [event](/docs/{{version}}/events) khác nhau được gửi bởi bộ nhớ cache:

<div class="overflow-auto">

| Event Name |
| --- |
| `Illuminate\Cache\Events\CacheHit` |
| `Illuminate\Cache\Events\CacheMissed` |
| `Illuminate\Cache\Events\KeyForgotten` |
| `Illuminate\Cache\Events\KeyWritten` |

</div>

Để tăng hiệu suất, bạn có thể disable các event bộ nhớ cache bằng cách set tùy chọn cấu hình `events` thành `false` cho một cache store nhất định trong file cấu hình `config/cache.php` của ứng dụng:

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```
