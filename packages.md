# Package Development

- [Giới thiệu](#introduction)
    - [Một chú ý về Facade](#a-note-on-facades)
- [Package Discovery](#package-discovery)
- [Service Provider](#service-providers)
- [Resources](#resources)
    - [Cấu hình](#configuration)
    - [Migration](#migrations)
    - [Route](#routes)
    - [Translation](#translations)
    - [View](#views)
- [Lệnh](#commands)
- [Public Assets](#public-assets)
- [Publishing File Groups](#publishing-file-groups)

<a name="introduction"></a>
## Giới thiệu

Package là cách chính để thêm chức năng cho Laravel. Các package có thể là bất cứ thứ gì từ một cách để làm việc với ngày tháng như [Carbon](https://github.com/briannesbitt/Carbon) hoặc toàn bộ framework BDD testing như [Behat](https://github.com/Behat/Behat).

Tất nhiên, có nhiều loại package khác nhau. Một số package là độc lập, có nghĩa là chúng hoạt động với bất kỳ framework PHP nào. Carbon và Behat là các ví dụ về các package độc lập. Bất kỳ package nào trong số này có thể được sử dụng với Laravel bằng cách nhập chúng vào trong file `composer.json` của bạn.

Mặt khác, các package khác có thể được dành riêng để sử dụng với Laravel. Các package này có thể có các route, controller, view và cấu hình dành riêng cho mục đích nâng cao application Laravel. Các hướng dẫn ở bên dưới sẽ chủ yếu nói về sự phát triển của các package dành riêng cho Laravel.

<a name="a-note-on-facades"></a>
### Một chú ý về Facade

Khi viết một application Laravel, thường không có vấn đề gì nếu bạn sử dụng contract hoặc facade vì cả hai đều cung cấp mức độ testability cơ bản bằng nhau. Tuy nhiên, khi viết các package, package của bạn thường sẽ không có quyền truy cập vào tất cả các helper testing của Laravel. Nếu bạn muốn có thể viết các test cho package của bạn như thể chúng tồn tại trong một application Laravel bình thường, bạn có thể sử dụng package [Orchestral Testbench](https://github.com/orchestral/testbench).

<a name="package-discovery"></a>
## Package Discovery

Trong file cấu hình `config/app.php` của application Laravel, tùy chọn `providers`  sẽ định nghĩa danh sách các service provider sẽ được load bởi Laravel. Khi ai đó cài đặt package của bạn, bạn thường muốn service provider của mình được đưa vào danh sách này. Thay vì yêu cầu người dùng thêm thủ công service provider của bạn vào danh sách, bạn có thể định nghĩa provider trong phần `extra` trong file` composer.json` của package của bạn. Ngoài các service provider, bạn cũng có thể liệt kê bất kỳ [facades](/docs/{{version}}/facades) nào bạn muốn được đăng ký:

    "extra": {
        "laravel": {
            "providers": [
                "Barryvdh\\Debugbar\\ServiceProvider"
            ],
            "aliases": {
                "Debugbar": "Barryvdh\\Debugbar\\Facade"
            }
        }
    },

Khi package của bạn đã được cấu hình để discovery, Laravel sẽ tự động đăng ký service provider và facade của package khi được cài đặt, vì thế nó sẽ tạo ra một trải nghiệm cài đặt tốt cho người dùng package của bạn.

### Opting Out Of Package Discovery

Nếu bạn là người dùng của một package và muốn tắt discovery package cho một package, bạn có thể liệt kê tên package trong phần `extra` của file `composer.json` của application của bạn:

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },

Bạn có thể vô hiệu hóa discovery package cho tất cả các package bằng cách sử dụng ký tự `*` bên trong lệnh `dont-discover` của application của bạn:

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },

<a name="service-providers"></a>
## Service Provider

[Service providers](/docs/{{version}}/providers) là các điểm kết nối giữa package của bạn và Laravel. service provider chịu trách nhiệm liên kết mọi thứ vào [service container](/docs/{{version}}/container) của Laravel và thông báo cho Laravel nơi load resources của package như view, cấu hình và file localization.

Một service provider sẽ mở rộng class `Illuminate\Support\ServiceProvider` và chứa hai phương thức: `register` và `boot`. Class cơ sở `ServiceProvider` nằm trong package `illuminate/support` Composer, là nơi mà bạn nên thêm vào các phụ thuộc cho package của bạn. Để tìm hiểu thêm về cấu trúc và mục đích của các service provider, hãy xem [tài liệu về nó](/docs/{{version}}/providers).

<a name="resources"></a>
## Resources

<a name="configuration"></a>
### Cấu hình

Thông thường, bạn sẽ cần export file cấu hình của package lên thư mục `config` của application. Điều này sẽ cho phép người dùng package của bạn dễ dàng ghi đè các tùy chọn cấu hình mặc định của bạn. Để cho phép các file cấu hình của bạn được export, hãy gọi phương thức `publishes` từ trong phương thức `boot` của service provider của bạn:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }

Bây giờ, khi người dùng package của bạn chạy lệnh `vendor:publish` của Laravel, file của bạn sẽ được sao chép vào vị trí export được chỉ định. Tất nhiên, khi cấu hình của bạn đã được export, các giá trị của nó có thể được truy cập như bất kỳ file cấu hình nào khác:

    $value = config('courier.option');

> {note} Bạn không nên định nghĩa Closures trong file cấu hình của bạn. Vì chúng không thể được serialize một cách chính xác khi người dùng chạy lệnh Artisan `config:cache`.

#### Default Package Configuration

Bạn cũng có thể merge file cấu hình package của bạn với bản copy được export của application. Điều này sẽ cho phép người dùng của bạn chỉ định nghĩa các tùy chọn họ thực sự muốn ghi đè trong bản copy được export của cấu hình. Để merge các cấu hình, sử dụng phương thức `mergeConfigFrom` trong phương thức `register` của service provider của bạn:
    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }

> {note} Phương thức này chỉ merge mức đầu tiên của mảng cấu hình. Nếu người dùng của bạn xác định một phần mảng cấu hình nhiều chiều, các tùy chọn bị thiếu sẽ không được merge.

<a name="routes"></a>
### Route

Nếu package của bạn chứa các route, bạn có thể load chúng bằng phương thức `loadRoutesFrom`. Phương thức này sẽ tự động xác định xem các route của application có được lưu trong bộ nhớ cache không và sẽ không tải file route của bạn nếu route đã được lưu trong bộ nhớ cache:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadRoutesFrom(__DIR__.'/routes.php');
    }

<a name="migrations"></a>
### Migration

Nếu package của bạn chứa [database migrations](/docs/{{version}}/migrations), bạn có thể sử dụng phương thức `loadMigationsFrom` để thông báo cho Laravel cách load chúng. Phương thức `loadMigationsFrom` chấp nhận đường dẫn đến các migration của package của bạn như là tham số duy nhất của nó:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }

Khi migration package của bạn đã được đăng ký, chúng sẽ tự động được chạy khi lệnh `php artisan migrate` được chạy. Bạn không cần export chúng vào thư mục `database/migrations` chính của application.

<a name="translations"></a>
### Translation

Nếu package của bạn chứa [translation files](/docs/{{version}}/localization), bạn có thể sử dụng phương thức `loadTranslationsFrom` để thông báo cho Laravel cách load chúng. Ví dụ: nếu package của bạn có tên là `courier`, bạn nên thêm code sau vào phương thức `boot` của service provider:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

Các bản dịch của package sẽ được tham chiếu bằng cách sử dụng quy ước cú pháp `package::file.line`. Vì thế, bạn có thể load dòng `welcome` của package `courier` từ file `messages` như sau:

    echo trans('courier::messages.welcome');

#### Publishing Translations

Nếu bạn muốn export các bản dịch của package của bạn lên thư mục `resources/lang/vendor` của application, bạn có thể sử dụng phương thức `publishes` của service provider. Phương thức `publishes` chấp nhận một loạt các đường dẫn package và vị trí export mong muốn của bạn. Ví dụ, để export các file dịch cho package `courier`, bạn có thể làm như sau:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }

Bây giờ, khi người dùng package của bạn chạy lệnh Artisan `vendor:publish` của Laravel, bản dịch của package của bạn sẽ được export đến vị trí export được khai báo sẵn.

<a name="views"></a>
### View

Để đăng ký [views](/docs/{{version}}/views) của package với Laravel, bạn cần cho Laravel biết vị trí của các view. Bạn có thể làm điều này bằng cách sử dụng phương thức `loadViewsFrom` của service provider. Phương thức `loadViewsFrom` chấp nhận hai tham số: đường dẫn đến các view template và tên package của bạn. Ví dụ: nếu tên package của bạn là `courier`, bạn sẽ thêm dòng sau vào phương thức `boot` của service provider:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

Các package view được tham chiếu bằng cách sử dụng quy ước cú pháp `package::view`. Vì thế, khi đường dẫn view của bạn được đăng ký trong một service provider, bạn có thể load view `admin` từ package `courier` như sau:

    Route::get('admin', function () {
        return view('courier::admin');
    });

#### Overriding Package Views

Khi bạn sử dụng phương thức `loadViewsFrom`, Laravel thực sự đăng ký hai vị trí cho các view của bạn: thư mục `resources/views/vendor` của application và thư mục bạn chỉ định. Vì vậy, bằng cách sử dụng ví dụ `courier`, trước tiên, Laravel sẽ kiểm tra xem phiên bản tùy chỉnh của view có được nhà phát triển cung cấp trong `resources/views/vendor/courier` hay không. Sau đó, nếu view chưa được tùy chỉnh, Laravel sẽ tìm kiếm thư mục view package mà bạn đã chỉ định trong lệnh gọi tới `loadViewsFrom`. Điều này giúp người dùng package dễ dàng tùy chỉnh / ghi đè lên view package của bạn.

#### Publishing Views

Nếu bạn muốn làm cho các view của bạn có sẵn để export vào thư mục `resources/views/vendor` của application, bạn có thể sử dụng phương thức` publishes` của service provider. Phương thức `publishes` chấp nhận một loạt các đường dẫn view package và vị trí export mong muốn của bạn:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }

Bây giờ, khi người dùng package của bạn chạy lệnh Artisan `vendor:publish` của Laravel, các view của package của bạn sẽ được export đến vị trí export được khai báo sẵn.

<a name="commands"></a>
## Lệnh

Để đăng ký các lệnh Artisan của package của bạn với Laravel, bạn có thể sử dụng phương thức `Command`. Phương thức này chấp nhận một mảng tên class của lệnh. Khi các lệnh đã được đăng ký, bạn có thể chạy chúng bằng cách sử dụng [Artisan CLI](/docs/{{version}}/artisan):

    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                FooCommand::class,
                BarCommand::class,
            ]);
        }
    }

<a name="public-assets"></a>
## Public Assets

Package của bạn có thể có các asset như JavaScript, CSS và hình ảnh. Để export các asset này lên thư mục `public` của application, hãy sử dụng phương thức `publishes` của service provider. Trong ví dụ này, chúng ta cũng sẽ thêm group tag asset `public`, có thể được sử dụng để export các group asset liên quan:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

Bây giờ, khi người dùng package của bạn chạy lệnh `vendor:publish`, asset của bạn sẽ được copy vào vị trí export được khai báo sẵn. Vì thông thường bạn sẽ cần ghi đè lên các asset mỗi khi package được cập nhật, nên bạn có thể sử dụng flag `--force`:

    php artisan vendor:publish --tag=public --force

<a name="publishing-file-groups"></a>
## Publishing File Groups

Bạn có thể muốn export các group asset và resources của package riêng biệt. Chẳng hạn, bạn có thể muốn cho phép người dùng export các file cấu hình của package mà không bị buộc phải export asset của package. Bạn có thể làm điều này bằng cách "gắn thẻ" chúng khi gọi phương thức `publishes` từ service provider của package. Ví dụ: hãy sử dụng các thẻ để xác định hai group export trong phương thức `boot` của service provider package:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'config');

        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'migrations');
    }

Bây giờ người dùng của bạn có thể export các group này một cách riêng biệt bằng cách tham chiếu thẻ của chúng khi chạy lệnh `vendor:publish`:

    php artisan vendor:publish --tag=config
