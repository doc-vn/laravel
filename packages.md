# Package Development

- [Giới thiệu](#introduction)
    - [Một chú ý về Facade](#a-note-on-facades)
- [Package Discovery](#package-discovery)
- [Service Provider](#service-providers)
- [Resources](#resources)
    - [Cấu hình](#configuration)
    - [Migration](#migrations)
    - [Factories](#factories)
    - [Route](#routes)
    - [Translation](#translations)
    - [View](#views)
    - [View Components](#view-components)
- [Lệnh](#commands)
- [Public Assets](#public-assets)
- [Publishing File Groups](#publishing-file-groups)

<a name="introduction"></a>
## Giới thiệu

Package là cách chính để thêm chức năng khác nhau cho Laravel. Các package có thể là bất cứ thứ gì từ việc làm việc với thời gian như [Carbon](https://github.com/briannesbitt/Carbon) hoặc toàn bộ framework BDD testing như [Behat](https://github.com/Behat/Behat).

Có nhiều loại package khác nhau. Một số package là độc lập, có nghĩa là chúng hoạt động với bất kỳ framework PHP nào. Carbon và Behat là các ví dụ về các package độc lập. Bất kỳ package nào trong số này có thể được sử dụng với Laravel bằng cách khai báo chúng vào trong file `composer.json` của bạn.

Mặt khác, có các package khác sẽ được dành riêng để sử dụng với Laravel. Các package này có thể có các route, controller, view và được cấu hình dành riêng cho mục đích sử dụng application Laravel. Các hướng dẫn ở bên dưới sẽ chủ yếu nói về các package dành riêng cho Laravel.

<a name="a-note-on-facades"></a>
### Một chú ý về Facade

Khi viết một application Laravel, thường không có vấn đề gì nếu bạn sử dụng contract hoặc facade vì cả hai đều cung cấp mức độ testability cơ bản là như nhau. Tuy nhiên, khi viết các package, package của bạn thường sẽ không có quyền truy cập vào tất cả các helper testing của Laravel. Nếu bạn muốn có thể viết các bài test cho package của bạn như thể chúng tồn tại trong một application Laravel bình thường, bạn có thể sử dụng package [Orchestral Testbench](https://github.com/orchestral/testbench).

<a name="package-discovery"></a>
## Package Discovery

Trong file cấu hình `config/app.php` của application Laravel, tùy chọn `providers` sẽ định nghĩa một danh sách các service provider sẽ được load bởi Laravel. Khi ai đó cài đặt package của bạn, bạn sẽ luôn muốn service provider của bạn được đưa vào trong danh sách này. Thay vì yêu cầu người dùng tự thêm service provider của bạn vào danh sách này, bạn có thể định nghĩa provider trong phần `extra` trong file `composer.json` trong package của bạn. Ngoài các service provider, bạn cũng có thể liệt kê bất kỳ [facades](/docs/{{version}}/facades) nào mà bạn muốn được đăng ký:

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

Khi package của bạn đã được cấu hình để tự động thêm, Laravel sẽ tự động đăng ký các service provider và các facade của package khi được cài đặt, vì thế nó sẽ tạo ra một trải nghiệm cài đặt tốt hơn cho người dùng package của bạn.

### Opting Out Of Package Discovery

Nếu bạn là người dùng package và muốn tắt chức năng tự động thêm cho một package, bạn có thể liệt kê tên package vào trong phần `extra` của file `composer.json` của application của bạn:

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },

Bạn có thể vô hiệu hóa chức năng tự động thêm cho tất cả các package bằng cách sử dụng ký tự `*` vào trong lệnh `dont-discover` của application của bạn:

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },

<a name="service-providers"></a>
## Service Provider

[Service providers](/docs/{{version}}/providers) là điểm kết nối giữa package của bạn với Laravel. Service provider sẽ chịu trách nhiệm liên kết mọi thứ vào trong [service container](/docs/{{version}}/container) của Laravel và thông báo cho Laravel biết nơi load các package resources như view, config và file localization.

Một service provider sẽ được extend từ class `Illuminate\Support\ServiceProvider` và chứa hai phương thức: `register` và `boot`. Class `ServiceProvider` sẽ nằm trong package `illuminate/support` của Composer, là nơi mà bạn sẽ thêm các phụ thuộc package của bạn. Để tìm hiểu thêm về cấu trúc và mục đích của các service provider, hãy xem [tài liệu về nó](/docs/{{version}}/providers).

<a name="resources"></a>
## Resources

<a name="configuration"></a>
### Cấu hình

Thông thường, bạn sẽ cần export file cấu hình của package vào thư mục `config` của application. Điều này cho phép người dùng package của bạn dễ dàng ghi đè các tùy chọn cấu hình mặc định mà bạn đã thiết lập. Để các file cấu hình của bạn có thể export, hãy gọi phương thức `publishes` từ trong phương thức `boot` của service provider của bạn:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }

Bây giờ, khi người dùng chạy lệnh `vendor:publish` của Laravel, thì file cấu hình của bạn sẽ được sao chép đến vị trí export đã được chỉ định. Khi file cấu hình của bạn đã được export, thì các giá trị của nó cũng có thể được truy cập như bất kỳ file cấu hình bình thường nào khác:

    $value = config('courier.option');

> {note} Bạn không nên định nghĩa Closures trong file cấu hình của bạn. Vì nó sẽ không thể chuyển đổi chính xác khi người dùng chạy lệnh Artisan `config:cache`.

#### Default Package Configuration

Bạn cũng có thể merge file cấu hình package của bạn với một bản copy được export từ application. Điều này sẽ cho phép người dùng của bạn chỉ định nghĩa các tùy chọn mà họ thực sự muốn ghi đè. Để merge các file cấu hình, sử dụng phương thức `mergeConfigFrom` trong phương thức `register` của service provider của bạn:

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }

> {note} Phương thức này chỉ merge ở mức độ đầu tiên của mảng. Nếu người dùng của bạn định nghĩa một mảng cấu hình lồng nhau, thì các tùy chọn bị thiếu sẽ không được merge.

<a name="routes"></a>
### Route

Nếu package của bạn chứa các route, thì bạn có thể load chúng bằng phương thức `loadRoutesFrom`. Phương thức này sẽ tự động kiểm tra xem các route hiện tại của application có đang được lưu trong bộ nhớ cache hay không và sẽ không tải lại file route của bạn nếu file route đó đã được lưu trong bộ nhớ cache:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadRoutesFrom(__DIR__.'/routes.php');
    }

<a name="migrations"></a>
### Migration

Nếu package của bạn chứa [database migrations](/docs/{{version}}/migrations), bạn có thể sử dụng phương thức `loadMigationsFrom` để thông báo cho Laravel biết cách load chúng. Phương thức `loadMigationsFrom` chấp nhận một đường dẫn đến các file migration của package của bạn như là tham số duy nhất của nó:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }

Khi file migration package của bạn đã được đăng ký, chúng sẽ được tự động chạy khi lệnh `php artisan migrate` được chạy. Bạn không cần export chúng vào thư mục `database/migrations` chính của application.

<a name="factories"></a>
### Factories

Nếu package của bạn chứa [database factory](/docs/{{version}}/database-testing#writing-factories), bạn có thể sử dụng phương thức `loadFactoriesFrom` để thông báo cho Laravel biết cách load chúng. Phương thức `loadFactoriesFrom` chấp nhận đường dẫn đến factory package của bạn làm tham số duy nhất của nó:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadFactoriesFrom(__DIR__.'/path/to/factories');
    }

Khi các factory của package của bạn đã được đăng ký xong, bạn có thể sử dụng chúng trong ứng dụng của bạn:

    factory(Package\Namespace\Model::class)->create();

<a name="translations"></a>
### Translation

Nếu package của bạn chứa các [translation files](/docs/{{version}}/localization), bạn có thể sử dụng phương thức `loadTranslationsFrom` để thông báo cho Laravel biết cách load chúng. Ví dụ: nếu package của bạn có tên là `courier`, thì bạn nên thêm code sau vào phương thức `boot` của service provider:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

Các bản translation của package của bạn sẽ được tham chiếu bằng cách sử dụng quy ước cú pháp như sau `package::file.line`. Vì vậy, bạn có thể load dòng `welcome` của package `courier` từ file `messages` như sau:

    echo trans('courier::messages.welcome');

#### Publishing Translations

Nếu bạn muốn export các bản translation của package của bạn sang thư mục `resources/lang/vendor` của application, bạn có thể sử dụng phương thức `publishes` của service provider. Phương thức `publishes` chấp nhận một mảng các đường dẫn đến file translation của package và vị trí export mà bạn mong muốn. Ví dụ, để export các file translation cho package `courier`, bạn có thể làm như sau:

    /**
     * Bootstrap any application services.
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

Bây giờ, khi người dùng package của bạn chạy lệnh Artisan `vendor:publish` của Laravel, bản translation của package của bạn sẽ được export đến vị trí export đã được khai báo.

<a name="views"></a>
### View

Để đăng ký [views](/docs/{{version}}/views) của package với Laravel, bạn cần cho Laravel biết vị trí của các view. Bạn có thể làm điều này bằng cách sử dụng phương thức `loadViewsFrom` của service provider. Phương thức `loadViewsFrom` chấp nhận hai tham số: một là đường dẫn đến các view template và hai là tên của package của bạn. Ví dụ: nếu tên của package của bạn là `courier`, thì bạn nên thêm dòng sau vào phương thức `boot` của service provider:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

Các view package được tham chiếu bằng cách sử dụng quy ước cú pháp như sau `package::view`. Vì vậy, khi đường dẫn view của bạn đã được đăng ký vào trong một service provider, bạn có thể load view `admin` từ package `courier` như sau:

    Route::get('admin', function () {
        return view('courier::admin');
    });

#### Overriding Package Views

Khi bạn sử dụng phương thức `loadViewsFrom`, Laravel sẽ đăng ký hai vị trí cho các view của bạn: một là thư mục `resources/views/vendor` của application và hai là thư mục mà bạn chỉ định. Vì vậy, nếu sử dụng ví dụ `courier` ở trên, thì trước tiên, Laravel sẽ kiểm tra trong thư mục `resources/views/vendor/courier` có bản tùy chỉnh nào hay không. Sau đó, nếu chưa có bản tùy chỉnh nào, Laravel sẽ tìm kiếm tiếp đến thư mục view package mà bạn đã chỉ định trong lệnh `loadViewsFrom`. Điều này sẽ giúp người dùng package của bạn dễ dàng tùy chỉnh và ghi đè lên các view package của bạn.

#### Publishing Views

Nếu bạn muốn export các view vào thư mục `resources/views/vendor` của application, bạn có thể sử dụng phương thức` publishes` của service provider. Phương thức `publishes` chấp nhận một mảng các đường dẫn view package của bạn và vị trí export mà bạn mong muốn:

    /**
     * Bootstrap any application services.
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

Bây giờ, nếu người dùng package của bạn chạy lệnh Artisan `vendor:publish` của Laravel, thì các view package của bạn sẽ được export đến vị trí export mà bạn đã khai báo.

<a name="view-components"></a>
### View Components

Nếu package của bạn chứa các [view component](/docs/{{version}}/blade#components), bạn có thể sử dụng phương thức `loadViewComponentsAs` để thông báo cho Laravel về cách load chúng. Phương thức `loadViewComponentsAs` chấp nhận hai tham số: một là tag tiền tố cho các view component của bạn và hai là một mảng gồm tên class của view component của bạn. Ví dụ: nếu tiền tố package của bạn là `courier` và bạn có hai view component là `Alert` và `Button`, bạn sẽ cần thêm code sau vào phương thức `boot` của service provider:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewComponentsAs('courier', [
            Alert::class,
            Button::class,
        ]);
    }

Sau khi các view component của bạn được đăng ký xong trong một service provider, bạn có thể tham chiếu đến chúng trong view của bạn như sau:

    <x-courier-alert />

    <x-courier-button />

#### Anonymous Components

Nếu package của bạn chứa các component ẩn, thì chúng phải được lưu trong thư mục `components` của thư mục "views" trong package của bạn (như được chỉ định bởi `loadViewsFrom`). Sau đó, bạn có thể hiển thị chúng bằng cách thêm tiền tố tên component với namespace view của package:

    <x-courier::alert />

<a name="commands"></a>
## Lệnh

Để đăng ký các lệnh Artisan trong package của bạn với Laravel, bạn có thể sử dụng phương thức `Command`. Phương thức này chấp nhận một mảng tên class của các lệnh. Khi các lệnh đã được đăng ký, bạn có thể chạy chúng bằng cách sử dụng [Artisan CLI](/docs/{{version}}/artisan):

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

Package của bạn có thể có các asset như JavaScript, CSS và hình ảnh. Để export các asset này vào thư mục `public` của application, hãy sử dụng phương thức `publishes` của service provider. Trong ví dụ này, chúng ta cũng sẽ thêm một tag group `public`, tag này có thể được sử dụng để export các group liên quan:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

Bây giờ, khi người dùng package của bạn chạy lệnh `vendor:publish`, asset sẽ được copy vào vị trí export mà bạn đã khai báo. Nhưng thông thường, bạn sẽ cần ghi đè lên các asset mỗi khi package được cập nhật, nên bạn có thể sử dụng flag `--force`:

    php artisan vendor:publish --tag=public --force

<a name="publishing-file-groups"></a>
## Publishing File Groups

Bạn có thể muốn export riêng rẽ các group asset và các resources của package. Chẳng hạn, bạn có thể muốn cho phép người dùng của bạn export các file cấu hình của package mà không phải export asset của package. Bạn có thể làm điều này bằng cách "gắn tag" cho chúng khi bạn gọi phương thức `publishes` từ service provider của package. Ví dụ: hãy sử dụng các tag để định nghĩa hai group export trong phương thức `boot` của service provider package:

    /**
     * Bootstrap any application services.
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

Bây giờ người dùng của bạn có thể export các group này một cách riêng rẽ bằng cách tham chiếu tag của chúng khi chạy lệnh `vendor:publish`:

    php artisan vendor:publish --tag=config
