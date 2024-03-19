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
    - [Controllers](#route-group-controllers)
    - [Subdomain Routing](#route-group-subdomain-routing)
    - [Tiền tố cho Route](#route-group-prefixes)
    - [Tiền tố cho tên Route](#route-group-name-prefixes)
- [Liên kết Route Model](#route-model-binding)
    - [Liên kết ngầm](#implicit-binding)
    - [Liên kết rõ ràng](#explicit-binding)
- [Route dự phòng](#fallback-routes)
- [Rate Limiting](#rate-limiting)
    - [Định nghĩa giới hạn tỷ lệ](#defining-rate-limiters)
    - [Gán giới hạn tỷ lệ vào route](#attaching-rate-limiters-to-routes)
- [Form Method giả](#form-method-spoofing)
- [Truy cập vào Route hiện tại](#accessing-the-current-route)
- [Cross-Origin Resource Sharing (CORS)](#cors)
- [Route Caching](#route-caching)

<a name="basic-routing"></a>
## Routing cơ bản

Các route cơ bản của Laravel chấp nhận một URI và một closure, cung cấp một phương thức định nghĩa route rất đơn giản, dễ hiểu và hoạt động mà không cần có file cấu hình route phức tạp:

    use Illuminate\Support\Facades\Route;

    Route::get('/greeting', function () {
        return 'Hello World';
    });

<a name="the-default-route-files"></a>
#### Các file Route mặc định

Tất cả các route của Laravel được định nghĩa trong các file route, và được lưu trong thư mục `routes`. Các file này được tự động load bởi `App\Providers\RouteServiceProvider` của application của bạn. File `routes/web.php` định nghĩa các route dành cho giao diện web của bạn. Các route này sẽ được gán với nhóm middleware `web`, cung cấp các tính năng như trạng thái session và bảo vệ CSRF. Các route trong `routes/api.php` là các route không có trạng thái và được gán với nhóm middleware `api`.

Đối với hầu hết các application, bạn sẽ bắt đầu bằng cách định nghĩa các route trong file `routes/web.php`. Các route đã được tạo trong file `routes/web.php` có thể được truy cập bằng cách nhập URL của route đó vào trong trình duyệt web của bạn. Ví dụ: bạn có thể truy cập vào route sau bằng cách nhập url là `http://example.com/user` trong trình duyệt web của bạn:

    use App\Http\Controllers\UserController;

    Route::get('/user', [UserController::class, 'index']);

Các route được định nghĩa trong file `routes/api.php` sẽ nằm trong một nhóm route bởi `RouteServiceProvider`. Trong nhóm này, tiền tố URI `/api` sẽ được tự động áp dụng, do đó bạn không cần phải tự làm cho mỗi route có trong file. Bạn có thể sửa đổi tiền tố và các tùy chọn khác cho nhóm route này bằng cách sửa đổi trong class `RouteServiceProvider` của bạn.

<a name="available-router-methods"></a>
#### Router Method có sẵn

Router cho phép bạn đăng ký route với nhiều phương thức HTTP:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Thỉnh thoảng, bạn có thể cần phải đăng ký một route với nhiều phương thức từ HTTP. Bạn có thể làm như vậy bằng cách sử dụng phương thức `match`. Hoặc, bạn thậm chí có thể đăng ký một route đáp ứng với tất cả các động từ HTTP bằng cách sử dụng phương thức `any`:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('/', function () {
        //
    });

> {tip} Khi định nghĩa nhiều route có chung một URI, các route sử dụng các phương thức `get`, `post`, `put`, `patch`, `delete` và `options` phải được định nghĩa trước các route `any`, `match` và `redirect`. Điều này giúp đảm bảo request sẽ được khớp với route chính xác.

<a name="dependency-injection"></a>
#### Dependency Injection

Bạn có thể khai báo bất kỳ sự phụ thuộc nào mà route của bạn yêu cầu trong định dạng của callback của route. Các phần phụ thuộc này sẽ tự động được resolve và đưa vào lệnh callback bởi [service container](/docs/{{version}}/container). Ví dụ: bạn có thể khai báo class `Illuminate\Http\Request` để yêu cầu HTTP hiện tại được tự động đưa vào lệnh callback route của bạn:

    use Illuminate\Http\Request;

    Route::get('/users', function (Request $request) {
        // ...
    });

<a name="csrf-protection"></a>
#### Bảo vệ CSRF

Hãy nhớ rằng, bất kỳ form HTML nào mà trỏ đến các route `POST`, `PUT`, `PATCH` hoặc `DELETE` mà được định nghĩa trong file route `web`, đều phải chứa một field CSRF token. Nếu không có field đó, request sẽ bị từ chối. Bạn có thể đọc thêm về bảo vệ CSRF trong [tài liệu CSRF](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### Chuyển hướng Route

Nếu bạn đang định nghĩa route chuyển hướng đến một URI khác, bạn có thể sử dụng phương thức `Route::redirect`. Phương pháp này cung cấp một lối tắt thuận tiện để bạn không phải định nghĩa một route đầy đủ hoặc một controller để thực hiện một chuyển hướng đơn giản:

    Route::redirect('/here', '/there');

Mặc định, `Route::redirect` sẽ trả về status code là `302`. Bạn có thể tùy chỉnh status code này bằng cách sử dụng tham số thứ ba:

    Route::redirect('/here', '/there', 301);

Hoặc, bạn có thể sử dụng phương thức `Route::permanentRedirect` để trả về status code là `301`:

    Route::permanentRedirect('/here', '/there');

> {note} Khi sử dụng tham số route trong route chuyển hướng, các tham số sau sẽ được Laravel dùng sẵn và không thể sử dụng: `destination` và `status`.

<a name="view-routes"></a>
### View Routes

Nếu route của bạn chỉ cần trả về một [view](/docs/{{version}}/views), thì bạn có thể sử dụng phương thức `Route::view`. Giống như phương thức `redirect`, phương thức này cung cấp một lối tắt đơn giản để bạn không phải định nghĩa một route hoặc một controller đầy đủ. Phương thức `view` chấp nhận URI làm tham số đầu tiên và tên view sẽ làm tham số thứ hai. Ngoài ra, bạn có thể cung cấp một mảng data để chuyển đến view dưới dạng là tùy chọn tham số thứ ba:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

> {note} Khi sử dụng tham số route trong view route, các tham số sau sẽ được Laravel dùng sẵn và không thể sử dụng: `view`, `data`, `status`, và `headers`.

<a name="route-parameters"></a>
## Route Parameters

<a name="required-parameters"></a>
### Tham số bắt buộc

Đôi khi bạn sẽ cần phải lấy các tham số của URI trong route của bạn. Ví dụ: bạn có thể cần lấy ID người dùng từ URL. Bạn có thể làm như vậy bằng cách định nghĩa các route parameter:

    Route::get('/user/{id}', function ($id) {
        return 'User '.$id;
    });

Bạn có thể định nghĩa nhiều route parameter bắt buộc trong route của bạn:

    Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Các tham số route luôn nằm trong các dấu ngoặc `{}` và phải chứa các ký tự chữ cái. Dấu gạch dưới (`_`) cũng được chấp nhận trong tên tham số route. Các tham số route sẽ được inject vào các route callback hoặc controller dựa thoe thứ tự của chúng - tên của các tham số route callback hoặc controller không quan trọng.

<a name="parameters-and-dependency-injection"></a>
#### Parameters & Dependency Injection

Nếu route của bạn có các phần phụ thuộc mà bạn muốn service container của Laravel tự động đưa vào lệnh callback của route, bạn nên liệt kê các tham số route nằm sau phần phụ thuộc của bạn:

    use Illuminate\Http\Request;

    Route::get('/user/{id}', function (Request $request, $id) {
        return 'User '.$id;
    });

<a name="parameters-optional-parameters"></a>
### Tham số tuỳ chọn

Đôi khi bạn có thể cần chỉ định một route parameter có thể không phải lúc nào cũng có trong URI. Bạn có thể làm như vậy bằng cách đặt dấu `?` sau tên tham số. Và hãy chắc chắn là biến tương ứng trong route có set một giá trị mặc định:

    Route::get('/user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('/user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Ràng buộc biểu thức chính quy

Bạn có thể hạn chế định dạng của các tham số route của bạn bằng cách sử dụng phương thức `where` trên một instance route. Phương thức `where` chấp nhận tên của tham số và biểu thức chính quy xác định cách tham số đó bị ràng buộc:

    Route::get('/user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('/user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('/user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

Để thuận tiện, một số pattern biểu thức chính quy thường được sử dụng sẽ có các phương thức helper tương ứng, cho phép bạn nhanh chóng thêm các ràng buộc pattern vào route của bạn:

    Route::get('/user/{id}/{name}', function ($id, $name) {
        //
    })->whereNumber('id')->whereAlpha('name');

    Route::get('/user/{name}', function ($name) {
        //
    })->whereAlphaNumeric('name');

    Route::get('/user/{id}', function ($id) {
        //
    })->whereUuid('id');

Nếu request đến không khớp với các ràng buộc pattern của route, thì response HTTP 404 sẽ được trả về.

<a name="parameters-global-constraints"></a>
#### Ràng buộc Global

Nếu bạn muốn một tham số route luôn bị ràng buộc bởi một biểu thức chính quy định, bạn có thể sử dụng phương thức `pattern`. Bạn có thể định nghĩa các pattern này trong phương thức `boot` của `App\Providers\RouteServiceProvider`:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');
    }

Sau khi pattern đã được định nghĩa xong, nó sẽ tự động được áp dụng cho tất cả các route sử dụng tên tham số đó:

    Route::get('/user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### Encoded Forward Slashes

Component route của Laravel cho phép tất cả các ký tự được đi qua ngoại trừ ký tự `/` có trong các giá trị tham số route. Đối với ký tự `/` bạn phải cho phép nó là một phần thay thế bằng cách sử dụng một biểu thức chính quy điều kiện `where`:

    Route::get('/search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> {note} Encoded forward slashes chỉ hỗ trợ tham số cuối cùng của route.

<a name="named-routes"></a>
## Tên của Route

Các tên của route cho phép tạo các URL hoặc các chuyển hướng đến các route cụ thể. Bạn có thể đặt tên cho một route bằng cách kết hợp phương thức `name` vào định nghĩa route:

    Route::get('/user/profile', function () {
        //
    })->name('profile');

Bạn cũng có thể đặt tên route cho các hành động của controller:

    Route::get(
        '/user/profile',
        [UserProfileController::class, 'show']
    )->name('profile');

> {note} Tên route phải luôn là duy nhất.

<a name="generating-urls-to-named-routes"></a>
#### Tạo URLs từ tên route

Khi bạn đã gán tên cho một route, bạn có thể sử dụng tên của route đó khi tạo URL hoặc chuyển hướng đến URL đó bằng phương thức helper `route` và `redirect` của Laravel:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

Nếu tên route của bạn có định nghĩa tham số, bạn có thể chuyển các tham số đó làm tham số thứ hai trong hàm `route`. Các tham số đó sẽ tự động được chèn vào URL được tạo và ở vị trí chính xác của chúng:

    Route::get('/user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

Nếu bạn truyền thêm các tham số vào mảng, thì các cặp khóa và giá trị của các tham số đó sẽ được tự động thêm vào chuỗi truy vấn của URL đã tạo:

    Route::get('/user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1, 'photos' => 'yes']);

    // /user/1/profile?photos=yes

> {tip} Thỉnh thoảng, bạn có thể muốn chỉ định các giá trị mặc định cho các tham số URL trên toàn bộ request, chẳng hạn như ngôn ngữ hiện tại. Để thực hiện điều này, bạn có thể sử dụng phương thức [`URL::defaults` method](/docs/{{version}}/urls#default-values).

<a name="inspecting-the-current-route"></a>
#### Kiểm tra Route hiện tại

Nếu bạn muốn xác định xem request hiện tại có đúng với một route đã được đặt tên hay không, bạn có thể sử dụng phương thức `named` trên một instance route. Ví dụ: bạn có thể kiểm tra tên route hiện tại từ một middleware route:

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

Nhóm route cho phép bạn chia sẻ các thuộc tính route, chẳng hạn như middleware trên một số lượng lớn các route mà không cần phải định nghĩa các thuộc tính đó trên mỗi route.

Đối với các nhóm lồng nhau thì sẽ thử "merge" các thuộc tính nhóm nhỏ với nhóm to hơn. Middleware và điều kiện `where` sẽ được merge trong khi tên và tiền tố sẽ được thêm vào. Dấu phân cách namespace và dấu gạch chéo trong tiền tố URI cũng sẽ tự động được thêm vào chỗ thích hợp.

<a name="route-group-middleware"></a>
### Middleware

Để gán [middleware](/docs/{{version}}/middleware) cho tất cả các route có trong một nhóm, bạn có thể sử dụng phương thức `middleware` ở trước định nghĩa của nhóm route đó. Middleware sẽ thực hiện theo thứ tự mà nó đã được liệt kê trong mảng:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second middleware...
        });

        Route::get('/user/profile', function () {
            // Uses first & second middleware...
        });
    });

<a name="route-group-controllers"></a>
### Controllers

Nếu một nhóm các route đều sử dụng cùng một [controller](/docs/{{version}}/controllers), bạn có thể sử dụng phương thức `controller` để định nghĩa một controller chung cho tất cả các route trong nhóm. Sau đó, khi định nghĩa xong các route, bạn chỉ cần cung cấp các phương thức mà chúng gọi:

    use App\Http\Controllers\OrderController;

    Route::controller(OrderController::class)->group(function () {
        Route::get('/orders/{id}', 'show');
        Route::post('/orders', 'store');
    });

<a name="route-group-subdomain-routing"></a>
### Routing cho tên miền phụ

Nhóm route cũng có thể được sử dụng để xử lý các route dành riêng cho tên miền phụ. Tên miền phụ có thể được định nghĩa thông qua tham số route giống như URI route, cho phép bạn lấy một phần tên miền phụ để sử dụng trong route hoặc trong controller của bạn. Tên miền phụ có thể được chỉ định bằng cách gọi phương thức `domain` ở trước định nghĩa nhóm route:

    Route::domain('{account}.example.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

> {note} Để đảm bảo có thể truy cập được vào các route tên miền phụ của bạn, bạn nên đăng ký các route tên miền phụ của bạn trước khi đăng ký các route tên miền gốc. Điều này sẽ ngăn các route miền gốc ghi đè vào các route tên miền phụ có cùng đường dẫn URI.

<a name="route-group-prefixes"></a>
### Tiền tố cho Route

Phương thức `prefix` có thể được sử dụng để làm tiền tố cho mỗi route trong nhóm route với một URI. Ví dụ: bạn có thể muốn đặt tiền tố cho tất cả các URI của route trong nhóm với tiền tối `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('/users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### Tiền tố cho tên Route

Phương thức `name` có thể được sử dụng để đặt tiền tố cho mỗi tên của route trong nhóm với một chuỗi. Ví dụ: bạn có thể muốn đặt tiền tố cho tất cả các tên của route trong một nhóm là `admin`. Chuỗi mà đã được đặt làm tiền tố sẽ được gán vào tên của mỗi route, và vì thế chúng ta nên chắc chắn là đã thêm dấu `.` vào trong tiền tố để dễ phân biệt tiền tố và tên route:

    Route::name('admin.')->group(function () {
        Route::get('/users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## Liên kết Route Model

Khi inject một model ID vào một route hoặc một controller action, bạn sẽ thường truy vấn database để lấy ra model tương ứng với ID đó ra. Liên kết route model của Laravel sẽ cung cấp một cách để tự động đưa ra các instance của model vào các route của bạn. Ví dụ, thay vì inject user ID, bạn có thể inject một instance model `User` khớp với ID đã cho.

<a name="implicit-binding"></a>
### Liên kết ngầm

Laravel sẽ tự động resolve các model Eloquent được định nghĩa trong các route hoặc trong controller action nếu tên biến khớp với tên tham số route. Ví dụ:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    });

Vì biến `$user` được khai báo có kiểu là model Eloquent `App\Models\User` và tên biến này cũng khớp với tham số URI `{user}`, nên Laravel sẽ tự động inject một instance model có ID là giá trị tương ứng từ URI request. Nếu không tìm thấy instance model nào phù hợp trong cơ sở dữ liệu, phản hồi HTTP 404 sẽ được đưa tạo.

Tất nhiên, cũng có thể thực hiện liên kết ngầm khi sử dụng các phương thức controller. Một lần nữa, hãy lưu ý rằng tham số URI `{user}` phải khớp với biến `$user` trong controller chứa khai báo kiểu `App\Models\User`:

    use App\Http\Controllers\UserController;
    use App\Models\User;

    // Route definition...
    Route::get('/users/{user}', [UserController::class, 'show']);

    // Controller method definition...
    public function show(User $user)
    {
        return view('user.profile', ['user' => $user]);
    }

<a name="implicit-soft-deleted-models"></a>
#### Soft Deleted Models

Thông thường, liên kết model ngầm sẽ không lấy ra các model đã bị [soft deleted](/docs/{{version}}/eloquent#soft-deleting). Tuy nhiên, bạn có thể hướng dẫn liên kết ngầm để lấy ra các model này bằng cách kết hợp thêm phương thức `withTrashed` vào định nghĩa route của bạn:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    })->withTrashed();

<a name="customizing-the-default-key-name"></a>
#### Customizing The Key

Thỉnh thoảng bạn có thể muốn resolve các model Eloquent bằng cách sử dụng một cột khác ngoài cột `id`. Để làm như vậy, bạn có thể chỉ định cột đó trong định nghĩa tham số route:

    use App\Models\Post;

    Route::get('/posts/{post:slug}', function (Post $post) {
        return $post;
    });

Nếu bạn muốn tuỳ biến một liên kết của một model luôn sử dụng một cột khác, khác với cột `id` trong cơ sở dữ liệu, bạn có thể ghi đè phương thức `getRouteKeyName` trong model Eloquent:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="implicit-model-binding-scoping"></a>
#### Custom Keys & Scoping

Khi liên kết ngầm nhiều model Eloquent trong một định nghĩa route duy nhất, bạn có thể muốn xác định phạm vi của model Eloquent thứ hai sao cho nó phải là con của model Eloquent trước đó. Ví dụ: hãy xem xét định nghĩa route sau để lấy ra một bài đăng blog bằng slug của một người dùng cụ thể:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

Khi sử dụng liên kết ngầm có khóa tùy chỉnh làm tham số route lồng nhau, Laravel sẽ tự động xác định phạm vi truy vấn để lấy ra model lồng nhau của cha mẹ bằng cách sử dụng các quy ước để đoán tên quan hệ trên cha mẹ. Trong trường hợp này, giả định rằng model `User` có quan hệ tên là `posts` (dạng số nhiều của tên tham số route) có thể được sử dụng để lấy ra model `Post`.

Nếu muốn, bạn có thể hướng dẫn Laravel xác định phạm vi liên kết "con" ngay cả khi khóa tùy chỉnh không được cung cấp. Để làm như vậy, bạn có thể gọi phương thức `scopeBindings` khi định nghĩa route của bạn:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    })->scopeBindings();

Hoặc, bạn có thể hướng dẫn nhóm route sử dụng các liên kết có phạm vi:

    Route::scopeBindings()->group(function () {
        Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
            return $post;
        });
    });

<a name="customizing-missing-model-behavior"></a>
#### Customizing Missing Model Behavior

Thông thường, response HTTP 404 sẽ được tạo nếu không tìm thấy model liên kết ngầm. Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách gọi phương thức `missing` khi định nghĩa route của bạn. Phương thức `missing` sẽ chấp nhận một closure sẽ được gọi nếu không thể tìm thấy model liên kết ngầm:

    use App\Http\Controllers\LocationsController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
            ->name('locations.view')
            ->missing(function (Request $request) {
                return Redirect::route('locations.index');
            });

<a name="explicit-binding"></a>
### Liên kết rõ ràng

Bạn không nhất thiết phải sử dụng liên kết ngầm của laravel, cái mà dựa vào quy ước đặt tên. Bạn cũng có thể định nghĩa rõ ràng cách mà các tham số route tương ứng với các model. Để đăng ký một liên kết rõ ràng, hãy sử dụng phương thức `model` trong router để định nghĩa một class với một tham số đã cho. Bạn nên định nghĩa các liên kết model rõ ràng của bạn ở đầu phương thức `boot` của class` RouteServiceProvider` của bạn:

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::model('user', User::class);

        // ...
    }

Tiếp theo, hãy định nghĩa một route chứa tham số `{user}`:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        //
    });

Vì chúng ta đã liên kết các tham số `{user}` vào trong model `App\Models\User`, nên một instance của class đó sẽ được tự động inject vào trong route của bạn. Vì vậy, ví dụ, nếu một request với uri là `users/1` thì sẽ tự động inject instance `User` có ID là `1` từ cơ sở dữ liệu.

Nếu không tìm thấy model instance phù hợp trong cơ sở dữ liệu, phản hồi HTTP 404 sẽ được đưa ra.

<a name="customizing-the-resolution-logic"></a>
#### Tuỳ chỉnh logic phụ thuộc

Nếu bạn muốn định nghĩa một tuỳ chỉnh logic cho liên kết model của bạn, bạn có thể sử dụng phương thức `Route::bind`. Closure của bạn sẽ được truyền đến phương thức `bind` và nhận vào giá trị của tham số URI, sau đó sẽ trả về một instance của class, và sẽ được inject vào trong route trước đó. Một lần nữa, việc tùy chỉnh này sẽ diễn ra trong phương thức `boot` của `RouteServiceProvider` trong ứng dụng của bạn:

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::bind('user', function ($value) {
            return User::where('name', $value)->firstOrFail();
        });

        // ...
    }

Ngoài ra, bạn có thể ghi đè phương thức `resolveRouteBinding` trên model Eloquent của bạn. Phương thức này sẽ nhận vào giá trị phân đoạn của tham số URI và sẽ trả về một instance của class sẽ được đưa vào route:

    /**
     * Retrieve the model for a bound value.
     *
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveRouteBinding($value, $field = null)
    {
        return $this->where('name', $value)->firstOrFail();
    }

Nếu một route đang sử dụng [phạm vi liên kết ngầm](#implicit-model-binding-scoping), phương thức `resolveChildRouteBinding` sẽ được sử dụng để resolve liên kết con của model cha:

    /**
     * Retrieve the child model for a bound value.
     *
     * @param  string  $childType
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveChildRouteBinding($childType, $value, $field)
    {
        return parent::resolveChildRouteBinding($childType, $value, $field);
    }

<a name="fallback-routes"></a>
## Route dự phòng

Sử dụng phương thức `Route::fallback`, bạn có thể định nghĩa một route sẽ được thực thi khi không có một route nào khác phù hợp với request đến. Thông thường, các request chưa được xử lý sẽ tự động hiển thị trang "404" thông qua trình xử lý exception của ứng dụng của bạn. Tuy nhiên, vì bạn hay định nghĩa route `fallback` trong file `routes/web.php` của bạn, nên tất cả midddleware trong nhóm midddleware `web` sẽ được áp dụng cho route này. Tất nhiên, bạn có thể thoải mái thêm midddleware vào trong route này nếu cần:

    Route::fallback(function () {
        //
    });

> {note} Route dự phòng phải luôn là route cuối cùng được đăng ký bởi application của bạn.

<a name="rate-limiting"></a>
## Rate Limiting

<a name="defining-rate-limiters"></a>
### Định nghĩa giới hạn tỷ lệ

Laravel có chứa các service giới hạn tỷ lệ mạnh mẽ và có thể tùy chỉnh mà bạn có thể sử dụng để hạn chế lưu lượng truy cập cho một route hoặc một nhóm route nhất định. Để bắt đầu, bạn nên định nghĩa cấu hình giới hạn tỷ lệ đáp ứng nhu cầu của ứng dụng. Thông thường, việc này phải được thực hiện trong phương thức `configureRateLimiting` của class `App\Providers\RouteServiceProvider` trong ứng dụng của bạn.

Giới hạn tỷ lệ được định nghĩa bằng phương thức `for` của facade `RateLimiter`. Phương thức `for` chấp nhận tên giới hạn tỷ lệ và một closure trả về cấu hình giới hạn sẽ được áp dụng cho các route mà được gán cho giới hạn tỷ lệ đó. Cấu hình giới hạn là các instance của class `Illuminate\Cache\RateLimiting\Limit`. Class này chứa các phương thức "builder" hữu ích để bạn có thể nhanh chóng định nghĩa giới hạn của bạn. Tên giới hạn tỷ lệ có thể là bất kỳ chuỗi nào bạn muốn:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Configure the rate limiters for the application.
     *
     * @return void
     */
    protected function configureRateLimiting()
    {
        RateLimiter::for('global', function (Request $request) {
            return Limit::perMinute(1000);
        });
    }

Nếu request gửi đến vượt quá giới hạn tỷ lệ đã chỉ định, Laravel sẽ tự động trả về response có mã trạng thái HTTP 429. Nếu bạn muốn định nghĩa một response khác của riêng bạn sẽ được trả về theo giới hạn tỷ lệ, bạn có thể sử dụng phương thức `response`:

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000)->response(function () {
            return response('Custom response...', 429);
        });
    });

Vì lệnh callback của giới hạn tỷ lệ sẽ nhận vào một instance request HTTP nên bạn có thể tự động build giới hạn tỷ lệ phù hợp dựa trên request hoặc số lượng người dùng được xác thực:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

<a name="segmenting-rate-limits"></a>
#### Segmenting Rate Limits

Thỉnh thoảng bạn có thể muốn phân giới hạn tỷ lệ theo một số giá trị tùy ý. Ví dụ: bạn có thể muốn cho phép người dùng truy cập vào một route nhất định 100 lần mỗi phút cho mỗi địa chỉ IP. Để thực hiện điều này, bạn có thể sử dụng phương thức `by` khi xây dựng giới hạn tỷ lệ của bạn:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

Hãy minh họa tính năng này bằng một ví dụ khác, chúng ta có thể giới hạn quyền truy cập vào route ở mức 100 lần mỗi phút cho mỗi ID người dùng được xác thực hoặc 10 lần mỗi phút cho mỗi địa chỉ IP đối với người chưa đăng nhập:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()
                    ? Limit::perMinute(100)->by($request->user()->id)
                    : Limit::perMinute(10)->by($request->ip());
    });

<a name="multiple-rate-limits"></a>
#### Multiple Rate Limits

Nếu cần, bạn có thể trả về một mảng giới hạn tỷ lệ cho một cấu hình giới hạn tỷ lệ nhất định. Mỗi giới hạn tỷ lệ sẽ được đánh giá cho route dựa trên thứ tự chúng được set trong mảng:

    RateLimiter::for('login', function (Request $request) {
        return [
            Limit::perMinute(500),
            Limit::perMinute(3)->by($request->input('email')),
        ];
    });

<a name="attaching-rate-limiters-to-routes"></a>
### Gán giới hạn tỷ lệ vào route

Giới hạn tỷ lệ có thể được gắn vào các route hoặc một nhóm route bằng cách sử dụng `throttle` [middleware](/docs/{{version}}/middleware). Middleware throttle sẽ chấp nhận tên của giới hạn tỷ lệ mà bạn muốn gán cho route:

    Route::middleware(['throttle:uploads'])->group(function () {
        Route::post('/audio', function () {
            //
        });

        Route::post('/video', function () {
            //
        });
    });

<a name="throttling-with-redis"></a>
#### Throttling With Redis

Thông thường, middleware `throttle` được ánh xạ tới class `Illuminate\Routing\Middleware\ThrottleRequests`. Ánh xạ này được định nghĩa trong file HTTP kernelHTTP kernel của ứng dụng của bạn (`App\Http\Kernel`). Tuy nhiên, nếu bạn đang sử dụng Redis làm driver cache của ứng dụng, bạn có thể muốn thay đổi ánh xạ này để sử dụng class `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis`. Class này hiệu quả hơn trong việc quản lý giới hạn tỷ lệ bằng Redis:

    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,

<a name="form-method-spoofing"></a>
## Form Method giả

Các HTML form không hỗ trợ các action `PUT`, `PATCH`, hoặc `DELETE`. Vì vậy, khi định nghĩa các route `PUT`, `PATCH`, hoặc `DELETE` được gửi từ HTML form, bạn sẽ cần phải thêm một input hidden `_method` vào form. Giá trị được gửi vào input hidden `_method` sẽ được sử dụng làm method request HTTP:

    <form action="/example" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Để thuận tiện, bạn có thể dùng lệnh `@method` của [Blade](/docs/{{version}}/blade) để tạo ra field input `_method`:

    <form action="/example" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Truy cập vào Route hiện tại

Bạn có thể dùng các phương thức `current`, `currentRouteName`, và `currentRouteAction` trong facade `Route` để truy cập vào các thông tin của route hiện tại đang được xử lý cho một request:

    use Illuminate\Support\Facades\Route;

    $route = Route::current(); // Illuminate\Routing\Route
    $name = Route::currentRouteName(); // string
    $action = Route::currentRouteAction(); // string

Bạn có thể tham khảo tài liệu API cho [class facade Route](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) và cả [instance Route](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) để biết tất cả các phương thức có sẵn trên facade route và các class route.

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

Laravel có thể tự động respond các CORS `OPTIONS` HTTP request với các giá trị mà bạn đã cấu hình. Tất cả các cài đặt CORS có thể được cấu hình trong file cấu hình `config/cors.php` của application. Mặc định, các `OPTIONS` request sẽ được tự động xử lý bởi [middleware](/docs/{{version}}/middleware) `HandleCors` nằm theo trong stack global middleware của bạn. Stack global middleware của bạn nằm trong file HTTP kernel của ứng dụng (`App\Http\Kernel`).

> {tip} Để biết thêm thông tin về CORS và header CORS, vui lòng tham khảo [tài liệu web MDN về CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

<a name="route-caching"></a>
## Route Caching

Khi deploy ứng dụng của bạn vào production, bạn nên tận dụng cache route của Laravel. Việc sử dụng cache route sẽ giảm đáng kể thời gian cần thiết để đăng ký tất cả các route vào trong ứng dụng của bạn. Để tạo cache route, hãy chạy lệnh Artisan `route:cache`:

    php artisan route:cache

Sau khi chạy lệnh này, file cache route của bạn sẽ được load theo mọi request. Hãy nhớ rằng, nếu bạn thêm bất kỳ route mới nào vào, thì bạn sẽ cần tạo lại một cache route mới. Vì lý do này, bạn chỉ nên chạy lệnh `route:cache` trong quá trình deploy dự án của bạn.

Bạn có thể sử dụng lệnh `route:clear` để xóa cache route:

    php artisan route:clear
