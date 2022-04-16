# Service Providers

- [Giới thiệu](#introduction)
- [Viết Service Provider](#writing-service-providers)
    - [Phương thức Register](#the-register-method)
    - [Phương thức Boot](#the-boot-method)
- [Đăng ký các Provider](#registering-providers)
- [Các Provider hoãn](#deferred-providers)

<a name="introduction"></a>
## Giới thiệu

Các service provider là trung tâm của tất cả quá trình khởi động của application Laravel. Application của bạn, cũng như tất cả các service cốt lõi của Laravel đều được khởi động thông qua các service provider.

Nhưng, "bootstrapped" nghĩa là gì? Nói chung, ý của chúng tôi có nghĩa là **đăng ký** những thứ, bao gồm cả đăng ký liên kết service container, event listener, middleware và thậm chí là cả các route. Các Service provider là trung tâm để cấu hình application của bạn.

Nếu bạn mở file `config/app.php` đi cùng với Laravel, bạn sẽ thấy một mảng các `providers`. Đây là tất cả các class service provider sẽ được load cho application của bạn. Nhiều trong số này là các provider "hoãn", nghĩa là nó sẽ không được load trong mọi request, mà chỉ khi các service này thực sự cần thiết nó mới được load.

Trong phần tổng quan này, bạn sẽ học cách viết các service provider của riêng bạn và đăng ký chúng với application Laravel.

<a name="writing-service-providers"></a>
## Viết Service Provider

Tất cả các service provider đều được extend từ class `Illuminate\Support\ServiceProvider`. Hầu hết các service provider đều chứa một phương thức `register` và một phương thức `boot`. Trong phương thức `register`, bạn **chỉ nên liên kết vào [service container](/docs/{{version}}/container)**. Bạn đừng bao giờ đăng ký bất kỳ event listener, routes hoặc bất kỳ phần chức năng nào khác vào trong phương thức `register`.

Artisan CLI có thể tạo một provider mới thông qua lệnh `make:provider`:

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### Phương thức Register

Như đã đề cập trước, trong phương thức `register`, bạn chỉ nên liên kết vào [service container](/docs/{{version}}/container). Bạn đừng bao giờ đăng ký bất kỳ event listener, routes hoặc bất kỳ phần chức năng nào khác vào trong phương thức `register`. Vì, bạn có thể vô tình sử dụng một service được cung cấp bởi một service provider khác mà chưa được load.

Chúng ta hãy cùng xem một service provider cơ bản. Trong bất kỳ phương thức nào của service provider, bạn luôn có quyền truy cập vào thuộc tính `$app`, mà cung cấp quyền truy cập vào service container:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Riak\Connection;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

Service provider này chỉ định nghĩa một phương thức `register` và sử dụng phương thức đó để định nghĩa một implementation của `Riak\Connection` trong service container. Nếu bạn không hiểu cách thức hoạt động của service container, hãy xem [tài liệu về nó](/docs/{{version}}/container).

#### Thuộc tính `bindings` và `singletons`

Nếu service provider của bạn đăng ký nhiều liên kết, thì bạn có thể muốn sử dụng thuộc tính `bindings` và `singletons` để đăng ký thay vì đăng ký thủ công từng liên kết một vào container. Khi service provider được load bởi framework, nó sẽ tự động kiểm tra các thuộc tính này và đăng ký các liên kết mà bạn đã khai báo:

    <?php

    namespace App\Providers;

    use App\Contracts\DowntimeNotifier;
    use App\Contracts\ServerProvider;
    use App\Services\DigitalOceanServerProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\ServerToolsProvider;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * All of the container bindings that should be registered.
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * All of the container singletons that should be registered.
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier::class,
            ServerProvider::class => ServerToolsProvider::class,
        ];
    }

<a name="the-boot-method"></a>
### Phương thức Boot

Vậy, điều gì sẽ xảy ra nếu chúng ta cần đăng ký một [view composer](/docs/{{version}}/views#view-composers) trong service provider của chúng ta? Điều này nên được thực hiện trong phương thức `boot`. **Phương thức này được gọi sau khi tất cả các service provider khác đã được đăng ký**, nghĩa là bạn có quyền truy cập vào tất cả các service khác đã được đăng ký theo framework:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

#### Phương thức Boot tích hợp khai báo phụ thuộc

Bạn có thể viết khai báo phụ thuộc vào trong phương thức `boot` của service provider của bạn. [service container](/docs/{{version}}/container) sẽ tự động tích hợp bất kỳ phụ thuộc nào mà bạn cần:

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Đăng ký Providers

Tất cả các service provider được đăng ký trong file cấu hình `config/app.php`. File này chứa một mảng các `providers` nơi mà bạn có thể liệt kê tên class của các service provider của bạn. Mặc định, một nhóm các service provider cốt lõi của Laravel đã được liệt kê ở trong mảng này. Các provider này sẽ khởi động các thành phần cốt lõi của Laravel, chẳng hạn như mailer, queue, cache, và các thành phần khác.

Để đăng ký provider của bạn, hãy thêm nó vào mảng:

    'providers' => [
        // Other Service Providers

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## Các Provider hoãn

Nếu provider của bạn **chỉ** đăng ký các liên kết trong [service container](/docs/{{version}}/container), bạn có thể chọn trì hoãn việc đăng ký cho đến khi một trong số các đăng ký liên kết thật sự cần thiết. Việc trì hoãn load của một provider như vậy sẽ cải thiện hiệu suất của ứng dụng của bạn, vì nó không được load từ filesystem cho mỗi request.

Laravel sẽ biên dịch và lưu trữ một danh sách tất cả các service mà được cung cấp dưới các service provider trì hoãn, cùng với tên của class service provider đó. Sau đó, chỉ khi bạn resolve một trong những service này thì Laravel mới tải service provider đó lên.

Để trì hoãn việc load của một provider, hãy implement interface `\Illuminate\Contracts\Support\DeferrableProvider` và định nghĩa một phương thức `provides`. Phương thức `provides` sẽ trả về các liên kết service container được đăng ký bởi provider:

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Support\DeferrableProvider;
    use Illuminate\Support\ServiceProvider;
    use Riak\Connection;

    class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }
    }
