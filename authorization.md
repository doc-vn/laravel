# Authorization

- [Giới thiệu](#introduction)
- [Gates](#gates)
    - [Viết Gates](#writing-gates)
    - [Authorizing Actions](#authorizing-actions-via-gates)
    - [Chặn Gate Check](#intercepting-gate-checks)
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

Ngoài việc mặc định cung cấp các dịch vụ [authentication](/docs/{{version}}/authentication), Laravel cũng cung cấp một cách đơn giản để authorize cho các hành động của người dùng đối với các resource đã có. Giống như authentication, cách tiếp cận authorize của Laravel cũng rất đơn giản và có hai cách chính để authorize một hành động đó là: gates và policies.

Hãy nghĩ về các gates và các policies như là các routes và các controllers. Gates cung cấp một cách tiếp cận đơn giản dựa trên Closure để authorization, trong khi các policies giống như là các controllers, là một nhóm logic sẽ liên quan đến một model hoặc một resource cụ thể. Chúng ta sẽ khám phá các gates trước và sau đó sẽ đến các policies.

Bạn không cần phải chọn giữa sử dụng gates hoặc sử dụng policies khi xây dựng application. Hầu hết các application rất có thể sẽ chứa hỗn hợp cả gates và policies, và điều đó là hoàn toàn tốt! Gates áp dụng cho các hành động không liên quan đến bất kỳ model hoặc resource nào, chẳng hạn như xem dashboard của administrator. Ngược lại, các policies nên được sử dụng khi bạn muốn authorize một hành động cho một model hoặc resource cụ thể.

<a name="gates"></a>
## Gates

<a name="writing-gates"></a>
### Viết Gates

Gates là các Closure để xác định xem người dùng có được phép thực hiện một hành động nhất định hay không và thường sẽ được định nghĩa trong class `App\Providers\AuthServiceProvider` bằng cách sử dụng facade `Gate`. Gates luôn nhận một instance user làm tham số đầu tiên của nó và có thể tùy chọn nhận thêm các tham số như Eloquent model có liên quan:

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

Gates cũng có thể được định nghĩa dưới dạng là chuỗi callback `Class@method`, giống như cách gọi controller trong route:

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

Bạn cũng có thể định nghĩa nhiều hành động trong Gate cùng một lúc bằng phương thức `resource`:

    Gate::resource('posts', 'App\Policies\PostPolicy');

Điều này sẽ tương đương với việc định nghĩa các Gate như sau:

    Gate::define('posts.view', 'App\Policies\PostPolicy@view');
    Gate::define('posts.create', 'App\Policies\PostPolicy@create');
    Gate::define('posts.update', 'App\Policies\PostPolicy@update');
    Gate::define('posts.delete', 'App\Policies\PostPolicy@delete');

Mặc định, các hành động `view`, `create`, `update`, và `delete` sẽ được định nghĩa. Bạn có thể ghi đè các hành động khác bằng cách truyền một mảng làm tham số thứ ba cho phương thức `resource`. Các key của mảng đó cần định nghĩa các tên của hành động trong khi các giá trị sẽ định nghĩa các tên của phương thức. Ví dụ: đoạn code sau sẽ chỉ tạo ra hai định nghĩa Gate mới là `posts.image` và `posts.photo`:

    Gate::resource('posts', 'PostPolicy', [
        'image' => 'updateImage',
        'photo' => 'updatePhoto',
    ]);

<a name="authorizing-actions-via-gates"></a>
### Authorizing Actions

Để authorize cho một hành động thông qua sử dụng gate, bạn cần sử dụng các phương thức `allows` hoặc `denies`. Lưu ý rằng bạn không cần phải truyền user mà đang login cho các phương thức này. Laravel sẽ tự động truyền user đó vào gate Closure này:

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }

    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }

Còn nếu bạn muốn kiểm tra một user cụ thể nào đó có được phép thực hiện một hành động hay không, bạn có thể sử dụng phương thức `forUser` trên facade `Gate`:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }

<a name="intercepting-gate-checks"></a>
#### Chặn Gate Check

Thỉnh thoảng, bạn có thể muốn cho phép tất cả các hành động cho một người dùng cụ thể. Bạn có thể sử dụng phương thức `before` để định nghĩa một callback sẽ được chạy trước khi tất cả các authorization khác được check:

    Gate::before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

Nếu callback `before` trả về một kết quả khác null thì kết quả đó sẽ được coi là kết quả của việc kiểm tra.

Bạn có thể sử dụng phương thức `after` để định nghĩa một callback sẽ được thực thi sau mỗi lần authorization check. Tuy nhiên, bạn không thể đổi được kết quả authorization check từ phương thức `after`:

    Gate::after(function ($user, $ability, $result, $arguments) {
        //
    });

<a name="creating-policies"></a>
## Tạo Policies

<a name="generating-policies"></a>
### Tạo Policies

Các Policy là các class tổng hợp các logic authorization liên quan đến một model hoặc resource cụ thể. Ví dụ: nếu application của bạn là một trang blog, bạn có thể có model `Post` và `PostPolicy` tương ứng để authorization cho các hành động của người dùng như tạo hoặc cập nhật bài đăng.

Bạn có thể tạo một policy bằng cách sử dụng [lệnh artisan](/docs/{{version}}/artisan) `make:policy`. Policy được tạo ra sẽ được lưu vào trong thư mục `app/Policies`. Nếu thư mục này không tồn tại trong application của bạn, Laravel sẽ tạo nó cho bạn:

    php artisan make:policy PostPolicy

Lệnh `make:policy` sẽ tạo ra một class policy trống. Nếu bạn muốn tạo ra với một class có các phương thức policy "CRUD" cơ bản, thì bạn có thể thêm một option `--model` khi thực hiện lệnh trên:

    php artisan make:policy PostPolicy --model=Post

> {tip} Tất cả các policy được resolve thông qua Laravel [service container](/docs/{{version}}/container), cho phép bạn khai báo bất kỳ phụ thuộc nào mà bạn cần trong hàm constructor của policy để chúng có thể được inject vào cho bạn.

<a name="registering-policies"></a>
### Đăng ký Policies

Sau khi policy đã tồn tại, bạn cần phải đăng ký nó. `AuthServiceProvider` đi kèm với application Laravel có chứa thuộc tính `policies` dùng để ánh xạ các Eloquent model của bạn tới các policy tương ứng với chúng. Đăng ký policy sẽ hướng dẫn cho Laravel khi nào nên sử dụng một policy cho một hành động của một model nhất định:

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

Khi policy đã được đăng ký, bạn có thể thêm các phương thức cho các hành động mà bạn cần authorize. Ví dụ: hãy định nghĩa thêm một phương thức `update` trên `PostPolicy` để kiểm tra xem `User` hiện tại có được phép cập nhật một `Post` đã cho hay không.

Phương thức `update` sẽ nhận vào một `User` và một `Post` làm tham số của nó và sẽ trả về giá trị `true` hoặc `false` cho biết liệu người dùng đó có được phép cập nhật `Post` đã cho hay không. Vì vậy, trong ví dụ dưới đây, nó sẽ kiểm tra bằng cách `id` của người dùng đưa vào có khớp với `user_id` của bài đăng đã cho hay không:

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

Bạn có thể tiếp tục định nghĩa thêm các phương thức mà bạn cần authorize cho các hành động khác. Ví dụ: bạn có thể định nghĩa thêm authorize các phương thức `view` hoặc `delete` dành cho một `Post`, ngoài ra bạn cũng có thể tạo thêm bất kỳ các phương thức policy với bất kỳ cái tên nào mà bạn mong muốn.

> {tip} Nếu bạn đã sử dụng option `--model` khi tạo policy thông qua Artisan console, thì nó sẽ chứa sẵn các phương thức cho các hành động `view`, `create`, `update`, và `delete`.

<a name="methods-without-models"></a>
### Các phương thức không dùng Models

Có một số phương thức policy chỉ nhận vào user hiện tại đang được authenticate mà không nhận thêm model ở tham số thứ hai. Tình huống này rất phổ biến, nhất là khi authorize cho các hành động `create`. Ví dụ: nếu bạn đang tạo một blog, bạn có thể muốn kiểm tra xem người dùng này có được phép tạo một bài đăng hay không.

Khi định nghĩa các phương thức policy không nhận vào tham số thứ hai, ví dụ như phương thức `create`, nó sẽ không nhận vào tham số thứ hai. Bạn nên định nghĩa phương thức đó chỉ chấp nhận một user đang được authenticate:

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

Đối với một số user, bạn có thể muốn cho phép pass qua tất cả các hành động có trong một policy. Để thực hiện điều này, hãy định nghĩa một phương thức `before` trong policy. Phương thức `before` sẽ được thực thi trước, và sau đó mới đến các phương thức khác có trong policy, nó cho phép bạn pass qua tất cả các hành động, trước khi một phương thức policy được gọi. Tính năng này được sử dụng để cho phép một quản trị viên có thể thực hiện bất kỳ hành động nào mà họ muốn:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

Nếu bạn muốn từ chối tất cả các quyền cho một user, bạn nên trả về `false` từ phương thức `before`. Nếu `null` được trả về, thì authorization sẽ chuyển tiếp sang phương thức policy.

> {note} Phương thức `before` của policy sẽ không được gọi nếu policy đó không chứa phương thức nào mà có tên khớp với tên của hành động đang được kiểm tra.

<a name="authorizing-actions-using-policies"></a>
## Authorizing Actions dùng Policies

<a name="via-the-user-model"></a>
### Thông qua User Model

Model `User` đi kèm trong ứng dụng Laravel của bạn có chứa sẵn hai phương thức hữu ích để authorize cho các hành động là: `can` và `cant`. Phương thức `can` nhận vào các hành động mà bạn muốn cho phép và các model có liên quan. Ví dụ: hãy kiểm tra xem người dùng đó có được phép cập nhật model `Post` hay không:

    if ($user->can('update', $post)) {
        //
    }

Nếu một [policy đã được đăng ký](#registering-policies) cho một model đã cho, thì phương thức `can` sẽ được tự động gọi policy đó và trả về kết quả boolean. Nếu không có policy nào đăng ký cho một model, phương thức `can` sẽ cố gắng gọi đến Closure dựa trên Gate khớp với tên hành động đã cho.

#### Actions That Don't Require Models

Hãy nhớ rằng, một số hành động như `create` có thể không yêu cầu tham số thứ hai. Trong những tình huống như này, bạn có thể truyền vào tên của một class cho phương thức `can`. Tên class sẽ được sử dụng để xác định policy nào sẽ được sử dụng trong khi authorize cho các hành động:

    use App\Post;

    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }

<a name="via-middleware"></a>
### Thông qua Middleware

Laravel có chứa sẵn một middleware có thể authorize cho các hành động trước khi request đến được với các route hoặc controller của bạn. Mặc định, middleware `Illuminate\Auth\Middleware\Authorize` sẽ được gán với key `can` trong class `App\Http\Kernel` của bạn. Hãy xem một ví dụ về việc sử dụng middleware `can` để authorize một user cập nhật một bài đăng trên blog:

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

Trong ví dụ trên, chúng ta đã truyền hai tham số vào middleware `can`. Đầu tiên là tên của hành động mà chúng ta muốn authorize và thứ hai là tham số route mà chúng ta muốn truyền cho phương thức policy. Trong ví dụ trên, vì chúng ta đang sử dụng [liên kết model ẩn](/docs/{{version}}/routing#implicit-binding), nên một model `Post` sẽ được truyền vào phương thức policy. Nếu người dùng hiện tại không được phép thực hiện các hành động đã được truyền vào, thì một HTTP response có status code `403` sẽ được tạo ra và trả vể bởi middleware.

#### Actions That Don't Require Models

Một lần nữa, một số hành động như `create` sẽ không yêu cầu một model. Trong những tình huống như thế này, bạn có thể truyền vào tên một class cho middleware. Tên class này sẽ được sử dụng để xác định policy nào sẽ được sử dụng khi authorize cho các hành động:

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### Thông qua Controller Helpers

Ngoài các phương thức hữu ích được cung cấp cho model `User`, Laravel còn cung cấp phương thức `authorize` cho bất kỳ controller nào mà được extend từ class `App\Http\Controllers\Controller`. Giống như phương thức `can`, phương thức này chấp nhận tên của một hành động mà bạn muốn authorize và một model ở tham số thứ hai. Nếu hành động không được authorize, phương thức `authorize` sẽ tạo ra một `Illuminate\Auth\Access\AuthorizationException`, mà trình xử lý exception mặc định của Laravel sẽ chuyển exception đó thành một HTTP response có status code `403`:

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

Như đã thảo luận ở phía trên, một số hành động như `create` có thể không yêu cầu một model ở tham số thứ hai. Trong những tình huống này, bạn có thể truyền vào tên của một class cho phương thức `authorize`. Tên class sẽ được sử dụng để xác định policy nào sẽ được sử dụng khi authorize cho các hành động:

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

Khi viết template Blade, bạn có thể hiển thị một phần của trang web cho những người dùng đã được authorize để thực hiện một số hành động nhất định. Ví dụ: bạn có thể muốn hiển thị một form cập nhật cho một bài đăng chỉ khi người dùng đó thực sự có quyền cập nhật bài đăng. Trong những tình huống như thế này, bạn có thể sử dụng lệnh `@can` và `@cannot`:

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

Các lệnh này là các shortcut thuận tiện để không phải viết các câu lệnh như `@if` và `@unless`. Các câu lệnh `@can` và `@cannot` ở trên có thể lần lượt được dịch sang các câu lệnh if như sau:

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Can't Update The Post -->
    @endunless

#### Actions That Don't Require Models

Giống như hầu hết các phương thức authorization khác, bạn có thể truyền tên class cho các lệnh `@can` và `@cannot` nếu hành động đó không yêu cầu một model:

    @can('create', App\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', App\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot
