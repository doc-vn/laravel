# Browser Tests (Laravel Dusk)

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Quản lý cài đặt ChromeDriver](#managing-chromedriver-installations)
    - [Dùng Browser khác](#using-other-browsers)
- [Bắt đầu](#getting-started)
    - [Tạo Test](#generating-tests)
    - [Reset lại cơ sở dữ liệu sau mỗi lần test](#resetting-the-database-after-each-test)
    - [Chạy Test](#running-tests)
    - [Xử lý file môi trường](#environment-handling)
- [Browser cơ bản](#browser-basics)
    - [Tạo Browser](#creating-browsers)
    - [Navigation](#navigation)
    - [Resizing Browser Windows](#resizing-browser-windows)
    - [Browser Macros](#browser-macros)
    - [Authentication](#authentication)
    - [Cookies](#cookies)
    - [Chạy JavaScript](#executing-javascript)
    - [Chụp screenshot](#taking-a-screenshot)
    - [Lưu output của console vào disk](#storing-console-output-to-disk)
    - [Lưu source page vào disk](#storing-page-source-to-disk)
- [Tương tác với Element](#interacting-with-elements)
    - [Dusk Selector](#dusk-selectors)
    - [Text, Values, và Attributes](#text-values-and-attributes)
    - [Tương tác với forms](#interacting-with-forms)
    - [Đính kèm Files](#attaching-files)
    - [Ấn Buttons](#pressing-buttons)
    - [Nhấn Links](#clicking-links)
    - [Dùng Keyboard](#using-the-keyboard)
    - [Dùng Mouse](#using-the-mouse)
    - [JavaScript Dialogs](#javascript-dialogs)
    - [Tương tác với inline frames](#interacting-with-iframes)
    - [Scoping Selectors](#scoping-selectors)
    - [Chờ Elements](#waiting-for-elements)
    - [Scrolling một phần tử vào view](#scrolling-an-element-into-view)
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
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)
    - [GitHub Actions](#running-tests-on-github-actions)
    - [Chipper CI](#running-tests-on-chipper-ci)

<a name="introduction"></a>
## Giới thiệu

[Laravel Dusk](https://github.com/laravel/dusk) cung cấp một cách kiểm thử API và tự động hóa trình duyệt một cách nhanh chóng và dễ sử dụng. Mặc định, Dusk không yêu cầu bạn phải cài đặt một JDK hoặc Selenium nào trên máy local của bạn. Thay vào đó, Dusk sử dụng một cài đặt độc lập [ChromeDriver](https://sites.google.com/chromium.org/driver). Tuy nhiên, bạn có thể tự do sử dụng bất kỳ driver nào tương thích với Selenium mà bạn muốn.

<a name="installation"></a>
## Cài đặt

Để bắt đầu, bạn nên cài đặt [Google Chrome](https://www.google.com/chrome) và thêm library Composer `laravel/dusk` vào project của bạn:

```shell
composer require laravel/dusk --dev
```

> [!WARNING]
> Nếu bạn đang đăng ký thủ công service provider của Dusk, thì bạn **đừng bao giờ** đăng ký nó trong môi trường production của bạn, vì làm như vậy sẽ có thể dẫn đến bất kỳ người dùng nào cũng có thể được authenticate vào application của bạn.

Sau khi cài đặt package Dusk, hãy chạy lệnh Artisan `dusk:install`. Lệnh `dusk:install` sẽ tạo ra thư mục `tests/Browser` và một example Dusk test, và cài đặt file binary Chrome Driver cho hệ điều hành của bạn:

```shell
php artisan dusk:install
```

Tiếp theo, cài đặt biến môi trường `APP_URL` trong file `.env` của application của bạn. Giá trị này phải giống với giá trị URL mà bạn đang sử dụng để truy cập vào application của bạn trên trình duyệt.

> [!NOTE]
> Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail) để quản lý môi trường phát triển local của bạn, vui lòng tham khảo thêm tài liệu của Sail về [set cấu hình và chạy Dusk test](/docs/{{version}}/sail#laravel-dusk).

<a name="managing-chromedriver-installations"></a>
### Quản lý cài đặt ChromeDriver

Nếu bạn muốn cài đặt một phiên bản ChromeDriver khác, khác với phiên bản được cài đặt bởi Laravel Dusk thông qua câu lệnh `dusk:install`, bạn có thể sử dụng lệnh `dusk:chrome-driver`:

```shell
# Install the latest version of ChromeDriver for your OS...
php artisan dusk:chrome-driver

# Install a given version of ChromeDriver for your OS...
php artisan dusk:chrome-driver 86

# Install a given version of ChromeDriver for all supported OSs...
php artisan dusk:chrome-driver --all

# Install the version of ChromeDriver that matches the detected version of Chrome / Chromium for your OS...
php artisan dusk:chrome-driver --detect
```

> [!WARNING]
> Dusk sẽ yêu cầu file `chromedriver` của nó phải có quyền chạy. Nếu như bạn đang gặp lỗi khi chạy Dusk, thì bạn nên đảm bảo là file đó đã có quyền chạy bằng lệnh sau: `chmod -R 0755 vendor/laravel/dusk/bin/`.

<a name="using-other-browsers"></a>
### Dùng Browser khác

Mặc định, Dusk sử dụng Google Chrome và cài đặt [ChromeDriver](https://sites.google.com/chromium.org/driver) để chạy các bài test browser của bạn. Tuy nhiên, bạn có thể khởi động một server Selenium riêng và chạy bài test đó với bất kỳ browser nào mà bạn muốn.

Để bắt đầu, hãy mở file `tests/DuskTestCase.php`, đây là file test case cơ bản của Dusk cho application của bạn. Trong file này, bạn có thể xoá dòng gọi đến phương thức `startChromeDriver`. Điều này sẽ ngăn Dusk tự động khởi động ChromeDriver:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     */
    public static function prepare(): void
    {
        // static::startChromeDriver();
    }

Tiếp theo, bạn cần phải sửa phương thức `driver` để kết nối tới URL và cổng mà bạn chọn. Ngoài ra, bạn cũng có thể sửa "các thông số cho trình duyệt" mà bạn muốn truyền đến WebDriver:

    use Facebook\WebDriver\Remote\RemoteWebDriver;

    /**
     * Create the RemoteWebDriver instance.
     */
    protected function driver(): RemoteWebDriver
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

```shell
php artisan dusk:make LoginTest
```

<a name="resetting-the-database-after-each-test"></a>
### Reset lại cơ sở dữ liệu sau mỗi lần test

Hầu hết các bài test mà bạn viết sẽ tương tác với các trang mà truy xuất dữ liệu từ cơ sở dữ liệu trong ứng dụng của bạn; tuy nhiên, các bài test Dusk của bạn không nên sử dụng trait `RefreshDatabase`. Trait `RefreshDatabase` này sẽ tận dụng các transaction cơ sở dữ liệu và không áp dụng hoặc khả dụng trên các request HTTP. Thay vào đó, bạn có hai tùy chọn thay thế là: trait `DatabaseMigrations` và trait `DatabaseTruncation`.

<a name="reset-migrations"></a>
#### Using Database Migrations

Trait `DatabaseMigrations` sẽ chạy migration cơ sở dữ liệu của bạn trước mỗi bài test. Tuy nhiên, việc loại bỏ và tạo lại các bảng cơ sở dữ liệu của bạn cho mỗi bài test thường chậm hơn so với việc truncate các bảng:

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;
    }

> [!WARNING]
> Cơ sở dữ liệu SQLite có thể không được sử dụng khi chạy các bài test Dusk. Vì trình duyệt chạy trong một process riêng của nó và nó sẽ không thể truy cập được vào file cơ sở dữ liệu của các process khác.

<a name="reset-truncation"></a>
#### Using Database Truncation

Trước khi sử dụng trait `DatabaseTruncation`, bạn phải cài đặt package `doctrine/dbal` bằng trình quản lý package Composer:

```shell
composer require --dev doctrine/dbal
```

Trait `DatabaseTruncation` sẽ migrate cơ sở dữ liệu của bạn trong lần kiểm tra đầu tiên để đảm bảo các bảng cơ sở dữ liệu của bạn được tạo đúng cách. Tuy nhiên, trong các thử nghiệm tiếp theo, các bảng của cơ sở dữ liệu sẽ bị truncate - giúp tăng tốc độ khi chạy lại tất cả các migration cơ sở dữ liệu của bạn:

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseTruncation;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseTruncation;
    }

Mặc định, trait này sẽ truncate tất cả các bảng ngoại trừ bảng `migrations`. Nếu bạn muốn tùy chỉnh các bảng cần được truncate, bạn có thể định nghĩa thuộc tính `$tablesToTruncate` trên test class của bạn:

    /**
     * Indicates which tables should be truncated.
     *
     * @var array
     */
    protected $tablesToTruncate = ['users'];

Ngoài ra, bạn có thể định nghĩa thuộc tính `$exceptTables` trên test class của bạn để chỉ định những bảng nào sẽ được bỏ qua việc truncate:

    /**
     * Indicates which tables should be excluded from truncation.
     *
     * @var array
     */
    protected $exceptTables = ['users'];

Để chỉ định kết nối cơ sở dữ liệu nào cần được truncate các bảng của chúng, bạn có thể định nghĩa một thuộc tính `$connectionsToTruncate` trên test class của bạn:

    /**
     * Indicates which connections should have their tables truncated.
     *
     * @var array
     */
    protected $connectionsToTruncate = ['mysql'];

Nếu bạn muốn chạy code trước hoặc sau, khi thực hiện truncation cơ sở dữ liệu, bạn có thể định nghĩa phương thức `beforeTruncatingDatabase` hoặc `afterTruncatingDatabase` trên class test của bạn:

    /**
     * Perform any work that should take place before the database has started truncating.
     */
    protected function beforeTruncatingDatabase(): void
    {
        //
    }

    /**
     * Perform any work that should take place after the database has finished truncating.
     */
    protected function afterTruncatingDatabase(): void
    {
        //
    }

<a name="running-tests"></a>
### Chạy Test

Để chạy test browser của bạn, hãy chạy lệnh Artisan `dusk`:

```shell
php artisan dusk
```

Khi bạn chạy lệnh `dusk`, nếu bạn gặp lỗi ở chỗ cuối cùng, thì bạn có thể tiết kiệm thời gian bằng cách chạy lại chỗ lỗi cuối cùng đó trước bằng lệnh `dusk:fails`:

```shell
php artisan dusk:fails
```

Lệnh `dusk` chấp nhận tất cả các tham số mà PHPUnit test chấp nhận, chẳng hạn như cho phép bạn chỉ chạy các bài test cho một [group](https://docs.phpunit.de/en/10.5/annotations.html#group) nhất định, vv...:

```shell
php artisan dusk --group=foo
```

> [!NOTE]
> Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail) để quản lý môi trường phát triển local của bạn, vui lòng tham khảo thêm tài liệu của Sail về [set cấu hình và chạy Dusk test](/docs/{{version}}/sail#laravel-dusk).

<a name="manually-starting-chromedriver"></a>
#### Manually Starting ChromeDriver

Mặc định, Dusk sẽ thử khởi động ChromeDriver. Nếu ChromeDriver không hoạt động trong hệ thống của bạn, bạn phải tự khởi động nó trước khi chạy lệnh `dusk`. Nếu bạn muốn tự khởi động ChromeDriver, bạn nên comment out dòng lệnh sau trong file `tests/DuskTestCase.php` của bạn:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     */
    public static function prepare(): void
    {
        // static::startChromeDriver();
    }

Ngoài ra, nếu bạn khởi động ChromeDriver trên một cổng khác, ví dụ như là 9515, bạn nên sửa phương thức `driver` trong một class để phản ánh đúng cổng mà bạn mong muốn:

    use Facebook\WebDriver\Remote\RemoteWebDriver;

    /**
     * Create the RemoteWebDriver instance.
     */
    protected function driver(): RemoteWebDriver
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### Xử lý file môi trường

Để bắt buộc Dusk phải sử dụng file môi trường của chính nó khi chạy test, hãy tạo file `.env.dusk.{environment}` trong thư mục root của project của bạn. Ví dụ, nếu bạn chạy lệnh `dusk` từ môi trường `local`, bạn hãy tạo một file `.env.dusk.local`.

Khi chạy test, Dusk sẽ back-up file `.env` gốc của bạn và đổi tên file môi trường của Dusk thành file `.env`. Khi các bài test đã được chạy xong, file `.env` của bạn sẽ được khôi phục.

<a name="browser-basics"></a>
## Browser cơ bản

<a name="creating-browsers"></a>
### Tạo Browser

Để bắt đầu, hãy viết một bài test để kiểm tra xem chúng ta có thể đăng nhập vào ứng dụng của chúng ta hay không. Sau khi đã tạo xong bài test, chúng ta có thể sửa nó để điều hướng đến trang đăng nhập, và sau đó nhập một số thông tin đăng nhập và nhấp vào nút "Đăng nhập". Để tạo một instance browser, bạn có thể gọi phương thức `browse` từ trong bài test Dusk của bạn:

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * A basic browser test example.
         */
        public function test_basic_example(): void
        {
            $user = User::factory()->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function (Browser $browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'password')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

Như bạn có thể thấy trong ví dụ trên, phương thức `browse` chấp nhận một closure. Một instance browser sẽ tự động được truyền vào closure đó và nó là đối tượng chính để tương tác và đưa ra các yêu cầu đối với application của bạn.

<a name="creating-multiple-browsers"></a>
#### Creating Multiple Browsers

Thỉnh thoảng bạn có thể cần chạy nhiều trình duyệt trong cùng một lúc để thực hiện test. Ví dụ: có thể cần chạy nhiều trình duyệt để kiểm tra một màn hình trò chuyện sử dụng websocket. Để tạo nhiều trình duyệt, chỉ cần thêm nhiều tham số trình duyệt vào parameter của closure được cung cấp cho phương thức `browse`:

    $this->browse(function (Browser $first, Browser $second) {
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

<a name="navigation"></a>
### Navigation

Phương thức `visit` có thể được sử dụng để điều hướng đến một URI nhất định trong ứng dụng của bạn:

    $browser->visit('/login');

Bạn có thể sử dụng phương thức `visitRoute` để điều hướng đến [một route đã được đặt tên](/docs/{{version}}/routing#named-routes):

    $browser->visitRoute('login');

Bạn có thể điều hướng "back" và "forward" bằng cách sử dụng các phương thức `back` và `forward`:

    $browser->back();

    $browser->forward();

Bạn có thể sử dụng phương thức `refresh` để refresh trang:

    $browser->refresh();

<a name="resizing-browser-windows"></a>
### Resizing Browser Windows

Bạn có thể sử dụng phương thức `resize` để điều chỉnh kích thước của browser window:

    $browser->resize(1920, 1080);

Phương thức `maximize` có thể được sử dụng để set browser window ở chế độ full screen:

    $browser->maximize();

Phương thức `fitContent` sẽ thay đổi kích thước của browser window để phù hợp với kích thước của chính nội dung đó:

    $browser->fitContent();

Khi kiểm tra không thành công, Dusk sẽ tự động thay đổi kích thước trình duyệt để phù hợp với nội dung trước khi chụp ảnh màn hình. Bạn có thể tắt tính năng này bằng cách gọi phương thức `disableFitOnFailure` trong bài test của bạn:

    $browser->disableFitOnFailure();

Bạn có thể sử dụng phương thức `move` để di chuyển cửa sổ trình duyệt đến một vị trí khác trên màn hình của bạn:

    $browser->move($x = 100, $y = 100);

<a name="browser-macros"></a>
### Browser Macros

Nếu bạn muốn định nghĩa một phương thức trình duyệt tùy biến mà bạn có thể sử dụng lại trong nhiều bài test của bạn, bạn có thể sử dụng phương thức `macro` trên class `Browser`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Laravel\Dusk\Browser;

    class DuskServiceProvider extends ServiceProvider
    {
        /**
         * Register Dusk's browser macros.
         */
        public function boot(): void
        {
            Browser::macro('scrollToElement', function (string $element = null) {
                $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

                return $this;
            });
        }
    }

Phương thức `macro` chấp nhận một tên làm tham số đầu tiên và một closure làm tham số thứ hai của nó. closure của macro sẽ được chạy khi bạn gọi macro dưới dạng một phương thức trên một instance `Browser`:

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/pay')
                ->scrollToElement('#credit-card-details')
                ->assertSee('Enter Credit Card Details');
    });

<a name="authentication"></a>
### Authentication

Thông thường, bạn sẽ cần test các trang mà cần được authentication. Bạn có thể sử dụng phương thức `loginAs` của Dusk để tránh phải tương tác với màn hình login trong ứng dụng của bạn trong mỗi lần kiểm tra. Phương thức `loginAs` chấp nhận khóa chính được liên kết với model xác thực của bạn hoặc một instance model xác thực:

    use App\Models\User;
    use Laravel\Dusk\Browser;

    $this->browse(function (Browser $browser) {
        $browser->loginAs(User::find(1))
              ->visit('/home');
    });

> [!WARNING]
> Sau khi sử dụng phương thức `loginAs`, session người dùng sẽ được tạo và duy trì cho tất cả các bài test trong file đó.

<a name="cookies"></a>
### Cookies

Bạn có thể sử dụng phương thức `cookie` để lấy hoặc set một giá trị cho một cookie đã được mã hóa. Mặc định, tất cả cookie do Laravel tạo đều được mã hóa:

    $browser->cookie('name');

    $browser->cookie('name', 'Taylor');

Bạn có thể sử dụng phương thức `plainCookie` để lấy hoặc set một giá trị cho một cookie không được mã hóa:

    $browser->plainCookie('name');

    $browser->plainCookie('name', 'Taylor');

Bạn có thể sử dụng phương thức `deleteCookie` để xóa một cookie đã cho:

    $browser->deleteCookie('name');

<a name="executing-javascript"></a>
### Chạy JavaScript

Bạn có thể sử dụng phương thức `script` để chạy các câu lệnh JavaScript tùy thích trong trình duyệt:

    $browser->script('document.documentElement.scrollTop = 0');

    $browser->script([
        'document.body.scrollTop = 0',
        'document.documentElement.scrollTop = 0',
    ]);

    $output = $browser->script('return window.location.pathname');

<a name="taking-a-screenshot"></a>
### Chụp screenshot

Bạn có thể sử dụng phương thức `screenshot` để chụp một screenshot và lưu nó với một tên file đã cho. Tất cả ảnh chụp screenshot sẽ được lưu trong thư mục `tests/Browser/screenshots`:

    $browser->screenshot('filename');

Phương thức `ResponseScreenshots` có thể được sử dụng để chụp lại một loạt ảnh chụp màn hình ở nhiều điểm dừng khác nhau:

    $browser->responsiveScreenshots('filename');

<a name="storing-console-output-to-disk"></a>
### Lưu output của console vào disk

Bạn có thể sử dụng phương thức `storeConsoleLog` để ghi output của console của browser hiện tại vào disk với một tên tệp đã cho. Output của console sẽ được lưu trong thư mục `tests/Browser/console`:

    $browser->storeConsoleLog('filename');

<a name="storing-page-source-to-disk"></a>
### Lưu source page vào disk

Bạn có thể sử dụng phương thức `storeSource` để ghi source của page hiện tại vào disk với một tên tệp đã cho. Source của page sẽ được lưu trong thư mục `tests/Browser/source`:

    $browser->storeSource('filename');

<a name="interacting-with-elements"></a>
## Tương tác với Element

<a name="dusk-selectors"></a>
### Dusk Selector

Chọn các CSS selector tốt để tương tác với các element là một trong những phần khó nhất khi viết bài test cho Dusk. Theo thời gian, các thay đổi ở frontend có thể khiến các CSS selector như sau có thể phá vỡ các bài test của bạn:

    // HTML...

    <button>Login</button>

    // Test...

    $browser->click('.login-page .container div > button');

Dusk selector cho phép bạn tập trung vào viết các bài test hiệu quả thay vì phải nhớ các CSS selector. Để định nghĩa một selector, hãy thêm thuộc tính `dusk` vào element HTML của bạn. Sau đó, khi tương tác với trình duyệt Dusk, thêm tiền tố `@` để thao tác với element đó trong bài test của bạn:

    // HTML...

    <button dusk="login-button">Login</button>

    // Test...

    $browser->click('@login-button');

Nếu muốn, bạn có thể tùy chỉnh thuộc tính HTML mà Dusk selector sử dụng thông qua phương thức `selectorHtmlAttribute`. Thông thường, phương thức này phải được gọi từ phương thức `boot` của `AppServiceProvider` của ứng dụng của bạn:

    use Laravel\Dusk\Dusk;

    Dusk::selectorHtmlAttribute('data-dusk');

<a name="text-values-and-attributes"></a>
### Text, Values, và Attributes

<a name="retrieving-setting-values"></a>
#### Retrieving và Setting Values

Dusk cung cấp một số phương thức để tương tác với các giá trị, text hiển thị và các thuộc tính hiện tại của element ở trên trang. Ví dụ, để lấy một "giá trị" của một CSS hoặc một element giống với selector đã cho, hãy sử dụng phương thức `value`:

    // Retrieve the value...
    $value = $browser->value('selector');

    // Set the value...
    $browser->value('selector', 'value');

ạn có thể sử dụng phương thức `inputValue` để lấy ra "giá trị" của phần tử input có một tên đã cho:

    $value = $browser->inputValue('field');

<a name="retrieving-text"></a>
#### Retrieving Text

Phương thức `text` có thể được sử dụng để lấy ra text của một element giống với một selector đã cho:

    $text = $browser->text('selector');

<a name="retrieving-attributes"></a>
#### Retrieving Attributes

Cuối cùng, phương thức `attribute` cũng có thể được sử dụng để lấy ra một giá trị của một thuộc tính của một element giống với một selector đã cho:

    $attribute = $browser->attribute('selector', 'value');

<a name="interacting-with-forms"></a>
### Tương tác với forms

<a name="typing-values"></a>
#### Typing Values

Dusk cung cấp nhiều phương thức để tương tác với các form và các element input. Trước tiên, hãy xem một ví dụ về cách nhập text vào field input:

    $browser->type('email', 'taylor@laravel.com');

Lưu ý rằng, phương thức này chấp nhận một tham số nếu cần, chúng ta không bắt buộc phải truyền vào một CSS selector cho phương thức `type`. Nếu CSS selector không được cung cấp, Dusk sẽ tìm kiếm field `input` hoặc field `textarea` có thuộc tính `name`.

Để nối text vào một field mà không xóa nội dung của nó đi, bạn có thể sử dụng phương thức `append`:

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

Bạn có thể xóa giá trị của một input bằng phương thức `clear`:

    $browser->clear('email');

Bạn có thể hướng dẫn Dusk nhập văn bản chậm bằng phương thức `typeSlowly`. Mặc định, Dusk sẽ tạm dừng trong 100 mili giây giữa các lần nhập. Để tùy chỉnh lượng thời gian giữa các lần nhập, bạn có thể truyền số mili giây mà bạn muốn làm tham số thứ ba cho phương thức:

    $browser->typeSlowly('mobile', '+1 (202) 555-5555');

    $browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);

Bạn có thể sử dụng phương thức `appendSlowly` để nối văn bản một cách từ từ:

    $browser->type('tags', 'foo')
            ->appendSlowly('tags', ', bar, baz');

<a name="dropdowns"></a>
#### Dropdowns

Để select một giá trị có sẵn trong một element `select`, bạn có thể sử dụng phương thức `select`. Giống như phương thức `type`, phương thức` select` không yêu cầu một CSS selector đầy đủ. Khi truyền một giá trị cho phương thức `select`, bạn nên truyền giá trị tùy chọn bên dưới thay vì text:

    $browser->select('size', 'Large');

Bạn có thể select một option ngẫu nhiên bằng cách bỏ qua tham số thứ hai:

    $browser->select('size');

Bằng cách cung cấp một mảng làm tham số thứ hai cho phương thức `select`, bạn có thể hướng dẫn phương thức chọn nhiều tùy chọn:

    $browser->select('categories', ['Art', 'Music']);

<a name="checkboxes"></a>
#### Checkboxes

Để "check" vào một input checkbox, bạn có thể sử dụng phương thức `check`. Giống như nhiều phương thức liên quan đến input khác, bạn không cần phải có CSS selector đầy đủ. Nếu không thể tìm thấy CSS selector, Dusk sẽ tìm kiếm một checkbox có thuộc tính `name`:

    $browser->check('terms');

Phương thức `uncheck` có thể được sử dụng để "bỏ chọn" một input checkbox:

    $browser->uncheck('terms');

<a name="radio-buttons"></a>
#### Radio Buttons

Để "chọn" một input `radio`, bạn có thể sử dụng phương thức `radio`. Giống như nhiều phương thức liên quan đến input khác, bạn không cần phải có CSS selector đầy đủ. Nếu không thể tìm thấy chính xác selector, Dusk sẽ tìm kiếm một input `radio` có thuộc tính `name` và `value`:

    $browser->radio('size', 'large');

<a name="attaching-files"></a>
### Đính kèm Files

Phương thức `attach` có thể được sử dụng để đính kèm một file vào một element input `file`. Giống như nhiều phương thức liên quan đến input khác, bạn không cần phải có CSS selector đầy đủ. Nếu không thể tìm thấy CSS selector, Dusk sẽ tìm kiếm một input `file` mà có thuộc tính `name`:

    $browser->attach('photo', __DIR__.'/photos/mountains.png');

> [!WARNING]
> Chức năng đính kèm sẽ yêu cầu bạn cài đặt và enable PHP extension `Zip` trong server của bạn.

<a name="pressing-buttons"></a>
### Ấn Buttons

Phương thức `press` có thể được sử dụng để nhấp vào một nút trên trang. Tham số được cung cấp cho phương thức `press` có thể là text hiển thị của nút hoặc selector CSS hoặc Dusk:

    $browser->press('Login');

Khi gửi form, nhiều ứng dụng sẽ disable nút gửi của form sau khi nhấn và sau đó enable lại nút khi yêu cầu HTTP của form gửi hoàn tất. Để nhấn một nút và đợi nút được bật lại, bạn có thể sử dụng phương thức `pressAndWaitFor`:

    // Press the button and wait a maximum of 5 seconds for it to be enabled...
    $browser->pressAndWaitFor('Save');

    // Press the button and wait a maximum of 1 second for it to be enabled...
    $browser->pressAndWaitFor('Save', 1);

<a name="clicking-links"></a>
### Nhấn Links

Để nhấp vào một link, bạn có thể sử dụng phương thức `clickLink` trên instance trình duyệt. Phương thức `clickLink` sẽ nhấp vào một link có text hiển thị đã cho:

    $browser->clickLink($linkText);

Bạn có thể sử dụng phương thức `seeLink` để xác định xem một link có text hiển thị đã cho có hiển thị trên trang hay không:

    if ($browser->seeLink($linkText)) {
        // ...
    }

> [!WARNING]
> Các phương thức này tương tác với jQuery. Nếu jQuery không có sẵn trên trang của bạn, Dusk sẽ tự động đưa nó vào trang để nó có sẵn trong thời gian chạy test.

<a name="using-the-keyboard"></a>
### Dùng Keyboard

Phương thức `keys` cho phép bạn cung cấp các chuỗi input phức tạp hơn cho một element so với phương thức` type` cho phép. Ví dụ: bạn có thể hướng dẫn Dusk để giữ các phím chức năng khi nhập các giá trị. Trong ví dụ này, phím `shift` sẽ được giữ trong khi `taylor` sẽ được nhập vào element giống với selector. Sau khi `taylor` đã được gõ xong, `swift` sẽ được gõ mà không cần ấn giữ bất kỳ phím nào khác:

    $browser->keys('selector', ['{shift}', 'taylor'], 'swift');

Một trường hợp sử dụng có giá trị khác của phương thức `keys` là gửi tổ hợp "phím tắt" tới selector CSS cho ứng dụng của bạn:

    $browser->keys('.app', ['{command}', 'j']);

> [!NOTE]
> Tất cả các modifier key chẳng hạn như `{command}` đã được chứa trong các ký tự `{}` đều giống với các hằng số đã được định nghĩa trong class `Facebook\WebDriver\WebDriverKeys`, bạn có thể [tìm thấy nó trên GitHub](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php).

<a name="fluent-keyboard-interactions"></a>
#### Fluent Keyboard Interactions

Dusk cũng cung cấp phương thức `withKeyboard`, cho phép bạn thực hiện dễ dàng với các tương tác bàn phím phức tạp thông qua class `Laravel\Dusk\Keyboard`. Class `Keyboard` cung cấp các phương thức `press`, `release`, `type` và `pause`:

    use Laravel\Dusk\Keyboard;

    $browser->withKeyboard(function (Keyboard $keyboard) {
        $keyboard->press('c')
            ->pause(1000)
            ->release('c')
            ->type(['c', 'e', 'o']);
    });

<a name="keyboard-macros"></a>
#### Keyboard Macros

Nếu bạn muốn định nghĩa các tương tác bàn phím để bạn có thể dễ dàng sử dụng lại trong toàn bộ test case của bạn, bạn có thể sử dụng phương thức `macro` do class `Keyboard` cung cấp. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Facebook\WebDriver\WebDriverKeys;
    use Illuminate\Support\ServiceProvider;
    use Laravel\Dusk\Keyboard;
    use Laravel\Dusk\OperatingSystem;

    class DuskServiceProvider extends ServiceProvider
    {
        /**
         * Register Dusk's browser macros.
         */
        public function boot(): void
        {
            Keyboard::macro('copy', function (string $element = null) {
                $this->type([
                    OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'c',
                ]);

                return $this;
            });

            Keyboard::macro('paste', function (string $element = null) {
                $this->type([
                    OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'v',
                ]);

                return $this;
            });
        }
    }

Phương thức `macro` chấp nhận tên của macro làm tham số đầu tiên và một closure làm tham số thứ hai. Closure của macro sẽ được chạy khi tên macro đó được gọi từ instanse `Keyboard`:

    $browser->click('@textarea')
        ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->copy())
        ->click('@another-textarea')
        ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->paste());

<a name="using-the-mouse"></a>
### Dùng Mouse

<a name="clicking-on-elements"></a>
#### Clicking On Elements

Phương thức `click` có thể được sử dụng để nhấp vào một element giống với một CSS hoặc một Dusk selector đã cho:

    $browser->click('.selector');

Phương thức `clickAtXPath` có thể được sử dụng để nhấp vào một phần tử khớp với biểu thức XPath đã cho:

    $browser->clickAtXPath('//div[@class = "selector"]');

Phương thức `clickAtPoint` có thể được sử dụng để nhấp vào phần tử đầu tiên tại một tọa độ nhất định của trình duyệt:

    $browser->clickAtPoint(0, 0);

Phương thức `doubleClick` có thể được sử dụng để mô phỏng hành động nhấp đúp của chuột:

    $browser->doubleClick();

    $browser->doubleClick('.selector');

Phương thức `rightClick` có thể được sử dụng để mô phỏng hành động nhấp chuột phải của chuột:

    $browser->rightClick();

    $browser->rightClick('.selector');

Phương thức `clickAndHold` có thể được sử dụng để mô phỏng một hành động nhấp và giữ chuột. Một lệnh gọi tiếp theo đến phương thức `releaseMouse` sẽ hoàn trả lại hành động này và thả nút giữ chuột ra:

    $browser->clickAndHold('.selector');

    $browser->clickAndHold()
            ->pause(1000)
            ->releaseMouse();

Phương thức `controlClick` có thể được sử dụng để mô phỏng một sự kiện `ctrl+click` trên trình duyệt:

    $browser->controlClick();

    $browser->controlClick('.selector');

<a name="mouseover"></a>
#### Mouseover

Phương thức `mouseover` có thể được sử dụng khi bạn cần di chuyển chuột qua một element giống với một CSS hoặc một Dusk selector đã cho:

    $browser->mouseover('.selector');

<a name="drag-drop"></a>
#### Drag và Drop

Phương thức `drag` có thể được sử dụng để kéo một element giống với một selector đã cho sang element khác:

    $browser->drag('.from-selector', '.to-selector');

Hoặc, bạn có thể kéo một element theo một hướng:

    $browser->dragLeft('.selector', $pixels = 10);
    $browser->dragRight('.selector', $pixels = 10);
    $browser->dragUp('.selector', $pixels = 10);
    $browser->dragDown('.selector', $pixels = 10);

Cuối cùng, bạn có thể drag một phần tử đi theo một khoảng nhất định:

    $browser->dragOffset('.selector', $x = 10, $y = 10);

<a name="javascript-dialogs"></a>
### JavaScript Dialogs

Dusk cung cấp nhiều phương thức khác nhau để tương tác với JavaScript Dialog. Ví dụ: bạn có thể sử dụng phương thức `waitForDialog` để đợi một dialog JavaScript xuất hiện. Phương thức này chấp nhận một tham số tùy chọn cho biết cần đợi bao nhiêu giây để một dialog xuất hiện:

    $browser->waitForDialog($seconds = null);

Phương thức `assertDialogOpened` có thể được sử dụng để xác nhận xem dialog đã được hiển thị và chứa một text hay chưa:

    $browser->assertDialogOpened('Dialog message');

Nếu một dialog JavaScript chứa một input, bạn có thể sử dụng phương thức `typeInDialog` để nhập một giá trị vào input đó:

    $browser->typeInDialog('Hello World');

Để đóng một dialog JavaScript đang mở bằng cách nhấp vào nút "OK", bạn có thể gọi phương thức `acceptDialog`:

    $browser->acceptDialog();

Để đóng một dialog JavaScript đang mở bằng cách nhấp vào nút "Cancel", bạn có thể gọi phương thức `dismissDialog`:

    $browser->dismissDialog();

<a name="interacting-with-iframes"></a>
### Tương tác với inline frames

Nếu bạn cần tương tác với một element trong một iframe, bạn có thể dùng phương thức `withinFrame`. Tất cả các tương tác element sẽ được diễn ra trong một closure và được cung cấp cho phương thức `withinFrame`, nó sẽ bị giới hạn trong iframe đã được chỉ định:

    $browser->withinFrame('#credit-card-details', function ($browser) {
        $browser->type('input[name="cardnumber"]', '4242424242424242')
            ->type('input[name="exp-date"]', '12/24')
            ->type('input[name="cvc"]', '123');
        })->press('Pay');
    });

<a name="scoping-selectors"></a>
### Scoping Selectors

Đôi khi bạn có thể muốn thực hiện một số thao tác trong một phạm vi selector đã cho. Ví dụ: bạn có thể muốn kiểm tra rằng có một số text chỉ được hiển thị trong một bảng và sau đó click vào một button trong bảng đó. Bạn có thể sử dụng phương thức `with` để thực hiện điều này. Tất cả các hoạt động được thực hiện trong hàm closure được đưa vào trong phương thức `with` và sẽ được thực hiện test trong phạm vi của selector đã chọn:

    $browser->with('.table', function (Browser $table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

Đôi khi bạn có thể cần chạy các kiểm tra bên ngoài phạm vi selector hiện tại. Bạn có thể sử dụng phương thức `elsewhere` và phương thức `elsewhereWhenAvailable` để thực hiện điều này:

    $browser->with('.table', function (Browser $table) {
        // Current scope is `body .table`...

        $browser->elsewhere('.page-title', function (Browser $title) {
            // Current scope is `body .page-title`...
            $title->assertSee('Hello World');
        });

        $browser->elsewhereWhenAvailable('.page-title', function (Browser $title) {
            // Current scope is `body .page-title`...
            $title->assertSee('Hello World');
        });
     });

<a name="waiting-for-elements"></a>
### Chờ Elements

Khi test các application sử dụng JavaScript chuyên sâu, thường phải "chờ" các element hoặc dữ liệu được load xong trước khi tiến hành test. Dusk thực hiện điều này một cách rất dễ dàng. Sử dụng nhiều phương thức khác nhau, bạn có thể đợi các element hiển thị hoàn toàn trên trang hoặc thậm chí đợi cho đến khi một biểu thức JavaScript trả về kết quả là `true`.

<a name="waiting"></a>
#### Waiting

Nếu bạn chỉ cần pause bài test trong một số mili giây nhất định, hãy sử dụng phương thức `pause`:

    $browser->pause(1000);

Nếu bạn chỉ cần tạm dừng kiểm tra nếu một điều kiện nhất định là `true`, thì hãy sử dụng phương thức `pauseIf`:

    $browser->pauseIf(App::environment('production'), 1000);

Tương tự, nếu bạn cần tạm dừng kiểm tra nếu một điều kiện nhất định là `false`, bạn có thể sử dụng phương thức `pauseUnless`:

    $browser->pauseUnless(App::environment('testing'), 1000);

<a name="waiting-for-selectors"></a>
#### Waiting For Selectors

Phương thức `waitFor` có thể được sử dụng để tạm dừng việc test cho đến khi element mà khớp với CSS hoặc Dusk selector được hiển thị trên trang. Mặc định, điều này sẽ tạm dừng bài test trong tối đa năm giây trước khi đưa ra một ngoại lệ. Nếu cần, bạn có thể truyền vào một ngưỡng thời gian chờ để làm tham số thứ hai cho phương thức:

    // Wait a maximum of five seconds for the selector...
    $browser->waitFor('.selector');

    // Wait a maximum of one second for the selector...
    $browser->waitFor('.selector', 1);

Bạn cũng có thể đợi cho đến khi một element khớp với một selector đã cho và chứa text đã cho:

    // Wait a maximum of five seconds for the selector to contain the given text...
    $browser->waitForTextIn('.selector', 'Hello World');

    // Wait a maximum of one second for the selector to contain the given text...
    $browser->waitForTextIn('.selector', 'Hello World', 1);

Bạn cũng có thể đợi cho đến khi một element khớp với một selector đã cho không còn hiển thị trên trang nữa:

    // Wait a maximum of five seconds until the selector is missing...
    $browser->waitUntilMissing('.selector');

    // Wait a maximum of one second until the selector is missing...
    $browser->waitUntilMissing('.selector', 1);

Hoặc, bạn có thể đợi cho đến khi một element khớp với một selector đã cho được enabled hoặc disabled:

    // Wait a maximum of five seconds until the selector is enabled...
    $browser->waitUntilEnabled('.selector');

    // Wait a maximum of one second until the selector is enabled...
    $browser->waitUntilEnabled('.selector', 1);

    // Wait a maximum of five seconds until the selector is disabled...
    $browser->waitUntilDisabled('.selector');

    // Wait a maximum of one second until the selector is disabled...
    $browser->waitUntilDisabled('.selector', 1);

<a name="scoping-selectors-when-available"></a>
#### Scoping Selectors When Available

Đôi khi, bạn có thể muốn đợi một element xuất hiện khớp với một selector nhất định và sau đó tương tác với element đó. Ví dụ, bạn có thể đợi cho đến khi một modal window được hiển thị và sau đó nhấn nút "OK" trong modal đó. Phương thức `whenAvailable` có thể được sử dụng để hoàn thành việc này. Tất cả các hoạt động của element được thực hiện trong closure sẽ nằm trong phạm vi của selector ban đầu:

    $browser->whenAvailable('.modal', function (Browser $modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

<a name="waiting-for-text"></a>
#### Waiting For Text

Phương thức `waitForText` có thể được sử dụng để đợi cho đến khi một text được hiển thị trên trang:

    // Wait a maximum of five seconds for the text...
    $browser->waitForText('Hello World');

    // Wait a maximum of one second for the text...
    $browser->waitForText('Hello World', 1);

Bạn có thể sử dụng phương thức `waitUntilMissingText` để đợi cho đến khi văn bản đang được hiển thị bị xóa khỏi trang:

    // Wait a maximum of five seconds for the text to be removed...
    $browser->waitUntilMissingText('Hello World');

    // Wait a maximum of one second for the text to be removed...
    $browser->waitUntilMissingText('Hello World', 1);

<a name="waiting-for-links"></a>
#### Waiting For Links

Phương thức `waitForLink` có thể được sử dụng để đợi cho đến khi một link đã cho được hiển thị trên trang:

    // Wait a maximum of five seconds for the link...
    $browser->waitForLink('Create');

    // Wait a maximum of one second for the link...
    $browser->waitForLink('Create', 1);

<a name="waiting-for-inputs"></a>
#### Waiting For Inputs

Phương thức `waitForInput` có thể được sử dụng để đợi cho đến khi input đã cho hiển thị trên trang:

    // Wait a maximum of five seconds for the input...
    $browser->waitForInput($field);

    // Wait a maximum of one second for the input...
    $browser->waitForInput($field, 1);

<a name="waiting-on-the-page-location"></a>
#### Waiting On The Page Location

Khi thực hiện kiểm tra đường dẫn, chẳng hạn như `$browser->assertPathIs('/home')`, kiểm tra đó có thể thất bại nếu `window.location.pathname` không đồng bộ với nhau. Bạn có thể sử dụng phương thức `waitForLocation` để đợi cho đến khi window.location là một giá trị nhất định:

    $browser->waitForLocation('/secret');

Phương thức `waitForLocation` cũng có thể được sử dụng để đợi cửa sổ hiện tại trỏ vào một URL điều kiện:

    $browser->waitForLocation('https://example.com/path');

Bạn cũng có thể đợi cửa sổ hiện tại thành [tên một của một route](/docs/{{version}}/routing#named-routes):

    $browser->waitForRoute($routeName, $parameters);

<a name="waiting-for-page-reloads"></a>
#### Waiting For Page Reloads

Nếu bạn cần đợi một trang load lại sau khi thực hiện một hành động nào đó, hãy sử dụng phương thức `waitForReload`:

    use Laravel\Dusk\Browser;

    $browser->waitForReload(function (Browser $browser) {
        $browser->press('Submit');
    })
    ->assertSee('Success!');

Vì nhu cầu đợi load lại trang thường xảy ra sau khi nhấp vào một nút nào đó, nên bạn có thể sử dụng phương thức `clickAndWaitForReload` để thực hiện nó:

    $browser->clickAndWaitForReload('.selector')
            ->assertSee('something');

<a name="waiting-on-javascript-expressions"></a>
#### Waiting On JavaScript Expressions

Thỉnh thoảng bạn có thể muốn tạm dừng việc kiểm tra cho đến khi một biểu thức JavaScript trả về giá trị là `true`. Bạn có thể dễ dàng thực hiện điều này bằng cách sử dụng phương thức `waitUntil`. Khi truyền một biểu thức cho phương thức này, bạn không cần thêm từ khóa `return` hoặc dấu chấm phẩy kết thúc:

    // Wait a maximum of five seconds for the expression to be true...
    $browser->waitUntil('App.data.servers.length > 0');

    // Wait a maximum of one second for the expression to be true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

<a name="waiting-on-vue-expressions"></a>
#### Waiting On Vue Expressions

Các phương thức `waitUntilVue` và `waitUntilVueIsNot` có thể được sử dụng để đợi cho đến khi một thuộc tính [Vue component](https://vuejs.org) có một giá trị nhất định:

    // Wait until the component attribute contains the given value...
    $browser->waitUntilVue('user.name', 'Taylor', '@user');

    // Wait until the component attribute doesn't contain the given value...
    $browser->waitUntilVueIsNot('user.name', null, '@user');

<a name="waiting-for-javascript-events"></a>
#### Waiting For JavaScript Events

Phương thức `waitForEvent` có thể được sử dụng để tạm dừng quá trình chạy thử nghiệm cho đến khi xảy ra một sự kiện JavaScript:

    $browser->waitForEvent('load');

Event listener sẽ được gắn vào phạm vi selector hiện tại, mặc định là element `body`. Khi sử dụng một phạm vi selector, event listener sẽ được gắn vào element đó:

    $browser->with('iframe', function (Browser $iframe) {
        // Wait for the iframe's load event...
        $iframe->waitForEvent('load');
    });

Bạn cũng có thể cung cấp một selector làm tham số thứ hai cho phương thức `waitForEvent` để gắn một event listener vào một element cụ thể:

    $browser->waitForEvent('load', '.selector');

Bạn cũng có thể đợi các sự kiện trên các đối tượng `document` và `window`:

    // Wait until the document is scrolled...
    $browser->waitForEvent('scroll', 'document');

    // Wait a maximum of five seconds until the window is resized...
    $browser->waitForEvent('resize', 'window', 5);

<a name="waiting-with-a-callback"></a>
#### Waiting With A Callback

Nhiều phương thức "chờ" trong Dusk được dựa trên phương thức `waitUsing` bên dưới. Bạn có thể sử dụng phương thức này trực tiếp để chờ cho đến khi một closure trả về giá trị `true`. Phương thức `waitUsing` nhận vào số giây chờ tối đa mà bài test có thể được thực hiện và một khoảng thời gian lặp cho closure và một closure và một tuỳ chọn thông báo lỗi:

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="scrolling-an-element-into-view"></a>
### Scrolling một phần tử vào view

Đôi khi bạn không thể nhấp vào một phần tử vì nó nằm ngoài vùng xem của trình duyệt. Phương thức `scrollIntoView` sẽ cuộn cửa sổ trình duyệt cho đến khi phần tử đó nằm trong vùng có thể xem được của trình duyệt:

    $browser->scrollIntoView('.selector')
            ->click('.selector');

<a name="available-assertions"></a>
## Assertion có sẵn

Dusk cung cấp nhiều yêu cầu kiểm tra mà bạn có thể đưa ra đối với ứng dụng của bạn. Tất cả các yêu cầu kiểm tra có sẵn đều được ghi lại trong danh sách dưới đây:

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertSchemeIs](#assert-scheme-is)
[assertSchemeIsNot](#assert-scheme-is-not)
[assertHostIs](#assert-host-is)
[assertHostIsNot](#assert-host-is-not)
[assertPortIs](#assert-port-is)
[assertPortIsNot](#assert-port-is-not)
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
[assertHasPlainCookie](#assert-has-plain-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertPlainCookieMissing](#assert-plain-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSeeAnythingIn](#assert-see-anything-in)
[assertSeeNothingIn](#assert-see-nothing-in)
[assertScript](#assert-script)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertIndeterminate](#assert-indeterminate)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertSelectMissingOption](#assert-select-missing-option)
[assertValue](#assert-value)
[assertValueIsNot](#assert-value-is-not)
[assertAttribute](#assert-attribute)
[assertAttributeContains](#assert-attribute-contains)
[assertAttributeDoesntContain](#assert-attribute-doesnt-contain)
[assertAriaAttribute](#assert-aria-attribute)
[assertDataAttribute](#assert-data-attribute)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertNotPresent](#assert-not-present)
[assertMissing](#assert-missing)
[assertInputPresent](#assert-input-present)
[assertInputMissing](#assert-input-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertButtonEnabled](#assert-button-enabled)
[assertButtonDisabled](#assert-button-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertAuthenticated](#assert-authenticated)
[assertGuest](#assert-guest)
[assertAuthenticatedAs](#assert-authenticated-as)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesntContain](#assert-vue-doesnt-contain)

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

<a name="assert-scheme-is"></a>
#### assertSchemeIs

Yêu cầu scheme của URL hiện tại phải đúng với scheme đã cho:

    $browser->assertSchemeIs($scheme);

<a name="assert-scheme-is-not"></a>
#### assertSchemeIsNot

Yêu cầu scheme của URL hiện tại không phải scheme đã cho:

    $browser->assertSchemeIsNot($scheme);

<a name="assert-host-is"></a>
#### assertHostIs

Yêu cầu host của URL hiện tại phải đúng với host đã cho:

    $browser->assertHostIs($host);

<a name="assert-host-is-not"></a>
#### assertHostIsNot

Yêu cầu host của URL hiện tại không phải host đã cho:

    $browser->assertHostIsNot($host);

<a name="assert-port-is"></a>
#### assertPortIs

Yêu cầu port của URL hiện tại phải đúng với port đã cho:

    $browser->assertPortIs($port);

<a name="assert-port-is-not"></a>
#### assertPortIsNot

Yêu cầu port của URL hiện tại không phải port đã cho:

    $browser->assertPortIsNot($port);

<a name="assert-path-begins-with"></a>
#### assertPathBeginsWith

Yêu cầu path của URL hiện tại phải bắt đầu từ path đã cho:

    $browser->assertPathBeginsWith('/home');

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

Yêu cầu URL hiện tại phải đúng với URL của một [route đã cho](/docs/{{version}}/routing#named-routes):

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

Yêu cầu fragment hiệnt tại của URL phải đúng với fragment đã cho:

    $browser->assertFragmentIs('anchor');

<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

Yêu cầu fragment hiệnt tại của URL phải bắt đầu từ fragment đã cho:

    $browser->assertFragmentBeginsWith('anchor');

<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

Yêu cầu fragment hiệnt tại của URL không phải là fragment đã cho:

    $browser->assertFragmentIsNot('anchor');

<a name="assert-has-cookie"></a>
#### assertHasCookie

Yêu cầu cookie mã hoá đã cho phải tồn tại:

    $browser->assertHasCookie($name);

<a name="assert-has-plain-cookie"></a>
#### assertHasPlainCookie

Yêu cầu cookie không mã hoá đã cho phải tồn tại:

    $browser->assertHasPlainCookie($name);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Yêu cầu cookie mã hoá đã cho phải không tồn tại:

    $browser->assertCookieMissing($name);

<a name="assert-plain-cookie-missing"></a>
#### assertPlainCookieMissing

Yêu cầu cookie không mã hoá đã cho phải không tồn tại:

    $browser->assertPlainCookieMissing($name);

<a name="assert-cookie-value"></a>
#### assertCookieValue

Yêu cầu cookie mã hoá phải có một giá trị đã cho:

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

<a name="assert-see-anything-in"></a>
#### assertSeeAnythingIn

Yêu cầu mọi text đều phải hiển thị trong selector:

    $browser->assertSeeAnythingIn($selector);

<a name="assert-see-nothing-in"></a>
#### assertSeeNothingIn

Yêu cầu không text nào hiển thị trong selector:

    $browser->assertSeeNothingIn($selector);

<a name="assert-script"></a>
#### assertScript

Yêu cầu biểu thức JavaScript đã cho sẽ được so sánh giá trị đã cho:

    $browser->assertScript('window.isLoaded')
            ->assertScript('document.readyState', 'complete');

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

<a name="assert-indeterminate"></a>
#### assertIndeterminate

Yêu cầu checkbox ở trạng thái indeterminate:

    $browser->assertIndeterminate($field);

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

<a name="assert-select-missing-option"></a>
#### assertSelectMissingOption

Yêu cầu giá trị đã cho không tồn tại để được chọn:

    $browser->assertSelectMissingOption($field, $value);

<a name="assert-value"></a>
#### assertValue

Yêu cầu element giống với selector đã cho:

    $browser->assertValue($selector, $value);

<a name="assert-value-is-not"></a>
#### assertValueIsNot

Yêu cầu element giống với một selector không có giá trị đã cho:

    $browser->assertValueIsNot($selector, $value);

<a name="assert-attribute"></a>
#### assertAttribute

Yêu cầu element giống với selector đã cho có giá trị thuộc tính là giá trị đã cung cấp:

    $browser->assertAttribute($selector, $attribute, $value);

<a name="assert-attribute-contains"></a>
#### assertAttributeContains

Yêu cầu element giống với selector đã cho chứa giá trị trong thuộc tính được cung cấp:

    $browser->assertAttributeContains($selector, $attribute, $value);

<a name="assert-attribute-doesnt-contain"></a>
#### assertAttributeDoesntContain

Yêu cầu element giống với selector đã cho không chứa giá trị trong thuộc tính được cung cấp:

    $browser->assertAttributeDoesntContain($selector, $attribute, $value);

<a name="assert-aria-attribute"></a>
#### assertAriaAttribute

Yêu cầu element giống với selector đã cho có giá trị thuộc tính aria là giá trị đã cung cấp:

    $browser->assertAriaAttribute($selector, $attribute, $value);

Ví dụ: với thẻ button `<button aria-label =" Add "> </button>`, bạn có thể kiểm tra thuộc tính `aria-label` như sau:

    $browser->assertAriaAttribute('button', 'label', 'Add')

<a name="assert-data-attribute"></a>
#### assertDataAttribute

Yêu cầu element giống với selector đã cho có giá trị thuộc tính data là giá trị đã cung cấp:

    $browser->assertDataAttribute($selector, $attribute, $value);

Ví dụ: với thẻ tr `<tr id="row-1" data-content="attendees"></tr>`, bạn có thể kiểm tra thuộc tính `data-label` như sau:

    $browser->assertDataAttribute('#row-1', 'content', 'attendees')

<a name="assert-visible"></a>
#### assertVisible

Yêu cầu element giống với selector đã cho là hiển thị:

    $browser->assertVisible($selector);

<a name="assert-present"></a>
#### assertPresent

Yêu cầu element giống với selector đã cho là tồn tại trong source:

    $browser->assertPresent($selector);

<a name="assert-not-present"></a>
#### assertNotPresent

Yêu cầu element giống với selector đã cho không có trong source:

    $browser->assertNotPresent($selector);

<a name="assert-missing"></a>
#### assertMissing

Yêu cầu element giống với selector đã cho là không hiển thị:

    $browser->assertMissing($selector);

<a name="assert-input-present"></a>
#### assertInputPresent

Yêu cầu có một input với tên đã cho:

    $browser->assertInputPresent($name);

<a name="assert-input-missing"></a>
#### assertInputMissing

Yêu cầu input có tên đã cho không có trong source:

    $browser->assertInputMissing($name);

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

<a name="assert-button-enabled"></a>
#### assertButtonEnabled

Yêu cầu button đã cho đang được enabled:

    $browser->assertButtonEnabled($button);

<a name="assert-button-disabled"></a>
#### assertButtonDisabled

Yêu cầu button đã cho đang bị disable:

    $browser->assertButtonDisabled($button);

<a name="assert-focused"></a>
#### assertFocused

Yêu cầu field đã cho đang bị focus:

    $browser->assertFocused($field);

<a name="assert-not-focused"></a>
#### assertNotFocused

Yêu cầu field đã cho không bị focus:

    $browser->assertNotFocused($field);

<a name="assert-authenticated"></a>
#### assertAuthenticated

Yêu cầu người dùng phải được xác thực:

    $browser->assertAuthenticated();

<a name="assert-guest"></a>
#### assertGuest

Yêu cầu người dùng chưa được xác thực:

    $browser->assertGuest();

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

Yêu cầu người dùng được xác thực phải là người dùng đã cho:

    $browser->assertAuthenticatedAs($user);

<a name="assert-vue"></a>
#### assertVue

Dusk thậm chí còn cho phép bạn đưa ra các assertion về trạng thái của dữ liệu trong [component Vue](https://vuejs.org). Ví dụ: hãy tưởng tượng ứng dụng của bạn chứa component Vue như sau:

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

Bạn có thể assert trạng thái của component Vue như sau:

    /**
     * A basic Vue test example.
     */
    public function test_vue(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

Yêu cầu một thuộc tính dữ liệu của Vue component khác với giá trị đã cho:

    $browser->assertVueIsNot($property, $value, $componentSelector = null);

<a name="assert-vue-contains"></a>
#### assertVueContains

Yêu cầu một thuộc tính dữ liệu của Vue component là một mảng và có chứa giá trị đã cho:

    $browser->assertVueContains($property, $value, $componentSelector = null);

<a name="assert-vue-doesnt-contain"></a>
#### assertVueDoesntContain

Yêu cầu một thuộc tính dữ liệu của Vue component là một mảng và không chứa giá trị đã cho:

    $browser->assertVueDoesntContain($property, $value, $componentSelector = null);

<a name="pages"></a>
## Page

Đôi khi, các bài test yêu cầu một số hành động phức tạp được thực hiện theo một trình tự nhất định. Điều này có thể làm cho bài test của bạn khó đọc hiểu hơn. Page của Dusk cho phép bạn định nghĩa các hành động có thể được thực hiện trên một trang nhất định thông qua một phương thức duy nhất. Page cũng cho phép bạn định nghĩa các short-cut cho các common selector trong application của bạn hoặc cho một trang.

<a name="generating-pages"></a>
### Tạo Page

Để tạo một page object, chạy lệnh Artisan `dusk:page`. Tất cả các page object sẽ được lưu trong thư mục `tests/Browser/Pages` của application của bạn:

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### Cài đặt Page

Mặc định, các page có ba phương thức: `url`, `assert`, và `elements`. Bây giờ chúng ta sẽ thảo luận về các phương thức `url` và `assert`. Phương thức `elements` sẽ được [thảo luận chi tiết hơn bên dưới](#shorthand-selectors).

<a name="the-url-method"></a>
#### The `url` Method

Phương thức `url` sẽ trả về đường dẫn của URL đến một trang. Dusk sẽ sử dụng URL này khi điều hướng đến trang đó trong trình duyệt:

    /**
     * Get the URL for the page.
     */
    public function url(): string
    {
        return '/login';
    }

<a name="the-assert-method"></a>
#### The `assert` Method

Phương thức `assert` có thể đưa ra bất kỳ yêu cầu nào cần thiết để kiểm tra là trình duyệt đã thực sự mở trang đó hay chưa. Thực sự không cần thiết phải đặt bất cứ thứ gì vào trong phương thức này; tuy nhiên, bạn có thể tự do đưa ra những yêu cầu này nếu muốn. Các yêu cầu này sẽ được chạy tự động khi mở trang đó:

    /**
     * Assert that the browser is on the page.
     */
    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### Điều hướng tới Page

Khi một page đã được định nghĩa, bạn có thể điều hướng đến nó bằng phương thức `visit`:

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

Phương thức `elements` trong class page cho phép bạn định nghĩa các shortcut nhanh, dễ nhớ cho bất kỳ CSS selector nào trên trang của bạn. Ví dụ: hãy định nghĩa shortcut cho field input "email" trong trang đăng nhập của application:

    /**
     * Get the element shortcuts for the page.
     *
     * @return array<string, string>
     */
    public function elements(): array
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

Khi shortcut đã được định nghĩa, bạn có thể sử dụng shorthand selector này ở bất cứ nơi nào mà bạn muốn sử dụng CSS selector đó:

    $browser->type('@email', 'taylor@laravel.com');

<a name="global-shorthand-selectors"></a>
#### Global Shorthand Selectors

Sau khi cài đặt Dusk, một class `Page` sẽ được lưu vào trong thư mục `tests/Browser/Pages` của bạn. Class này chứa một phương thức `siteElements` có thể được sử dụng để định nghĩa các global shorthand selector mà sẽ được khai báo sẵn trên mỗi trang trong application của bạn:

    /**
     * Get the global element shortcuts for the site.
     *
     * @return array<string, string>
     */
    public static function siteElements(): array
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
         */
        public function createPlaylist(Browser $browser, string $name): void
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

Khi phương thức đã được định nghĩa xong, bạn có thể sử dụng nó trong bất kỳ bài test nào mà bạn đang sử dụng page đó. Instance browser sẽ được tự động truyền tham số đầu tiên cho các phương thức page tùy chỉnh:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## Component

Các component cũng tương tự như các đối tượng page của Dusk, nhưng được dành cho các thành phần của UI và các chức năng được sử dụng lại trong toàn bộ application của bạn, như một navigation bar hoặc một notification window. Như vậy, các component không bị ràng buộc với các URL cụ thể.

<a name="generating-components"></a>
### Tạo Component

Để tạo một component, bạn hãy chạy lệnh Artisan `dusk:component`. Các component mới sẽ được lưu vào trong thư mục `tests/Browser/Components`:

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
         */
        public function selector(): string
        {
            return '.date-picker';
        }

        /**
         * Assert that the browser page contains the component.
         */
        public function assert(Browser $browser): void
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * Get the element shortcuts for the component.
         *
         * @return array<string, string>
         */
        public function elements(): array
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@year-list' => 'div > div.datepicker-years',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * Select the given date.
         */
        public function selectDate(Browser $browser, int $year, int $month, int $day): void
        {
            $browser->click('@date-field')
                    ->within('@year-list', function (Browser $browser) use ($year) {
                        $browser->click($year);
                    })
                    ->within('@month-list', function (Browser $browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function (Browser $browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### Dùng Component

Khi component đã được định nghĩa xong, chúng ta có thể dễ dàng chọn một ngày trong date picker, từ bất kỳ bài test nào. Và, nếu chúng ta cần thay đổi logic chọn ngày, chúng ta chỉ cần cập nhật component:

    <?php

    namespace Tests\Browser;

    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        /**
         * A basic component test example.
         */
        public function test_basic_example(): void
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function (Browser $browser) {
                            $browser->selectDate(2019, 1, 30);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>
## Test tích hợp

> [!WARNING]
> Hầu hết các cấu hình continuous integration của Dusk đều yêu cầu ứng dụng Laravel của bạn được khởi tạo bằng cách sử dụng máy chủ được tích hợp sẵn trong PHP trên cổng 8000. Do đó, trước khi tiếp tục, bạn nên đảm bảo rằng môi trường continuous integration của bạn có giá trị biến môi trường `APP_URL` là `http://127.0.0.1:8000`.

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
            "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve --no-reload > /dev/null 2>&1 &' && php artisan dusk"
          }
        }
      }
    }

<a name="running-tests-on-travis-ci"></a>
### Travis CI

Để chạy các bài Dusk test của bạn trên [Travis CI](https://travis-ci.org), bạn có thể dùng file cấu hình `.travis.yml` sau. Vì Travis CI không phải là một môi trường đồ họa, nên chúng ta sẽ cần thực hiện thêm một số bước để chạy trình duyệt Chrome. Ngoài ra, chúng ta cũng sẽ sử dụng `php artisan serve` để chạy server web tích hợp sẵn của PHP:

```yaml
language: php

php:
  - 7.3

addons:
  chrome: stable

install:
  - cp .env.testing .env
  - travis_retry composer install --no-interaction --prefer-dist
  - php artisan key:generate
  - php artisan dusk:chrome-driver

before_script:
  - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
  - php artisan serve --no-reload &

script:
  - php artisan dusk
```

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

Nếu bạn đang sử dụng [GitHub Actions](https://github.com/features/actions) để chạy các bài test Laravel Dusk, bạn có thể sử dụng file cấu hình ở dưới để bắt đầu. Giống như TravisCI, chúng ta sẽ sử dụng lệnh `php artisan serve` để khởi chạy một web server được tích hợp sẵn trong PHP:

```yaml
name: CI
on: [push]
jobs:

  dusk-php:
    runs-on: ubuntu-latest
    env:
      APP_URL: "http://127.0.0.1:8000"
      DB_USERNAME: root
      DB_PASSWORD: root
      MAIL_MAILER: log
    steps:
      - uses: actions/checkout@v4
      - name: Prepare The Environment
        run: cp .env.example .env
      - name: Create Database
        run: |
          sudo systemctl start mysql
          mysql --user="root" --password="root" -e "CREATE DATABASE \`my-database\` character set UTF8mb4 collate utf8mb4_bin;"
      - name: Install Composer Dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader
      - name: Generate Application Key
        run: php artisan key:generate
      - name: Upgrade Chrome Driver
        run: php artisan dusk:chrome-driver --detect
      - name: Start Chrome Driver
        run: ./vendor/laravel/dusk/bin/chromedriver-linux &
      - name: Run Laravel Server
        run: php artisan serve --no-reload &
      - name: Run Dusk Tests
        run: php artisan dusk
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - name: Upload Console Logs
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: console
          path: tests/Browser/console
```

<a name="running-tests-on-chipper-ci"></a>
### Chipper CI

Nếu bạn dùng [Chipper CI](https://chipperci.com) để chạy Dusk test của bạn, bạn có thể dùng file cấu hình dưới đây để bắt đầu. Chúng ta sẽ sử dụng một server có sẵn của PHP để chạy Laravel, vì vậy bạn có thể listen cho request:

```yaml
# file .chipperci.yml
version: 1

environment:
  php: 8.2
  node: 16

# Include Chrome in the build environment
services:
  - dusk

# Build all commits
on:
   push:
      branches: .*

pipeline:
  - name: Setup
    cmd: |
      cp -v .env.example .env
      composer install --no-interaction --prefer-dist --optimize-autoloader
      php artisan key:generate

      # Create a dusk env file, ensuring APP_URL uses BUILD_HOST
      cp -v .env .env.dusk.ci
      sed -i "s@APP_URL=.*@APP_URL=http://$BUILD_HOST:8000@g" .env.dusk.ci

  - name: Compile Assets
    cmd: |
      npm ci --no-audit
      npm run build

  - name: Browser Tests
    cmd: |
      php -S [::0]:8000 -t public 2>server.log &
      sleep 2
      php artisan dusk:chrome-driver $CHROME_DRIVER
      php artisan dusk --env=ci
```

Để biết thêm về chạy Dusk test trên Chipper CI, bao gồm cả việc làm sao để dùng cơ sở dữ liệu, bạn hãy xem [tài liệu Chipper CI chính thức](https://chipperci.com/docs/testing/laravel-dusk-new/).
