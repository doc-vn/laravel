# Testing: Getting Started

- [Giới thiệu](#introduction)
- [Environment](#environment)
- [Tạo và chạy Test](#creating-and-running-tests)

<a name="introduction"></a>
## Giới thiệu

Laravel được xây cùng với việc test. Thực tế là, Laravel đã mặc định hỗ trợ việc test với PHPUnit và một file `phpunit.xml` đã được set up sẵn vào trong application của bạn. Framework này cũng có các phương thức trợ giúp thuận tiện cho phép bạn test các application của bạn.

Mặc định, thư mục `tests` của application của bạn chứa hai thư mục: `Feature` và `Unit`. Các test unit là các bài test tập trung vào một phần rất nhỏ, tách biệt trong code của bạn. Thực tế là, hầu hết các bài test unit có thể tập trung vào một phương thức duy nhất. Chức năng test cũng có thể test một phần lớn hơn của code của bạn, chứa cả cách mà một số đối tượng tương tác với nhau hoặc thậm chí là một HTTP request đầy đủ gửi tới một JSON endpoint.

Một file `exampleTest.php` được cung cấp sẵn trong cả hai thư mục test `Feature` và `Unit`. Sau khi cài đặt một application Laravel mới, hãy chạy `phpunit` trên cửa sổ dòng lệnh để chạy test của bạn.

<a name="environment"></a>
## Environment

Khi chạy test thông qua `phpunit`, Laravel sẽ tự động set cấu hình môi trường là `testing` bởi vì các biến môi trường đã được định nghĩa trong file `phpunit.xml`. Laravel cũng tự động cấu hình session và cache cho driver `array` trong khi test, điều này nghĩa là không có session hoặc cache nào được duy trì trong khi test.

Bạn có thể tự do định nghĩa các giá trị cấu hình khác cho môi trường test nếu cần thiết. Các biến môi trường `testing` có thể được cấu hình trong file `phpunit.xml`, nhưng hãy đảm bảo là bạn đã xóa cấu hình cache của bạn bằng cách sử dụng lệnh Artisan `config:clear` trước khi chạy test của bạn!

<a name="creating-and-running-tests"></a>
## Tạo và chạy Test

Để tạo một test case mới, hãy sử dụng lệnh Artisan `make:test`:

    // Create a test in the Feature directory...
    php artisan make:test UserTest

    // Create a test in the Unit directory...
    php artisan make:test UserTest --unit

Khi file test đã được tạo, bạn có thể định nghĩa các phương thức test như bình thường khi sử dụng PHPUnit. Để chạy test của bạn, hãy chạy lệnh `phpunit` từ terminal của bạn:

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

> {note} Nếu bạn định nghĩa phương thức `setUp` của riêng bạn trong một test class, hãy nhớ gọi `parent::setUp()`.
