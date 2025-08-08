# HTTP Client

- [Giới thiệu](#introduction)
- [Tạo Request](#making-requests)
    - [Request Data](#request-data)
    - [Headers](#headers)
    - [Authentication](#authentication)
    - [Timeout](#timeout)
    - [Retries](#retries)
    - [Xử lý lỗi](#error-handling)
    - [Guzzle Middleware](#guzzle-middleware)
    - [Guzzle Options](#guzzle-options)
- [Request đồng thời](#concurrent-requests)
- [Macros](#macros)
- [Testing](#testing)
    - [Faking Responses](#faking-responses)
    - [Kiểm tra Requests](#inspecting-requests)
    - [Chặn request đi lạc](#preventing-stray-requests)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một API nhỏ, rõ ràng dựa trên thư viện [Guzzle HTTP client](http://docs.guzzlephp.org/en/stable/), cho phép bạn nhanh chóng thực hiện các HTTP request giao tiếp với các ứng dụng web khác. API này tập trung vào các trường hợp sử dụng phổ biến và giúp tăng trải nghiệm tuyệt vời dành cho nhà phát triển.

Trước khi bắt đầu, bạn nên đảm bảo là bạn đã cài đặt package Guzzle vào trong ứng dụng của bạn. Mặc định, Laravel đã chứa thư viện này. Tuy nhiên, nếu trước đó bạn đã gỡ package này ra rồi, thì bạn có thể cài đặt lại package này qua Composer:

```shell
composer require guzzlehttp/guzzle
```

<a name="making-requests"></a>
## Tạo Request

Để tạo request, bạn có thể sử dụng các phương thức `head`, `get`, `post`, `put`, `patch`, và `delete` được cung cấp bởi facade `Http`. Đầu tiên, hãy xem cách tạo ra một request `GET` cơ bản to another URL:

    use Illuminate\Support\Facades\Http;

    $response = Http::get('http://example.com');

Phương thức `get` sẽ trả về một instance của `Illuminate\Http\Client\Response` và cung cấp nhiều phương thức có thể được sử dụng để kiểm tra response:

    $response->body() : string;
    $response->json($key = null, $default = null) : array|mixed;
    $response->object() : object;
    $response->collect($key = null) : Illuminate\Support\Collection;
    $response->status() : int;
    $response->successful() : bool;
    $response->redirect(): bool;
    $response->failed() : bool;
    $response->clientError() : bool;
    $response->header($header) : string;
    $response->headers() : array;

Đối tượng `Illuminate\Http\Client\Response` cũng được implement từ interface `ArrayAccess` của PHP, cho phép bạn truy cập trực tiếp vào dữ liệu JSON trong response:

    return Http::get('http://example.com/users/1')['name'];

Ngoài các phương thức response được liệt kê ở trên, các phương thức sau có thể được sử dụng để xác định xem response có mã status code nhất định hay không:

    $response->ok() : bool;                  // 200 OK
    $response->created() : bool;             // 201 Created
    $response->accepted() : bool;            // 202 Accepted
    $response->noContent() : bool;           // 204 No Content
    $response->movedPermanently() : bool;    // 301 Moved Permanently
    $response->found() : bool;               // 302 Found
    $response->badRequest() : bool;          // 400 Bad Request
    $response->unauthorized() : bool;        // 401 Unauthorized
    $response->paymentRequired() : bool;     // 402 Payment Required
    $response->forbidden() : bool;           // 403 Forbidden
    $response->notFound() : bool;            // 404 Not Found
    $response->requestTimeout() : bool;      // 408 Request Timeout
    $response->conflict() : bool;            // 409 Conflict
    $response->unprocessableEntity() : bool; // 422 Unprocessable Entity
    $response->tooManyRequests() : bool;     // 429 Too Many Requests
    $response->serverError() : bool;         // 500 Internal Server Error

<a name="uri-templates"></a>
#### URI Templates

HTTP client cũng cho phép bạn khởi tạo các URL request bằng cách sử dụng [URI template](https://www.rfc-editor.org/rfc/rfc6570). Để định nghĩa thêm các tham số URL bằng URI template, bạn có thể sử dụng phương thức `withUrlParameters`:

```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '9.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

<a name="dumping-requests"></a>
#### Dumping Requests

Nếu bạn muốn dump ra instance request trước khi nó được gửi đi và dừng quá trình chạy lại, bạn có thể thêm phương thức `dd` vào đầu định nghĩa request của bạn:

    return Http::dd()->get('http://example.com');

<a name="request-data"></a>
### Request Data

Tất nhiên, thông thường khi tạo request `POST`, `PUT` và `PATCH` bạn sẽ cần gửi thêm dữ liệu vào request của bạn, vì thế, các phương thức này sẽ chấp nhận thêm một mảng dữ liệu làm tham số thứ hai của chúng. Mặc định, dữ liệu sẽ được gửi theo kiểu `application/json`:

    use Illuminate\Support\Facades\Http;

    $response = Http::post('http://example.com/users', [
        'name' => 'Steve',
        'role' => 'Network Administrator',
    ]);

<a name="get-request-query-parameters"></a>
#### GET Request Query Parameters

Khi thực hiện các request `GET`, bạn có thể muốn nối một chuỗi query vào sau URL hoặc truyền một mảng gồm các cặp khóa và giá trị làm tham số thứ hai cho phương thức `get`:

    $response = Http::get('http://example.com/users', [
        'name' => 'Taylor',
        'page' => 1,
    ]);

Ngoài ra, phương thức `withQueryParameters` cũng có thể được sử dụng:

    Http::retry(3, 100)->withQueryParameters([
        'name' => 'Taylor',
        'page' => 1,
    ])->get('http://example.com/users')

<a name="sending-form-url-encoded-requests"></a>
#### Sending Form URL Encoded Requests

Nếu bạn muốn gửi dữ liệu theo kiểu `application/x-www-form-urlencoded`, bạn nên gọi phương thức `asForm` trước khi tạo request của bạn:

    $response = Http::asForm()->post('http://example.com/users', [
        'name' => 'Sara',
        'role' => 'Privacy Consultant',
    ]);

<a name="sending-a-raw-request-body"></a>
#### Sending A Raw Request Body

Bạn có thể sử dụng phương thức `withBody` nếu bạn muốn đưa vào một nội dung request thô khi tạo request. Content type cũng có thể được cung cấp thông qua tham số thứ hai của phương thức:

    $response = Http::withBody(
        base64_encode($photo), 'image/jpeg'
    )->post('http://example.com/photo');

<a name="multi-part-requests"></a>
#### Multi-Part Requests

Nếu bạn muốn gửi một file dưới dạng request multi-part, bạn nên gọi phương thức `attach` trước khi tạo request của bạn. Phương thức này chấp nhận tên của file và nội dung của file đó. Nếu cần, bạn cũng có thể cung cấp thêm tham số thứ ba sẽ được coi là filename của file, trong khi tham số thứ tư có thể được sử dụng để cung cấp các header liên quan đến file:

    $response = Http::attach(
        'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
    )->post('http://example.com/attachments');

Thay vì truyền nội dung thô của một file, bạn có thể truyền một stream resource:

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
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
    ]);

Bạn có thể sử dụng phương thức `accept` để chỉ định content type nào mà ứng dụng của bạn mong đợi để đáp ứng yêu cầu của bạn:

    $response = Http::accept('application/json')->get('http://example.com/users');

Để thuận tiện, bạn có thể sử dụng phương thức `acceptJson` để nhanh chóng xác định rằng ứng dụng của bạn mong đợi content type `application/json` sẽ đáp ứng yêu cầu của bạn:

    $response = Http::acceptJson()->get('http://example.com/users');

Phương thức `withHeaders` sẽ nối các header mới vào các header đã có của request. Nếu cần, bạn có thể thay đổi toàn bộ các header bằng phương thức `replaceHeaders`:

```php
$response = Http::withHeaders([
    'X-Original' => 'foo',
])->replaceHeaders([
    'X-Replacement' => 'bar',
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

<a name="authentication"></a>
### Authentication

Bạn có thể chỉ định thông tin xác thực là basic authentication hay digest authentication bằng cách sử dụng các phương thức `withBasicAuth` và `withDigestAuth`:

    // Basic authentication...
    $response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(/* ... */);

    // Digest authentication...
    $response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(/* ... */);

<a name="bearer-tokens"></a>
#### Bearer Tokens

Nếu bạn muốn thêm nhanh header `Authorization` bearer token vào trong header `Authorization` của request, bạn có thể sử dụng phương thức `withToken`:

    $response = Http::withToken('token')->post(/* ... */);

<a name="timeout"></a>
### Timeout

Phương thức `timeout` có thể được sử dụng để chỉ định số giây tối đa có thể chờ một response. Mặc định, HTTP client sẽ đợi response trong 30 giây:

    $response = Http::timeout(3)->get(/* ... */);

Nếu thời gian chờ bị vượt quá, một instance của `Illuminate\Http\Client\ConnectionException` sẽ được đưa ra.

Bạn có thể chỉ định thời gian timeout trong khi kết nối với server bằng phương thức `connectTimeout`:

    $response = Http::connectTimeout(3)->get(/* ... */);

<a name="retries"></a>
### Retries

Nếu bạn muốn HTTP client tự động thử lại request nếu xảy ra lỗi ở phía client hoặc ở phía server, bạn có thể sử dụng phương thức `retry`. Phương thức `retry` sẽ chấp nhận hai tham số: một là số lần request tối đa có thể được thử lại và hai là số mili giây mà Laravel sẽ đợi giữa các lần thử:

    $response = Http::retry(3, 100)->post(/* ... */);

Nếu bạn muốn tự điều chỉnh số mili giây chờ đợi giữa các lần thử, bạn có thể truyền một closure làm tham số thứ hai cho phương thức `retry`:

    use Exception;

    $response = Http::retry(3, function (int $attempt, Exception $exception) {
        return $attempt * 100;
    })->post(/* ... */);

Để thuận tiện, bạn cũng có thể cung cấp một mảng làm tham số đầu tiên cho phương thức `retry`. Mảng này sẽ được sử dụng để xác định xem số mili giây chờ đợi giữa các lần thử tiếp theo:

    $response = Http::retry([100, 200])->post(/* ... */);

Nếu cần, bạn có thể truyền tham số thứ ba cho phương thức `retry`. Tham số thứ ba phải là một tham số callable để xác định xem có thực sự nên thử lại hay không. Ví dụ: bạn có thể chỉ muốn thử lại request nếu request ban đầu gặp phải lỗi `ConnectionException`:

    use Exception;
    use Illuminate\Http\Client\PendingRequest;

    $response = Http::retry(3, 100, function (Exception $exception, PendingRequest $request) {
        return $exception instanceof ConnectionException;
    })->post(/* ... */);

Nếu lần thử của request bị thất bại, bạn có thể muốn thực hiện một thay đổi đối với request trước khi nó được thực hiện lại. Bạn có thể đạt được điều này bằng cách sửa tham số request được cung cấp cho lệnh callable mà bạn đã cung cấp cho phương thức `retry`. Ví dụ: bạn có thể muốn thử lại request bằng một mã authorization token mới nếu lần thử đầu tiên trả về lỗi authentication:

    use Exception;
    use Illuminate\Http\Client\PendingRequest;
    use Illuminate\Http\Client\RequestException;

    $response = Http::withToken($this->getToken())->retry(2, 0, function (Exception $exception, PendingRequest $request) {
        if (! $exception instanceof RequestException || $exception->response->status() !== 401) {
            return false;
        }

        $request->withToken($this->getNewToken());

        return true;
    })->post(/* ... */);

Nếu tất cả các request đều thất bại, thì một instance của `Illuminate\Http\Client\RequestException` sẽ được đưa ra. Nếu bạn muốn disable hành động này, bạn có thể cung cấp tham số `throw` với giá trị `false`. Khi bị disable, response cuối cùng mà client nhận được sẽ được trả về sau khi tất cả các lần thử lại được thực hiện:

    $response = Http::retry(3, 100, throw: false)->post(/* ... */);

> [!WARNING]
> Nếu tất cả các request đều không thành công do một sự cố kết nối, thì `Illuminate\Http\Client\ConnectionException` vẫn sẽ được đưa ra ngay cả khi tham số `throw` được set thành `false`.

<a name="error-handling"></a>
### Xử lý lỗi

Không giống như hành vi mặc định của thư viện Guzzle, HTTP client wrapper của Laravel sẽ không đưa ra các ngoại lệ đối với các lỗi client hoặc server (như response `400` và` 500` từ server). Bạn có thể xác định xem một trong những lỗi này có được trả về hay không bằng cách sử dụng các phương thức `successful`, `clientError`, hoặc `serverError`:

    // Determine if the status code is >= 200 and < 300...
    $response->successful();

    // Determine if the status code is >= 400...
    $response->failed();

    // Determine if the response has a 400 level status code...
    $response->clientError();

    // Determine if the response has a 500 level status code...
    $response->serverError();

    // Immediately execute the given callback if there was a client or server error...
    $response->onError(callable $callback);

<a name="throwing-exceptions"></a>
#### Throwing Exceptions

Nếu bạn có một instance response và muốn đưa ra một instance `Illuminate\Http\Client\RequestException` nếu response status code trả về là một lỗi của client hoặc là của server, bạn có thể sử dụng phương thức `throw` hoặc `throwIf`:

    use Illuminate\Http\Client\Response;

    $response = Http::post(/* ... */);

    // Throw an exception if a client or server error occurred...
    $response->throw();

    // Throw an exception if an error occurred and the given condition is true...
    $response->throwIf($condition);

    // Throw an exception if an error occurred and the given closure resolves to true...
    $response->throwIf(fn (Response $response) => true);

    // Throw an exception if an error occurred and the given condition is false...
    $response->throwUnless($condition);

    // Throw an exception if an error occurred and the given closure resolves to false...
    $response->throwUnless(fn (Response $response) => false);

    // Throw an exception if the response has a specific status code...
    $response->throwIfStatus(403);

    // Throw an exception unless the response has a specific status code...
    $response->throwUnlessStatus(200);

    return $response['user']['id'];

Instance `Illuminate\Http\Client\RequestException` có một thuộc tính public là `$response` sẽ cho phép bạn kiểm tra response được trả về.

Phương thức `throw` sẽ trả về một instance response nếu như không có lỗi xảy ra, cho phép bạn kết hợp các thao tác khác nhau vào phương thức `throw`:

    return Http::post(/* ... */)->throw()->json();

Nếu bạn muốn thực hiện một số logic bổ sung trước khi đưa ra exception, bạn có thể truyền một closure cho phương thức `throw`. Exception này sẽ được đưa ra tự động sau khi closure được gọi, do đó bạn không cần phải đưa lại exception này từ bên trong closure:

    use Illuminate\Http\Client\Response;
    use Illuminate\Http\Client\RequestException;

    return Http::post(/* ... */)->throw(function (Response $response, RequestException $e) {
        // ...
    })->json();

<a name="guzzle-middleware"></a>
### Guzzle Middleware

Vì HTTP client của Laravel được cung cấp bởi Guzzle, nên bạn có thể tận dụng [Guzzle Middleware](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html) để thao tác với các request gửi đi hoặc kiểm tra response gửi đến. Để xử lý request gửi đi, hãy đăng ký một Guzzle middleware thông qua phương thức `withRequestMiddleware`:

    use Illuminate\Support\Facades\Http;
    use Psr\Http\Message\RequestInterface;

    $response = Http::withRequestMiddleware(
        function (RequestInterface $request) {
            return $request->withHeader('X-Example', 'Value');
        }
    )->get('http://example.com');

Tương tự, bạn có thể kiểm tra response HTTP phản hồi bằng cách đăng ký một middleware thông qua phương thức `withResponseMiddleware`:

    use Illuminate\Support\Facades\Http;
    use Psr\Http\Message\ResponseInterface;

    $response = Http::withResponseMiddleware(
        function (ResponseInterface $response) {
            $header = $response->getHeader('X-Example');

            // ...

            return $response;
        }
    )->get('http://example.com');

<a name="global-middleware"></a>
#### Global Middleware

Thỉnh thoảng, bạn có thể muốn đăng ký một middleware áp dụng cho mọi request được gửi đi và response được nhận về. Để thực hiện điều này, bạn có thể sử dụng các phương thức `globalRequestMiddleware` và `globalResponseMiddleware`. Thông thường, các phương thức này phải được gọi trong phương thức `boot` của `AppServiceProvider` trong ứng dụng của bạn:

```php
use Illuminate\Support\Facades\Http;

Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));

Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

<a name="guzzle-options"></a>
### Guzzle Options

Bạn có thể chỉ định thêm các [tuỳ chọn Guzzle request](http://docs.guzzlephp.org/en/stable/request-options.html) bằng cách sử dụng phương thức `withOptions`. Phương thức `withOptions` sẽ chấp nhận một mảng gồm các cặp khóa và giá trị:

    $response = Http::withOptions([
        'debug' => true,
    ])->get('http://example.com/users');

<a name="concurrent-requests"></a>
## Request đồng thời

Thỉnh thoảng, bạn có thể muốn thực hiện nhiều request HTTP cùng một lúc. Nói cách khác, bạn muốn một số request được gửi đi cùng lúc thay vì gửi đi các request một cách tuần tự. Điều này có thể giúp cải thiện hiệu suất đáng kể khi tương tác với các API HTTP chậm.

Rất may, bạn có thể thực hiện việc này bằng phương thức `pool`. Phương thức `pool` chấp nhận một closure nhận instance `Illuminate\Http\Client\Pool`, cho phép bạn dễ dàng thêm request vào một nhóm request để gửi đi:

    use Illuminate\Http\Client\Pool;
    use Illuminate\Support\Facades\Http;

    $responses = Http::pool(fn (Pool $pool) => [
        $pool->get('http://localhost/first'),
        $pool->get('http://localhost/second'),
        $pool->get('http://localhost/third'),
    ]);

    return $responses[0]->ok() &&
           $responses[1]->ok() &&
           $responses[2]->ok();

Như bạn có thể thấy, mỗi instance response có thể được truy cập dựa trên thứ tự của nó được thêm vào nhóm. Nếu muốn, bạn có thể đặt tên cho các request của bạn bằng phương thức `as`, phương thức này cho phép bạn truy cập các response tương ứng theo tên:

    use Illuminate\Http\Client\Pool;
    use Illuminate\Support\Facades\Http;

    $responses = Http::pool(fn (Pool $pool) => [
        $pool->as('first')->get('http://localhost/first'),
        $pool->as('second')->get('http://localhost/second'),
        $pool->as('third')->get('http://localhost/third'),
    ]);

    return $responses['first']->ok();

<a name="customizing-concurrent-requests"></a>
#### Customizing Concurrent Requests

Phương thức `pool` sẽ không thể nối với các phương thức HTTP client khác như phương thức `withHeaders` hoặc `middleware`. Nếu bạn muốn thực hiện một header tùy chỉnh hoặc middleware cho các request có trong nhóm, bạn cần cấu hình các tùy chọn đó trong mỗi request trong nhóm đó:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$headers = [
    'X-Example' => 'example',
];

$responses = Http::pool(fn (Pool $pool) => [
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
]);
```

<a name="macros"></a>
## Macros

Laravel HTTP client cho phép bạn định nghĩa "macro", cái mà có thể hoạt động như một cơ chế hiệu quả tuyệt vời để cấu hình các path và các header cho một request chung khi nó tương tác với các service trong ứng dụng của bạn. Để bắt đầu, bạn có thể định nghĩa một macro trong phương thức `boot` của class `App\Providers\AppServiceProvider` trong ứng dụng của bạn:

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::macro('github', function () {
        return Http::withHeaders([
            'X-Example' => 'example',
        ])->baseUrl('https://github.com');
    });
}
```

Khi macro của bạn đã được cấu hình xong, bạn có thể gọi nó từ bất kỳ đâu trong ứng dụng của bạn để tạo ra một pending request với cấu hình đã chỉ định:

```php
$response = Http::github()->get('/');
```

<a name="testing"></a>
## Testing

Nhiều service của Laravel cung cấp các chức năng giúp bạn viết các bài test một cách dễ dàng và rõ ràng, và HTTP client của Laravel cũng không phải là một ngoại lệ. Phương thức `fake` của facade `Http` cho phép bạn hướng dẫn HTTP client trả về một response stubbed / dummy khi một request được tạo.

<a name="faking-responses"></a>
### Faking Responses

Ví dụ: để hướng dẫn HTTP client trả về một response trống và có status code `200` cho mọi request, bạn có thể gọi phương thức `fake` với không tham số truyền vào:

    use Illuminate\Support\Facades\Http;

    Http::fake();

    $response = Http::post(/* ... */);

<a name="faking-specific-urls"></a>
#### Faking Specific URLs

Ngoài ra, bạn cũng có thể truyền một mảng cho phương thức `fake`. Các khóa của mảng phải đại diện cho các pattern URL mà bạn muốn làm fake còn các giá trị là các response tương ứng với chúng. Ký tự `*` có thể được sử dụng làm ký tự đại diện. Bất kỳ request nào được tạo với một URL không bị fake sẽ được thực thi ngay lập tức. Bạn có thể sử dụng phương thức `response` của facade `Http` để tạo các response stub / fake cho các endpoint này:

    Http::fake([
        // Stub a JSON response for GitHub endpoints...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),

        // Stub a string response for Google endpoints...
        'google.com/*' => Http::response('Hello World', 200, $headers),
    ]);

Nếu bạn muốn chỉ định một pattern URL dự phòng sẽ được dùng cho tất cả các URL mà chưa khớp với các pattern URL đã cho, bạn có thể sử dụng một ký tự `*` để cài đặt chuyện đó:

    Http::fake([
        // Stub a JSON response for GitHub endpoints...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Stub a string response for all other endpoints...
        '*' => Http::response('Hello World', 200, ['Headers']),
    ]);

<a name="faking-response-sequences"></a>
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

<a name="fake-callback"></a>
#### Fake Callback

Nếu bạn yêu cầu một logic phức tạp hơn để xác định response nào sẽ trả về cho một số endpoint nhất định, bạn có thể truyền voà một lệnh closure cho phương thức `fake`. Lệnh closure này sẽ nhận vào một instance của `Illuminate\Http\Client\Request` và sẽ trả về một instance response. Trong closure của bạn, bạn có thể thực hiện bất kỳ logic nào cần thiết để xác định loại response nào sẽ trả về:

    use Illuminate\Http\Client\Request;

    Http::fake(function (Request $request) {
        return Http::response('Hello World', 200);
    });

<a name="preventing-stray-requests"></a>
### Chặn request đi lạc

Nếu bạn muốn đảm bảo rằng tất cả các request được gửi qua  HTTP client sẽ bị fake trong suốt quá trình test, bạn có thể gọi phương thức `preventStrayRequests`. Sau khi gọi phương thức này, mọi request không có một fake response thì nó sẽ đưa ra một ngoại lệ thay vì thực hiện một request HTTP thực tế:

    use Illuminate\Support\Facades\Http;

    Http::preventStrayRequests();

    Http::fake([
        'github.com/*' => Http::response('ok'),
    ]);

    // An "ok" response is returned...
    Http::get('https://github.com/laravel/framework');

    // An exception is thrown...
    Http::get('https://laravel.com');

<a name="inspecting-requests"></a>
### Kiểm tra Requests

Khi fake response, đôi khi bạn có thể muốn kiểm tra các request mà client nhận được để đảm bảo là ứng dụng của bạn đang gửi dữ liệu hoặc tiêu đề chính xác. Bạn có thể thực hiện điều này bằng cách gọi phương thức `Http::assertSent` sau khi gọi `Http::fake`.

Phương thức `assertSent` sẽ chấp nhận một closure sẽ nhận vào một instance `Illuminate\Http\Client\Request` và sẽ trả về một giá trị boolean cho biết là request có phù hợp với mong đợi của bạn hay không. Để pass qua bài test, thì ít nhất một request phải phù hợp với các kỳ vọng mà bạn đưa ra:

    use Illuminate\Http\Client\Request;
    use Illuminate\Support\Facades\Http;

    Http::fake();

    Http::withHeaders([
        'X-First' => 'foo',
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertSent(function (Request $request) {
        return $request->hasHeader('X-First', 'foo') &&
               $request->url() == 'http://example.com/users' &&
               $request['name'] == 'Taylor' &&
               $request['role'] == 'Developer';
    });

Nếu cần thiết, bạn có thể kiểm tra rằng một request sẽ không được gửi đi bằng phương thức `assertNotSent`:

    use Illuminate\Http\Client\Request;
    use Illuminate\Support\Facades\Http;

    Http::fake();

    Http::post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertNotSent(function (Request $request) {
        return $request->url() === 'http://example.com/posts';
    });

Bạn có thể sử dụng phương thức `assertSentCount` để kiểm tra có bao nhiêu request đã được "gửi" trong quá trình test:

    Http::fake();

    Http::assertSentCount(5);

Hoặc, bạn có thể sử dụng phương thức `assertNothingSent` để kiểm tra không có request nào được gửi đi trong quá trình test:

    Http::fake();

    Http::assertNothingSent();

<a name="recording-requests-and-responses"></a>
#### Recording Requests / Responses

Bạn có thể sử dụng phương thức `recorded` để thu nhận tất cả các request và các response tương ứng của chúng. Phương thức `recorded` sẽ trả về một collection các mảng chứa các instance của `Illuminate\Http\Client\Request` và `Illuminate\Http\Client\Response`:

```php
Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded();

[$request, $response] = $recorded[0];
```

Ngoài ra, phương thức `recorded` sẽ cũng chấp nhận một closure sẽ nhận vào một instance của `Illuminate\Http\Client\Request` và `Illuminate\Http\Client\Response` và có thể được sử dụng để lọc các cặp request / response dựa trên mong đợi của bạn:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Http\Client\Response;

Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded(function (Request $request, Response $response) {
    return $request->url() !== 'https://laravel.com' &&
           $response->successful();
});
```

<a name="events"></a>
## Events

Laravel kích hoạt ba event trong quá trình gửi request HTTP. Event `RequestSending` sẽ được kích hoạt trước khi request được gửi đi, trong khi event `ResponseReceived` sẽ được kích hoạt sau khi nhận được phản hồi cho một request nhất định. Và event `ConnectionFailed` sẽ được kích hoạt nếu không nhận được phản hồi nào cho một request nhất định.

Cả hai event `RequestSending` và `ConnectionFailed` đều chứa thuộc tính public `$request` mà bạn có thể sử dụng để kiểm tra instance `Illuminate\Http\Client\Request`. Tương tự, event `ResponseReceived` cũng chứa thuộc tính `$request` cũng như thuộc tính `$response` có thể được sử dụng để kiểm tra instance `Illuminate\Http\Client\Response`. Bạn cũng có thể đăng ký event listener cho event này trong service provider `App\Providers\EventServiceProvider` của bạn:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Http\Client\Events\RequestSending' => [
            'App\Listeners\LogRequestSending',
        ],
        'Illuminate\Http\Client\Events\ResponseReceived' => [
            'App\Listeners\LogResponseReceived',
        ],
        'Illuminate\Http\Client\Events\ConnectionFailed' => [
            'App\Listeners\LogConnectionFailed',
        ],
    ];
