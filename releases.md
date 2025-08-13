# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 10](#laravel-10)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Laravel và các package khác của nó tuân theo [Phiên bản Semantic](https://semver.org). Các phiên bản được phát hành chính thức của framework được phát hành một năm một lần (~Q1), trong khi các bản phát hành nhỏ hơn và các bản sửa lỗi có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ và các bản sửa lỗi sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `^10.0`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

<a name="named-arguments"></a>
#### Named Arguments

[Đặt tên cho tham số](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) không nằm trong nguyên tắc tương thích ngược của Laravel. Chúng tôi có thể đổi tên các tham số bất cứ khi nào để cải thiện codebase của Laravel. Do đó, việc sử dụng các kiểu đặt tên cho tham số khi gọi các phương thức của Laravel nên được thực hiện một cách cẩn trọng và nên hiểu rằng tên tham số có thể thay đổi trong tương lai.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với tất cả các bản phát hành chính thức, các bản sửa lỗi sẽ được cung cấp trong 18 tháng và các bản sửa lỗi bảo mật được cung cấp trong 2 năm. Đối với tất cả các thư viện, bao gồm cả Lumen, chỉ bản phát hành chính thức mới nhất mới nhận được các bản sửa lỗi. Ngoài ra, hãy xem các phiên bản cơ sở dữ liệu [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

<div class="overflow-auto">

| Version | PHP (*) | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- | --- |
| 8 | 7.3 - 8.1 | ngày 8 tháng 9 năm 2020 | ngày 26 tháng 7 năm 2022 | ngày 24 tháng 1 năm 2023 |
| 9 | 8.0 - 8.2 | ngày 8 tháng 2 năm 2022 | ngày 8 tháng 8 năm 2023 | ngày 6 tháng 2 năm 2024 |
| 10 | 8.1 - 8.3 | ngày 14 tháng 2 năm 2023 | ngày 6 tháng 8 năm 2024 | ngày 4 tháng 2 năm 2025 |
| 11 | 8.2 - 8.4 | ngày 12 tháng 3 năm 2024 | ngày 3 tháng 9 năm 2025 | ngày 12 tháng 3 năm 2026 |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>End of life</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>Security fixes only</div>
    </div>
</div>

(*) Supported PHP versions

<a name="laravel-10"></a>
## Laravel 10

Như bạn có thể biết, Laravel đã chuyển sang phát hành theo năm từ bản phát hành Laravel 8. Trước đây, các phiên bản chính được phát hành sau mỗi 6 tháng. Sự chuyển đổi này nhằm mục đích giảm bớt gánh nặng bảo trì cho cộng đồng và thách thức nhóm phát triển của chúng tôi cung cấp các tính năng mới tuyệt vời, mạnh mẽ mà không giới thiệu các thay đổi nghiêm trọng. Do đó, chúng tôi đã cung cấp nhiều tính năng mạnh mẽ cho Laravel 9 mà không phá vỡ khả năng tương thích ngược.

Do đó, cam kết cung cấp những tính năng mới tuyệt vời trong bản phát hành hiện tại nên có thể sẽ khiến các bản phát hành "chính" trong tương lai chủ yếu được sử dụng cho các công việc "bảo trì" như nâng cấp các library, có thể được thấy trong các release note này.

Laravel 10 vẫn sẽ tiếp tục những cải tiến được thực hiện trong Laravel 9.x bằng cách giới thiệu thêm các kiểu tham số và cách trả về của chúng cho tất cả các phương thức có trong ứng dụng, cũng như tất cả các file được sử dụng để tạo các class trong toàn bộ framework. Ngoài ra, một class trừu tượng mới, thân thiện với nhà phát triển cũng đã được giới thiệu để khởi động và tương tác với các process bên ngoài. Hơn nữa, Laravel Pennant cũng đã được giới thiệu để cung cấp một phương pháp tuyệt vời để quản lý các "feature flags" có trong ứng dụng.

<a name="php-8"></a>
### PHP 8.1

Laravel 10.x sẽ yêu cầu phiên bản PHP thấp nhất là 8.1.

<a name="types"></a>
### Types

_Application skeleton và stub type-hint được đóng góp bởi [Nuno Maduro](https://github.com/nunomaduro)_.

Trong lần phát hành đầu tiên, Laravel đã tận dụng tất cả các tính năng gợi ý kiểu dữ liệu có sẵn trong PHP tại thời điểm đó. Tuy nhiên, nhiều tính năng mới đã được bổ sung vào PHP trong những năm gần đây, bao gồm cả các gợi ý kiểu dữ liệu nguyên thủy, kiểu trả về và kiểu mix.

Laravel 10.x đã cập nhật toàn diện bộ framework và tất cả các file mà được framework sử dụng để thêm các tham số và các kiểu dữ liệu trả về vào tất cả các khai báo của phương thức. Ngoài ra, thông tin gợi ý theo kiểu "doc block" do không cần thiết nên đã bị xóa bỏ.

Thay đổi này hoàn toàn tương thích với các ứng dụng hiện có. Do đó, các ứng dụng hiện có mà không có gợi ý theo kiểu này, thì vẫn sẽ tiếp tục hoạt động bình thường.

<a name="laravel-pennant"></a>
### Laravel Pennant

_Laravel Pennant được phát triển bởi [Tim MacDonald](https://github.com/timacdonald)_.

Một package mới của chúng tôi, Laravel Pennant, đã được phát hành. Laravel Pennant cung cấp một giải pháp gọn nhẹ, hợp lý để quản lý các feature flag của ứng dụng. Pennant được tích hợp sẵn các driver `array` dùng bộ nhớ để lưu trữ và driver `database` để lưu trữ feature dài hạn.

Các feature có thể được định nghĩa dễ dàng thông qua phương thức `Feature::define`:

```php
use Laravel\Pennant\Feature;
use Illuminate\Support\Lottery;

Feature::define('new-onboarding-flow', function () {
    return Lottery::odds(1, 10);
});
```

Sau khi một feature đã được định nghĩa xong, bạn có thể dễ dàng xác định xem người dùng hiện tại có quyền truy cập vào feature đó hay không:

```php
if (Feature::active('new-onboarding-flow')) {
    // ...
}
```

Tất nhiên, để thuận tiện hơn, các lệnh Blade cũng có sẵn:

```blade
@feature('new-onboarding-flow')
    <div>
        <!-- ... -->
    </div>
@endfeature
```

Pennant cung cấp nhiều tính năng và API nâng cao. Để biết thêm thông tin, vui lòng tham khảo [tài liệu Pennant](/docs/{{version}}/pennant).

<a name="process"></a>
### Process Interaction

_Layer process abstraction được đóng góp bởi [Nuno Maduro](https://github.com/nunomaduro) và [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel 10.x có giới thiệu một layer trừu tượng đẹp đẽ để bắt đầu và tương tác với các process bên ngoài thông qua một facade `Process` mới:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

Các process thậm chí có thể được bắt đầu trong một pool, cho phép thực hiện và quản lý các process bất đồng bộ một cách hiệu quả:

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->command('cat first.txt');
    $pool->command('cat second.txt');
    $pool->command('cat third.txt');
});

return $first->output();
```

Ngoài ra, các process có thể được fake để thuận tiện cho việc testing:

```php
Process::fake();

// ...

Process::assertRan('ls -la');
```

Để biết thêm thông tin về cách tương tác với các process, vui lòng tham khảo [tài liệu process](/docs/{{version}}/processes).

<a name="test-profiling"></a>
### Test Profiling

_Test profiling được đóng góp bởi [Nuno Maduro](https://github.com/nunomaduro)_.

Lệnh `test` của Artisan đã nhận được một tùy chọn `--profile` mới cho phép bạn dễ dàng xác định các bài kiểm tra chạy chậm nhất trong ứng dụng của bạn:

```shell
php artisan test --profile
```

Để thuận tiện, các bài kiểm tra chậm nhất sẽ được hiển thị trực tiếp trong output của CLI:

<p align="center">
    <img width="100%" src="https://user-images.githubusercontent.com/5457236/217328439-d8d983ec-d0fc-4cde-93d9-ae5bccf5df14.png"/>
</p>

<a name="pest-scaffolding"></a>
### Pest Scaffolding

Các dự án Laravel mới có thể tạo cùng với các bài test Pest mặc định. Để chọn tính năng này, hãy cung cấp flag `--pest` khi tạo dự án mới thông qua Laravel installer:

```shell
laravel new example-application --pest
```

<a name="generator-cli-prompts"></a>
### Generator CLI Prompts

_Generator CLI prompt được đóng góp bởi [Jess Archer](https://github.com/jessarcher)_.

Để cải thiện trải nghiệm cho nhà phát triển trong framework, tất cả các lệnh `make` được tích hợp sẵn của Laravel sẽ không còn yêu cầu bất kỳ dữ liệu input nào cả. Nếu các lệnh được gọi mà không có dữ liệu input, thì bạn sẽ được nhắc yêu cầu nhập cho các tham số cần thiết:

```shell
php artisan make:controller
```

<a name="horizon-telescope-facelift"></a>
### Horizon / Telescope Facelift

[Horizon](/docs/{{version}}/horizon) và [Telescope](/docs/{{version}}/telescope) đã được cập nhật với giao diện mới và hiện đại hơn có kiểu chữ mới, khoảng cách và thiết kế được cải thiện:

<img src="https://laravel.com/img/docs/horizon-example.png">
