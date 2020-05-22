# Cache

- [Giới thiệu](#configuration)
    - [Yêu cầu driver](#driver-prerequisites)
- [Sử dụng cache](#cache-usage)
    - [Lấy một instance cache](#obtaining-a-cache-instance)
    - [Lấy item trong cache](#retrieving-items-from-the-cache)
    - [Lưu item trong cache](#storing-items-in-the-cache)
    - [Xoá item trong cache](#removing-items-from-the-cache)
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

Laravel cung cấp một API thống nhất, dễ hiểu cho các caching backend khác nhau. Cấu hình cache được đặt tại `config/cache.php`. Trong file này, bạn có thể chỉ định cache driver nào bạn muốn được sử dụng mặc định trong suốt application của mình. Mặc định, Laravel hỗ trợ các caching backend phổ biến như [Memcached](https://memcached.org) và [Redis](https://redis.io).

File cấu hình cache cũng chứa nhiều tùy chọn khác, được ghi lại trong file, vì vậy hãy đảm bảo bạn đã đọc qua các tùy chọn này. Mặc định, Laravel được cấu hình để sử dụng cache driver `file`, lưu trữ các đối tượng ở dưới dạng byte, và được cache trong filesystem. Đối với các application lớn hơn, bạn nên sử dụng driver mạnh hơn như Memcached hoặc Redis. Bạn thậm chí có thể cấu hình nhiều cấu hình cache cho cùng một driver.

<a name="driver-prerequisites"></a>
### Yêu cầu driver

#### Database

Khi sử dụng cache driver `database`, bạn sẽ cần thiết lập một bảng để chứa các item cache. Bạn có thể làm như ví dụ ở bên dưới khai báo `Schema` cho bảng:

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} Bạn cũng có thể sử dụng lệnh Artisan `php artisan cache:table` để tạo migration với schema phù hợp.

#### Memcached

Sử dụng driver Memcached yêu cầu [Memcached PECL package](https://pecl.php.net/package/memcached) được cài đặt. Bạn có thể list tất cả các máy chủ Memcached của bạn trong file cấu hình `config/cache.php`:

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

Trước khi sử dụng cache Redis với Laravel, bạn sẽ cần cài đặt package `predis/predis` (~ 1.0) thông qua Composer hoặc cài đặt extension PhpRedis PHP qua PECL.

Để biết thêm thông tin về cách định cấu hình Redis, hãy tham khảo [trang tài liệu của Laravel](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>
## Sử dụng cache

<a name="obtaining-a-cache-instance"></a>
### Lấy một instance cache

[Contracts](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Factory` và `Illuminate\Contracts\Cache\Repository` cung cấp quyền truy cập vào các service cache của Laravel. Contract `Factory` cung cấp quyền truy cập vào tất cả các cache driver được định nghĩa cho application của bạn. Contract `Repository` thường được chỉ định là một implementation mặc định của cache driver cho application của bạn bởi file cấu hình `cache` của bạn.

Tuy nhiên, bạn cũng có thể sử dụng facade `Cache`, đây là thứ chúng ta sẽ sử dụng trong suốt tài liệu này. Facade `Cache` cung cấp quyền truy cập nhanh chóng, thuận tiện cho việc triển khai cơ bản của các contract cache của Laravel:

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

Sử dụng facade `Cache`, bạn có thể truy cập các cache store khác nhau thông qua phương thức `store`. Key sẽ được pass cho phương thức `store` phải tương ứng với một trong các store được liệt kê trong mảng cấu hình `store` trong file cấu hình `cache` của bạn:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### Lấy item trong cache

Phương thức `get` trên facade `Cache` được sử dụng để lấy các item từ cache. Nếu item không tồn tại trong cache, `null` sẽ được trả về. Nếu bạn muốn, bạn có thể pass một tham số thứ hai cho phương thức `get` chỉ định giá trị mặc định mà bạn muốn được trả về nếu item đó không tồn tại:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

Bạn thậm chí có thể pass một `Closure` làm giá trị mặc định. Kết quả của `Closure` sẽ được trả về nếu item được chỉ định không tồn tại trong cache. Pass một Closure cho phép bạn trì hoãn việc lấy ra các giá trị mặc định từ cơ sở dữ liệu hoặc service bên ngoài khác:

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

#### Checking For Item Existence

Phương thức `has` có thể được sử dụng để xác định xem một item có tồn tại trong cache hay không. Phương thức này sẽ trả về `false` nếu giá trị là `null` hoặc `false`:

    if (Cache::has('key')) {
        //
    }

#### Incrementing / Decrementing Values

Các phương thức `increment` và `decrement` có thể được sử dụng để điều chỉnh giá trị của các item integer trong cache. Cả hai phương thức này đều chấp nhận một tham số thứ hai tùy chọn cho biết số lượng tăng hoặc giảm giá trị của item:

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### Retrieve & Store

Thỉnh thoảng bạn có thể muốn lấy ra một item từ cache, nhưng cũng lưu trữ một giá trị mặc định nếu item được yêu cầu không tồn tại. Ví dụ: bạn có thể muốn lấy ra tất cả người dùng từ cache hoặc nếu không tồn tại, hãy lấy chúng từ cơ sở dữ liệu và thêm chúng vào cache. Bạn có thể làm điều này bằng cách sử dụng phương thức `Cache::remember`:

    $value = Cache::remember('users', $minutes, function () {
        return DB::table('users')->get();
    });

Nếu item không tồn tại trong cache, thì `Closure` được pass cho phương thức `remember` sẽ được thực thi và kết quả của nó sẽ được đặt vào cache.

Bạn có thể sử dụng phương thức `rememberForever` để lấy một item từ cache hoặc lưu trữ nó mãi mãi:

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### Retrieve & Delete

Nếu bạn cần lấy một item từ cache và sau đó xóa item đó, bạn có thể sử dụng phương thức `pull`. Giống như phương thức `get`, `null` sẽ được trả về nếu item không tồn tại trong cache:

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### Lưu item trong cache

Bạn có thể sử dụng phương thức `put` trên facade `Cache` để lưu trữ các item trong cache. Khi bạn đặt một item vào cache, bạn cần chỉ định số phút mà giá trị sẽ được lưu trong cache:

    Cache::put('key', 'value', $minutes);

Thay vì pass số phút dưới dạng integer, bạn cũng có thể pass một instance `DateTime` biểu thị thời gian hết hạn của item được lưu trong bộ nhớ cache:

    $expiresAt = now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### Store If Not Present

Phương thức `add` sẽ chỉ thêm item vào cache nếu nó chưa tồn tại trong cache store. Phương thức sẽ trả về `true` nếu item thực sự được thêm vào cache. Nếu không, phương thức sẽ trả về `false`:

    Cache::add('key', 'value', $minutes);

#### Storing Items Forever

Phương thức `forever` có thể được sử dụng để lưu trữ một item trong cache vĩnh viễn. Vì các item này sẽ không bao giờ hết hạn, chúng phải được xóa thủ công khỏi cache bằng phương thức `forget`:

    Cache::forever('key', 'value');

> {tip} Nếu bạn đang sử dụng driver Memcached, các item được lưu trữ "forever" có thể bị xóa khi cache đạt đến giới hạn kích thước.

<a name="removing-items-from-the-cache"></a>
### Xoá item trong cache

Bạn có thể xóa các item khỏi cache bằng phương thức `forget`:

    Cache::forget('key');

Bạn có thể xóa toàn bộ cache bằng phương thức `flush`:

    Cache::flush();

> {note} Xóa cache không liên quan cache prefix và sẽ xóa tất cả các item khỏi cache. Hãy xem xét điều này một cách cẩn thận khi xóa bộ nhớ cache đang được chia sẻ bởi các application khác.

<a name="the-cache-helper"></a>
### Cache helper

Ngoài việc sử dụng facade `Cache` hoặc [cache contract](/docs/{{version}}/contracts), bạn cũng có thể sử dụng golabl helper `cache` để lấy và lưu trữ dữ liệu qua cache. Khi hàm `cache` được gọi với một tham số chuỗi, nó sẽ trả về giá trị của key đó:

    $value = cache('key');

Nếu bạn cung cấp một mảng các cặp key / value và thời gian hết hạn cho hàm, nó sẽ lưu các giá trị vào trong cache với khoảng thời gian được chỉ định:

    cache(['key' => 'value'], $minutes);

    cache(['key' => 'value'], now()->addSeconds(10));

> {tip} Khi testing gọi đến hàm golabl `cache`, bạn có thể sử dụng phương thức `Cache::shouldReceive` giống như khi bạn [testing a facade](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>
## Cache tag

> {note} Cache tag không được hỗ trợ khi sử dụng cache driver `file` hoặc `database`. Hơn nữa, khi sử dụng nhiều tag với bộ nhớ cache được lưu trữ "forever", hiệu suất sẽ tốt nhất với driver như `memcached`, tự động xóa các bản ghi cũ.

<a name="storing-tagged-cache-items"></a>
### Lưu item vào cache tag

Cache tag cho phép bạn gắn tag các item liên quan trong cache và sau đó xóa tất cả các giá trị được lưu trong bộ nhớ cache đã được gán một tag. Bạn có thể truy cập cache được gắn tag bằng cách chuyển vào một mảng tên tag theo thứ tự. Ví dụ: hãy truy cập cache được gắn tag và `put` giá trị trong cache:

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

<a name="accessing-tagged-cache-items"></a>
### Truy cập item từ cache tag

Để lấy một item cache được gắn tag, pass cùng một danh sách các tag theo thứ tự cho phương thức `tags` và sau đó gọi phương thức` get` bằng key bạn muốn lấy:

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### Xoá item từ cache tag

Bạn có thể xóa tất cả các item được gán một tag hoặc danh sách các tag. Ví dụ, câu lệnh này sẽ xóa tất cả các bộ nhớ cache được gắn tag `people`, `authors` hoặc cả hai. Vì vậy, cả `Anne` và `John` sẽ bị xóa khỏi cache:

    Cache::tags(['people', 'authors'])->flush();

Ngược lại, câu lệnh này sẽ chỉ xóa các bộ nhớ cache được gắn thẻ `authors`, vì vậy `Anne` sẽ bị xóa, nhưng không xóa `John`:

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## Thêm tuỳ biến cache driver

<a name="writing-the-driver"></a>
### Viết driver

Để tạo tùy biến cache driver của chúng ta, trước tiên chúng ta cần implement [contract](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Store`. Vì vậy, việc implementation cache MongoDB sẽ trông giống như thế này:

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

Chúng ta chỉ cần implement từng phương thức này bằng một kết nối MongoDB. Để biết ví dụ về cách implement từng phương thức này, hãy xem `Illuminate\Cache\MemcachedStore` trong source code framework. Khi việc implement của chúng ta hoàn tất, chúng ta có thể hoàn tất đăng ký tùy biến driver.

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} Nếu bạn đang tự hỏi nên đặt code tùy biến cache driver ở đâu, bạn có thể tạo một namespace `Extensions` trong thư mục `app` của bạn. Tuy nhiên, hãy nhớ rằng Laravel không có cấu trúc application theo kiểu cứng nhắc và bạn có thể thoải mái tự tổ chức application của bạn theo sở thích của bạn.

<a name="registering-the-driver"></a>
### Đăng ký driver

Để đăng ký tùy biến cache driver với Laravel, chúng ta sẽ sử dụng phương thức `extend` trong facade `Cache`. Gọi tới `Cache::extend` có thể được thực hiện trong phương thức `boot` của `App\Providers\AppServiceProvider` được đi kèm các application Laravel mới hoặc bạn có thể tạo service provider của riêng mình để chứa extension đó - chỉ cần không quên đăng ký provider đó trong mảng provider `config/app.php`:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Tham số đầu tiên được pass cho phương thức `extend` là tên của driver. Điều này sẽ tương ứng với tùy chọn `driver` của bạn trong file cấu hình `config/cache.php`. Tham số thứ hai là một Closure sẽ trả về một instance `Illuminate\Cache\Repository`. Closure sẽ được pass một instance `$app`, là một instance của [service container](/docs/{{version}}/container).

Khi extension của bạn được đăng ký, hãy cập nhật tùy chọn `driver` trong file cấu hình `config/cache.php` của bạn thành tên của extension.

<a name="events"></a>
## Event

Để thực thi code trên mọi thao tác cache, bạn có thể listen cho các [event](/docs/{{version}}/events) được kích hoạt bởi cache. Thông thường, bạn nên đặt những event listener này trong `EventServiceProvider`:

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
