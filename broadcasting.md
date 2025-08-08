# Broadcasting

- [Giới thiệu](#introduction)
- [Cài đặt phía Server](#server-side-installation)
    - [Cấu hình](#configuration)
    - [Pusher Channels](#pusher-channels)
    - [Ably](#ably)
    - [Open Source Alternatives](#open-source-alternatives)
- [Cài đặt phía Client](#client-side-installation)
    - [Pusher Channels](#client-pusher-channels)
    - [Ably](#client-ably)
- [Khái niệm tổng quan](#concept-overview)
    - [Sử dụng một application mẫu](#using-example-application)
- [Định nghĩa Broadcast Event](#defining-broadcast-events)
    - [Broadcast Name](#broadcast-name)
    - [Broadcast Data](#broadcast-data)
    - [Broadcast Queue](#broadcast-queue)
    - [Broadcast Condition](#broadcast-conditions)
    - [Broadcasting và Database Transactions](#broadcasting-and-database-transactions)
- [Authorizing Channel](#authorizing-channels)
    - [Định nghĩa Authorization Route](#defining-authorization-routes)
    - [Định nghĩa Authorization Callback](#defining-authorization-callbacks)
    - [Định nghĩa Channel Class](#defining-channel-classes)
- [Broadcasting Event](#broadcasting-events)
    - [Only To Others](#only-to-others)
    - [Tuỳ chỉnh Connection](#customizing-the-connection)
- [Nhận Broadcast](#receiving-broadcasts)
    - [Listening cho Event](#listening-for-events)
    - [Rời một Channel](#leaving-a-channel)
    - [Namespaces](#namespaces)
- [Presence Channel](#presence-channels)
    - [Authorizing Presence Channel](#authorizing-presence-channels)
    - [Tham gia Presence Channel](#joining-presence-channels)
    - [Broadcasting tới Presence Channel](#broadcasting-to-presence-channels)
- [Model Broadcasting](#model-broadcasting)
    - [Model Broadcasting Conventions](#model-broadcasting-conventions)
    - [Listening For Model Broadcasts](#listening-for-model-broadcasts)
- [Client Event](#client-events)
- [Notification](#notifications)

<a name="introduction"></a>
## Giới thiệu

Trong nhiều application hoặc web hiện đại, WebSockets được sử dụng để thực hiện cập nhật trực tiếp lên giao diện người dùng theo thời gian thực. Sau khi dữ liệu được cập nhật lên máy chủ, thì một thông báo cũng sẽ được gửi qua kết nối WebSocket để được phía client xử lý. WebSockets cung cấp một sự thay thế mạnh mẽ và hiệu quả hơn để liên tục đồng bộ máy chủ của ứng dụng của bạn cho các thay đổi dữ liệu sẽ được phản ánh lên giao diện người dùng.

Ví dụ: hãy tưởng tượng ứng dụng của bạn có thể export dữ liệu của người dùng sang file CSV và gửi email cho họ. Tuy nhiên, việc tạo file CSV này mất vài phút nên bạn chọn cách tạo và gửi file CSV qua mail trong một [queued job](/docs/{{version}}/queues). Khi CSV đã được tạo và gửi qua mail cho người dùng, chúng ta có thể sử dụng broadcasting để gửi một event `App\Events\UserDataExported` mà JavaScript của ứng dụng của chúng ta nhận được. Sau khi nhận được event, chúng ta có thể hiển thị thông báo cho người dùng là file CSV của họ đã được gửi qua email và họ không cần phải refresh lại trang.

Để hỗ trợ bạn trong việc xây dựng các loại chức năng này, Laravel giúp bạn dễ dàng "broadcast" Laravel [events](/docs/{{version}}/events) phía máy chủ của bạn qua kết nối WebSocket. Broadcasting các event Laravel của bạn cho phép bạn chia sẻ cùng tên event và dữ liệu giữa ứng dụng Laravel bên server và ứng dụng JavaScript phía client của bạn.

Các khái niệm cốt lõi đằng sau việc broadcasting rất đơn giản: các client kết nối với các channel được đặt tên trong giao diện người dùng, trong khi ứng dụng Laravel của bạn broadcast các event tới các channel này trong phần backend. Những event này có thể chứa bất kỳ dữ liệu nào mà bạn muốn cung cấp cho giao diện người dùng.

<a name="supported-drivers"></a>
#### Supported Drivers

Mặc định, Laravel có chứa driver broadcasting cho server-side để bạn lựa chọn: [Pusher Channels](https://pusher.com/channels) và [Ably](https://ably.com). Tuy nhiên, các package do cộng đồng phát triển như [soketi](https://docs.soketi.app/) cũng cung cấp thêm các driver broadcasting và không yêu cầu một giấy phép broadcasting thương mại.

> [!NOTE]
> Trước khi đi sâu vào broadcasting event, hãy đảm bảo là bạn đã đọc tài liệu của Laravel về [event và listener](/docs/{{version}}/events).

<a name="server-side-installation"></a>
## Cài đặt phía Server

Để bắt đầu sử dụng chức năng broadcasting event của Laravel, chúng ta cần thực hiện một số cấu hình trong ứng dụng Laravel cũng như cài đặt một vài package.

Broadcasting event được thực hiện bởi một driver broadcasting server-side và nó sẽ broadcasting các event Laravel của bạn để Laravel Echo (một thư viện JavaScript) có thể nhận được trong ứng dụng client trên trình duyệt. Đừng lo lắng - chúng tôi sẽ hướng dẫn từng bước của quy trình cài đặt.

<a name="configuration"></a>
### Cấu hình

Tất cả các cấu hình event broadcasting của application đều được lưu trữ trong file cấu hình `config/broadcasting.php`. Mặc định, Laravel hỗ trợ một số broadcast driver: [Pusher Channels](https://pusher.com/channels), [Redis](/docs/{{version}}/redis), và driver `log` dành cho lúc phát triển và lúc gỡ lỗi. Ngoài ra, driver `null` cũng được cung cấp cho phép bạn tắt hoàn toàn broadcasting trong khi test. Một số cấu hình mẫu cũng sẽ được cung cấp trong file cấu hình `config/broadcasting.php`.

<a name="broadcast-service-provider"></a>
#### Broadcast Service Provider

Trước khi broadcasting bất kỳ event nào, đầu tiên bạn sẽ cần phải đăng ký `App\Providers\BroadcastServiceProvider`. Trong một application Laravel mới, bạn chỉ cần bỏ comment provider này trong mảng `providers` của file cấu hình `config/app.php`. `BroadcastServiceProvider` này sẽ chứa một số code cho phép bạn đăng ký các route authorization broadcasting và các callback của chúng.

<a name="queue-configuration"></a>
#### Queue Configuration

Bạn cũng sẽ cần cấu hình và chạy một [queue worker](/docs/{{version}}/queues). Tất cả việc broadcasting event sẽ được thực hiện thông qua các queued job để thời gian phản hồi của ứng dụng của bạn không bị ảnh hưởng nghiêm trọng bởi các event đang được broadcast.

<a name="pusher-channels"></a>
### Pusher Channels

Nếu bạn có kế hoạch broadcast các event của bạn bằng cách sử dụng [Pusher Channels](https://pusher.com/channels), thì bạn nên cài đặt SDK PHP của Pusher Channels bằng trình quản lý package Composer:

```shell
composer require pusher/pusher-php-server
```

Tiếp theo, bạn nên cấu hình thông tin đăng nhập Pusher Channel của bạn trong file cấu hình `config/broadcasting.php`. Một ví dụ về cấu hình Pusher Channel đã được chứa trong file này, cho phép bạn nhanh chóng chỉ định khóa, secret và ID ứng dụng của bạn. Thông thường, các giá trị này phải được set thông qua [biến môi trường](/docs/{{version}}/configuration#environment-configuration) `PUSHER_APP_KEY`, `PUSHER_APP_SECRET` và `PUSHER_APP_ID`:

```ini
PUSHER_APP_ID=your-pusher-app-id
PUSHER_APP_KEY=your-pusher-key
PUSHER_APP_SECRET=your-pusher-secret
PUSHER_APP_CLUSTER=mt1
```

Cấu hình `pusher` của file `config/broadcasting.php` cũng cho phép bạn chỉ định thêm các `options` được hỗ trợ bởi Channel, chẳng hạn như cluster.

Tiếp theo, bạn sẽ cần thay đổi driver broadcast của bạn thành `pusher` trong file `.env` của bạn:

```ini
BROADCAST_DRIVER=pusher
```

Cuối cùng, bạn đã sẵn sàng để cài đặt và cấu hình [Laravel Echo](#client-side-installation) và sẽ nhận các broadcast event ở phía client.

<a name="pusher-compatible-open-source-alternatives"></a>
#### Open Source Pusher Alternatives

[soketi](https://docs.soketi.app/) cung cấp một server WebSocket tương thích với Pusher cho Laravel, cho phép bạn tận dụng toàn bộ sức mạnh của Laravel Broadcasting mà không cần các nhà cung cấp WebSocket thương mại. Để biết thêm thông tin về cách cài đặt và hướng dẫn sử dụng package mã nguồn mở cho broadcasting, vui lòng tham khảo tài liệu của chúng tôi về [các lựa chọn thay thế mã nguồn mở](#open-source-alternatives).

<a name="ably"></a>
### Ably

> [!NOTE]
> Tài liệu dưới đây sẽ thảo luận về cách dùng Ably trong chế độ "tương thích với Pusher". Tuy nhiên, Ably team rất khuyến khích bạn và duy trì một broadcaster, một Echo client có thể tận dụng tối đa các khả năng độc đáo do Ably cung cấp. Để biết thêm thông tin về cách sử dụng các driver được Ably cung cấp, vui lòng [tham khảo tài liệu về broadcaster Laravel của Ably](https://github.com/ably/laravel-broadcaster).

Nếu bạn định broadcast các event của bạn bằng [Ably](https://ably.com), thì bạn nên cài đặt Ably PHP SDK bằng trình quản lý package Composer:

```shell
composer require ably/ably-php
```

Tiếp theo, bạn nên cấu hình thông tin đăng nhập Ably của bạn trong file cấu hình `config/broadcasting.php`. Một ví dụ cấu hình Ably đã được chứa trong file này, cho phép bạn nhanh chóng chỉ định khóa của bạn. Thông thường, giá trị này phải được set thông qua [biến môi trường](/docs/{{version}}/configuration#environment-configuration) `ABLY_KEY`:

```ini
ABLY_KEY=your-ably-key
```

Tiếp theo, bạn sẽ cần thay đổi driver broadcast của bạn thành `ably` trong file `.env` của bạn:

```ini
BROADCAST_DRIVER=ably
```

Cuối cùng, bạn đã sẵn sàng để cài đặt và cấu hình [Laravel Echo](#client-side-installation) và sẽ nhận các broadcast event ở phía client.

<a name="open-source-alternatives"></a>
### Open Source Alternatives

<a name="open-source-alternatives-node"></a>
#### Node

[Soketi](https://github.com/soketi/soketi) là một máy chủ WebSocket tương thích với Pusher, dựa trên Node và dành cho Laravel. Về cơ bản, Soketi sử dụng µWebSockets.js để có tốc độ và khả năng mở rộng cực cao. Package này cho phép bạn tận dụng toàn bộ sức mạnh của Laravel Broadcasting mà không cần phải nhà cung cấp WebSocket thương mại. Để biết thêm thông tin về cách cài đặt và sử dụng package này, vui lòng tham khảo [tài liệu chính thức](https://docs.soketi.app/) của nó.

<a name="client-side-installation"></a>
## Cài đặt phía Client

<a name="client-pusher-channels"></a>
### Pusher Channels

[Laravel Echo](https://github.com/laravel/echo) là một thư viện JavaScript giúp bạn dễ dàng đăng ký channel và lắng nghe các event do các driver broadcasting server-side của bạn. Bạn có thể cài đặt Echo thông qua trình quản lý package NPM. Trong ví dụ này, chúng ta cũng sẽ cài đặt package `pusher-js` vì chúng ta sẽ sử dụng driver broadcaster Pusher Channel:

```shell
npm install --save-dev laravel-echo pusher-js
```

Sau khi cài đặt Echo, bạn đã sẵn sàng tạo một instance Echo mới trong JavaScript của ứng dụng. Một nơi tuyệt vời để làm điều này là ở dưới cùng của file `resources/js/bootstrap.js` đã được chứa trong Laravel framework. Mặc định, một cấu hình Echo ví dụ đã được chứa trong file này - bạn chỉ cần bỏ comment nó:

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```

Khi bạn đã bỏ comment và điều chỉnh cấu hình Echo theo nhu cầu của bạn, bạn có thể biên dịch các asset của ứng dụng:

```shell
npm run build
```

> [!NOTE]
> Để tìm hiểu thêm về cách biên dịch asset JavaScript cho ứng dụng của bạn, vui lòng tham khảo tài liệu về [Vite](/docs/{{version}}/vite).

<a name="using-an-existing-client-instance"></a>
#### Using An Existing Client Instance

Nếu bạn đã có một instance client Pusher Channel được cấu hình sẵn mà bạn muốn Echo sử dụng, bạn có thể truyền nó tới Echo thông qua tùy chọn cấu hình `client`:

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

const options = {
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key'
}

window.Echo = new Echo({
    ...options,
    client: new Pusher(options.key, options)
});
```

<a name="client-ably"></a>
### Ably

> [!NOTE]
> Tài liệu dưới đây sẽ thảo luận về cách dùng Ably trong chế độ "tương thích với Pusher". Tuy nhiên, Ably team rất khuyến khích bạn và duy trì một broadcaster, một Echo client có thể tận dụng tối đa các khả năng độc đáo do Ably cung cấp. Để biết thêm thông tin về cách sử dụng các driver được Ably cung cấp, vui lòng [tham khảo tài liệu về broadcaster Laravel của Ably](https://github.com/ably/laravel-broadcaster).

[Laravel Echo](https://github.com/laravel/echo) là một thư viện JavaScript giúp bạn dễ dàng đăng ký channel và lắng nghe các event do các driver broadcasting server-side của bạn. Bạn có thể cài đặt Echo thông qua trình quản lý package NPM. Trong ví dụ này, chúng ta cũng sẽ cài đặt package `pusher-js`.

Bạn có thể thắc mắc tại sao chúng tôi lại cài đặt thư viện JavaScript `pusher-js` mặc dù chúng tôi đang sử dụng Ably để broadcast các event của bạn. Rất may, Ably đã chứa chế độ tương thích với Pusher cho phép chúng ta sử dụng giao thức Pusher khi lắng nghe các event trong ứng dụng client-side của chúng ta:

```shell
npm install --save-dev laravel-echo pusher-js
```

**Trước khi tiếp tục, bạn nên bật hỗ trợ giao thức Pusher trong cài đặt ứng dụng Ably của bạn. Bạn có thể bật chức năng này trong phần "Protocol Adapter Settings" trên bảng điều khiển cài đặt của ứng dụng Ably.**

Sau khi cài đặt Echo, bạn đã sẵn sàng tạo một instance Echo mới trong JavaScript của ứng dụng. Một nơi tuyệt vời để làm điều này là ở dưới cùng của file `resources/js/bootstrap.js` đã được chứa trong Laravel framework. Mặc định, một cấu hình Echo ví dụ đã được chứa trong file này; tuy nhiên, cấu hình mặc định trong file `bootstrap.js` là dành cho Pusher. Bạn có thể sao chép cấu hình bên dưới để chuyển cấu hình của bạn sang Ably:

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    wsHost: 'realtime-pusher.ably.io',
    wsPort: 443,
    disableStats: true,
    encrypted: true,
});
```

Lưu ý rằng cấu hình Ably Echo của chúng ta đang tham chiếu đến biến môi trường `VITE_ABLY_PUBLIC_KEY`. Giá trị của biến này phải là khóa công khai Ably của bạn. Khóa công khai của bạn là một phần của khóa Ably xuất hiện trước ký tự `:`.

Khi bạn đã bỏ comment và điều chỉnh cấu hình Echo theo nhu cầu của bạn, bạn có thể biên dịch các asset của ứng dụng:

```shell
npm run dev
```

> [!NOTE]
> Để tìm hiểu thêm về cách biên dịch asset JavaScript cho ứng dụng của bạn, vui lòng tham khảo tài liệu về [Vite](/docs/{{version}}/vite).

<a name="concept-overview"></a>
## Khái niệm tổng quan

Broadcasting event của Laravel cho phép bạn broadcast các event Laravel ở phía máy chủ của bạn tới các application ở JavaScript bên phía client bằng cách sử dụng các phương pháp tiếp cận dựa trên các driver WebSockets. Hiện tại, Laravel hỗ trợ [Pusher Channels](https://pusher.com/channels) và driver [Ably](https://ably.com). Các event có thể được sử dụng dễ dàng ở phía client bằng cách sử dụng package Javascript [Laravel Echo](#client-side-installation).

Các event được broadcast qua các "channels", có thể chỉ định là công khai hoặc là riêng tư. Bất kỳ client nào truy cập vào application của bạn đều có thể đăng ký channel công khai mà không cần bất kỳ authentication hoặc authorization nào; tuy nhiên, để đăng ký channel private, người dùng phải được authentication và authorization để listen trên channel đó.

> [!NOTE]
> Nếu bạn muốn sử dụng một open source để thay thế cho Pusher, hãy xem thử [các lựa chọn package thay thế nguồn mở](#open-source-alternatives).

<a name="using-example-application"></a>
### Sử dụng một application mẫu

Trước khi đi sâu vào từng thành phần của event broadcasting, bạn có thể có cái nhìn tổng quan bằng cách sử dụng một cửa hàng thương mại điện tử làm ví dụ mẫu.

Trong application của chúng ta, giả sử chúng ta có một trang cho phép người dùng xem trạng thái giao hàng của đơn hàng của họ. Chúng ta cũng giả sử rằng một event `OrderShipmentStatusUpdated` sẽ được kích hoạt khi một shipping được cập nhật trạng thái bởi application:

    use App\Events\OrderShipmentStatusUpdated;

    OrderShipmentStatusUpdated::dispatch($order);

<a name="the-shouldbroadcast-interface"></a>
#### The `ShouldBroadcast` Interface

Khi người dùng đang xem một trong các đơn hàng của họ, chúng ta không muốn họ phải refresh trang để xem lại trạng thái của đơn hàng đó. Thay vào đó, chúng ta muốn broadcast các cập nhật trạng thái cho application của chúng ta khi chúng được tạo. Vì thế, chúng ta cần đánh dấu event `OrderShipmentStatusUpdated` bằng interface `ShouldBroadcast`. Điều này sẽ hướng dẫn Laravel là tạo một broadcast event khi event đó được kích hoạt:

    <?php

    namespace App\Events;

    App\Models\Order;
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class OrderShipmentStatusUpdated implements ShouldBroadcast
    {
        /**
         * The order instance.
         *
         * @var \App\Order
         */
        public $order;
    }

Interface `ShouldBroadcast` yêu cầu event của chúng ta cần định nghĩa một phương thức là `broadcastOn`. Phương thức này sẽ chịu trách nhiệm trả về các channel mà event này sẽ broadcast trên đó. Một empty stub của phương thức này sẽ được định nghĩa sẵn cho chúng ta trên các class event đã được tạo ra, vì vậy chúng ta sẽ chỉ cần điền các thông tin chi tiết về nó. Chúng ta muốn chỉ duy nhất người đã tạo đơn hàng này mới có thể xem trạng thái, vì vậy chúng ta sẽ cần broadcast event này trên một channel private được gắn với đơn đặt hàng:

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\PrivateChannel;

    /**
     * Get the channel the event should broadcast on.
     */
    public function broadcastOn(): Channel
    {
        return new PrivateChannel('orders.'.$this->order->id);
    }

Nếu bạn muốn event được broadcast đến nhiều channel, bạn có thể trả về một `array`:

    use Illuminate\Broadcasting\PrivateChannel;

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('orders.'.$this->order->id),
            // ...
        ];
    }

<a name="example-application-authorizing-channels"></a>
#### Authorizing Channels

Hãy nhớ rằng, người dùng phải có phép thì mới có thể listen trên các channel private. Chúng ta có thể định nghĩa các quy tắc authorization cho channel này trong file `routes/channels.php` của application. Trong ví dụ này, chúng ta cần kiểm tra rằng bất kỳ người dùng nào đang cố gắng listen trên channel private `orders.1` này có phải là người đã tạo ra đơn đặt hàng hay không:

    use App\Models\Order;
    use App\Models\User;

    Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Phương thức `channel` này chấp nhận hai tham số: một là tên của channel và một là callback trả về `true` hoặc `false` cho biết người dùng đó có được phép listen trên channel hay không.

Tất cả các authorization callback này đều nhận vào tham số đầu tiên là người dùng hiện tại đang được authenticate và tham số tiếp theo là tham số đại diện cho biến được truyền theo tên của channel. Trong ví dụ này, chúng ta đang sử dụng biến `{orderId}` để lấy ra phần "ID" trong tên của channel.

<a name="listening-for-event-broadcasts"></a>
#### Listening For Event Broadcasts

Tiếp theo, tất cả những gì còn lại là listen event trong JavaScript của chúng ta. Chúng ta có thể làm điều này bằng cách sử dụng [Laravel Echo](#client-side-installation). Đầu tiên, chúng ta sẽ sử dụng phương thức `private` để đăng ký channel private. Sau đó, chúng ta có thể sử dụng phương thức `listen` để listen event `OrderShipmentStatusUpdated`. Mặc định, tất cả các thuộc tính công khai của event sẽ được đưa vào trong broadcast event:

```js
Echo.private(`orders.${orderId}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order);
    });
```

<a name="defining-broadcast-events"></a>
## Định nghĩa Broadcast Event

Để thông báo cho Laravel rằng một event sẽ được broadcast, bạn phải implement interface `Illuminate\Contracts\Broadcasting\ShouldBroadcast` trong class event. Mặc định, interface này sẽ được import vào tất cả các class event mà được tạo bởi framework để bạn có thể dễ dàng thêm nó vào bất kỳ event nào của bạn.

Interface `ShouldBroadcast` yêu cầu bạn implement một phương thức: `broadcastOn`. Phương thức `broadcastOn` sẽ trả về tên một channel hoặc một mảng tên các channel mà event sẽ được broadcast trên đó. Các channel phải là các instance của `Channel`, `PrivateChannel`, hoặc `PresenceChannel`. Các instance của `Channel` là đại diện cho các channel public mà bất kỳ người dùng nào cũng có thể vào, trong khi `PrivateChannels` và `PresenceChannels` là đại diện cho các channel private yêu cầu [channel authorization](#authorizing-channels):

    <?php

    namespace App\Events;

    use App\Models\User;
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        /**
         * Create a new event instance.
         */
        public function __construct(
            public User $user,
        ) {}

        /**
         * Get the channels the event should broadcast on.
         *
         * @return array<int, \Illuminate\Broadcasting\Channel>
         */
        public function broadcastOn(): array
        {
            return [
                new PrivateChannel('user.'.$this->user->id),
            ];
        }
    }

Sau khi implement interface `ShouldBroadcast`, bạn chỉ cần [kích hoạt event](/docs/{{version}}/events) như bình thường. Khi event đó đã được kích hoạt, [queued job](/docs/{{version}}/queues) sẽ tự động broadcast event đó thông qua driver broadcast mà chúng ta đã định nghĩa.

<a name="broadcast-name"></a>
### Broadcast Name

Mặc định, Laravel sẽ broadcast event bằng tên class của event. Tuy nhiên, bạn có thể tùy chỉnh tên của broadcast bằng cách định nghĩa phương thức `broadcastAs` trong event:

    /**
     * The event's broadcast name.
     */
    public function broadcastAs(): string
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

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

Tuy nhiên, nếu bạn muốn có quyền kiểm soát chi tiết hơn đối với payload broadcast của bạn, bạn có thể thêm một phương thức `broadcastWith` vào event của bạn. Phương thức này sẽ trả về mảng dữ liệu mà bạn muốn broadcast dưới dạng payload event:

    /**
     * Get the data to broadcast.
     *
     * @return array<string, mixed>
     */
    public function broadcastWith(): array
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### Broadcast Queue

Mặc định, mỗi broadcast event sẽ được đặt trên một queue mặc định với một kết nối queue mặc định được định nghĩa trong file cấu hình `queue.php` của bạn. Bạn có thể tùy chỉnh kết nối queue và tên được broadcaster sử dụng bằng cách định nghĩa các thuộc tính `connection` và `queue` trên các event class của bạn:

    /**
     * The name of the queue connection to use when broadcasting the event.
     *
     * @var string
     */
    public $connection = 'redis';

    /**
     * The name of the queue on which to place the broadcasting job.
     *
     * @var string
     */
    public $queue = 'default';

Ngoài ra, bạn có thể tùy chỉnh tên queue bằng cách định nghĩa phương thức `broadcastQueue` cho event của bạn:

    /**
     * The name of the queue on which to place the broadcasting job.
     */
    public function broadcastQueue(): string
    {
        return 'default';
    }

Nếu bạn muốn broadcast event của bạn bằng queue `sync` thay vì driver queue mặc định, bạn có thể implement interface `ShouldBroadcastNow` thay vì `ShouldBroadcast`:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class OrderShipmentStatusUpdated implements ShouldBroadcastNow
    {
        // ...
    }

<a name="broadcast-conditions"></a>
### Broadcast Conditions

Thỉnh thoảng bạn cũng có thể muốn broadcast event của bạn trong một điều kiện nhất định. Bạn có thể định nghĩa các điều kiện này bằng cách thêm một phương thức `broadcastWhen` vào trong class event của bạn:

    /**
     * Determine if this event should broadcast.
     */
    public function broadcastWhen(): bool
    {
        return $this->order->value > 100;
    }

<a name="broadcasting-and-database-transactions"></a>
#### Broadcasting và Database Transactions

Khi các broadcast event được gửi đi trong các database transaction, chúng có thể được queue xử lý trước khi database transaction được thực hiện. Khi điều này xảy ra, bất kỳ cập nhật nào mà bạn đã thực hiện đối với model hoặc record cơ sở dữ liệu trong quá trình database transaction có thể chưa được phản ánh trong cơ sở dữ liệu. Ngoài ra, bất kỳ model hoặc record cơ sở dữ liệu nào được tạo trong transaction có thể không tồn tại trong cơ sở dữ liệu. Nếu event của bạn phụ thuộc vào các model này, lỗi không mong muốn có thể xảy ra khi job broadcast event được xử lý.

Nếu tùy chọn cấu hình `after_commit` của queue connection của bạn được set thành `false`, thì bạn vẫn có thể cho biết một broadcast event sẽ được gửi đi sau khi tất cả các database transaction đã được thực hiện bằng cách implement interface `ShouldDispatchAfterCommit` trên class event đó:

    <?php

    namespace App\Events;

    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
    use Illuminate\Queue\SerializesModels;

    class ServerCreated implements ShouldBroadcast, ShouldDispatchAfterCommit
    {
        use SerializesModels;
    }

> [!NOTE]
> Để tìm hiểu thêm về cách khắc phục những sự cố như thế này, vui lòng xem lại tài liệu về [queued job và database transaction](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="authorizing-channels"></a>
## Authorizing Channels

Các channel private sẽ yêu cầu bạn authorize rằng người dùng hiện tại đang được authenticate có thể có listen trên channel private này hay không. Điều này có thể được thực hiện bằng cách tạo một HTTP request đến application Laravel của bạn với tên channel và sau đó application của bạn có thể xác định xem người dùng đó có thể listen trên channel đó hay không. Khi sử dụng [Laravel Echo](#client-side-installation), thì HTTP request authorize này sẽ được tạo ra tự động; tuy nhiên, bạn sẽ cần định nghĩa thêm các route để respond lại các request này.

<a name="defining-authorization-routes"></a>
### Định nghĩa Authorization Route

Rất may, Laravel đã giúp việc định nghĩa các route này một cách dễ dàng. Trong class `App\Providers\BroadcastServiceProvider` mà đi cùng với application Laravel, bạn sẽ thấy nó gọi đến một phương thức `Broadcast::routes`. Phương thức này sẽ đăng ký route `/broadcasting/auth` để xử lý các authorization request:

    Broadcast::routes();

Phương thức `Broadcast::routes` sẽ tự động đăng ký route của nó vào trong group middleware `web`; tuy nhiên, bạn có thể truyền một mảng các thuộc tính của route đó vào phương thức này nếu bạn muốn tùy chỉnh các thuộc tính đó:

    Broadcast::routes($attributes);

<a name="customizing-the-authorization-endpoint"></a>
#### Tuỳ biến điểm cuối để Authorization

Mặc định, Echo sẽ sử dụng điểm cuối `/broadcasting/auth` để authorize quyền truy cập vào channel. Tuy nhiên, bạn có thể chỉ định điểm cuối authorize của riêng bạn bằng cách thêm tùy chọn cấu hình `authEndpoint` cho instance Echo của bạn:

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    authEndpoint: '/custom/endpoint/auth'
});
```

<a name="customizing-the-authorization-request"></a>
#### Customizing The Authorization Request

Bạn có thể tùy chỉnh cách Laravel Echo thực hiện các authorization request bằng cách cung cấp môt tùy chỉnh authorizer khi khởi tạo Echo:

```js
window.Echo = new Echo({
    // ...
    authorizer: (channel, options) => {
        return {
            authorize: (socketId, callback) => {
                axios.post('/api/broadcasting/auth', {
                    socket_id: socketId,
                    channel_name: channel.name
                })
                .then(response => {
                    callback(null, response.data);
                })
                .catch(error => {
                    callback(error);
                });
            }
        };
    },
})
```

<a name="defining-authorization-callbacks"></a>
### Định nghĩa Authorization Callback

Tiếp theo, chúng ta cần định nghĩa các logic sẽ được determine if the currently authenticated user can listen to a given channel. Điều này sẽ được thực hiện trong file `routes/channels.php` đi kèm với application. Trong file này, bạn có thể sử dụng phương thức `Broadcast::channel` để đăng ký các callback authorization channel:

    use App\Models\User;

    Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Phương thức `channel` chấp nhận hai tham số: một là tên của channel và một là callback trả về `true` hoặc `false` cho biết người dùng đó có được phép listen trên channel này hay không.

Tất cả các callback authorization đều nhận vào tham số đầu tiên là người dùng hiện tại đang được authenticate và tham số tiếp theo là một tham số đại diện. Trong ví dụ này, chúng ta đang sử dụng biến `{orderId}` để chỉ ra phần "ID" của tên channel.

Bạn có thể xem một danh sách các callback của broadcast authorization trong ứng dụng của bằng lệnh Artisan `channel:list`:

```shell
php artisan channel:list
```

<a name="authorization-callback-model-binding"></a>
#### Authorization Callback Model Binding

Giống như các route HTTP, các route channel cũng có thể tận dụng các [route model binding](/docs/{{version}}/routing#route-model-binding). Ví dụ, thay vì nhận một chuỗi ID hoặc một chuỗi số thứ tự, bạn có thể yêu cầu một instance model `Order`:

    use App\Models\Order;
    use App\Models\User;

    Broadcast::channel('orders.{order}', function (User $user, Order $order) {
        return $user->id === $order->user_id;
    });

> [!WARNING]
> Không giống như liên kết model route HTTP, liên kết model channel không cung cấp hỗ trợ tự động [scope theo liên kết model ngầm](/docs/{{version}}/routing#implicit-model-binding-scoping). Tuy nhiên, đây hiếm khi là vấn đề vì hầu hết các channel có thể được xác định scope dựa trên khóa chính, duy nhất của một model.

<a name="authorization-callback-authentication"></a>
#### Authorization Callback Authentication

Các channel broadcast private và presence sẽ xác thực người dùng hiện tại thông qua authentication guard mặc định của ứng dụng. Nếu người dùng không được xác thực, channel authorization cũng sẽ tự động bị từ chối và lệnh authorization callback cũng sẽ không bao giờ được thực thi. Tuy nhiên, bạn có thể chỉ định nhiều guard tùy chỉnh khác sẽ xác thực request đến nếu cần:

    Broadcast::channel('channel', function () {
        // ...
    }, ['guards' => ['web', 'admin']]);

<a name="defining-channel-classes"></a>
### Định nghĩa Channel Class

Nếu ứng dụng của bạn sử dụng nhiều channel khác nhau, thì file `routes/channels.php` của bạn có thể trở nên rất cồng kềnh. Vì vậy, thay vì sử dụng closure để cấp quyền cho các channel, bạn có thể sử dụng các class channel. Để tạo một class channel mới, hãy sử dụng lệnh Artisan `make:channel`. Lệnh này sẽ lưu một class channel mới vào trong thư mục `App/Broadcasting`.

```shell
php artisan make:channel OrderChannel
```

Tiếp theo, đăng ký channel của bạn vào trong file `routes/channels.php`:

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('orders.{order}', OrderChannel::class);

Cuối cùng, bạn có thể viết các logic cấp quyền cho channel của bạn vào trong phương thức `join` của class channel. Phương thức `join` sẽ chứa cùng một logic với code mà bạn thường viết trong closure cấp quyền channel của bạn. Bạn cũng có thể tận dụng lợi thế của liên kết model channel:

    <?php

    namespace App\Broadcasting;

    use App\Models\Order;
    use App\Models\User;

    class OrderChannel
    {
        /**
         * Create a new channel instance.
         */
        public function __construct()
        {
            // ...
        }

        /**
         * Authenticate the user's access to the channel.
         */
        public function join(User $user, Order $order): array|bool
        {
            return $user->id === $order->user_id;
        }
    }

> [!NOTE]
> Giống như nhiều class khác trong Laravel, các class channel sẽ tự động được resolve bởi [service container](/docs/{{version}}/container). Vì vậy, bạn có thể khai báo bất kỳ phụ thuộc nào mà channel của bạn cần trong hàm tạo của nó.

<a name="broadcasting-events"></a>
## Broadcasting Event

Khi bạn đã định nghĩa một event và đánh dấu nó bằng một interface `ShouldBroadcast`, bạn chỉ cần kích hoạt event đó bằng hàm dispatch của event. Dispatcher của event sẽ hiểu được rằng event đó đã được đánh dấu bằng một interface `ShouldBroadcast` nên nó sẽ tạo một queue event để broadcasting:

    use App\Events\OrderShipmentStatusUpdated;

    OrderShipmentStatusUpdated::dispatch($order);

<a name="only-to-others"></a>
### Only To Others

Khi xây dựng một application sử dụng event broadcasting, đôi khi bạn có thể cần broadcast một event cho tất cả những người đăng ký trên một channel nhất định ngoại trừ người dùng hiện tại. Bạn có thể thực hiện việc này bằng cách sử dụng helper `broadcast` và phương thức `toOthers`:

    use App\Events\OrderShipmentStatusUpdated;

    broadcast(new OrderShipmentStatusUpdated($update))->toOthers();

Để hiểu rõ hơn lý do mà bạn có thể muốn sử dụng phương thức `toOthers`, hãy tưởng tượng một application quản lý danh sách các task trong đó người dùng có thể tạo ra một task mới bằng cách nhập tên task. Để tạo một task mới, application của bạn có thể tạo một request đến url `/task` để broadcasts tạo task và trả về một JSON là một task mới. Khi JavaScript của bạn nhận được phản hồi từ route, nó có thể trực tiếp chèn task mới này vào danh sách các task đã tồn tại như sau:

```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

Tuy nhiên, hãy nhớ rằng chúng ta đang broadcast một event tạo task. Nếu JavaScript của bạn cũng đang listening event này, để thêm task mới vào danh sách task, thì bạn có thể có các task trùng lặp trong danh sách của bạn: một là từ route và một là từ broadcast. Bạn có thể giải quyết điều này bằng cách sử dụng phương thức `toOthers` để hướng dẫn broadcaster không broadcast event tới người dùng hiện tại.

> [!WARNING]
> Event của bạn phải sử dụng trait `Illuminate\Broadcasting\InteractsWithSockets` để gọi phương thức `toOthers`.

<a name="only-to-others-configuration"></a>
#### Cấu hình

Khi bạn khởi tạo một instance Laravel Echo, một ID socket cũng sẽ được khởi tạo. Nếu bạn đang sử dụng một global instance [Axios](https://github.com/mzabriskie/axios) để thực hiện các request HTTP từ ứng dụng JavaScript của bạn, thì ID socket đó sẽ được tự động đính kèm vào mọi request gửi đi dưới dạng `X-Socket-ID`. Sau đó, khi bạn gọi phương thức `toOthers`, Laravel sẽ lấy ID socket từ header và hướng dẫn broadcaster sẽ không broadcast đến bất kỳ kết nối nào mà trùng với ID socket đó.

Nếu bạn không sử dụng một global Axios instance, bạn sẽ cần phải tự cấu hình JavaScript của bạn để gửi header `X-Socket-ID` với tất cả các request gửi đi. Bạn có thể lấy ra ID socket bằng phương thức `Echo.socketId`:

```js
var socketId = Echo.socketId();
```

<a name="customizing-the-connection"></a>
### Tuỳ chỉnh Connection

Nếu ứng dụng của bạn tương tác với nhiều kết nối broadcast và bạn muốn broadcast một event bằng cách sử dụng một broadcaster khác, khác với mặc định của bạn, thì bạn có thể chỉ định kết nối đó bằng cách sử dụng phương thức `via`:

    use App\Events\OrderShipmentStatusUpdated;

    broadcast(new OrderShipmentStatusUpdated($update))->via('pusher');

Ngoài ra, bạn có thể chỉ định kết nối broadcast của event bằng cách gọi phương thức `broadcastVia` trong hàm khởi tạo của event. Tuy nhiên, trước khi làm như vậy, bạn nên đảm bảo rằng class event đã sử dụng trait `InteractsWithBroadcasting`:

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithBroadcasting;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class OrderShipmentStatusUpdated implements ShouldBroadcast
    {
        use InteractsWithBroadcasting;

        /**
         * Create a new event instance.
         */
        public function __construct()
        {
            $this->broadcastVia('pusher');
        }
    }

<a name="receiving-broadcasts"></a>
## Receiving Broadcasts

<a name="listening-for-events"></a>
### Listening cho Event

Khi bạn đã [cài đặt và khởi tạo Laravel Echo xong](#client-side-installation), bạn đã sẵn sàng để bắt đầu listening các event được broadcast từ ứng dụng Laravel của bạn. Đầu tiên, hãy sử dụng phương thức `channel` để lấy ra một instance của một channel, sau đó gọi phương thức `listen` để listen một event cụ thể:

```js
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

Nếu bạn muốn listen các event trên một private channel, hãy sử dụng phương thức `private`. Bạn có thể khai báo nhiều phương thức `listen` để listen nhiều event trên một channel:

```js
Echo.private(`orders.${this.order.id}`)
    .listen(/* ... */)
    .listen(/* ... */)
    .listen(/* ... */);
```

<a name="stop-listening-for-events"></a>
#### Stop Listening For Events

Nếu bạn muốn dừng listening một event mà không phải [rời khỏi channel](#leaving-a-channel), bạn có thể sử dụng phương thức `stopListening`:

```js
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated')
```

<a name="leaving-a-channel"></a>
### Rời một Channel

Để rời khỏi một channel, bạn có thể gọi phương thức `leaveChannel` trên instance Echo của bạn:

```js
Echo.leaveChannel(`orders.${this.order.id}`);
```

Nếu bạn muốn rời khỏi một channel cũng như các channel riêng tư và presence channel khác, bạn có thể gọi phương thức `leave`:

```js
Echo.leave(`orders.${this.order.id}`);
```
<a name="namespaces"></a>
### Namespaces

Bạn có thể đã nhận thấy trong các ví dụ ở trên, chúng ta không chỉ định namespace `App\Events` cho các class event. Điều này là do Echo đã tự động giả định các event được lưu trong namespace `App\Events`. Tuy nhiên, bạn có thể cấu hình namespace khi bạn khởi tạo Echo bằng cách truyền vào một tùy chọn cấu hình `namespace`:

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    namespace: 'App.Other.Namespace'
});
```

Ngoài ra, bạn có thể thêm tiền tố cho các class event với một dấu `.` khi theo dõi các event bằng Echo. Điều này sẽ cho phép bạn chỉ định tên class:

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        // ...
    });
```

<a name="presence-channels"></a>
## Presence Channel

Các presence channel được xây dựng dựa trên tính bảo mật của các private channel đồng thời thêm các chức năng về những người đã theo dõi channel. Điều này giúp dễ dàng xây dựng các tính năng mạnh mẽ cho application chẳng hạn như thông báo cho người dùng biết khi một người dùng khác đang xem cùng một trang hoặc liệt kê những người đang trong một phòng trò chuyện.

<a name="authorizing-presence-channels"></a>
### Authorizing Presence Channel

Tất cả các presence channel cũng là các private channel; do đó, người dùng phải được [authorized để truy cập đến nó](#authorizing-channels). Tuy nhiên, khi định nghĩa các authorization callback cho các presence channel, bạn sẽ không trả về `true` nếu người dùng đã được authorize tham gia channel. Thay vào đó, bạn nên trả về một mảng dữ liệu về người dùng đó.

Dữ liệu được trả về bởi authorization callback cũng sẽ được cung cấp cho những người khác đang listen event trong presence channel của JavaScript của bạn. Nếu người dùng không được phép tham gia presence channel, bạn nên trả về `false` hoặc `null`:

    use App\Models\User;

    Broadcast::channel('chat.{roomId}', function (User $user, int $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### Tham gia Presence Channel

Để tham gia một presence channel, bạn có thể sử dụng phương thức `join` của Echo. Phương thức `join` sẽ trả về một implementation `PresenceChannel`, cùng với việc thêm phương thức `listen`, cho phép bạn theo dõi các event `here`, `joining`, và `leaving`.

```js
Echo.join(`chat.${roomId}`)
    .here((users) => {
        // ...
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    })
    .error((error) => {
        console.error(error);
    });
```

Callback `here` sẽ được thực hiện ngay sau khi kết nối thành công đến channel và sẽ nhận về một mảng chứa thông tin của tất cả các người dùng đang đăng ký channel. Phương thức `joining` sẽ được thực thi khi một người dùng mới tham gia vào channel, trong khi phương thức `leaving` sẽ được thực thi khi một người dùng rời khỏi channel. Phương thức `error` sẽ được thực thi khi xác thực trả về một HTTP status khác 200 hoặc nếu có sự cố khi phân tích cú pháp JSON được trả về.

<a name="broadcasting-to-presence-channels"></a>
### Broadcasting tới Presence Channel

Các presence channel có thể nhận các event giống như các public hoặc private channel. Ví dụ như về một chatroom, chúng ta có thể muốn broadcast các event `NewMessage` lên một room. Để làm như vậy, chúng ta sẽ trả về một instance của `PresenceChannel` từ phương thức `broadcastOn` của event:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return [
            new PresenceChannel('chat.'.$this->message->room_id),
        ];
    }

Cũng như các event khác, bạn có thể sử dụng helper `broadcast` và phương thức `toOthers` để loại người dùng hiện tại ra khỏi việc nhận broadcast:

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

Là điển hình của các loại event khác, bạn có thể listen các event được gửi đến các presence channel bằng phương thức `listen` của Echo:

```js
Echo.join(`chat.${roomId}`)
    .here(/* ... */)
    .joining(/* ... */)
    .leaving(/* ... */)
    .listen('NewMessage', (e) => {
        // ...
    });
```

<a name="model-broadcasting"></a>
## Model Broadcasting

> [!WARNING]
> Trước khi đọc tài liệu dưới đây về model broadcasting, chúng tôi khuyên bạn nên làm quen với các khái niệm chung về các service model broadcasting của Laravel cũng như cách tạo và lắng nghe các broadcast event.

Thông thường sẽ broadcast các event khi [model Eloquent](/docs/{{version}}/eloquent) của ứng dụng của bạn được tạo, cập nhật hoặc xóa. Tất nhiên, điều này có thể dễ dàng được thực hiện bằng cách [định nghĩa các custom event cho các thay đổi trạng thái của model Eloquent](/docs/{{version}}/eloquent#events) và đánh dấu các event đó bằng interface `ShouldBroadcast`.

Tuy nhiên, nếu bạn không sử dụng các event này cho bất kỳ mục đích nào khác trong ứng dụng của bạn, thì việc tạo các class event cho mục đích duy nhất là broadcasting chúng có thể rất khó khăn. Để khắc phục điều này, Laravel cho phép bạn chỉ ra một Eloquent model sẽ tự động broadcast các thay đổi trạng thái của nó.

Để bắt đầu, model Eloquent của bạn nên sử dụng trait `Illuminate\Database\Eloquent\BroadcastsEvents`. Ngoài ra, model nên định nghĩa một phương thức `broadcastOn`, phương thức này sẽ trả về một mảng các channel mà các event của model đó sẽ broadcast trên đó:

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Database\Eloquent\BroadcastsEvents;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    use BroadcastsEvents, HasFactory;

    /**
     * Get the user that the post belongs to.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the channels that model events should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>
     */
    public function broadcastOn(string $event): array
    {
        return [$this, $this->user];
    }
}
```

Khi model của bạn đã chứa trait này và định nghĩa các channel broadcast nó, nó sẽ bắt đầu tự động broadcast các event khi một instance model được tạo, cập nhật, xóa, soft delete hoặc restore.

Ngoài ra, bạn có thể nhận thấy rằng phương thức `broadcastOn` có nhận được một tham số `$event` là string. Tham số này chứa loại event đã xảy ra trên model và sẽ có giá trị là `created`, `updated`, `deleted`, `trashed` hoặc `restored`. Bằng cách kiểm tra giá trị của biến này, bạn có thể xác định xem channel nào (nếu có) mà model sẽ broadcast với một event cụ thể:

```php
/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<string, array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>>
 */
public function broadcastOn(string $event): array
{
    return match ($event) {
        'deleted' => [],
        default => [$this, $this->user],
    };
}
```

<a name="customizing-model-broadcasting-event-creation"></a>
#### Customizing Model Broadcasting Event Creation

Thỉnh thoảng, bạn có thể muốn tùy chỉnh cách Laravel tạo event model broadcasting. Bạn có thể thực hiện việc này bằng cách định nghĩa phương thức `newBroadcastableEvent` trên model Eloquent của bạn. Phương thức này sẽ trả về một instance `Illuminate\Database\Eloquent\BroadcastableModelEventOccurred`:

```php
use Illuminate\Database\Eloquent\BroadcastableModelEventOccurred;

/**
 * Create a new broadcastable model event for the model.
 */
protected function newBroadcastableEvent(string $event): BroadcastableModelEventOccurred
{
    return (new BroadcastableModelEventOccurred(
        $this, $event
    ))->dontBroadcastToCurrentUser();
}
```

<a name="model-broadcasting-conventions"></a>
### Model Broadcasting Conventions

<a name="model-broadcasting-channel-conventions"></a>
#### Channel Conventions

Như bạn có thể nhận thấy, phương thức `broadcastOn` trong ví dụ model ở trên không trả về các instance `Channel`. Thay vào đó, các model Eloquent được trả về trực tiếp. Nếu một instance model Eloquent được trả về trực tiếp bởi phương thức `broadcastOn` của model của bạn (hoặc được chứa trong một mảng được phương thức này trả về), Laravel sẽ tự động khởi tạo một instance private channel cho model đó bằng cách sử dụng tên class của model và identifier khóa chính của model làm tên channel.

Vì vậy, model `App\Models\User` có `id` là `1` sẽ được chuyển thành instance `Illuminate\Broadcasting\PrivateChannel` với tên là `App.Models.User.1`. Tất nhiên, ngoài việc trả về các instance model Eloquent từ phương thức `broadcastOn` của model, bạn có thể trả về các instance `Channel` hoàn chỉnh để có toàn quyền kiểm soát tên channel của model:

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(string $event): array
{
    return [
        new PrivateChannel('user.'.$this->id)
    ];
}
```

Nếu bạn định trả về một instance channel từ phương thức `broadcastOn` của model, thì bạn có thể truyền một instance model Eloquent cho hàm khởi tạo của channel. Khi làm như vậy, Laravel sẽ sử dụng các quy ước về model channel đã thảo luận ở trên để chuyển đổi model Eloquent thành chuỗi tên channel:

```php
return [new Channel($this->user)];
```

Nếu bạn cần xác định xem tên channel của một model, bạn có thể gọi phương thức `broadcastChannel` trên bất kỳ instance model nào. Ví dụ: phương thức này trả về chuỗi `App.Models.User.1` cho một model `App\Models\User` với `id` là `1`:

```php
$user->broadcastChannel()
```

<a name="model-broadcasting-event-conventions"></a>
#### Event Conventions

Vì các event model broadcast không được liên kết với một event "thực tế" trong thư mục `App\Events` của ứng dụng của bạn, nên chúng được gán một tên và payload dựa trên các quy ước. Quy ước của Laravel là broadcast event bằng cách sử dụng tên class của model (không bao gồm namespace) và tên của event model đã kích hoạt broadcast.

Vì vậy, ví dụ: cập nhật của model `App\Models\Post` sẽ broadcast ra một event tới ứng dụng phía client của bạn dưới dạng là `PostUpdated` với payload như sau:

```json
{
    "model": {
        "id": 1,
        "title": "My first post"
        ...
    },
    ...
    "socket": "someSocketId",
}
```

Việc xóa một model `App\Models\User` sẽ broadcast ra một event có tên là `UserDeleted`.

Nếu muốn, bạn có thể định nghĩa một tuỳ biến tên broadcast và payload của nó bằng cách thêm hai phương thức `broadcastAs` và `broadcastWith` vào model của bạn. Các phương thức này nhận vào tên của event và hoạt động model đang diễn ra, cho phép bạn tùy biến tên và payload của event cho từng hoạt động của model. Nếu `null` được trả về từ phương thức `broadcastAs`, Laravel sẽ sử dụng các quy ước tên event model broadcast đã thảo luận ở trên khi broadcasting event:

```php
/**
 * The model event's broadcast name.
 */
public function broadcastAs(string $event): string|null
{
    return match ($event) {
        'created' => 'post.created',
        default => null,
    };
}

/**
 * Get the data to broadcast for the model.
 *
 * @return array<string, mixed>
 */
public function broadcastWith(string $event): array
{
    return match ($event) {
        'created' => ['title' => $this->title],
        default => ['model' => $this],
    };
}
```

<a name="listening-for-model-broadcasts"></a>
### Listening For Model Broadcasts

Khi bạn đã thêm trait `BroadcastsEvents` vào model của bạn và định nghĩa phương thức `broadcastOn` của model, bạn đã sẵn sàng bắt đầu lắng nghe các event model broadcast trong ứng dụng bên phía client của bạn. Trước khi bắt đầu, bạn có thể muốn tham khảo tài liệu về [lắng nghe event](#listening-for-events).

Trước tiên, hãy sử dụng phương thức `private` để lấy ra một instance của channel, sau đó gọi phương thức `listen` để lắng nghe một event cụ thể. Thông thường, tên channel sẽ được set cho phương thức `private` phải tương ứng với [quy ước model broadcast](#model-broadcasting-conventions) của Laravel.

Sau khi bạn đã nhận được instance channel, bạn có thể sử dụng phương thức `listen` để lắng nghe một event cụ thể. Vì các event model broadcast không được liên kết với một event "thực tế" nào trong thư mục `App\Events` của ứng dụng của bạn, nên [tên event](#model-broadcasting-event-conventions) phải có tiền tố là `.` để biểu thị nó không thuộc về một namespace cụ thể nào. Mỗi event model broadcast có một thuộc tính `model` chứa tất cả các thuộc tính có thể broadcast của model:

```js
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.PostUpdated', (e) => {
        console.log(e.model);
    });
```

<a name="client-events"></a>
## Client Event

> [!NOTE]
> Khi sử dụng [Pusher Channels](https://pusher.com/channels), bạn phải bật tùy chọn "Client Events" trong phần "App Settings" của [bảng điều khiển ứng dụng](https://dashboard.pusher.com/) để gửi các client event.

Thỉnh thoảng bạn có thể muốn broadcast một event cho những client được kết nối khác mà không cần gọi application Laravel của bạn. Điều này có thể đặc biệt hữu ích cho những việc như thông báo "đang gõ", bạn muốn thông báo cho người dùng application của bạn rằng có một người dùng khác đang gõ một tin nhắn trên màn hình.

Để broadcast các client event, bạn có thể sử dụng phương thức `whisper` của Echo:

```js
Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        name: this.user.name
    });
```

Để listen các client event, bạn có thể sử dụng phương thức `listenForWhisper`:

```js
Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

<a name="notifications"></a>
## Notification

Bằng cách kết nối event broadcasting với [notifications](/docs/{{version}}/notifications), JavaScript của bạn có thể nhận được thông báo mới khi chúng xảy ra mà không cần refresh trang. Trước khi bắt đầu, hãy nhớ đọc tài liệu về việc sử dụng [channel thông báo broadcast](/docs/{{version}}/notifications#broadcast-notifications).

Khi bạn đã cấu hình thông báo sử dụng broadcast channel, bạn có thể listen các broadcast event bằng phương thức `notification` của Echo. Hãy nhớ rằng, tên channel phải khớp với tên class nhận thông báo:

```js
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

Trong ví dụ trên, tất cả các thông báo được gửi đến instance `App\Models\User` thông qua channel `broadcast` sẽ được nhận được thông qua hàm callback. Một callback authorization cho channel `App.Models.User.{id}` sẽ có sẵn trong `BroadcastServiceProvider` đi kèm với framework Laravel.
