# Views

- [Giới thiệu](#introduction)
    - [Viết view trong React và Vue](#writing-views-in-react-or-vue)
- [Tạo và render view](#creating-and-rendering-views)
    - [Thư mục view lồng nhau](#nested-view-directories)
    - [Tạo view có sẵn đầu tiên](#creating-the-first-available-view)
    - [Xác định nếu một View tồn tại](#determining-if-a-view-exists)
- [Truyền dữ liệu đến view](#passing-data-to-views)
    - [Chia sẽ dữ liệu với tất cả các view](#sharing-data-with-all-views)
- [View Composers](#view-composers)
    - [View Creators](#view-creators)
- [Optimizing Views](#optimizing-views)

<a name="introduction"></a>
## Giới thiệu

Tất nhiên, việc trả về toàn bộ chuỗi code HTML trực tiếp từ route hoặc controller của bạn là không thực tế. Rất may, các view cung cấp một cách thuận tiện để đặt tất cả các code HTML của chúng ta vào các file riêng biệt.

View giúp tách logic controller và logic của ứng dụng ra khỏi logic hiển thị của bạn và được lưu trong thư mục `resources/views`. Khi sử dụng Laravel, các template thường được viết bằng [ngôn ngữ Blade template](/docs/{{version}}/blade). Một view đơn giản có thể trông giống như thế này:

```blade
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Vì view được lưu ở trong `resources/views/greeting.blade.php`, nên chúng ta có thể gọi nó bằng cách dùng global helper `view` như sau:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

> [!NOTE]
> Bạn đang tìm kiếm thêm thông tin về cách viết Blade template? Hãy xem [tài liệu đầy đủ về Blade](/docs/{{version}}/blade) để bắt đầu.

<a name="writing-views-in-react-or-vue"></a>
### Viết view trong React và Vue

Thay vì viết các template frontend của mình bằng PHP thông qua Blade, nhiều nhà phát triển đã bắt đầu thích viết các template của họ bằng React hoặc Vue. Laravel giúp việc này trở nên dễ dàng hơn nhờ [Inertia](https://inertiajs.com/), một thư viện sẽ giúp bạn dễ dàng liên kết frontend React hoặc Vue của bạn với backend Laravel mà không cần đến những thứ phức tạp thường thấy khi xây dựng SPA (Single Page Application).

[Bộ công cụ khởi động](/docs/{{version}}/starter-kits) Breeze và Jetstream của chúng tôi cung cấp cho bạn một điểm khởi đầu tuyệt vời cho ứng dụng Laravel tiếp theo của bạn mà được hỗ trợ bởi Inertia. Ngoài ra, [Laravel Bootcamp](https://bootcamp.laravel.com) cũng cung cấp bản demo đầy đủ về cách xây dựng ứng dụng Laravel được hỗ trợ bởi Inertia, bao gồm các ví dụ trong Vue và React.

<a name="creating-and-rendering-views"></a>
## Tạo và render view

Bạn có thể tạo view bằng cách đặt một file có phần mở rộng `.blade.php` vào trong thư mục `resources/views` trong ứng dụng của bạn hoặc bằng cách sử dụng lệnh Artisan `make:view`:

```shell
php artisan make:view greeting
```

Phần mở rộng `.blade.php` sẽ thông báo cho framework biết rằng file này là file chứa [Blade template](/docs/{{version}}/blade). Blade template sẽ chứa code HTML cũng như các lệnh Blade cho phép bạn dễ dàng hiển thị các giá trị, tạo câu lệnh "if", lặp dữ liệu, và nhiều hơn thế.

Khi bạn đã tạo xong view, bạn có thể trả view đó từ một trong các route hoặc controller của ứng dụng bằng cách sử dụng helper global `view`:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

View cũng có thể được trả về bằng cách sử dụng facade `View`:

    use Illuminate\Support\Facades\View;

    return View::make('greeting', ['name' => 'James']);

Như bạn có thể thấy, tham số đầu tiên được truyền tới helper `view` là tên của file view có trong thư mục `resources/views`. Tham số thứ hai là một mảng dữ liệu được truyền vào view. Trong trường hợp này, chúng ta đang truyền biến `name` cho view, và được hiển thị trong view bằng [Blade syntax](/docs/{{version}}/blade).

<a name="nested-view-directories"></a>
### Thư mục view lồng nhau

View cũng có thể được nằm trong một thư mục con của thư mục `resources/views`. Ký tự "chấm" có thể được sử dụng để gọi đến những thư mục view con đó. Ví dụ: nếu view của bạn được lưu tại `resources/views/admin/profile.blade.php`, bạn có thể trả nó từ một trong các route hoặc controller của ứng dụng của bạn như sau:

    return view('admin.profile', $data);

> [!WARNING]
> Tên thư mục view sẽ không được chứa ký tự `.`.

<a name="creating-the-first-available-view"></a>
### Tạo view có sẵn đầu tiên

Bằng cách sử dụng phương thức `first` của facade `View`, bạn có thể tạo view đầu tiên tồn tại trong một mảng các view nhất định. Điều này có thể hữu ích nếu ứng dụng hoặc package của bạn cho phép tùy chỉnh hoặc ghi đè view:

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="determining-if-a-view-exists"></a>
### Xác định nếu một View tồn tại

Nếu bạn cần kiểm tra một view có tồn tại hay không, bạn có thể sử dụng facade `View`. Phương thức `exists` sẽ trả về `true` nếu view đó tồn tại:

    use Illuminate\Support\Facades\View;

    if (View::exists('admin.profile')) {
        // ...
    }

<a name="passing-data-to-views"></a>
## Truyền dữ liệu đến view

Như bạn có thể thấy trong các ví dụ trước, bạn có thể truyền một mảng dữ liệu cho view để cung cấp dữ liệu đó cho view:

    return view('greetings', ['name' => 'Victoria']);

Khi truyền thông tin theo cách này, dữ liệu phải là một mảng với các cặp key / value. Sau khi cung cấp dữ liệu cho một view, bạn có thể truy cập vào các giá trị trong view của bạn bằng cách sử dụng các key của dữ liệu, chẳng hạn như `<?php echo $key; ?>`.

Để thay thế cho việc truyền một mảng dữ liệu cho hàm helper `view`, bạn có thể sử dụng phương thức `with` để thêm từng phần dữ liệu vào view. Phương thức `with` sẽ trả về một instance của đối tượng view để bạn có thể tiếp tục kết hợp thêm các phương thức khác trước khi trả về view:

    return view('greeting')
                ->with('name', 'Victoria')
                ->with('occupation', 'Astronaut');

<a name="sharing-data-with-all-views"></a>
### Chia sẽ dữ liệu với tất cả các view

Đôi khi, bạn có thể cần chia sẻ dữ liệu với tất cả các view có trong application của bạn. Bạn có thể làm như vậy bằng cách sử dụng phương thức `share` trong facade `View`. Thông thường, bạn nên thực hiện gọi phương thức `share` trong phương thức `boot` của service provider. Bạn có thể thêm chúng vào class `App\Providers\AppServiceProvider` hoặc tạo một service provider riêng để chứa chúng:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            View::share('key', 'value');
        }
    }

<a name="view-composers"></a>
## View Composers

Các View composer là các callback hoặc là các phương thức class được gọi khi một view được render. Nếu bạn có dữ liệu mà bạn muốn liên kết nó với một view mỗi khi view đó được render, thì một view composer có thể giúp bạn sắp xếp logic đó. View composer có thể tỏ ra đặc biệt hữu ích nếu cùng một view được trả về bởi nhiều route hoặc controller trong ứng dụng của bạn và luôn cần một lượng dữ liệu cụ thể.

Thông thường, view composer sẽ được đăng ký vào trong một trong các [service providers](/docs/{{version}}/providers) của ứng dụng của bạn. Trong ví dụ này, chúng tôi sẽ giả định rằng chúng tôi đã tạo một `App\Providers\ViewServiceProvider` mới để chứa logic này.

Chúng tôi sẽ sử dụng phương thức `composer` của facade `View` để đăng ký view composer. Laravel không chứa một thư mục mặc định cho các class dựa trên view composer, nên bạn có thể tổ chức chúng theo cách bạn muốn. Ví dụ: bạn có thể tạo thư mục `app/Http/View/Composers` để chứa tất cả các view composer của ứng dụng của bạn:

    <?php

    namespace App\Providers;

    use App\View\Composers\ProfileComposer;
    use Illuminate\Support\Facades;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\View\View;

    class ViewServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            // Using class based composers...
            Facades\View::composer('profile', ProfileComposer::class);

            // Using closure based composers...
            Facades\View::composer('welcome', function (View $view) {
                // ...
            });

            Facades\View::composer('dashboard', function (View $view) {
                // ...
            });
        }
    }

> [!WARNING]
> Hãy nhớ rằng, nếu bạn tạo một service provider mới để chứa các đăng ký view composer, bạn sẽ cần thêm service provider đó vào mảng `providers` trong file cấu hình `config/app.php`.

Sau khi chúng ta đã đăng ký xong composer, phương thức `compose` của class `App\View\Composers\ProfileComposer` sẽ được thực thi mỗi khi view `profile` được render. Hãy xem một ví dụ về class composer:

    <?php

    namespace App\View\Composers;

    use App\Repositories\UserRepository;
    use Illuminate\View\View;

    class ProfileComposer
    {
        /**
         * Create a new profile composer.
         */
        public function __construct(
            protected UserRepository $users,
        ) {}

        /**
         * Bind data to the view.
         */
        public function compose(View $view): void
        {
            $view->with('count', $this->users->count());
        }
    }

Như bạn có thể thấy, tất cả các view composer được resolve thông qua [service container](/docs/{{version}}/container), do đó bạn có thể khai báo bất kỳ phụ thuộc nào mà bạn cần vào trong hàm khởi tạo của composer.

<a name="attaching-a-composer-to-multiple-views"></a>
#### Gắn một Composer vào nhiều Views

Bạn có thể gắn một view composer cho nhiều view cùng một lúc bằng cách truyền một mảng các view làm tham số đầu tiên của phương thức `composer`:

    use App\Views\Composers\MultiComposer;
    use Illuminate\Support\Facades\View;

    View::composer(
        ['profile', 'dashboard'],
        MultiComposer::class
    );

Phương thức `composer` cũng chấp nhận một ký tự `*` làm ký tự đại diện, cho phép bạn gắn một composer cho tất cả các view:

    use Illuminate\Support\Facades;
    use Illuminate\View\View;

    Facades\View::composer('*', function (View $view) {
        // ...
    });

<a name="view-creators"></a>
#### View Creators

View "creators" giống với view composer; tuy nhiên, chúng được thực thi ngay lập tức sau khi view được khởi tạo thay vì đợi cho đến khi view sắp được hiển thị. Để đăng ký một view creator, hãy sử dụng phương thức `creator`:

    use App\View\Creators\ProfileCreator;
    use Illuminate\Support\Facades\View;

    View::creator('profile', ProfileCreator::class);

<a name="optimizing-views"></a>
## Optimizing Views

Mặc định, các view template Blade sẽ được biên dịch theo từng request. Khi một request được thực hiện làm hiển thị một view, thì Laravel sẽ xác định xem có tồn tại một phiên bản đã biên dịch của view đó hay không. Nếu file có tồn tại, Laravel sẽ xác định xem gần đây view chưa biên dịch có gì sửa đổi hơn với view đã được biên dịch hay không. Nếu view đã biên dịch không tồn tại hoặc view chưa được biên dịch đã có sửa đổi mới, thì Laravel sẽ biên dịch lại view.

Việc biên dịch các view trong quá trình request có thể sẽ ảnh hưởng nhỏ đến hiệu suất, vì vậy Laravel cung cấp lệnh Artisan `view:cache` để biên dịch trước tất cả các view mà được ứng dụng của bạn sử dụng. Để tăng hiệu suất, bạn có thể muốn chạy lệnh này như là một phần trong quá trình deploy của bạn:

```shell
php artisan view:cache
```

Bạn có thể sử dụng lệnh `view:clear` để xóa cache của view:

```shell
php artisan view:clear
```
