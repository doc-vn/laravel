# URL Generation

- [Giới thiệu](#introduction)
- [Cơ bản](#the-basics)
    - [Tạo một URL](#generating-urls)
    - [Truy cập vào URL hiện tại](#accessing-the-current-url)
- [URLs cho Named Routes](#urls-for-named-routes)
    - [Signed URLs](#signed-urls)
- [URLs cho Controller Actions](#urls-for-controller-actions)
- [Fluent URI Objects](#fluent-uri-objects)
- [Giá trị mặc định](#default-values)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một số helper để hỗ trợ bạn tạo URL cho application của bạn. Những helper này chủ yếu hữu ích khi tạo link trong các template và API response hoặc khi tạo response chuyển hướng đến một phần khác trong application của bạn.

<a name="the-basics"></a>
## Cơ bản

<a name="generating-basic-urls"></a>
### Tạo một URL

Helper `url` có thể được sử dụng để tạo các URL tùy biến cho application của bạn. URL được tạo ra sẽ tự động sử dụng scheme (HTTP hoặc HTTPS) và host từ request hiện tại đang được xử lý bởi ứng dụng:

```php
$post = App\Models\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

Để tạo ra một URL với các tham số query string, bạn có thể sử dụng phương thức `query`:

```php
echo url()->query('/posts', ['search' => 'Laravel']);

// https://example.com/posts?search=Laravel

echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);

// http://example.com/posts?sort=latest&search=Laravel
```

Việc cung cấp các tham số query string đã tồn tại trong path sẽ ghi đè lên giá trị hiện có của chúng:

```php
echo url()->query('/posts?sort=latest', ['sort' => 'oldest']);

// http://example.com/posts?sort=oldest
```

Các mảng giá trị cũng có thể được truyền dưới dạng tham số query string. Các giá trị này sẽ được gán key và mã hóa đúng cách trong URL được tạo ra:

```php
echo $url = url()->query('/posts', ['columns' => ['title', 'body']]);

// http://example.com/posts?columns%5B0%5D=title&columns%5B1%5D=body

echo urldecode($url);

// http://example.com/posts?columns[0]=title&columns[1]=body
```

<a name="accessing-the-current-url"></a>
### Truy cập vào URL hiện tại

Nếu như không có đường dẫn nào được truyền vào cho helper `url`, thì một instance `Illuminate\Routing\UrlGenerator` sẽ được trả về, cho phép bạn truy cập vào thông tin về URL hiện tại:

```php
// Get the current URL without the query string...
echo url()->current();

// Get the current URL including the query string...
echo url()->full();
```

Mỗi phương thức này cũng có thể được truy cập thông qua facade [URL](/docs/{{version}}/facades):

```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

<a name="accessing-the-previous-url"></a>
#### Accessing the Previous URL

Thỉnh thoảng việc biết URL trước đó mà người dùng vừa truy cập sẽ rất hữu ích. Bạn có thể truy cập vào URL trước đó thông qua các phương thức `previous` và `previousPath` của helper `url`:

```php
// Get the full URL for the previous request...
echo url()->previous();

// Get the path for the previous request...
echo url()->previousPath();
```

Hoặc, thông qua [session](/docs/{{version}}/session), bạn có thể truy cập vào URL trước đó dưới dạng một instance [URI](#fluent-uri-objects):

```php
use Illuminate\Http\Request;

Route::post('/users', function (Request $request) {
    $previousUri = $request->session()->previousUri();

    // ...
});
```

Bạn cũng có thể lấy tên route của URL đã truy cập trước đó thông qua session:

```php
$previousRoute = $request->session()->previousRoute();
```

<a name="urls-for-named-routes"></a>
## URLs cho Named Routes

Helper `route` có thể được sử dụng để tạo URL tới một [route đã được đặt tên](/docs/{{version}}/routing#named-routes). Các route đã được đặt tên cho phép bạn tạo URL mà không cần phải biết URL thực tế đang được định nghĩa như thế nào. Do đó, nếu URL của route có thay đổi, thì bạn cũng không cần phải thực hiện thay đổi gì cho các lệnh gọi hàm `route` của bạn. Ví dụ: hãy tưởng tượng application của bạn chứa một route đang được định nghĩa như sau:

```php
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');
```

Để tạo URL tới route này, bạn có thể sử dụng helper `route` như sau:

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

Dĩ nhiên, helper `route` cũng có thể được sử dụng để tạo URL cho các route có nhiều tham số:

```php
Route::get('/post/{post}/comment/{comment}', function () {
    // ...
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.com/post/1/comment/3
```

Bất kỳ phần tử bổ sung nào không tương ứng với các tham số định nghĩa trên route sẽ được thêm vào chuỗi truy vấn của URL:

```php
echo route('post.show', ['post' => 1, 'search' => 'rocket']);

// http://example.com/post/1?search=rocket
```

<a name="eloquent-models"></a>
#### Eloquent Models

Bạn sẽ thường tạo URL bằng cách sử dụng route key (thường là khóa chính) của [model Eloquent](/docs/{{version}}/eloquent). Vì lý do này, bạn có thể truyền các model Eloquent làm giá trị tham số. Helper `route` sẽ tự động lấy route key của model:

```php
echo route('post.show', ['post' => $post]);
```

<a name="signed-urls"></a>
### Signed URLs

Laravel cho phép bạn dễ dàng tạo các URL "signed" tới các route đã được đặt tên. Các URL này có một "signed" hash được thêm vào sau chuỗi truy vấn cho phép Laravel xác minh được rằng URL sẽ không bị sửa kể từ khi nó được tạo ra. URL signed đặc biệt hữu ích cho các route có thể truy cập công khai nhưng cần thêm một lớp bảo vệ để chống lại việc sửa đổi URL.

Ví dụ: bạn có thể sử dụng các signed URL để tạo link "hủy đăng ký" được gửi qua email cho khách hàng của bạn. Để tạo một signed URL cho một route đã được đặt tên, hãy sử dụng phương thức `signedRoute` trong facade `URL`:

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

Bạn có thể bỏ tên miền ra khỏi signed URL hash bằng cách cung cấp tham số `absolute` cho phương thức `signedRoute`:

```php
return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
```

Nếu bạn muốn tạo một route URL signed tạm thời sau một khoảng thời gian xác định, bạn có thể sử dụng phương thức `temporarySignedRoute`. Khi Laravel xác thực một route URL signed tạm thời, nó sẽ đảm bảo rằng giá trị timestamp hết hạn được mã hóa vào trong URL signed sẽ chưa hết hạn:

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

<a name="validating-signed-route-requests"></a>
#### Validating Signed Route Requests

Để xác minh một request có signed hợp lệ hay không, bạn có thể gọi phương thức `hasValidSignature` trên instance `Illuminate\Http\Request` đó:

```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```

Thỉnh thoảng, bạn có thể cần cho phép frontend của ứng dụng thêm dữ liệu vào một signed URL, chẳng hạn như khi thực hiện phân trang ở client-side. Do đó, bạn có thể chỉ định các tham số request cần được bỏ qua khi xác thực signed URL bằng phương thức `hasValidSignatureWhileIgnoring`. Hãy nhớ rằng, việc bỏ qua các tham số này sẽ cho phép bất kỳ ai cũng có thể sửa các tham số đó trong request:

```php
if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
    abort(401);
}
```

Thay vì xác thực các signed URL bằng cách sử dụng instance request, bạn có thể gán một [middleware](/docs/{{version}}/middleware) `signed` (`Illuminate\Routing\Middleware\ValidateSignature`) cho một route. Nếu request đến không có chữ ký hợp lệ, middleware sẽ tự động trả về HTTP response `403`:

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

Nếu trong signed URL của bạn không chứa tên miền trong URL hash, bạn nên cung cấp tham số `relative` cho middleware:

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed:relative');
```

<a name="responding-to-invalid-signed-routes"></a>
#### Responding To Invalid Signed Routes

Khi ai đó truy cập một URL signed đã hết hạn, họ sẽ nhận được một trang lỗi chung có mã trạng thái HTTP `403`. Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách định nghĩa một closure "render" tùy chỉnh cho exception `InvalidSignatureException` trong file `bootstrap/app.php` của ứng dụng của bạn:

```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (InvalidSignatureException $e) {
        return response()->view('errors.link-expired', status: 403);
    });
})
```

<a name="urls-for-controller-actions"></a>
## URLs cho Controller Actions

Hàm `action` giúp tạo ra một URL cho một controller action:

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

Nếu phương thức controller yêu cầu truyền một route parameter, bạn có thể truyền một mảng các tham số route làm tham số thứ hai cho hàm như sau:

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

<a name="fluent-uri-objects"></a>
## Fluent URI Objects

Class `Uri` của Laravel cung cấp một interface tiện lợi và rõ ràng để tạo và thao tác với các URI thông qua các đối tượng. Class này chứa các chức năng được cung cấp bởi package League URI và tích hợp mượt mà với hệ thống routing của Laravel.

Bạn có thể tạo một instance `Uri` dễ dàng bằng các phương thức static:

```php
use App\Http\Controllers\UserController;
use App\Http\Controllers\InvokableController;
use Illuminate\Support\Uri;

// Generate a URI instance from the given string...
$uri = Uri::of('https://example.com/path');

// Generate URI instances to paths, named routes, or controller actions...
$uri = Uri::to('/dashboard');
$uri = Uri::route('users.show', ['user' => 1]);
$uri = Uri::signedRoute('users.show', ['user' => 1]);
$uri = Uri::temporarySignedRoute('user.index', now()->plus(minutes: 5));
$uri = Uri::action([UserController::class, 'index']);
$uri = Uri::action(InvokableController::class);

// Generate a URI instance from the current request URL...
$uri = $request->uri();

// Generate a URI instance from the previous request URL...
$uri = $request->session()->previousUri();
```

Once you have a URI instance, you can fluently modify it:

```php
$uri = Uri::of('https://example.com')
    ->withScheme('http')
    ->withHost('test.com')
    ->withPort(8000)
    ->withPath('/users')
    ->withQuery(['page' => 2])
    ->withFragment('section-1');
```

For more information on working with fluent URI objects, consult the [URI documentation](/docs/{{version}}/helpers#uri).

<a name="default-values"></a>
## Giá trị mặc định

Đối với một số application, bạn có thể muốn định nghĩa các giá trị mặc định cho các tham số URL trong toàn bộ request. Ví dụ: hãy tưởng tượng nhiều route của bạn định nghĩa tham số `{locale}`:

```php
Route::get('/{locale}/posts', function () {
    // ...
})->name('post.index');
```

Sẽ thật là cồng kềnh khi luôn luôn phải truyền một tham số `locale` mỗi khi bạn gọi helper `route`. Vì vậy, bạn có thể sử dụng phương thức `URL::defaults` để định nghĩa một giá trị mặc định cho tham số này và lúc nào cũng được áp dụng trong request hiện tại. Bạn có thể muốn gọi phương thức này từ [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) để bạn có quyền truy cập vào request hiện tại:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\URL;
use Symfony\Component\HttpFoundation\Response;

class SetDefaultLocaleForUrls
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```

Khi giá trị mặc định cho tham số `locale` đã được cài đặt, bạn sẽ không cần phải truyền giá trị của nó khi tạo URL thông qua helper `route`.

<a name="url-defaults-middleware-priority"></a>
#### URL Defaults và Middleware Priority

Việc set giá trị mặc định của URL có thể cản trở việc xử lý các liên kết ngầm model của Laravel. Do đó, bạn nên [ưu tiên middleware của bạn](/docs/{{version}}/middleware#sorting-middleware) về set mặc định URL được chạy trước middleware `SubstituteBindings` của Laravel. Bạn có thể thực hiện điều này bằng cách sử dụng phương thức `priority` middleware method trong file `bootstrap/app.php` của ứng dụng của bạn:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->prependToPriorityList(
        before: \Illuminate\Routing\Middleware\SubstituteBindings::class,
        prepend: \App\Http\Middleware\SetDefaultLocaleForUrls::class,
    );
})
```
