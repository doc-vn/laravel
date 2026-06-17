# CSRF Protection

- [Giới thiệu](#csrf-introduction)
- [Chặn request CSRF](#preventing-csrf-requests)
    - [Xác minh origin](#origin-verification)
    - [Loại bỏ các URI khỏi CSRF Protection](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Giới thiệu

Giả mạo request trên nhiều trang là một loại khai thác mã độc theo đó các lệnh trái phép được thực hiện thay cho người dùng đã được xác thực. Rất may, Laravel giúp dễ dàng bảo vệ ứng dụng của bạn khỏi các cuộc tấn công [giả mạo request trên nhiều trang web](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF).

<a name="csrf-explanation"></a>
#### An Explanation Of The Vulnerability

Trong trường hợp bạn không quen với việc giả mạo request trên nhiều trang web, hãy thảo luận về một ví dụ về cách thức mà lỗ hổng này có thể bị khai thác. Hãy tưởng tượng ứng dụng của bạn có một route `/user/email` và chấp nhận request method là `POST` để cập nhật địa chỉ email của người dùng đã được xác thực. Rất có thể, route này sẽ có trường `email` chứa địa chỉ email mà người dùng muốn bắt đầu sử dụng.

Nếu không có bảo vệ CSRF, một trang web độc hại có thể tạo ra một form HTML trỏ đến route `/user/email` của ứng dụng của bạn và gửi địa chỉ email của chính kẻ tấn công:

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

Nếu trang web độc hại này tự động gửi form đó mỗi khi trang được load, thì kẻ tấn công chỉ cần thu hút người dùng ứng dụng của bạn truy cập vào trang web của kẻ đó và địa chỉ email của người dùng của bạn sẽ được thay đổi ngay trong chính ứng dụng của bạn.

Để ngăn chặn lỗ hổng này, chúng ta cần phải kiểm tra mọi request `POST`, `PUT`, `PATCH` hoặc `DELETE` với một giá trị secret session mà ứng dụng độc hại sẽ không thể truy cập được.

<a name="preventing-csrf-requests"></a>
## Ngăn request CSRF

Mặc định, [middleware](/docs/{{version}}/middleware) `Illuminate\Foundation\Http\Middleware\PreventRequestForgery`, đã có trong group middleware `web`, nó sẽ bảo vệ ứng dụng của bạn khỏi các hành vi giả mạo cross-site request bằng cách sử dụng phương pháp tiếp cận hai lớp.

Đầu tiên, middleware sẽ kiểm tra header `Sec-Fetch-Site` của trình duyệt. Các trình duyệt hiện đại sẽ tự động set header này cho mỗi request, cho biết nó bắt nguồn từ cùng một origin, cùng một trang web hay là một nguồn cross-site khác. Nếu header này cho biết request đến từ cùng một origin, yêu cầu đó sẽ được cho phép ngay lập tức mà không cần bất kỳ xác minh token nào.

Nếu việc xác minh origin không thành công — ví dụ: vì request đến từ một trình duyệt cũ hơn không gửi header `Sec-Fetch-Site` hoặc vì kết nối không được bảo mật — middleware sẽ quay lại xác thực token CSRF như truyền thống.

Laravel sẽ tự động tạo một "token" CSRF cho mỗi [session người dùng](/docs/{{version}}/session) đang hoạt động do ứng dụng quản lý. Token này được sử dụng để xác minh rằng người dùng đang được xác thực là người thực sự đưa ra request cho ứng dụng. Vì token này sẽ được lưu trữ trong session của người dùng và thay đổi mỗi khi session được tạo lại nên một ứng dụng độc hại sẽ không thể truy cập token này.

Token CSRF của session hiện tại có thể được truy cập thông qua session của request hoặc thông qua function helper `csrf_token`:

```php
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

Bất cứ khi nào bạn định nghĩa một HTML form "POST", "PUT", "PATCH", hoặc "DELETE" nào trong ứng dụng của bạn, bạn nên tạo một field hidden chứa mã CSRF `_token` để middleware protection CSRF có thể kiểm tra request đó. Để thuận tiện, Bạn có thể sử dụng lệnh `@csrf` của Blade để tạo field hidden chứa mã token đó:

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

<a name="csrf-tokens-and-spas"></a>
#### CSRF Tokens và SPAs

Nếu bạn đang xây dựng một SPA đang sử dụng Laravel làm backend API, bạn nên tham khảo [tài liệu về Laravel Sanctum](/docs/{{version}}/sanctum) để biết thông tin về cách xác thực API của bạn và bảo vệ chống lại các lỗ hổng CSRF.

<a name="origin-verification"></a>
### Xác minh origin

Như đã thảo luận ở trên, middleware chống giả mạo request của Laravel sẽ kiểm tra header `Sec-Fetch-Site` trước để xác định xem request có đến từ cùng một origin hay không. Mặc định, nếu bước kiểm tra này không pass, middleware sẽ quay lại xác thực token CSRF.

Tuy nhiên, nếu bạn muốn chỉ muốn dựa vào việc xác minh origin và disable hoàn toàn việc kiểm tra token CSRF, bạn có thể thực hiện việc đó bằng cách sử dụng phương thức `preventRequestForgery` trong file `bootstrap/app.php` của ứng dụng:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(originOnly: true);
})
```

Khi sử dụng chế độ origin-only, các request không pass xác minh origin sẽ nhận được response HTTP `403` thay vì response `419` thường liên quan đến việc token CSRF không khớp.

> [!WARNING]
> Header `Sec-Fetch-Site` chỉ được trình duyệt gửi qua các kết nối an toàn (HTTPS). Nếu ứng dụng của bạn không được chạy trên HTTPS, xác minh origin sẽ không khả dụng và middleware sẽ quay lại xác thực token CSRF.

Nếu ứng dụng của bạn cần chấp nhận các request từ các subdomain (ví dụ: `dashboard.example.com` sẽ chấp nhận các request từ `example.com`), bạn có thể cho phép các request same-site:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(allowSameSite: true);
})
```

<a name="csrf-excluding-uris"></a>
### Loại bỏ các URI khỏi CSRF Protection

Đôi khi bạn có thể muốn loại bỏ một URI ra khỏi CSRF protection. Ví dụ: nếu bạn đang sử dụng [Stripe](https://stripe.com) để xử lý thanh toán và đang sử dụng hệ thống webhook của họ, bạn sẽ cần phải loại bỏ những route mà xử lý những webhook Stripe đó ra khỏi CSRF protection vì Stripe sẽ không biết mã token CSRF nào sẽ được gửi đến route của bạn.

Thông thường, bạn nên đặt các loại route này ra ngoài group middleware `web`, group này được Laravel áp dụng cho tất cả các route có trong file `routes/web.php`. Tuy nhiên, bạn cũng có thể loại bỏ các route cụ thể bằng cách cung cấp URI của chúng cho phương thức `preventRequestForgery` trong file `bootstrap/app.php` của ứng dụng của bạn:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->preventRequestForgery(except: [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ]);
})
```

> [!NOTE]
> Để thuận tiện, CSRF middleware sẽ tự động bị disable cho tất cả các route khi [đang chạy test](/docs/{{version}}/testing).

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Ngoài việc kiểm tra mã token CSRF dưới dạng là một tham số của POST, middleware `PreventRequestForgery` cũng sẽ kiểm tra request header `X-CSRF-TOKEN`. Ví dụ, bạn có thể lưu trữ mã token vào trong một thẻ `meta` HTML:

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Sau đó bạn có thể cài đặt một thư viện như jQuery tự động thêm mã token đó vào tất cả các request header. Điều này sẽ giúp việc thực CSRF Protection sẽ đơn giản và thuận tiện cho những application mà dựa trên AJAX sử dụng công nghệ JavaScript legacy:

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel lưu trữ mã token CSRF trong cookie mã hoá `XSRF-TOKEN` được chứa trong mỗi response được tạo bởi framework. Bạn có thể sử dụng giá trị cookie này để set vào request header `X-XSRF-TOKEN`.

Cookie này được gửi về chủ yếu là tạo sự thuận tiện cho nhà phát triển vì một số framework và thư viện JavaScript, như Angular và Axios, sẽ tự động set giá trị của nó vào trong header `X-XSRF-TOKEN` cho các request có cùng origin.
