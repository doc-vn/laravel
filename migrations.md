# Database: Migrations

- [Giới thiệu](#introduction)
- [Tạo Migration](#generating-migrations)
    - [Dồn Migration](#squashing-migrations)
- [Cấu trúc Migration](#migration-structure)
- [Chạy Migration](#running-migrations)
    - [Rollback Migration](#rolling-back-migrations)
- [Table](#tables)
    - [Tạo Tables](#creating-tables)
    - [Cập nhật Tables](#updating-tables)
    - [Đổi tên / Xoá Table](#renaming-and-dropping-tables)
- [Column](#columns)
    - [Tạo Column](#creating-columns)
    - [Các loại Column có sẵn](#available-column-types)
    - [Column Modifiers](#column-modifiers)
    - [Sửa Column](#modifying-columns)
    - [Sửa tên Column](#renaming-columns)
    - [Xoá Column](#dropping-columns)
- [Index](#indexes)
    - [Tạo Index](#creating-indexes)
    - [Đổi tên Index](#renaming-indexes)
    - [Xoá Index](#dropping-indexes)
    - [Rằng buộc khoá ngoại](#foreign-key-constraints)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Migration giống như là một version control cho cơ sở dữ liệu, cho phép team của bạn định nghĩa và chia sẻ các định nghĩa database schema của application. Nếu bạn đã từng phải nói với các thành viên trong team của bạn là tự thêm một cột vào database schema ở local của họ sau khi họ lấy code mới về, thì bạn đã từng gặp phải vấn đề về migration cơ sở dữ liệu.

Laravel [facade](/docs/{{version}}/facades) `Schema` cung cấp một cách để tạo và thao tác với các bảng trên tất cả các hệ thống cơ sở dữ liệu được hỗ trợ bởi Laravel mà không cần quan tâm về loại database. Thông thường, migration sẽ sử dụng facade này để tạo và sửa các bảng và các cột trong cơ sở dữ liệu.

<a name="generating-migrations"></a>
## Tạo Migration

Bạn có thể sử dụng [lệnh Artisan](/docs/{{version}}/artisan) `make:migration` để tạo ra một file migration cơ sở dữ liệu mới. Migration mới này sẽ được lưu trong thư mục `database/migrations` của bạn. Mỗi tên file migration sẽ chứa một timestamp cho phép Laravel xác định thứ tự chạy migration:

```shell
php artisan make:migration create_flights_table
```

Laravel sẽ sử dụng tên của migration để cố gắng đoán ra tên của bảng và liệu migration đó có tạo ra bảng mới hay không. Nếu Laravel xác định được tên bảng từ tên migration, thì Laravel sẽ khai báo trước tên bảng vào file migration đã tạo. Nếu như không xác định được, bạn có thể phải chỉ định tên bảng vào trong file migration.

Nếu bạn muốn chỉ định một path riêng cho migration được tạo ra, bạn có thể sử dụng tùy chọn `--path` khi chạy lệnh `make:migration`. Path được chỉ định phải bắt đầu từ path base của ứng dụng của bạn tạo ra.

> [!NOTE]
> Các stub của migration có thể được tùy chỉnh bằng cách sử dụng [export stub](/docs/{{version}}/artisan#stub-customization)

<a name="squashing-migrations"></a>
### Dồn Migration

Khi bạn xây dựng ứng dụng của bạn, bạn có thể bị tích tụ ngày càng nhiều file migration theo thời gian. Điều này có thể khiến thư mục `database/migrations` của bạn trở nên quá tải với hàng trăm file migration. Nếu muốn, bạn có thể "dồn" migration của bạn vào một file SQL. Để bắt đầu, hãy chạy lệnh `schema:dump`:

```shell
php artisan schema:dump

# Dump the current database schema and prune all existing migrations...
php artisan schema:dump --prune
```

Khi bạn chạy lệnh này, Laravel sẽ ghi ra một file "schema" vào thư mục `database/schema` trong ứng dụng của bạn. Tên file schema sẽ tương ứng với kết nối cơ sở dữ liệu. Bây giờ, khi bạn chạy migrate cơ sở dữ liệu của bạn mà chưa chạy file migration nào khác, thì Laravel sẽ chạy đầu tiên là các câu lệnh SQL trong file schema kết nối cơ sở dữ liệu mà bạn đang sử dụng. Sau khi chạy xong các câu lệnh của file SQL schema, Laravel sẽ chạy tiếp các file migrate còn lại mà không có trong schema dump.

Nếu các bài kiểm tra của ứng dụng của bạn sử dụng kết nối cơ sở dữ liệu nào khác, khác với kết nối mà bạn thường sử dụng trong quá trình phát triển ở local, bạn nên đảm bảo là bạn đã dump một file schema bằng kết nối cơ sở dữ liệu đó để các bài kiểm tra của bạn có thể build cơ sở dữ liệu của bạn. Bạn có thể muốn thực hiện việc này sau khi dump kết nối cơ sở dữ liệu mà bạn thường sử dụng trong quá trình phát triển ở local:

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

Bạn nên commit file schema của cơ sở dữ liệu của bạn vào trong source control để các nhà phát triển mới khác ở trong team của bạn có thể nhanh chóng tạo ra cơ sở dữ liệu cho ứng dụng của bạn.

> [!WARNING]
> Tính năng dồn migration này, hiện tại sẽ chỉ có khả dụng cho cơ sở dữ liệu MariaDB, MySQL, PostgreSQL và SQLite, sử dụng command-line của cơ sở dữ liệu bên phía client.

<a name="migration-structure"></a>
## Cấu trúc Migration

Một class migration sẽ chứa hai phương thức: `up` và `down`. Phương thức `up` sẽ được sử dụng để thêm một bảng, một cột hoặc một index mới vào cơ sở dữ liệu của bạn, trong khi phương thức `down` sẽ quay ngược lại các hành động mà được thực hiện bởi phương thức `up`.

Trong cả hai phương thức này, bạn đều có thể sử dụng schema builder của Laravel để tạo và sửa các bảng một cách rõ ràng. Để tìm hiểu về tất cả các phương thức có sẵn trong `Schema` builder, [hãy xem tài liệu về nó](#creating-tables). Ví dụ, migration ở dưới sẽ tạo ra một bảng `flights`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

<a name="setting-the-migration-connection"></a>
#### Setting The Migration Connection

Nếu migration của bạn tương tác với một kết nối cơ sở dữ liệu mà không phải là kết nối cơ sở dữ liệu mặc định của ứng dụng, bạn nên set thuộc tính `$connection` cho migration của bạn:

```php
/**
 * The database connection that should be used by the migration.
 *
 * @var string
 */
protected $connection = 'pgsql';

/**
 * Run the migrations.
 */
public function up(): void
{
    // ...
}
```

<a name="skipping-migrations"></a>
#### Skipping Migrations

Thỉnh thoảng một migration có thể được dùng để hỗ trợ một tính năng chưa được kích hoạt và bạn không muốn nó chạy ngay tại thời điểm hiện tại. Trong trường hợp này, bạn có thể định nghĩa một phương thức `shouldRun` trong migration. Nếu phương thức `shouldRun` trả về `false`, migration sẽ bị bỏ qua:

```php
use App\Models\Flight;
use Laravel\Pennant\Feature;

/**
 * Determine if this migration should run.
 */
public function shouldRun(): bool
{
    return Feature::active(Flight::class);
}
```

<a name="running-migrations"></a>
## Chạy Migration

Để chạy tất cả các migration của bạn, hãy chạy lệnh Artisan `migrate`:

```shell
php artisan migrate
```

Nếu bạn muốn xem những file migration nào đã chạy và những file nào đang chờ, bạn có thể sử dụng lệnh Artisan `migrate:status`:

```shell
php artisan migrate:status
```

Nếu bạn muốn xem các câu lệnh SQL sẽ được chạy bởi lệnh migration trước khi thực sự chạy chúng, bạn có thể cung cấp flag `--pretend` cho lệnh `migrate`:

```shell
php artisan migrate --pretend
```

<a name="isolating-migration-execution"></a>
#### Isolating Migration Execution

Nếu bạn đang deploy ứng dụng của bạn trên nhiều máy chủ và chạy migration như một phần của quy trình deploy, bạn có thể không muốn hai máy chủ cùng chạy migration cơ sở dữ liệu cùng một lúc. Để tránh điều này, bạn có thể sử dụng tùy chọn `isolated` khi gọi lệnh `migrate`.

Khi tùy chọn `isolated` được cung cấp, Laravel sẽ lấy khóa atomic bằng driver bộ nhớ cache của ứng dụng trước khi thử chạy lệnh migration của bạn. Mọi nỗ lực khác để chạy lệnh `migrate` trong khi khóa đó đang được giữ sẽ không thành công; tuy nhiên, lệnh vẫn sẽ hiển thị với mã trạng thái thành công:

```shell
php artisan migrate --isolated
```

> [!WARNING]
> Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng driver cache `memcached`, `redis`, `dynamodb`, `database`, `file` hoặc `array` làm driver cache mặc định cho ứng dụng của bạn. Ngoài ra, tất cả các server phải giao tiếp cùng với một server cache trung tâm.

<a name="forcing-migrations-to-run-in-production"></a>
#### Forcing Migrations To Run In Production

Một số hành động migration có thể là nguy hiểm, có nghĩa là chúng có thể khiến bạn mất dữ liệu. Để bảo vệ bạn khỏi việc chạy các lệnh này đối với cơ sở dữ liệu production, bạn sẽ được nhắc xác nhận trước khi chạy các lệnh được này. Để bắt các lệnh này chạy mà không nhắc xác nhận, hãy sử dụng cờ `--force`:

```shell
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### Rollback Migration

Để rollback về migration mới nhất, bạn có thể sử dụng lệnh Artisan `rollback`. Lệnh này sẽ rollback lại "batch" migration cuối cùng mà bạn dùng, nó có thể có chứa nhiều file migration:

```shell
php artisan migrate:rollback
```

Bạn có thể muốn rollback lại một số migration cần thiết bằng cách cung cấp thêm tùy chọn `step` cho lệnh `rollback`. Ví dụ: lệnh sau sẽ rollback lại năm lần trước khi đến batch cuối cùng:

```shell
php artisan migrate:rollback --step=5
```

Bạn có thể roll back lại một "batch" migrations cụ thể bằng cách cung cấp tùy chọn `batch` cho lệnh `rollback`, trong đó tùy chọn `batch` tương ứng với giá trị batch trong bảng cơ sở dữ liệu `migrations` của ứng dụng. Ví dụ, lệnh sau sẽ roll back lại tất cả các migrations trong batch thứ ba:

```shell
php artisan migrate:rollback --batch=3
```

Nếu bạn muốn xem các câu lệnh SQL sẽ được thực thi bởi quá trình migration mà không muốn chạy chúng, bạn có thể cung cấp flag `--pretend` cho lệnh `migrate:rollback`:

```shell
php artisan migrate:rollback --pretend
```

Lệnh `migrate:reset` sẽ rollback lại tất cả các migration của application của bạn:

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### Roll Back và Migrate Using A Single Command

Lệnh `migrate:refresh` sẽ rollback lại tất cả các migration của bạn và sau đó thực hiện lại lệnh `migrate`. Lệnh này sẽ tạo lại toàn bộ cơ sở dữ liệu của bạn:

```shell
php artisan migrate:refresh

# Refresh the database and run all database seeds...
php artisan migrate:refresh --seed
```

Bạn có thể rollback và migrate lại một số migration cần thiết bằng cách cung cấp tùy chọn `step` cho lệnh `refresh`. Ví dụ: lệnh sau sẽ rollback và migrate lại năm lần trước so với migration gần nhất:

```shell
php artisan migrate:refresh --step=5
```

<a name="drop-all-tables-migrate"></a>
#### Drop All Tables và Migrate

Lệnh `migrate:fresh` sẽ xóa tất cả các bảng ra khỏi cơ sở dữ liệu và sau đó thực thi lại lệnh `migrate`:

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

Mặc định, lệnh `migrate:fresh` sẽ chỉ xóa các bảng khỏi kết nối cơ sở dữ liệu mặc định. Tuy nhiên, bạn có thể sử dụng tùy chọn `--database` để chỉ định kết nối cơ sở dữ liệu nào sẽ cần migration. Tên kết nối cơ sở dữ liệu phải tương ứng với kết nối được định nghĩa trong [file cấu hình](/docs/{{version}}/configuration) `database` của ứng dụng:

```shell
php artisan migrate:fresh --database=admin
```

> [!WARNING]
> Lệnh `migrate:fresh` sẽ xoá tất cả các bảng cơ sở dữ liệu bất kể prefix của chúng là gì. Lệnh này nên được sử dụng thận trọng khi đang phát triển trên những cơ sở dữ liệu mà nó được chia sẻ với các ứng dụng khác.

<a name="tables"></a>
## Table

<a name="creating-tables"></a>
### Tạo Tables

Để tạo một bảng cơ sở dữ liệu mới, hãy sử dụng phương thức `create` trên facade `Schema`. Phương thức `create` chấp nhận hai tham số: tham số đầu tiên là tên của bảng, trong khi tham số thứ hai là một closure nhận vào một đối tượng `Blueprint` có thể được sử dụng để định nghĩa một bảng mới:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email');
    $table->timestamps();
});
```

Khi tạo bảng, bạn có thể sử dụng bất kỳ [column methods](#creating-columns) nào của schema builder để định nghĩa các cột của bảng.

<a name="determining-table-column-existence"></a>
#### Determining Table / Column Existence

Bạn có thể xác định sự tồn tại của một bảng, một cột, hoặc một index bằng các phương thức `hasTable`, `hasColumn`, và `hasIndex`:

```php
if (Schema::hasTable('users')) {
    // The "users" table exists...
}

if (Schema::hasColumn('users', 'email')) {
    // The "users" table exists and has an "email" column...
}

if (Schema::hasIndex('users', ['email'], 'unique')) {
    // The "users" table exists and has a unique index on the "email" column...
}
```

<a name="database-connection-table-options"></a>
#### Database Connection và Table Options

Nếu bạn muốn thực hiện một schema trên một kết nối cơ sở dữ liệu không phải là kết nối mặc định của application của bạn, hãy sử dụng phương thức `connection`:

```php
Schema::connection('sqlite')->create('users', function (Blueprint $table) {
    $table->id();
});
```

Ngoài ra, có một số thuộc tính và phương thức khác có thể được sử dụng để định nghĩa các khía cạnh khác của việc tạo bảng. Thuộc tính `engine` có thể được sử dụng để chỉ định storage engine của bảng đó khi sử dụng MariaDB hoặc MySQL:

```php
Schema::create('users', function (Blueprint $table) {
    $table->engine('InnoDB');

    // ...
});
```

Thuộc tính `charset` và `collation` có thể được sử dụng để chỉ định character set và collation cho bảng được tạo khi sử dụng MariaDB hoặc MySQL:

```php
Schema::create('users', function (Blueprint $table) {
    $table->charset('utf8mb4');
    $table->collation('utf8mb4_unicode_ci');

    // ...
});
```

Phương thức `temporary` có thể được sử dụng để chỉ ra rằng bảng này sẽ phải là "temporary". Các bảng temporary này chỉ hiển thị trong session kết nối cơ sở dữ liệu hiện tại và sẽ tự động bị xoá đi khi kết nối bị đóng:

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});
```

Nếu bạn muốn thêm một "comment" vào bảng cơ sở dữ liệu, bạn có thể gọi phương thức `comment` trên instance table. Comment trên table hiện chỉ được hỗ trợ trong MariaDB, MySQL, và PostgreSQL:

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});
```

<a name="updating-tables"></a>
### Cập nhật Tables

Phương thức `table` trên facade `Schema` có thể được sử dụng để cập nhật các bảng hiện có. Giống như phương thức `create`, phương thức `table` sẽ chấp nhận hai tham số: một là tên của bảng hiện tại và một là một closure nhận vào một instance `Blueprint` mà bạn có thể sử dụng để thêm cột hoặc index vào bảng:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="renaming-and-dropping-tables"></a>
### Đổi tên / Xoá Table

Để đổi tên một bảng đã tồn tại trong cơ sở dữ liệu, hãy sử dụng phương thức `rename`:

```php
use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);
```

Để xóa một bảng hiện có, bạn có thể sử dụng các phương thức `drop` hoặc `dropIfExists`:

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

<a name="renaming-tables-with-foreign-keys"></a>
#### Renaming Tables With Foreign Keys

Trước khi đổi tên một bảng, bạn nên kiểm tra khóa ngoại trỏ đến bảng đó đã có trong file migration thay vì để Laravel tự gán tên dựa trên quy ước của nó. Nếu không, tên khóa ngoại đó sẽ tham chiếu đến tên bảng cũ.

<a name="columns"></a>
## Column

<a name="creating-columns"></a>
### Tạo Column

Phương thức `table` trên facade `Schema` có thể được sử dụng để cập nhật các bảng đã tồn tại. Giống như phương thức `create`, phương thức `table` chấp nhận hai tham số: một là tên một bảng và hai là một closure nhận vào một instance `Illuminate\Database\Schema\Blueprint` mà bạn có thể sử dụng nó để thêm các cột vào trong bảng:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="available-column-types"></a>
### Các loại Column có sẵn

Schema builder blueprint cung cấp nhiều phương thức tương ứng với các loại cột khác nhau mà bạn có thể thêm vào bảng cơ sở dữ liệu của bạn. Các phương thức có sẵn sẽ được liệt kê trong bảng dưới đây:

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="booleans-method-list"></a>
#### Boolean Types

<div class="collection-method-list" markdown="1">

[boolean](#column-method-boolean)

</div>

<a name="strings-and-texts-method-list"></a>
#### String & Text Types

<div class="collection-method-list" markdown="1">

[char](#column-method-char)
[longText](#column-method-longText)
[mediumText](#column-method-mediumText)
[string](#column-method-string)
[text](#column-method-text)
[tinyText](#column-method-tinyText)

</div>

<a name="numbers--method-list"></a>
#### Numeric Types

<div class="collection-method-list" markdown="1">

[bigIncrements](#column-method-bigIncrements)
[bigInteger](#column-method-bigInteger)
[decimal](#column-method-decimal)
[double](#column-method-double)
[float](#column-method-float)
[id](#column-method-id)
[increments](#column-method-increments)
[integer](#column-method-integer)
[mediumIncrements](#column-method-mediumIncrements)
[mediumInteger](#column-method-mediumInteger)
[smallIncrements](#column-method-smallIncrements)
[smallInteger](#column-method-smallInteger)
[tinyIncrements](#column-method-tinyIncrements)
[tinyInteger](#column-method-tinyInteger)
[unsignedBigInteger](#column-method-unsignedBigInteger)
[unsignedInteger](#column-method-unsignedInteger)
[unsignedMediumInteger](#column-method-unsignedMediumInteger)
[unsignedSmallInteger](#column-method-unsignedSmallInteger)
[unsignedTinyInteger](#column-method-unsignedTinyInteger)

</div>

<a name="dates-and-times-method-list"></a>
#### Date & Time Types

<div class="collection-method-list" markdown="1">

[dateTime](#column-method-dateTime)
[dateTimeTz](#column-method-dateTimeTz)
[date](#column-method-date)
[time](#column-method-time)
[timeTz](#column-method-timeTz)
[timestamp](#column-method-timestamp)
[timestamps](#column-method-timestamps)
[timestampsTz](#column-method-timestampsTz)
[softDeletes](#column-method-softDeletes)
[softDeletesTz](#column-method-softDeletesTz)
[year](#column-method-year)

</div>

<a name="binaries-method-list"></a>
#### Binary Types

<div class="collection-method-list" markdown="1">

[binary](#column-method-binary)

</div>

<a name="object-and-jsons-method-list"></a>
#### Object & Json Types

<div class="collection-method-list" markdown="1">

[json](#column-method-json)
[jsonb](#column-method-jsonb)

</div>

<a name="uuids-and-ulids-method-list"></a>
#### UUID & ULID Types

<div class="collection-method-list" markdown="1">

[ulid](#column-method-ulid)
[ulidMorphs](#column-method-ulidMorphs)
[uuid](#column-method-uuid)
[uuidMorphs](#column-method-uuidMorphs)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)

</div>

<a name="spatials-method-list"></a>
#### Spatial Types

<div class="collection-method-list" markdown="1">

[geography](#column-method-geography)
[geometry](#column-method-geometry)

</div>

<a name="relationship-method-list"></a>
#### Relationship Types

<div class="collection-method-list" markdown="1">

[foreignId](#column-method-foreignId)
[foreignIdFor](#column-method-foreignIdFor)
[foreignUlid](#column-method-foreignUlid)
[foreignUuid](#column-method-foreignUuid)
[morphs](#column-method-morphs)
[nullableMorphs](#column-method-nullableMorphs)

</div>

<a name="spacifics-method-list"></a>
#### Specialty Types

<div class="collection-method-list" markdown="1">

[enum](#column-method-enum)
[set](#column-method-set)
[macAddress](#column-method-macAddress)
[ipAddress](#column-method-ipAddress)
[rememberToken](#column-method-rememberToken)
[vector](#column-method-vector)

</div>

<a name="column-method-bigIncrements"></a>
#### `bigIncrements()` {.collection-method .first-collection-method}

Phương thức `bigIncrements` sẽ tạo một cột tương ứng với `UNSIGNED BIGINT` (khóa chính) sẽ tự động tăng:

```php
$table->bigIncrements('id');
```

<a name="column-method-bigInteger"></a>
#### `bigInteger()` {.collection-method}

Phương thức `bigInteger` sẽ tạo một cột tương ứng với `BIGINT`:

```php
$table->bigInteger('votes');
```

<a name="column-method-binary"></a>
#### `binary()` {.collection-method}

Phương thức `binary` sẽ tạo một cột tương ứng với `BLOB`:

```php
$table->binary('photo');
```

Khi sử dụng MySQL, MariaDB hoặc SQL Server, bạn có thể truyền các tham số `length` và `fixed` để tạo ra các cột tương đương `VARBINARY` hoặc `BINARY`:

```php
$table->binary('data', length: 16); // VARBINARY(16)

$table->binary('data', length: 16, fixed: true); // BINARY(16)
```

<a name="column-method-boolean"></a>
#### `boolean()` {.collection-method}

Phương thức `boolean` sẽ tạo một cột tương ứng với `BOOLEAN`:

```php
$table->boolean('confirmed');
```

<a name="column-method-char"></a>
#### `char()` {.collection-method}

Phương thức `char` sẽ tạo một một cột tương ứng với `CHAR` và độ dài nhất định:

```php
$table->char('name', length: 100);
```

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

Phương thức `dateTimeTz` sẽ tạo một cột tương ứng với `DATETIME` (cùng timezone) với một tuỳ chọn độ chính xác của giây tính đến hàng phân số phía sau dấu chấm:

```php
$table->dateTimeTz('created_at', precision: 0);
```

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

Phương thức `dateTime` sẽ tạo một cột tương ứng với `DATETIME` và một tuỳ chọn độ chính xác của giây tính đến hàng phân số phía sau dấu chấm:

```php
$table->dateTime('created_at', precision: 0);
```

<a name="column-method-date"></a>
#### `date()` {.collection-method}

Phương thức `date` sẽ tạo một cột tương ứng với `DATE`:

```php
$table->date('created_at');
```

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

Phương thức `decimal` sẽ tạo một cột tương ứng với `DECIMAL` và độ chính xác (tổng chữ số) và độ dài (chữ số thập phân):

```php
$table->decimal('amount', total: 8, places: 2);
```

<a name="column-method-double"></a>
#### `double()` {.collection-method}

Phương thức `double` sẽ tạo một cột tương ứng với `DOUBLE`:

```php
$table->double('amount');
```

<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

Phương thức `enum` sẽ tạo một cột tương ứng với `ENUM` và các giá trị hợp lệ:

```php
$table->enum('difficulty', ['easy', 'hard']);
```

Tất nhiên, bạn có thể sử dụng phương thức `Enum::cases()` thay vì tự định nghĩa một mảng các giá trị hợp lệ:

```php
use App\Enums\Difficulty;

$table->enum('difficulty', Difficulty::cases());
```

<a name="column-method-float"></a>
#### `float()` {.collection-method}

Phương thức `float` sẽ tạo một cột tương ứng với `FLOAT` và độ chính xác:

```php
$table->float('amount', precision: 53);
```

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

Phương thức `foreignId` sẽ tạo một cột tương ứng với `UNSIGNED BIGINT`:

```php
$table->foreignId('user_id');
```

<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

Phương thức `foreignIdFor` sẽ thêm một cột tương ứng với `{column}_id` cho một model class. Kiểu cột sẽ là `UNSIGNED BIGINT`, `CHAR(36)` hoặc `CHAR(26)` tùy thuộc vào kiểu khóa model:

```php
$table->foreignIdFor(User::class);
```

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

Phương thức `foreignUlid` sẽ tạo ra một cột tương ứng với `ULID`:

```php
$table->foreignUlid('user_id');
```

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

Phương thức `foreignUuid` sẽ tạo một cột tương ứng với `UUID`:

```php
$table->foreignUuid('user_id');
```

<a name="column-method-geography"></a>
#### `geography()` {.collection-method}

Phương thức `geography` sẽ tạo một cột tương ứng với `GEOGRAPHY` với kiểu không gian và SRID (Spatial Reference System Identifier) đã cho:

```php
$table->geography('coordinates', subtype: 'point', srid: 4326);
```

> [!NOTE]
> Việc hỗ trợ cho các kiểu dữ liệu không gian phụ thuộc vào driver cơ sở dữ liệu của bạn. Vui lòng tham khảo tài liệu của cơ sở dữ liệu của bạn. Nếu ứng dụng của bạn đang sử dụng cơ sở dữ liệu PostgreSQL, bạn phải cài đặt extension [PostGIS](https://postgis.net) trước khi có thể sử dụng phương thức `geography`.

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

Phương thức `geometry` sẽ tạo một cột tương ứng với `GEOMETRY` với kiểu không gian và SRID (Spatial Reference System Identifier) đã cho:

```php
$table->geometry('positions', subtype: 'point', srid: 0);
```

> [!NOTE]
> Việc hỗ trợ cho các kiểu dữ liệu không gian phụ thuộc vào driver cơ sở dữ liệu của bạn. Vui lòng tham khảo tài liệu của cơ sở dữ liệu của bạn. Nếu ứng dụng của bạn đang sử dụng cơ sở dữ liệu PostgreSQL, bạn phải cài đặt extension [PostGIS](https://postgis.net) trước khi có thể sử dụng phương thức `geometry`.

<a name="column-method-id"></a>
#### `id()` {.collection-method}

Phương thức `id` là lối tắt của phương thức `bigIncrements`. Mặc định, phương thức sẽ tạo một cột `id`; tuy nhiên, bạn có thể truyền vào tên của cột nếu bạn muốn gán một tên khác cho cột:

```php
$table->id();
```

<a name="column-method-increments"></a>
#### `increments()` {.collection-method}

Phương thức `increments` sẽ tạo một cột tương ứng với `UNSIGNED INTEGER` tự động tăng làm khóa chính:

```php
$table->increments('id');
```

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

Phương thức `integer` sẽ tạo một cột tương ứng với `INTEGER`:

```php
$table->integer('votes');
```

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

Phương thức `ipAddress` sẽ tạo một cột tương ứng với `VARCHAR`:

```php
$table->ipAddress('visitor');
```

Khi sử dụng PostgreSQL, cột `INET` sẽ được tạo.

<a name="column-method-json"></a>
#### `json()` {.collection-method}

Phương thức `json` sẽ tạo một cột tương ứng với `JSON`:

```php
$table->json('options');
```

Khi sử dụng SQLite, một cột `TEXT` sẽ được tạo.

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

Phương thức `jsonb` sẽ tạo một cột tương ứng với `JSONB`:

```php
$table->jsonb('options');
```

Khi sử dụng SQLite, một cột `TEXT` sẽ được tạo.

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

Phương thức `longText` sẽ tạo một cột tương ứng với `LONGTEXT`:

```php
$table->longText('description');
```

Khi sử dụng MySQL hoặc MariaDB, bạn có thể áp dụng bộ ký tự `binary` cho cột để tạo ra một cột tương đương `LONGBLOB`:

```php
$table->longText('data')->charset('binary'); // LONGBLOB
```

<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

Phương thức `macAddress` sẽ tạo một cột dùng để chứa địa chỉ MAC. Một số hệ thống cơ sở dữ liệu, chẳng hạn như PostgreSQL, có một loại cột chuyên biệt cho loại dữ liệu này. Các hệ thống cơ sở dữ liệu khác sẽ sử dụng cột tương ứng với string:

```php
$table->macAddress('device');
```

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

Phương thức `mediumIncrements` sẽ tạo một cột tương ứng với `UNSIGNED MEDIUMINT` tự động tăng làm khóa chính:

```php
$table->mediumIncrements('id');
```

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

Phương thức `mediumInteger` sẽ tạo một cột tương ứng với `MEDIUMINT`:

```php
$table->mediumInteger('votes');
```

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

Phương thức `mediumText` sẽ tạo một cột tương ứng với `MEDIUMTEXT`:

```php
$table->mediumText('description');
```

Khi sử dụng MySQL hoặc MariaDB, bạn có thể áp dụng bộ ký tự `binary` cho cột để tạo ra một cột tương đương `MEDIUMBLOB`:

```php
$table->mediumText('data')->charset('binary'); // MEDIUMBLOB
```

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

Phương thức `morphs` là một phương thức rất tiện lợi, nó sẽ thêm một cột tương ứng với `{column}_id` và một cột khác là `{column}_type` `VARCHAR`. Kiểu cột cho `{column}_id` sẽ là `UNSIGNED BIGINT`, `CHAR(36)` hoặc `CHAR(26)` tùy thuộc vào kiểu khóa của model.

Mục đích phương thức này là nhằm sử dụng khi định nghĩa các cột cần thiết cho [quan hệ đa hình](/docs/{{version}}/eloquent-relationships). Trong ví dụ dưới, các cột `taggable_id` và `taggable_type` sẽ được tạo:

```php
$table->morphs('taggable');
```

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [morphs](#column-method-morphs); tuy nhiên, các cột được tạo sẽ có giá trị "nullable":

```php
$table->nullableMorphs('taggable');
```

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [ulidMorphs](#column-method-ulidMorphs); tuy nhiên, các cột được tạo sẽ có giá trị "nullable":

```php
$table->nullableUlidMorphs('taggable');
```

<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [uuidMorphs](#column-method-uuidMorphs); tuy nhiên, các cột được tạo sẽ có giá trị "nullable":

```php
$table->nullableUuidMorphs('taggable');
```

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

Phương thức `rememberToken` sẽ tạo một cột tương ứng với `VARCHAR(100)` và có thể nullable, dùng để lưu trữ chức năng "remember me" [authentication token](/docs/{{version}}/authentication#remembering-users):

```php
$table->rememberToken();
```

<a name="column-method-set"></a>
#### `set()` {.collection-method}

Phương thức `set` sẽ tạo một cột tương ứng với `SET` và một danh sách các giá trị hợp lệ:

```php
$table->set('flavors', ['strawberry', 'vanilla']);
```

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

Phương thức `smallIncrements` sẽ tạo một cột tương ứng với `UNSIGNED SMALLINT` tự động tăng làm khóa chính:

```php
$table->smallIncrements('id');
```

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

Phương thức `smallInteger` sẽ tạo một cột tương ứng với `SMALLINT`:

```php
$table->smallInteger('votes');
```

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

Phương thức `softDeletesTz` sẽ thêm một cột tương ứng với `deleted_at` `TIMESTAMP` (cùng timezone) và có thể nullable cùng độ chính xác giây tính đến hàng phân số phía sau dấu chấm. Cột này nhằm mục đích để lưu trữ timestamp `deleted_at` sẽ cần thiết cho chức năng "soft delete" của Eloquent:

```php
$table->softDeletesTz($column = 'deleted_at', $precision = 0);
```

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

Phương thức `softDeletes` sẽ thêm một cột tương ứng với `deleted_at` `TIMESTAMP` và có thể nullable cùng độ chính xác giây tính đến hàng phân số phía sau dấu chấm. Cột này nhằm mục đích để lưu trữ timestamp `deleted_at` sẽ cần thiết cho chức năng "soft delete" của Eloquent:

```php
$table->softDeletes('deleted_at', precision: 0);
```

<a name="column-method-string"></a>
#### `string()` {.collection-method}

Phương thức `string` sẽ tạo một cột tương ứng với `VARCHAR` và độ dài cho trước:

```php
$table->string('name', length: 100);
```

<a name="column-method-text"></a>
#### `text()` {.collection-method}

Phương thức `text` sẽ tạo một cột tương ứng với `TEXT`:

```php
$table->text('description');
```

Khi sử dụng MySQL hoặc MariaDB, bạn có thể áp dụng bộ ký tự `binary` cho cột để tạo ra một cột tương đương `BLOB`:

```php
$table->text('data')->charset('binary'); // BLOB
```

<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

Phương thức `timeTz` sẽ tạo cột tương ứng với `TIME` (cùng timezone) với độ chính xác giây tính đến hàng phân số phía sau dấu chấm:

```php
$table->timeTz('sunrise', precision: 0);
```

<a name="column-method-time"></a>
#### `time()` {.collection-method}

Phương thức `time` sẽ tạo cột tương ứng với `TIME` với độ chính xác giây tính đến hàng phân số phía sau dấu chấm:

```php
$table->time('sunrise', precision: 0);
```

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

Phương thức `timestampTz` sẽ tạo cột tương ứng với `TIMESTAMP` (cùng timezone) với độ chính xác giây tính đến hàng phân số phía sau dấu chấm:

```php
$table->timestampTz('added_at', precision: 0);
```

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

Phương thức `timestamp` sẽ tạo cột tương ứng với `TIMESTAMP` với độ chính xác giây tính đến hàng phân số phía sau dấu chấm:

```php
$table->timestamp('added_at', precision: 0);
```

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

Phương thức `timestampsTz` sẽ tạo các cột `created_at` và `updated_at` `TIMESTAMP` (cùng timezone) với độ chính xác giây tính đến hàng phân số phía sau dấu chấm:

```php
$table->timestampsTz(precision: 0);
```

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

Phương thức `timestamps` sẽ tạo các cột `created_at` và `updated_at` `TIMESTAMP` với độ chính xác giây tính đến hàng phân số phía sau dấu chấm:

```php
$table->timestamps(precision: 0);
```

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

Phương thức `tinyIncrements` sẽ tạo một cột tương ứng với `UNSIGNED TINYINT` tự động tăng làm khóa chính:

```php
$table->tinyIncrements('id');
```

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

Phương thức `tinyInteger` sẽ tạo một cột tương ứng với `TINYINT`:

```php
$table->tinyInteger('votes');
```

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

Phương thức `tinyText` sẽ tạo một cột tương ứng với `TINYTEXT`:

```php
$table->tinyText('notes');
```

Khi sử dụng MySQL hoặc MariaDB, bạn có thể áp dụng bộ ký tự `binary` cho cột để tạo ra một cột tương đương `TINYBLOB`:

```php
$table->tinyText('data')->charset('binary'); // TINYBLOB
```

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

Phương thức `unsignedBigInteger` sẽ tạo một cột tương ứng với `UNSIGNED BIGINT`:

```php
$table->unsignedBigInteger('votes');
```

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

Phương thức `unsignedInteger` sẽ tạo một cột tương ứng với `UNSIGNED INTEGER`:

```php
$table->unsignedInteger('votes');
```

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

Phương thức `unsignedMediumInteger` sẽ tạo một cột tương ứng với `UNSIGNED MEDIUMINT`:

```php
$table->unsignedMediumInteger('votes');
```

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

Phương thức `unsignedSmallInteger` sẽ tạo một cột tương ứng với `UNSIGNED SMALLINT`:

```php
$table->unsignedSmallInteger('votes');
```

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

Phương thức `unsignedTinyInteger` sẽ tạo một cột tương ứng với `UNSIGNED TINYINT`:

```php
$table->unsignedTinyInteger('votes');
```

<a name="column-method-ulidMorphs"></a>
#### `ulidMorphs()` {.collection-method}

Phương thức `ulidMorphs` là một phương thức rất tiện lợi, nó sẽ thêm một cột tương ứng với `{column}_id` `CHAR(26)` và một cột khác là `{column}_type` `VARCHAR`.

Mục đích phương thức này là nhằm sử dụng khi định nghĩa các cột cần thiết cho [quan hệ đa hình](/docs/{{version}}/eloquent-relationships). Trong ví dụ dưới, các cột `taggable_id` và `taggable_type` sẽ được tạo:

```php
$table->ulidMorphs('taggable');
```

<a name="column-method-uuidMorphs"></a>
#### `uuidMorphs()` {.collection-method}

Phương thức `uuidMorphs` là một phương thức rất tiện lợi, nó sẽ thêm một cột tương ứng với `{column}_id` `CHAR(36)` và một cột khác là `{column}_type` `VARCHAR`.

Mục đích phương thức này là nhằm sử dụng khi định nghĩa các cột cần thiết cho [quan hệ đa hình](/docs/{{version}}/eloquent-relationships). Trong ví dụ dưới, các cột `taggable_id` và `taggable_type` sẽ được tạo:

```php
$table->uuidMorphs('taggable');
```

<a name="column-method-ulid"></a>
#### `ulid()` {.collection-method}

Phương thức `ulid` sẽ tạo một cột tương ứng với `ULID`:

```php
$table->ulid('id');
```

<a name="column-method-uuid"></a>
#### `uuid()` {.collection-method}

Phương thức `uuid` sẽ tạo một cột tương ứng với `UUID`:

```php
$table->uuid('id');
```

<a name="column-method-vector"></a>
#### `vector()` {.collection-method}

Phương thức `vector` sẽ tạo một cột tương ứng với `vector`:

```php
$table->vector('embedding', dimensions: 100);
```

<a name="column-method-year"></a>
#### `year()` {.collection-method}

Phương thức `year` sẽ tạo một cột tương ứng với `YEAR`:

```php
$table->year('birth_year');
```

<a name="column-modifiers"></a>
### Column Modifiers

Ngoài các loại cột được liệt kê ở trên, có một số "modifiers" cột mà bạn có thể sử dụng khi thêm một cột vào trong bảng cơ sở dữ liệu. Ví dụ, để tạo một cột chấp nhận "nullable", bạn có thể sử dụng phương thức `nullable`:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

Bảng dưới đây sẽ chứa tất cả các column modifier có sẵn. Danh sách này không bao gồm [index modifiers](#creating-indexes):

div class="overflow-auto">

| Modifier                            | Description                                                                                                       |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `->autoIncrement()`                 | Set một cột kiểu `INTEGER` là tự động tăng (primary key).                                                         |
| `->after('column')`                 | Set một column vào "sau" một column khác (MariaDB / MySQL).                                                       |
| `->charset('utf8mb4')`              | Khai báo character set cho cột (MariaDB / MySQL).                                                                 |
| `->collation('utf8mb4_unicode_ci')` | Khai báo collation cho cột.                                                                                       |
| `->comment('my comment')`           | Thêm comment vào một column (MariaDB / MySQL / PostgreSQL).                                                       |
| `->default($value)`                 | Khai báo giá trị "default" cho cột.                                                                               |
| `->first()`                         | Set một column vào vị trí "đầu tiên" trong table (MariaDB / MySQL).                                               |
| `->from($integer)`                  | Set giá trị bắt đầu của field tự động tăng (MariaDB / MySQL / PostgreSQL).                                        |
| `->invisible()`                     | Làm cho cột "ẩn" đi đối với các truy vấn `SELECT *` (MariaDB / MySQL).                                            |
| `->nullable($value = true)`         | Cho phép giá trị mặc định là `NULL` khi tạo bản ghi mới.                                                          |
| `->storedAs($expression)`           | Tạo một cột lấy data từ cột khác lưu vào chính nó (MariaDB / MySQL / PostgreSQL / SQLite).                        |
| `->unsigned()`                      | Set một cột kiểu `INTEGER` là `UNSIGNED` (MariaDB / MySQL).                                                       |
| `->useCurrent()`                    | Set cột `TIMESTAMP` dùng `CURRENT_TIMESTAMP` làm giá trị mặc định.                                                |
| `->useCurrentOnUpdate()`            | Set cột `TIMESTAMP` dùng `CURRENT_TIMESTAMP` khi bản ghi được cập nhật (MariaDB / MySQL).                         |
| `->virtualAs($expression)`          | Tạo một cột lấy data từ cột khác nhưng không được lưu trữ (MariaDB / MySQL / SQLite).                             |
| `->generatedAs($expression)`        | Tạo một cột identity với tùy chọn tăng dần được chỉ định (PostgreSQL).                                            |
| `->always()`                        | Định nghĩa mức độ ưu tiên của các giá trị tăng dần so với giá trị đầu vào cho một cột identity (PostgreSQL).      |

</div>

<a name="default-expressions"></a>
#### Default Expressions

Modifier `default` sẽ chấp nhận một giá trị hoặc một instance `Illuminate\Database\Query\Expression`. Việc sử dụng một instance `Expression` sẽ ngăn chặn việc Laravel đưa các giá trị vào trong dấu ngoặc kép và cho phép bạn sử dụng các chức năng cụ thể của cơ sở dữ liệu. Một tình huống mà điều này đặc biệt hữu ích đó là khi bạn cần gán một giá trị mặc định cho các cột JSON:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Query\Expression;
use Illuminate\Database\Migrations\Migration;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
            $table->timestamps();
        });
    }
};
```

> [!WARNING]
> Hỗ trợ các default expression cũng tùy thuộc vào driver cơ sở dữ liệu, phiên bản cơ sở dữ liệu và loại field của bạn. Vui lòng tham khảo thêm tài liệu database của bạn.

<a name="column-order"></a>
#### Column Order

Khi sử dụng cơ sở dữ liệu MariaDB hoặc MySQL, phương thức `after` có thể được sử dụng để thêm các cột vào phía sau một cột hiện có trong schema:

```php
$table->after('password', function (Blueprint $table) {
    $table->string('address_line1');
    $table->string('address_line2');
    $table->string('city');
});
```

<a name="modifying-columns"></a>
### Sửa Column

Phương thức `change` cho phép bạn sửa kiểu và thuộc tính của các cột hiện có. Ví dụ, bạn có thể muốn tăng kích thước của cột `string`. Để xem phương thức `change` hoạt động như thế nào, hãy tăng kích thước của cột `name` từ 25 lên 50. Để thực hiện điều này, chúng ta chỉ cần định nghĩa trạng thái mới của cột rồi gọi phương thức `change`:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

Khi sửa một cột, bạn phải ghi lại tất cả các modifier mà bạn muốn giữ lại trong định nghĩa cột - bất kỳ thuộc tính nào bị thiếu thì khi chạy thuộc tính đó sẽ bị loại bỏ. Ví dụ, để giữ lại các thuộc tính `unsigned`, `default` và `comment`, bạn phải gọi các modifier đó khi thay đổi cột:

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('my comment')->change();
});
```

Phương thức `change` sẽ không thay đổi các index của cột. Do đó, bạn có thể sử dụng các bộ điều chỉnh index modifier để thêm hoặc xóa một index khi sửa cột:

```php
// Add an index...
$table->bigIncrements('id')->primary()->change();

// Drop an index...
$table->char('postal_code', 10)->unique(false)->change();
```

<a name="renaming-columns"></a>
### Sửa tên Column

Để đổi tên một cột, bạn có thể sử dụng phương thức `renameColumn` được cung cấp bởi schema builder:

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

<a name="dropping-columns"></a>
### Xoá Column

Để xóa một cột, bạn có thể sử dụng phương thức `dropColumn` trong schema builder:

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

Bạn có thể xóa nhiều cột từ một bảng bằng cách truyền một mảng gồm tên các cột vào trong phương thức `dropColumn`:

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

<a name="available-command-aliases"></a>
#### Available Command Aliases

Laravel cung cấp một số phương thức thuận tiện liên quan đến việc xoá các cột phổ biến. Mỗi phương thức này được mô tả trong bảng dưới đây:

<div class="overflow-auto">

| Command                             | Description                                           |
| ----------------------------------- | ----------------------------------------------------- |
| `$table->dropMorphs('morphable');`  | Xoá cột `morphable_id` và cột `morphable_type`.       |
| `$table->dropRememberToken();`      | Xoá cột `remember_token`.                             |
| `$table->dropSoftDeletes();`        | Xoá cột `deleted_at`.                                 |
| `$table->dropSoftDeletesTz();`      | Lối tắt của phương thức `dropSoftDeletes()`.          |
| `$table->dropTimestamps();`         | Xoá cột `created_at` và `updated_at`.                 |
| `$table->dropTimestampsTz();`       | Lối tắt của phương thức `dropTimestamps()`.           |

</div>

<a name="indexes"></a>
## Index

<a name="creating-indexes"></a>
### Tạo Index

Schema builder của Laravel có hỗ trợ một số loại index. Ví dụ sau sẽ tạo một cột `email` mới và yêu cầu rằng cột đó phải là unique. Để tạo một index, chúng ta có thể kết hợp thêm phương thức `unique` vào trong định nghĩa của cột:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->unique();
});
```

Ngoài ra, bạn có thể tạo index sau khi định nghĩa cột đó. Để làm như vậy, bạn nên gọi phương thức `unique` trên schema builder blueprint. Phương thức này chấp nhận tên của cột mà sẽ được unique index:

```php
$table->unique('email');
```

Bạn thậm chí có thể truyền một mảng gồm các cột cho một phương thức index để tạo index gộp (hoặc index hỗn hợp):

```php
$table->index(['account_id', 'created_at']);
```

Khi tạo một index, Laravel sẽ tự động tạo tên index dựa trên tên bảng, tên cột và kiểu index, nhưng bạn có thể truyền thêm tham số thứ hai cho phương thức để khai báo tên index của bạn:

```php
$table->unique('email', 'unique_email');
```

<a name="available-index-types"></a>
#### Available Index Types

Class schema builder blueprint của Laravel sẽ cung cấp các phương thức khác nhau để tạo ra từng loại index mà được Laravel hỗ trợ. Mỗi phương thức của index chấp nhận một tham số thứ hai tùy chọn để chỉ định tên của index. Nếu bỏ qua tuỳ chọn này, thì tên sẽ được lấy từ tên của (các) bảng và các cột để sử dụng cho index, cũng như loại index. Các phương thức tạo index sẽ được mô tả trong bảng dưới đây:

<div class="overflow-auto">

| Command                                          | Description                                                    |
| ------------------------------------------------ | -------------------------------------------------------------- |
| `$table->primary('id');`                         | Thêm một primary key.                                          |
| `$table->primary(['id', 'parent_id']);`          | Thêm key hỗn hợp.                                              |
| `$table->unique('email');`                       | Thêm một unique index.                                         |
| `$table->index('state');`                        | Thêm một index.                                                |
| `$table->fullText('body');`                      | Thêm một full text index (MariaDB / MySQL / PostgreSQL).       |
| `$table->fullText('body')->language('english');` | Thêm một full text index của một ngôn ngữ cụ thể (PostgreSQL). |
| `$table->spatialIndex('location');`              | Thêm một spatial index. (trừ SQLite).                          |

</div>

<a name="renaming-indexes"></a>
### Đổi tên Index

Để đổi tên một index, bạn có thể sử dụng phương thức `renameIndex` được cung cấp bởi schema builder blueprint. Phương thức này chấp nhận tên index hiện tại làm tham số đầu tiên và một tên làm tham số thứ hai:

```php
$table->renameIndex('from', 'to')
```

<a name="dropping-indexes"></a>
### Xoá Index

Để xóa một index, bạn có thể khai báo một tên index. Mặc định, Laravel sẽ tự động gán một tên index dựa trên tên bảng và tên cột của index và loại index. Dưới đây là một số ví dụ:

<div class="overflow-auto">

| Command                                                  | Description                                                 |
| -------------------------------------------------------- | ----------------------------------------------------------- |
| `$table->dropPrimary('users_id_primary');`               |  Xoá một primary key từ bảng "users".                       |
| `$table->dropUnique('users_email_unique');`              |  Xoá một unique index từ bảng "users".                      |
| `$table->dropIndex('geo_state_index');`                  |  Xoá một index từ bảng "geo" table.                         |
| `$table->dropFullText('posts_body_fulltext');`           |  Xoá một full text index từ bảng "posts".                   |
| `$table->dropSpatialIndex('geo_location_spatialindex');` |  Xoá một spatial index từ bảng "geo" (trừ SQLite).          |

</div>

Nếu bạn truyền một mảng gồm các cột vào trong một phương thức xoá index, thì quy ước tên index sẽ được tạo dựa trên tên bảng, tên cột, và loại index:

```php
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // Drops index 'geo_state_index'
});
```

<a name="foreign-key-constraints"></a>
### Rằng buộc khoá ngoại

Laravel cũng cung cấp hỗ trợ để tạo các ràng buộc khóa ngoại, được sử dụng để đảm bảo tính toàn vẹn cho cơ sở dữ liệu. Ví dụ: hãy định nghĩa một cột `user_id` trong bảng `posts` là khoá ngoại của cột `id` trong bảng` users`:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');

    $table->foreign('user_id')->references('id')->on('users');
});
```

Vì cú pháp này khá dài dòng, nên Laravel đã cung cấp thêm các phương thức bổ sung, ngắn gọn hơn sử dụng nhiều quy ước để cung cấp trải nghiệm tốt hơn cho nhà phát triển. Khi sử dụng phương thức `foreignId` để tạo cột của bạn, ví dụ trên có thể được viết lại như sau:

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

Phương thức `foreignId` sẽ tạo một cột tương ứng với `UNSIGNED BIGINT`, trong khi phương thức `constrained` sẽ sử dụng các quy ước để xác định bảng và cột đang được tham chiếu. Nếu tên bảng của bạn không phù hợp với các quy ước của Laravel, bạn có thể cung cấp nó cho phương thức `constrained`. Ngoài ra, tên cần được gán cho index cũng có thể chỉ định:

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained(
        table: 'users', indexName: 'posts_user_id'
    );
});
```

Bạn cũng có thể khai báo hành động mong muốn cho các thuộc tính của ràng buộc "khi xóa" hoặc "khi cập nhật":

```php
$table->foreignId('user_id')
    ->constrained()
    ->onUpdate('cascade')
    ->onDelete('cascade');
```

Một cú pháp thay thế, hàm ý cũng được cung cấp cho những hành động này:

<div class="overflow-auto">

| Method                        | Description                                       |
| ----------------------------- | ------------------------------------------------- |
| `$table->cascadeOnUpdate();`  | Cập nhật theo.                                    |
| `$table->restrictOnUpdate();` | Hạn chế cập nhật theo.                            |
| `$table->nullOnUpdate();`     | Khi cập nhật sẽ set giá trị khoá ngoại thành null.|
| `$table->noActionOnUpdate();` | Không action khi update.                          |
| `$table->cascadeOnDelete();`  | Xoá theo.                                         |
| `$table->restrictOnDelete();` | Hạn chế xoá theo.                                 |
| `$table->nullOnDelete();`     | Set khoá ngoại là null, nếu khoá chính bị xoá.    |
| `$table->noActionOnDelete();` | Sẽ chặn việc xóa nếu tồn tại các records con.     |

</div>

Bất kỳ [các sửa đổi bổ sung cho cột](#column-modifiers) sẽ đều phải được gọi trước phương thức `constrained`:

```php
$table->foreignId('user_id')
    ->nullable()
    ->constrained();
```

<a name="dropping-foreign-keys"></a>
#### Dropping Foreign Keys

Để xoá khóa ngoại, bạn có thể sử dụng phương thức `dropForeign` và truyền vào tên khóa ngoại sẽ bị xóa dưới dạng tham số. Các ràng buộc khóa ngoại sẽ được sử dụng theo quy ước đặt tên giống với các index. Nói cách khác, tên của ràng buộc khóa ngoại sẽ dựa trên tên của bảng và tên cột trong ràng buộc, theo sau là hậu tố "\_foreign":

```php
$table->dropForeign('posts_user_id_foreign');
```

Ngoài ra, bạn có thể truyền một mảng chứa tên các cột chứa khóa ngoại vào phương thức `dropForeign`. Mảng sẽ được chuyển thành tên ràng buộc khóa ngoại bằng cách sử dụng quy ước đặt tên ràng buộc của Laravel:

```php
$table->dropForeign(['user_id']);
```

<a name="toggling-foreign-key-constraints"></a>
#### Toggling Foreign Key Constraints

Bạn có thể bật hoặc tắt các ràng buộc khóa ngoại trong migration của bạn bằng cách sử dụng các phương thức sau:

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();

Schema::withoutForeignKeyConstraints(function () {
    // Constraints disabled within this closure...
});
```

> [!WARNING]
> Mặc định, SQLite sẽ vô hiệu hóa các ràng buộc khóa ngoại. Khi sử dụng SQLite, bạn hãy chắc chắn rằng là [đã bật hỗ trợ khóa ngoại](/docs/{{version}}/database#configuration) trong cấu hình cơ sở dữ liệu của bạn trước khi tạo chúng trong quá trình migration của bạn.

<a name="events"></a>
## Events

Để thuận tiện, mỗi thao tác migration sẽ gửi một [event](/docs/{{version}}/events). Tất cả các event sau đây đều được extend từ class `Illuminate\Database\Events\MigrationEvent`:

<div class="overflow-auto">

| Class                                            | Description                                      |
| ------------------------------------------------ | ------------------------------------------------ |
| `Illuminate\Database\Events\MigrationsStarted`   | Một tập hợp các file migration sắp được thực hiện.   |
| `Illuminate\Database\Events\MigrationsEnded`     | Một tập hợp các file migration đã thực hiện xong.    |
| `Illuminate\Database\Events\MigrationStarted`    | Một file migration sắp được thực hiện.      |
| `Illuminate\Database\Events\MigrationEnded`      | Một file migration đã thực hiện xong.       |
| `Illuminate\Database\Events\NoPendingMigrations` | Một lệnh migration không tìm thấy bất kỳ migration nào đang chờ xử lý. |
| `Illuminate\Database\Events\SchemaDumped`        | Một bản sao schema cơ sở dữ liệu đã hoàn thành. |
| `Illuminate\Database\Events\SchemaLoaded`        | Một bản sao schema cơ sở dữ liệu đã được load. |

</div>
