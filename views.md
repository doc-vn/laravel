# Views

- [Tạo Views](#creating-views)
- [Pass dữ liệu đến Views](#passing-data-to-views)
    - [Chia sẽ dữ liệu với tất cả các Views](#sharing-data-with-all-views)
- [View Composers](#view-composers)

<a name="creating-views"></a>
## Tạo Views

> {tip} Bạn đang tìm kiếm thêm thông tin về cách viết Blade templates? Hãy kiểm tra [Blade documentation](/docs/{{version}}/blade) để bắt đầu.

Views chứa HTML được cung cấp bởi application của bạn và tách logic controller và application khỏi logic trình bày của bạn. Views được lưu trữ trong thư mục `resources/views`. Nhìn đơn giản nó có thể trông giống như thế này:

    <!-- View stored in resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

Vì view này được lưu ở `resources/views/greeting.blade.php`, chúng ta có thể return nó bằng cách dùng global helper `view` như sau:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Như bạn có thể thấy, tham số đầu tiên được pass tới helper `view` tương ứng với tên của file view trong thư mục `resources/views`. Tham số thứ hai là một mảng dữ liệu được cung cấp đến view. Trong trường hợp này, chúng tôi đang pass biến `name`, được hiển thị trong view bằng [Blade syntax](/docs/{{version}}/blade).

Tất nhiên, views cũng có thể được nằm trong các thư mục con của thư mục `resources/views`. Ký tự "Dot" có thể được sử dụng để tham chiếu views con. Ví dụ: nếu view của bạn được lưu trữ tại `resources/views/admin/profile.blade.php`, bạn có thể tham chiếu nó như sau:

    return view('admin.profile', $data);

#### Xác định nếu một View tồn tại

Nếu bạn cần xác định xem một view có tồn tại hay không, bạn có thể sử dụng facade `View`. Phương thức `exists` sẽ trả về `true` nếu view tồn tại:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

#### Tạo view có sẵn đầu tiên

Sử dụng phương thức `first`, bạn có thể tạo view đầu tiên tồn tại trong một mảng view nhất định. Điều này hữu ích nếu application hoặc package của bạn cho phép view được tùy chỉnh hoặc ghi đè:

    return view()->first(['custom.admin', 'admin'], $data);

Tất nhiên, bạn cũng có thể gọi phương thức này thông qua [facade](/docs/{{version}}/facades) `View`:

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="passing-data-to-views"></a>
## Pass dữ liệu đến Views

Như bạn đã thấy trong các ví dụ trước, bạn có thể pass một mảng dữ liệu cho view:

    return view('greetings', ['name' => 'Victoria']);

Khi pass thông tin theo cách này, dữ liệu phải là một mảng với các cặp key / value. Trong view của bạn, bạn có thể truy cập vào từng giá trị bằng khóa tương ứng của nó, chẳng hạn như `<?php echo $key; ?>`. Để thay thế cho việc truyền một mảng dữ liệu  cho hàm helper `view`, bạn có thể sử dụng phương thức `with` để thêm các từng phần dữ liệu vào view:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Chia sẻ dữ liệu với tất cả View

Đôi khi, bạn có thể cần chia sẻ một phần dữ liệu với tất cả các view được application của bạn. Bạn có thể làm như vậy bằng cách sử dụng phương thức `share` của facade. Thông thường, bạn nên thực hiện gọi `share` trong phương thức` boot` của service provider. Bạn có thể tự do thêm chúng vào `AppServiceProvider` hoặc tạo một service provider riêng để chứa chúng:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="view-composers"></a>
## View Composers

Các View composer là các callback hoặc các phương thức class được gọi khi một view được render. Nếu bạn có dữ liệu mà bạn muốn liên kết nó vào một view mỗi khi view đó được render, một view composer có thể giúp bạn sắp xếp logic đó ở một vị trí duy nhất.

Trong ví dụ này, hãy đăng ký các view composer trong một [service provider](/docs/{{version}}/providers). Chúng ta sẽ sử dụng facade `View` để truy cập vào contract implementation `Illuminate\Contracts\View\Factory`. Hãy nhớ rằng, Laravel không chứa một thư mục mặc định cho các view composer. Bạn có thể tự do tổ chức chúng theo cách bạn muốn. Ví dụ: bạn có thể tạo thư mục `app/Http/ViewComposers`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} Hãy nhớ rằng, nếu bạn tạo một service provider mới để chứa các đăng ký view composer của mình, bạn sẽ cần thêm service provider vào mảng `providers` trong file cấu hình `config/app.php`.

Bây giờ chúng ta đã đăng ký composer, phương thức `ProfileComposer@compose` sẽ được thực thi mỗi khi view `profile` được render. Vì vậy, hãy định nghĩa class composer:

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Ngay trước khi view được render, phương thức `compose` của composer được gọi với một instance `Illuminate\View\View`. Bạn có thể sử dụng phương thức `with` để liên kết dữ liệu với view.

> {tip} Tất cả view composer được resolve thông qua [service container](/docs/{{version}}/container), do đó bạn có thể khai báo theo dạng kiểu bất kỳ phụ thuộc nào bạn cần trong hàm khởi tạo của composer.

#### Gắn một Composer vào nhiều Views

Bạn có thể gắn một view composer cho nhiều view cùng một lúc bằng cách chuyển một mảng các view làm tham số đầu tiên cho phương thức `composer`:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

Phương thức `composer` cũng chấp nhận ký tự `*` làm ký tự đại diện, cho phép bạn gắn một composer cho tất cả các view:

    View::composer('*', function ($view) {
        //
    });

#### View Creators

View **creators** giống với view composer; tuy nhiên, chúng được thực thi ngay lập tức sau khi view được khởi tạo thay vì đợi cho đến khi view sắp hiển thị. Để đăng ký một view creator, hãy sử dụng phương thức `creator`:

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
