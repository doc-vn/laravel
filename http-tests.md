# HTTP Tests

- [Giới thiệu](#introduction)
- [Tạo request](#making-requests)
    - [Tuỳ biến Request Header](#customizing-request-headers)
    - [Cookies](#cookies)
    - [Session / Authentication](#session-and-authentication)
    - [Debugging Responses](#debugging-responses)
    - [Exception Handling](#exception-handling)
- [Testing JSON APIs](#testing-json-apis)
    - [Fluent JSON Testing](#fluent-json-testing)
- [Testing File Uploads](#testing-file-uploads)
- [Testing Views](#testing-views)
    - [Rendering Blade & Components](#rendering-blade-and-components)
- [Available Assertions](#available-assertions)
    - [Response Assertions](#response-assertions)
    - [Authentication Assertions](#authentication-assertions)
    - [Validation Assertions](#validation-assertions)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một API rất dễ hiểu để thực hiện các HTTP request đến application của bạn và kiểm tra response. Ví dụ, hãy xem một bài test chức năng mẫu được định nghĩa ở dưới đây:

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
        public function test_a_basic_request()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

Phương thức `get` tạo một request `GET` vào application, trong khi phương thức `assertStatus` xác nhận rằng response được trả về có phải có mã trạng thái HTTP đã cho hay không. Ngoài assertion đơn giản này, Laravel còn chứa nhiều assertion khác nhau để kiểm tra các header response, nội dung, cấu trúc JSON, vv...

<a name="making-requests"></a>
## Tạo request

Để tạo ra một request cho ứng dụng của bạn, bạn có thể gọi các phương thức `get`, `post`, `put`, `patch` hoặc `delete` trong test của bạn. Các phương thức này không thực sự đưa ra request HTTP "thực" nào cho ứng dụng của bạn. Thay vào đó, toàn bộ network request được mô phỏng nội bộ.

Thay vì trả về một instance `Illuminate\Http\Response`, các phương thức test request này trả về một instance `Illuminate\Testing\TestResponse`, cung cấp [nhiều câu lệnh kiểm tra hữu ích](#available-assertions) cho phép bạn kiểm tra response ứng dụng của bạn:

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
        public function test_a_basic_request()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

Nói chung, mỗi bài test của bạn chỉ nên đưa ra một yêu cầu kiểm tra cho ứng dụng của bạn. Hành vi không mong muốn có thể xảy ra nếu cho nhiều yêu cầu kiểm tra cho một bài test.

> **Note**
> Để thuận tiện, CSRF middleware sẽ tự động bị tắt khi chạy test.

<a name="customizing-request-headers"></a>
### Tuỳ biến Request Header

Bạn có thể sử dụng phương thức `withHeaders` để tùy biến các header của request trước khi nó được gửi đến application. Phương thức này cho phép bạn thêm bất kỳ header nào bạn muốn vào trong request:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function test_interacting_with_headers()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->post('/user', ['name' => 'Sally']);

            $response->assertStatus(201);
        }
    }

<a name="cookies"></a>
### Cookies

Bạn có thể sử dụng phương thức `withCookie` hoặc `withCookies` để set giá trị của cookie trước khi tạo ra một request. Phương thức `withCookie` chấp nhận tên của cookie và một giá trị làm tham số thứ hai của nó, trong khi phương thức` withCookies` chấp nhận một mảng các cặp tên và giá trị:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_interacting_with_cookies()
        {
            $response = $this->withCookie('color', 'blue')->get('/');

            $response = $this->withCookies([
                'color' => 'blue',
                'name' => 'Taylor',
            ])->get('/');
        }
    }

<a name="session-and-authentication"></a>
### Session / Authentication

Laravel cũng cung cấp một số helper để làm việc với session trong quá trình test HTTP. Đầu tiên, bạn có thể set dữ liệu session thành một mảng nhất định bằng phương thức `withSession`. Điều này hữu ích để load session với dữ liệu đã có trước, trước khi gửi request cho application của bạn:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_interacting_with_the_session()
        {
            $response = $this->withSession(['banned' => false])->get('/');
        }
    }

Cách dùng chủ yếu của session là để duy trì trạng thái người dùng đã được xác thực. Phương thức helper `actingAs` sẽ cung cấp một cách đơn giản để xác thực một người dùng. Ví dụ: chúng ta có thể sử dụng một [model factory](/docs/{{version}}/eloquent-factories) để tạo và xác thực một người dùng:

    <?php

    namespace Tests\Feature;

    use App\Models\User;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_an_action_that_requires_authentication()
        {
            $user = User::factory()->create();

            $response = $this->actingAs($user)
                             ->withSession(['banned' => false])
                             ->get('/');
        }
    }

Bạn cũng có thể khai báo guard nào sẽ được sử dụng để xác thực người dùng bằng cách truyền tên guard làm tham số thứ hai cho phương thức `actingAs`. Guard được cung cấp cho phương thức `actingAs` cũng sẽ trở thành guard mặc định trong suốt thời gian test:

    $this->actingAs($user, 'web')

<a name="debugging-responses"></a>
### Debugging Responses

Sau khi tạo ra một bài test request cho ứng dụng của bạn, các phương thức `dump`, `dumpHeaders`, và `dumpSession` có thể được sử dụng để kiểm tra và debug nội dung response:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function test_basic_test()
        {
            $response = $this->get('/');

            $response->dumpHeaders();

            $response->dumpSession();

            $response->dump();
        }
    }

Ngoài ra, bạn có thể sử dụng các phương thức `dd`, `ddHeaders` và `ddSession` để dump ra các thông tin về response và sau đó dừng chạy chương trình:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function test_basic_test()
        {
            $response = $this->get('/');

            $response->ddHeaders();

            $response->ddSession();

            $response->dd();
        }
    }

<a name="exception-handling"></a>
### Exception Handling

Thỉnh thoảng bạn có thể muốn kiểm tra xem ứng dụng của bạn có đang xảy ra một ngoại lệ nào đó hay không. Để đảm bảo ngoại lệ này không bị xử lý bởi exception handler của Laravel, bạn có thể gọi phương thức `withoutExceptionHandling` trước khi thực hiện request của bạn:

    $response = $this->withoutExceptionHandling()->get('/');

Ngoài ra, nếu bạn muốn đảm bảo rằng ứng dụng của bạn không sử dụng các tính năng đã bị loại bỏ bởi ngôn ngữ PHP hoặc các thư viện mà ứng dụng của bạn đang sử dụng, bạn có thể gọi phương thức `withoutDeprecationHandling` trước khi tạo request. Khi việc xử lý các tính bị loại bỏ này bị vô hiệu hóa, các cảnh báo về việc không dùng các tính năng này sẽ bị chuyển đổi thành các ngoại lệ, do đó nó là nguyên nhân khiến cho các test của bạn không thành công:

    $response = $this->withoutDeprecationHandling()->get('/');

<a name="testing-json-apis"></a>
## Test JSON API

Laravel cũng cung cấp một số helper để kiểm tra API JSON và response của chúng. Ví dụ, các phương thức `json`, `getJson`, `postJson`, `putJson`, `patchJson`, `deleteJson`, và `optionJson` có thể được sử dụng để đưa vào các JSON request với các method HTTP khác nhau. Bạn cũng có thể dễ dàng truyền dữ liệu và các header cho các phương thức này. Để bắt đầu, hãy viết một bài test để thực hiện một request `POST` đến `/api/user` và xác nhận rằng dữ liệu JSON mà bạn mong muốn sẽ trả về:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function test_making_an_api_request()
        {
            $response = $this->postJson('/api/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

Ngoài ra, dữ liệu JSON response có thể được truy cập dưới dạng như là một biến trong mảng trên response, giúp bạn thuận tiện kiểm tra các giá trị được trả về trong JSON response:

    $this->assertTrue($response['created']);

> **Note**
> Phương thức `assertJson` sẽ chuyển response thành một mảng và sử dụng `PHPUnit::assertArraySubset` để kiểm tra mảng đó có tồn tại trong response JSON mà được application trả về hay không. Vì vậy, nếu có các thuộc tính khác trong response JSON, bài test này vẫn sẽ được pass miễn là có đoạn đã cho.

<a name="verifying-exact-match"></a>
#### Asserting Exact JSON Matches

Như đã đề cập trước đó, phương thức `assertJson` có thể được sử dụng để kiểm tra một đoạn JSON có tồn tại trong một JSON response hay không. Nếu bạn muốn kiểm tra một mảng đã cho là **giống chính xác** với một response JSON mà được application của bạn trả về, bạn nên sử dụng phương thức `assertExactJson`:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function test_asserting_an_exact_json_match()
        {
            $response = $this->postJson('/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="verifying-json-paths"></a>
#### Asserting On JSON Paths

Nếu bạn muốn kiểm tra rằng response JSON phải chứa một dữ liệu nhất định tại một đường dẫn cụ thể, bạn nên sử dụng phương thức `assertJsonPath`:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function test_asserting_a_json_paths_value()
        {
            $response = $this->postJson('/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJsonPath('team.owner.name', 'Darian');
        }
    }

Phương thức `assertJsonPath` cũng sẽ chấp nhận một closure, có thể được sử dụng để xác định xem bài kiểm tra này có pass hay không:

    $response->assertJsonPath('team.owner.name', fn ($name) => strlen($name) >= 3);

<a name="fluent-json-testing"></a>
### Fluent JSON Testing

Laravel cũng cung cấp một cách hay để kiểm tra dễ dàng các JSON response trong ứng dụng của bạn. Để bắt đầu, hãy truyền một closure cho phương thức `assertJson`. Closure này sẽ được gọi bằng một instance của `Illuminate\Testing\Fluent\AssertableJson`, instance này có thể được sử dụng để đưa ra các yêu cầu đối với JSON được ứng dụng của bạn trả về. Phương thức `where` có thể được sử dụng để đưa ra các yêu cầu đối với một thuộc tính cụ thể trong chuỗi JSON, trong khi phương thức `missing` có thể được sử dụng để yêu cầu một thuộc tính không tồn tại trong JSON:

    use Illuminate\Testing\Fluent\AssertableJson;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function test_fluent_json()
    {
        $response = $this->getJson('/users/1');

        $response
            ->assertJson(fn (AssertableJson $json) =>
                $json->where('id', 1)
                     ->where('name', 'Victoria Faith')
                     ->where('email', fn ($email) => str($email)->is('victoria@gmail.com'))
                     ->whereNot('status', 'pending')
                     ->missing('password')
                     ->etc()
            );
    }

#### Understanding The `etc` Method

Trong ví dụ trên, bạn có thể nhận thấy chúng ta đã gọi phương thức `etc` ở cuối chuỗi yêu cầu. Phương thức này thông báo cho Laravel là có thể có các thuộc tính khác tồn tại trong JSON object. Nếu phương thức `etc` không được sử dụng, quá trình kiểm tra sẽ thất bại nếu có thuộc tính khác mà bạn không đưa yêu cầu tồn tại cho nó.

Mục đích đằng sau của hành động này là để bảo vệ bạn khỏi vô tình làm lộ thông tin nhạy cảm trong JSON response của bạn bằng cách buộc bạn phải đưa ra các yêu cầu rõ ràng cho thuộc tính hoặc cho phép thêm các thuộc tính khác thông qua phương thức `etc`.

Tuy nhiên, bạn nên lưu ý rằng việc không thêm phương thức `etc` vào chuỗi yêu cầu của bạn sẽ không đảm bảo là các thuộc tính bổ sung sẽ không được thêm vào trong mảng lồng nhau trong đối tượng JSON của bạn. Phương thức `etc` chỉ đảm bảo là không có thuộc tính bổ sung nào tồn tại ở cùng cấp độ lồng ở chỗ phương thức `etc` được gọi.

<a name="asserting-json-attribute-presence-and-absence"></a>
#### Asserting Attribute Presence / Absence

Để yêu cầu một thuộc tính tồn tại hoặc không tồn tại, bạn có thể sử dụng phương thức `has` và `missing`:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('data')
             ->missing('message')
    );

Ngoài ra, phương thức `hasAll` và `missingAll` cho phép yêu cầu tồn tại hoặc không tồn tại của nhiều thuộc tính cùng một lúc:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->hasAll(['status', 'data'])
             ->missingAll(['message', 'code'])
    );

Bạn có thể sử dụng phương thức `hasAny` để xác định xem có tồn tại ít nhất một thuộc tính trong danh sách các thuộc tính nhất định hay không:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('status')
             ->hasAny('data', 'message', 'code')
    );

<a name="asserting-against-json-collections"></a>
#### Asserting Against JSON Collections

Thông thường, route của bạn sẽ trả về một JSON response chứa nhiều item, chẳng hạn như nhiều user cùng lúc:

    Route::get('/users', function () {
        return User::all();
    });

Trong những trường hợp như thế này, chúng ta có thể sử dụng phương thức `has` của JSON object để đưa ra các yêu cầu cho các user có trong response. Ví dụ: hãy giả sử rằng JSON response của chúng ta có chứa ba user. Tiếp theo, chúng ta sẽ đưa ra một số yêu cầu nhất định cho user đầu tiên có trong collection bằng phương thức `first`. Phương thức `first` chấp nhận một closure nhận vào một chuỗi JSON có thể yêu cầu khác mà chúng ta có thể sử dụng để đưa ra các yêu cầu cho object đầu tiên có trong collection JSON:

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has(3)
                 ->first(fn ($json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->where('email', fn ($email) => str($email)->is('victoria@gmail.com'))
                         ->missing('password')
                         ->etc()
                 )
        );

<a name="scoping-json-collection-assertions"></a>
#### Scoping JSON Collection Assertions

Đôi khi, các route có trong ứng dụng của bạn sẽ trả về một tập collection JSON được gán cho các khóa:

    Route::get('/users', function () {
        return [
            'meta' => [...],
            'users' => User::all(),
        ];
    })

Khi kiểm tra các route này, bạn có thể sử dụng phương thức `has` để yêu cầu số lượng item có trong collection. Ngoài ra, bạn có thể sử dụng phương thức `has` để xác định phạm vi một chuỗi yêu cầu:

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has('meta')
                 ->has('users', 3)
                 ->has('users.0', fn ($json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->where('email', fn ($email) => str($email)->is('victoria@gmail.com'))
                         ->missing('password')
                         ->etc()
                 )
        );

Tuy nhiên, thay vì thực hiện hai lệnh gọi riêng biệt đến phương thức `has` để yêu cầu cho collection `users`, bạn có thể thực hiện một lệnh gọi duy nhất cung cấp một closure bằng tham số thứ ba. Khi làm như vậy, closure sẽ tự động được gọi và nằm trong phạm vi của mục đầu tiên có trong collection:

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has('meta')
                 ->has('users', 3, fn ($json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->where('email', fn ($email) => str($email)->is('victoria@gmail.com'))
                         ->missing('password')
                         ->etc()
                 )
        );

<a name="asserting-json-types"></a>
#### Asserting JSON Types

Bạn có thể chỉ muốn yêu cầu các thuộc tính có trong JSON response sẽ thuộc vào một loại nhất định. Class `Illuminate\Testing\Fluent\AssertableJson` sẽ cung cấp các phương thức `whereType` và `whereAllType` để thực hiện việc đó:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->whereType('id', 'integer')
             ->whereAllType([
                'users.0.name' => 'string',
                'meta' => 'array'
            ])
    );

Bạn có thể chỉ định nhiều loại bằng cách sử dụng ký tự `|` hoặc truyền một mảng các loại làm tham số thứ hai cho phương thức `whereType`. Yêu cầu sẽ thành công nếu giá trị response là bất kỳ loại nào được liệt kê trong danh sách:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->whereType('name', 'string|null')
             ->whereType('id', ['string', 'integer'])
    );

Phương thức `whereType` và `whereAllType` sẽ nhận dạng các loại sau: `string`, `integer`, `double`, `boolean`, `array` và `null`.

<a name="testing-file-uploads"></a>
## Test File Upload

Class `Illuminate\Http\UploadedFile` cung cấp một phương thức `fake` có thể được sử dụng để tạo ra các file giả hoặc hình ảnh giả để test. Nó kết hợp cùng với phương thức `fake` của facade `Storage`, sẽ đơn giản hóa rất nhiều cho việc test các file upload. Ví dụ: bạn có thể kết hợp hai chức năng này để dễ dàng test cho một form upload avatar:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_avatars_can_be_uploaded()
        {
            Storage::fake('avatars');

            $file = UploadedFile::fake()->image('avatar.jpg');

            $response = $this->post('/avatar', [
                'avatar' => $file,
            ]);

            Storage::disk('avatars')->assertExists($file->hashName());
        }
    }

Nếu bạn muốn yêu cầu một file nhất định sẽ không tồn tại, bạn có thể sử dụng phương thức `assertMissing` được cung cấp bởi facade `Storage`:

    Storage::fake('avatars');

    // ...

    Storage::disk('avatars')->assertMissing('missing.jpg');

<a name="fake-file-customization"></a>
#### Fake File Customization

Khi tạo file bằng phương thức `fake` được cung cấp bởi class `UploadedFile`, bạn có thể khai báo width, height, và size của hình ảnh (theo kilobytes) để test tốt hơn cho các validation trong application của bạn:

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

Ngoài việc tạo hình ảnh, bạn có thể tạo ra các file thuộc bất kỳ loại nào khác bằng phương thức `create`:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

Nếu cần, bạn có thể truyền thêm tham số `$mimeType` vào phương thức để khai báo kiểu MIME sẽ được trả về theo file:

    UploadedFile::fake()->create(
        'document.pdf', $sizeInKilobytes, 'application/pdf'
    );

<a name="testing-views"></a>
## Testing Views

Laravel cũng cho phép bạn render ra một view mà không cần thực hiện một request HTTP cho ứng dụng. Để thực hiện điều này, bạn có thể gọi phương thức `view` trong bài test của bạn. Phương thức `view` sẽ chấp nhận một tên view và một mảng dữ liệu tùy chọn. Phương thức này sẽ trả về một instance của `Illuminate\Testing\TestView`, phương thức này cung cấp một số phương thức để đưa ra các yêu cầu một cách thuận lợi hơn về nội dung của view:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_a_welcome_view_can_be_rendered()
        {
            $view = $this->view('welcome', ['name' => 'Taylor']);

            $view->assertSee('Taylor');
        }
    }

Class `TestView` sẽ cung cấp các phương thức các yêu cầu sau: `assertSee`, `assertSeeInOrder`, `assertSeeText`, `assertSeeTextInOrder`, `assertDontSee` và `assertDontSeeText`.

Nếu cần, bạn có thể lấy ra nội dung raw đã được render bằng cách casting instance `TestView` thành một chuỗi:

    $contents = (string) $this->view('welcome');

<a name="sharing-errors"></a>
#### Sharing Errors

Có một số view có thể phụ thuộc vào các lỗi được chia sẻ trong [global error bag được cung cấp bởi Laravel](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Để tái tạo lại error bag với các error message, bạn có thể sử dụng phương thức `withViewErrors`:

    $view = $this->withViewErrors([
        'name' => ['Please provide a valid name.']
    ])->view('form');

    $view->assertSee('Please provide a valid name.');

<a name="rendering-blade-and-components"></a>
### Rendering Blade & Components

Nếu cần, bạn có thể sử dụng phương thức `blade` để so sánh và hiển thị chuỗi raw [Blade](/docs/{{version}}/blade). Giống như phương thức `view`, phương thức `blade` trả về một instance của `Illuminate\Testing\TestView`:

    $view = $this->blade(
        '<x-component :name="$name" />',
        ['name' => 'Taylor']
    );

    $view->assertSee('Taylor');

Bạn có thể sử dụng phương thức `component` để so sánh và hiển thị một [Blade component](/docs/{{version}}/blade#components). Phương thức `component` sẽ trả về một instance của `Illuminate\Testing\TestComponent`:

    $view = $this->component(Profile::class, ['name' => 'Taylor']);

    $view->assertSee('Taylor');

<a name="available-assertions"></a>
## Available Assertions

<a name="response-assertions"></a>
### Response Assertions

Class `Illuminate\Testing\TestResponse` của Laravel cung cấp nhiều phương thức assertion tùy chỉnh khác nhau mà bạn có thể sử dụng để kiểm tra ứng dụng của bạn. Các assertion này có thể được truy cập trên các response được trả về bởi các phương thức test `json`, `get`, `post`, `put`, và `delete`:

<style>
    .collection-method-list > p {
        columns: 14.4em 2; -moz-columns: 14.4em 2; -webkit-columns: 14.4em 2;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
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
[assertDownload](#assert-download)
[assertExactJson](#assert-exact-json)
[assertForbidden](#assert-forbidden)
[assertHeader](#assert-header)
[assertHeaderMissing](#assert-header-missing)
[assertJson](#assert-json)
[assertJsonCount](#assert-json-count)
[assertJsonFragment](#assert-json-fragment)
[assertJsonIsArray](#assert-json-is-array)
[assertJsonIsObject](#assert-json-is-object)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonMissingValidationErrors](#assert-json-missing-validation-errors)
[assertJsonPath](#assert-json-path)
[assertJsonMissingPath](#assert-json-missing-path)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertJsonValidationErrorFor](#assert-json-validation-error-for)
[assertLocation](#assert-location)
[assertContent](#assert-content)
[assertNoContent](#assert-no-content)
[assertStreamedContent](#assert-streamed-content)
[assertNotFound](#assert-not-found)
[assertOk](#assert-ok)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertRedirectContains](#assert-redirect-contains)
[assertRedirectToRoute](#assert-redirect-to-route)
[assertRedirectToSignedRoute](#assert-redirect-to-signed-route)
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
[assertUnprocessable](#assert-unprocessable)
[assertValid](#assert-valid)
[assertInvalid](#assert-invalid)
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

Yêu cầu response phải có HTTP status code là 201:

    $response->assertCreated();

<a name="assert-dont-see"></a>
#### assertDontSee

Yêu cầu chuỗi đã cho không có trong response được ứng dụng trả về. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`:

   $response->assertDontSee($value, $escaped = true);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

Yêu cầu chuỗi đã cho không có trong text response. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`. Phương thức này sẽ truyền nội dung của response tới hàm PHP `strip_tags` trước khi kiểm tra:

   $response->assertDontSeeText($value, $escaped = true);

<a name="assert-download"></a>
#### assertDownload

Yêu cầu response phải là một "download". Thông thường, điều này có nghĩa là route được gọi sẽ trả về một response có chứa một `Response::download`, `BinaryFileResponse` hoặc `Storage::download`:

    $response->assertDownload();

Nếu muốn, bạn có thể yêu cầu file download sẽ được gán với một tên file nhất định:

    $response->assertDownload('image.jpg');

<a name="assert-exact-json"></a>
#### assertExactJson

Yêu cầu response phải chứa kết quả khớp chính xác với dữ liệu JSON đã cho:

    $response->assertExactJson(array $data);

<a name="assert-forbidden"></a>
#### assertForbidden

Yêu cầu response phải chứa một HTTP status code forbidden (403):

    $response->assertForbidden();

<a name="assert-header"></a>
#### assertHeader

Yêu cầu header và giá trị đã cho phải có trong response:

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>
#### assertHeaderMissing

Yêu cầu header đã cho không có trong response:

    $response->assertHeaderMissing($headerName);

<a name="assert-json"></a>
#### assertJson

Yêu cầu response phải chứa dữ liệu JSON đã cho:

    $response->assertJson(array $data, $strict = false);

Phương thức `assertJson` sẽ chuyển đổi response thành một mảng và sử dụng `PHPUnit::assertArraySubset` để xác minh mảng đã cho có tồn tại trong response JSON được ứng dụng trả về hay không. Vì vậy, nếu có các thuộc tính khác có trong response JSON, thì bài test này sẽ vẫn pass miễn là có phần đã cho.

<a name="assert-json-count"></a>
#### assertJsonCount

Yêu cầu JSON response phải chứa một mảng với số lượng item nhất định trong một key đã cho:

    $response->assertJsonCount($count, $key = null);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

Yêu cầu response phải chứa đoạn JSON data ở bất kỳ nơi nào trong response:

    Route::get('/users', function () {
        return [
            'users' => [
                [
                    'name' => 'Taylor Otwell',
                ],
            ],
        ];
    });

    $response->assertJsonFragment(['name' => 'Taylor Otwell']);

<a name="assert-json-is-array"></a>
#### assertJsonIsArray

Yêu cầu JSON response phải là một mảng:

    $response->assertJsonIsArray();

<a name="assert-json-is-object"></a>
#### assertJsonIsObject

Yêu cầu JSON response phải là một đối tượng:

    $response->assertJsonIsObject();

<a name="assert-json-missing"></a>
#### assertJsonMissing

Yêu cầu response không chứa JSON data đã cho:

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

Yêu cầu response không chứa chính xác JSON data đã cho:

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

Yêu cầu response không chứa các lỗi JSON validation cho các khóa đã cho:

    $response->assertJsonMissingValidationErrors($keys);

> **Note**
> Phương thức [assertValid](#assert-valid) có thể được sử dụng để xác nhận các response được trả về dưới dạng JSON không có lỗi validation **và** không có lỗi nào được load vào bộ nhớ session.

<a name="assert-json-path"></a>
#### assertJsonPath

Yêu cầu response phải chứa một số dữ liệu đã cho tại một đường dẫn cụ thể:

    $response->assertJsonPath($path, $expectedValue);

Ví dụ: nếu JSON response dưới đây được ứng dụng của bạn trả về:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Bạn có thể yêu cầu thuộc tính `name` của đối tượng `user` khớp với một giá trị nhất định như sau:

    $response->assertJsonPath('user.name', 'Steve Schoger');

<a name="assert-json-missing-path"></a>
#### assertJsonMissingPath

Yêu cầu response không chứa đường dẫn đã cho:

    $response->assertJsonMissingPath($path);

Ví dụ: nếu JSON response sau đây được ứng dụng của bạn trả về:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Bạn có thể yêu cầu nó không chứa thuộc tính `email` trong đối tượng `user`:

    $response->assertJsonMissingPath('user.email');

<a name="assert-json-structure"></a>
#### assertJsonStructure

Yêu cầu response có cấu trúc JSON đã cho:

    $response->assertJsonStructure(array $structure);

Ví dụ: nếu JSON response được ứng dụng của bạn trả về chứa dữ liệu sau:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Bạn có thể yêu cầu cấu trúc JSON sẽ phù hợp với mong đợi của bạn như sau:

    $response->assertJsonStructure([
        'user' => [
            'name',
        ]
    ]);

Đôi khi, các JSON response được ứng dụng của bạn trả về có thể chứa các mảng đối tượng:

```json
{
    "user": [
        {
            "name": "Steve Schoger",
            "age": 55,
            "location": "Earth"
        },
        {
            "name": "Mary Schoger",
            "age": 60,
            "location": "Earth"
        }
    ]
}
```

Trong tình huống này, bạn có thể sử dụng ký tự `*` để yêu cầu cấu trúc của tất cả các đối tượng trong mảng:

    $response->assertJsonStructure([
        'user' => [
            '*' => [
                 'name',
                 'age',
                 'location'
            ]
        ]
    ]);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

Yêu cầu JSON response phải trả về lỗi validation cho key đã cho. Nên sử dụng phương thức này khi yêu cầu các response mà trong đó lỗi validation sẽ được trả về dưới dạng cấu trúc JSON thay vì được load vào session:

    $response->assertJsonValidationErrors(array $data, $responseKey = 'errors');

> **Note**
> Phương thức [assertInvalid](#assert-invalid) có thể được sử dụng để yêu cầu một response được trả về dưới dạng JSON có lỗi validation **hoặc** các lỗi đó đã được load vào bộ lưu trữ session.

<a name="assert-json-validation-error-for"></a>
#### assertJsonValidationErrorFor

Yêu cầu response phải chứa bất kỳ lỗi JSON validation nào đối với key đã cho:

    $response->assertJsonValidationErrorFor(string $key, $responseKey = 'errors');

<a name="assert-location"></a>
#### assertLocation

Yêu cầu response có giá trị URI trong header `Location`:

    $response->assertLocation($uri);

<a name="assert-content"></a>
#### assertContent

Yêu cầu response content khớp với một chuỗi đã cho:

    $response->assertContent($value);

<a name="assert-no-content"></a>
#### assertNoContent

Yêu cầu response có HTTP status code đã cho và không có content:

    $response->assertNoContent($status = 204);

<a name="assert-streamed-content"></a>
#### assertStreamedContent

Yêu cầu streamed response content khớp với một chuỗi đã cho:

    $response->assertStreamedContent($value);

<a name="assert-not-found"></a>
#### assertNotFound

Yêu cầu response có một HTTP status code not found (404):

    $response->assertNotFound();

<a name="assert-ok"></a>
#### assertOk

Yêu cầu response có một HTTP status code 200:

    $response->assertOk();

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

Yêu cầu response phải chứa một cookie không được mã hóa đã cho:

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### assertRedirect

Yêu cầu response là một redirect đến một URI đã cho:

    $response->assertRedirect($uri);

<a name="assert-redirect-contains"></a>
#### assertRedirectContains

Yêu cầu response có đang chuyển hướng đế một URI có chứa chuỗi đã cho hay không:

    $response->assertRedirectContains($string);

<a name="assert-redirect-to-route"></a>
#### assertRedirectToRoute

Yêu cầu response là một chuyển hướng đến một [route đã được đặt tên](/docs/{{version}}/routing#named-routes):

    $response->assertRedirectToRoute($name = null, $parameters = []);

<a name="assert-redirect-to-signed-route"></a>
#### assertRedirectToSignedRoute

Yêu cầu response là một chuyển hướng đến một [signed route](/docs/{{version}}/urls#signed-urls):

    $response->assertRedirectToSignedRoute($name = null, $parameters = []);

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

Yêu cầu chuỗi đã cho có trong text response. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`. Phương thức này sẽ truyền nội dung của response tới hàm PHP `strip_tags` trước khi kiểm tra:

    $response->assertSeeText($value, $escaped = true);

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

Yêu cầu các chuỗi đã cho được chứa theo thứ tự trong response text. Yêu cầu này sẽ tự động thoát trừ khi bạn truyền một tham số thứ hai là `false`. Phương thức này sẽ truyền nội dung của response tới hàm PHP `strip_tags` trước khi kiểm tra:

    $response->assertSeeTextInOrder(array $values, $escaped = true);

<a name="assert-session-has"></a>
#### assertSessionHas

Yêu cầu session có chứa một phần dữ liệu đã cho:

    $response->assertSessionHas($key, $value = null);

Nếu cần, một closure có thể được cung cấp làm tham số thứ hai cho phương thức `assertSessionHas`. Yêu cầu sẽ được pass nếu closure trả về `true`:

    $response->assertSessionHas($key, function ($value) {
        return $value->name === 'Taylor Otwell';
    });

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

Yêu cầu session có chứa một giá trị trong [flashed input array](/docs/{{version}}/responses#redirecting-with-flashed-session-data):

    $response->assertSessionHasInput($key, $value = null);

Nếu cần, một closure có thể được cung cấp làm tham số thứ hai cho phương thức `assertSessionHasInput`. Yêu cầu sẽ được pass nếu closure trả về `true`:

    $response->assertSessionHasInput($key, function ($value) {
        return Crypt::decryptString($value) === 'secret';
    });

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

Yêu cầu session contains a given array of key / value pairs:

    $response->assertSessionHasAll(array $data);

Ví dụ: nếu session trong ứng dụng của bạn có chứa khóa `name` và `status`, bạn có thể yêu cầu rằng cả hai phải đều tồn tại và có các giá trị được chỉ định như sau:

    $response->assertSessionHasAll([
        'name' => 'Taylor Otwell',
        'status' => 'active',
    ]);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

Yêu cầu session có chứa lỗi của các field `$keys`. Nếu `$keys` là một mảng associative, thì sẽ yêu cầu là session sẽ chứa một message error cụ thể (giá trị) cho mỗi field (khóa). Phương thức này nên được sử dụng khi kiểm tra các route mà load các error validation vào session thay vì trả về chúng dưới dạng cấu trúc JSON:

    $response->assertSessionHasErrors(
        array $keys, $format = null, $errorBag = 'default'
    );

Ví dụ: để yêu cầu các field `name` và `email` có một thông báo lỗi validation đã được load vào session, bạn có thể gọi phương thức `assertSessionHasErrors` như sau:

    $response->assertSessionHasErrors(['name', 'email']);

Hoặc, bạn có thể yêu cầu một field nhất định có thông báo lỗi validation cụ thể:

    $response->assertSessionHasErrors([
        'name' => 'The given name was invalid.'
    ]);

> **Note**
> Phương thức [assertInvalid](#assert-invalid) tổng quát hơn có thể được sử dụng để yêu cầu một response có lỗi xác thực phải được trả về dưới dạng JSON **hoặc** lỗi đó đã được có trong session storage.

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

Yêu cầu session có chứa lỗi của các field `$keys` trong một [error bag](/docs/{{version}}/validation#named-error-bags) cụ thể. Nếu `$keys` là một mảng associative, thì sẽ yêu cầu là session sẽ chứa một message error cụ thể (giá trị) cho mỗi field (khóa), trong error bag cụ thể:

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

Yêu cầu session không chứa validation lỗi:

    $response->assertSessionHasNoErrors();

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

Yêu cầu session không chứa các validation lỗi cho các khóa đã cho:

    $response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');

> **Note**
> Phương thức [assertValid](#assert-valid) tổng quát hơn có thể được sử dụng để yêu cầu một response không có lỗi xác thực được trả về dưới dạng JSON **hoặc** lỗi đó không có trong session storage.

<a name="assert-session-missing"></a>
#### assertSessionMissing

Yêu cầu session không chứa key đã cho:

    $response->assertSessionMissing($key);

<a name="assert-status"></a>
#### assertStatus

Yêu cầu response trả về HTTP status code đã cho:

    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

Yêu cầu response trả về một HTTP status code thành công (>= 200 và < 300):

    $response->assertSuccessful();

<a name="assert-unauthorized"></a>
#### assertUnauthorized

Yêu cầu response trả về một HTTP status code lỗi không quyền truy cập (401):

    $response->assertUnauthorized();

<a name="assert-unprocessable"></a>
#### assertUnprocessable

Yêu cầu response trả về một HTTP status code không thể xử lý (422):

    $response->assertUnprocessable();

<a name="assert-valid"></a>
#### assertValid

Yêu cầu response không có lỗi validation đối với các khóa đã cho. Phương thức này có thể được sử dụng để yêu cầu các response mà trong đó lỗi validation được trả về dưới dạng cấu trúc JSON hoặc là lỗi validation đã được load vào session:

    // Assert that no validation errors are present...
    $response->assertValid();

    // Assert that the given keys do not have validation errors...
    $response->assertValid(['name', 'email']);

<a name="assert-invalid"></a>
#### assertInvalid

Yêu cầu response có lỗi validation đối với các khóa đã cho. Phương thức này có thể được sử dụng để yêu cầu các response mà trong đó lỗi validation được trả về dưới dạng cấu trúc JSON hoặc là lỗi validation đã được load vào session:

    $response->assertInvalid(['name', 'email']);

Bạn cũng có thể yêu cầu một khóa nhất định có một error message validation cụ thể. Khi làm như vậy, bạn có thể cung cấp toàn bộ message hoặc chỉ một phần nhỏ của message:

    $response->assertInvalid([
        'name' => 'The name field is required.',
        'email' => 'valid email address',
    ]);

<a name="assert-view-has"></a>
#### assertViewHas

Yêu cầu response view có chứa một phần dữ liệu:

    $response->assertViewHas($key, $value = null);

Việc truyền một closure làm tham số thứ hai cho phương thức `assertViewHas` sẽ cho phép bạn kiểm tra và đưa ra các yêu cầu đối với một phần dữ liệu cụ thể của view:

    $response->assertViewHas('user', function (User $user) {
        return $user->name === 'Taylor';
    });

Ngoài ra, view data có thể truy cập được dưới dạng các biến của mảng trong response, cho phép bạn thuận tiện kiểm tra nó:

    $this->assertEquals('Taylor', $response['name']);

<a name="assert-view-has-all"></a>
#### assertViewHasAll

Yêu cầu response view có chứa một mảng dữ liệu nhất định:

    $response->assertViewHasAll(array $data);

Phương thức này có thể được sử dụng để yêu cầu view chỉ chứa các dữ liệu khớp với các khóa đã cho:

    $response->assertViewHasAll([
        'name',
        'email',
    ]);

Hoặc, bạn có thể yêu cầu dữ liệu trong view có tồn tại và có các giá trị cụ thể:

    $response->assertViewHasAll([
        'name' => 'Taylor Otwell',
        'email' => 'taylor@example.com,',
    ]);

<a name="assert-view-is"></a>
#### assertViewIs

Yêu cầu view đã cho sẽ được trả về từ một route:

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

Yêu cầu key data đã cho không có trong view được trả về trong response của ứng dụng:

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>
### Authentication Assertions

Laravel cũng cung cấp nhiều yêu cầu liên quan đến xác thực mà bạn có thể sử dụng trong các bài test tính năng của ứng dụng. Lưu ý rằng các phương thức này được gọi trên chính class test chứ không phải instance `Illuminate\Testing\TestResponse` được trả về bởi các phương thức như `get` và `post`.

<a name="assert-authenticated"></a>
#### assertAuthenticated

Yêu cầu một user đã được xác thực:

    $this->assertAuthenticated($guard = null);

<a name="assert-guest"></a>
#### assertGuest

Yêu cầu một user chưa được xác thực:

    $this->assertGuest($guard = null);

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

Yêu cầu một user cụ thể đã được xác thực:

    $this->assertAuthenticatedAs($user, $guard = null);

<a name="validation-assertions"></a>
## Validation Assertions

Laravel cung cấp hai phương thức yêu cầu chính liên quan đến validation mà bạn có thể sử dụng để đảm bảo dữ liệu trong request của bạn là hợp lệ hoặc không hợp lệ.

<a name="validation-assert-valid"></a>
#### assertValid

Yêu cầu response không có lỗi xác thực đối với các key đã cho. Phương thức này có thể được sử dụng để yêu cầu các response mà trong đó lỗi validation được trả về dưới dạng cấu trúc JSON hoặc là lỗi validation đã được load vào session:

    // Assert that no validation errors are present...
    $response->assertValid();

    // Assert that the given keys do not have validation errors...
    $response->assertValid(['name', 'email']);

<a name="validation-assert-invalid"></a>
#### assertInvalid

Yêu cầu response có lỗi xác thực đối với các key đã cho. Phương thức này có thể được sử dụng để yêu cầu các response mà trong đó lỗi validation được trả về dưới dạng cấu trúc JSON hoặc là lỗi validation đã được load vào session:

    $response->assertInvalid(['name', 'email']);

Bạn cũng có thể yêu cầu một khóa nhất định có một validation error message. Khi làm như vậy, bạn có thể cung cấp toàn bộ message hoặc chỉ một phần nhỏ của message:

    $response->assertInvalid([
        'name' => 'The name field is required.',
        'email' => 'valid email address',
    ]);
