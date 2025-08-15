# Queues

- [Giới thiệu](#introduction)
    - [Connection và Queue](#connections-vs-queues)
    - [Driver chú ý và điều kiện](#driver-prerequisites)
- [Tạo Job](#creating-jobs)
    - [Tạo class Job](#generating-job-classes)
    - [Cấu trúc class](#class-structure)
    - [Unique Jobs](#unique-jobs)
    - [Encrypted Jobs](#encrypted-jobs)
- [Job Middleware](#job-middleware)
    - [Giới hạn tỷ lệ](#rate-limiting)
    - [Chặn Job chồng nhau](#preventing-job-overlaps)
    - [Ngoại lệ](#throttling-exceptions)
- [Gửi Job](#dispatching-jobs)
    - [Delay gửi](#delayed-dispatching)
    - [Đồng bộ gửi](#synchronous-dispatching)
    - [Jobs và Database Transactions](#jobs-and-database-transactions)
    - [Kết hợp Job](#job-chaining)
    - [Tuỳ biến Queue và Connection](#customizing-the-queue-and-connection)
    - [Khai báo số lần chạy Job tối đa / giá trị timeout](#max-job-attempts-and-timeout)
    - [Xử lý Error](#error-handling)
- [Job Batching](#job-batching)
    - [Định nghĩa Batchable Jobs](#defining-batchable-jobs)
    - [Gửi Batches](#dispatching-batches)
    - [Chains và Batches](#chains-and-batches)
    - [Thêm Jobs vào Batches](#adding-jobs-to-batches)
    - [Kiểm tra Batches](#inspecting-batches)
    - [Huỷ Batches](#cancelling-batches)
    - [Batch Failures](#batch-failures)
    - [Xoá Batches](#pruning-batches)
    - [Lưu batches trong DynamoDB](#storing-batches-in-dynamodb)
- [Queueing Closures](#queueing-closures)
- [Chạy Queue Worker](#running-the-queue-worker)
    - [Lệnh `queue:work`](#the-queue-work-command)
    - [Queue ưu tiên](#queue-priorities)
    - [Queue Worker và Deployment](#queue-workers-and-deployment)
    - [Job hết hạn và timeout](#job-expirations-and-timeouts)
- [Cấu hình Supervisor](#supervisor-configuration)
- [Xử lý Job failed](#dealing-with-failed-jobs)
    - [Dọn dẹp sau khi Job failed](#cleaning-up-after-failed-jobs)
    - [Chạy lại Job failed](#retrying-failed-jobs)
    - [Bỏ qua các model bị thiếu](#ignoring-missing-models)
    - [Xoá job failed](#pruning-failed-jobs)
    - [Lưu job failed vào trong DynamoDB](#storing-failed-jobs-in-dynamodb)
    - [Disable lưu job failed](#disabling-failed-job-storage)
    - [Failed Job Events](#failed-job-events)
- [Xoá job từ queue](#clearing-jobs-from-queues)
- [Giám sát queue](#monitoring-your-queues)
- [Testing](#testing)
    - [Fake một tập hợp Jobs](#faking-a-subset-of-jobs)
    - [Testing Job Chains](#testing-job-chains)
    - [Testing Job Batches](#testing-job-batches)
- [Job Event](#job-events)

<a name="introduction"></a>
## Giới thiệu

Trong khi xây dựng ứng dụng web, bạn có thể có một số task, chẳng hạn như chuyển đổi và lưu trữ file CSV đã tải lên, và quá trình này mất nhiều thời gian để thực hiện trong một request web thông thường. Rất may, Laravel cho phép bạn dễ dàng tạo ra các queued job có thể được dùng để xử lý các công việc ở trên trong background. Bằng cách chuyển các task tốn nhiều thời gian sang queue, ứng dụng của bạn có thể đáp ứng các request web với tốc độ nhanh và cung cấp trải nghiệm người dùng tốt hơn cho khách hàng của bạn.

Queue của Laravel cung cấp một queueing API hợp nhất trên nhiều loại queue backend khác nhau, chẳng hạn như [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), hoặc thậm chí là một database.

Các tùy chọn cấu hình queue của Laravel được lưu trong file cấu hình `config/queue.php` trong ứng dụng của bạn. Trong file này, bạn sẽ tìm thấy các cấu hình connection cho từng loại driver queue có trong framework, gồm có database, [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), và [Beanstalkd](https://beanstalkd.github.io/), cũng như một driver chạy đồng bộ job (để sử dụng trong quá trình phát triển). Driver queue `null` cũng đã được khai báo để loại bỏ các job đã được queue.

> [!NOTE]
> Laravel hiện cung cấp Horizon là một hệ thống cấu hình và điều khiển cho các queue mà được tạo bởi Redis của bạn. Hãy xem toàn bộ [tài liệu Horizon](/docs/{{version}}/horizon) để biết thêm thông tin chi tiết.

<a name="connections-vs-queues"></a>
### Connection và Queue

Trước khi bắt đầu với Laravel queue, điều quan trọng là phải hiểu sự khác biệt giữa "connections" và "queues". Trong file cấu hình `config/queue.php` của bạn, có một mảng cấu hình là `connections`. Tùy chọn này sẽ định nghĩa các connection đến các queue backend service như Amazon SQS, Beanstalk hoặc Redis. Tuy nhiên, bất kỳ connection nào cũng có thể có nhiều "queue", và nó có thể được coi là một ngăn xếp hoặc một mảng các job khác nhau.

Lưu ý rằng mỗi ví dụ cấu hình connection trong file cấu hình `queue` có chứa một thuộc tính `queue`. Đây là queue mặc định mà các job sẽ được gửi tới mỗi khi chúng được gửi đến một connection. Nói cách khác, nếu bạn gửi một job mà không khai báo rõ queue nào sẽ được dùng, thì job đó sẽ được lưu vào queue mà đã được định nghĩa trong thuộc tính `queue` của cấu hình connection:

    use App\Jobs\ProcessPodcast;

    // This job is sent to the default connection's default queue...
    ProcessPodcast::dispatch();

    // This job is sent to the default connection's "emails" queue...
    ProcessPodcast::dispatch()->onQueue('emails');

Một số application có thể không cần phải tạo nhiều job trong nhiều queue, thay vào đó một queue có thể là phù hợp hơn. Tuy nhiên, việc tạo các job lên nhiều queue cũng có thể đặc biệt hữu ích cho các application mà muốn ưu tiên hoặc là phân chia cách xử lý cho từng job, vì Laravel queue worker cho phép bạn khai báo các queue sẽ được xử lý theo mức độ ưu tiên. Ví dụ: nếu bạn tạo một job lên queue `high`, thì bạn có thể chạy một worker có mức độ ưu tiên xử lý cao hơn:

```shell
php artisan queue:work --queue=high,default
```

<a name="driver-prerequisites"></a>
### Driver chú ý và điều kiện

<a name="database"></a>
#### Database

Để sử dụng driver `database` queue, bạn sẽ cần một bảng cơ sở dữ liệu để lưu các job. Để tạo một migration tạo bảng, hãy chạy lệnh Artisan `queue:table`. Khi migration đã được tạo, bạn có thể migrate cơ sở dữ liệu của bạn bằng lệnh `migrate`:

```shell
php artisan queue:table

php artisan migrate
```

Cuối cùng, đừng quên bảo ứng dụng của bạn sử dụng driver `database` bằng cách cập nhật biến `QUEUE_CONNECTION` trong file `.env` của ứng dụng của bạn:

    QUEUE_CONNECTION=database

<a name="redis"></a>
#### Redis

Để sử dụng driver `redis` queue, bạn nên cấu hình connection tới Redis database trong file cấu hình `config/database.php` của bạn.

> [!WARNING]
> Các tùy chọn `serializer` và `compression` của Redis sẽ không được driver `redis` queue hỗ trợ.

**Redis Cluster**

Nếu connection Redis của bạn sử dụng một Cluster Redis, thì tên queue của bạn phải chứa một [key hash tag](https://redis.io/docs/reference/cluster-spec/#hash-tags). Điều này là bắt buộc để đảm bảo rằng tất cả các key Redis cho queue sẽ được set vào cùng một vị trí hash:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

**Blocking**

Khi sử dụng queue Redis, bạn có thể sử dụng tùy chọn cấu hình `block_for` để chỉ định khoảng thời gian mà driver sẽ đợi cho job được bắt đầu trước khi nó lặp lại vòng lặp worker và thăm dò lại cơ sở dữ liệu Redis.

Điều chỉnh giá trị này dựa trên queue load của bạn, nó có thể hiệu quả hơn việc liên tục thăm dò cơ sở dữ liệu Redis để tìm ra các job mới. Ví dụ: bạn có thể set giá trị là `5` để chỉ ra rằng driver sẽ chặn năm giây trong khi chờ job sẵn sàng:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => 'default',
        'retry_after' => 90,
        'block_for' => 5,
    ],

> [!WARNING]
> Việc set `block_for` thành `0` sẽ khiến các queue worker chặn vô thời hạn cho đến khi có job. Điều này cũng sẽ chặn các tín hiệu như `SIGTERM` được xử lý cho đến khi job tiếp theo được xử lý.

<a name="other-driver-prerequisites"></a>
#### Other Driver Prerequisites

Các library sau sẽ cần thiết cho driver queue cũng sẽ được liệt kê. Những library này có thể được cài đặt thông qua Composer package manager:

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~4.0`
- Redis: `predis/predis ~1.0` or phpredis PHP extension

</div>

<a name="creating-jobs"></a>
## Tạo Job

<a name="generating-job-classes"></a>
### Tạo class Job

Mặc định, tất cả các job cho application của bạn được lưu trong thư mục `app/Jobs`. Nếu thư mục `app/Jobs` chưa tồn tại, thì nó có thể được tạo ra khi bạn chạy lệnh Artisan `make:job`:

```shell
php artisan make:job ProcessPodcast
```

Class được tạo ra sẽ implement interface `Illuminate\Contracts\Queue\ShouldQueue`, và cho Laravel biết là job này sẽ được đưa vào queue để chạy không đồng bộ.

> [!NOTE]
> Các stub của Job có thể được tùy chỉnh bằng cách sử dụng [export stub](/docs/{{version}}/artisan#stub-customization).

<a name="class-structure"></a>
### Cấu trúc class

Các class của job rất đơn giản, thông thường chỉ chứa một phương thức `handle` được gọi khi job được xử lý bởi queue. Để bắt đầu, chúng ta hãy xem một class của một job ví dụ. Trong ví dụ này, chúng ta sẽ chạy là chúng ta quản lý một service xuất bản podcast và cần xử lý các file podcast đã được tải lên trước khi được xuất bản:

    <?php

    namespace App\Jobs;

    use App\Models\Podcast;
    use App\Services\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Create a new job instance.
         */
        public function __construct(
            public Podcast $podcast,
        ) {}

        /**
         * Execute the job.
         */
        public function handle(AudioProcessor $processor): void
        {
            // Process uploaded podcast...
        }
    }

Trong ví dụ trên, hãy lưu ý rằng chúng ta có thể truyền một [Eloquent model](/docs/{{version}}/eloquent) trực tiếp vào hàm khởi tạo của queued job. Do trait `SerializesModels` này đang được job sử dụng, nên các model Eloquent và các quan hệ của nó cũng sẽ được serialize và unserialize ngược lại khi job được xử lý.

Nếu queued job của bạn chấp nhận một model Eloquent trong hàm khởi tạo, thì chỉ có mã định danh cho model sẽ được serialize trên queue. Và khi job đó được xử lý, thì hệ thống queue sẽ tự động lấy ra lại full instance của model đó và các quan hệ của nó cũng từ cơ sở dữ liệu. Cách chuyển đổi model này cho phép gửi job nhỏ hơn nhiều đến queue driver của bạn.

<a name="handle-method-dependency-injection"></a>
#### `handle` Method Dependency Injection

Phương thức `handle` được gọi khi job được xử lý bởi queue. Lưu ý rằng chúng ta có thể khai báo các phụ thuộc vào phương thức `handle` của job. Laravel [service container](/docs/{{version}}/container) sẽ tự động inject các phụ thuộc này.

Nếu bạn muốn toàn quyền kiểm soát cách container đưa các phụ thuộc vào phương thức `handle`, bạn có thể sử dụng phương thức `bindMethod` của container. Phương thức `bindMethod` chấp nhận một callback nhận vào một job và container. Trong lệnh callback, bạn có thể thoải mái gọi phương thức `handle` theo bất kỳ cách nào mà bạn muốn. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của [service provider](/docs/{{version}}/providers) `App\Providers\AppServiceProvider`:

    use App\Jobs\ProcessPodcast;
    use App\Services\AudioProcessor;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
        return $job->handle($app->make(AudioProcessor::class));
    });

> [!WARNING]
> Dữ liệu nhị phân, chẳng hạn như một nội dung ảnh thô, phải được truyền qua hàm `base64_encode` trước khi được truyền đến một queued job. Nếu không làm điều này, thì job đó có thể serialize thành chuỗi JSON không đúng khi được đặt lên queue.

<a name="handling-relationships"></a>
#### Queued Relationships

Bởi vì tất cả các quan hệ của Eloquent model cũng sẽ được serialize trong khi một job được đưa vào queue, nên thỉnh thoảng job có thể trở nên khá lớn. Hơn thế nữa, khi một job được deserialize lại thì các quan hệ model sẽ được lấy lại toàn bộ từ cơ sở dữ liệu. Bất kỳ các ràng buộc quan hệ nào đã được thay đổi trước khi model được serialize trong quá trình queue job thì sẽ không lưu lại những thay đổi đó khi job được deserialize lại. Do đó, nếu bạn muốn làm việc với một tập hợp của một quan hệ nhất định, bạn nên ràng buộc lại quan hệ đó trong queued job của bạn.

Hoặc, để ngăn việc các quan hệ bị serialize, bạn có thể gọi phương thức `withoutRelations` trên model khi set giá trị thuộc tính. Phương thức này sẽ trả về một instance của model mà không có quan hệ được load:

    /**
     * Create a new job instance.
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast->withoutRelations();
    }

Nếu bạn đang sử dụng chức năng thuộc tính của hàm constructor property promotion PHP và muốn rằng model Eloquent sẽ không serialize các quan hệ của nó, bạn có thể sử dụng thuộc tính `WithoutRelations`:

    use Illuminate\Queue\Attributes\WithoutRelations;

    /**
     * Create a new job instance.
     */
    public function __construct(
        #[WithoutRelations]
        public Podcast $podcast
    ) {
    }

Nếu một job nhận vào một collection hoặc một mảng các model Eloquent thay vì một model duy nhất, các model trong collection đó sẽ không thể khôi phục được quan hệ của chúng khi job được deserialize và được thực thi. Điều này nhằm ngăn chặn việc sử dụng tài nguyên quá mức trên các job xử lý số lượng lớn các model.

<a name="unique-jobs"></a>
### Unique Jobs

> [!WARNING]
> Unique Job sẽ yêu cầu một cache driver hỗ trợ [locks](/docs/{{version}}/cache#atomic-locks). Hiện tại, cache driver `memcached`, `redis`, `dynamodb`, `database`, `file` và `array` đều hỗ trợ atomic lock. Ngoài ra, các ràng buộc unique job không áp dụng cho các job có trong batch.

Thỉnh thoảng, bạn có thể muốn đảm bảo rằng chỉ có một instance của một job cụ thể có trong queue tại bất kỳ thời điểm nào. Bạn có thể làm như vậy bằng cách implement interface `ShouldBeUnique` trên class job của bạn. Interface này không yêu cầu bạn định nghĩa thêm bất kỳ phương thức nào trên class của bạn:

    <?php

    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Contracts\Queue\ShouldBeUnique;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        ...
    }

Trong ví dụ trên, job `UpdateSearchIndex` là unique. Vì vậy, job sẽ không được gửi đi nếu một instance khác của job đã có trong queue và chưa được xử lý xong.

Trong một số trường hợp nhất định, bạn có thể muốn định nghĩa một "key" cụ thể để làm cho job trở nên unique hoặc bạn có thể muốn chỉ định một khoảng thời gian chờ mà vượt qua khoảng thời gian đó job sẽ không còn unique nữa. Để thực hiện điều này, bạn có thể định nghĩa các thuộc tính hoặc phương thức `uniqueId` và `uniqueFor` trên class job của bạn:

    <?php

    use App\Models\Product;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Contracts\Queue\ShouldBeUnique;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        /**
         * The product instance.
         *
         * @var \App\Product
         */
        public $product;

        /**
         * The number of seconds after which the job's unique lock will be released.
         *
         * @var int
         */
        public $uniqueFor = 3600;

        /**
         * Get the unique ID for the job.
         */
        public function uniqueId(): string
        {
            return $this->product->id;
        }
    }

Trong ví dụ trên, job `UpdateSearchIndex` là unique theo ID product. Vì vậy, mọi job mới được gửi mà có cùng ID product sẽ bị bỏ qua cho đến khi job hiện tại hoàn tất xử lý. Ngoài ra, nếu job hiện tại không được xử lý trong vòng một giờ, khóa unique sẽ được giải phóng và một job khác có cùng khóa unique có thể được gửi đến queue.

> [!WARNING]
> Nếu ứng dụng của bạn phân phối các job từ nhiều máy chủ web hoặc container, bạn nên đảm bảo là tất cả các máy chủ của bạn đang giao tiếp với cùng một máy chủ cache trung tâm để Laravel có thể xác định chính xác xem job có duy nhất hay không.

<a name="keeping-jobs-unique-until-processing-begins"></a>
#### Keeping Jobs Unique Until Processing Begins

Mặc định, các job unique sẽ được "mở khóa" sau khi một job hoàn tất quá trình xử lý hoặc bị thất bại trong tất cả các lần thử lại của nó. Tuy nhiên, có thể có những trường hợp bạn muốn job của mình được mở khóa ngay lập tức trước khi nó được xử lý. Để thực hiện điều này, job của bạn nên implement contract `ShouldBeUniqueUntilProcessing` thay vì contract `ShouldBeUnique`:

    <?php

    use App\Models\Product;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
    {
        // ...
    }

<a name="unique-job-locks"></a>
#### Unique Job Locks

Ở hậu trường, khi một job `ShouldBeUnique` được gửi đi, Laravel sẽ cố gắng lấy [lock](/docs/{{version}}/cache#atomic-locks) bằng khóa `uniqueId`. Nếu không lấy được khóa, job đó sẽ không được gửi đi. Khóa này sẽ được giải phóng khi một job hoàn tất quá trình xử lý hoặc thất bại trong tất cả các lần thử lại. Mặc định, Laravel sẽ sử dụng cache driver mặc định để lấy khóa này. Tuy nhiên, nếu bạn muốn sử dụng một driver khác để lấy khóa, bạn có thể định nghĩa một phương thức `uniqueVia` để trả về cache driver sẽ được sử dụng:

    use Illuminate\Contracts\Cache\Repository;
    use Illuminate\Support\Facades\Cache;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        ...

        /**
         * Get the cache driver for the unique job lock.
         */
        public function uniqueVia(): Repository
        {
            return Cache::driver('redis');
        }
    }

> [!NOTE]
> Nếu bạn chỉ cần giới hạn quá trình xử lý đồng thời của một job, bạn hãy sử dụng middleware job [`WithoutOverlapping`](/docs/{{version}}/queues#preventing-job-overlaps) thay thế.

<a name="encrypted-jobs"></a>
### Encrypted Jobs

Laravel cho phép bạn đảm bảo tính riêng tư và tính toàn vẹn dữ liệu của job thông qua [encryption](/docs/{{version}}/encryption). Để bắt đầu, bạn chỉ cần thêm interface `ShouldBeEncrypted` vào class của job. Sau khi interface này được thêm vào class, Laravel sẽ tự động mã hóa job của bạn trước khi đẩy nó vào queue:

    <?php

    use Illuminate\Contracts\Queue\ShouldBeEncrypted;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeEncrypted
    {
        // ...
    }

<a name="job-middleware"></a>
## Job Middleware

Job middleware cho phép bạn custom logic của toàn bộ việc chạy các queued job, giảm việc viết code trong các job đó. Ví dụ: phương thức `handle` sau đây có sử dụng các tính năng giới hạn tốc độ của Redis trong Laravel để chỉ cho phép cứ năm giây xử lý một job:

    use Illuminate\Support\Facades\Redis;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
            info('Lock obtained...');

            // Handle job...
        }, function () {
            // Could not obtain lock...

            return $this->release(5);
        });
    }

Mặc dù code này đúng, nhưng implementation của phương thức `handle` đã trở nên quá phức tạp vì nó không đồng nhất với logic giới hạn tốc độ của Redis. Ngoài ra, logic giới hạn tốc độ này cũng phải được copy cho bất kỳ job nào khác mà chúng ta muốn set giới hạn tốc độ.

Thay vì giới hạn tốc độ trong phương thức handle, chúng ta có thể định nghĩa một job middleware xử lý giới hạn tốc độ. Laravel không có một vị trí mặc định cho các job middleware này, vì vậy bạn có thể đặt job middleware ở bất kỳ đâu trong ứng dụng của bạn. Trong ví dụ này, chúng ta sẽ đặt middleware trong thư mục `app/Jobs/Middleware`:

    <?php

    namespace App\Jobs\Middleware;

    use Closure;
    use Illuminate\Support\Facades\Redis;

    class RateLimited
    {
        /**
         * Process the queued job.
         *
         * @param  \Closure(object): void  $next
         */
        public function handle(object $job, Closure $next): void
        {
            Redis::throttle('key')
                    ->block(0)->allow(1)->every(5)
                    ->then(function () use ($job, $next) {
                        // Lock obtained...

                        $next($job);
                    }, function () use ($job) {
                        // Could not obtain lock...

                        $job->release(5);
                    });
        }
    }

Như bạn có thể thấy, chẳng hạn như [route middleware](/docs/{{version}}/middleware), job middleware sẽ nhận vào một job đang được xử lý và lệnh callback sẽ được gọi để tiếp tục xử lý job đó.

Sau khi tạo xong job middleware, chúng ta có thể được gắn chúng vào một job bằng cách trả lại chúng từ phương thức `middleware` của job. Phương thức này không tồn tại trên các job được tạo bởi lệnh Artisan `make:job`, vì vậy bạn sẽ cần phải tự thêm nó vào định nghĩa job class của bạn:

    use App\Jobs\Middleware\RateLimited;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [new RateLimited];
    }

> [!NOTE]
> Middleware job cũng có thể được chỉ định cho các queueable event listeners, mailables, và notifications.

<a name="rate-limiting"></a>
### Giới hạn tỷ lệ

Mặc dù chúng ta chỉ trình bày cách viết middleware giới hạn tỷ lệ của riêng bạn, nhưng Laravel thực sự đã chứa một middleware giới hạn tỷ lệ mà bạn có thể sử dụng để xếp hạng các job giới hạn. Giống như [giới hạn tỷ lệ của route](/docs/{{version}}/routing#defining-rate-limiters), bộ giới hạn tỷ lệ job được định nghĩa bằng phương thức `for` của facade `RateLimiter`.

Ví dụ: bạn có thể muốn cho phép người dùng backup lại dữ liệu của họ mỗi giờ một lần trong khi không áp đặt giới hạn đó cho khách hàng cao cấp. Để thực hiện điều này, bạn có thể định nghĩa một `RateLimiter` trong phương thức `boot` của `AppServiceProvider` của bạn:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        RateLimiter::for('backups', function (object $job) {
            return $job->user->vipCustomer()
                        ? Limit::none()
                        : Limit::perHour(1)->by($job->user->id);
        });
    }

Trong ví dụ trên, chúng ta đã định nghĩa giới hạn tỷ lệ theo giờ; tuy nhiên, bạn có thể dễ dàng định nghĩa giới hạn tỷ lệ này dựa trên số phút bằng phương thức `perMinute`. Ngoài ra, bạn có thể truyền bất kỳ giá trị nào mà bạn muốn vào phương thức `by` của giới hạn tỷ lệ; tuy nhiên, giá trị này thường được sử dụng nhiều nhất để phân chia giới hạn tỷ lệ theo khách hàng:

    return Limit::perMinute(50)->by($job->user->id);

Khi bạn đã định nghĩa xong giới hạn tỷ lệ của bạn, bạn có thể gán giới hạn tỷ lệ này vào job của bạn bằng cách sử dụng middleware `Illuminate\Queue\Middleware\RateLimited`. Mỗi khi job mà vượt quá giới hạn tỷ lệ, middleware này sẽ giải phóng job trở lại về queue với một độ trễ thích hợp dựa trên khoảng thời gian giới hạn tỷ lệ mà bạn đã truyền vào.

    use Illuminate\Queue\Middleware\RateLimited;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [new RateLimited('backups')];
    }

Việc đưa một job có giới hạn tỷ lệ trở lại queue sẽ vẫn làm tăng tổng số `attempts` của job đó. Bạn có thể muốn điều chỉnh các thuộc tính `tries` và `maxExceptions` trên class job của bạn cho phù hợp. Hoặc, bạn có thể muốn sử dụng [phương thúc `retryUntil`](#time-based-attempts) để định nghĩa khoảng thời gian cho đến khi job không còn được thực hiện nữa.

Nếu bạn không muốn thử lại một job khi nó bị giới hạn tỷ lệ, bạn có thể sử dụng phương thức `dontRelease`:

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [(new RateLimited('backups'))->dontRelease()];
    }

> [!NOTE]
> Nếu đang sử dụng Redis, bạn có thể sử dụng middleware `Illuminate\Queue\Middleware\RateLimitedWithRedis`, middleware này được tinh chỉnh cho Redis và hiệu quả hơn middleware giới hạn tỷ lệ cơ bản.

<a name="preventing-job-overlaps"></a>
### Chặn Job chồng nhau

Laravel có chứa một middleware `Illuminate\Queue\Middleware\WithoutOverlapping` cho phép bạn ngăn chặn sự chồng nhau của job dựa trên một khóa tùy ý. Điều này có thể hữu ích khi một queued job đang sửa một tài nguyên mà mỗi lần chỉ nên được sửa bởi một job.

Ví dụ: hãy tưởng tượng bạn có một queued job cập nhật điểm tín dụng của người dùng và bạn muốn ngăn chặn sự trùng nhau của job cập nhật điểm tín dụng cho cùng một ID người dùng. Để thực hiện điều này, bạn có thể trả về middleware `WithoutOverlapping` từ phương thức `middleware` của job của bạn:

    use Illuminate\Queue\Middleware\WithoutOverlapping;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [new WithoutOverlapping($this->user->id)];
    }

Bất kỳ job nào mà cùng loại chồng nhau sẽ được giải phóng trở lại queue. Bạn cũng có thể chỉ định số giây mà job phải đợi trước khi job đó sẽ được thử lại:

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
    }

Nếu bạn muốn xóa ngay lập tức bất kỳ job nào chồng nhau để chúng không được thử lại bất kỳ lần nào nữa, bạn có thể sử dụng phương thức `dontRelease`:

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [(new WithoutOverlapping($this->order->id))->dontRelease()];
    }

Middleware `WithoutOverlapping` được thực hiện dựa trên tính năng atomic lock của Laravel. Thỉnh thoảng, job của bạn có thể bị lỗi hoặc hết thời gian chờ khiến cho khóa không được mở. Do đó, bạn có thể định nghĩa một thời gian hết hạn cho khóa bằng phương thức `expireAfter`. Ví dụ ở bên dưới sẽ bảo Laravel mở khóa `WithoutOverlapping` sau ba phút sau khi job được bắt đầu xử lý:

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [(new WithoutOverlapping($this->order->id))->expireAfter(180)];
    }

> [!WARNING]
> Middleware `WithoutOverlapping` yêu cầu một cache driver hỗ trợ [locks](/docs/{{version}}/cache#atomic-locks). Hiện tại, cache driver `memcached`, `redis`, `dynamodb`, `database`, `file` và `array` dã hỗ trợ atomic lock.

<a name="sharing-lock-keys"></a>
#### Sharing Lock Keys Across Job Classes

Mặc định, middleware `WithoutOverlapping` sẽ chỉ ngăn chặn các job chồng chéo lên nhau của cùng một class. Vì vậy, mặc dù hai class job khác nhau nhưng vẫn có thể sử dụng cùng một khóa, nên chúng sẽ không bị chặn khỏi việc chồng chéo. Tuy nhiên, bạn có thể hướng dẫn Laravel áp dụng khóa trên các class job bằng phương thức `shared`:

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

class ProviderIsDown
{
    // ...


    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}

class ProviderIsUp
{
    // ...


    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}
```

<a name="throttling-exceptions"></a>
### Ngoại lệ

Laravel có chứa một middleware `Illuminate\Queue\Middleware\ThrottlesExceptions` cho phép bạn điều tiết các ngoại lệ. Khi một job đưa ra một ngoại lệ nhất định, tất cả các bước tiếp theo để chạy job sẽ bị trì hoãn cho đến khi hết một khoảng thời gian nhất định. Middleware này đặc biệt hữu ích cho các job tương tác với các service của bên thứ ba.

Ví dụ: hãy tưởng tượng một queued job tương tác với một API của bên thứ ba bắt đầu đưa ra các ngoại lệ. Để điều tiết các ngoại lệ này, bạn có thể trả về middleware `ThrottlesExceptions` từ phương thức `middleware` trong job của bạn. Thông thường, middleware này phải được nối với một job mà implement phương thức [lần thử dựa trên thời gian](#time-based-attempts):

    use DateTime;
    use Illuminate\Queue\Middleware\ThrottlesExceptions;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [new ThrottlesExceptions(10, 5)];
    }

    /**
     * Determine the time at which the job should timeout.
     */
    public function retryUntil(): DateTime
    {
        return now()->addMinutes(5);
    }

Tham số đầu tiên mà được middleware chấp nhận là số lượng ngoại lệ mà job có thể đưa ra trước khi được điều chỉnh, trong khi tham số thứ hai là số phút mà trước khi job đó được thử lại sau khi nó được điều chỉnh. Trong code ví dụ ở trên, nếu job đưa ra 10 ngoại lệ trong vòng 5 phút, chúng ta sẽ đợi thêm 5 phút trước khi thử lại job.

Khi một job đưa ra một ngoại lệ nhưng vẫn chưa đạt đến ngưỡng ngoại lệ, job đó thường sẽ được thử lại ngay lập tức. Tuy nhiên, bạn có thể chỉ định số phút mà một job như vậy sẽ bị trì hoãn bằng cách gọi phương thức `backoff` khi gắn middleware vào job:

    use Illuminate\Queue\Middleware\ThrottlesExceptions;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [(new ThrottlesExceptions(10, 5))->backoff(5)];
    }

Ở bên trong, middleware này sẽ sử dụng cache system của Laravel để thực hiện giới hạn tỷ lệ và tên class của job sẽ được sử dụng để làm "khóa" của cache. Bạn có thể ghi đè khóa này bằng cách gọi phương thức `by` khi gắn middleware vào job của bạn. Điều này có thể hữu ích nếu bạn có nhiều job tương tác với cùng một service của bên thứ ba và bạn muốn chúng chia sẻ một "nhóm" điều tiết chung:

    use Illuminate\Queue\Middleware\ThrottlesExceptions;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [(new ThrottlesExceptions(10, 10))->by('key')];
    }

> [!NOTE]
> Nếu bạn đang sử dụng Redis, bạn có thể sử dụng middleware `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis`, middleware này được tinh chỉnh cho Redis và hiệu quả hơn middleware bình thường.

<a name="dispatching-jobs"></a>
## Gửi Job

Khi bạn đã viết xong các class job của bạn, bạn có thể dispatch nó bằng cách sử dụng phương thức `dispatch` trên chính job đó. Các tham số được truyền vào cho phương thức `dispatch` sẽ được truyền lại vào hàm khởi tạo của job:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         */
        public function store(Request $request): RedirectResponse
        {
            $podcast = Podcast::create(/* ... */);

            // ...

            ProcessPodcast::dispatch($podcast);

            return redirect('/podcasts');
        }
    }

Nếu bạn muốn gửi một job có điều kiện, bạn có thể sử dụng các phương thức `dispatchIf` và `dispatchUnless`:

    ProcessPodcast::dispatchIf($accountActive, $podcast);

    ProcessPodcast::dispatchUnless($accountSuspended, $podcast);

Trong các ứng dụng Laravel mới, driver `sync` là driver queue mặc định. Driver này sẽ chạy các job đồng bộ với reques hiện tại, thường thuận tiện trong quá trình phát triển local. Nếu bạn thực sự muốn bắt đầu queue các job này để xử lý dưới background, bạn có thể chỉ định một driver queue khác trong file cấu hình `config/queue.php` của ứng dụng.

<a name="delayed-dispatching"></a>
### Delay gửi

Nếu bạn muốn chỉ định rằng một job sẽ không được xử lý ngay bởi một queue worker, thì bạn có thể sử dụng phương thức `delay` khi đang dispatch một job. Ví dụ: hãy khai báo một job không nên được xử lý cho đến 10 phút sau khi job đó được dispatch:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         */
        public function store(Request $request): RedirectResponse
        {
            $podcast = Podcast::create(/* ... */);

            // ...

            ProcessPodcast::dispatch($podcast)
                        ->delay(now()->addMinutes(10));

            return redirect('/podcasts');
        }
    }

> [!WARNING]
> service SQS queue của Amazon có thời gian delay tối đa là 15 phút.

<a name="dispatching-after-the-response-is-sent-to-browser"></a>
#### Dispatching After The Response Is Sent To Browser

Ngoài ra, phương thức `dispatchAfterResponse` sẽ làm chậm việc gửi một job cho đến khi HTTP response được gửi về trình duyệt của người dùng nếu máy chủ web của bạn đang sử dụng FastCGI. Điều này vẫn sẽ cho phép người dùng bắt đầu sử dụng ứng dụng ngay cả khi queued job vẫn đang được thực hiện. Điều này thường chỉ được sử dụng cho các job ngắn thường một giây, chẳng hạn như việc gửi email. Vì chúng được xử lý trong request HTTP hiện tại nên các job được gửi theo cách này sẽ không yêu cầu queue worker phải chạy ngay để chúng được xử lý:

    use App\Jobs\SendNotification;

    SendNotification::dispatchAfterResponse();

Bạn cũng có thể `dispatch` một closure và kết hợp thêm phương thức `afterResponse` vào helper `dispatch` để thực hiện một closure sau khi HTTP response đã được gửi về trình duyệt:

    use App\Mail\WelcomeMessage;
    use Illuminate\Support\Facades\Mail;

    dispatch(function () {
        Mail::to('taylor@example.com')->send(new WelcomeMessage);
    })->afterResponse();

<a name="synchronous-dispatching"></a>
### Đồng bộ gửi

Nếu bạn muốn gửi một job được chạy ngay lập tức (một cách đồng bộ), bạn có thể sử dụng phương thức `dispatchSync`. Khi sử dụng phương thức này, job sẽ không được queue và sẽ được chạy ngay lập tức trong process hiện tại:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         */
        public function store(Request $request): RedirectResponse
        {
            $podcast = Podcast::create(/* ... */);

            // Create podcast...

            ProcessPodcast::dispatchSync($podcast);

            return redirect('/podcasts');
        }
    }

<a name="jobs-and-database-transactions"></a>
### Jobs và Database Transactions

Mặc dù việc gửi job trong các transaction của cơ sở dữ liệu là hoàn toàn ổn, nhưng bạn nên đặc biệt cẩn thận để đảm bảo rằng job của bạn thực sự có thể thực hiện thành công. Khi gửi một job trong một transaction, có thể job sẽ được xử lý bởi một worker trước khi transaction đó được thực hiện. Khi điều này xảy ra, mọi cập nhật mà bạn đã thực hiện đối với model hoặc bản ghi cơ sở dữ liệu trong transaction cơ sở dữ liệu có thể chưa được lưu vào trong cơ sở dữ liệu. Ngoài ra, mọi model hoặc bản ghi cơ sở dữ liệu được tạo trong transaction có thể không tồn tại trong cơ sở dữ liệu.

Rất may, Laravel cung cấp một số phương thức để giải quyết vấn đề này. Trước tiên, bạn có thể set tùy chọn kết nối `after_commit` trong mảng cấu hình của kết nối queue của bạn:

    'redis' => [
        'driver' => 'redis',
        // ...
        'after_commit' => true,
    ],

Khi tùy chọn `after_commit` được set là `true`, bạn có thể gửi job trong transaction của cơ sở dữ liệu; tuy nhiên, Laravel sẽ đợi cho đến khi tất cả các transaction của cơ sở dữ liệu được hoàn tất trước khi thực sự gửi job. Tất nhiên, nếu hiện tại không có transaction cơ sở dữ liệu nào thì job sẽ được gửi đi ngay lập tức.

Nếu một transaction bị roll back lại do một ngoại lệ xảy ra trong quá trình transaction, thì các job đã được gửi trong transaction đó sẽ bị loại bỏ.

> [!NOTE]
> Việc set tùy chọn cấu hình `after_commit` thành `true` cũng sẽ khiến mọi event listener, mailable, notification và các broadcast event mà đã được queue lại sẽ được gửi đi sau khi tất cả các transaction cơ sở dữ liệu được thực hiện xong.

<a name="specifying-commit-dispatch-behavior-inline"></a>
#### Specifying Commit Dispatch Behavior Inline

Nếu bạn không set tùy chọn cấu hình kết nối queue `after_commit` thành `true`, bạn vẫn có thể chỉ định một job cụ thể sẽ được gửi đi sau khi tất cả các transaction cơ sở dữ liệu đã được hoàn tất. Để thực hiện điều này, bạn có thể kết hợp thêm phương thức `afterCommit` vào sau phương thức gửi của bạn:

    use App\Jobs\ProcessPodcast;

    ProcessPodcast::dispatch($podcast)->afterCommit();

Tương tự, nếu tùy chọn cấu hình `after_commit` được set thành `true`, bạn có thể chỉ định một job cụ thể sẽ được gửi đi ngay lập tức mà không cần đợi bất kỳ transaction cơ sở dữ liệu nào được thực hiện:

    ProcessPodcast::dispatch($podcast)->beforeCommit();

<a name="job-chaining"></a>
### Kết hợp Job

Kết hợp job cho phép bạn khai báo một danh sách các queued job sẽ được chạy theo một trình tự nhất định sau khi job chính được thực thi thành công. Nếu một job trong danh sách bị thất bại, thì các job còn lại cũng sẽ không được chạy. Để thực hiện một danh sách queued job, bạn có thể sử dụng phương thức `chain` được cung cấp bởi facade `Bus`. Lệnh bus của Laravel là một component cấp thấp, chức năng gửi queued job được xây dựng dựa trên nó:

    use App\Jobs\OptimizePodcast;
    use App\Jobs\ProcessPodcast;
    use App\Jobs\ReleasePodcast;
    use Illuminate\Support\Facades\Bus;

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->dispatch();

Ngoài việc kết hợp các instance của job class, bạn cũng có thể kết hợp thêm các closures:

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        function () {
            Podcast::update(/* ... */);
        },
    ])->dispatch();

> [!WARNING]
> Việc xóa nhiều job bằng phương thức `$this->delete()` trong một job cụ thể sẽ không ngăn một chuỗi job ngừng xử lý. Chuỗi job sẽ chỉ bị ngừng xử lý nếu một job trong chuỗi job đó bị thất bại.

<a name="chain-connection-queue"></a>
#### Chain Connection và Queue

Nếu bạn muốn chỉ định kết nối và queue sẽ được sử dụng cho một chuỗi job, bạn có thể sử dụng phương thức `onConnection` và `onQueue`. Các phương thức này sẽ chỉ định kết nối queue và tên queue sẽ được sử dụng trừ khi queue job của bạn được chỉ định trong một kết nối hoặc một queue khác:

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->onConnection('redis')->onQueue('podcasts')->dispatch();

<a name="chain-failures"></a>
#### Chain Failures

Khi kết hợp các job, bạn có thể sử dụng phương thức `catch` để chỉ định một closure sẽ được gọi nếu một job trong chuỗi bị lỗi. Callback đã cho sẽ nhận vào instance `Throwable` gây ra lỗi job:

    use Illuminate\Support\Facades\Bus;
    use Throwable;

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->catch(function (Throwable $e) {
        // A job within the chain has failed...
    })->dispatch();

> [!WARNING]
> Vì các chuỗi callback sẽ được chuyển đổi và thực thi sau đó bởi Laravel queue, nên bạn không nên sử dụng biến `$this` trong các chuỗi callback.

<a name="customizing-the-queue-and-connection"></a>
### Tuỳ biến Queue và Connection

<a name="dispatching-to-a-particular-queue"></a>
#### Dispatching đến một Queue cụ thể

Bằng cách tạo các job đến các queue khác nhau, bạn có thể "phân loại" các queued job của bạn và thậm chí là ưu tiên bao nhiêu worker sẽ được gán cho mỗi queue. Hãy nhớ rằng, điều này không đẩy job đến các queue "connection" khác mà có ở trong file định nghĩa cấu hình queue của bạn, mà chỉ đẩy đến các queue có trong một connection cụ thể. Để khai báo queue, sử dụng phương thức `onQueue` khi gửi job:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         */
        public function store(Request $request): RedirectResponse
        {
            $podcast = Podcast::create(/* ... */);

            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onQueue('processing');

            return redirect('/podcasts');
        }
    }

Ngoài ra, bạn có thể chỉ định queue của job bằng cách gọi phương thức `onQueue` trong hàm khởi tạo của job:

    <?php

    namespace App\Jobs;

     use Illuminate\Bus\Queueable;
     use Illuminate\Contracts\Queue\ShouldQueue;
     use Illuminate\Foundation\Bus\Dispatchable;
     use Illuminate\Queue\InteractsWithQueue;
     use Illuminate\Queue\SerializesModels;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Create a new job instance.
         */
        public function __construct()
        {
            $this->onQueue('processing');
        }
    }

<a name="dispatching-to-a-particular-connection"></a>
#### Dispatching To A Particular Connection

Nếu application của bạn đang làm việc với nhiều queue connection, bạn có thể khai báo connection nào sẽ sử dụng cho job đó bằng cách sử dụng phương thức `onConnection`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         */
        public function store(Request $request): RedirectResponse
        {
            $podcast = Podcast::create(/* ... */);

            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onConnection('sqs');

            return redirect('/podcasts');
        }
    }

Bạn có thể kết hợp các phương thức `onConnection` và `onQueue` với nhau để khai báo connection và queue cho một job:

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');

Ngoài ra, bạn có thể chỉ định kết nối của job bằng cách gọi phương thức `onConnection` trong hàm khởi tạo của job:

    <?php

    namespace App\Jobs;

     use Illuminate\Bus\Queueable;
     use Illuminate\Contracts\Queue\ShouldQueue;
     use Illuminate\Foundation\Bus\Dispatchable;
     use Illuminate\Queue\InteractsWithQueue;
     use Illuminate\Queue\SerializesModels;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Create a new job instance.
         */
        public function __construct()
        {
            $this->onConnection('sqs');
        }
    }

<a name="max-job-attempts-and-timeout"></a>
### Khai báo số lần chạy Job tối đa / giá trị timeout

<a name="max-attempts"></a>
#### Max Attempts

Nếu một trong các queued job của bạn gặp lỗi, bạn có thể không muốn nó tiếp tục thử lại vô thời hạn. Do đó, Laravel cung cấp nhiều cách khác nhau để xác định số lần hoặc thời gian thực hiện một job.

Một cách tiếp cận để khai báo số lần tối đa mà một job có thể được chạy là thông qua switch `--tries` trên lệnh Artisan. Điều này sẽ áp dụng cho tất cả các job do worker này xử lý trừ khi job đang được xử lý chỉ định một số cụ thể mà job đó có thể được thực hiện:

```shell
php artisan queue:work --tries=3
```

Nếu một job vượt quá số lần thử tối đa, nó sẽ bị coi là một job "thất bại". Để biết thêm thông tin về cách xử lý những job thất bại, hãy tham khảo [tài liệu về job thất bại](#dealing-with-failed-jobs). Nếu `--tries=0` được cung cấp cho lệnh `queue:work`, thì job đó sẽ được thử lại vô số lần.

Bạn có thể thực hiện một cách tiếp cận chi tiết hơn bằng cách định nghĩa số lần chạy tối đa mà một job có thể thử trên chính class của job. Nếu số lần chạy tối đa được chỉ định trong job, thì nó sẽ ưu tiên giá trị `--tries` này hơn là giá trị được cung cấp trên dòng lệnh:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of times the job may be attempted.
         *
         * @var int
         */
        public $tries = 5;
    }

Nếu bạn cần kiểm số lần thử tối đa của một job cụ thể, bạn có thể định nghĩa phương thức `tries` trên job đó:

    /**
     * Determine number of times the job may be attempted.
     */
    public function tries(): int
    {
        return 5;
    }

<a name="time-based-attempts"></a>
#### Time Based Attempts

Thay thế cho việc định nghĩa số lần một job có thể được chạy trước khi nó thất bại, bạn có thể định nghĩa thời gian mà job đó sẽ không còn được thử lại. Điều này cho phép một job có thể được chạy thoải mái trong một khoảng thời gian nhất định. Để định nghĩa thời gian mà một job sẽ không còn được thử lại, hãy thêm phương thức `retryUntil` vào class job của bạn. Phương thức này sẽ trả về một instance `DateTime`:

    use DateTime;

    /**
     * Determine the time at which the job should timeout.
     */
    public function retryUntil(): DateTime
    {
        return now()->addMinutes(10);
    }

> [!NOTE]
> Bạn cũng có thể định nghĩa một thuộc tính `tries` hoặc phương thức `retryUntil` trên các [queued event listener](/docs/{{version}}/events#queued-event-listeners) của bạn.

<a name="max-exceptions"></a>
#### Max Exceptions

Thỉnh thoảng bạn có thể muốn chỉ định một job có thể được thử lại nhiều lần, nhưng sẽ thất bại nếu trong các lần thử lại được kích hoạt bởi một số lượng exception chưa được xử lý nhất định (ngược lại với việc được giải phóng trực tiếp bằng phương thức `release`). Để thực hiện điều này, bạn có thể định nghĩa một thuộc tính `maxExceptions` trên class job của bạn:

    <?php

    namespace App\Jobs;

    use Illuminate\Support\Facades\Redis;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The maximum number of unhandled exceptions to allow before failing.
         *
         * @var int
         */
        public $tries = 25;

        /**
         * The maximum number of exceptions to allow before failing.
         *
         * @var int
         */
        public $maxExceptions = 3;

        /**
         * Execute the job.
         */
        public function handle(): void
        {
            Redis::throttle('key')->allow(10)->every(60)->then(function () {
                // Lock obtained, process the podcast...
            }, function () {
                // Unable to obtain lock...
                return $this->release(10);
            });
        }
    }

Trong ví dụ này, job sẽ được giải phóng trong 10 giây nếu ứng dụng không thể lấy được Redis lock và sẽ tiếp tục được thử lại tối đa 25 lần. Tuy nhiên, job sẽ thất bại nếu job đưa ra quá ba exception.

<a name="timeout"></a>
#### Timeout

Thông thường, bạn có thể ước lượng queued job của bạn sẽ mất bao lâu thời gian để chạy. Vì lý do này, Laravel cho phép bạn chỉ định giá trị "timeout". Mặc định, giá trị timeout này là 60 giây. Nếu một job đang xử lý lâu hơn số giây được chỉ định bởi giá trị timeout, thì worker đang xử lý job đó sẽ bị thoát ra với một lỗi. Thông thường, worker sẽ được khởi động lại tự động bởi [trình quản lý process được cấu hình trên máy chủ của bạn](#supervisor-configuration).

Thời gian hết hạn của một job có thể được khai báo bằng cách sử dụng switch `--timeout` trên lệnh Artisan:

```shell
php artisan queue:work --timeout=30
```

Nếu job vượt quá số lần thử tối đa do liên tục hết thời gian chờ, nó sẽ bị đánh dấu là thất bại.

Bạn cũng có thể định nghĩa thời gian hết hạn của một job trên chính class của job đó. Nếu thời gian hết hạn được khai báo trong job, nó sẽ được ưu tiên hơn bất kỳ thời gian hết hạn nào được khai báo trên dòng lệnh:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of seconds the job can run before timing out.
         *
         * @var int
         */
        public $timeout = 120;
    }

Thỉnh thoảng, các process IO blocking như socket hoặc outgoing HTTP connection có thể không tuân theo thời gian hết hạn đã được chỉ định của bạn. Do đó, khi sử dụng các tính năng này, bạn cũng nên cố gắng chỉ định một thời gian hết hạn bằng cách sử dụng các API của chúng. Ví dụ: khi sử dụng Guzzle, bạn phải luôn chỉ định một connection và một giá trị mà request sẽ hết hạn.

> [!WARNING]
> PHP extension `pcntl` phải được cài đặt để chỉ định thời gian timeout của job. Ngoài ra, giá trị "timeout" của job phải luôn nhỏ hơn giá trị ["retry after"](#job-expiration) của nó. Nếu không, job có thể bị thử lại trước khi hoàn tất công việc hoặc hết thời gian timeout.

<a name="failing-on-timeout"></a>
#### Failing On Timeout

Nếu bạn muốn chỉ định một job phải được đánh dấu là [thất bại](#dealing-with-failed-jobs) khi hết thời gian chờ, bạn có thể định nghĩa thuộc tính `$failOnTimeout` trên class job:

```php
/**
 * Indicate if the job should be marked as failed on timeout.
 *
 * @var bool
 */
public $failOnTimeout = true;
```

<a name="error-handling"></a>
### Error Handling

Nếu một ngoại lệ được đưa ra trong khi một job đang được xử lý, job đó sẽ tự động được giải phóng trở lại về queue để có thể thử lại. Job đó sẽ tiếp tục được thực hiện cho đến khi nó được thử với số lần tối đa mà ứng dụng của bạn cho phép. Số lần thử tối đa được định nghĩa bởi switch `--tries` được sử dụng trong lệnh Artisan `queue:work`. Ngoài ra, số lần thử tối đa cũng có thể được định nghĩa trên chính class job đó. Thông tin thêm về cách chạy queue worker [có thể tìm thấy bên dưới](#running-the-queue-worker).

<a name="manually-releasing-a-job"></a>
#### Manually Releasing A Job

Thỉnh thoảng bạn có thể muốn đưa một job trở lại về queue theo cách thủ công để có thể thử lại sau đó. Bạn có thể thực hiện điều này bằng cách gọi phương thức `release`:

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // ...

        $this->release();
    }

Mặc định, phương thức `release` sẽ giải phóng job trở lại queue ngay lập tức. Tuy nhiên, bạn có thể ra lệnh cho queue sẽ không xử lý job cho đến khi hết một số giây nhất định bằng cách truyền một số nguyên hoặc một date vào phương thức `release`:

    $this->release(10);

    $this->release(now()->addSeconds(10));

<a name="manually-failing-a-job"></a>
#### Manually Failing A Job

Đôi khi bạn có thể cần đánh dấu một job là "thất bại". Để làm như vậy, bạn có thể gọi phương thức `fail`:

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // ...

        $this->fail();
    }

Nếu bạn muốn đánh dấu job của bạn là thất bại vì một ngoại lệ mà bạn đã gặp phải, bạn có thể truyền ngoại lệ đó cho phương thức `fail`. Hoặc để thuận tiện hơn, bạn có thể truyền một thông báo lỗi dưới dạng string để nó sẽ được chuyển đổi thành ngoại lệ cho bạn:

    $this->fail($exception);

    $this->fail('Something went wrong.');

> [!NOTE]
> Để biết thêm thông tin về các job thất bại, hãy xem [tài liệu về cách xử lý cho các job thất bại](#dealing-with-failed-jobs).

<a name="job-batching"></a>
## Job Batching

Job batching của Laravel cho phép bạn dễ dàng thực hiện một loạt job và sau đó thực hiện một số hành động khi một loạt job đó đã hoàn thành việc thực thi. Trước khi bắt đầu, bạn nên tạo một migration cơ sở dữ liệu để tạo một bảng mà sẽ chứa các thông tin meta về các batch job của bạn, chẳng hạn như tỷ lệ hoàn thành của chúng. Migration này có thể được tạo bằng lệnh Artisan `queue:batches-table`:

```shell
php artisan queue:batches-table

php artisan migrate
```

<a name="defining-batchable-jobs"></a>
### Định nghĩa Batchable Jobs

Để định nghĩa một batch job, bạn nên [tạo một queue job](#creating-jobs) như bình thường; tuy nhiên, bạn nên thêm trait `Illuminate\Bus\Batchable` vào class job. Trait này cung cấp quyền truy cập vào phương thức `batch` có thể được sử dụng để lấy ra các batch hiện có mà job đang được thực thi:

    <?php

    namespace App\Jobs;

    use Illuminate\Bus\Batchable;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class ImportCsv implements ShouldQueue
    {
        use Batchable, Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Execute the job.
         */
        public function handle(): void
        {
            if ($this->batch()->cancelled()) {
                // Determine if the batch has been cancelled...

                return;
            }

            // Import a portion of the CSV file...
        }
    }

<a name="dispatching-batches"></a>
### Gửi Batches

Để gửi một batch job, bạn nên sử dụng phương thức `batch` của facade `Bus`. Tất nhiên, việc tạo batch chủ yếu hữu ích khi kết hợp với các lệnh callback. Vì vậy, bạn có thể sử dụng các phương thức `then`, `catch` và `final` để định nghĩa các lệnh callback cho batch. Mỗi lệnh callback này sẽ nhận vào một instance `Illuminate\Bus\Batch` khi chúng được gọi. Trong ví dụ này, chúng ta sẽ tưởng tượng là chúng ta đang queue một batch job mà mỗi job sẽ xử lý một số hàng nhất định từ file CSV:

    use App\Jobs\ImportCsv;
    use Illuminate\Bus\Batch;
    use Illuminate\Support\Facades\Bus;
    use Throwable;

    $batch = Bus::batch([
        new ImportCsv(1, 100),
        new ImportCsv(101, 200),
        new ImportCsv(201, 300),
        new ImportCsv(301, 400),
        new ImportCsv(401, 500),
    ])->before(function (Batch $batch) {
        // The batch has been created but no jobs have been added...
    })->progress(function (Batch $batch) {
        // A single job has completed successfully...
    })->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->catch(function (Batch $batch, Throwable $e) {
        // First batch job failure detected...
    })->finally(function (Batch $batch) {
        // The batch has finished executing...
    })->dispatch();

    return $batch->id;

ID của batch có thể được lấy ra thông qua thuộc tính `$batch->id`, nó có thể được sử dụng để [truy vấn lệnh bus của Laravel](#inspecting-batches) để biết thêm thông tin về batch sau khi nó được gửi đi.

> [!WARNING]
> Vì các lệnh callback batch được serialize và thực thi sau đó bởi Laravel queue, nên bạn không nên sử dụng biến `$this` trong các lệnh callback.

<a name="naming-batches"></a>
#### Naming Batches

Một số công cụ như Laravel Horizon và Laravel Telescope có thể cung cấp thông tin gỡ lỗi thân thiện hơn cho các batch nếu các batch đó đã được đặt tên. Để gán tên cho một batch, bạn có thể gọi phương thức `name` trong khi định nghĩa batch:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->name('Import CSV')->dispatch();

<a name="batch-connection-queue"></a>
#### Batch Connection và Queue

Nếu bạn muốn chỉ định kết nối và queue nào sẽ được sử dụng cho các batch job, bạn có thể sử dụng các phương thức `onConnection` và `onQueue`. Tất cả các batch job sẽ phải chạy trong cùng một kết nối và queue:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->onConnection('redis')->onQueue('imports')->dispatch();

<a name="chains-and-batches"></a>
### Chains and Batches

Bạn có thể định nghĩa một tập hợp gồm các [chuỗi job](#job-chaining) trong một batch bằng cách set các chuỗi job đó vào trong một mảng. Ví dụ: chúng ta có thể thực hiện song song hai chuỗi job và thực hiện lệnh callback khi cả hai chuỗi job đã được xử lý xong:

    use App\Jobs\ReleasePodcast;
    use App\Jobs\SendPodcastReleaseNotification;
    use Illuminate\Bus\Batch;
    use Illuminate\Support\Facades\Bus;

    Bus::batch([
        [
            new ReleasePodcast(1),
            new SendPodcastReleaseNotification(1),
        ],
        [
            new ReleasePodcast(2),
            new SendPodcastReleaseNotification(2),
        ],
    ])->then(function (Batch $batch) {
        // ...
    })->dispatch();

Ngược lại, bạn có thể chạy nhiều batch job trong một [chuỗi job](#job-chaining) bằng cách định nghĩa các batch này trong một chuỗi. Ví dụ, trước tiên bạn có thể chạy một batch job để phát hành nhiều podcast sau đó là một batch job khác để gửi thông báo phát hành:

    use App\Jobs\FlushPodcastCache;
    use App\Jobs\ReleasePodcast;
    use App\Jobs\SendPodcastReleaseNotification;
    use Illuminate\Support\Facades\Bus;

    Bus::chain([
        new FlushPodcastCache,
        Bus::batch([
            new ReleasePodcast(1),
            new ReleasePodcast(2),
        ]),
        Bus::batch([
            new SendPodcastReleaseNotification(1),
            new SendPodcastReleaseNotification(2),
        ]),
    ])->dispatch();

<a name="adding-jobs-to-batches"></a>
### Thêm Jobs vào Batches

Thỉnh thoảng việc thêm một job vào trong một batch từ bên trong một job có thể có hữu ích. Điều này có thể có hữu ích khi bạn cần xử lý hàng nghìn job mà có thể mất quá nhiều thời gian để gửi đi trong một request web. Vì vậy, thay vào đó, bạn có thể muốn gửi một loạt job "loader" đầu tiên để cung cấp cho batch đó rồi sẽ thêm nhiều job hơn nữa vào sau đó:

    $batch = Bus::batch([
        new LoadImportBatch,
        new LoadImportBatch,
        new LoadImportBatch,
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->name('Import Contacts')->dispatch();

Trong ví dụ này, chúng ta sẽ sử dụng job `LoadImportBatch` để tái tạo lại batch với các job bổ sung. Để thực hiện điều này, chúng ta có thể sử dụng phương thức `add` trên instance batch có thể được truy cập thông qua phương thức `batch` của job:

    use App\Jobs\ImportContacts;
    use Illuminate\Support\Collection;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            return;
        }

        $this->batch()->add(Collection::times(1000, function () {
            return new ImportContacts;
        }));
    }

> [!WARNING]
> Bạn chỉ có thể thêm job vào một batch từ bên trong job thuộc cùng một batch.

<a name="inspecting-batches"></a>
### Kiểm tra Batches

Instance `Illuminate\Bus\Batch` được cung cấp cho lệnh callback batch sẽ có nhiều thuộc tính và phương thức khác nhau để hỗ trợ bạn tương tác và kiểm tra một batch job nhất định:

    // The UUID of the batch...
    $batch->id;

    // The name of the batch (if applicable)...
    $batch->name;

    // The number of jobs assigned to the batch...
    $batch->totalJobs;

    // The number of jobs that have not been processed by the queue...
    $batch->pendingJobs;

    // The number of jobs that have failed...
    $batch->failedJobs;

    // The number of jobs that have been processed thus far...
    $batch->processedJobs();

    // The completion percentage of the batch (0-100)...
    $batch->progress();

    // Indicates if the batch has finished executing...
    $batch->finished();

    // Cancel the execution of the batch...
    $batch->cancel();

    // Indicates if the batch has been cancelled...
    $batch->cancelled();

<a name="returning-batches-from-routes"></a>
#### Returning Batches From Routes

Tất cả các instance `Illuminate\Bus\Batch` đều có thể serialize JSON, nghĩa là bạn có thể trả về chúng trực tiếp từ một trong các route của ứng dụng để lấy ra payload JSON chứa thông tin về batch, bao gồm cả tiến trình hoàn thành của nó. Điều này giúp việc hiển thị thêm thông tin về tiến trình hoàn thành của batch trong giao diện người dùng ứng dụng của bạn trở nên thuận tiện.

Để lấy ra một batch theo ID của nó, bạn có thể sử dụng phương thức `findBatch` của facade `Bus`:

    use Illuminate\Support\Facades\Bus;
    use Illuminate\Support\Facades\Route;

    Route::get('/batch/{batchId}', function (string $batchId) {
        return Bus::findBatch($batchId);
    });

<a name="cancelling-batches"></a>
### Huỷ Batches

Thỉnh thoảng bạn có thể cần hủy việc thực thi của một batch nhất định. Điều này có thể được thực hiện bằng cách gọi phương thức `cancel` trên instance `Illuminate\Bus\Batch`:

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        if ($this->user->exceedsImportLimit()) {
            return $this->batch()->cancel();
        }

        if ($this->batch()->cancelled()) {
            return;
        }
    }

Như bạn có thể thấy trong ví dụ trước, các batch job thường phải xác định xem batch của nó đã bị hủy chưa trước khi tiếp tục chạy. Tuy nhiên, để thuận tiện, bạn có thể gán [middleware](#job-middleware) `SkipIfBatchCancelled` cho job. Như tên gọi của nó, middleware này sẽ hướng dẫn Laravel không xử lý job này nếu batch tương ứng của nó đã bị hủy:

    use Illuminate\Queue\Middleware\SkipIfBatchCancelled;

    /**
     * Get the middleware the job should pass through.
     */
    public function middleware(): array
    {
        return [new SkipIfBatchCancelled];
    }

<a name="batch-failures"></a>
### Batch Failures

Khi một batch job thất bại, lệnh callback `catch` (nếu được chỉ định) sẽ được gọi. Lệnh callback này chỉ được gọi cho job đầu tiên bị thất bại trong batch.

<a name="allowing-failures"></a>
#### Allowing Failures

Khi một job trong một batch bị thất bại, Laravel sẽ tự động đánh dấu batch đó là "cancelled". Nếu muốn, bạn có thể vô hiệu hóa hành vi này để khi job thất bại thì không tự động đánh dấu batch là cancelled. Điều này có thể được thực hiện bằng cách gọi phương thức `allowFailures` trong khi gửi batch:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // All jobs completed successfully...
    })->allowFailures()->dispatch();

<a name="retrying-failed-batch-jobs"></a>
#### Retrying Failed Batch Jobs

Để thuận tiện, Laravel cung cấp lệnh Artisan `queue:retry-batch` cho phép bạn dễ dàng thử lại tất cả các job thất bại có trong một batch nhất định. Lệnh `queue:retry-batch` sẽ chấp nhận UUID của batch mà có job thất bại cần được thử lại:

```shell
php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5
```

<a name="pruning-batches"></a>
### Xoá Batches

Nếu không xoá, bảng `job_batches` có thể tích lũy các record rất nhanh. Để giảm thiểu tình trạng này, bạn nên [schedule](/docs/{{version}}/scheduling) lệnh Artisan `queue:prune-batches` để chạy hàng ngày:

    $schedule->command('queue:prune-batches')->daily();

Mặc định, tất cả các batch đã hoàn thành quá 24 giờ sẽ bị xoá. Bạn có thể sử dụng tùy chọn `hours` khi gọi command để xác định thời gian lưu giữ dữ liệu batch. Ví dụ: command sau sẽ xóa tất cả các batch đã hoàn thành hơn 48 giờ trước:

    $schedule->command('queue:prune-batches --hours=48')->daily();

Thỉnh thoảng, bảng `jobs_batches` của bạn có thể tích lũy các record batch cho các batch chưa được hoàn thành, chẳng hạn như các batch có job không thành công và job đó chưa bao giờ được thử lại thành công. Bạn có thể hướng dẫn lệnh `queue:prune-batches` để xoá các record batch chưa hoàn thành này bằng tùy chọn `unfinished`:

    $schedule->command('queue:prune-batches --hours=48 --unfinished=72')->daily();

Tương tự như vậy, bảng `jobs_batches` của bạn cũng có thể tích lũy các record batch đã bị hủy một cách rất nhanh. Bạn có thể hướng dẫn lệnh `queue:prune-batches` để xoá bỏ một phần các batch record đã bị hủy bằng tùy chọn `cancelled`:

    $schedule->command('queue:prune-batches --hours=48 --cancelled=72')->daily();

<a name="storing-batches-in-dynamodb"></a>
### Lưu batches trong DynamoDB

Laravel cũng cung cấp hỗ trợ lưu thông tin meta của batch trong [DynamoDB](https://aws.amazon.com/dynamodb) thay cho cơ sở dữ liệu quan hệ. Tuy nhiên, bạn sẽ cần phải tạo thủ công một bảng DynamoDB để lưu trữ tất cả các bản ghi batch.

Thông thường, bảng này sẽ được đặt tên là `job_batches`, nhưng bạn nên đặt tên bảng này dựa theo giá trị cấu hình `queue.batching.table` trong file cấu hình `queue` của ứng dụng.

<a name="dynamodb-batch-table-configuration"></a>
#### DynamoDB Batch Table Configuration

Bảng `job_batches` phải có một khóa chính dạng string có tên là `application` và một khóa chính khác cũng là ở dạng string và có tên là `id`. Phần `application` của khóa sẽ chứa tên ứng dụng của bạn như được định nghĩa bởi giá trị cấu hình `name` trong file cấu hình `app` của ứng dụng. Vì tên ứng dụng là một phần của khóa trong bảng DynamoDB, nên bạn có thể sử dụng cùng một bảng để lưu trữ các batch job cho nhiều ứng dụng Laravel khác nhau.

Ngoài ra, bạn có thể định nghĩa thuộc tính `ttl` cho bảng của bạn nếu bạn muốn tận dụng tính năng [loại bỏ batch tự động](#pruning-batches-in-dynamodb).

<a name="dynamodb-configuration"></a>
#### DynamoDB Configuration

Tiếp theo, hãy cài đặt AWS SDK để ứng dụng Laravel của bạn có thể giao tiếp với Amazon DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Sau đó, set giá trị của tùy chọn cấu hình `queue.batching.driver` thành `dynamodb`. Ngoài ra, bạn nên định nghĩa các tùy chọn cấu hình `key`, `secret` và `region` trong mảng cấu hình `batching`. Các tùy chọn này sẽ được sử dụng để xác thực với AWS. Khi sử dụng driver `dynamodb`, tùy chọn cấu hình `queue.batching.database` sẽ không cần thiết nữa:

```php
'batching' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
],
```

<a name="pruning-batches-in-dynamodb"></a>
#### Pruning Batches in DynamoDB

Khi sử dụng [DynamoDB](https://aws.amazon.com/dynamodb) để lưu trữ thông tin batch job, các lệnh xoá thông thường được sử dụng để xoá các batch được lưu trữ trong cơ sở dữ liệu quan hệ sẽ không còn hoạt động được nữa. Thay vào đó, bạn có thể sử dụng [chức năng TTL native của DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html) để tự động xóa các bản ghi cho các batch cũ.

Nếu bạn định nghĩa bảng DynamoDB của bạn bằng thuộc tính `ttl`, bạn có thể định nghĩa thêm các tham số cấu hình để hướng dẫn Laravel về cách loại bỏ các bản ghi batch cũ. Giá trị cấu hình `queue.batching.ttl_attribute` sẽ định nghĩa tên của thuộc tính TTL, trong khi giá trị cấu hình `queue.batching.ttl` định nghĩa số giây mà sau đó một bản ghi batch có thể bị xóa ra khỏi bảng DynamoDB, so với lần cuối cùng mà bản ghi đó được cập nhật:

```php
'batching' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
    'ttl_attribute' => 'ttl',
    'ttl' => 60 * 60 * 24 * 7, // 7 days...
],
```

<a name="queueing-closures"></a>
## Queueing Closures

Thay vì gửi một job loại class vào queue, thì bạn cũng có thể gửi một closure. Điều này rất tốt cho các công việc cần nhanh chóng, đơn giản và được thực hiện bên ngoài chu kỳ request hiện tại. Khi gửi các closure đến queue, nội dung code của closure sẽ được ký bằng mật mã để không thể sửa đổi nó trong quá trình vận chuyển:

    $podcast = App\Podcast::find(1);

    dispatch(function () use ($podcast) {
        $podcast->publish();
    });

Bằng cách sử dụng phương thức `catch`, bạn có thể cung cấp một closure sẽ được thực thi nếu queued closure không hoàn thành sau khi dùng hết tất cả [các lần thử lại đã được cấu hình](#max-job-attempts-and-timeout):

    use Throwable;

    dispatch(function () use ($podcast) {
        $podcast->publish();
    })->catch(function (Throwable $e) {
        // This job has failed...
    });

> [!WARNING]
> Vì lệnh callback `catch` sẽ được chuyển đổi và thực thi sau đó bởi Laravel queue, nên bạn không nên sử dụng biến `$this` trong lệnh callback `catch`.

<a name="running-the-queue-worker"></a>
## Chạy Queue Worker

<a name="the-queue-work-command"></a>
### Lệnh `queue:work`

Laravel có chứa một lệnh Artisan sẽ chạy một queue worker và thực hiện các job mới khi chúng được tạo lên queue. Bạn có thể chạy worker đó bằng lệnh Artisan `queue:work`. Lưu ý rằng một khi lệnh `queue:work` đã được chạy, thì nó sẽ tiếp tục chạy cho đến khi nào nó được dừng bằng cách thủ công hoặc bạn đóng terminal của bạn:

```shell
php artisan queue:work
```

> [!NOTE]
> Để giữ cho process `queue:work` luôn hoạt động trong background, bạn nên sử dụng một trình giám sát process, chẳng hạn như [Supervisor](#supervisor-configuration) để đảm bảo rằng queue worker không bị dừng giữa chừng.

Bạn có thể thêm flag `-v` khi gọi lệnh `queue:work` nếu bạn muốn ID của job đã xử lý được thêm vào output của command này:

```shell
php artisan queue:work -v
```

Hãy nhớ rằng, các queue worker là các process tồn tại lâu dài và lưu trữ trạng thái của application vào trong bộ nhớ. Do đó, chúng sẽ không nhận biết dược những thay đổi trong source code của bạn sau khi bạn đã chạy chúng. Vì vậy, trong khi quá trình deploy của bạn, hãy đảm bảo là [bạn đã khởi động lại queue worker của bạn](#queue-workers-and-deployment). Ngoài ra, hãy nhớ rằng bất kỳ trạng thái tĩnh nào được tạo hoặc được sửa bởi ứng dụng của bạn cũng sẽ không được tự động reset giữa các job.

Ngoài ra, bạn có thể chạy lệnh `queue:listen`. Khi sử dụng lệnh `queue:listen`, bạn không phải khởi động lại worker theo cách thủ công khi bạn muốn reload lại code đã cập nhật hoặc reset lại trạng thái ứng dụng; tuy nhiên, lệnh này kém hiệu quả hơn đáng kể so với lệnh `queue:work`:

```shell
php artisan queue:listen
```

<a name="running-multiple-queue-workers"></a>
#### Running Multiple Queue Workers

Để chỉ định nhiều worker vào một queue và xử lý job đồng thời, bạn chỉ cần start nhiều process `queue:work`. Việc này có thể được thực hiện local thông qua nhiều tab trong terminal của bạn hoặc trong production bằng cách sử dụng cài đặt cấu hình của trình quản lý process của bạn. [Khi sử dụng Supervisor](#supervisor-configuration), bạn có thể sử dụng giá trị cấu hình `numprocs`.

<a name="specifying-the-connection-queue"></a>
#### Specifying The Connection & Queue

Bạn cũng có thể khai báo queue connection mà worker sẽ sử dụng. Tên connection được truyền vào lệnh `work` phải tương ứng với một trong các connection đã được khai báo ở trong file cấu hình `config/queue.php` của bạn:

```shell
php artisan queue:work redis
```

Mặc định, lệnh `queue:work` chỉ xử lý các job cho queue mặc định trên một kết nối nhất định. Tuy nhiên, bạn cũng có thể tùy chỉnh queue worker của bạn nhiều hơn nữa bằng cách chỉ xử lý các queue cụ thể cho một connection nhất định. Ví dụ: nếu tất cả các email của bạn được xử lý trong queue `emails` trên một connection là `redis`, thì bạn có thể gọi lệnh sau để chạy một worker xử lý cho riêng queue đó:

```shell
php artisan queue:work redis --queue=emails
```

<a name="processing-a-specified-number-of-jobs"></a>
#### Processing A Specified Number Of Jobs

Tùy chọn `--once` có thể được sử dụng để lệnh worker chỉ xử lý một job duy nhất từ queue:

```shell
php artisan queue:work --once
```

Tùy chọn `--max-jobs` có thể được sử dụng để hướng dẫn worker xử lý một số lượng job nhất định rồi thoát ra. Tùy chọn này có thể hữu ích khi kết hợp với [Supervisor](#supervisor-configuration) để worker của bạn tự động khởi động lại sau khi xử lý một số job nhất định, giải phóng mọi bộ nhớ mà nó có thể đã tích lũy:

```shell
php artisan queue:work --max-jobs=1000
```

<a name="processing-all-queued-jobs-then-exiting"></a>
#### Processing All Queued Jobs và Then Exiting

Tùy chọn `--stop-when-empty` có thể được sử dụng để hướng dẫn worker xử lý tất cả các job và sau đó sẽ thoát nếu không còn job nữa. Tùy chọn này có thể hữu ích khi đang thực hiện các queue Laravel trong một Docker container nếu bạn muốn tắt container đó sau khi queue được trống:

```shell
php artisan queue:work --stop-when-empty
```

<a name="processing-jobs-for-a-given-number-of-seconds"></a>
#### Processing Jobs For A Given Number Of Seconds

Tùy chọn `--max-time` có thể được sử dụng để hướng dẫn worker xử lý job trong số giây nhất định rồi thoát ra. Tùy chọn này có thể hữu ích khi kết hợp với [Supervisor](#supervisor-configuration) để worker của bạn tự động khởi động lại sau khi xử lý job trong một khoảng thời gian nhất định, giải phóng mọi bộ nhớ mà nó có thể đã tích lũy:

```shell
# Process jobs for one hour and then exit...
php artisan queue:work --max-time=3600
```

<a name="worker-sleep-duration"></a>
#### Worker Sleep Duration

Khi các job có sẵn trên queue, worker sẽ tiếp tục xử lý các job mà không có độ trễ ở giữa chúng. Tuy nhiên, tùy chọn `sleep` sẽ xác định worker sẽ "sleep" bao nhiêu giây nếu không có job mới nào. Trong khi sleep, worker sẽ không xử lý bất kỳ job mới nào - job sẽ được xử lý sau khi worker thức dậy trở lại.

```shell
php artisan queue:work --sleep=3
```

<a name="maintenance-mode-queues"></a>
#### Maintenance Mode and Queues

Trong khi ứng dụng của bạn đang ở trong [chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode), thì sẽ không có một queued job được xử lý. Các job sẽ tiếp tục sẽ được xử lý bình thường khi ứng dụng ra khỏi chế độ bảo trì.

Để bắt những queue worker của bạn xử lý tiếp job ngay cả khi chế độ bảo trì được bật, bạn có thể sử dụng tùy chọn `--force`:

```shell
php artisan queue:work --force
```

<a name="resource-considerations"></a>
#### Resource Considerations

Daemon queue worker sẽ không "khởi động lại" framework trước khi xử lý mỗi job. Do đó, bạn nên giải phóng tất cả resources nặng sau khi hoàn thành xử lý job. Ví dụ, nếu bạn đang thực hiện tác vụ chỉnh sửa ảnh với thư viện GD, bạn nên giải phóng bộ nhớ với câu lệnh `imagedestroy` khi bạn hoàn thành việc xử lý ảnh đó.

<a name="queue-priorities"></a>
### Queue ưu tiên

Thỉnh thoảng bạn có thể muốn ưu tiên xử lý một queue. Ví dụ, trong file cấu hình `config/queue.php`, bạn có thể set mặc định `queue` cho connection `redis` là `low`. Tuy nhiên, đôi khi bạn có thể muốn tạo ra một job ở trên queue ưu tiên `high` như sau:

    dispatch((new Job)->onQueue('high'));

Để chạy một worker cho việc xử lý các queue job `high` trước rồi sau đó mới xử lý đến các job trong queue `low`, thì bạn có thể truyền vào một danh sách tên queue sẽ phân cách nhau bằng dấu phẩy cho lệnh `work`:

```shell
php artisan queue:work --queue=high,low
```

<a name="queue-workers-and-deployment"></a>
### Queue Worker và Deployment

Vì queue worker là các process tồn tại lâu dài, nên nếu không được khởi động lại, thì chúng sẽ không biết được các thay đổi trên code của bạn. Vì vậy, cách đơn giản nhất để deploy một application sử dụng queue worker là khởi động lại worker trong quá trình deploy của bạn. Bạn có thể khởi động lại tất cả các worker bằng cách chạy lệnh `queue:restart`:

```shell
php artisan queue:restart
```

Để không làm job đó bị mất, lệnh này sẽ thoát tất cả các queue workers sau khi chúng xử lý xong các job hiện tại. Vì các queue worker sẽ thoát khi lệnh `queue:restart` được chạy, vì thế bạn nên chạy một process quản lý, chẳng hạn như [Supervisor](#supervisor-configuration) để tự động khởi động lại các queue worker đó.

> [!NOTE]
> Queue sẽ sử dụng [cache](/docs/{{version}}/cache) để lưu trữ tín hiệu khởi động lại, vì vậy bạn nên kiểm tra driver cache đã được cấu hình cho application của bạn đúng chưa trước khi sử dụng tính năng này.

<a name="job-expirations-and-timeouts"></a>
### Job hết hạn và timeout

<a name="job-expiration"></a>
#### Job Expiration

Trong file cấu hình `config/queue.php` của bạn, mỗi queue connection sẽ định nghĩa một tùy chọn `retry_after`. Tùy chọn này sẽ khai báo cho queue connection biết sẽ phải đợi bao nhiêu giây trước khi chạy lại một job đang được xử lý. Ví dụ: nếu giá trị của `retry_after` được set là `90`, thì job đó sẽ được giải phóng trở lại vào queue nếu nó đã được xử lý quá 90 giây mà không bị giải phóng hoặc xóa. Thông thường, bạn nên set giá trị `retry_after` là số giây tối đa mà một job của bạn có thể sẽ mất để hoàn thành tất cả xử lý.

> [!WARNING]
> Chỉ có queue connection của Amazon SQS sẽ không chứa giá trị `retry_after`. SQS sẽ retry một job dựa trên [Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) được quản lý trong AWS console.

<a name="worker-timeouts"></a>
#### Worker Timeouts

Lệnh Artisan `queue:work` có một tùy chọn là `--timeout`. Mặc định, giá trị `--timeout` này là 60 giây. Nếu một job đang xử lý lâu hơn số giây được chỉ định bởi giá trị timeout, thì worker đang xử lý job đó sẽ bị thoát ra cùng với lỗi. Thông thường, worker sẽ được khởi động lại tự động bởi [trình quản lý process được cấu hình trên máy chủ của bạn](#supervisor-configuration):

```shell
php artisan queue:work --timeout=60
```

Tùy chọn cấu hình `retry_after` và tùy chọn CLI `--timeout` tuy khác nhau, nhưng nếu phối hợp với nhau thì sẽ giúp bạn đảm bảo rằng các job sẽ không bị mất và các job chỉ được xử lý thành công trong một lần.

> [!WARNING]
> Giá trị `--timeout` phải luôn luôn có thời gian ngắn hoặc ít hơn vài giây so với giá trị cấu hình `retry_after`. Điều này sẽ đảm bảo là một worker đang xử lý một job bị đơ luôn được kết thúc trước khi job đó được chạy lại. Nếu tùy chọn `--timeout` của bạn dài hơn giá trị cấu hình `retry_after`, thì job đó của bạn có thể bị xử lý hai lần.

<a name="supervisor-configuration"></a>
## Cấu hình Supervisor

Trong production, bạn cần một cách để duy trì hoạt động của các process `queue:work`. Process `queue:work` có thể ngừng chạy vì nhiều lý do, chẳng hạn như vượt quá thời gian chờ của worker hoặc việc thực thi lệnh `queue:restart`.

Vì lý do này, bạn cần cấu hình một trình giám sát process có thể phát hiện ra khi các process `queue:work` của bạn bị thoát và tự động khởi động lại chúng. Ngoài ra, trình giám sát process có thể cho phép bạn chỉ định số lượng process `queue:work` mà bạn muốn chạy đồng thời. Supervisor là trình giám sát process thường được sử dụng trong môi trường Linux và chúng ta sẽ thảo luận về cách cấu hình nó trong tài liệu sau.

<a name="installing-supervisor"></a>
#### Installing Supervisor

Supervisor là trình giám sát process cho hệ điều hành Linux và sẽ tự động khởi động lại process `queue:work` của bạn nếu chúng thất bại. Để cài đặt Supervisor trên Ubuntu, bạn có thể sử dụng lệnh sau:

```shell
sudo apt-get install supervisor
```

> [!NOTE]
> Nếu bạn không muốn cấu hình và quản lý Supervisor, hãy xem xét việc sử dụng [Laravel Forge](https://forge.laravel.com), nó sẽ tự động cài đặt và cấu hình Supervisor cho các dự án production Laravel của bạn.

<a name="configuring-supervisor"></a>
#### Configuring Supervisor

Các file cấu hình của Supervisor thường được lưu trong thư mục `/etc/supervisor/conf.d`. Trong thư mục này, bạn có thể tạo nhiều file cấu hình để hướng dẫn supervisor cách mà các process của bạn sẽ được giám sát. Ví dụ: hãy tạo một file `laravel-worker.conf` để bắt đầu và giám sát các process `queue:work`:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
stopwaitsecs=3600
```

Trong ví dụ trên, lệnh `numprocs` sẽ hướng dẫn Supervisor chạy tám process `queue:work` và giám sát tất cả chúng, tự động khởi động lại nếu chúng thất bại. Bạn nên thay đổi lệnh `command` của cấu hình để phản ánh queue connection và tuỳ chọn worker mà bạn mong muốn.

> [!WARNING]
> Bạn nên chắc chắn rằng giá trị của `stopwaitsecs` sẽ luôn lớn hơn số giây lâu nhất mà job của bạn đang chạy. Nếu không, Supervisor có thể kết thúc job đó trước khi nó được xử lý xong.

<a name="starting-supervisor"></a>
#### Starting Supervisor

Khi file cấu hình đã hoàn thành, bạn có thể cập nhật cấu hình Supervisor và chạy các process bằng các lệnh sau:

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start "laravel-worker:*"
```

Để biết thêm thông tin về Supervisor, hãy tham khảo [tài liệu Supervisor](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>
## Xử lý Job failed

Thỉnh thoảng, queued job của bạn sẽ gặp thất bại. Đừng lo lắng, mọi thứ không phải lúc nào cũng theo như kế hoạch! Laravel có chứa một cách để [khai báo số lần tối đa mà một job được chạy lại](#max-job-attempts-and-timeout). Sau khi một job vượt quá số lần chạy này, nó sẽ được thêm vào bảng cơ sở dữ liệu `failed_jobs`. [Các job được chạy đồng bộ với request](/docs/{{version}}/queues#synchronous-dispatching) mà bị thất bại, thì sẽ không được lưu vào trong bảng này và các trường hợp ngoại lệ của chúng sẽ được ứng dụng xử lý ngay lập tức.

Một migration để tạo bảng `failed_jobs` sẽ có sẵn trong các ứng dụng Laravel mới. Tuy nhiên, nếu ứng dụng của bạn không chứa file migration cho bảng này, bạn có thể sử dụng lệnh `queue:failed-table` để tạo file migration đó:

```shell
php artisan queue:failed-table

php artisan migrate
```

Khi chạy một process [queue worker](#running-the-queue-worker), bạn có thể khai báo số lần chạy tối đa mà một job được chạy bằng cách sử dụng switch `--tries` trên lệnh `queue:work`. Nếu bạn không khai báo giá trị tùy chọn `--tries`, thì các job sẽ chỉ được chạy một lần duy nhất hoặc nhiều lần theo quy định của thuộc tính `$tries` trong class job:

```shell
php artisan queue:work redis --tries=3
```

Dùng tuỳ chọn `--backoff`, bạn có thể chỉ định cho Laravel biết sẽ đợi bao nhiêu giây trước khi thử lại một job bị gặp ngoại lệ. Mặc định, một job ngay lập tức được đưa trở lại queue để có thể thử lại:

```shell
php artisan queue:work redis --tries=3 --backoff=3
```

Nếu bạn muốn cấu hình Laravel sẽ đợi bao nhiêu giây trước khi thử lại job khi bị gặp ngoại lệ, bạn có thể làm như vậy bằng cách định nghĩa một thuộc tính `backoff` trong class job của bạn:

    /**
     * The number of seconds to wait before retrying the job.
     *
     * @var int
     */
    public $backoff = 3;

Nếu bạn yêu cầu các logic phức tạp hơn để xác định thời gian thử lại của job, bạn có thể định nghĩa một phương thức `backoff` trên class job của bạn:

    /**
    * Calculate the number of seconds to wait before retrying the job.
    */
    public function backoff(): int
    {
        return 3;
    }

Bạn có thể dễ dàng cấu hình thời gian thử lại "theo cấp số nhân" bằng cách trả về một mảng các giá trị thời gian thử lại từ phương thức `backoff`. Trong ví dụ này, độ trễ thử lại sẽ là 1 giây cho lần thử đầu tiên, và 5 giây cho lần thử lại thứ hai, 10 giây cho lần thử lại thứ ba, và 10 giây cho mỗi lần thử lại tiếp theo nếu còn nhiều lần thử tiếp theo hơn:

    /**
    * Calculate the number of seconds to wait before retrying the job.
    *
    * @return array<int, int>
    */
    public function backoff(): array
    {
        return [1, 5, 10];
    }

<a name="cleaning-up-after-failed-jobs"></a>
### Dọn dẹp sau khi Job failed

Khi một job bị thất bại, bạn có thể muốn gửi thông báo cho người dùng của bạn hoặc revert lại mọi hành động đã được job đó hoàn thành. Để thực hiện điều này, bạn có thể định nghĩa một phương thức `failed` trên class job của bạn. Instance `Throwable` khiến cho job thất bại và sẽ được chuyển sang phương thức `failed`:

    <?php

    namespace App\Jobs;

    use App\Models\Podcast;
    use App\Services\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;
    use Throwable;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Create a new job instance.
         */
        public function __construct(
            public Podcast $podcast,
        ) {}

        /**
         * Execute the job.
         */
        public function handle(AudioProcessor $processor): void
        {
            // Process uploaded podcast...
        }

        /**
         * Handle a job failure.
         */
        public function failed(?Throwable $exception): void
        {
            // Send user notification of failure, etc...
        }
    }

> [!WARNING]
> Một instance mới của job sẽ được khởi tạo trước khi gọi phương thức `failed`; do đó, mọi thay đổi thuộc tính class có trong phương thức `handle` sẽ bị mất.

<a name="retrying-failed-jobs"></a>
### Retrying Failed Jobs

Để xem tất cả các job thất bại đã được thêm vào trong bảng cơ sở dữ liệu `failed_jobs` của bạn, bạn có thể sử dụng lệnh Artisan `queue:failed`:

```shell
php artisan queue:failed
```

Lệnh `queue:failed` sẽ liệt kê các ID job, kết nối, queue, thời gian lỗi và các thông tin khác về job. ID job có thể được sử dụng để thử lại job đó. Ví dụ: để thử lại một job thất bại có ID là `ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece`, hãy chạy lệnh sau:

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece
```

Nếu cần, bạn có thể truyền nhiều ID cho lệnh:

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece 91401d2c-0784-4f43-824c-34f94a33c24d
```

Bạn cũng có thể thử lại tất cả các job thất bại cho một queue cụ thể:

```shell
php artisan queue:retry --queue=name
```

Để thử lại tất cả các job thất bại của bạn, hãy chạy lệnh `queue:retry` và truyền `all` làm ID:

```shell
php artisan queue:retry all
```

Nếu bạn muốn xóa một job thất bại, bạn có thể sử dụng lệnh `queue:forget`:

```shell
php artisan queue:forget 91401d2c-0784-4f43-824c-34f94a33c24d
```

> [!NOTE]
> Khi sử dụng [Horizon](/docs/{{version}}/horizon), bạn nên sử dụng lệnh `horizon:forget` để xóa các job không thành công thay vì lệnh `queue:forget`.

Để xóa tất cả các job thất bại ra khỏi bảng `failed_jobs`, bạn có thể sử dụng lệnh `queue:flush`:

```shell
php artisan queue:flush
```

<a name="ignoring-missing-models"></a>
### Ignoring Missing Models

Khi tích hợp một model Eloquent vào một job, model đó sẽ tự động được serialize trước khi được đưa vào queue và được lấy lại từ cơ sở dữ liệu khi job được xử lý. Tuy nhiên, nếu model đã bị xóa trong khi job đang chờ worker xử lý thì job của bạn có thể thất bại với lỗi `ModelNotFoundException`.

Để thuận tiện, bạn có thể chọn tự động xóa các job mà có model bị thiếu bằng cách set thuộc tính `deleteWhenMissingModels` trong job của bạn thành `true`. Khi thuộc tính này được set thành `true`, Laravel sẽ lặng lẽ xoá job mà không đưa ra ngoại lệ:

    /**
     * Delete the job if its models no longer exist.
     *
     * @var bool
     */
    public $deleteWhenMissingModels = true;

<a name="pruning-failed-jobs"></a>
### Xoá job failed

Bạn có thể xóa hết các record có trong bảng `failed_jobs` của ứng dụng bằng cách gọi lệnh Artisan `queue:prune-failed`:

```shell
php artisan queue:prune-failed
```

Mặc định, tất cả các bản ghi job bị thất bại có thời gian tồn tại lớn hơn 24 giờ sẽ bị xoá bỏ. Nếu bạn cung cấp tùy chọn `--hours` cho lệnh, thì chỉ những record job nào mà được thêm vào trong số N giờ mới được giữ lại. Ví dụ: lệnh sau sẽ xóa tất cả các record job bị lỗi đã được thêm vào trước 48 giờ trước:

```shell
php artisan queue:prune-failed --hours=48
```

<a name="storing-failed-jobs-in-dynamodb"></a>
### Lưu job failed vào trong DynamoDB

Laravel cũng cung cấp hỗ trợ lưu trữ các record của job thất bại của bạn vào trong [DynamoDB](https://aws.amazon.com/dynamodb) thay vì phải lưu vào bảng cơ sở dữ liệu quan hệ. Tuy nhiên, bạn phải tự tạo bảng DynamoDB để lưu trữ tất cả record của job thất bại. Thông thường, bảng này phải được đặt tên là `failed_jobs`, nhưng bạn nên đặt tên bảng dựa trên giá trị của giá trị cấu hình `queue.failed.table` trong file cấu hình `queue` trong ứng dụng của bạn.

Bảng `failed_jobs` phải có khóa chính phân vùng tên là `application` và khóa chính sắp xếp có tên là `uuid`. Phần `application` của khóa sẽ chứa tên ứng dụng của bạn được định nghĩa bởi giá trị cấu hình `name` trong file cấu hình `app` của ứng dụng của bạn. Vì tên ứng dụng là một phần khóa của bảng DynamoDB nên bạn có thể sử dụng cùng một bảng để lưu các job thất bại cho nhiều ứng dụng Laravel.

Ngoài ra, hãy đảm bảo là bạn đã cài đặt AWS SDK để ứng dụng Laravel của bạn có thể giao tiếp với Amazon DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Tiếp theo, hãy set giá trị của tùy chọn cấu hình `queue.failed.driver` thành `dynamodb`. Ngoài ra, bạn nên định nghĩa các tùy chọn cấu hình `key`, `secret` và `region` trong mảng cấu hình job thất bại. Các tùy chọn này sẽ được sử dụng để xác thực với AWS. Khi sử dụng driver `dynamodb`, tùy chọn cấu hình `queue.failed.database` sẽ không dùng đến:

```php
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'failed_jobs',
],
```

<a name="disabling-failed-job-storage"></a>
### Disable lưu job failed

Bạn có thể hướng dẫn Laravel loại bỏ các job thất bại mà không lưu chúng vào cơ sỏ dữ liệu bằng cách set giá trị của tùy chọn cấu hình `queue.failed.driver` thành `null`. Thông thường, điều này có thể được thực hiện thông qua biến môi trường `QUEUE_FAILED_DRIVER`:

```ini
QUEUE_FAILED_DRIVER=null
```

<a name="failed-job-events"></a>
### Event Job failed

Nếu bạn muốn đăng ký một listener event sẽ được gọi khi một job thất bại, bạn có thể sử dụng phương thức `failing` của facade `Queue`. Ví dụ: chúng ta có thể đính kèm một closure cho event này từ phương thức `boot` của `AppServiceProvider` được chứa trong Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobFailed;

    class AppServiceProvider extends ServiceProvider
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
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }
    }

<a name="clearing-jobs-from-queues"></a>
## Xoá job từ queue

> [!NOTE]
> Khi sử dụng [Horizon](/docs/{{version}}/horizon), bạn nên sử dụng lệnh `horizon:clear` để xóa các job ra khỏi queue thay vì lệnh `queue:clear`.

Nếu bạn muốn xóa tất cả job ra khỏi queue mặc định của kết nối mặc định, bạn có thể làm như vậy bằng cách sử dụng lệnh Artisan `queue:clear`:

```shell
php artisan queue:clear
```

Bạn cũng có thể cung cấp tham số `connection` và tùy chọn `queue` để xóa các job trên một kết nối và queue cụ thể:

```shell
php artisan queue:clear redis --queue=emails
```

> [!WARNING]
> Việc xóa job ra khỏi queue chỉ có cho các driver queue cơ sở dữ liệu, Redis và SQS. Ngoài ra, quá trình xóa message SQS có thể mất tới 60 giây, do đó, các job được gửi đến queue SQS sau 60 giây sau khi bạn xóa queue cũng có thể bị xóa.

<a name="monitoring-your-queues"></a>
## Giám sát queue

Nếu queue của bạn nhận được một lượng lớn job đột ngột, nó có thể bị quá tải, dẫn đến thời gian chờ job hoàn thành có thể bị kéo dài. Nếu bạn muốn, Laravel có thể cảnh báo bạn khi có một số lượng job queue của bạn vượt quá ngưỡng được chỉ định.

Để bắt đầu, bạn nên tạo schedule lệnh `queue:monitor` để [chạy mỗi phút](/docs/{{version}}/scheduling). Lệnh chấp nhận tên của queue mà bạn muốn theo dõi cũng như ngưỡng số lượng job mà bạn mong muốn:

```shell
php artisan queue:monitor redis:default,redis:deployments --max=100
```

Schedule cho lệnh này là không đủ để kích hoạt cảnh báo cho bạn về tình trạng quá tải của queue. Khi lệnh gặp một queue có số lượng công việc vượt quá ngưỡng của bạn, một event `Illuminate\Queue\Events\QueueBusy` sẽ được gửi đi. Bạn có thể lắng nghe event này trong `EventServiceProvider` của ứng dụng để gửi thông báo cho bạn hoặc nhóm phát triển của bạn:

```php
use App\Notifications\QueueHasLongWaitTime;
use Illuminate\Queue\Events\QueueBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Register any other events for your application.
 */
public function boot(): void
{
    Event::listen(function (QueueBusy $event) {
        Notification::route('mail', 'dev@example.com')
                ->notify(new QueueHasLongWaitTime(
                    $event->connection,
                    $event->queue,
                    $event->size
                ));
    });
}
```

<a name="testing"></a>
## Testing

Khi kiểm tra code gửi job, bạn có thể muốn hướng dẫn Laravel là không cần thiết phải chạy job đó, vì code của job có thể được kiểm tra trực tiếp và riêng biệt với code gửi job đó. Tất nhiên, để kiểm tra job, bạn có thể khởi tạo một instance job và gọi phương thức `handle` trực tiếp trong bài kiểm tra của bạn.

Bạn có thể sử dụng phương thức `fake` của facade `Queue` để chặn các queued job thực sự được đưa vào queue. Sau khi gọi phương thức `fake` của facade `Queue`, sau đó bạn có thể kiểm tra ứng dụng đã đưa các job vào queue chưa:

    <?php

    namespace Tests\Feature;

    use App\Jobs\AnotherJob;
    use App\Jobs\FinalJob;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Queue;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped(): void
        {
            Queue::fake();

            // Perform order shipping...

            // Assert that no jobs were pushed...
            Queue::assertNothingPushed();

            // Assert a job was pushed to a given queue...
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // Assert a job was pushed twice...
            Queue::assertPushed(ShipOrder::class, 2);

            // Assert a job was not pushed...
            Queue::assertNotPushed(AnotherJob::class);

            // Assert that a Closure was pushed to the queue...
            Queue::assertClosurePushed();

            // Assert the total number of jobs that were pushed...
            Queue::assertCount(3);
        }
    }

Bạn có thể truyền một closure cho các phương thức `assertPushed` hoặc `assertNotPushed` để kiểm tra một job đã được đẩy vào queue và pass qua được "truth test" đã cho. Nếu có ít nhất một job đã được đẩy vào và pass qua truth test đã cho thì kiểm tra sẽ thành công:

    Queue::assertPushed(function (ShipOrder $job) use ($order) {
        return $job->order->id === $order->id;
    });

<a name="faking-a-subset-of-jobs"></a>
### Fake một tập hợp Jobs

Nếu bạn chỉ cần fake các job cụ thể trong khi cho phép các job khác được chạy bình thường, bạn có thể truyền tên class của các job cần fake vào phương thức `fake`:

    public function test_orders_can_be_shipped(): void
    {
        Queue::fake([
            ShipOrder::class,
        ]);

        // Perform order shipping...

        // Assert a job was pushed twice...
        Queue::assertPushed(ShipOrder::class, 2);
    }

Bạn có thể fake tất cả các job ngoại trừ một tập hợp các job được chỉ định bằng phương thức `except`:

    Queue::fake()->except([
        ShipOrder::class,
    ]);

<a name="testing-job-chains"></a>
### Testing Job Chains

Để kiểm tra một chuỗi job, bạn sẽ cần sử dụng khả năng fake của facade `Bus`. Phương thức `assertChained` của facade `Bus` có thể được sử dụng để kiểm tra một [chuỗi job](/docs/{{version}}/queues#job-chaining) đã được gửi hay chưa. Phương thức `assertChained` sẽ chấp nhận một mảng các job trong một chuỗi làm tham số đầu tiên của nó:

    use App\Jobs\RecordShipment;
    use App\Jobs\ShipOrder;
    use App\Jobs\UpdateInventory;
    use Illuminate\Support\Facades\Bus;

    Bus::fake();

    // ...

    Bus::assertChained([
        ShipOrder::class,
        RecordShipment::class,
        UpdateInventory::class
    ]);

Như bạn có thể thấy trong ví dụ trên, mảng job trong một chuỗi có thể là một mảng gồm các tên class của job. Tuy nhiên, bạn cũng có thể cung cấp một mảng các instance job thực tế. Khi làm như vậy, Laravel sẽ đảm bảo là các instance job đó sẽ thuộc cùng một class và cùng giá trị thuộc tính khi được gửi đi bởi ứng dụng của bạn:

    Bus::assertChained([
        new ShipOrder,
        new RecordShipment,
        new UpdateInventory,
    ]);

Bạn có thể sử dụng phương thức `assertDispatchedWithoutChain` để kiểm tra một job đã được đẩy đi mà không nằm trong bất kỳ chuỗi job nào:

    Bus::assertDispatchedWithoutChain(ShipOrder::class);

<a name="testing-chained-batches"></a>
#### Testing Chained Batches

Nếu chuỗi job của bạn [có chứa một batch job](#chains-and-batches), bạn có thể kiểm tra batch job được nối đó phù hợp với kỳ vọng của bạn bằng cách chèn thêm một định nghĩa `Bus::chainedBatch` vào kiểm tra chuỗi job của bạn:

    use App\Jobs\ShipOrder;
    use App\Jobs\UpdateInventory;
    use Illuminate\Bus\PendingBatch;
    use Illuminate\Support\Facades\Bus;

    Bus::assertChained([
        new ShipOrder,
        Bus::chainedBatch(function (PendingBatch $batch) {
            return $batch->jobs->count() === 3;
        }),
        new UpdateInventory,
    ]);

<a name="testing-job-batches"></a>
### Testing Job Batches

Phương thức `assertBatched` của facade `Bus` có thể được sử dụng để kiểm tra một [batch job](/docs/{{version}}/queues#job-batching) đã được gửi hay chưa. Closure được cung cấp cho phương thức `assertBatched` sẽ nhận vào một instance của `Illuminate\Bus\PendingBatch`, có thể được sử dụng để kiểm tra các job có trong batch:

    use Illuminate\Bus\PendingBatch;
    use Illuminate\Support\Facades\Bus;

    Bus::fake();

    // ...

    Bus::assertBatched(function (PendingBatch $batch) {
        return $batch->name == 'import-csv' &&
               $batch->jobs->count() === 10;
    });

Bạn có thể sử dụng phương thức `assertBatchCount` để kiểm tra số lượng batch đã được gửi đi:

    Bus::assertBatchCount(3);

Bạn có thể sử dụng `assertNothingBatched` để kiểm tra không có batch nào được gửi đi:

    Bus::assertNothingBatched();

<a name="testing-job-batch-interaction"></a>
#### Testing Job / Batch Interaction

Ngoài ra, đôi khi bạn có thể cần kiểm tra tương tác của một job với batch của nó. Ví dụ, bạn có thể cần kiểm tra xem một job có hủy xử lý tiếp theo của batch của nó hay không. Để thực hiện việc này, bạn cần chỉ định một batch fake cho job đó thông qua phương thức `withFakeBatch`. Phương thức `withFakeBatch` này sẽ trả về một mảng chứa instance job và batch fake:

    [$job, $batch] = (new ShipOrder)->withFakeBatch();

    $job->handle();

    $this->assertTrue($batch->cancelled());
    $this->assertEmpty($batch->added);

<a name="job-events"></a>
## Job Event

Sử dụng các phương thức `before` và `after` trong [facade](/docs/{{version}}/facades) `Queue`, bạn có thể khai báo các callback sẽ được thực hiện trước hoặc sau khi một queued job được xử lý. Các callback này là một cách tuyệt vời để thực hiện thêm logging hoặc ghi thông kê cho bảng điều khiển. Thông thường, bạn nên gọi các phương thức này từ phương thức `boot` của [service provider](/docs/{{version}}/providers). Ví dụ: chúng ta có thể sử dụng `AppServiceProvider` được đi kèm với Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
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
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }
    }

Sử dụng phương thức `looping` trong [facade](/docs/{{version}}/facades) `Queue`, bạn có thể khai báo các callback sẽ được thực thi trước khi worker lấy một job từ queue. Ví dụ: bạn có thể đăng ký một closure để rollback bất kỳ các transaction nào đang bị làm dở bởi một job đã bị thất bại trước đó:

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Queue;

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
