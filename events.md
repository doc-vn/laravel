# Events

- [Giới thiệu](#introduction)
- [Tạo Event và Listener](#generating-events-and-listeners)
- [Đăng ký Event và Listener](#registering-events-and-listeners)
    - [Event Discovery](#event-discovery)
    - [Đăng ký Event thủ công](#manually-registering-events)
    - [Closure Listeners](#closure-listeners)
- [Khai báo Event](#defining-events)
- [Khai báo Listener](#defining-listeners)
- [Queued Event Listener](#queued-event-listeners)
    - [Tương tác thủ công với queue](#manually-interacting-with-the-queue)
    - [Queued Event Listeners và Database Transactions](#queued-event-listeners-and-database-transactions)
    - [Queued Listener Middleware](#queued-listener-middleware)
    - [Encrypted Queued Listeners](#encrypted-queued-listeners)
    - [Unique Event Listeners](#unique-event-listeners)
        - [Giữ Listeners Unique cho đến khi Processing Begins](#keeping-listeners-unique-until-processing-begins)
        - [Unique Listener Locks](#unique-listener-locks)
    - [Xử lý Failed Job](#handling-failed-jobs)
- [Dispatching Event](#dispatching-events)
    - [Dispatching Events After Database Transactions](#dispatching-events-after-database-transactions)
    - [Deferring Events](#deferring-events)
- [Event Subscriber](#event-subscribers)
    - [Viết Event Subscriber](#writing-event-subscribers)
    - [Đăng ký Event Subscriber](#registering-event-subscribers)
- [Testing](#testing)
    - [Faking a Subset of Events](#faking-a-subset-of-events)
    - [Scoped Events Fakes](#scoped-event-fakes)

<a name="introduction"></a>
## Giới thiệu

Các event của Laravel cung cấp một pattern observer implementation đơn giản, cho phép bạn subscribe và listen các event khác nhau xảy ra trong application của bạn. Các class event thường được lưu trong thư mục `app/Events`, còn các listen của các event đó được lưu trong thư mục `app/Listeners`. Đừng lo lắng nếu bạn không thấy các thư mục này trong application của bạn vì chúng sẽ được tạo khi bạn tạo các event và listener bằng các lệnh của Artisan console.

Các event đóng vai trò là một cách tuyệt vời để tách các khía cạnh khác nhau của application, vì một event có thể có nhiều listener mà không phụ thuộc vào lẫn nhau. Ví dụ: bạn có thể muốn gửi thông báo đến Slack cho người dùng của bạn mỗi khi đơn hàng đã được giao. Thay vì ghép code xử lý đơn đặt hàng với code thông báo của Slack, bạn có thể đưa ra một event `App\Events\OrderShipped`, mà listener có thể nhận và dùng để gửi một thông báo đến Slack.

<a name="generating-events-and-listeners"></a>
## Tạo Event và Listener

Để tạo một event và một listener nhanh chóng, bạn có thể sử dụng lệnh Artisan `make:event` và `make:listener`:

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

Bạn cũng có thể gọi lệnh Artisan `make:event` và `make:listener` mà không cần thêm tham số. Khi bạn làm như vậy, Laravel sẽ tự động yêu cầu bạn nhập thêm tên class và khi bạn tạo thì một listener, thì event mà listener đó sẽ listen cũng sẽ được yêu cầu:

```shell
php artisan make:event

php artisan make:listener
```

<a name="registering-events-and-listeners"></a>
## Registering Events and Listeners

<a name="event-discovery"></a>
### Event Discovery

Laravel sẽ tự động tìm và đăng ký các event listener của bạn bằng cách scan thư mục `Listeners` có trong application của bạn. Khi Laravel tìm thấy bất kỳ phương thức của class listener nào mà bắt đầu bằng `handle` hoặc `__invoke`, thì Laravel sẽ đăng ký các phương thức đó như các event listener cho event được khai báo trong signature của phương thức:

```php
use App\Events\PodcastProcessed;

class SendPodcastNotification
{
    /**
     * Handle the event.
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }
}
```

Bạn có thể listen nhiều event cùng một lúc bằng cách sử dụng kiểu union của PHP:

```php
/**
 * Handle the event.
 */
public function handle(PodcastProcessed|PodcastPublished $event): void
{
    // ...
}
```

Nếu bạn định lưu các listener của bạn trong một thư mục khác hoặc trong nhiều thư mục, bạn có thể hướng dẫn Laravel scan các thư mục đó bằng cách sử dụng phương thức `withEvents` có trong file `bootstrap/app.php` của ứng dụng:

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])
```

Bạn có thể scan các listener trong nhiều thư mục cùng mức bằng cách sử dụng ký tự `*` làm ký tự đại diện:

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/*/Listeners',
])
```

Lệnh `event:list` có thể được dùng để liệt kê ra tất cả các listener đã được đăng ký có trong ứng dụng của bạn:

```shell
php artisan event:list
```

<a name="event-discovery-in-production"></a>
#### Event Discovery in Production

Để tăng tốc ứng dụng của bạn, bạn nên cache lại một manifest của tất cả các listener của ứng dụng bằng cách sử dụng lệnh Artisan `optimize` hoặc `event:cache`. Thông thường, lệnh này nên được chạy như một phần của [quy trình triển khai](/docs/{{version}}/deployment#optimization) ứng dụng của bạn. Manifest này sẽ được framework sử dụng để tăng tốc quá trình đăng ký event. Lệnh `event:clear` có thể được sử dụng để hủy bộ nhớ cache của event.

<a name="manually-registering-events"></a>
### Đăng ký Event thủ công

Sử dụng facade `Event`, bạn có thể tự đăng ký các event và listener tương ứng của chúng trong phương thức `boot` của `AppServiceProvider` trong ứng dụng của bạn:

```php
use App\Domain\Orders\Events\PodcastProcessed;
use App\Domain\Orders\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(
        PodcastProcessed::class,
        SendPodcastNotification::class,
    );
}
```

Lệnh `event:list` có thể được dùng để liệt kê ra tất cả các listener đã được đăng ký có trong ứng dụng của bạn:

```shell
php artisan event:list
```

<a name="closure-listeners"></a>
### Closure Listeners

Thông thường, các listener được định nghĩa dưới dạng class; tuy nhiên, bạn cũng có thể tự đăng ký các listener dựa trên closure có trong phương thức `boot` của `AppServiceProvider` trong ứng dụng của bạn:

```php
use App\Events\PodcastProcessed;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (PodcastProcessed $event) {
        // ...
    });
}
```

<a name="queuable-anonymous-event-listeners"></a>
#### Queueable Anonymous Event Listeners

Khi đăng ký event listener dựa trên closure, bạn có thể bọc listener closure trong hàm `Illuminate\Events\queueable` để hướng dẫn Laravel thực thi listener này bằng cách sử dụng [queue](/docs/{{version}}/queues):

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    }));
}
```

Giống như queued job, bạn có thể sử dụng các phương thức `onConnection`, `onQueue`, và `delay` để tùy chỉnh việc thực thi queued listener:

```php
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->plus(seconds: 10)));
```

Nếu bạn muốn xử lý các lỗi nonymous queued listener, bạn có thể cung cấp một closure cho phương thức `catch` trong khi định nghĩa listener `queueable`. Closure này sẽ nhận vào một instance event và một instance `Throwable` đã gây ra lỗi cho listener:

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;
use Throwable;

Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->catch(function (PodcastProcessed $event, Throwable $e) {
    // The queued listener failed...
}));
```

<a name="wildcard-event-listeners"></a>
#### Wildcard Event Listeners

Bạn cũng có thể đăng ký listener bằng cách sử dụng ký tự `*` làm tham số đại diện, cho phép bạn nhận được nhiều event trên cùng một listener. Và nó nhận tên event là tham số đầu tiên và toàn bộ mảng dữ liệu event là tham số thứ hai:

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

<a name="defining-events"></a>
## Khai báo Event

Một event class về cơ bản là một data container chứa các thông tin liên quan đến event. Ví dụ: giả sử event `App\Events\OrderShipped` của chúng ta sẽ nhận một đối tượng [Eloquent ORM](/docs/{{version}}/eloquent):

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

Như bạn có thể thấy, event class này không chứa code logic. Nó là một container chứa instance `App\Models\Order` đã được mua. Trait `SerializesModels` được sử dụng trong event này để khôi phục lại bất kỳ model Eloquent nào nếu nó đã bị chuyển đổi bằng hàm `serialize` của PHP, chẳng hạn như khi sử dụng [queued listeners](#queued-event-listeners).

<a name="defining-listeners"></a>
## Khai báo Listener

Tiếp theo, chúng ta hãy xem một listener mẫu cho một event. Listener của event sẽ nhận vào một instance event trong phương thức `handle`. Lệnh Artisan `make:listener`, khi được gọi với tùy chọn `--event`, sẽ tự động import class event và khai báo nó vào trong phương thức `handle`. Trong phương thức `handle`, bạn có thể thực hiện bất kỳ hành động nào cần thiết để xử lý event:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     */
    public function __construct() {}

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Access the order using $event->order...
    }
}
```

> [!NOTE]
> Listener event của bạn cũng có thể khai báo bất kỳ sự phụ thuộc nào cần thiết ở trong hàm khởi tạo. Tất cả các listener event sẽ được resolve thông qua [service container](/docs/{{version}}/container), do đó, các phụ thuộc cũng sẽ được tự động thêm vào.

<a name="stopping-the-propagation-of-an-event"></a>
#### Stopping The Propagation Of An Event

Thỉnh thoảng, bạn có thể muốn ngừng việc truyền một event đến những listener khác. Bạn có thể làm như vậy bằng cách trả về `false` từ phương thức` handle` của listener của bạn.

<a name="queued-event-listeners"></a>
## Queued Event Listener

Queueing listener có thể có lợi nếu listener của bạn thực hiện một nhiệm vụ mà không cần phải phản hồi ngay lập tức như việc gửi email hoặc tạo một HTTP request. Trước khi dùng queued listener, hãy đảm bảo là bạn đã [cấu hình queue](/docs/{{version}}/queues) và chạy một queue worker trên server hoặc môi trường develop của bạn.

Để khai báo một listener sẽ được queue, hãy thêm interface `ShouldQueue` vào class listener. Listener được tạo bởi lệnh Artisan `make:listener` sẽ khai báo sẵn interface này và import nó vào namespace hiện tại, vì vậy bạn có thể sử dụng nó ngay lập tức:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

Và chỉ có thế! Bây giờ, khi an event handled by this listener is dispatched, the listener sẽ tự động được queue bởi event dispatcher bằng cách sử dụng [queue system](/docs/{{version}}/queues) của Laravel. Nếu không có ngoại lệ nào được đưa ra khi listener được thực thi bởi queue, thì queue job đó sẽ tự động bị xóa sau khi xử lý xong.

<a name="customizing-the-queue-connection-queue-name"></a>
#### Customizing The Queue Connection, Name, & Delay

Nếu bạn muốn tùy chỉnh kết nối của queue, tên queue hoặc delay time của queue được sử dụng bởi event listener, bạn có thể dùng các thuộc tính `Connection`, `Queue`, và `Delay` trong class listener của bạn:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Connection;
use Illuminate\Queue\Attributes\Delay;
use Illuminate\Queue\Attributes\Queue;

#[Connection('sqs')]
#[Queue('listeners')]
#[Delay(60)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```
Nếu bạn muốn định nghĩa một listener connection của queue, tên queue, hoặc một delay time khi ứng dụng chạy, bạn có thể định nghĩa các phương thức `viaConnection`, `viaQueue`, hoặc `withDelay` trên listener:

```php
/**
 * Get the name of the listener's queue connection.
 */
public function viaConnection(): string
{
    return 'sqs';
}

/**
 * Get the name of the listener's queue.
 */
public function viaQueue(): string
{
    return 'listeners';
}

/**
 * Get the number of seconds before the job should be processed.
 */
public function withDelay(OrderShipped $event): int
{
    return $event->highPriority ? 0 : 60;
}
```

<a name="conditionally-queueing-listeners"></a>
#### Conditionally Queueing Listeners

Thỉnh thoảng, bạn có thể cần phải xác định xem một listener có nên được queue hay không dựa vào một số dữ liệu chỉ có trong lúc runtime. Để thực hiện điều này, phương thức `shouldQueue` có thể được thêm vào trong listener để xác định xem listener này có nên được queue hay không. Nếu phương thức `shouldQueue` trả về `false`, listener sẽ không được queue:

```php
<?php

namespace App\Listeners;

use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;

class RewardGiftCard implements ShouldQueue
{
    /**
     * Reward a gift card to the customer.
     */
    public function handle(OrderCreated $event): void
    {
        // ...
    }

    /**
     * Determine whether the listener should be queued.
     */
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

<a name="manually-interacting-with-the-queue"></a>
### Tương tác thủ công với queue

Nếu bạn cần tự truy cập các phương thức `delete` và `release` của queue job, bạn có thể làm như vậy bằng cách sử dụng trait `Illuminate\Queue\InteractsWithQueue`. Trait này sẽ được mặc định import sẵn vào trong các listener nếu nó được tạo bằng lệnh artisan và cung cấp quyền truy cập vào các phương thức `delete` và `release`:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        if ($condition) {
            $this->release(30);
        }
    }
}
```

<a name="queued-event-listeners-and-database-transactions"></a>
### Queued Event Listeners và Database Transactions

Khi các queued listener được gửi đi trong các database transaction, chúng có thể được xử lý bởi queue trước khi database transaction được thực hiện. Khi điều này xảy ra, bất kỳ cập nhật nào bạn đã thực hiện đối với model hoặc record cơ sở dữ liệu trong quá trình database transaction có thể chưa được lưu vào trong cơ sở dữ liệu. Ngoài ra, bất kỳ model hoặc record cơ sở dữ liệu nào được tạo trong transaction cũng có thể không tồn tại trong cơ sở dữ liệu. Nếu listener của bạn phụ thuộc vào các model này, các lỗi không mong muốn có thể xảy ra khi xử lý các job được gửi đi từ queued listener.

Nếu tùy chọn `after_commit` trong cấu hình queue connection được set thành `false`, thì bạn vẫn có thể cho biết một queued listener sẽ được gửi đi sau khi tất cả các database transaction đã được thực hiện bằng cách implement một interface `ShouldQueueAfterCommit` trên class listener:

```php
<?php

namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueueAfterCommit
{
    use InteractsWithQueue;
}
```

> [!NOTE]
> Để tìm hiểu về cách khắc phục những sự cố này, vui lòng xem lại tài liệu về [queued job và database transaction](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="queued-listener-middleware"></a>
### Queued Listener Middleware

Các queued listener cũng có thể sử dụng [job middleware](/docs/{{version}}/queues#job-middleware). Job middleware cho phép bạn tùy chỉnh logic bao bọc việc thực thi của các queued listener, giúp giảm thiểu các code lặp lại trong chính các listener đó. Sau khi tạo job middleware, chúng có thể được gán vào một listener bằng cách trả về chúng từ phương thức `middleware` của listener:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use App\Jobs\Middleware\RateLimited;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Process the event...
    }

    /**
     * Get the middleware the listener should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(OrderShipped $event): array
    {
        return [new RateLimited];
    }
}
```

<a name="encrypted-queued-listeners"></a>
#### Encrypted Queued Listeners

Laravel cho phép bạn đảm bảo tính privacy và toàn vẹn của dữ liệu trong queued listener thông qua [encryption](/docs/{{version}}/encryption). Để bắt đầu, chỉ cần thêm interface `ShouldBeEncrypted` vào class listener. Sau khi interface này được thêm vào class, Laravel sẽ tự động mã hóa listener của bạn trước khi đưa nó vào queue:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="unique-event-listeners"></a>
### Unique Event Listeners

> Các unique listener yêu cầu một cache driver hỗ trợ [khoá atomic](/docs/{{version}}/cache#atomic-locks). Hiện tại, các cache driver `memcached`, `redis`, `dynamodb`, `database`, `file`, và `array` đều hỗ trợ khoá atomic.

Thỉnh thoảng, bạn muốn đảm bảo là chỉ có một instance duy nhất của một listener nằm trong queue tại bất kỳ thời điểm nào. Bạn có thể làm như vậy bằng cách implement interface `ShouldBeUnique` trong class listener của bạn:

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    public function __invoke(LicenseSaved $event): void
    {
        // ...
    }
}
```

Trong ví dụ trên, listener `AcquireProductKey` là duy nhất. Vì vậy, listener sẽ không được đưa vào queue nếu một instance khác của listener này đã nằm trong queue và chưa được xử lý xong. Điều này đảm bảo rằng chỉ có một product key duy nhất được tạo cho mỗi license, ngay cả khi license đó được lưu nhiều lần trong thời gian ngắn.

Trong một số trường hợp nhất định, bạn có thể muốn định nghĩa một "key" cụ thể để làm cho listener trở nên duy nhất hoặc bạn muốn chỉ định một khoảng thời gian timeout mà sau thời gian đó listener sẽ không còn là duy nhất nữa. Để thực hiện điều này, bạn có thể khai báo các thuộc tính hoặc phương thức `uniqueId` và `uniqueFor` trong class listener của bạn. Các phương thức này sẽ nhận vào một instance của event, cho phép bạn sử dụng dữ liệu của event để tạo ra giá trị trả về:

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    /**
     * The number of seconds after which the listener's unique lock will be released.
     *
     * @var int
     */
    public $uniqueFor = 3600;

    public function __invoke(LicenseSaved $event): void
    {
        // ...
    }

    /**
     * Get the unique ID for the listener.
     */
    public function uniqueId(LicenseSaved $event): string
    {
        return 'listener:'.$event->license->id;
    }
}
```

Trong ví dụ trên, listener `AcquireProductKey` là duy nhất dựa theo ID của license. Do đó, bất kỳ lần gửi mới nào của listener cho cùng một license đó sẽ bị bỏ qua cho đến khi listener hiện tại được xử lý xong. Điều này sẽ ngăn chặn việc tạo ra các product key trùng lặp cho cùng một license. Ngoài ra, nếu listener hiện tại không được xử lý trong vòng một giờ, khóa unique đó sẽ được giải phóng và một listener khác với cùng unique key đó có thể được đưa vào queue.

> Nếu ứng dụng của bạn đang gửi event từ nhiều web server hoặc container khác nhau, bạn nên đảm bảo rằng tất cả các server đều giao tiếp với cùng một server cache trung tâm để Laravel có thể xác định chính xác xem một listener có phải là duy nhất hay không.

<a name="keeping-listeners-unique-until-processing-begins"></a>
#### Giữ Listeners Unique cho đến khi Processing Begins

Mặc định, các unique listener sẽ được "giải phóng khóa" sau khi listener hoàn tất việc xử lý hoặc thất bại trong tất cả các lần thử lại. Tuy nhiên, có thể có những trường hợp bạn muốn listener của bạn giải phóng khóa ngay trước khi nó bắt đầu được xử lý. Để thực hiện điều này, listener của bạn nên implement interface `ShouldBeUniqueUntilProcessing` thay vì `ShouldBeUnique`:

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

<a name="unique-listener-locks"></a>
#### Unique Listener Locks

Ở phía sau, khi một listener `ShouldBeUnique` được gửi, Laravel sẽ cố gắng lấy một [khóa atomic](/docs/{{version}}/cache#atomic-locks) cùng với key là `uniqueId`. Nếu khóa này đã bị giữ, listener sẽ không được gửi đi. Khóa này sẽ được giải phóng khi listener hoàn tất việc xử lý hoặc thất bại trong tất cả các lần thử lại. Mặc định, Laravel sẽ sử dụng cache driver mặc định để lấy khóa này. Tuy nhiên, nếu bạn muốn sử dụng một driver khác để lấy khóa, bạn có thể định nghĩa một phương thức `uniqueVia` trả về cache driver mà bạn muốn sử dụng:

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    // ...

    /**
     * Get the cache driver for the unique listener lock.
     */
    public function uniqueVia(LicenseSaved $event): Repository
    {
        return Cache::driver('redis');
    }
}
```

> Nếu bạn chỉ cần giới hạn việc xử lý đồng bộ của một listener, hãy sử dụng middleware job [WithoutOverlapping](/docs/{{version}}/queues#preventing-job-overlaps) để thay thế.

<a name="handling-failed-jobs"></a>
### Xử lý Failed Job

Thỉnh thoảng queue của event listener của bạn có thể bị thất bại. Nếu queued listener chạy vượt quá số lần thử tối đa được định nghĩa bởi queue worker của bạn, phương thức `failed` sẽ được gọi trong listener của bạn. Phương thức `failed` nhận vào instance event và một `Throwable` nguyên nhân gây ra lỗi:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // ...
    }

    /**
     * Handle a job failure.
     */
    public function failed(OrderShipped $event, Throwable $exception): void
    {
        // ...
    }
}
```

<a name="specifying-queued-listener-maximum-attempts"></a>
#### Specifying Queued Listener Maximum Attempts

Nếu một trong những queued listener của bạn gặp phải lỗi, bạn có thể không muốn nó tiếp tục thử lại nó một lần nào nữa. Do đó, Laravel cung cấp nhiều cách khác nhau để chỉ định số lần thử lại hoặc khoảng thời gian của một listener có thể được thử lại.

Bạn có thể dùng thuộc tính `Tries` trên class listener của bạn để chỉ định số lần mà listener có thể được thử lại trước khi nó được coi là thất bại:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\InteractsWithQueue;

#[Tries(5)]
class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    // ...
}
```

Là một giải pháp thay thế cho việc xác định số lần mà một listener có thể được thử trước khi nó thất bại, bạn có thể định nghĩa thời điểm mà listener không còn được thử lại nữa. Điều này cho phép listener được thử bao nhiêu tuỳ thích trong một khoảng thời gian nhất định. Để định nghĩa khoảng thời gian mà một listener không còn được thử nữa, bạn hãy thêm một phương thức `retryUntil` vào class listener của bạn. Phương thức này sẽ trả về một instance `DateTime`:

```php
use DateTime;

/**
 * Determine the time at which the listener should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 5);
}
```

Nếu cả `retryUntil` và `tries` được định nghĩa, Laravel sẽ ưu tiên phương thức `retryUntil`.

<a name="specifying-queued-listener-backoff"></a>
#### Specifying Queued Listener Backoff

Nếu bạn muốn cấu hình số giây mà Laravel sẽ đợi trước khi thử lại một listener bị exception, bạn có thể làm như vậy bằng cách dùng thuộc tính `Backoff` trên class listener của bạn:

```php
<?php

namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Backoff;

#[Backoff(3)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

Nếu bạn muốn yêu cầu một logic phức tạp hơn để xác định thời gian backoff của listener, bạn có thể định nghĩa một phương thức `backoff` trên class listener của bạn:

```php
/**
 * Calculate the number of seconds to wait before retrying the queued listener.
 */
public function backoff(): int
{
    return 3;
}
```

Bạn có thể dễ dàng cấu hình "backoff" theo cấp số nhân bằng cách trả về một mảng các giá trị backoff từ phương thức `backoff`. Trong ví dụ này, độ trễ thử lại sẽ là 1 giây cho lần thử lại đầu tiên, và 5 giây cho lần thử lại thứ hai, và 10 giây cho lần thử lại thứ ba và 10 giây cho các lần thử lại tiếp theo nếu còn nhiều lần thử khác:

```php
/**
 * Calculate the number of seconds to wait before retrying the queued listener.
 *
 * @return array<int, int>
 */
public function backoff(): array
{
    return [1, 5, 10];
}
```

<a name="specifying-queued-listener-max-exceptions"></a>
#### Specifying Queued Listener Max Exceptions

Thỉnh thoảng bạn có thể muốn chỉ định một queued listener có thể được thử lại nhiều lần, nhưng nó sẽ bị thất bại nếu các lần thử lại đó bị thất bại bởi một số lượng exception nhất định chưa được xử lý (trái ngược với việc được release trực tiếp bởi phương thức `release`). Để thực hiện điều này, bạn có thể sử dụng các thuộc tính `Tries` và `MaxExceptions` trên class listener của bạn:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\MaxExceptions;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\InteractsWithQueue;

#[Tries(25)]
#[MaxExceptions(3)]
class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Process the event...
    }
}
```

Trong ví dụ này, listener sẽ được thử tối đa 25 lần. Tuy nhiên, listener sẽ bị thất bại nếu có ba exception chưa được xử lý được đưa ra.

<a name="specifying-queued-listener-timeout"></a>
#### Specifying Queued Listener Timeout

Thông thường, bạn biết thời gian mà bạn mong muốn các queued listener của bạn được thực hiện. Vì lý do này, Laravel cho phép bạn chỉ định một giá trị "timeout". Nếu một listener đang xử lý lâu hơn số giây được chỉ định bởi giá trị timeout, worker đang xử lý listener đó sẽ thoát với một lỗi. Bạn có thể định nghĩa số giây tối đa mà một listener được phép chạy bằng cách dùng thuộc tính `Timeout` trên class listener của bạn:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Timeout;

#[Timeout(120)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

Nếu bạn muốn chỉ định một listener sẽ bị đánh dấu là thất bại khi bị timeout, bạn có thể dùng thuộc tính `FailOnTimeout` trên class listener:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\FailOnTimeout;

#[FailOnTimeout]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

<a name="dispatching-events"></a>
## Dispatching Event

Để gửi một event, bạn có thể gọi phương thức static `dispatch` trong event. Phương thức này được cung cấp trên event bởi trait `Illuminate\Foundation\Events\Dispatchable`. Mọi tham số được truyền cho phương thức `dispatch` sẽ được truyền cho hàm tạo của event:

```php
<?php

namespace App\Http\Controllers;

use App\Events\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // Order shipment logic...

        OrderShipped::dispatch($order);

        return redirect('/orders');
    }
}
```

Nếu bạn muốn gửi một event có điều kiện, bạn có thể sử dụng các phương thức `dispatchIf` và `dispatchUnless`:

```php
OrderShipped::dispatchIf($condition, $order);

OrderShipped::dispatchUnless($condition, $order);
```

> [!NOTE]
> Khi testing, nếu bạn cần kiểm tra một số event được gửi đi mà không cần chạy đến các listener của các event. [Helper testing mặc định](#testing) của Laravel giúp việc này trở nên dễ dàng.

<a name="dispatching-events-after-database-transactions"></a>
### Dispatching Events After Database Transactions

Thỉnh thoảng, bạn có thể muốn hướng dẫn Laravel chỉ gửi event sau khi transaction đã được commit. Để làm như vậy, bạn có thể implement interface `ShouldDispatchAfterCommit` trên class event.

Interface này sẽ hướng dẫn Laravel không gửi event cho đến khi transaction hiện tại được commit. Nếu transaction bị lỗi, event sẽ bị hủy. Nếu không có transaction nào đang thực hiện khi event được gửi đi, thì event đó sẽ được gửi đi ngay lập tức:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

<a name="deferring-events"></a>
### Deferring Events

Deferred events cho phép bạn trì hoãn việc gửi các model event và chạy các event listener cho đến sau khi một đoạn code cụ thể đã hoàn thành. Điều này đặc biệt hữu ích khi bạn cần đảm bảo tất cả các record liên quan đã được tạo trước khi các event listener được kích hoạt.

Để trì hoãn các event, hãy cung cấp một closure cho phương thức `Event::defer()`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);

    $user->posts()->create(['title' => 'My first post!']);
});
```

Tất cả các event được kích hoạt bên trong closure sẽ được gửi đi sau khi closure đã được chạy. Điều này đảm bảo rằng các event listener có quyền truy cập vào tất cả các record liên quan đã được tạo trong quá trình event được chạy. Nếu có một exception xảy ra bên trong closure, các event sẽ không được gửi đi.

Để chỉ trì hoãn các event cụ thể, hãy truyền một mảng các event làm tham số thứ hai cho phương thức `defer`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);

    $user->posts()->create(['title' => 'My first post!']);
}, ['eloquent.created: '.User::class]);
```

<a name="event-subscribers"></a>
## Event Subscriber

<a name="writing-event-subscribers"></a>
### Viết Event Subscriber

Event subscriber là các class có thể đăng ký nhiều event từ trong chính class subscriber đó, cho phép bạn định nghĩa nhiều xử lý event trong cùng một class. Subscriber nên định nghĩa một phương thức `subscribe`, nó sẽ nhận một instance event dispatcher. Bạn có thể gọi phương thức `listen` trong dispatcher đó để đăng ký event listener:

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * Register the listeners for the subscriber.
     */
    public function subscribe(Dispatcher $events): void
    {
        $events->listen(
            Login::class,
            [UserEventSubscriber::class, 'handleUserLogin']
        );

        $events->listen(
            Logout::class,
            [UserEventSubscriber::class, 'handleUserLogout']
        );
    }
}
```

Nếu các phương thức event listener của bạn được định nghĩa trong chính subscriber, bạn có thể thấy thuận tiện hơn khi trả về một mảng các event và các tên phương thức từ phương thức `subscribe` trong subscriber. Laravel sẽ tự động xác định tên class của subscriber khi đăng ký event listener:

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * Register the listeners for the subscriber.
     *
     * @return array<string, string>
     */
    public function subscribe(Dispatcher $events): array
    {
        return [
            Login::class => 'handleUserLogin',
            Logout::class => 'handleUserLogout',
        ];
    }
}
```

<a name="registering-event-subscribers"></a>
### Đăng ký Event Subscriber

Sau khi đã tạo xong subscriber, Laravel sẽ tự động đăng ký các phương thức handler có trong subscriber nếu chúng tuân thủ các [quy ước discovery event](#event-discovery) của Laravel. Nếu không phải vậy, bạn có thể đăng ký subscriber của bạn theo cách của bạn bằng cách sử dụng phương thức `subscribe` của facade `Event`. Thông thường, việc này nên được thực hiện trong phương thức `boot` của `AppServiceProvider` của ứng dụng:

```php
<?php

namespace App\Providers;

use App\Listeners\UserEventSubscriber;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::subscribe(UserEventSubscriber::class);
    }
}
```

<a name="testing"></a>
## Testing

Khi test code gửi event, bạn có thể muốn hướng dẫn Laravel không thực hiện event listener, vì code của listener có thể được kiểm tra trực tiếp và riêng biệt với code gửi event. Tất nhiên, để kiểm tra listener, bạn có thể khởi tạo một instance listener và gọi phương thức `handle` trực tiếp trong bài test của bạn.

Bằng cách sử dụng phương thức `fake` của facade `Event`, bạn có thể ngăn listener được chạy, và chạy code đang được kiểm tra và sau đó xác nhận event nào đã được ứng dụng của bạn gửi bằng các phương thức `assertDispatched`, `assertNotDispatched` và `assertNothingDispatched`:

```php tab=Pest
<?php

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;

test('orders can be shipped', function () {
    Event::fake();

    // Perform order shipping...

    // Assert that an event was dispatched...
    Event::assertDispatched(OrderShipped::class);

    // Assert an event was dispatched twice...
    Event::assertDispatched(OrderShipped::class, 2);

    // Assert an event was dispatched once...
    Event::assertDispatchedOnce(OrderShipped::class);

    // Assert an event was not dispatched...
    Event::assertNotDispatched(OrderFailedToShip::class);

    // Assert that no events were dispatched...
    Event::assertNothingDispatched();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order shipping.
     */
    public function test_orders_can_be_shipped(): void
    {
        Event::fake();

        // Perform order shipping...

        // Assert that an event was dispatched...
        Event::assertDispatched(OrderShipped::class);

        // Assert an event was dispatched twice...
        Event::assertDispatched(OrderShipped::class, 2);

        // Assert an event was dispatched once...
        Event::assertDispatchedOnce(OrderShipped::class);

        // Assert an event was not dispatched...
        Event::assertNotDispatched(OrderFailedToShip::class);

        // Assert that no events were dispatched...
        Event::assertNothingDispatched();
    }
}
```

Bạn có thể truyền một closure cho các phương thức `assertDispatched` hoặc `assertNotDispatched` để yêu cầu một event đã được gửi đi và pass qua "bài kiểm tra" đã cho. Nếu có ít nhất một event đã được gửi đi và pass qua bài kiểm tra đã cho thì yêu cầu sẽ thành công:

```php
Event::assertDispatched(function (OrderShipped $event) use ($order) {
    return $event->order->id === $order->id;
});
```

Nếu bạn chỉ muốn yêu cầu listener của một event đang nhận một event nhất định, bạn có thể sử dụng phương thức `assertListening`:

```php
Event::assertListening(
    OrderShipped::class,
    SendShipmentNotification::class
);
```

> [!WARNING]
> Sau khi gọi `Event::fake()`, sẽ không có listener event nào được thực thi. Vì vậy, nếu các bài kiểm tra của bạn sử dụng các model factory dựa trên các event, chẳng hạn như tạo UUID trong event `creating` của nodel, bạn nên gọi `Event::fake()` **sau** khi sử dụng các factory của bạn.

<a name="faking-a-subset-of-events"></a>
### Faking a Subset of Events

Nếu bạn chỉ muốn làm fake một listener event cho một tập hợp event cụ thể, bạn có thể truyền chúng cho phương thức `fake` hoặc `fakeFor`:

```php tab=Pest
test('orders can be processed', function () {
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Other events are dispatched as normal...
    $order->update([
        // ...
    ]);
});
```

```php tab=PHPUnit
/**
 * Test order process.
 */
public function test_orders_can_be_processed(): void
{
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Other events are dispatched as normal...
    $order->update([
        // ...
    ]);
}
```

Bạn có thể fake tất cả các event ngoại trừ một tập hợp các event được chỉ định bằng phương thức `except`:

```php
Event::fake()->except([
    OrderCreated::class,
]);
```

<a name="scoped-event-fakes"></a>
### Scoped Event Fakes

Nếu bạn chỉ muốn fake listener event cho một phần bài test của bạn, bạn có thể sử dụng phương thức `fakeFor`:

```php tab=Pest
<?php

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;

test('orders can be processed', function () {
    $order = Event::fakeFor(function () {
        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        return $order;
    });

    // Events are dispatched as normal and observers will run...
    $order->update([
        // ...
    ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order process.
     */
    public function test_orders_can_be_processed(): void
    {
        $order = Event::fakeFor(function () {
            $order = Order::factory()->create();

            Event::assertDispatched(OrderCreated::class);

            return $order;
        });

        // Events are dispatched as normal and observers will run...
        $order->update([
            // ...
        ]);
    }
}
```
