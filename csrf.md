# CSRF Protection

- [Giới thiệu](#csrf-introduction)
- [Loại bỏ các URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Giới thiệu

Laravel giúp dễ dàng bảo vệ application của bạn khỏi các cuộc tấn công [giả mạo request](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF). Các giả mạo request trên nhiều trang web là một loại khai thác độc hại, theo đó các lệnh trái phép sẽ được thực hiện với tư cách là người dùng đã được xác thực.

Laravel tự động tạo một mã CSRF "token" cho mỗi session người dùng hoạt động được application quản lý. Mã token này được sử dụng để xác minh rằng người dùng được xác thực là người thực sự thực hiện các request vào application.

Bất cứ khi nào bạn định nghĩa một HTML form trong ứng dụng của bạn, bạn nên chứa một  field ẩn, mã CSRF token trong form để middleware bảo vệ CSRF có thể xác thực request. Bạn có thể sử dụng helper `csrf_field` để tạo field mã token:

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

`VerifyCsrfToken` [middleware](/docs/{{version}}/middleware), được bao gồm trong group middleware `web`, sẽ tự động xác minh rằng mã token trong input của request có khớp với mã token được lưu trữ trong session hay không.

#### CSRF Tokens và JavaScript

Khi xây dụng application điều khiển bằng JavaScript, nó sẽ rất thuận tiện khi thư viện JavaScript HTTP của bạn sẽ tự động đính kèm mã token CSRF vào mọi request gửi đi. Mặc định, file `resources/assets/js/bootstrap.js` sẽ đăng ký giá trị của thẻ meta `csrf-token` với thư viện Axios HTTP. Nếu bạn không sử dụng thư viện này, bạn sẽ cần phải cấu hình thủ công hành vi này cho application của mình.

<a name="csrf-excluding-uris"></a>
## Loại bỏ các URI khỏi CSRF Protection

Đôi khi bạn có thể muốn loại bỏ một URI khỏi bảo vệ CSRF. Ví dụ: nếu bạn đang sử dụng [Stripe](https://stripe.com) để xử lý thanh toán và đang sử dụng hệ thống webhook của họ, bạn sẽ cần loại bỏ route handler webhook Stripe của bạn khỏi bảo vệ CSRF vì Stripe sẽ không biết mã token CSRF nào sẽ được gửi đến route của bạn.

Thông thường, bạn nên đặt các loại route này ra ngoài group middleware `web` cái mà `RouteServiceProvider` áp dụng cho tất cả các route trong tệp `routes/web.php`. Tuy nhiên, bạn cũng có thể loại bỏ các route bằng cách thêm URI của chúng vào thuộc tính `$except` của middleware `VerifyCsrfToken`:

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

Ngoài việc kiểm tra mã token CSRF dưới dạng tham số POST, middleware `ConfirmCsrfToken` cũng sẽ kiểm tra request header `X-CSRF-TOKEN`. Ví dụ, bạn có thể lưu trữ mã token vào trong thẻ `meta` HTML:

    <meta name="csrf-token" content="{{ csrf_token() }}">

Sau đó, khi bạn đã tạo thẻ `meta`, bạn có thể hướng dẫn một thư viện như jQuery tự động thêm mã token vào tất cả các request header. Điều này cung cấp bảo vệ CSRF đơn giản, thuận tiện cho các application dựa trên AJAX của bạn:

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {tip} Mặc định, tệp `resources/assets/js/bootstrap.js` đăng ký giá trị của thẻ meta `csrf-token` với thư viện Axios HTTP. Nếu bạn không sử dụng thư viện này, bạn sẽ cần phải cấu hình thủ công hành vi này cho ứng dụng của mình.

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel lưu trữ mã token CSRF hiện tại trong cookie `XSRF-TOKEN` được chứa trong mỗi response được tạo ra bởi framework. Bạn có thể sử dụng giá trị cookie này để đặt request header `X-XSRF-TOKEN`.

Cookie này được gửi đến clinet chủ yếu để thuận tiện vì một số framework và thư viện JavaScript, như Angular và Axios, tự động đặt giá trị của nó vào trong header `X-XSRF-TOKEN`.
