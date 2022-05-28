# Views

- [Tạo Views](#creating-views)
- [Truyền dữ liệu đến Views](#passing-data-to-views)
    - [Chia sẽ dữ liệu với tất cả các Views](#sharing-data-with-all-views)
- [View Composers](#view-composers)
- [Optimizing Views](#optimizing-views)

<a name="creating-views"></a>
## Tạo Views

> {tip} Bạn đang tìm kiếm thông tin về template Blade? Hãy xem [Blade documentation](/docs/{{version}}/blade) để bắt đầu.

Views chứa HTML được cung cấp bởi application của bạn sẽ giúp tách logic controller và application ra khỏi logic hiển thị của bạn. Views được lưu trữ trong thư mục `resources/views`. Nhìn đơn giản nó có thể trông giống như thế này:

    <!-- View stored in resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

Vì view được lưu ở trong `resources/views/greeting.blade.php`, nên chúng ta có thể gọi nó bằng cách dùng global helper `view` như sau:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Như bạn có thể thấy, tham số đầu tiên được truyền tới helper `view` là tên của file view có trong thư mục `resources/views`. Tham số thứ hai là một mảng dữ liệu được truyền vào view. Trong trường hợp này, chúng ta đang truyền biến `name` cho view, và được hiển thị trong view bằng [Blade syntax](/docs/{{version}}/blade).

View cũng có thể được nằm trong một thư mục con của thư mục `resources/views`. Ký tự "chấm" có thể được sử dụng để gọi đến những thư mục view con đó. Ví dụ: nếu view của bạn được lưu trữ tại `resources/views/admin/profile.blade.php`, bạn có thể gọi đến chúng như sau:

    return view('admin.profile', $data);

> {note} Tên thư mục view sẽ không được chứa ký tự `.`.

#### Xác định nếu một View tồn tại

Nếu bạn cần kiểm tra một view có tồn tại hay không, bạn có thể sử dụng facade `View`. Phương thức `exists` sẽ trả về `true` nếu view đó tồn tại:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

#### Tạo view có sẵn đầu tiên

Sử dụng phương thức `first`, bạn có thể trả về view đầu tiên tồn tại trong một mảng view nhất định. Điều này sẽ hữu ích nếu application hoặc package của bạn cho phép được tùy chỉnh hoặc ghi đè view:

    return view()->first(['custom.admin', 'admin'], $data);

Bạn cũng có thể gọi phương thức này thông qua [facade](/docs/{{version}}/facades) `View`:

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="passing-data-to-views"></a>
## Truyền dữ liệu đến Views

Như bạn có thể thấy trong các ví dụ trước, bạn có thể truyền một mảng dữ liệu cho view:

    return view('greetings', ['name' => 'Victoria']);

Khi truyền thông tin theo cách này, dữ liệu phải là một mảng với các cặp key / value. Trong view của bạn, bạn có thể truy cập vào các giá trị đó bằng khóa tương ứng của chúng, chẳng hạn như `<?php echo $key; ?>`. Để thay thế cho việc truyền một mảng dữ liệu cho hàm helper `view`, bạn có thể sử dụng phương thức `with` để thêm từng phần dữ liệu vào view:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Chia sẻ dữ liệu với tất cả View

Đôi khi, bạn có thể cần chia sẻ một phần dữ liệu với tất cả các view có trong application của bạn. Bạn có thể làm như vậy bằng cách sử dụng phương thức `share` trong facade. Thông thường, bạn nên thực hiện gọi phương thức `share` trong phương thức `boot` của service provider. Bạn có thể thêm chúng vào `AppServiceProvider` hoặc tạo một service provider riêng để chứa chúng:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }
    }

<a name="view-composers"></a>
## View Composers

Các View composer là các callback hoặc là các phương thức class được gọi khi một view được render. Nếu bạn có dữ liệu mà bạn muốn liên kết nó với một view mỗi khi view đó được render, thì một view composer có thể giúp bạn sắp xếp logic đó.

Trong ví dụ này, hãy đăng ký các view composer trong một [service provider](/docs/{{version}}/providers). Chúng ta sẽ sử dụng facade `View` để truy cập vào contract implementation của `Illuminate\Contracts\View\Factory`. Hãy nhớ rằng, Laravel không chứa một thư mục mặc định cho các view composer. Bạn có thể tổ chức chúng theo cách bạn muốn. Ví dụ: bạn có thể tạo thư mục `app/Http/View/Composers`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ViewServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\View\Composers\ProfileComposer'
            );

            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }
    }

> {note} Hãy nhớ rằng, nếu bạn tạo một service provider mới để chứa các đăng ký view composer, bạn sẽ cần thêm service provider đó vào mảng `providers` trong file cấu hình `config/app.php`.

Sau khi chúng ta đã đăng ký xong composer, phương thức `ProfileComposer@compose` sẽ được thực thi mỗi khi view `profile` được render. Vì vậy, hãy định nghĩa class composer:

    <?php

    namespace App\Http\View\Composers;

    use App\Repositories\UserRepository;
    use Illuminate\View\View;

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

Ngay trước khi view được render, phương thức `compose` của composer sẽ được gọi với một instance `Illuminate\View\View`. Bạn có thể sử dụng phương thức `with` để liên kết dữ liệu với view đó.

> {tip} Tất cả các view composer được resolve thông qua [service container](/docs/{{version}}/container), do đó bạn có thể khai báo bất kỳ phụ thuộc nào mà bạn cần vào trong hàm khởi tạo của composer.

#### Gắn một Composer vào nhiều Views

Bạn có thể gắn một view composer cho nhiều view cùng một lúc bằng cách truyền một mảng các view làm tham số đầu tiên của phương thức `composer`:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\View\Composers\MyViewComposer'
    );

Phương thức `composer` cũng chấp nhận một ký tự `*` làm ký tự đại diện, cho phép bạn gắn một composer cho tất cả các view:

    View::composer('*', function ($view) {
        //
    });

#### View Creators

View **creators** giống với view composer; tuy nhiên, chúng được thực thi ngay lập tức sau khi view được khởi tạo thay vì đợi cho đến khi view sắp được hiển thị. Để đăng ký một view creator, hãy sử dụng phương thức `creator`:

    View::creator('profile', 'App\Http\View\Creators\ProfileCreator');

<a name="optimizing-views"></a>
## Optimizing Views

Mặc định, các view sẽ được biên dịch theo từng request. Khi một request được thực hiện làm hiển thị một view, thì Laravel sẽ xác định xem có tồn tại một phiên bản đã biên dịch của view đó hay không. Nếu file có tồn tại, Laravel sẽ xác định xem gần đây view chưa biên dịch có gì sửa đổi hơn với view đã được biên dịch hay không. Nếu view đã biên dịch không tồn tại hoặc view chưa được biên dịch đã có sửa đổi mới, thì Laravel sẽ biên dịch lại view.

Việc biên dịch các view trong quá trình request sẽ ảnh hưởng tiêu cực đến hiệu suất, vì vậy Laravel cung cấp lệnh Artisan `view:cache` để biên dịch trước tất cả các view mà được ứng dụng của bạn sử dụng. Để tăng hiệu suất, bạn có thể muốn chạy lệnh này như là một phần trong quá trình deploy của bạn:

    php artisan view:cache

Bạn có thể sử dụng lệnh `view:clear` để xóa cache của view:

    php artisan view:clear
