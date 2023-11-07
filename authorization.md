# Authorization

- [Giới thiệu](#introduction)
- [Gates](#gates)
    - [Viết Gates](#writing-gates)
    - [Authorizing Actions](#authorizing-actions-via-gates)
    - [Gate Responses](#gate-responses)
    - [Chặn Gate Check](#intercepting-gate-checks)
    - [Inline Authorization](#inline-authorization)
- [Tạo Policies](#creating-policies)
    - [Tạo Policies](#generating-policies)
    - [Đăng ký Policies](#registering-policies)
- [Viết Policies](#writing-policies)
    - [Các phương thức trong Policy](#policy-methods)
    - [Policy Responses](#policy-responses)
    - [Các phương thức không dùng Models](#methods-without-models)
    - [Guest Users](#guest-users)
    - [Policy Filters](#policy-filters)
- [Authorizing Actions dùng Policies](#authorizing-actions-using-policies)
    - [Thông qua User Model](#via-the-user-model)
    - [Thông qua Controller Helpers](#via-controller-helpers)
    - [Thông qua Middleware](#via-middleware)
    - [Thông qua Blade Templates](#via-blade-templates)
    - [Cung cấp thêm thông tin](#supplying-additional-context)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc cung cấp sẵn các service [authentication](/docs/{{version}}/authentication), Laravel cũng cung cấp một cách đơn giản để authorize cho các hành động của người dùng đối với các resource đã có. Ví dụ: mặc dù người dùng đã được xác thực, nhưng họ có thể không được phép cập nhật hoặc xóa một số model Eloquent hoặc các record cơ sở dữ liệu do ứng dụng của bạn quản lý. Các tính năng authorization của Laravel cung cấp một cách dễ dàng, có tổ chức để quản lý các loại kiểm tra cho các loại authorization này.

Laravel cung cấp hai cách chính để authorization cho các hành động: [gates](#gates) và [policies](#creating-policies). Hãy nghĩ về các gates và các policies như là các routes và các controllers. Gates cung cấp một cách tiếp cận đơn giản dựa trên closure để authorization, trong khi các policies giống như là các controllers, là một nhóm logic sẽ liên quan đến một model hoặc một resource cụ thể. In this documentation, chúng ta sẽ khám phá các gates trước và sau đó sẽ đến các policies.

Bạn không cần phải chọn giữa sử dụng gates hoặc sử dụng policies khi xây dựng application. Hầu hết các application rất có thể sẽ chứa một số hỗn hợp cả gates và policies, và điều đó là hoàn toàn tốt! Gates áp dụng cho các hành động không liên quan đến bất kỳ model hoặc resource nào, chẳng hạn như xem dashboard của administrator. Ngược lại, các policies nên được sử dụng khi bạn muốn authorize một hành động cho một model hoặc một resource cụ thể.

<a name="gates"></a>
## Gates

<a name="writing-gates"></a>
### Viết Gates

> {note} Gate là một cách tuyệt vời để tìm hiểu những điều cơ bản về các tính năng authorization của Laravel; tuy nhiên, khi xây dựng các ứng dụng Laravel mạnh mẽ, bạn nên cân nhắc sử dụng [policies](#creating-policies) để tổ chức các quy tắc authorization của bạn.

Gate chỉ đơn giản là một closure để xác định xem người dùng có được phép thực hiện một hành động nhất định hay không. Thông thường, các gate được định nghĩa trong phương thức `boot` của class `App\Providers\AuthServiceProvider` bằng cách sử dụng facade `Gate`. Gates luôn nhận một instance user làm tham số đầu tiên của nó và có thể tùy chọn nhận thêm các tham số như Eloquent model có liên quan.

Trong ví dụ này, chúng ta sẽ định nghĩa một gate để xác định xem người dùng có thể cập nhật model `App\Models\Post` nào đó hay không. Gate sẽ thực hiện điều này bằng cách so sánh `id` của người dùng với `user_id` của người dùng đã tạo ra bài post:

    use App\Models\Post;
    use App\Models\User;
    use Illuminate\Support\Facades\Gate;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id;
        });
    }

Giống như controller, gate cũng có thể được định nghĩa bằng cách sử dụng một mảng class callback:

    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', [PostPolicy::class, 'update']);
    }

<a name="authorizing-actions-via-gates"></a>
### Authorizing Actions

Để authorize cho một hành động thông qua sử dụng gate, bạn cần sử dụng các phương thức `allows` hoặc `denies` được cũng cấp bởi facade `Gate`. Lưu ý rằng bạn không cần phải truyền user mà đang login cho các phương thức này. Laravel sẽ tự động truyền user đó vào gate closure này. Thông thường, hãy gọi các phương thức gate authorization trong controller của ứng dụng của bạn trước khi thực hiện bất kỳ hành động yêu cầu authorization nào:

     <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Gate;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \App\Models\Post  $post
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request, Post $post)
        {
            if (! Gate::allows('update-post', $post)) {
                abort(403);
            }

            // Update the post...
        }
    }

Nếu bạn muốn xác định xem một người dùng khác không phải là người dùng đang xác thực hiện tại có được phép thực hiện một hành động hay không, bạn có thể sử dụng phương thức `forUser` trên facade `Gate`:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }

Bạn có thể cấp phép cho nhiều hành động cùng một lúc bằng phương thức `any` hoặc `none`:

    if (Gate::any(['update-post', 'delete-post'], $post)) {
        // The user can update or delete the post...
    }

    if (Gate::none(['update-post', 'delete-post'], $post)) {
        // The user can't update or delete the post...
    }

<a name="authorizing-or-throwing-exceptions"></a>
#### Authorizing Or Throwing Exceptions

Nếu bạn muốn thử authorize cho một action và tự động đưa ra một `Illuminate\Auth\Access\AuthorizationException` nếu người dùng đó không được phép thực hiện action đã cho, bạn có thể sử dụng phương thức `authorize` của facade `Gate`. Instance của `AuthorizationException` sẽ được tự động chuyển đổi thành HTTP response 403 bởi exception handle của Laravel:

    Gate::authorize('update-post', $post);

    // The action is authorized...

<a name="gates-supplying-additional-context"></a>
#### Supplying Additional Context

Các phương thức của gate để authorize các quyền (`allows`, `denies`, `check`, `any`, `none`, `authorize`, `can`, `cannot`) và các [lệnh Blade](#via-blade-templates) về authorization (`@can`, `@cannot`, `@canany`) có thể nhận vào một mảng làm tham số thứ hai của chúng. Các phần tử trong mảng này được truyền vào dưới dạng tham số cho closure gate và có thể được sử dụng để thêm thông tin khi đưa ra quyết định authorization:

    use App\Models\Category;
    use App\Models\User;
    use Illuminate\Support\Facades\Gate;

    Gate::define('create-post', function (User $user, Category $category, $pinned) {
        if (! $user->canPublishToGroup($category->group)) {
            return false;
        } elseif ($pinned && ! $user->canPinPosts()) {
            return false;
        }

        return true;
    });

    if (Gate::check('create-post', [$category, $pinned])) {
        // The user can create the post...
    }

<a name="gate-responses"></a>
### Gate Responses

Hiện tại, chúng ta mới chỉ kiểm tra các gate trả về giá trị boolean đơn giản. Tuy nhiên, đôi khi bạn có thể muốn trả về một response chi tiết hơn, chứa cả một thông báo lỗi. Để làm như vậy, bạn có thể trả về một `Illuminate\Auth\Access\Response` từ gate của bạn:

    use App\Models\User;
    use Illuminate\Auth\Access\Response;
    use Illuminate\Support\Facades\Gate;

    Gate::define('edit-settings', function (User $user) {
        return $user->isAdmin
                    ? Response::allow()
                    : Response::deny('You must be an administrator.');
    });

Thậm chí khi bạn trả về một response authorization từ gate của bạn, phương thức `Gate::allows` sẽ vẫn trả về một giá trị boolean; tuy nhiên, bạn có thể sử dụng phương thức `Gate::inspect` để nhận được response authorization đầy đủ do gate trả về:

    $response = Gate::inspect('edit-settings');

    if ($response->allowed()) {
        // The action is authorized...
    } else {
        echo $response->message();
    }

Tất nhiên, khi sử dụng phương thức `Gate::authorize`, cái mà đưa ra một `AuthorizationException` nếu action không được authorize, thông báo lỗi mà được cung cấp bởi response authorize sẽ được truyền tới response HTTP:

    Gate::authorize('edit-settings');

    // The action is authorized...

<a name="intercepting-gate-checks"></a>
### Chặn Gate Check

Thỉnh thoảng, bạn có thể muốn cho phép tất cả các hành động cho một người dùng cụ thể. Bạn có thể sử dụng phương thức `before` để định nghĩa một closure sẽ được chạy trước khi tất cả các authorization khác được check:

    use Illuminate\Support\Facades\Gate;

    Gate::before(function ($user, $ability) {
        if ($user->isAdministrator()) {
            return true;
        }
    });

Nếu closure `before` trả về một kết quả khác null thì kết quả đó sẽ được coi là kết quả của việc authorization check.

Bạn có thể sử dụng phương thức `after` để định nghĩa một closure sẽ được thực thi sau tất cả các lần authorization check.

    Gate::after(function ($user, $ability, $result, $arguments) {
        if ($user->isAdministrator()) {
            return true;
        }
    });

Tương tự như phương thức `before`, nếu closure `after` trả về một kết quả khác null thì kết quả đó sẽ được coi là kết quả của việc authorization check.

<a name="inline-authorization"></a>
### Inline Authorization

Đôi khi, bạn có thể muốn xác định xem người dùng đang được xác thực có được phép thực hiện một hành động nhất định mà không cần viết gate tương ứng với hành động đó hay không. Laravel cho phép bạn thực hiện các loại kiểm tra authorization "inline" này thông qua các phương thức `Gate::allowIf` và `Gate::denyIf`:

```php
use Illuminate\Support\Facades\Auth;

Gate::allowIf(fn ($user) => $user->isAdministrator());

Gate::denyIf(fn ($user) => $user->banned());
```

Nếu hành động không được phép hoặc nếu không có người dùng nào đang được xác thực, thì Laravel sẽ tự động đưa ra một exception `Illuminate\Auth\Access\AuthorizationException`. Các instance của `AuthorizationException` được exception handler của Laravel tự động chuyển thành HTTP response 403:

<a name="creating-policies"></a>
## Tạo Policies

<a name="generating-policies"></a>
### Tạo Policies

Các Policy là các class tổng hợp các logic authorization liên quan đến một model hoặc resource cụ thể. Ví dụ: nếu application của bạn là một trang blog, bạn có thể có model `App\Models\Post` và `App\Policies\PostPolicy` tương ứng để authorization cho các hành động của người dùng như tạo hoặc cập nhật bài đăng.

Bạn có thể tạo một policy bằng cách sử dụng [lệnh Artisan](/docs/{{version}}/artisan) `make:policy`. Policy được tạo ra sẽ được lưu vào trong thư mục `app/Policies`. Nếu thư mục này không tồn tại trong application của bạn, Laravel sẽ tạo nó cho bạn:

    php artisan make:policy PostPolicy

Lệnh `make:policy` sẽ tạo ra một class policy trống. Nếu bạn muốn tạo ra với một class với các ví dụ về các phương thức policy liên quan đến xem, tạo, cập nhật và xóa resource, bạn có thể cung cấp tùy chọn `--model` khi thực thi lệnh:

    php artisan make:policy PostPolicy --model=Post

<a name="registering-policies"></a>
### Đăng ký Policies

Sau khi class policy đã được tạo, nó cần phải được đăng ký. Đăng ký policy là cách chúng ta có thể thông báo cho Laravel là policy nào sẽ được sử dụng khi authorize cho các hành động đối với một loại model nhất định.

`App\Providers\AuthServiceProvider` đi kèm với application Laravel có chứa thuộc tính `policies` dùng để ánh xạ các Eloquent model của bạn tới các policy tương ứng với chúng. Đăng ký policy sẽ hướng dẫn cho Laravel khi nào nên sử dụng một policy cho một hành động của một Eloquent model nhất định:

    <?php

    namespace App\Providers;

    use App\Models\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;

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

<a name="policy-auto-discovery"></a>
#### Tự động dăng ký policy

Thay vì đăng ký policy cho model theo cách thủ công, Laravel có thể tự động đăng ký các policy miễn là các model và policy tuân theo một quy tắc đặt tên theo tiêu chuẩn Laravel. Cụ thể, các policy phải nằm trong thư mục `Policies` hoặc ở trên thư mục chứa các model của bạn. Vì vậy, ví dụ: các model có thể được lưu trong thư mục `app/Models` trong khi các policy có thể được lưu trong thư mục `app/Policies`. Trong tình huống này, Laravel sẽ kiểm tra các policy trong thư mục `app/Models/Policies` rồi mới đến thư mục `app/Policies`. Ngoài ra, tên policy phải khớp với tên của model và có hậu tố `Policy`. Vì vậy, một model `User` sẽ tương ứng với một class policy như sau: `UserPolicy`.

Nếu bạn muốn tự định nghĩa logic đăng ký policy theo cách của bạn, bạn có thể đăng ký một tùy biến đăng ký policy callback bằng cách sử dụng phương thức `Gate::guessPolicyNamesUsing`. Thông thường, phương thức này sẽ được gọi từ phương thức `boot` trong `AuthServiceProvider` trong ứng dụng của bạn:

    use Illuminate\Support\Facades\Gate;

    Gate::guessPolicyNamesUsing(function ($modelClass) {
        // Return the name of the policy class for the given model...
    });

> {note} Bất kỳ policy nào được ánh xạ trong `AuthServiceProvider` cũng sẽ được ưu tiên hơn các policy khác được đăng ký tự động.

<a name="writing-policies"></a>
## Viết Policies

<a name="policy-methods"></a>
### Các phương thức trong Policy

Khi class policy đã được đăng ký, bạn có thể thêm các phương thức cho các hành động mà bạn cần authorize. Ví dụ: hãy định nghĩa thêm một phương thức `update` trên `PostPolicy` để kiểm tra xem `App\Models\User` hiện tại có được phép cập nhật một `App\Models\Post` đã cho hay không.

Phương thức `update` sẽ nhận vào một `User` và một `Post` làm tham số của nó và sẽ trả về giá trị `true` hoặc `false` cho biết liệu người dùng đó có được phép cập nhật `Post` đã cho hay không. Vì vậy, trong ví dụ dưới đây, chúng ta sẽ kiểm tra bằng cách `id` của người dùng đưa vào có khớp với `user_id` của bài đăng đã cho hay không:

    <?php

    namespace App\Policies;

    use App\Models\Post;
    use App\Models\User;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

Bạn có thể tiếp tục định nghĩa thêm các phương thức mà bạn cần authorize cho các hành động khác. Ví dụ: bạn có thể định nghĩa thêm các phương thức `view` hoặc `delete` để authorize cho `Post` liên quan tới hành động, ngoài ra bạn cũng có thể tạo thêm bất kỳ các phương thức policy với bất kỳ cái tên nào mà bạn mong muốn.

Nếu bạn đã sử dụng option `--model` khi tạo policy thông qua Artisan console, thì nó sẽ chứa sẵn các phương thức cho các hành động `viewAny`, `view`, `create`, `update`, `delete`, `restore`, và `forceDelete`.

> {tip} Tất cả các policy được gọi thông qua Laravel [service container](/docs/{{version}}/container), cho phép bạn khai báo bất kỳ phụ thuộc cần thiết nào trong hàm constructor của policy để chúng có thể tự động được inject.

<a name="policy-responses"></a>
### Policy Responses

Hiện tại, chúng ta mới chỉ kiểm tra các phương thức policy trả về giá trị boolean đơn giản. Tuy nhiên, đôi khi bạn có thể muốn trả về một response chi tiết hơn, chứa cả một thông báo lỗi. Để làm như vậy, bạn có thể trả về instance `Illuminate\Auth\Access\Response` từ phương thức policy của bạn:

    use App\Models\Post;
    use App\Models\User;
    use Illuminate\Auth\Access\Response;

    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\Models\User  $user
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Auth\Access\Response
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id
                    ? Response::allow()
                    : Response::deny('You do not own this post.');
    }

Khi trả về một response authorization từ policy của bạn, phương thức `Gate::allows` sẽ vẫn trả về một giá trị boolean đơn giản; tuy nhiên, bạn có thể sử dụng phương thức `Gate::inspect` để nhận được response authorization đầy đủ do gate trả về:

    use Illuminate\Support\Facades\Gate;

    $response = Gate::inspect('update', $post);

    if ($response->allowed()) {
        // The action is authorized...
    } else {
        echo $response->message();
    }

Khi sử dụng phương thức `Gate::authorize`, cái để đưa ra một `AuthorizationException` nếu action không được authorize, thông báo lỗi mà được cung cấp bởi response authorize sẽ được truyền tới response HTTP:

    Gate::authorize('update', $post);

    // The action is authorized...

<a name="methods-without-models"></a>
### Các phương thức không dùng Models

Một số phương thức policy chỉ nhận vào một instance của người dùng đang được xác thực. Tình huống này phổ biến nhất khi đang authorize các hành động `create`. Ví dụ: nếu bạn đang tạo một blog, bạn có thể muốn xác định xem người dùng có được phép tạo bất kỳ bài post nào không. Trong những tình huống như vậy, phương thức policy của bạn sẽ chỉ nhận vào một instance người dùng:

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\Models\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        return $user->role == 'writer';
    }

<a name="guest-users"></a>
### Guest Users

Mặc định, tất cả các gate và policy sẽ tự động trả về `false` nếu request đó không được tạo bởi một người dùng đã được authenticate. Tuy nhiên, nếu bạn muốn, bạn cũng có thể cho phép các request này đi qua các gate và policy của bạn bằng cách khai báo thêm "optional" hoặc cung cấp giá trị `null` mặc định cho định nghĩa tham số user:

    <?php

    namespace App\Policies;

    use App\Models\Post;
    use App\Models\User;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Post  $post
         * @return bool
         */
        public function update(?User $user, Post $post)
        {
            return optional($user)->id === $post->user_id;
        }
    }

<a name="policy-filters"></a>
### Policy Filters

Đối với một số user, bạn có thể muốn cho phép pass qua tất cả các hành động có trong một policy. Để thực hiện điều này, hãy định nghĩa một phương thức `before` trong policy. Phương thức `before` sẽ được thực thi trước, và sau đó mới đến các phương thức khác có trong policy, nó cho phép bạn pass qua tất cả các hành động, trước khi một phương thức policy được gọi. Tính năng này được sử dụng để cho phép một quản trị viên có thể thực hiện bất kỳ hành động nào mà họ muốn:

    use App\Models\User;

    /**
     * Perform pre-authorization checks.
     *
     * @param  \App\Models\User  $user
     * @param  string  $ability
     * @return void|bool
     */
    public function before(User $user, $ability)
    {
        if ($user->isAdministrator()) {
            return true;
        }
    }

Nếu bạn muốn từ chối tất cả các kiểm tra authorization cho một loại người dùng cụ thể thì bạn có thể trả về `false` từ phương thức `before`. Nếu `null` được trả về, thì authorization check sẽ chuyển sang phương thức policy.

> {note} Phương thức `before` của policy sẽ không được gọi nếu policy đó không chứa phương thức nào mà có tên khớp với tên của hành động đang được kiểm tra.

<a name="authorizing-actions-using-policies"></a>
## Authorizing Actions dùng Policies

<a name="via-the-user-model"></a>
### Thông qua User Model

Model `App\Models\User` đi kèm trong ứng dụng Laravel của bạn có chứa sẵn hai phương thức hữu ích để authorize cho các hành động là: `can` và `cant`. Phương thức `can` và `cannot` sẽ nhận vào tên của các hành động mà bạn muốn cho phép và các model có liên quan. Ví dụ: hãy kiểm tra xem người dùng đó có được phép cập nhật model `App\Models\Post` hay không. Thông thường, điều này sẽ được thực hiện trong một phương thức của controller:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \App\Models\Post  $post
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request, Post $post)
        {
            if ($request->user()->cannot('update', $post)) {
                abort(403);
            }

            // Update the post...
        }
    }

Nếu một [policy được đăng ký](#registering-policies) cho một model nhất định, thì phương thức `can` sẽ tự động gọi policy phù hợp và trả về kết quả boolean. Nếu không có policy nào được đăng ký cho model đó, phương thức `can` sẽ thử gọi vào gate dựa trên closure với tên hành động đã cho.

<a name="user-model-actions-that-dont-require-models"></a>
#### Actions That Don't Require Models

Hãy nhớ rằng, một số hành động có thể tương ứng với các phương thức policy như `create` nhưng lại không yêu cầu một instance model. Trong những trường hợp này, bạn có thể chuyển tên classs cho phương thức `can`. Tên class này sẽ được sử dụng để xác định policy nào sẽ được sử dụng khi authorize cho hành động:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Create a post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            if ($request->user()->cannot('create', Post::class)) {
                abort(403);
            }

            // Create the post...
        }
    }

<a name="via-controller-helpers"></a>
### Thông qua Controller Helpers

Ngoài các phương thức hữu ích được cung cấp cho model `App\Models\User`, Laravel còn cung cấp phương thức `authorize` cho bất kỳ controller nào mà được extend từ class `App\Http\Controllers\Controller`.

Giống như phương thức `can`, phương thức này chấp nhận tên của một hành động mà bạn muốn authorize và một model ở tham số thứ hai. Nếu hành động không được authorize, phương thức `authorize` sẽ tạo ra một exception `Illuminate\Auth\Access\AuthorizationException`, mà trình xử lý exception của Laravel sẽ chuyển exception đó thành một HTTP response có status code 403:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \App\Models\Post  $post
         * @return \Illuminate\Http\Response
         *
         * @throws \Illuminate\Auth\Access\AuthorizationException
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

<a name="controller-actions-that-dont-require-models"></a>
#### Actions That Don't Require Models

Như đã thảo luận ở phía trên, một số phương thức policy như `create` sẽ không yêu cầu một model ở tham số thứ hai. Trong những tình huống này, bạn nên truyền vào tên của một class cho phương thức `authorize`. Tên class sẽ được sử dụng để xác định policy nào sẽ được sử dụng khi authorize cho các hành động:

    use App\Models\Post;
    use Illuminate\Http\Request;

    /**
     * Create a new blog post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

<a name="authorizing-resource-controllers"></a>
#### Authorizing Resource Controllers

Nếu bạn đang sử dụng [resource controller](/docs/{{version}}/controllers#resource-controllers), bạn có thể sử dụng phương thức `authorizeResource` trong hàm constructor của controller của bạn. Phương thức này sẽ gán một định nghĩa middleware `can` thích hợp cho các phương thức trong resource controller đó.

Phương thức `authorizeResource` sẽ nhận tên class của model làm tham số đầu tiên và tên của tham số route chứa ID của model làm tham số thứ hai của nó. Bạn nên đảm bảo [resource controller](/docs/{{version}}/controllers#resource-controllers) của bạn cũng được tạo cùng với một flag `--model` để nó yêu cầu các phương thức bắt buộc và khai báo thêm cho loại model đó:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Create the controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->authorizeResource(Post::class, 'post');
        }
    }

Các phương thức controller sau sẽ được ánh xạ tới các phương thức policy tương ứng với chúng. Khi các request được chuyển đến phương thức controller đã cho, phương thức policy tương ứng sẽ tự động được gọi trước khi phương thức controller được thực thi:

| Controller Method | Policy Method |
| --- | --- |
| index | viewAny |
| show | view |
| create | create |
| store | create |
| edit | update |
| update | update |
| destroy | delete |

> {tip} Bạn có thể sử dụng lệnh `make:policy` với tùy chọn `--model` để tạo nhanh một class policy cho một model nhất định: `php artisan make:policy PostPolicy --model=Post`.

<a name="via-middleware"></a>
### Via Middleware

Laravel có chứa một middleware có thể authorize cho các hành động trước khi request vào thậm chí trước cả các route hoặc controller của bạn. Mặc định, middleware `Illuminate\Auth\Middleware\Authorize` sẽ được gán với từ khóa `can` trong class `App\Http\Kernel` của bạn. Hãy khám phá một ví dụ về việc sử dụng middleware `can` để cho phép người dùng có thể cập nhật bài đăng:

    use App\Models\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

Trong ví dụ này, chúng ta đang truyền hai tham số của middleware `can`. Đầu tiên là tên của hành động mà chúng ta muốn authorize và thứ hai là tham số route mà chúng ta muốn truyền đến phương thức policy. Trong trường hợp này, vì chúng ta đang sử dụng [liên kết model ngầm](/docs/{{version}}/routing#implicit-binding), một model `App\Models\Post` sẽ được chuyển cho phương thức policy. Nếu người dùng hiện tại không được phép thực hiện một hành động nhất định,  HTTP response có status code 403 sẽ được middleware trả về.

Để thuận tiện, bạn cũng có thể đính kèm middleware `can` vào route của bạn bằng phương thức `can`:

    use App\Models\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->can('update', 'post');

<a name="middleware-actions-that-dont-require-models"></a>
#### Actions That Don't Require Models

Một lần nữa, một số phương thức policy như `create` sẽ không yêu cầu một model ở tham số thứ hai. Trong những tình huống này, bạn nên truyền vào tên của một class cho middleware. Tên class sẽ được sử dụng để xác định policy nào sẽ được sử dụng khi authorize cho các hành động:

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Models\Post');

Để chỉ định toàn bộ tê các class trong định nghĩa chuỗi middleware có thể trở nên rườm rà. Vì lý do đó, bạn có thể chọn đính kèm middleware `can` vào route của bạn bằng phương thức `can`:

    use App\Models\Post;

    Route::post('/post', function () {
        // The current user may create posts...
    })->can('create', Post::class);

<a name="via-blade-templates"></a>
### Thông qua Blade Templates

Khi viết template Blade, bạn có thể hiển thị một phần của trang web cho những người dùng đã được authorize để thực hiện một số hành động nhất định. Ví dụ: bạn có thể muốn hiển thị một form cập nhật cho một bài đăng chỉ khi người dùng đó thực sự có quyền cập nhật bài đăng. Trong những tình huống như thế này, bạn có thể sử dụng lệnh `@can` và `@cannot`:

```html
@can('update', $post)
    <!-- The current user can update the post... -->
@elsecan('create', App\Models\Post::class)
    <!-- The current user can create new posts... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- The current user cannot update the post... -->
@elsecannot('create', App\Models\Post::class)
    <!-- The current user cannot create new posts... -->
@endcannot
```

Các lệnh này là các shortcut thuận tiện để không phải viết các câu lệnh như `@if` và `@unless`. Các câu lệnh `@can` và `@cannot` ở trên cũng tương đương với các câu lệnh if như sau:

```html
@if (Auth::user()->can('update', $post))
    <!-- The current user can update the post... -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- The current user cannot update the post... -->
@endunless
```

Bạn cũng có thể kiểm tra xem người dùng có được phép thực hiện bất kỳ hành động nào từ một loạt các hành động nhất định. Để thực hiện việc này, hãy sử dụng lệnh `@canany`:

```html
@canany(['update', 'view', 'delete'], $post)
    <!-- The current user can update, view, or delete the post... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- The current user can create a post... -->
@endcanany
```

<a name="blade-actions-that-dont-require-models"></a>
#### Actions That Don't Require Models

Giống như hầu hết các phương thức authorization khác, bạn có thể truyền tên class cho các lệnh `@can` và `@cannot` nếu hành động đó không yêu cầu một model:

```html
@can('create', App\Models\Post::class)
    <!-- The current user can create posts... -->
@endcan

@cannot('create', App\Models\Post::class)
    <!-- The current user can't create posts... -->
@endcannot
```

<a name="supplying-additional-context"></a>
### Cung cấp thêm thông tin

Khi authorize các action bằng policy, bạn có thể truyền một mảng làm tham số thứ hai cho hàm và các helper authorize khác nhau. Phần tử đầu tiên trong mảng sẽ được sử dụng để xác định policy nào sẽ được gọi, trong khi phần tử còn lại của mảng được truyền dưới dạng tham số cho phương thức policy và có thể được sử dụng để thêm thông tin khi đưa ra quyết định authorize. Ví dụ: hãy xem xét định nghĩa phương thức `PostPolicy` sau đây có chứa tham số thêm `$category` bổ sung:

    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\Models\User  $user
     * @param  \App\Models\Post  $post
     * @param  int  $category
     * @return bool
     */
    public function update(User $user, Post $post, int $category)
    {
        return $user->id === $post->user_id &&
               $user->canUpdateCategory($category);
    }

Khi thử xác định xem người dùng hiện tại có thể cập nhật một bài đăng nhất định hay không, chúng ta có thể gọi phương thức policy này như sau:

    /**
     * Update the given blog post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', [$post, $request->category]);

        // The current user can update the blog post...
    }
