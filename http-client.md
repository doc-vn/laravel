# HTTP Client

- [Giới thiệu](#introduction)
- [Tạo Request](#making-requests)
    - [Request Data](#request-data)
    - [Headers](#headers)
    - [Authentication](#authentication)
    - [Timeout](#timeout)
    - [Retries](#retries)
    - [Xử lý lỗi](#error-handling)
    - [Guzzle Options](#guzzle-options)
- [Testing](#testing)
    - [Faking Responses](#faking-responses)
    - [Kiểm tra Requests](#inspecting-requests)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một API nhỏ, rõ ràng dựa trên thư viện [Guzzle HTTP client](http://docs.guzzlephp.org/en/stable/), cho phép bạn nhanh chóng thực hiện các HTTP request giao tiếp với các ứng dụng web khác. API này tập trung vào các trường hợp sử dụng phổ biến và giúp tăng trải nghiệm tuyệt vời dành cho nhà phát triển.

Trước khi bắt đầu, bạn nên đảm bảo là bạn đã cài đặt package Guzzle vào trong ứng dụng của bạn. Mặc định, Laravel đã chứa thư viện này:

    composer require guzzlehttp/guzzle

<a name="making-requests"></a>
## Tạo Request

Để tạo request, bạn có thể sử dụng các phương thức `get`, `post`, `put`, `patch`, và `delete`. Đầu tiên, hãy xem cách tạo ra một request `GET` cơ bản:

    use Illuminate\Support\Facades\Http;

    $response = Http::get('http://test.com');

Phương thức `get` sẽ trả về một instance của `Illuminate\Http\Client\Response` và cung cấp nhiều phương thức có thể được sử dụng để kiểm tra response:

    $response->body() : string;
    $response->json() : array|mixed;
    $response->status() : int;
    $response->ok() : bool;
    $response->successful() : bool;
    $response->failed() : bool;
    $response->serverError() : bool;
    $response->clientError() : bool;
    $response->header($header) : string;
    $response->headers() : array;

Đối tượng `Illuminate\Http\Client\Response` cũng được implement từ interface `ArrayAccess` của PHP, cho phép bạn truy cập trực tiếp vào dữ liệu JSON trong response:

    return Http::get('http://test.com/users/1')['name'];

<a name="request-data"></a>
### Request Data

Tất nhiên, thông thường khi sử dụng `POST`, `PUT` và `PATCH` bạn sẽ cần gửi thêm dữ liệu vào request của bạn. Nên vì thế, các phương thức này sẽ chấp nhận thêm một mảng dữ liệu làm tham số thứ hai của chúng. Mặc định, dữ liệu sẽ được gửi theo kiểu `application/json`:

    $response = Http::post('http://test.com/users', [
        'name' => 'Steve',
        'role' => 'Network Administrator',
    ]);

#### GET Request Query Parameters

Khi thực hiện các request `GET`, bạn có thể muốn nối một chuỗi query vào sau URL hoặc truyền một mảng gồm các cặp khóa và giá trị làm tham số thứ hai cho phương thức `get`:

    $response = Http::get('http://test.com/users', [
        'name' => 'Taylor',
        'page' => 1,
    ]);

#### Sending Form URL Encoded Requests

Nếu bạn muốn gửi dữ liệu theo kiểu `application/x-www-form-urlencoded`, bạn nên gọi phương thức `asForm` trước khi tạo request của bạn:

    $response = Http::asForm()->post('http://test.com/users', [
        'name' => 'Sara',
        'role' => 'Privacy Consultant',
    ]);

#### Sending A Raw Request Body

Bạn có thể sử dụng phương thức `withBody` nếu bạn muốn đưa vào một nội dung request thô khi tạo request:

    $response = Http::withBody(
        base64_encode($photo), 'image/jpeg'
    )->post('http://test.com/photo');

#### Multi-Part Requests

Nếu bạn muốn gửi một file dưới dạng request multi-part, bạn nên gọi phương thức `attach` trước khi tạo request của bạn. Phương thức này chấp nhận tên của file và nội dung của file đó. Ngoài ra, bạn cũng có thể cung cấp thêm tham số thứ ba sẽ được coi là filename của file:

    $response = Http::attach(
        'attachment', file_get_contents('photo.jpg'), 'photo.jpg'
    )->post('http://test.com/attachments');

Thay vì truyền nội dung thô của một file, bạn cũng có thể truyền một stream resource:

    $photo = fopen('photo.jpg', 'r');

    $response = Http::attach(
        'attachment', $photo, 'photo.jpg'
    )->post('http://test.com/attachments');

<a name="headers"></a>
### Headers

Header có thể được thêm vào các request bằng phương thức `withHeaders`. Phương thức `withHeaders` này chấp nhận một mảng các cặp khóa và giá trị:

    $response = Http::withHeaders([
        'X-First' => 'foo',
        'X-Second' => 'bar'
    ])->post('http://test.com/users', [
        'name' => 'Taylor',
    ]);

<a name="authentication"></a>
### Authentication

Bạn có thể chỉ định thông tin xác thực là basic authentication hay digest authentication bằng cách sử dụng các phương thức `withBasicAuth` và `withDigestAuth`:

    // Basic authentication...
    $response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(...);

    // Digest authentication...
    $response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(...);

#### Bearer Tokens

Nếu bạn muốn thêm nhanh header `Authorization` bearer token vào trong request, bạn có thể sử dụng phương thức `withToken`:

    $response = Http::withToken('token')->post(...);

<a name="timeout"></a>
### Timeout

Phương thức `timeout` có thể được sử dụng để chỉ định số giây tối đa có thể chờ một response:

    $response = Http::timeout(3)->get(...);

Nếu thời gian chờ bị vượt quá, một instance của `Illuminate\Http\Client\ConnectionException` sẽ được đưa ra.

<a name="retries"></a>
### Retries

Nếu bạn muốn HTTP client tự động thử lại request nếu xảy ra lỗi ở phía client hoặc ở phía server, bạn có thể sử dụng phương thức `retry`. Phương thức `retry` sẽ chấp nhận tham đối số: một là số lần request có thể được thử lại và hai là số mili giây mà Laravel sẽ đợi giữa các lần thử:

    $response = Http::retry(3, 100)->post(...);

Nếu tất cả các request đều thất bại, thì một instance của `Illuminate\Http\Client\RequestException` sẽ được đưa ra.

<a name="error-handling"></a>
### Xử lý lỗi

Không giống như hành vi mặc định của thư viện Guzzle, HTTP client wrapper của Laravel sẽ không đưa ra các ngoại lệ đối với các lỗi client hoặc server (như response `400` và` 500` từ server). Bạn có thể xác định xem một trong những lỗi này có được trả về hay không bằng cách sử dụng các phương thức `successful`, `clientError`, hoặc `serverError`:

    // Determine if the status code was >= 200 and < 300...
    $response->successful();

    // Determine if the status code was >= 400...
    $response->failed();

    // Determine if the response has a 400 level status code...
    $response->clientError();

    // Determine if the response has a 500 level status code...
    $response->serverError();

#### Throwing Exceptions

Nếu bạn có một instance response và muốn đưa ra một instance `Illuminate\Http\Client\RequestException` nếu response đó là một lỗi của client hoặc server, bạn có thể sử dụng phương thức `throw`:

    $response = Http::post(...);

    // Throw an exception if a client or server error occurred...
    $response->throw();

    return $response['user']['id'];

Instance `Illuminate\Http\Client\RequestException` có một thuộc tính public là `$response` sẽ cho phép bạn kiểm tra response được trả về.

Phương thức `throw` sẽ trả về một instance response nếu như không có lỗi xảy ra, cho phép bạn kết hợp các thao tác khác nhau vào phương thức `throw`:

    return Http::post(...)->throw()->json();

<a name="guzzle-options"></a>
### Guzzle Options

Bạn có thể chỉ định thêm các [tuỳ chọn Guzzle request](http://docs.guzzlephp.org/en/stable/request-options.html) bằng cách sử dụng phương thức `withOptions`. Phương thức `withOptions` sẽ chấp nhận một mảng gồm các cặp khóa và giá trị:

    $response = Http::withOptions([
        'debug' => true,
    ])->get('http://test.com/users');

<a name="testing"></a>
## Testing

Nhiều service của Laravel cung cấp các chức năng giúp bạn viết các bài test một cách dễ dàng và rõ ràng, và HTTP client wrapper của Laravel cũng không phải là một ngoại lệ. Phương thức `fake` của facade `Http` cho phép bạn hướng dẫn HTTP client trả về một response stubbed / dummy khi một request được tạo.

<a name="faking-responses"></a>
### Faking Responses

Ví dụ: để hướng dẫn HTTP client trả về một response trống và có status code `200` cho mọi request, bạn có thể gọi phương thức `fake` với không tham số truyền vào:

    use Illuminate\Support\Facades\Http;

    Http::fake();

    $response = Http::post(...);

#### Faking Specific URLs

Ngoài ra, bạn cũng có thể truyền một mảng cho phương thức `fake`. Các khóa của mảng phải đại diện cho các pattern URL mà bạn muốn làm fake còn các giá trị là các response tương ứng với chúng. Ký tự `*` có thể được sử dụng làm ký tự đại diện. Bất kỳ request nào được tạo với một URL không bị fake sẽ được thực thi ngay lập tức. Bạn có thể sử dụng phương thức `response` để tạo các response stub / fake cho các endpoint này:

    Http::fake([
        // Stub a JSON response for GitHub endpoints...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Stub a string response for Google endpoints...
        'google.com/*' => Http::response('Hello World', 200, ['Headers']),
    ]);

Nếu bạn muốn chỉ định một pattern URL dự phòng sẽ được dùng cho tất cả các URL mà chưa khớp với các pattern URL đã cho, bạn có thể sử dụng một ký tự `*` để cài đặt chuyện đó:

    Http::fake([
        // Stub a JSON response for GitHub endpoints...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Stub a string response for all other endpoints...
        '*' => Http::response('Hello World', 200, ['Headers']),
    ]);

#### Faking Response Sequences

Đôi khi bạn có thể cần chỉ định là một URL sẽ trả về một loạt các response fake theo một trình tự cụ thể. Bạn có thể thực hiện điều này bằng cách sử dụng phương thức `Http::sequence` để xây dựng các response:

    Http::fake([
        // Stub a series of responses for GitHub endpoints...
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->pushStatus(404),
    ]);

Khi tất cả các response trong một trình tự response đã được sử dụng xong, thì bất kỳ request nào khác sẽ khiến trình tự response sẽ đưa ra một ngoại lệ. Nếu bạn muốn chỉ định một response mặc định sẽ được trả về khi một trình chạy xong, bạn có thể sử dụng phương thức `whenEmpty`:

    Http::fake([
        // Stub a series of responses for GitHub endpoints...
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->whenEmpty(Http::response()),
    ]);

Nếu bạn muốn fake một trình tự response nhưng không muốn chỉ định pattern URL nào sẽ được làm fake, bạn có thể sử dụng phương thức `Http::fakeSequence`:

    Http::fakeSequence()
            ->push('Hello World', 200)
            ->whenEmpty(Http::response());

#### Fake Callback

Nếu bạn yêu cầu một logic phức tạp hơn để xác định response nào sẽ trả về cho một số endpoint nhất định, bạn có thể truyền voà một lệnh callback cho phương thức `fake`. Lệnh callback này sẽ nhận vào một instance của `Illuminate\Http\Client\Request` và sẽ trả về một instance response:

    Http::fake(function ($request) {
        return Http::response('Hello World', 200);
    });

<a name="inspecting-requests"></a>
### Kiểm tra Requests

Khi fake response, đôi khi bạn có thể muốn kiểm tra các request mà client nhận được để đảm bảo là ứng dụng của bạn đang gửi dữ liệu hoặc tiêu đề chính xác. Bạn có thể thực hiện điều này bằng cách gọi phương thức `Http::assertSent` sau khi gọi `Http::fake`.

Phương thức `assertSent` sẽ chấp nhận một callback sẽ được cung cấp một instance `Illuminate\Http\Client\Request` và sẽ trả về một giá trị boolean cho biết là request có phù hợp với mong đợi của bạn hay không. Để pass qua bài test, thì ít nhất một request phải phù hợp với các kỳ vọng mà bạn đưa ra:

    Http::fake();

    Http::withHeaders([
        'X-First' => 'foo',
    ])->post('http://test.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertSent(function ($request) {
        return $request->hasHeader('X-First', 'foo') &&
               $request->url() == 'http://test.com/users' &&
               $request['name'] == 'Taylor' &&
               $request['role'] == 'Developer';
    });

Nếu cần thiết, bạn có thể kiểm tra rằng một request sẽ không được gửi đi bằng phương thức `assertNotSent`:

    Http::fake();

    Http::post('http://test.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertNotSent(function (Request $request) {
        return $request->url() === 'http://test.com/posts';
    });

Hoặc, nếu bạn muốn kiểm tra rằng sẽ không có request nào được phép gửi đi, bạn có thể sử dụng phương thức `assertNothingSent`:

    Http::fake();

    Http::assertNothingSent();
