# Service Container

- [Giới thiệu](#introduction)
- [Liên kết](#binding)
    - [Liên kết cơ bản](#binding-basics)
    - [Liên kết Interfaces tới Implementations](#binding-interfaces-to-implementations)
    - [Liên kết bối cảnh](#contextual-binding)
    - [Thẻ](#tagging)
    - [Liên kết mở rộng](#extending-bindings)
- [Resolving](#resolving)
    - [Tạo phương thức](#the-make-method)
    - [Automatic Injection](#automatic-injection)
- [Container Event](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## Giới thiệu

Laravel service container là một công cụ mạnh mẽ để quản lý các class phụ thuộc và thực hiện tích hợp class phụ thuộc. Tích hợp class phụ thuộc là một cụm từ tuyệt vời có nghĩa cơ bản là: class phụ thuộc sẽ được "tích hợp" vào một class khác thông qua hàm tạo hoặc trong một số trường hợp, hàm "setter".

Hãy nhìn một ví dụ đơn giản:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

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

Trong ví dụ trên, `UserController` sẽ cần lấy user từ một data source. Vì vậy, chúng ta sẽ **tích hợp** một service có thể lấy user. Trong ngữ cảnh này, trong class `UserRepository` của chúng ta có thể sử dụng [Eloquent](/docs/{{version}}/eloquent) để lấy thông tin user từ database. Tuy nhiên, vì repository đã được tích hợp, chúng ta có thể dễ dàng chuyển nó sang một implementation khác. Chúng ta cũng có thể dễ dàng "làm giả", hoặc tạo một implementation giả của `UserRepository` khi test application của chúng ta.

Hiểu sâu về Laravel service container sẽ một điều cần thiết để tạo một application lớn, mạnh mẽ, cũng như phát triển phần lõi của Laravel.

<a name="binding"></a>
## Liên kết

<a name="binding-basics"></a>
### Liên kết cơ bản

Hầu như tất cả các liên kết của service container sẽ được đăng ký trong [service providers](/docs/{{version}}/providers), vì vậy hầu hết các ví dụ này sẽ được thể hiện bằng cách dùng container trong ngữ cảnh đó.

> {tip} Bạn sẽ không cần phải liên kết class vào container, nếu chúng không phụ thuộc vào bất kỳ interfaces nào. Bạn cũng không cần phải được cài đặt Container làm thế nào để tạo một đối tượng, vì nó có thể tự động resolve đối tượng bằng cách sử dụng class động.

#### Liên kết đơn giản

Trong một service provider, bạn luôn có quyền truy cập vào container thông qua thuộc tính `$this->app`. Chúng ta có thể đăng ký một liên kết bằng cách sử dụng phương thức `bind`, truyền tên class hoặc tên interface mà chúng ta muốn đăng ký cùng với một `Closure` trả về một instance của class:

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });


Lưu ý rằng chúng ta nhận bản thân container như là một đối số để resolver. Sau đó chúng ta có thể sử dụng container để resolve các phụ thuộc phụ của đối tượng chúng ta đang xây dựng.

#### Liên kết singleton

Phương thức `singleton` sẽ liên kết một class hoặc interface vào trong container chỉ resolve một lần duy nhất. Khi một liên kết singleton đã được resolve, thì lần tiếp theo khi gọi vào container thì đối tượng đó sẽ được trả về:

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### Liên kết instances

Bạn cũng có thể liên kết một object instance đã tồn tại vào container bằng cách sử dụng phương thức `instance`. Và instance đó sẽ luôn được trả về cho các lần gọi tiếp theo vào container:

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

#### Liên kết primitives

Thỉnh thoảng bạn có một class nhận vào một số tích hợp classes, nhưng cũng cần một giá trị tích hợp kiểu nguyên thuỷ như integer. Bạn có thể dễ dàng sử dụng liên kết theo ngữ cảnh đó để đưa vào bất kỳ giá trị nào mà class của bạn có thể cần:

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
### Liên kết Interfaces tới Implementations

Một tính năng rất mạnh mẽ của service container là khả năng liên kết một interface với một implementation nhất định. Ví dụ: giả sử chúng ta có interface `EventPusher` và implementation `RedisEventPusher`. Khi chúng ta đã code implementation `RedisEventPusher` của chúng ta vào interface đó, chúng ta có thể đăng ký nó với service container như sau:

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

Câu lệnh đó sẽ nói với container rằng nó nên tích hợp `RedisEventPusher` khi một class nào đó cần implementation `EventPusher`. Bây giờ chúng ta có thể gõ interface `EventPusher` vào trong hàm khởi tạo hoặc bất kỳ vị trí nào khác nơi mà các phụ thuộc được tích hợp bởi service container:

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Liên kết theo ngữ cảnh

Thỉnh thoảng bạn có thể có hai class sử dụng chung một interface, nhưng bạn muốn tích hợp các implementation khác nhau vào mỗi class. Ví dụ, hai controller có thể phụ thuộc vào các implementation khác nhau của `Illuminate\Contracts\Filesystem\Filesystem` [contract](/docs/{{version}}/contracts). Laravel cung cấp một interface đơn giản, và dễ dàng để xác định hành vi này:

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="tagging"></a>
### Thẻ

Đôi khi, bạn có thể cần phải giải quyết tất cả một "category" liên kết nhất định. Ví dụ, có lẽ bạn đang xây dựng một trình tổng hợp report nhận được một mảng gồm nhiều implementation interface `Report` khác nhau. Sau khi đăng ký các implementation `Report`, bạn có thể gán cho chúng một thẻ bằng phương thức` tag`:

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Khi các service đã được gắn thẻ, bạn có thể dễ dàng resolve tất cả chúng thông qua phương thức `tagged`:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="extending-bindings"></a>
### Liên kết mở rộng

Phương thức `extend` cho phép sửa đổi các service đã resolve. Ví dụ: khi một service đã được resolve, bạn có thể thêm code bổ sung để bổ sung hoặc định cấu hình service. Phương thức `extend` chấp nhận một Callback, sẽ trả về service đã sửa đổi, như là đối số duy nhất của nó:

    $this->app->extend(Service::class, function($service) {
        return new DecoratedService($service);
    });

<a name="resolving"></a>
## Resolving

<a name="the-make-method"></a>
#### Phương thức `make`

Bạn có thể sử dụng phương thức `make` để resolve một class instance ra khỏi container. Phương thức `make` chấp nhận tên của lớp hoặc interface mà bạn muốn resolve:

    $api = $this->app->make('HelpSpot\API');

Nếu bạn đang ở trong một vị trí mà code không có quyền truy cập vào biến `$app`, bạn có thể dùng global helper `resolve`:

    $api = resolve('HelpSpot\API');

Nếu một số phụ thuộc class của bạn không thể resolve được thông qua container, bạn có thể tích hợp chúng bằng cách chuyển chúng dưới dạng một mảng kết hợp vào phương thức `makeWith`:

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

<a name="automatic-injection"></a>
#### Tự động tích hợp

Ngoài ra, và rất quan trọng, bạn có thể viết sự phụ thuộc vào trong hàm khởi tạo của lớp được resolve bởi container, bao gồm [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [queue jobs](/docs/{{version}}/queues), [middleware](/docs/{{version}}/middleware), vv... Trong thực tế, đây là cách mà hầu hết các đối tượng của bạn sẽ được resolve bằng container.

Ví dụ: bạn có thể viết một repository được xác định bởi application của bạn trong hàm khởi tạo của controller. Repository sẽ tự động được resolve và đưa vào trong class:

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
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
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## Container Event

Service container kích hoạt một event mỗi khi nó resolve một đối tượng. Bạn có thể listen event này bằng phương thức `resolving`:

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // Called when container resolves objects of type "HelpSpot\API"...
    });

Như bạn có thể thấy, đối tượng đang được resolve sẽ được gọi đến một hàm callback, cho phép bạn đặt bất kỳ thuộc tính bổ sung nào trên đối tượng trước khi nó được trao cho người resolve nó.

<a name="psr-11"></a>
## PSR-11

Service container của Laravel implements một interface [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Do đó, bạn có thể viết một interface container PSR-11 để có được một instance của container Laravel:

    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get('Service');

        //
    });

> {note} Gọi phương thức `get` sẽ đưa ra một exception nếu mã định danh chưa được liên kết vào container.
