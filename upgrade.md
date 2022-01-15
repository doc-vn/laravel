# Upgrade Guide

- [Upgrade đến 5.6.30 từ 5.6](#upgrade-5.6.30)
- [Upgrade đến 5.6.0 từ 5.5](#upgrade-5.6.0)

<a name="upgrade-5.6.30"></a>
## Upgrade đến 5.6.30 từ 5.6 (Bản phát hành bảo mật)

Laravel 5.6.30 là một bản phát hành bảo mật riêng của Laravel và được khuyến cáo là phải nâng cấp ngay cho tất cả mọi người dùng. Laravel 5.6.30 cũng chứa một thay đổi nghiêm trọng cho việc mã hóa và serialization của cookie, vì vậy hãy đọc kỹ các lưu ý sau khi nâng cấp application của bạn.

**Lỗ hổng này chỉ có thể bị khai thác nếu khóa mã hóa của application của bạn (ở đây là biến môi trường APP_KEY) đã bị người khác truy cập được.** Thông thường, người dùng application của bạn không thể truy cập vào giá trị này. Tuy nhiên, nếu nhân viên cũ của bạn có thể truy cập vào khóa mã hóa này thì họ có thể sử dụng khóa này để tấn công lại vào application của bạn. Nếu bạn có một lý do nào đó để tin rằng khóa mã hóa này của bạn đang nằm trong tay của một nhận viên cũ hoặc một kẻ phá hoại, thì bạn phải **tạo** lại giá trị của khóa đó.

### Cookie Serialization

Laravel 5.6.30 sẽ vô hiệu hóa tất cả các serialization / unserialization của các giá trị cookie. Vì tất cả các cookie của Laravel đều đã được mã hóa và được ký, nên các giá trị cookie thường được coi là an toàn cho việc phân biệt các hành động giả mạo khách hàng. **Tuy nhiên, nếu khóa mã hóa của application của bạn nằm trong tay của một kẻ phá hoại, thì người đó có thể tạo các giá trị cookie bằng cách sử dụng khóa mã hóa đó và khai thác các lỗ hổng thừa kế để serialization / unserialization các đối tượng PHP, như là gọi các phương thức class tùy ý trong application của bạn.**

Vô hiệu hóa serialization trên tất cả các giá trị cookie sẽ làm mất hiệu lực tất cả các session của bạn và người dùng sẽ cần phải đăng nhập lại vào application (trừ khi họ có set `remember_token`, trong trường hợp đó, người dùng sẽ tự động đăng nhập vào một session mới). Ngoài ra, mọi cookie mã hóa khác mà application của bạn đang cài đặt sẽ trở lên không hợp lệ. Vì lý do này, bạn có thể muốn thêm logic vào application của bạn để validate các giá trị cookie này của bạn giống với danh sách các giá trị mà bạn mong muốn; nếu không, bạn nên xoá chúng.

#### Configuring Cookie Serialization

Vì lỗ hổng này không thể bị khai thác nếu kẻ tấn công không thể truy cập vào khóa mã hóa của bạn, nên chúng tôi đã cung cấp một cách để bạn có thể bật lại serialization cookie mã hóa trong lúc bạn làm cho application của bạn tương thích với những thay đổi này. Để bật / tắt serialization cookie, bạn có thể thay đổi thuộc tính static `serialize` của [middleware](https://github.com/laravel/laravel/blob/5.6/app/Http/Middleware/EncryptCookies.php) `App\Http\Middleware\EncryptCookies`:

    /**
     * Indicates if cookies should be serialized.
     *
     * @var bool
     */
    protected static $serialize = true;

> **Lưu ý:** Khi bật serialization cookie mã hóa, application của bạn có thể bị tấn công nếu khóa mã hóa của application bị truy cập bởi kẻ tấn công. Nếu bạn tin rằng khóa của bạn có thể đang nằm trong tay của kẻ tấn công, bạn nên tạo một khóa mới trước khi bật serialization cookie mã hóa.

### Dusk 4.0.0

Dusk 4.0.0 đã được phát hành và không serialize cookie. Nếu bạn chọn bật serialize cookie, bạn nên tiếp tục sử dụng Dusk 3.0.0. Nếu không, bạn nên nâng cấp lên bản Dusk 4.0.0.

### Passport 6.0.7

Passport 6.0.7 đã được phát hành với một phương thức mới `Laravel\Passport\Passport::withoutCookieSerialization()`. Khi bạn đã tắt serialization cookie, bạn nên gọi phương thức này trong `AppServiceProvider` của ứng dụng.

<a name="upgrade-5.6.0"></a>
## Upgrade đến 5.6.0 từ 5.5

#### Estimated Upgrade Time: 10 - 30 Minutes

> {note} Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này có thể thực sự ảnh hưởng đến application của bạn.

### PHP

Laravel 5.6 yêu cầu PHP 7.1.3 trở lên.

### Updating Dependencies

Cập nhật library `laravel/framework` của bạn thành `5.6.*` và library `fideloper/proxy` của bạn thành `^4.0` trong file `composer.json` của bạn.

Nếu bạn đang sử dụng package `laravel/browser-kit-testing`, bạn nên cập nhật package này lên thành `4.*` trong file composer.json của bạn.

Ngoài ra, nếu bạn đang sử dụng các package sau đây, thì bạn nên nâng cấp chúng lên bản phát hành mới nhất:

<div class="content-list" markdown="1">
- Dusk (Cập nhật tới `^3.0`)
- Passport (Cập nhật tới `^6.0`)
- Scout (Cập nhật tới `^4.0`)
</div>

Tất nhiên, đừng quên kiểm tra bất kỳ package của bên thứ 3 nào được ứng dụng của bạn sử dụng và xác nhận rằng bạn đang sử dụng phiên bản thích hợp để hỗ trợ Laravel 5.6.

#### Symfony 4

Tất cả các component của Symfony mà được Laravel sử dụng đều đã được nâng cấp lên phiên bản Symfony `~4.0`. Nếu bạn đang sử dụng trực tiếp các component Symfony trong ứng dụng của bạn, thì bạn nên xem lại [những thay đổi Symfony](https://github.com/symfony/symfony/blob/4.0/UPGRADE-4.0.md).

#### PHPUnit

Bạn nên cập nhật library `phpunit/phpunit` của ứng dụng lên `^7.0`.

### Arrays

#### The `Arr::wrap` Method

Truyền `null` vào phương thức `Arr::wrap` bây giờ sẽ trả về một mảng trống.

### Artisan

#### The `optimize` Command

Lệnh Artisan `optimize` không được dùng ở các phiên bản trước đã bị xóa. Với những nâng cấp gần đây đối với PHP, bao gồm cả OPcache, thì lệnh `optimize` không còn cung cấp bất kỳ lợi ích hiệu suất nào nữa. Do đó, bạn có thể xóa `php artisan optimize` ra khỏi `scripts` trong file `composer.json` của bạn.

### Blade

#### HTML Entity Encoding

Trong các phiên bản trước của Laravel, Blade (và helper `e`) sẽ không mã hóa kép các thực thể HTML. Đây không phải là hành vi mặc định của hàm `htmlspecialchars` của PHP và có thể dẫn đến các hành vi không mong muốn khi hiển thị các nội dung hoặc chuyển nội dung đó sang JSON để truyền vào các framework JavaScript.

Trong Laravel 5.6, mặc định Blade và helper `e` sẽ mã hóa kép các ký tự đặc biệt. Điều này làm cho các tính năng này phù hợp hơn với các hành vi mặc định của hàm `htmlspecialchars` của PHP. Nếu bạn muốn tiếp tục hành vi trước đây, bạn có thể sử dụng phương thức `Blade::withoutDoubleEncoding`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
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
            Blade::withoutDoubleEncoding();
        }
    }

### Cache

#### The Rate Limiter `tooManyAttempts` Method

Tham số `$decayMinutes` do được không sử dụng nên đã bị xóa khỏi phương thức `tooManyAttempts`. Nếu bạn đang ghi đè phương thức này bằng một implementation của riêng bạn, bạn cũng nên xóa tham số đó ra khỏi phương thức của bạn.

### Database

#### Index Order Of Morph Columns

Việc lập index cho các cột được built bằng phương thức migration `morphs` đã được đảo ngược để có hiệu suất tốt hơn. Nếu bạn đang sử dụng phương thức `morphs` trong các file migration của bạn, bạn có thể gặp lỗi khi chạy phương thức `down` trong quá trình migration. Nếu ứng dụng của bạn vẫn đang được phát triển, thì bạn có thể sử dụng lệnh `migrate:fresh` để built lại cơ sở dữ liệu từ đầu. Nếu ứng dụng của bạn đang được deploy, thì bạn nên truyền một tên index cho phương thức `morphs`.

#### `MigrationRepositoryInterface` Method Addition

Một phương thức `getMigrationsBatches` mới đã được thêm vào `MigrationRepositoryInterface`. Nếu bạn đang định nghĩa implementation của class này, thì bạn nên thêm phương thức này vào trong quá trình implementation của bạn. Bạn có thể xem phần implementation trong framework làm ví dụ.

### Eloquent

#### The `getDateFormat` Method

Phương thức `getDateFormat`, bây giờ sẽ là `public` thay vì `protected`.

### Hashing

#### New Configuration File

Tất cả các cấu hình của hàm hash sẽ được lưu trong file cấu hình `config/hashing.php` của chính nó. Bạn nên lưu một bản sao của [file cấu hình mặc định này](https://github.com/laravel/laravel/blob/5.6/config/hashing.php) trong ứng dụng của bạn. Rất có thể, bạn nên tiếp tục sử dụng driver `bcrypt` làm driver mặc định của bạn. Tuy nhiên, `argon` cũng được hỗ trợ.

### Helpers

#### The `e` Helper

Đây không phải là hành vi mặc định của hàm `htmlspecialchars` của PHP và có thể dẫn đến các hành vi không mong muốn khi hiển thị các nội dung hoặc chuyển nội dung đó sang JSON để truyền vào các framework JavaScript.

Trong Laravel 5.6, mặc định Blade và helper `e` sẽ mã hóa kép các ký tự đặc biệt. Điều này làm cho các tính năng này phù hợp hơn với các hành vi mặc định của hàm `htmlspecialchars` của PHP. Nếu bạn muốn tiếp tục hành vi trước đây, bạn có thể truyền `false` vào làm tham số thứ hai cho helper `e`:

    <?php echo e($string, false); ?>

### Logging

#### New Configuration File

Tất cả cấu hình ghi log hiện đang được lưu trong file cấu hình `config/logging.php`. Bạn nên lưu một bản sao của [file cấu hình mặc định này](https://github.com/laravel/laravel/blob/5.6/config/logging.php) vào trong ứng dụng của bạn và chỉnh sửa nó dựa theo nhu cầu của ứng dụng của bạn.

Các tùy chọn cấu hình `log` và `log_level` có thể bị xóa ra khỏi file cấu hình `config/app.php`.

#### The `configureMonologUsing` Method

Nếu bạn đang sử dụng phương thức `configureMonologUsing` để tùy chỉnh instance Monolog cho ứng dụng của bạn, thì bây giờ bạn nên tạo một channel Log `custom`. Để biết thêm thông tin về cách tạo một channel, hãy xem [tài liệu ghi log đầy đủ](/docs/5.6/logging#creating-custom-channels).

#### The Log `Writer` Class

Class `Illuminate\Log\Writer` đã được đổi tên thành `Illuminate\Log\Logger`. Nếu bạn đang khai báo class này là class phụ thuộc của một trong những class trong ứng dụng của bạn, thì bạn nên cập nhật lại tham chiếu của class đó thành tên mới. Hoặc, nếu có thể, thì bạn nên cân nhắc khai báo interface `Psr\Log\LoggerInterface`.

#### The `Illuminate\Contracts\Logging\Log` Interface

Interface này đã bị xóa vì interface này đã trùng lặp với interface `Psr\Log\LoggerInterface`. Thay vào đó, bạn nên khai báo interface `Psr\Log\LoggerInterface`.

### Mail

#### `withSwiftMessage` Callbacks

Trong các bản phát hành trước của Laravel, các callback tùy chỉnh của Swift Messages được đăng ký bằng cách sử dụng `withSwiftMessage` được gọi _sau khi_ nội dung đã được mã hóa và được thêm vào message. Còn bây giờ các callback sẽ được gọi là _trước khi_ khi nội dung được thêm vào, cho phép bạn tùy chỉnh mã hóa hoặc các tùy chọn message khác nếu cần.

### Pagination

#### Bootstrap 4

Các link phân trang được tạo bởi paginator, hiện tại mặc định là Bootstrap 4. Để hướng dẫn paginator tạo link cho Bootstrap 3, hãy gọi phương thức `Paginator::useBootstrapThree` trong phương thức `boot` của `AppServiceProvider` của bạn:

    <?php

    namespace App\Providers;

    use Illuminate\Pagination\Paginator;
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
            Paginator::useBootstrapThree();
        }
    }

### Resources

#### The `original` Property

Thuộc tính `original` của [resource responses](/docs/5.6/eloquent-resources) hiện tại đang được set thành original của model thay vì một chuỗi JSON hoặc một mảng . Điều này cho phép bạn kiểm tra model của response dễ dàng hơn trong quá trình test.

### Routing

#### Returning Newly Created Models

Khi trả về một model Eloquent mới được tạo trực tiếp từ một route, response status bây giờ sẽ tự động được set thành `201` thay vì `200`. Nếu bất kỳ bài test nào trong ứng dụng của bạn yêu cầu response là `200`, thì những bài test đó nên được cập nhật thành yêu cầu là `201`.

### Trusted Proxies

Do những thay đổi cơ bản trong chức năng trusted proxy của Symfony HttpFoundation, mà bạn phải thực hiện các thay đổi nhỏ đối với middleware `App\Http\Middleware\TrustProxies` của ứng dụng.

Thuộc tính `$headers`, trước đây là một mảng, bây giờ là một thuộc tính bit chấp nhận một số giá trị khác nhau. Ví dụ: để trust tất cả các header được forward, bạn có thể cập nhật thuộc tính `$headers` của bạn thành giá trị sau:

    use Illuminate\Http\Request;

    /**
     * The headers that should be used to detect proxies.
     *
     * @var int
     */
    protected $headers = Request::HEADER_X_FORWARDED_ALL;

Để biết thêm thông tin về các giá trị `$headers` có sẵn, hãy xem toàn bộ tài liệu về [trust proxy](/docs/5.6/requests#configuring-trusted-proxies).

### Validation

#### The `ValidatesWhenResolved` Interface

Phương thức `validate` của interface hoặc trait `ValidatesWhenResolved` đã được đổi tên thành `validateResolved` để tránh xung đột với phương thức `$request->validate()`.

### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [Công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/5.5...5.6) và chọn bản cập nhật nào quan trọng với bạn.
