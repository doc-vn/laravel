# Upgrade Guide

- [Nâng cấp đến 7.0 từ 6.0](#upgrade-7.0)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Authentication Scaffolding](#authentication-scaffolding)
- [Date Serialization](#date-serialization)
- [Symfony 5 Related Upgrades](#symfony-5-related-upgrades)

</div>

<a name="medium-impact-changes"></a>
## Những thay đổi có tác động trung bình

<div class="content-list" markdown="1">

- [Blade Components & "Blade X"](#blade-components-and-blade-x)
- [CORS Support](#cors-support)
- [Factory Types](#factory-types)
- [Markdown Mail Template Updates](#markdown-mail-template-updates)
- [The `Blade::component` Method](#the-blade-component-method)
- [The `assertSee` Assertion](#assert-see)
- [The `different` Validation Rule](#the-different-rule)
- [Unique Route Names](#unique-route-names)

</div>

<a name="upgrade-7.0"></a>
## Nâng cấp đến 7.0 từ 6.0

#### Estimated Upgrade Time: 15 Minutes

> {note} Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này mới có thể thực sự ảnh hưởng đến application của bạn.

### Symfony 5 Required

**Likelihood Of Impact: High**

Laravel 7 đã nâng cấp các component Symfony cơ bản của nó lên phiên bản 5.x, đây cũng là phiên bản tương thích tối thiểu.

### PHP 7.2.5 Required

**Likelihood Of Impact: Low**

Phiên bản PHP tối thiểu bây giờ là 7.2.5.

<a name="updating-dependencies"></a>
### Updating Dependencies

Cập nhật các library sau trong file `composer.json` của bạn:

- `laravel/framework` to `^7.0`
- `nunomaduro/collision` to `^4.1`
- `phpunit/phpunit` to `^8.5`
- `laravel/tinker` to `^2.0`
- `facade/ignition` to `^2.0`

Các package sau đây có các bản phát hành mới để hỗ trợ Laravel 7. Nếu bạn có dùng các package này, hãy đọc qua các hướng dẫn nâng cấp của chúng trước khi nâng cấp Laravel 7:

- [Browser Kit Testing v6.0](https://github.com/laravel/browser-kit-testing/blob/master/UPGRADE.md)
- [Envoy v2.0](https://github.com/laravel/envoy/blob/master/UPGRADE.md)
- [Horizon v4.0](https://github.com/laravel/horizon/blob/master/UPGRADE.md)
- [Nova v3.0](https://nova.laravel.com/releases)
- [Passport v9.0](https://github.com/laravel/passport/blob/master/UPGRADE.md)
- [Scout v8.0](https://github.com/laravel/scout/blob/master/UPGRADE.md)
- [Telescope v3.0](https://github.com/laravel/telescope/releases)
- [Tinker v2.0](https://github.com/laravel/tinker/blob/2.x/CHANGELOG.md)
- UI v2.0 (No changes necessary)

Cuối cùng, kiểm tra bất kỳ package bên thứ 3 nào mà được ứng dụng của bạn sử dụng và xác nhận rằng bạn đang sử dụng phiên bản phù hợp của package để hỗ trợ Laravel 7.

<a name="symfony-5-related-upgrades"></a>
### Symfony 5 Related Upgrades

**Likelihood Of Impact: High**

Laravel 7 sử dụng phiên bản 5.x của các component Symfony. Có một số thay đổi nhỏ đối với ứng dụng của bạn là bắt buộc để phù hợp với phiên bản nâng cấp này.

Đầu tiên, các phương thức `report`, `render`, `shouldReport`, và `renderForConsole` của class `App\Exceptions\Handler` trong ứng dụng của bạn sẽ phải chấp nhận các instance của interface `Throwable` thay vì các instance `Exception`:

    use Throwable;

    public function report(Throwable $exception);
    public function shouldReport(Throwable $exception);
    public function render($request, Throwable $exception);
    public function renderForConsole($output, Throwable $exception);

Ngoài ra, mọi phương thức `failed` được implement trên các queued job sẽ chấp nhận các instance của `Throwable` thay vì các instance của `Exception`.

Tiếp theo, hãy cập nhật tùy chọn `secure` trong file cấu hình `session` của bạn để có giá trị fallback là `null`:

    'secure' => env('SESSION_SECURE_COOKIE', null),

Symfony Console, là component cơ bản cung cấp sức mạnh cho Artisan, chấp nhận tất cả các lệnh trả về một integer. Do đó, bạn nên đảm bảo rằng bất kỳ lệnh nào của bạn sẽ trả về một giá trị integer:

    public function handle()
    {
        // Before...
        return true;

        // After...
        return 0;
    }

### Authentication

<a name="authentication-scaffolding"></a>
#### Scaffolding

**Likelihood Of Impact: High**

Tất cả authentication scaffolding đã được chuyển sang repository `laravel/ui`. Nếu bạn đang sử dụng authentication scaffolding của Laravel, bạn nên cài đặt bản phát hành `^2.0` của package này và package phải này được cài đặt vào trong tất cả các môi trường. Nếu trước đó bạn đã thêm package này vào trong phần `require-dev` của file `composer.json` trong ứng dụng của bạn, bạn nên chuyển nó sang phần `require`:

    composer require laravel/ui "^2.0"

#### The `TokenRepositoryInterface`

**Likelihood Of Impact: Low**

Phương thức `recentlyCreatedToken` đã được thêm vào interface `Illuminate\Auth\Passwords\TokenRepositoryInterface`. Nếu bạn đang viết một implement tùy chỉnh cho interface này, bạn nên thêm phương thức này vào implement của bạn.

### Blade

<a name="the-blade-component-method"></a>
#### The `component` Method

**Likelihood Of Impact: Medium**

Phương thức `Blade::component` đã được đổi tên thành `Blade::aliasComponent`. Vui lòng cập nhật các phương thức của bạn gọi sang phương thức này cho phù hợp.

<a name="blade-components-and-blade-x"></a>
#### Blade Components & "Blade X"

**Likelihood Of Impact: Medium**

Laravel 7 đã hỗ trợ các "tag components" của Blade. Nếu bạn muốn tắt chức năng tag component tích sẵn của Blade, bạn có thể gọi phương thức `withoutComponentTags` từ phương thức `boot` của `AppServiceProvider` của bạn:

    use Illuminate\Support\Facades\Blade;

    Blade::withoutComponentTags();

### Eloquent

#### The `addHidden` / `addVisible` Methods

**Likelihood Of Impact: Low**

Phương thức `addHidden` và `addVisible` không được ghi chép nên đã bị xóa. Thay vào đó, hãy sử dụng các phương thức `makeHidden` và `makeVisible`.

#### The `booting` / `booted` Methods

**Likelihood Of Impact: Low**

Các phương thức `booting` và `booted` đã được thêm vào Eloquent để cung cấp một nơi định nghĩa các logic sẽ được thực thi trong quá trình "khởi động" của model. Nếu bạn đã có các phương thức trong model với những tên này, bạn sẽ cần phải đổi tên các phương thức của mình để chúng không xung đột với các phương thức mới được thêm vào.

<a name="date-serialization"></a>
#### Date Serialization

**Likelihood Of Impact: High**

Laravel 7 sử dụng định dạng chuyển đổi kiểu ngày mới khi sử dụng phương thức `toArray` hoặc `toJson` trên các model Eloquent. Để định dạng ngày khi chuyển đổi, framework sẽ sử dụng phương thức `toJSON` của Carbon, để tạo ra ngày tương thích với ISO-8601 bao gồm cả thông tin múi giờ và phần phân số sau số giây. Ngoài ra, thay đổi này cũng cung cấp hỗ trợ và tích hợp tốt hơn với các thư viện phân tích cú pháp ngày tháng năm của phía client.

Trước đây, các ngày sẽ được chuyển hóa thành định dạng như sau: `2019-12-02 20:01:00`. Các ngày được chuyển hóa bằng định dạng mới sẽ xuất hiện như: `2019-12-02T20:01:00.283041Z`. Xin lưu ý rằng ngày ISO-8601 luôn được biểu thị bằng UTC.

Nếu bạn muốn tiếp tục sử dụng hành vi của phiên bản trước đó, bạn có thể ghi đè phương thức `serializeDate` trên model của bạn:

    use DateTimeInterface;

    /**
     * Prepare a date for array / JSON serialization.
     *
     * @param  \DateTimeInterface  $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date)
    {
        return $date->format('Y-m-d H:i:s');
    }

> {tip} Thay đổi này chỉ ảnh hưởng đến việc chuyển hóa các model hoặc các collection của model thành mảng hoặc JSON. Thay đổi này không ảnh hưởng đến cách lưu trữ dữ liệu ngày tháng trong cơ sở dữ liệu của bạn.

<a name="factory-types"></a>
#### Factory Types

**Likelihood Of Impact: Medium**

Laravel 7 đã loại bỏ tính năng "factory types". Tính năng này đã không còn được cung cấp tài liệu kể từ tháng 10 năm 2016. Nếu bạn vẫn đang sử dụng tính năng này, bạn nên nâng cấp lên [factory states](/docs/{{version}}/database-testing#factory-states) để mang lại tính linh hoạt cao hơn.

#### The `getOriginal` Method

**Likelihood Of Impact: Low**

Phương thức `$model->getOriginal()` bây giờ sẽ lấy dữ liệu cùng với bất kỳ kiểu cast hoặc kiểu mutator nào được định nghĩa trên model. Trước đây, phương thức này trả về các thuộc tính uncast hoặc raw. Nếu bạn muốn tiếp tục lấy ra các giá trị raw, hoặc uncast, bạn có thể sử dụng phương thức `getRawOriginal` để thay thế.

#### Route Binding

**Likelihood Of Impact: Low**

Phương thức `ResolutionRouteBinding` của interface `Illuminate\Contracts\Routing\UrlRoutable` bây giờ sẽ chấp nhận tham số `$field`. Nếu bạn đang implement interface này bằng tay, thì bạn nên cập nhật phần implement của bạn.

Ngoài ra, phương thức `resolveRouteBinding` của class `Illuminate\Database\Eloquent\Model` bây giờ cũng chấp nhận tham số `$field`. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật phương thức của bạn để chấp nhận tham số này.

Cuối cùng, phương thức `resolveRouteBinding` của trait `Illuminate\Http\Resources\DelegatesToResources` bây giờ cũng chấp nhận tham số `$field`. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật phương thức của bạn để chấp nhận tham số này.

### HTTP

#### PSR-7 Compatibility

**Likelihood Of Impact: Low**

Thư viện Zend Diactoros để tạo các response PSR-7 đã không được dùng nữa. Nếu bạn đang sử dụng package này để tương thích với chuẩn PSR-7, hãy cài đặt package Composer `nyholm/psr7` để thay thế. Ngoài ra, hãy cài đặt bản phát hành `^2.0` của package Composer `symfony/psr-http-message-bridge`.

### Mail

#### Configuration File Changes

**Likelihood Of Impact: Optional**

Để hỗ trợ multiple mailer, file cấu hình `mail` mặc định đã được thay đổi trong Laravel 7.x để chứa một mảng các `mailers`. Tuy nhiên, để duy trì khả năng tương thích với phiên bản cũ, định dạng Laravel 6.x của file cấu hình này vẫn được hỗ trợ. Vì vậy, không cần thay đổi **nào** khi nâng cấp lên Laravel 7.x; tuy nhiên, bạn có thể muốn [kiểm tra file cấu hình `mail` mới](https://github.com/laravel/laravel/blob/{{version}}/config/mail.php) cấu trúc và cập nhật file của bạn để phản ánh những thay đổi.

Ngoài ra, biến môi trường `MAIL_DRIVER` đã được đổi tên thành `MAIL_MAILER`.

<a name="markdown-mail-template-updates"></a>
#### Markdown Mail Template Updates

**Likelihood Of Impact: Medium**

Các template Markdown mail mặc định đã được làm mới với thiết kế chuyên nghiệp và hấp dẫn hơn. Ngoài ra, component Markdown mail `promotion` mà không được ghi chép cũng đã bị xóa.

Bởi vì thụt lề có ý nghĩa đặc biệt quan trọng trong Markdown, nên các template Markdown mail chấp nhận HTML không có dấu. Nếu trước đó bạn đã export các template mail mặc định của Laravel, thì bạn sẽ cần export lại các template mail của bạn hoặc hủy liên kết của chúng:

    php artisan vendor:publish --tag=laravel-mail --force

#### Swift Mailer Bindings

**Likelihood Of Impact: Low**

Laravel 7.x sẽ không cung cấp các container binding `swift.mailer` và `swift.transport`. Bây giờ bạn có thể truy cập các đối tượng này thông qua binding `mailer`:

    $swiftMailer = app('mailer')->getSwiftMailer();

    $swiftTransport = $swiftMailer->getTransport();

### Resources

#### The `Illuminate\Http\Resources\Json\Resource` Class

**Likelihood Of Impact: Low**

Class `Illuminate\Http\Resources\Json\Resource` không còn được sử dụng nên đã bị xóa. Thay vào đó, resource của bạn nên extend từ class `Illuminate\Http\Resources\Json\JsonResource` thay thế.

### Routing

#### The Router `getRoutes` Method

**Likelihood Of Impact: Low**

Phương thức `getRoutes` của router bây giờ sẽ trả về một instance của `Illuminate\Routing\RouteCollectionInterface` thay vì `Illuminate\Routing\RouteCollection`.

<a name="unique-route-names"></a>
#### Unique Route Names

**Likelihood Of Impact: Medium**

Mặc dù chưa bao giờ được ghi lại chính thức, các bản phát hành Laravel trước đây cho phép bạn định nghĩa hai route khác nhau có cùng tên. Trong Laravel 7, điều này sẽ không còn khả thi nữa và bạn phải luôn cung cấp các tên riêng cho các route của bạn. Các route mà có tên trùng nhau có thể gây ra các hành vi không mong muốn trong nhiều chỗ của framework.

<a name="cors-support"></a>
#### CORS Support

**Likelihood Of Impact: Medium**

Cross-Origin Resource Sharing (CORS) bây giờ sẽ được hỗ trợ và được tích hợp mặc định. Nếu bạn đang sử dụng bất kỳ thư viện CORS nào của bên thứ ba, bạn nên sử dụng [file cấu hình `cors` mới](https://github.com/laravel/laravel/blob/master/config/cors.php).

Tiếp theo, cài đặt thư viện CORS ở bên dưới vào ứng dụng của bạn:

    composer require fruitcake/laravel-cors

Cuối cùng, thêm middleware `\Fruitcake\Cors\HandleCors::class` vào danh sách global middleware `App\Http\Kernel` của bạn.

### Session

#### The `array` Session Driver

**Likelihood Of Impact: Low**

Dữ liệu session driver `array` bây giờ sẽ tồn tại cho các request hiện tại. Trước đây, dữ liệu này được lưu trong session `array` và không thể được truy xuất được ngay cả trong request hiện tại.

### Testing

<a name="assert-see"></a>
#### The `assertSee` Assertion

**Likelihood Of Impact: Medium**

Các kiểm tra `assertSee`, `assertDontSee`, `assertSeeText`, `assertDontSeeText`, `assertSeeInOrder` và `assertSeeTextInOrder` trên class `TestResponse` bây giờ sẽ tự động thoát. Nếu bạn đang thoát theo cách thủ công bằng một giá trị nào đó được truyền cho các kiểm tra này, thì bạn không nên làm như vậy nữa. Nếu bạn cần kiểm tra cho các giá trị mà không thoát, thì bạn có thể truyền giá trị `false` làm tham số thứ hai cho phương thức.

<a name="test-response"></a>
#### The `TestResponse` Class

**Likelihood Of Impact: Low**

Class `Illuminate\Foundation\Testing\TestResponse` đã được đổi tên thành `Illuminate\Testing\TestResponse`. Nếu bạn đang extend class này, hãy đảm bảo là đã cập nhật namespace.

<a name="assert-class"></a>
#### The `Assert` Class

**Likelihood Of Impact: Low**

Class `Illuminate\Foundation\Testing\Assert` đã được đổi tên thành `Illuminate\Testing\Assert`. Nếu bạn đang extend class này, hãy đảm bảo là đã cập nhật namespace.

### Validation

<a name="the-different-rule"></a>
#### The `different` Rule

**Likelihood Of Impact: Medium**

Quy tắc `different` bây giờ sẽ thất bại nếu một trong các tham số được chỉ định bị thiếu trong request.

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [Công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/6.x...7.x) và chọn bản cập nhật nào quan trọng với bạn.
