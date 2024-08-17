# URL Generation

- [Giới thiệu](#introduction)
- [Cơ bản](#the-basics)
    - [Tạo một URL](#generating-urls)
    - [Truy cập vào URL hiện tại](#accessing-the-current-url)
- [URLs cho Named Routes](#urls-for-named-routes)
    - [Signed URLs](#signed-urls)
- [URLs cho Controller Actions](#urls-for-controller-actions)
- [Giá trị mặc định](#default-values)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một số helper để hỗ trợ bạn tạo URL cho application của bạn. Những helper này chủ yếu hữu ích khi tạo link trong các template và API response hoặc khi tạo response chuyển hướng đến một phần khác trong application của bạn.

<a name="the-basics"></a>
## Cơ bản

<a name="generating-basic-urls"></a>
### Tạo một URL

Helper `url` có thể được sử dụng để tạo các URL tùy biến cho application của bạn. URL được tạo ra sẽ tự động sử dụng scheme (HTTP hoặc HTTPS) và host từ request hiện tại đang được xử lý bởi ứng dụng:

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Truy cập vào URL hiện tại

Nếu như không có đường dẫn nào được truyền vào cho helper `url`, thì một instance `Illuminate\Routing\UrlGenerator` sẽ được trả về, cho phép bạn truy cập vào thông tin về URL hiện tại:

    // Get the current URL without the query string...
    echo url()->current();

    // Get the current URL including the query string...
    echo url()->full();

    // Get the full URL for the previous request...
    echo url()->previous();

Các phương thức này cũng có thể được truy cập thông qua [facade](/docs/{{version}}/facades) `URL`:

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URLs cho Named Routes

Helper `route` có thể được sử dụng để tạo URL tới một [route đã được đặt tên](/docs/{{version}}/routing#named-routes). Các route đã được đặt tên cho phép bạn tạo URL mà không cần phải biết URL thực tế đang được định nghĩa như thế nào. Do đó, nếu URL của route có thay đổi, thì bạn cũng không cần phải thực hiện thay đổi gì cho các lệnh gọi hàm `route` của bạn. Ví dụ: hãy tưởng tượng application của bạn chứa một route đang được định nghĩa như sau:

    Route::get('/post/{post}', function (Post $post) {
        //
    })->name('post.show');

Để tạo URL tới route này, bạn có thể sử dụng helper `route` như sau:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Dĩ nhiên, helper `route` cũng có thể được sử dụng để tạo URL cho các route có nhiều tham số:

    Route::get('/post/{post}/comment/{comment}', function () {
        //
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

Bất kỳ phần tử bổ sung nào không tương ứng với các tham số định nghĩa trên route sẽ được thêm vào chuỗi truy vấn của URL:

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

<a name="eloquent-models"></a>
#### Eloquent Models

Bạn sẽ thường tạo URL bằng cách sử dụng route key (thường là khóa chính) của [model Eloquent](/docs/{{version}}/eloquent). Vì lý do này, bạn có thể truyền các model Eloquent làm giá trị tham số. Helper `route` sẽ tự động lấy route key của model:

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### Signed URLs

Laravel cho phép bạn dễ dàng tạo các URL "signed" tới các route đã được đặt tên. Các URL này có một "signed" hash được thêm vào sau chuỗi truy vấn cho phép Laravel xác minh được rằng URL sẽ không bị sửa kể từ khi nó được tạo ra. URL signed đặc biệt hữu ích cho các route có thể truy cập công khai nhưng cần thêm một lớp bảo vệ để chống lại việc sửa đổi URL.

Ví dụ: bạn có thể sử dụng các signed URL để tạo link "hủy đăng ký" được gửi qua email cho khách hàng của bạn. Để tạo một signed URL cho một route đã được đặt tên, hãy sử dụng phương thức `signedRoute` trong facade `URL`:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

Nếu bạn muốn tạo một route URL signed tạm thời sau một khoảng thời gian xác định, bạn có thể sử dụng phương thức `temporarySignedRoute`. Khi Laravel xác thực một route URL signed tạm thời, nó sẽ đảm bảo rằng giá trị timestamp hết hạn được mã hóa vào trong URL signed sẽ chưa hết hạn:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

<a name="validating-signed-route-requests"></a>
#### Validating Signed Route Requests

Để xác minh một request có signed hợp lệ hay không, bạn có thể gọi phương thức `hasValidSignature` trên instance `Illuminate\Http\Request` đó:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Thỉnh thoảng, bạn có thể cần cho phép frontend của ứng dụng thêm dữ liệu vào một signed URL, chẳng hạn như khi thực hiện phân trang ở client-side. Do đó, bạn có thể chỉ định các tham số request cần được bỏ qua khi xác thực signed URL bằng phương thức `hasValidSignatureWhileIgnoring`. Hãy nhớ rằng, việc bỏ qua các tham số này sẽ cho phép bất kỳ ai cũng có thể sửa các tham số đó trong request:

    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }

Thay vì xác thực các signed URL bằng cách sử dụng instance request, bạn có thể gán một [middleware](/docs/{{version}}/middleware) `Illuminate\Routing\Middleware\ValidateSignature` cho một route. Nếu bạn chưa đăng ký middleware này, bạn nên gán cho middleware đó một khóa trong mảng `routeMiddleware` của file kernel HTTP của bạn:

    /**
     * The application's route middleware.
     *
     * These middleware may be assigned to groups or used individually.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

Sau khi bạn đã đăng ký xong middleware trong file kernel của bạn, bạn có thể gán nó vào một route. Nếu have không có chữ ký hợp lệ, middleware sẽ tự động trả về HTTP response `403`:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="responding-to-invalid-signed-routes"></a>
#### Responding To Invalid Signed Routes

Khi ai đó truy cập một URL signed đã hết hạn, họ sẽ nhận được một trang lỗi chung có mã trạng thái HTTP `403`. Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách định nghĩa một closure "renderable" tùy chỉnh cho exception `InvalidSignatureException` trong quá trình xử lý exception của bạn. Closure này sẽ trả về một HTTP response:

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (InvalidSignatureException $e) {
            return response()->view('error.link-expired', [], 403);
        });
    }

<a name="urls-for-controller-actions"></a>
## URLs cho Controller Actions

Hàm `action` giúp tạo ra một URL cho một controller action:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Nếu phương thức controller yêu cầu truyền một route parameter, bạn có thể truyền một mảng các tham số route làm tham số thứ hai cho hàm như sau:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>
## Giá trị mặc định

Đối với một số application, bạn có thể muốn định nghĩa các giá trị mặc định cho các tham số URL trong toàn bộ request. Ví dụ: hãy tưởng tượng nhiều route của bạn định nghĩa tham số `{locale}`:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

Sẽ thật là cồng kềnh khi luôn luôn phải truyền một tham số `locale` mỗi khi bạn gọi helper `route`. Vì vậy, bạn có thể sử dụng phương thức `URL::defaults` để định nghĩa một giá trị mặc định cho tham số này và lúc nào cũng được áp dụng trong request hiện tại. Bạn có thể muốn gọi phương thức này từ [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) để bạn có quyền truy cập vào request hiện tại:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        /**
         * Handle the incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return \Illuminate\Http\Response
         */
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

Khi giá trị mặc định cho tham số `locale` đã được cài đặt, bạn sẽ không cần phải truyền giá trị của nó khi tạo URL thông qua helper `route`.

<a name="url-defaults-middleware-priority"></a>
#### URL Defaults & Middleware Priority

Việc set giá trị mặc định của URL có thể cản trở việc xử lý các liên kết ngầm model của Laravel. Do đó, bạn nên [ưu tiên middleware của bạn](/docs/{{version}}/middleware#sorting-middleware) về set mặc định URL được chạy trước middleware `SubstituteBindings` của Laravel. Bạn có thể thực hiện điều này bằng cách đưa middleware của bạn lên trước middleware `SubstituteBindings` trong thuộc tính `$middlewarePriority` của HTTP kernel của ứng dụng của bạn.

Thuộc tính `$middlewarePriority` được định nghĩa trong class base `Illuminate\Foundation\Http\Kernel`. Bạn có thể copy định nghĩa của nó từ trong class đó và ghi đè nó trong HTTP kernel của ứng dụng để thay đổi nó:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var array
     */
    protected $middlewarePriority = [
        // ...
         \App\Http\Middleware\SetDefaultLocaleForUrls::class,
         \Illuminate\Routing\Middleware\SubstituteBindings::class,
         // ...
    ];
