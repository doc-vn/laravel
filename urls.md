# URL Generation

- [Giới thiệu](#introduction)
- [Cơ bản](#the-basics)
    - [Tạo một URL cơ bản](#generating-basic-urls)
    - [Truy cập vào URL hiện tại](#accessing-the-current-url)
- [URLs cho Named Routes](#urls-for-named-routes)
- [URLs cho Controller Actions](#urls-for-controller-actions)
- [Giá trị mặc định](#default-values)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một số helper để hỗ trợ bạn tạo URL cho application của bạn. Tất nhiên, những điều này chủ yếu hữu ích khi build link trong các template và API response của bạn hoặc khi tạo response redirect đến một phần khác trong application của bạn.

<a name="the-basics"></a>
## Cơ bản

<a name="generating-basic-urls"></a>
### Tạo một URL cơ bản

Helper `url` có thể được sử dụng để tạo các URL tùy ý cho application của bạn. URL được tạo sẽ tự động sử dụng scheme (HTTP hoặc HTTPS) và host từ request hiện tại:

    $post = App\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Truy cập vào URL hiện tại

Nếu như không có đường dẫn nào được cung cấp cho helper `url`, thì một instance `Illuminate\Routing\UrlGenerator` sẽ được trả về, cho phép bạn truy cập thông tin về URL hiện tại:

    // Get the current URL without the query string...
    echo url()->current();

    // Get the current URL including the query string...
    echo url()->full();

    // Get the full URL for the previous request...
    echo url()->previous();

Mỗi phương thức này cũng có thể được truy cập thông qua [facade](/docs/{{version}}/facades) `URL`:

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URLs cho Named Routes

Helper `route` có thể được sử dụng để tạo URL tới một route đã được đặt tên. Các route được đặt tên cho phép bạn tạo URL mà không cần thiết phải ghép nối với URL thực tế được xác định trong route. Do đó, nếu URL của route có thay đổi, thì cũng không cần thực hiện thay đổi nào cho các lệnh gọi hàm `route` của bạn. Ví dụ: hãy tưởng tượng application của bạn chứa một route được xác định như sau:

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');

Để tạo URL tới route này, bạn có thể sử dụng helper `route` như sau:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Bạn thường sẽ phải tạo URL bằng kprimary key của [Eloquent models](/docs/{{version}}/eloquent). Vì lý do đó, bạn có thể pass các model Eloquent làm giá trị tham số. helper `route` sẽ tự động lấy primary key trong model ra:

    echo route('post.show', ['post' => $post]);

<a name="urls-for-controller-actions"></a>
## URLs cho Controller Actions

Hàm `action` giúp tạo ra một URL cho controller action. Bạn không cần phải pass toàn bộ namespace của controller. Thay vào đó, chỉ pass tên class của controller mà được liên kết với namespace `App\Http\Controllers`:

    $url = action('HomeController@index');

Nếu phương thức controller chấp nhận route parameter, bạn có thể pass chúng làm tham số thứ hai cho hàm:

    $url = action('UserController@profile', ['id' => 1]);

<a name="default-values"></a>
## Giá trị mặc định

Đối với một số application, bạn có thể muốn định nghĩa các giá trị mặc định cho các tham số URL nhất định trong toàn bộ request. Ví dụ: hãy tưởng tượng nhiều route của bạn định nghĩa tham số `{locale}`:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

Sẽ thật là cồng kềnh khi luôn luôn pass một tham sô `locale` mỗi khi bạn gọi helper `route`. Vì vậy, bạn có thể sử dụng phương thức `URL::defaults` để định nghĩa một giá trị mặc định cho tham số này và lúc nào cũng được áp dụng trong request hiện tại. Bạn có thể muốn gọi phương thức này từ [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) để bạn có quyền truy cập vào request hiện tại:

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

Khi giá trị mặc định cho tham số `locale` đã được set, bạn không còn cần phải truyền giá trị của nó khi tạo URL thông qua helper `route`.
