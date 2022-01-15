# Broadcasting

- [Giới thiệu](#introduction)
    - [Cấu hình](#configuration)
    - [Yêu cầu driver](#driver-prerequisites)
- [Khái niệm tổng quan](#concept-overview)
    - [Sử dụng một application mẫu](#using-example-application)
- [Định nghĩa Broadcast Event](#defining-broadcast-events)
    - [Broadcast Name](#broadcast-name)
    - [Broadcast Data](#broadcast-data)
    - [Broadcast Queue](#broadcast-queue)
    - [Broadcast Condition](#broadcast-conditions)
- [Authorizing Channel](#authorizing-channels)
    - [Định nghĩa Authorization Route](#defining-authorization-routes)
    - [Định nghĩa Authorization Callback](#defining-authorization-callbacks)
    - [Định nghĩa Channel Class](#defining-channel-classes)
- [Broadcasting Event](#broadcasting-events)
    - [Only To Others](#only-to-others)
- [Nhận Broadcast](#receiving-broadcasts)
    - [Cài đặt Laravel Echo](#installing-laravel-echo)
    - [Listening cho Event](#listening-for-events)
    - [Rời một Channel](#leaving-a-channel)
    - [Namespaces](#namespaces)
- [Presence Channel](#presence-channels)
    - [Authorizing Presence Channel](#authorizing-presence-channels)
    - [Tham gia Presence Channel](#joining-presence-channels)
    - [Broadcasting tới Presence Channel](#broadcasting-to-presence-channels)
- [Client Event](#client-events)
- [Notification](#notifications)

<a name="introduction"></a>
## Giới thiệu

Trong nhiều application hoặc web hiện đại, WebSockets được sử dụng để thực hiện cập nhật trực tiếp lên giao diện người dùng theo thời gian thực. Sau khi dữ liệu được cập nhật lên máy chủ, thì một thông báo cũng sẽ được gửi qua kết nối WebSocket để được phía client xử lý. Điều này cung cấp một sự thay thế mạnh mẽ và hiệu quả hơn để liên tục đồng bộ cho application của bạn.

Để hỗ trợ bạn xây dựng các loại application này, Laravel giúp bạn dễ dàng "broadcast" [events](/docs/{{version}}/events) thông qua kết nối WebSocket. Broadcasting Laravel event cho phép bạn chia sẻ cùng tên event giữa các code backend ở phía máy chủ và code JavaScript ở phía client.

> {tip} Trước khi đi sâu vào event broadcasting, hãy đảm bảo là bạn đã đọc hết tất cả các tài liệu liên quan đến Laravel [events and listeners](/docs/{{version}}/events).

<a name="configuration"></a>
### Cấu hình

Tất cả các cấu hình event broadcasting của application đều được lưu trữ trong file cấu hình `config/broadcasting.php`. Mặc định, Laravel hỗ trợ một số broadcast driver: [Pusher Channels](https://pusher.com/channels), [Redis](/docs/{{version}}/redis), và driver `log` dành cho lúc phát triển và lúc gỡ lỗi. Ngoài ra, driver `null` cũng được cung cấp cho phép bạn tắt hoàn toàn broadcasting. Một số cấu hình mẫu cũng sẽ được cung cấp trong file cấu hình `config/broadcasting.php`.

#### Broadcast Service Provider

Trước khi broadcasting bất kỳ event nào, đầu tiên bạn sẽ cần phải đăng ký `App\Providers\BroadcastServiceProvider`. Trong một application Laravel mới, bạn chỉ cần bỏ comment provider này trong mảng `providers` của file cấu hình `config/app.php`. Provider này sẽ cho phép bạn đăng ký các route authorization broadcasting và các callback của chúng.

#### CSRF Token

[Laravel Echo](#installing-laravel-echo) sẽ cần quyền truy cập vào token CSRF của session hiện tại. nên vì thế bạn cần chắc chắn là `head` của HTML element của application của bạn đã định nghĩa một thẻ `meta` có chứa token CSRF:

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### Yêu cầu driver

#### Pusher Channels

Nếu bạn đang broadcasting các event của bạn thông qua [Pusher Channels](https://pusher.com/channels), bạn nên cài đặt SDK PHP của Pusher Channels bằng trình quản lý package Composer:

    composer require pusher/pusher-php-server "~3.0"

Tiếp theo, bạn nên cấu hình thông tin đăng nhập Channel của bạn trong file cấu hình `config/broadcasting.php`. Một ví dụ mẫu về cấu hình Channel đã có sẵn trong file này, cho phép bạn nhanh chóng chỉ định key, secret và application ID của Channel. Cấu hình `pusher` của file `config/broadcasting.php` cũng cho phép bạn chỉ định thêm các `options` được hỗ trợ bởi Channel, chẳng hạn như cluster:

    'options' => [
        'cluster' => 'eu',
        'useTLS' => true
    ],

Khi sử dụng Channel và [Laravel Echo](#installing-laravel-echo), bạn nên chỉ định `pusher` là broadcaster mà bạn muốn dùng, khi khởi tạo một instance Echo trong file `resources/assets/js/bootstrap.js` của bạn:

    import Echo from "laravel-echo"

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

#### Redis

Nếu bạn đang sử dụng broadcaster Redis, bạn nên cài đặt thư viện Predis:

    composer require predis/predis

Broadcaster Redis sẽ broadcast các tin nhắn bằng tính năng pub và sub của Redis; tuy nhiên, bạn sẽ cần phải kết nối nó với một máy chủ WebSocket để có thể nhận được tin nhắn từ Redis và broadcast chúng lên các channel WebSocket của bạn.

Khi broadcaster Redis publish một event, thì nó sẽ được publish trên các channel mà được chỉ định bởi event đó và payload của nó sẽ là một chuỗi JSON có chứa tên event, một payload `data` và một socket ID của event đó mà người dùng đã tạo ra (nếu có thể).

#### Socket.IO

Nếu bạn muốn kết nối broadcaster Redis với một máy chủ Socket.IO, bạn sẽ cần thêm thư viện client JavaScript Socket.IO vào trong application của bạn. Bạn có thể cài đặt nó thông qua NPM package manager:

    npm install --save socket.io-client

Tiếp theo, bạn sẽ cần khởi tạo Echo với connector `socket.io` và một `host`.

    import Echo from "laravel-echo"

    window.io = require('socket.io-client');

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

Cuối cùng, bạn sẽ cần chạy một máy chủ Socket.IO. Laravel không chứa việc triển khai một máy chủ Socket.IO; tuy nhiên, máy chủ Socket.IO do cộng đồng quản lý hiện đang được duy trì tại kho lưu trữ [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) trên GitHub .

#### Queue Prerequisites

Trước khi broadcasting các event, bạn cũng sẽ cần cấu hình và chạy một [queue listener](/docs/{{version}}/queues). Tất cả các broadcasting event sẽ được thực hiện thông qua các queued job để thời gian response application của bạn không bị ảnh hưởng quá nhiều bởi các event.

<a name="concept-overview"></a>
## Khái niệm tổng quan

Broadcasting event của Laravel cho phép bạn broadcast các event Laravel ở phía máy chủ của bạn tới các application ở JavaScript bên phía client bằng cách sử dụng các phương pháp tiếp cận dựa trên các driver WebSockets. Hiện tại, Laravel hỗ trợ [Pusher Channels](https://pusher.com/channels) và driver Redis. Các event có thể được sử dụng dễ dàng ở phía client bằng cách sử dụng package Javascript [Laravel Echo](#installing-laravel-echo).

Các event được broadcast qua các "channels", có thể chỉ định là công khai hoặc là riêng tư. Bất kỳ client nào truy cập vào application của bạn đều có thể đăng ký channel công khai mà không cần bất kỳ authentication hoặc authorization nào; tuy nhiên, để đăng ký channel private, người dùng phải được authentication và authorization để listen trên channel đó.

<a name="using-example-application"></a>
### Sử dụng một application mẫu

Trước khi đi sâu vào từng thành phần của event broadcasting, bạn có thể có cái nhìn tổng quan bằng cách sử dụng một cửa hàng thương mại điện tử làm ví dụ mẫu. Chúng ta sẽ không thảo luận chi tiết về cách cấu hình [Pusher Channels](https://pusher.com/channels) hoặc [Laravel Echo](#installing-laravel-echo), vì điều đó sẽ được thảo luận chi tiết trong các phần khác của tài liệu này.

Trong application của chúng ta, giả sử chúng ta có một trang cho phép người dùng xem trạng thái giao hàng của đơn hàng của họ. Chúng ta cũng giả sử rằng một event `ShippingStatusUpdated` sẽ được kích hoạt khi một shipping được cập nhật trạng thái bởi application:

    event(new ShippingStatusUpdated($update));

#### The `ShouldBroadcast` Interface

Khi người dùng đang xem một trong các đơn hàng của họ, chúng ta không muốn họ phải refresh trang để xem lại trạng thái của đơn hàng đó. Thay vào đó, chúng ta muốn broadcast các cập nhật trạng thái cho application của chúng ta khi chúng được tạo. Vì thế, chúng ta cần đánh dấu event `ShippingStatusUpdated` bằng interface `ShouldBroadcast`. Điều này sẽ hướng dẫn Laravel là tạo một broadcast event khi event đó được kích hoạt:

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        /**
         * Information about the shipping status update.
         *
         * @var string
         */
        public $update;
    }

Interface `ShouldBroadcast` yêu cầu event của chúng ta cần định nghĩa một phương thức là `broadcastOn`. Phương thức này sẽ chịu trách nhiệm trả về các channel mà event này sẽ broadcast trên đó. Một empty stub của phương thức này sẽ được định nghĩa sẵn cho chúng ta trên các class event đã được tạo ra, vì vậy chúng ta sẽ chỉ cần điền các thông tin chi tiết về nó. Chúng ta muốn chỉ duy nhất người đã tạo đơn hàng này mới có thể xem trạng thái, vì vậy chúng ta sẽ cần broadcast event này trên một channel private được gắn với đơn đặt hàng:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### Authorizing Channels

Hãy nhớ rằng, người dùng phải có phép thì mới có thể listen trên các channel private. Chúng ta có thể định nghĩa các quy tắc authorization cho channel này trong file `routes/channels.php`. Trong ví dụ này, chúng ta cần kiểm tra rằng bất kỳ người dùng nào đang cố gắng listen trên channel private `order.1` này có phải là người đã tạo ra đơn đặt hàng hay không:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Phương thức `channel` này chấp nhận hai tham số: một là tên của channel và một là callback trả về `true` hoặc `false` cho biết người dùng đó có được phép listen trên channel hay không.

Tất cả các authorization callback này đều nhận vào tham số đầu tiên là người dùng hiện tại đang được authenticate và tham số tiếp theo là tham số đại diện cho biến được truyền theo tên của channel. Trong ví dụ này, chúng ta đang sử dụng biến `{orderId}` để lấy ra phần "ID" trong tên của channel.

#### Listening For Event Broadcasts

Tiếp theo, tất cả những gì còn lại là listen event trong JavaScript của chúng ta. Chúng ta có thể làm điều này bằng cách sử dụng Laravel Echo. Đầu tiên, chúng ta sẽ sử dụng phương thức `private` để đăng ký channel private. Sau đó, chúng ta có thể sử dụng phương thức `listen` để listen event `ShippingStatusUpdated`. Mặc định, tất cả các thuộc tính công khai của event sẽ được đưa vào trong broadcast event:

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## Định nghĩa Broadcast Event

Để thông báo cho Laravel rằng một event sẽ được broadcast, hãy implement interface`Illuminate\Contracts\Broadcasting\ShouldBroadcast` trong class event. Mặc định, interface này sẽ được import vào tất cả các class event mà được tạo bởi framework để bạn có thể dễ dàng thêm nó vào bất kỳ event nào của bạn.

Interface `ShouldBroadcast` yêu cầu bạn implement một phương thức: `broadcastOn`. Phương thức `broadcastOn` sẽ trả về tên một channel hoặc một mảng tên các channel mà event sẽ được broadcast trên đó. Các channel phải là các instance của `Channel`, `PrivateChannel`, hoặc `PresenceChannel`. Các instance của `Channel` là đại diện cho các channel public mà bất kỳ người dùng nào cũng có thể vào, trong khi `PrivateChannels` và `PresenceChannels` là đại diện cho các channel private yêu cầu [channel authorization](#authorizing-channels):

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should broadcast on.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

Sau đó, bạn chỉ cần [kích hoạt event](/docs/{{version}}/events) như bình thường. Khi event đó đã được kích hoạt, [queued job](/docs/{{version}}/queues) sẽ tự động broadcast event đó qua driver broadcast mà chúng ta đã định nghĩa.

<a name="broadcast-name"></a>
### Broadcast Name

Mặc định, Laravel sẽ broadcast event bằng tên class của event. Tuy nhiên, bạn có thể tùy chỉnh tên của broadcast bằng cách định nghĩa phương thức `broadcastAs` trong event:

    /**
     * The event's broadcast name.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

Nếu bạn tùy chỉnh tên broadcast bằng phương thức `broadcastAs`, bạn nên đảm bảo rằng đã đăng ký listener của bạn với một ký tự `.` ở đầu. Điều này sẽ hướng dẫn Echo không thêm namespace application cho event của bạn:

    .listen('.server.created', function (e) {
        ....
    });

<a name="broadcast-data"></a>
### Broadcast Data

Khi một event đã được broadcast, thì tất cả các thuộc tính `public` của nó sẽ tự động serialize và broadcast dưới dạng payload của một event, cho phép bạn truy cập bất kỳ dữ liệu công khai nào từ JavaScript của bạn. Vì thế, ví dụ, nếu event của bạn có một thuộc tính `$user` công khai là một model Eloquent, thì payload của broadcast event sẽ là:

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

Tuy nhiên, nếu bạn muốn có quyền kiểm soát chi tiết hơn đối với payload broadcast của bạn, bạn có thể thêm một phương thức `broadcastWith` vào event của bạn. Phương thức này sẽ trả về mảng dữ liệu mà bạn muốn broadcast dưới dạng payload event:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### Broadcast Queue

Mặc định, mỗi broadcast event sẽ được đặt trên một queue mặc định với một kết nối queue mặc định được định nghĩa trong file cấu hình `queue.php` của bạn. Bạn có thể tùy chỉnh queue mà được sử dụng bởi broadcaster bằng cách định nghĩa thêm một thuộc tính `broadcastQueue` trong class event của bạn. Thuộc tính này sẽ định nghĩa tên queue mà bạn muốn sử dụng khi broadcasting:

    /**
     * The name of the queue on which to place the event.
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

Nếu bạn muốn broadcast event của bạn bằng cách sử dụng queue `sync` thay vì driver queue mặc định, bạn có thể implement interface `ShouldBroadcastNow` thay vì `ShouldBroadcast`:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class ShippingStatusUpdated implements ShouldBroadcastNow
    {
        //
    }
<a name="broadcast-conditions"></a>
### Broadcast Conditions

Thỉnh thoảng bạn cũng có thể muốn broadcast event của bạn trong một điều kiện nhất định. Bạn có thể định nghĩa các điều kiện này bằng cách thêm một phương thức `broadcastWhen` vào trong class event của bạn:

    /**
     * Determine if this event should broadcast.
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->value > 100;
    }

<a name="authorizing-channels"></a>
## Authorizing Channels

Các channel private sẽ yêu cầu bạn authorize rằng người dùng hiện tại đang được authenticate có thể có listen trên channel private này hay không. Điều này có thể được thực hiện bằng cách tạo một HTTP request đến application Laravel của bạn với tên channel và sau đó application của bạn có thể xác định xem người dùng đó có thể listen trên channel đó hay không. Khi sử dụng [Laravel Echo](#installing-laravel-echo), thì HTTP request authorize này sẽ được tạo ra tự động; tuy nhiên, bạn sẽ cần định nghĩa thêm các route để respond lại các request này.

<a name="defining-authorization-routes"></a>
### Định nghĩa Authorization Route

Rất may, Laravel đã giúp việc định nghĩa các route này một cách dễ dàng. Trong class `BroadcastServiceProvider` mà đi cùng với application Laravel, bạn sẽ thấy nó gọi đến một phương thức `Broadcast::routes`. Phương thức này sẽ đăng ký route `/broadcasting/auth` để xử lý các authorization request:

    Broadcast::routes();

Phương thức `Broadcast::routes` sẽ tự động đăng ký route của nó vào trong group middleware `web`; tuy nhiên, bạn có thể truyền một mảng các thuộc tính của route đó vào phương thức này nếu bạn muốn tùy chỉnh các thuộc tính đó:

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### Định nghĩa Authorization Callback

Tiếp theo, chúng ta cần định nghĩa các logic sẽ được thực hiện để authorization channel. Điều này sẽ được thực hiện trong file `routes/channels.php` đi kèm với application. Trong file này, bạn có thể sử dụng phương thức `Broadcast::channel` để đăng ký các callback authorization channel:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Phương thức `channel` chấp nhận hai tham số: một là tên của channel và một là callback trả về `true` hoặc `false` cho biết người dùng đó có được phép listen trên channel này hay không.

Tất cả các callback authorization đều nhận vào tham số đầu tiên là người dùng hiện tại đang được authenticate và tham số tiếp theo là một tham số đại diện. Trong ví dụ này, chúng ta đang sử dụng biến `{orderId}` để chỉ ra phần "ID" của tên channel.

#### Authorization Callback Model Binding

Giống như các route HTTP, các route channel cũng có thể tận dụng các [route model binding](/docs/{{version}}/routing#route-model-binding). Ví dụ, thay vì nhận một chuỗi ID hoặc một chuỗi số thứ tự, bạn có thể yêu cầu một instance model `Order`:

    use App\Order;

    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

<a name="defining-channel-classes"></a>
### Định nghĩa Channel Class

Nếu ứng dụng của bạn sử dụng nhiều channel khác nhau, thì file `routes/channels.php` của bạn có thể trở nên rất cồng kềnh. Vì vậy, thay vì sử dụng Closure để cấp quyền cho các channel, bạn có thể sử dụng các class channel. Để tạo một class channel mới, hãy sử dụng lệnh Artisan `make:channel`. Lệnh này sẽ lưu một class channel mới vào trong thư mục `App/Broadcasting`.

    php artisan make:channel OrderChannel

Tiếp theo, đăng ký channel của bạn vào trong file `routes/channels.php`:

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('order.{order}', OrderChannel::class);

Cuối cùng, bạn có thể viết các logic cấp quyền cho channel của bạn vào trong phương thức `join` của class channel. Phương thức `join` sẽ chứa cùng một logic với code mà bạn thường viết trong Closure cấp quyền channel của bạn. Tất nhiên, bạn cũng có thể tận dụng lợi thế của liên kết model channel:

    <?php

    namespace App\Broadcasting;

    use App\User;
    use App\Order;

    class OrderChannel
    {
        /**
         * Create a new channel instance.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Authenticate the user's access to the channel.
         *
         * @param  \App\User  $user
         * @param  \App\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

> {tip} Giống như nhiều class khác trong Laravel, các class channel sẽ tự động được resolve bởi [service container](/docs/{{version}}/container). Vì vậy, bạn có thể khai báo bất kỳ phụ thuộc nào mà channel của bạn cần trong hàm tạo của nó.

<a name="broadcasting-events"></a>
## Broadcasting Event

Khi bạn đã định nghĩa một event và đánh dấu nó bằng một interface `ShouldBroadcast`, bạn chỉ cần kích hoạt event đó bằng hàm `event`. Dispatcher của event sẽ hiểu được rằng event đó đã được đánh dấu bằng một interface `ShouldBroadcast` nên nó sẽ tạo một queue event để broadcasting:

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### Only To Others

Khi xây dựng một application sử dụng event broadcasting, bạn có thể thay thế hàm `event` bằng hàm `broadcast`. Giống như hàm `event`, hàm `broadcast` cũng gửi event đến các listener bên phía máy chủ của bạn:

    broadcast(new ShippingStatusUpdated($update));

Tuy nhiên, hàm `broadcast` cũng có phương thức `toOthers` cho phép bạn loại người dùng hiện tại ra khỏi danh sách người nhận của broadcast:

    broadcast(new ShippingStatusUpdated($update))->toOthers();

Để hiểu rõ hơn lý do mà bạn có thể muốn sử dụng phương thức `toOthers`, hãy tưởng tượng một application quản lý danh sách các task trong đó người dùng có thể tạo ra một task mới bằng cách nhập tên task. Để tạo một task mới, application của bạn có thể tạo một request đến route `/task` để broadcasts tạo task và trả về một JSON là một task mới. Khi JavaScript của bạn nhận được phản hồi từ route, nó có thể trực tiếp chèn task mới này vào danh sách các task đã tồn tại như sau:

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

Tuy nhiên, hãy nhớ rằng chúng ta đang broadcast một event tạo task. Nếu JavaScript của bạn đang listening event này, để thêm task mới vào danh sách task, thì bạn có thể có các task trùng lặp trong danh sách của bạn: một là từ route và một là từ broadcast. Bạn có thể giải quyết điều này bằng cách sử dụng phương thức `toOthers` để hướng dẫn broadcaster không broadcast event tới người dùng hiện tại.

> {note} Event của bạn phải sử dụng trait `Illuminate\Broadcasting\InteractsWithSockets` để gọi phương thức `toOthers`.

#### Cấu hình

Khi bạn khởi tạo một instance Laravel Echo, một ID socket cũng sẽ được khởi tạo. Nếu bạn đang sử dụng [Vue](https://vuejs.org) và [Axios](https://github.com/mzabriskie/axios), thì ID socket đó sẽ được tự động đính kèm vào mọi request gửi đi dưới dạng `X-Socket-ID`. Sau đó, khi bạn gọi phương thức `toOthers`, Laravel sẽ lấy ID socket từ header đó và hướng dẫn broadcaster sẽ không broadcast đến bất kỳ kết nối nào mà trùng với ID socket đó.

Nếu bạn không sử dụng Vue và Axios, bạn sẽ cần phải tự cấu hình JavaScript của bạn để gửi header `X-Socket-ID`. Bạn có thể lấy ra ID socket bằng phương thức `Echo.socketId`:

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## Nhận Broadcast

<a name="installing-laravel-echo"></a>
### Cài đặt Laravel Echo

Laravel Echo là một thư viện JavaScript khiến cho việc theo dõi các channel và listen các event được broadcast bởi Laravel một cách dễ dàng hơn. Bạn có thể cài đặt Echo thông qua trình quản lý package NPM. Trong ví dụ này, chúng ta cũng sẽ cài đặt package `pusher-js` vì chúng ta đang sử dụng broadcaster Pusher Channel:

    npm install --save laravel-echo pusher-js

Khi Echo đã được cài đặt, bạn đã sẵn sàng tạo một instance Echo mới trong JavaScript của application. Một nơi tuyệt vời để làm điều này là ở dưới cùng file `resources/assets/js/bootstrap.js` được đi kèm trong framework Laravel:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

Khi tạo một instance Echo sử dụng kết nối `pusher`, bạn cũng có thể chỉ định một `cluster` và kết nối đó có phải được thực hiện thông qua TLS hay không (mặc định, khi `forceTLS` là `false`, một kết nối không phải là TLS sẽ được tạo nếu như trang hiện tại của bạn đang được load dưới HTTP hoặc nếu kết nối TLS thất bại):

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        cluster: 'eu',
        forceTLS: true
    });

<a name="listening-for-events"></a>
### Listening cho Event

Khi bạn đã cài đặt và khởi tạo Echo xong, bạn đã sẵn sàng để bắt đầu listening các event broadcast. Đầu tiên, hãy sử dụng phương thức `channel` để lấy ra một instance của một channel, sau đó gọi phương thức `listen` để listen một event cụ thể:

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

Nếu bạn muốn listen các event trên một private channel, hãy sử dụng phương thức `private`. Bạn có thể khai báo nhiều phương thức `listen` để listen nhiều event trên một channel:

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="leaving-a-channel"></a>
### Rời một Channel

Để rời khỏi một channel, bạn có thể gọi phương thức `leave` trên instance Echo của bạn:

    Echo.leave('orders');

<a name="namespaces"></a>
### Namespaces

Bạn có thể đã nhận thấy trong các ví dụ ở trên, chúng ta không chỉ định namespace cho các class event. Điều này là do Echo đã tự động giả định các event được lưu trong namespace `App\Events`. Tuy nhiên, bạn có thể cấu hình namespace khi bạn khởi tạo Echo bằng cách truyền vào một tùy chọn cấu hình `namespace`:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        namespace: 'App.Other.Namespace'
    });

Ngoài ra, bạn có thể thêm tiền tố cho các class event với một dấu `.` khi theo dõi các event bằng Echo. Điều này sẽ cho phép bạn chỉ định tên class:

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## Presence Channel

Các presence channel được xây dựng dựa trên tính bảo mật của các private channel đồng thời thêm các chức năng về những người đã theo dõi channel. Điều này giúp dễ dàng xây dựng các tính năng mạnh mẽ cho application chẳng hạn như thông báo cho người dùng biết khi một người dùng khác đang xem cùng một trang.

<a name="authorizing-presence-channels"></a>
### Authorizing Presence Channel

Tất cả các presence channel cũng là các private channel; do đó, người dùng phải được [authorized để truy cập đến nó](#authorizing-channels). Tuy nhiên, khi định nghĩa các authorization callback cho các presence channel, bạn sẽ không trả về `true` nếu người dùng đã được authorize tham gia channel. Thay vào đó, bạn nên trả về một mảng dữ liệu về người dùng đó.

Dữ liệu được trả về bởi authorization callback cũng sẽ được cung cấp cho những người khác đang listen event trong presence channel của JavaScript của bạn. Nếu người dùng không được phép tham gia presence channel, bạn nên trả về `false` hoặc `null`:

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### Tham gia Presence Channel

Để tham gia một presence channel, bạn có thể sử dụng phương thức `join` của Echo. Phương thức `join` sẽ trả về một implementation `PresenceChannel`, cùng với việc thêm phương thức `listen`, cho phép bạn theo dõi các event `here`, `joining`, và `leaving`.

    Echo.join(`chat.${roomId}`)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

Callback `here` sẽ được thực hiện ngay sau khi kết nối thành công đến channel và sẽ nhận về một mảng chứa thông tin của tất cả các người dùng đang đăng ký channel. Phương thức `joining` sẽ được thực thi khi một người dùng mới tham gia vào channel, trong khi phương thức `leaving` sẽ được thực thi khi một người dùng rời khỏi channel.

<a name="broadcasting-to-presence-channels"></a>
### Broadcasting tới Presence Channel

Các presence channel có thể nhận các event giống như các public hoặc private channel. Ví dụ như về một chatroom, chúng ta có thể muốn broadcast các event `NewMessage` lên một room. Để làm như vậy, chúng ta sẽ trả về một instance của `PresenceChannel` từ phương thức `broadcastOn` của event:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

Giống như các event public hoặc private, các event của presence channel có thể được broadcast bằng hàm `broadcast`. Cũng như các event khác, bạn có thể sử dụng phương thức `toOthers` để loại bỏ người dùng hiện tại ra khỏi việc broadcast:

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

Bạn có thể listen event tham gia thông qua phương thức `listen` của Echo:

    Echo.join(`chat.${roomId}`)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="client-events"></a>
## Client Event

> {tip} Khi sử dụng [Pusher Channels](https://pusher.com/channels), bạn phải bật tùy chọn "Client Events" trong phần "App Settings" của [bảng điều khiển ứng dụng](https://dashboard.pusher.com/) để gửi các client event.

Thỉnh thoảng bạn có thể muốn broadcast một event cho những client được kết nối khác mà không cần gọi application Laravel của bạn. Điều này có thể đặc biệt hữu ích cho những việc như thông báo "đang gõ", bạn muốn thông báo cho người dùng application của bạn rằng có một người dùng khác đang gõ một tin nhắn trên màn hình.

Để broadcast các client event, bạn có thể sử dụng phương thức `whisper` của Echo:

    Echo.private('chat')
        .whisper('typing', {
            name: this.user.name
        });

Để listen các client event, bạn có thể sử dụng phương thức `listenForWhisper`:

    Echo.private('chat')
        .listenForWhisper('typing', (e) => {
            console.log(e.name);
        });

<a name="notifications"></a>
## Notification

Bằng cách kết nối event broadcasting với [notifications](/docs/{{version}}/notifications), JavaScript của bạn có thể nhận được thông báo mới khi chúng xảy ra mà không cần refresh trang. Trước tiên, hãy nhớ đọc tài liệu về việc sử dụng [the broadcast notification channel](/docs/{{version}}/notifications#broadcast-notifications).

Khi bạn đã cấu hình thông báo sử dụng broadcast channel, bạn có thể listen các broadcast event bằng phương thức `notification` của Echo. Hãy nhớ rằng, tên channel phải khớp với tên class nhận thông báo:

    Echo.private(`App.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

Trong ví dụ trên, tất cả các thông báo được gửi đến instance `App\User` thông qua channel `broadcast` sẽ được nhận được thông qua hàm callback. Một callback authorization cho channel `App.User.{id}` sẽ có sẵn trong `BroadcastServiceProvider` đi kèm với framework Laravel.
