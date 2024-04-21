# Testing: Getting Started

- [Giới thiệu](#introduction)
- [Environment](#environment)
- [Tạo testcase](#creating-tests)
- [Chạy testcase](#running-tests)
    - [Chạy testcase đồng thời](#running-tests-in-parallel)
    - [Báo cáo phạm vi chạy testcase](#reporting-test-coverage)

<a name="introduction"></a>
## Giới thiệu

Laravel được xây dựng với mục đích để test. Thực tế là, Laravel đã mặc định hỗ trợ việc test với PHPUnit và một file `phpunit.xml` đã được cài đặt sẵn trong application của bạn. Framework này cũng có các phương thức trợ giúp thuận tiện cho phép bạn test application của bạn.

Mặc định, thư mục `tests` của application sẽ chứa hai thư mục: `Feature` và `Unit`. Các test unit là các bài test tập trung vào một phần rất nhỏ, tách biệt hoàn toàn trong code của bạn. Thực tế là, hầu hết các bài test unit có thể tập trung vào duy nhất một phương thức. Các bài test trong thư mục test "Unit" sẽ không khởi động ứng dụng Laravel của bạn và do đó sẽ không thể truy cập được vào cơ sở dữ liệu của ứng dụng hoặc các service khác của framework .

Feature test cũng có thể test được một phần lớn hơn code của bạn, chứa cả cách mà một số đối tượng tương tác với nhau hoặc thậm chí là một HTTP request được gửi tới một JSON endpoint. **Nói chung, hầu hết các bài test của bạn phải là bài feature test. Loại test này sẽ mang lại sự tự tin cao nhất cho toàn bộ hệ thống của bạn đang hoạt động như dự kiến.**

Một file `exampleTest.php` mẫu cũng đã được cung cấp sẵn ở trong hai thư mục test `Feature` và `Unit`. Sau khi bạn đã cài đặt một application Laravel mới, bạn hãy chạy `vendor/bin/phpunit` hoặc `php artisan test` trên cửa sổ dòng lệnh để chạy bài test của bạn.

<a name="environment"></a>
## Environment

Khi chạy test, Laravel sẽ tự động set [cấu hình môi trường](/docs/{{version}}/configuration#environment-configuration) là `testing` bởi vì các biến môi trường đã được định nghĩa trong file `phpunit.xml`. Laravel cũng tự động cấu hình session và cache là driver `array` trong khi test, điều này nghĩa là không có session hoặc cache nào được duy trì trong khi bạn test.

Bạn có thể tự do định nghĩa các giá trị cấu hình khác cho môi trường test nếu cần thiết. Các biến môi trường `testing` có thể được cấu hình trong file `phpunit.xml` của application của bạn, nhưng hãy đảm bảo là bạn đã xóa cấu hình cache của bạn bằng cách sử dụng lệnh Artisan `config:clear` trước khi chạy bài test của bạn!

<a name="the-env-testing-environment-file"></a>
#### The `.env.testing` Environment File

Ngoài ra, bạn có thể tạo file `.env.testing` trong thư mục gốc của project của bạn. File này sẽ được dùng để thay thế file `.env` khi chạy các bài test PHPUnit hoặc chạy các lệnh Artisan với tùy chọn `--env=testing`.

<a name="the-creates-application-trait"></a>
#### The `CreatesApplication` Trait

Laravel có chứa một trait `CreatesApplication` được áp dụng cho class `TestCase` base trong ứng dụng của bạn. Trait này chứa phương thức `createApplication` để khởi động ứng dụng Laravel trước khi chạy bài test của bạn. Điều quan trọng là bạn phải để trait này ở vị trí ở đầu vì có một số tính năng, chẳng hạn như tính năng test song song của Laravel, phụ thuộc vào nó.

<a name="creating-tests"></a>
## Tạo testcase

Để tạo một test case mới, hãy sử dụng lệnh Artisan `make:test`. Mặc định, các bài test sẽ được lưu trong thư mục `tests/Feature`:

```shell
php artisan make:test UserTest
```

Nếu bạn muốn tạo một bài test trong thư mục `tests/Unit`, bạn có thể sử dụng tùy chọn `--unit` khi chạy lệnh `make:test`:

```shell
php artisan make:test UserTest --unit
```

Nếu muốn tạo một bài test [Pest PHP](https://pestphp.com), bạn có thể cung cấp tùy chọn `--pest` cho lệnh `make:test`:

```shell
php artisan make:test UserTest --pest
php artisan make:test UserTest --unit --pest
```

> **Note**
> Các stub của test có thể được tùy chỉnh bằng cách sử dụng [export stub](/docs/{{version}}/artisan#stub-customization).

Khi file test đã được tạo xong, bạn có thể định nghĩa các phương thức test như khi sử dụng với [PHPUnit](https://phpunit.de). Để chạy test của bạn, hãy chạy lệnh `vendor/bin/phpunit` hoặc lệnh `php artisan test` từ terminal của bạn:

    <?php

    namespace Tests\Unit;

    use PHPUnit\Framework\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function test_basic_test()
        {
            $this->assertTrue(true);
        }
    }

> **Warning**
> Nếu bạn định nghĩa một phương thức `setUp` / `tearDown` của riêng bạn trong một test class, hãy nhớ gọi các phương thức `parent::setUp()` / `parent::tearDown()` tương ứng ở trong class parent.

<a name="running-tests"></a>
## Chạy testcase

Như đã đề cập trước đó, khi bạn đã viết xong bài test, bạn có thể chạy chúng bằng cách sử dụng `phpunit`:

```shell
./vendor/bin/phpunit
```

Ngoài lệnh `phpunit`, bạn có thể sử dụng lệnh Artisan `test` để chạy các bài test của bạn. Artisan test runner cung cấp các báo cáo test chi tiết để dễ dàng phát triển và gỡ lỗi:

```shell
php artisan test
```

Bất kỳ tham số nào mà có thể được truyền vào cho lệnh `phpunit` thì cũng có thể được truyền vào cho lệnh Artisan `test`:

```shell
php artisan test --testsuite=Feature --stop-on-failure
```

<a name="running-tests-in-parallel"></a>
### Chạy testcase đồng thời

Mặc định, Laravel và PHPUnit thực hiện các bài test của bạn theo thứ tự trong một process duy nhất. Tuy nhiên, bạn có thể giảm đáng kể lượng thời gian cần thiết để chạy các bài test bằng cách chạy các bài test đó đồng thời trên nhiều process. Để bắt đầu, hãy đảm bảo ứng dụng của bạn sử dụng library phiên bản `^5.3` trở lên của package `nunomaduro/collision`. Sau đó, thêm tùy chọn `--parallel` khi chạy lệnh Artisan `test`:

```shell
php artisan test --parallel
```

Mặc định, Laravel sẽ tạo số process bằng với số lõi CPU có sẵn trên máy của bạn. Tuy nhiên, bạn có thể điều chỉnh số lượng process này bằng tùy chọn `--processes`:

```shell
php artisan test --parallel --processes=4
```

> **Warning**
> Khi chạy test đồng thời, một số tùy chọn PHPUnit (chẳng hạn như `--do-not-cache-result`) có thể không khả dụng.

<a name="parallel-testing-and-databases"></a>
#### Parallel Testing và Databases

Miễn là bạn đã cấu hình kết nối cơ sở dữ liệu của bạn, laravel sẽ tự động xử lý việc tạo và migration cơ sở dữ liệu test cho từng process song song đang chạy test của bạn. Cơ sở dữ liệu test sẽ được gắn với một process token duy nhất cho mỗi process. Ví dụ: nếu bạn có hai process test song song, Laravel sẽ tạo và sử dụng cơ sở dữ liệu test là `your_db_test_1` và `your_db_test_2`.

Mặc định, cơ sở dữ liệu test vẫn tồn tại giữa các lần gọi lệnh Artisan `test` để chúng có thể được sử dụng lại cho các lần gọi `test` tiếp theo. Tuy nhiên, bạn có thể tạo lại chúng bằng tùy chọn `--recreate-databases`:

```shell
php artisan test --parallel --recreate-databases
```

<a name="parallel-testing-hooks"></a>
#### Parallel Testing Hooks

Đôi khi, bạn có thể cần chuẩn bị một số resource nhất định được dùng bởi các bài test của ứng dụng để chúng có thể được sử dụng một cách an toàn trong nhiều process test.

Bằng cách sử dụng facade `ParallelTesting`, bạn có thể chỉ định code nào sẽ được chạy trên `setUp` và `tearDown` của một process hoặc một test case. Các closure đã cho sẽ nhận các biến `$token` và `$testCase` lần lượt là process token và test case hiện tại:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Artisan;
    use Illuminate\Support\Facades\ParallelTesting;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            ParallelTesting::setUpProcess(function ($token) {
                // ...
            });

            ParallelTesting::setUpTestCase(function ($token, $testCase) {
                // ...
            });

            // Executed when a test database is created...
            ParallelTesting::setUpTestDatabase(function ($database, $token) {
                Artisan::call('db:seed');
            });

            ParallelTesting::tearDownTestCase(function ($token, $testCase) {
                // ...
            });

            ParallelTesting::tearDownProcess(function ($token) {
                // ...
            });
        }
    }

<a name="accessing-the-parallel-testing-token"></a>
#### Accessing The Parallel Testing Token

Nếu bạn muốn truy cập vào "token" process song song hiện tại từ bất kỳ vị trí nào trong code kiểm tra của ứng dụng, bạn có thể sử dụng phương thức `token`. token này là một chuỗi nhận dạng, và duy nhất cho một process test riêng biệt và có thể được sử dụng để phân chia resource trên các process test song song. Ví dụ: Laravel tự động thêm token này vào cuối của tên cơ sở dữ liệu test được tạo bởi process test song song:

    $token = ParallelTesting::token();

<a name="reporting-test-coverage"></a>
### Báo cáo phạm vi chạy testcase

> **Warning**
> Tính năng này yêu cầu [Xdebug](https://xdebug.org) hoặc [PCOV](https://pecl.php.net/package/pcov).

Khi chạy test ứng dụng, bạn có thể muốn xác định xem các test case của bạn có thực sự bao phủ code ứng dụng của bạn hay không và có bao nhiêu code trong ứng dụng của bạn được sử dụng khi chạy test. Để thực hiện điều này, bạn có thể cung cấp tùy chọn `--coverage` khi gọi lệnh `test`:

```shell
php artisan test --coverage
```

<a name="enforcing-a-minimum-coverage-threshold"></a>
#### Enforcing A Minimum Coverage Threshold

Bạn có thể sử dụng tùy chọn `--min` để định nghĩa ngưỡng phạm vi kiểm tra tối thiểu cho ứng dụng của bạn. Bộ testcase sẽ không thành công nếu ngưỡng này không được đáp ứng:

```shell
php artisan test --coverage --min=80.3
```
