# HTTP Requests

- [Giới thiệu](#introduction)
- [Tương tác với request](#interacting-with-the-request)
    - [Truy cập vào Request](#accessing-the-request)
    - [Request Path và Method](#request-path-and-method)
    - [Request Headers](#request-headers)
    - [Request IP Address](#request-ip-address)
    - [Content Negotiation](#content-negotiation)
    - [PSR-7 Requests](#psr7-requests)
- [Input](#input)
    - [Lấy Input](#retrieving-input)
    - [Xác nhận nếu Input tồn tại](#determining-if-input-is-present)
    - [Merge thêm giá trị Input](#merging-additional-input)
    - [Old Input](#old-input)
    - [Cookies](#cookies)
    - [Cắt và chuẩn hoá Input](#input-trimming-and-normalization)
- [Files](#files)
    - [Retrieving Uploaded Files](#retrieving-uploaded-files)
    - [Storing Uploaded Files](#storing-uploaded-files)
- [Cấu hình Trusted Proxies](#configuring-trusted-proxies)
- [Cấu hình Trusted Hosts](#configuring-trusted-hosts)

<a name="introduction"></a>
## Giới thiệu

Class `Illuminate\Http\Request` của Laravel cung cấp một cách hướng đối tượng để tương tác với các request HTTP hiện tại đang được ứng dụng của bạn xử lý cũng như truy xuất dữ liệu input, cookie và các file đã được gửi cùng với request.

<a name="interacting-with-the-request"></a>
## Tương tác với request

<a name="accessing-the-request"></a>
## Truy cập vào Request

Để có được một instance của request HTTP thông qua việc khai báo phụ thuộc, bạn có thể khai báo kiểu class `Illuminate\Http\Request` trong route closure hoặc phương thức controller của bạn. Instance của request sẽ tự động được inject bởi Laravel [service container](/docs/{{version}}/container):

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

Như đã đề cập, bạn cũng có thể khai báo kiểu class `Illuminate\Http\Request` trong một route Closure. Service container sẽ tự động inject request vào Closure khi được thực thi:

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="dependency-injection-route-parameters"></a>
#### Dependency Injection và Route Parameters

Nếu phương thức controller của bạn cũng đang yêu cầu input từ một tham số route, bạn nên liệt kê các tham số route đó đằng sau các phụ thuộc. Ví dụ: nếu route của bạn được định nghĩa như sau:

    use App\Http\Controllers\UserController;

    Route::put('/user/{id}', [UserController::class, 'update']);

Bạn vẫn có thể khai báo kiểu `Illuminate\Http\Request` và truy cập vào tham số `id` của route bằng cách định nghĩa phương thức controller như sau:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  string  $id
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="request-path-and-method"></a>
### Request Path và Method

Instance `Illuminate\Http\Request` cung cấp nhiều phương thức để kiểm tra HTTP request đến và nó được extend từ class `Symfony\Component\HttpFoundation\Request`. Chúng tôi sẽ nói về một số phương thức quan trọng dưới đây.

<a name="retrieving-the-request-path"></a>
#### Lấy Request Path

Phương thức `path` trả về thông tin path của request. Vì vậy, nếu request được targeted là `http://example.com/foo/bar`, thì phương thức `path` sẽ trả về `foo/bar`:

    $uri = $request->path();

<a name="inspecting-the-request-path"></a>
#### Inspecting The Request Path / Route

Phương thức `is` cho phép bạn kiểm tra path của request có khớp với một pattern đã cho hay không. Bạn có thể sử dụng ký tự `*` làm ký tự đại diện khi sử dụng phương thức này:

    if ($request->is('admin/*')) {
        //
    }

Bằng cách sử dụng phương thức `routeIs`, bạn có thể xác định xem request đến có khớp với [tên của một route](/docs/{{version}}/routing#named-routes) hay không:

    if ($request->routeIs('admin.*')) {
        //
    }

<a name="retrieving-the-request-url"></a>
#### Lấy Request URL

Để lấy full URL của request, bạn có thể sử dụng các phương thức `url` hoặc `fullUrl`. Phương thức `url` sẽ trả về URL mà không có chuỗi truy vấn, trong khi phương thức `fullUrl` sẽ bao gồm chuỗi truy vấn:

    $url = $request->url();

    $urlWithQueryString = $request->fullUrl();

Nếu bạn muốn nối thêm biến vào URL hiện tại, bạn có thể gọi phương thức `fullUrlWithQuery`. Phương thức này sẽ nối một mảng các biến đã cho vào các biến hiện tại:

    $request->fullUrlWithQuery(['type' => 'phone']);

<a name="retrieving-the-request-method"></a>
#### Lấy Request Method

Phương thức `method` sẽ trả về mothed HTTP của request. Bạn có thể sử dụng phương thức `isMethod` để kiểm tra mothed HTTP có khớp với một chuỗi đã cho hay không:

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="request-headers"></a>
### Request Headers

Bạn có thể lấy ra header của request từ instance `Illuminate\Http\Request` bằng phương thức `header`. Nếu header không tồn tại trong request, `null` sẽ được trả về. Tuy nhiên, phương thức `header` cũng chấp nhận tham số thứ hai tùy chọn sẽ được trả về nếu header không có trong request:

    $value = $request->header('X-Header-Name');

    $value = $request->header('X-Header-Name', 'default');

Phương thức `hasHeader` có thể được sử dụng để xác định xem request có chứa header nhất định hay không:

    if ($request->hasHeader('X-Header-Name')) {
        //
    }

Để thuận tiện, phương thức `bearerToken` có thể được sử dụng để lấy ra mã token từ header `Authorization`. Nếu không có header nào như vậy, một chuỗi trống sẽ được trả về:

    $token = $request->bearerToken();

<a name="request-ip-address"></a>
### Request IP Address

Phương thức `ip` có thể được sử dụng để lấy ra địa chỉ IP của client đã gửi request tới ứng dụng của bạn:

    $ipAddress = $request->ip();

<a name="content-negotiation"></a>
### Content Negotiation

Laravel cung cấp một số phương thức để kiểm tra các content type được yêu cầu bởi request gửi đến thông qua header `Accept`. Đầu tiên, phương thức `getAcceptableContentTypes` sẽ trả về một mảng chứa tất cả các content type được request chấp nhận:

    $contentTypes = $request->getAcceptableContentTypes();

Phương thức `accepts` sẽ chấp nhận một mảng các content type và sẽ trả về `true` nếu có bất kỳ content type nào được chấp nhận bởi request. Nếu không, `false` sẽ được trả về:

    if ($request->accepts(['text/html', 'application/json'])) {
        // ...
    }

Bạn có thể sử dụng phương thức `prefers` để xác định xem content type nào có trong một mảng content type được request ưu tiên hơn. Nếu không có content type nào được request chấp nhận, `null` sẽ được trả về:

    $preferred = $request->prefers(['text/html', 'application/json']);

Vì nhiều ứng dụng chỉ cần HTML hoặc JSON nên bạn có thể sử dụng phương thức `expectsJson` để nhanh chóng xác định xem request đến có yêu cầu response JSON hay không:

    if ($request->expectsJson()) {
        // ...
    }

<a name="psr7-requests"></a>
### PSR-7 Requests

[Tiêu chuẩn PSR-7](https://www.php-fig.org/psr/psr-7/) định nghĩa interface cho các message HTTP, bao gồm cả các request và response. Nếu bạn muốn có một instance của PSR-7 request thay vì Laravel request, trước tiên bạn sẽ cần cài đặt một vài thư viện. Laravel sẽ sử dụng component *Symfony HTTP Message Bridge* để chuyển đổi các request và response của Laravel thành các implementation tương thích PSR-7:

    composer require symfony/psr-http-message-bridge
    composer require nyholm/psr7

Khi bạn đã cài đặt xong các thư viện trên, bạn có thể lấy được PSR-7 request bằng cách khai báo kiểu request interface trên vào route closure hoặc controller method:

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} Nếu bạn muốn trả về một instance response PSR-7 từ một route hoặc một controller, nó sẽ tự động được chuyển đổi trở lại thành một instance response Laravel và được hiển thị bởi framework.

<a name="input"></a>
## Input

<a name="retrieving-input"></a>
### Lấy Input

<a name="retrieving-all-input-data"></a>
#### Lấy tất cả Input Data

Bạn có thể lấy ra tất cả các dữ liệu input của incoming request dưới dạng một `array` bằng cách sử dụng phương thức `all`. Phương thức này có thể được sử dụng bất kể incoming request là từ HTML form hay là request XHR:

    $input = $request->all();

Bằng cách sử dụng phương thức `collect`, bạn có thể lấy ra tất cả dữ liệu input của incoming request dưới dạng một [collection](/docs/{{version}}/collections):

    $input = $request->collect();

Phương thức `collect` cũng cho phép bạn lấy ra một tập con của input của incoming request dưới dạng một collection:

    $request->collect('users')->each(function ($user) {
        // ...
    });

<a name="retrieving-an-input-value"></a>
#### Lấy một Input Value

Sử dụng một vài phương thức đơn giản sau đây, bạn có thể truy cập tất cả các input của người dùng từ instance `Illuminate\Http\Request` của bạn mà không cần lo lắng về method HTTP nào được sử dụng cho request. Bất kể method HTTP nào, phương thức `input` có thể được sử dụng để lấy dữ liệu mà người dùng đã nhập:

    $name = $request->input('name');

Bạn có thể truyền vào một giá trị mặc định làm tham số thứ hai cho phương thức `input`. Giá trị này sẽ được trả về nếu giá trị input trong request không tồn tại:

    $name = $request->input('name', 'Sally');

Khi làm việc với các form có chứa một mảng input, hãy sử dụng ký tự "chấm" để truy cập vào mảng:

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

Bạn có thể gọi phương thức `input` mà không có bất kỳ tham số nào để lấy ra tất cả các giá trị input dưới dạng một mảng:

    $input = $request->input();

<a name="retrieving-input-from-the-query-string"></a>
#### Lấy Input từ Query String

Trong khi phương thức `input` lấy ra các giá trị từ toàn bộ request (bao gồm cả query string), thì phương thức `query` sẽ chỉ lấy các giá trị từ query string:

    $name = $request->query('name');

Nếu như không tồn tại giá trị trong query string, tham số thứ hai của phương thức này sẽ được trả về:

    $name = $request->query('name', 'Helen');

Bạn có thể gọi phương thức `query` mà không có bất kỳ tham số nào để lấy ra tất cả các giá trị query string dưới dạng một mảng:

    $query = $request->query();

<a name="retrieving-json-input-values"></a>
#### Lấy JSON Input Values

Khi gửi các JSON request cho application của bạn, bạn có thể truy cập dữ liệu JSON thông qua phương thức `input` miễn là `Content-Type` header của request đó được set là `application/json`. Bạn thậm chí có thể sử dụng cú pháp "chấm" để lấy ra các phần tử con trong mảng JSON:

    $name = $request->input('user.name');

<a name="retrieving-boolean-input-values"></a>
#### Lấy giá trị input là boolean

Khi xử lý các phần tử HTML như checkbox, ứng dụng của bạn có thể nhận vào các giá trị "truthy" là các chuỗi chứ không phải dạng boolean. Ví dụ: "true" hoặc "on". Để thuận tiện, bạn có thể sử dụng phương thức `boolean` để lấy ra các giá trị này dưới dạng boolean. Phương thức `boolean` sẽ trả về `true` cho 1, "1", true, "true", "on" và "yes". Tất cả các giá trị khác sẽ trả về `false`:

    $archived = $request->boolean('archived');

<a name="retrieving-date-input-values"></a>
#### Retrieving Date Input Values

Để thuận tiện, các giá trị input chứa ngày, giờ có thể được lấy ra dưới dạng instances Carbon bằng phương thức `date`. Nếu request không chứa các giá trị input có tên đã cho, `null` sẽ được trả về:

    $birthday = $request->date('birthday');

Tham số thứ hai và thứ ba được phương thức `date` chấp nhận là sử dụng để chỉ định định dạng và timezone của date tương ứng:

    $elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');

Nếu có giá trị input nhưng có định dạng không hợp lệ, lỗi `InvalidArgumentException` sẽ được đưa ra; do đó, bạn nên xác thực dữ liệu input trước khi gọi phương thức `date`.

<a name="retrieving-input-via-dynamic-properties"></a>
#### Retrieving Input Via Dynamic Properties

Bạn cũng có thể truy cập vào thông tin input của người dùng bằng cách sử dụng các thuộc tính động trên instance `Illuminate\Http\Request`. Ví dụ: nếu một trong các form ứng dụng của bạn chứa field `name`, bạn có thể truy cập vào giá trị của field như sau:

    $name = $request->name;

Khi sử dụng thuộc tính động, đầu tiên Laravel sẽ tìm giá trị của tham số trong payload của request. Nếu nó không tồn tại, Laravel sẽ tìm kiếm field đó trong các tham số của route.

<a name="retrieving-a-portion-of-the-input-data"></a>
#### Lấy A Portion Of The Input Data

Nếu bạn cần truy xuất một tập con của dữ liệu input, bạn có thể sử dụng các phương thức `only` hoặc `except`. Cả hai phương thức này đều chấp nhận một `array` hoặc một danh sách động các tham số:

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> {note} Phương thức `only` trả về tất cả các cặp key / value mà bạn yêu cầu; tuy nhiên, nó sẽ không trả về các cặp key / value mà không có trong request.

<a name="determining-if-input-is-present"></a>
### Xác nhận nếu Input tồn tại

Bạn có thể sử dụng phương thức `has` để xác định xem giá trị đó có tồn tại trong request hay không. Phương thức `has` sẽ trả về `true` nếu giá trị tồn tại trong request:

    if ($request->has('name')) {
        //
    }

Khi được cung cấp một mảng, phương thức `has` sẽ xác định xem tất cả các giá trị có trong mảng đó có tồn tại hay không:

    if ($request->has(['name', 'email'])) {
        //
    }

Phương thức `whenHas` sẽ chạy closure đã cho nếu có một giá trị trong request tồn tại:

    $request->whenHas('name', function ($input) {
        //
    });

Closure thứ hai có thể được truyền cho phương thức `whenHas` và sẽ được chạy nếu giá trị được chỉ định không tồn tại trong request:

    $request->whenHas('name', function ($input) {
        // The "name" value is present...
    }, function () {
        // The "name" value is not present...
    });

Phương thức `hasAny` trả về `true` nếu có bất kỳ giá trị nào tồn tại:

    if ($request->hasAny(['name', 'email'])) {
        //
    }

Nếu bạn muốn xác định xem một giá trị có tồn tại trong request và không trống hay không, bạn có thể sử dụng phương thức `filled`:

    if ($request->filled('name')) {
        //
    }

Phương thức `whenFilled` sẽ chạy closure đã cho nếu có một giá trị trong request và không trống:

    $request->whenFilled('name', function ($input) {
        //
    });

Closure thứ hai có thể được truyền đến phương thức `whenFilled` sẽ được chạy nếu giá trị được chỉ định không có nội dung:

    $request->whenFilled('name', function ($input) {
        // The "name" value is filled...
    }, function () {
        // The "name" value is not filled...
    });

Để xác định xem một khóa nào đó có bị thiếu trong request hay không, bạn có thể sử dụng phương thức `missing`:

    if ($request->missing('name')) {
        //
    }

<a name="merging-additional-input"></a>
### Merge thêm giá trị Input

Thỉnh thoảng, bạn có thể cần tự merge thêm dữ liệu input vào dữ liệu input hiện có của request. Để thực hiện điều này, bạn có thể sử dụng phương thức `merge`:

    $request->merge(['votes' => 0]);

Phương thức `mergeIfMissing` có thể được sử dụng để merge input vào request nếu các khóa chưa tồn tại trong dữ liệu input của request:

    $request->mergeIfMissing(['votes' => 0]);

<a name="old-input"></a>
### Old Input

Laravel cho phép bạn giữ giá trị input từ request trước đó đến request tiếp theo. Tính năng này đặc biệt hữu ích cho việc điền lại các giá trị vào form sau khi phát hiện có lỗi dữ liệu. Tuy nhiên, nếu bạn đang sử dụng [validation features](/docs/{{version}}/validation) của Laravel, có thể bạn sẽ không cần phải sử dụng trực tiếp phương thức flash vào session input theo cách thủ công, hệ thống built-in validation của Laravel sẽ tự động gọi đến chúng.

<a name="flashing-input-to-the-session"></a>
#### Flashing input đến Session

Phương thức `flash` trong class `Illuminate\Http\Request` sẽ flash tất cả input của request hiện tại vào [session](/docs/{{version}}/session) để những input đó được chuyển đến request tiếp theo của người dùng:

    $request->flash();

Bạn cũng có thể sử dụng các phương thức `flashOnly` và `flashExcept` để flash một tập hợp con của dữ liệu request vào session. Các phương thức này rất hữu ích để giữ thông tin nhạy cảm không bị lưu vào session như mật khẩu:

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

<a name="flashing-input-then-redirecting"></a>
#### Flashing Input vào chuyển hướng

Vì bạn thường xuyên phải flash input vào session và sau đó chuyển về trang trước đó, bạn có thể dễ dàng đưa những input đó vào chuyển hướng đó bằng cách sử dụng phương thức `withInput`:

    return redirect('form')->withInput();

    return redirect()->route('user.create')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

<a name="retrieving-old-input"></a>
#### Lấy Old Input

Để lấy một input flash từ request trước đó, hãy gọi phương thức `old` trong instance của `Illuminate\Http\Request`. Phương thức `old` sẽ lấy dữ liệu input được flash trước đó từ [session](/docs/{{version}}/session):

    $username = $request->old('username');

Laravel cũng cung cấp một global helper `old`. Nếu bạn đang muốn hiển thị một old input vào trong một [Blade template](/docs/{{version}}/blade), nó sẽ thuận tiện hơn khi bạn sử dụng helper `old` để tạo lại form. Nếu old input không tồn tại, `null` sẽ được trả về:

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### Cookies

<a name="retrieving-cookies-from-requests"></a>
#### Lấy Cookies từ Request

Tất cả các cookie được tạo bởi Laravel framework đều được mã hóa và được ký bằng mã xác thực, nghĩa là chúng sẽ bị coi là không hợp lệ nếu chúng bị client thay đổi. Để lấy giá trị cookie từ request, hãy sử dụng phương thức `cookie` trong instance `Illuminate\Http\Request`:

    $value = $request->cookie('name');

<a name="input-trimming-and-normalization"></a>
## Cắt và chuẩn hoá Input

Mặc định, Laravel sẽ chứa các middleware `App\Http\Middleware\TrimStrings` và `App\Http\Middleware\ConvertEmptyStringsToNull` trong stack middleware global của application. Các middleware trên được liệt kê trong stack bởi class `App\Http\Kernel`. Các middleware này sẽ tự động trim tất cả các field dạng chuỗi trên request, cũng như chuyển đổi bất kỳ field nào đang ở dạng chuỗi trống thành `null`. Điều này cho phép bạn cần không phải lo lắng về những định dạng chuỗi có trong các route và controller của bạn.

Nếu bạn muốn vô hiệu hóa hành vi này, bạn có thể xóa chúng ra khỏi stack middleware của application bằng cách xóa chúng ra khỏi thuộc tính `$middleware` của class `App\Http\Kernel` của bạn.

<a name="files"></a>
## Files

<a name="retrieving-uploaded-files"></a>
### Lấy Uploaded Files

Bạn có thể ra các file đã được upload từ một instance`Illuminate\Http\Request` bằng cách sử dụng phương thức `file` hoặc sử dụng các thuộc tính động. Phương thức `file` trả về một instance của class `Illuminate\Http\UploadedFile`, được extend từ class PHP `SplFileInfo` và cung cấp nhiều phương thức để tương tác với file:

    $file = $request->file('photo');

    $file = $request->photo;

Bạn có thể kiểm tra một file có tồn tại trong request hay không bằng cách sử dụng phương thức `hasFile`:

    if ($request->hasFile('photo')) {
        //
    }

<a name="validating-successful-uploads"></a>
#### Kiểm tra upload thành công hay không

Ngoài việc kiểm tra xem file có tồn tại hay không, bạn cũng có thể cần xác minh rằng không có vấn đề gì khi tải file lên, qua phương thức `isValid`:

    if ($request->file('photo')->isValid()) {
        //
    }

<a name="file-paths-extensions"></a>
#### File Paths và Extensions

Class `UploadedFile` cũng chứa các phương thức có thể truy cập đến đường dẫn của file và phần đuôi mở rộng của nó. Phương thức `extension` sẽ cố gắng đoán phần đuôi mở rộng của file dựa trên nội dung của nó. Phần đuôi rộng này có thể khác với phần đuôi mở rộng được cung cấp bởi client:

    $path = $request->photo->path();

    $extension = $request->photo->extension();

<a name="other-file-methods"></a>
#### Other File Methods

Có nhiều phương thức khác có sẵn trong các instance `UploadedFile`. Hãy kiểm tra [tài liệu API cho class này](https://api.symfony.com/master/Symfony/Component/HttpFoundation/File/UploadedFile.html) để biết thêm thông tin về các phương thức này.

<a name="storing-uploaded-files"></a>
### Lưu file upload

Để lưu trữ một file đã được tải lên, thông thường bạn sẽ sử dụng một trong các [filesystems](/docs/{{version}}/filesystem) đã được cấu hình của bạn. Class `UploadedFile` có một phương thức `store` sẽ chuyển file đã tải lên vào một trong các disk của bạn, nó có thể là một vị trí trong local filesystem hoặc là một vị trí được lưu trữ trên đám mây như Amazon S3.

Phương thức `store` chấp nhận một đường dẫn, nơi mà file sẽ được lưu, và được bắt đầu tính từ đường dẫn gốc được cấu hình trong file filesystem. Đường dẫn này không được chứa tên file, vì một unique ID sẽ tự động được tạo để đặt tên file.

Phương thức `store` cũng chấp nhận một tham số thứ hai tùy chọn tên disk nào sẽ được sử dụng để lưu trữ file. Phương thức sẽ trả về đường dẫn của file liên kết đến thư mục gốc của disk đó:

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

Nếu bạn không muốn tên tệp được tự động tạo, bạn có thể sử dụng phương thức `storeAs`, nó sẽ chấp nhận một đường dẫn, một tên tệp và một tên disk làm tham số của nó:

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

> {tip} Để biết thêm thông tin về việc lưu file trong Laravel, hãy xem [tài liệu về lưu file](/docs/{{version}}/filesystem).

<a name="configuring-trusted-proxies"></a>
## Cấu hình Trusted Proxies

Khi application của bạn đang chạy sau một hệ thống load balancer, mà không dùng chứng chỉ TLS / SSL để kết nối đến server của bạn. Đôi khi bạn sẽ cảm thấy rằng application của bạn sẽ không trả về liên kết HTTPS khi dùng helper `url`. Thông thường, điều này là do application của bạn đang bị chuyển tiếp lưu lượng truy cập từ load balancer vào cổng 80 và không biết rằng nó đang tạo ra các liên kết không an toàn.

Để giải quyết vấn đề này, bạn có thể sử dụng middleware `App\Http\Middleware\TrustProxies` có trong application Laravel của bạn, cho phép bạn nhanh chóng tùy chỉnh các load balancer hoặc các proxy mà application đang sử dụng, mà bạn tin tưởng. Các proxy mà bạn tin tưởng nên được liệt kê dưới dạng một mảng trong thuộc tính `$proxies` của middleware này. Ngoài việc cấu hình proxy tin tưởng, bạn cũng có thể cấu hình proxy `$headers` mà bạn tin tưởng:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Middleware\TrustProxies as Middleware;
    use Illuminate\Http\Request;

    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var string|array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];

        /**
         * The headers that should be used to detect proxies.
         *
         * @var int
         */
        protected $headers = Request::HEADER_X_FORWARDED_FOR | Request::HEADER_X_FORWARDED_HOST | Request::HEADER_X_FORWARDED_PORT | Request::HEADER_X_FORWARDED_PROTO;
    }

> {tip} Nếu bạn đang sử dụng AWS Elastic Load Balancing, thì giá trị `$headers` của bạn phải là `Request::HEADER_X_FORWARDED_AWS_ELB`. Để biết thêm thông tin về các hằng số có thể được sử dụng trong thuộc tính `$headers`, hãy xem tài liệu của Symfony về [trusting proxies](https://symfony.com/doc/current/deployment/proxies.html).

<a name="trusting-all-proxies"></a>
#### Trusting tất cả Proxies

Nếu bạn đang sử dụng Amazon AWS hoặc các "cloud" khác cung cấp load balancer, bạn có thể không biết địa chỉ IP thật sự của load balancer. Trong trường hợp này, bạn có thể sử dụng `*` để trust tất cả các proxy:

    /**
     * The trusted proxies for this application.
     *
     * @var string|array
     */
    protected $proxies = '*';

<a name="configuring-trusted-hosts"></a>
## Cấu hình Trusted Hosts

Mặc định, Laravel sẽ respond tất cả các request mà nó nhận được bất kể nội dung của header `Host` của request HTTP đó là gì. Ngoài ra, giá trị của header `Host` sẽ được sử dụng khi tạo URL cho ứng dụng của bạn trong khi web request.

Thông thường, bạn nên cấu hình máy chủ web của bạn, chẳng hạn như Nginx hoặc Apache, để chỉ gửi request đến ứng dụng giống với một host name nhất định. Tuy nhiên, nếu bạn không có khả năng tùy chỉnh trực tiếp máy chủ web của bạn và cần hướng dẫn Laravel chỉ phản hồi với một số host name nhất định, bạn có thể làm như vậy bằng cách bật middleware `App\Http\Middleware\TrustHosts` trong ứng dụng của bạn.

Middleware `TrustHosts` đã được khai báo có sẵn trong stack `$middleware` trong ứng dụng của bạn; tuy nhiên, bạn cần uncomment để nó hoạt động. Trong phương thức `hosts` của middleware này, bạn có thể chỉ định host name mà ứng dụng của bạn sẽ respond. Các request đến với các giá trị header `Host` khác sẽ bị từ chối:

    /**
     * Get the host patterns that should be trusted.
     *
     * @return array
     */
    public function hosts()
    {
        return [
            'laravel.test',
            $this->allSubdomainsOfApplicationUrl(),
        ];
    }

Phương thức helper `allSubdomainsOfApplicationUrl` sẽ trả về một biểu thức chính quy khớp với tất cả các subdomain của giá trị đã được cấu hình `app.url` trong ứng dụng của bạn. Phương thức helper này cung cấp một cách thuận tiện để cho phép tất cả các subdomain của ứng dụng của bạn khi bạn xây dựng một ứng dụng sử dụng wildcard subdomain.
