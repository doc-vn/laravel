# Redis

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Cụm](#clusters)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Tương tác với Redis](#interacting-with-redis)
    - [Transactions](#transactions)
    - [Lệnh Pipeline](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Giới thiệu

[Redis](https://redis.io) là một dự án mã nguồn mở, dùng để lưu trữ các giá trị key-value. Nó thường được coi như là một server cấu trúc dữ liệu vì các key của nó có thể lưu [strings](https://redis.io/docs/data-types/strings/), [hashes](https://redis.io/docs/data-types/hashes/), [lists](https://redis.io/docs/data-types/lists/), [sets](https://redis.io/docs/data-types/sets/), và [sorted sets](https://redis.io/docs/data-types/sorted-sets/).

Trước khi sử dụng Redis cho Laravel, chúng tôi khuyến khích bạn cài đặt và sử dụng extension [PhpRedis](https://github.com/phpredis/phpredis) của PHP thông qua PECL. Extension này phức tạp hơn để cài đặt compared to "user-land" PHP packages nhưng có thể mang lại hiệu suất tốt hơn cho các ứng dụng mà sử dụng nhiều Redis. Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail), thì extension này đã được cài đặt sẵn trong Docker container của ứng dụng của bạn.

Nếu bạn không thể cài đặt extension PhpRedis, bạn có thể cài đặt package `predis/predis` thông qua Composer. Predis là một client Redis được viết hoàn toàn bằng PHP và nó không yêu cầu cài thêm bất kỳ extension nào:

```shell
composer require predis/predis
```

<a name="configuration"></a>
## Cấu hình

Bạn có thể cấu hình cài đặt Redis của ứng dụng thông qua file cấu hình `config/database.php`. Trong file này, bạn sẽ thấy một mảng `redis` chứa các server Redis được application của bạn sử dụng:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_CACHE_DB', 1),
        ],

    ],

Mỗi một server Redis được định nghĩa trong file cấu hình của bạn yêu cầu phải có tên, host và một cổng trừ khi bạn định nghĩa một URL để đại diện cho kết nối Redis đó:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'url' => 'tcp://127.0.0.1:6379?database=0',
        ],

        'cache' => [
            'url' => 'tls://user:password@127.0.0.1:6380?database=1',
        ],

    ],

<a name="configuring-the-connection-scheme"></a>
#### Configuring The Connection Scheme

Mặc định, các Redis client sẽ sử dụng scheme `tcp` khi kết nối với Redis server của bạn; tuy nhiên, bạn có thể sử dụng mã hóa TLS / SSL bằng cách chỉ định một tùy chọn cấu hình `scheme` trong mảng cấu hình của Redis server của bạn:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'scheme' => 'tls',
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

    ],

<a name="clusters"></a>
### Cụm

Nếu application của bạn đang sử dụng một cụm server Redis, bạn nên định nghĩa các cụm này bằng một key là `clusters` trong file cấu hình Redis của bạn. Mặc định, khóa cấu hình này không tồn tại, do đó bạn sẽ cần tạo nó trong file cấu hình `config/database.php` của ứng dụng:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD'),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

Mặc định, các cụm này sẽ thực hiện client-side sharding trên các node của bạn, cho phép bạn gộp các node lại và tạo ra một lượng lớn RAM nhất có thể. Tuy nhiên, client-side sharding không xử lý được khi bị thất bại; do đó, nó chủ yếu phù hợp cho việc cache tạm thời các dữ liệu mà có sẵn từ một primary data store khác.

Nếu bạn muốn sử dụng cụm Redis thay vì client-side sharding, bạn có thể chỉ định điều này bằng cách set giá trị cấu hình `options.cluster` thành `redis` trong file cấu hình `config/database.php` của ứng dụng của bạn:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

Nếu bạn muốn ứng dụng của bạn tương tác với Redis thông qua package Predis, bạn nên set giá trị của biến môi trường `REDIS_CLIENT` là `predis`:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'predis'),

        // ...
    ],

Ngoài các tùy chọn cấu hình server mặc định như là `host`, `port`, `database`, và `password`, Predis còn hỗ trợ thêm các [tham số kết nối](https://github.com/nrk/predis/wiki/Connection-Parameters) có thể định nghĩa cho mỗi server Redis của bạn. Để sử dụng thêm các tùy chọn cấu hình này, hãy thêm chúng vào cấu hình server Redis của bạn trong file cấu hình `config/database.php` của application của bạn:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="the-redis-facade-alias"></a>
#### The Redis Facade Alias

File cấu hình `config/app.php` của Laravel chứa một mảng `aliases` định nghĩa tất cả các alias của class sẽ được framework đăng ký. Mặc định, sẽ không có alias `Redis` vì nó xung đột với class tên `Redis` do extension PhpRedis cung cấp. Nếu bạn đang sử dụng Predis client và muốn thêm alias `Redis`, bạn có thể thêm alias này vào trong mảng `aliases` trong file cấu hình `config/app.php` của ứng dụng của bạn:

    'aliases' => Facade::defaultAliases()->merge([
        'Redis' => Illuminate\Support\Facades\Redis::class,
    ])->toArray(),

<a name="phpredis"></a>
### PhpRedis

Mặc định, Laravel sẽ sử dụng extension PhpRedis để giao tiếp với Redis. Client mà Laravel sẽ sử dụng để giao tiếp với Redis được quyết định bởi giá trị của tùy chọn cấu hình `redis.client`, thường phản ánh bởi giá trị của biến môi trường `REDIS_CLIENT`:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        // Rest of Redis configuration...
    ],

Ngoài các tham số kết nối mặc định `scheme`, `host`, `port`, `database`, và `password`, PhpRedis cũng hỗ trợ thêm các tham số kết nối bổ sung như sau: `name`, `persistent`, `persistent_id`, `prefix`, `read_timeout`, `retry_interval`, `timeout`, và `context`. Bạn có thể thêm bất kỳ tùy chọn nào vào cấu hình server Redis của bạn trong file cấu hình `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
        'context' => [
            // 'auth' => ['username', 'secret'],
            // 'stream' => ['verify_peer' => false],
        ],
    ],

<a name="phpredis-serialization"></a>
#### PhpRedis Serialization và Compression

Extension PhpRedis cũng có thể được cấu hình để sử dụng nhiều thuật toán nén và serializers khác nhau. Các thuật toán này có thể được cấu hình thông qua mảng `options` trong cấu hình Redis của bạn:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'serializer' => Redis::SERIALIZER_MSGPACK,
            'compression' => Redis::COMPRESSION_LZ4,
        ],

        // Rest of Redis configuration...
    ],

Các serializers được hỗ trợ hiện tại là: `Redis::SERIALIZER_NONE` (mặc định), `Redis::SERIALIZER_PHP`, `Redis::SERIALIZER_JSON`, `Redis::SERIALIZER_IGBINARY` và `Redis::SERIALIZER_MSGPACK`.

Các thuật toán nén được hỗ trợ là: `Redis::COMPRESSION_NONE` (mặc định), `Redis::COMPRESSION_LZF`, `Redis::COMPRESSION_ZSTD` và `Redis::COMPRESSION_LZ4`.

<a name="interacting-with-redis"></a>
## Tương tác với Redis

Bạn có thể tương tác với Redis bằng cách gọi các phương thức khác nhau trên [facade](/docs/{{version}}/facades) `Redis`. Facade `Redis` hỗ trợ các phương thức động, nghĩa là bạn có thể gọi bất kỳ [lệnh Redis](https://redis.io/commands) nào trên facade và lệnh đó sẽ được chuyển trực tiếp đến Redis để thực hiện. Trong ví dụ này, chúng ta sẽ gọi lệnh Redis `GET` bằng cách gọi phương thức `get` trên facade `Redis`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         */
        public function show(string $id): View
        {
            return view('user.profile', [
                'user' => Redis::get('user:profile:'.$id)
            ]);
        }
    }

Như đã đề cập ở trên, bạn có thể gọi bất kỳ lệnh Redis nào trên facade `Redis`. Laravel sẽ sử dụng các phương thức magic để truyền các lệnh đó đến server Redis. Nếu lệnh Redis yêu cầu các tham số, bạn cũng có thể truyền chúng sang phương thức tương ứng của facade:

    use Illuminate\Support\Facades\Redis;

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

Ngoài ra, bạn có thể truyền lệnh đến server Redis bằng phương thức `command` của facade `Redis`, chấp nhận tên của lệnh làm tham số đầu tiên và một mảng các giá trị làm tham số thứ hai:

    $values = Redis::command('lrange', ['name', 5, 10]);

<a name="using-multiple-redis-connections"></a>
#### Using Multiple Redis Connections

File cấu hình `config/database.php` của ứng dụng của bạn cho phép bạn định nghĩa nhiều kết nối máy chủ Redis. Bạn có thể nhận được kết nối đến một kết nối Redis cụ thể bằng cách sử dụng phương thức `connection` của facade `Redis`:

    $redis = Redis::connection('connection-name');

Để có được một instance của kết nối Redis mặc định, bạn có thể gọi phương thức `connection` mà không cần thêm bất kỳ tham số nào:

    $redis = Redis::connection();

<a name="transactions"></a>
### Transactions

Phương thức `transaction` của facade `Redis` cung cấp một wrapper thuận tiện xung quanh các lệnh `MULTI` và `EXEC` gốc của Redis. Phương thức `transaction` chấp nhận một closure làm tham số duy nhất của nó. Closure này sẽ nhận vào một instance kết nối Redis và bạn có thể đưa vào bất kỳ lệnh nào mà bạn muốn cho instance này. Tất cả các lệnh Redis được đưa vào trong closure sẽ được thực thi trong một transaction nguyên tử duy nhất:

    use Redis;
    use Illuminate\Support\Facades;

    Facades\Redis::transaction(function (Redis $redis) {
        $redis->incr('user_visits', 1);
        $redis->incr('total_visits', 1);
    });

> [!WARNING]
> Khi định nghĩa một transaction Redis, bạn không được lấy bất kỳ giá trị nào từ kết nối Redis. Hãy nhớ rằng, transaction của bạn được thực thi dưới dạng một thao tác duy nhất và thao tác đó không được thực thi cho đến khi toàn bộ closure của bạn thực thi xong các lệnh của nó.

#### Lua Scripts

Phương thức `eval` sẽ cung cấp một phương thức khác để thực thi nhiều lệnh Redis trong một thao tác duy nhất. Tuy nhiên, phương thức `eval` có lợi ích là có thể tương tác và kiểm tra các giá trị khóa của Redis trong quá trình hoạt động đó. Script lệnh Redis được viết bằng [ngôn ngữ lập trình Lua](https://www.lua.org).

Phương thức `eval` lúc đầu có thể hơi đáng sợ, nhưng chúng ta sẽ xem một ví dụ cơ bản để hiểu thêm về nó. Phương thức `eval` yêu cầu một số tham số. Trước tiên, bạn nên truyền script Lua (dưới dạng chuỗi) cho phương thức. Thứ hai, bạn nên truyền số lượng khóa (dưới dạng integer) mà script tương tác. Thứ ba, bạn nên truyền tên của các khoá đó. Cuối cùng, bạn có thể truyền thêm bất kỳ tham số nào khác mà bạn cần truy cập trong script của bạn.

Trong ví dụ này, chúng ta sẽ tăng counter, kiểm tra giá trị mới của nó và tăng counter thứ hai nếu giá trị của counter thứ nhất lớn hơn năm. Cuối cùng, chúng ta sẽ trả về giá trị của counter đầu tiên:

    $value = Redis::eval(<<<'LUA'
        local counter = redis.call("incr", KEYS[1])

        if counter > 5 then
            redis.call("incr", KEYS[2])
        end

        return counter
    LUA, 2, 'first-counter', 'second-counter');

> [!WARNING]
> Vui lòng tham khảo [tài liệu về Redis](https://redis.io/commands/eval) để biết thêm thông tin về script Redis.

<a name="pipelining-commands"></a>
### Lệnh Pipeline

Thỉnh thoảng bạn có thể cần thực thi nhiều lệnh Redis. Thay vì thực hiện truy vấn tới server Redis của bạn cho từng lệnh một, bạn có thể sử dụng phương thức `pipeline`. Phương thức `pipeline` chấp nhận một tham số là: một closure nhận vào một instance Redis. Bạn có thể đưa vào tất cả các lệnh của bạn cho instance Redis này và tất cả chúng sẽ được truyền trực tiếp đến server đồng thời để giảm các lượt truy vấn đến server. Các lệnh vẫn sẽ được thực thi theo thứ tự chúng được ghi:

    use Redis;
    use Illuminate\Support\Facades;

    Facades\Redis::pipeline(function (Redis $pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Pub / Sub

Laravel cung cấp một interface thuận tiện cho các lệnh `publish` và `subscribe` của Redis. Các lệnh Redis này cho phép bạn listen các tin nhắn trên một "channel" nhất định. Bạn có thể publish tin nhắn lên channel từ một application khác hoặc thậm chí sử dụng một ngôn ngữ lập trình khác, cho phép giao tiếp dễ dàng giữa các application và process.

Đầu tiên, hãy thiết lập một listen channel bằng phương thức `subscribe`. Chúng ta sẽ thực hiện gọi phương thức này trong một [lệnh Artisan](/docs/{{version}}/artisan) vì khi gọi phương thức `subscribe` là sẽ bắt đầu chạy một process lâu dài:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         */
        public function handle(): void
        {
            Redis::subscribe(['test-channel'], function (string $message) {
                echo $message;
            });
        }
    }

Bây giờ chúng ta có thể publish tin nhắn lên channel bằng phương thức `publish`:

    use Illuminate\Support\Facades\Redis;

    Route::get('/publish', function () {
        // ...

        Redis::publish('test-channel', json_encode([
            'name' => 'Adam Wathan'
        ]));
    });

<a name="wildcard-subscriptions"></a>
#### Wildcard Subscriptions

Sử dụng phương thức `psubscribe`, bạn có thể theo dõi một nhóm các channel, nó có thể hữu ích để lấy tất cả các tin nhắn trên tất cả các channel. Tên channel sẽ được truyền làm tham số thứ hai cho closure:

    Redis::psubscribe(['*'], function (string $message, string $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function (string $message, string $channel) {
        echo $message;
    });
