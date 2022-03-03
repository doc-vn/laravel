# Events

- [Giới thiệu](#introduction)
- [Đăng ký Event và Listener](#registering-events-and-listeners)
    - [Tạo Event và Listener](#generating-events-and-listeners)
    - [Đăng ký Event thủ công](#manually-registering-events)
- [Khai báo Event](#defining-events)
- [Khai báo Listener](#defining-listeners)
- [Queued Event Listener](#queued-event-listeners)
    - [Truy cập Queue thủ công](#manually-accessing-the-queue)
    - [Xử lý Failed Job](#handling-failed-jobs)
- [Dispatching Event](#dispatching-events)
- [Event Subscriber](#event-subscribers)
    - [Viết Event Subscriber](#writing-event-subscribers)
    - [Đăng ký Event Subscriber](#registering-event-subscribers)

<a name="introduction"></a>
## Giới thiệu

Các event của Laravel cung cấp một observer implementation đơn giản, cho phép bạn subscribe và listen các event khác nhau xảy ra trong application của bạn. Các class event thường được lưu trong thư mục `app/Events`, còn các listen của các event đó được lưu trong thư mục `app/Listeners`. Đừng lo lắng nếu bạn không thấy các thư mục này trong application của bạn, vì chúng sẽ được tạo khi bạn tạo các event và listener bằng các lệnh của Artisan console.

Các event đóng vai trò là một cách tuyệt vời để tách các khía cạnh khác nhau của application, vì một event có thể có nhiều listener mà không phụ thuộc vào lẫn nhau. Ví dụ: bạn có thể muốn gửi thông báo đến Slack cho người dùng của bạn mỗi khi đơn hàng đã được giao. Thay vì ghép code xử lý đơn đặt hàng với code thông báo của Slack, bạn có thể đưa ra một event `OrderShipped`, mà listener có thể nhận và chuyển nó thành một thông báo đến Slack.

<a name="registering-events-and-listeners"></a>
## Đăng ký Event và Listener

`EventServiceProvider` đi kèm trong application Laravel cung cấp một cách đăng ký dễ dàng cho tất cả các listener event trong application của bạn. Thuộc tính `listen` chứa một mảng gồm các event (là các key) và listener (là các giá trị). Bạn có thể thêm nhiều event vào mảng này khi application của bạn yêu cầu. Ví dụ: hãy thêm một event `OrderShipped` như sau:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### Tạo Event và Listener

Tất nhiên, việc tạo bằng tay các file cho các event và listener này là rất công kềnh. Thay vào đó, hãy thêm listener và event của nó vào trong `EventServiceProvider` của bạn và sử dụng lệnh `event:generate`. Lệnh này sẽ tạo ra bất kỳ các event hoặc các listener nào được liệt kê trong mảng `EventServiceProvider`. Các event và listener đã được tạo thì sẽ không bị ảnh hưởng:

    php artisan event:generate

<a name="manually-registering-events"></a>
### Đăng ký Event thủ công

Thông thường, các event nên được đăng ký thông qua `EventServiceProvider` vào mảng `$listen`; tuy nhiên, bạn cũng có thể đăng ký các event dựa trên Closure bằng cách đưa nó vào trong phương thức `boot` của `EventServiceProvider`:

    /**
     * Register any other events for your application.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### Wildcard Event Listeners

Bạn thậm chí có thể đăng ký listener bằng cách sử dụng ký tự đại diện `*` làm tham số, cho phép bạn nhận được nhiều event trên cùng một listener. Và nó nhận tên event là tham số đầu tiên và toàn bộ mảng dữ liệu event là tham số thứ hai:

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="defining-events"></a>
## Khai báo Event

Một event class là một data container chứa các thông tin liên quan đến event. Ví dụ: giả sử event `OrderShipped` của chúng ta sẽ nhận một đối tượng [Eloquent ORM](/docs/{{version}}/eloquent):

    <?php

    namespace App\Events;

    use App\Order;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use SerializesModels;

        public $order;

        /**
         * Create a new event instance.
         *
         * @param  \App\Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

Như bạn có thể thấy, event class này không chứa code logic. Nó là một container chứa instance `Order` đã được mua. Trait `SerializesModels` được sử dụng trong event này để khôi phục lại bất kỳ model Eloquent nào nếu nó đã bị chuyển đổi bằng hàm `serialize` của PHP.

<a name="defining-listeners"></a>
## Khai báo Listener

Tiếp theo, chúng ta hãy xem một listener mẫu cho một event. Listener của event sẽ nhận vào một instance event trong phương thức `handle`. Lệnh `event:generate` sẽ tự động import class event và khai báo nó vào trong phương thức `handle`. Trong phương thức `handle`, bạn có thể thực hiện bất kỳ hành động nào cần thiết để xử lý event:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Access the order using $event->order...
        }
    }

> {tip} Listener event của bạn cũng có thể khai báo bất kỳ sự phụ thuộc nào cần thiết ở trong hàm khởi tạo. Tất cả các listener event sẽ được resolve thông qua [service container](/docs/{{version}}/container), do đó, các phụ thuộc cũng sẽ được tự động thêm vào.

#### Stopping The Propagation Of An Event

Thỉnh thoảng, bạn có thể muốn ngừng việc truyền một event đến những listener khác. Bạn có thể làm như vậy bằng cách trả về `false` từ phương thức` handle` của listener của bạn.

<a name="queued-event-listeners"></a>
## Queued Event Listener

Queueing listener có thể có lợi nếu listener của bạn thực hiện một nhiệm vụ mà không cần phải phản hồi ngay lập tức như việc gửi e-mail hoặc tạo một HTTP request. Trước khi bắt đầu với queued listener, hãy đảm bảo là bạn đã [cấu hình queue](/docs/{{version}}/queues) và chạy một queue listener trên server hoặc môi trường develop của bạn.

Để khai báo một listener sẽ được queue, hãy thêm interface `ShouldQueue` vào class listener. Listener được tạo bởi lệnh Artisan `event:generate` sẽ khai báo sẵn interface này và import nó vào namespace hiện tại, vì vậy bạn có thể sử dụng nó ngay lập tức:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

Và chỉ có thế! Bây giờ, khi listener này được gọi trong một event, nó sẽ tự động được queue bởi event dispatcher bằng cách sử dụng [queue system](/docs/{{version}}/queues) của Laravel. Nếu không có ngoại lệ nào được đưa ra khi listener được thực thi bởi queue, thì queue job đó sẽ tự động bị xóa sau khi xử lý xong.

#### Customizing The Queue Connection & Queue Name

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

<a name="manually-accessing-the-queue"></a>
### Truy cập Queue thủ công

Nếu bạn cần tự truy cập các phương thức `delete` và `release` của queue job, bạn có thể làm như vậy bằng cách sử dụng trait `Illuminate\Queue\InteractsWithQueue`. Trait này sẽ được mặc định import sẵn vào trong các listener nếu nó được tạo bằng lệnh artisan và cung cấp quyền truy cập vào các phương thức `delete` và `release`:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="handling-failed-jobs"></a>
### Xử lý Failed Job

Thỉnh thoảng queue của event listener của bạn có thể bị thất bại. Nếu queued listener chạy vượt quá số lần thử tối đa được định nghĩa bởi queue worker của bạn, phương thức `failed` sẽ được gọi trong listener của bạn. Phương thức `failed` nhận vào instance event và ngoại lệ gây ra lỗi:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            //
        }

        /**
         * Handle a job failure.
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Exception  $exception
         * @return void
         */
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="dispatching-events"></a>
## Dispatching Event

Để gửi một event, bạn có thể truyền một instance của event cho helper `event`. Helper này sẽ gửi event đó đến tất cả những listener đã đăng ký với nó. Vì helper `event` là global helper, nên bạn có thể gọi nó bất kỳ đâu trong application của bạn:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // Order shipment logic...

            event(new OrderShipped($order));
        }
    }

> {tip} Khi testing, nếu bạn cần kiểm tra một số event được gửi đi mà không cần chạy đến các listener của các event. [built-in testing helpers](/docs/{{version}}/mocking#event-fake) có thể làm điều đó trở lên dễ dàng.

<a name="event-subscribers"></a>
## Event Subscriber

<a name="writing-event-subscribers"></a>
### Viết Event Subscriber

Event subscriber là các class có thể đăng ký nhiều event từ trong chính class đó, cho phép bạn định nghĩa nhiều xử lý event trong cùng một class. Subscriber nên định nghĩa một phương thức `subscribe`, nó sẽ nhận vào một instance event dispatcher. Bạn có thể gọi phương thức `listen` trong dispatcher đó để đăng ký event listener:

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}

        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  \Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }
    }

<a name="registering-event-subscribers"></a>
### Đăng ký Event Subscriber

Sau khi đã tạo xong subscriber, bạn có thể đăng ký nó với event dispatcher. Bạn có thể đăng ký subscriber bằng cách sử dụng thuộc tính `$subscribe` trong `EventServiceProvider`. Ví dụ: hãy thêm `UserEventSubscriber` vào trong danh sách:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
