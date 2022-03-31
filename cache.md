# Cache

- [Giới thiệu](#configuration)
    - [Yêu cầu driver](#driver-prerequisites)
- [Sử dụng cache](#cache-usage)
    - [Lấy một instance cache](#obtaining-a-cache-instance)
    - [Lấy item trong cache](#retrieving-items-from-the-cache)
    - [Lưu item trong cache](#storing-items-in-the-cache)
    - [Xoá item trong cache](#removing-items-from-the-cache)
    - [Atomic Lock](#atomic-locks)
    - [Cache helper](#the-cache-helper)
- [Cache tag](#cache-tags)
    - [Lưu item vào cache tag](#storing-tagged-cache-items)
    - [Truy cập item từ cache tag](#accessing-tagged-cache-items)
    - [Xoá item từ cache tag](#removing-tagged-cache-items)
- [Thêm tuỳ biến cache driver](#adding-custom-cache-drivers)
    - [Viết driver](#writing-the-driver)
    - [Đăng ký driver](#registering-the-driver)
- [Event](#events)

<a name="configuration"></a>
## Giới thiệu

Laravel cung cấp một API thống nhất, dễ hiểu cho các backend caching khác nhau. Cấu hình cache được lưu trong file `config/cache.php`. Trong file này, các bạn có thể chỉ định cache driver nào bạn muốn được sử dụng mặc định trong application của bạn. Mặc định, Laravel hỗ trợ các backend caching phổ biến như [Memcached](https://memcached.org) và [Redis](https://redis.io).

File cấu hình cache cũng chứa nhiều tùy chọn khác, vì vậy hãy chắc chắn là bạn đã đọc qua các tùy chọn đó. Mặc định, Laravel được cấu hình để sử dụng cache driver `file`, lưu trữ các đối tượng ở dưới dạng byte, và được cache trong filesystem. Đối với các application lớn, bạn nên sử dụng driver mạnh hơn như Memcached hoặc Redis. Thậm chí bạn có thể cài đặt nhiều cấu hình cache cho cùng một driver.

<a name="driver-prerequisites"></a>
### Yêu cầu driver

#### Database

Khi sử dụng cache driver `database`, bạn sẽ cần thiết lập một bảng để chứa các item cache. Bạn có thể làm như ví dụ ở bên dưới, khai báo một `Schema` cho một bảng:

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} Bạn cũng có thể sử dụng lệnh Artisan `php artisan cache:table` để tạo migration với một schema phù hợp.

#### Memcached

Sử dụng driver Memcached, sẽ yêu cầu [Memcached PECL package](https://pecl.php.net/package/memcached) phải được cài đặt. Bạn có thể list tất cả các máy chủ Memcached của bạn trong file cấu hình `config/cache.php`:

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

Bạn cũng có thể set tùy chọn `host` thành một đường dẫn socket UNIX. Nếu bạn làm điều này, tùy chọn `port` nên được set thành `0`:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

Trước khi sử dụng cache Redis với Laravel, bạn sẽ cần cài đặt package `predis/predis` (~ 1.0) thông qua Composer hoặc cài đặt extension PhpRedis của PHP thông qua PECL.

Để biết thêm thông tin về cách cấu hình Redis, hãy tham khảo [tài liệu của Laravel](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>
## Sử dụng cache

<a name="obtaining-a-cache-instance"></a>
### Lấy một instance cache

[Contracts](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Factory` và `Illuminate\Contracts\Cache\Repository` cung cấp quyền truy cập vào các service cache của Laravel. Contract `Factory` cung cấp quyền truy cập vào tất cả các cache driver mà đã được định nghĩa trong application của bạn. Còn contract `Repository` sẽ được định nghĩa một implementation của một cache driver mặc định trong application của bạn, cache driver mặc định này có thể cấu hình trong file `cache` của bạn.

Tuy nhiên, bạn cũng có thể sử dụng facade `Cache` để truy cập vào cache, đây là thứ mà chúng ta sẽ dùng trong suốt tài liệu này. Facade `Cache` sẽ cung cấp các quyền truy cập nhanh chóng và thuận tiện vào các class implementation cơ bản của các contract cache của Laravel:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### Accessing Multiple Cache Stores

Sử dụng facade `Cache`, bạn có thể truy cập các cache store khác nhau thông qua phương thức `store`. Key mà được truyền vào trong phương thức `store` cũng phải tương ứng với một store mà đã được liệt kê trong mảng `store` trong file cấu hình `cache` của bạn:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes

<a name="retrieving-items-from-the-cache"></a>
### Lấy item trong cache

Phương thức `get` trên facade `Cache` được sử dụng để lấy các item từ cache. Nếu item không tồn tại trong cache, giá trị `null` sẽ được trả về. Nếu bạn muốn, bạn có thể truyền vào tham số thứ hai cho phương thức `get` chỉ định giá trị mặc định mà bạn muốn trả về nếu item đó không tồn tại:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

Bạn thậm chí có thể truyền vào một `Closure` làm giá trị mặc định. Kết quả của `Closure` sẽ được trả về nếu item cần lấy không tồn tại trong cache. Việc có thể truyền vào một Closure cho phép bạn có thể lấy ra giá trị trong cache trước khi phải lấy giá trị đó ra từ một cơ sở dữ liệu hoặc một service bên ngoài:

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

#### Checking For Item Existence

Phương thức `has` có thể được sử dụng để xác định xem một item có tồn tại trong cache hay không. Phương thức này sẽ trả về `false` nếu giá trị là `null`:

    if (Cache::has('key')) {
        //
    }

#### Incrementing / Decrementing Values

Các phương thức `increment` và `decrement` có thể được sử dụng để điều chỉnh giá trị của các item integer trong cache. Cả hai phương thức này đều chấp nhận một tham số tùy chọn thứ hai cho biết số lượng tăng hoặc giảm giá trị của item:

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### Retrieve & Store

Thỉnh thoảng bạn cũng có thể muốn lấy ra một item từ cache và cũng muốn lưu lại một giá trị mặc định vào cache nếu item đó không tồn tại. Ví dụ: bạn có thể muốn lấy ra tất cả các người dùng từ cache, nếu trong cache chưa tồn tại dữ liệu đó, thì bạn có thể lấy chúng ra từ cơ sở dữ liệu và thêm chúng vào cache. Bạn có thể làm điều này bằng cách sử dụng phương thức `Cache::remember`:

    $value = Cache::remember('users', $seconds, function () {
        return DB::table('users')->get();
    });

Nếu item đó không tồn tại trong cache, thì `Closure` được truyền vào trong phương thức `remember` sẽ được thực thi và kết quả của nó sẽ được lưu vào cache.

Bạn có thể sử dụng phương thức `rememberForever` để lấy một item từ cache và lưu trữ nó mãi mãi:

    $value = Cache::rememberForever('users', function () {
        return DB::table('users')->get();
    });

#### Retrieve & Delete

Nếu bạn cần lấy một item từ cache và sau đó xóa item đó đi, bạn có thể sử dụng phương thức `pull`. Giống như phương thức `get`, thì `null` sẽ được trả về nếu item đó không tồn tại trong cache:

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### Lưu item trong cache

Bạn có thể sử dụng phương thức `put` trên facade `Cache` để lưu trữ một item vào trong cache:

    Cache::put('key', 'value', $seconds);

Nếu thời gian lưu trữ không được truyền cho phương thức `put`, thì item sẽ được lưu trữ vô thời hạn:

    Cache::put('key', 'value');

Thay vì truyền vào số giây dưới dạng integer, bạn cũng có thể truyền vào một instance `DateTime` dùng để biểu thị thời gian hết hạn của item được lưu trong bộ nhớ cache:

    Cache::put('key', 'value', now()->addMinutes(10));

#### Store If Not Present

Phương thức `add` sẽ chỉ thêm item vào cache nếu giá trị chưa tồn tại trong cache store. Phương thức này sẽ trả về `true` nếu item đó thực sự được thêm vào cache. Nếu không, phương thức sẽ trả về `false`:

    Cache::add('key', 'value', $seconds);

#### Storing Items Forever

Phương thức `forever` có thể được sử dụng để lưu trữ một item trong cache vĩnh viễn. Vì các item này sẽ không bao giờ hết hạn, nên chúng sẽ phải được xóa khỏi cache một cách thủ công bằng phương thức `forget`:

    Cache::forever('key', 'value');

> {tip} Nếu bạn đang sử dụng driver Memcached, các item được lưu trữ "forever" có thể bị xóa đi khi cache đạt tới một giới hạn kích thước nhất định.

<a name="removing-items-from-the-cache"></a>
### Xoá item trong cache

Bạn có thể xóa các item khỏi cache bằng phương thức `forget`:

    Cache::forget('key');

Bạn cũng có thể xóa các item bằng cách cung cấp TTL bằng 0 hoặc âm:

    Cache::put('key', 'value', 0);

    Cache::put('key', 'value', -5);

Bạn có thể xóa toàn bộ cache bằng phương thức `flush`:

    Cache::flush();

> {note} Khi xóa toàn bộ cache thì nó sẽ xóa tất cả các item ra khỏi cache mà không tâm đến cache prefix. Hãy xem xét điều này một cách cẩn thận trước khi xóa, nếu cache đó đang được dùng để chia sẻ cho các application khác.

<a name="atomic-locks"></a>
### Atomic Lock

> {note} Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng cache driver `memcached`, `dynamodb`, hoặc `redis` làm cache driver mặc định. Ngoài ra, tất cả các server phải được giao tiếp với cùng một server cache trung tâm.

Atomic lock cho phép thao tác với các khóa phân tán mà không cần lo lắng về việc nhiều thread cùng truy cập cùng lúc. Ví dụ: [Laravel Forge](https://forge.laravel.com) sử dụng Atomic lock để đảm bảo rằng chỉ có một tác vụ đang được thực thi trên một máy chủ tại một thời điểm. Bạn có thể tạo và quản lý các khóa bằng phương thức `Cache::lock`:

    use Illuminate\Support\Facades\Cache;

    $lock = Cache::lock('foo', 10);

    if ($lock->get()) {
        // Lock acquired for 10 seconds...

        $lock->release();
    }

Phương thức `get` cũng chấp nhận một Closure. Sau khi thực thi Closure xong, Laravel sẽ tự động giải phóng khóa:

    Cache::lock('foo')->get(function () {
        // Lock acquired indefinitely and automatically released...
    });

Nếu khóa chưa sẵn sàng tại thời điểm bạn yêu cầu, bạn có thể hướng dẫn Laravel đợi trong một số giây cụ thể. Nếu không thể lấy được khóa trong thời hạn đã chỉ định, một lỗi `Illuminate\Contracts\Cache\LockTimeoutException` sẽ được đưa ra:

    use Illuminate\Contracts\Cache\LockTimeoutException;

    $lock = Cache::lock('foo', 10);

    try {
        $lock->block(5);

        // Lock acquired after waiting maximum of 5 seconds...
    } catch (LockTimeoutException $e) {
        // Unable to acquire lock...
    } finally {
        optional($lock)->release();
    }

    Cache::lock('foo', 10)->block(5, function () {
        // Lock acquired after waiting maximum of 5 seconds...
    });

#### Managing Locks Across Processes

Thỉnh thoảng, bạn có thể muốn có được một khóa trong một process và giải phóng nó trong một process khác. Ví dụ: bạn có thể muốn có được khóa trong một web request và muốn mở khóa khi kết thúc một queued job được kích hoạt bởi chính request đó. Trong trường hợp này, bạn nên truyền vào một "owner token" trong scope của khóa cho queued job để job có thể khởi tạo lại khóa đó bằng cách sử dụng token được truyền vào:

    // Within Controller...
    $podcast = Podcast::find($id);

    $lock = Cache::lock('foo', 120);

    if ($result = $lock->get()) {
        ProcessPodcast::dispatch($podcast, $lock->owner());
    }

    // Within ProcessPodcast Job...
    Cache::restoreLock('foo', $this->owner)->release();

Nếu bạn muốn giải phóng khóa mà bỏ qua owner hiện tại của khoá, bạn có thể sử dụng phương thức `forceRelease`:

    Cache::lock('foo')->forceRelease();

<a name="the-cache-helper"></a>
### Cache helper

Ngoài việc sử dụng facade `Cache` hoặc [cache contract](/docs/{{version}}/contracts), bạn cũng có thể sử dụng global helper `cache` để lấy hoặc lưu trữ dữ liệu vào cache. Khi hàm helper `cache` được gọi với một tham số key, nó sẽ trả về giá trị của key đó:

    $value = cache('key');

Nếu bạn gọi tới hàm helper đó với một mảng gồm các cặp key / value và thời gian hết hạn của chúng, thì nó sẽ lưu các giá trị đó vào trong cache với khoảng thời gian đã được cho:

    cache(['key' => 'value'], $seconds);

    cache(['key' => 'value'], now()->addMinutes(10));

Khi phương thức `cache` được gọi mà không truyền vào bất kỳ tham số nào, thì nó sẽ trả về một instance của `Illuminate\Contracts\Cache\Factory` implementation, cho phép bạn gọi các phương thức khác trong bộ nhớ đệm:

    cache()->remember('users', $seconds, function () {
        return DB::table('users')->get();
    });

> {tip} Khi testing cần gọi đến hàm global `cache`, bạn có thể sử dụng phương thức `Cache::shouldReceive` giống như khi bạn [testing một facade](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>
## Cache tag

> {note} Cache tag không được hỗ trợ khi sử dụng cache driver `file` hoặc `database`. Hơn nữa, khi sử dụng nhiều tag với các bộ nhớ cache được lưu trữ "forever", thì hiệu suất sẽ tốt nhất với các driver như `memcached`, các loại mà tự động xóa các bản ghi cũ.

<a name="storing-tagged-cache-items"></a>
### Lưu item vào cache tag

Cache tag cho phép bạn gắn tag vào các item liên quan tới nhau vào trong cache và sau đó xóa tất cả các giá trị được lưu trong bộ nhớ cache mà đã được gán với một tag nhất định. Bạn có thể truy cập vào cache được gắn tag bằng cách truyền vào một mảng các tag theo thứ tự. Ví dụ: hãy truy cập vào cache được gắn tag và `put` một giá trị vào trong cache đó:

    Cache::tags(['people', 'artists'])->put('John', $john, $seconds);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);

<a name="accessing-tagged-cache-items"></a>
### Truy cập item từ cache tag

Để lấy ra một item cache mà được gắn tag, thì chúng ta truyền vào một danh sách các tag theo thứ tự cho phương thức `tags` và sau đó gọi phương thức `get` bằng key mà bạn muốn lấy:

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### Xoá item từ cache tag

Bạn có thể xóa tất cả các item được gán một tag hoặc một danh sách các tag. Ví dụ, câu lệnh này sẽ xóa tất cả các bộ nhớ cache được gắn tag `people`, `authors` hoặc cả hai. Vì vậy, cả `Anne` và `John` sẽ bị xóa khỏi cache:

    Cache::tags(['people', 'authors'])->flush();

Ngược lại, câu lệnh này sẽ chỉ xóa các bộ nhớ cache được gắn thẻ `authors`, vì vậy `Anne` sẽ bị xóa, nhưng không xóa `John`:

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## Thêm tuỳ biến cache driver

<a name="writing-the-driver"></a>
### Viết driver

Để tạo một tùy biến cache driver, trước tiên chúng ta cần implement [contract](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Store`. Và việc implementation cache MongoDB sẽ trông giống như thế này:

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

Chúng ta chỉ cần implement từng phương thức này bằng một kết nối đến MongoDB. Để biết thêm về cách implement cho từng phương thức này, hãy xem `Illuminate\Cache\MemcachedStore` trong source code framework. Khi việc implement của chúng ta hoàn tất, chúng ta có thể đăng ký tùy biến driver như sau:

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} Nếu bạn đang tự hỏi nên lưu code tùy biến cache driver ở đâu, thì bạn có thể tạo ra một namespace `Extensions` trong thư mục `app` của bạn. Tuy nhiên, hãy nhớ rằng Laravel không có cấu trúc application theo kiểu cứng nhắc và bạn có thể thoải mái tự tổ chức application của bạn theo sở thích của bạn.

<a name="registering-the-driver"></a>
### Đăng ký driver

Để đăng ký tùy biến cache driver cho Laravel, chúng ta có thể sử dụng phương thức `extend` trong facade `Cache`. Việc sử dụng `Cache::extend` này có thể được thực hiện trong phương thức `boot` trong file `App\Providers\AppServiceProvider` được đi kèm với application Laravel hoặc bạn có thể tạo ra một service provider của riêng bạn để chứa extension này - chỉ cần không quên đăng ký provider này vào trong mảng provider trong file `config/app.php`:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }
    }

Tham số đầu tiên được truyền vào phương thức `extend` là tên của driver. Điều này sẽ tương ứng với option `driver` trong file cấu hình `config/cache.php`. Tham số thứ hai là một Closure sẽ trả về một instance `Illuminate\Cache\Repository`. Closure cũng sẽ được truyền vào một instance [service container](/docs/{{version}}/container) `$app`.

Khi extension của bạn đã được đăng ký, hãy cập nhật option `driver` trong file cấu hình `config/cache.php` của bạn thành tên của extension của bạn.

<a name="events"></a>
## Event

Để thực thi một đoạn code trên các thao tác cache, bạn có thể listen cho các [event](/docs/{{version}}/events) được kích hoạt bởi cache. Thông thường, bạn nên lưu những event listener này trong file `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
