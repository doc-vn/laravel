# Laravel Folio

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Page Paths và URIs](#page-paths-uris)
    - [Subdomain Routing](#subdomain-routing)
- [Tạo Routes](#creating-routes)
    - [Route lồng nhau](#nested-routes)
    - [Index Routes](#index-routes)
- [Route Parameters](#route-parameters)
- [Route Model Binding](#route-model-binding)
    - [Soft Deleted Models](#soft-deleted-models)
- [Tạo Hooks](#render-hooks)
- [Named Routes](#named-routes)
- [Middleware](#middleware)
- [Route Caching](#route-caching)

<a name="introduction"></a>
## Giới thiệu

[Laravel Folio](https://github.com/laravel/folio) là một bộ router mạnh mẽ được thiết kế để đơn giản hóa việc routing trong các ứng dụng Laravel. Với Laravel Folio, việc tạo một route trở nên dễ dàng như tạo một template Blade trong thư mục `resources/views/pages` của ứng dụng.

Ví dụ, để tạo một trang có thể truy cập tại địa chỉ URL `/greeting`, chỉ cần tạo file `greeting.blade.php` trong thư mục `resources/views/pages` của ứng dụng:

```php
<div>
    Hello World
</div>
```

<a name="installation"></a>
## Cài đặt

Để bắt đầu, hãy cài đặt Folio vào project của bạn bằng trình quản lý package Composer:

```bash
composer require laravel/folio
```

Sau khi cài đặt xong Folio, bạn có thể chạy lệnh Artisan `folio:install`, để cài đặt service provider của Folio vào ứng dụng của bạn. Service provider này sẽ đăng ký một thư mục mới nơi folio sẽ tìm kiếm các route / page:

```bash
php artisan folio:install
```

<a name="page-paths-uris"></a>
### Page Paths và URIs

Mặc định, Folio sẽ chạy các page từ thư mục `resources/views/pages` của ứng dụng, nhưng bạn có thể tùy chỉnh các thư mục này trong phương thức `boot` của service provider của folio.

Ví dụ, bạn đôi khi có thể cảm thấy thuận tiện khi chỉ định nhiều path folio trong cùng một ứng dụng Laravel. Bạn có thể muốn có một thư mục riêng để chứa các trang folio cho phần "admin" của ứng dụng, trong khi sử dụng một thư mục khác cho phần còn lại của các trang ứng dụng của bạn.

Bạn có thể thực hiện điều này bằng cách sử dụng các phương thức `Folio::path` và phương thức `Folio::uri`. Phương thức `path` sẽ đăng ký một thư mục mà folio sẽ quét các trang ở trong đó khi đang chuyển hướng các request HTTP đến, trong khi phương thức `uri` chỉ định một "URI base" cho thư mục của các trang đó:

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages/guest'))->uri('/');

Folio::path(resource_path('views/pages/admin'))
    ->uri('/admin')
    ->middleware([
        '*' => [
            'auth',
            'verified',

            // ...
        ],
    ]);
```

<a name="subdomain-routing"></a>
### Subdomain Routing

Bạn cũng có thể route đến các trang dựa trên tên subdomain của request đến. Ví dụ, bạn có thể muốn route các request từ `admin.example.com` đến một thư mục trang khác, khác với các trang Folio còn lại của bạn. Bạn có thể thực hiện điều này bằng cách gọi phương thức `domain` sau khi gọi phương thức `Folio::path`:

```php
use Laravel\Folio\Folio;

Folio::domain('admin.example.com')
    ->path(resource_path('views/pages/admin'));
```

Phương thức `domain` cũng cho phép bạn khai báo các phần của domain hoặc subdomain dưới dạng các tham số. Các tham số này sẽ được đưa vào template trang của bạn:

```php
use Laravel\Folio\Folio;

Folio::domain('{account}.example.com')
    ->path(resource_path('views/pages/admin'));
```

<a name="creating-routes"></a>
## Tạo Routes

Bạn có thể tạo route Folio bằng cách đặt một template Blade vào bất kỳ thư mục nào đã được gắn Folio của bạn. Mặc định, Folio sẽ gắn thư mục `resources/views/pages`, nhưng bạn có thể tùy chỉnh các thư mục này trong phương thức `boot` của service provider của Folio.

Sau khi template Blade đã được đặt vào trong thư mục có gắn với Folio, bạn có thể truy cập ngay lập tức vào nó thông qua trình duyệt của bạn. Ví dụ, một trang được đặt trong `pages/schedule.blade.php` có thể được truy cập trong trình duyệt của bạn tại `http://example.com/schedule`.

Để xem nhanh danh sách tất cả các page / route Folio của bạn, bạn có thể gọi lệnh Artisan `folio:list`:

```bash
php artisan folio:list
```

<a name="nested-routes"></a>
### Route lồng nhau

Bạn có thể tạo một route lồng nhau bằng cách tạo một hoặc nhiều thư mục trong một hoặc nhiều trong một thư mục khác của Folio. Ví dụ, để tạo một trang có thể truy cập qua `/user/profile`, hãy tạo một template `profile.blade.php` trong thư mục `pages/user`:

```bash
php artisan make:folio user/profile

# pages/user/profile.blade.php → /user/profile
```

<a name="index-routes"></a>
### Index Routes

Thỉnh thoảng, bạn có thể muốn tạo một trang thành "mặc định" của một thư mục. Bằng cách đặt một template `index.blade.php` vào trong một thư mục Folio, mọi request đến thư mục gốc của thư mục đó sẽ được route ngay đến trang đó:

```bash
php artisan make:folio index
# pages/index.blade.php → /

php artisan make:folio users/index
# pages/users/index.blade.php → /users
```

<a name="route-parameters"></a>
## Route Parameters

Thông thường, bạn sẽ cần phải có các tham số URL của request được đưa vào trang của bạn để bạn có thể tương tác với chúng. Ví dụ, bạn có thể cần lấy "ID" của người dùng có profile đang được hiển thị. Để thực hiện điều này, bạn có thể khai báo một tham số của trang trong dấu ngoặc vuông:

```bash
php artisan make:folio "users/[id]"

# pages/users/[id].blade.php → /users/1
```

Các tham số đã được khai báo thể được truy cập dưới dạng biến trong template Blade của bạn:

```html
<div>
    User {{ $id }}
</div>
```

Để khai báo nhiều tham số, bạn có thể thêm tiền tố vào tham số được khai báo bằng ba dấu chấm `...`:

```bash
php artisan make:folio "users/[...ids]"

# pages/users/[...ids].blade.php → /users/1/2/3
```

Khi khai báo nhiều tham số, các tham số được khai báo sẽ được đưa vào trang dưới dạng một mảng:

```html
<ul>
    @foreach ($ids as $id)
        <li>User {{ $id }}</li>
    @endforeach
</ul>
```

<a name="route-model-binding"></a>
## Route Model Binding

Nếu một tham số wildcard trong template trang của bạn tương ứng với một trong các model Eloquent của ứng dụng, Folio sẽ tự động tận dụng khả năng liên kết model route của Laravel và sẽ thử tích hợp instance model đó vào trong trang của bạn:

```bash
php artisan make:folio "users/[User]"

# pages/users/[User].blade.php → /users/1
```

Có thể truy cập vào các model đã được tích hợp dưới dạng các biến trong template Blade của bạn. Tên biến của model sẽ được chuyển thành kiểu "camel case":

```html
<div>
    User {{ $user->id }}
</div>
```

#### Customizing the Key

Thỉnh thoảng bạn có thể muốn resolve các model Eloquent bị ràng buộc bằng một cột khác, khác cột `id`. Để làm như vậy, bạn có thể chỉ định một cột trong filename của trang. Ví dụ, một trang có filename là: `[Post:slug].blade.php` sẽ cố gắng resolve model bị ràng buộc thông qua cột `slug` thay vì cột `id`.

Trên Windows, bạn nên sử dụng ký tự `-` để tách tên model ra khỏi khóa: `[Post-slug].blade.php`.

#### Model Location

Mặc định, Folio sẽ tìm kiếm model của bạn trong thư mục `app/Models` của ứng dụng. Tuy nhiên, nếu cần, bạn có thể chỉ định tên class model đủ điều kiện trong filename template của bạn:

```bash
php artisan make:folio "users/[.App.Models.User]"

# pages/users/[.App.Models.User].blade.php → /users/1
```

<a name="soft-deleted-models"></a>
### Soft Deleted Models

Mặc định, các model đã bị soft delete sẽ không được lấy ra khi resolve các liên kết model ẩn. Tuy nhiên, nếu muốn, bạn có thể hướng dẫn Folio lấy ra các model đã bị soft delete bằng cách gọi hàm `withTrashed` trong template của trang:

```php
<?php

use function Laravel\Folio\{withTrashed};

withTrashed();

?>

<div>
    User {{ $user->id }}
</div>
```

<a name="render-hooks"></a>
## Tạo Hooks

Mặc định, Folio sẽ trả về nội dung template Blade của trang dưới dạng response cho request đến. Tuy nhiên, bạn có thể tùy chỉnh response này bằng cách gọi hàm `render` trong template của trang.

Hàm `render` sẽ chấp nhận một closure nhận vào một instance `View` đang được Folio render, cho phép bạn thêm dữ liệu vào view hoặc tùy chỉnh toàn bộ response. Ngoài việc nhận vào một instance `View`, thì bất kỳ tham số route nào hoặc bất kỳ ràng buộc model nào cũng sẽ được cung cấp cho closure `render`:

```php
<?php

use App\Models\Post;
use Illuminate\Support\Facades\Auth;
use Illuminate\View\View;

use function Laravel\Folio\render;

render(function (View $view, Post $post) {
    if (! Auth::user()->can('view', $post)) {
        return response('Unauthorized', 403);
    }

    return $view->with('photos', $post->author->photos);
}); ?>

<div>
    {{ $post->content }}
</div>

<div>
    This author has also taken {{ count($photos) }} photos.
</div>
```

<a name="named-routes"></a>
## Named Routes

Bạn có thể chỉ định tên route của một trang bằng cách sử dụng hàm `name`:

```php
<?php

use function Laravel\Folio\name;

name('users.index');
```

Giống như cách đặt tên route của Laravel, bạn có thể sử dụng hàm `route` để tạo URL tới các trang Folio đã được đặt tên:

```php
<a href="{{ route('users.index') }}">
    All Users
</a>
```

Nếu trang đó có tham số cần truyền, bạn có thể chỉ các giá trị cần truyền đó thông qua hàm `route`:

```php
route('users.show', ['user' => $user]);
```

<a name="middleware"></a>
## Middleware

Bạn có thể áp dụng middleware cho một trang cụ thể bằng cách gọi hàm `middleware` trong template của trang:

```php
<?php

use function Laravel\Folio\{middleware};

middleware(['auth', 'verified']);

?>

<div>
    Dashboard
</div>
```

Hoặc, để gán một middleware cho một group trang, bạn có thể thêm phương thức `middleware` vào sau khi gọi phương thức `Folio::path`.

Để chỉ định những trang nào mà middleware sẽ được áp dụng, mảng middleware có thể được định dạng bằng cách sử dụng các pattern URL tương ứng của các trang mà chúng sẽ được áp dụng. Ký tự `*` có thể được sử dụng như là một ký tự đại diện:

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        // ...
    ],
]);
```

Bạn có thể thêm các closure vào trong mảng middleware để định nghĩa thêm các middleware ẩn một cách trực tiếp:

```php
use Closure;
use Illuminate\Http\Request;
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        function (Request $request, Closure $next) {
            // ...

            return $next($request);
        },
    ],
]);
```

<a name="route-caching"></a>
## Route Caching

Khi sử dụng Folio, bạn cần luôn tận dụng [khả năng lưu trữ cache route của Laravel](/docs/{{version}}/routing#route-caching). Folio sẽ lắng nghe lệnh Artisan `route:cache` để đảm bảo rằng các định nghĩa trang Folio và tên route sẽ được lưu cache đúng cách để có hiệu suất tốt nhất.
