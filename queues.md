# Queues

- [Giới thiệu](#introduction)
    - [Connection và Queue](#connections-vs-queues)
    - [Yêu cầu Driver](#driver-prerequisites)
- [Tạo Job](#creating-jobs)
    - [Tạo class Job](#generating-job-classes)
    - [Cấu trúc class](#class-structure)
- [Dispatching Job](#dispatching-jobs)
    - [Delayed Dispatching](#delayed-dispatching)
    - [Kết hợp Job](#job-chaining)
    - [Tuỳ biến Queue và Connection](#customizing-the-queue-and-connection)
    - [Khai báo số lần chạy Job tối đa / giá trị timeout](#max-job-attempts-and-timeout)
    - [Giới hạn tỷ lệ chạy](#rate-limiting)
    - [Xử lý Error](#error-handling)
- [Chạy Queue Worker](#running-the-queue-worker)
    - [Queue ưu tiên](#queue-priorities)
    - [Queue Worker và Deployment](#queue-workers-and-deployment)
    - [Job hết hạn và timeout](#job-expirations-and-timeouts)
- [Cấu hình Supervisor](#supervisor-configuration)
- [Xử lý Job failed](#dealing-with-failed-jobs)
    - [Dọn dẹp sau khi Job failed](#cleaning-up-after-failed-jobs)
    - [Event Job failed](#failed-job-events)
    - [Chạy lại Job failed](#retrying-failed-jobs)
- [Job Event](#job-events)

<a name="introduction"></a>
## Giới thiệu

> {tip} Laravel hiện cung cấp Horizon là một hệ thống cấu hình và điều khiển cho các queue mà được tạo bởi Redis của bạn. Hãy xem toàn bộ [tài liệu Horizon](/docs/{{version}}/horizon) để biết thêm thông tin chi tiết.

Queue của Laravel cung cấp một API hợp nhất trên nhiều loại queue backend khác nhau, chẳng hạn như Beanstalk, Amazon SQS, Redis hoặc thậm chí là một database. Queue cho phép bạn trì hoãn việc xử lý một tác vụ tốn nhiều thời gian, chẳng hạn như việc gửi email sau một thời gian nhất định. Trì hoãn các tác vụ tiêu tốn thời gian này sẽ tăng tốc các request web đến application của bạn.

File cấu hình queue được lưu trữ trong `config/queue.php`. Trong file này, bạn sẽ tìm thấy các cấu hình connection cho từng loại driver queue có trong framework, bao gồm database, [Beanstalkd](https://kr.github.io/beanstalkd/), [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), và một driver chạy đồng bộ job (để sử dụng dưới local). Driver queue `null` cũng đã được khai báo để loại bỏ các job đã được queue.

<a name="connections-vs-queues"></a>
### Connection và Queue

Trước khi bắt đầu với Laravel queue, điều quan trọng là phải hiểu sự khác biệt giữa "connections" và "queues". Trong file cấu hình `config/queue.php` của bạn, có một tùy chọn cấu hình là `connections`. Tùy chọn này sẽ định nghĩa một connection đến một backend service như Amazon SQS, Beanstalk hoặc Redis. Tuy nhiên, bất kỳ connection nào cũng có thể có nhiều "queue", và nó có thể được coi là một ngăn xếp hoặc một mảng các job khác nhau.

Lưu ý rằng mỗi ví dụ cấu hình connection trong file cấu hình `queue` có chứa một thuộc tính `queue`. Đây là queue mặc định mà các job sẽ được gửi tới mỗi khi chúng được gửi đến một connection. Nói cách khác, nếu bạn gửi một job mà không khai báo rõ queue nào sẽ được dùng, thì job đó sẽ được lưu vào queue mà đã được định nghĩa trong thuộc tính `queue` của cấu hình connection:

    // This job is sent to the default queue...
    Job::dispatch();

    // This job is sent to the "emails" queue...
    Job::dispatch()->onQueue('emails');

Một số application có thể không cần phải tạo nhiều job trong nhiều queue, thay vào đó một queue có thể là phù hợp hơn. Tuy nhiên, việc tạo các job lên nhiều queue cũng có thể đặc biệt hữu ích cho các application mà muốn ưu tiên hoặc là phân chia cách xử lý cho từng job, vì Laravel queue worker cho phép bạn khai báo các queue sẽ được xử lý theo mức độ ưu tiên. Ví dụ: nếu bạn tạo một job lên queue `high`, thì bạn có thể chạy một worker có mức độ ưu tiên xử lý cao hơn:

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### Yêu cầu Driver

#### Database

Để sử dụng driver `database` queue, bạn sẽ cần một bảng cơ sở dữ liệu để lưu các job. Để tạo một migration tạo bảng, hãy chạy lệnh Artisan `queue:table`. Khi migration đã được tạo, bạn có thể migrate cơ sở dữ liệu của bạn bằng lệnh `migrate`:

    php artisan queue:table

    php artisan migrate

#### Redis

Để sử dụng driver `redis` queue, bạn nên cấu hình connection tới Redis database trong file cấu hình `config/database.php` của bạn.

Nếu connection Redis của bạn sử dụng một Cluster Redis, thì tên queue của bạn phải chứa một [key hash tag](https://redis.io/topics/cluster-spec#keys-hash-tags). Điều này là bắt buộc để đảm bảo rằng tất cả các key Redis cho queue sẽ được set vào cùng một vị trí hash:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

#### Other Driver Prerequisites

Các library sau sẽ cần thiết cho driver queue cũng sẽ được liệt kê:

<div class="content-list" markdown="1">
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`
</div>

<a name="creating-jobs"></a>
## Tạo Job

<a name="generating-job-classes"></a>
### Tạo class Job

Mặc định, tất cả các job cho application của bạn được lưu trong thư mục `app/Jobs`. Nếu thư mục `app/Jobs` chưa tồn tại, thì nó có thể được tạo ra khi bạn chạy lệnh Artisan `make:job`. Bạn có thể tạo một queue job mới bằng cách sử dụng Artisan CLI:

    php artisan make:job ProcessPodcast

Class được tạo ra sẽ implement interface `Illuminate\Contracts\Queue\ShouldQueue`, và cho Laravel biết là job này sẽ được đưa vào queue để chạy không đồng bộ.

<a name="class-structure"></a>
### Cấu trúc class

Các class của job rất đơn giản, thông thường chỉ chứa một phương thức `handle` được gọi khi job được xử lý bởi queue. Để bắt đầu, chúng ta hãy xem một class của một job ví dụ. Trong ví dụ này, chúng ta sẽ chạy là chúng ta quản lý một service xuất bản podcast và cần xử lý các file podcast đã được tải lên trước khi được xuất bản:

    <?php

    namespace App\Jobs;

    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }
    }

Trong ví dụ trên, hãy lưu ý rằng chúng ta có thể truyền một [Eloquent model](/docs/{{version}}/eloquent) trực tiếp vào hàm khởi tạo của queued job. Do trait `SerializesModels` này đang được job sử dụng, nên các model Eloquent sẽ được serialize và unserialize ngược lại khi job được xử lý. Nếu queued job của bạn chấp nhận một model Eloquent trong hàm khởi tạo, thì chỉ có mã định danh cho model sẽ được serialize trên queue. Và khi job đó được xử lý, thì hệ thống queue sẽ tự động lấy ra lại full instance của model đó từ cơ sở dữ liệu. Tất cả đều là hoàn toàn an toàn đối với application của bạn và ngăn chặn các vấn đề có thể phát sinh từ việc serialize full instance của model Eloquent.

Phương thức `handle` được gọi khi job được xử lý bởi queue. Lưu ý rằng chúng ta có thể khai báo các phụ thuộc vào phương thức `handle` của job. Laravel [service container](/docs/{{version}}/container) sẽ tự động inject các phụ thuộc này.

> {note} Dữ liệu nhị phân, chẳng hạn như một nội dung ảnh thô, phải được truyền qua hàm `base64_encode` trước khi được truyền đến một queued job. Nếu không làm điều này, thì job đó có thể serialize thành chuỗi JSON không đúng khi được đặt lên queue.

<a name="dispatching-jobs"></a>
## Dispatching Job

Khi bạn đã viết xong các class job của bạn, bạn có thể dispatch nó bằng cách sử dụng phương thức `dispatch` trên chính job đó. Các tham số được truyền vào cho phương thức `dispatch` sẽ được truyền lại vào hàm khởi tạo của job:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast);
        }
    }

<a name="delayed-dispatching"></a>
### Delayed Dispatching

Nếu bạn muốn delay việc thực hiện một queued job, bạn có thể sử dụng phương thức `delay` khi đang dispatch một job. Ví dụ: hãy khai báo một job không nên được xử lý cho đến 10 phút sau khi job đó được dispatch:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));
        }
    }

> {note} service SQS queue của Amazon có thời gian delay đa là 15 phút.

<a name="job-chaining"></a>
### Kết hợp Job

Kết hợp job cho phép bạn khai báo một danh sách các queued job sẽ được chạy theo một trình tự nhất định. Nếu một job trong danh sách bị thất bại, thì các job còn lại cũng sẽ không được chạy. Để thực hiện một danh sách queued job, bạn có thể sử dụng phương thức `withChain` trên bất kỳ dispatchable job nào của bạn:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->dispatch();

<a name="customizing-the-queue-and-connection"></a>
### Tuỳ biến Queue và Connection

#### Dispatching đến một Queue cụ thể

Bằng cách tạo các job đến các queue khác nhau, bạn có thể "phân loại" các queued job của bạn và thậm chí là ưu tiên bao nhiêu worker sẽ được gán cho mỗi queue. Hãy nhớ rằng, điều này không đẩy job đến các queue "connection" khác mà có ở trong file định nghĩa cấu hình queue của bạn, mà chỉ đẩy đến các queue có trong một connection cụ thể. Để khai báo queue, sử dụng phương thức `onQueue` khi gửi job:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onQueue('processing');
        }
    }

#### Dispatching To A Particular Connection

Nếu bạn đang làm việc với nhiều queue connection, bạn có thể khai báo connection nào sẽ dùng cho một job. Để khai báo connection, sử dụng phương thức `onConnection` khi gửi job:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onConnection('sqs');
        }
    }

Tất nhiên, bạn có thể kết hợp các phương thức `onConnection` và `onQueue` để khai báo connection và queue cho một job:

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');

<a name="max-job-attempts-and-timeout"></a>
### Khai báo số lần chạy Job tối đa / giá trị timeout

#### Max Attempts

Một cách tiếp cận để khai báo số lần tối đa mà một job có thể được chạy là thông qua switch `--tries` trên lệnh Artisan:

    php artisan queue:work --tries=3

Tuy nhiên, bạn có thể thực hiện một cách tiếp cận chi tiết hơn bằng cách định nghĩa số lần chạy tối đa trên chính class của job. Nếu số lần chạy tối đa được chỉ định trong job, thì nó sẽ ưu tiên giá trị này hơn là giá trị được cung cấp trên dòng lệnh:

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

<a name="time-based-attempts"></a>
#### Time Based Attempts

Thay thế cho việc định nghĩa số lần một job có thể được chạy trước khi nó thất bại, bạn có thể định nghĩa thời gian mà job đó sẽ hết hạn. Điều này cho phép một job có thể được chạy thoải mái trong một khoảng thời gian nhất định. Để định nghĩa thời gian mà một job sẽ hết hạn, hãy thêm phương thức `retryUntil` vào class job của bạn:

    /**
     * Determine the time at which the job should timeout.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} Bạn cũng có thể định nghĩa phương thức `retryUntil` trên các queued event listener của bạn.

#### Timeout

> {note} Tính năng `timeout` được tối ưu hóa cho PHP 7.1+ và PHP extension `pcntl`.

Tương tự, thời gian hết hạn của một job có thể được khai báo bằng cách sử dụng switch `--timeout` trên lệnh Artisan:

    php artisan queue:work --timeout=30

Tuy nhiên, bạn cũng có thể định nghĩa thời gian hết hạn của một job trên chính class của job đó. Nếu thời gian hết hạn được khai báo trong job, nó sẽ được ưu tiên hơn bất kỳ thời gian hết hạn nào được khai báo trên dòng lệnh:

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

<a name="rate-limiting"></a>
### Giới hạn tỷ lệ chạy

> {note} Tính năng này yêu cầu application của bạn cài đặt một [Redis server](/docs/{{version}}/redis).

Nếu application của bạn có tương tác với Redis, bạn có thể điều tiết các queued job chạy theo thời gian hoặc đồng thời. Tính năng này có thể hỗ trợ khi các queued job của bạn mà đang tương tác với các API mà có giới hạn về tỷ lệ chạy. Ví dụ, bằng cách sử dụng phương thức `throttle`, bạn có thể điều tiết một loại job sẽ chỉ được chạy 10 lần trong vòng 60 giây. Nếu không thể lấy được lock, bạn nên giải phóng job đó trở lại queue để có thể chạy lại ở lần sau:

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

> {tip} Trong ví dụ trên, `key` có thể là bất kỳ chuỗi nào dùng để xác định một loại job mà bạn muốn giới hạn tỷ lệ chạy. Ví dụ, bạn có thể muốn khởi tạo một key dựa trên tên class của job đó và một ID của model Eloquent mà nó hoạt động.

Ngoài ra, bạn có thể khai báo số lượng worker tối đa mà có thể xử lý đồng thời một job. Điều này có thể hữu ích khi một queued job đang sửa một resource thì chỉ được sửa trên một job tại một thời điểm nhất định. Ví dụ, bằng cách sử dụng phương thức `funnel`, bạn có thể giới hạn các job thuộc loại đã cho chỉ được xử lý bởi một worker tại một thời điểm:

    Redis::funnel('key')->limit(1)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

> {tip} Khi sử dụng giới hạn tỷ lệ chạy, thì số lần cần để chạy job thành công có thể rất khó xác định. Do đó, sẽ hữu ích hơn khi kết hợp giới hạn tỷ lệ chạy với [số lần chạy dựa trên thời gian](#time-based-attempts).

<a name="error-handling"></a>
### Xử lý Error

Nếu một ngoại lệ được đưa ra trong khi job đang được xử lý, thì job đó sẽ tự động được giải phóng trở lại vào queue để có thể chạy lại lần sau. Job đó sẽ tiếp tục được chạy lại cho đến khi nó được chạy quá số lần tối đa cho phép của application của bạn. Số lần chạy tối đa được xác định bởi switch `--tries` được sử dụng trên lệnh Artisan `queue:work`. Ngoài ra, số lần chạy tối đa này cũng có thể được xác định trên chính class của job. Thông tin chi tiết về việc chạy queue worker [có thể được tìm thấy ở bên dưới](#running-the-queue-worker).

<a name="running-the-queue-worker"></a>
## Chạy Queue Worker

Laravel có chứa một queue worker sẽ xử lý các job mới khi chúng được tạo lên queue. Bạn có thể chạy worker đó bằng lệnh Artisan `queue:work`. Lưu ý rằng một khi lệnh `queue:work` đã được chạy, thì nó sẽ tiếp tục chạy cho đến khi nào nó được dừng bằng cách thủ công hoặc bạn đóng terminal của bạn:

    php artisan queue:work

> {tip} Để giữ cho process `queue:work` luôn hoạt động trong background, bạn nên sử dụng một trình giám sát process, chẳng hạn như [Supervisor](#supervisor-configuration) để đảm bảo rằng queue worker không bị dừng giữa chừng.

Hãy nhớ rằng, các queue worker là các process tồn tại lâu dài và lưu trữ trạng thái của application vào trong bộ nhớ. Do đó, chúng sẽ không nhận biết dược những thay đổi trong source code của bạn sau khi bạn đã chạy chúng. Vì vậy, trong khi quá trình deploy của bạn, hãy đảm bảo là [bạn đã khởi động lại queue worker của bạn](#queue-workers-and-deployment).

#### Processing A Single Job

Tùy chọn `--once` có thể được sử dụng để lệnh worker chỉ xử lý một job duy nhất từ queue:

    php artisan queue:work --once

#### Specifying The Connection & Queue

Bạn cũng có thể khai báo queue connection mà worker sẽ sử dụng. Tên connection được truyền vào lệnh `work` phải tương ứng với một trong các connection đã được khai báo ở trong file cấu hình `config/queue.php` của bạn:

    php artisan queue:work redis

Bạn cũng có thể tùy chỉnh queue worker của bạn nhiều hơn nữa bằng cách chỉ xử lý các queue cụ thể cho một connection nhất định. Ví dụ: nếu tất cả các email của bạn được xử lý trong queue `emails` trên một connection là `redis`, thì bạn có thể gọi lệnh sau để chạy một worker chỉ xử lý cho riêng queue đó:

    php artisan queue:work redis --queue=emails

#### Resource Considerations

Daemon queue worker sẽ không "khởi động lại" framework trước khi xử lý mỗi job. Do đó, bạn nên giải phóng tất cả resources nặng sau khi hoàn thành xử lý job. Ví dụ, nếu bạn đang thực hiện tác vụ chỉnh sửa ảnh với thư viện GD, bạn nên giải phóng bộ nhớ với câu lệnh `imagedestroy` khi bạn hoàn thành.

<a name="queue-priorities"></a>
### Queue ưu tiên

Thỉnh thoảng bạn có thể muốn ưu tiên xử lý một queue. Ví dụ, trong file `config/queue.php`, bạn có thể set mặc định `queue` cho connection `redis` là `low`. Tuy nhiên, đôi khi bạn có thể muốn tạo ra một job ở trên queue ưu tiên `high` như sau:

    dispatch((new Job)->onQueue('high'));

Để chạy một worker cho việc xử lý các queue job `high` trước rồi sau đó mới xử lý đến các job trong queue `low`, thì bạn có thể truyền vào một danh sách tên queue sẽ phân cách nhau bằng dấu phẩy cho lệnh `work`:

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>
### Queue Worker và Deployment

Vì queue worker là các process tồn tại lâu dài, nên nếu không được khởi động lại, thì chúng sẽ không biết được các thay đổi trên code của bạn. Vì vậy, cách đơn giản nhất để deploy một application sử dụng queue worker là khởi động lại worker trong quá trình deploy của bạn. Bạn có thể khởi động lại tất cả các worker bằng cách chạy lệnh `queue:restart`:

    php artisan queue:restart

Để không làm job đó bị mất, lệnh này sẽ làm tất cả các queue workers "die" sau khi chúng xử lý xong các job hiện tại. Vì các queue worker sẽ bị die khi lệnh `queue:restart` được chạy, vì thế bạn nên chạy một process quản lý, chẳng hạn như [Supervisor](#supervisor-configuration) để tự động khởi động lại các queue worker đó.

> {tip} Queue sẽ sử dụng [cache](/docs/{{version}}/cache) để lưu trữ tín hiệu khởi động lại, vì vậy bạn nên kiểm tra driver cache đã được cấu hình cho application của bạn đúng chưa trước khi sử dụng tính năng này.

<a name="job-expirations-and-timeouts"></a>
### Job hết hạn và timeout

#### Job Expiration

Trong file cấu hình `config/queue.php` của bạn, mỗi queue connection sẽ định nghĩa một tùy chọn `retry_after`. Tùy chọn này sẽ khai báo cho queue connection biết sẽ phải đợi bao nhiêu giây trước khi chạy lại một job đang được xử lý. Ví dụ: nếu giá trị của `retry_after` được set là `90`, thì job đó sẽ được giải phóng trở lại vào queue nếu nó đã được xử lý quá 90 giây mà không bị xóa. Thông thường, bạn nên set giá trị `retry_after` là số giây tối đa mà một job của bạn có thể sẽ mất để hoàn thành tất cả xử lý.

> {note} Chỉ có queue connection của Amazon SQS sẽ không chứa giá trị `retry_after`. SQS sẽ retry một job dựa trên [Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) được quản lý trong AWS console.

#### Worker Timeouts

Lệnh Artisan `queue:work` có một tùy chọn là `--timeout`. Tùy chọn `--timeout` này sẽ khai báo process queue master của Laravel sẽ đợi bao lâu trước khi killing một queue worker đang xử lý bởi một job. Thỉnh thoảng, một queue process có thể bị "đơ" vì nhiều lý do, chẳng hạn như sử dụng một HTTP có thể call ra bên ngoài, nhưng lại không có respond trở lại. Tùy chọn `--timeout` sẽ loại bỏ các process bị đơ vượt quá giới hạn thời gian đã được khai báo:

    php artisan queue:work --timeout=60

Tùy chọn cấu hình `retry_after` và tùy chọn CLI `--timeout` tuy khác nhau, nhưng nếu phối hợp với nhau thì sẽ giúp bạn đảm bảo rằng các job sẽ không bị mất và các job chỉ được xử lý thành công trong một lần.

> {note} Giá trị `--timeout` phải luôn luôn có thời gian ngắn hoặc ít hơn vài giây so với giá trị cấu hình `retry_after`. Điều này sẽ đảm bảo là một worker sẽ xử lý một job nhất định luôn bị killed trước khi job đó được chạy lại. Nếu tùy chọn `--timeout` của bạn dài hơn giá trị cấu hình `retry_after`, thì job đó của bạn có thể bị xử lý hai lần.

#### Worker Sleep Duration

Khi job đã được tạo trong queue, worker sẽ tiếp tục xử lý các job mà không có thời gian nghỉ giữa chúng. Tuy nhiên, với tùy chọn `sleep` sẽ xác định xem worker sẽ "sleep" trong bao lâu nếu không có job mới. Trong khi sleep, worker sẽ không xử lý bất kỳ job mới nào - các job đó sẽ được xử lý sau khi thời gian worker đã nghỉ.

    php artisan queue:work --sleep=3

<a name="supervisor-configuration"></a>
## Cấu hình Supervisor

#### Installing Supervisor

Supervisor là trình giám sát process cho hệ điều hành Linux và sẽ tự động khởi động lại process `queue:work` của bạn nếu nó thất bại. Để cài đặt Supervisor trên Ubuntu, bạn có thể sử dụng lệnh sau:

    sudo apt-get install supervisor

> {tip} Nếu bạn không muốn tự cấu hình Supervisor, hãy xem xét việc sử dụng [Laravel Forge](https://forge.laravel.com), nó sẽ tự động cài đặt và cấu hình Supervisor cho các dự án Laravel của bạn.

#### Configuring Supervisor

Các file cấu hình của Supervisor thường được lưu trong thư mục `/etc/supervisor/conf.d`. Trong thư mục này, bạn có thể tạo nhiều file cấu hình để hướng dẫn supervisor cách mà các process của bạn sẽ được giám sát. Ví dụ: hãy tạo một file `laravel-worker.conf` để bắt đầu và giám sát process `queue:work`:

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

Trong ví dụ trên, lệnh `numprocs` sẽ hướng dẫn Supervisor chạy 8 process `queue:work` và giám sát tất cả chúng, tự động khởi động lại nếu chúng thất bại. Tất nhiên, bạn nên thay đổi phần `queue:work sqs` của lệnh `command` để phản ánh queue connection mà bạn mong muốn.

#### Starting Supervisor

Khi file cấu hình đã hoàn thành, bạn có thể cập nhật cấu hình Supervisor và chạy các process bằng các lệnh sau:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

Để biết thêm thông tin về Supervisor, hãy tham khảo [tài liệu Supervisor](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>
## Xử lý Job failed

Thỉnh thoảng, queued job của bạn sẽ gặp thất bại. Đừng lo lắng, mọi thứ không phải lúc nào cũng theo như kế hoạch! Laravel có chứa một cách để khai báo số lần tối đa một job được chạy lại. Sau khi một job vượt quá số lần chạy này, nó sẽ được thêm vào bảng cơ sở dữ liệu `failed_jobs`. Để tạo migration cho bảng `failed_jobs`, bạn có thể sử dụng lệnh `queue:failed-table`:

    php artisan queue:failed-table

    php artisan migrate

Sau khi chạy [queue worker](#running-the-queue-worker), bạn nên khai báo số lần chạy tối đa mà một job được chạy bằng cách sử dụng switch `--tries` trên lệnh `queue:work`. Nếu bạn không khai báo giá trị tùy chọn `--tries`, thì các job sẽ được chạy lại vô thời hạn:

    php artisan queue:work redis --tries=3

<a name="cleaning-up-after-failed-jobs"></a>
### Dọn dẹp sau khi Job failed

Bạn có thể định nghĩa một phương thức `failed` trực tiếp vào class job của bạn, cho phép bạn thực hiện việc dọn dẹp job khi xảy ra lỗi. Đây là một vị trí hoàn hảo để gửi các cảnh báo đến người dùng hoặc revert lại các hành động trước khi thực hiện job. `Exception` sẽ khiến job thất bại và sẽ được chuyển sang phương thức `failed`:

    <?php

    namespace App\Jobs;

    use Exception;
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }

        /**
         * The job failed to process.
         *
         * @param  Exception  $exception
         * @return void
         */
        public function failed(Exception $exception)
        {
            // Send user notification of failure, etc...
        }
    }

<a name="failed-job-events"></a>
### Event Job failed

Nếu bạn muốn đăng ký một event sẽ được gọi khi một job thất bại, bạn có thể sử dụng phương thức `Queue::failing`. Event này là một cách tuyệt vời để thông báo cho team của bạn qua email hoặc [HipChat](https://www.hipchat.com). Ví dụ: chúng ta có thể đính kèm một callback cho event này từ `AppServiceProvider` được chứa trong Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Queue\Events\JobFailed;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="retrying-failed-jobs"></a>
### Chạy lại Job failed

Để xem tất cả các job đã bị thất bại, bạn có thể sử dụng lệnh Artisan `queue:failed`, những job này sẽ được thêm vào trong bảng cơ sở dữ liệu `failed_jobs` của bạn:

    php artisan queue:failed

Lệnh `queue:failed` sẽ liệt kê các ID, connection, queue và thời gian bị thất bại của job. ID của job có thể được sử dụng để chạy lại những job đã bị thất bại. Chẳng hạn, để chạy lại một của job đã bị thất bại có ID là `5`, thì hãy chạy lệnh như sau:

    php artisan queue:retry 5

Để chạy lại tất cả các job bị thất bại, bạn hãy chạy lệnh `queue:retry` và truyền vào `all` làm ID:

    php artisan queue:retry all

Nếu bạn muốn xóa một job bị thất bại, bạn có thể sử dụng lệnh `queue:forget`:

    php artisan queue:forget 5

Để xóa tất cả các job bị thất bại, bạn có thể sử dụng lệnh `queue:flush`:

    php artisan queue:flush

<a name="job-events"></a>
## Job Event

Sử dụng các phương thức `before` và `after` trong [facade](/docs/{{version}}/facades) `Queue`, bạn có thể khai báo các callback sẽ được thực hiện trước hoặc sau khi một queued job được xử lý. Các callback này là một cách tuyệt vời để thực hiện thêm logging hoặc ghi thông kê cho bảng điều khiển. Thông thường, bạn nên gọi các phương thức này từ [service provider](/docs/{{version}}/providers). Ví dụ: chúng ta có thể sử dụng `AppServiceProvider` được đi kèm với Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
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

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Sử dụng phương thức `looping` trong [facade](/docs/{{version}}/facades) `Queue`, bạn có thể khai báo các callback sẽ được thực thi trước khi worker lấy một job từ queue. Ví dụ: bạn có thể đăng ký một Closure để rollback bất kỳ các transaction nào đang bị làm dở bởi một job đã bị thất bại trước đó:

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
