# Service Container

- [Giới thiệu](#introduction)
    - [Injection không cần cấu hình](#zero-configuration-resolution)
    - [Khi nào nên sử dụng container](#when-to-use-the-container)
- [Liên kết](#binding)
    - [Liên kết cơ bản](#binding-basics)
    - [Liên kết Interfaces tới Implementations](#binding-interfaces-to-implementations)
    - [Liên kết bối cảnh](#contextual-binding)
    - [Liên kết kiểu dữ liệu đơn giản](#binding-primitives)
    - [Liên kết nhiều loại](#binding-typed-variadics)
    - [Thẻ](#tagging)
    - [Liên kết mở rộng](#extending-bindings)
- [Resolving](#resolving)
    - [Tạo phương thức](#the-make-method)
    - [Automatic Injection](#automatic-injection)
- [Khởi động hàm và injection](#method-invocation-and-injection)
- [Container Event](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## Giới thiệu

Laravel service container là một công cụ mạnh mẽ để quản lý các class phụ thuộc và thực hiện tích hợp các class phụ thuộc đó vào các class khác. Tích hợp class phụ thuộc là một cụm từ tuyệt vời có nghĩa cơ bản là: class phụ thuộc sẽ được "tích hợp" vào một class khác thông qua hàm khởi tạo hoặc trong một số trường hợp là hàm "setter".

Hãy xem một ví dụ đơn giản:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Repositories\UserRepository;
    use App\Models\User;

    class UserController extends Controller
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

Trong ví dụ trên, `UserController` sẽ cần lấy user từ một data source. Vì vậy, chúng ta sẽ **tích hợp** một service có thể lấy user. Theo ngữ cảnh này, trong class `UserRepository` của chúng ta có thể sử dụng [Eloquent](/docs/{{version}}/eloquent) để lấy thông tin user trực tiếp từ database. Tuy nhiên, vì repository đã được tích hợp, nên chúng ta có thể dễ dàng chuyển việc đó với một implementation khác. Và chúng ta cũng có thể dễ dàng "làm giả", hoặc tạo một implementation giả của `UserRepository` khi test application của chúng ta.

Hiểu sâu về Laravel service container sẽ một điều cần thiết để tạo một application lớn, mạnh mẽ, cũng như phát triển phần lõi của Laravel.

<a name="zero-configuration-resolution"></a>
### Injection không cần cấu hình

Nếu có một class mà không phụ thuộc hoặc chỉ phụ thuộc vào các class cụ thể (không phải interface), container sẽ không cần phải hướng dẫn về cách resolve ra class đó. Ví dụ: bạn có thể viết đoạn mã sau vào file `routes/web.php` của bạn:

    <?php

    class Service
    {
        //
    }

    Route::get('/', function (Service $service) {
        die(get_class($service));
    });

Trong ví dụ này, nhấn vào route `/` của ứng dụng sẽ tự động resolve class `Service` và đưa nó vào trong xử lý route của bạn. Đây là điều sẽ thay đổi cuộc chơi. Điều đó có nghĩa là bạn có thể phát triển ứng dụng của bạn và tận dụng tính năng injection mà không phải lo lắng về các file cấu hình sẽ bị cồng kềnh.

Rất may, nhiều class bạn sẽ cần phải viết khi xây dựng ứng dụng của mình sẽ được tự động nhận các phụ thuộc của chúng thông qua container, bao gồm [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [middleware](/docs/{{version}}/middleware), và nhiều hơn thế. Ngoài ra, bạn có thể khai báo phụ thuộc vào trong phương thức `handle` của [queued jobs](/docs/{{version}}/queues). Một khi bạn đã trải nghiệm sức mạnh của việc injection phụ thuộc tự động mà không cần phải cấu hình, bạn sẽ cảm thấy không thể phát triển nếu thiếu nó.

<a name="when-to-use-the-container"></a>
### Khi nào nên sử dụng container

Nhờ vào việc injection mà không cần cấu hình, bạn sẽ thường xuyên khai báo các phụ thuộc trên routes, controllers, event listeners, và các nơi khác mà không cần tương tác với container. Ví dụ: bạn có thể khai báo đối tượng `Illuminate\Http\Request` trên định nghĩa route của bạn để bạn có thể dễ dàng truy cập vào request hiện tại. Mặc dù chúng ta không bao giờ phải tương tác với container để viết những code này, nhưng nó đang quản lý việc inject các phụ thuộc này ở trong hậu trường:

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        // ...
    });

Trong nhiều trường hợp, nhờ tính năng injection phụ thuộc tự động và [facades](/docs/{{version}}/facades), bạn có thể xây dựng các ứng dụng Laravel mà **không cần** liên kết hoặc resolve thủ công bất kỳ thứ gì từ container. **Vậy, khi nào bạn sẽ phải tương tác với container?** Hãy xem xét hai tình huống sau.

Đầu tiên, nếu bạn phải viết một class mà implement kauh một interface và bạn muốn khai báo interface đó vào trong một route hoặc hàm khởi tạo của một class, bạn phải [cho container biết cách resolve interface đó](#binding-interfaces-to-implementations). Thứ hai, nếu bạn đang [viết một package Laravel](/docs/{{version}}/packages) và bạn dự định chia sẻ với các nhà phát triển Laravel khác, bạn có thể cần phải liên kết các service của package của bạn vào container.

<a name="binding"></a>
## Liên kết

<a name="binding-basics"></a>
### Liên kết cơ bản

<a name="simple-bindings"></a>
#### Simple Bindings

Hầu như tất cả các liên kết của service container sẽ được đăng ký trong [service providers](/docs/{{version}}/providers), nên vì thế hầu hết các ví dụ này sẽ được thực hiện bằng cách sử dụng container trong ngữ cảnh này.

Trong một service provider, bạn luôn có quyền truy cập vào container thông qua thuộc tính `$this->app`. Chúng ta có thể đăng ký một liên kết bằng cách sử dụng phương thức `bind`, truyền tên class hoặc tên interface mà chúng ta muốn đăng ký cùng với một closure trả về một instance của class:

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $this->app->bind(Transistor::class, function ($app) {
        return new Transistor($app->make(PodcastParser::class));
    });

Lưu ý rằng chúng ta nhận container vào như là một tham số resolver. Sau đó chúng ta có thể sử dụng chính container đó để resolve các phụ thuộc con của đối tượng mà chúng ta đang xây dựng. Như ví dụ ở trên thì tham số của container chính là `$app`, chúng ta nhận tham số đó vào và resolve thêm một phụ thuộc con là `HttpClient` để tạo ra một instance HelpSpot\API mới và trả về với tên là `HelpSpot\API`.

Như đã đề cập, thông thường bạn sẽ tương tác với container bên trong các service provider; tuy nhiên, nếu bạn muốn tương tác với container bên ngoài service provider, bạn có thể làm như sau thông qua `App` [facade](/docs/{{version}}/facades):

    use App\Services\Transistor;
    use Illuminate\Support\Facades\App;

    App::bind(Transistor::class, function ($app) {
        // ...
    });

> {tip} Không cần phải liên kết các class vào container nếu chúng không phụ thuộc vào bất kỳ interface nào. Bạn không cần phải hướng dẫn container về cách xây dựng các đối tượng này, vì nó có thể tự động resolve các đối tượng này bằng cách sử dụng tham chiếu.

<a name="binding-a-singleton"></a>
#### Liên kết singleton

Phương thức `singleton` sẽ liên kết một class hoặc một interface vào trong container và chỉ resolve nó một lần duy nhất. Khi một liên kết singleton đã được resolve, thì lần tiếp theo khi gọi vào container thì đối tượng đó sẽ được trả về:

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $this->app->singleton(Transistor::class, function ($app) {
        return new Transistor($app->make(PodcastParser::class));
    });

<a name="binding-scoped"></a>
#### Binding Scoped Singletons

Phương thức `scoped` sẽ liên kết một class hoặc một interface vào container và chỉ được resolve một lần trong cả vòng đời request hoặc một job Laravel nhất định. Mặc dù phương thức này tương tự như phương thức `singleton`, nhưng các instance đã đăng ký sử dụng phương thức `scoped` sẽ bị xóa bất cứ khi nào ứng dụng Laravel bắt đầu một "vòng đời" mới, chẳng hạn như khi một [Laravel Octane](/docs/{{version}}/octane) worker xử lý một request mới hoặc khi Laravel [queue worker](/docs/{{version}}/queues) xử lý một job mới:

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $this->app->scoped(Transistor::class, function ($app) {
        return new Transistor($app->make(PodcastParser::class));
    });

<a name="binding-instances"></a>
#### Liên kết instances

Bạn cũng có thể liên kết một object instance đã tồn tại vào container bằng cách sử dụng phương thức `instance`. Và instance đó sẽ luôn được trả về cho các lần gọi tiếp theo vào container:

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $service = new Transistor(new PodcastParser);

    $this->app->instance(Transistor::class, $service);

<a name="binding-interfaces-to-implementations"></a>
### Liên kết Interfaces tới Implementations

Một tính năng rất mạnh mẽ của service container là khả năng liên kết một interface tới một implementation nhất định. Ví dụ: giả sử chúng ta có interface `EventPusher` và implementation `RedisEventPusher`. Khi mà chúng ta đã code xong implementation `RedisEventPusher` của interface, chúng ta có thể đăng ký nó với service container như sau:

    use App\Contracts\EventPusher;
    use App\Services\RedisEventPusher;

    $this->app->bind(EventPusher::class, RedisEventPusher::class);

Câu lệnh trên sẽ nói với container rằng nó cần tích hợp `RedisEventPusher` vào một class nếu class đó cần một implementation của interface `EventPusher`. Bây giờ chúng ta có thể viết interface `EventPusher` vào hàm khởi tạo của một class và được resolve bởi container. Hãy nhớ rằng, controllers, event listeners, middleware, và nhiều loại class khác trong ứng dụng Laravel luôn được resolve bằng cách sử dụng container:

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  \App\Contracts\EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Liên kết theo ngữ cảnh

Thỉnh thoảng bạn cũng có thể có hai class sử dụng chung một interface, nhưng bạn lại muốn tích hợp các implementation khác nhau đó vào các class khác nhau. Ví dụ, có hai controller bị phụ thuộc vào các implementation khác nhau của class `Illuminate\Contracts\Filesystem\Filesystem` [contract](/docs/{{version}}/contracts). Laravel cung cấp một interface đơn giản, và dễ dàng để thực hiện hành vi này:

    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\UploadController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;
    use Illuminate\Support\Facades\Storage;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when([VideoController::class, UploadController::class])
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="binding-primitives"></a>
### Liên kết kiểu dữ liệu đơn giản

Thỉnh thoảng, bạn có một class nhận vào một số các class tích hợp, nhưng bạn cũng có thể muốn thêm một số các giá trị khác nhau để thêm vào những class đó, ví dụ như là một giá trị integer. Bạn có thể dễ dàng sử dụng liên kết theo ngữ cảnh đó để đưa vào một giá trị mà class của bạn có thể cần:

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

Thỉnh thoảng một class có thể gắn vào một mảng các instance đã được [gắn tag](#tagging). Sử dụng phương thức `giveTagged`, bạn có thể dễ dàng gắn tất cả các liên kết container này với tag đó:

    $this->app->when(ReportAggregator::class)
        ->needs('$reports')
        ->giveTagged('reports');

Nếu bạn cần inject một giá trị từ một trong các file cấu hình của ứng dụng, bạn có thể sử dụng phương thức `giveConfig`:

    $this->app->when(ReportAggregator::class)
        ->needs('$timezone')
        ->giveConfig('app.timezone');

<a name="binding-typed-variadics"></a>
### Liên kết nhiều loại

Đôi khi, bạn có thể có một class nhận vào một mảng các đối tượng thông qua khai báo một tham số trong phương thức khởi tạo của class:

    <?php

    use App\Models\Filter;
    use App\Services\Logger;

    class Firewall
    {
        /**
         * The logger instance.
         *
         * @var \App\Services\Logger
         */
        protected $logger;

        /**
         * The filter instances.
         *
         * @var array
         */
        protected $filters;

        /**
         * Create a new class instance.
         *
         * @param  \App\Services\Logger  $logger
         * @param  array  $filters
         * @return void
         */
        public function __construct(Logger $logger, Filter ...$filters)
        {
            $this->logger = $logger;
            $this->filters = $filters;
        }
    }

Sử dụng liên kết theo ngữ cảnh đó, bạn có thể resolve sự phụ thuộc này bằng cách cung cấp phương thức `give` với một closure trả về một mảng các instance `Filter`:

    $this->app->when(Firewall::class)
              ->needs(Filter::class)
              ->give(function ($app) {
                    return [
                        $app->make(NullFilter::class),
                        $app->make(ProfanityFilter::class),
                        $app->make(TooLongFilter::class),
                    ];
              });

Để thuận tiện, bạn cũng có thể chỉ cần cung cấp một mảng tên class để container resolve bất cứ khi nào `Firewall` cần các instances `Filter`:

    $this->app->when(Firewall::class)
              ->needs(Filter::class)
              ->give([
                  NullFilter::class,
                  ProfanityFilter::class,
                  TooLongFilter::class,
              ]);

<a name="variadic-tag-dependencies"></a>
#### Variadic Tag Dependencies

Thỉnh thoảng một class có thể có nhiều phụ thuộc khác nhau được khai báo như một class (`Report ...$reports`). Sử dụng các phương thức `needs` và `giveTagged`, bạn có thể dễ dàng gắn tất cả các liên kết container này với một [tag](#tagging) đã cho:

    $this->app->when(ReportAggregator::class)
        ->needs(Report::class)
        ->giveTagged('reports');

<a name="tagging"></a>
### Thẻ

Đôi khi, bạn có thể cần phải resolve tất cả một "category" liên kết. Ví dụ, giả sử bạn đang xây dựng một report phân tích nhận vào một mảng gồm nhiều implementation khác nhau của interface `Report`. Sau khi đăng ký các implementation của interface `Report`, bạn có thể gán cho chúng vào một thẻ bằng phương thức `tag`:

    $this->app->bind(CpuReport::class, function () {
        //
    });

    $this->app->bind(MemoryReport::class, function () {
        //
    });

    $this->app->tag([CpuReport::class, MemoryReport::class], 'reports');

Khi các service đã được gắn thẻ, bạn có thể dễ dàng resolve tất cả chúng thông qua phương thức `tagged` của container:

    $this->app->bind(ReportAnalyzer::class, function ($app) {
        return new ReportAnalyzer($app->tagged('reports'));
    });

<a name="extending-bindings"></a>
### Liên kết mở rộng

Phương thức `extend` cho phép sửa đổi các service đã được resolve. Ví dụ: khi một service đã được resolve, bạn có thể chạy thêm code để bổ sung hoặc cấu hình service đó. Phương thức `extend` chấp nhận một closure, sẽ trả về một service đã được sửa đổi. Closure này sẽ nhận vào một service đang được resolve và một instance container:

    $this->app->extend(Service::class, function ($service, $app) {
        return new DecoratedService($service);
    });

<a name="resolving"></a>
## Resolving

<a name="the-make-method"></a>
### Phương thức `make`

Bạn có thể sử dụng phương thức `make` để resolve một instance của class từ container. Phương thức `make` sẽ chấp nhận một tên của một class hoặc một interface mà bạn muốn resolve:

    use App\Services\Transistor;

    $transistor = $this->app->make(Transistor::class);

Nếu một số phụ thuộc trong class của bạn không thể resolve được thông qua container, bạn có thể inject chúng vào bằng cách truyền chúng dưới dạng một mảng vào phương thức `makeWith`. Ví dụ: chúng ta có thể truyền tham số khởi tạo `$id` trực tiếp theo yêu cầu của service `Transistor`:

    use App\Services\Transistor;

    $transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);

Nếu bạn ở ngoài service provider, ở vị trí mà code của bạn không có quyền truy cập vào biến `$app`, thì bạn có thể sử dụng `App` [facade](/docs/{{version}}/facades) để resolve một instance của class từ container:

    use App\Services\Transistor;
    use Illuminate\Support\Facades\App;

    $transistor = App::make(Transistor::class);

Nếu bạn muốn instance container Laravel cũng được inject vào class mà đang được container resolve, bạn có thể khai báo class `Illuminate\Container\Container` trong hàm khởi tạo class của bạn:

    use Illuminate\Container\Container;

    /**
     * Create a new class instance.
     *
     * @param  \Illuminate\Container\Container  $container
     * @return void
     */
    public function __construct(Container $container)
    {
        $this->container = $container;
    }

<a name="automatic-injection"></a>
### Tự động tích hợp

Ngoài ra, và rất quan trọng, bạn có thể khai báo sự phụ thuộc vào trong hàm khởi tạo để nó có thể được resolve bởi container, như ở trong [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [middleware](/docs/{{version}}/middleware), và nhiều lớp khác. Ngoài ra, bạn có thể khai báo phụ thuộc ở trong phương thức `handle` của [queued job](/docs/{{version}}/queues). Trong thực tế, đây là cách mà hầu hết các đối tượng của bạn sẽ được resolve bằng container.

Ví dụ: bạn có thể khai báo một repository của bạn trong hàm khởi tạo của một controller. Repository đó sẽ tự động được resolve và đưa vào trong class:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         *
         * @var \App\Repositories\UserRepository
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  \App\Repositories\UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="method-invocation-and-injection"></a>
## Khởi động hàm và injection

Thỉnh thoảng, bạn có thể muốn gọi một phương thức trên một instance đối tượng trong khi cho phép container tự động inject các phụ thuộc trong phương thức đó. Ví dụ: như class sau:

    <?php

    namespace App;

    use App\Repositories\UserRepository;

    class UserReport
    {
        /**
         * Generate a new user report.
         *
         * @param  \App\Repositories\UserRepository  $repository
         * @return array
         */
        public function generate(UserRepository $repository)
        {
            // ...
        }
    }

Bạn có thể gọi phương thức `generate` thông qua container như sau:

    use App\UserReport;
    use Illuminate\Support\Facades\App;

    $report = App::call([new UserReport, 'generate']);

Phương thức `call` sẽ chấp nhận bất kỳ PHP callable nào. Phương thức `call` của container thậm chí có thể được sử dụng để gọi một closure trong khi đang tự động inject các phụ thuộc của nó:

    use App\Repositories\UserRepository;
    use Illuminate\Support\Facades\App;

    $result = App::call(function (UserRepository $repository) {
        // ...
    });

<a name="container-events"></a>
## Container Event

Service container sẽ kích hoạt một event mỗi khi nó resolve một đối tượng. Bạn có thể listen event này bằng phương thức `resolving`:

    use App\Services\Transistor;

    $this->app->resolving(Transistor::class, function ($transistor, $app) {
        // Called when container resolves objects of type "Transistor"...
    });

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

Như bạn có thể thấy, đối tượng đang được resolve sẽ được truyền vào một hàm callback, cho phép bạn đặt thêm bất kỳ thuộc tính nào vào trong đối tượng trước khi nó được trả về cho người resolve nó.

<a name="psr-11"></a>
## PSR-11

Service container của Laravel là một implement của một interface [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Do đó, bạn có thể khai báo một interface container PSR-11 để có được một instance của container Laravel:

    use App\Services\Transistor;
    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get(Transistor::class);

        //
    });

Một ngoại lệ sẽ được đưa ra nếu định dang đã cho không thể resolve được. Ngoại lệ này sẽ là một instance của `Psr\Container\NotFoundExceptionInterface` nếu định dang này không bị ràng buộc. Nếu định dang này bị ràng buộc nhưng không thể resolve được, thì một instance của `Psr\Container\ContainerExceptionInterface` sẽ được đưa ra.
