# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 13](#laravel-13)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Laravel và các package khác của nó tuân theo [Phiên bản Semantic](https://semver.org). Các phiên bản được phát hành chính thức của framework được phát hành một năm một lần (~Q1), trong khi các bản phát hành nhỏ hơn và các bản sửa lỗi có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ và các bản sửa lỗi sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `^13.0`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

<a name="named-arguments"></a>
#### Named Arguments

[Đặt tên cho tham số](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) không nằm trong nguyên tắc tương thích ngược của Laravel. Chúng tôi có thể đổi tên các tham số bất cứ khi nào để cải thiện codebase của Laravel. Do đó, việc sử dụng các kiểu đặt tên cho tham số khi gọi các phương thức của Laravel nên được thực hiện một cách cẩn trọng và nên hiểu rằng tên tham số có thể thay đổi trong tương lai.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với tất cả các bản phát hành chính thức, các bản sửa lỗi sẽ được cung cấp trong 18 tháng và các bản sửa lỗi bảo mật được cung cấp trong 2 năm. Đối với tất cả các thư viện, chỉ bản phát hành chính thức mới nhất mới nhận được các bản sửa lỗi. Ngoài ra, hãy xem các phiên bản cơ sở dữ liệu [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

<div class="overflow-auto">

| Version | PHP (*)   | Release                  | Bug Fixes Until          | Security Fixes Until       |
| ------- |-----------| -----------------------  | ------------------------ | -------------------------- |
| 10      | 8.1 - 8.3 | ngày 14 tháng 2 năm 2023 | ngày 6 tháng 8 năm 2024  | ngày 4 tháng 2 năm 2025    |
| 11      | 8.2 - 8.4 | ngày 12 tháng 3 năm 2024 | ngày 3 tháng 9 năm 2025  | ngày 12 tháng 3 năm 2026   |
| 12      | 8.2 - 8.5 | ngày 24 tháng 2 năm 2025 | ngày 13 tháng 8 năm 2026 | ngày 24 tháng 2 năm 2027   |
| 13      | 8.3 - 8.5 | Ngày 17 tháng 3 năm 2026 | Quý 3 năm 2027           | Ngày 17 tháng 3 năm 2028   |

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

<a name="laravel-13"></a>
## Laravel 13

Laravel 13 tiếp tục chu kỳ phát hành hàng năm của Laravel với sự tập trung vào các luồng công việc AI-native, các giá trị mặc định mạnh mẽ hơn và các API dành cho nhà phát triển thuận tiện hơn. Bản phát hành này chứa các chức năng AI chính thức, các resource JSON:API, khả năng tìm kiếm ngữ nghĩa, vector và các cải tiến gia tăng trên các hệ thống queue, cache và bảo mật.

<a name="minimal-breaking-changes"></a>
### Minimal Breaking Changes

Trọng tâm của chúng tôi trong lần phát hành này là giảm thiểu các thay đổi lớn có thể gây lỗi. Thay vào đó, chúng tôi sẽ cố gắng mang lại những cải tiến liên tục về chất lượng trong suốt cả năm mà không làm ảnh hưởng đến các ứng dụng hiện có.

Do đó, bản phát hành Laravel 13 là một bản nâng cấp tương đối nhỏ về mặt công sức thực hiện, trong khi vẫn mang lại các khả năng mới đáng kể. Xét theo khía cạnh này, hầu hết các ứng dụng Laravel có thể nâng cấp lên Laravel 13 mà không cần thực hiện bất kỳ thay đổi code nào.

<a name="php-8"></a>
### PHP 8.3

Laravel 13.x sẽ yêu cầu phiên bản PHP tối thiểu là 8.3.

<a name="ai-sdk"></a>
### Laravel AI SDK

Laravel 13 giới thiệu bộ [Laravel AI SDK](https://laravel.com/ai) chính thức, cung cấp một API thống nhất cho việc tạo văn bản, các tool cho agent, embedding, âm thanh, hình ảnh và tích hợp vector-store.

Với AI SDK, bạn có thể xây dựng các tính năng AI mà không phụ thuộc vào bất kỳ nhà cung cấp nào, đồng thời vẫn giữ được trải nghiệm nhà phát triển Laravel-native nhất quán.

Ví dụ: một agent cơ bản có thể được thực hiện chỉ với một lệnh duy nhất:

```php
use App\Ai\Agents\SalesCoach;

$response = SalesCoach::make()->prompt('Analyze this sales transcript...');

return (string) $response;
```

Laravel AI SDK cũng có thể tạo ra hình ảnh, âm thanh và embedding:

Đối với các trường hợp sử dụng tạo hình ảnh, SDK cung cấp một API rõ ràng để tạo hình ảnh từ các câu lệnh bằng ngôn ngữ tự nhiên:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

Đối với trải nghiệm giọng nói, bạn có thể tạo âm thanh tự nhiên từ văn bản cho các trợ lý AI, lời dẫn truyện và các tính năng hỗ trợ khác:

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();

$rawContent = (string) $audio;
```

Và đối với các luồng công việc tìm kiếm ngữ nghĩa và lấy ra, bạn có thể tạo embedding trực tiếp từ các chuỗi string:

```php
use Illuminate\Support\Str;

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

<a name="json-api"></a>
### JSON:API Resources

Laravel cũng thêm các [resource JSON:API](/docs/{{version}}/eloquent-resources#jsonapi-resources) chính thức, giúp trả về các response tuân thủ đặc tả JSON:API một cách đơn giản.

Resource JSON:API sẽ xử lý việc chuyển đổi đối tượng resource, bao gồm các quan hệ, lọc field, link và các header response tuân thủ JSON:API.

<a name="request-forgery-protection"></a>
### Request Forgery Protection

Về bảo mật, middleware [ngăn chặn giả mạo request](/docs/{{version}}/csrf#preventing-csrf-requests) của Laravel đã được tăng cường và chính thức trở thành `PreventRequestForgery`, bổ sung các tính năng xác minh request nhận biết qua origin trong khi vẫn duy trì khả năng tương thích với tính năng bảo vệ CSRF dựa trên token.

<a name="queue-routing"></a>
### Queue Routing

Laravel 13 bổ sung tính năng [queue routing theo class](/docs/{{version}}/queues#queue-routing) thông qua `Queue::route(...)`, cho phép bạn định nghĩa các quy tắc routing queue, kết nối mặc định cho các job cụ thể ở một nơi duy nhất:

```php
Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
```

<a name="php-attributes"></a>
### Expanded PHP Attributes

Laravel 13 tiếp tục mở rộng hỗ trợ PHP attribute chính thức trên toàn bộ framework, giúp cho các việc cấu hình và các hành động phổ biến của Laravel trở nên dễ khai báo hơn và được đặt cùng vị trí với các class và phương thức của bạn.

Các attribute đáng chú ý như các attribute cho controller và authorization như [`#[Middleware]`](/docs/{{version}}/controllers#controller-middleware) và [`#[Authorize]`](/docs/{{version}}/controllers#authorization-attributes), cũng như các thuộc tính điều khiển job cho queue như [`#[Tries]`](/docs/{{version}}/queues#max-job-attempts-and-timeout), [`#[Backoff]`](/docs/{{version}}/queues#dealing-with-failed-jobs), [`#[Timeout]`](/docs/{{version}}/queues#max-job-attempts-and-timeout) và [`#[FailOnTimeout]`](/docs/{{version}}/queues#failing-on-timeout).

Ví dụ: Các bài kiểm tra middleware và policy của controller hiện có thể được khai báo trực tiếp trên các class và phương thức:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Routing\Attributes\Controllers\Authorize;
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
class CommentController
{
    #[Middleware('subscribed')]
    #[Authorize('create', [Comment::class, 'post'])]
    public function store(Post $post)
    {
        // ...
    }
}
```

Các attribute bổ sung cũng đã được giới thiệu trong các API của Eloquent, event, thông báo, validation, testing và chuyển hoá resource API, cung cấp cho bạn một tùy chọn tốt hơn trong nhiều mặt của framework.

<a name="cache-touch"></a>
### Cache TTL Extension

Laravel hiện đã tích hợp [`Cache::touch(...)`](/docs/{{version}}/cache), cho phép bạn gia hạn thời gian (TTL) của một item cache mà không cần phải lấy ra và lưu lại giá trị của nó.

<a name="semantic-search"></a>
### Semantic / Vector Search

Laravel 13 làm sâu sắc thêm câu chuyện tìm kiếm ngữ nghĩa với sự hỗ trợ truy vấn vector, các luồng công việc embedding và các API liên quan được tài liệu hóa trong [search](/docs/{{version}}/search#semantic-vector-search), [truy vấn](/docs/{{version}}/queries#vector-similarity-clauses) và [AI SDK](/docs/{{version}}/ai-sdk#embeddings).

Các tính năng này giúp việc xây dựng các trải nghiệm tìm kiếm được hỗ trợ bởi AI bằng PostgreSQL + `pgvector` trở nên đơn giản, bao gồm cả tìm kiếm tương đồng dựa trên các vector embedding được tạo trực tiếp từ các chuỗi string.

Ví dụ: bạn có thể chạy các tìm kiếm tương đồng về mặt ngữ nghĩa trực tiếp từ query builder:

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)
    ->get();
```
