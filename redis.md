# Redis

- [Giới thiệu](#introduction)
    - [Cấu hình](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Tương tác với Redis](#interacting-with-redis)
    - [Lệnh Pipeline](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Giới thiệu

[Redis](https://redis.io) là một dự án mã nguồn mở, dùng để lưu trữ các giá trị key-value. Nó thường được coi như là một server cấu trúc dữ liệu vì các key của nó có thể lưu [strings](https://redis.io/topics/data-types#strings), [hashes](https://redis.io/topics/data-types#hashes), [lists](https://redis.io/topics/data-types#lists), [sets](https://redis.io/topics/data-types#sets), và [sorted sets](https://redis.io/topics/data-types#sorted-sets).

Trước khi sử dụng Redis cho Laravel, chúng tôi khuyến khích bạn cài đặt và sử dụng extension [PhpRedis](https://github.com/phpredis/phpredis) của PHP thông qua PECL. Extension này phức tạp hơn để cài đặt nhưng có thể mang lại hiệu suất tốt hơn cho các ứng dụng mà sử dụng nhiều Redis.

Ngoài ra, bạn có thể cài đặt package `predis/predis` thông qua Composer:

    composer require predis/predis

> {note} Predis đã bị từ bỏ bởi tác giả gốc của package và có thể bị xóa khỏi Laravel trong một bản phát hành trong tương lai.

<a name="configuration"></a>
### Cấu hình

Cấu hình Redis cho application của bạn nằm trong file cấu hình `config/database.php`. Trong file này, bạn sẽ thấy một mảng `redis` chứa các server Redis được application của bạn sử dụng:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_CACHE_DB', 1),
        ],

    ],

Cấu hình server mặc định sẽ đủ cho môi trường phát triển. Tuy nhiên, bạn có thể tự do sửa mảng này dựa trên môi trường của bạn. Mỗi một server Redis được định nghĩa trong file cấu hình của bạn yêu cầu phải có tên, host và cổng.

#### Configuring Clusters

Nếu application của bạn đang sử dụng một cụm server Redis, bạn nên định nghĩa các cụm này bằng một key là `clusters` trong file cấu hình Redis của bạn:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

Mặc định, các cụm này sẽ thực hiện client-side sharding trên các node của bạn, cho phép bạn gộp các node lại và tạo ra một lượng lớn RAM nhất có thể. Tuy nhiên, lưu ý rằng client-side sharding không xử lý được khi bị thất bại; do đó, chủ yếu phù hợp cho việc cache các dữ liệu mà có sẵn từ một primary data store khác. Nếu bạn muốn sử dụng cụm Redis của bạn, bạn nên khai báo điều này trong key `options` trong cấu hình Redis của bạn:

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

Để sử dụng extension Predis, bạn nên thay đổi biến môi trường `REDIS_CLIENT` từ `phpredis` thành `predis`:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'predis'),

        // Rest of Redis configuration...
    ],

Ngoài các tùy chọn cấu hình server mặc định như là `host`, `port`, `database`, và `password`, Predis còn hỗ trợ thêm các [tham số kết nối](https://github.com/nrk/predis/wiki/Connection-Parameters) có thể định nghĩa cho mỗi server Redis của bạn. Để sử dụng thêm các tùy chọn cấu hình này, hãy thêm chúng vào cấu hình server Redis của bạn trong file cấu hình `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

Extension PhpRedis được cấu hình mặc định tại biến môi trường `REDIS_CLIENT` trong file `config/database.php` của bạn:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        // Rest of Redis configuration...
    ],

Nếu bạn muốn sử dụng extension PhpRedis cùng với tên viết tắt Facade `Redis`, thì bạn nên đổi tên nó thành một thứ khác, chẳng hạn như `RedisManager`, để tránh trùng lặp với class Redis. Bạn có thể làm điều đó trong phần tên viết tắt của file cấu hình `app.php` của bạn.

    'RedisManager' => Illuminate\Support\Facades\Redis::class,

Ngoài các tùy chọn cấu hình server mặc định là `host`, `port`, `database`, và `password`, PhpRedis cũng hỗ trợ thêm các tham số kết nối bổ sung như sau: `persistent`, `prefix`, `read_timeout` và `timeout`. Bạn có thể thêm bất kỳ tùy chọn nào vào cấu hình server Redis của bạn trong file cấu hình `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

#### The Redis Facade

Để tránh trùng lặp đặt tên class với chính extension Redis PHP, bạn sẽ cần phải xóa hoặc đổi tên tên viết tắt của facade `Illuminate\Support\Facades\Redis` của mảng `aliases` trong file cấu hình `app` của bạn. Nói chung, bạn nên xóa toàn bộ tên viết tắt này và chỉ tham chiếu đến facade bằng tên class đầy đủ của nó trong khi sử dụng extension Redis PHP.

<a name="interacting-with-redis"></a>
## Tương tác với Redis

Bạn có thể tương tác với Redis bằng cách gọi các phương thức khác nhau trên [facade](/docs/{{version}}/facades) `Redis`. Facade `Redis` hỗ trợ các phương thức động, nghĩa là bạn có thể gọi bất kỳ [lệnh Redis](https://redis.io/commands) nào trên facade và lệnh đó sẽ được chuyển trực tiếp đến Redis để thực hiện. Trong ví dụ này, chúng ta sẽ gọi lệnh Redis `GET` bằng cách gọi phương thức `get` trên facade `Redis`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

Như đã đề cập ở trên, bạn có thể gọi bất kỳ lệnh Redis nào trên facade `Redis`. Laravel sẽ sử dụng các phương thức magic để truyền các lệnh này đến server Redis, do đó, hãy truyền các tham số mà lệnh Redis yêu cầu:

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

Ngoài ra, bạn cũng có thể truyền lệnh đến server Redis bằng phương thức `command`, chấp nhận tên của lệnh làm tham số đầu tiên và một mảng các giá trị làm tham số thứ hai:

    $values = Redis::command('lrange', ['name', 5, 10]);

#### Using Multiple Redis Connections

Bạn có thể lấy một instance Redis bằng cách gọi phương thức `Redis::connection`:

    $redis = Redis::connection();

Điều này sẽ cung cấp cho bạn một instance mặc định của server Redis. Bạn cũng có thể truyền vào tên một kết nối hoặc tên của một cụm cho phương thức `connection` để có được một server hoặc một cụm server cụ thể như đã được định nghĩa trong file cấu hình Redis của bạn:

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>
### Lệnh Pipeline

Pipeline nên được sử dụng khi bạn cần gửi nhiều lệnh đến server. Phương thức `pipeline` chấp nhận một tham số là: một `Closure` nhận vào một instance Redis. Bạn có thể đưa vào tất cả các lệnh của bạn cho instance Redis này và tất cả chúng sẽ được truyền trực tiếp đến server, do đó cung cấp hiệu suất tốt hơn:

    Redis::pipeline(function ($pipe) {
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
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

Bây giờ chúng ta có thể publish tin nhắn lên channel bằng phương thức `publish`:

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### Wildcard Subscriptions

Sử dụng phương thức `psubscribe`, bạn có thể theo dõi một nhóm các channel, nó có thể hữu ích để lấy tất cả các tin nhắn trên tất cả các channel. Tên `$channel` sẽ được truyền làm tham số thứ hai cho callback `Closure`:

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
