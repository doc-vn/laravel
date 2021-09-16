# CSRF Protection

- [Giới thiệu](#csrf-introduction)
- [Loại bỏ các URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Giới thiệu

Laravel giúp dễ dàng bảo vệ application của bạn khỏi các cuộc tấn công [giả mạo request](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF). Các cuộc tấn công giả mạo request là một loại khai thác độc hại, mà theo đó các lệnh trái phép sẽ được thực hiện với tư cách là người dùng đã được xác thực.

Laravel sẽ tự động tạo ra một mã CSRF "token" cho mỗi session người dùng và được quản lý bởi application. Mã token này sẽ được sử dụng để xác minh rằng người dùng đã được xác thực là người thực hiện các request vào application.

Bất cứ khi nào bạn định nghĩa một HTML form trong ứng dụng của bạn, bạn nên tạo một field ẩn chứa mã CSRF token để middleware protection CSRF có thể xác thực request đó. Bạn có thể sử dụng helper `csrf_field` để tạo field ẩn chứa mã token đó:

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

`VerifyCsrfToken` [middleware](/docs/{{version}}/middleware), đã được khai báo sẵn trong group middleware `web`, sẽ tự động xác minh mã token trong input của request có khớp với mã token được lưu trữ trong session hay không.

#### CSRF Tokens và JavaScript

Khi xây dụng application điều khiển bằng JavaScript, nó sẽ rất thuận tiện khi thư viện JavaScript HTTP của bạn sẽ tự động gắn mã token CSRF vào mỗi request gửi đi. Mặc định, file `resources/assets/js/bootstrap.js` sẽ đăng ký giá trị của thẻ meta `csrf-token` với thư viện Axios HTTP. Nếu bạn không sử dụng thư viện này, bạn sẽ cần phải cài đặt bằng tay hành vi này cho application của bạn.

<a name="csrf-excluding-uris"></a>
## Loại bỏ các URI khỏi CSRF Protection

Đôi khi bạn có thể muốn loại bỏ một URI ra khỏi CSRF protection. Ví dụ: nếu bạn đang sử dụng [Stripe](https://stripe.com) để xử lý thanh toán và đang sử dụng hệ thống webhook của họ, bạn sẽ cần phải loại bỏ route xử lý những webhook Stripe đó ra khỏi CSRF protection vì Stripe sẽ không biết mã token CSRF nào sẽ được gửi đến route của bạn.

Thông thường, bạn nên đặt các loại route này ra ngoài group middleware `web`, group này được khai báo trong file `RouteServiceProvider` và áp dụng cho tất cả các route có trong tệp `routes/web.php`. Tuy nhiên, bạn cũng có thể loại bỏ các route này bằng cách thêm URI của chúng vào thuộc tính `$except` của middleware `VerifyCsrfToken`:

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

Ngoài việc kiểm tra mã token CSRF dưới dạng tham số của một POST, middleware `ConfirmCsrfToken` cũng sẽ kiểm tra request header `X-CSRF-TOKEN`. Ví dụ, bạn có thể lưu trữ mã token vào trong thẻ `meta` HTML:

    <meta name="csrf-token" content="{{ csrf_token() }}">

Sau khi bạn đã tạo thẻ `meta`, bạn có thể cài đặt một thư viện như jQuery tự động thêm mã token đó vào tất cả các request header. Điều này cung cấp bảo vệ CSRF đơn giản, thuận tiện cho application dựa trên AJAX của bạn:

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {tip} Mặc định, tệp `resources/assets/js/bootstrap.js` đã đăng ký giá trị của thẻ meta `csrf-token` với thư viện Axios HTTP. Nên nếu bạn không sử dụng thư viện này, bạn sẽ cần phải cài đặt thủ công hành vi này cho ứng dụng của bạn.

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel lưu trữ mã token CSRF hiện tại trong cookie `XSRF-TOKEN` được chứa trong mỗi response tạo bởi framework. Bạn có thể sử dụng giá trị cookie này để đặt vào request header `X-XSRF-TOKEN`.

Cookie này được gửi đến clinet chủ yếu để thuận tiện cho một số framework và thư viện JavaScript, như Angular và Axios, tự động cài đặt giá trị đó vào trong header `X-XSRF-TOKEN`.
