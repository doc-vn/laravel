# Events

- [Giới thiệu](#introduction)
- [Đăng ký Event và Listener](#registering-events-and-listeners)
    - [Tạo Event và Listener](#generating-events-and-listeners)
    - [Đăng ký Event thủ công](#manually-registering-events)
    - [Event Discovery](#event-discovery)
- [Khai báo Event](#defining-events)
- [Khai báo Listener](#defining-listeners)
- [Queued Event Listener](#queued-event-listeners)
    - [Tương tác thủ công với queue](#manually-interacting-with-the-queue)
    - [Queued Event Listeners và Database Transactions](#queued-event-listeners-and-database-transactions)
    - [Xử lý Failed Job](#handling-failed-jobs)
- [Dispatching Event](#dispatching-events)
    - [Dispatching Events After Database Transactions](#dispatching-events-after-database-transactions)
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

<a name="registering-events-and-listeners"></a>
## Đăng ký Event và Listener

`App\Providers\EventServiceProvider` đi kèm trong application Laravel cung cấp một cách đăng ký dễ dàng cho tất cả các listener event trong application của bạn. Thuộc tính `listen` chứa một mảng gồm các event (là các key) và listener (là các giá trị). Bạn có thể thêm nhiều event vào mảng này khi application của bạn yêu cầu. Ví dụ: hãy thêm một event `OrderShipped` như sau:

    use App\Events\OrderShipped;
    use App\Listeners\SendShipmentNotification;

    /**
     * The event listener mappings for the application.
     *
     * @var array<class-string, array<int, class-string>>
     */
    protected $listen = [
        OrderShipped::class => [
            SendShipmentNotification::class,
        ],
    ];

> [!NOTE]
> Lệnh `event:list` có thể được sử dụng để hiển thị danh sách tất cả các event và listener đã được đăng ký bởi ứng dụng của bạn.

<a name="generating-events-and-listeners"></a>
### Tạo Event và Listener

Tất nhiên, việc tạo bằng tay các file cho các event và listener này là rất công kềnh. Thay vào đó, hãy thêm listener và event của nó vào trong `EventServiceProvider` của bạn và sử dụng lệnh Artisan `event:generate`. Lệnh này sẽ tạo ra bất kỳ các event hoặc các listener nào được liệt kê trong mảng `EventServiceProvider` mà chưa tồn tại:

```shell
php artisan event:generate
```

Ngoài ra, bạn có thể sử dụng các lệnh Artisan `make:event` và `make:listener` để tạo các event và listener riêng lẻ:

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

<a name="manually-registering-events"></a>
### Đăng ký Event thủ công

Thông thường, các event nên được đăng ký thông qua `EventServiceProvider` vào mảng `$listen`; tuy nhiên, bạn cũng có thể đăng ký các event listener dựa trên class hoặc closure bằng cách đưa nó vào trong phương thức `boot` của `EventServiceProvider`:

    use App\Events\PodcastProcessed;
    use App\Listeners\SendPodcastNotification;
    use Illuminate\Support\Facades\Event;

    /**
     * Register any other events for your application.
     */
    public function boot(): void
    {
        Event::listen(
            PodcastProcessed::class,
            SendPodcastNotification::class,
        );

        Event::listen(function (PodcastProcessed $event) {
            // ...
        });
    }

<a name="queuable-anonymous-event-listeners"></a>
#### Queueable Anonymous Event Listeners

Khi đăng ký event listener dựa trên closure, bạn có thể bọc listener closure trong hàm `Illuminate\Events\queueable` để hướng dẫn Laravel thực thi listener này bằng cách sử dụng [queue](/docs/{{version}}/queues):

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;

    /**
     * Register any other events for your application.
     */
    public function boot(): void
    {
        Event::listen(queueable(function (PodcastProcessed $event) {
            // ...
        }));
    }

Giống như queued job, bạn có thể sử dụng các phương thức `onConnection`, `onQueue`, và `delay` để tùy chỉnh việc thực thi queued listener:

    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    })->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));

Nếu bạn muốn xử lý các lỗi nonymous queued listener, bạn có thể cung cấp một closure cho phương thức `catch` trong khi định nghĩa listener `queueable`. Closure này sẽ nhận vào một instance event và một instance `Throwable` đã gây ra lỗi cho listener:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;

    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // The queued listener failed...
    }));

<a name="wildcard-event-listeners"></a>
#### Wildcard Event Listeners

Bạn thậm chí có thể đăng ký listener bằng cách sử dụng ký tự đại diện `*` làm tham số, cho phép bạn nhận được nhiều event trên cùng một listener. Và nó nhận tên event là tham số đầu tiên và toàn bộ mảng dữ liệu event là tham số thứ hai:

    Event::listen('event.*', function (string $eventName, array $data) {
        // ...
    });

<a name="event-discovery"></a>
### Event Discovery

Thay vì phải đăng ký các event và listener theo cách thủ công trong mảng `$listen` của `EventServiceProvider`, bạn có thể bật tính năng event discovery. Khi tính năng event discovery được bật, Laravel sẽ tự động tìm kiếm và đăng ký các event, listener của bạn bằng cách quét thư mục `Listeners` của ứng dụng của bạn. Ngoài ra, mọi event được liệt kê trong `EventServiceProvider` vẫn sẽ được đăng ký.

Laravel sẽ tìm các event listener bằng cách quét các class listener dùng class động của PHP. Khi Laravel tìm thấy bất kỳ phương thức class listener nào bắt đầu bằng `handle` hoặc `__invoke`, Laravel sẽ đăng ký các phương thức đó làm event listener cho các event được khai báo trong signature của phương thức:

    use App\Events\PodcastProcessed;

    class SendPodcastNotification
    {
        /**
         * Handle the given event.
         */
        public function handle(PodcastProcessed $event): void
        {
            // ...
        }
    }

Mặc định tính năng event discovery sẽ bị tắt, nhưng bạn có thể bật tính năng này bằng cách ghi đè phương thức `shouldDiscoverEvents` của file `EventServiceProvider` trong ứng dụng của bạn:

    /**
     * Determine if events and listeners should be automatically discovered.
     */
    public function shouldDiscoverEvents(): bool
    {
        return true;
    }

Mặc định, tất cả các class listener trong thư mục `app/Listeners` của ứng dụng của bạn sẽ được quét. Nếu bạn muốn định nghĩa thêm các thư mục để quét, bạn có thể ghi đè phương thức `discoverEventsWithin` trong file `EventServiceProvider` của bạn:

    /**
     * Get the listener directories that should be used to discover events.
     *
     * @return array<int, string>
     */
    protected function discoverEventsWithin(): array
    {
        return [
            $this->app->path('Listeners'),
        ];
    }

<a name="event-discovery-in-production"></a>
#### Event Discovery In Production

Trong bản production, bạn có thể không muốn framework quét tất cả các listener của bạn trong mọi request. Do đó, trong quá trình deploy, bạn nên chạy lệnh Artisan `event:cache` để lưu cache một file gồm danh sách tất cả các event và listener có trong ứng dụng của bạn. Danh sách này sẽ được framework sử dụng để tăng tốc trong quá trình đăng ký event. Lệnh `event:clear` có thể được sử dụng để hủy bỏ file cache này.

<a name="defining-events"></a>
## Khai báo Event

Một event class về cơ bản là một data container chứa các thông tin liên quan đến event. Ví dụ: giả sử event `App\Events\OrderShipped` của chúng ta sẽ nhận một đối tượng [Eloquent ORM](/docs/{{version}}/eloquent):

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

Như bạn có thể thấy, event class này không chứa code logic. Nó là một container chứa instance `App\Models\Order` đã được mua. Trait `SerializesModels` được sử dụng trong event này để khôi phục lại bất kỳ model Eloquent nào nếu nó đã bị chuyển đổi bằng hàm `serialize` của PHP, chẳng hạn như khi sử dụng [queued listeners](#queued-event-listeners).

<a name="defining-listeners"></a>
## Khai báo Listener

Tiếp theo, chúng ta hãy xem một listener mẫu cho một event. Listener của event sẽ nhận vào một instance event trong phương thức `handle`. Lệnh Artisan `event:generate` và `make:listener` sẽ tự động import class event và khai báo nó vào trong phương thức `handle`. Trong phương thức `handle`, bạn có thể thực hiện bất kỳ hành động nào cần thiết để xử lý event:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         */
        public function __construct()
        {
            // ...
        }

        /**
         * Handle the event.
         */
        public function handle(OrderShipped $event): void
        {
            // Access the order using $event->order...
        }
    }

> [!NOTE]
> Listener event của bạn cũng có thể khai báo bất kỳ sự phụ thuộc nào cần thiết ở trong hàm khởi tạo. Tất cả các listener event sẽ được resolve thông qua [service container](/docs/{{version}}/container), do đó, các phụ thuộc cũng sẽ được tự động thêm vào.

<a name="stopping-the-propagation-of-an-event"></a>
#### Stopping The Propagation Of An Event

Thỉnh thoảng, bạn có thể muốn ngừng việc truyền một event đến những listener khác. Bạn có thể làm như vậy bằng cách trả về `false` từ phương thức` handle` của listener của bạn.

<a name="queued-event-listeners"></a>
## Queued Event Listener

Queueing listener có thể có lợi nếu listener của bạn thực hiện một nhiệm vụ mà không cần phải phản hồi ngay lập tức như việc gửi email hoặc tạo một HTTP request. Trước khi dùng queued listener, hãy đảm bảo là bạn đã [cấu hình queue](/docs/{{version}}/queues) và chạy một queue worker trên server hoặc môi trường develop của bạn.

Để khai báo một listener sẽ được queue, hãy thêm interface `ShouldQueue` vào class listener. Listener được tạo bởi lệnh Artisan `event:generate` và `make:listener` sẽ khai báo sẵn interface này và import nó vào namespace hiện tại, vì vậy bạn có thể sử dụng nó ngay lập tức:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        // ...
    }

Và chỉ có thế! Bây giờ, khi an event handled by this listener is dispatched, the listener sẽ tự động được queue bởi event dispatcher bằng cách sử dụng [queue system](/docs/{{version}}/queues) của Laravel. Nếu không có ngoại lệ nào được đưa ra khi listener được thực thi bởi queue, thì queue job đó sẽ tự động bị xóa sau khi xử lý xong.

<a name="customizing-the-queue-connection-queue-name"></a>
#### Customizing The Queue Connection, Name, & Delay

Nếu bạn muốn tùy chỉnh kết nối của queue, tên queue hoặc delay time của queue được sử dụng bởi event listener, bạn có thể định nghĩa các thuộc tính `$connection`, `$queue`, hoặc `$delay` trong class listener của bạn:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * The name of the connection the job should be sent to.
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * The name of the queue the job should be sent to.
         *
         * @var string|null
         */
        public $queue = 'listeners';

        /**
         * The time (seconds) before the job should be processed.
         *
         * @var int
         */
        public $delay = 60;
    }

Nếu bạn muốn định nghĩa một listener connection của queue, tên queue, hoặc một delay time khi ứng dụng chạy, bạn có thể định nghĩa các phương thức `viaConnection`, `viaQueue`, hoặc `withDelay` trên listener:

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

<a name="conditionally-queueing-listeners"></a>
#### Conditionally Queueing Listeners

Thỉnh thoảng, bạn có thể cần phải xác định xem một listener có nên được queue hay không dựa vào một số dữ liệu chỉ có trong lúc runtime. Để thực hiện điều này, phương thức `shouldQueue` có thể được thêm vào trong listener để xác định xem listener này có nên được queue hay không. Nếu phương thức `shouldQueue` trả về `false`, listener sẽ không được thực thi:

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

<a name="manually-interacting-with-the-queue"></a>
### Tương tác thủ công với queue

Nếu bạn cần tự truy cập các phương thức `delete` và `release` của queue job, bạn có thể làm như vậy bằng cách sử dụng trait `Illuminate\Queue\InteractsWithQueue`. Trait này sẽ được mặc định import sẵn vào trong các listener nếu nó được tạo bằng lệnh artisan và cung cấp quyền truy cập vào các phương thức `delete` và `release`:

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
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="queued-event-listeners-and-database-transactions"></a>
### Queued Event Listeners và Database Transactions

Khi các queued listener được gửi đi trong các database transaction, chúng có thể được xử lý bởi queue trước khi database transaction được thực hiện. Khi điều này xảy ra, bất kỳ cập nhật nào bạn đã thực hiện đối với model hoặc record cơ sở dữ liệu trong quá trình database transaction có thể chưa được lưu vào trong cơ sở dữ liệu. Ngoài ra, bất kỳ model hoặc record cơ sở dữ liệu nào được tạo trong transaction cũng có thể không tồn tại trong cơ sở dữ liệu. Nếu listener của bạn phụ thuộc vào các model này, các lỗi không mong muốn có thể xảy ra khi xử lý các job được gửi đi từ queued listener.

Nếu tùy chọn `after_commit` trong cấu hình queue connection được set thành `false`, thì bạn vẫn có thể cho biết một queued listener sẽ được gửi đi sau khi tất cả các database transaction đã được thực hiện bằng cách implement một interface `ShouldHandleEventsAfterCommit` trên class listener:

    <?php

    namespace App\Listeners;

    use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue, ShouldHandleEventsAfterCommit
    {
        use InteractsWithQueue;
    }

> [!NOTE]
> Để tìm hiểu về cách khắc phục những sự cố này, vui lòng xem lại tài liệu về [queued job và database transaction](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="handling-failed-jobs"></a>
### Xử lý Failed Job

Thỉnh thoảng queue của event listener của bạn có thể bị thất bại. Nếu queued listener chạy vượt quá số lần thử tối đa được định nghĩa bởi queue worker của bạn, phương thức `failed` sẽ được gọi trong listener của bạn. Phương thức `failed` nhận vào instance event và một `Throwable` nguyên nhân gây ra lỗi:

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

<a name="specifying-queued-listener-maximum-attempts"></a>
#### Specifying Queued Listener Maximum Attempts

Nếu một trong những queued listener của bạn gặp phải lỗi, bạn có thể không muốn nó tiếp tục thử lại nó một lần nào nữa. Do đó, Laravel cung cấp nhiều cách khác nhau để chỉ định số lần thử lại hoặc khoảng thời gian của một listener có thể được thử lại.

Bạn có thể định nghĩa một thuộc tính `$tries` trên class listener của bạn để chỉ định số lần mà listener có thể được thử lại trước khi nó được coi là thất bại:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * The number of times the queued listener may be attempted.
         *
         * @var int
         */
        public $tries = 5;
    }

Là một giải pháp thay thế cho việc xác định số lần mà một listener có thể được thử trước khi nó thất bại, bạn có thể định nghĩa thời điểm mà listener không còn được thử lại nữa. Điều này cho phép listener được thử bao nhiêu tuỳ thích trong một khoảng thời gian nhất định. Để định nghĩa khoảng thời gian mà một listener không còn được thử nữa, bạn hãy thêm một phương thức `retryUntil` vào class listener của bạn. Phương thức này sẽ trả về một instance `DateTime`:

    use DateTime;

    /**
     * Determine the time at which the listener should timeout.
     */
    public function retryUntil(): DateTime
    {
        return now()->addMinutes(5);
    }

<a name="dispatching-events"></a>
## Dispatching Event

Để gửi một event, bạn có thể gọi phương thức static `dispatch` trong event. Phương thức này được cung cấp trên event bởi trait `Illuminate\Foundation\Events\Dispatchable`. Mọi tham số được truyền cho phương thức `dispatch` sẽ được truyền cho hàm tạo của event:

    <?php

    namespace App\Http\Controllers;

    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;
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

Nếu bạn muốn gửi một event có điều kiện, bạn có thể sử dụng các phương thức `dispatchIf` và `dispatchUnless`:

    OrderShipped::dispatchIf($condition, $order);

    OrderShipped::dispatchUnless($condition, $order);

> [!NOTE]
> Khi testing, nếu bạn cần kiểm tra một số event được gửi đi mà không cần chạy đến các listener của các event. [Helper testing mặc định](#testing) của Laravel giúp việc này trở nên dễ dàng.

<a name="dispatching-events-after-database-transactions"></a>
### Dispatching Events After Database Transactions

Thỉnh thoảng, bạn có thể muốn hướng dẫn Laravel chỉ gửi event sau khi transaction đã được commit. Để làm như vậy, bạn có thể implement interface `ShouldDispatchAfterCommit` trên class event.

Interface này sẽ hướng dẫn Laravel không gửi event cho đến khi transaction hiện tại được commit. Nếu transaction bị lỗi, event sẽ bị hủy. Nếu không có transaction nào đang thực hiện khi event được gửi đi, thì event đó sẽ được gửi đi ngay lập tức:

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

<a name="event-subscribers"></a>
## Event Subscriber

<a name="writing-event-subscribers"></a>
### Viết Event Subscriber

Event subscriber là các class có thể đăng ký nhiều event từ trong chính class subscriber đó, cho phép bạn định nghĩa nhiều xử lý event trong cùng một class. Subscriber nên định nghĩa một phương thức `subscribe`, nó sẽ nhận vào một instance event dispatcher. Bạn có thể gọi phương thức `listen` trong dispatcher đó để đăng ký event listener:

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

Nếu các phương thức event listener của bạn được định nghĩa trong chính subscriber, bạn có thể thấy thuận tiện hơn khi trả về một mảng các event và các tên phương thức từ phương thức `subscribe` trong subscriber. Laravel sẽ tự động xác định tên class của subscriber khi đăng ký event listener:

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

<a name="registering-event-subscribers"></a>
### Đăng ký Event Subscriber

Sau khi đã tạo xong subscriber, bạn có thể đăng ký nó với event dispatcher. Bạn có thể đăng ký subscriber bằng cách sử dụng thuộc tính `$subscribe` trong `EventServiceProvider`. Ví dụ: hãy thêm `UserEventSubscriber` vào trong danh sách:

    <?php

    namespace App\Providers;

    use App\Listeners\UserEventSubscriber;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            // ...
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            UserEventSubscriber::class,
        ];
    }

<a name="testing"></a>
## Testing

Khi test code gửi event, bạn có thể muốn hướng dẫn Laravel không thực hiện listener của event, vì code của listener có thể được kiểm tra trực tiếp và riêng biệt với code gửi event. Tất nhiên, để kiểm tra listener, bạn có thể khởi tạo một instance listener và gọi phương thức `handle` trực tiếp trong bài test của bạn.

Bằng cách sử dụng phương thức `fake` của facade `Event`, bạn có thể ngăn listener được chạy, và chạy code đang được kiểm tra và sau đó xác nhận event nào đã được ứng dụng của bạn gửi bằng các phương thức `assertDispatched`, `assertNotDispatched` và `assertNothingDispatched`:

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

            // Assert an event was not dispatched...
            Event::assertNotDispatched(OrderFailedToShip::class);

            // Assert that no events were dispatched...
            Event::assertNothingDispatched();
        }
    }

Bạn có thể truyền một closure cho các phương thức `assertDispatched` hoặc `assertNotDispatched` để yêu cầu một event đã được gửi đi và pass qua "bài kiểm tra" đã cho. Nếu có ít nhất một event đã được gửi đi và pass qua bài kiểm tra đã cho thì yêu cầu sẽ thành công:

    Event::assertDispatched(function (OrderShipped $event) use ($order) {
        return $event->order->id === $order->id;
    });

Nếu bạn chỉ muốn yêu cầu listener của một event đang nhận một event nhất định, bạn có thể sử dụng phương thức `assertListening`:

    Event::assertListening(
        OrderShipped::class,
        SendShipmentNotification::class
    );

> [!WARNING]
> Sau khi gọi `Event::fake()`, sẽ không có listener event nào được thực thi. Vì vậy, nếu các bài kiểm tra của bạn sử dụng các model factory dựa trên các event, chẳng hạn như tạo UUID trong event `creating` của nodel, bạn nên gọi `Event::fake()` **sau** khi sử dụng các factory của bạn.

<a name="faking-a-subset-of-events"></a>
### Faking a Subset of Events

Nếu bạn chỉ muốn làm fake một listener event cho một tập hợp event cụ thể, bạn có thể truyền chúng cho phương thức `fake` hoặc `fakeFor`:

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
        $order->update([...]);
    }

Bạn có thể fake tất cả các event ngoại trừ một tập hợp các event được chỉ định bằng phương thức `except`:

    Event::fake()->except([
        OrderCreated::class,
    ]);

<a name="scoped-event-fakes"></a>
### Scoped Event Fakes

Nếu bạn chỉ muốn fake listener event cho một phần bài test của bạn, bạn có thể sử dụng phương thức `fakeFor`:

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

            // Events are dispatched as normal and observers will run ...
            $order->update([...]);
        }
    }
