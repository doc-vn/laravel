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

Trong nhiều application web hiện đại, WebSockets được sử dụng để thực hiện cập nhật trực tiếp giao diện người dùng theo thời gian thực. Khi một số dữ liệu được cập nhật trên máy chủ, một thông báo thường được gửi qua kết nối WebSocket để được client xử lý. Điều này cung cấp một sự thay thế mạnh mẽ, hiệu quả hơn để liên tục polling cho application của bạn để thay đổi.

Để hỗ trợ bạn xây dựng các loại application này, Laravel giúp bạn dễ dàng "broadcast" [events](/docs/{{version}}/events) của bạn qua kết nối WebSocket. Broadcasting Laravel event của bạn cho phép bạn chia sẻ cùng tên event giữa code phía máy chủ và application JavaScript phía client của bạn.

> {tip} Trước khi đi sâu vào event broadcasting, hãy đảm bảo bạn đã đọc tất cả các tài liệu liên quan đến Laravel [events and listeners](/docs/{{version}}/events).

<a name="configuration"></a>
### Cấu hình

Tất cả các cấu hình event broadcasting của application của bạn được lưu trữ trong file cấu hình `config/broadcasting.php`. Mặc định, Laravel hỗ trợ một số broadcast driver: [Pusher Channels](https://pusher.com/channels), [Redis](/docs/{{version}}/redis), và driver `log` dành cho lúc phát triển và gỡ lỗi. Ngoài ra, driver `null` cũng được cung cấp cho phép bạn tắt hoàn toàn broadcasting. Một số cấu hình ví dụ đều được cung cấp cho mỗi driver này ở trong file cấu hình `config/broadcasting.php`.

#### Broadcast Service Provider

Trước khi broadcasting bất kỳ event nào, trước tiên bạn sẽ cần phải đăng ký `App\Providers\BroadcastServiceProvider`. Trong các application Laravel mới, bạn chỉ cần bỏ comment provider này trong mảng `providers` của file cấu hình `config/app.php` của bạn. Provider này sẽ cho phép bạn đăng ký các route authorization broadcasting và callback.

#### CSRF Token

[Laravel Echo](#installing-laravel-echo) sẽ cần quyền truy cập vào token CSRF của session hiện tại. nên vì thế bạn cần chắc chắn rằng `head` HTML element của application của bạn đã định nghĩa một thẻ `meta` có chứa token CSRF:

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### Yêu cầu driver

#### Pusher Channels

Nếu bạn đang broadcasting các event của bạn qua [Pusher Channels](https://pusher.com/channels), bạn nên cài đặt SDK PHP của Pusher Channels bằng trình quản lý package Composer:

    composer require pusher/pusher-php-server "~3.0"

Tiếp theo, bạn nên cấu hình thông tin đăng nhập Channel của bạn trong file cấu hình `config/broadcasting.php`. Một ví dụ về cấu hình Channel đã được có sẵn trong file này, cho phép bạn nhanh chóng chỉ định key, secret và application ID của Channel. Cấu hình `pusher` của file `config/broadcasting.php` cũng cho phép bạn chỉ định các `options` bổ sung được hỗ trợ bởi Channel, chẳng hạn như cluster:

    'options' => [
        'cluster' => 'eu',
        'useTLS' => true
    ],

Khi sử dụng Channel và [Laravel Echo](#installing-laravel-echo), bạn nên chỉ định `pusher` là broadcaster mong muốn của bạn khi khởi tạo instance Echo trong file `resources/assets/js/bootstrap.js` của bạn:

    import Echo from "laravel-echo"

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

#### Redis

Nếu bạn đang sử dụng broadcaster Redis, bạn nên cài đặt thư viện Predis:

    composer require predis/predis

Broadcaster Redis sẽ broadcast các tin nhắn bằng tính năng pub / sub của Redis; tuy nhiên, bạn sẽ cần pair nó với một máy chủ WebSocket có thể nhận tin nhắn từ Redis và broadcast chúng lên các channel WebSocket của bạn.

Khi broadcaster Redis publish một event, nó sẽ được publish trên các tên channel được chỉ định của event và payload sẽ là một chuỗi được mã hóa dưới dạng JSON chứa tên event, một payload `data` và người dùng đã tạo socket ID của event (nếu có thể).

#### Socket.IO

Nếu bạn định pair broadcaster Redis với một máy chủ Socket.IO, bạn sẽ cần khai báo thư viện client JavaScript của Socket.IO trong `head` HTML element của application. Khi máy chủ Socket.IO được khởi động, nó sẽ tự động hiển thị thư viện JavaScript của client tại một URL. Ví dụ: nếu bạn đang chạy máy chủ Socket.IO trên cùng tên miền với application web của bạn, bạn có thể truy cập thư viện client như sau:

    <script src="//{{ Request::getHost() }}:6001/socket.io/socket.io.js"></script>

Tiếp theo, bạn sẽ cần khởi tạo Echo với connector `socket.io` và một `host`.

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

Cuối cùng, bạn sẽ cần chạy một máy chủ Socket.IO tương thích. Laravel không chứa việc triển khai máy chủ Socket.IO; tuy nhiên, máy chủ Socket.IO do cộng đồng quản lý hiện đang được duy trì tại kho lưu trữ [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) của GitHub .

#### Queue Prerequisites

Trước khi broadcasting các event, bạn cũng sẽ cần cấu hình và chạy một [queue listener](/docs/{{version}}/queues). Tất cả broadcasting event được thực hiện thông qua các queued job để thời gian response của application của bạn không bị ảnh hưởng quá nghiêm trọng.

<a name="concept-overview"></a>
## Khái niệm tổng quan

Broadcasting event của Laravel cho phép bạn broadcast các event Laravel ở phía máy chủ của bạn tới application JavaScript ở phía client bằng cách sử dụng phương pháp tiếp cận dựa trên driver WebSockets. Hiện tại, Laravel ship với [Pusher Channels](https://pusher.com/channels) và driver Redis. Các event có thể được sử dụng dễ dàng ở phía client bằng cách sử dụng package Javascript [Laravel Echo](#installing-laravel-echo).

Các event được broadcast "channels", có thể được chỉ định là công khai hoặc private tư. Bất kỳ khách nào truy cập vào application của bạn đều có thể đăng ký channel công khai mà không cần bất kỳ authentication hoặc authorization nào; tuy nhiên, để đăng ký channel private, người dùng phải được authentication và authorization để listen trên channel đó.

<a name="using-example-application"></a>
### Sử dụng một application mẫu

Trước khi đi sâu vào từng thành phần của event broadcasting, bạn có thể có cái nhìn tônge quan bằng cách sử dụng một cửa hàng thương mại điện tử làm ví dụ. Chúng ta sẽ không thảo luận chi tiết về cách cấu hình [Pusher Channels](https://pusher.com/channels) hoặc [Laravel Echo](#installing-laravel-echo) vì điều đó sẽ được thảo luận chi tiết trong các phần khác của tài liệu này.

Trong application của chúng ta, giả sử chúng ta có một trang cho phép người dùng xem trạng thái giao hàng cho đơn hàng của họ. Chúng ta cũng giả sử rằng một event `ShippingStatusUpdated` được kích hoạt khi một shipping status update đã được xử lý bởi application:

    event(new ShippingStatusUpdated($update));

#### The `ShouldBroadcast` Interface

Khi người dùng đang xem một trong các đơn đặt hàng của họ, chúng ta không muốn họ phải refresh trang để xem cập nhật trạng thái. Thay vào đó, chúng ta muốn broadcast các bản cập nhật cho application khi chúng được tạo. Vì vậy, chúng ta cần đánh dấu event `ShippingStatusUpdated` bằng interface `ShouldBroadcast`. Điều này sẽ hướng dẫn Laravel sẽ tạo broadcast event khi nó được phát hành:

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

Interface `ShouldBroadcast` yêu cầu event của chúng ta định nghĩa một phương thức `broadcastOn`. Phương pháp này chịu trách nhiệm trả lại các channel mà event sẽ broadcast trên đó. Một empty stub của phương thức này sẽ được định nghĩa sẵn trên các class event đã được tạo, vì vậy chúng ta chỉ cần điền vào các thông tin chi tiết của nó. Chúng ta chỉ muốn duy nhất người đã tạo đơn hàng có thể xem các cập nhật trạng thái, vì vậy chúng ta sẽ broadcast event trên một channel private được gắn với đơn đặt hàng:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### Authorizing Channels

Hãy nhớ rằng, người dùng phải được phép listen trên các channel private. Chúng ta có thể định nghĩa các quy tắc authorization channel của bạn trong file `routes/channels.php`. Trong ví dụ này, chúng ta cần xác minh rằng bất kỳ người dùng nào đang cố gắng listen trên channel private tư `order.1` thực sự là người tạo ra đơn đặt hàng:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Phương thức `channel` chấp nhận hai tham số: tên của channel và một callback trả về `true` hoặc `false` cho biết người dùng có được phép listen trên channel hay không.

Tất cả các authorization callback đều nhận vào tham số đầu tiên là người dùng hiện tại đang được authenticate và tham số tiếp theo là tham số ký tự đại diện thêm. Trong ví dụ này, chúng ta đang sử dụng biến `{orderId}` để chỉ ra phần "ID" của tên channel là một ký tự đại diện.

#### Listening For Event Broadcasts

Tiếp theo, tất cả những gì còn lại là listen event trong application JavaScript của chúng ta. Chúng ta có thể làm điều này bằng cách sử dụng Laravel Echo. Đầu tiên, chúng ta sẽ sử dụng phương thức `private` để đăng ký channel private. Sau đó, chúng ta có thể sử dụng phương thức `listen` để listen event `ShippingStatusUpdated`. Mặc định, tất cả các thuộc tính công khai của event sẽ được đưa vào broadcast event:

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## Định nghĩa Broadcast Event

Để thông báo cho Laravel rằng một event nhất định sẽ được broadcast, hãy implement interface`Illuminate\Contracts\Broadcasting\ShouldBroadcast` trong class event. Interface này đã được import vào tất cả các class event được tạo bởi framework để bạn có thể dễ dàng thêm nó vào bất kỳ event nào của bạn.

Interface `ShouldBroadcast` yêu cầu bạn thực hiện một phương thức: `broadcastOn`. Phương thức `broadcastOn` sẽ trả về một channel hoặc một mảng các channel mà event sẽ broadcast trên đó. Các channel phải là các instance của `Channel`, `PrivateChannel`, hoặc `PresenceChannel`. Các instance của `Channel` đại diện cho các channel public mà bất kỳ người dùng nào cũng có thể theo dõi, trong khi `PrivateChannels` và `PresenceChannels` đại diện cho các channel private yêu cầu [channel authorization](#authorizing-channels):

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

Sau đó, bạn chỉ cần [kích hoạt event](/docs/{{version}}/events) như bình thường. Khi event đã được kích hoạt, [queued job](/docs/{{version}}/queues) sẽ tự động broadcast event qua driver broadcast được chỉ định của bạn.

<a name="broadcast-name"></a>
### Broadcast Name

Mặc định, Laravel sẽ broadcast event bằng tên class của event. Tuy nhiên, bạn có thể tùy chỉnh tên chương trình broadcast bằng cách định nghĩa phương thức `broadcastAs` trong event:

    /**
     * The event's broadcast name.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

Nếu bạn tùy chỉnh tên broadcast bằng phương thức `broadcastAs`, bạn nên đảm bảo rằng đăng ký listener của bạn với một ký tự `.` ở đầu. Điều này sẽ hướng dẫn Echo không trả trước namespace của application cho event:

    .listen('.server.created', function (e) {
        ....
    });

<a name="broadcast-data"></a>
### Broadcast Data

Khi một event được broadcast, tất cả các thuộc tính `public` của nó sẽ tự động được serialize và broadcast dưới dạng payload của event, cho phép bạn truy cập bất kỳ dữ liệu công khai nào từ application JavaScript của bạn. Vì thế, ví dụ, nếu event của bạn có một thuộc tính `$user` công khai có chứa modek Eloquent, thì payload phát sóng của event sẽ là:

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

Tuy nhiên, nếu bạn muốn có quyền kiểm soát chi tiết hơn đối với payload broadcast của bạn, bạn có thể thêm một phương thức `broadcastWith` vào event của bạn. Phương thức này sẽ trả về mảng dữ liệu mà bạn muốn phát dưới dạng payload event:

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

Mặc định, mỗi broadcast event mặc định được đặt trên queue cho kết nối queue mặc định được chỉ định trong file cấu hình `queue.php` của bạn. Bạn có thể tùy chỉnh queue được sử dụng bởi broadcaster bằng cách định nghĩa một thuộc tính `broadcastQueue` trong class event của bạn. Thuộc tính này sẽ chỉ định tên của queue mà bạn muốn sử dụng khi broadcasting:

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

Thỉnh thoảng bạn muốn broadcast event của bạn chỉ khi một điều kiện nhất định. Bạn có thể định nghĩa các điều kiện này bằng cách thêm một phương thức `broadcastWhen` vào class event của bạn:

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

Các channel private yêu cầu bạn cho phép người dùng hiện tại đang được authenticate có thể listen trên channel. Điều này được thực hiện bằng cách thực hiện một HTTP request đến application Laravel của bạn với tên channel và cho phép application của bạn xác định xem người dùng có thể listen trên channel đó hay không. Khi sử dụng [Laravel Echo](#installing-laravel-echo), HTTP request cho phép authorize để theo dõi các channel private sẽ được thực hiện tự động; tuy nhiên, bạn cần định nghĩa các route thích hợp để đáp ứng các request này.

<a name="defining-authorization-routes"></a>
### Định nghĩa Authorization Route

Rất may, Laravel giúp dễ dàng xác định các route để đáp ứng các authorization request channel. Class `BroadcastServiceProvider` đi cùng với application Laravel của bạn, bạn sẽ thấy sẽ gọi đến phương thức `Broadcast::routes`. Phương thức này sẽ đăng ký route `/broadcasting/auth` để xử lý các authorization request:

    Broadcast::routes();

Phương thức `Broadcast::routes` sẽ tự động set các route của nó trong group middleware `web`; tuy nhiên, bạn có thể pass một mảng các thuộc tính route cho phương thức nếu bạn muốn tùy chỉnh các thuộc tính được gán:

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### Định nghĩa Authorization Callback

Tiếp theo, chúng ta cần định nghĩa logic sẽ thực hiện authorization channel. Điều này được thực hiện trong file `routes/channels.php` đi kèm với application của bạn. Trong file này, bạn có thể sử dụng phương thức `Broadcast::channel` để đăng ký các callback authorization channel:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Phương thức `channel` chấp nhận hai tham số: tên của channel và một callback trả về `true` hoặc `false` cho biết người dùng có được phép listen trên channel hay không.

Tất cả các callback authorization đều nhận vào tham số đầu tiên là người dùng hiện tại đang được authenticate và tham số tiếp theo là tham số ký tự đại diện thêm. Trong ví dụ này, chúng ta đang sử dụng biến `{orderId}` để chỉ ra phần "ID" của tên channel là một ký tự đại diện.

#### Authorization Callback Model Binding

Giống như các route HTTP, các route channel cũng có thể tận dụng [route model binding](/docs/{{version}}/routing#route-model-binding). Ví dụ, thay vì nhận chuỗi ID hoặc chuỗi số thứ tự, bạn có thể yêu cầu một instance model `Order` thực tế:

    use App\Order;

    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

<a name="broadcasting-events"></a>
## Broadcasting Event

Khi bạn đã xác định một event và đánh dấu nó bằng interface `ShouldBroadcast`, bạn chỉ cần kích hoạt event bằng hàm `event`. Dispatcher event sẽ nhận thấy rằng event được đánh dấu bằng interface `ShouldBroadcast` và sẽ queue event để broadcasting:

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### Only To Others

Khi xây dựng một application sử dụng event broadcasting, bạn có thể thay thế hàm `event` bằng hàm `broadcast`. Giống như hàm `event`, hàm `broadcast` gửi event đến các listener phía máy chủ của bạn:

    broadcast(new ShippingStatusUpdated($update));

Tuy nhiên, hàm `broadcast` cũng hiển thị phương thức `toOthers` cho phép bạn bỏ qua người dùng hiện tại khỏi người nhận của broadcast:

    broadcast(new ShippingStatusUpdated($update))->toOthers();

Để hiểu rõ hơn khi bạn có thể muốn sử dụng phương thức `toOthers`, hãy tưởng tượng một application danh sách nhiệm vụ trong đó người dùng có thể tạo một tác vụ mới bằng cách nhập tên tác vụ. Để tạo một tác vụ, application của bạn có thể đưa tạo một request đến end-point `/task` để broadcasts tạo tác vụ và trả về một JSON của tác vụ mới. Khi application JavaScript của bạn nhận được phản hồi từ end-point, nó có thể trực tiếp chèn tác vụ mới vào danh sách tác vụ của nó như sau:

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

Tuy nhiên, hãy nhớ rằng chúng ta cũng broadcast tạo của nhiệm vụ. Nếu application JavaScript của bạn đang listening event này để thêm các tác vụ vào danh sách tác vụ, bạn sẽ có các tác vụ trùng lặp trong danh sách của bạn: một là từ end-point và một là từ broadcast.

Bạn có thể giải quyết điều này bằng cách sử dụng phương thức `toOthers` để hướng dẫn broadcaster không broadcast event tới người dùng hiện tại.

#### Cấu hình

Khi bạn khởi tạo một instance Laravel Echo, một ID socket sẽ được gán cho kết nối. Nếu bạn đang sử dụng [Vue](https://vuejs.org) và [Axios](https://github.com/mzabriskie/axios), ID socket sẽ tự động được đính kèm vào mọi request gửi đi dưới dạng `X-Socket-ID`. Sau đó, khi bạn gọi phương thức `toOthers`, Laravel sẽ trích xuất ID socket từ header và hướng dẫn broadcaster không broadcast đến bất kỳ kết nối nào với ID socket đó.

Nếu bạn không sử dụng Vue và Axios, bạn sẽ cần phải tự cấu hìnhg application JavaScript của bạn để gửi header `X-Socket-ID`. Bạn có thể lấy ra ID socket bằng phương thức `Echo.socketId`:

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## Nhận Broadcast

<a name="installing-laravel-echo"></a>
### Cài đặt Laravel Echo

Laravel Echo là một thư viện JavaScript khiến cho việc theo dõi các channel và listen các event được broadcast bởi Laravel dễ dàng hơn. Bạn có thể cài đặt Echo thông qua trình quản lý package NPM. Trong ví dụ này, chúng tôi cũng sẽ cài đặt package `pusher-js` vì chúng ta sẽ sử dụng broadcaster Pusher Channel:

    npm install --save laravel-echo pusher-js

Khi Echo đã được cài đặt, bạn đã sẵn sàng tạo một instance Echo mới trong JavaScript của application. Một nơi tuyệt vời để làm điều này là ở dưới cùng của tệp `resources/assets/js/bootstrap.js` được đi kèm trong framework Laravel:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

Khi tạo một instance Echo sử dụng kết nối `pusher`, bạn cũng có thể chỉ định một `cluster` cũng như liệu kết nối phải được thực hiện qua TLS (mặc định, khi `forceTLS` là `false`, một kết nối không phải là TLS sẽ được tạo nếu trang được load qua HTTP hoặc như một fallback nếu kết nối TLS thất bại):

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        cluster: 'eu',
        forceTLS: true
    });

<a name="listening-for-events"></a>
### Listening cho Event

Khi bạn đã cài đặt và khởi tạo Echo, bạn đã sẵn sàng để bắt đầu listening các event broadcast. Đầu tiên, sử dụng phương thức `channel` để lấy ra một instance của một channel, sau đó gọi phương thức `listen` để listen một event đã được chỉ định:

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

Bạn có thể đã nhận thấy trong các ví dụ ở trên rằng chúng ta không chỉ định toàn bộ namespace cho các class event. Điều này là do Echo sẽ tự động giả định các event được đặt trong namespace `App\Events`. Tuy nhiên, bạn có thể cấu hình namespace gốc khi bạn khởi tạo Echo bằng cách pass tùy chọn cấu hình `namespace`:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        namespace: 'App.Other.Namespace'
    });

Ngoài ra, bạn có thể thêm tiền tố cho các class event với một `.` khi theo dõi chúng bằng Echo. Điều này sẽ cho phép bạn luôn chỉ định tên class:

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## Presence Channel

Các presence channel được xây dựng dựa trên tính bảo mật của các private channel đồng thời thêm các tính năng bổ sung về những người đã theo dõi channel. Điều này giúp dễ dàng xây dựng các tính năng application mạnh mẽ, cùng nhau như thông báo cho người dùng khi một người dùng khác đang xem cùng một trang.

<a name="authorizing-presence-channels"></a>
### Authorizing Presence Channel

Tất cả các presence channel cũng là các private channel; do đó, người dùng phải được [authorized để truy cập đến chúng](#authorizing-channels). Tuy nhiên, khi định nghĩa các authorization callback cho các presence channel, bạn sẽ không trả về `true` nếu người dùng đã được authorize tham gia channel. Thay vào đó, bạn nên trả về một mảng dữ liệu về người dùng đó.

Dữ liệu được trả về bởi authorization callback cũng sẽ được cung cấp cho những người đang listen event trong presence channel của application JavaScript của bạn. Nếu người dùng không được phép tham gia presence channel, bạn nên trả về `false` hoặc `null`:

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### Tham gia Presence Channel

Để tham gia một presence channel, bạn có thể sử dụng phương thức `join` của Echo. Phương thức `join` sẽ trả về một implementation `PresenceChannel`, cùng với việc hiển thị phương thức `listen`, cho phép bạn theo dõi các event `here`, `joining`, và `leaving`.

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

Callback `here` sẽ được thực hiện ngay khi channel được kết nối thành công và sẽ nhận được một mảng chứa thông tin người dùng đó cho tất cả những người dùng khác hiện tại đang đăng ký channel biết. Phương thức `joining` sẽ được thực thi khi người dùng mới tham gia channel, trong khi phương thức `leaving` sẽ được thực thi khi người dùng rời khỏi channel.

<a name="broadcasting-to-presence-channels"></a>
### Broadcasting tới Presence Channel

Các presence channel có thể nhận các event giống như các public hoặc private channel. Dùng một ví dụ về một chatroom, chúng ta có thể muốn broadcast các event `NewMessage` lên presence channel của room. Để làm như vậy, chúng tôi sẽ trả về một instance của `PresenceChannel` từ phương thức `broadcastOn` của event:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

Giống như các event public hoặc private, các event của presence channel có thể được broadcast bằng hàm `broadcast`. Cũng như các event khác, bạn có thể sử dụng phương thức `toOthers` để bỏ qua người dùng hiện tại khỏi việc broadcast:

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

Thỉnh thoảng bạn có thể muốn broadcast một event cho những client khác được kết nối  mà không cần gọi application Laravel của bạn. Điều này có thể đặc biệt hữu ích cho những việc như thông báo "đang gõ", bạn muốn thông báo cho người dùng application của bạn rằng có một người dùng khác đang gõ tin nhắn trên một màn hình. Để broadcast các client event, bạn có thể sử dụng phương thức `whisper` của Echo:

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

Bằng cách pair event broadcasting với [notifications](/docs/{{version}}/notifications), application JavaScript của bạn có thể nhận được thông báo mới khi chúng xảy ra mà không cần refresh trang. Trước tiên, hãy nhớ đọc tài liệu về việc sử dụng [the broadcast notification channel](/docs/{{version}}/notifications#broadcast-notifications).

Khi bạn đã cấu hình thông báo để sử dụng broadcast channel, bạn có thể listen các  broadcast event bằng phương thức `notification` của Echo. Hãy nhớ rằng, tên channel phải khớp với tên lớp class thực thể nhận thông báo:

    Echo.private(`App.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

Trong ví dụ trên, tất cả các thông báo được gửi đến các instance `App\User` thông qua channel `broadcast` sẽ được nhận bởi callback. Một authorization callback channel cho channel `App.User.{id}` được chứa mặc định trong `BroadcastServiceProvider` đi kèm với framework Laravel.
