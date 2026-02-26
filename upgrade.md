# Upgrade Guide

- [Nâng cấp đến 12.0 từ 11.x](#upgrade-12.0)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Updating Dependencies](#updating-dependencies)
- [Updating the Laravel Installer](#updating-the-laravel-installer)

</div>

<a name="medium-impact-changes"></a>
## Medium Impact Changes

<div class="content-list" markdown="1">

- [Models and UUIDv7](#models-and-uuidv7)

</div>

<a name="low-impact-changes"></a>
## Low Impact Changes

<div class="content-list" markdown="1">

- [Carbon 3](#carbon-3)
- [Concurrency Result Index Mapping](#concurrency-result-index-mapping)
- [Container Class Dependency Resolution](#container-class-dependency-resolution)
- [Image Validation Now Excludes SVGs](#image-validation)
- [Local Filesystem Disk Default Root Path](#local-filesystem-disk-default-root-path)
- [Multi-Schema Database Inspecting](#multi-schema-database-inspecting)
- [Nested Array Request Merging](#nested-array-request-merging)

</div>

<a name="upgrade-12.0"></a>
## Nâng cấp đến 12.0 từ 11.x

#### Estimated Upgrade Time: 5 Minutes

> [!NOTE]
> Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này mới có thể thực sự ảnh hưởng đến application của bạn. Bạn muốn tiết kiệm thời gian? Bạn có thể sử dụng [Laravel Shift](https://laravelshift.com/) để giúp tự động hóa việc nâng cấp ứng dụng của bạn.

<a name="updating-dependencies"></a>
### Updating Dependencies

**Likelihood Of Impact: High**

Bạn nên cập nhật các library sau vào file `composer.json` của ứng dụng:

<div class="content-list" markdown="1">

- `laravel/framework` to `^12.0`
- `phpunit/phpunit` to `^11.0`
- `pestphp/pest` to `^3.0`

</div>

<a name="carbon-3"></a>
#### Carbon 3

**Likelihood Of Impact: Low**

Sự hỗ trợ cho [Carbon 2.x](https://carbon.nesbot.com/docs/) đã bị loại bỏ. Tất cả các ứng dụng Laravel 12 bây giờ sẽ yêu cầu [Carbon 3.x](https://carbon.nesbot.com/docs/#api-carbon-3).

<a name="updating-the-laravel-installer"></a>
### Updating the Laravel Installer

Nếu bạn đang sử dụng công cụ CLI Laravel installer để tạo các ứng dụng Laravel mới, bạn nên cập nhật bản cài đặt installer của bạn để tương thích với Laravel 12.x và [các starter kit Laravel mới](https://laravel.com/starter-kits). Nếu bạn đã cài đặt Laravel installer thông qua `composer global require`, bạn có thể cập nhật installer bằng lệnh `composer global update`:

```shell
composer global update laravel/installer
```

Nếu ban đầu bạn cài đặt PHP và Laravel thông qua `php.new`, bạn chỉ cần chạy lại các lệnh cài đặt `php.new` cho hệ điều hành của bạn để cài đặt phiên bản PHP mới nhất và Laravel installer:

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

```shell tab=Windows PowerShell
# Run as administrator...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

Hoặc, nếu bạn đang sử dụng bản copy Laravel installer đi kèm với [Laravel Herd](https://herd.laravel.com), bạn nên cập nhật bản cài đặt Herd của bạn để lên phiên bản mới nhất.

<a name="authentication"></a>
### Authentication

<a name="updated-databasetokenrepository-constructor-signature"></a>
#### Updated `DatabaseTokenRepository` Constructor Signature

**Likelihood Of Impact: Very Low**

Constructor của class `Illuminate\Auth\Passwords\DatabaseTokenRepository` hiện tại sẽ yêu cầu tham số `$expires` được tính bằng giây, thay vì bằng phút.

<a name="concurrency"></a>
### Concurrency

<a name="concurrency-result-index-mapping"></a>
#### Concurrency Result Index Mapping

**Likelihood Of Impact: Low**

Khi gọi phương thức `Concurrency::run` với một mảng, thì kết quả của các phương thức chạy song song sẽ được trả về cùng với các key của mảng:

```php
$result = Concurrency::run([
    'task-1' => fn () => 1 + 1,
    'task-2' => fn () => 2 + 2,
]);

// ['task-1' => 2, 'task-2' => 4]
```

<a name="container"></a>
### Container

<a name="container-class-dependency-resolution"></a>
#### Container Class Dependency Resolution

**Likelihood Of Impact: Low**

Dependency injection container hiện tuân thủ giá trị mặc định của các thuộc tính class khi resolve một class instance. Nếu trước đây bạn dựa vào container để resolve một class instance mà không có giá trị mặc định, thì bạn có thể cần phải điều chỉnh lại ứng dụng của bạn để phù hợp với hành vi mới này:

```php
class Example
{
    public function __construct(public ?Carbon $date = null) {}
}

$example = resolve(Example::class);

// <= 11.x
$example->date instanceof Carbon;

// >= 12.x
$example->date === null;
```

<a name="database"></a>
### Database

<a name="multi-schema-database-inspecting"></a>
#### Multi-Schema Database Inspecting

**Likelihood Of Impact: Low**

Mặc định các phương thức `Schema::getTables()`, `Schema::getViews()` và `Schema::getTypes()` sẽ chứa kết quả từ tất cả các schema. Bạn có thể truyền tham số `schema` để chỉ lấy kết quả cho schema đó:

```php
// All tables on all schemas...
$tables = Schema::getTables();

// All tables on the 'main' schema...
$tables = Schema::getTables(schema: 'main');

// All tables on the 'main' and 'blog' schemas...
$tables = Schema::getTables(schema: ['main', 'blog']);
```

Mặc định phương thức `Schema::getTableListing()` sẽ trả về tên bảng được định danh theo schema. Bạn có thể truyền tham số `schemaQualified` để thay đổi hành vi theo ý muốn:

```php
$tables = Schema::getTableListing();
// ['main.migrations', 'main.users', 'blog.posts']

$tables = Schema::getTableListing(schema: 'main');
// ['main.migrations', 'main.users']

$tables = Schema::getTableListing(schema: 'main', schemaQualified: false);
// ['migrations', 'users']
```

Các lệnh `db:table` và `db:show` sẽ xuất ra kết quả của tất cả các schema trên MySQL, MariaDB và SQLite, giống như PostgreSQL và SQL Server.

<a name="updated-blueprint-constructor-signature"></a>
#### Updated `Blueprint` Constructor Signature

**Likelihood Of Impact: Very Low**

Constructor của class `Illuminate\Database\Schema\Blueprint` sẽ yêu cầu một instance của `Illuminate\Database\Connection` làm tham số đầu tiên của nó.

<a name="eloquent"></a>
### Eloquent

<a name="models-and-uuidv7"></a>
#### Models and UUIDv7

**Likelihood Of Impact: Medium**

Trait `HasUuids` sẽ trả về UUID tương thích với phiên bản 7 của đặc tả UUID (ordered UUID). Nếu bạn muốn tiếp tục sử dụng chuỗi UUIDv4 có thứ tự cho ID model của bạn, thì bạn nên sử dụng trait `HasVersion4Uuids`:

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids; // [tl! remove]
use Illuminate\Database\Eloquent\Concerns\HasVersion4Uuids as HasUuids; // [tl! add]
```

Trait `HasVersion7Uuids` đã bị loại bỏ. Nếu trước đây bạn đã sử dụng trait này, bạn nên sử dụng trait `HasUuids` để thay thế, trait này hiện cung cấp hành vi tương tự.

<a name="requests"></a>
### Requests

<a name="nested-array-request-merging"></a>
#### Nested Array Request Merging

**Likelihood Of Impact: Low**

Phương thức `$request->mergeIfMissing()` sẽ cho phép merge dữ liệu mảng lồng nhau bằng cách sử dụng ký tự "chấm". Nếu trước đây bạn dựa vào phương thức này để tạo key mảng có chứa ký tự "chấm", thì bạn có thể cần phải điều chỉnh ứng dụng của bạn để phù hợp với hành vi mới này:

```php
$request->mergeIfMissing([
    'user.last_name' => 'Otwell',
]);
```

<a name="storage"></a>
### Storage

<a name="local-filesystem-disk-default-root-path"></a>
#### Local Filesystem Disk Default Root Path

**Likelihood Of Impact: Low**

Nếu ứng dụng của bạn không định nghĩa rõ ràng disk `local` trong cấu hình filesystem, Laravel hiện sẽ để mặc định root của disk local thành `storage/app/private`. Trong các phiên bản trước, giá trị mặc định này là `storage/app`. Do đó, các lệnh gọi đến `Storage::disk('local')` sẽ đọc và ghi vào `storage/app/private` trừ khi được cấu hình khác. Để khôi phục hành vi trước đó, bạn có thể định nghĩa disk `local` theo cách thủ công và set một đường dẫn root khác mà bạn mong muốn.

<a name="validation"></a>
### Validation

<a name="image-validation"></a>
#### Image Validation Now Excludes SVGs

**Likelihood Of Impact: Low**

Mặc định, rule validation `image` không còn cho phép hình ảnh SVG. Nếu bạn muốn cho phép SVG khi sử dụng rule `image`, bạn phải cấp phép cho chúng một cách rõ ràng:

```php
use Illuminate\Validation\Rules\File;

'photo' => 'required|image:allow_svg'

// Or...
'photo' => ['required', File::image(allowSvg: true)],
```

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/11.x...12.x) và chọn bản cập nhật nào quan trọng với bạn.
