# Browser Tests (Laravel Dusk)

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Dùng Browser khác](#using-other-browsers)
- [Bắt đầu](#getting-started)
    - [Tạo Test](#generating-tests)
    - [Chạy Test](#running-tests)
    - [Xử lý file môi trường](#environment-handling)
    - [Tạo Browser](#creating-browsers)
    - [Authentication](#authentication)
    - [Database Migration](#migrations)
- [Tương tác với Element](#interacting-with-elements)
    - [Dusk Selector](#dusk-selectors)
    - [Clicking Link](#clicking-links)
    - [Text, Values, và Attributes](#text-values-and-attributes)
    - [Dùng Forms](#using-forms)
    - [Đính kèm Files](#attaching-files)
    - [Dùng Keyboard](#using-the-keyboard)
    - [Dùng Mouse](#using-the-mouse)
    - [Scoping Selectors](#scoping-selectors)
    - [Chờ Elements](#waiting-for-elements)
    - [Tạo Vue Assertions](#making-vue-assertions)
- [Assertion có sẵn](#available-assertions)
- [Page](#pages)
    - [Tạo Page](#generating-pages)
    - [Cài đặt Page](#configuring-pages)
    - [Điều hướng tới Page](#navigating-to-pages)
    - [Shorthand Selectors](#shorthand-selectors)
    - [Phương thức của Page](#page-methods)
- [Component](#components)
    - [Tạo Component](#generating-components)
    - [Dùng Component](#using-components)
- [Test tích hợp](#continuous-integration)
    - [CircleCI](#running-tests-on-circle-ci)
    - [Codeship](#running-tests-on-codeship)
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)

<a name="introduction"></a>
## Giới thiệu

Laravel Dusk cung cấp một cách kiểm thử API và tự động hóa trình duyệt một cách nhanh chóng và dễ sử dụng. Mặc định, Dusk không yêu cầu bạn phải cài đặt một JDK hoặc Selenium nào trên máy của bạn. Thay vào đó, Dusk sử dụng một cài đặt độc lập [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home). Tuy nhiên, bạn có thể tự do sử dụng bất kỳ driver nào tương thích với Selenium mà bạn muốn.

<a name="installation"></a>
## Cài đặt

Để bắt đầu, bạn cần thêm library `laravel/dusk` cho Composer trong project của bạn:

    composer require --dev laravel/dusk:"^4.0"

Khi Dusk đã được cài đặt xong, bạn cần đăng ký service provider `Laravel\Dusk\DuskServiceProvider`. Thông thường, việc này sẽ được thực hiện thông qua đăng ký tự động service provider của Laravel.

> {note} Nếu bạn đang đăng ký thủ công service provider của Dusk, thì bạn **đừng bao giờ** đăng ký nó trong môi trường production của bạn, vì làm như vậy sẽ có thể dẫn đến bất kỳ người dùng nào cũng có thể được authenticate vào application của bạn.

Sau khi cài đặt package Dusk, hãy chạy lệnh Artisan `dusk:install`:

    php artisan dusk:install

Một thư mục `Browser` sẽ được tạo trong thư mục `tests` và sẽ chứa một bài test mẫu. Tiếp theo, cài đặt biến môi trường `APP_URL` trong file `.env` của bạn. Giá trị này phải giống với giá trị URL mà bạn đang sử dụng để truy cập vào application của bạn trên trình duyệt.

Để chạy các bài test của bạn, hãy sử dụng lệnh Artisan `dusk`. Lệnh `dusk` chấp nhận tất cả các tham số mà lệnh `phpunit` chấp nhận:

    php artisan dusk

<a name="using-other-browsers"></a>
### Dùng Browser khác

Mặc định, Dusk sử dụng Google Chrome và cài đặt [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) để chạy các bài test browser của bạn. Tuy nhiên, bạn có thể khởi động một server Selenium riêng và chạy bài test đó với bất kỳ browser nào mà bạn muốn.

Để bắt đầu, hãy mở file `tests/DuskTestCase.php`, đây là file test case cơ bản của Dusk cho application của bạn. Trong file này, bạn có thể xoá dòng gọi đến phương thức `startChromeDriver`. Điều này sẽ ngăn Dusk tự động khởi động ChromeDriver:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Tiếp theo, bạn cần phải sửa phương thức `driver` để kết nối tới URL và cổng mà bạn chọn. Ngoài ra, bạn cũng có thể sửa "các thông số cho trình duyệt" mà bạn muốn truyền đến WebDriver:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## Bắt đầu

<a name="generating-tests"></a>
### Tạo Test

Để tạo một bài test Dusk, hãy sử dụng lệnh Artisan `dusk:make`. Bài test sẽ được tạo và nằm trong thư mục `tests/Browser`:

    php artisan dusk:make LoginTest

<a name="running-tests"></a>
### Chạy Test

Để chạy test browser của bạn, hãy sử dụng lệnh Artisan `dusk`:

    php artisan dusk

Lệnh `dusk` chấp nhận tất cả các tham số mà PHPUnit test chấp nhận, cho phép bạn chỉ chạy các bài test cho một [group](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group) nhất định, vv...:

    php artisan dusk --group=foo

#### Manually Starting ChromeDriver

Mặc định, Dusk sẽ thử khởi động ChromeDriver. Nếu ChromeDriver không hoạt động trong hệ thống của bạn, bạn phải tự khởi động nó trước khi chạy lệnh `dusk`. Nếu bạn muốn tự khởi động ChromeDriver, bạn nên comment out dòng lệnh sau trong file `tests/DuskTestCase.php` của bạn:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Ngoài ra, nếu bạn khởi động ChromeDriver trên một cổng khác, ví dụ như là 9515, bạn nên sửa phương thức `driver` trong một class đó:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### Xử lý file môi trường

Để bắt buộc Dusk phải sử dụng file môi trường của chính nó khi chạy test, hãy tạo file `.env.dusk.{environment}` trong thư mục root của project của bạn. Ví dụ, nếu bạn chạy lệnh `dusk` từ môi trường `local`, bạn hãy tạo một file `.env.dusk.local`.

Khi chạy test, Dusk sẽ back-up file `.env` gốc của bạn và đổi tên file môi trường của Dusk thành file `.env`. Khi các bài test đã được chạy xong, file `.env` của bạn sẽ được khôi phục.

<a name="creating-browsers"></a>
### Tạo Browser

Để bắt đầu, hãy viết một bài test để kiểm tra xem chúng ta có thể đăng nhập vào ứng dụng của chúng ta hay không. Sau khi đã tạo xong bài test, chúng ta có thể sửa nó để điều hướng đến trang đăng nhập, và sau đó nhập một số thông tin đăng nhập và nhấp vào nút "Đăng nhập". Để tạo một instance browser, hãy gọi phương thức `browse`:

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * A basic browser test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'secret')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

Như bạn có thể thấy trong ví dụ trên, phương thức `browse` chấp nhận một callback. Một instance browser sẽ tự động được truyền vào callback đó và nó là đối tượng chính để tương tác và đưa ra các yêu cầu đối với application của bạn.

> {tip} Bài test này có thể được sử dụng để kiểm tra màn hình đăng nhập được tạo bởi lệnh Artisan `make:auth`.

#### Creating Multiple Browsers

Thỉnh thoảng bạn có thể cần chạy nhiều trình duyệt trong cùng một lúc để thực hiện test. Ví dụ: có thể cần chạy nhiều trình duyệt để kiểm tra một màn hình trò chuyện sử dụng websocket. Để tạo nhiều trình duyệt, hãy khai báo nhiều trình duyệt trong hàm callback được sử dụng cho phương thức `browse`:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

#### Resizing Browser Windows

Bạn có thể sử dụng phương thức `resize` để điều chỉnh kích thước của browser window:

    $browser->resize(1920, 1080);

Phương thức `maximize` có thể được sử dụng để set browser window ở chế độ full screen:

    $browser->maximize();

<a name="authentication"></a>
### Authentication

Thông thường, bạn sẽ cần test các trang mà cần được authentication. Bạn có thể sử dụng phương thức `loginAs` của Dusk để tránh phải tương tác với các màn hình đăng nhập trong mỗi lần test. Phương thức `loginAs` chấp nhận ID người dùng hoặc một instance model user:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });

> {note} Sau khi sử dụng phương thức `loginAs`, session người dùng sẽ được tạo và duy trì cho tất cả các bài test trong file đó.

<a name="migrations"></a>
### Database Migration

Khi bài test của bạn yêu cầu migration, như ví dụ bài test authentication mẫu ở trên, bạn không nên sử dụng trait `RefreshDatabase`. Trait `RefreshDatabase` sẽ tạo ra các database transaction sẽ không áp dụng trên các HTTP request. Thay vào đó, hãy sử dụng trait `DatabaseMigrations`:

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;
    }

<a name="interacting-with-elements"></a>
## Tương tác với Element

<a name="dusk-selectors"></a>
### Dusk Selector

Chọn các CSS selector tốt để tương tác với các element là một trong những phần khó nhất khi viết bài test cho Dusk. Theo thời gian, các thay đổi ở frontend có thể khiến các CSS selector như sau có thể phá vỡ các bài test của bạn:

    // HTML...

    <button>Login</button>

    // Test...

    $browser->click('.login-page .container div > button');

Dusk selector cho phép bạn tập trung vào viết các bài test hiệu quả thay vì phải nhớ các CSS selector. Để định nghĩa một selector, hãy thêm thuộc tính `dusk` vào element HTML của bạn. Sau đó, thêm tiền tố `@` để thao tác với element đó trong bài test cho Dusk của bạn:

    // HTML...

    <button dusk="login-button">Login</button>

    // Test...

    $browser->click('@login-button');

<a name="clicking-links"></a>
### Clicking Link

Để click vào một link, bạn có thể sử dụng phương thức `clickLink` trên instance browser. Phương thức `clickLink` sẽ click vào một link mà có nội dung đã cho:

    $browser->clickLink($linkText);

> {note} Phương thức này tương tác với jQuery. Nếu jQuery không có sẵn trong trang, Dusk sẽ tự động tích hợp nó vào trang để nó có sẵn trong thời gian test.

<a name="text-values-and-attributes"></a>
### Text, Values, và Attributes

#### Retrieving & Setting Values

Dusk cung cấp một số phương thức để tương tác với text được hiển thị, các giá trị và các thuộc tính hiện tại của element ở trên trang. Ví dụ, để lấy một "giá trị" của một element giống với selector đã cho, hãy sử dụng phương thức `value`:

    // Retrieve the value...
    $value = $browser->value('selector');

    // Set the value...
    $browser->value('selector', 'value');

#### Retrieving Text

Phương thức `text` có thể được sử dụng để lấy ra text của một element giống với một selector đã cho:

    $text = $browser->text('selector');

#### Retrieving Attributes

Cuối cùng, phương thức `attribute` cũng có thể được sử dụng để lấy ra một thuộc tính của một element giống với một selector đã cho:

    $attribute = $browser->attribute('selector', 'value');

<a name="using-forms"></a>
### Dùng Forms

#### Typing Values

Dusk cung cấp nhiều phương thức để tương tác với các form và các element input. Trước tiên, hãy xem một ví dụ về cách nhập text vào field input:

    $browser->type('email', 'taylor@laravel.com');

Lưu ý rằng, phương thức này chấp nhận một tham số nếu cần, chúng ta không bắt buộc phải truyền vào một CSS selector cho phương thức `type`. Nếu CSS selector không được cung cấp, Dusk sẽ tìm kiếm field input có thuộc tính `name`. Cuối cùng, Dusk sẽ thử tìm một `textarea` có thuộc tính` name`.

Để nối text vào một field mà không xóa nội dung của nó đi, bạn có thể sử dụng phương thức `append`:

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

Bạn có thể xóa giá trị của một input bằng phương thức `clear`:

    $browser->clear('email');

#### Dropdowns

Để select một giá trị trong một dropdown selection box, bạn có thể sử dụng phương thức `select`. Giống như phương thức `type`, phương thức` select` không yêu cầu một CSS selector đầy đủ. Khi truyền một giá trị cho phương thức `select`, bạn nên truyền giá trị tùy chọn bên dưới thay vì text:

    $browser->select('size', 'Large');

Bạn có thể select một option ngẫu nhiên bằng cách bỏ qua tham số thứ hai:

    $browser->select('size');

#### Checkboxes

Để "check" vào một checkbox, bạn có thể sử dụng phương thức `check`. Giống như nhiều phương thức liên quan đến input khác, bạn không cần phải có CSS selector đầy đủ. Nếu không thể tìm thấy chính xác selector, Dusk sẽ tìm kiếm một checkbox có thuộc tính `name`:

    $browser->check('terms');

    $browser->uncheck('terms');

#### Radio Buttons

Để "chọn" một radio button, bạn có thể sử dụng phương thức `radio`. Giống như nhiều phương thức liên quan đến input khác, bạn không cần phải có CSS selector đầy đủ. Nếu không thể tìm thấy chính xác selector, Dusk sẽ tìm kiếm một radio có thuộc tính `name` và `value`:

    $browser->radio('version', 'php7');

<a name="attaching-files"></a>
### Đính kèm Files

Phương thức `attach` có thể được sử dụng để đính kèm một file vào một element input `file`. Giống như nhiều phương thức liên quan đến input khác, bạn không cần phải có CSS selector đầy đủ. Nếu không thể tìm thấy selector chính xác, Dusk sẽ tìm kiếm một input file mà có thuộc tính `name`:

    $browser->attach('photo', __DIR__.'/photos/me.png');

> {note} Chức năng đính kèm sẽ yêu cầu bạn cài đặt và enable PHP extension `Zip` trong server của bạn.

<a name="using-the-keyboard"></a>
### Dùng Keyboard

Phương thức `keys` cho phép bạn cung cấp các chuỗi input phức tạp hơn cho một element so với phương thức` type` cho phép. Ví dụ: bạn có thể giữ các phím chức năng để nhập các giá trị. Trong ví dụ này, phím `shift` sẽ được giữ trong khi `taylor` sẽ được nhập vào element giống với selector. Sau khi `taylor` đã được gõ xong, `otwell` sẽ được gõ mà không cần ấn giữ bất kỳ phím nào khác:

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

Bạn thậm chí có thể gửi một "hot key" tới CSS selector chính chứa application của bạn:

    $browser->keys('.app', ['{command}', 'j']);

> {tip} Tất cả các modifier key được bao trong các ký tự `{}` đều giống với các hằng số đã được định nghĩa trong class `Facebook\WebDriver\WebDriverKeys`, bạn có thể [tìm thấy nó trên GitHub](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php).

<a name="using-the-mouse"></a>
### Dùng Mouse

#### Clicking On Elements

Phương thức `click` có thể được sử dụng để "click" vào một element giống với một selector đã cho:

    $browser->click('.selector');

#### Mouseover

Phương thức `mouseover` có thể được sử dụng khi bạn cần di chuyển chuột qua một element giống với một selector đã cho:

    $browser->mouseover('.selector');

#### Drag & Drop

Phương thức `drag` có thể được sử dụng để kéo một element giống với một selector đã cho sang element khác:

    $browser->drag('.from-selector', '.to-selector');

Hoặc, bạn có thể kéo một element theo một hướng:

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

<a name="scoping-selectors"></a>
### Scoping Selectors

Đôi khi bạn có thể muốn thực hiện một số thao tác trong một phạm vi selector đã cho. Ví dụ: bạn có thể muốn kiểm tra rằng có một số text chỉ được hiển thị trong một bảng và sau đó click vào một button trong bảng đó. Bạn có thể sử dụng phương thức `with` để thực hiện điều này. Tất cả các hoạt động được thực hiện trong hàm callback được đưa vào trong phương thức `with` và sẽ được thực hiện test trong phạm vi của selector đã chọn:

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

<a name="waiting-for-elements"></a>
### Chờ Elements

Khi test các application sử dụng JavaScript chuyên sâu, thường phải "chờ" các element hoặc dữ liệu được load xong trước khi tiến hành test. Dusk thực hiện điều này một cách rất dễ dàng. Sử dụng nhiều phương thức khác nhau, bạn có thể đợi các element hiển thị hoàn toàn trên trang hoặc thậm chí đợi cho đến khi một biểu thức JavaScript trả về kết quả là `true`.

#### Waiting

Nếu bạn cần pause bài test trong một số mili giây nhất định, hãy sử dụng phương thức `pause`:

    $browser->pause(1000);

#### Waiting For Selectors

Phương thức `waitFor` có thể được sử dụng để tạm dừng việc test cho đến khi element mà khớp với CSS selector được hiển thị trên trang. Mặc định, điều này sẽ tạm dừng bài test trong tối đa năm giây trước khi đưa ra một ngoại lệ. Nếu cần, bạn có thể truyền vào một ngưỡng thời gian chờ để làm tham số thứ hai cho phương thức:

    // Wait a maximum of five seconds for the selector...
    $browser->waitFor('.selector');

    // Wait a maximum of one second for the selector...
    $browser->waitFor('.selector', 1);

Bạn cũng có thể đợi cho đến khi một selector của bạn bị ẩn đi khỏi trang:

    $browser->waitUntilMissing('.selector');

    $browser->waitUntilMissing('.selector', 1);

#### Scoping Selectors When Available

Đôi khi, bạn có thể muốn đợi một selector nhất định và sau đó tương tác với element khớp với selector đó. Ví dụ, bạn có thể đợi cho đến khi một modal window được hiển thị và sau đó nhấn nút "OK" trong modal đó. Phương thức `whenAvailable` có thể được sử dụng cho trường hợp này. Tất cả các hoạt động của element được thực hiện trong callback sẽ nằm trong phạm vi của selector ban đầu:

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

#### Waiting For Text

Phương thức `waitForText` có thể được sử dụng để đợi cho đến khi một text được hiển thị trên trang:

    // Wait a maximum of five seconds for the text...
    $browser->waitForText('Hello World');

    // Wait a maximum of one second for the text...
    $browser->waitForText('Hello World', 1);

#### Waiting For Links

Phương thức `waitForLink` có thể được sử dụng để đợi cho đến khi một link đã cho được hiển thị trên trang:

    // Wait a maximum of five seconds for the link...
    $browser->waitForLink('Create');

    // Wait a maximum of one second for the link...
    $browser->waitForLink('Create', 1);

#### Waiting On The Page Location

Khi thực hiện kiểm tra đường dẫn, chẳng hạn như `$browser->assertPathIs('/home')`, kiểm tra đó có thể thất bại nếu `window.location.pathname` không đồng bộ với nhau. Bạn có thể sử dụng phương thức `waitForLocation` để đợi cho đến khi window.location là một giá trị nhất định:

    $browser->waitForLocation('/secret');

Bạn cũng có thể đợi location của một route đã được đặt tên:

    $browser->waitForRoute($routeName, $parameters);

#### Waiting for Page Reloads

Nếu bạn cần thực hiện các kiểm tra sau khi một trang đã được reload, hãy sử dụng phương thức `waitForReload`:

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');

#### Waiting On JavaScript Expressions

Thỉnh thoảng bạn có thể muốn tạm dừng việc kiểm tra cho đến khi một biểu thức JavaScript trả về giá trị là `true`. Bạn có thể dễ dàng thực hiện điều này bằng cách sử dụng phương thức `waitUntil`. Khi truyền một biểu thức cho phương thức này, bạn không cần thêm từ khóa `return` hoặc dấu chấm phẩy kết thúc:

    // Wait a maximum of five seconds for the expression to be true...
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // Wait a maximum of one second for the expression to be true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

#### Waiting With A Callback

Nhiều phương thức "chờ" trong Dusk được dựa trên phương thức `waitUsing` bên dưới. Bạn có thể sử dụng phương thức này trực tiếp để chờ cho đến khi một callback trả về giá trị `true`. Phương thức `waitUsing` nhận vào số giây chờ tối đa mà bài test có thể được thực hiện và một khoảng thời gian lặp cho Closure và một Closure và một tuỳ chọn thông báo lỗi:

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="making-vue-assertions"></a>
### Tạo Vue Assertions

Dusk thậm chí cho phép bạn thực hiện các kiểm tra về trạng thái dữ liệu của [Vue](https://vuejs.org) component. Ví dụ: hãy giả sử application của bạn chứa một Vue component sau:

    // HTML...

    <profile dusk="profile-component"></profile>

    // Component Definition...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                  name: 'Taylor'
                }
            };
        }
    });

Bạn có thể kiểm tra trạng thái của Vue component như sau:

    /**
     * A basic Vue test example.
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="available-assertions"></a>
## Assertion có sẵn

Dusk cung cấp nhiều yêu cầu kiểm tra mà bạn có thể đưa ra đối với ứng dụng của bạn. Tất cả các yêu cầu kiểm tra có sẵn đều được ghi lại trong danh sách dưới đây:

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
[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertPathBeginsWith](#assert-path-begins-with)
[assertPathIs](#assert-path-is)
[assertPathIsNot](#assert-path-is-not)
[assertRouteIs](#assert-route-is)
[assertQueryStringHas](#assert-query-string-has)
[assertQueryStringMissing](#assert-query-string-missing)
[assertFragmentIs](#assert-fragment-is)
[assertFragmentBeginsWith](#assert-fragment-begins-with)
[assertFragmentIsNot](#assert-fragment-is-not)
[assertHasCookie](#assert-has-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertValue](#assert-value)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertMissing](#assert-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesNotContain](#assert-vue-does-not-contain)
</div>

<a name="assert-title"></a>
#### assertTitle

Yêu cầu title của page phải đúng với text đã cho:

    $browser->assertTitle($title);

<a name="assert-title-contains"></a>
#### assertTitleContains

Yêu cầu title của page phải chứa text đã cho:

    $browser->assertTitleContains($title);

<a name="assert-url-is"></a>
#### assertUrlIs

Yêu cầu URL hiện tại (bỏ phần query string) phải đúng với chuỗi đã cho:

    $browser->assertUrlIs($url);

<a name="assert-path-begins-with"></a>
#### assertPathBeginsWith

Yêu cầu path của URL hiện tại phải bắt đầu từ path đã cho:

    $browser->assertPathBeginsWith($path);

<a name="assert-path-is"></a>
#### assertPathIs

Yêu cầu path hiện tại phải đúng với path đã cho:

    $browser->assertPathIs('/home');

<a name="assert-path-is-not"></a>
#### assertPathIsNot

Yêu cầu path hiện tại không phải là path đã cho:

    $browser->assertPathIsNot('/home');

<a name="assert-route-is"></a>
#### assertRouteIs

Yêu cầu URL hiện tại phải đúng với URL của một route đã cho:

    $browser->assertRouteIs($name, $parameters);

<a name="assert-query-string-has"></a>
#### assertQueryStringHas

Yêu cầu tham số query string phải tồn tại:

    $browser->assertQueryStringHas($name);

Yêu cầu tham số query string phải tồn tại và có giá trị đã cho:

    $browser->assertQueryStringHas($name, $value);

<a name="assert-query-string-missing"></a>
#### assertQueryStringMissing

Yêu cầu tham số query string bị thiếu:

    $browser->assertQueryStringMissing($name);
    
<a name="assert-fragment-is"></a>
#### assertFragmentIs

Yêu cầu fragment hiệnt tại phải đúng với fragment đã cho:

    $browser->assertFragmentIs('anchor');
    
<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

Yêu cầu fragment hiệnt tại phải bắt đầu từ fragment đã cho:

    $browser->assertFragmentBeginsWith('anchor');
    
<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

Yêu cầu fragment hiệnt tại không phải là fragment đã cho:

    $browser->assertFragmentIsNot('anchor');

<a name="assert-has-cookie"></a>
#### assertHasCookie

Yêu cầu cookie đã cho phải tồn tại:

    $browser->assertHasCookie($name);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Yêu cầu cookie đã cho phải không tồn tại:

    $browser->assertCookieMissing($name);

<a name="assert-cookie-value"></a>
#### assertCookieValue

Yêu cầu cookie phải có một giá trị đã cho:

    $browser->assertCookieValue($name, $value);

<a name="assert-plain-cookie-value"></a>
#### assertPlainCookieValue

Yêu cầu một cookie chưa được mã hóa có một giá trị nhất định:

    $browser->assertPlainCookieValue($name, $value);

<a name="assert-see"></a>
#### assertSee

Yêu cầu text đã cho có trong page:

    $browser->assertSee($text);

<a name="assert-dont-see"></a>
#### assertDontSee

Yêu cầu text đã cho không có trong page:

    $browser->assertDontSee($text);

<a name="assert-see-in"></a>
#### assertSeeIn

Yêu cầu text đã cho phải có trong selector:

    $browser->assertSeeIn($selector, $text);

<a name="assert-dont-see-in"></a>
#### assertDontSeeIn

Yêu cầu text đã cho không có trong selector:

    $browser->assertDontSeeIn($selector, $text);

<a name="assert-source-has"></a>
#### assertSourceHas

Yêu cầu source code đã cho có trong page:

    $browser->assertSourceHas($code);

<a name="assert-source-missing"></a>
#### assertSourceMissing

Yêu cầu source code đã cho không có trong page:

    $browser->assertSourceMissing($code);

<a name="assert-see-link"></a>
#### assertSeeLink

Yêu cầu link đã cho có trong page:

    $browser->assertSeeLink($linkText);

<a name="assert-dont-see-link"></a>
#### assertDontSeeLink

Yêu cầu link đã cho không có trong page:

    $browser->assertDontSeeLink($linkText);

<a name="assert-input-value"></a>
#### assertInputValue

Yêu cầu input field phải có giá trị đã cho:

    $browser->assertInputValue($field, $value);

<a name="assert-input-value-is-not"></a>
#### assertInputValueIsNot

Yêu cầu input field phải không có giá trị đã cho:

    $browser->assertInputValueIsNot($field, $value);

<a name="assert-checked"></a>
#### assertChecked

Yêu cầu checkbox phải được chọn:

    $browser->assertChecked($field);

<a name="assert-not-checked"></a>
#### assertNotChecked

Yêu cầu checkbox không được chọn:

    $browser->assertNotChecked($field);

<a name="assert-radio-selected"></a>
#### assertRadioSelected

Yêu cầu radio phải chọn giá trị đã cho:

    $browser->assertRadioSelected($field, $value);

<a name="assert-radio-not-selected"></a>
#### assertRadioNotSelected

Yêu cầu radio không được chọn giá trị đã cho:

    $browser->assertRadioNotSelected($field, $value);

<a name="assert-selected"></a>
#### assertSelected

Yêu cầu dropdown phải chọn giá trị đã cho:

    $browser->assertSelected($field, $value);

<a name="assert-not-selected"></a>
#### assertNotSelected

Yêu cầu dropdown không được chọn giá trị đã cho:

    $browser->assertNotSelected($field, $value);

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions

Yêu cầu một mảng giá trị có thể được chọn:

    $browser->assertSelectHasOptions($field, $values);

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

Yêu cầu một mảng giá trị không thể được chọn:

    $browser->assertSelectMissingOptions($field, $values);

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

Yêu cầu một giá trị có thể được chọn trên một field:

    $browser->assertSelectHasOption($field, $value);

<a name="assert-value"></a>
#### assertValue

Yêu cầu element giống với selector đã cho:

    $browser->assertValue($selector, $value);

<a name="assert-visible"></a>
#### assertVisible

Yêu cầu element giống với selector đã cho là hiển thị:

    $browser->assertVisible($selector);

<a name="assert-present"></a>
#### assertPresent

Yêu cầu element giống với selector đã cho là tồn tại:

    $browser->assertPresent($selector);

<a name="assert-missing"></a>
#### assertMissing

Yêu cầu element giống với selector đã cho là không hiển thị:

    $browser->assertMissing($selector);

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

Yêu cầu một dialog JavaScript cũng với message đã cho đang được hiển thị:

    $browser->assertDialogOpened($message);

<a name="assert-enabled"></a>
#### assertEnabled

Yêu cầu field đã cho đang được enabled:

    $browser->assertEnabled($field);

<a name="assert-disabled"></a>
#### assertDisabled

Yêu cầu field đã cho đang bị disable:

    $browser->assertDisabled($field);

<a name="assert-focused"></a>
#### assertFocused

Yêu cầu field đã cho đang bị focus:

    $browser->assertFocused($field);

<a name="assert-not-focused"></a>
#### assertNotFocused

Yêu cầu field đã cho không bị focus:

    $browser->assertNotFocused($field);

<a name="assert-vue"></a>
#### assertVue

Yêu cầu một thuộc tính dữ liệu của Vue component giống với giá trị đã cho:

    $browser->assertVue($property, $value, $componentSelector = null);

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

Yêu cầu một thuộc tính dữ liệu của Vue component khác với giá trị đã cho:

    $browser->assertVueIsNot($property, $value, $componentSelector = null);

<a name="assert-vue-contains"></a>
#### assertVueContains

Yêu cầu một thuộc tính dữ liệu của Vue component là một mảng và có chứa giá trị đã cho:

    $browser->assertVueContains($property, $value, $componentSelector = null);

<a name="assert-vue-does-not-contain"></a>
#### assertVueDoesNotContain

Yêu cầu một thuộc tính dữ liệu của Vue component là một mảng và không chứa giá trị đã cho:

    $browser->assertVueDoesNotContain($property, $value, $componentSelector = null);

<a name="pages"></a>
## Page

Đôi khi, các bài test yêu cầu một số hành động phức tạp được thực hiện theo một trình tự nhất định. Điều này có thể làm cho bài test của bạn khó đọc hiểu hơn. Page cho phép bạn định nghĩa các hành động có thể được thực hiện trên một trang nhất định bằng một phương thức duy nhất. Page cũng cho phép bạn định nghĩa các short-cut cho các common selector trong application của bạn hoặc trong một trang.

<a name="generating-pages"></a>
### Tạo Page

Để tạo một page object, sử dụng lệnh Artisan `dusk:page`. Tất cả các page object sẽ được lưu trong thư mục `tests/Browser/Pages`:

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### Cài đặt Page

Mặc định, các page có ba phương thức: `url`, `assert`, và `elements`. Bây giờ chúng ta sẽ thảo luận về các phương thức `url` và `assert`. Phương thức `elements` sẽ được [thảo luận chi tiết hơn bên dưới](#shorthand-selectors).

#### The `url` Method

Phương thức `url` sẽ trả về đường dẫn của URL đến một trang. Dusk sẽ sử dụng URL này khi điều hướng đến trang đó trong trình duyệt:

    /**
     * Get the URL for the page.
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

#### The `assert` Method

Phương thức `assert` có thể đưa ra bất kỳ yêu cầu nào cần thiết để kiểm tra là trình duyệt đã thực sự mở trang đó hay chưa. Hoàn thành phương thức này là không cần thiết; tuy nhiên, bạn có thể tự do đưa ra những yêu cầu này nếu muốn. Các yêu cầu này sẽ được chạy tự động khi mở trang đó:

    /**
     * Assert that the browser is on the page.
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### Điều hướng tới Page

Khi một page đã được cấu hình, bạn có thể điều hướng đến nó bằng phương thức `visit`:

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

Thỉnh thoảng bạn có thể đã ở trên một trang và cần "load" lại các selector và phương thức của trang đó vào test hiện tại. Điều này rất phổ biến khi bạn nhấn một nút và được chuyển hướng đến một trang khác mà không điều hướng đến nó. Trong tình huống này, bạn có thể sử dụng phương thức `on` để load trang:

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### Shorthand Selectors

Phương thức `elements` của trang cho phép bạn định nghĩa các shortcut nhanh, dễ nhớ cho bất kỳ CSS selector nào trên trang của bạn. Ví dụ: hãy định nghĩa shortcut cho field input "email" trong trang đăng nhập của application:

    /**
     * Get the element shortcuts for the page.
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

Bây giờ, bạn có thể sử dụng shorthand selector này ở bất cứ nơi nào mà bạn dự kiến sử dụng CSS selector đó:

    $browser->type('@email', 'taylor@laravel.com');

#### Global Shorthand Selectors

Sau khi cài đặt Dusk, một class `Page` sẽ được lưu vào trong thư mục `tests/Browser/Pages` của bạn. Class này chứa một phương thức `siteElements` có thể được sử dụng để định nghĩa các global shorthand selector mà sẽ được khai báo sẵn trên mỗi trang trong application của bạn:

    /**
     * Get the global element shortcuts for the site.
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### Phương thức của Page

Ngoài các phương thức mặc định được định nghĩa trên các trang, bạn có thể định nghĩa thêm các phương thức có thể được sử dụng trong suốt qua trình test của bạn. Ví dụ: hãy giả sử rằng chúng ta đang xây dựng một application quản lý nhạc. Một hành động chung cho một trang của application có thể là tạo một playlist. Thay vì viết lại logic để tạo playlist cho mỗi bài test, bạn có thể định nghĩa phương thức `createPlaylist` trên một class page:

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // Other page methods...

        /**
         * Create a new playlist.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

Khi phương thức đã được định nghĩa xong, bạn có thể sử dụng nó trong bất kỳ bài test nào mà bạn đang sử dụng page đó. Instance browser sẽ được tự động chuyển đến phương thức của page đó:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## Component

Các component cũng tương tự như các đối tượng page của Dusk, nhưng được dành cho các thành phần của UI và các chức năng được sử dụng lại trong toàn bộ application của bạn, như một navigation bar hoặc một notification window. Như vậy, các component không bị ràng buộc với các URL cụ thể.

<a name="generating-components"></a>
### Tạo Component

Để tạo một component, bạn hãy sử dụng lệnh Artisan `dusk:component`. Các component mới sẽ được lưu vào trong thư mục `test/Browser/Components`:

    php artisan dusk:component DatePicker

Như câu lệnh ở trên, một "date picker" có thể là một ví dụ mẫu cho một component tồn tại trong toàn bộ application của bạn và trong nhiều trang khác nhau. Nó sẽ rất cồng kềnh nếu viết các logic của browser để chọn một ngày cho hàng chục bài test có trong test suite của bạn. Thay vào đó, chúng ta có thể định nghĩa một component Dusk để đại diện cho date picker, cho phép chúng ta gói gọn các logic đó vào trong một component:

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * Get the root selector for the component.
         *
         * @return string
         */
        public function selector()
        {
            return '.date-picker';
        }

        /**
         * Assert that the browser page contains the component.
         *
         * @param  Browser  $browser
         * @return void
         */
        public function assert(Browser $browser)
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * Get the element shortcuts for the component.
         *
         * @return array
         */
        public function elements()
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * Select the given date.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  int  $month
         * @param  int  $year
         * @return void
         */
        public function selectDate($browser, $month, $year)
        {
            $browser->click('@date-field')
                    ->within('@month-list', function ($browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function ($browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### Dùng Component

Khi component đã được định nghĩa xong, chúng ta có thể dễ dàng chọn một ngày trong date picker, từ bất kỳ bài test nào. Và, nếu chúng ta cần thay đổi logic chọn ngày, chúng ta chỉ cần cập nhật component:

    <?php

    namespace Tests\Browser;

    use Tests\DuskTestCase;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        /**
         * A basic component test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function ($browser) {
                            $browser->selectDate(1, 2018);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>
## Test tích hợp

<a name="running-tests-on-circle-ci"></a>
### CircleCI

#### CircleCI 1.0

Nếu bạn đang sử dụng CircleCI 1.0 để chạy các bài test, bạn có thể sử dụng file cấu hình này làm file khởi đầu. Giống như TravisCI, chúng ta sẽ sử dụng lệnh `php artisan serve` để khởi chạy web server của PHP:

	dependencies:
	  pre:
	      - curl -L -o google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
	      - sudo dpkg -i google-chrome.deb
	      - sudo sed -i 's|HERE/chrome\"|HERE/chrome\" --disable-setuid-sandbox|g' /opt/google/chrome/google-chrome
	      - rm google-chrome.deb

    test:
        pre:
            - "./vendor/laravel/dusk/bin/chromedriver-linux":
                background: true
            - cp .env.testing .env
            - "php artisan serve":
                background: true

        override:
            - php artisan dusk

 #### CircleCI 2.0

Nếu bạn đang sử dụng CircleCI 2.0 để chạy các bài test, bạn có thể thêm các bước sau vào bản build của bạn:

     version: 2
     jobs:
         build:
             steps:
                - run: sudo apt-get install -y libsqlite3-dev
                - run: cp .env.testing .env
                - run: composer install -n --ignore-platform-reqs
                - run: npm install
                - run: npm run production
                - run: vendor/bin/phpunit

                - run:
                   name: Start Chrome Driver
                   command: ./vendor/laravel/dusk/bin/chromedriver-linux
                   background: true

                - run:
                   name: Run Laravel Server
                   command: php artisan serve
                   background: true

                - run:
                   name: Run Laravel Dusk Tests
                   command: php artisan dusk

<a name="running-tests-on-codeship"></a>
### Codeship

Để chạy các bài test trên [Codeship](https://codeship.com), hãy thêm các lệnh sau vào Codeship project của bạn. Tất nhiên, các lệnh này là các lệnh cơ bản và bạn có thể tự do thêm các lệnh khác khi cần:

    phpenv local 7.1
    cp .env.testing .env
    composer install --no-interaction
    nohup bash -c "./vendor/laravel/dusk/bin/chromedriver-linux 2>&1 &"
    nohup bash -c "php artisan serve 2>&1 &" && sleep 5
    php artisan dusk

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

Để chạy các bài Dusk test trên [Heroku CI](https://www.heroku.com/continuous-integration), hãy thêm gói và tập lệnh sau của Google Chrome vào file Heroku `app.json` của bạn:

    {
      "environments": {
        "test": {
          "buildpacks": [
            { "url": "heroku/php" },
            { "url": "https://github.com/heroku/heroku-buildpack-google-chrome" }
          ],
          "scripts": {
            "test-setup": "cp .env.testing .env",
            "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve > /dev/null 2>&1 &' && php artisan dusk"
          }
        }
      }
    }

<a name="running-tests-on-travis-ci"></a>
### Travis CI

Để chạy các bài Dusk test của bạn trên Travis CI, chúng ta sẽ cần sử dụng môi trường Ubuntu 14.04 (Trusty) "sudo-enabled". Vì Travis CI không phải là một môi trường đồ họa, nên chúng ta sẽ cần thực hiện thêm một số bước để chạy trình duyệt Chrome. Ngoài ra, chúng ta cũng sẽ sử dụng `php artisan serve` để chạy server web tích hợp sẵn của PHP:

    sudo: required
    dist: trusty

    addons:
       chrome: stable

    install:
       - cp .env.testing .env
       - travis_retry composer install --no-interaction --prefer-dist --no-suggest

    before_script:
       - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
       - php artisan serve &

    script:
       - php artisan dusk
