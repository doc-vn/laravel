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
    - [Subdomain Routing](#route-group-subdomain-routing)
    - [Tiền tố cho Route](#route-group-prefixes)
    - [Tiền tố cho tên Route](#route-group-name-prefixes)
- [Liên kết Route Model](#route-model-binding)
    - [Liên kết ngầm](#implicit-binding)
    - [Liên kết rõ ràng](#explicit-binding)
- [Route dự phòng](#fallback-routes)
- [Rate Limiting](#rate-limiting)
- [Form Method giả](#form-method-spoofing)
- [Truy cập vào Route hiện tại](#accessing-the-current-route)
- [Cross-Origin Resource Sharing (CORS)](#cors)

<a name="basic-routing"></a>
## Routing cơ bản

Các route cơ bản của Laravel chấp nhận một URI và một `Closure`, cung cấp một phương thức định nghĩa route rất đơn giản và dễ hiểu:

    Route::get('foo', function () {
        return 'Hello World';
    });

#### Các file Route mặc định

Tất cả các route của Laravel được định nghĩa trong các file route, và được lưu trong thư mục `routes`. Các file này được tự động load bởi framework. File `routes/web.php` định nghĩa các route dành cho giao diện web của bạn. Các route này sẽ được gán với nhóm middleware `web`, cung cấp các tính năng như trạng thái session và bảo vệ CSRF. Các route trong `routes/api.php` là các route không có trạng thái và được gán với nhóm middleware `api`.

Đối với hầu hết các application, bạn sẽ bắt đầu bằng cách định nghĩa các route trong file `routes/web.php`. Các route đã được tạo trong file `routes/web.php` có thể được truy cập bằng cách nhập URL của route đó vào trong trình duyệt web của bạn. Ví dụ: bạn có thể truy cập vào route sau bằng cách nhập url là `http://your-app.test/user` trong trình duyệt web của bạn:

    Route::get('/user', 'UserController@index');

Các route được định nghĩa trong file `routes/api.php` sẽ nằm trong một nhóm route bởi `RouteServiceProvider`. Trong nhóm này, tiền tố URI `/api` sẽ được tự động áp dụng, do đó bạn không cần phải tự làm cho mỗi route có trong file. Bạn có thể sửa đổi tiền tố và các tùy chọn khác cho nhóm route này bằng cách sửa đổi trong class `RouteServiceProvider` của bạn.

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

#### Bảo vệ CSRF

Bất kỳ form HTML nào mà trỏ đến các route `POST`, `PUT`, `PATCH` hoặc `DELETE` được định nghĩa trong file route `web`, đều phải chứa một field CSRF token. Nếu không có field đó, request sẽ bị từ chối. Bạn có thể đọc thêm về bảo vệ CSRF trong [tài liệu CSRF](/docs/{{version}}/csrf):

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

Bạn có thể sử dụng phương thức `Route::permanentRedirect` để trả về status code là `301`:

    Route::permanentRedirect('/here', '/there');

<a name="view-routes"></a>
### View Routes

Nếu route của bạn chỉ cần trả về một view, thì bạn có thể sử dụng phương thức `Route::view`. Giống như phương thức `redirect`, phương thức này cung cấp một lối tắt đơn giản để bạn không phải định nghĩa một route hoặc một controller đầy đủ. Phương thức `view` chấp nhận URI làm tham số đầu tiên và tên view sẽ làm tham số thứ hai. Ngoài ra, bạn có thể cung cấp một mảng data để chuyển đến view dưới dạng là tùy chọn tham số thứ ba:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## Route Parameters

<a name="required-parameters"></a>
### Tham số bắt buộc

Đôi khi bạn sẽ cần phải lấy các tham số của URI trong route của bạn. Ví dụ: bạn có thể cần lấy ID người dùng từ URL. Bạn có thể làm như vậy bằng cách định nghĩa các route parameter:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

Bạn có thể định nghĩa nhiều route parameter bắt buộc trong route của bạn:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Các tham số route luôn nằm trong các dấu ngoặc `{}` và phải chứa các ký tự chữ cái và không được chứa ký tự `-`. Thay vì sử dụng ký tự `-`, hãy sử dụng dấu gạch dưới (`_`). Các tham số route sẽ được inject vào các route callback hoặc controller dựa thoe thứ tự của chúng - tên của các tham số callback hoặc controller không quan trọng.

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

Bạn có thể hạn chế định dạng của các tham số route của bạn bằng cách sử dụng phương thức `where` trên một instance route. Phương thức `where` chấp nhận tên của tham số và biểu thức chính quy xác định cách tham số đó bị ràng buộc:

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

Nếu bạn muốn một tham số route luôn bị ràng buộc bởi một biểu thức chính quy định, bạn có thể sử dụng phương thức `pattern`. Bạn có thể định nghĩa các pattern này trong phương thức `boot` của `RouteServiceProvider`:

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

Sau khi pattern đã được định nghĩa xong, nó sẽ tự động được áp dụng cho tất cả các route sử dụng tên tham số đó:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### Encoded Forward Slashes

Component component của Laravel cho phép tất cả các ký tự được thông qua ngoại trừ ký tự `/`. Đối với ký tự `/` bạn phải cho phép nó là một phần thay thế bằng cách sử dụng một biểu thức chính quy điều kiện `where`:

    Route::get('search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> {note} Encoded forward slashes chỉ hỗ trợ tham số cuối cùng của route.

<a name="named-routes"></a>
## Tên của Route

Các tên của route cho phép tạo các URL hoặc các chuyển hướng đến các route cụ thể. Bạn có thể đặt tên cho một route bằng cách kết hợp phương thức `name` vào định nghĩa route:

    Route::get('user/profile', function () {
        //
    })->name('profile');

Bạn cũng có thể đặt tên route cho các hành động của controller:

    Route::get('user/profile', 'UserProfileController@show')->name('profile');

> {note} Tên route phải luôn là duy nhất.

#### Tạo URLs từ tên route

Khi bạn đã gán tên cho một route, bạn có thể sử dụng tên của route đó khi tạo URL hoặc chuyển hướng qua hàm `route` global:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

Nếu tên route của bạn có định nghĩa tham số, bạn có thể chuyển các tham số đó làm tham số thứ hai trong hàm `route`. Các tham số đó sẽ tự động được chèn vào URL và ở vị trí chính xác của chúng:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

Nếu bạn truyền thêm các tham số vào mảng, thì các cặp khóa và giá trị của các tham số đó sẽ được tự động thêm vào chuỗi truy vấn của URL đã tạo:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1, 'photos' => 'yes']);

    // /user/1/profile?photos=yes

> {tip} Thỉnh thoảng, bạn có thể muốn chỉ định các giá trị mặc định cho các tham số URL trên toàn bộ request, chẳng hạn như ngôn ngữ hiện tại. Để thực hiện điều này, bạn có thể sử dụng phương thức [`URL::defaults` method](/docs/{{version}}/urls#default-values).

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

Nhóm route cho phép bạn chia sẻ các thuộc tính route, chẳng hạn như middleware hoặc namespaces trên một số lượng lớn các route mà không cần phải định nghĩa các thuộc tính đó trên mỗi route. Các thuộc tính được chia sẻ sẽ được chỉ định trong một mảng là tham số đầu tiên của phương thức `Route::group`.

Đối với các nhóm lồng nhau thì sẽ thử "merge" các thuộc tính nhóm nhỏ với nhóm to hơn. Middleware và điều kiện `where` sẽ được merge trong khi tên, namespace và tiền tố sẽ được thêm vào. Dấu phân cách namespace và dấu gạch chéo trong tiền tố URI cũng sẽ tự động được thêm vào chỗ thích hợp.

<a name="route-group-middleware"></a>
### Middleware

Để gán middleware cho tất cả các route có trong một nhóm, bạn có thể sử dụng phương thức `middleware` ở trước định nghĩa của nhóm route đó. Middleware sẽ thực hiện theo thứ tự mà nó đã được liệt kê trong mảng:

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

Một trường hợp sử dụng phổ biến khác cho các nhóm route là gán vào cùng một namespace PHP cho một nhóm các controller bằng cách sử dụng phương thức `namespace`:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Nhớ rằng, mặc định, `RouteServiceProvider` đã khai báo tất cả các file route của bạn vào trong một nhóm namespace từ trước, cho phép bạn đăng ký các controller vào các route mà không cần chỉ định namespace của nó là `App\Http\Controllers`. Vì vậy, bạn chỉ cần định nghĩa phần namespace xuất hiện sau phần namespace `App\Http\Controllers`.

<a name="route-group-subdomain-routing"></a>
### Routing cho tên miền phụ

Nhóm route cũng có thể được sử dụng để xử lý các route dành riêng cho tên miền phụ. Tên miền phụ có thể được định nghĩa thông qua tham số route giống như URI route, cho phép bạn lấy một phần tên miền phụ để sử dụng trong route hoặc trong controller của bạn. Tên miền phụ có thể được chỉ định bằng cách gọi phương thức `domain` ở trước định nghĩa nhóm route:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

> {note} Để đảm bảo có thể truy cập được vào các route tên miền phụ của bạn, bạn nên đăng ký các route tên miền phụ của bạn trước khi đăng ký các route tên miền gốc. Điều này sẽ ngăn các route miền gốc ghi đè vào các route tên miền phụ có cùng đường dẫn URI.

<a name="route-group-prefixes"></a>
### Tiền tố cho Route

Phương thức `prefix` có thể được sử dụng để làm tiền tố cho mỗi route trong nhóm route với một URI. Ví dụ: bạn có thể muốn đặt tiền tố cho tất cả các URI của route trong nhóm với tiền tối `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### Tiền tố cho tên Route

Phương thức `name` có thể được sử dụng để đặt tiền tố cho mỗi tên của route trong nhóm với một chuỗi. Ví dụ: bạn có thể muốn đặt tiền tố cho tất cả các tên của route trong một nhóm là `admin`. Chuỗi mà đã được đặt làm tiền tố sẽ được gán vào tên của mỗi route, và vì thế chúng ta nên chắc chắn là đã thêm dấu `.` vào trong tiền tố để dễ phân biệt tiền tố và tên route:

    Route::name('admin.')->group(function () {
        Route::get('users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## Liên kết Route Model

Khi inject một model ID vào một route hoặc một controller action, bạn sẽ thường truy vấn để lấy ra model tương ứng với ID đó ra. Liên kết route model của Laravel sẽ cung cấp một cách để tự động đưa ra các instance của model vào các route của bạn. Ví dụ, thay vì inject user ID, bạn có thể inject một instance model `User` khớp với ID đã cho.

<a name="implicit-binding"></a>
### Liên kết ngầm

Laravel sẽ tự động resolve các model Eloquent được định nghĩa trong các route hoặc trong controller action nếu tên biến khớp với tên tham số route. Ví dụ:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Vì biến `$user` được khai báo có kiểu là model Eloquent `App\User` và tên biến này cũng khớp với tham số URI `{user}`, nên Laravel sẽ tự động inject một instance model có ID là giá trị tương ứng từ URI request. Nếu không tìm thấy instance model nào phù hợp trong cơ sở dữ liệu, phản hồi HTTP 404 sẽ được đưa tạo.

#### Tuỳ chỉnh Key

Thỉnh thoảng bạn có thể muốn resolve các model Eloquent bằng cách sử dụng một cột khác, khác với cột `id`. Để làm như vậy, bạn có thể chỉ định một cột khác trong định nghĩa tham route:

    Route::get('api/posts/{post:slug}', function (App\Post $post) {
        return $post;
    });

#### Custom Keys & Scoping

Thỉnh thoảng, khi liên kết ngầm nhiều model Eloquent trong một định nghĩa route, bạn có thể muốn scope model Eloquent thứ hai sao cho nó phải là con của model Eloquent thứ nhất. Ví dụ: hãy xem tình huống sau lấy ra một bài đăng trong blog bằng slug cho một user cụ thể:

    use App\Post;
    use App\User;

    Route::get('api/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

Khi sử dụng liên kết ngầm có key tùy biến làm một tham số route lồng nhau, Laravel sẽ tự động scope truy vấn để lấy ra các model lồng nhau thông qua cha của nó bằng cách sử dụng các quy ước để đặt tên quan hệ trên cha. Trong trường hợp này, sẽ giả định rằng model `User` có một quan hệ có tên là `posts` (số nhiều của tên tham số route) có thể được sử dụng để lấy ra model `Post`.

#### Customizing The Default Key Name

Nếu bạn muốn tuỳ biến một liên kết của một model mà sử dụng một cột khác, khác với cột `id` trong cơ sở dữ liệu, bạn có thể ghi đè phương thức `getRouteKeyName` trong model Eloquent:

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

Để đăng ký một liên kết rõ ràng, hãy sử dụng phương thức `model` trong router để định nghĩa một class với một tham số đã cho. Bạn nên định nghĩa các liên kết model rõ ràng của bạn trong phương thức `boot` của class` RouteServiceProvider`:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Tiếp theo, hãy định nghĩa một route chứa tham số `{user}`:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

Vì chúng ta đã liên kết các tham số `{user}` vào trong model `App\User`, nên một instance `User` sẽ được tự động inject vào trong route của bạn. Vì vậy, ví dụ, nếu một request với uri là `profile/1` thì sẽ tự động inject instance `User` có ID là `1` từ cơ sở dữ liệu.

Nếu không tìm thấy model instance phù hợp trong cơ sở dữ liệu, phản hồi HTTP 404 sẽ được đưa ra.

#### Tuỳ chỉnh logic phụ thuộc

Nếu bạn muốn sử dụng tuỳ chỉnh logic phụ thuộc của bạn, bạn có thể sử dụng phương thức `Route::bind`. `Closure` của bạn sẽ được truyền đến phương thức `bind` và nhận vào giá trị của tham số URI, sau đó sẽ trả về một instance của class, và sẽ được inject vào trong route trước đó:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->firstOrFail();
        });
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

<a name="fallback-routes"></a>
## Route dự phòng

Sử dụng phương thức `Route::fallback`, bạn có thể định nghĩa một route sẽ được thực thi khi không có một route nào khác phù hợp với request đến. Thông thường, các request chưa được xử lý sẽ tự động hiển thị trang "404" thông qua trình xử lý exception của ứng dụng của bạn. Tuy nhiên, vì bạn có thể định nghĩa route dự phòng trong file `routes/web.php` của bạn, nên tất cả midddleware trong nhóm midddleware `web` sẽ được áp dụng cho route này. Tất nhiên, bạn có thể thoải mái thêm midddleware vào trong route này nếu cần:

    Route::fallback(function () {
        //
    });

> {note} Route dự phòng phải luôn là route cuối cùng được đăng ký bởi application của bạn.

<a name="rate-limiting"></a>
## Rate Limiting

Laravel có chứa một [middleware](/docs/{{version}}/middleware) để giới hạn số lượt truy cập vào một route trong ứng dụng của bạn. Để bắt đầu, hãy gán middleware `throttle` cho một route hoặc một nhóm các route. Middleware `throttle` sẽ chấp nhận hai tham số xác định số lượng request tối đa có thể được thực hiện trong một số phút nhất định. Ví dụ: hãy thử định nghĩa là người dùng hiện tại có thể truy cập vào một nhóm route 60 lần cho một phút:

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Dynamic Rate Limiting

Bạn có thể linh hoạt chỉ định số lượng request tối đa dựa vào một thuộc tính của model `User`. Ví dụ: nếu model `User` của bạn chứa thuộc tính `rate_limit`, bạn có thể truyền tên của thuộc tính đó vào middleware `throttle` để nó sử dụng để tính toán số lượng request tối đa:

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Distinct Guest & Authenticated User Rate Limits

Bạn có thể chỉ định các giới hạn request cho guest và người dùng đã xác thực. Ví dụ: bạn có thể chỉ định tối đa `10` request mỗi phút cho guest và `60` request cho người dùng đã xác thực:

    Route::middleware('throttle:10|60,1')->group(function () {
        //
    });

Bạn cũng có thể kết hợp chức năng này với các giới hạn request động. Ví dụ: nếu model `User` của bạn có chứa thuộc tính `rate_limit`, bạn có thể truyền tên của thuộc tính này cho middleware `throttle` để nó được sử dụng và tính toán số lượng request tối đa mà người dùng đã xác thực có thể thực hiện:

    Route::middleware('auth:api', 'throttle:10|rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Rate Limit Segments

Thông thường, bạn có thể sẽ chỉ định một giới hạn request cho toàn bộ API của bạn. Tuy nhiên, ứng dụng của bạn có thể yêu cầu các giới hạn request khác nhau cho các phân khúc khác nhau của API. Nếu đúng như vậy, bạn sẽ cần phải truyền tên của mỗi phân khúc làm tham số thứ ba của middleware `throttle`:

    Route::middleware('auth:api')->group(function () {
        Route::middleware('throttle:60,1,default')->group(function () {
            Route::get('/servers', function () {
                //
            });
        });

        Route::middleware('throttle:60,1,deletes')->group(function () {
            Route::delete('/servers/{id}', function () {
                //
            });
        });
    });

<a name="form-method-spoofing"></a>
## Form Method giả

Các HTML form không hỗ trợ các action `PUT`, `PATCH` hoặc `DELETE`. Vì vậy, khi định nghĩa các route `PUT`, `PATCH` hoặc `DELETE` được gửi từ HTML form, bạn sẽ cần phải thêm một input hidden `_method` vào form. Giá trị được gửi vào input hidden `_method` sẽ được sử dụng làm method request HTTP:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Bạn có thể dùng lệnh `@method` của Blade để tạo ra input `_method`:

    <form action="/foo/bar" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Truy cập vào Route hiện tại

Bạn có thể dùng các phương thức `current`, `currentRouteName`, và `currentRouteAction` trong facade `Route` để truy cập vào các thông tin của route hiện tại đang được xử lý cho một request:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Tham khảo tài liệu API cho [class facade Route](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) và cả [instance Route](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) để biết thêm các phương thức khác có thể truy cập được.

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

Laravel có thể tự động respond các CORS OPTIONS request với các giá trị mà bạn đã cấu hình. Tất cả các cài đặt CORS có thể được cấu hình trong file cấu hình `cors` của bạn và mặc định, các OPTIONS request sẽ được tự động xử lý bởi middleware `HandleCors` nằm theo trong stack global middleware của bạn.

> {tip} Để biết thêm thông tin về CORS và header CORS, vui lòng tham khảo [tài liệu web MDN về CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).
