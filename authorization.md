# Authorization

- [Giới thiệu](#introduction)
- [Gates](#gates)
    - [Viết Gates](#writing-gates)
    - [Authorizing Actions](#authorizing-actions-via-gates)
- [Tạo Policies](#creating-policies)
    - [Tạo Policies](#generating-policies)
    - [Đăng ký Policies](#registering-policies)
- [Viết Policies](#writing-policies)
    - [Các phương thức trong Policy](#policy-methods)
    - [Các phương thức không dùng Models](#methods-without-models)
    - [Policy Filters](#policy-filters)
- [Authorizing Actions dùng Policies](#authorizing-actions-using-policies)
    - [Thông qua User Model](#via-the-user-model)
    - [Thông qua Middleware](#via-middleware)
    - [Thông qua Controller Helpers](#via-controller-helpers)
    - [Thông qua Blade Templates](#via-blade-templates)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc mặc định cung cấp các dịch vụ [authentication](/docs/{{version}}/authentication), Laravel cũng cung cấp một cách đơn giản để authorize cho các hành động của người dùng đối với resource đã cho. Giống như authentication, cách tiếp cận authorize của Laravel rất đơn giản và có hai cách chính để authorize hành động: gates và policies.

Hãy nghĩ về các gates và policies như các routes và controllers. Gates cung cấp một cách tiếp cận đơn giản dựa trên Closure để authorization trong khi các policies, như controllers, nhóm logic của chúng sẽ liên quan đến một model hoặc resource cụ thể. Chúng ta sẽ khám phá các gates đầu tiên và sau đó kiểm tra các policies.

Bạn không cần phải chọn giữa sử dụng gates hoặc sử dụng policies khi xây dựng application. Hầu hết các application rất có thể sẽ chứa hỗn hợp các gates và policies, và điều đó là hoàn toàn tốt! Gates áp dụng cho các hành động không liên quan đến bất kỳ model hoặc resource nào, chẳng hạn như xem bảng điều khiển của quản trị viên. Ngược lại, các policies nên được sử dụng khi bạn muốn authorize một hành động cho một model hoặc resource cụ thể.

<a name="gates"></a>
## Gates

<a name="writing-gates"></a>
### Viết Gates

Gates là các Closure để xác định xem người dùng có được phép thực hiện một hành động nhất định hay không và thường được định nghĩa trong class `App\Providers\AuthServiceProvider` bằng cách sử dụng facade `Gate`. Gates luôn nhận về một instance user làm tham số đầu tiên của nó và có thể tùy chọn nhận thêm các tham số như Eloquent model có liên quan:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

Gates cũng có thể được định nghĩa dưới dạng chuỗi callback `Class@method`, giống controller:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', 'App\Policies\PostPolicy@update');
    }

#### Resource Gates

Bạn cũng có thể định nghĩa nhiều ability trong Gate cùng một lúc bằng phương thức `resource`:

    Gate::resource('posts', 'App\Policies\PostPolicy');

Điều này sẽ giống hệt với việc tự định nghĩa như các định nghĩa Gate sau:

    Gate::define('posts.view', 'App\Policies\PostPolicy@view');
    Gate::define('posts.create', 'App\Policies\PostPolicy@create');
    Gate::define('posts.update', 'App\Policies\PostPolicy@update');
    Gate::define('posts.delete', 'App\Policies\PostPolicy@delete');

Mặc định, các ability `view`, `create`, `update`, và `delete` sẽ được định nghĩa. Bạn có thể ghi đè hoặc thêm vào các ability mặc định bằng cách pass một mảng làm tham số thứ ba cho phương thức `resource`. Các key của mảng định nghĩa tên của các ability trong khi các giá trị định nghĩa tên phương thức. Ví dụ: đoạn mã sau sẽ tạo ra hai định nghĩa Gate mới - `posts.image` và `posts.photo`:

    Gate::resource('posts', 'PostPolicy', [
        'image' => 'updateImage',
        'photo' => 'updatePhoto',
    ]);

<a name="authorizing-actions-via-gates"></a>
### Authorizing Actions

Để authorize cho một hành động bằng cách sử dụng các gate, bạn nên sử dụng các phương thức `allows` hoặc `denies`. Lưu ý rằng bạn không bắt buộc phải pass user hiện đang được xác thực cho các phương thức này. Laravel sẽ tự động pass user vào gate Closure:

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }

    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }

Nếu bạn muốn xác định xem một người dùng có được phép thực hiện một hành động hay không, bạn có thể sử dụng phương thức `forUser` trên facade `Gate`:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }

<a name="creating-policies"></a>
## Tạo Policies

<a name="generating-policies"></a>
### Tạo Policies

Các Policy là các class tổng hợp logic authorization liên quan đến một model hoặc resource cụ thể. Ví dụ: nếu application của bạn là blog, bạn có thể có model `Post` và `PostPolicy` tương ứng để authorization cho các hành động của người dùng như tạo hoặc cập nhật bài đăng.

Bạn có thể tạo một policy bằng cách sử dụng [lệnh artisan](/docs/{{version}}/artisan) `make:policy`. Policy được tạo sẽ được đặt trong thư mục `app/Policies`. Nếu thư mục này không tồn tại trong application của bạn, Laravel sẽ tạo nó cho bạn:

    php artisan make:policy PostPolicy

Lệnh `make:policy` sẽ tạo ra một class policy trống. Nếu bạn muốn tạo một class với các phương thức policy "CRUD" cơ bản đã được tạo sẵn trong class, bạn có thể chỉ định một `--model` khi thực hiện lệnh:

    php artisan make:policy PostPolicy --model=Post

> {tip} Tất cả các policy được resolve thông qua Laravel [service container](/docs/{{version}}/container), cho phép bạn khai báo dạng kiểu cho bất kỳ phụ thuộc cần thiết nào trong hàm tạo của policy để chúng được tự động injecte.

<a name="registering-policies"></a>
### Đăng ký Policies

Một khi policy đã tồn tại, nó cần phải được đăng ký. `AuthServiceProvider` đi kèm với các application Laravel mới chứa thuộc tính `policies` dùng để ánh xạ các Eloquent model của bạn tới các policy tương ứng của chúng. Đăng ký policy sẽ hướng dẫn cho Laravel sử dụng policy nào khi authorizing cho các hành động đối với một model nhất định:

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## Viết Policies

<a name="policy-methods"></a>
### Các phương thức trong Policy

Khi policy đã được đăng ký, bạn có thể thêm các phương thức cho từng hành động được authorize. Ví dụ: hãy định nghĩa một phương thức `update` trên `PostPolicy` để xác định xem một `User` đã cho có thể cập nhật một instance `Post` đã cho hay không.

Phương thức `update` sẽ nhận về một instance `User` và một `Post` làm tham số của nó và sẽ trả về `true` hoặc `false` cho biết liệu người dùng có được phép cập nhật `Post` đã cho hay không. Vì vậy, trong ví dụ này, hãy xác minh rằng `id` của người dùng khớp với` user_id` trên bài đăng:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

Bạn có thể tiếp tục định nghĩa thêm các phương thức cho policy cần cho các hành động khác nhau mà nó authorize. Ví dụ: bạn có thể định nghĩa các phương thức `view` hoặc `delete` để authorize cho các hành động `Post` khác nhau, nhưng hãy nhớ rằng bạn có thể thoải mái tạo ra các phương thức policy của bạn với bất kỳ tên nào bạn thích.

> {tip} Nếu bạn đã sử dụng tùy chọn `--model` khi tạo policy của bạn thông qua Artisan console, thì nó sẽ chứa các phương thức cho các hành động `view`, `create`, `update`, và `delete`.

<a name="methods-without-models"></a>
### Các phương thức không dùng Models

Một số phương thức policy chỉ nhận vào user hiện đang được authenticate và không phải là một instance của model mà họ authorize. Tình huống này là phổ biến nhất khi authorize cho các hành động `create`. Ví dụ: nếu bạn đang tạo một blog, bạn có thể muốn kiểm tra xem người dùng có được phép tạo bất kỳ bài đăng nào không.

Khi định nghĩa các phương thức policy không nhận được một instance model, chẳng hạn như phương thức `create`, nó sẽ không nhận được một instance model. Thay vào đó, bạn nên định nghĩa phương thức là chỉ chấp nhận user đang được xác thực:

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

<a name="policy-filters"></a>
### Policy Filters

Đối với một số user, bạn có thể muốn authorize với tất cả các hành động có trong một policy. Để thực hiện điều này, hãy định nghĩa một phương thức `before` trong policy. Phương thức `before` sẽ được thực thi trước bất kỳ phương thức nào khác có trong policy, nó cho phép bạn chạy hành động trước khi phương thức policy dự kiến thực sự được gọi. Tính năng này được sử dụng phổ biến nhất để authorize cho quản trị viên application thực hiện bất kỳ hành động nào:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

Nếu bạn muốn từ chối tất cả các authorization cho một user, bạn nên trả về `false` từ phương thức `before`. Nếu `null` được trả về, authorization sẽ chuyển sang phương thức policy.

> {note} Phương thức `before` của class policy sẽ không được gọi nếu class không chứa phương thức có tên khớp với tên của ability đang được kiểm tra.

<a name="authorizing-actions-using-policies"></a>
## Authorizing Actions dùng Policies

<a name="via-the-user-model"></a>
### Thông qua User Model

Model `User` đã được tạo trong applicationg Laravel của bạn và chứa hai phương thức hữu ích để authorize cho các hành động: `can` và `cant`. Phương thức `can` nhận hành động mà bạn muốn authorize và model có liên quan. Ví dụ: hãy xác định xem người dùng có được phép cập nhật model `Post` không:

    if ($user->can('update', $post)) {
        //
    }

Nếu một [policy đã được đăng ký](#registering-policies) cho model đã cho, phương thức `can` sẽ tự động gọi policy phù hợp và trả về kết quả boolean. Nếu không có policy nào được đăng ký cho model, phương thức `can` sẽ cố gắng gọi đến Closure dựa trên Gate khớp với tên hành động đã cho.

#### Actions That Don't Require Models

Hãy nhớ rằng, một số hành động như `create` có thể không yêu cầu một instance model. Trong những tình huống này, bạn có thể pass một tên class cho phương thức `can`. Tên class sẽ được sử dụng để xác định policy nào sẽ được sử dụng khi authorize cho hành động:

    use App\Post;

    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }

<a name="via-middleware"></a>
### Thông qua Middleware

Laravel có chứa một middleware có thể authorize cho các hành động trước khi incoming request đến được các route hoặc controller của bạn. Mặc định, middleware `Illuminate\Auth\Middleware\Authorize` được gán key `can` trong class `App\Http\Kernel` của bạn. Hãy khám phá một ví dụ về việc sử dụng middleware `can` để authorize một user có thể cập nhật bài đăng trên blog:

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

Trong ví dụ trên, chúng ta sẽ pass hai tham số vào middleware `can`. Đầu tiên là tên của hành động chúng ta muốn authorize và thứ hai là tham số route chúng ta muốn pass cho phương thức policy. Trong trường hợp này, vì chúng ta đang sử dụng [liên kết model ẩn](/docs/{{version}}/routing#implicit-binding), một model `Post` sẽ được truyền cho phương thức policy. Nếu người dùng không được phép thực hiện hành động đã cho, HTTP response có status code `403` sẽ được tạo bởi middleware.

#### Actions That Don't Require Models

Một lần nữa, một số hành động như `create` có thể không yêu cầu một instance model. Trong những tình huống này, bạn có thể pass một tên class cho middleware. Tên class sẽ được sử dụng để xác định policy nào sẽ được sử dụng khi authorize cho hành động:

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### Thông qua Controller Helpers

Ngoài các phương thức hữu ích được cung cấp cho model `User`, Laravel còn cung cấp phương thức `authorize` hữu ích cho bất kỳ controller nào của bạn và được mở rộng từ lớp cơ sở `App\Http\Controllers\Controller`. Giống như phương thức `can`, phương thức này chấp nhận tên của hành động bạn muốn authorize và model có liên quan. Nếu hành động không được authorize, phương thức `authorize` sẽ tạo một `Illuminate\Auth\Access\AuthorizationException`, mà trình xử lý ngoại lệ mặc định của Laravel sẽ chuyển đổi thành HTTP response có status code `403`:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

#### Actions That Don't Require Models

Như đã thảo luận trước đây, một số hành động như `create` có thể không yêu cầu một instance model. Trong những tình huống này, bạn có thể pass một tên lớp cho phương thức `authorize`. Tên class sẽ được sử dụng để xác định policy nào sẽ được sử dụng khi authorize cho hành động:

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

<a name="via-blade-templates"></a>
### Thông qua Blade Templates

Khi viết template Blade, bạn có thể hiển thị một phần của trang web cho người dùng đã được authorize để thực hiện một số hành động nhất định. Ví dụ: bạn có thể muốn hiển thị một form cập nhật cho một bài đăng blog chỉ khi người dùng thực sự có quyền cập nhật bài viết. Trong tình huống này, bạn có thể sử dụng dòng lệnh `@can` và `@cannot`:

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @elsecan('create', App\Post::class)
        <!-- The Current User Can Create New Post -->
    @endcan

    @cannot('update', $post)
        <!-- The Current User Can't Update The Post -->
    @elsecannot('create', App\Post::class)
        <!-- The Current User Can't Create New Post -->
    @endcannot

Các lệnh này là các shortcut thuận tiện để viết các câu lệnh `@if` và `@unless`. Các câu lệnh `@can` và `@cannot` ở trên lần lượt dịch sang các câu lệnh như sau:

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Can't Update The Post -->
    @endunless

#### Actions That Don't Require Models

Giống như hầu hết các phương thức authorization khác, bạn có thể pass tên class cho các lệnh `@can` và `@cannot` nếu hành động không yêu cầu một instance model:

    @can('create', App\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', App\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot
