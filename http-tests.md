# HTTP Tests

- [Giới thiệu](#introduction)
    - [Tuỳ biến Request Header](#customizing-request-headers)
    - [Cookies](#cookies)
    - [Debugging Responses](#debugging-responses)
- [Session / Authentication](#session-and-authentication)
- [Test JSON API](#testing-json-apis)
- [Test File Upload](#testing-file-uploads)
- [Available Assertions](#available-assertions)
    - [Response Assertions](#response-assertions)
    - [Authentication Assertions](#authentication-assertions)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một API rất dễ hiểu để thực hiện các HTTP request đến application của bạn và kiểm tra kết quả. Ví dụ, hãy xem một bài test chức năng mẫu được định nghĩa ở dưới đây:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

Phương thức `get` tạo một request `GET` vào application, trong khi phương thức `assertStatus` xác nhận rằng response được trả về có phải có mã trạng thái HTTP đã cho hay không. Ngoài assertion đơn giản này, Laravel còn chứa nhiều assertion khác nhau để kiểm tra các header response, nội dung, cấu trúc JSON, vv...

<a name="customizing-request-headers"></a>
### Tuỳ biến Request Header

Bạn có thể sử dụng phương thức `withHeaders` để tùy biến các header của request trước khi nó được gửi đến application. Điều này cho phép bạn thêm bất kỳ header nào bạn muốn vào trong request:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} CSRF middleware sẽ tự động bị vô hiệu hóa khi chạy test.

<a name="cookies"></a>
### Cookies

Bạn có thể sử dụng phương thức `withCookie` hoặc `withCookies` để set giá trị của cookie trước khi tạo ra một request. Phương thức `withCookie` chấp nhận tên của cookie và một giá trị làm tham số thứ hai của nó, trong khi phương thức` withCookies` chấp nhận một mảng các cặp tên và giá trị:

    <?php

    class ExampleTest extends TestCase
    {
        public function testCookies()
        {
            $response = $this->withCookie('color', 'blue')->get('/');

            $response = $this->withCookies([
                'color' => 'blue',
                'name' => 'Taylor',
            ])->get('/');
        }
    }

<a name="debugging-responses"></a>
### Debugging Responses

Sau khi tạo ra một bài test request cho ứng dụng của bạn, các phương thức `dump`, `dumpHeaders`, và `dumpSession` có thể được sử dụng để kiểm tra và debug nội dung response:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->dumpHeaders();

            $response->dumpSession();

            $response->dump();
        }
    }

<a name="session-and-authentication"></a>
## Session / Authentication

Laravel cũng cung cấp một số helper để làm việc với session trong quá trình test HTTP. Đầu tiên, bạn có thể set dữ liệu session thành một mảng nhất định bằng phương thức `withSession`. Điều này hữu ích để load session với dữ liệu đã có trước, trước khi gửi request cho application của bạn:

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Cách dùng chủ yếu của session là để duy trì trạng thái người dùng đã được xác thực. Phương thức helper `actingAs` sẽ cung cấp một cách đơn giản để xác thực một người dùng. Ví dụ: chúng ta có thể sử dụng một [model factory](/docs/{{version}}/database-testing#writing-factories) để tạo và xác thực một người dùng:

    <?php

    use App\User;

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(User::class)->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Bạn cũng có thể khai báo guard nào sẽ được sử dụng để xác thực người dùng bằng cách truyền tên guard làm tham số thứ hai cho phương thức `actingAs`:

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
## Test JSON API

Laravel cũng cung cấp một số helper để kiểm tra API JSON và response của chúng. Ví dụ, các phương thức `json`, `getJson`, `postJson`, `putJson`, `patchJson`, `deleteJson`, và `optionJson` có thể được sử dụng để đưa vào các JSON request với các method HTTP khác nhau. Bạn cũng có thể dễ dàng truyền dữ liệu và các header cho các phương thức này. Để bắt đầu, hãy viết một bài test để thực hiện một request `POST` đến `/user` và xác nhận rằng dữ liệu mà bạn mong muốn sẽ trả về:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->postJson('/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} Phương thức `assertJson` sẽ chuyển response thành một mảng và sử dụng `PHPUnit::assertArraySubset` để kiểm tra mảng đó có tồn tại trong response JSON mà được application trả về hay không. Vì vậy, nếu có các thuộc tính khác trong response JSON, bài test này vẫn sẽ được pass miễn là có đoạn đã cho.

Ngoài ra, dữ liệu JSON response có thể truy cập được dưới dạng các biến của mảng trong response:

    $this->assertTrue($response['created']);

<a name="verifying-exact-match"></a>
### Verifying An Exact JSON Match

Nếu bạn muốn kiểm tra một mảng đã cho là giống **chính xác** với một response JSON mà được application trả về, bạn nên sử dụng phương thức `assertExactJson`:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="verifying-json-paths"></a>
### Verifying JSON Paths

Nếu bạn muốn kiểm tra rằng response JSON phải chứa một số dữ liệu nhất định tại một đường dẫn cụ thể, bạn nên sử dụng phương thức `assertJsonPath`:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJsonPath('team.owner.name', 'foo')
        }
    }

<a name="testing-file-uploads"></a>
## Test File Upload

Class `Illuminate\Http\UploadedFile` cung cấp một phương thức `fake` có thể được sử dụng để tạo ra các file giả hoặc hình ảnh giả để test. Nó kết hợp cùng với phương thức `fake` của facade `Storage` sẽ đơn giản hóa rất nhiều cho việc test các file upload. Ví dụ: bạn có thể kết hợp hai chức năng này để dễ dàng test cho một form upload avatar:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $file = UploadedFile::fake()->image('avatar.jpg');

            $response = $this->json('POST', '/avatar', [
                'avatar' => $file,
            ]);

            // Assert the file was stored...
            Storage::disk('avatars')->assertExists($file->hashName());

            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

#### Fake File Customization

Khi tạo file bằng phương thức `fake`, bạn có thể khai báo width, height, và size của hình ảnh để test tốt hơn cho các validation của bạn:

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

Ngoài việc tạo hình ảnh, bạn có thể tạo ra các file thuộc bất kỳ loại nào khác bằng phương thức `create`:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

Nếu cần, bạn có thể truyền thêm tham số `$mimeType` vào phương thức để khai báo kiểu MIME sẽ được trả về theo file:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes, 'application/pdf');

<a name="available-assertions"></a>
## Available Assertions

<a name="response-assertions"></a>
### Response Assertions

Laravel cung cấp nhiều phương thức assertion để tùy biến cho các bài test chức năng [PHPUnit](https://phpunit.de/) của bạn. Các assertion này có thể được truy cập trên các response được trả về từ các phương thức test `json`, `get`, `post`, `put`, và `delete`:

<style>
    .collection-method-list > p {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieNotExpired](#assert-cookie-not-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertCreated](#assert-created)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertExactJson](#assert-exact-json)
[assertForbidden](#assert-forbidden)
[assertHeader](#assert-header)
[assertHeaderMissing](#assert-header-missing)
[assertJson](#assert-json)
[assertJsonCount](#assert-json-count)
[assertJsonFragment](#assert-json-fragment)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonMissingValidationErrors](#assert-json-missing-validation-errors)
[assertJsonPath](#assert-json-path)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertLocation](#assert-location)
[assertNoContent](#assert-no-content)
[assertNotFound](#assert-not-found)
[assertOk](#assert-ok)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertSee](#assert-see)
[assertSeeInOrder](#assert-see-in-order)
[assertSeeText](#assert-see-text)
[assertSeeTextInOrder](#assert-see-text-in-order)
[assertSessionHas](#assert-session-has)
[assertSessionHasInput](#assert-session-has-input)
[assertSessionHasAll](#assert-session-has-all)
[assertSessionHasErrors](#assert-session-has-errors)
[assertSessionHasErrorsIn](#assert-session-has-errors-in)
[assertSessionHasNoErrors](#assert-session-has-no-errors)
[assertSessionDoesntHaveErrors](#assert-session-doesnt-have-errors)
[assertSessionMissing](#assert-session-missing)
[assertStatus](#assert-status)
[assertSuccessful](#assert-successful)
[assertUnauthorized](#assert-unauthorized)
[assertViewHas](#assert-view-has)
[assertViewHasAll](#assert-view-has-all)
[assertViewIs](#assert-view-is)
[assertViewMissing](#assert-view-missing)

</div>

<a name="assert-cookie"></a>
#### assertCookie

Yêu cầu response phải chứa cookie đã cho:

    $response->assertCookie($cookieName, $value = null);

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

Yêu cầu response phải chứa cookie đã cho và nó đã hết hạn:

    $response->assertCookieExpired($cookieName);

<a name="assert-cookie-not-expired"></a>
#### assertCookieNotExpired

Yêu cầu response phải chứa cookie đã cho và nó chưa hết hạn:

    $response->assertCookieNotExpired($cookieName);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Yêu cầu response không chứa cookie đã cho:

    $response->assertCookieMissing($cookieName);

<a name="assert-created"></a>
#### assertCreated

Yêu cầu response phải có status code là 201:

    $response->assertCreated();

<a name="assert-dont-see"></a>
#### assertDontSee

Yêu cầu chuỗi đã cho không có trong response. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`:

   $response->assertDontSee($value, $escaped = true);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

Yêu cầu chuỗi đã cho không có trong text response. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`:

   $response->assertDontSeeText($value, $escaped = true);

<a name="assert-exact-json"></a>
#### assertExactJson

Yêu cầu response phải chứa kết quả khớp chính xác với dữ liệu JSON đã cho:

    $response->assertExactJson(array $data);

<a name="assert-forbidden"></a>
#### assertForbidden

Yêu cầu response phải chứa một forbidden status code (403):

    $response->assertForbidden();

<a name="assert-header"></a>
#### assertHeader

Yêu cầu header đã cho phải có trong response:

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>
#### assertHeaderMissing

Yêu cầu header đã cho không có trong response:

    $response->assertHeaderMissing($headerName);

<a name="assert-json"></a>
#### assertJson

Yêu cầu response phải chứa dữ liệu JSON đã cho:

    $response->assertJson(array $data, $strict = false);

<a name="assert-json-count"></a>
#### assertJsonCount

Yêu cầu JSON response phải chứa một mảng với số lượng item nhất định trong một key đã cho:

    $response->assertJsonCount($count, $key = null);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

Yêu cầu response phải chứa đoạn JSON đã cho:

    $response->assertJsonFragment(array $data);

<a name="assert-json-missing"></a>
#### assertJsonMissing

Yêu cầu response không chứa đoạn JSON đã cho:

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

Yêu cầu response không chứa chính xác đoạn JSON đã cho:

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

Yêu cầu response không chứa các lỗi JSON validation cho các khóa đã cho:

    $response->assertJsonMissingValidationErrors($keys);

<a name="assert-json-path"></a>
#### assertJsonPath

Yêu cầu response phải chứa một số dữ liệu đã cho tại một đường dẫn cụ thể:

    $response->assertJsonPath($path, array $data, $strict = false);

<a name="assert-json-structure"></a>
#### assertJsonStructure

Yêu cầu response có cấu trúc JSON đã cho:

    $response->assertJsonStructure(array $structure);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

Yêu cầu JSON response trả về lỗi validation:

    $response->assertJsonValidationErrors(array $data);

<a name="assert-location"></a>
#### assertLocation

Yêu cầu response có giá trị URI trong header `Location`:

    $response->assertLocation($uri);

<a name="assert-no-content"></a>
#### assertNoContent

Yêu cầu response có status code đã cho và không có content.

    $response->assertNoContent($status = 204);

<a name="assert-not-found"></a>
#### assertNotFound

Yêu cầu response có một not found status code:

    $response->assertNotFound();

<a name="assert-ok"></a>
#### assertOk

Yêu cầu response có một status code 200:

    $response->assertOk();

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

Yêu cầu response phải chứa một cookie đã cho (không được mã hóa):

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### assertRedirect

Yêu cầu response là một redirect đến một URI đã cho:

    $response->assertRedirect($uri);

<a name="assert-see"></a>
#### assertSee

Yêu cầu chuỗi đã cho có trong response. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`:

    $response->assertSee($value, $escaped = true);

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

Yêu cầu các chuỗi đã cho được chứa trong response theo thứ tự. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`:

    $response->assertSeeInOrder(array $values, $escaped = true);

<a name="assert-see-text"></a>
#### assertSeeText

Yêu cầu chuỗi đã cho có trong text response. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`:

    $response->assertSeeText($value, $escaped = true);

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

Yêu cầu các chuỗi đã cho được chứa theo thứ tự trong response text. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`:

    $response->assertSeeTextInOrder(array $values, $escaped = true);

<a name="assert-session-has"></a>
#### assertSessionHas

Yêu cầu session có chứa một phần dữ liệu đã cho:

    $response->assertSessionHas($key, $value = null);

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

Yêu cầu session có chứa một giá trị trong flashed input array:

    $response->assertSessionHasInput($key, $value = null);

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

Yêu cầu session có chứa một mảng các giá trị nhất định:

    $response->assertSessionHasAll(array $data);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

Yêu cầu session có chứa lỗi của các field `$keys`. Nếu `$keys` là một mảng associative, thì sẽ yêu cầu là session sẽ chứa một message error cụ thể (giá trị) cho mỗi field (khóa):

    $response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

Yêu cầu session có chứa lỗi của các field `$keys`, trong một error bag cụ thể. Nếu `$keys` là một mảng associative, thì sẽ yêu cầu là session sẽ chứa một message error cụ thể (giá trị) cho mỗi field (khóa), trong error bag cụ thể:

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

Yêu cầu session không chứa lỗi:

    $response->assertSessionHasNoErrors();

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

Yêu cầu session không chứa các lỗi cho các khóa đã cho:

    $response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');

<a name="assert-session-missing"></a>
#### assertSessionMissing

Yêu cầu session không chứa key đã cho:

    $response->assertSessionMissing($key);

<a name="assert-status"></a>
#### assertStatus

Yêu cầu response trả về status code đã cho:

    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

Yêu cầu response trả về một status code thành công (>= 200 và < 300):

    $response->assertSuccessful();

<a name="assert-unauthorized"></a>
#### assertUnauthorized

Yêu cầu response trả về một status code lỗi không quyền truy cập (401):

    $response->assertUnauthorized();

<a name="assert-view-has"></a>
#### assertViewHas

Yêu cầu response view có chứa một phần dữ liệu:

    $response->assertViewHas($key, $value = null);

Ngoài ra, view data có thể truy cập được dưới dạng các biến của mảng trong response:

    $this->assertEquals('Taylor', $response['name']);

<a name="assert-view-has-all"></a>
#### assertViewHasAll

Yêu cầu response view có chứa một mảng dữ liệu nhất định:

    $response->assertViewHasAll(array $data);

<a name="assert-view-is"></a>
#### assertViewIs

Yêu cầu view đã cho sẽ được trả về từ một route:

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

Yêu cầu response view thiếu một phần dữ liệu bị ràng buộc:

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>
### Authentication Assertions

Laravel cũng sẽ cung cấp nhiều cách authentication liên quan đến assertion cho các bài test chức năng [PHPUnit](https://phpunit.de/) của bạn:

Method  | Description
------------- | -------------
`$this->assertAuthenticated($guard = null);`  |  Yêu cầu user phải được authentication.
`$this->assertGuest($guard = null);`  |  Yêu cầu user không cần phải được authentication.
`$this->assertAuthenticatedAs($user, $guard = null);`  |  Yêu cầu user đã cho phải được authentication.
`$this->assertCredentials(array $credentials, $guard = null);`  |  Yêu cầu thông tin đã cho là hợp lệ.
`$this->assertInvalidCredentials(array $credentials, $guard = null);`  |  Yêu cầu thông tin đã cho là không hợp lệ.
