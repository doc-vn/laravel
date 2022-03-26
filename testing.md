# Testing: Getting Started

- [Giới thiệu](#introduction)
- [Environment](#environment)
- [Tạo và chạy Test](#creating-and-running-tests)

<a name="introduction"></a>
## Giới thiệu

Laravel được xây dựng với mục đích để test. Thực tế là, Laravel đã mặc định hỗ trợ việc test với PHPUnit và một file `phpunit.xml` đã được cài đặt sẵn trong application của bạn. Framework này cũng có các phương thức trợ giúp thuận tiện cho phép bạn test application của bạn.

Mặc định, thư mục `tests` của application sẽ chứa hai thư mục: `Feature` và `Unit`. Các test unit là các bài test tập trung vào một phần rất nhỏ, tách biệt hoàn toàn trong code của bạn. Thực tế là, hầu hết các bài test unit có thể tập trung vào duy nhất một phương thức. Chức năng test cũng có thể test được một phần lớn hơn code của bạn, chứa cả cách mà một số đối tượng tương tác với nhau hoặc thậm chí là một HTTP request được gửi tới một JSON endpoint.

Một file `exampleTest.php` mẫu cũng đã được cung cấp sẵn ở trong hai thư mục test `Feature` và `Unit`. Sau khi bạn đã cài đặt một application Laravel mới, bạn hãy chạy `phpunit` trên cửa sổ dòng lệnh để chạy bài test của bạn.

<a name="environment"></a>
## Environment

Khi chạy test thông qua `phpunit`, Laravel sẽ tự động set cấu hình môi trường là `testing` bởi vì các biến môi trường đã được định nghĩa trong file `phpunit.xml`. Laravel cũng tự động cấu hình session và cache là driver `array` trong khi test, điều này nghĩa là không có session hoặc cache nào được duy trì trong khi bạn test.

Bạn có thể tự do định nghĩa các giá trị cấu hình khác cho môi trường test nếu cần thiết. Các biến môi trường `testing` có thể được cấu hình trong file `phpunit.xml`, nhưng hãy đảm bảo là bạn đã xóa cấu hình cache của bạn bằng cách sử dụng lệnh Artisan `config:clear` trước khi chạy bài test của bạn!

Ngoài ra, bạn có thể tạo file `.env.testing` trong thư mục gốc của project của bạn. File này sẽ ghi đè lên file `.env` khi chạy các bài test PHPUnit hoặc chạy các lệnh Artisan với tùy chọn `--env=testing`.

<a name="creating-and-running-tests"></a>
## Tạo và chạy Test

Để tạo một test case mới, hãy sử dụng lệnh Artisan `make:test`:

    // Create a test in the Feature directory...
    php artisan make:test UserTest

    // Create a test in the Unit directory...
    php artisan make:test UserTest --unit

Khi file test đã được tạo xong, bạn có thể định nghĩa các phương thức test như khi sử dụng với PHPUnit. Để chạy test của bạn, hãy chạy lệnh `phpunit` từ terminal của bạn:

    <?php

    namespace Tests\Unit;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $this->assertTrue(true);
        }
    }

> {note} Nếu bạn định nghĩa một phương thức `setUp` / `tearDown` của riêng bạn trong một test class, hãy nhớ gọi các phương thức `parent::setUp()` / `parent::tearDown()` tương ứng ở trong class parent.
