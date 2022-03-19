# URL Generation

- [Giới thiệu](#introduction)
- [Cơ bản](#the-basics)
    - [Tạo một URL cơ bản](#generating-basic-urls)
    - [Truy cập vào URL hiện tại](#accessing-the-current-url)
- [URLs cho Named Routes](#urls-for-named-routes)
    - [Signed URLs](#signed-urls)
- [URLs cho Controller Actions](#urls-for-controller-actions)
- [Giá trị mặc định](#default-values)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một số helper để hỗ trợ bạn tạo URL cho application của bạn. Những điều này chủ yếu hữu ích khi tạo link trong các template và API response hoặc khi tạo response chuyển hướng đến một phần khác trong application của bạn.

<a name="the-basics"></a>
## Cơ bản

<a name="generating-basic-urls"></a>
### Tạo một URL cơ bản

Helper `url` có thể được sử dụng để tạo các URL tùy biến cho application của bạn. URL được tạo ra sẽ tự động sử dụng scheme (HTTP hoặc HTTPS) và host từ request hiện tại:

    $post = App\Post::find(1);

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

Helper `route` có thể được sử dụng để tạo URL tới một route đã được đặt tên. Các route đã được đặt tên cho phép bạn tạo URL mà không cần phải biết URL thực tế đang được định nghĩa như thế nào. Do đó, nếu URL của route có thay đổi, thì bạn cũng không cần phải thực hiện thay đổi gì cho các lệnh gọi hàm `route` của bạn. Ví dụ: hãy tưởng tượng application của bạn chứa một route đang được định nghĩa như sau:

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');

Để tạo URL tới route này, bạn có thể sử dụng helper `route` như sau:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Bạn thường sẽ phải tạo URL bằng primary key của [Eloquent models](/docs/{{version}}/eloquent). Vì lý do đó, bạn có thể truyền trực tiếp các model Eloquent làm giá trị tham số. Helper `route` sẽ tự động lấy primary key trong model đó ra:

    echo route('post.show', ['post' => $post]);

Helper `route` cũng có thể được sử dụng để tạo URL cho các route có nhiều tham số:

    Route::get('/post/{post}/comment/{comment}', function () {
        //
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

<a name="signed-urls"></a>
### Signed URLs

Laravel cho phép bạn dễ dàng tạo các URL "signed" tới các route đã được đặt tên. Các URL này có một "signed" hash được thêm vào sau chuỗi truy vấn cho phép Laravel xác minh được rằng URL sẽ không bị sửa kể từ khi nó được tạo ra. URL signed đặc biệt hữu ích cho các route có thể truy cập công khai nhưng cần thêm một lớp bảo vệ để chống lại việc sửa đổi URL.

Ví dụ: bạn có thể sử dụng các signed URL để tạo link "hủy đăng ký" được gửi qua email cho khách hàng của bạn. Để tạo một signed URL cho một route đã được đặt tên, hãy sử dụng phương thức `signedRoute` trong facade `URL`:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

Nếu bạn muốn tạo một route URL signed tạm thời, bạn có thể sử dụng phương thức `temporarySignedRoute`:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

#### Validating Signed Route Requests

Để xác minh một request có signed hợp lệ hay không, bạn có thể gọi phương thức `hasValidSignature` trên `Request` đó:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Ngoài ra, bạn có thể gán một middleware `Illuminate\Routing\Middleware\ValidateSignature` cho một route. Nếu bạn chưa đăng ký middleware này, bạn nên gán cho middleware đó một khóa trong mảng `routeMiddleware` của file kernel HTTP của bạn:

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

Sau khi bạn đã đăng ký xong middleware trong file kernel của bạn, bạn có thể gán nó vào một route. Nếu have không có chữ ký hợp lệ, middleware sẽ tự động trả về response lỗi `403`:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="urls-for-controller-actions"></a>
## URLs cho Controller Actions

Hàm `action` giúp tạo ra một URL cho một controller action. Bạn không cần phải truyền toàn bộ namespace của controller đó vào. Mà thay vào đó, chỉ cần truyền tên class của controller mà được liên kết với namespace `App\Http\Controllers`:

    $url = action('HomeController@index');

Bạn cũng có thể tham chiếu đến các action với cú pháp mảng:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Nếu phương thức controller yêu cầu truyền một route parameter, bạn có thể truyền chúng làm tham số thứ hai cho hàm như sau:

    $url = action('UserController@profile', ['id' => 1]);

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
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

Khi giá trị mặc định cho tham số `locale` đã được cài đặt, bạn sẽ không cần phải truyền giá trị của nó khi tạo URL thông qua helper `route`.
