# HTTP Tests

- [Giới thiệu](#introduction)
    - [Tuỳ biến Request Header](#customizing-request-headers)
- [Session / Authentication](#session-and-authentication)
- [Test JSON API](#testing-json-apis)
- [Test File Upload](#testing-file-uploads)
- [Available Assertions](#available-assertions)
    - [Response Assertions](#response-assertions)
    - [Authentication Assertions](#authentication-assertions)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một API rất dễ hiểu để thực hiện các HTTP request đến application của bạn và kiểm tra đầu ra. Ví dụ, hãy xem một bài test được định nghĩa ở dưới đây:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

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

Bạn có thể sử dụng phương thức `withHeaders` để tùy biến các header của request trước khi nó được gửi đến application. Điều này cho phép bạn thêm bất kỳ header tùy biến nào bạn muốn vào trong request:

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
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

<a name="session-and-authentication"></a>
## Session / Authentication

Laravel cũng cung cấp một số helper để làm việc với session trong quá trình test HTTP. Đầu tiên, bạn có thể set dữ liệu session thành một mảng nhất định bằng phương thức `withSession`. Điều này hữu ích để load session với dữ liệu trước khi gửi request cho application của bạn:

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Tất nhiên, cách dùng chủ yếu của session là để duy trì trạng thái cho người dùng đã được xác thực. Phương thức helper `actingAs` cung cấp một cách đơn giản để xác thực một người dùng nhất định. Ví dụ: chúng ta có thể sử dụng một [model factory](/docs/{{version}}/database-testing#writing-factories) để tạo và xác thực một người dùng:

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

Bạn cũng có thể khai báo guard nào sẽ được sử dụng để xác thực người dùng đã cho bằng cách truyền tên guard làm tham số thứ hai cho phương thức `actingAs`:

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
## Test JSON API

Laravel cũng cung cấp một số helper để kiểm tra API JSON và response của chúng. Ví dụ, các phương thức `json`, `get`, `post`, `put`, `patch`, và `delete` có thể được sử dụng để đưa ra các request với các method HTTP khác nhau. Bạn cũng có thể dễ dàng truyền dữ liệu và các header cho các phương thức này. Để bắt đầu, hãy viết một bài test để thực hiện một request `POST` đến `/user` và xác nhận rằng một dữ liệu mà bạn mong đợi sẽ trả về:

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
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} Phương thức `assertJson` sẽ chuyển đổi response thành một mảng và sử dụng `PHPUnit::assertArraySubset` để kiểm tra mảng đã cho có tồn tại trong response JSON được application trả về hay không. Vì vậy, nếu có các thuộc tính khác trong response JSON, bài test này vẫn sẽ được pass miễn là có đoạn đã cho.

<a name="verifying-exact-match"></a>
### Verifying An Exact JSON Match

Nếu bạn muốn kiểm tra mảng đã cho là giống **chính xác** với response JSON được application trả về, bạn nên sử dụng phương thức `assertExactJson`:

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
                ->assertStatus(200)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="testing-file-uploads"></a>
## Test File Upload

Class `Illuminate\Http\UploadedFile` cung cấp một phương thức `fake` có thể được sử dụng để tạo các file giả hoặc hình ảnh giả để test. Điều này, kết hợp với phương thức `fake` của facade `Storage` sẽ đơn giản hóa rất nhiều cho việc test các file upload. Ví dụ: bạn có thể kết hợp hai chức năng này để dễ dàng test cho một form upload avatar:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // Assert the file was stored...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

#### Fake File Customization

Khi tạo file bằng phương thức `fake`, bạn có thể khai báo width, height, và size của hình ảnh để test tốt hơn cho các validation của bạn:

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

Ngoài việc tạo hình ảnh, bạn có thể tạo các file thuộc bất kỳ loại nào khác bằng phương thức `create`:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

<a name="available-assertions"></a>
## Available Assertions

<a name="response-assertions"></a>
### Response Assertions

Laravel cung cấp nhiều phương thức assertion để tùy biến cho các bài test [PHPUnit](https://phpunit.de/) của bạn. Các assertion này có thể được truy cập trên response được trả về từ các phương thức test `json`, `get`, `post`, `put`, và `delete`:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertExactJson](#assert-exact-json)
[assertHeader](#assert-header)
[assertHeaderMissing](#assert-header-missing)
[assertJson](#assert-json)
[assertJsonFragment](#assert-json-fragment)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertSee](#assert-see)
[assertSeeText](#assert-see-text)
[assertSessionHas](#assert-session-has)
[assertSessionHasAll](#assert-session-has-all)
[assertSessionHasErrors](#assert-session-has-errors)
[assertSessionHasErrorsIn](#assert-session-has-errors-in)
[assertSessionMissing](#assert-session-missing)
[assertStatus](#assert-status)
[assertSuccessful](#assert-successful)
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

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Yêu cầu response không chứa cookie đã cho:

    $response->assertCookieMissing($cookieName);

<a name="assert-dont-see"></a>
#### assertDontSee

Yêu cầu chuỗi đã cho không có trong response:

    $response->assertDontSee($value);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

Yêu cầu chuỗi đã cho không có trong text response:

    $response->assertDontSeeText($value);

<a name="assert-exact-json"></a>
#### assertExactJson

Yêu cầu response phải chứa kết quả khớp chính xác với dữ liệu JSON đã cho:

    $response->assertExactJson(array $data);

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

    $response->assertJson(array $data);

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

<a name="assert-json-structure"></a>
#### assertJsonStructure

Yêu cầu response có cấu trúc JSON đã cho:

    $response->assertJsonStructure(array $structure);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

Yêu cầu JSON response trả về lỗi validation cho các key:

    $response->assertJsonValidationErrors($keys);

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

Yêu cầu chuỗi đã cho có trong response:

    $response->assertSee($value);

<a name="assert-see-text"></a>
#### assertSeeText

Yêu cầu chuỗi đã cho có trong text response:

    $response->assertSeeText($value);

<a name="assert-session-has"></a>
#### assertSessionHas

Yêu cầu session có chứa một phần dữ liệu đã cho:

    $response->assertSessionHas($key, $value = null);

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

Yêu cầu session có chứa một mảng các giá trị nhất định:

    $response->assertSessionHasAll(array $data);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

Yêu cầu session có chứa lỗi của các field đã cho:

    $response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

Yêu cầu session có chứa lỗi đã cho:

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

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

Yêu cầu response trả về một status code thành công:

    $response->assertSuccessful();

<a name="assert-view-has"></a>
#### assertViewHas

Yêu cầu response view có chứa một phần dữ liệu:

    $response->assertViewHas($key, $value = null);

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

Laravel cũng cung cấp nhiều cách authentication liên quan đến assertion cho các bài test [PHPUnit](https://phpunit.de/) của bạn:

Method  | Description
------------- | -------------
`$this->assertAuthenticated($guard = null);`  |  Yêu cầu user phải được authentication.
`$this->assertGuest($guard = null);`  |  Yêu cầu user không cần phải được authentication.
`$this->assertAuthenticatedAs($user, $guard = null);`  |  Yêu cầu user đã cho phải được authentication.
`$this->assertCredentials(array $credentials, $guard = null);`  |  Yêu cầu thông tin đã cho là hợp lệ.
`$this->assertInvalidCredentials(array $credentials, $guard = null);`  |  Yêu cầu thông tin đã cho là không hợp lệ.
