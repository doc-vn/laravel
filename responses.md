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
- [Streamed Responses](#streamed-responses)
    - [Consuming Streamed Responses](#consuming-streamed-responses)
    - [Streamed JSON Responses](#streamed-json-responses)
    - [Event Streams (SSE)](#event-streams)
    - [Streamed Downloads](#streamed-downloads)
- [Response Macros](#response-macros)

<a name="creating-responses"></a>
## Tạo Responses

<a name="strings-arrays"></a>
#### Strings và Arrays

Tất cả các route và các controller đều sẽ trả về một response và được gửi về cho trình duyệt web của người dùng. Laravel cung cấp một số cách khác nhau để trả về một response. Response cơ bản nhất là trả về một chuỗi từ một route hoặc một controller. Framework sẽ tự động chuyển đổi chuỗi đó thành một HTTP response đầy đủ:

```php
Route::get('/', function () {
    return 'Hello World';
});
```

Ngoài việc trả về một chuỗi từ route và controller của bạn, bạn cũng có thể trả về một mảng. Framework sẽ tự động chuyển mảng đó thành một JSON response:

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

> [!NOTE]
> Bạn có biết rằng bạn cũng có thể trả về [Eloquent collections](/docs/{{version}}/eloquent-collections) từ một route hoặc một controller của bạn không? Chúng sẽ tự động được chuyển đổi thành JSON. Bạn cứ thử đi!

<a name="response-objects"></a>
#### Response Objects

Thông thường, bạn sẽ không chỉ trả về một chuỗi hoặc một mảng từ route action của bạn. Thay vào đó, bạn có thể sẽ muốn trả lại cả một instance `Illuminate\Http\Response` hoặc một [views](/docs/{{version}}/views).

Trả về cả một instance `Response` cho phép bạn tùy biến status code và header của response's HTTP. Một instance `Response` sẽ được extend từ class `Symfony\Component\HttpFoundation\Response`, cung cấp nhiều phương thức để xây dựng các HTTP response:

```php
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

<a name="eloquent-models-and-collections"></a>
#### Eloquent Models và Collections

Bạn cũng có thể trả về các model và collection [Eloquent ORM](/docs/{{version}}/eloquent) trực tiếp từ các route và controller của bạn. Khi bạn làm như vậy, Laravel sẽ tự động chuyển đổi các model và collection thành JSON response trong khi vẫn giữ các [thuộc tính ẩn](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json):

```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

<a name="attaching-headers-to-responses"></a>
### Gắn Header vào Responses

Hãy nhớ rằng hầu hết các phương thức response đều có thể kết hợp lại với nhau, cho phép bạn dễ dàng khởi tạo một response instance. Ví dụ, bạn có thể sử dụng phương thức `header` để thêm một danh sách header cho response trước khi gửi chúng về cho người dùng:

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

Hoặc, bạn có thể sử dụng phương thức `withHeaders` để chỉ định một mảng các header sẽ được thêm vào response:

```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

<a name="cache-control-middleware"></a>
#### Cache Control Middleware

Laravel có chứa một middleware `cache.headers`, middleware này có thể được sử dụng để thiết lập nhanh một header `Cache-Control` cho một nhóm các route. Các nội dung được cung cấp cho header này có thể được chỉ thị bằng cách sử dụng "snake case" tương ứng với chỉ thị corresponding cache-control và phải được phân tách bằng dấu chấm phẩy. Nếu `etag` được chỉ định trong danh sách lệnh, một hash MD5 của nội dung response sẽ được tự động set làm ETag identifier:

```php
Route::middleware('cache.headers:public;max_age=30;s_maxage=300;stale_while_revalidate=600;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

<a name="attaching-cookies-to-responses"></a>
### Gắn Cookies vào Responses

Bạn có thể đính kèm cookie vào instance `Illuminate\Http\Response` được gửi đi bằng phương thức `cookie`. Bạn nên truyền tên, giá trị và số phút hết hạn tới phương thức này:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

Phương thức `cookie` cũng cho phép thêm một vài tham số ít sử dụng hơn. Nói chung, các tham số này có cùng mục đích và ý nghĩa như các tham số được cung cấp bởi phương thức [setcookie](https://secure.php.net/manual/en/function.setcookie.php) của PHP:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

Nếu bạn muốn đảm bảo rằng cookie được gửi cùng với response khi trả về nhưng bạn chưa có instance của response trả về đó, bạn có thể sử dụng facade `Cookie` để "queue" cookie cho việc gán vào response khi nó được trả về từ application. Phương thức `queue` chấp nhận một instance `Cookie` hoặc các tham số cần thiết để tạo một instance `Cookie`. Các cookie này sẽ được gán vào response trước khi nó được gửi về cho trình duyệt:

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

<a name="generating-cookie-instances"></a>
#### Generating Cookie Instances

Nếu bạn muốn tạo một instance `Symfony\Component\HttpFoundation\Cookie` có thể được gắn vào một instance response sau này, bạn có thể sử dụng global helper `cookie`. Cookie này sẽ không được gửi về cho khách hàng trừ khi nó được gán với một instance response:

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="expiring-cookies-early"></a>
#### Expiring Cookies Early

Bạn có thể xóa một cookie bằng cách làm nó hết hạn thông qua phương thức `withoutCookie` của response:

```php
return response('Hello World')->withoutCookie('name');
```

Nếu bạn chưa có instance của response, bạn có thể sử dụng phương thức `expire` của facade `Cookie` để hết hạn cookie:

```php
Cookie::expire('name');
```

<a name="cookies-and-encryption"></a>
### Cookies và Encryption

Mặc định, nhờ vào middleware `Illuminate\Cookie\Middleware\EncryptCookies`, tất cả các cookie được tạo bởi Laravel đều được mã hóa và được ký để client không thể sửa hoặc đọc chúng. Nếu bạn muốn tắt mã hóa cho một số cookie mà bạn tạo ra, bạn có thể sử dụng phương thức `encryptCookies` trong file `bootstrap/app.php` của ứng dụng của bạn:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

> [!NOTE]
> Nhìn chung, bạn không nên vô hiệu hóa tính năng mã hóa cookie, vì điều này có thể dẫn đến việc lộ dữ liệu và can thiệp từ phía client.

<a name="redirects"></a>
## Redirect

Redirect response là một instance của class `Illuminate\Http\RedirectResponse` và chứa các header cần thiết để chuyển hướng người dùng đến một URL khác. Có một số cách để tạo ra một instance `RedirectResponse`. Phương pháp đơn giản nhất là sử dụng global helper `redirect`:

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

Thỉnh thoảng bạn có thể muốn chuyển hướng người dùng đến một trang trước đó, chẳng hạn như khi nhập form không hợp lệ. Bạn có thể làm như vậy bằng cách sử dụng hàm global helper `back`. Vì chức năng này sử dụng [session](/docs/{{version}}/session), nên hãy đảm bảo là route được gọi bởi hàm `back` cũng đang dùng group middleware `web`:

```php
Route::post('/user/profile', function () {
    // Validate the request...

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
### Redirecting đến Named Route

Khi bạn gọi helper `redirect` mà không truyền vào tham số, thì một instance của `Illuminate\Routing\Redirector` sẽ được trả về, cho phép bạn gọi bất kỳ phương thức nào trên instance `Redirector`. Ví dụ: để tạo một `RedirectResponse` cho một route mà bạn đã đặt tên, bạn có thể sử dụng phương thức` route`:

```php
return redirect()->route('login');
```

Nếu route đó có yêu cầu truyền tham số, bạn có thể truyền chúng qua tham số thứ 2 của phương thức `route`:

```php
// For a route with the following URI: /profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

<a name="populating-parameters-via-eloquent-models"></a>
#### Nhúng Parameter thông qua Eloquent Model

Nếu bạn đang chuyển hướng đến một route mà có tham số "ID" đang được nhúng trong một model Eloquent, bạn có thể truyền chính model đó vào làm tham số. ID sẽ được trích xuất tự động:

```php
// For a route with the following URI: /profile/{id}

return redirect()->route('profile', [$user]);
```

Nếu bạn muốn tùy biến giá trị được lấy trong tham số route, bạn có thể chỉ định cột trong định nghĩa tham số route (`/profile/{id:slug}`) hoặc bạn có thể ghi đè phương thức `getRouteKey` trên model Eloquent của bạn:

```php
/**
 * Get the value of the model's route key.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
### Redirecting đến Controller Action

Bạn cũng có thể tạo chuyển hướng đến một [controller actions](/docs/{{version}}/controllers). Để làm như vậy, hãy truyền một controller và tên action của nó cho phương thức `action`:

```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

Nếu controller route của bạn yêu cầu tham số, bạn có thể truyền chúng làm tham số thứ hai cho phương thức `action`:

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

<a name="redirecting-external-domains"></a>
### Redirecting đến Domain bên ngoài

Thỉnh thoảng bạn có thể cần chuyển hướng đến một domain ở bên ngoài application của bạn. Bạn có thể làm như vậy bằng cách gọi phương thức `Away`, phương thức này tạo ra một `RedirectResponse` mà không cần bất kỳ URL encoding, validation hoặc verification bổ sung nào:

```php
return redirect()->away('https://www.google.com');
```

<a name="redirecting-with-flashed-session-data"></a>
### Redirecting cùng Flashed Session Data

Chuyển hướng đến một URL mới và [flashing data tới session](/docs/{{version}}/session#flash-data) thường được thực hiện cùng một lúc. Thông thường, điều này được sử dụng sau khi thực hiện thành công một action nào đó và bạn muốn flash một mesage báo thành công đến session. Để thuận tiện, bạn có thể tạo một instance `RedirectResponse` và flash dữ liệu đó vào session với chỉ một chuỗi phương thức đơn giản:

```php
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```

Sau khi người dùng đã được chuyển hướng, bạn có thể hiển thị thông báo được flash từ [session](/docs/{{version}}/session). Ví dụ: sử dụng [Blade syntax](/docs/{{version}}/blade):

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

<a name="redirecting-with-input"></a>
#### Redirecting With Input

Bạn có thể sử dụng phương thức `withInput` do instance `RedirectResponse` cung cấp để flash dữ liệu input của request hiện tại vào session trước khi chuyển hướng người dùng đến một vị trí mới. Điều này thường được thực hiện nếu người dùng gặp phải lỗi xác thực. Sau khi dữ liệu input đã được chuyển sang session, bạn có thể dễ dàng [lấy ra nó](/docs/{{version}}/requests#retrieving-old-input) trong request tiếp theo để điền lại vào form:

```php
return back()->withInput();
```

<a name="other-response-types"></a>
## Các loại Response khác

Helper `response` có thể được sử dụng để tạo các loại instance response khác nhau. Khi helper `response` được gọi mà không có tham số, một implementation của `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts) sẽ được trả về. Contract này sẽ cung cấp một số phương thức hữu ích để tạo response.

<a name="view-responses"></a>
### View Responses

Nếu bạn cần kiểm soát trạng thái và header của response nhưng cũng cần trả về một [view](/docs/{{version}}/views) làm nội dung của response, bạn nên sử dụng phương thức `view`:

```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

Và dĩ nhiên, nếu bạn không cần tuỳ chỉnh HTTP status code hoặc custom header, bạn có thể dùng hàm global helper `view`.

<a name="json-responses"></a>
### JSON Responses

Phương thức `json` sẽ tự động set header `Content-Type` của response thành `application/json`, cũng như chuyển đổi mảng đã cho thành JSON bằng cách sử dụng hàm PHP `json_encode`:

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

Nếu bạn muốn tạo một JSONP response, bạn có thể sử dụng phương thức `json` kết hợp với phương thức `withCallback`:

```php
return response()
    ->json(['name' => 'Abigail', 'state' => 'CA'])
    ->withCallback($request->input('callback'));
```

<a name="file-downloads"></a>
### File Downloads

Phương thức `download` có thể được sử dụng để tạo response buộc trình duyệt của người dùng download file tại một đường dẫn đã cho. Phương thức `download` chấp nhận một tên file làm tham số thứ hai cho phương thức, nó sẽ được hiển thị khi người dùng download file. Cuối cùng, bạn có thể chuyển một mảng các HTTP header làm tham số thứ ba cho phương thức:

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

> [!WARNING]
> Quản lý file download Symfony HttpFoundation yêu cầu file download phải có tên file là ASCII.

<a name="file-responses"></a>
### File Responses

Phương thức `file` có thể được sử dụng để hiển thị một file, chẳng hạn như một hình ảnh hoặc một file PDF trực tiếp trên trình duyệt của người dùng thay vì bắt người dùng tải xuống. Phương thức này sẽ chấp nhận một đường dẫn tuyệt đối đến file làm tham số đầu tiên và một mảng các header làm tham số thứ hai:

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

<a name="streamed-responses"></a>
## Streamed Responses

Bằng cách streaming data đến client ngay khi nó được tạo ra, bạn có thể giảm đáng kể việc sử dụng bộ nhớ và cải thiện hiệu suất, đặc biệt đối với các response rất lớn. Các response được streaming cho phép client bắt đầu xử lý dữ liệu trước khi server hoàn tất việc gửi chúng:

```php
Route::get('/stream', function () {
    return response()->stream(function (): void {
        foreach (['developer', 'admin'] as $string) {
            echo $string;
            ob_flush();
            flush();
            sleep(2); // Simulate delay between chunks...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```

Để thuận tiện, nếu closure mà bạn cung cấp cho phương thức `stream` trả về một [Generator](https://www.php.net/manual/en/language.generators.overview.php), Laravel sẽ tự động trả về dữ liệu trong output buffer giữa các chuỗi mà không cần đợi chạy hết, đồng thời tắt tính năng buffering output của Nginx:

```php
Route::post('/chat', function () {
    return response()->stream(function (): Generator {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

<a name="consuming-streamed-responses"></a>
### Consuming Streamed Responses

Các stream response có thể được sử dụng bằng package npm `stream` của Laravel, cung cấp một API thuận tiện để tương tác với các stream response và event của Laravel. Để bắt đầu, hãy cài đặt package `@laravel/stream-react` hoặc `@laravel/stream-vue`:

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

Sau đó, `useStream` có thể được sử dụng để nhận event stream. Sau khi cung cấp URL stream của bạn, hook sẽ tự động cập nhật biến `data` với các nội dung response được nối lại với nhau khi nội dung được trả về từ ứng dụng Laravel của bạn:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, isFetching, isStreaming, send } = useStream("chat");

    const sendMessage = () => {
        send({
            message: `Current timestamp: ${Date.now()}`,
        });
    };

    return (
        <div>
            <div>{data}</div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
            <button onClick={sendMessage}>Send Message</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data, isFetching, isStreaming, send } = useStream("chat");

const sendMessage = () => {
    send({
        message: `Current timestamp: ${Date.now()}`,
    });
};
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
        <button @click="sendMessage">Send Message</button>
    </div>
</template>
```

Khi dữ liệu được gửi lại trong stream thông qua `send`, kết nối hiện tại tới stream sẽ bị hủy trước khi gửi dữ liệu mới. Tất cả các request đều được gửi dưới dạng request JSON `POST`.

> [!WARNING]
> Vì hook `useStream` tạo một request `POST` tới ứng dụng của bạn, nên nó yêu cầu một CSRF token hợp lệ. Cách dễ nhất để cung cấp CSRF token là [thêm nó thông qua một thẻ meta trong phần head của layout ứng dụng của bạn](/docs/{{version}}/csrf#csrf-x-csrf-token).

Tham số thứ hai được truyền cho `useStream` là một object options nơi mà bạn có thể sử dụng để tùy chỉnh hành vi nhận stream. Các giá trị mặc định cho object này được ghi bên dưới:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        id: undefined,
        initialInput: undefined,
        headers: undefined,
        csrfToken: undefined,
        onResponse: (response: Response) => void,
        onData: (data: string) => void,
        onCancel: () => void,
        onFinish: () => void,
        onError: (error: Error) => void,
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data } = useStream("chat", {
    id: undefined,
    initialInput: undefined,
    headers: undefined,
    csrfToken: undefined,
    onResponse: (response: Response) => void,
    onData: (data: string) => void,
    onCancel: () => void,
    onFinish: () => void,
    onError: (error: Error) => void,
});
</script>

<template>
    <div>{{ data }}</div>
</template>
```

`onResponse` sẽ được kích hoạt sau khi phản hồi ban đầu từ stream thành công và [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) raw sẽ được truyền vào callback. `onData` sẽ được gọi mỗi khi chunk dữ liệu được nhận - chunk hiện tại sẽ được truyền vào callback. `onFinish` sẽ được gọi khi stream kết thúc hoặc khi có lỗi xảy ra trong chu kỳ fetch và read.

Mặc định, request không được tạo tới stream khi khởi tạo. Bạn có thể truyền payload ban đầu cho stream bằng cách sử dụng tùy chọn `initialInput`:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        initialInput: {
            message: "Introduce yourself.",
        },
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data } = useStream("chat", {
    initialInput: {
        message: "Introduce yourself.",
    },
});
</script>

<template>
    <div>{{ data }}</div>
</template>
```

Để có thể hủy một stream, bạn có thể sử dụng phương thức `cancel` được trả về từ hook:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, cancel } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <button onClick={cancel}>Cancel</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data, cancel } = useStream("chat");
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <button @click="cancel">Cancel</button>
    </div>
</template>
```

Mỗi khi hook `useStream` được sử dụng, thì một `id` ngẫu nhiên sẽ được tạo ra để xác định stream. ID này sẽ được gửi lên server trong mỗi request thông qua header `X-STREAM-ID`. Khi sử dụng cùng một stream từ nhiều component khác nhau, bạn có thể đọc và viết vào stream đó bằng cách cung cấp `id` của riêng bạn:

```tsx tab=React
// App.tsx
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, id } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <StreamStatus id={id} />
        </div>
    );
}

// StreamStatus.tsx
import { useStream } from "@laravel/stream-react";

function StreamStatus({ id }) {
    const { isFetching, isStreaming } = useStream("chat", { id });

    return (
        <div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
        </div>
    );
}
```

```vue tab=Vue
<!-- App.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
import StreamStatus from "./StreamStatus.vue";

const { data, id } = useStream("chat");
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <StreamStatus :id="id" />
    </div>
</template>

<!-- StreamStatus.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const props = defineProps<{
    id: string;
}>();

const { isFetching, isStreaming } = useStream("chat", { id: props.id });
</script>

<template>
    <div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
    </div>
</template>
```

<a name="streamed-json-responses"></a>
### Streamed JSON Responses

Nếu bạn cần stream dữ liệu JSON tăng dần, bạn có thể sử dụng phương thức `streamJson`. Phương thức này đặc biệt hữu ích cho các tập dữ liệu lớn mà cần được gửi dần dần đến trình duyệt ở định dạng có thể dễ dàng phân tích cú pháp bằng JavaScript:

```php
use App\Models\User;

Route::get('/users.json', function () {
    return response()->streamJson([
        'users' => User::cursor(),
    ]);
});
```

Hook `useJsonStream` giống hệt với [hook useStream](#consuming-streamed-responses), ngoại trừ việc nó sẽ thử parse dữ liệu dưới dạng JSON sau khi quá trình stream kết thúc:

```tsx tab=React
import { useJsonStream } from "@laravel/stream-react";

type User = {
    id: number;
    name: string;
    email: string;
};

function App() {
    const { data, send } = useJsonStream<{ users: User[] }>("users");

    const loadUsers = () => {
        send({
            query: "taylor",
        });
    };

    return (
        <div>
            <ul>
                {data?.users.map((user) => (
                    <li>
                        {user.id}: {user.name}
                    </li>
                ))}
            </ul>
            <button onClick={loadUsers}>Load Users</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useJsonStream } from "@laravel/stream-vue";

type User = {
    id: number;
    name: string;
    email: string;
};

const { data, send } = useJsonStream<{ users: User[] }>("users");

const loadUsers = () => {
    send({
        query: "taylor",
    });
};
</script>

<template>
    <div>
        <ul>
            <li v-for="user in data?.users" :key="user.id">
                {{ user.id }}: {{ user.name }}
            </li>
        </ul>
        <button @click="loadUsers">Load Users</button>
    </div>
</template>
```

<a name="event-streams"></a>
### Event Streams (SSE)

Phương thức `eventStream` có thể được sử dụng để trả về một stream response được gửi từ server-sent event (SSE) bằng cách sử dụng content type là `text/event-stream`. Phương thức `eventStream` sẽ chấp nhận một closure mà nên [yield](https://www.php.net/manual/en/language.generators.overview.php) các response vào stream khi response trở nên có sẵn:

```php
Route::get('/chat', function () {
    return response()->eventStream(function () {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

Nếu bạn muốn tùy chỉnh tên của event, bạn có thể yield một instance của class `StreamedEvent`:

```php
use Illuminate\Http\StreamedEvent;

yield new StreamedEvent(
    event: 'update',
    data: $response->choices[0],
);
```

<a name="consuming-event-streams"></a>
#### Consuming Event Streams

Các event stream có thể được sử dụng bằng package npm `stream` của Laravel, cung cấp một API thuận tiện để tương tác với các event stream của Laravel. Để bắt đầu, hãy cài đặt package `@laravel/stream-react` hoặc `@laravel/stream-vue`:

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

Sau đó, `useEventStream` có thể được sử dụng để nhận event stream. Sau khi cung cấp URL stream của bạn, hook sẽ tự động cập nhật biến `message` với nội dung response được nối lại với nhau khi chúng được trả về từ ứng dụng Laravel của bạn:

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/chat");

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";

const { message } = useEventStream("/chat");
</script>

<template>
  <div>{{ message }}</div>
</template>
```

Tham số thứ hai được truyền cho `useEventStream` là một object options nơi mà bạn có thể sử dụng để tùy chỉnh hành vi nhận stream. Các giá trị mặc định cho object này được ghi bên dưới:

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/stream", {
    eventName: "update",
    onMessage: (message) => {
      //
    },
    onError: (error) => {
      //
    },
    onComplete: () => {
      //
    },
    endSignal: "</stream>",
    glue: " ",
  });

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";

const { message } = useEventStream("/chat", {
  eventName: "update",
  onMessage: (message) => {
    // ...
  },
  onError: (error) => {
    // ...
  },
  onComplete: () => {
    // ...
  },
  endSignal: "</stream>",
  glue: " ",
});
</script>
```

Các event stream cũng có thể được nhận thông qua một [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource). Phương thức `eventStream` sẽ tự động gửi một update `</stream>` đến event stream khi stream hoàn tất:

```js
const source = new EventSource('/chat');

source.addEventListener('update', (event) => {
    if (event.data === '</stream>') {
        source.close();

        return;
    }

    console.log(event.data);
})
```

Để tùy chỉnh event cuối cùng được gửi đến event stream, bạn có thể cung cấp một instance `StreamedEvent` cho tham số `endStreamWith` của phương thức `eventStream`:

```php
return response()->eventStream(function () {
    // ...
}, endStreamWith: new StreamedEvent(event: 'update', data: '</stream>'));
```

<a name="streamed-downloads"></a>
### Streamed Downloads

Thỉnh thoảng bạn có thể muốn biến chuỗi response của một hoạt động nhất định thành một download response mà không cần phải ghi nội dung của hoạt động đó vào disk. Bạn có thể sử dụng phương thức `streamDownload` trong trường hợp đó. Phương thức này chấp nhận một callback, một tên file và một mảng header tùy chọn làm tham số của nó:

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
        ->contents()
        ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

<a name="response-macros"></a>
## Response Macros

Nếu bạn muốn định nghĩa một response tùy biến mà bạn có thể sử dụng lại trong nhiều route hoặc các controller khác nhau, bạn có thể sử dụng phương thức `macro` trong facade `Response`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của một trong các [service provider](/docs/{{version}}/providers) trong ứng dụng của bạn, chẳng hạn như service provider `App\Providers\AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

Hàm `macro` chấp nhận tên hàm làm tham số đầu tiên và một closure làm tham số thứ hai. closure của macro sẽ được thực thi khi gọi tên của hàm macro từ implementation `ResponseFactory` hoặc helper `response`:

```php
return response()->caps('foo');
```
