# Cache

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Yêu cầu driver](#driver-prerequisites)
- [Sử dụng cache](#cache-usage)
    - [Lấy một instance cache](#obtaining-a-cache-instance)
    - [Lấy item trong cache](#retrieving-items-from-the-cache)
    - [Lưu item trong cache](#storing-items-in-the-cache)
    - [Xoá item trong cache](#removing-items-from-the-cache)
    - [Memory cache](#cache-memoization)
    - [Cache helper](#the-cache-helper)
- [Cache Tags](#cache-tags)
- [Atomic Locks](#atomic-locks)
    - [Quản lý Locks](#managing-locks)
    - [Quản lý Locks trong Processes](#managing-locks-across-processes)
    - [Giới hạn đồng bộ](#concurrency-limiting)
- [Dự phòng cache](#cache-failover)
- [Thêm tùy biến cache driver](#adding-custom-cache-drivers)
    - [Viết driver](#writing-the-driver)
    - [Đăng ký driver](#registering-the-driver)
- [Event](#events)

<a name="introduction"></a>
## Giới thiệu

Một số task sẽ lấy hoặc xử lý dữ liệu do ứng dụng của bạn thực hiện có thể tốn nhiều CPU hoặc mất vài giây để hoàn thành. Trong trường hợp này, dữ liệu đã lấy ra được lưu vào bộ nhớ cache trong một thời gian để có thể lấy ra nhanh dữ liệu này trong các request tiếp theo cho cùng một dữ liệu. Dữ liệu được lưu trong bộ nhớ cache thường được lưu trong kho lưu dữ liệu nhanh, chẳng hạn như [Memcached](https://memcached.org) hoặc [Redis](https://redis.io).

Rất may, Laravel đã cung cấp một API hợp nhất, rõ ràng cho nhiều backend cache khác nhau, cho phép bạn tận dụng khả năng lấy dữ liệu cực nhanh của chúng và tăng tốc ứng dụng web của bạn.

<a name="configuration"></a>
## Cấu hình

Cấu hình cache của application được lưu trong file `config/cache.php`. Trong file này, các bạn có thể chỉ định cache store nào bạn muốn được sử dụng mặc định trong application của bạn. Mặc định, Laravel hỗ trợ các backend caching phổ biến như [Memcached](https://memcached.org), [Redis](https://redis.io), [DynamoDB](https://aws.amazon.com/dynamodb), và mặc định là cơ sở dữ liệu quan hệ. Ngoài ra, có sẵn driver cache dựa trên file, trong khi driver cache `array` và `null` cung cấp backend cache thuận tiện cho các automated test của bạn.

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

```php
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
```

Nếu cần, bạn có thể set tùy chọn `host` thành một đường dẫn socket UNIX. Nếu bạn làm điều này, tùy chọn `port` nên được set thành `0`:

```php
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
```

<a name="redis"></a>
#### Redis

Trước khi sử dụng cache Redis với Laravel, bạn sẽ cần cài đặt extension PhpRedis của PHP thông qua PECL hoặc cài đặt package `predis/predis` (~2.0) thông qua Composer. [Laravel Sail](/docs/{{version}}/sail) đã chứa extension này. Ngoài ra, mặc định, các nền tảng ứng dụng Laravel chính thức như [Laravel Cloud](https://cloud.laravel.com) và [Laravel Forge](https://forge.laravel.com) cũng đã được cài đặt extension PhpRedis.

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

```php
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
```

<a name="accessing-multiple-cache-stores"></a>
#### Accessing Multiple Cache Stores

Sử dụng facade `Cache`, bạn có thể truy cập các cache store khác nhau thông qua phương thức `store`. Key mà được truyền vào trong phương thức `store` cũng phải tương ứng với một store mà đã được liệt kê trong mảng `store` trong file cấu hình `cache` của bạn:

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes
```

<a name="retrieving-items-from-the-cache"></a>
### Lấy item trong cache

Phương thức `get` của facade `Cache` được sử dụng để lấy các item từ cache. Nếu item không tồn tại trong cache, giá trị `null` sẽ được trả về. Nếu bạn muốn, bạn có thể truyền vào tham số thứ hai cho phương thức `get` chỉ định giá trị mặc định mà bạn muốn trả về nếu item đó không tồn tại:

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

Bạn thậm chí có thể truyền vào một closure làm giá trị mặc định. Kết quả của closure sẽ được trả về nếu item cần lấy không tồn tại trong cache. Việc có thể truyền vào một closure cho phép bạn có thể lấy ra giá trị trong cache trước khi phải lấy giá trị đó ra từ một cơ sở dữ liệu hoặc một service bên ngoài:

```php
$value = Cache::get('key', function () {
    return DB::table(/* ... */)->get();
});
```

<a name="determining-item-existence"></a>
#### Determining Item Existence

Phương thức `has` có thể được sử dụng để xác định xem một item có tồn tại trong cache hay không. Phương thức này cũng sẽ trả về `false` nếu item có tồn tại nhưng giá trị của nó là `null`:

```php
if (Cache::has('key')) {
    // ...
}
```

<a name="incrementing-decrementing-values"></a>
#### Incrementing / Decrementing Values

Các phương thức `increment` và `decrement` có thể được sử dụng để điều chỉnh giá trị của các item integer trong cache. Cả hai phương thức này đều chấp nhận một tham số tùy chọn thứ hai cho biết số lượng tăng hoặc giảm giá trị của item:

```php
// Initialize the value if it does not exist...
Cache::add('key', 0, now()->plus(hours: 4));

// Increment or decrement the value...
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

<a name="retrieve-store"></a>
#### Retrieve và Store

Thỉnh thoảng bạn cũng có thể muốn lấy ra một item từ cache và cũng muốn lưu lại một giá trị mặc định vào cache nếu item đó không tồn tại. Ví dụ: bạn có thể muốn lấy ra tất cả các người dùng từ cache, nếu trong cache chưa tồn tại dữ liệu đó, thì bạn có thể lấy chúng ra từ cơ sở dữ liệu và thêm chúng vào cache. Bạn có thể làm điều này bằng cách sử dụng phương thức `Cache::remember`:

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

Nếu item đó không tồn tại trong cache, thì closure được truyền vào trong phương thức `remember` sẽ được thực thi và kết quả của nó sẽ được lưu vào cache.

Bạn có thể sử dụng phương thức `rememberForever` để lấy một item từ cache hoặc lưu trữ nó mãi mãi nếu nó không tồn tại:

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

<a name="swr"></a>
#### Stale While Revalidate

Khi sử dụng phương thức `Cache::remember`, một số người dùng có thể gặp phải tình trạng phản hồi chậm nếu giá trị được lưu trong bộ nhớ cache đã hết hạn. Đối với một số loại dữ liệu nhất định, việc cho phép dữ liệu cũ được lấy ra trong khi một giá trị mới đang được lưu vào trong bộ nhớ cache có thể hữu ích, để giúp người dùng tránh được tình trạng phản hồi chậm trong khi giá trị mới đang được lưu vào trong bộ nhớ cache. Điều này thường được gọi là pattern "stale-while-revalidate", và phương thức `Cache::flexible` cung cấp một implementation cho pattern này.

Phương thức flexible sẽ chấp nhận một mảng chỉ định thời gian giá trị được lưu trong bộ nhớ cache sẽ được coi là "mới" và khi nào nó trở thành giá trị "cũ". Giá trị đầu tiên trong mảng định nghĩa số giây mà bộ nhớ cache được coi là mới, trong khi giá trị thứ hai sẽ định nghĩa thời gian nó có thể được lấy như dữ liệu cũ trước khi cần lấy mới.

Nếu một request được thực hiện trong khoảng thời gian đầu (trước giá trị đầu tiên), bộ nhớ cache sẽ được trả về ngay lập tức mà không cần tính toán lại. Nếu một request được thực hiện trong khoảng thời gian cũ (giữa hai giá trị), giá trị cũ sẽ được cung cấp cho người dùng và một [hàm chạy sau](/docs/{{version}}/helpers#deferred-functions) sẽ được đăng ký để làm mới giá trị đã được lưu trong bộ nhớ cache sau khi response được gửi đến người dùng. Nhưng nếu một request được thực hiện sau giá trị thứ hai, thì bộ nhớ cache sẽ được coi như là đã hết hạn và giá trị sẽ được tính toán lại, và điều này đó có thể dẫn đến response chậm cho người dùng:

```php
$value = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```

<a name="retrieve-delete"></a>
#### Retrieve và Delete

Nếu bạn cần lấy một item từ cache và sau đó xóa item đó đi, bạn có thể sử dụng phương thức `pull`. Giống như phương thức `get`, thì `null` sẽ được trả về nếu item đó không tồn tại trong cache:

```php
$value = Cache::pull('key');

$value = Cache::pull('key', 'default');
```

<a name="storing-items-in-the-cache"></a>
### Lưu item trong cache

Bạn có thể sử dụng phương thức `put` trên facade `Cache` để lưu trữ một item vào trong cache:

```php
Cache::put('key', 'value', $seconds = 10);
```

Nếu thời gian lưu trữ không được truyền cho phương thức `put`, thì item sẽ được lưu trữ vô thời hạn:

```php
Cache::put('key', 'value');
```

Thay vì truyền vào số giây dưới dạng integer, bạn cũng có thể truyền vào một instance `DateTime` dùng để biểu thị thời gian hết hạn mong muốn của item được lưu trong bộ nhớ cache:

```php
Cache::put('key', 'value', now()->plus(minutes: 10));
```

<a name="store-if-not-present"></a>
#### Store If Not Present

Phương thức `add` sẽ chỉ thêm item vào cache nếu giá trị chưa tồn tại trong cache store. Phương thức này sẽ trả về `true` nếu item đó thực sự được thêm vào cache. Nếu không, phương thức sẽ trả về `false`. Phương thức `add` là một hành động duy nhất:

```php
Cache::add('key', 'value', $seconds);
```

<a name="storing-items-forever"></a>
#### Storing Items Forever

Phương thức `forever` có thể được sử dụng để lưu trữ một item trong cache vĩnh viễn. Vì các item này sẽ không bao giờ hết hạn, nên chúng sẽ phải được xóa khỏi cache một cách thủ công bằng phương thức `forget`:

```php
Cache::forever('key', 'value');
```

> [!NOTE]
> Nếu bạn đang sử dụng driver Memcached, các item được lưu trữ "forever" có thể bị xóa đi khi cache đạt tới một giới hạn kích thước nhất định.

<a name="removing-items-from-the-cache"></a>
### Xoá item trong cache

Bạn có thể xóa các item khỏi cache bằng phương thức `forget`:

```php
Cache::forget('key');
```

Bạn cũng có thể xóa các item bằng cách cung cấp thời gian hết hạn là 0 hoặc là một số âm:

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

Bạn có thể xóa toàn bộ cache bằng phương thức `flush`:

```php
Cache::flush();
```

> [!WARNING]
> Khi xóa toàn bộ cache thì nó sẽ xóa tất cả các item ra khỏi cache đã cấu hình mà không tâm đến cache prefix. Hãy xem xét điều này một cách cẩn thận trước khi xóa, nếu cache đó đang được dùng để chia sẻ cho các application khác.

<a name="cache-memoization"></a>
### Memory cache

Driver cache `memo` của Laravel cho phép bạn lưu trữ tạm thời các giá trị cache đã được resolve vào bộ nhớ memory trong một request hoặc một job duy nhất. Điều này giúp ngăn chặn việc truy cập lại vào bộ nhớ cache lặp đi lặp lại trong cùng một lần thực thi, giúp cải thiện đáng kể hiệu năng.

Để sử dụng memory cache, hãy gọi phương thức `memo`:

```php
use Illuminate\Support\Facades\Cache;

$value = Cache::memo()->get('key');
```

Phương thức `memo` có thêm tùy chọn nhận vào tên của một cache store, dùng để xác định cache store cơ sở mà driver memoized sẽ bao bọc:

```php
// Using the default cache store...
$value = Cache::memo()->get('key');

// Using the Redis cache store...
$value = Cache::memo('redis')->get('key');
```

Lần gọi `get` đầu tiên cho một key nhất định sẽ lấy giá trị từ cache store của bạn, nhưng các lần gọi tiếp theo trong cùng một request hoặc một job sẽ lấy giá trị từ bộ nhớ memory:

```php
// Hits the cache...
$value = Cache::memo()->get('key');

// Does not hit the cache, returns memoized value...
$value = Cache::memo()->get('key');
```

Khi gọi các phương thức làm thay đổi giá trị cache (chẳng hạn như `put`, `increment`, `remember`, v.v.), memory cache cũng sẽ tự động xóa giá trị đã được lưu và chuyển lệnh gọi phương thức thay đổi đó đến cache store nơi lưu giá trị:

```php
Cache::memo()->put('name', 'Taylor'); // Writes to underlying cache...
Cache::memo()->get('name');           // Hits underlying cache...
Cache::memo()->get('name');           // Memoized, does not hit cache...

Cache::memo()->put('name', 'Tim');    // Forgets memoized value, writes new value...
Cache::memo()->get('name');           // Hits underlying cache again...
```

<a name="the-cache-helper"></a>
### The Cache Helper

Ngoài việc sử dụng facade `Cache`, bạn cũng có thể sử dụng hàm global `cache` để lấy và lưu trữ dữ liệu vào bộ nhớ cache. Khi hàm `cache` được gọi với một tham số chuỗi, nó sẽ trả về giá trị của khóa tương ứng với chuỗi đó:

```php
$value = cache('key');
```

Nếu bạn cung cấp một mảng các cặp khóa, giá trị và một khoảng thời gian hết hạn cho hàm, thì nó sẽ lưu trữ các cặp khoá, giá trị đó vào trong bộ nhớ cache với khoảng thời gian hết hạn đã được chỉ định đó:

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->plus(minutes: 10));
```

Khi hàm `cache` được gọi mà không có bất kỳ tham số nào được truyền vào, thì nó sẽ trả về một instance của implementation `Illuminate\Contracts\Cache\Factory`, cho phép bạn gọi các phương thức caching khác:

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

> [!NOTE]
> Khi testing tới các lệnh gọi hàm global `cache`, bạn có thể sử dụng phương thức `Cache::shouldReceive` giống như bạn đang [testing một facade](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>
## Cache Tags

> [!WARNING]
> Cache tags không được hỗ trợ khi sử dụng các driver cache `file`, `dynamodb`, hoặc `database`.

<a name="storing-tagged-cache-items"></a>
### Storing Tagged Cache Items

Cache tag cho phép bạn gắn tag các item liên quan có trong cache và sau đó xóa tất cả các giá trị cache đã được gán cho một tag nhất định. Bạn có thể truy cập vào cache đã được gắn tag bằng cách truyền vào một mảng tên tag theo thứ tự. Ví dụ: hãy truy cập vào một cache đã được gắn tag và `put` một giá trị vào cache:

```php
use Illuminate\Support\Facades\Cache;

Cache::tags(['people', 'artists'])->put('John', $john, $seconds);
Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);
```

<a name="accessing-tagged-cache-items"></a>
### Accessing Tagged Cache Items

Các item được lưu thông qua tag không thể được truy cập nếu không cung cấp các tag đã được sử dụng để lưu giá trị đó. Để lấy ra một item cache đã được gắn tag, hãy truyền cùng một danh sách các tag đã được sắp xếp vào phương thức `tags`, sau đó gọi phương thức `get` với key mà bạn muốn lấy:

```php
$john = Cache::tags(['people', 'artists'])->get('John');

$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

<a name="removing-tagged-cache-items"></a>
### Removing Tagged Cache Items

Bạn có thể xóa tất cả các item đã được gán tag hoặc một danh sách các tag. Ví dụ, đoạn code sau sẽ xóa tất cả các cache được gắn tag là `people`, `authors`, hoặc cả hai. Vì vậy, cả `Anne` và `John` sẽ bị xóa khỏi cache:

```php
Cache::tags(['people', 'authors'])->flush();
```

Ngược lại, đoạn code dưới đây sẽ chỉ xóa các giá trị cache được gắn tag `authors`, vì vậy `Anne` sẽ bị xóa, nhưng `John` thì không:

```php
Cache::tags('authors')->flush();
```

<a name="atomic-locks"></a>
## Atomic Locks

> [!WARNING]
> Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng cache driver `memcached`, `redis`, `dynamodb`, `database`, `file`, hoặc `array` làm cache driver mặc định của ứng dụng của bạn. Ngoài ra, tất cả các server phải được giao tiếp với cùng một server cache trung tâm.

<a name="managing-locks"></a>
### Quản lý Locks

Atomic lock cho phép thao tác với các khóa phân tán mà không cần lo lắng về việc nhiều thread cùng truy cập cùng lúc. Ví dụ: [Laravel Cloud](https://cloud.laravel.com) sử dụng Atomic lock để đảm bảo rằng chỉ có một tác vụ đang được thực thi trên một máy chủ tại một thời điểm. Bạn có thể tạo và quản lý các khóa bằng phương thức `Cache::lock`:

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // Lock acquired for 10 seconds...

    $lock->release();
}
```

Phương thức `get` cũng chấp nhận một closure. Sau khi thực thi closure xong, Laravel sẽ tự động giải phóng khóa:

```php
Cache::lock('foo', 10)->get(function () {
    // Lock acquired for 10 seconds and automatically released...
});
```

Nếu khóa chưa sẵn sàng tại thời điểm bạn yêu cầu, bạn có thể hướng dẫn Laravel đợi trong một số giây cụ thể. Nếu không thể lấy được khóa trong thời hạn đã chỉ định, một lỗi `Illuminate\Contracts\Cache\LockTimeoutException` sẽ được đưa ra:

```php
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
```

Ví dụ trên có thể được đơn giản hóa bằng cách truyền một closure cho phương thức `block`. Khi một closure được truyền cho phương thức này, Laravel sẽ cố lấy khóa trong số giây đã chỉ định và sẽ tự động giải phóng khóa sau khi quá trình closure đã được thực thi:

```php
Cache::lock('foo', 10)->block(5, function () {
    // Lock acquired for 10 seconds after waiting a maximum of 5 seconds...
});
```

<a name="managing-locks-across-processes"></a>
### Quản lý Locks trong Processes

Thỉnh thoảng, bạn có thể muốn có được một khóa trong một process và giải phóng nó trong một process khác. Ví dụ: bạn có thể muốn có được khóa trong một web request và muốn mở khóa khi kết thúc một queued job được kích hoạt bởi chính request đó. Trong trường hợp này, bạn nên truyền vào một "owner token" trong scope của khóa cho queued job để job có thể khởi tạo lại khóa đó bằng cách sử dụng token được truyền vào.

Trong ví dụ bên dưới, chúng ta sẽ gửi một queued job nếu một khóa được lấy thành công. Ngoài ra, chúng ta sẽ truyền token chủ sở hữu của khóa đó cho queued job thông qua phương thức `owner` của khóa:

```php
$podcast = Podcast::find($id);

$lock = Cache::lock('processing', 120);

if ($lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}
```

Trong job `ProcessPodcast` của ứng dụng, chúng ta có thể khôi phục và giải phóng khóa bằng cách sử dụng token chủ sở hữu:

```php
Cache::restoreLock('processing', $this->owner)->release();
```

Nếu bạn muốn giải phóng khóa mà bỏ qua owner hiện tại của khoá, bạn có thể sử dụng phương thức `forceRelease`:

```php
Cache::lock('processing')->forceRelease();
```

<a name="concurrency-limiting"></a>
### Giới hạn đồng bộ

Tính năng khoá atomic của Laravel cũng cung cấp một số cách để giới hạn việc chạy đồng bộ của các closure. Hãy sử dụng `withoutOverlapping` khi bạn muốn chỉ cho phép một instance duy nhất được chạy trong toàn bộ hạ tầng của bạn:

```php
Cache::withoutOverlapping('foo', function () {
    // Lock acquired after waiting a maximum of 10 seconds...
});
```

Mặc định, khoá sẽ được giữ cho đến khi closure được chạy xong và phương thức sẽ đợi tối đa 10 giây để lấy khoá. Bạn có thể tùy chỉnh các giá trị này bằng cách sử dụng thêm các tham số:

```php
Cache::withoutOverlapping('foo', function () {
    // Lock acquired for 120 seconds after waiting a maximum of 5 seconds...
}, lockFor: 120, waitFor: 5);
```

Nếu không thể lấy được khoá trong thời gian chờ đã được chỉ định, một exception `Illuminate\Contracts\Cache\LockTimeoutException` sẽ được đưa ra.

Nếu bạn muốn kiểm soát số lượng được chạy đồng thời, hãy sử dụng phương thức `funnel` để thiết lập số lượng tối đa được chạy đồng thời. Phương thức `funnel` sẽ hoạt động với bất kỳ driver cache nào có hỗ trợ khoá:

```php
Cache::funnel('foo')
    ->limit(3)
    ->releaseAfter(60)
    ->block(10)
    ->then(function () {
        // Concurrency lock acquired...
    }, function () {
        // Could not acquire concurrency lock...
    });
```

Key `funnel` giúp xác định resource nào đang bị giới hạn. Phương thức `limit` định nghĩa số lượng tối đa được chạy đồng thời. Phương thức `releaseAfter` thiết lập thời gian chờ tính bằng giây trước khi một chỗ được tự động release. Phương thức `block` thiết lập số giây cần chờ để có một chỗ.

Nếu bạn muốn xử lý việc timeout thông qua exception thay vì cung cấp một closure thất bại, bạn có thể bỏ qua closure thứ hai. Một exception `Illuminate\Cache\Limiters\LimiterTimeoutException` sẽ được đưa ra nếu không thể lấy được khoá trong thời gian chờ đã chỉ định:

```php
use Illuminate\Cache\Limiters\LimiterTimeoutException;

try {
    Cache::funnel('foo')
        ->limit(3)
        ->releaseAfter(60)
        ->block(10)
        ->then(function () {
            // Concurrency lock acquired...
        });
} catch (LimiterTimeoutException $e) {
    // Unable to acquire concurrency lock...
}
```

Nếu bạn muốn sử dụng một cache store cụ thể cho giới hạn đồng bộ, bạn có thể gọi phương thức `funnel` trên store mà bạn mong muốn:

```php
Cache::store('redis')->funnel('foo')
    ->limit(3)
    ->block(10)
    ->then(function () {
        // Concurrency lock acquired using the "redis" store...
    });
```

> [!NOTE]
> Phương thức `funnel` yêu cầu cache store phải implement interface `Illuminate\Contracts\Cache\LockProvider`. Nếu bạn cố gắng sử dụng `funnel` với một cache store không hỗ trợ khoá, một `BadMethodCallException` sẽ được đưa ra.

<a name="cache-failover"></a>
## Dự phòng cache

Driver cache `failover` cung cấp chức năng dự phòng tự động khi tương tác với cache. Nếu primary cache store của store `failover` bị lỗi vì bất kỳ lý do gì, Laravel sẽ tự động sử dụng store tiếp theo được cấu hình trong danh sách. Điều này đặc biệt hữu ích để đảm bảo tính sẵn sàng cao trong môi trường production nơi độ tin cậy của cache là rất quan trọng.

Để cấu hình một failover cache store, hãy chỉ định driver `failover` và cung cấp một mảng gồm các tên store được sắp xếp theo thứ tự ưu tiên. Mặc định, Laravel có chứa một ví dụ về cấu hình failover trong file cấu hình `config/cache.php` của ứng dụng:

```php
'failover' => [
    'driver' => 'failover',
    'stores' => [
        'database',
        'array',
    ],
],
```

Sau khi bạn đã cấu hình một store sử dụng driver `failover`, bạn sẽ cần thiết lập failover store đó làm cache store mặc định trong file `.env` của ứng dụng để có thể sử dụng tính năng này:

```ini
CACHE_STORE=failover
```

Khi một thao tác cache store thất bại và failover sẽ được kích hoạt, Laravel sẽ gửi đi một event `Illuminate\Cache\Events\CacheFailedOver` cho phép bạn report hoặc ghi log một cache store đã bị lỗi.

<a name="adding-custom-cache-drivers"></a>
## Thêm tùy biến cache driver

<a name="writing-the-driver"></a>
### Viết driver

Để tạo một tùy biến cache driver, trước tiên chúng ta cần implement [contract](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Store`. Và việc implementation cache MongoDB sẽ có thể trông giống như thế này:

```php
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
```

Chúng ta chỉ cần implement từng phương thức này bằng một kết nối đến MongoDB. Để biết thêm về cách implement cho từng phương thức này, hãy xem `Illuminate\Cache\MemcachedStore` trong [source code của framework Laravel](https://github.com/laravel/framework). Khi việc implement của chúng ta hoàn tất, chúng ta có thể đăng ký tùy biến driver như sau by calling the `Cache` facade's `extend` method:

```php
Cache::extend('mongo', function (Application $app) {
    return Cache::repository(new MongoStore);
});
```

> [!NOTE]
> Nếu bạn đang tự hỏi nên lưu code tùy biến cache driver ở đâu, thì bạn có thể tạo ra một namespace `Extensions` trong thư mục `app` của bạn. Tuy nhiên, hãy nhớ rằng Laravel không có cấu trúc application theo kiểu cứng nhắc và bạn có thể thoải mái tự tổ chức application của bạn theo sở thích của bạn.

<a name="registering-the-driver"></a>
### Đăng ký driver

Để đăng ký tùy biến cache driver cho Laravel, chúng ta có thể sử dụng phương thức `extend` trong facade `Cache`. Vì các service provider khác có thể cố gắng đọc các giá trị được lưu trong bộ nhớ cache trong phương thức `boot` của họ, nên chúng ta sẽ đăng ký driver tùy chỉnh của chúng ta trong một callback `booting`. Bằng cách sử dụng callback `booting`, chúng ta có thể đảm bảo rằng driver tùy chỉnh được đăng ký ngay trước khi phương thức `boot` được gọi trên các service provider khác của ứng dụng và sau khi phương thức `register` được gọi trên tất cả các service provider khác. Chúng ta sẽ đăng ký callback `booting` trong phương thức `register` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
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
```

Tham số đầu tiên được truyền vào phương thức `extend` là tên của driver. Điều này sẽ tương ứng với option `driver` trong file cấu hình `config/cache.php`. Tham số thứ hai là một closure sẽ trả về một instance `Illuminate\Cache\Repository`. Closure cũng sẽ được truyền vào một instance [service container](/docs/{{version}}/container) `$app`.

Khi extension của bạn đã được đăng ký, hãy cập nhật biến môi trường `CACHE_STORE` hoặc tùy chọn `default` trong file cấu hình `config/cache.php` của ứng dụng của bạn thành tên của extension của bạn.

<a name="events"></a>
## Event

Để chạy một đoạn code khi bạn thao tác trên bộ nhớ cache, bạn có thể listen nhiều [event](/docs/{{version}}/events) khác nhau được gửi bởi bộ nhớ cache:

<div class="overflow-auto">

| Event Name                                   |
|----------------------------------------------|
| `Illuminate\Cache\Events\CacheFlushed`       |
| `Illuminate\Cache\Events\CacheFlushing`      |
| `Illuminate\Cache\Events\CacheHit`           |
| `Illuminate\Cache\Events\CacheMissed`        |
| `Illuminate\Cache\Events\ForgettingKey`      |
| `Illuminate\Cache\Events\KeyForgetFailed`    |
| `Illuminate\Cache\Events\KeyForgotten`       |
| `Illuminate\Cache\Events\KeyWriteFailed`     |
| `Illuminate\Cache\Events\KeyWritten`         |
| `Illuminate\Cache\Events\RetrievingKey`      |
| `Illuminate\Cache\Events\RetrievingManyKeys` |
| `Illuminate\Cache\Events\WritingKey`         |
| `Illuminate\Cache\Events\WritingManyKeys`    |

</div>

Để tăng hiệu suất, bạn có thể disable các event bộ nhớ cache bằng cách set tùy chọn cấu hình `events` thành `false` cho một cache store nhất định trong file cấu hình `config/cache.php` của ứng dụng:

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```
