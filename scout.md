# Laravel Scout

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Queueing](#queueing)
- [Yêu cầu của driver](#driver-prerequisites)
- [Configuration](#configuration)
    - [Configuring Searchable Data](#configuring-searchable-data)
- [Database và Collection Engines](#database-and-collection-engines)
    - [Database Engine](#database-engine)
    - [Collection Engine](#collection-engine)
- [Cấu hình Engine của Third-Party](#third-party-engine-configuration)
    - [Cấu hình Model Indexes](#configuring-model-indexes)
    - [Algolia](#algolia-configuration)
    - [Meilisearch](#meilisearch-configuration)
    - [Typesense](#typesense-configuration)
- [Third-Party Engine Indexing](#indexing)
    - [Batch Import](#batch-import)
    - [Adding Records](#adding-records)
    - [Updating Records](#updating-records)
    - [Removing Records](#removing-records)
    - [Pausing Indexing](#pausing-indexing)
    - [Điều kiện Model Searchable](#conditionally-searchable-model-instances)
- [Searching](#searching)
    - [Where Clauses](#where-clauses)
    - [Pagination](#pagination)
    - [Soft Deleting](#soft-deleting)
    - [Tuỳ chỉnh Engine Search](#customizing-engine-searches)
- [Custom Engines](#custom-engines)

<a name="introduction"></a>
## Giới thiệu

[Laravel Scout](https://github.com/laravel/scout) cung cấp một giải pháp dựa trên driver đơn giản để thêm chức năng tìm kiếm full-text vào [các model Eloquent](/docs/{{version}}/eloquent). Sử dụng model observer, Scout sẽ tự động giữ các index tìm kiếm và đồng bộ nó với các bản ghi trong Eloquent của bạn.

Scout có sẵn một engine `database` được tích hợp sẵn, sử dụng các index full-text của MySQL, PostgreSQL và các mệnh đề `LIKE` để tìm kiếm trong cơ sở dữ liệu hiện tại của bạn — không yêu cầu dịch vụ bên ngoài. Đối với hầu hết các ứng dụng, đây là tất cả những gì bạn cần. Để có cái nhìn tổng quan về tất cả các tùy chọn tìm kiếm có sẵn trong Laravel, hãy tham khảo [tài liệu tìm kiếm](/docs/{{version}}/search).

Scout cũng chứa các driver cho [Algolia](https://www.algolia.com/), [Meilisearch](https://www.meilisearch.com) và [Typesense](https://typesense.org) khi bạn cần các tính năng nâng cao như xử lý lỗi chính tả, lọc đa chiều, hoặc tìm kiếm theo vị trí địa lý ở quy mô rất lớn. Driver "collection" cũng có sẵn để phát triển ở local và bạn cũng có thể tự viết [các engine tùy chỉnh](#custom-engines) của riêng bạn.

<a name="installation"></a>
## Cài đặt

Đầu tiên, hãy cài đặt Scout thông qua package manager Composer:

```shell
composer require laravel/scout
```

Sau khi cài đặt Scout xong, bạn nên export file cấu hình của Scout bằng lệnh Artisan `vendor:publish`. Lệnh này sẽ export file cấu hình `scout.php` vào thư mục `config` của application của bạn:

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

Cuối cùng, thêm trait `Laravel\Scout\Searchable` vào model mà bạn muốn thêm chức năng tìm kiếm. Trait này sẽ đăng ký một model observer để tự động giữ cho các model được đồng bộ với driver tìm kiếm của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;
}
```

<a name="queueing"></a>
### Queueing

Khi sử dụng một engine không phải là engine `database` hoặc `collection`, bạn nên cân nhắc cấu hình một [queue driver](/docs/{{version}}/queues) trước khi sử dụng thư viện. Chạy một queue worker sẽ cho phép Scout queue tất cả các hoạt động đồng bộ hóa thông tin model của bạn với các search index, giúp cung cấp thời gian phản hồi nhanh hơn cho giao diện ứng dụng web của bạn.

Khi bạn đã cấu hình xong queue driver, hãy set giá trị của tùy chọn `queue` trong file cấu hình `config/scout.php` của bạn là `true`:

```php
'queue' => true,
```

Ngay cả khi tùy chọn `queue` được set thành `false`, thì điều quan trọng bạn cần nhớ là một số driver Scout như Algolia và Meilisearch vẫn luôn lập index cho các bản ghi theo chế độ không đồng bộ. Nói cách khác, ngay cả khi hoạt động lập index đã hoàn tất trong ứng dụng Laravel của bạn, thì bản thân công cụ tìm kiếm vẫn có thể không phản ánh ngay lập tức các bản ghi mới hoặc các bản ghi đã được cập nhật.

Để chỉ định kết nối và queue nào mà job Scout của bạn sử dụng, bạn có thể định nghĩa tùy chọn cấu hình `queue` dưới dạng một mảng:

```php
'queue' => [
    'connection' => 'redis',
    'queue' => 'scout'
],
```

Tất nhiên, nếu bạn muốn tùy chỉnh kết nối và queue mà các tác vụ Scout sử dụng, bạn nên chạy một queue worker để xử lý các tác vụ trên kết nối và queue đó:

```shell
php artisan queue:work redis --queue=scout
```

<a name="driver-prerequisites"></a>
### Yêu cầu của driver

<a name="algolia"></a>
#### Algolia

Khi sử dụng driver Algolia, bạn nên cấu hình thông tin đăng nhập `id` và `secret` trong file cấu hình `config/scout.php` của bạn. Khi thông tin đăng nhập của bạn đã được cấu hình xong, bạn cũng sẽ cần cài đặt thêm SDK PHP Algolia thông qua package manager Composer:

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
#### MeiliSearch

[MeiliSearch](https://www.meilisearch.com) là một công cụ tìm kiếm mã nguồn mở, có tốc độ nhanh. Nếu bạn không chắc chắn về cách cài đặt MeiliSearch trên máy local của bạn, bạn có thể sử dụng [Laravel Sail](/docs/{{version}}/sail#meilisearch), một môi trường phát triển Docker được hỗ trợ chính thức của Laravel.

Khi sử dụng driver MeiliSearch, bạn sẽ cần cài đặt MeiliSearch PHP SDK thông qua Composer package manager:

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

Sau đó, set biến môi trường `SCOUT_DRIVER` cũng như thông tin đăng nhập MeiliSearch `host` và `key` vào trong file `.env` của ứng dụng của bạn:

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

Để biết thêm thông tin về MeiliSearch, vui lòng tham khảo [tài liệu MeiliSearch](https://docs.meilisearch.com/learn/getting_started/quick_start.html).

Ngoài ra, bạn nên đảm bảo rằng bạn đã cài đặt phiên bản `meilisearch/meilisearch-php` tương thích với phiên bản binary MeiliSearch của bạn bằng cách xem lại [tài liệu của MeiliSearch về khả năng tương thích binary](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch).

> [!WARNING]
> Khi upgrade Scout trên một ứng dụng đã sử dụng MeiliSearch, bạn phải luôn [xem lại những thay đổi nghiêm trọng](https://github.com/meilisearch/MeiliSearch/releases) đối với chính service MeiliSearch của bạn.

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) là một công cụ tìm kiếm mã nguồn mở cực nhanh và hỗ trợ tìm kiếm theo từ khóa, tìm kiếm ngữ nghĩa, tìm kiếm theo địa lý và tìm kiếm theo vector.

Bạn có thể [tự dựng server](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting) Typesense hoặc sử dụng [Typesense Cloud](https://cloud.typesense.org).

Để bắt đầu sử dụng Typesense với Scout, hãy cài đặt Typesense PHP SDK thông qua trình quản lý package Composer:

```shell
composer require typesense/typesense-php
```

Sau đó, hãy set biến môi trường `SCOUT_DRIVER` cũng như thông tin xác thực host Typesense và API key trong file .env của ứng dụng:

```env
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail), bạn có thể cần điều chỉnh biến môi trường `TYPESENSE_HOST` để giống với tên container Docker. Bạn cũng có thể tùy chọn chỉ định cổng, đường dẫn và giao thức trong cài đặt của bạn:

```ini
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

Bạn có thể tìm thấy các thiết lập bổ sung và các định nghĩa schema cho collection Typesense của bạn trong file cấu hình `config/scout.php` của ứng dụng. Để biết thêm thông tin về Typesense, vui lòng tham khảo [tài liệu Typesense](https://typesense.org/docs/guide/#quick-start).

<a name="configuration"></a>
## Configuration

<a name="configuring-searchable-data"></a>
### Configuring Searchable Data

Mặc định, toàn bộ định dạng `toArray` của một model sẽ được lưu vào index tìm kiếm của nó. Nếu bạn muốn tùy chỉnh dữ liệu được đồng bộ với index tìm kiếm, bạn có thể ghi đè phương thức `toSearchableArray` trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Get the indexable data array for the model.
     *
     * @return array<string, mixed>
     */
    public function toSearchableArray(): array
    {
        $array = $this->toArray();

        // Customize the data array...

        return $array;
    }
}
```

<a name="configuring-search-engines-per-model"></a>
#### Configuring Model Engines

Khi tìm kiếm, Scout sẽ sử dụng engine tìm kiếm mặc định được cấu hình trong file cấu hình `scout` của ứng dụng. Tuy nhiên, engine tìm kiếm cho một model cụ thể cũng có thể được thay đổi bằng cách ghi đè phương thức `searchableUsing` trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Engines\Engine;
use Laravel\Scout\Scout;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the engine used to index the model.
     */
    public function searchableUsing(): Engine
    {
        return Scout::engine('meilisearch');
    }
}
```

<a name="database-and-collection-engines"></a>
## Database / Collection Engines

<a name="database-engine"></a>
### Database Engine

> [!WARNING]
> Engine database hiện hỗ trợ MySQL và PostgreSQL, cả hai đều cung cấp hỗ trợ cho việc lập index full-text của cột nhanh chóng.

Engine `database` sử dụng các index full-text của MySQL, PostgreSQL và các mệnh đề `LIKE` để tìm kiếm trực tiếp trong cơ sở dữ liệu hiện tại của bạn. Đối với nhiều ứng dụng, đây là cách đơn giản và thiết thực nhất để thêm chức năng tìm kiếm — không yêu cầu dịch vụ bên ngoài hoặc cơ sở hạ tầng bổ sung.

Để sử dụng engine database, hãy set biến môi trường `SCOUT_DRIVER` thành `database`:

```ini
SCOUT_DRIVER=database
```

Sau khi được cấu hình, bạn có thể [định nghĩa dữ liệu tìm kiếm](#configuring-searchable-data) và bắt đầu [thực hiện các truy vấn tìm kiếm](#searching) cho các model của bạn. Không giống như các engine của bên thứ ba, engine database không yêu cầu bước lập index — nó tìm kiếm trực tiếp trên các bảng cơ sở dữ liệu của bạn.

#### Customizing Database Searching Strategies

Mặc định, engine database sẽ thực hiện một truy vấn `LIKE` đối với mọi thuộc tính model mà bạn đã [cấu hình là tìm kiếm được](#configuring-searchable-data). Tuy nhiên, bạn có thể gán thêm các chiến lược tìm kiếm hiệu quả hơn cho các cột cụ thể. Thuộc tính `SearchUsingFullText` sẽ sử dụng index full-text của cơ sở dữ liệu cho cột đó, trong khi thuộc tính `SearchUsingPrefix` sẽ chỉ tìm phần đầu của chuỗi (`example%`) thay vì tìm kiếm trong toàn bộ chuỗi (`%example%`).

Để định nghĩa hành vi này, hãy gán các thuộc tính PHP cho phương thức `toSearchableArray` của model. Bất kỳ cột nào không có thuộc tính sẽ tiếp tục sử dụng chiến lược `LIKE` mặc định:

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

> [!WARNING]
> Trước khi chỉ định một cột sẽ sử dụng tìm kiếm full-text, bạn hãy đảm bảo cột đó đã được gán một [index full-text](/docs/{{version}}/migrations#available-index-types).

<a name="collection-engine"></a>
### Collection Engine

Engine "collection" dành cho các thử bản nghiệm nhanh, các tập dữ liệu cực nhỏ (vài trăm bản ghi) hoặc chạy test. Nó lấy tất cả các bản ghi có thể có từ cơ sở dữ liệu của bạn và sử dụng helper `Str::is` của Laravel để lọc chúng trong PHP, vì vậy nó không yêu cầu bất kỳ tính năng lập index hay tính năng cụ thể nào của cơ sở dữ liệu. Đối với bất kỳ trường hợp nào khác với các trường hợp sử dụng đơn giản, bạn nên sử dụng [engine database](#database-engine) để thay thế.

Để sử dụng engine collection, bạn có thể chỉ cần set giá trị của biến môi trường `SCOUT_DRIVER` thành `collection` hoặc chỉ định driver `collection` trực tiếp trong file cấu hình `scout` của ứng dụng:

```ini
SCOUT_DRIVER=collection
```

Sau khi bạn đã chỉ định driver collection làm driver chính của bạn, bạn có thể bắt đầu [thực hiện các truy vấn tìm kiếm](#searching) đối với các model của bạn. Việc lập index cho các công cụ tìm kiếm, chẳng hạn như việc lập index để nạp dữ liệu cho các index của Algolia, Meilisearch hoặc Typesense, sẽ không cần thiết khi bạn sử dụng engine collection.

#### Differences From Database Engine

Trong khi engine database sử dụng các index full-text và các mệnh đề `LIKE` để tìm các bản ghi phù hợp một cách hiệu quả, engine collection sẽ lấy tất cả các bản ghi và lọc chúng trong PHP. Engine collection là tùy chọn linh hoạt nhất vì nó hoạt động trên tất cả các cơ sở dữ liệu quan hệ được Laravel hỗ trợ (bao gồm SQLite và SQL Server); tuy nhiên, nó kém hiệu quả hơn đáng kể so với engine database và không nên được sử dụng cho các tập dữ liệu lớn.

<a name="third-party-engine-configuration"></a>
## Cấu hình Engine của Third-Party

Các tùy chọn cấu hình sau đây sẽ chỉ phù hợp khi sử dụng engine tìm kiếm của bên thứ ba như Algolia, Meilisearch hoặc Typesense. Nếu bạn đang sử dụng [engine database](#database-engine), bạn có thể bỏ qua phần này.

<a name="configuring-model-indexes"></a>
### Cấu hình Model Indexes

Khi sử dụng một engine của bên thứ ba, mỗi một model Eloquent sẽ được đồng bộ với một "index" tìm kiếm nhất định, nó sẽ chứa tất cả các bản ghi có thể được tìm kiếm cho model đó. Mặc định, mỗi model sẽ được lưu trữ theo một index khớp với tên "bảng" của model. Thông thường, là dạng số nhiều của tên model; tuy nhiên, bạn có thể tùy biến index của model bằng cách ghi đè phương thức `searchableAs` trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Get the name of the index associated with the model.
     */
    public function searchableAs(): string
    {
        return 'posts_index';
    }
}
```

> [!NOTE]
> Phương thức `searchableAs` không có tác dụng khi sử dụng engine database, vì nó luôn tìm kiếm trực tiếp trên bảng cơ sở dữ liệu của model.

<a name="configuring-the-model-id"></a>
#### Configuring the Model ID

Mặc định, Scout sẽ sử dụng khóa chính của model làm khóa trong index tìm kiếm. Nếu bạn cần tùy chỉnh hành vi này khi sử dụng engine của bên thứ ba, bạn có thể ghi đè các phương thức `getScoutKey` và `getScoutKeyName` trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the value used to index the model.
     */
    public function getScoutKey(): mixed
    {
        return $this->email;
    }

    /**
     * Get the key name used to index the model.
     */
    public function getScoutKeyName(): mixed
    {
        return 'email';
    }
}
```

> [!NOTE]
> Các phương thức `getScoutKey` và `getScoutKeyName` không có tác dụng khi sử dụng engine database, vì nó luôn sử dụng khóa chính của model.

<a name="algolia-configuration"></a>
### Algolia

<a name="algolia-index-settings"></a>
#### Index Settings

Thỉnh thoảng bạn có thể muốn cấu hình thêm các cài đặt trên các index Algolia của bạn. Mặc dù bạn có thể quản lý các cài đặt này thông qua giao diện người dùng của Algolia, nhưng đôi khi việc quản lý trạng thái của cấu hình index trực tiếp từ file cấu hình `config/scout.php` của ứng dụng cũng sẽ hiệu quả hơn.

Cách tiếp cận này cho phép bạn deploy các cài đặt thông qua quy trình deploy tự động của ứng dụng, tránh việc cấu hình thủ công và đảm bảo tính nhất quán trên nhiều môi trường. Bạn có thể cấu hình các thuộc tính filterable, ranking, faceting hoặc [bất kỳ cài đặt nào khác được hỗ trợ](https://www.algolia.com/doc/rest-api/search/#tag/Indices/operation/setSettings).

Để bắt đầu, thêm các cài đặt cho mỗi index trong file cấu hình `config/scout.php` của ứng dụng:

```php
use App\Models\User;
use App\Models\Flight;

'algolia' => [
    'id' => env('ALGOLIA_APP_ID', ''),
    'secret' => env('ALGOLIA_SECRET', ''),
    'index-settings' => [
        User::class => [
            'searchableAttributes' => ['id', 'name', 'email'],
            'attributesForFaceting'=> ['filterOnly(email)'],
            // Other settings fields...
        ],
        Flight::class => [
            'searchableAttributes'=> ['id', 'destination'],
        ],
    ],
],
```

Nếu model cơ bản của một index nhất định có thể bị soft delete và được chứa trong mảng `index-settings`, Scout sẽ tự động hỗ trợ phân loại trên các model đã bị soft delete trên index đó. Nếu bạn không có thuộc tính phân loại nào khác để định nghĩa cho một index model có thể bị soft delete, bạn có thể chỉ cần thêm một mục trống vào mảng `index-settings` cho model đó:

```php
'index-settings' => [
    Flight::class => []
],
```

Sau khi cấu hình xong các cài đặt index của ứng dụng, bạn phải chạy lại lệnh Artisan `scout:sync-index-settings`. Lệnh này sẽ thông báo cho Algolia về các cài đặt index hiện tại của bạn. Để thuận tiện, bạn có thể muốn đưa lệnh này vào quy trình deploy của bạn:

```shell
php artisan scout:sync-index-settings
```

<a name="algolia-identifying-users"></a>
#### Identifying Users

Scout cho phép bạn tự động xác định người dùng khi sử dụng Algolia. Việc liên kết người dùng với các hoạt động tìm kiếm có thể hữu ích khi xem phân tích tìm kiếm của bạn trong dashboard của Algolia. Bạn có thể bật tính năng xác định người dùng bằng cách định nghĩa biến môi trường `SCOUT_IDENTIFY` là `true` trong file `.env` của ứng dụng:

```ini
SCOUT_IDENTIFY=true
```

Việc bật tính năng này cũng sẽ gửi địa chỉ IP của request và ID của người dùng đến Algolia để dữ liệu này được liên kết với bất kỳ request tìm kiếm nào do người dùng thực hiện.

<a name="meilisearch-configuration"></a>
### Meilisearch

<a name="meilisearch-index-settings"></a>
#### Index Settings

Meilisearch yêu cầu bạn phải định nghĩa trước các cài đặt tìm kiếm index như thuộc tính có thể lọc, thuộc tính có thể sắp xếp và [các trường cài đặt được hỗ trợ khác](https://docs.meilisearch.com/reference/api/settings.html).

Thuộc tính có thể lọc là bất kỳ thuộc tính nào bạn muốn lọc khi gọi phương thức `where` của Scout, trong khi thuộc tính có thể sắp xếp là bất kỳ thuộc tính nào bạn muốn sắp xếp khi gọi phương thức `orderBy` của Scout. Để định nghĩa cài đặt index của bạn, hãy điều chỉnh phần `index-settings` của mục cấu hình `meilisearch` trong file cấu hình `scout` của ứng dụng:

```php
use App\Models\User;
use App\Models\Flight;

'meilisearch' => [
    'host' => env('MEILISEARCH_HOST', 'http://localhost:7700'),
    'key' => env('MEILISEARCH_KEY', null),
    'index-settings' => [
        User::class => [
            'filterableAttributes'=> ['id', 'name', 'email'],
            'sortableAttributes' => ['created_at'],
            // Other settings fields...
        ],
        Flight::class => [
            'filterableAttributes'=> ['id', 'destination'],
            'sortableAttributes' => ['updated_at'],
        ],
    ],
],
```

Nếu model cơ sở của một index nhất định là loại có thể soft delete và được chứa trong mảng `index-settings`, thì Scout sẽ tự động hỗ trợ việc lọc trên các model đã soft delete trên index đó. Nếu bạn không có thuộc tính có thể lọc hoặc thuộc tính có thể sắp xếp nào khác để định nghĩa cho index model soft delete, bạn chỉ cần thêm một mục trống vào mảng `index-settings` cho model đó:

```php
'index-settings' => [
    Flight::class => []
],
```

Sau khi cấu hình xong cài đặt index của ứng dụng, bạn phải gọi lệnh Artisan `scout:sync-index-settings`. Lệnh này sẽ thông báo cho Meilisearch về cài đặt index hiện được cấu hình của bạn. Để thuận tiện, bạn có thể muốn đưa lệnh này vào process deploy của bạn:

```shell
php artisan scout:sync-index-settings
```

<a name="meilisearch-data-types"></a>
#### Searchable Data Types

Meilisearch sẽ chỉ thực hiện các hoạt động lọc (`>`, `<`, vv...) trên đúng kiểu dữ liệu. Khi tùy chỉnh dữ liệu có thể tìm kiếm của bạn, bạn nên đảm bảo rằng các giá trị số được cast về đúng kiểu của chúng:

```php
public function toSearchableArray()
{
    return [
        'id' => (int) $this->id,
        'name' => $this->name,
        'price' => (float) $this->price,
    ];
}
```

<a name="typesense-configuration"></a>
### Typesense

<a name="typesense-searchable-data"></a>
#### Preparing Searchable Data

Khi sử dụng Typesense, các model có thể tìm kiếm của bạn phải định nghĩa phương thức `toSearchableArray` để cast khóa chính của model thành một chuỗi và ngày tạo thành UNIX timestamp:

```php
/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
public function toSearchableArray(): array
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

Bạn cũng nên định nghĩa các schema cho collection Typesense trong file `config/scout.php` của ứng dụng. Một collection schema mô tả các kiểu dữ liệu của mỗi trường có thể tìm kiếm thông qua Typesense. Để biết thêm thông tin về tất cả các tùy chọn schema có sẵn, vui lòng tham khảo [tài liệu Typesense](https://typesense.org/docs/latest/api/collections.html#schema-parameters).

Nếu bạn cần thay đổi schema của collection Typesense sau khi nó đã được định nghĩa, bạn có thể chạy `scout:flush` và `scout:import` để xóa tất cả dữ liệu đã lập index hiện tại và tạo lại schema. Hoặc, bạn có thể sử dụng API của Typesense để sửa schema của collection mà không cần xóa bất kỳ dữ liệu nào đã lập index.

Nếu model tìm kiếm của bạn có thể bị soft delete, bạn nên định nghĩa một trường `__soft_deleted` trong schema Typesense tương ứng với model đó bên trong file cấu hình `config/scout.php` của ứng dụng:

```php
User::class => [
    'collection-schema' => [
        'fields' => [
            // ...
            [
                'name' => '__soft_deleted',
                'type' => 'int32',
                'optional' => true,
            ],
        ],
    ],
],
```

<a name="typesense-dynamic-search-parameters"></a>
#### Dynamic Search Parameters

Typesense cho phép bạn sửa các [tham số tìm kiếm](https://typesense.org/docs/latest/api/search.html#search-parameters) của bạn một cách linh hoạt khi thực hiện các hoạt động tìm kiếm thông qua phương thức `options`:

```php
use App\Models\Todo;

Todo::search('Groceries')->options([
    'query_by' => 'title, description'
])->get();
```

<a name="indexing"></a>
## Third-Party Engine Indexing

> [!NOTE]
> Các chức năng lập index được mô tả ở trong phần này chủ yếu liên quan khi sử dụng engine của bên thứ ba (Algolia, Meilisearch hoặc Typesense). Engine database sẽ tìm kiếm trực tiếp trên các bảng cơ sở dữ liệu của bạn, nên vì vậy nó không yêu cầu bạn phải quản lý index.

<a name="batch-import"></a>
### Batch Import

Nếu bạn đang cài đặt Scout cho một project đã tồn tại, có thể bạn đã có các bản ghi trong cơ sở dữ liệu và bạn cần import nó vào index của bạn. Scout cung cấp một lệnh Artisan `scout:import` mà bạn có thể sử dụng để import tất cả các bản ghi hiện có vào các index tìm kiếm của bạn:

```shell
php artisan scout:import "App\Models\Post"
```

Lệnh `scout:queue-import` có thể được sử dụng để import tất cả các bản ghi hiện có của bạn bằng cách sử dụng [các queued job](/docs/{{version}}/queues):

```shell
php artisan scout:queue-import "App\Models\Post" --chunk=500
```

Lệnh `flush` có thể được sử dụng để xóa tất cả các bản ghi của model ra khỏi các search index của bạn:

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### Modifying The Import Query

Nếu bạn muốn sửa truy vấn được sử dụng để lấy ra tất cả các model của bạn để import hàng loạt, bạn có thể định nghĩa phương thức `makeAllSearchableUsing` trên model của bạn. Đây là nơi tuyệt vời để thêm bất kỳ quan hệ eager loading nào có thể cần thiết trước khi import model của bạn:

```php
use Illuminate\Database\Eloquent\Builder;

/**
 * Modify the query used to retrieve models when making all of the models searchable.
 */
protected function makeAllSearchableUsing(Builder $query): Builder
{
    return $query->with('author');
}
```

> [!WARNING]
> Phương thức `makeAllSearchableUsing` có thể không áp dụng được khi sử dụng queue để import model. Các quan hệ sẽ [không được khôi phục](/docs/{{version}}/queues#handling-relationships) khi các collection model được xử lý bởi các job.

<a name="adding-records"></a>
### Adding Records

Khi mà bạn đã thêm trait `Laravel\Scout\Searchable` vào một model, tất cả những gì bạn cần làm là `save` hoặc `create` một instance model và nó sẽ tự động được thêm vào index tìm kiếm cho bạn. Nếu bạn đã cấu hình Scout để [sử dụng queue](#queueing) thì thao tác này sẽ được thực hiện dưới background bởi queue worker của bạn:

```php
use App\Models\Order;

$order = new Order;

// ...

$order->save();
```

<a name="adding-records-via-query"></a>
#### Adding Records Via Query

Nếu bạn muốn thêm một collection model vào index tìm kiếm của bạn thông qua truy vấn của Eloquent, bạn có thể kết hợp thêm phương thức `searchable` vào truy vấn của Eloquent. Phương thức `searchable` sẽ [chunk các kết quả](/docs/{{version}}/eloquent#chunking-results) của truy vấn và thêm các bản ghi vào index tìm kiếm của bạn. Một lần nữa, nếu bạn đã cấu hình Scout để sử dụng queue, tất cả các chunk sẽ được import vào dưới background bởi các queue worker của bạn:

```php
use App\Models\Order;

Order::where('price', '>', 100)->searchable();
```

Bạn cũng có thể gọi phương thức `searchable` trên instance quan hệ Eloquent:

```php
$user->orders()->searchable();
```

Hoặc, nếu bạn đã có một collection các model Eloquent trong bộ nhớ, bạn có thể gọi phương thức `searchable` trên instance collection để thêm các instance model vào index tương ứng của chúng:

```php
$orders->searchable();
```

> [!NOTE]
> Phương thức `searchable` có thể được coi như là một hành động "updateOrCreate". Nói cách khác, nếu bản ghi model đã có trong index của bạn, nó sẽ được cập nhật. Nếu nó không tồn tại trong index, nó sẽ được thêm vào index.

<a name="updating-records"></a>
### Updating Records

Để cập nhật một model mà có thể tìm kiếm, bạn chỉ cần cập nhật các thuộc tính của instance model và `save` model đó vào cơ sở dữ liệu của bạn. Scout sẽ tự động lưu các thay đổi đối với index tìm kiếm của bạn:

```php
use App\Models\Order;

$order = Order::find(1);

// Update the order...

$order->save();
```

Bạn cũng có thể gọi phương thức `searchable` trên một instance truy vấn của Eloquent để cập nhật một collection model. Nếu model không tồn tại trong index tìm kiếm của bạn, chúng sẽ được tạo mới:

```php
Order::where('price', '>', 100)->searchable();
```

Nếu bạn muốn cập nhật các bản ghi search index cho tất cả các model trong một quan hệ, bạn có thể gọi `searchable` trên instance quan hệ:

```php
$user->orders()->searchable();
```

Hoặc, nếu bạn đã có một collection các model Eloquent trong bộ nhớ, bạn có thể gọi phương thức `searchable` trên instance collection để cập nhật các instance model vào trong index tương ứng của chúng:

```php
$orders->searchable();
```

<a name="modifying-records-before-importing"></a>
#### Modifying Records Before Importing

Thỉnh thoảng bạn có thể cần chuẩn bị collection các model trước khi chúng được tìm kiếm. Ví dụ, bạn có thể muốn eager load một quan hệ để dữ liệu quan hệ này có thể được thêm vào index tìm kiếm của bạn. Để thực hiện điều này, hãy định nghĩa một phương thức `makeSearchableUsing` trên model tương ứng:

```php
use Illuminate\Database\Eloquent\Collection;

/**
 * Modify the collection of models being made searchable.
 */
public function makeSearchableUsing(Collection $models): Collection
{
    return $models->load('author');
}
```

<a name="conditionally-updating-the-search-index"></a>
#### Conditionally Updating the Search Index

Mặc định, Scout sẽ lập lại index cho một model đã được cập nhật bất kể thuộc tính nào đã được sửa1. Nếu bạn muốn tùy chỉnh hành vi này, bạn có thể định nghĩa phương thức `searchIndexShouldBeUpdated` trên model của bạn:

```php
/**
 * Determine if the search index should be updated.
 */
public function searchIndexShouldBeUpdated(): bool
{
    return $this->wasRecentlyCreated || $this->wasChanged(['title', 'body']);
}
```

<a name="removing-records"></a>
### Removing Records

Để xóa một bản ghi ra khỏi index của bạn, bạn chỉ đơn giản là xóa model đó ra khỏi cơ sở dữ liệu. Điều này có thể được thực hiện ngay cả khi bạn đang sử dụng các mode [đã bị soft delete](/docs/{{version}}/eloquent#soft-deleting):

```php
use App\Models\Order;

$order = Order::find(1);

$order->delete();
```

Nếu bạn không muốn lấy ra model trước khi xóa nó, bạn có thể sử dụng phương thức `unsearchable` trên một instance truy vấn của Eloquent:

```php
Order::where('price', '>', 100)->unsearchable();
```

Nếu bạn muốn xóa bản ghi search index cho tất cả các model trong một quan hệ, bạn có thể gọi `unsearchable` trên instance quan hệ đó:

```php
$user->orders()->unsearchable();
```

Hoặc, nếu bạn đã có một collection các model Eloquent trong bộ nhớ, bạn có thể gọi phương thức `unsearchable` trên instance collection để xóa các instance model ra khỏi index tìm kiếm của chúng:

```php
$orders->unsearchable();
```

Để xóa tất cả các record model ra khỏi index tìm kiếm của chúng, bạn có thể gọi phương thức `removeAllFromSearch`:

```php
Order::removeAllFromSearch();
```

<a name="pausing-indexing"></a>
### Pausing Indexing

Thỉnh thoảng bạn có thể cần thực hiện một loạt các hành động Eloquent trên một model mà không muốn đồng bộ dữ liệu của model đó với index tìm kiếm. Bạn có thể làm điều này bằng cách sử dụng phương thức `withoutSyncingToSearch`. Phương thức này sẽ chấp nhận một closure sẽ được thực hiện ngay lập tức. Bất kỳ hành động model nào xảy ra trong closure này đều sẽ không được đồng bộ với index của model:

```php
use App\Models\Order;

Order::withoutSyncingToSearch(function () {
    // Perform model actions...
});
```

<a name="conditionally-searchable-model-instances"></a>
### Điều kiện Model Searchable

Thỉnh thoảng bạn có thể muốn tìm kiếm trong model searchable có thêm một số điều kiện nhất định. Ví dụ: hãy tưởng tượng bạn có model `App\Models\Post` có thể ở một trong hai trạng thái: "draft" và "published". Bạn có thể chỉ muốn tìm kiếm các bài đăng đã được "published". Để thực hiện điều này, bạn có thể định nghĩa một phương thức `shouldBeSearchable` trên model của bạn:

```php
/**
 * Determine if the model should be searchable.
 */
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

Phương thức `shouldBeSearchable` chỉ được áp dụng khi bạn thao tác với model thông qua phương thức `save` và `create`, các câu lệnh truy vấn hoặc các quan hệ. Bạn gọi trực tiếp phương thức `searchable` qua model hoặc qua các collection searchable, thì nó sẽ ghi đè kết quả của phương thức `shouldBeSearchable`.

> [!WARNING]
> Phương thức `shouldBeSearchable` không áp dụng được khi sử dụng "database" engine của Scout, vì tất cả searchable data luôn được lưu trong cơ sở dữ liệu. Để đạt được hành vi tương tự khi sử dụng database engine, bạn nên sử dụng [lệnh where](#where-clauses) thay thế.

<a name="searching"></a>
## Searching

Bạn có thể bắt đầu tìm kiếm một model bằng phương thức `search`. Phương thức search chấp nhận một chuỗi string để tìm kiếm model của bạn. Sau đó, bạn nên kết hợp thêm phương thức `get` vào truy vấn tìm kiếm để lấy ra các model Eloquent phù hợp với truy vấn tìm kiếm đã cho:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->get();
```

Vì các tìm kiếm Scout trả về một collection của model Eloquent, nên bạn thậm chí có thể trả về kết quả trực tiếp từ một route hoặc một controller và chúng sẽ tự động được chuyển thành dạng JSON:

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/search', function (Request $request) {
    return Order::search($request->search)->get();
});
```

Nếu bạn muốn nhận kết quả search raw trước khi chúng được chuyển đổi thành các model Eloquent, bạn nên sử dụng phương thức `raw`:

```php
$orders = Order::search('Star Trek')->raw();
```

<a name="custom-indexes"></a>
#### Custom Indexes

Khi tìm kiếm bằng các engine của bên thứ ba, các câu lệnh truy vấn tìm kiếm thường sẽ được thực hiện trên index được chỉ định bởi phương thức [searchableAs](#configuring-model-indexes) của model. Tuy nhiên, bạn có thể sử dụng phương thức `within` để chỉ định thêm một index khác sẽ được tìm kiếm:

```php
$orders = Order::search('Star Trek')
    ->within('tv_shows_popularity_desc')
    ->get();
```

<a name="where-clauses"></a>
### Where Clauses

Scout cho phép bạn thêm các điều kiện "where" vào các truy vấn tìm kiếm của bạn. Hiện tại, các điều kiện này chỉ hỗ trợ so sánh cơ bản và chủ yếu sử dụng cho việc query tìm kiếm theo ID.

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

Ngoài ra, phương thức `whereIn` có thể được sử dụng để xác minh các giá trị của một cột nhất định có nằm trong một mảng hay không:

```php
$orders = Order::search('Star Trek')->whereIn(
    'status', ['open', 'paid']
)->get();
```

Phương thức `whereNotIn` sẽ xác minh giá trị của cột đã cho không có trong một mảng đã cho:

```php
$orders = Order::search('Star Trek')->whereNotIn(
    'status', ['closed']
)->get();
```

> [!WARNING]
> Nếu ứng dụng của bạn đang sử dụng Meilisearch, bạn phải cấu hình [các thuộc tính có thể lọc](#meilisearch-index-settings) của ứng dụng trước khi sử dụng lệnh "where" của Scout.

<a name="customizing-the-eloquent-results-query"></a>
#### Customizing the Eloquent Results Query

Sau khi Scout đã lấy danh sách các model Eloquent phù hợp từ công cụ tìm kiếm của ứng dụng, Eloquent sẽ được sử dụng để lấy tất cả các model phù hợp theo khóa chính của chúng. Bạn có thể tùy chỉnh truy vấn này bằng cách gọi thêm phương thức `query`. Phương thức `query` này sẽ chấp nhận một closure sẽ nhận vào instance của Eloquent query builder làm tham số:

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

Khi sử dụng engine của bên thứ ba, callback này sẽ được gọi sau khi các model đã được lấy ra từ công cụ tìm kiếm, vì vậy nó không nên được sử dụng để "lọc" kết quả — thay vào đó hãy sử dụng [mệnh đề where của Scout](#where-clauses). Tuy nhiên, khi sử dụng engine database, các câu lệnh của phương thức `query` sẽ được áp dụng trực tiếp vào truy vấn cơ sở dữ liệu, vì vậy bạn cũng có thể sử dụng nó để lọc.

<a name="pagination"></a>
### Pagination

Ngoài việc lấy ra một collection của model, bạn có thể phân trang kết quả tìm kiếm của bạn bằng phương thức `paginate`. Phương thức này sẽ trả về một instance `Illuminate\Pagination\LengthAwarePaginator` giống như bạn đã đọc ở [tài liệu phân trang truy vấn Eloquent](/docs/{{version}}/pagination) trong các phần trước:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->paginate();
```

Bạn có thể chỉ định số lượng model được trả về trên mỗi trang bằng cách truyền vào số lượng đó làm tham số đầu tiên cho phương thức `paginate`:

```php
$orders = Order::search('Star Trek')->paginate(15);
```

Khi sử dụng engine database, bạn cũng có thể sử dụng phương thức `simplePaginate`. Không giống như `paginate`, vốn lấy tổng số bản ghi để có thể hiển thị số trang, `simplePaginate` chỉ xác định xem có kết quả nào khác nằm ngoài trang hiện tại hay không — điều này giúp nó hiệu quả hơn đối với các tập dữ liệu lớn khi bạn chỉ cần xác định có link "trước" và "sau" hay không:

```php
$orders = Order::search('Star Trek')->simplePaginate(15);
```

Khi bạn đã lấy ra được kết quả, bạn có thể hiển thị kết quả và tạo ra các page links bằng [Blade](/docs/{{version}}/blade) giống như khi bạn thực hiện phân trang truy vấn Eloquent:

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

Tất nhiên, nếu bạn muốn lấy ra kết quả pagination dưới dạng JSON, bạn có thể trả về instance pagination trực tiếp từ một route hoặc một controller:

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    return Order::search($request->input('query'))->paginate(15);
});
```

> [!WARNING]
> Vì các công cụ tìm kiếm không biết về định nghĩa global scope của model Eloquent của bạn, bạn không nên sử dụng global scope trong các ứng dụng mà sử dụng phân trang của Scout. Hoặc, bạn nên tạo lại các ràng buộc của global scope khi tìm kiếm thông qua Scout.

<a name="soft-deleting"></a>
### Soft Deleting

Nếu các model index của bạn là loại có thể [soft deleting](/docs/{{version}}/eloquent#soft-deleting) và bạn cần tìm kiếm các model đã bị soft delete của bạn, hãy set tùy chọn `soft_delete` vào trong file cấu hình `config/scout.php` của bạn thành `true`:

```php
'soft_delete' => true,
```

Khi tùy chọn cấu hình này thành `true`, Scout sẽ không xóa các model đó ra khỏi search index. Thay vào đó, nó sẽ set thuộc tính ẩn `__soft_deleted` trên bản ghi đó. Và sau đó, bạn có thể sử dụng phương thức `withTrashed` hoặc `onlyTrashed` để lấy ra các bản ghi đã soft delete khi tìm kiếm:

```php
use App\Models\Order;

// Include trashed records when retrieving results...
$orders = Order::search('Star Trek')->withTrashed()->get();

// Only include trashed records when retrieving results...
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

> [!NOTE]
> Khi một model đã bị xóa vĩnh viễn bằng cách sử dụng `forceDelete`, Scout sẽ tự động xóa model đó ra khỏi search index.

<a name="customizing-engine-searches"></a>
### Tuỳ chỉnh Engine Search

Nếu bạn cần thực hiện một tùy chỉnh nâng cao cho hành động tìm kiếm của một engine, bạn có thể truyền một lệnh closure làm tham số thứ hai cho phương thức `search`. Ví dụ: bạn có thể sử dụng lệnh closure này để thêm dữ liệu vị trí vào các tùy chọn tìm kiếm trước khi câu lệnh tìm kiếm được truyền đến Algolia:

```php
use Algolia\AlgoliaSearch\SearchIndex;
use App\Models\Order;

Order::search(
    'Star Trek',
    function (SearchIndex $algolia, string $query, array $options) {
        $options['body']['query']['bool']['filter']['geo_distance'] = [
            'distance' => '1000km',
            'location' => ['lat' => 36, 'lon' => 111],
        ];

        return $algolia->search($query, $options);
    }
)->get();
```

<a name="custom-engines"></a>
## Custom Engines

<a name="writing-the-engine"></a>
#### Writing The Engine

Nếu một trong những engine tìm kiếm của Scout không phù hợp với nhu cầu của bạn, bạn có thể viết một engine mới của riêng bạn và đăng ký nó với Scout. Engine của bạn sẽ được extend từ abstract class `Laravel\Scout\Engines\Engine`. Abstract class này chứa tám phương thức mà engine mới của bạn phải implement:

```php
use Laravel\Scout\Builder;

abstract public function update($models);
abstract public function delete($models);
abstract public function search(Builder $builder);
abstract public function paginate(Builder $builder, $perPage, $page);
abstract public function mapIds($results);
abstract public function map(Builder $builder, $results, $model);
abstract public function getTotalCount($results);
abstract public function flush($model);
```

Bạn có thể tham khảo class `Laravel\Scout\Engines\AlgoliaEngine` để biết thêm cách bạn nên implement các phương thức đó như thế nào. Class này sẽ cung cấp cho bạn một điểm khởi đầu tốt để bạn có thể học cách implement từng phương thức này trong engine của riêng bạn.

<a name="registering-the-engine"></a>
#### Registering The Engine

Khi bạn đã viết xong engine mới của bạn, bạn có thể đăng ký nó với Scout bằng phương thức `extend` trong engine manager của Scout. Engine manager của Scout có thể được resolve từ service container của Laravel. Bạn nên gọi phương thức `extend` từ phương thức `boot` của class `App\Providers\AppServiceProvider` hoặc bất kỳ service provider nào khác được application của bạn sử dụng.

```php
use App\ScoutExtensions\MySqlSearchEngine;
use Laravel\Scout\EngineManager;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    resolve(EngineManager::class)->extend('mysql', function () {
        return new MySqlSearchEngine;
    });
}
```

Khi engine của bạn đã được đăng ký, bạn có thể khai báo nó làm Scout `driver` mặc định trong file cấu hình `config/scout.php` của application của bạn:

```php
'driver' => 'mysql',
```
