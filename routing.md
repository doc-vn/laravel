# Routing

- [Routing cở bản](#basic-routing)
    - [Chuyển hướng Route](#redirect-routes)
    - [View Routes](#view-routes)
- [Route Parameters](#route-parameters)
    - [Required Parameters](#required-parameters)
    - [Optional Parameters](#parameters-optional-parameters)
    - [Ràng buộc biểu thức chính quy](#parameters-regular-expression-constraints)
- [Tên Route](#named-routes)
- [Nhóm Route](#route-groups)
    - [Middleware](#route-group-middleware)
    - [Namespaces](#route-group-namespaces)
    - [Sub-Domain Routing](#route-group-sub-domain-routing)
    - [Tiền tố cho Route](#route-group-prefixes)
    - [Tiền tố cho tên Route](#route-group-name-prefixes)
- [Liên kết Route Model](#route-model-binding)
    - [Liên kết ngầm](#implicit-binding)
    - [Liên kết rõ ràng](#explicit-binding)
- [Form Method giả](#form-method-spoofing)
- [Truy cập vào Route hiện tại](#accessing-the-current-route)

<a name="basic-routing"></a>
## Routing cơ bản

Các route cơ bản nhất của Laravel chấp nhận một URI và một `Closure`, cung cấp một phương pháp xác định route rất đơn giản và có ý nghĩa:

    Route::get('foo', function () {
        return 'Hello World';
    });

#### Các file Route mặc định

Tất cả các route của Laravel được định nghĩa trong các file route của bạn, được đặt trong thư mục `routes`. Các file này được tự động load bởi framework. File `routes/web.php` xác định các route dành cho giao diện web của bạn. Các route này sẽ được gán với nhóm middleware `web`, cung cấp các tính năng như trạng thái session và bảo vệ CSRF. Các route trong `routes/api.php` là không có trạng thái và được chỉ định gán với nhóm middleware `api`.

Đối với hầu hết các application, bạn sẽ bắt đầu bằng cách xác định các route trong file `routes/web.php` của bạn. Các route đã được tạo trong file `routes/web.php` có thể được truy cập bằng cách nhập URL của route đó vào trong trình duyệt của bạn. Ví dụ: bạn có thể truy cập vào route sau bằng cách điều hướng tới `http://your-app.dev/user` trong trình duyệt của bạn:

    Route::get('/user', 'UserController@index');

Các route được định nghĩa trong file `routes/api.php` sẽ được lồng trong một nhóm route bởi` RouteServiceProvider`. Trong nhóm này, tiền tố URI `/api` sẽ được tự động áp dụng, do đó bạn không cần phải áp dụng thủ công cho mọi route trong file. Bạn có thể sửa đổi tiền tố này và các tùy chọn khác của nhóm route bằng cách sửa đổi class `RouteServiceProvider` của bạn.

#### Router Method có sẵn

Router cho phép bạn đăng ký route đáp ứng nhiều phương thức HTTP:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Thỉnh thoảng, bạn có thể cần phải đăng ký một route đáp ứng với nhiều phương thức từ HTTP. Bạn có thể làm như vậy bằng cách sử dụng phương thức `match`. Hoặc, bạn thậm chí có thể đăng ký một route đáp ứng với tất cả các động từ HTTP bằng cách sử dụng phương thức `any`:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### Bảo vệ CSRF

Bất kỳ form HTML nào trỏ đến các route  `POST`,` PUT` hoặc `DELETE` được xác định trong tệp route `web` phải chứa một field CSRF token. Nếu không, request sẽ bị từ chối. Bạn có thể đọc thêm về bảo vệ CSRF trong [tài liệu CSRF](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

<a name="redirect-routes"></a>
### Chuyển hướng Route

Nếu bạn đang xác định route chuyển hướng đến một URI khác, bạn có thể sử dụng phương thức `Route::redirect`. Phương pháp này cung cấp một lối tắt thuận tiện để bạn không phải xác định route đầy đủ hoặc controller để thực hiện một chuyển hướng đơn giản:

    Route::redirect('/here', '/there', 301);

<a name="view-routes"></a>
### View Routes

Nếu route của bạn chỉ cần trả về một view, bạn có thể sử dụng phương thức `Route::view`. Giống như phương thức `redirect`, phương thức này cung cấp một lối tắt đơn giản để bạn không phải xác định một route hoặc controller đầy đủ. Phương thức `view` chấp nhận URI làm đối số đầu tiên và tên view làm đối số thứ hai. Ngoài ra, bạn có thể cung cấp một mảng data để chuyển đến view dưới dạng tùy chọn đối số thứ ba:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## Route Parameters

<a name="required-parameters"></a>
### Tham số bắt buộc

Tất nhiên, đôi khi bạn sẽ cần phải lấy các tham số của URI trong route của bạn. Ví dụ: bạn có thể cần lấy ID người dùng từ URL. Bạn có thể làm như vậy bằng cách xác định các route parameter:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

Bạn có thể định nghĩa nhiều route parameter bắt buộc trong route của bạn:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Các tham số route luôn nằm trong các dấu ngoặc `{}` và phải chứa các ký tự chữ cái và không được chứa ký tự `-`. Thay vì sử dụng ký tự `-`, hãy sử dụng dấu gạch dưới (`_`). Các tham số route được injected vào các route callback hoặc controller dựa trên thứ tự của chúng - tên của các đối số callback hoặc controller không quan trọng.

<a name="parameters-optional-parameters"></a>
### Tham số tuỳ chọn

Đôi khi bạn có thể cần chỉ định một route parameter theo hướng tùy chọn. Bạn có thể làm như vậy bằng cách đặt dấu `?` sau tên tham số. Và hãy chắc chắn là biến tương ứng trong route có set một giá trị mặc định:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Ràng buộc biểu thức chính quy

Bạn có thể hạn chế định dạng của các tham số route của mình bằng cách sử dụng phương thức `where` trên một instance route. Phương thức `where` chấp nhận tên của tham số và biểu thức chính quy xác định cách tham số bị ràng buộc:

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### Ràng buộc Global

Nếu bạn muốn một tham số route luôn bị ràng buộc bởi một biểu thức chính quy định, bạn có thể sử dụng phương thức `pattern`. Bạn có thể định nghĩa các pattern này trong phương thức `boot` trong` RouteServiceProvider`:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

Khi pattern đã được định định, nó sẽ tự động được áp dụng cho tất cả các route sử dụng tên tham số đó:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="named-routes"></a>
## Tên của Route

Các tên của route cho phép thuận tiện tạo các URL hoặc các chuyển hướng cho các route cụ thể. Bạn có thể đặt tên cho một route bằng cách kết nối phương thức `name` vào định nghĩa route:

    Route::get('user/profile', function () {
        //
    })->name('profile');

Bạn cũng có thể đặt tên route cho các hành động của controller:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### Tạo URLs từ tên route

Khi bạn đã gán tên cho một route, bạn có thể sử dụng tên của route khi tạo URL hoặc chuyển hướng qua hàm `route` global:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

Nếu tên route của bạn có định nghĩa tham số, bạn có thể chuyển các tham số đó làm đối số thứ hai trong hàm `route`. Các tham số đó sẽ tự động được chèn vào URL ở vị trí chính xác của chúng:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### Kiểm tra Route hiện tại

Nếu bạn muốn xác định xem request hiện tại có đúng với một route đã đặt tên hay không, bạn có thể sử dụng phương thức `named` trên một instance route. Ví dụ: bạn có thể kiểm tra tên route hiện tại từ một middleware route:

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## Nhóm Route

Nhóm route cho phép bạn chia sẻ các thuộc tính route, chẳng hạn như middleware hoặc namespaces trên một số lượng lớn các route mà không cần định nghĩa các thuộc tính đó trên mỗi route. Các thuộc tính được chia sẻ được chỉ định trong một định dạng mảng là tham số đầu tiên cho phương thức `Route::group`.

<a name="route-group-middleware"></a>
### Middleware

Để gán middleware cho tất cả các route trong một nhóm, bạn có thể sử dụng phương thức `middleware` ở trước định nghĩa nhóm route. Middleware được thực hiện theo thứ tự mà nó đã được liệt kê trong mảng:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second Middleware
        });

        Route::get('user/profile', function () {
            // Uses first & second Middleware
        });
    });

<a name="route-group-namespaces"></a>
### Namespaces

Một trường hợp sử dụng phổ biến khác cho các nhóm route là gán cùng một namespace PHP cho một nhóm các controller bằng cách sử dụng phương thức `namespace`:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Nhớ rằng, mặc định, `RouteServiceProvider` đã khai báo tất cả các file route của bạn vào trong một nhóm namespace từ trước, cho phép bạn đăng ký các controller cho route mà không cần chỉ định namespace đầy đủ của nó là `App\Http\Controllers`. Vì vậy, bạn chỉ cần định nghĩa phần namespace xuất hiện sau namespace `App\Http\Controllers`.

<a name="route-group-sub-domain-routing"></a>
### Routing cho tên miền phụ

Nhóm route cũng có thể được sử dụng để xử lý các route dành cho tên miền phụ. Tên miền phụ có thể được định nghĩa thông qua tham số route giống như URI route, cho phép bạn lấy một phần của tên miền phụ để sử dụng trong route hoặc controller của bạn. Tên miền phụ có thể được chỉ định bằng cách gọi phương thức `domain` ở trước định nghĩa nhóm route:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### Tiền tố cho Route

Phương thức `prefix` có thể được sử dụng để làm tiền tố cho mỗi route trong nhóm với một URI đã cho. Ví dụ: bạn có thể muốn đặt tiền tố cho tất cả các URI route trong nhóm với tiền tối là `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### Tiền tố cho tên Route

Phương thức `name` có thể được sử dụng để đặt tiền tố cho mỗi tên route trong nhóm với một chuỗi. Ví dụ: bạn có thể muốn đặt tiền tố cho tất cả các tên của route đã được nhóm với tên `admin`. Cái chuỗi đã được đặt tiền tố sẽ được gán với tên route, nên vì thế chúng ta nên chắc chắn là đã thêm dấu `.` vào trong tiền tố để dễ phân biệt tiền tố và tên route:

    Route::name('admin.')->group(function () {
        Route::get('users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## Liên kết Route Model

Khi inject một model ID vào một route hoặc một controller action, bạn sẽ thường truy vấn để lấy ra model tương ứng với ID đó. Liên kết route model của Laravel sẽ cung cấp một cách thuận tiện để tự động đưa các instance của model trực tiếp vào các route của bạn. Ví dụ, thay vì inject user ID, bạn có thể inject một instance model `User` khớp với ID đã cho.

<a name="implicit-binding"></a>
### Liên kết ngầm

Laravel sẽ tự động resolve các model Eloquent được định nghĩa trong các route hoặc controller action có kiểu tên biến khớp với tên tham số route. Ví dụ:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Do biến `$user` được có kiểu là model `App\User` Eloquent và tên biến khớp với tham số URI `{user}`, nên Laravel sẽ tự động inject một instance model có ID khớp với giá trị tương ứng từ URI request. Nếu không tìm thấy instance model phù hợp trong cơ sở dữ liệu, phản hồi HTTP 404 sẽ tự động được tạo.

#### Tuỳ chỉnh tên Key

Nếu bạn muốn tuỳ chỉnh liên kết model sử dụng một column khác, khác với column `id` trong cơ sở dữ liệu khi truy xuất một model class, bạn có thể ghi đè phương thức `getRouteKeyName` trong model Eloquent:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### Liên kết rõ ràng

Để đăng ký một liên kết rõ ràng,  hãy sử dụng phương thức `model` của router để định định một class cho một tham số đã cho. Bạn nên định nghĩa các liên kết model rõ ràng của mình trong phương thức `boot` trong class` RouteServiceProvider`:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Tiếp theo, hãy định nghĩa một route chứa tham số `{user}`:

    Route::get('profile/{user}', function ($user) {
        //
    });

Vì chúng ta đã liên kết tất cả các tham số `{user}` vào model `App\User`, nên một instance `User` sẽ được inject vào route. Vì vậy, ví dụ, một request với uri là `profile/1` sẽ inject instance `User` từ cơ sở dữ liệu có ID là `1`.

Nếu không tìm thấy model instance phù hợp trong cơ sở dữ liệu, phản hồi HTTP 404 sẽ được tạo tự động.

#### Tuỳ chỉnh logic phụ thuộc

Nếu bạn muốn sử dụng logic phụ thuộc của riêng mình, bạn có thể sử dụng phương thức `Route::bind`. `Closure` của bạn sẽ được truyền đến phương thức` bind` và sẽ nhận giá trị tham số của URI và sẽ trả về một instance của class cái mà được inject vào trong route trước đó:

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }

<a name="form-method-spoofing"></a>
## Form Method giả

Các HTML form không hỗ trợ các hành động `PUT`, `PATCH` hoặc `DELETE`. Vì vậy, khi định nghĩa các route `PUT`, `PATCH` hoặc `DELETE` được gửi từ HTML form, bạn sẽ cần thêm input hidden `_method` vào form. Giá trị được gửi vào input hidden `_method` đó sẽ được sử dụng làm request method HTTP:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Bạn có thể dùng helper `method_field` để tạo input `_method`:

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## Truy cập vào Route hiện tại

Bạn có thể dùng các phương thức `current`, `currentRouteName`, và `currentRouteAction` trong facade `Route` để truy cập các thông tin về route đang được gọi để xử lý cho một request đến:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Tham khảo tài liệu API cho [class Route facade](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) và cả [Route instance](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) để biết tất cả các phương thức có thể truy cập được.
