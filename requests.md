# HTTP Requests

- [Truy cập vào Request](#accessing-the-request)
    - [Request Path và Method](#request-path-and-method)
    - [PSR-7 Requests](#psr7-requests)
- [Input Trimming và Normalization](#input-trimming-and-normalization)
- [Retrieving Input](#retrieving-input)
    - [Old Input](#old-input)
    - [Cookies](#cookies)
- [Files](#files)
    - [Retrieving Uploaded Files](#retrieving-uploaded-files)
    - [Storing Uploaded Files](#storing-uploaded-files)
- [Configuring Trusted Proxies](#configuring-trusted-proxies)

<a name="accessing-the-request"></a>
## Truy cập vào Request

Để có được một instance của request HTTP thông qua việc khai báo phụ thuộc, bạn có thể khai báo kiểu class `Illuminate\Http\Request` trong phương thức controller của bạn. Instance của request sẽ tự động được inject bởi [service container](/docs/{{version}}/container):

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

#### Dependency Injection và Route Parameters

Nếu phương thức controller của bạn cũng đang yêu cầu input từ một tham số route, bạn nên liệt kê các tham số route đó đằng sau các phụ thuộc. Ví dụ: nếu route của bạn được định nghĩa như sau:

    Route::put('user/{id}', 'UserController@update');

Bạn vẫn có thể khai báo kiểu `Illuminate\Http\Request` và truy cập vào tham số `id` của route bằng cách định nghĩa phương thức controller như sau:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

#### Truy cập vào Request thông qua Route Closure

Bạn cũng có thể khai báo kiểu class `Illuminate\Http\Request` trong một route Closure. Service container sẽ tự động inject request vào Closure khi được thực thi:

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="request-path-and-method"></a>
### Request Path và Method

Instance `Illuminate\Http\Request` cung cấp nhiều phương thức để kiểm tra HTTP request cho application của bạn và nó được extend từ class `Symfony\Component\HttpFoundation\Request`. Chúng tôi sẽ nói về một số phương thức quan trọng dưới đây.

#### Lấy Request Path

Phương thức `path` trả về thông tin path của request. Vì vậy, nếu request được targeted là `http://domain.com/foo/bar`, thì phương thức `path` sẽ trả về `foo/bar`:

    $uri = $request->path();

Phương thức `is` cho phép bạn kiểm tra path của request có khớp với một pattern đã cho hay không. Bạn có thể sử dụng ký tự `*` làm ký tự đại diện khi sử dụng phương thức này:

    if ($request->is('admin/*')) {
        //
    }

#### Lấy Request URL

Để lấy full URL của request, bạn có thể sử dụng các phương thức `url` hoặc `fullUrl`. Phương thức `url` sẽ trả về URL mà không có chuỗi truy vấn, trong khi phương thức `fullUrl` sẽ bao gồm chuỗi truy vấn:

    // Without Query String...
    $url = $request->url();

    // With Query String...
    $url = $request->fullUrl();

#### Lấy Request Method

Phương thức `method` sẽ trả về mothed HTTP của request. Bạn có thể sử dụng phương thức `isMethod` để kiểm tra mothed HTTP có khớp với một chuỗi đã cho hay không:

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>
### PSR-7 Requests

[Tiêu chuẩn PSR-7](http://www.php-fig.org/psr/psr-7/) định nghĩa interface cho các message HTTP, bao gồm cả các request và response. Nếu bạn muốn có một instance của PSR-7 request thay vì Laravel request, trước tiên bạn sẽ cần cài đặt một vài thư viện. Laravel sẽ sử dụng component *Symfony HTTP Message Bridge* để chuyển đổi các request và response của Laravel thành các implementation tương thích PSR-7:

    composer require symfony/psr-http-message-bridge
    composer require zendframework/zend-diactoros

Khi bạn đã cài đặt xong các thư viện trên, bạn có thể lấy được PSR-7 request bằng cách khai báo kiểu request interface trên vào route Closure hoặc controller method:

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} Nếu bạn muốn trả về một instance response PSR-7 từ một route hoặc một controller, nó sẽ tự động được chuyển đổi trở lại thành một instance response Laravel và được hiển thị bởi framework.

<a name="input-trimming-and-normalization"></a>
## Input Trimming và Normalization

Mặc định, Laravel sẽ chứa các middleware `TrimStrings` và `ConvertEmptyStringsToNull` trong stack middleware global của application. Các middleware trên được liệt kê trong stack bởi class `App\Http\Kernel`. Các middleware này sẽ tự động trim tất cả các field dạng chuỗi trên request, cũng như chuyển đổi bất kỳ field nào đang ở dạng chuỗi trống thành `null`. Điều này cho phép bạn cần không phải lo lắng về những định dạng chuỗi có trong các route và controller của bạn.

Nếu bạn muốn vô hiệu hóa hành vi này, bạn có thể xóa chúng ra khỏi stack middleware của application.

<a name="retrieving-input"></a>
## Lấy Input

#### Lấy tất cả Input Data

Bạn cũng có thể lấy tất cả dữ liệu input dưới dạng một `array` bằng cách sử dụng phương thức `all`:

    $input = $request->all();

#### Lấy một Input Value

Sử dụng một vài phương thức đơn giản sau đây, bạn có thể truy cập tất cả các input của người dùng từ instance `Illuminate\Http\Request` của bạn mà không cần lo lắng về method HTTP nào được sử dụng cho request. Bất kể method HTTP nào, phương thức `input` có thể được sử dụng để lấy dữ liệu mà người dùng đã nhập:

    $name = $request->input('name');

Bạn có thể truyền vào một giá trị mặc định làm tham số thứ hai cho phương thức `input`. Giá trị này sẽ được trả về nếu giá trị input trong request không tồn tại:

    $name = $request->input('name', 'Sally');

Khi làm việc với các form có chứa một mảng input, hãy sử dụng ký tự "chấm" để truy cập vào mảng:

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

#### Lấy Input từ Query String

Trong khi phương thức `input` lấy ra các giá trị từ toàn bộ request (bao gồm cả query string), thì phương thức `query` sẽ chỉ lấy các giá trị từ query string:

    $name = $request->query('name');

Nếu như không tồn tại giá trị trong query string, tham số thứ hai của phương thức này sẽ được trả về:

    $name = $request->query('name', 'Helen');

Bạn có thể gọi phương thức `query` mà không có bất kỳ tham số nào để lấy ra tất cả các giá trị query string dưới dạng một mảng:

    $query = $request->query();

#### Lấy Input thông qua thuộc tính động

Bạn cũng có thể truy cập vào input của người dùng bằng các thuộc tính động trên instance `Illuminate\Http\Request`. Ví dụ: nếu một trong các form của application của bạn chứa field `name`, bạn có thể truy cập giá trị của field đó như sau:

    $name = $request->name;

Khi sử dụng các thuộc tính động, trước tiên, Laravel sẽ tìm giá trị trong các tham số trong request. Nếu nó không tồn tại, Laravel sẽ tìm kiếm field đó trong các tham số của route.

#### Lấy JSON Input Values

Khi gửi các JSON request cho application của bạn, bạn có thể truy cập dữ liệu JSON thông qua phương thức `input` miễn là `Content-Type` header của request đó được set là `application/json`. Bạn thậm chí có thể sử dụng cú pháp "chấm" để lấy các phần tử con trong mảng JSON:

    $name = $request->input('user.name');

#### Lấy A Portion Of The Input Data

Nếu bạn cần truy xuất một tập con của dữ liệu input, bạn có thể sử dụng các phương thức `only` hoặc `except`. Cả hai phương thức này đều chấp nhận một `array` hoặc một danh sách động các tham số:

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> {tip} Phương thức `only` trả về tất cả các cặp key / value mà bạn yêu cầu; tuy nhiên, nó sẽ không trả về các cặp key / value mà không có trong request.

#### Xác định một giá trị input có tồn tại hay không

Bạn nên sử dụng phương thức `has` để xác định xem giá trị đó có tồn tại trong request hay không. Phương thức `has` sẽ trả về `true` nếu giá trị tồn tại trong request:

    if ($request->has('name')) {
        //
    }

Khi được cung cấp một mảng, phương thức `has` sẽ xác định xem tất cả các giá trị có trong mảng đó có tồn tại hay không:

    if ($request->has(['name', 'email'])) {
        //
    }

Nếu bạn muốn xác định xem một giá trị có tồn tại trong request và không trống hay không, bạn có thể sử dụng phương thức `filled`:

    if ($request->filled('name')) {
        //
    }

<a name="old-input"></a>
### Old Input

Laravel cho phép bạn giữ giá trị input từ request trước đó đến request tiếp theo. Tính năng này đặc biệt hữu ích cho việc điền lại các giá trị vào form sau khi phát hiện có lỗi dữ liệu. Tuy nhiên, nếu bạn đang sử dụng [validation features](/docs/{{version}}/validation) của Laravel, thì bạn sẽ không cần chỉnh sửa gì cho tính năng này, hệ thống built-in validation của Laravel sẽ tự động gọi đến chúng.

#### Flashing input đến Session

Phương thức `flash` trong class `Illuminate\Http\Request` sẽ flash tất cả input của request hiện tại vào [session](/docs/{{version}}/session) để những input đó được chuyển đến request tiếp theo của người dùng:

    $request->flash();

Bạn cũng có thể sử dụng các phương thức `flashOnly` và `flashExcept` để flash một tập hợp con của dữ liệu request vào session. Các phương thức này rất hữu ích để giữ thông tin nhạy cảm không bị lưu vào session như mật khẩu:

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

#### Flashing Input vào chuyển hướng

Vì bạn thường xuyên phải flash input vào session và sau đó chuyển về trang trước đó, bạn có thể dễ dàng đưa những input đó vào chuyển hướng đó bằng cách sử dụng phương thức `withInput`:

    return redirect('form')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

#### Lấy Old Input

Để lấy một input flash từ request trước đó, hãy sử dụng phương thức `old` trong instance `Request`. Phương thức `old` sẽ lấy dữ liệu input được flash trước đó từ [session](/docs/{{version}}/session):

    $username = $request->old('username');

Laravel cũng cung cấp một global helper `old`. Nếu bạn đang muốn hiển thị một old input vào trong một [Blade template](/docs/{{version}}/blade), nó sẽ thuận tiện hơn khi bạn sử dụng helper `old`. Nếu old input không tồn tại, `null` sẽ được trả về:

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### Cookies

#### Lấy Cookies từ Request

Tất cả các cookie được tạo bởi Laravel framework đều được mã hóa và được ký bằng mã xác thực, nghĩa là chúng sẽ bị coi là không hợp lệ nếu chúng bị client thay đổi. Để lấy giá trị cookie từ request, hãy sử dụng phương thức `cookie` trong instance `Illuminate\Http\Request`:

    $value = $request->cookie('name');

Ngoài ra, bạn có thể sử dụng facade `Cookie` để truy cập các giá trị của cookie:

    $value = Cookie::get('name');

#### Gắn Cookies vào Response

Bạn có thể gắn một cookie vào một instance `Illuminate\Http\Response` bằng cách sử dụng phương thức `cookie`. Bạn cần truyền vào tên, giá trị và số phút mà cookie có thể dùng được cho phương thức này:

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

Phương thức `cookie` cũng chấp nhận thêm một vài tham số được sử dụng ít thường xuyên hơn. Nói chung, các tham số này có cùng mục đích và ý nghĩa như các tham số sẽ được truyền cho phương thức [setcookie](https://secure.php.net/manual/en/function.setcookie.php) của PHP:

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

Ngoài ra, bạn có thể sử dụng facade `Cookie` để "queue" cookie cho việc gán vào response trả về từ application. Phương thức `queue` chấp nhận một instance `Cookie` hoặc các tham số cần thiết để tạo một instance `Cookie`. Các cookie này sẽ được gán vào response trước khi nó được gửi đến trình duyệt:

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

#### Tạo Cookie instance

Nếu bạn muốn tạo một instance `Symfony\Component\HttpFoundation\Cookie` để gán cho một instance response, bạn có thể sử dụng global helper `cookie`. Cookie được tạo ra sẽ không được gửi cho client trừ khi nó được gán vào một instance response:

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="files"></a>
## Files

<a name="retrieving-uploaded-files"></a>
### Lấy Uploaded Files

Bạn có thể truy cập đến các file đã được upload từ một instance`Illuminate\Http\Request` bằng cách sử dụng phương thức `file` hoặc sử dụng các thuộc tính động. Phương thức `file` trả về một instance của class `Illuminate\Http\UploadedFile`, được extend từ class PHP `SplFileInfo` và cung cấp nhiều phương thức để tương tác với file:

    $file = $request->file('photo');

    $file = $request->photo;

Bạn có thể kiểm tra một file có tồn tại trong request hay không bằng cách sử dụng phương thức `hasFile`:

    if ($request->hasFile('photo')) {
        //
    }

#### Kiểm tra upload thành công hay không

Ngoài việc kiểm tra xem file có tồn tại hay không, bạn cũng có thể cần xác minh rằng không có vấn đề gì khi tải file lên, qua phương thức `isValid`:

    if ($request->file('photo')->isValid()) {
        //
    }

#### File Paths và Extensions

Class `UploadedFile` cũng chứa các phương thức có thể truy cập đến đường dẫn của file và phần đuôi mở rộng của nó. Phương thức `extension` sẽ cố gắng đoán phần đuôi mở rộng của file dựa trên nội dung của nó. Phần đuôi rộng này có thể khác với phần đuôi mở rộng được cung cấp bởi client:

    $path = $request->photo->path();

    $extension = $request->photo->extension();

#### Other File Methods

Có nhiều phương thức khác có sẵn trong các instance `UploadedFile`. Hãy kiểm tra [tài liệu API cho class này](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) để biết thêm thông tin về các phương thức này.

<a name="storing-uploaded-files"></a>
### Lưu file upload

Để lưu trữ một file đã được tải lên, thông thường bạn sẽ sử dụng một trong các [filesystems](/docs/{{version}}/filesystem) đã được cấu hình của bạn. Class `UploadedFile` có một phương thức `store` sẽ chuyển file đã tải lên vào một trong các disk của bạn, nó có thể là một vị trí trong local filesystem hoặc thậm chí là một vị trí được lưu trữ trên đám mây như Amazon S3.

Phương thức `store` chấp nhận một đường dẫn, nơi mà file sẽ được lưu, và được bắt đầu tính từ đường dẫn gốc được cấu hình trong file filesystem. Đường dẫn này không được chứa tên file, vì một unique ID sẽ tự động được tạo để đặt tên file.

Phương thức `store` cũng chấp nhận một tham số thứ hai tùy chọn tên disk nào sẽ được sử dụng để lưu trữ file. Phương thức sẽ trả về đường dẫn của file liên kết đến thư mục gốc của disk đó:

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

Nếu bạn không muốn tên tệp được tự động tạo, bạn có thể sử dụng phương thức `storeAs`, nó sẽ chấp nhận một đường dẫn, một tên tệp và một tên disk làm tham số của nó:

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

<a name="configuring-trusted-proxies"></a>
## Cấu hình Trusted Proxies

Khi application của bạn đang chạy sau một hệ thống load balancer, mà không dùng chứng chỉ TLS / SSL để kết nối đến server của bạn. Đôi khi bạn sẽ cảm thấy rằng application của bạn sẽ không trả về liên kết HTTPS. Thông thường, điều này là do application của bạn đang bị chuyển tiếp lưu lượng truy cập từ load balancer vào cổng 80 và không biết rằng nó đang tạo ra các liên kết không an toàn.

Để giải quyết vấn đề này, bạn có thể sử dụng middleware `App\Http\Middleware\TrustProxies` có trong application Laravel của bạn, cho phép bạn nhanh chóng tùy chỉnh các load balancer hoặc các proxy mà application đang sử dụng, mà bạn tin tưởng. Các proxy mà bạn tin tưởng nên được liệt kê dưới dạng một mảng trong thuộc tính `$proxies` của middleware này. Ngoài việc cấu hình proxy tin tưởng, bạn có thể cấu hình các header đang được gửi bởi proxy của bạn với thông tin về original request:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];

        /**
         * The current proxy header mappings.
         *
         * @var array
         */
        protected $headers = [
            Request::HEADER_FORWARDED => 'FORWARDED',
            Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
            Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
            Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
            Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        ];
    }

#### Trusting tất cả Proxies

Nếu bạn đang sử dụng Amazon AWS hoặc các "cloud" khác cung cấp load balancer, bạn có thể không biết địa chỉ IP thật sự của load balancer. Trong trường hợp này, bạn có thể sử dụng `**` để trust tất cả các proxy:

    /**
     * The trusted proxies for this application.
     *
     * @var array
     */
    protected $proxies = '**';
