# Controllers

- [Giới thiệu](#introduction)
- [Viết controllers](#writing-controllers)
    - [Controller cơ bản](#basic-controllers)
    - [Single Action Controller](#single-action-controllers)
- [Controller Middleware](#controller-middleware)
- [Resource Controller](#resource-controllers)
    - [Partial Resource Routes](#restful-partial-resource-routes)
    - [Nested Resources](#restful-nested-resources)
    - [Naming Resource Routes](#restful-naming-resource-routes)
    - [Naming Resource Route Parameters](#restful-naming-resource-route-parameters)
    - [Scoping Resource Routes](#restful-scoping-resource-routes)
    - [Localizing Resource URIs](#restful-localizing-resource-uris)
    - [Supplementing Resource Controller](#restful-supplementing-resource-controllers)
    - [Singleton Resource Controllers](#singleton-resource-controllers)
- [Dependency Injection và Controller](#dependency-injection-and-controllers)

<a name="introduction"></a>
## Giới thiệu

Thay vì định nghĩa tất cả các logic xử lý cho request trong file route của bạn với closures, thì bạn có thể muốn tổ chức các hành vi này bằng cách sử dụng class "controller". Các controller có thể nhóm các logic xử lý request có liên quan đến nhau thành một class duy nhất. Ví dụ: class `UserController` có thể xử lý tất cả các request liên quan đến người dùng, bao gồm hiển thị, tạo, cập nhật và xóa người dùng. Mặc định, controller sẽ được lưu trong thư mục `app/Http/Controllers`.

<a name="writing-controllers"></a>
## Viết controllers

<a name="basic-controllers"></a>
### Controller cơ bản

Hãy xem một ví dụ về một class controller cơ bản. Lưu ý rằng controller được extend từ một class controller cơ sở đã được đi kèm với Laravel: `App\Http\Controllers\Controller`:

    <?php

    namespace App\Http\Controllers;

    use App\Models\User;

    class UserController extends Controller
    {
        /**
         * Show the profile for a given user.
         *
         * @param  int  $id
         * @return \Illuminate\View\View
         */
        public function show($id)
        {
            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

Bạn có định nghĩa một route tới một phương thức của controller này như sau:

    use App\Http\Controllers\UserController;

    Route::get('/user/{id}', [UserController::class, 'show']);

Khi một request khớp với URI route đã chỉ định, phương thức `show` trên class `App\Http\Controllers\UserController` sẽ được gọi và các tham số route sẽ được truyền cho phương thức.

> **Note**
> Các controller không **bắt buộc** phải extend từ một class base. Tuy nhiên, bạn sẽ không có quyền truy cập vào các chức năng tiện lợi như phương thức `middleware` và `authorize`.

<a name="single-action-controllers"></a>
### Single Action Controllers

Nếu một controller action đặc biệt phức tạp, bạn có thể thấy thuận tiện khi dành toàn bộ class controller đó cho một action. Để thực hiện điều này, bạn có thể định nghĩa một phương thức `__invoke` trong controller:

    <?php

    namespace App\Http\Controllers;

    use App\Models\User;

    class ProvisionServer extends Controller
    {
        /**
         * Provision a new web server.
         *
         * @return \Illuminate\Http\Response
         */
        public function __invoke()
        {
            // ...
        }
    }

Khi đăng ký route cho một single action controller, bạn sẽ không cần phải khai báo thêm tên phương thức của controller nữa. Thay vào đó, bạn có thể chỉ cần truyền tên của controller đó cho router:

    use App\Http\Controllers\ProvisionServer;

    Route::post('/server', ProvisionServer::class);

Bạn có thể tạo một controller chỉ có một action duy nhất bằng cách sử dụng tùy chọn `--invokable` trong lệnh Artisan `make:controller`:

```shell
php artisan make:controller ProvisionServer --invokable
```

> **Note**
> Bạn có thể tùy chỉnh các stub của controller bằng cách [export chúng](/docs/{{version}}/artisan#stub-customization).

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/{{version}}/middleware) có thể được gán vào route của controller trong file route của bạn:

    Route::get('profile', [UserController::class, 'show'])->middleware('auth');

Hoặc, bạn có thể thấy thuận tiện hơn khi khai báo middleware đó trong hàm khởi tạo của controller của bạn. Sử dụng phương thức `middleware` trong hàm khởi tạo của controller, bạn có thể gán middleware vào các action của controller.

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

Controller cũng cho phép bạn đăng ký các middleware bằng cách sử dụng một closure. Điều này cung cấp một cách thuận tiện để định nghĩa middleware bên trong một single Controller mà không cần phải định nghĩa thêm một class middleware:

    $this->middleware(function ($request, $next) {
        return $next($request);
    });

<a name="resource-controllers"></a>
## Resource Controllers

Nếu bạn coi mỗi model Eloquent trong ứng dụng của bạn là một "resource", thì thông thường bạn sẽ thực hiện nhiều hành động giống nhau cho từng resource trong ứng dụng của bạn. Ví dụ: hãy tưởng tượng ứng dụng của bạn chứa một model `Photo` và một model `Movie`. Và người dùng có thể tạo, đọc, cập nhật hoặc xóa các resource này.

Do đây là những trường hợp rất hay sử dụng, nên route của Laravel resource sẽ chỉ định các route tạo, đọc, cập nhật và xóa ("CRUD") mặc định cho controller với một số dòng code mặc định. Để bắt đầu, chúng ta có thể sử dụng tùy chọn `--resource` của lệnh `make:controller` Artisan để nhanh chóng tạo một controller để xử lý các hành động này:

```shell
php artisan make:controller PhotoController --resource
```

Lệnh này sẽ tạo ra một controller tại `app/Http/Controllers/PhotoController.php`. Controller này sẽ chứa một phương thức cho mỗi hành động resource có sẵn. Tiếp theo, bạn có thể đăng ký một resource route trỏ đến controller:

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class);

Khai báo một single route như ở trên sẽ tạo ra một loạt route để xử lý một loạt các hành động khác nhau trên resource. Controller đã được tạo sẽ có sẵn luôn các phương thức cho từng hành động này. Hãy nhớ rằng, bạn luôn có thể xem qua các route của ứng dụng của bạn bằng cách chạy lệnh `route:list` Artisan.

Bạn thậm chí có thể đăng ký nhiều resource controller cùng một lúc bằng cách truyền vào một mảng cho phương thức `resources`:

    Route::resources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

<a name="actions-handled-by-resource-controller"></a>
#### Các hành động được xử lý bởi Resource Controller

Verb      | URI                    | Action       | Route Name
----------|------------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

<a name="customizing-missing-model-behavior"></a>
#### Customizing Missing Model Behavior

Thông thường, response HTTP 404 sẽ được tạo nếu không tìm thấy resource model mà ràng buộc ngầm. Tuy nhiên, bạn có thể tùy chỉnh hành động này bằng cách gọi phương thức `missing` khi định nghĩa resource route của bạn. Phương thức `missing` sẽ chấp nhận một closure sẽ được gọi đến nếu không thể tìm thấy một model khi ràng buộc ngầm cho bất kỳ route nào cho resource:

    use App\Http\Controllers\PhotoController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::resource('photos', PhotoController::class)
            ->missing(function (Request $request) {
                return Redirect::route('photos.index');
            });

<a name="soft-deleted-models"></a>
#### Soft Deleted Models

Thông thường, liên kết model ẩn sẽ không thể lấy ra các model đã bị [soft delete](/docs/{{version}}/eloquent#soft-deleting) và thay vào đó sẽ trả về response HTTP 404. Tuy nhiên, bạn có thể hướng dẫn framework cho phép lấy ra các model bị soft delete bằng cách gọi phương thức `withTrashed` khi định nghĩa route resource của bạn:

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->withTrashed();

Việc gọi `withTrashed` không có tham số sẽ cho phép lấy ra các model bị soft delete cho các route resource `show`, `edit` và `update`. Bạn có thể chỉ định một tập con của các route bằng cách truyền một mảng tới phương thức `withTrashed`:

    Route::resource('photos', PhotoController::class)->withTrashed(['show']);

<a name="specifying-the-resource-model"></a>
#### Khai báo Resource Model

Nếu bạn đang sử dụng [liên kết model route](/docs/{{version}}/routing#route-model-binding) và muốn các phương thức của resource controller khai báo sẵn một tham số đầu vào là một model instance, bạn có thể sử dụng tùy chọn `--model` khi tạo controller:

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

<a name="generating-form-requests"></a>
#### Generating Form Requests

Bạn có thể cung cấp tùy chọn `--requests` khi tạo resource controller để hướng dẫn Artisan tạo [các class request validation form](/docs/{{version}}/validation#form-request-validation) cho các phương thức cập nhật và lưu trữ của controller:

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

<a name="restful-partial-resource-routes"></a>
### Partial Resource Routes

Khi khai báo một resource route, bạn có thể chỉ định một tập hợp các hành động mà được controller xử lý thay vì toàn bộ các hành động mặc định:

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->only([
        'index', 'show'
    ]);

    Route::resource('photos', PhotoController::class)->except([
        'create', 'store', 'update', 'destroy'
    ]);

<a name="api-resource-routes"></a>
#### API Resource Routes

Khi khai báo một resource route mà sẽ được sử dụng bởi các API, bạn sẽ muốn loại bỏ các route mà phải nhập form HTML như `create` và` edit`. Để thuận tiện, bạn có thể sử dụng phương thức `apiResource` để tự động loại bỏ hai route trên:

    use App\Http\Controllers\PhotoController;

    Route::apiResource('photos', PhotoController::class);

Bạn có thể đăng ký nhiều resource controller cho API cùng một lúc bằng cách truyền một mảng vào phương thức `apiResources`:

    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\PostController;

    Route::apiResources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

Để tạo nhanh một API resource controller mà không chứa các phương thức `create` hoặc `edit`, hãy sử dụng switch `--api` khi chạy lệnh `make:controller`:

```shell
php artisan make:controller PhotoController --api
```

<a name="restful-nested-resources"></a>
### Nested Resources

Thỉnh thoảng bạn có thể cần định nghĩa các route đến một resource lồng nhau. Ví dụ: một resource photo có thể có nhiều nhận xét được đính kèm vào ảnh. Để lồng các resource controller, bạn có thể sử dụng ký tự "chấm" trong khai báo route của bạn:

    use App\Http\Controllers\PhotoCommentController;

    Route::resource('photos.comments', PhotoCommentController::class);

Route này sẽ đăng ký một resource có thể truy cập được bằng các URI như sau:

    /photos/{photo}/comments/{comment}

<a name="scoping-nested-resources"></a>
#### Scoping Nested Resources

Chức năng [liên kết ngầm model](/docs/{{version}}/routing#implicit-model-binding-scoping) của Laravel có thể tự động xác định scope của các liên kết lồng nhau, sao cho các model con đã được resolve thì sẽ được xác nhận là thuộc về model cha. Bằng cách sử dụng phương thức `scoped` khi định nghĩa resource lồng nhau, bạn có thể bật chế độ scope tự động cũng như hướng dẫn Laravel, trường nào của resource con sẽ được lấy ra. Để biết thêm thông tin về cách thực hiện việc này, vui lòng xe thêm tài liệu về [route resource theo scope](#restful-scoping-resource-routes).

<a name="shallow-nesting"></a>
#### Shallow Nesting

Thông thường, không nhất thiết phải có cả ID cha và ID con trong một URI vì ID con có thể có chứa ID của cha. Khi sử dụng ID, chẳng hạn như một khóa chính tự động tăng dần để xác định model của bạn trong các phân đoạn URI, bạn có thể chọn sử dụng "shallow nesting":

    use App\Http\Controllers\CommentController;

    Route::resource('photos.comments', CommentController::class)->shallow();

Định nghĩa route này sẽ định nghĩa ra các route như sau:

Verb      | URI                               | Action       | Route Name
----------|-----------------------------------|--------------|---------------------
GET       | `/photos/{photo}/comments`        | index        | photos.comments.index
GET       | `/photos/{photo}/comments/create` | create       | photos.comments.create
POST      | `/photos/{photo}/comments`        | store        | photos.comments.store
GET       | `/comments/{comment}`             | show         | comments.show
GET       | `/comments/{comment}/edit`        | edit         | comments.edit
PUT/PATCH | `/comments/{comment}`             | update       | comments.update
DELETE    | `/comments/{comment}`             | destroy      | comments.destroy

<a name="restful-naming-resource-routes"></a>
### Naming Resource Routes

Mặc định, tất cả các hành động của resource controller đều có đi kèm với một tên route; tuy nhiên, bạn có thể ghi đè các tên này bằng cách truyền vào một mảng `names` cùng với tên route mà bạn mong muốn:

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->names([
        'create' => 'photos.build'
    ]);

<a name="restful-naming-resource-route-parameters"></a>
### Naming Resource Route Parameters

Mặc định, `Route::resource` sẽ tạo các tham số route cho các resource route dựa trên tên "số ít" của các resource. Bạn có thể dễ dàng ghi đè điều này trên từng resource bằng cách sử dụng phương thức `parameters`. Mảng được truyền cho phương thức `parameters` này phải là một mảng kết hợp giữa tên resource và tên tham số:

    use App\Http\Controllers\AdminUserController;

    Route::resource('users', AdminUserController::class)->parameters([
        'users' => 'admin_user'
    ]);

Ví dụ ở trên sẽ tạo ra một URI như ở dưới cho một route `show` của resource:

    /users/{admin_user}

<a name="restful-scoping-resource-routes"></a>
### Scoping Resource Routes

Chức năng [scope liên kết ngầm model](/docs/{{version}}/routing#implicit-model-binding-scoping) của Laravel có thể tự động xác định scope của các liên kết lồng nhau, sao cho các model con đã được resolve thì sẽ được xác nhận là thuộc về model cha. Bằng cách sử dụng phương thức `scoped` khi định nghĩa resource lồng nhau, bạn có thể bật chế độ scope tự động cũng như hướng dẫn Laravel, trường nào của resource con sẽ được lấy ra bằng:

    use App\Http\Controllers\PhotoCommentController;

    Route::resource('photos.comments', PhotoCommentController::class)->scoped([
        'comment' => 'slug',
    ]);

Route này sẽ đăng ký một scoped nested resource có thể được truy cập bằng các URI như sau:

    /photos/{photo}/comments/{comment:slug}

Khi sử dụng liên kết ngầm có key tùy biến làm một tham số route lồng nhau, Laravel sẽ tự động scope truy vấn để lấy ra các model lồng nhau thông qua cha của nó bằng cách sử dụng các quy ước để đặt tên quan hệ trên cha. Trong trường hợp này, sẽ giả định rằng model `Photo` có một quan hệ có tên là `comments` (số nhiều của tên tham số route) có thể được sử dụng để lấy ra model `Comment`.

<a name="restful-localizing-resource-uris"></a>
### Localizing Resource URIs

Mặc định, `Route::resource` sẽ tạo các URI resource bằng các động từ và quy tắc số nhiều trong tiếng Anh. Nếu bạn cần bản địa hóa các động từ này như `create` và `edit`, bạn có thể sử dụng phương thức `Route::resourceVerbs`. Điều này có thể được thực hiện ở đầu của phương thức `boot` trong `App\Providers\RouteServiceProvider` của ứng dụng của bạn:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);

        // ...
    }

Bộ quy tắc số nhiều của Laravel hỗ trợ [một số ngôn ngữ khác nhau mà bạn có thể cấu hình dựa trên nhu cầu của bạn](/docs/{{version}}/localization#pluralization-language). Khi các động từ và quy tắc số nhiều đã được tùy biến xong, nếu bạn đăng ký resource route là `Route::resource('publicacion', PublicacionController::class)` thì sẽ tạo ra các URI như sau:

    /publicacion/crear

    /publicacion/{publicaciones}/editar

<a name="restful-supplementing-resource-controllers"></a>
### Supplementing Resource Controllers

Nếu bạn cần thêm các route vào các resource controller ngoài các route mặc định, bạn nên định nghĩa các route đó trước khi gọi tới phương thức `Route::resource`; Mặt khác, các route được định nghĩa bởi phương thức `resource` có thể vô tình được ưu tiên hơn các route vừa được thêm của bạn:

    use App\Http\Controller\PhotoController;

    Route::get('/photos/popular', [PhotoController::class, 'popular']);
    Route::resource('photos', PhotoController::class);

> **Note**
>  Hãy nhớ giữ cho controller của bạn được tập trung. Nếu bạn cảm thấy bạn thường xuyên cần phải thêm các phương thức bên ngoài bộ resource action mặc định, hãy xem xét việc chia controller của bạn thành hai controller nhỏ hơn.

<a name="singleton-resource-controllers"></a>
### Singleton Resource Controllers

Thỉnh thoảng, ứng dụng của bạn sẽ có các resources mà chỉ có thể có một instance duy nhất. Ví dụ: "profile" của người dùng có thể được chỉnh sửa hoặc cập nhật, nhưng một người dùng chỉ thể có nhiều nhất một "profile". Tương tự như vậy, một hình ảnh có thể có một "hình thu nhỏ". Những resources này được gọi là "resources singleton", nghĩa là một và chỉ một instance của resources có thể tồn tại. Trong những trường hợp như thế này, bạn có thể đăng ký resource controller "singleton":

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);
```

Định nghĩa resources singleton ở trên sẽ tạo ra các đăng ký route như sau. Như bạn có thể thấy, các route "creation" không được đăng ký cho các resources singleton và các route đã đăng ký sẽ không chấp nhận id vì chỉ có một instance của resource có thể tồn tại:

Verb      | URI                               | Action       | Route Name
----------|-----------------------------------|--------------|---------------------
GET       | `/profile`                        | show         | profile.show
GET       | `/profile/edit`                   | edit         | profile.edit
PUT/PATCH | `/profile`                        | update       | profile.update

Resources singleton cũng có thể được lồng trong một resource tiêu chuẩn:

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

Trong ví dụ này, resource `photos` sẽ nhận được tất cả [route resource tiêu chuẩn](#actions-handled-by-resource-controller); tuy nhiên, resource `thumbnail` sẽ là resource singleton với các route như sau:

| Verb      | URI                              | Action  | Route Name               |
|-----------|----------------------------------|---------|--------------------------|
| GET       | `/photos/{photo}/thumbnail`      | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit` | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | update  | photos.thumbnail.update  |

<a name="creatable-singleton-resources"></a>
#### Creatable Singleton Resources

Đôi khi, bạn có thể muốn định nghĩa thêm các route creation và storage cho một resource singleton. Để thực hiện điều này, bạn có thể gọi phương thức `creatable` khi đăng ký route resource singleton:

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

Trong ví dụ này, các route sau sẽ được đăng ký. Như bạn có thể thấy, route `DELETE` cũng sẽ được đăng ký cho các resource singleton:

| Verb      | URI                                | Action  | Route Name               |
|-----------|------------------------------------|---------|--------------------------|
| GET       | `/photos/{photo}/thumbnail/create` | create  | photos.thumbnail.create  |
| POST      | `/photos/{photo}/thumbnail`        | store   | photos.thumbnail.store   |
| GET       | `/photos/{photo}/thumbnail`        | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit`   | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`        | update  | photos.thumbnail.update  |
| DELETE    | `/photos/{photo}/thumbnail`        | destroy | photos.thumbnail.destroy |

Nếu bạn muốn Laravel đăng ký route `DELETE` cho một resource singleton nhưng không đăng ký các route creation hoặc storage khác, bạn có thể sử dụng phương thức `destroyable`:

```php
Route::singleton(...)->destroyable();
```

<a name="api-singleton-resources"></a>
#### API Singleton Resources

Phương thức `apiSingleton` có thể được sử dụng để đăng ký một resource singleton sẽ được thao tác thông qua API, và do đó các route `create` và `edit` sẽ trở nên không cần thiết:

```php
Route::apiSingleton('profile', ProfileController::class);
```

Tất nhiên, các resource singleton API cũng có thể là loại `creatable`, sẽ đăng ký các route `store` và `destroy` cho resource:

```php
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();
```

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection và Controllers

<a name="constructor-injection"></a>
#### Constructor Injection

Laravel [service container](/docs/{{version}}/container) sẽ được sử dụng để resolve tất cả các controller của Laravel. Kết quả là, bạn có thể khai báo cho bất kỳ phụ thuộc nào mà controller của bạn cần trong hàm khởi tạo của nó. Các phụ thuộc được khai báo sẽ tự động được resolve và được đưa vào trong controller instance:

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
         * @param  \App\Repositories\UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

<a name="method-injection"></a>
#### Method Injection

Ngoài việc khai báo vào hàm khởi tạo class, bạn cũng có thể khai báo sự phụ thuộc vào trực tiếp các phương thức của controller. Một trường hợp được sử dụng phổ biến cho việc khai báo theo kiểu này là việc khai báo instance `Illuminate\Http\Request` cho các phương thức controller của bạn:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

Nếu phương thức controller của bạn cũng đang sử dụng các tham số route, bạn có thể liệt kê các tham số đó sau các phụ thuộc của bạn. Ví dụ: nếu route của bạn đang được định nghĩa như sau:

    use App\Http\Controllers\UserController;

    Route::put('/user/{id}', [UserController::class, 'update']);

Thì bạn vẫn có thể khai báo `Illuminate\Http\Request` và truy cập tham số `id` của bạn bằng cách định nghĩa phương thức controller của bạn như thế này:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the given user.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  string  $id
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }
