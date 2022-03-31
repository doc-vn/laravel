# HTTP Responses

- [Tạo Responses](#creating-responses)
    - [Gắn Headers vào Responses](#attaching-headers-to-responses)
    - [Gắn Cookies vào Responses](#attaching-cookies-to-responses)
    - [Cookies và Encryption](#cookies-and-encryption)
- [Redirect](#redirects)
    - [Redirect tới Named Routes](#redirecting-named-routes)
    - [Redirect tới Controller Actions](#redirecting-controller-actions)
    - [Redirect tới External Domains](#redirecting-external-domains)
    - [Redirect với Flashed Session Data](#redirecting-with-flashed-session-data)
- [Các loại Response khác](#other-response-types)
    - [View Responses](#view-responses)
    - [JSON Responses](#json-responses)
    - [File Downloads](#file-downloads)
    - [File Responses](#file-responses)
- [Response Macros](#response-macros)

<a name="creating-responses"></a>
## Tạo Responses

#### Strings và Arrays

Tất cả các route và các controller đều sẽ trả về một response và được gửi về cho trình duyệt web của người dùng. Laravel cung cấp một số cách khác nhau để trả về một response. Response cơ bản nhất là trả về một chuỗi từ một route hoặc một controller. Framework sẽ tự động chuyển đổi chuỗi đó thành một HTTP response đầy đủ:

    Route::get('/', function () {
        return 'Hello World';
    });

Ngoài việc trả về một chuỗi từ route và controller của bạn, bạn cũng có thể trả về một mảng. Framework sẽ tự động chuyển mảng đó thành một JSON response:

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} Bạn có biết rằng bạn cũng có thể trả về [Eloquent collections](/docs/{{version}}/eloquent-collections) từ một route hoặc một controller của bạn không? Chúng sẽ tự động được chuyển đổi thành JSON. Bạn cứ thử đi!

#### Response Objects

Thông thường, bạn sẽ không chỉ trả về một chuỗi hoặc một mảng từ route action của bạn. Thay vào đó, bạn có thể sẽ muốn trả lại cả một instance `Illuminate\Http\Response` hoặc một [views](/docs/{{version}}/views).

Trả về cả một instance `Response` cho phép bạn tùy biến status code và header của response's HTTP. Một instance `Response` sẽ được extend từ class `Symfony\Component\HttpFoundation\Response`, cung cấp nhiều phương thức để xây dựng các HTTP response:

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="attaching-headers-to-responses"></a>
#### Gắn Header vào Responses

Hãy nhớ rằng hầu hết các phương thức response đều có thể kết hợp lại với nhau, cho phép bạn dễ dàng khởi tạo một response instance. Ví dụ, bạn có thể sử dụng phương thức `header` để thêm một danh sách header cho response trước khi gửi chúng về cho người dùng:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

Hoặc, bạn có thể sử dụng phương thức `withHeaders` để chỉ định một mảng các header sẽ được thêm vào response:

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

##### Cache Control Middleware

Laravel có chứa một middleware `cache.headers`, middleware này có thể được sử dụng để thiết lập nhanh một header `Cache-Control` cho một nhóm các route. Nếu `etag` được chỉ định trong danh sách lệnh, một hash MD5 của nội dung response sẽ được tự động set làm ETag identifier:

    Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function() {
        Route::get('privacy', function () {
            // ...
        });

        Route::get('terms', function () {
            // ...
        });
    });

<a name="attaching-cookies-to-responses"></a>
#### Gắn Cookies vào Responses

Phương thức `cookie` trong response instance cho phép bạn dễ dàng gắn một cookie vào response. Ví dụ, bạn có thể sử dụng phương thức `cookie` để tạo một cookie và dễ dàng gắn nó vào response như sau:

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

Phương thức `cookie` cũng cho phép thêm một vài tham số ít sử dụng hơn. Nói chung, các tham số này có cùng mục đích và ý nghĩa như các tham số được cung cấp bởi phương thức [setcookie](https://secure.php.net/manual/en/function.setcookie.php) của PHP:

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

Ngoài ra, bạn có thể sử dụng facade `Cookie` để "queue" cookie cho việc gán vào response trả về từ application. Phương thức `queue` chấp nhận một instance `Cookie` hoặc các tham số cần thiết để tạo một instance `Cookie`. Các cookie này sẽ được gán vào response trước khi nó được gửi về cho trình duyệt:

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

<a name="cookies-and-encryption"></a>
#### Cookies và Encryption

Mặc định, tất cả các cookie được tạo bởi Laravel đều được mã hóa và được ký để client không thể sửa đổi hoặc đọc chúng. Nếu bạn muốn tắt mã hóa cho một số cookie mà bạn tạo ra, bạn có thể sử dụng thuộc tính `$except` của middleware `App\Http\Middleware\EncryptCookies`, nằm trong thư mục `app/Http/Middleware`:

    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## Redirect

Redirect response là một instance của class `Illuminate\Http\RedirectResponse` và chứa các header cần thiết để chuyển hướng người dùng đến một URL khác. Có một số cách để tạo ra một instance `RedirectResponse`. Phương pháp đơn giản nhất là sử dụng global helper `redirect`:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

Thỉnh thoảng bạn có thể muốn chuyển hướng người dùng đến một trang trước đó, chẳng hạn như khi nhập form không hợp lệ. Bạn có thể làm như vậy bằng cách sử dụng hàm global helper `back`. Vì chức năng này sử dụng [session](/docs/{{version}}/session), nên hãy đảm bảo là route được gọi bởi hàm `back` cũng đang dùng group middleware `web` hoặc đang dùng tất cả các middleware session:

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### Redirecting đến Named Route

Khi bạn gọi helper `redirect` mà không truyền vào tham số, thì một instance của `Illuminate\Routing\Redirector` sẽ được trả về, cho phép bạn gọi bất kỳ phương thức nào trên instance `Redirector`. Ví dụ: để tạo một `RedirectResponse` cho một route mà bạn đã đặt tên, bạn có thể sử dụng phương thức` route`:

    return redirect()->route('login');

Nếu route đó có yêu cầu truyền tham số, bạn có thể truyền chúng qua tham số thứ 2 của phương thức `route`:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### Nhúng Parameter thông qua Eloquent Model

Nếu bạn đang chuyển hướng đến một route mà có tham số "ID" đang được nhúng trong một model Eloquent, bạn có thể truyền chính model đó vào làm tham số. ID sẽ được trích xuất tự động:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

Nếu bạn muốn tùy biến giá trị được lấy trong tham số route, bạn cần ghi đè phương thức `getRouteKey` trên model Eloquent của bạn:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### Redirecting đến Controller Action

Bạn cũng có thể tạo chuyển hướng đến một [controller actions](/docs/{{version}}/controllers). Để làm như vậy, hãy truyền một controller và tên action của nó cho phương thức `action`. Hãy nhớ rằng, bạn không cần thêm namespace cho controller vì `RouteServiceProvider` của Laravel đã tự động cài đặt sẵn namespace cho controller đó:

    return redirect()->action('HomeController@index');

Nếu controller route của bạn yêu cầu tham số, bạn có thể truyền chúng làm tham số thứ hai cho phương thức `action`:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### Redirecting đến Domain bên ngoài

Thỉnh thoảng bạn có thể cần chuyển hướng đến một domain ở bên ngoài application của bạn. Bạn có thể làm như vậy bằng cách gọi phương thức `Away`, phương thức này tạo ra một `RedirectResponse` mà không cần bất kỳ URL encoding, validation hoặc verification bổ sung nào:

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>
### Redirecting cùng Flashed Session Data

Chuyển hướng đến một URL mới và [flashing data tới session](/docs/{{version}}/session#flash-data) thường được thực hiện cùng một lúc. Thông thường, điều này được sử dụng sau khi thực hiện thành công một action nào đó và bạn muốn flash một mesage báo thành công đến session. Để thuận tiện, bạn có thể tạo một instance `RedirectResponse` và flash dữ liệu đó vào session với chỉ một chuỗi phương thức đơn giản:

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

Sau khi người dùng đã được chuyển hướng, bạn có thể hiển thị thông báo được flash từ [session](/docs/{{version}}/session). Ví dụ: sử dụng [Blade syntax](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## Các loại Response khác

Helper `response` có thể được sử dụng để tạo các loại instance response khác nhau. Khi helper `response` được gọi mà không có tham số, một implementation của `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts) sẽ được trả về. Contract này sẽ cung cấp một số phương thức hữu ích để tạo response.

<a name="view-responses"></a>
### View Responses

Nếu bạn cần kiểm soát trạng thái và header của response nhưng cũng cần trả về một [view](/docs/{{version}}/views) làm nội dung của response, bạn nên sử dụng phương thức `view`:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

Và dĩ nhiên, nếu bạn không cần tuỳ chỉnh HTTP status code hoặc custom header, bạn nên dùng hàm global helper `view`.

<a name="json-responses"></a>
### JSON Responses

Phương thức `json` sẽ tự động set header `Content-Type` của response thành `application/json`, cũng như chuyển đổi mảng đã cho thành JSON bằng cách sử dụng hàm PHP `json_encode`:

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

Nếu bạn muốn tạo một JSONP response, bạn có thể sử dụng phương thức `json` kết hợp với phương thức `withCallback`:

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### File Downloads

Phương thức `download` có thể được sử dụng để tạo response buộc trình duyệt của người dùng download file tại một đường dẫn đã cho. Phương thức `download` chấp nhận một tên file làm tham số thứ hai cho phương thức, nó sẽ được hiển thị khi người dùng download file. Cuối cùng, bạn có thể chuyển một mảng các HTTP header làm tham số thứ ba cho phương thức:

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

    return response()->download($pathToFile)->deleteFileAfterSend();

> {note} Quản lý file download Symfony HttpFoundation yêu cầu file download phải có tên file là ASCII.

#### Streamed Downloads

Thỉnh thoảng bạn có thể muốn biến chuỗi response của một hoạt động nhất định thành một download response mà không cần phải ghi nội dung của hoạt động đó vào disk. Bạn có thể sử dụng phương thức `streamDownload` trong trường hợp đó. Phương thức này chấp nhận một callback, một tên file và một mảng header tùy chọn làm tham số của nó:

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

<a name="file-responses"></a>
### File Responses

Phương thức `file` có thể được sử dụng để hiển thị một file, chẳng hạn như file image hoặc file PDF, cho phép xem trực tiếp ngay tại trình duyệt của người dùng thay vì phải download. Phương thức này chấp nhận đường dẫn đến file làm tham số đầu tiên và một mảng các header làm tham số thứ hai của nó:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## Response Macros

Nếu bạn muốn định nghĩa một response tùy biến mà bạn có thể sử dụng lại trong nhiều route hoặc các controller khác nhau, bạn có thể sử dụng phương thức `macro` trong facade `Response`. Ví dụ: từ phương thức `boot` trong [service provider's](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's response macros.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

Hàm `macro` chấp nhận tên hàm làm tham số đầu tiên và một Closure làm tham số thứ hai. Closure của macro sẽ được thực thi khi gọi tên của hàm macro từ implementation `ResponseFactory` hoặc helper `response`:

    return response()->caps('foo');
