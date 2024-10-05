# Eloquent: Getting Started

- [Giới thiệu](#introduction)
- [Tạo class model](#generating-model-classes)
- [Quy ước tên Eloquent Model](#eloquent-model-conventions)
    - [Table Names](#table-names)
    - [Primary Keys](#primary-keys)
    - [UUID và ULID Keys](#uuid-and-ulid-keys)
    - [Timestamps](#timestamps)
    - [Database Connections](#database-connections)
    - [Giá trị thuộc tính mặc định](#default-attribute-values)
    - [Cấu hình Eloquent Strictness](#configuring-eloquent-strictness)
- [Lấy ra Model](#retrieving-models)
    - [Collection](#collections)
    - [Phân kết quả](#chunking-results)
    - [Chunking dùng Lazy Collections](#chunking-using-lazy-collections)
    - [Cursors](#cursors)
    - [Advanced Subqueries](#advanced-subqueries)
- [Lấy ra một Model / một thống kê](#retrieving-single-models)
    - [Lấy hoặc tạo model](#retrieving-or-creating-models)
    - [Lấy ra một thống kê](#retrieving-aggregates)
- [Thêm và cập nhật Model](#inserting-and-updating-models)
    - [Thêm](#inserts)
    - [Cập nhật](#updates)
    - [Mass Assignment](#mass-assignment)
    - [Upserts](#upserts)
- [Xoá Model](#deleting-models)
    - [Soft Delete](#soft-deleting)
    - [Query Model Soft Deleted](#querying-soft-deleted-models)
- [Pruning Models](#pruning-models)
- [Replicating Models](#replicating-models)
- [Query Scope](#query-scopes)
    - [Global Scope](#global-scopes)
    - [Local Scope](#local-scopes)
- [So sánh Model](#comparing-models)
- [Event](#events)
    - [Dùng Closures](#events-using-closures)
    - [Observer](#observers)
    - [Tắt event](#muting-events)

<a name="introduction"></a>
## Giới thiệu

Laravel có chứa Eloquent, một mapper object-relational (ORM) giúp tương tác với cơ sở dữ liệu của bạn trở nên thú vị hơn. Khi sử dụng Eloquent, mỗi table cơ sở dữ liệu có một "Model" tương ứng được sử dụng để tương tác với bảng đó. Ngoài việc truy xuất các bản ghi từ bảng cơ sở dữ liệu, thì các model Eloquent còn cho phép bạn thêm, sửa và xóa các bản ghi ra khỏi bảng.

> [!NOTE]
> Trước khi bắt đầu, bạn hãy chắc chắn là đã cấu hình kết nối cơ sở dữ liệu trong file cấu hình `config/database.php` của application của bạn. Để biết thêm thông tin về cách cấu hình cơ sở dữ liệu của bạn, hãy xem [tài liệu cấu hình cơ sở dữ liệu](/docs/{{version}}/database#configuration).

#### Laravel Bootcamp

Nếu bạn mới làm quen với Laravel, vui lòng tham gia [Laravel Bootcamp](https://bootcamp.laravel.com). Laravel Bootcamp sẽ hướng dẫn bạn xây dựng ứng dụng Laravel bằng Eloquent từ những bước đầu tiên. Đó là một cách tuyệt vời để tìm hiểu mọi thứ mà Laravel và Eloquent cung cấp.

<a name="generating-model-classes"></a>
## Tạo class model

Để bắt đầu, bạn hãy tạo một model Eloquent. Các model thường được lưu trong thư mục `app\Models` và extend class `Illuminate\Database\Eloquent\Model`. Bạn có thể sử dụng lệnh `make:model` [Artisan command](/docs/{{version}}/artisan) để tạo một model mới:

```shell
php artisan make:model Flight
```

Nếu bạn muốn tạo cả file [migration cho cơ sở dữ liệu](/docs/{{version}}/migrations) khi bạn tạo model, bạn có thể sử dụng tùy chọn `--migration` hoặc `-m`:

```shell
php artisan make:model Flight --migration
```

Bạn có thể tạo nhiều loại class khác nhau khi tạo model, chẳng hạn như factory, seeder, policy, controller và form request. Ngoài ra, các tùy chọn này cũng có thể được kết hợp với nhau để tạo nhiều class cùng một lúc:

```shell
# Generate a model and a FlightFactory class...
php artisan make:model Flight --factory
php artisan make:model Flight -f

# Generate a model and a FlightSeeder class...
php artisan make:model Flight --seed
php artisan make:model Flight -s

# Generate a model and a FlightController class...
php artisan make:model Flight --controller
php artisan make:model Flight -c

# Generate a model, FlightController resource class, and form request classes...
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR

# Generate a model and a FlightPolicy class...
php artisan make:model Flight --policy

# Generate a model and a migration, factory, seeder, and controller...
php artisan make:model Flight -mfsc

# Shortcut to generate a model, migration, factory, seeder, policy, controller, and form requests...
php artisan make:model Flight --all

# Generate a pivot model...
php artisan make:model Member --pivot
php artisan make:model Member -p
```

<a name="inspecting-models"></a>
#### Inspecting Models

Thỉnh thoảng có thể khó xác định tất cả các thuộc tính và các quan hệ sẵn có của một model chỉ bằng cách đọc lướt qua code của nó. Thay vào đó, bạn hãy thử lệnh Artisan `model:show`, lệnh này sẽ cung cấp một cái nhìn tổng quan về tất cả các thuộc tính và các quan hệ của model:

```shell
php artisan model:show Flight
```

<a name="eloquent-model-conventions"></a>
## Quy ước tên Eloquent Model

Các model được tạo bởi lệnh `make:model` sẽ được lưu trong thư mục `app/Models`. Hãy xem qua một class model cơ bản và thảo luận về một số quy ước chính của Eloquent:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        // ...
    }

<a name="table-names"></a>
### Table Names

Sau khi xem qua ví dụ trên, bạn có thể nhận thấy rằng chúng ta đã không cho Eloquent biết bảng cơ sở dữ liệu nào tương ứng với model `Flight` của chúng ta. Mặc định, "snake case" cộng với tên số nhiều của class sẽ được sử dụng làm tên bảng trừ khi bạn khai báo một tên khác. Vì vậy, trong trường hợp này, Eloquent sẽ giả định rằng: model `Flight` sẽ lưu các bản ghi vào trong bảng `flights`, trong khi model `AirTrafficController` sẽ lưu các bản ghi vào trong bảng `air_traffic_controllers`.

Nếu bảng cơ sở dữ liệu tương ứng của model của bạn không phù hợp với quy ước này, bạn có thể chỉ định tên bảng của model theo cách thủ công bằng cách định nghĩa thuộc tính `table` trên model:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

<a name="primary-keys"></a>
### Primary Keys

Eloquent cũng sẽ giả định rằng mỗi bảng cơ sở dữ liệu tương ứng của model có một cột khóa chính có tên là `id`. Nếu cần thiết, bạn có thể định nghĩa một thuộc tính protected `$primaryKey` trên model của bạn để chỉ định một cột khác đóng vai trò làm khóa chính của model:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The primary key associated with the table.
         *
         * @var string
         */
        protected $primaryKey = 'flight_id';
    }

Ngoài ra, Eloquent cũng giả định rằng khóa chính là một giá trị integer tăng dần đều, có nghĩa là Eloquent sẽ được tự động cast khóa chính thành một số nguyên. Nếu bạn muốn sử dụng khóa chính không tăng dần hoặc không phải dạng integer, thì khoá chính của bạn phải được định nghĩa trong thuộc tính public `$incrementing` trên model của bạn và set nó là `false`:

    <?php

    class Flight extends Model
    {
        /**
         * Indicates if the model's ID is auto-incrementing.
         *
         * @var bool
         */
        public $incrementing = false;
    }

Nếu khóa chính của model của bạn không phải là dạng integer, thì bạn nên định nghĩa một thuộc tính protected `$keyType` trên model của bạn. Thuộc tính này phải có giá trị là `string`:

    <?php

    class Flight extends Model
    {
        /**
         * The data type of the primary key ID.
         *
         * @var string
         */
        protected $keyType = 'string';
    }

<a name="composite-primary-keys"></a>
#### "Composite" Primary Keys

Eloquent yêu cầu mỗi model phải có ít nhất một "ID" nhận dạng duy nhất để có thể làm khóa chính. Các khóa chính "Composite" không được hỗ trợ bởi các model Eloquent. Tuy nhiên, bạn có thể tự do thêm các index, nhiều cột vào các bảng cơ sở dữ liệu của bạn ngoài khóa chính để xác định tính duy nhất của bảng.

<a name="uuid-and-ulid-keys"></a>
### UUID và ULID Keys

Thay vì sử dụng một số tự động tăng làm khóa chính cho model Eloquent, bạn có thể chọn sử dụng UUID thay thế. UUID là một mã định danh chữ và số duy nhất trên toàn cầu có độ dài 36 ký tự.

Nếu bạn muốn một model sử dụng khóa UUID thay vì khóa số nguyên tự động tăng, bạn có thể sử dụng trait `Illuminate\Database\Eloquent\Concerns\HasUuids` trên model. Tất nhiên, bạn nên đảm bảo rằng model có [một cột khóa chính tương ứng với UUID](/docs/{{version}}/migrations#column-method-uuid):

    use Illuminate\Database\Eloquent\Concerns\HasUuids;
    use Illuminate\Database\Eloquent\Model;

    class Article extends Model
    {
        use HasUuids;

        // ...
    }

    $article = Article::create(['title' => 'Traveling to Europe']);

    $article->id; // "8f8e8478-9035-4d23-b9a7-62f4d2612ce5"

Mặc định, trait `HasUuids` sẽ tạo ra [UUID "có thể sắp xếp"](/docs/{{version}}/strings#method-str-ordered-uuid) cho model của bạn. Các UUID này hiệu quả cho việc lưu trữ index trong cơ sở dữ liệu vì chúng có thể được sắp xếp theo kiểu từ điển.

Bạn có thể ghi đè process tạo UUID cho một model nhất định bằng cách định nghĩa một phương thức `newUniqueId` trên model. Ngoài ra, bạn có thể chỉ định cột nào đó sẽ nhận UUID bằng cách định nghĩa phương thức `uniqueIds` trên model:

    use Ramsey\Uuid\Uuid;

    /**
     * Generate a new UUID for the model.
     */
    public function newUniqueId(): string
    {
        return (string) Uuid::uuid4();
    }

    /**
     * Get the columns that should receive a unique identifier.
     *
     * @return array<int, string>
     */
    public function uniqueIds(): array
    {
        return ['id', 'discount_code'];
    }

Nếu muốn, bạn có thể chọn sử dụng "ULIDs" thay vì dùng UUID. ULID tương tự như UUID; tuy nhiên, chúng chỉ có độ dài 26 ký tự. Giống như các UUID có thể được sắp xếp, các ULID có thể được sắp xếp theo kiểu từ điển để lập index cho cơ sở dữ liệu một cách hiệu quả hơn. Để sử dụng ULID, bạn nên sử dụng trait `Illuminate\Database\Eloquent\Concerns\HasUlids` trên model của bạn. Bạn cũng nên đảm bảo rằng model có [cột khóa chính tương ứng ULID](/docs/{{version}}/migrations#column-method-ulid):

    use Illuminate\Database\Eloquent\Concerns\HasUlids;
    use Illuminate\Database\Eloquent\Model;

    class Article extends Model
    {
        use HasUlids;

        // ...
    }

    $article = Article::create(['title' => 'Traveling to Asia']);

    $article->id; // "01gd4d3tgrrfqeda94gdbtdk5c"

<a name="timestamps"></a>
### Timestamps

Mặc định, Eloquent mong đợi các cột `created_at` và `updated_at` tồn tại trong bảng cơ sở dữ liệu tương ứng với model của bạn. Eloquent sẽ tự động set các giá trị của các cột này khi các model được tạo hoặc cập nhật. Nếu bạn không muốn Eloquent tự động quản lý các cột này, bạn nên định nghĩa thuộc tính `$timestamps` trong model của bạn với giá trị `false`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }

Nếu bạn cần tùy biến định dạng timestamp của model của bạn, hãy định nghĩa thuộc tính `$dateFormat` trong model của bạn. Thuộc tính này sẽ định nghĩa cách mà các thuộc tính date được lưu vào trong cơ sở dữ liệu cũng như định dạng của chúng khi model được chuyển đổi thành các loại dữ liệu như một mảng hoặc một dạng JSON:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

Nếu bạn cần tùy biến tên của các cột được sử dụng để lưu timestamp, bạn có thể định nghĩa các hằng số `CREATED_AT` và `UPDATED_AT` trên model của bạn:

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'updated_date';
    }

Nếu bạn muốn thực hiện các thao tác trên model mà không cần sửa timestamp `updated_at` của model, bạn có thể thao tác trên model trong một closure được cung cấp trong phương thức `withoutTimestamps`:

    Model::withoutTimestamps(fn () => $post->increment(['reads']));

<a name="database-connections"></a>
### Database Connection

Mặc định, tất cả các model Eloquent sẽ sử dụng kết nối cơ sở dữ liệu mặc định đã được cấu hình trong application của bạn. Nếu bạn muốn khai báo một kết nối khác sẽ được sử dụng khi tương tác với một model cụ thể, thì bạn nên định nghĩa thuộc tính `$connection` trên model:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The database connection that should be used by the model.
         *
         * @var string
         */
        protected $connection = 'sqlite';
    }

<a name="default-attribute-values"></a>
### Giá trị thuộc tính mặc định

Mặc định, một instance model mới khi được tạo sẽ không chứa bất kỳ giá trị thuộc tính nào. Nếu bạn muốn định nghĩa giá trị mặc định cho một số thuộc tính của model, bạn có thể định nghĩa thuộc tính `$attributes` trên model của bạn. Các giá trị thuộc tính được set trong mảng `$attributes` phải ở định dạng raw, "lưu trữ được" giống như chúng vừa được đọc ra từ cơ sở dữ liệu:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The model's default values for attributes.
         *
         * @var array
         */
        protected $attributes = [
            'options' => '[]',
            'delayed' => false,
        ];
    }

<a name="configuring-eloquent-strictness"></a>
### Cấu hình Eloquent Strictness

Laravel cung cấp một số phương thức cho phép bạn cấu hình hành vi và "sự nghiêm ngặt" của Eloquent trong nhiều tình huống khác nhau.

Đầu tiên, phương thức `preventLazyLoading` sẽ chấp nhận một tham số boolean tùy chọn để cho biết xem liệu có nên chặn việc lazy loading hay không. Ví dụ: bạn có thể muốn tắt lazy loading trong môi trường không phải production để môi trường production của bạn có thể tiếp tục hoạt động bình thường ngay cả khi một lazy load quan hệ vô tình xuất hiện trong code production. Thông thường, phương thức này nên được gọi trong phương thức `boot` của `AppServiceProvider` trong ứng dụng của bạn:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

Ngoài ra, bạn có thể hướng dẫn Laravel đưa ra một ngoại lệ khi cố gắng đưa vào một thuộc tính không thể đưa bằng cách gọi phương thức `preventSilentlyDiscardingAttributes`. Điều này có thể giúp ngăn ngừa các lỗi không mong muốn trong quá trình phát triển ở local khi cố gắng set một thuộc tính mà chưa được thêm vào trong mảng `fillable` của model:

```php
Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```

<a name="retrieving-models"></a>
## Lấy ra Model

Khi bạn đã tạo một model và [bảng cơ sở dữ liệu được liên kết với model đó](/docs/{{version}}/migrations#writing-migrations), bạn có thể bắt đầu truy xuất dữ liệu từ cơ sở dữ liệu của bạn. Bạn có thể nghĩ về mỗi model Eloquent như là một [query builder](/docs/{{version}}/queries) cho phép bạn truy vấn vào bảng cơ sở dữ liệu được liên kết với model đó. Phương thức `all` của model sẽ lấy ra tất cả các bản ghi từ bảng cơ sở dữ liệu được liên kết của model:

    use App\Models\Flight;

    foreach (Flight::all() as $flight) {
        echo $flight->name;
    }

<a name="building-queries"></a>
#### Building Queries

Phương thức `all` của Eloquent sẽ trả về tất cả các bản ghi có trong bảng của model. Tuy nhiên, vì mỗi model Eloquent đóng vai trò là một [query builder](/docs/{{version}}/queries), nên bạn có thể thêm các ràng buộc bổ sung cho các truy vấn của bạn, và sau đó gọi phương thức `get` để lấy ra kết quả:

    $flights = Flight::where('active', 1)
                   ->orderBy('name')
                   ->take(10)
                   ->get();

> [!NOTE]
> Vì các model Eloquent là các query builder, nên bạn nên xem lại tất cả các phương thức đã được cung cấp bởi [query builder](/docs/{{version}}/queries) của Laravel. Bạn có thể sử dụng bất kỳ phương thức nào khi viết các truy vấn Eloquent của bạn.

<a name="refreshing-models"></a>
#### Refreshing Models

Nếu bạn đã có một instance của model Eloquent được lấy từ cơ sở dữ liệu, bạn có thể "refresh" các model bằng cách sử dụng các phương thức `fresh` và `refresh`. Phương thức `fresh` sẽ lấy ra một model mới từ cơ sở dữ liệu. Instance model hiện tại sẽ không bị ảnh hưởng:

    $flight = Flight::where('number', 'FR 900')->first();

    $freshFlight = $flight->fresh();

Phương thức `refresh` sẽ tái tạo lại model hiện tại bằng cách sử dụng dữ liệu mới từ cơ sở dữ liệu. Ngoài ra, tất cả các quan hệ đã được load cũng sẽ bị refresh:

    $flight = Flight::where('number', 'FR 900')->first();

    $flight->number = 'FR 456';

    $flight->refresh();

    $flight->number; // "FR 900"

<a name="collections"></a>
### Collection

Như chúng ta đã thấy, các phương thức Eloquent như `all` và `get` sẽ lấy nhiều record từ cơ sở dữ liệu. Tuy nhiên, các phương thức này không trả về một mảng PHP. Thay vì, một instance của `Illuminate\Database\Eloquent\Collection` sẽ được trả về.

Class Eloquent `Collection` được mở rộng từ class `Illuminate\Support\Collection` của Laravel, trong đó cung cấp [nhiều phương thức hữu ích](/docs/{{version}}/eloquent-collections#available-methods) để tương tác với các dữ liệu collection. Ví dụ: phương thức `reject` có thể được sử dụng để xóa các model ra khỏi collection dựa trên kết quả của một closure được gọi:

```php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function (Flight $flight) {
    return $flight->cancelled;
});
```

Ngoài các phương thức được cung cấp bởi class collection base của Laravel, class collection Eloquent cũng cung cấp [một vài phương thức bổ sung](/docs/{{version}}/eloquent-collections#available-methods) được dành riêng để tương tác với các collection của những Eloquent model.

Vì tất cả các collection của Laravel đều implement các interface iterable của PHP, nên bạn có thể lặp qua các collection như thể chúng là một mảng:

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

<a name="chunking-results"></a>
### Phân kết quả

Ứng dụng của bạn có thể hết memory nếu bạn cố load hàng chục nghìn record Eloquent thông qua các phương thức `all` hoặc `get`. Thay vì sử dụng các phương thức này, phương thức `chunk` có thể được sử dụng để xử lý số lượng lớn model một cách hiệu quả hơn.

Phương thức `chunk` sẽ lấy ra một số ít các model Eloquent, và truyền chúng đến một closure để xử lý. Vì mỗi lần chỉ lấy ra chunk hiện tại của các model Eloquent, nên phương thức `chunk` sẽ giúp giảm đáng kể mức sử dụng memory khi làm việc với một số lượng lớn model:

```php
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;

Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```

Tham số đầu tiên được truyền cho phương thức `chunk` là số lượng bản ghi mà bạn muốn nhận vào cho mỗi đoạn "chunk". Closure sẽ được truyền thông qua tham số thứ hai, và nó sẽ được gọi cho mỗi đoạn được lấy từ cơ sở dữ liệu. Các truy vấn cơ sở dữ liệu sẽ được thực hiện để truy xuất từng đoạn của bản ghi và được chuyển vào cho closure.

Nếu bạn đang lọc kết quả từ phương thức `chunk` dựa trên một cột mà cột đó bạn cũng sẽ cập nhật trong khi lặp lại kết quả, thì bạn nên sử dụng phương thức `chunkById`. Việc sử dụng phương thức `chunk` trong các trường hợp này có thể dẫn đến kết quả không mong muốn và không nhất quán. Phương thức `chunkById` sẽ luôn lấy ra các model có cột `id` lớn hơn model cuối cùng trong đoạn chunk trước:

```php
Flight::where('departed', true)
    ->chunkById(200, function (Collection $flights) {
        $flights->each->update(['departed' => false]);
    }, $column = 'id');
```

<a name="chunking-using-lazy-collections"></a>
### Chunking dùng Lazy Collections

Phương thức `lazy` hoạt động tương tự như [phương thức `chunk`](#chunking-results) theo nghĩa là, nó cũng thực thi truy vấn theo chunk. Tuy nhiên, thay vì truyền từng chunk trực tiếp vào một hàm callback như hiện tại, thì phương thức `lazy` trả về [`LazyCollection`](/docs/{{version}}/collections#lazy-collections) của các model Eloquent, cho phép bạn tương tác với các kết quả dưới dạng một luồng duy nhất:

```php
use App\Models\Flight;

foreach (Flight::lazy() as $flight) {
    // ...
}
```

Nếu bạn đang lọc kết quả của phương thức `lazy` dựa trên một cột mà bạn cũng sẽ cập nhật cột đó trong khi lặp, thì bạn nên sử dụng phương thức `lazyById`. Phương thức `lazyById` sẽ luôn lấy ra các model có cột `id` lớn hơn model cuối cùng trong đoạn chunk trước:

```php
Flight::where('departed', true)
    ->lazyById(200, $column = 'id')
    ->each->update(['departed' => false]);
```

Bạn có thể lọc kết quả dựa trên thứ tự giảm dần của `id` bằng cách sử dụng phương thức `lazyByIdDesc`.

<a name="cursors"></a>
### Cursors

Tương tự như phương thức `lazy`, phương thức `cursor` cũng có thể được sử dụng để giảm đáng kể mức tiêu thụ bộ nhớ của ứng dụng khi lặp qua hàng chục nghìn bản ghi model Eloquent.

Phương thức `cursor` sẽ chỉ thực hiện một truy vấn vào cơ sở dữ liệu; tuy nhiên, các model Eloquent sẽ không được cung cấp bộ nhớ cho đến khi chúng thực sự được lặp qua. Do đó, chỉ có một model Eloquent được lưu trong bộ nhớ tại bất kỳ thời điểm nào trong khi lặp.

> [!WARNING]
> Vì phương thức `cursor` sẽ chỉ lưu một model Eloquent trong bộ nhớ tại một thời điểm nên nó không thể eager load các quan hệ. Nếu bạn cần eager load các quan hệ, hãy cân nhắc sử dụng [phương pháp `lazy`](#chunking-using-lazy-collections) để thay thế.

Phương thức `cursor` sử dụng PHP [generators](https://www.php.net/manual/en/lingu.generators.overview.php) để implement chức năng này:

```php
use App\Models\Flight;

foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    // ...
}
```

Phương thức `cursor` sẽ trả về một instance `Illuminate\Support\LazyCollection`. [Lazy collections](/docs/{{version}}/collections#lazy-collections) cho phép bạn sử dụng nhiều phương thức có sẵn trên các collection Laravel cở bản trong khi chỉ load một model duy nhất vào bộ nhớ tại một thời điểm:

```php
use App\Models\User;

$users = User::cursor()->filter(function (User $user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

Mặc dù phương thức `cursor` sử dụng ít bộ nhớ hơn nhiều so với truy vấn thông thường (bằng cách chỉ giữ một model Eloquent duy nhất trong bộ nhớ tại một thời điểm), nhưng cuối cùng nó vẫn sẽ hết bộ nhớ. Điều này là [do driver PDO của PHP sẽ lưu nội bộ tất cả các kết quả truy vấn trong bộ cache của nó](https://www.php.net/manual/en/mysqlinfo.concepts.buffering.php). Nếu bạn đang xử lý một số lượng rất lớn các bản ghi Eloquent, hãy cân nhắc sử dụng [phương thức `lazy`](#chunking-using-lazy-collections) để thay thế.

<a name="advanced-subqueries"></a>
### Advanced Subqueries

<a name="subquery-selects"></a>
#### Subquery Selects

Eloquent cũng cung cấp hỗ trợ các truy vấn con nâng cao, cho phép bạn lấy thông tin từ các bảng liên quan trong một truy vấn duy nhất. Ví dụ, hãy tưởng tượng rằng chúng ta có một bảng các chuyến bay `destinations` và một bảng `flights` đến các destination. Bảng `flights` chứa một cột `arrived_at` cho biết thời điểm chuyến bay đến destination.

Sử dụng chức năng truy vấn phụ có trong phương thức `select` và `addSelect` của query builder, chúng ta có thể lấy ra tất cả các `destinations` và tên của chuyến bay đã đến điểm đến đó gần đây nhất chỉ bằng một câu lệnh truy vấn duy nhất:

    use App\Models\Destination;
    use App\Models\Flight;

    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
    ])->get();

<a name="subquery-ordering"></a>
#### Subquery Ordering

Ngoài ra, hàm `orderBy` của query builder cũng hỗ trợ các truy vấn con. Tiếp tục sử dụng ví dụ trên, chúng ta có thể sử dụng chức năng này để sắp xếp tất cả các điểm đến dựa trên thời điểm chuyến bay cuối cùng đến điểm đến đó. Một lần nữa, điều này có thể được thực hiện chỉ trong một truy vấn cơ sở dữ liệu:

    return Destination::orderByDesc(
        Flight::select('arrived_at')
            ->whereColumn('destination_id', 'destinations.id')
            ->orderByDesc('arrived_at')
            ->limit(1)
    )->get();

<a name="retrieving-single-models"></a>
## Lấy ra một Model / một thống kê

Ngoài việc truy xuất tất cả các bản ghi phù hợp với một truy vấn, bạn cũng có thể truy xuất một bản ghi bằng cách sử dụng phương thức `find`, `first`, hoặc `firstWhere`. Thay vì trả về một tập hợp các model, thì các phương thức này sẽ trả về một instance model duy nhất:

     use App\Models\Flight;

    // Retrieve a model by its primary key...
    $flight = Flight::find(1);

    // Retrieve the first model matching the query constraints...
    $flight = Flight::where('active', 1)->first();

    // Alternative to retrieving the first model matching the query constraints...
    $flight = Flight::firstWhere('active', 1);

Thỉnh thoảng, bạn có thể muốn thực hiện một số hành động khác nếu không tìm thấy kết quả nào. Phương thức `findOr` và phương thức `firstOr` sẽ trả về một instance model hoặc nếu không tìm thấy kết quả nào khác, thì sẽ thực hiện một closure đã cho. Giá trị được trả về bởi closure sẽ được coi là kết quả của phương thức:

    $flight = Flight::findOr(1, function () {
        // ...
    });

    $flight = Flight::where('legs', '>', 3)->firstOr(function () {
        // ...
    });

<a name="not-found-exceptions"></a>
#### Not Found Exceptions

Thỉnh thoảng bạn có thể muốn đưa ra một ngoại lệ nếu không tìm thấy model. Điều này đặc biệt hữu ích trong các route hoặc controller. Các phương thức `findOrFail` và `firstOrFail` sẽ lấy ra kết quả đầu tiên của truy vấn; tuy nhiên, nếu không tìm thấy kết quả nào, một `Illuminate\Database\Eloquent\ModelNotFoundException` sẽ được đưa ra:

    $flight = Flight::findOrFail(1);

    $flight = Flight::where('legs', '>', 3)->firstOrFail();

Nếu ngoại lệ `ModelNotFoundException` không được xử lý, một HTTP response `404` sẽ tự động được gửi về cho người dùng:

    use App\Models\Flight;

    Route::get('/api/flights/{id}', function (string $id) {
        return Flight::findOrFail($id);
    });

<a name="retrieving-or-creating-models"></a>
### Lấy hoặc tạo model

Phương thức `firstOrCreate` sẽ thử lấy một bản ghi cơ sở dữ liệu bằng cách sử dụng mảng cột và giá trị đã cho. Nếu không thể tìm thấy model trong cơ sở dữ liệu, một record mới sẽ được thêm vào với thuộc tính là từ mảng của tham số thứ nhất cộng với tham số thứ hai:

Phương thức `firstOrNew` cũng giống như phương thức `firstOrCreate` là cũng sẽ cố gắng thử xác định một bản ghi có trong cơ sở dữ liệu có khớp với các thuộc tính đã cho hay không. Tuy nhiên, nếu không tìm thấy model, một instance model mới sẽ được trả về. Lưu ý rằng model được trả về bởi `firstOrNew` vẫn chưa được lưu vào trong cơ sở dữ liệu. Bạn sẽ cần phải gọi phương thức `save` để lưu nó:

    use App\Models\Flight;

    // Retrieve flight by name or create it if it doesn't exist...
    $flight = Flight::firstOrCreate([
        'name' => 'London to Paris'
    ]);

    // Retrieve flight by name or create it with the name, delayed, and arrival_time attributes...
    $flight = Flight::firstOrCreate(
        ['name' => 'London to Paris'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

    // Retrieve flight by name or instantiate a new Flight instance...
    $flight = Flight::firstOrNew([
        'name' => 'London to Paris'
    ]);

    // Retrieve flight by name or instantiate with the name, delayed, and arrival_time attributes...
    $flight = Flight::firstOrNew(
        ['name' => 'Tokyo to Sydney'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

<a name="retrieving-aggregates"></a>
### Lấy ra một thống kê

Khi tương tác với các model Eloquent, bạn cũng có thể sử dụng các phương thức `count`, `sum`, `max`, và [các phương thức thống kê](/docs/{{version}}/queries#aggregates) khác được cung cấp bởi [query builder](/docs/{{version}}/queries) của Laravel. Như bạn có thể cảm thấy, các phương thức này trả về giá trị thay vì một instance model Eloquent:

    $count = Flight::where('active', 1)->count();

    $max = Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Thêm và cập nhật Model

<a name="inserts"></a>
### Thêm

Tất nhiên, khi sử dụng Eloquent, chúng ta không chỉ cần lấy các model từ cơ sở dữ liệu. Chúng ta cũng cần thêm các bản ghi mới. Rất may, Eloquent làm cho nó đơn giản. Để thêm một bản ghi mới vào cơ sở dữ liệu, bạn nên khởi tạo một instance model mới và set các thuộc tính cho model đó. Sau đó, gọi phương thức `save` trên instance model đó:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Flight;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class FlightController extends Controller
    {
        /**
         * Store a new flight in the database.
         */
        public function store(Request $request): RedirectResponse
        {
            // Validate the request...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();

            return redirect('/flights');
        }
    }

Trong ví dụ này, chúng ta đã gán field `name` trong HTTP request cho một thuộc tính `name` của instance model `App\Flight`. Khi chúng ta gọi phương thức `save`, một bản ghi sẽ được thêm vào cơ sở dữ liệu. Các timestamp `created_at` và `update_at` của model cũng sẽ tự động được set khi gọi phương thức `save`, do đó bạn không cần phải set chúng.

Ngoài ra, bạn có thể sử dụng phương thức `create` để "lưu" một model mới bằng cách sử dụng một câu lệnh PHP. Instance model được thêmn vào sẽ được trả về cho bạn bằng phương thức `create`:

    use App\Models\Flight;

    $flight = Flight::create([
        'name' => 'London to Paris',
    ]);

Tuy nhiên, trước khi làm như vậy, bạn sẽ cần phải khai báo thuộc tính `fillable` hoặc `guarded` trên model, vì mặc định, tất cả các model Eloquent đều được bảo vệ để chống lại việc mass-assignment. Để tìm hiểu thêm về mass assignment, vui lòng tham khảo [tài liệu mass assignment](#mass-assignment).

<a name="updates"></a>
### Cập nhật

Phương thức `save` cũng có thể được sử dụng để cập nhật một model đã tồn tại trong cơ sở dữ liệu. Để cập nhật một model, bạn nên truy xuất nó ra và set lại bất kỳ các thuộc tính nào mà bạn muốn cập nhật. Then, you should gọi phương thức `save` của model. Timestamp `update_at` sẽ tự động được cập nhật lại theo nó, do đó bạn không cần phải tự phải set lại giá trị cho nó:

    use App\Models\Flight;

    $flight = Flight::find(1);

    $flight->name = 'Paris to London';

    $flight->save();

<a name="mass-updates"></a>
#### Mass Updates

Cập nhật cũng có thể được thực hiện đối với các model tương ứng với một câu lệnh truy vấn duy nhất. Trong ví dụ này, tất cả các flight `đang hoạt động` và có `điểm đến` là `San Diego` sẽ bị đánh dấu là delay:

    Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);

Phương thức `update` yêu cầu một mảng gồm các cặp: tên cột và giá trị cần được cập nhật. Phương thức `update` này sẽ trả về số hàng bị ảnh hưởng.

> [!WARNING]
> Khi chạy một mass update thông qua Eloquent, thì các event của model như `saving`, `saved`, `updating`, và `updated` sẽ không được kích hoạt. Điều này là do các model đã không được lấy ra khi chạy một mass update.

<a name="examining-attribute-changes"></a>
#### Examining Attribute Changes

Eloquent cung cấp các phương thức `isDirty`, `isClean` và `wasChanged` để kiểm tra xem trạng thái của model của bạn và xác định các thuộc tính của model đã bị thay đổi như thế nào so với khi model được lấy ra.

Phương thức `isDirty` sẽ xác định xem có bất kỳ thuộc tính nào bị thay đổi kể từ khi model được lấy ra hay không. Bạn có thể truyền vào tên một thuộc tính cụ thể hoặc một mảng các thuộc tính vào phương thức `isDirty` để xác định xem có thuộc tính nào bị "thay đổi" hay không. Phương thức `isClean` sẽ xác định xem một thuộc tính có thay đổi không kể từ khi model được lấy ra. Phương thức này cũng chấp nhận một tùy chọn tham số cho tên thuộc tính:

    use App\Models\User;

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';

    $user->isDirty(); // true
    $user->isDirty('title'); // true
    $user->isDirty('first_name'); // false
    $user->isDirty(['first_name', 'title']); // true

    $user->isClean(); // false
    $user->isClean('title'); // false
    $user->isClean('first_name'); // true
    $user->isClean(['first_name', 'title']); // false

    $user->save();

    $user->isDirty(); // false
    $user->isClean(); // true

Phương thức `wasChanged` sẽ xác định xem đã có bất kỳ thuộc tính nào bị thay đổi khi model được lưu vào lần cuối trong request hiện tại hay không. Nếu cần, bạn có thể truyền tên một thuộc tính để xem liệu thuộc tính đó có bị thay đổi hay không:

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';

    $user->save();

    $user->wasChanged(); // true
    $user->wasChanged('title'); // true
    $user->wasChanged(['title', 'slug']); // true
    $user->wasChanged('first_name'); // false
    $user->wasChanged(['first_name', 'title']); // true

Phương thức `getOriginal` sẽ trả về một mảng chứa các thuộc tính ban đầu của model bất kể có thay đổi nào kể từ khi nó được lấy ra. Nếu cần, bạn có thể truyền vào tên của một thuộc tính cụ thể để nhận về giá trị ban đầu của một thuộc tính đó:

    $user = User::find(1);

    $user->name; // John
    $user->email; // john@example.com

    $user->name = "Jack";
    $user->name; // Jack

    $user->getOriginal('name'); // John
    $user->getOriginal(); // Array of original attributes...

<a name="mass-assignment"></a>
### Mass Assignment

Bạn có thể sử dụng phương thức `create` để "lưu" một model mới với một câu lệnh PHP duy nhất. Model được thêm vào và sẽ được trả về cho bạn bằng phương thức đó.

    use App\Models\Flight;

    $flight = Flight::create([
        'name' => 'London to Paris',
    ]);

Tuy nhiên, trước khi làm như vậy, bạn sẽ cần phải khai báo thuộc tính `fillable` hoặc `guarded` trên model, vì mặc định, tất cả các model Eloquent đều được bảo vệ để chống lại việc mass-assignment.

Lỗ hổng mass assignment xảy ra khi người dùng truyền một field HTTP request và các field này thay đổi giá trị một cột trong cơ sở dữ liệu của bạn mà bạn không mong muốn. Ví dụ: kẻ xấu có thể gửi một tham số `is_admin` thông qua một request HTTP, sau đó được truyền đến phương thức `create` trong model của bạn, cho phép người đó có thể nâng quyền lên quyền admin.

Vì vậy, để bắt đầu, bạn nên định nghĩa các thuộc tính  mà bạn muốn mass assignable. Bạn có thể làm điều này bằng cách sử dụng thuộc tính `$fillable` trên model. Ví dụ: hãy tạo thuộc tính `name` trong model `Flight` có thể được sử dụng để mass assignable:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Khi you have specified which các thuộc tính có thể được sử dụng để mass assignable, thì bạn có thể sử dụng phương thức `create` để thêm một bản ghi mới vào trong cơ sở dữ liệu. Phương thức `create` sẽ trả về instance model mà đã được tạo mới:

    $flight = Flight::create(['name' => 'London to Paris']);

Nếu bạn đã có một instance model, bạn có thể sử dụng phương thức `fill` để thêm vào model một mảng các thuộc tính:

    $flight->fill(['name' => 'Amsterdam to Frankfurt']);

<a name="mass-assignment-json-columns"></a>
#### Mass Assignment và JSON Columns

Khi gán các cột JSON, mỗi khóa mass assignable của cột đó phải được chỉ định trong mảng `$fillable` trong model của bạn. Để bảo mật, Laravel không hỗ trợ cập nhật các thuộc tính JSON lồng nhau khi sử dụng thuộc tính `guarded`:

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'options->enabled',
    ];

<a name="allowing-mass-assignment"></a>
#### Allowing Mass Assignment

Nếu bạn muốn làm cho tất cả các thuộc tính của bạn đều có thể được sử dụng mass assignable, bạn có thể định nghĩa thuộc tính `$guarded` trong model của bạn là một mảng trống. Nếu bạn chọn không dùng guard model, bạn nên đặc biệt cẩn thận với các mảng được truyền cho các phương thức `fill`, `create` và `update` của Eloquent:

    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = [];

<a name="mass-assignment-exceptions"></a>
#### Mass Assignment Exceptions

Mặc định, các thuộc tính không có trong mảng `$fillable` sẽ bị loại ra khi thực hiện các thao tác mass-assignment. Trong môi trường production, đây là hành vi được mong đợi; tuy nhiên, trong quá trình phát triển ở local, điều này có thể dẫn đến sự nhầm lẫn về lý do tại sao có những thay đổi về mặt model lại không có hiệu lực.

Nếu muốn, bạn có thể hướng dẫn Laravel đưa ra một ngoại lệ khi cố gắng đưa vào một thuộc tính không thể đưa bằng cách gọi phương thức `preventSilentlyDiscardingAttributes`. Thông thường, phương thức này nên được gọi trong một phương thức `boot` của một service provider trong ứng dụng của bạn:

    use Illuminate\Database\Eloquent\Model;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Model::preventSilentlyDiscardingAttributes($this->app->isLocal());
    }

<a name="upserts"></a>
### Upserts

Đôi khi, bạn có thể cần cập nhật model hiện có hoặc tạo một model mới nếu không có model đó. Giống như phương thức `firstOrCreate`, phương thức `updateOrCreate` sẽ lưu model luôn, mà không cần gọi phương thức `save` theo cách thủ công.

Trong ví dụ bên dưới, nếu một chuyến bay có một vị trí `khởi hành` là `Oakland` và vị trí `đến` là `San Diego`, thì các cột `price` và `discounted` của chuyến bay đó sẽ được cập nhật. Nếu như không có chuyến bay nào tồn tại, thì một chuyến bay mới sẽ được tạo và có các thuộc tính từ mảng tham số thứ nhất cùng với mảng tham số thứ hai:

    $flight = Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99, 'discounted' => 1]
    );

Nếu bạn muốn thực hiện nhiều "uperts" trong một truy vấn, thì bạn nên sử dụng phương thức `upsert` để thay thế. Tham số đầu tiên của phương thức sẽ chứa các giá trị để thêm hoặc cập nhật, trong khi tham số thứ hai là liệt kê (các) cột để xác định tính duy nhất của các bản ghi trong bảng cơ sở dữ liệu. Tham số thứ ba và cũng là tham số cuối cùng của phương thức là một mảng các cột sẽ được cập nhật nếu một bản ghi phù hợp đã tồn tại trong cơ sở dữ liệu. Phương thức `upsert` sẽ tự động set timestamp cho cột `created_at` và `updated_at` nếu timestamp được enabled trên model:

    Flight::upsert([
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ], ['departure', 'destination'], ['price']);

> [!WARNING]
> Tất cả các cơ sở dữ liệu ngoại trừ SQL Server đều yêu cầu các cột trong tham số thứ hai của phương thức `upsert` phải có một cột "primary" hoặc một "unique" index trong đó. Ngoài ra, driver cơ sở dữ liệu MySQL sẽ bỏ qua tham số thứ hai của phương thức `upsert` và luôn sử dụng các "primary" và "unique" indexe của bảng để phát hiện các bản ghi hiện có.

<a name="deleting-models"></a>
## Xoá Model

Để xóa một model, bạn có thể gọi phương thức `delete` trên instance model đó:

    use App\Models\Flight;

    $flight = Flight::find(1);

    $flight->delete();

Bạn có thể gọi phương thức `truncate` để xóa tất cả các bản ghi trong cơ sở dữ liệu được liên kết của model. Thao tác `truncate` này cũng sẽ set lại bất kỳ ID tự động tăng nào có trong bảng được liên kết của model:

    Flight::truncate();

<a name="deleting-an-existing-model-by-its-primary-key"></a>
#### Deleting An Existing Model By Its Primary Key

Trong ví dụ trên, chúng ta đang lấy một model từ cơ sở dữ liệu trước khi gọi phương thức `delete`. Tuy nhiên, nếu bạn biết khóa chính của model, bạn có thể xóa trực tiếp model này mà không cần phải truy xuất nó ra bằng cách gọi phương thức `destroy`. Ngoài một khóa chính làm tham số của nó ra, phương thức `destroy` cũng sẽ chấp nhận nhiều khóa chính cùng một lúc như một mảng khóa chính hoặc một [collection](/docs/{{version}}/collections) khóa chính:

    Flight::destroy(1);

    Flight::destroy(1, 2, 3);

    Flight::destroy([1, 2, 3]);

    Flight::destroy(collect([1, 2, 3]));

> [!WARNING]
> Phương thức `destroy` sẽ load từng model và gọi phương thức `delete` trên từng model đó để kích hoạt các event `deleting` và `deleted`.

<a name="deleting-models-using-queries"></a>
#### Deleting Models Using Queries

Bạn cũng có thể chạy một câu lệnh xóa trên một tập các model. Trong ví dụ này, chúng ta sẽ xóa tất cả các flight có đánh dấu là không hoạt động. Giống như mass update, mass delete cũng sẽ không kích hoạt bất kỳ event nào của model khi các model bị xóa:

    $deleted = Flight::where('active', 0)->delete();

> [!WARNING]
> Khi thực hiện câu lệnh mass delete thông qua Eloquent, các event model như là `deleting` và `deleted` sẽ không được kích hoạt cho các model đã bị xóa. Điều này là do các model đã không được lấy ra khi thực hiện câu lệnh xóa.

<a name="soft-deleting"></a>
### Soft Delete

Ngoài việc xóa các bản ghi ra khỏi cơ sở dữ liệu của bạn, Eloquent cũng có thể sử dụng "soft delete" cho các model. Khi các model bị soft delete, thì chúng sẽ không thực sự bị xóa ra khỏi cơ sở dữ liệu của bạn. Thay vào đó, một thuộc tính `deleted_at` sẽ được set vào model cho biết ngày và giờ model bị "xóa". Để kích hoạt soft delete cho một model, hãy thêm trait `Illuminate\Database\Eloquent\SoftDeletes` trên model:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;
    }

> [!NOTE]
> Trait `SoftDeletes` sẽ tự động cast thuộc tính `deleted_at` thành một instance `DateTime` / `Carbon` cho bạn.

Bạn cũng cần thêm cột `deleted_at` vào bảng cơ sở dữ liệu của bạn. [Schema builder](/docs/{{version}}/migrations) của Laravel có chứa một phương thức helper để tạo cột này:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('flights', function (Blueprint $table) {
        $table->softDeletes();
    });

    Schema::table('flights', function (Blueprint $table) {
        $table->dropSoftDeletes();
    });

Bây giờ, khi bạn gọi phương thức `delete` trên model, cột `deleted_at` sẽ được set thành ngày giờ của hiện tại. Tuy nhiên, bản ghi cơ sở dữ liệu của model sẽ được để lại trong bảng. Khi truy vấn một model mà có sử dụng soft delete, thì các model mà đã bị soft delete thì sẽ bị tự động loại khỏi ra tất cả các kết quả truy vấn.

Để xác định xem một instance model đã cho có bị soft delete hay chưa, Bạn có thể sử dụng phương thức `trashed`:

    if ($flight->trashed()) {
        // ...
    }

<a name="restoring-soft-deleted-models"></a>
#### Restoring Soft Deleted Models

Thỉnh thoảng bạn có thể muốn "un-delete" một model đã soft delete. Để khôi phục một model đã soft delete, bạn có thể gọi phương thức `restore` trên một instance model. Phương thức `restore` sẽ set lại cột `deleted_at` của model thành `null`:

    $flight->restore();

Bạn cũng có thể sử dụng phương thức `restore` trong truy vấn để khôi phục nhiều model. Một lần nữa, giống như các hoạt động "mass" khác, thao tác này sẽ không gửi bất kỳ event model nào cho các model được khôi phục:

    Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Phương thức `restore` cũng có thể được sử dụng khi xây dựng các truy vấn [relationship](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();

<a name="permanently-deleting-models"></a>
#### Permanently Deleting Models

Thỉnh thoảng bạn có thể cần phải thực sự xóa một model ra khỏi cơ sở dữ liệu của bạn. Bạn có thể sử dụng phương thức `forceDelete` để xóa vĩnh viễn model soft delete ra khỏi bảng cơ sở dữ liệu đó:

    $flight->forceDelete();

Bạn cũng có thể sử dụng phương thức `forceDelete` trên các query quan hệ của Eloquent:

    $flight->history()->forceDelete();

<a name="querying-soft-deleted-models"></a>
### Query Model Soft Deleted

<a name="including-soft-deleted-models"></a>
#### Including Soft Deleted Models

Như đã lưu ý ở trên, các model bị soft delete sẽ bị tự động loại ra khỏi tất cả các kết quả truy vấn. Tuy nhiên, bạn có thể cho các model đã bị soft delete vào kết quả của truy vấn bằng cách gọi phương thức `withTrashed` trong câu truy vấn:

    use App\Models\Flight;

    $flights = Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

Phương thức `withTrashed` cũng có thể được gọi khi tạo query cho [quan hệ](/docs/{{version}}/eloquent-relationships):

    $flight->history()->withTrashed()->get();

<a name="retrieving-only-soft-deleted-models"></a>
#### Retrieving Only Soft Deleted Models

Phương thức `onlyTrashed` sẽ **chỉ** truy xuất vào các model đã bị soft deleted:

    $flights = Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

<a name="pruning-models"></a>
## Pruning Models

Thỉnh thoảng bạn có thể muốn xóa định kỳ các model không còn cần thiết nữa. Để thực hiện điều này, bạn có thể thêm trait `Illuminate\Database\Eloquent\Prunable` hoặc trait `Illuminate\Database\Eloquent\MassPrunable` cho các model mà bạn muốn thực hiện xoá định kỳ. Sau khi thêm một trong các trait vào model, hãy implement phương thức `prunable` và trả về một Eloquent query builder để tìm ra các model không còn cần thiết:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Prunable;

    class Flight extends Model
    {
        use Prunable;

        /**
         * Get the prunable model query.
         */
        public function prunable(): Builder
        {
            return static::where('created_at', '<=', now()->subMonth());
        }
    }

Khi đánh dấu model là `Prunable`, bạn cũng có thể định nghĩa một thức phương thức `pruning` trong model. Phương thức này sẽ được gọi trước khi model bị xóa. Phương thức này có thể hữu ích để xóa thêm bất kỳ resource nào được liên kết với model, chẳng hạn như các file được lưu trữ, trước khi model bị xóa vĩnh viễn khỏi cơ sở dữ liệu:

    /**
     * Prepare the model for pruning.
     */
    protected function pruning(): void
    {
        // ...
    }

Sau khi cấu hình model prunable của bạn, bạn nên tạo schedule chạy lệnh `model:prune` Artisan trong class `App\Console\Kernel` của ứng dụng của bạn. Bạn có thể tự do chọn khoảng thời gian thích hợp để chạy lệnh này:

    /**
     * Define the application's command schedule.
     */
    protected function schedule(Schedule $schedule): void
    {
        $schedule->command('model:prune')->daily();
    }

Hậu trường, lệnh `model:prune` sẽ tự động tìm các model "Prunablec" trong thư mục `app/Models` của ứng dụng của bạn. Nếu các model của bạn ở một vị trí khác, thì bạn có thể sử dụng tùy chọn `--model` để chỉ định tên class của model:

    $schedule->command('model:prune', [
        '--model' => [Address::class, Flight::class],
    ])->daily();

Nếu bạn muốn bỏ qua một số model ra khỏi pruned trong khi đang pruning tất cả các model khác, thì bạn có thể sử dụng tùy chọn `--except`:

    $schedule->command('model:prune', [
        '--except' => [Address::class, Flight::class],
    ])->daily();

Bạn có thể kiểm tra truy vấn `prunable` của bạn bằng cách thực hiện lệnh `model:prune` với tùy chọn `--pretend`. Khi chạy với tuỳ chọn đó, lệnh `model:prune` sẽ chỉ báo cáo ra là có bao nhiêu record sẽ bị pruned nếu lệnh này thực sự chạy:

```shell
php artisan model:prune --pretend
```

> [!WARNING]
> Các model soft delete sẽ bị xóa vĩnh viễn (`forceDelete`) nếu chúng phù hợp với câu lệnh truy vấn prunable.

<a name="mass-pruning"></a>
#### Mass Pruning

Khi các model được đánh dấu bằng trait `Illuminate\Database\Eloquent\MassPrunable`, thì các model đó sẽ bị xóa ra khỏi cơ sở dữ liệu bằng truy vấn mass-deletion. Do đó, phương thức `pruning` sẽ không được gọi, cũng như các event model `deleting` và `deleted` cũng sẽ không được gọi. Điều này là do các model không thực sự được lấy ra trước khi xóa, do đó làm cho quá trình pruning hiệu quả hơn:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\MassPrunable;

    class Flight extends Model
    {
        use MassPrunable;

        /**
         * Get the prunable model query.
         */
        public function prunable(): Builder
        {
            return static::where('created_at', '<=', now()->subMonth());
        }
    }

<a name="replicating-models"></a>
## Replicating Models

Bạn có thể tạo một bản sao chưa lưu của một instance model đã tồn tại bằng cách sử dụng phương thức `replicate`. Phương thức này đặc biệt hữu ích khi bạn có các instance model dùng chung nhiều thuộc tính giống nhau:

    use App\Models\Address;

    $shipping = Address::create([
        'type' => 'shipping',
        'line_1' => '123 Example Street',
        'city' => 'Victorville',
        'state' => 'CA',
        'postcode' => '90001',
    ]);

    $billing = $shipping->replicate()->fill([
        'type' => 'billing'
    ]);

    $billing->save();

Để bỏ qua một hoặc nhiều thuộc tính sẽ không replicate lại sang model mới, bạn có thể truyền một mảng cho phương thức `replicate`:

    $flight = Flight::create([
        'destination' => 'LAX',
        'origin' => 'LHR',
        'last_flown' => '2020-03-04 11:00:00',
        'last_pilot_id' => 747,
    ]);

    $flight = $flight->replicate([
        'last_flown',
        'last_pilot_id'
    ]);

<a name="query-scopes"></a>
## Query Scope

<a name="global-scopes"></a>
### Global Scope

Global scope cho phép bạn thêm các ràng buộc cho tất cả các truy vấn của một model nhất định. Chức năng [soft delete](#soft-deleting) của Laravel cũng sử dụng global scope để chỉ lấy ra các model "non-deleted" ra khỏi cơ sở dữ liệu. Viết global scope của riêng bạn có thể cung cấp một cách thuận tiện và dễ dàng để đảm bảo rằng mọi truy vấn cho một model nhất định đều có được các ràng buộc nhất định.

<a name="generating-scopes"></a>
#### Generating Scopes

Để tạo một global scope mới, bạn có thể gọi lệnh Artisan `make:scope`, lệnh này sẽ lưu scope đã tạo vào thư mục `app/Models/Scopes` của ứng dụng:

```shell
php artisan make:scope AncientScope
```

<a name="writing-global-scopes"></a>
#### Writing Global Scopes

Viết một global scope rất đơn giản. Đầu tiên, dùng lệnh `make:scope` để tạo một class implement từ interface `Illuminate\Database\Eloquent\Scope`. Interface `Scope` sẽ yêu cầu bạn implement một phương thức: `apply`. Phương thức `apply` có thể thêm các ràng buộc `where` hoặc các điều kiện khác cho các truy vấn khi cần:

    <?php

    namespace App\Models\Scopes;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Scope;

    class AncientScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         */
        public function apply(Builder $builder, Model $model): void
        {
            $builder->where('created_at', '<', now()->subYears(2000));
        }
    }

> [!NOTE]
> Nếu global scope của bạn đang thêm các cột vào trong câu lệnh select, thì bạn nên sử dụng phương thức `addSelect` thay vì `select`. Điều này sẽ ngăn việc bạn vô tình thay thế lệnh select hiện tại của truy vấn.

<a name="applying-global-scopes"></a>
#### Applying Global Scopes

Để gán một global scope cho một model, bạn có thể chỉ cần đặt thêm một thuộc tính `ScopedBy` vào model:

    <?php

    namespace App\Models;

    use App\Models\Scopes\AncientScope;
    use Illuminate\Database\Eloquent\Attributes\ScopedBy;

    #[ScopedBy([AncientScope::class])]
    class User extends Model
    {
        //
    }

Hoặc, bạn có thể tự đăng ký global scope bằng cách ghi đè phương thức `booted` của model và gọi phương thức `addGlobalScope` của model. Phương thức `addGlobalScope` chấp nhận một instance scope làm tham số duy nhất của nó:

    <?php

    namespace App\Models;

    use App\Models\Scopes\AncientScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booted" method of the model.
         */
        protected static function booted(): void
        {
            static::addGlobalScope(new AncientScope);
        }
    }

Sau khi thêm scope trong ví dụ trên vào model `App\Models\User`, lệnh gọi phương thức `User::all()` sẽ chạy truy vấn SQL sau:

```sql
select * from `users` where `created_at` < 0021-02-18 00:00:00
```

<a name="anonymous-global-scopes"></a>
#### Anonymous Global Scopes

Eloquent cũng cho phép bạn định nghĩa global scope bằng cách sử dụng closures, điều này đặc biệt hữu ích cho các scope đơn giản không cần phải tạo một class riêng cho nó. Khi định nghĩa global scope bằng closure xong, bạn nên cung cấp tên scope của bạn làm tham số đầu tiên cho phương thức `addGlobalScope`:

    <?php

   namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booted" method of the model.
         */
        protected static function booted(): void
        {
            static::addGlobalScope('ancient', function (Builder $builder) {
                $builder->where('created_at', '<', now()->subYears(2000));
            });
        }
    }

<a name="removing-global-scopes"></a>
#### Removing Global Scopes

Nếu bạn muốn xóa một global trong cho một truy vấn nhất định, bạn có thể sử dụng phương thức `withoutGlobalScope`. Phương thức này chấp nhận tên class của global scope làm tham số duy nhất của nó:

    User::withoutGlobalScope(AncientScope::class)->get();

Hoặc, nếu bạn đã định nghĩa global scope mà dùng closure, thì bạn nên truyền tên của scope đó:

    User::withoutGlobalScope('ancient')->get();

Nếu bạn muốn xóa một vài hoặc thậm chí là tất cả các global scope của query, bạn cũng có thể sử dụng phương thức `withoutGlobalScopes`:

    // Remove all of the global scopes...
    User::withoutGlobalScopes()->get();

    // Remove some of the global scopes...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### Local Scope

Local scope cho phép bạn định nghĩa một nhóm ràng buộc query chung mà bạn có thể dễ dàng sử dụng lại trong suốt qua trình xử lý của application của bạn. Ví dụ: bạn có thể cần thường xuyên truy xuất tất cả người dùng được coi là "popular". Để định nghĩa một scope, hãy set tiền tố `scope` cho tên phương thức trong model Eloquent.

Scope sẽ luôn phải trả về một instance query builder giống nhau hoặc `void`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         */
        public function scopePopular(Builder $query): void
        {
            $query->where('votes', '>', 100);
        }

        /**
         * Scope a query to only include active users.
         */
        public function scopeActive(Builder $query): void
        {
            $query->where('active', 1);
        }
    }

<a name="utilizing-a-local-scope"></a>
#### Utilizing A Local Scope

Khi một scope đã được định nghĩa xong, bạn có thể gọi phương thức scope khi truy vấn model. Tuy nhiên, bạn không cần phải ghi tiền tố `scope` khi gọi phương thức. Bạn thậm chí có thể kết hợp nó với các scope khác:

    use App\Models\User;

    $users = User::popular()->active()->orderBy('created_at')->get();

Việc kết hợp nhiều scope cho model Eloquent thông qua truy vấn `or` có thể yêu cầu sử dụng closure để [nhóm logic](/docs/{{version}}/queries#logical-grouping) một cách chính xác nhất:

    $users = User::popular()->orWhere(function (Builder $query) {
        $query->active();
    })->get();

Tuy nhiên, vì điều này có thể phức tạp, Laravel cung cấp một phương thức "higher order" là `orWhere` cho phép bạn kết hợp các scope với nhau một cách thuận tiện mà không cần sử dụng closure:

    $users = User::popular()->orWhere->active()->get();

<a name="dynamic-scopes"></a>
#### Dynamic Scopes

Thỉnh thoảng bạn cũng có thể muốn định nghĩa một scope nhận vào các tham số. Để bắt đầu, chỉ cần thêm các tham số bổ sung vào trong câu lệnh trong phương thức scope của bạn. Các tham số scope cần được định nghĩa sau tham số `$query`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         */
        public function scopeOfType(Builder $query, string $type): void
        {
            $query->where('type', $type);
        }
    }

Khi các tham số được thêm vào định dạng của phương thức scope, bạn có thể truyền tham số khi gọi scope:

    $users = User::ofType('admin')->get();

<a name="comparing-models"></a>
## So sánh Model

Thỉnh thoảng bạn có thể cần xác định xem hai model có "giống nhau" hay không. Phương thức `is` và `isNot` có thể được sử dụng để xác minh hai model đó có cùng khóa chính,cùng bảng và cùng kết nối cơ sở dữ liệu:

    if ($post->is($anotherPost)) {
        // ...
    }

    if ($post->isNot($anotherPost)) {
        // ...
    }

Các phương thức `is` và `isNot` cũng khả dụng khi sử dụng các [quan hệ](/docs/{{version}}/eloquent-relationships) `belongsTo`, `hasOne`, `morphTo` và `morphOne` . Phương thức này đặc biệt hữu ích khi bạn muốn so sánh một model quan hệ mà không cần chạy truy vấn để lấy model đó ra:

    if ($post->author()->is($user)) {
        // ...
    }

<a name="events"></a>
## Event

> [!NOTE]
> Want to broadcast your Eloquent events directly to your client-side application? Check out Laravel's [model event broadcasting](/docs/{{version}}/broadcasting#model-broadcasting).

Các eloquent model sẽ kích hoạt một số event, cho phép bạn hook đến các chỗ khác trong vòng đời của một model: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `trashed`, `forceDeleting`, `forceDeleted`, `restoring`, `restored`, và `replicating`.

Event `retrieved` sẽ được kích hoạt khi một model được lấy ra khỏi cơ sở dữ liệu. Khi một model mới được lưu vào lần đầu tiên, các event `creating` và `created` sẽ kích hoạt. Các event `updating` / `updated` sẽ kích hoạt khi một model đang tồn tại có sửa đổi và gọi đến phương thức `save`. Các event `saving` / `saved` sẽ kích hoạt khi một model mới được tạo hoặc cập nhật - thậm chí cả khi các thuộc tính của model đó không bị thay đổi. Các tên event kết thúc bằng `-ing` được gửi đi trước khi bất kỳ thay đổi nào của model được lưu, trong khi các event kết thúc bằng `-ed` sẽ được gửi sau khi các thay đổi của model được lưu.

Để bắt đầu, hãy định nghĩa một thuộc tính `$dispatchesEvents` trên model Eloquent của bạn để nối các thời điểm khác nhau trong vòng đời của model Eloquent đó vào các [event classes](/docs/{{version}}/events) của bạn. Mỗi class event của model sẽ nhận được một instance của model bị ảnh hưởng thông qua hàm tạo của nó:

    <?php

    namespace App\Models;

    use App\Events\UserDeleted;
    use App\Events\UserSaved;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The event map for the model.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

Sau khi định nghĩa và ánh xạ các event Eloquent của bạn, bạn có thể sử dụng [event listeners](/docs/{{version}}/events#defining-listeners) để xử lý các event đó.

> [!WARNING]
> Khi bạn cập nhật một loạt dữ liệu thông qua Eloquent, thì các event của model như `saved`, `updated`, `deleting`, và `deleted` sẽ không được kích hoạt cho các model đó. Điều này là do các model không thực sự được lấy ra khi bạn chạy các cập nhật hoặc xoá bỏ.

<a name="events-using-closures"></a>
### Dùng Closures

Thay vì sử dụng các class event tùy biến, bạn có thể đăng ký một closures để được chạy khi các event model khác nhau được gửi. Thông thường, bạn nên đăng ký các closures này trong phương thức `booted` của model của bạn:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booted" method of the model.
         */
        protected static function booted(): void
        {
            static::created(function (User $user) {
                // ...
            });
        }
    }

Nếu cần, bạn có thể sử dụng một [queue event listener ẩn danh](/docs/{{version}}/events#queuable-anonymous-event-listeners) khi đăng ký event model. Thao tác này sẽ hướng dẫn Laravel thực thi event listener của model trong background bằng cách sử dụng [queue](/docs/{{version}}/queues) của ứng dụng của bạn:

    use function Illuminate\Events\queueable;

    static::created(queueable(function (User $user) {
        // ...
    }));

<a name="observers"></a>
### Observer

<a name="defining-observers"></a>
#### Defining Observers

Nếu bạn đang listen nhiều event trên một model, bạn có thể sử dụng các observer để nhóm tất cả các listen của bạn vào trong một class duy nhất. Các class observer có tên phương thức chính là tên các event Eloquent mà bạn muốn listen. Mỗi phương thức này nhận vào model bị ảnh hưởng làm tham số duy nhất của chúng. Lệnh Artisan `make:Observer` là cách dễ nhất để tạo một class observer mới:

```shell
php artisan make:observer UserObserver --model=User
```

Lệnh này sẽ lưu file observer mới vào trong thư mục `app/Observers` của bạn. Nếu thư mục này không tồn tại, Artisan sẽ tạo nó cho bạn. Class observer mới của bạn sẽ trông giống như sau:

    <?php

    namespace App\Observers;

    use App\Models\User;

    class UserObserver
    {
        /**
         * Handle the User "created" event.
         */
        public function created(User $user): void
        {
            // ...
        }

        /**
         * Handle the User "updated" event.
         */
        public function updated(User $user): void
        {
            // ...
        }

        /**
         * Handle the User "deleted" event.
         */
        public function deleted(User $user): void
        {
            // ...
        }

        /**
         * Handle the User "restored" event.
         */
        public function restored(User $user): void
        {
            // ...
        }

        /**
         * Handle the User "forceDeleted" event.
         */
        public function forceDeleted(User $user): void
        {
            // ...
        }
    }

Để đăng ký một observer, bạn có thể thêm thuộc tính `ObservedBy` vào model:

    use App\Observers\UserObserver;
    use Illuminate\Database\Eloquent\Attributes\ObservedBy;

    #[ObservedBy([UserObserver::class])]
    class User extends Authenticatable
    {
        //
    }

Hoặc, bạn có thể tự đăng ký một observer bằng cách gọi phương thức `observe` trên model mà bạn muốn observe. Bạn có thể đăng ký observe trong phương thức `boot` của service provider `App\Providers\EventServiceProvider` của application:

    use App\Models\User;
    use App\Observers\UserObserver;

    /**
     * Register any events for your application.
     */
    public function boot(): void
    {
        User::observe(UserObserver::class);
    }

> [!NOTE]
> Có thêm các event mà observer có thể listen, chẳng hạn như `saving` và `retrieved`. Những event này được mô tả trong tài liệu [events](#events).

<a name="observers-and-database-transactions"></a>
#### Observers và Database Transactions

Khi các model đang được tạo trong một database transaction, bạn có thể muốn hướng dẫn một observer chỉ thực hiện các event của nó sau khi database transaction được thực hiện. Bạn có thể thực hiện việc này bằng cách implement interface `ShouldHandleEventsAfterCommit`trên observer của bạn. Nếu một database transaction không được thực hiện, thì event đó sẽ được thực thi ngay lập tức:

    <?php

    namespace App\Observers;

    use App\Models\User;
    use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;

    class UserObserver implements ShouldHandleEventsAfterCommit
    {
        /**
         * Handle the User "created" event.
         */
        public function created(User $user): void
        {
            // ...
        }
    }

<a name="muting-events"></a>
### Tắt event

Đôi khi bạn có thể cần tạm thời "tắt" tất cả các event do một model kích hoạt. Bạn có thể làm được điều này bằng cách sử dụng phương thức `withoutEvents`. Phương thức `withoutEvents` chấp nhận một closure làm tham số duy nhất của nó. Bất kỳ code nào được chạy trong closure này sẽ không gửi bất kỳ event nào của model và bất kỳ giá trị nào được trả về bởi closure cũng là giá trị sẽ được trả về bởi phương thức `withoutEvents`:

    use App\Models\User;

    $user = User::withoutEvents(function () {
        User::findOrFail(1)->delete();

        return User::find(2);
    });

<a name="saving-a-single-model-without-events"></a>
#### Saving A Single Model Without Events

Thỉnh thoảng bạn có thể muốn "lưu" một model nhất định mà không gửi bất kỳ event nào. Bạn có thể thực hiện việc này bằng cách sử dụng phương thức `saveQuietly`:

    $user = User::findOrFail(1);

    $user->name = 'Victoria Faith';

    $user->saveQuietly();

Bạn cũng có thể "update", "delete", "soft delete", "restore", và "replicate" một model nhất định mà không gửi bất kỳ event nào:

    $user->deleteQuietly();
    $user->forceDeleteQuietly();
    $user->restoreQuietly();
