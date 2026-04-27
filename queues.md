# Queues

- [Giới thiệu](#introduction)
    - [Connection và Queue](#connections-vs-queues)
    - [Driver chú ý và điều kiện](#driver-prerequisites)
- [Tạo Job](#creating-jobs)
    - [Tạo class Job](#generating-job-classes)
    - [Cấu trúc class](#class-structure)
    - [Unique Jobs](#unique-jobs)
    - [Encrypted Jobs](#encrypted-jobs)
    - [Debounced Jobs](#debounced-jobs)
- [Job Middleware](#job-middleware)
    - [Giới hạn tỷ lệ](#rate-limiting)
    - [Chặn Job chồng nhau](#preventing-job-overlaps)
    - [Ngoại lệ](#throttling-exceptions)
    - [Bỏ qua Job](#skipping-jobs)
- [Gửi Job](#dispatching-jobs)
    - [Delay gửi](#delayed-dispatching)
    - [Đồng bộ gửi](#synchronous-dispatching)
    - [Jobs và Database Transactions](#jobs-and-database-transactions)
    - [Kết hợp Job](#job-chaining)
    - [Tuỳ biến Queue và Connection](#customizing-the-queue-and-connection)
    - [Khai báo số lần chạy Job tối đa / giá trị timeout](#max-job-attempts-and-timeout)
    - [SQS FIFO và Fair Queues](#sqs-fifo-and-fair-queues)
    - [Queue Failover](#queue-failover)
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
    - [Dừng và tiếp tục Queue Workers](#pausing-and-resuming-queue-workers)
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
    - [Testing Job / Queue Interactions](#testing-job-queue-interactions)
- [Job Event](#job-events)

<a name="introduction"></a>
## Giới thiệu

Trong khi xây dựng ứng dụng web, bạn có thể có một số task, chẳng hạn như chuyển đổi và lưu trữ file CSV đã tải lên, và quá trình này mất nhiều thời gian để thực hiện trong một request web thông thường. Rất may, Laravel cho phép bạn dễ dàng tạo ra các queued job có thể được dùng để xử lý các công việc ở trên trong background. Bằng cách chuyển các task tốn nhiều thời gian sang queue, ứng dụng của bạn có thể đáp ứng các request web với tốc độ nhanh và cung cấp trải nghiệm người dùng tốt hơn cho khách hàng của bạn.

Queue của Laravel cung cấp một queueing API hợp nhất trên nhiều loại queue backend khác nhau, chẳng hạn như [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), hoặc thậm chí là một database.

Các tùy chọn cấu hình queue của Laravel được lưu trong file cấu hình `config/queue.php` trong ứng dụng của bạn. Trong file này, bạn sẽ tìm thấy các cấu hình connection cho từng loại driver queue có trong framework, gồm có database, [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), và [Beanstalkd](https://beanstalkd.github.io/), cũng như một driver chạy đồng bộ job (để sử dụng trong quá trình phát triển hoặc testing). Driver queue `null` cũng đã được khai báo để loại bỏ các job đã được queue.

> [!NOTE]
> Laravel Horizon là một hệ thống cấu hình và điều khiển cho các queue mà được tạo bởi Redis của bạn. Hãy xem toàn bộ [tài liệu Horizon](/docs/{{version}}/horizon) để biết thêm thông tin chi tiết.

<a name="connections-vs-queues"></a>
### Connection và Queue

Trước khi bắt đầu với Laravel queue, điều quan trọng là phải hiểu sự khác biệt giữa "connections" và "queues". Trong file cấu hình `config/queue.php` của bạn, có một mảng cấu hình là `connections`. Tùy chọn này sẽ định nghĩa các connection đến các queue backend service như Amazon SQS, Beanstalk hoặc Redis. Tuy nhiên, bất kỳ connection nào cũng có thể có nhiều "queue", và nó có thể được coi là một ngăn xếp hoặc một mảng các job khác nhau.

Lưu ý rằng mỗi ví dụ cấu hình connection trong file cấu hình `queue` có chứa một thuộc tính `queue`. Đây là queue mặc định mà các job sẽ được gửi tới mỗi khi chúng được gửi đến một connection. Nói cách khác, nếu bạn gửi một job mà không khai báo rõ queue nào sẽ được dùng, thì job đó sẽ được lưu vào queue mà đã được định nghĩa trong thuộc tính `queue` của cấu hình connection:

```php
use App\Jobs\ProcessPodcast;

// This job is sent to the default connection's default queue...
ProcessPodcast::dispatch();

// This job is sent to the default connection's "emails" queue...
ProcessPodcast::dispatch()->onQueue('emails');
```

Một số application có thể không cần phải tạo nhiều job trong nhiều queue, thay vào đó một queue có thể là phù hợp hơn. Tuy nhiên, việc tạo các job lên nhiều queue cũng có thể đặc biệt hữu ích cho các application mà muốn ưu tiên hoặc là phân chia cách xử lý cho từng job, vì Laravel queue worker cho phép bạn khai báo các queue sẽ được xử lý theo mức độ ưu tiên. Ví dụ: nếu bạn tạo một job lên queue `high`, thì bạn có thể chạy một worker có mức độ ưu tiên xử lý cao hơn:

```shell
php artisan queue:work --queue=high,default
```

<a name="driver-prerequisites"></a>
### Driver chú ý và điều kiện

<a name="database"></a>
#### Database

Để sử dụng driver `database` queue, bạn sẽ cần một bảng cơ sở dữ liệu để lưu các job. Thông thường, bảng này đã có sẵn trong file migration mặc định `0001_01_01_000002_create_jobs_table.php` của Laravel [database migration](/docs/{{version}}/migrations); tuy nhiên, nếu ứng dụng của bạn không chứa file migration này, bạn có thể sử dụng lệnh Artisan `make:queue-table` để tạo ra nó:

```shell
php artisan make:queue-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Để sử dụng driver `redis` queue, bạn nên cấu hình connection tới Redis database trong file cấu hình `config/database.php` của bạn.

> [!WARNING]
> Các tùy chọn `serializer` và `compression` của Redis sẽ không được driver `redis` queue hỗ trợ.

<a name="redis-cluster"></a>
##### Redis Cluster

Nếu connection Redis của bạn sử dụng một [Redis Cluster](https://redis.io/docs/latest/operate/rs/databases/durability-ha/clustering), thì tên queue của bạn phải chứa một [key hash tag](https://redis.io/docs/latest/develop/using-commands/keyspace/#hashtags). Điều này là bắt buộc để đảm bảo rằng tất cả các key Redis cho queue sẽ được set vào cùng một vị trí hash:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', '{default}'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => null,
    'after_commit' => false,
],
```

<a name="blocking"></a>
##### Blocking

Khi sử dụng queue Redis, bạn có thể sử dụng tùy chọn cấu hình `block_for` để chỉ định khoảng thời gian mà driver sẽ đợi cho job được bắt đầu trước khi nó lặp lại vòng lặp worker và thăm dò lại cơ sở dữ liệu Redis.

Điều chỉnh giá trị này dựa trên queue load của bạn, nó có thể hiệu quả hơn việc liên tục thăm dò cơ sở dữ liệu Redis để tìm ra các job mới. Ví dụ: bạn có thể set giá trị là `5` để chỉ ra rằng driver sẽ chặn năm giây trong khi chờ job sẵn sàng:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', 'default'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => 5,
    'after_commit' => false,
],
```

> [!WARNING]
> Việc set `block_for` thành `0` sẽ khiến các queue worker chặn vô thời hạn cho đến khi có job. Điều này cũng sẽ chặn các tín hiệu như `SIGTERM` được xử lý cho đến khi job tiếp theo được xử lý.

<a name="other-driver-prerequisites"></a>
#### Other Driver Prerequisites

Các library sau sẽ cần thiết cho driver queue cũng sẽ được liệt kê. Những library này có thể được cài đặt thông qua Composer package manager:

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~5.0`
- Redis: `predis/predis ~2.0` or phpredis PHP extension
- [MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/): `mongodb/laravel-mongodb`

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

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

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
```

Trong ví dụ trên, hãy lưu ý rằng chúng ta có thể truyền một [Eloquent model](/docs/{{version}}/eloquent) trực tiếp vào hàm khởi tạo của queued job. Do trait `Queueable` này đang được job sử dụng, nên các model Eloquent và các quan hệ của nó cũng sẽ được serialize và unserialize ngược lại khi job được xử lý.

Nếu queued job của bạn chấp nhận một model Eloquent trong hàm khởi tạo, thì chỉ có mã định danh cho model sẽ được serialize trên queue. Và khi job đó được xử lý, thì hệ thống queue sẽ tự động lấy ra lại full instance của model đó và các quan hệ của nó cũng từ cơ sở dữ liệu. Cách chuyển đổi model này cho phép gửi job nhỏ hơn nhiều đến queue driver của bạn.

<a name="handle-method-dependency-injection"></a>
#### `handle` Method Dependency Injection

Phương thức `handle` được gọi khi job được xử lý bởi queue. Lưu ý rằng chúng ta có thể khai báo các phụ thuộc vào phương thức `handle` của job. Laravel [service container](/docs/{{version}}/container) sẽ tự động inject các phụ thuộc này.

Nếu bạn muốn toàn quyền kiểm soát cách container đưa các phụ thuộc vào phương thức `handle`, bạn có thể sử dụng phương thức `bindMethod` của container. Phương thức `bindMethod` chấp nhận một callback nhận vào một job và container. Trong lệnh callback, bạn có thể thoải mái gọi phương thức `handle` theo bất kỳ cách nào mà bạn muốn. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của [service provider](/docs/{{version}}/providers) `App\Providers\AppServiceProvider`:

```php
use App\Jobs\ProcessPodcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Foundation\Application;

$this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

> [!WARNING]
> Dữ liệu nhị phân, chẳng hạn như một nội dung ảnh thô, phải được truyền qua hàm `base64_encode` trước khi được truyền đến một queued job. Nếu không làm điều này, thì job đó có thể serialize thành chuỗi JSON không đúng khi được đặt lên queue.

<a name="handling-relationships"></a>
#### Queued Relationships

Bởi vì tất cả các quan hệ của Eloquent model cũng sẽ được serialize trong khi một job được đưa vào queue, nên thỉnh thoảng job có thể trở nên khá lớn. Hơn thế nữa, khi một job được deserialize lại thì các quan hệ model sẽ được lấy lại toàn bộ từ cơ sở dữ liệu. Bất kỳ các ràng buộc quan hệ nào đã được thay đổi trước khi model được serialize trong quá trình queue job thì sẽ không lưu lại những thay đổi đó khi job được deserialize lại. Do đó, nếu bạn muốn làm việc với một tập hợp của một quan hệ nhất định, bạn nên ràng buộc lại quan hệ đó trong queued job của bạn.

Hoặc, để ngăn việc các quan hệ bị serialize, bạn có thể gọi phương thức `withoutRelations` trên model khi set giá trị thuộc tính. Phương thức này sẽ trả về một instance của model mà không có quan hệ được load:

```php
/**
 * Create a new job instance.
 */
public function __construct(
    Podcast $podcast,
) {
    $this->podcast = $podcast->withoutRelations();
}
```

Nếu bạn chỉ cần xóa một quan hệ cụ thể trong khi vẫn giữ những quan hệ khác, bạn có thể sử dụng phương thức `withoutRelation`:

```php
$this->podcast = $podcast->withoutRelation('comments');
```

Nếu bạn đang sử dụng [chức năng thuộc tính của hàm constructor property promotion PHP](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion) và muốn rằng model Eloquent sẽ không serialize các quan hệ của nó, bạn có thể sử dụng thuộc tính `WithoutRelations`:

```php
use Illuminate\Queue\Attributes\WithoutRelations;

/**
 * Create a new job instance.
 */
public function __construct(
    #[WithoutRelations]
    public Podcast $podcast,
) {}
```

Để thuận tiện, nếu bạn muốn serialize tất cả các model mà bỏ qua quan hệ, bạn có thể sử dụng attribute `WithoutRelations` cho toàn bộ class thay vì sử dụng attribute đó cho từng model:

```php
<?php

namespace App\Jobs;

use App\Models\DistributionPlatform;
use App\Models\Podcast;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\WithoutRelations;

#[WithoutRelations]
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
        public DistributionPlatform $platform,
    ) {}
}
```

Nếu một job nhận vào một collection hoặc một mảng các model Eloquent thay vì một model duy nhất, các model trong collection đó sẽ không thể khôi phục được quan hệ của chúng khi job được deserialize và được thực thi. Điều này nhằm ngăn chặn việc sử dụng tài nguyên quá mức trên các job xử lý số lượng lớn các model.

<a name="unique-jobs"></a>
### Unique Jobs

> [!WARNING]
> Unique Job sẽ yêu cầu một cache driver hỗ trợ [locks](/docs/{{version}}/cache#atomic-locks). Hiện tại, cache driver `memcached`, `redis`, `dynamodb`, `database`, `file` và `array` đều hỗ trợ atomic lock.

> [!WARNING]
> Các ràng buộc unique job không áp dụng cho các job có trong batch.

Thỉnh thoảng, bạn có thể muốn đảm bảo rằng chỉ có một instance của một job cụ thể có trong queue tại bất kỳ thời điểm nào. Bạn có thể làm như vậy bằng cách implement interface `ShouldBeUnique` trên class job của bạn. Interface này không yêu cầu bạn định nghĩa thêm bất kỳ phương thức nào trên class của bạn:

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...
}
```

Trong ví dụ trên, job `UpdateSearchIndex` là unique. Vì vậy, job sẽ không được gửi đi nếu một instance khác của job đã có trong queue và chưa được xử lý xong.

Trong một số trường hợp nhất định, bạn có thể muốn định nghĩa một "key" cụ thể để làm cho job trở nên unique hoặc bạn có thể muốn chỉ định một khoảng thời gian chờ mà vượt qua khoảng thời gian đó job sẽ không còn unique nữa. Để thực hiện điều này, bạn có thể dùng thuộc tính `UniqueFor` và định nghĩa một phương thức `uniqueId` trên class job của bạn:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Queue\Attributes\UniqueFor;

#[UniqueFor(3600)]
class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    /**
     * The product instance.
     *
     * @var \App\Models\Product
     */
    public $product;

    /**
     * Get the unique ID for the job.
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }
}
```
Trong ví dụ trên, job `UpdateSearchIndex` là unique theo ID product. Vì vậy, mọi job mới được gửi mà có cùng ID product sẽ bị bỏ qua cho đến khi job hiện tại hoàn tất xử lý. Ngoài ra, nếu job hiện tại không được xử lý trong vòng một giờ, khóa unique sẽ được giải phóng và một job khác có cùng khóa unique có thể được gửi đến queue.

> [!WARNING]
> Nếu ứng dụng của bạn phân phối các job từ nhiều máy chủ web hoặc container, bạn nên đảm bảo là tất cả các máy chủ của bạn đang giao tiếp với cùng một máy chủ cache trung tâm để Laravel có thể xác định chính xác xem job có duy nhất hay không.

<a name="keeping-jobs-unique-until-processing-begins"></a>
#### Keeping Jobs Unique Until Processing Begins

Mặc định, các job unique sẽ được "mở khóa" sau khi một job hoàn tất quá trình xử lý hoặc bị thất bại trong tất cả các lần thử lại của nó. Tuy nhiên, có thể có những trường hợp bạn muốn job của mình được mở khóa ngay lập tức trước khi nó được xử lý. Để thực hiện điều này, job của bạn nên implement contract `ShouldBeUniqueUntilProcessing` thay vì contract `ShouldBeUnique`:

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

<a name="unique-job-locks"></a>
#### Unique Job Locks

Ở hậu trường, khi một job `ShouldBeUnique` được gửi đi, Laravel sẽ cố gắng lấy [khóa](/docs/{{version}}/cache#atomic-locks) bằng khóa `uniqueId`. Nếu khóa đã được giữ, job đó sẽ không được gửi đi. Khóa này sẽ được giải phóng khi một job hoàn tất quá trình xử lý hoặc thất bại trong tất cả các lần thử lại. Mặc định, Laravel sẽ sử dụng cache driver mặc định để lấy khóa này. Tuy nhiên, nếu bạn muốn sử dụng một driver khác để lấy khóa, bạn có thể định nghĩa một phương thức `uniqueVia` để trả về cache driver sẽ được sử dụng:

```php
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
```

> [!NOTE]
> Nếu bạn chỉ cần giới hạn quá trình xử lý đồng thời của một job, bạn hãy sử dụng middleware job [WithoutOverlapping](/docs/{{version}}/queues#preventing-job-overlaps) thay thế.

<a name="debounced-jobs"></a>
### Debounced Jobs

Thỉnh thoảng, bạn có thể muốn đảm bảo rằng khi cùng một job được gửi nhiều lần trong một khoảng thời gian ngắn, thì chỉ lần gửi cuối mới được thực thi. Bạn có thể làm như vậy bằng cách thêm thuộc tính `DebounceFor` vào job của bạn:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\DebounceFor;

#[DebounceFor(30)]
class UpdateSearchIndex implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(public int $productId)
    {
    }

    /**
     * Get the debounce ID for the job.
     */
    public function debounceId(): string
    {
        return (string) $this->productId;
    }
}
```

Trong ví dụ trên, việc liên tục gửi `UpdateSearchIndex` cho cùng một sản phẩm trong vòng `30` giây sẽ debounce job đó để chỉ lần gửi cuối cùng mới được thực thi.

Nếu bạn muốn giới hạn thời gian một job thường xuyên được gửi có thể bị hoãn, bạn có thể cung cấp tham số `maxWait` cho thuộc tính `DebounceFor`:

```php
#[DebounceFor(30, maxWait: 120)]
class UpdateSearchIndex implements ShouldQueue
{
    use Queueable;

    // ...
}
```

Bạn có thể tùy chỉnh cache store sẽ được sử dụng để theo dõi debounce bằng cách định nghĩa phương thức `debounceVia` trên job của bạn:

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

public function debounceVia(): Repository
{
    return Cache::driver('redis');
}
```

Nếu một debounced job bị thay bởi một lần gửi mới hơn, Laravel sẽ gửi event `Illuminate\Queue\Events\JobDebounced` và xóa job bị thay thế đó ra khỏi queue.

> [!WARNING]
> Các debounced job và các unique job sẽ loại bỏ lẫn nhau. Một job đang sử dụng thuộc tính `DebounceFor` thì không nên implement `ShouldBeUnique`.

> [!WARNING]
> Nếu ứng dụng của bạn gửi các debounced job từ nhiều web server hoặc container, bạn nên đảm bảo rằng tất cả các server của bạn đang kết nối đến cùng một cache server trung tâm.

<a name="encrypted-jobs"></a>
### Encrypted Jobs

Laravel cho phép bạn đảm bảo tính riêng tư và tính toàn vẹn dữ liệu của job thông qua [encryption](/docs/{{version}}/encryption). Để bắt đầu, bạn chỉ cần thêm interface `ShouldBeEncrypted` vào class của job. Sau khi interface này được thêm vào class, Laravel sẽ tự động mã hóa job của bạn trước khi đẩy nó vào queue:

```php
<?php

use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="job-middleware"></a>
## Job Middleware

Job middleware cho phép bạn custom logic của toàn bộ việc chạy các queued job, giảm việc viết code trong các job đó. Ví dụ: phương thức `handle` sau đây có sử dụng các tính năng giới hạn tốc độ của Redis trong Laravel để chỉ cho phép cứ năm giây xử lý một job:

```php
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
```

Mặc dù code này đúng, nhưng implementation của phương thức `handle` đã trở nên quá phức tạp vì nó không đồng nhất với logic giới hạn tốc độ của Redis. Ngoài ra, logic giới hạn tốc độ này cũng phải được copy cho bất kỳ job nào khác mà chúng ta muốn set giới hạn tốc độ. Thay vì giới hạn tốc độ trong phương thức handle, chúng ta có thể định nghĩa một job middleware xử lý giới hạn tốc độ:

```php
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
```

Như bạn có thể thấy, chẳng hạn như [route middleware](/docs/{{version}}/middleware), job middleware sẽ nhận vào một job đang được xử lý và lệnh callback sẽ được gọi để tiếp tục xử lý job đó.

Bạn có thể tạo ra một class middleware job mới bằng cách sử dụng lệnh Artisan `make:job-middleware`. Sau khi tạo xong job middleware, chúng ta có thể được gắn chúng vào một job bằng cách trả lại chúng từ phương thức `middleware` của job. Phương thức này không tồn tại trên các job được tạo bởi lệnh Artisan `make:job`, vì vậy bạn sẽ cần phải tự thêm nó vào định nghĩa job class của bạn:

```php
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
```

> [!NOTE]
> Middleware job cũng có thể được chỉ định cho các [queueable event listeners](/docs/{{version}}/events#queued-event-listeners), [mailables](/docs/{{version}}/mail#queueing-mail), và [notifications](/docs/{{version}}/notifications#queueing-notifications).

<a name="rate-limiting"></a>
### Giới hạn tỷ lệ

Mặc dù chúng ta chỉ trình bày cách viết middleware giới hạn tỷ lệ của riêng bạn, nhưng Laravel thực sự đã chứa một middleware giới hạn tỷ lệ mà bạn có thể sử dụng để xếp hạng các job giới hạn. Giống như [giới hạn tỷ lệ của route](/docs/{{version}}/routing#defining-rate-limiters), bộ giới hạn tỷ lệ job được định nghĩa bằng phương thức `for` của facade `RateLimiter`.

Ví dụ: bạn có thể muốn cho phép người dùng backup lại dữ liệu của họ mỗi giờ một lần trong khi không áp đặt giới hạn đó cho khách hàng cao cấp. Để thực hiện điều này, bạn có thể định nghĩa một `RateLimiter` trong phương thức `boot` của `AppServiceProvider` của bạn:

```php
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
```

Trong ví dụ trên, chúng ta đã định nghĩa giới hạn tỷ lệ theo giờ; tuy nhiên, bạn có thể dễ dàng định nghĩa giới hạn tỷ lệ này dựa trên số phút bằng phương thức `perMinute`. Ngoài ra, bạn có thể truyền bất kỳ giá trị nào mà bạn muốn vào phương thức `by` của giới hạn tỷ lệ; tuy nhiên, giá trị này thường được sử dụng nhiều nhất để phân chia giới hạn tỷ lệ theo khách hàng:

```php
return Limit::perMinute(50)->by($job->user->id);
```

Khi bạn đã định nghĩa xong giới hạn tỷ lệ của bạn, bạn có thể gán giới hạn tỷ lệ này vào job của bạn bằng cách sử dụng middleware `Illuminate\Queue\Middleware\RateLimited`. Mỗi khi job mà vượt quá giới hạn tỷ lệ, middleware này sẽ giải phóng job trở lại về queue với một độ trễ thích hợp dựa trên khoảng thời gian giới hạn tỷ lệ mà bạn đã truyền vào:

```php
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
```

Việc đưa một job có giới hạn tỷ lệ trở lại queue sẽ vẫn làm tăng tổng số `attempts` của job đó. Bạn có thể muốn điều chỉnh các thuộc tính `Tries` và `MaxExceptions` trên class job của bạn cho phù hợp. Hoặc, bạn có thể muốn sử dụng [phương thức retryUntil](#time-based-attempts) để định nghĩa khoảng thời gian cho đến khi job không còn được thực hiện nữa.

Sử dụng phương thức `releaseAfter`, bạn cũng có thể chỉ định số giây cần phải trôi qua trước khi job được release và được thử lại lần nữa:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->releaseAfter(60)];
}
```

Nếu bạn không muốn thử lại một job khi nó bị giới hạn tỷ lệ, bạn có thể sử dụng phương thức `dontRelease`:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->dontRelease()];
}
```

<a name="rate-limiting-with-redis"></a>
#### Rate Limiting With Redis

Nếu đang sử dụng Redis, bạn có thể sử dụng middleware `Illuminate\Queue\Middleware\RateLimitedWithRedis`, middleware này được tinh chỉnh cho Redis và hiệu quả hơn middleware giới hạn tỷ lệ cơ bản:

```php
use Illuminate\Queue\Middleware\RateLimitedWithRedis;

public function middleware(): array
{
    return [new RateLimitedWithRedis('backups')];
}
```

Phương thức `connection` có thể được sử dụng để xác định kết nối Redis nào mà middleware nên sử dụng:

```php
return [(new RateLimitedWithRedis('backups'))->connection('limiter')];
```

<a name="preventing-job-overlaps"></a>
### Chặn Job chồng nhau

Laravel có chứa một middleware `Illuminate\Queue\Middleware\WithoutOverlapping` cho phép bạn ngăn chặn sự chồng nhau của job dựa trên một khóa tùy ý. Điều này có thể hữu ích khi một queued job đang sửa một tài nguyên mà mỗi lần chỉ nên được sửa bởi một job.

Ví dụ: hãy tưởng tượng bạn có một queued job cập nhật điểm tín dụng của người dùng và bạn muốn ngăn chặn sự trùng nhau của job cập nhật điểm tín dụng cho cùng một ID người dùng. Để thực hiện điều này, bạn có thể trả về middleware `WithoutOverlapping` từ phương thức `middleware` của job của bạn:

```php
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
```

Việc release một job chồng nhau trở lại queue vẫn sẽ làm tăng tổng số lần thử của job đó. Bạn có thể muốn điều chỉnh các thuộc tính `Tries` và `MaxExceptions` trong class job của bạn cho phù hợp. Ví dụ: nếu để `Tries` là 1 như mặc định thì sẽ ngăn các job chồng nhau được thử lại sau đó.

Bất kỳ job nào mà cùng loại chồng nhau sẽ được giải phóng trở lại queue. Bạn cũng có thể chỉ định số giây mà job phải đợi trước khi job đó sẽ được thử lại:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
}
```

Nếu bạn muốn xóa ngay lập tức bất kỳ job nào chồng nhau để chúng không được thử lại bất kỳ lần nào nữa, bạn có thể sử dụng phương thức `dontRelease`:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->dontRelease()];
}
```

Middleware `WithoutOverlapping` được thực hiện dựa trên tính năng atomic lock của Laravel. Thỉnh thoảng, job của bạn có thể bị lỗi hoặc hết thời gian chờ khiến cho khóa không được mở. Do đó, bạn có thể định nghĩa một thời gian hết hạn cho khóa bằng phương thức `expireAfter`. Ví dụ ở bên dưới sẽ bảo Laravel mở khóa `WithoutOverlapping` sau ba phút sau khi job được bắt đầu xử lý:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->expireAfter(180)];
}
```

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

```php
use DateTime;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new ThrottlesExceptions(10, 5 * 60)];
}

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(30);
}
```

Tham số đầu tiên mà được middleware chấp nhận là số lượng ngoại lệ mà job có thể đưa ra trước khi được điều chỉnh, trong khi tham số thứ hai là số giây mà trước khi job đó được thử lại sau khi nó được điều chỉnh. Trong code ví dụ ở trên, nếu job đưa ra 10 ngoại lệ liên tiếp, chúng ta sẽ đợi thêm 5 phút trước khi thử lại job, trong giới hạn thời gian 30 phút.

Khi một job đưa ra một ngoại lệ nhưng vẫn chưa đạt đến ngưỡng ngoại lệ, job đó thường sẽ được thử lại ngay lập tức. Tuy nhiên, bạn có thể chỉ định số phút mà một job như vậy sẽ bị trì hoãn bằng cách gọi phương thức `backoff` khi gắn middleware vào job:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 5 * 60))->backoff(5)];
}
```

Ở bên trong, middleware này sẽ sử dụng cache system của Laravel để thực hiện giới hạn tỷ lệ và tên class của job sẽ được sử dụng để làm "khóa" của cache. Bạn có thể ghi đè khóa này bằng cách gọi phương thức `by` khi gắn middleware vào job của bạn. Điều này có thể hữu ích nếu bạn có nhiều job tương tác với cùng một service của bên thứ ba và bạn muốn chúng chia sẻ một "nhóm" điều tiết chung đảm bảo chúng tuân thủ một giới hạn chung duy nhất:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->by('key')];
}
```

Mặc định, middleware này sẽ điều tiết mọi exception. Bạn có thể sửa hành vi này bằng cách gọi phương thức `when` khi gắn middleware vào job của bạn. Exception sẽ chỉ được điều tiết nếu closure được cung cấp cho phương thức `when` trả về `true`:

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->when(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

Không giống như phương thức `when`, dùng để release job trở lại queue hoặc đưa ra một ngoại lệ, phương thức `deleteWhen` cho phép bạn xóa hoàn toàn job đó khi một ngoại lệ nhất định xảy ra:

```php
use App\Exceptions\CustomerDeletedException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(2, 10 * 60))->deleteWhen(CustomerDeletedException::class)];
}
```

Nếu bạn muốn các exception bị điều tiết được report cho exception handler của ứng dụng, bạn có thể làm như vậy bằng cách gọi phương thức `report` khi gắn middleware vào job của bạn. Bạn có thể tuỳ ý cung cấp một closure cho phương thức `report` và exception sẽ chỉ được report nếu closure được cung cấp trả về `true`:

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->report(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

<a name="throttling-exceptions-with-redis"></a>
#### Throttling Exceptions With Redis

Nếu bạn đang sử dụng Redis, bạn có thể sử dụng middleware `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis`, middleware này được tinh chỉnh cho Redis và hiệu quả hơn middleware bình thường:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis;

public function middleware(): array
{
    return [new ThrottlesExceptionsWithRedis(10, 10 * 60)];
}
```

Phương thức `connection` có thể được sử dụng để xác định kết nối Redis nào mà middleware nên sử dụng:

```php
return [(new ThrottlesExceptionsWithRedis(10, 10 * 60))->connection('limiter')];
```

<a name="skipping-jobs"></a>
### Bỏ qua Job

Middleware `Skip` cho phép bạn chỉ định rằng một job sẽ được bỏ qua hoặc xóa mà không cần sửa logic của job. Phương thức `Skip::when` sẽ xóa job nếu điều kiện được cung cấp trả về `true`, trong khi phương thức `Skip::unless` sẽ xóa job nếu điều kiện được cung cấp trả về `false`:

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [
        Skip::when($condition),
    ];
}
```

Bạn có thể truyền vào một `Closure` cho phương thức `when` và `unless` để thực hiện một điều kiện phức tạp hơn:

```php
use Illuminate\Queue\Middleware\Skip;

/**
* Get the middleware the job should pass through.
*/
public function middleware(): array
{
    return [
        Skip::when(function (): bool {
            return $this->shouldSkip();
        }),
    ];
}
```

<a name="dispatching-jobs"></a>
## Gửi Job

Khi bạn đã viết xong các class job của bạn, bạn có thể dispatch nó bằng cách sử dụng phương thức `dispatch` trên chính job đó. Các tham số được truyền vào cho phương thức `dispatch` sẽ được truyền lại vào hàm khởi tạo của job:

```php
<?php

namespace App\Http\Controllers;

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
```

Nếu bạn muốn gửi một job có điều kiện, bạn có thể sử dụng các phương thức `dispatchIf` và `dispatchUnless`:

```php
ProcessPodcast::dispatchIf($accountActive, $podcast);

ProcessPodcast::dispatchUnless($accountSuspended, $podcast);
```

Trong các ứng dụng Laravel mới, kết nối `database` sẽ được định nghĩa là queue mặc định. Bạn có thể chỉ định một kết nối queue mặc định khác bằng cách thay đổi biến môi trường `QUEUE_CONNECTION` trong file `.env` của ứng dụng.

<a name="delayed-dispatching"></a>
### Delay gửi

Nếu bạn muốn chỉ định rằng một job sẽ không được xử lý ngay bởi một queue worker, thì bạn có thể sử dụng phương thức `delay` khi đang dispatch một job. Ví dụ: hãy khai báo một job không nên được xử lý cho đến 10 phút sau khi job đó được dispatch:

```php
<?php

namespace App\Http\Controllers;

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
```

Trong một số trường hợp, các job có thể có thời gian delay mặc định đã được cấu hình sẵn. Nếu bạn cần bỏ qua thời gian delay này và gửi ngay một job để xử lý ngay lập tức, bạn có thể sử dụng phương thức `withoutDelay`:

```php
ProcessPodcast::dispatch($podcast)->withoutDelay();
```

> [!WARNING]
> service SQS queue của Amazon có thời gian delay tối đa là 15 phút.

<a name="synchronous-dispatching"></a>
### Đồng bộ gửi

Nếu bạn muốn gửi một job được chạy ngay lập tức (một cách đồng bộ), bạn có thể sử dụng phương thức `dispatchSync`. Khi sử dụng phương thức này, job sẽ không được queue và sẽ được chạy ngay lập tức trong process hiện tại:

```php
<?php

namespace App\Http\Controllers;

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
```

<a name="deferred-dispatching"></a>
#### Deferred Dispatching

Bằng cách sử dụng deferred synchronous dispatching, bạn có thể gửi một job để xử lý trong process hiện tại nhưng sau khi HTTP response đã được gửi đến người dùng. Điều này cho phép bạn xử lý các job "queued" một cách đồng bộ mà không làm chậm trải nghiệm ứng dụng của người dùng. Để trì hoãn việc chạy một job đồng bộ, hãy gửi job đó tới connection `deferred`:

```php
RecordDelivery::dispatch($order)->onConnection('deferred');
```

Connection `deferred` cũng đóng vai trò là [failover queue](#queue-failover) mặc định.

Tương tự, connection `background` xử lý các job sau khi HTTP response đã được gửi đến người dùng; tuy nhiên, job sẽ được xử lý trong một process PHP riêng, cho phép PHP-FPM hoặc application worker sẵn sàng xử lý các HTTP request khác mà không cần chờ đợi:

```php
RecordDelivery::dispatch($order)->onConnection('background');
```

<a name="jobs-and-database-transactions"></a>
### Jobs và Database Transactions

Mặc dù việc gửi job trong các transaction của cơ sở dữ liệu là hoàn toàn ổn, nhưng bạn nên đặc biệt cẩn thận để đảm bảo rằng job của bạn thực sự có thể thực hiện thành công. Khi gửi một job trong một transaction, có thể job sẽ được xử lý bởi một worker trước khi transaction đó được thực hiện. Khi điều này xảy ra, mọi cập nhật mà bạn đã thực hiện đối với model hoặc bản ghi cơ sở dữ liệu trong transaction cơ sở dữ liệu có thể chưa được lưu vào trong cơ sở dữ liệu. Ngoài ra, mọi model hoặc bản ghi cơ sở dữ liệu được tạo trong transaction có thể không tồn tại trong cơ sở dữ liệu.

Rất may, Laravel cung cấp một số phương thức để giải quyết vấn đề này. Trước tiên, bạn có thể set tùy chọn kết nối `after_commit` trong mảng cấu hình của kết nối queue của bạn:

```php
'redis' => [
    'driver' => 'redis',
    // ...
    'after_commit' => true,
],
```

Khi tùy chọn `after_commit` được set là `true`, bạn có thể gửi job trong transaction của cơ sở dữ liệu; tuy nhiên, Laravel sẽ đợi cho đến khi tất cả các transaction của cơ sở dữ liệu được hoàn tất trước khi thực sự gửi job. Tất nhiên, nếu hiện tại không có transaction cơ sở dữ liệu nào thì job sẽ được gửi đi ngay lập tức.

Nếu một transaction bị roll back lại do một ngoại lệ xảy ra trong quá trình transaction, thì các job đã được gửi trong transaction đó sẽ bị loại bỏ.

> [!NOTE]
> Việc set tùy chọn cấu hình `after_commit` thành `true` cũng sẽ khiến mọi event listener, mailable, notification và các broadcast event mà đã được queue lại sẽ được gửi đi sau khi tất cả các transaction cơ sở dữ liệu được thực hiện xong.

<a name="specifying-commit-dispatch-behavior-inline"></a>
#### Specifying Commit Dispatch Behavior Inline

Nếu bạn không set tùy chọn cấu hình kết nối queue `after_commit` thành `true`, bạn vẫn có thể chỉ định một job cụ thể sẽ được gửi đi sau khi tất cả các transaction cơ sở dữ liệu đã được hoàn tất. Để thực hiện điều này, bạn có thể kết hợp thêm phương thức `afterCommit` vào sau phương thức gửi của bạn:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->afterCommit();
```

Tương tự, nếu tùy chọn cấu hình `after_commit` được set thành `true`, bạn có thể chỉ định một job cụ thể sẽ được gửi đi ngay lập tức mà không cần đợi bất kỳ transaction cơ sở dữ liệu nào được thực hiện:

```php
ProcessPodcast::dispatch($podcast)->beforeCommit();
```

<a name="job-chaining"></a>
### Kết hợp Job

Kết hợp job cho phép bạn khai báo một danh sách các queued job sẽ được chạy theo một trình tự nhất định sau khi job chính được thực thi thành công. Nếu một job trong danh sách bị thất bại, thì các job còn lại cũng sẽ không được chạy. Để thực hiện một danh sách queued job, bạn có thể sử dụng phương thức `chain` được cung cấp bởi facade `Bus`. Lệnh bus của Laravel là một component cấp thấp, chức năng gửi queued job được xây dựng dựa trên nó:

```php
use App\Jobs\OptimizePodcast;
use App\Jobs\ProcessPodcast;
use App\Jobs\ReleasePodcast;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();
```

Ngoài việc kết hợp các instance của job class, bạn cũng có thể kết hợp thêm các closures:

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    function () {
        Podcast::update(/* ... */);
    },
])->dispatch();
```

> [!WARNING]
> Việc xóa nhiều job bằng phương thức `$this->delete()` trong một job cụ thể sẽ không ngăn một chuỗi job ngừng xử lý. Chuỗi job sẽ chỉ bị ngừng xử lý nếu một job trong chuỗi job đó bị thất bại.

<a name="chain-connection-queue"></a>
#### Chain Connection và Queue

Nếu bạn muốn chỉ định kết nối và queue sẽ được sử dụng cho một chuỗi job, bạn có thể sử dụng phương thức `onConnection` và `onQueue`. Các phương thức này sẽ chỉ định kết nối queue và tên queue sẽ được sử dụng trừ khi queue job của bạn được chỉ định trong một kết nối hoặc một queue khác:

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->onConnection('redis')->onQueue('podcasts')->dispatch();
```

<a name="adding-jobs-to-the-chain"></a>
#### Adding Jobs to the Chain

Đôi khi, bạn có thể cần thêm một job vào đầu hoặc cuối của một chuỗi job hiện tại, từ bên trong một job khác có trong chuỗi job đó. Bạn có thể thực hiện điều này bằng cách sử dụng các phương thức `prependToChain` và `appendToChain`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    // Prepend to the current chain, run job immediately after current job...
    $this->prependToChain(new TranscribePodcast);

    // Append to the current chain, run job at end of chain...
    $this->appendToChain(new TranscribePodcast);
}
```

<a name="chain-failures"></a>
#### Chain Failures

Khi kết hợp các job, bạn có thể sử dụng phương thức `catch` để chỉ định một closure sẽ được gọi nếu một job trong chuỗi bị lỗi. Callback đã cho sẽ nhận vào instance `Throwable` gây ra lỗi job:

```php
use Illuminate\Support\Facades\Bus;
use Throwable;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->catch(function (Throwable $e) {
    // A job within the chain has failed...
})->dispatch();
```

> [!WARNING]
> Vì các chuỗi callback sẽ được chuyển đổi và thực thi sau đó bởi Laravel queue, nên bạn không nên sử dụng biến `$this` trong các chuỗi callback.

<a name="customizing-the-queue-and-connection"></a>
### Tuỳ biến Queue và Connection

<a name="dispatching-to-a-particular-queue"></a>
#### Dispatching đến một Queue cụ thể

Bằng cách tạo các job đến các queue khác nhau, bạn có thể "phân loại" các queued job của bạn và thậm chí là ưu tiên bao nhiêu worker sẽ được gán cho mỗi queue. Hãy nhớ rằng, điều này không đẩy job đến các queue "connection" khác mà có ở trong file định nghĩa cấu hình queue của bạn, mà chỉ đẩy đến các queue có trong một connection cụ thể. Để khai báo queue, sử dụng phương thức `onQueue` khi gửi job:

```php
<?php

namespace App\Http\Controllers;

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
```

Ngoài ra, bạn có thể chỉ định queue của job bằng cách gọi phương thức `onQueue` trong hàm khởi tạo của job:

```php
<?php

namespace App\Jobs;

 use Illuminate\Contracts\Queue\ShouldQueue;
 use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onQueue('processing');
    }
}
```

<a name="dispatching-to-a-particular-connection"></a>
#### Dispatching To A Particular Connection

Nếu application của bạn đang làm việc với nhiều queue connection, bạn có thể khai báo connection nào sẽ sử dụng cho job đó bằng cách sử dụng phương thức `onConnection`:

```php
<?php

namespace App\Http\Controllers;

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
```

Bạn có thể kết hợp các phương thức `onConnection` và `onQueue` với nhau để khai báo connection và queue cho một job:

```php
ProcessPodcast::dispatch($podcast)
    ->onConnection('sqs')
    ->onQueue('processing');
```

Ngoài ra, bạn có thể chỉ định kết nối của job bằng cách gọi phương thức `onConnection` trong hàm khởi tạo của job:

```php
<?php

namespace App\Jobs;

 use Illuminate\Contracts\Queue\ShouldQueue;
 use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onConnection('sqs');
    }
}
```

<a name="queue-routing"></a>
#### Queue Routing

Bạn có thể sử dụng phương thức `route` của facade `Queue` để xác định một connection và queue mặc định cho các job class nhất định. Điều này hữu ích khi bạn muốn đảm bảo một số job nhất định luôn sử dụng các queue đã định sẵn mà không cần khai báo connection hoặc queue ngay trên chính job đó.

Ngoài việc route các job class, bạn cũng có thể truyền một interface, trait hoặc class cha vào phương thức `route`. Khi bạn làm điều này, bất kỳ job nào mà implement interface đó hoặc sử dụng trait đó hoặc kế thừa class cha đó sẽ tự động sử dụng connection và queue mà bạn đã cấu hình.

Thông thường, bạn nên gọi phương thức `route` từ phương thức `boot` của một service provider:

```php
use App\Concerns\RequiresVideo;
use App\Jobs\ProcessPodcast;
use App\Jobs\ProcessVideo;
use Illuminate\Support\Facades\Queue;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
    Queue::route(RequiresVideo::class, queue: 'video');
}
```

Khi một connection được khai báo mà không có queue, thì job sẽ được gửi đến queue mặc định:

```php
Queue::route(ProcessPodcast::class, connection: 'redis');
```

Bạn cũng có thể route nhiều job class cùng một lúc bằng cách truyền một mảng vào phương thức `route`:

```php
Queue::route([
    ProcessPodcast::class => ['podcasts', 'redis'], // Queue and connection
    ProcessVideo::class => 'videos', // Queue only (uses default connection)
]);
```

> [!NOTE]
> Việc route queue vẫn có thể được ghi đè bởi chính job đó.

<a name="max-job-attempts-and-timeout"></a>
### Khai báo số lần chạy Job tối đa / giá trị timeout

<a name="max-attempts"></a>
#### Max Attempts

Số lần thử job là một khái niệm cốt lõi của hệ thống queue trong Laravel và là nền tảng cho rất nhiều tính năng nâng cao. Mặc dù ban đầu chúng có vẻ gây nhầm lẫn, nhưng điều quan trọng là phải hiểu cách chúng hoạt động trước khi thay đổi cấu hình mặc định.

Khi một job được gửi đi, nó sẽ được đẩy vào queue. Một worker sau đó sẽ lấy nó ra và cố gắng thực thi nó. Đây chính là một lần thử job.

Tuy nhiên, một lần thử không nhất thiết có nghĩa là phương thức `handle` của job đã được thực thi. Các lần thử cũng có thể bị "tiêu tốn" theo nhiều cách khác nhau:

<div class="content-list" markdown="1">

- Job gặp một ngoại lệ chưa được xử lý trong quá trình thực thi.
- Job bị release trở lại queue bằng cách sử dụng `$this->release()`.
- Middleware chẳng hạn như `WithoutOverlapping` hoặc `RateLimited` không lấy được khóa và release job.
- Job đã bị timeout.
- Phương thức `handle` của job chạy và hoàn tất mà không đưa ra ngoại lệ.

</div>

Có lẽ bạn không muốn thực hiện một job vô tận. Do đó, Laravel cung cấp nhiều cách khác nhau để xác định số lần hoặc thời gian thực hiện một job.

> [!NOTE]
> Mặc định, Laravel sẽ chỉ thử thực hiện một job trong một lần duy nhất. Nếu job của bạn sử dụng middleware như `WithoutOverlapping` hoặc `RateLimited`, hoặc nếu bạn đang gọi lệnh release job, bạn có thể sẽ cần tăng số lần thử được phép thông qua tùy chọn `tries`.

Một cách tiếp cận để khai báo số lần tối đa mà một job có thể được chạy là thông qua switch `--tries` trên lệnh Artisan. Điều này sẽ áp dụng cho tất cả các job do worker này xử lý trừ khi job đang được xử lý chỉ định một số cụ thể mà job đó có thể được thực hiện:

```shell
php artisan queue:work --tries=3
```

Nếu một job vượt quá số lần thử tối đa, nó sẽ bị coi là một job "thất bại". Để biết thêm thông tin về cách xử lý những job thất bại, hãy tham khảo [tài liệu về job thất bại](#dealing-with-failed-jobs). Nếu `--tries=0` được cung cấp cho lệnh `queue:work`, thì job đó sẽ được thử lại vô số lần.

Bạn có thể thực hiện một cách tiếp cận chi tiết hơn bằng cách định nghĩa số lần chạy tối đa mà một job có thể thử trên chính class của job bằng cách dùng thuộc tính `Tries`. Nếu số lần chạy tối đa được chỉ định trong job, thì nó sẽ ưu tiên giá trị `--tries` này hơn là giá trị được cung cấp trên dòng lệnh:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\Tries;

#[Tries(5)]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

Nếu bạn cần kiểm số lần thử tối đa của một job cụ thể, bạn có thể định nghĩa phương thức `tries` trên job đó:

```php
/**
 * Determine number of times the job may be attempted.
 */
public function tries(): int
{
    return 5;
}
```

<a name="time-based-attempts"></a>
#### Time Based Attempts

Thay thế cho việc định nghĩa số lần một job có thể được chạy trước khi nó thất bại, bạn có thể định nghĩa thời gian mà job đó sẽ không còn được thử lại. Điều này cho phép một job có thể được chạy thoải mái trong một khoảng thời gian nhất định. Để định nghĩa thời gian mà một job sẽ không còn được thử lại, hãy thêm phương thức `retryUntil` vào class job của bạn. Phương thức này sẽ trả về một instance `DateTime`:

```php
use DateTime;

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(10);
}
```

If both `retryUntil` and `tries` are defined, Laravel gives precedence to the `retryUntil` method.

> [!NOTE]
> Bạn cũng có thể định nghĩa một thuộc tính `Tries` hoặc phương thức `retryUntil` trên các [queued event listener](/docs/{{version}}/events#queued-event-listeners) và [queued notifications](/docs/{{version}}/notifications#queueing-notifications) của bạn.

<a name="max-exceptions"></a>
#### Max Exceptions

Thỉnh thoảng bạn có thể muốn chỉ định một job có thể được thử lại nhiều lần, nhưng sẽ thất bại nếu trong các lần thử lại được kích hoạt bởi một số lượng exception chưa được xử lý nhất định (ngược lại với việc được giải phóng trực tiếp bằng phương thức `release`). Để thực hiện điều này, bạn có thể dùng thuộc tính `Tries` và thuộc tính `maxExceptions` trên class job của bạn:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\MaxExceptions;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Support\Facades\Redis;

#[Tries(25)]
#[MaxExceptions(3)]
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

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
```

Trong ví dụ này, job sẽ được giải phóng trong 10 giây nếu ứng dụng không thể lấy được Redis lock và sẽ tiếp tục được thử lại tối đa 25 lần. Tuy nhiên, job sẽ thất bại nếu job đưa ra quá ba exception.

<a name="timeout"></a>
#### Timeout

Thông thường, bạn có thể ước lượng queued job của bạn sẽ mất bao lâu thời gian để chạy. Vì lý do này, Laravel cho phép bạn chỉ định giá trị "timeout". Mặc định, giá trị timeout này là 60 giây. Nếu một job đang xử lý lâu hơn số giây được chỉ định bởi giá trị timeout, thì worker đang xử lý job đó sẽ bị thoát ra với một lỗi. Thông thường, worker sẽ được khởi động lại tự động bởi [trình quản lý process được cấu hình trên máy chủ của bạn](#supervisor-configuration).

Thời gian hết hạn của một job có thể được khai báo bằng cách sử dụng switch `--timeout` trên lệnh Artisan:

```shell
php artisan queue:work --timeout=30
```

Nếu job vượt quá số lần thử tối đa do liên tục hết thời gian chờ, nó sẽ bị đánh dấu là thất bại.

Bạn cũng có thể định nghĩa thời gian hết hạn của một job bằng cách dùng thuộc tính `Timeout` trên chính class của job. Nếu thời gian hết hạn được khai báo trong job, nó sẽ được ưu tiên hơn bất kỳ thời gian hết hạn nào được khai báo trên dòng lệnh:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\Timeout;

#[Timeout(120)]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

Thỉnh thoảng, các process IO blocking như socket hoặc outgoing HTTP connection có thể không tuân theo thời gian hết hạn đã được chỉ định của bạn. Do đó, khi sử dụng các tính năng này, bạn cũng nên cố gắng chỉ định một thời gian hết hạn bằng cách sử dụng các API của chúng. Ví dụ: khi sử dụng [Guzzle](https://docs.guzzlephp.org), bạn phải luôn chỉ định một connection và một giá trị mà request sẽ hết hạn.

> [!WARNING]
> PHP extension [PCNTL](https://www.php.net/manual/en/book.pcntl.php) phải được cài đặt để chỉ định thời gian timeout của job. Ngoài ra, giá trị "timeout" của job phải luôn nhỏ hơn giá trị ["retry after"](#job-expiration) của nó. Nếu không, job có thể bị thử lại trước khi hoàn tất công việc hoặc hết thời gian timeout.

<a name="failing-on-timeout"></a>
#### Failing On Timeout

Nếu bạn muốn chỉ định một job phải được đánh dấu là [thất bại](#dealing-with-failed-jobs) khi hết thời gian chờ, bạn có thể dùng thuộc tính `FailOnTimeout` trên class job:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\FailOnTimeout;

#[FailOnTimeout]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

> [!NOTE]
> Mặc định, khi một job bị timeout, nó sẽ mất một lần thử và được release trở lại queue (nếu còn lần thử lại). Tuy nhiên, nếu bạn cấu hình job bị thất bại khi bị timeout, thì nó sẽ không được thử lại nữa, bất kể giá trị nào đã được set cho số lần thử.

<a name="sqs-fifo-and-fair-queues"></a>
### SQS FIFO và Fair Queues

Laravel hỗ trợ các queue [Amazon SQS FIFO (First-In-First-Out)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fifo-queues.html) và [fair](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fair-queues.html), cho phép bạn xử lý các job theo đúng thứ tự mà chúng đã được gửi đi, đồng thời đảm bảo việc xử lý chính xác trong một lần duy nhất thông qua việc loại bỏ các job trùng lặp.

Các queue FIFO yêu cầu một message group ID để xác định xem những job nào có thể được xử lý song song. Các job có cùng group ID sẽ được xử lý theo thứ tự, trong khi các job có group ID khác nhau có thể được xử lý đồng thời.

Laravel cung cấp một phương thức `onGroup` linh hoạt để chỉ định message group ID khi gửi job:

```php
ProcessOrder::dispatch($order)
    ->onGroup("customer-{$order->customer_id}");
```

Các queue SQS FIFO hỗ trợ tính năng loại bỏ các job trùng lặp để đảm bảo việc xử lý chính xác một lần duy nhất. Hãy implement một phương thức `deduplicationId` trong class job của bạn để cung cấp một deduplication ID tùy chỉnh:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessSubscriptionRenewal implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Get the job's deduplication ID.
     */
    public function deduplicationId(): string
    {
        return "renewal-{$this->subscription->id}";
    }
}
```

<a name="fair-queues"></a>
#### Fair Queues

Nếu bạn đang sử dụng SQS standard queue, việc thiết lập một message group sẽ cho phép bạn bật tính năng fair queueing. Nói cách khác, một khi bạn đã chỉ định group, SQS sẽ sử dụng chúng để duy trì sự phân phối thông qua các tenant và khối lượng công việc. Bạn không cần phải cấu hình gì thêm cho Laravel.

Thay vì gọi `onGroup` tại thời điểm gửi job, bạn cũng có thể định nghĩa phương thức `messageGroup` trực tiếp trên job:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessOrder implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Get the job's message group.
     */
    public function messageGroup(): string
    {
        return "customer-{$this->order->customer_id}";
    }
}
```

<a name="fifo-listeners-mail-and-notifications"></a>
#### FIFO Listeners, Mail, and Notifications

Khi sử dụng các queue FIFO, bạn cũng cần định nghĩa các message group trên các listeners, mail và notifications. Ngoài ra, bạn có thể gửi các instance queue của các đối tượng này tới một queue không phải FIFO.

Để định nghĩa message group cho một [queued event listener](/docs/{{version}}/events#queued-event-listeners), hãy định nghĩa phương thức `messageGroup` trên listener đó. Bạn cũng có thể tùy chọn định nghĩa thêm một phương thức `deduplicationId`:

```php
<?php

namespace App\Listeners;

class SendShipmentNotification
{
    // ...

    /**
     * Get the job's message group.
     */
    public function messageGroup(): string
    {
        return 'shipments';
    }

    /**
     * Get the job's deduplication ID.
     */
    public function deduplicationId(): string
    {
        return "shipment-notification-{$this->shipment->id}";
    }
}
```

Khi chuẩn bị gửi một [mail message](/docs/{{version}}/mail) vào queue FIFO, bạn nên gọi phương thức `onGroup` và có thể tùy chọn thêm phương thức `withDeduplicator` khi gửi:

```php
use App\Mail\InvoicePaid;
use Illuminate\Support\Facades\Mail;

$invoicePaid = (new InvoicePaid($invoice))
    ->onGroup('invoices')
    ->withDeduplicator(fn () => 'invoices-'.$invoice->id);

Mail::to($request->user())->send($invoicePaid);
```

Khi chuẩn bị gửi một [notification](/docs/{{version}}/notifications) vào queue FIFO, bạn nên gọi phương thức `onGroup` và có thể tùy chọn thêm phương thức `withDeduplicator` khi gửi:

```php
use App\Notifications\InvoicePaid;

$invoicePaid = (new InvoicePaid($invoice))
    ->onGroup('invoices')
    ->withDeduplicator(fn () => 'invoices-'.$invoice->id);

$user->notify($invoicePaid);
```

<a name="queue-failover"></a>
### Queue Failover

Driver queue `failover` sẽ cung cấp chức năng failover tự động khi đẩy các job vào queue. Nếu kết nối queue chính của cấu hình `failover` bị lỗi vì bất kỳ lý do gì, thì Laravel sẽ tự động đẩy job sang kết nối tiếp theo được cấu hình trong danh sách. Điều này đặc biệt hữu ích để đảm bảo tính sẵn sàng cao trong môi trường production, nơi mà độ tin cậy của queue là cực kỳ quan trọng.

Để cấu hình một kết nối queue failover, hãy chỉ định driver là `failover` và cung cấp một mảng gồm tên các kết nối để thử theo thứ tự. Mặc định, Laravel sẽ chứa một cấu hình failover mẫu trong file cấu hình `config/queue.php` của ứng dụng:

```php
'failover' => [
    'driver' => 'failover',
    'connections' => [
        'redis',
        'database',
        'sync',
    ],
],
```

Sau khi bạn đã cấu hình một kết nối sử dụng driver `failover`, bạn sẽ cần đặt kết nối failover này làm kết nối queue mặc định trong file `.env` của ứng dụng để sử dụng chức năng failover:

```ini
QUEUE_CONNECTION=failover
```

Tiếp theo, hãy chạy ít nhất một worker cho mỗi kết nối có trong danh sách kết nối failover của bạn:

```bash
php artisan queue:work redis
php artisan queue:work database
```

> [!NOTE]
> Bạn không cần phải chạy worker cho các kết nối sử dụng driver `sync`, `background`, hoặc `deferred` vì các driver này xử lý các job ngay trong process PHP hiện tại.

Khi một thao tác kết nối queue thất bại và chức năng failover được kích hoạt, Laravel sẽ gửi event `Illuminate\Queue\Events\QueueFailedOver`, cho phép bạn report hoặc ghi log lại một kết nối queue đã bị lỗi.

> [!NOTE]
> Nếu bạn sử dụng Laravel Horizon, hãy nhớ rằng Horizon chỉ quản lý các queue Redis. Nếu danh sách failover của bạn có chứa `database`, bạn nên chạy một process `php artisan queue:work database` song song với Horizon.

<a name="error-handling"></a>
### Error Handling

Nếu một ngoại lệ được đưa ra trong khi một job đang được xử lý, job đó sẽ tự động được giải phóng trở lại về queue để có thể thử lại. Job đó sẽ tiếp tục được thực hiện cho đến khi nó được thử với số lần tối đa mà ứng dụng của bạn cho phép. Số lần thử tối đa được định nghĩa bởi switch `--tries` được sử dụng trong lệnh Artisan `queue:work`. Ngoài ra, số lần thử tối đa cũng có thể được định nghĩa trên chính class job đó. Thông tin thêm về cách chạy queue worker [có thể tìm thấy bên dưới](#running-the-queue-worker).

<a name="manually-releasing-a-job"></a>
#### Manually Releasing A Job

Thỉnh thoảng bạn có thể muốn đưa một job trở lại về queue theo cách thủ công để có thể thử lại sau đó. Bạn có thể thực hiện điều này bằng cách gọi phương thức `release`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    $this->release();
}
```

Mặc định, phương thức `release` sẽ giải phóng job trở lại queue ngay lập tức. Tuy nhiên, bạn có thể ra lệnh cho queue sẽ không xử lý job cho đến khi hết một số giây nhất định bằng cách truyền một số nguyên hoặc một date vào phương thức `release`:

```php
$this->release(10);

$this->release(now()->plus(seconds: 10));
```

<a name="manually-failing-a-job"></a>
#### Manually Failing A Job

Đôi khi bạn có thể cần đánh dấu một job là "thất bại". Để làm như vậy, bạn có thể gọi phương thức `fail`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    $this->fail();
}
```

Nếu bạn muốn đánh dấu job của bạn là thất bại vì một ngoại lệ mà bạn đã gặp phải, bạn có thể truyền ngoại lệ đó cho phương thức `fail`. Hoặc để thuận tiện hơn, bạn có thể truyền một thông báo lỗi dưới dạng string để nó sẽ được chuyển đổi thành ngoại lệ cho bạn:

```php
$this->fail($exception);

$this->fail('Something went wrong.');
```

> [!NOTE]
> Để biết thêm thông tin về các job thất bại, hãy xem [tài liệu về cách xử lý cho các job thất bại](#dealing-with-failed-jobs).

<a name="fail-jobs-on-exceptions"></a>
#### Failing Jobs on Specific Exceptions

[Job middleware](#job-middleware) `FailOnException` cho phép bạn ngắt quãng các lần thử lại khi các ngoại lệ cụ thể được đưa ra. Điều này cho phép thử lại đối với các ngoại lệ có tính tạm thời chẳng hạn như lỗi API bên ngoài, nhưng làm cho job thất bại luôn khi gặp các ngoại lệ cố định, chẳng hạn như quyền của một người dùng bị thu hồi:

```php
<?php

namespace App\Jobs;

use App\Models\User;
use Illuminate\Auth\Access\AuthorizationException;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\Middleware\FailOnException;
use Illuminate\Support\Facades\Http;

#[Tries(3)]
class SyncChatHistory implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        $this->user->authorize('sync-chat-history');

        $response = Http::throw()->get(
            "https://chat.laravel.test/?user={$this->user->uuid}"
        );

        // ...
    }

    /**
     * Get the middleware the job should pass through.
     */
    public function middleware(): array
    {
        return [
            new FailOnException([AuthorizationException::class])
        ];
    }
}
```

<a name="job-batching"></a>
## Job Batching

Tính năng job batching của Laravel cho phép bạn dễ dàng thực thi một nhóm job cùng lúc và thực hiện một số hành động nhất định ngay khi nhóm job đó đã hoàn thành việc thực thi.

Trước khi bắt đầu, bạn nên tạo một migration cơ sở dữ liệu để tạo một bảng mà sẽ chứa các thông tin meta về các batch job của bạn, chẳng hạn như tỷ lệ hoàn thành của chúng. Migration này có thể được tạo bằng lệnh Artisan `make:queue-batches-table`:

```shell
php artisan make:queue-batches-table

php artisan migrate
```

<a name="defining-batchable-jobs"></a>
### Định nghĩa Batchable Jobs

Để định nghĩa một batch job, bạn nên [tạo một queue job](#creating-jobs) như bình thường; tuy nhiên, bạn nên thêm trait `Illuminate\Bus\Batchable` vào class job. Trait này cung cấp quyền truy cập vào phương thức `batch` có thể được sử dụng để lấy ra các batch hiện có mà job đang được thực thi:

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Batchable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ImportCsv implements ShouldQueue
{
    use Batchable, Queueable;

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
```

<a name="dispatching-batches"></a>
### Gửi Batches

Để gửi một batch job, bạn nên sử dụng phương thức `batch` của facade `Bus`. Tất nhiên, việc tạo batch chủ yếu hữu ích khi kết hợp với các lệnh callback. Vì vậy, bạn có thể sử dụng các phương thức `then`, `catch` và `final` để định nghĩa các lệnh callback cho batch. Mỗi lệnh callback này sẽ nhận vào một instance `Illuminate\Bus\Batch` khi chúng được gọi.

Khi chạy nhiều queue worker, các job trong batch sẽ được xử lý song song. Vì vậy, thứ tự mà các job hoàn thành có thể không giống với thứ tự mà chúng đã được thêm vào batch. Hãy xem tài liệu của chúng tôi về [job chains và batches](#chains-and-batches) để biết thông tin về cách chạy một batch job theo thứ tự.

Trong ví dụ này, chúng ta sẽ tưởng tượng là chúng ta đang queue một batch job mà mỗi job sẽ xử lý một số hàng nhất định từ file CSV:

```php
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
    // Batch job failure detected...
})->finally(function (Batch $batch) {
    // The batch has finished executing...
})->dispatch();

return $batch->id;
```

ID của batch có thể được lấy ra thông qua thuộc tính `$batch->id`, nó có thể được sử dụng để [truy vấn lệnh bus của Laravel](#inspecting-batches) để biết thêm thông tin về batch sau khi nó được gửi đi.

> [!WARNING]
> Vì các lệnh callback của batch sẽ được chuyển đổi rồi mới được thực thi sau đó bởi Laravel queue, nên bạn không nên sử dụng biến `$this` trong các lệnh callback. Ngoài ra, vì các job trong batch được bọc trong các transaction cơ sở dữ liệu, nên các câu lệnh cơ sở dữ liệu gây ra các commit ngầm không nên được thực thi ở bên trong các job.

<a name="naming-batches"></a>
#### Naming Batches

Một số công cụ như [Laravel Horizon](/docs/{{version}}/horizon) và [Laravel Telescope](/docs/{{version}}/telescope) có thể cung cấp thông tin gỡ lỗi thân thiện hơn cho các batch nếu các batch đó đã được đặt tên. Để gán tên cho một batch, bạn có thể gọi phương thức `name` trong khi định nghĩa batch:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->name('Import CSV')->dispatch();
```

<a name="batch-connection-queue"></a>
#### Batch Connection và Queue

Nếu bạn muốn chỉ định kết nối và queue nào sẽ được sử dụng cho các batch job, bạn có thể sử dụng các phương thức `onConnection` và `onQueue`. Tất cả các batch job sẽ phải chạy trong cùng một kết nối và queue:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->onConnection('redis')->onQueue('imports')->dispatch();
```

<a name="chains-and-batches"></a>
### Chains and Batches

Bạn có thể định nghĩa một tập hợp gồm các [chuỗi job](#job-chaining) trong một batch bằng cách set các chuỗi job đó vào trong một mảng. Ví dụ: chúng ta có thể thực hiện song song hai chuỗi job và thực hiện lệnh callback khi cả hai chuỗi job đã được xử lý xong:

```php
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
    // All jobs completed successfully...
})->dispatch();
```

Ngược lại, bạn có thể chạy nhiều batch job trong một [chuỗi job](#job-chaining) bằng cách định nghĩa các batch này trong một chuỗi. Ví dụ, trước tiên bạn có thể chạy một batch job để phát hành nhiều podcast sau đó là một batch job khác để gửi thông báo phát hành:

```php
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
```

<a name="adding-jobs-to-batches"></a>
### Thêm Jobs vào Batches

Thỉnh thoảng việc thêm một job vào trong một batch từ bên trong một job có thể có hữu ích. Điều này có thể có hữu ích khi bạn cần xử lý hàng nghìn job mà có thể mất quá nhiều thời gian để gửi đi trong một request web. Vì vậy, thay vào đó, bạn có thể muốn gửi một nhóm job "loader" đầu tiên để cung cấp cho batch đó rồi sẽ thêm nhiều job hơn nữa vào sau đó:

```php
$batch = Bus::batch([
    new LoadImportBatch,
    new LoadImportBatch,
    new LoadImportBatch,
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->name('Import Contacts')->dispatch();
```

Trong ví dụ này, chúng ta sẽ sử dụng job `LoadImportBatch` để tái tạo lại batch với các job bổ sung. Để thực hiện điều này, chúng ta có thể sử dụng phương thức `add` trên instance batch có thể được truy cập thông qua phương thức `batch` của job:

```php
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
```

> [!WARNING]
> Bạn chỉ có thể thêm job vào một batch từ bên trong job thuộc cùng một batch.

<a name="inspecting-batches"></a>
### Kiểm tra Batches

Instance `Illuminate\Bus\Batch` được cung cấp cho lệnh callback batch sẽ có nhiều thuộc tính và phương thức khác nhau để hỗ trợ bạn tương tác và kiểm tra một batch job nhất định:

```php
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
```

<a name="returning-batches-from-routes"></a>
#### Returning Batches From Routes

Tất cả các instance `Illuminate\Bus\Batch` đều có thể serialize JSON, nghĩa là bạn có thể trả về chúng trực tiếp từ một trong các route của ứng dụng để lấy ra payload JSON chứa thông tin về batch, bao gồm cả tiến trình hoàn thành của nó. Điều này giúp việc hiển thị thêm thông tin về tiến trình hoàn thành của batch trong giao diện người dùng ứng dụng của bạn trở nên thuận tiện.

Để lấy ra một batch theo ID của nó, bạn có thể sử dụng phương thức `findBatch` của facade `Bus`:

```php
use Illuminate\Support\Facades\Bus;
use Illuminate\Support\Facades\Route;

Route::get('/batch/{batchId}', function (string $batchId) {
    return Bus::findBatch($batchId);
});
```

<a name="cancelling-batches"></a>
### Huỷ Batches

Thỉnh thoảng bạn có thể cần hủy việc thực thi của một batch nhất định. Điều này có thể được thực hiện bằng cách gọi phương thức `cancel` trên instance `Illuminate\Bus\Batch`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    if ($this->user->exceedsImportLimit()) {
        $this->batch()->cancel();

        return;
    }

    if ($this->batch()->cancelled()) {
        return;
    }
}
```

Như bạn có thể thấy trong ví dụ trước, các batch job thường phải xác định xem batch của nó đã bị hủy chưa trước khi tiếp tục chạy. Tuy nhiên, để thuận tiện, bạn có thể gán [middleware](#job-middleware) `SkipIfBatchCancelled` cho job. Như tên gọi của nó, middleware này sẽ hướng dẫn Laravel không xử lý job này nếu batch tương ứng của nó đã bị hủy:

```php
use Illuminate\Queue\Middleware\SkipIfBatchCancelled;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [new SkipIfBatchCancelled];
}
```

<a name="batch-failures"></a>
### Batch Failures

Khi một batch job thất bại, lệnh callback `catch` (nếu được chỉ định) sẽ được gọi. Lệnh callback này chỉ được gọi cho job đầu tiên bị thất bại trong batch.

<a name="allowing-failures"></a>
#### Allowing Failures

Khi một job trong một batch bị thất bại, Laravel sẽ tự động đánh dấu batch đó là "cancelled". Nếu muốn, bạn có thể vô hiệu hóa hành vi này để khi job thất bại thì không tự động đánh dấu batch là cancelled. Điều này có thể được thực hiện bằng cách gọi phương thức `allowFailures` trong khi gửi batch:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->allowFailures()->dispatch();
```

Bạn có thể tùy chọn cung cấp một closure cho phương thức `allowFailures`, closure này sẽ được chạy trên mỗi lần job thất bại:

```php
$batch = Bus::batch([
    // ...
])->allowFailures(function (Batch $batch, $exception) {
    // Handle individual job failures...
})->dispatch();
```

<a name="retrying-failed-batch-jobs"></a>
#### Retrying Failed Batch Jobs

Để thuận tiện, Laravel cung cấp lệnh Artisan `queue:retry-batch` cho phép bạn dễ dàng thử lại tất cả các job thất bại có trong một batch nhất định. Lệnh này sẽ chấp nhận UUID của batch mà có job thất bại cần được thử lại:

```shell
php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5
```

<a name="pruning-batches"></a>
### Xoá Batches

Nếu không xoá, bảng `job_batches` có thể tích lũy các record rất nhanh. Để giảm thiểu tình trạng này, bạn nên [schedule](/docs/{{version}}/scheduling) lệnh Artisan `queue:prune-batches` để chạy hàng ngày:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches')->daily();
```

Mặc định, tất cả các batch đã hoàn thành quá 24 giờ sẽ bị xoá. Bạn có thể sử dụng tùy chọn `hours` khi gọi command để xác định thời gian lưu giữ dữ liệu batch. Ví dụ: command sau sẽ xóa tất cả các batch đã hoàn thành hơn 48 giờ trước:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48')->daily();
```

Thỉnh thoảng, bảng `job_batches` của bạn có thể tích lũy các record batch cho các batch chưa được hoàn thành, chẳng hạn như các batch có job không thành công và job đó chưa bao giờ được thử lại thành công. Bạn có thể hướng dẫn lệnh `queue:prune-batches` để xoá các record batch chưa hoàn thành này bằng tùy chọn `unfinished`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48 --unfinished=72')->daily();
```

Tương tự như vậy, bảng `job_batches` của bạn cũng có thể tích lũy các record batch đã bị hủy một cách rất nhanh. Bạn có thể hướng dẫn lệnh `queue:prune-batches` để xoá bỏ một phần các batch record đã bị hủy bằng tùy chọn `cancelled`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48 --cancelled=72')->daily();
```

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
    'driver' => env('QUEUE_BATCHING_DRIVER', 'dynamodb'),
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

```php
use App\Models\Podcast;

$podcast = Podcast::find(1);

dispatch(function () use ($podcast) {
    $podcast->publish();
});
```

Để gán tên cho queued closure dùng cho các dashboard báo cáo queue, cũng như hiển thị bởi lệnh `queue:work`, bạn có thể sử dụng phương thức `name`:

```php
dispatch(function () {
    // ...
})->name('Publish Podcast');
```

Bằng cách sử dụng phương thức `catch`, bạn có thể cung cấp một closure sẽ được thực thi nếu queued closure không hoàn thành sau khi dùng hết tất cả [các lần thử lại đã được cấu hình](#max-job-attempts-and-timeout):

```php
use Throwable;

dispatch(function () use ($podcast) {
    $podcast->publish();
})->catch(function (Throwable $e) {
    // This job has failed...
});
```

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

Bạn có thể thêm flag `-v` khi gọi lệnh `queue:work` nếu bạn muốn ID của job đã xử lý, connection names, and queue names được thêm vào output của command này:

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

Daemon queue worker sẽ không "khởi động lại" framework trước khi xử lý mỗi job. Do đó, bạn nên giải phóng tất cả resources nặng sau khi hoàn thành xử lý job. Ví dụ, nếu bạn đang thực hiện tác vụ chỉnh sửa ảnh với [thư viện GD](https://www.php.net/manual/en/book.image.php), bạn nên giải phóng bộ nhớ với câu lệnh `imagedestroy` khi bạn hoàn thành việc xử lý ảnh đó.

<a name="queue-priorities"></a>
### Queue ưu tiên

Thỉnh thoảng bạn có thể muốn ưu tiên xử lý một queue. Ví dụ, trong file cấu hình `config/queue.php`, bạn có thể set mặc định `queue` cho connection `redis` là `low`. Tuy nhiên, đôi khi bạn có thể muốn tạo ra một job ở trên queue ưu tiên `high` như sau:

```php
dispatch((new Job)->onQueue('high'));
```

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

<a name="pausing-and-resuming-queue-workers"></a>
### Dừng và tiếp tục Queue Workers

Thỉnh thoảng bạn có thể cần tạm thời dừng một queue worker xử lý các job mới mà không cần dừng hoàn toàn worker đó. Ví dụ: bạn có thể muốn tạm dừng việc xử lý job trong quá trình bảo trì hệ thống. Laravel cung cấp các lệnh Artisan `queue:pause` và `queue:continue` để tạm dừng và tiếp tục các queue worker.

Để tạm dừng một queue cụ thể, hãy cung cấp tên kết nối queue và tên queue:

```shell
php artisan queue:pause database:default
```

Trong ví dụ này, `database` là tên kết nối queue và `default` là tên queue. Khi một queue bị tạm dừng, bất kỳ worker nào đang xử lý các job từ queue đó sẽ tiếp tục hoàn thành job hiện tại của chúng, nhưng sẽ không lấy thêm bất kỳ job mới nào cho đến khi queue được tiếp tục trở lại.

Để tiếp tục xử lý các job trên một queue đang bị tạm dừng, hãy sử dụng lệnh `queue:continue`:

```shell
php artisan queue:continue database:default
```

Sau khi tiếp tục một queue, các worker sẽ bắt đầu xử lý các job mới từ queue đó ngay lập tức. Lưu ý rằng việc tạm dừng một queue không làm dừng process của worker đó mà nó chỉ ngăn worker xử lý các job mới từ queue đã chỉ định.

<a name="worker-restart-and-pause-signals"></a>
#### Worker Restart and Pause Signals

Mặc định, các queue worker sẽ thăm dò cache driver để tìm ra các tín hiệu restart và pause sau mỗi lần xử lý job. Mặc dù việc thăm dò này là cần thiết để phản hồi lại các lệnh `queue:restart` và `queue:pause`, nhưng nó cũng gây ra một chút ảnh hưởng về hiệu năng.

Nếu bạn muốn tối ưu hóa hiệu năng và không cần đến các tính năng can thiệp này, bạn có thể tắt việc thăm dò global bằng cách gọi phương thức `withoutInterruptionPolling` trên facade `Queue`. Việc này thường được thực hiện trong phương thức `boot` của `AppServiceProvider` của bạn:

```php
use Illuminate\Support\Facades\Queue;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Queue::withoutInterruptionPolling();
}
```

Ngoài ra, bạn có thể tắt việc thăm dò restart hoặc pause riêng lẻ bằng cách thiết lập các thuộc tính static `$restartable` hoặc `$pausable` trên class `Illuminate\Queue\Worker`:

```php
use Illuminate\Queue\Worker;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Worker::$restartable = false;
    Worker::$pausable = false;
}
```

> [!WARNING]
> Khi tính năng thăm dò bị tắt, các worker sẽ không phản hồi các lệnh `queue:restart` hoặc lệnh `queue:pause` (tùy thuộc vào tính năng nào đã bị tắt).

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
> Nếu bạn không muốn cấu hình và quản lý Supervisor, hãy xem xét việc sử dụng [Laravel Cloud](https://cloud.laravel.com), nơi cung cấp một nền tảng được quản lý hoàn toàn để chạy các queue worker của Laravel.

<a name="configuring-supervisor"></a>
#### Configuring Supervisor

Các file cấu hình của Supervisor thường được lưu trong thư mục `/etc/supervisor/conf.d`. Trong thư mục này, bạn có thể tạo nhiều file cấu hình để hướng dẫn supervisor cách mà các process của bạn sẽ được giám sát. Ví dụ: hãy tạo một file `laravel-worker.conf` để bắt đầu và giám sát các process `queue:work`:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work --sleep=3 --tries=3 --max-time=3600
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

Một migration để tạo bảng `failed_jobs` sẽ có sẵn trong các ứng dụng Laravel mới. Tuy nhiên, nếu ứng dụng của bạn không chứa file migration cho bảng này, bạn có thể sử dụng lệnh `make:queue-failed-table` để tạo file migration đó:

```shell
php artisan make:queue-failed-table

php artisan migrate
```

Khi chạy một process [queue worker](#running-the-queue-worker), bạn có thể khai báo số lần chạy tối đa mà một job được chạy bằng cách sử dụng switch `--tries` trên lệnh `queue:work`. Nếu bạn không khai báo giá trị tùy chọn `--tries`, thì các job sẽ chỉ được chạy một lần duy nhất hoặc nhiều lần theo quy định của thuộc tính `Tries` trong class job:

```shell
php artisan queue:work redis --tries=3
```

Dùng tuỳ chọn `--backoff`, bạn có thể chỉ định cho Laravel biết sẽ đợi bao nhiêu giây trước khi thử lại một job bị gặp ngoại lệ. Mặc định, một job ngay lập tức được đưa trở lại queue để có thể thử lại:

```shell
php artisan queue:work redis --tries=3 --backoff=3
```

Nếu bạn muốn cấu hình Laravel sẽ đợi bao nhiêu giây trước khi thử lại job khi bị gặp ngoại lệ, bạn có thể dùng thuộc tính `Backoff` trong class job của bạn:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\Backoff;

#[Backoff(3)]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

Nếu bạn yêu cầu các logic phức tạp hơn để xác định thời gian thử lại của job, bạn có thể định nghĩa một phương thức `backoff` trên class job của bạn:

```php
/**
* Calculate the number of seconds to wait before retrying the job.
*/
public function backoff(): int
{
    return 3;
}
```

Bạn có thể dễ dàng cấu hình thời gian thử lại "theo cấp số nhân" bằng cách định nghĩa một mảng các giá trị thời gian thử lại. Trong ví dụ này, độ trễ thử lại sẽ là 1 giây cho lần thử đầu tiên, và 5 giây cho lần thử lại thứ hai, 10 giây cho lần thử lại thứ ba, và 10 giây cho mỗi lần thử lại tiếp theo nếu còn nhiều lần thử tiếp theo hơn:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\Backoff;

#[Backoff([1, 5, 10])]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

<a name="cleaning-up-after-failed-jobs"></a>
### Dọn dẹp sau khi Job failed

Khi một job bị thất bại, bạn có thể muốn gửi thông báo cho người dùng của bạn hoặc revert lại mọi hành động đã được job đó hoàn thành. Để thực hiện điều này, bạn có thể định nghĩa một phương thức `failed` trên class job của bạn. Instance `Throwable` khiến cho job thất bại và sẽ được chuyển sang phương thức `failed`:

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Throwable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

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
```

> [!WARNING]
> Một instance mới của job sẽ được khởi tạo trước khi gọi phương thức `failed`; do đó, mọi thay đổi thuộc tính class có trong phương thức `handle` sẽ bị mất.

Một job thất bại không nhất thiết phải là một job gặp ngoại lệ chưa được xử lý. Một job cũng có thể được coi là thất bại khi nó đã dùng hết tất cả các lần thử cho phép. Những lần thử này có thể bị mất theo nhiều cách khác nhau:

<div class="content-list" markdown="1">

- Job bị timeout.
- Job gặp một ngoại lệ không được xử lý trong quá trình thực thi.
- Job bị release trở lại queue theo cách thủ công hoặc bởi một middleware.

</div>

Nếu lần thử cuối cùng mà thất bại do một ngoại lệ được đưa ra trong quá trình thực thi job, ngoại lệ đó sẽ được truyền vào phương thức `failed` của job. Tuy nhiên, nếu job thất bại vì đã đạt đến số lần thử tối đa cho phép, `$exception` sẽ là một instance của `Illuminate\Queue\MaxAttemptsExceededException`. Tương tự, nếu job thất bại do vượt quá thời gian timeout, thì `$exception` sẽ là một instance của `Illuminate\Queue\TimeoutExceededException`.

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

Lệnh `queue:flush` sẽ xoá tất cả các record job bị fail từ queue của bạn, bất kể job đó đã fail từ khi nào. Bạn có thể sử dụng tùy chọn `--hours` để chỉ xóa những job đã bị fail từ một số giờ trước đó hoặc sớm hơn:

```shell
php artisan queue:flush --hours=48
```

<a name="ignoring-missing-models"></a>
### Ignoring Missing Models

Khi tích hợp một model Eloquent vào một job, model đó sẽ tự động được serialize trước khi được đưa vào queue và được lấy lại từ cơ sở dữ liệu khi job được xử lý. Tuy nhiên, nếu model đã bị xóa trong khi job đang chờ worker xử lý thì job của bạn có thể thất bại với lỗi `ModelNotFoundException`.

Để thuận tiện, bạn có thể chọn tự động xóa các job mà có model bị thiếu bằng cách sử dụng thuộc tính `DeleteWhenMissingModels` trên job class của bạn. Khi thuộc tính này xuất hiện, Laravel sẽ lặng lẽ xoá job mà không đưa ra bất kỳ ngoại lệ nào:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\DeleteWhenMissingModels;

#[DeleteWhenMissingModels]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

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

```php
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
```

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

Schedule cho lệnh này là không đủ để kích hoạt cảnh báo cho bạn về tình trạng quá tải của queue. Khi lệnh gặp một queue có số lượng công việc vượt quá ngưỡng của bạn, một event `Illuminate\Queue\Events\QueueBusy` sẽ được gửi đi. Bạn có thể lắng nghe event này trong `AppServiceProvider` của ứng dụng để gửi thông báo cho bạn hoặc nhóm phát triển của bạn:

```php
use App\Notifications\QueueHasLongWaitTime;
use Illuminate\Queue\Events\QueueBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (QueueBusy $event) {
        Notification::route('mail', 'dev@example.com')
            ->notify(new QueueHasLongWaitTime(
                $event->connectionName,
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

```php tab=Pest
<?php

use App\Jobs\AnotherJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;

test('orders can be shipped', function () {
    Queue::fake();

    // Perform order shipping...

    // Assert that no jobs were pushed...
    Queue::assertNothingPushed();

    // Assert a job was pushed to a given queue...
    Queue::assertPushedOn('queue-name', ShipOrder::class);

    // Assert a job was pushed
    Queue::assertPushed(ShipOrder::class);

    // Assert a job was pushed twice...
    Queue::assertPushedTimes(ShipOrder::class, 2);

    // Assert a job was not pushed...
    Queue::assertNotPushed(AnotherJob::class);

    // Assert that a closure was pushed to the queue...
    Queue::assertClosurePushed();

    // Assert that a closure was not pushed...
    Queue::assertClosureNotPushed();

    // Assert the total number of jobs that were pushed...
    Queue::assertCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Jobs\AnotherJob;
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

        // Assert a job was pushed
        Queue::assertPushed(ShipOrder::class);

        // Assert a job was pushed twice...
        Queue::assertPushedTimes(ShipOrder::class, 2);

        // Assert a job was not pushed...
        Queue::assertNotPushed(AnotherJob::class);

        // Assert that a closure was pushed to the queue...
        Queue::assertClosurePushed();

        // Assert that a closure was not pushed...
        Queue::assertClosureNotPushed();

        // Assert the total number of jobs that were pushed...
        Queue::assertCount(3);
    }
}
```

Bạn có thể truyền một closure cho các phương thức `assertPushed`, `assertNotPushed`, `assertClosurePushed`, hoặc `assertClosureNotPushed` để kiểm tra một job đã được đẩy vào queue và pass qua được "truth test" đã cho. Nếu có ít nhất một job đã được đẩy vào và pass qua truth test đã cho thì kiểm tra sẽ thành công:

```php
use Illuminate\Queue\CallQueuedClosure;

Queue::assertPushed(function (ShipOrder $job) use ($order) {
    return $job->order->id === $order->id;
});

Queue::assertClosurePushed(function (CallQueuedClosure $job) {
    return $job->name === 'validate-order';
});
```

<a name="faking-a-subset-of-jobs"></a>
### Fake một tập hợp Jobs

Nếu bạn chỉ cần fake các job cụ thể trong khi cho phép các job khác được chạy bình thường, bạn có thể truyền tên class của các job cần fake vào phương thức `fake`:

```php tab=Pest
test('orders can be shipped', function () {
    Queue::fake([
        ShipOrder::class,
    ]);

    // Perform order shipping...

    // Assert a job was pushed twice...
    Queue::assertPushedTimes(ShipOrder::class, 2);
});
```

```php tab=PHPUnit
public function test_orders_can_be_shipped(): void
{
    Queue::fake([
        ShipOrder::class,
    ]);

    // Perform order shipping...

    // Assert a job was pushed twice...
    Queue::assertPushedTimes(ShipOrder::class, 2);
}
```

Bạn có thể fake tất cả các job ngoại trừ một tập hợp các job được chỉ định bằng phương thức `except`:

```php
Queue::fake()->except([
    ShipOrder::class,
]);
```

<a name="testing-job-chains"></a>
### Testing Job Chains

Để kiểm tra một chuỗi job, bạn sẽ cần sử dụng khả năng fake của facade `Bus`. Phương thức `assertChained` của facade `Bus` có thể được sử dụng để kiểm tra một [chuỗi job](/docs/{{version}}/queues#job-chaining) đã được gửi hay chưa. Phương thức `assertChained` sẽ chấp nhận một mảng các job trong một chuỗi làm tham số đầu tiên của nó:

```php
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
```

Như bạn có thể thấy trong ví dụ trên, mảng job trong một chuỗi có thể là một mảng gồm các tên class của job. Tuy nhiên, bạn cũng có thể cung cấp một mảng các instance job thực tế. Khi làm như vậy, Laravel sẽ đảm bảo là các instance job đó sẽ thuộc cùng một class và cùng giá trị thuộc tính khi được gửi đi bởi ứng dụng của bạn:

```php
Bus::assertChained([
    new ShipOrder,
    new RecordShipment,
    new UpdateInventory,
]);
```

Bạn có thể sử dụng phương thức `assertDispatchedWithoutChain` để kiểm tra một job đã được đẩy đi mà không nằm trong bất kỳ chuỗi job nào:

```php
Bus::assertDispatchedWithoutChain(ShipOrder::class);
```

<a name="testing-chain-modifications"></a>
#### Testing Chain Modifications

Nếu một job ở trong chuỗi có [thêm một job vào đầu hoặc cuối của chuỗi hiện tại](#adding-jobs-to-the-chain), thì bạn có thể sử dụng phương thức `assertHasChain` của job đó để yêu cầu job này sẽ có chuỗi job còn lại như bạn mong muốn:

```php
$job = new ProcessPodcast;

$job->handle();

$job->assertHasChain([
    new TranscribePodcast,
    new OptimizePodcast,
    new ReleasePodcast,
]);
```

Phương thức `assertDoesntHaveChain` có thể được sử dụng để yêu cầu chuỗi job còn lại là trống:

```php
$job->assertDoesntHaveChain();
```

<a name="testing-chained-batches"></a>
#### Testing Chained Batches

Nếu chuỗi job của bạn [có chứa một batch job](#chains-and-batches), bạn có thể kiểm tra batch job được nối đó phù hợp với kỳ vọng của bạn bằng cách chèn thêm một định nghĩa `Bus::chainedBatch` vào kiểm tra chuỗi job của bạn:

```php
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
```

<a name="testing-job-batches"></a>
### Testing Job Batches

Phương thức `assertBatched` của facade `Bus` có thể được sử dụng để kiểm tra một [batch job](/docs/{{version}}/queues#job-batching) đã được gửi hay chưa. Closure được cung cấp cho phương thức `assertBatched` sẽ nhận vào một instance của `Illuminate\Bus\PendingBatch`, có thể được sử dụng để kiểm tra các job có trong batch:

```php
use Illuminate\Bus\PendingBatch;
use Illuminate\Support\Facades\Bus;

Bus::fake();

// ...

Bus::assertBatched(function (PendingBatch $batch) {
    return $batch->name == 'Import CSV' &&
           $batch->jobs->count() === 10;
});
```

Phương thức `hasJobs` có thể được sử dụng trên pending batch để xác minh rằng batch đó có chứa các job mà bạn mong muốn hay không. Phương thức này chấp nhận một mảng chứa các instance job, tên class hoặc closure:

```php
Bus::assertBatched(function (PendingBatch $batch) {
    return $batch->hasJobs([
        new ProcessCsvRow(row: 1),
        new ProcessCsvRow(row: 2),
        new ProcessCsvRow(row: 3),
    ]);
});
```

Khi sử dụng closure, closure đó sẽ nhận vào instance của job. Loại job mà bạn mong muốn sẽ được suy luận từ khai báo của closure:

```php
Bus::assertBatched(function (PendingBatch $batch) {
    return $batch->hasJobs([
        fn (ProcessCsvRow $job) => $job->row === 1,
        fn (ProcessCsvRow $job) => $job->row === 2,
        fn (ProcessCsvRow $job) => $job->row === 3,
    ]);
});
```

Bạn có thể sử dụng phương thức `assertBatchCount` để kiểm tra số lượng batch đã được gửi đi:

```php
Bus::assertBatchCount(3);
```

Bạn có thể sử dụng `assertNothingBatched` để kiểm tra không có batch nào được gửi đi:

```php
Bus::assertNothingBatched();
```

<a name="testing-job-batch-interaction"></a>
#### Testing Job / Batch Interaction

Ngoài ra, đôi khi bạn có thể cần kiểm tra tương tác của một job với batch của nó. Ví dụ, bạn có thể cần kiểm tra xem một job có hủy xử lý tiếp theo của batch của nó hay không. Để thực hiện việc này, bạn cần chỉ định một batch fake cho job đó thông qua phương thức `withFakeBatch`. Phương thức `withFakeBatch` này sẽ trả về một mảng chứa instance job và batch fake:

```php
[$job, $batch] = (new ShipOrder)->withFakeBatch();

$job->handle();

$this->assertTrue($batch->cancelled());
$this->assertEmpty($batch->added);
```

<a name="testing-job-queue-interactions"></a>
### Testing Job / Queue Interactions

Thỉnh thoảng, bạn có thể cần kiểm tra một queued job có [tự giải phóng nó trở lại queue hay không](#manually-releasing-a-job). Hoặc, bạn có thể cần kiểm tra job đó có tự xóa nó hay không. Bạn có thể kiểm tra các tương tác queue này bằng cách khởi tạo job và gọi phương thức `withFakeQueueInteractions`.

Sau khi các tương tác queue của job đã được fake, bạn có thể gọi phương thức `handle` trên job. Sau khi gọi job, bạn có thể sử dụng các phương thức assertion khác nhau để kiểm tra các tương tác queue của job:

```php
use App\Exceptions\CorruptedAudioException;
use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
$job->assertDeleted();
$job->assertNotDeleted();
$job->assertFailed();
$job->assertFailedWith(CorruptedAudioException::class);
$job->assertNotFailed();
```

<a name="job-events"></a>
## Job Event

Sử dụng các phương thức `before` và `after` trong [facade](/docs/{{version}}/facades) `Queue`, bạn có thể khai báo các callback sẽ được thực hiện trước hoặc sau khi một queued job được xử lý. Các callback này là một cách tuyệt vời để thực hiện thêm logging hoặc ghi thông kê cho bảng điều khiển. Thông thường, bạn nên gọi các phương thức này từ phương thức `boot` của [service provider](/docs/{{version}}/providers). Ví dụ: chúng ta có thể sử dụng `AppServiceProvider` được đi kèm với Laravel:

```php
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
```

Sử dụng phương thức `looping` trong [facade](/docs/{{version}}/facades) `Queue`, bạn có thể khai báo các callback sẽ được thực thi trước khi worker lấy một job từ queue. Ví dụ: bạn có thể đăng ký một closure để rollback bất kỳ các transaction nào đang bị làm dở bởi một job đã bị thất bại trước đó:

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Queue;

Queue::looping(function () {
    while (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```
