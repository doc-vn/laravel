# CSRF Protection

- [Giới thiệu](#csrf-introduction)
- [Loại bỏ các URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Giới thiệu

Laravel giúp dễ dàng bảo vệ application của bạn khỏi các cuộc tấn công [giả mạo request](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF). Các cuộc tấn công giả mạo request là một loại khai thác độc hại, mà theo đó các lệnh trái phép sẽ được thực hiện với tư cách là người dùng đã được xác thực.

Laravel sẽ tự động tạo ra một mã CSRF "token" cho mỗi session người dùng và được quản lý bởi application. Mã token này sẽ được sử dụng để kiểm tra người dùng đã được xác thực chính là người đã thực hiện các request vào application.

Bất cứ khi nào bạn định nghĩa một HTML form trong ứng dụng của bạn, bạn nên tạo một field hidden chứa mã CSRF token để middleware protection CSRF có thể kiểm tra request đó. Bạn có thể sử dụng helper `csrf_field` để tạo field hidden chứa mã token đó:

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

`VerifyCsrfToken` [middleware](/docs/{{version}}/middleware), đã được khai báo sẵn trong group middleware `web`, sẽ tự động kiểm tra mã token trong input của request có khớp với mã token được lưu trữ trong session trên server hay không.

#### CSRF Tokens và JavaScript

Khi xây dụng một application mà điều khiển bằng JavaScript, sẽ rất thuận tiện nếu thư viện JavaScript HTTP của bạn tự động gắn mã token CSRF vào mỗi request. Mặc định, file `resources/assets/js/bootstrap.js` sẽ đăng ký giá trị của thẻ meta `csrf-token` với thư viện Axios HTTP. Nếu bạn không sử dụng thư viện này, bạn sẽ cần phải tự cài đặt hành vi này cho application của bạn.

<a name="csrf-excluding-uris"></a>
## Loại bỏ các URI khỏi CSRF Protection

Đôi khi bạn có thể muốn loại bỏ một URI ra khỏi CSRF protection. Ví dụ: nếu bạn đang sử dụng [Stripe](https://stripe.com) để xử lý thanh toán và đang sử dụng hệ thống webhook của họ, bạn sẽ cần phải loại bỏ những route mà xử lý những webhook Stripe đó ra khỏi CSRF protection vì Stripe sẽ không biết mã token CSRF nào sẽ được gửi đến route của bạn.

Thông thường, bạn nên đặt các loại route này ra ngoài group middleware `web`, group này được khai báo trong file `RouteServiceProvider` và áp dụng cho tất cả các route có trong file `routes/web.php`. Tuy nhiên, bạn cũng có thể loại bỏ các route này bằng cách thêm URI của chúng vào thuộc tính `$except` trong middleware `VerifyCsrfToken`:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Ngoài việc kiểm tra mã token CSRF dưới dạng là một tham số của một POST, middleware `ConfirmCsrfToken` cũng sẽ kiểm tra request header `X-CSRF-TOKEN`. Ví dụ, bạn có thể lưu trữ mã token vào trong thẻ `meta` HTML:

    <meta name="csrf-token" content="{{ csrf_token() }}">

Sau khi bạn đã tạo thẻ `meta`, bạn có thể cài đặt một thư viện như jQuery tự động thêm mã token đó vào tất cả các request header. Điều này sẽ giúp việc thực CSRF Protection sẽ đơn giản và thuận tiện cho những application mà dựa trên AJAX:

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {tip} Mặc định, file `resources/assets/js/bootstrap.js` đã cài đặt giá trị của thẻ meta `csrf-token` với thư viện Axios HTTP. Nên nếu bạn không sử dụng thư viện này, bạn sẽ cần phải tự cài đặt hành vi này cho ứng dụng của bạn.

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel lưu trữ mã token CSRF trong cookie `XSRF-TOKEN` được chứa trong mỗi response mà tạo bởi framework. Bạn có thể sử dụng giá trị cookie này để set vào request header `X-XSRF-TOKEN`.

Cookie này được gửi đến clinet chủ yếu để giúp cho một số framework và thư viện JavaScript, như Angular và Axios, tự động cài đặt giá trị đó vào trong header `X-XSRF-TOKEN`.
