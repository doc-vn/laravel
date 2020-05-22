# Controllers

- [Giới thiệu](#introduction)
- [Controller cơ bản](#basic-controllers)
    - [Định nghĩa Controller](#defining-controllers)
    - [Controller và Namespaces](#controllers-and-namespaces)
    - [Single Action Controller](#single-action-controllers)
- [Controller Middleware](#controller-middleware)
- [Resource Controller](#resource-controllers)
    - [Partial Resource Routes](#restful-partial-resource-routes)
    - [Naming Resource Routes](#restful-naming-resource-routes)
    - [Naming Resource Route Parameters](#restful-naming-resource-route-parameters)
    - [Localizing Resource URIs](#restful-localizing-resource-uris)
    - [Supplementing Resource Controller](#restful-supplementing-resource-controllers)
- [Dependency Injection và Controller](#dependency-injection-and-controllers)
- [Route Caching](#route-caching)

<a name="introduction"></a>
## Giới thiệu

Thay vì định nghĩa tất cả các code logic xử lý cho request của bạn trong file route với Closures, bạn có thể thay đổi hành vi này bằng cách dùng class Controller. Các controller có thể nhóm các code logic xử lý request liên quan đến nhau thành một class duy nhất. Các controller sẽ được lưu trữ trong thư mục `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Controller cơ bản

<a name="defining-controllers"></a>
### Định nghĩa Controller

Dưới đây là một ví dụ về một class controller cơ bản. Lưu ý rằng controller được extend từ một class controller cơ sở được đi kèm với Laravel. Class cơ sở cung cấp một vài phương thức tiện lợi, như phương thức `middleware`, có thể được sử dụng để gắn middleware vào các controller action:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Bạn có định nghĩa một route tới controller này như sau:

    Route::get('user/{id}', 'UserController@show');

Bây giờ, khi một request khớp với URI route đã được đinh nghĩa, phương thức `show` trong class `UserController` sẽ được thực thi. Dĩ nhiên, các tham số route cũng sẽ được truyền đến phương thức.

> {tip} Các controller không **yêu cầu** bạn phải extend từ một class cơ sở. Nhưng, bạn sẽ không thể truy cập vào một số phương thức tiện lợi như các phương thức `middleware`, `validate` và `dispatch`.

<a name="controllers-and-namespaces"></a>
### Controllers và Namespaces

Đây là một điều rất quan trọng cần lưu ý là chúng ta không cần khai báo toàn bộ namespace đến controller khi định nghĩa controller cho route. Vì `RouteServiceProvider` sẽ tải các file route của bạn vào trong một group route có chứa namespace, nên chúng ta chỉ khai báo phần tên class xuất hiện sau phần `App\Http\Controllers` của namespace.

Nếu bạn có một controller ở trong thư mục con của thư mục `App\Http\Controllers`, hãy khai báo tên class bắt đầu từ sau namespace `App\Http\Controllers`. Vì thế, nếu path đầy đủ của controller của bạn là `App\Http\Controllers\Photos\AdminController`, bạn nên đăng ký các route đến controller như sau:

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### Single Action Controllers

Nếu bạn muốn định nghĩa một controller chỉ xử lý một hành động, bạn có thể đặt một single phương thức `__invoke` trên controller:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Khi đăng ký route cho single action controller, bạn sẽ không cần khai báo tên phương thức nữa:

    Route::get('user/{id}', 'ShowProfile');

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/{{version}}/middleware) có thể được gán vào route của controller trong file route của bạn:

    Route::get('profile', 'UserController@show')->middleware('auth');

Tuy nhiên, sẽ thuận tiện hơn khi khai báo middleware trong hàm khởi tạo của controller của bạn. Sử dụng phương thức `middleware` từ hàm khởi tạo của controller, bạn có thể dễ dàng gán middleware cho các action của controller. Bạn thậm chí có thể giới hạn middleware chỉ một số phương thức nhất định trên class controller:

    class UserController extends Controller
    {
        /**
         * Instantiate a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log')->only('index');

            $this->middleware('subscribed')->except('store');
        }
    }

Controller cũng cho phép bạn đăng ký middleware bằng cách sử dụng một Closure. Điều này cung cấp một cách thuận tiện để định nghĩa middleware cho một single Controller mà không cần định nghĩa toàn bộ class middleware:

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} Bạn có thể gán middleware cho một tập con các controller action; tuy nhiên, nó có thể làm cho controller của bạn phát triển quá lớn. Thay vào đó, hãy xem xét việc chia controller của bạn thành nhiều controller nhỏ hơn.

<a name="resource-controllers"></a>
## Resource Controllers

Laravel resource routing sẽ gán một loạt route theo kiểu "CRUD" vào một controller với chỉ một dòng code. Ví dụ: bạn muốn tạo một controller xử lý tất cả các request HTTP cho "photos" được lưu trữ bởi ứng dụng của bạn. Sử dụng lệnh Artisan `make:controller`, chúng ta có thể nhanh chóng tạo ra một controller như vậy:

    php artisan make:controller PhotoController --resource

Lệnh này sẽ tạo ra một controller tại `app/Http/Controllers/PhotoController.php`. Controller đó sẽ chứa một phương thức cho từng resource có sẵn.

Tiếp theo, bạn có thể đăng ký một resourceful route tới controller:

    Route::resource('photos', 'PhotoController');

Khai báo single route này sẽ tạo ra một loạt route để xử lý một loạt các hành động khác nhau trên resource. Controller đã được tạo sẽ có sẵn luôn các phương thức cho từng hành động này, bao gồm các note thông báo cho bạn về các method HTTP và URI mà chúng xử lý.

Bạn có thể đăng ký nhiều resource controller cùng một lúc bằng cách pass một array vào phương thức `resources`:

    Route::resources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

#### Các hành động được xử lý bởi Resource Controller

Verb      | URI                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### Khai báo Resource Model

Nếu bạn đang sử dụng liên kết model route và muốn các phương thức của resource controller khai báo sẵn tham số đầu vào là model instance, bạn có thể sử dụng tùy chọn `--model` khi tạo controller:

    php artisan make:controller PhotoController --resource --model=Photo

#### Giả Form Method

Vì HTML form không thể tạo cái request `PUT`, `PATCH`, hoặc `DELETE`, nên bạn cần thêm một hidden field `_method` để giả method HTTP. Helper `method_field` có thể tạo trường này cho bạn:

    {{ method_field('PUT') }}

<a name="restful-partial-resource-routes"></a>
### Partial Resource Routes

Khi khai báo một resource route, bạn có thể chỉ định một tập hợp các hành động mà controller sẽ xử lý thay vì toàn bộ các hành động mặc định:

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

#### API Resource Routes

Khi khai báo một resource route sẽ được sử dụng bởi các API, bạn thường muốn loại bỏ các route mà phải nhập form HTML như `create` và` edit`. Để thuận tiện, bạn có thể sử dụng phương thức `apiResource` để tự động loại bỏ hai route trên:

    Route::apiResource('photo', 'PhotoController');

Bạn có thể đăng ký nhiều resource controller API cùng một lúc bằng cách pass một array vào phương thức `apiResources`:

    Route::apiResources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

<a name="restful-naming-resource-routes"></a>
### Naming Resource Routes

Mặc định, tất cả các hành động của resource controller đều có một tên route; tuy nhiên, bạn có thể ghi đè các tên này bằng cách pass một mảng `names` cùng với các tên của bạn:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
### Naming Resource Route Parameters

Mặc định, `Route::resource` sẽ tạo các tham số route cho các resource route của bạn dựa theo tên "số ít" của tên resource. Bạn có thể dễ dàng ghi đè lên điều này trên cơ sở từng resource bằng cách pass `parameters` trong một mảng. Mảng `parameters` phải là một mảng kết hợp giữa tên resource và tên tham số:

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

Ví dụ ở trên sẽ tạo ra URI như ở dưới cho route `show` của resource:

    /user/{admin_user}

<a name="restful-localizing-resource-uris"></a>
### Localizing Resource URIs

Mặc định, `Route::resource` sẽ tạo các URI resource bằng các động từ tiếng Anh. Nếu bạn cần bản địa hóa các động từ hành động như `create` và` edit`, bạn có thể sử dụng phương thức `Route::resourceVerbs`. Điều này có thể được thực hiện trong phương thức `boot` của `AppServiceProvider`:

    use Illuminate\Support\Facades\Route;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);
    }

Khi các động từ đã được tùy biến, đăng ký resource route là `Route::resource('fotos', 'PhotoController')` thì sẽ tạo ra các URI như sau:

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>
### Supplementing Resource Controllers

Nếu bạn cần thêm các route vào resource controller ngoài route resource mặc định, bạn nên định nghĩa các route đó trước khi gọi tới `Route::resource`; mặt khác, các route được định nghĩa bởi phương thức `resource` có thể vô tình được ưu tiên hơn các route vừa thêm của bạn:

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} Hãy nhớ giữ cho controller của bạn được focus. Nếu bạn thấy bạn thường xuyên cần các phương thức bên ngoài bộ resource action mặc định, hãy xem xét việc chia controller của bạn thành hai, controller nhỏ hơn.

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection và Controllers

#### Constructor Injection

Laravel [service container](/docs/{{version}}/container) sẽ được sử dụng để resolve tất cả các controller của Laravel. Kết quả là, bạn có thể khai báo kiểu cho bất kỳ phụ thuộc nào mà controller của bạn có thể cần trong hàm khởi tạo của nó. Các phụ thuộc được khai báo sẽ tự động được resolve và đưa vào controller instance:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

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
    }

Và dĩ nhiên, bạn cũng có thể khai báo kiểu cho bất kỳ [Laravel contract](/docs/{{version}}/contracts) nào. Nếu container có thể resolve nó, bạn có thể khai báo nó. Tùy thuộc vào ứng dụng của bạn, việc đưa các phụ thuộc của bạn vào controller của bạn có thể cung cấp khả năng kiểm tra tốt hơn.

#### Method Injection

Ngoài việc inject vào hàm khởi tạo class, bạn cũng có thể khai báo kiểu phụ thuộc vào trực tiếp các phương thức của controller. Một trường hợp được sử dụng phổ biến cho  inject phương thức là inject instance `Illuminate\Http\Request` vào các phương thức controller của bạn:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

Nếu phương thức controller của bạn cũng đang expect một tham số route, hãy liệt kê các tham số route đó sau các phụ thuộc của bạn. Ví dụ: nếu route của bạn được định nghĩa như vậy:

    Route::put('user/{id}', 'UserController@update');

Thì bạn vẫn có thể khai báo kiểu `Illuminate\Http\Request` và truy cập tham số `id` của bạn bằng cách định nghĩa phương thức controller của bạn như sau:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the given user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## Route Caching

> {note} Các route mà được dựa trên Closure thì sẽ không thể cache. Để sử dụng route caching, bạn phải chuyển đổi hết các Closure route nào thành các class controller.

Nếu ứng dụng của bạn chỉ sử dụng các route mà được dựa trên controller, bạn nên tận dụng cache route của Laravel. Sử dụng cache route sẽ giảm đáng kể thời gian cần thiết để đăng ký tất cả các route của ứng dụng của bạn. Trong một số trường hợp, việc đăng ký route thậm chí có thể nhanh hơn tới 100 lần. Để tạo cache route, chỉ cần thực hiện lệnh Artisan `route:cache`:

    php artisan route:cache

Sau khi chạy lệnh trên, file route sẽ được lưu trong bộ nhớ cache của bạn và sẽ được load theo mỗi yêu cầu. Hãy nhớ rằng, nếu bạn thêm bất kỳ route mới nào, bạn sẽ cần tạo bộ cache route mới. Vì điều thế, bạn chỉ nên chạy lệnh `route:cache` trong quá trình release dự án của bạn.

Bạn có thể dùng lệnh `route:clear` để xoá route cache:

    php artisan route:clear
