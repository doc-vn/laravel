# Contracts

- [Giới thiệu](#introduction)
    - [Contracts và Facades](#contracts-vs-facades)
- [Khi nào dùng Contracts](#when-to-use-contracts)
    - [Loose Coupling](#loose-coupling)
    - [Simplicity](#simplicity)
- [Làm thế nào để dùng Contracts](#how-to-use-contracts)
- [Contract Reference](#contract-reference)

<a name="introduction"></a>
## Giới thiệu

Contract của Laravel là một bộ interface dùng để định nghĩa các core service mà framework cung cấp. Ví dụ: một `Illuminate\Contracts\Queue\Queue` contract định nghĩa các phương thức cần thiết cho việc queueing job, trong khi `Illuminate\Contracts\Mail\Mailer` contract định nghĩa các phương thức cần thiết để gửi e-mail.

Mỗi contract có một implementation tương ứng được cung cấp bởi framework. Ví dụ: Laravel cung cấp một implementation queue với nhiều driver khác nhau và một implementation mailer được phát triển bởi [SwiftMailer](https://swiftmailer.symfony.com/).

Tất cả các contract của Laravel đều được lưu trữ trong [GitHub của chúng](https://github.com/illuminate/contracts). Điều này cung cấp một điểm tham chiếu nhanh cho tất cả các contract có sẵn, đồng thời, các nhà phát triển package có thể sử dụng các package một cách riêng rẽ và độc lập.

<a name="contracts-vs-facades"></a>
### Contracts và Facades

[Facade](/docs/{{version}}/facades) của Laravel và các helper function sẽ cung cấp một cách đơn giản để sử dụng các service của Laravel mà không cần phải khai báo và resolve các contract ra khỏi container service. Trong hầu hết các trường hợp, mỗi facade có một contract tương ứng.

Không giống như facade, không yêu cầu bạn phải khởi tạo chúng trong hàm khởi tạo của class, các contract cho phép bạn định nghĩa rõ các phụ thuộc cho các class của bạn. Một số nhà phát triển thích định nghĩa rõ sự phụ thuộc trong class của họ giống như cách này và do đó họ thích sử dụng contract, trong khi các nhà phát triển khác lại thích sự tiện lợi của facade.

> {tip} Hầu hết các ứng dụng sẽ hoạt động tốt bất kể bạn thích facade hay contract. Tuy nhiên, nếu bạn đang xây dựng một package, bạn nên cân nhắc việc sử dụng các contract vì chúng sẽ dễ kiểm tra hơn trong bối cảnh là một package.

<a name="when-to-use-contracts"></a>
## Khi nào dùng Contract

Như đã nói ở trên, phần lớn quyết định sử dụng contract hay là facade sẽ tùy thuộc vào sở thích cá nhân và thị hiếu của nhóm phát triển. Cả contract và facade đều có thể được sử dụng để tạo ra application Laravel mạnh mẽ và được thử nghiệm tốt. Miễn là bạn giữ cho class trong giới hạn của nó, bạn sẽ nhận thấy có rất ít sự khác biệt giữa việc sử dụng contract và facade.

Tuy nhiên, bạn vẫn có thể có một số câu hỏi liên quan đến contract. Ví dụ, tại sao lại sử dụng interface? Việc sử dụng interface có phức tạp hơn không? Các lý do để sử dụng interface sẽ được thể hiện trong các mục dưới đây: liên kết lỏng và tính đơn giản.

<a name="loose-coupling"></a>
### Liên kết lỏng

Trước tiên, hãy xem xét một số code có liên kết chặt chẽ tới một implementation của cache. Cùng xem ví dụ sau:

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Retrieve an Order by ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

Trong class này, code có liên kết chặt chẽ tới một implementation của cache đã cho. Nó được liên kết chặt chẽ bởi vì chúng ta phụ thuộc vào class Cache từ một package vendor. Nếu API của package đó thay đổi, code của chúng ta cũng phải thay đổi.

Tương tự như vậy, nếu chúng ta muốn thay đổi loại cache (Memcached) sang một loại khác như (Redis), một lần nữa chúng ta sẽ phải thay đổi source code của mình. Source code của chúng ta không nên biết quá nhiều về người cung cấp dữ liệu hoặc làm thế nào để họ cung cấp dữ liệu đó.

**Thay vì cách tiếp cận trên, chúng ta có thể cải thiện code của mình bằng cách phụ thuộc vào một interface đơn giản mà không có bất kỳ liên quan gì đến package vendor:**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

Bây giờ code của chúng ta sẽ không còn được liên kết với bất kỳ vendor nào nữa, hoặc thậm chí là Laravel. Vì contracts package không chứa implementation và cũng không chứa phụ thuộc nào, bạn có thể dễ dàng viết một implementation thay thế cho bất kỳ contract nào, cho phép bạn thay thế việc implementation của cache mà không phải sửa đổi bất kỳ mã nguồn nào mà đang sử dụng đoạn code cache ở trên.

<a name="simplicity"></a>
### Tính đơn giản

Khi tất cả các service của Laravel được định nghĩa vào trong các interface đơn giản, thì sẽ dễ dàng hơn để xác định chức năng mà được cung cấp bởi service đó. **Các contract đóng vai trò là một tài liệu ngắn gọn cho các tính năng của framework.**

Ngoài ra, khi bạn phụ thuộc vào các interface đơn giản, code của bạn sẽ dễ hiểu và dễ bảo trì hơn. Thay vì bạn phải tìm các phương thức nào có thể sử dụng trong một class lớn và phức tạp, thì bạn có thể tham khảo một interface đơn giản, rõ ràng.

<a name="how-to-use-contracts"></a>
## Làm thế nào để dùng Contract

Vậy, làm thế nào để bạn có thể lấy ra được một implementation của một contract? Nó thực sự khá đơn giản.

Có nhiều loại class trong Laravel được resolve thông qua [service container](/docs/{{version}}/container), bao gồm cả controller, event listener, middleware, queued job và thậm chí là route Closure. Vì vậy, để lấy được một implementation của một contract, bạn chỉ cần khai báo interface vào trong hàm khởi tạo của class mà cần được resolve.

Ví dụ, hãy xem event listener này:

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\OrderWasPlaced;
    use Illuminate\Contracts\Redis\Factory;

    class CacheOrderInformation
    {
        /**
         * The Redis factory implementation.
         */
        protected $redis;

        /**
         * Create a new event handler instance.
         *
         * @param  Factory  $redis
         * @return void
         */
        public function __construct(Factory $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Handle the event.
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

Khi event listener được resolve, service container sẽ đọc các khai báo có trong hàm khởi tạo của class và đưa vào các giá trị phù hợp. Để tìm hiểu thêm về việc đăng ký trong service container, hãy xem [tài liệu này](/docs/{{version}}/container).

<a name="contract-reference"></a>
## Contract tham khảo

Bảng này cung cấp một tài liệu tham khảo nhanh cho tất cả các contract của Laravel và các facade tương ứng với chúng:

Contract  |  References Facade
------------- | -------------
[Illuminate\Contracts\Auth\Access\Authorizable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Authorizable.php) | &nbsp;
[Illuminate\Contracts\Auth\Access\Gate](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Gate.php) | `Gate`
[Illuminate\Contracts\Auth\Authenticatable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Authenticatable.php) | &nbsp;
[Illuminate\Contracts\Auth\CanResetPassword](https://github.com/illuminate/contracts/blob/{{version}}/Auth/CanResetPassword.php) | &nbsp;
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php) | `Auth`
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Guard.php) | `Auth::guard()`
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php) | `Password::broker()`
[Illuminate\Contracts\Auth\PasswordBrokerFactory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBrokerFactory.php) | `Password`
[Illuminate\Contracts\Auth\StatefulGuard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/StatefulGuard.php) | &nbsp;
[Illuminate\Contracts\Auth\SupportsBasicAuth](https://github.com/illuminate/contracts/blob/{{version}}/Auth/SupportsBasicAuth.php) | &nbsp;
[Illuminate\Contracts\Auth\UserProvider](https://github.com/illuminate/contracts/blob/{{version}}/Auth/UserProvider.php) | &nbsp;
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php) | `Bus`
[Illuminate\Contracts\Bus\QueueingDispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/QueueingDispatcher.php) | `Bus::dispatchToQueue()`
[Illuminate\Contracts\Broadcasting\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Factory.php) | `Broadcast`
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php)  | `Broadcast::connection()`
[Illuminate\Contracts\Broadcasting\ShouldBroadcast](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcast.php) | &nbsp;
[Illuminate\Contracts\Broadcasting\ShouldBroadcastNow](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcastNow.php) | &nbsp;
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php) | `Cache`
[Illuminate\Contracts\Cache\Lock](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Lock.php) | &nbsp;
[Illuminate\Contracts\Cache\LockProvider](https://github.com/illuminate/contracts/blob/{{version}}/Cache/LockProvider.php) | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php) | `Cache::driver()`
[Illuminate\Contracts\Cache\Store](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Store.php) | &nbsp;
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php) | `Config`
[Illuminate\Contracts\Console\Application](https://github.com/illuminate/contracts/blob/{{version}}/Console/Application.php) | &nbsp;
[Illuminate\Contracts\Console\Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Console/Kernel.php) | `Artisan`
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php) | `App`
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php) | `Cookie`
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php) | `Cookie::queue()`
[Illuminate\Contracts\Database\ModelIdentifier](https://github.com/illuminate/contracts/blob/{{version}}/Database/ModelIdentifier.php) | &nbsp;
[Illuminate\Contracts\Debug\ExceptionHandler](https://github.com/illuminate/contracts/blob/{{version}}/Debug/ExceptionHandler.php) | &nbsp;
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php) | `Crypt`
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php) | `Event`
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php) | `Storage::cloud()`
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php) | `Storage`
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php) | `Storage::disk()`
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php) | `App`
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php) | `Hash`
[Illuminate\Contracts\Http\Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Http/Kernel.php) | &nbsp;
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php) | `Mail::queue()`
[Illuminate\Contracts\Mail\Mailable](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailable.php) | &nbsp;
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php) | `Mail`
[Illuminate\Contracts\Notifications\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Dispatcher.php) | `Notification`
[Illuminate\Contracts\Notifications\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Factory.php) | `Notification`
[Illuminate\Contracts\Pagination\LengthAwarePaginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/LengthAwarePaginator.php) | &nbsp;
[Illuminate\Contracts\Pagination\Paginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/Paginator.php) | &nbsp;
[Illuminate\Contracts\Pipeline\Hub](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Hub.php) | &nbsp;
[Illuminate\Contracts\Pipeline\Pipeline](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Pipeline.php) | &nbsp;
[Illuminate\Contracts\Queue\EntityResolver](https://github.com/illuminate/contracts/blob/{{version}}/Queue/EntityResolver.php) | &nbsp;
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php) | `Queue`
[Illuminate\Contracts\Queue\Job](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Job.php) | &nbsp;
[Illuminate\Contracts\Queue\Monitor](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Monitor.php) | `Queue`
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php) | `Queue::connection()`
[Illuminate\Contracts\Queue\QueueableCollection](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableCollection.php) | &nbsp;
[Illuminate\Contracts\Queue\QueueableEntity](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableEntity.php) | &nbsp;
[Illuminate\Contracts\Queue\ShouldQueue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/ShouldQueue.php) | &nbsp;
[Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Factory.php) | `Redis`
[Illuminate\Contracts\Routing\BindingRegistrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/BindingRegistrar.php) | `Route`
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php) | `Route`
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php) | `Response`
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php) | `URL`
[Illuminate\Contracts\Routing\UrlRoutable](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlRoutable.php) | &nbsp;
[Illuminate\Contracts\Session\Session](https://github.com/illuminate/contracts/blob/{{version}}/Session/Session.php) | `Session::driver()`
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Htmlable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Htmlable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\MessageBag](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageBag.php) | &nbsp;
[Illuminate\Contracts\Support\MessageProvider](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageProvider.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Support\Responsable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Responsable.php) | &nbsp;
[Illuminate\Contracts\Translation\Loader](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Loader.php) | &nbsp;
[Illuminate\Contracts\Translation\Translator](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Translator.php) | `Lang`
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php) | `Validator`
[Illuminate\Contracts\Validation\ImplicitRule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ImplicitRule.php) | &nbsp;
[Illuminate\Contracts\Validation\Rule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Rule.php) | &nbsp;
[Illuminate\Contracts\Validation\ValidatesWhenResolved](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ValidatesWhenResolved.php) | &nbsp;
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php) | `Validator::make()`
[Illuminate\Contracts\View\Engine](https://github.com/illuminate/contracts/blob/{{version}}/View/Engine.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php) | `View`
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php) | `View::make()`
