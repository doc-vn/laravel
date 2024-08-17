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

> **Note**
> Các stub của migration có thể được tùy chỉnh bằng cách sử dụng [export stub](/docs/{{version}}/artisan#stub-customization)

<a name="squashing-migrations"></a>
### Dồn Migration

Khi bạn xây dựng ứng dụng của bạn, bạn có thể bị tích tụ ngày càng nhiều file migration theo thời gian. Điều này có thể khiến thư mục `database/migrations` của bạn trở nên quá tải với hàng trăm file migration. Nếu muốn, bạn có thể "dồn" migration của bạn vào một file SQL. Để bắt đầu, hãy chạy lệnh `schema:dump`:

```shell
php artisan schema:dump

# Dump the current database schema and prune all existing migrations...
php artisan schema:dump --prune
```

Khi bạn chạy lệnh này, Laravel sẽ ghi ra một file "schema" vào thư mục `database/schema` trong ứng dụng của bạn. Tên file schema sẽ tương ứng với kết nối cơ sở dữ liệu. Bây giờ, khi bạn chạy migrate cơ sở dữ liệu của bạn mà chưa chạy file migration nào khác, thì Laravel sẽ chạy đầu tiên là các câu lệnh SQL của file schema kết nối cơ sở dữ liệu mà bạn đang sử dụng. Sau khi chạy xong các câu lệnh của file schema, Laravel sẽ chạy tiếp các file migrate còn lại mà không có trong schema dump.

Nếu các bài kiểm tra của ứng dụng của bạn sử dụng kết nối cơ sở dữ liệu nào khác, khác với kết nối mà bạn thường sử dụng trong quá trình phát triển ở local, bạn nên đảm bảo là bạn đã dump một file schema bằng kết nối cơ sở dữ liệu đó để các bài kiểm tra của bạn có thể build cơ sở dữ liệu của bạn. Bạn có thể muốn thực hiện việc này sau khi dump kết nối cơ sở dữ liệu mà bạn thường sử dụng trong quá trình phát triển ở local:

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

Bạn nên commit file schema của cơ sở dữ liệu của bạn vào trong source control để các nhà phát triển mới khác ở trong team của bạn có thể nhanh chóng tạo ra cơ sở dữ liệu cho ứng dụng của bạn.

> **Warning**
> Tính năng dồn migration này, hiện tại sẽ chỉ có khả dụng cho cơ sở dữ liệu MySQL, PostgreSQL và SQLite, sử dụng command-line của các cơ sở dữ liệu này. File schema dump này có thể không restore lại được cho cơ sở dữ liệu in-memory SQLite.

<a name="migration-structure"></a>
## Cấu trúc Migration

Một class migration sẽ chứa hai phương thức: `up` và `down`. Phương thức `up` sẽ được sử dụng để thêm một bảng, một cột hoặc một index mới vào cơ sở dữ liệu của bạn, trong khi phương thức `down` sẽ quay ngược lại các hành động mà được thực hiện bởi phương thức `up`.

Trong cả hai phương thức này, bạn đều có thể sử dụng schema builder của Laravel để tạo và sửa các bảng một cách rõ ràng. Để tìm hiểu về tất cả các phương thức có sẵn trong `Schema` builder, [hãy xem tài liệu về nó](#creating-tables). Ví dụ, migration ở dưới sẽ tạo ra một bảng `flights`:

    <?php

    use Illuminate\Database\Migrations\Migration;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    return new class extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
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
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    };

<a name="setting-the-migration-connection"></a>
#### Setting The Migration Connection

Nếu migration của bạn tương tác với một kết nối cơ sở dữ liệu mà không phải là kết nối cơ sở dữ liệu mặc định của ứng dụng, bạn nên set thuộc tính `$connection` cho migration của bạn:

    /**
     * The database connection that should be used by the migration.
     *
     * @var string
     */
    protected $connection = 'pgsql';

    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        //
    }

<a name="running-migrations"></a>
## Chạy Migration

Để chạy tất cả các migration của bạn, hãy chạy lệnh Artisan `migrate`:

```shell
php artisan migrate
```

Nếu bạn muốn xem những file migration nào đã được chạy từ trước cho đến nay, bạn có thể sử dụng lệnh Artisan `migrate:status`:

```shell
php artisan migrate:status
```

Nếu bạn muốn xem các câu lệnh SQL sẽ được chạy bởi lệnh migration trước khi thực sự chạy chúng, bạn có thể cung cấp flag `--pretend` cho lệnh `migrate`:

```shell
php artisan migrate --pretend
```

#### Isolating Migration Execution

Nếu bạn đang deploy ứng dụng của bạn trên nhiều máy chủ và chạy migration như một phần của quy trình deploy, bạn có thể không muốn hai máy chủ cùng chạy migration cơ sở dữ liệu cùng một lúc. Để tránh điều này, bạn có thể sử dụng tùy chọn `isolated` khi gọi lệnh `migrate`.

Khi tùy chọn `isolated` được cung cấp, Laravel sẽ lấy khóa atomic bằng driver bộ nhớ cache của ứng dụng trước khi thử chạy lệnh migration của bạn. Mọi nỗ lực khác để chạy lệnh `migrate` trong khi khóa đó đang được giữ sẽ không thành công; tuy nhiên, lệnh vẫn sẽ hiển thị với mã trạng thái thành công:

```shell
php artisan migrate --isolated
```

> **Warning**
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

Lệnh `migrate:reset` sẽ rollback lại tất cả các migration của application của bạn:

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### Roll Back & Migrate Using A Single Command

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
#### Drop All Tables & Migrate

Lệnh `migrate:fresh` sẽ xóa tất cả các bảng ra khỏi cơ sở dữ liệu và sau đó thực thi lại lệnh `migrate`:

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

> **Warning**
> Lệnh `migrate:fresh` sẽ xoá tất cả các bảng cơ sở dữ liệu bất kể prefix của chúng là gì. Lệnh này nên được sử dụng thận trọng khi đang phát triển trên những cơ sở dữ liệu mà nó được chia sẻ với các ứng dụng khác.

<a name="tables"></a>
## Table

<a name="creating-tables"></a>
### Tạo Tables

Để tạo một bảng cơ sở dữ liệu mới, hãy sử dụng phương thức `create` trên facade `Schema`. Phương thức `create` chấp nhận hai tham số: tham số đầu tiên là tên của bảng, trong khi tham số thứ hai là một closure nhận vào một đối tượng `Blueprint` có thể được sử dụng để định nghĩa một bảng mới:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email');
        $table->timestamps();
    });

Khi tạo bảng, bạn có thể sử dụng bất kỳ [column methods](#creating-columns) nào của schema builder để định nghĩa các cột của bảng.

<a name="checking-for-table-column-existence"></a>
#### Checking For Table / Column Existence

Bạn có thể kiểm tra sự tồn tại của một bảng hoặc một cột bằng các phương thức `hasTable` và `hasColumn`:

    if (Schema::hasTable('users')) {
        // The "users" table exists...
    }

    if (Schema::hasColumn('users', 'email')) {
        // The "users" table exists and has an "email" column...
    }

<a name="database-connection-table-options"></a>
#### Database Connection & Table Options

Nếu bạn muốn thực hiện một schema trên một kết nối cơ sở dữ liệu không phải là kết nối mặc định của application của bạn, hãy sử dụng phương thức `connection`:

    Schema::connection('sqlite')->create('users', function (Blueprint $table) {
        $table->id();
    });

Ngoài ra, có một số thuộc tính và phương thức khác có thể được sử dụng để định nghĩa các khía cạnh khác của việc tạo bảng. Thuộc tính `engine` có thể được sử dụng để chỉ định storage engine của bảng đó khi sử dụng MySQL:

    Schema::create('users', function (Blueprint $table) {
        $table->engine = 'InnoDB';

        // ...
    });

Thuộc tính `charset` và `collation` có thể được sử dụng để chỉ định character set và collation cho bảng được tạo khi sử dụng MySQL:

    Schema::create('users', function (Blueprint $table) {
        $table->charset = 'utf8mb4';
        $table->collation = 'utf8mb4_unicode_ci';

        // ...
    });

Phương thức `temporary` có thể được sử dụng để chỉ ra rằng bảng này sẽ phải là "temporary". Các bảng temporary này chỉ hiển thị trong session kết nối cơ sở dữ liệu hiện tại và sẽ tự động bị xoá đi khi kết nối bị đóng:

    Schema::create('calculations', function (Blueprint $table) {
        $table->temporary();

        // ...
    });

Nếu bạn muốn thêm một "comment" vào bảng cơ sở dữ liệu, bạn có thể gọi phương thức `comment` trên instance table. Comment trên table hiện chỉ được hỗ trợ trong MySQL và Postgres:

    Schema::create('calculations', function (Blueprint $table) {
        $table->comment('Business calculations');

        // ...
    });

<a name="updating-tables"></a>
### Cập nhật Tables

Phương thức `table` trên facade `Schema` có thể được sử dụng để cập nhật các bảng hiện có. Giống như phương thức `create`, phương thức `table` sẽ chấp nhận hai tham số: một là tên của bảng hiện tại và một là một closure nhận vào một instance `Blueprint` mà bạn có thể sử dụng để thêm cột hoặc index vào bảng:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->integer('votes');
    });

<a name="renaming-and-dropping-tables"></a>
### Đổi tên / Xoá Table

Để đổi tên một bảng đã tồn tại trong cơ sở dữ liệu, hãy sử dụng phương thức `rename`:

    use Illuminate\Support\Facades\Schema;

    Schema::rename($from, $to);

Để xóa một bảng hiện có, bạn có thể sử dụng các phương thức `drop` hoặc `dropIfExists`:

    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="renaming-tables-with-foreign-keys"></a>
#### Renaming Tables With Foreign Keys

Trước khi đổi tên một bảng, bạn nên kiểm tra khóa ngoại trỏ đến bảng đó đã có trong file migration thay vì để Laravel tự gán tên dựa trên quy ước của nó. Nếu không, tên khóa ngoại đó sẽ tham chiếu đến tên bảng cũ.

<a name="columns"></a>
## Column

<a name="creating-columns"></a>
### Tạo Column

Phương thức `table` trên facade `Schema` có thể được sử dụng để cập nhật các bảng đã tồn tại. Giống như phương thức `create`, phương thức `table` chấp nhận hai tham số: một là tên một bảng và hai là một closure nhận vào một instance `Illuminate\Database\Schema\Blueprint` mà bạn có thể sử dụng nó để thêm các cột vào trong bảng:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->integer('votes');
    });

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

<div class="collection-method-list" markdown="1">

[bigIncrements](#column-method-bigIncrements)
[bigInteger](#column-method-bigInteger)
[binary](#column-method-binary)
[boolean](#column-method-boolean)
[char](#column-method-char)
[dateTimeTz](#column-method-dateTimeTz)
[dateTime](#column-method-dateTime)
[date](#column-method-date)
[decimal](#column-method-decimal)
[double](#column-method-double)
[enum](#column-method-enum)
[float](#column-method-float)
[foreignId](#column-method-foreignId)
[foreignIdFor](#column-method-foreignIdFor)
[foreignUlid](#column-method-foreignUlid)
[foreignUuid](#column-method-foreignUuid)
[geometryCollection](#column-method-geometryCollection)
[geometry](#column-method-geometry)
[id](#column-method-id)
[increments](#column-method-increments)
[integer](#column-method-integer)
[ipAddress](#column-method-ipAddress)
[json](#column-method-json)
[jsonb](#column-method-jsonb)
[lineString](#column-method-lineString)
[longText](#column-method-longText)
[macAddress](#column-method-macAddress)
[mediumIncrements](#column-method-mediumIncrements)
[mediumInteger](#column-method-mediumInteger)
[mediumText](#column-method-mediumText)
[morphs](#column-method-morphs)
[multiLineString](#column-method-multiLineString)
[multiPoint](#column-method-multiPoint)
[multiPolygon](#column-method-multiPolygon)
[nullableMorphs](#column-method-nullableMorphs)
[nullableTimestamps](#column-method-nullableTimestamps)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)
[point](#column-method-point)
[polygon](#column-method-polygon)
[rememberToken](#column-method-rememberToken)
[set](#column-method-set)
[smallIncrements](#column-method-smallIncrements)
[smallInteger](#column-method-smallInteger)
[softDeletesTz](#column-method-softDeletesTz)
[softDeletes](#column-method-softDeletes)
[string](#column-method-string)
[text](#column-method-text)
[timeTz](#column-method-timeTz)
[time](#column-method-time)
[timestampTz](#column-method-timestampTz)
[timestamp](#column-method-timestamp)
[timestampsTz](#column-method-timestampsTz)
[timestamps](#column-method-timestamps)
[tinyIncrements](#column-method-tinyIncrements)
[tinyInteger](#column-method-tinyInteger)
[tinyText](#column-method-tinyText)
[unsignedBigInteger](#column-method-unsignedBigInteger)
[unsignedDecimal](#column-method-unsignedDecimal)
[unsignedInteger](#column-method-unsignedInteger)
[unsignedMediumInteger](#column-method-unsignedMediumInteger)
[unsignedSmallInteger](#column-method-unsignedSmallInteger)
[unsignedTinyInteger](#column-method-unsignedTinyInteger)
[ulidMorphs](#column-method-ulidMorphs)
[uuidMorphs](#column-method-uuidMorphs)
[ulid](#column-method-ulid)
[uuid](#column-method-uuid)
[year](#column-method-year)

</div>

<a name="column-method-bigIncrements"></a>
#### `bigIncrements()` {.collection-method .first-collection-method}

Phương thức `bigIncrements` sẽ tạo một cột tương ứng với `UNSIGNED BIGINT` (khóa chính) sẽ tự động tăng:

    $table->bigIncrements('id');

<a name="column-method-bigInteger"></a>
#### `bigInteger()` {.collection-method}

Phương thức `bigInteger` sẽ tạo một cột tương ứng với `BIGINT`:

    $table->bigInteger('votes');

<a name="column-method-binary"></a>
#### `binary()` {.collection-method}

Phương thức `binary` sẽ tạo một cột tương ứng với `BLOB`:

    $table->binary('photo');

<a name="column-method-boolean"></a>
#### `boolean()` {.collection-method}

Phương thức `boolean` sẽ tạo một cột tương ứng với `BOOLEAN`:

    $table->boolean('confirmed');

<a name="column-method-char"></a>
#### `char()` {.collection-method}

Phương thức `char` sẽ tạo một một cột tương ứng với `CHAR` và độ dài nhất định:

    $table->char('name', 100);

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

Phương thức `dateTimeTz` sẽ tạo một cột tương ứng với `DATETIME` (cùng timezone) với độ chính xác (tổng chữ số):

    $table->dateTimeTz('created_at', $precision = 0);

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

Phương thức `dateTime` sẽ tạo một cột tương ứng với `DATETIME` và độ chính xác (tổng chữ số):

    $table->dateTime('created_at', $precision = 0);

<a name="column-method-date"></a>
#### `date()` {.collection-method}

Phương thức `date` sẽ tạo một cột tương ứng với `DATE`:

    $table->date('created_at');

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

Phương thức `decimal` sẽ tạo một cột tương ứng với `DECIMAL` và độ chính xác (tổng chữ số) và độ dài (chữ số thập phân):

    $table->decimal('amount', $precision = 8, $scale = 2);

<a name="column-method-double"></a>
#### `double()` {.collection-method}

Phương thức `double` sẽ tạo một cột tương ứng với `DOUBLE` và độ chính xác (tổng chữ số) và độ dài (chữ số thập phân):

    $table->double('amount', 8, 2);

<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

Phương thức `enum` sẽ tạo một cột tương ứng với `ENUM` và các giá trị hợp lệ:

    $table->enum('difficulty', ['easy', 'hard']);

<a name="column-method-float"></a>
#### `float()` {.collection-method}

Phương thức `float` sẽ tạo một cột tương ứng với `FLOAT` và độ chính xác (tổng chữ số) và độ dài (chữ số thập phân):

    $table->float('amount', 8, 2);

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

Phương thức `foreignId` sẽ tạo một cột tương ứng với `UNSIGNED BIGINT`:

    $table->foreignId('user_id');

<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

Phương thức `foreignIdFor` sẽ thêm một cột tương ứng với `{column}_id UNSIGNED BIGINT` cho một model class:

    $table->foreignIdFor(User::class);

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

Phương thức `foreignUlid` sẽ tạo ra một cột tương ứng với `ULID`:

    $table->foreignUlid('user_id');

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

Phương thức `foreignUuid` sẽ tạo một cột tương ứng với `UUID`:

    $table->foreignUuid('user_id');

<a name="column-method-geometryCollection"></a>
#### `geometryCollection()` {.collection-method}

Phương thức `geometryCollection` sẽ tạo một cột tương ứng với `GEOMETRYCOLLECTION`:

    $table->geometryCollection('positions');

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

Phương thức `geometry` sẽ tạo một cột tương ứng với `GEOMETRY`:

    $table->geometry('positions');

<a name="column-method-id"></a>
#### `id()` {.collection-method}

Phương thức `id` là lối tắt của phương thức `bigIncrements`. Mặc định, phương thức sẽ tạo một cột `id`; tuy nhiên, bạn có thể truyền vào tên của cột nếu bạn muốn gán một tên khác cho cột:

    $table->id();

<a name="column-method-increments"></a>
#### `increments()` {.collection-method}

Phương thức `increments` sẽ tạo một cột tương ứng với `UNSIGNED INTEGER` tự động tăng làm khóa chính:

    $table->increments('id');

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

Phương thức `integer` sẽ tạo một cột tương ứng với `INTEGER`:

    $table->integer('votes');

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

Phương thức `ipAddress` sẽ tạo một cột tương ứng với `VARCHAR`:

    $table->ipAddress('visitor');

<a name="column-method-json"></a>
#### `json()` {.collection-method}

Phương thức `json` sẽ tạo một cột tương ứng với `JSON`:

    $table->json('options');

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

Phương thức `jsonb` sẽ tạo một cột tương ứng với `JSONB`:

    $table->jsonb('options');

<a name="column-method-lineString"></a>
#### `lineString()` {.collection-method}

Phương thức `lineString` sẽ tạo một cột tương ứng với `LINESTRING`:

    $table->lineString('positions');

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

Phương thức `longText` sẽ tạo một cột tương ứng với `LONGTEXT`:

    $table->longText('description');

<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

Phương thức `macAddress` sẽ tạo một cột dùng để chứa địa chỉ MAC. Một số hệ thống cơ sở dữ liệu, chẳng hạn như PostgreSQL, có một loại cột chuyên biệt cho loại dữ liệu này. Các hệ thống cơ sở dữ liệu khác sẽ sử dụng cột tương ứng với string:

    $table->macAddress('device');

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

Phương thức `mediumIncrements` sẽ tạo một cột tương ứng với `UNSIGNED MEDIUMINT` tự động tăng làm khóa chính:

    $table->mediumIncrements('id');

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

Phương thức `mediumInteger` sẽ tạo một cột tương ứng với `MEDIUMINT`:

    $table->mediumInteger('votes');

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

Phương thức `mediumText` sẽ tạo một cột tương ứng với `MEDIUMTEXT`:

    $table->mediumText('description');

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

Phương thức `morphs` là một phương thức rất tiện lợi, nó sẽ thêm một cột tương ứng với `{column}_id` `UNSIGNED BIGINT` và một cột khác là `{column}_type` `VARCHAR`.

Mục đích phương thức này là nhằm sử dụng khi định nghĩa các cột cần thiết cho [quan hệ đa hình](/docs/{{version}}/eloquent-relationships). Trong ví dụ dưới, các cột `taggable_id` và `taggable_type` sẽ được tạo:

    $table->morphs('taggable');

<a name="column-method-multiLineString"></a>
#### `multiLineString()` {.collection-method}

Phương thức `multiLineString` sẽ tạo một cột tương ứng với `MULTILINESTRING`:

    $table->multiLineString('positions');

<a name="column-method-multiPoint"></a>
#### `multiPoint()` {.collection-method}

Phương thức `multiPoint` sẽ tạo một cột tương ứng với `MULTIPOINT`:

    $table->multiPoint('positions');

<a name="column-method-multiPolygon"></a>
#### `multiPolygon()` {.collection-method}

Phương thức `multiPolygon` sẽ tạo một cột tương ứng với `MULTIPOLYGON`:

    $table->multiPolygon('positions');

<a name="column-method-nullableTimestamps"></a>
#### `nullableTimestamps()` {.collection-method}

Phương thức `nullableTimestamps` là lối tắt của phương thức [timestamps](#column-method-timestamps):

    $table->nullableTimestamps(0);

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [morphs](#column-method-morphs); tuy nhiên, các cột được tạo sẽ có giá trị "nullable":

    $table->nullableMorphs('taggable');

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [ulidMorphs](#column-method-ulidMorphs); tuy nhiên, các cột được tạo sẽ có giá trị "nullable":

    $table->nullableUlidMorphs('taggable');

<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [uuidMorphs](#column-method-uuidMorphs); tuy nhiên, các cột được tạo sẽ có giá trị "nullable":

    $table->nullableUuidMorphs('taggable');

<a name="column-method-point"></a>
#### `point()` {.collection-method}

Phương thức `point` sẽ tạo một cột tương ứng với `POINT`:

    $table->point('position');

<a name="column-method-polygon"></a>
#### `polygon()` {.collection-method}

Phương thức `polygon` sẽ tạo một cột tương ứng với `POLYGON`:

    $table->polygon('position');

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

Phương thức `rememberToken` sẽ tạo một cột tương ứng với `VARCHAR(100)` và có thể nullable, dùng để lưu trữ chức năng "remember me" [authentication token](/docs/{{version}}/authentication#remembering-users):

    $table->rememberToken();

<a name="column-method-set"></a>
#### `set()` {.collection-method}

Phương thức `set` sẽ tạo một cột tương ứng với `SET` và một danh sách các giá trị hợp lệ:

    $table->set('flavors', ['strawberry', 'vanilla']);

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

Phương thức `smallIncrements` sẽ tạo một cột tương ứng với `UNSIGNED SMALLINT` tự động tăng làm khóa chính:

    $table->smallIncrements('id');

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

Phương thức `smallInteger` sẽ tạo một cột tương ứng với `SMALLINT`:

    $table->smallInteger('votes');

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

Phương thức `softDeletesTz` sẽ thêm một cột tương ứng với `deleted_at` `TIMESTAMP` (cùng timezone) và có thể nullable cùng độ chính xác (tổng chữ số). Cột này nhằm mục đích để lưu trữ timestamp `deleted_at` sẽ cần thiết cho chức năng "soft delete" của Eloquent:

    $table->softDeletesTz($column = 'deleted_at', $precision = 0);

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

Phương thức `softDeletes` sẽ thêm một cột tương ứng với `deleted_at` `TIMESTAMP` và có thể nullable cùng độ chính xác (tổng chữ số). Cột này nhằm mục đích để lưu trữ timestamp `deleted_at` sẽ cần thiết cho chức năng "soft delete" của Eloquent:

    $table->softDeletes($column = 'deleted_at', $precision = 0);

<a name="column-method-string"></a>
#### `string()` {.collection-method}

Phương thức `string` sẽ tạo một cột tương ứng với `VARCHAR` và độ dài cho trước:

    $table->string('name', 100);

<a name="column-method-text"></a>
#### `text()` {.collection-method}

Phương thức `text` sẽ tạo một cột tương ứng với `TEXT`:

    $table->text('description');

<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

Phương thức `timeTz` sẽ tạo cột tương ứng với `TIME` (cùng timezone) với độ chính xác (tổng chữ số):

    $table->timeTz('sunrise', $precision = 0);

<a name="column-method-time"></a>
#### `time()` {.collection-method}

Phương thức `timeTz` sẽ tạo cột tương ứng với `TIME` với độ chính xác (tổng chữ số):

    $table->time('sunrise', $precision = 0);

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

Phương thức `timestampTz` sẽ tạo cột tương ứng với `TIMESTAMP` (cùng timezone) với độ chính xác (tổng chữ số):

    $table->timestampTz('added_at', $precision = 0);

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

Phương thức `timestampTz` sẽ tạo cột tương ứng với `TIMESTAMP` với độ chính xác (tổng chữ số):

    $table->timestamp('added_at', $precision = 0);

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

Phương thức `timestampTz` sẽ tạo các cột `created_at` và `updated_at` `TIMESTAMP` (cùng timezone) với độ chính xác (tổng chữ số):

    $table->timestampsTz($precision = 0);

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

Phương thức `timestampTz` sẽ tạo các cột `created_at` và `updated_at` `TIMESTAMP` với độ chính xác (tổng chữ số):

    $table->timestamps($precision = 0);

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

Phương thức `tinyIncrements` sẽ tạo một cột tương ứng với `UNSIGNED TINYINT` tự động tăng làm khóa chính:

    $table->tinyIncrements('id');

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

Phương thức `tinyInteger` sẽ tạo một cột tương ứng với `TINYINT`:

    $table->tinyInteger('votes');

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

Phương thức `tinyText` sẽ tạo một cột tương ứng với `TINYTEXT`:

    $table->tinyText('notes');

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

Phương thức `unsignedBigInteger` sẽ tạo một cột tương ứng với `UNSIGNED BIGINT`:

    $table->unsignedBigInteger('votes');

<a name="column-method-unsignedDecimal"></a>
#### `unsignedDecimal()` {.collection-method}

Phương thức `unsignedDecimal` sẽ tạo một cột tương ứng với `UNSIGNED DECIMAL` và độ chính xác (tổng chữ số) và độ dài (chữ số thập phân):

    $table->unsignedDecimal('amount', $precision = 8, $scale = 2);

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

Phương thức `unsignedInteger` sẽ tạo một cột tương ứng với `UNSIGNED INTEGER`:

    $table->unsignedInteger('votes');

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

Phương thức `unsignedMediumInteger` sẽ tạo một cột tương ứng với `UNSIGNED MEDIUMINT`:

    $table->unsignedMediumInteger('votes');

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

Phương thức `unsignedSmallInteger` sẽ tạo một cột tương ứng với `UNSIGNED SMALLINT`:

    $table->unsignedSmallInteger('votes');

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

Phương thức `unsignedTinyInteger` sẽ tạo một cột tương ứng với `UNSIGNED TINYINT`:

    $table->unsignedTinyInteger('votes');

<a name="column-method-ulidMorphs"></a>
#### `ulidMorphs()` {.collection-method}

Phương thức `ulidMorphs` là một phương thức rất tiện lợi, nó sẽ thêm một cột tương ứng với `{column}_id` `CHAR(26)` và một cột khác là `{column}_type` `VARCHAR`.

Mục đích phương thức này là nhằm sử dụng khi định nghĩa các cột cần thiết cho [quan hệ đa hình](/docs/{{version}}/eloquent-relationships). Trong ví dụ dưới, các cột `taggable_id` và `taggable_type` sẽ được tạo:

    $table->ulidMorphs('taggable');

<a name="column-method-uuidMorphs"></a>
#### `uuidMorphs()` {.collection-method}

Phương thức `uuidMorphs` là một phương thức rất tiện lợi, nó sẽ thêm một cột tương ứng với `{column}_id` `CHAR(36)` và một cột khác là `{column}_type` `VARCHAR`.

Mục đích phương thức này là nhằm sử dụng khi định nghĩa các cột cần thiết cho [quan hệ đa hình](/docs/{{version}}/eloquent-relationships). Trong ví dụ dưới, các cột `taggable_id` và `taggable_type` sẽ được tạo:

    $table->uuidMorphs('taggable');

<a name="column-method-ulid"></a>
#### `ulid()` {.collection-method}

Phương thức `ulid` sẽ tạo một cột tương ứng với `ULID`:

    $table->ulid('id');

<a name="column-method-uuid"></a>
#### `uuid()` {.collection-method}

Phương thức `uuid` sẽ tạo một cột tương ứng với `UUID`:

    $table->uuid('id');

<a name="column-method-year"></a>
#### `year()` {.collection-method}

Phương thức `year` sẽ tạo một cột tương ứng với `YEAR`:

    $table->year('birth_year');

<a name="column-modifiers"></a>
### Column Modifiers

Ngoài các loại cột được liệt kê ở trên, có một số "modifiers" cột mà bạn có thể sử dụng khi thêm một cột vào trong bảng cơ sở dữ liệu. Ví dụ, để tạo một cột chấp nhận "nullable", bạn có thể sử dụng phương thức `nullable`:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

Bảng dưới đây sẽ chứa tất cả các column modifier có sẵn. Danh sách này không bao gồm [index modifiers](#creating-indexes):

Modifier  |  Mô tả
--------  |  -----------
`->after('column')`  |  Set một column vào "sau" một column khác (MySQL).
`->autoIncrement()`  |  Set một cột kiểu INTEGER là tự động tăng (primary key).
`->charset('utf8mb4')`  |  Khai báo character set cho cột (MySQL).
`->collation('utf8mb4_unicode_ci')`  |  Khai báo collation cho cột (MySQL/PostgreSQL/SQL Server).
`->comment('my comment')`  |  Thêm comment vào một column (MySQL/PostgreSQL).
`->default($value)`  |  Khai báo giá trị "default" cho cột.
`->first()`  |  Set một column vào vị trí "đầu tiên" trong table (MySQL).
`->from($integer)`  |  Set giá trị bắt đầu của field tự động tăng (MySQL / PostgreSQL).
`->invisible()`  |  Làm cho cột "ẩn" đi đối với các truy vấn `SELECT *` (MySQL).
`->nullable($value = true)`  |  Cho phép giá trị mặc định là NULL khi tạo bản ghi mới.
`->storedAs($expression)`  |  Tạo một cột lấy data từ cột khác lưu vào chính nó (MySQL / PostgreSQL).
`->unsigned()`  |  Set một cột kiểu INTEGER là luôn dương (MySQL).
`->useCurrent()`  |  Set cột TIMESTAMP dùng CURRENT_TIMESTAMP làm giá trị mặc định.
`->useCurrentOnUpdate()`  |  Set cột TIMESTAMP dùng CURRENT_TIMESTAMP khi bản ghi được cập nhật.
`->virtualAs($expression)`  |  Tạo một cột lấy data từ cột khác nhưng không được lưu trữ (MySQL / PostgreSQL / SQLite).
`->generatedAs($expression)`  |  Tạo một cột identity với tùy chọn tăng dần được chỉ định (PostgreSQL).
`->always()`  |  Định nghĩa mức độ ưu tiên của các giá trị tăng dần so với giá trị đầu vào cho một cột identity (PostgreSQL).
`->isGeometry()`  |  Set cột thành `geometry` - loại mặc định là `geography` (PostgreSQL).

<a name="default-expressions"></a>
#### Default Expressions

Modifier `default` sẽ chấp nhận một giá trị hoặc một instance `Illuminate\Database\Query\Expression`. Việc sử dụng một instance `Expression` sẽ ngăn chặn việc Laravel đưa các giá trị vào trong dấu ngoặc kép và cho phép bạn sử dụng các chức năng cụ thể của cơ sở dữ liệu. Một tình huống mà điều này đặc biệt hữu ích đó là khi bạn cần gán một giá trị mặc định cho các cột JSON:

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Query\Expression;
    use Illuminate\Database\Migrations\Migration;

    return new class extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->id();
                $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
                $table->timestamps();
            });
        }
    };

> **Warning**
> Hỗ trợ các default expression cũng tùy thuộc vào driver cơ sở dữ liệu, phiên bản cơ sở dữ liệu và loại field của bạn. Vui lòng tham khảo thêm tài liệu database của bạn. Ngoài ra, không thể kết hợp các raw `default` expression (sử dụng `DB::raw`) với các thay đổi cột thông qua phương thức `change`.

<a name="column-order"></a>
#### Column Order

Khi sử dụng cơ sở dữ liệu MySQL, phương thức `after` có thể được sử dụng để thêm các cột vào phía sau một cột hiện có trong schema:

    $table->after('password', function ($table) {
        $table->string('address_line1');
        $table->string('address_line2');
        $table->string('city');
    });

<a name="modifying-columns"></a>
### Sửa Column

<a name="prerequisites"></a>
#### Prerequisites

Trước khi sửa một cột, bạn phải cài đặt package `doctrine/dbal` bằng Composer package manager. Thư viện Doctrine DBAL được sử dụng để xác định trạng thái hiện tại của cột và để tạo ra các truy vấn SQL cần thiết để thực hiện các yêu cầu thay đổi cột của bạn:

    composer require doctrine/dbal

Nếu bạn muốn sửa các cột đã được tạo bằng phương thức `timestamp`, bạn cũng phải thêm cấu hình sau vào file cấu hình `config/database.php` của ứng dụng của bạn:

```php
use Illuminate\Database\DBAL\TimestampType;

'dbal' => [
    'types' => [
        'timestamp' => TimestampType::class,
    ],
],
```

> **Warning**
> Nếu ứng dụng của bạn đang sử dụng Microsoft SQL Server, hãy đảm bảo rằng bạn đã cài đặt `doctrine/dbal:^3.0`.

<a name="updating-column-attributes"></a>
#### Updating Column Attributes

Phương thức `change` cho phép bạn sửa một số loại và thuộc tính của cột hiện có. Ví dụ: bạn có thể muốn tăng kích thước của cột `string`. Để xem phương thức `change` hoạt động như thế nào, hãy thử tăng kích thước của cột `name` từ 25 lên 50. Để thực hiện điều này, chúng ta chỉ cần định nghĩa trạng thái mới của cột và sau đó gọi phương thức `change`:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

Chúng ta cũng có thể sửa một cột thành nullable:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> **Warning**
> Các loại cột sau mới có thể thay đổi: `bigInteger`, `binary`, `boolean`, `char`, `date`, `dateTime`, `dateTimeTz`, `decimal`, `double`, `integer`, `json`, `longText`, `mediumText`, `smallInteger`, `string`, `text`, `time`, `tinyText`, `unsignedBigInteger`, `unsignedInteger`, `unsignedSmallInteger`, và `uuid`.  Để sửa cột `timestamp`, bạn phải [đăng ký Doctrine type](#prerequisites).

<a name="renaming-columns"></a>
### Sửa tên Column

Để đổi tên một cột, bạn có thể sử dụng phương thức `renameColumn` được cung cấp bởi schema builder:

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

<a name="renaming-columns-on-legacy-databases"></a>
#### Sửa tên Column On Legacy Databases

Nếu bạn đang chạy cơ sở dữ liệu mà cũ hơn một trong những bản phát hành sau, bạn sẽ phải đảm bảo là bạn đã cài đặt thư viện `doctrine/dbal` thông qua trình quản lý package Composer trước khi đổi tên cột:

<div class="content-list" markdown="1">

- MySQL < `8.0.3`
- MariaDB < `10.5.2`
- SQLite < `3.25.0`

</div>

<a name="dropping-columns"></a>
### Xoá Column

Để xóa một cột, bạn có thể sử dụng phương thức `dropColumn` trong schema builder:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

Bạn có thể xóa nhiều cột từ một bảng bằng cách truyền một mảng gồm tên các cột vào trong phương thức `dropColumn`:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });


<a name="dropping-columns-on-legacy-databases"></a>
#### Dropping Columns On Legacy Databases

Nếu bạn đang chạy phiên bản SQLite cũ hơn phiên bản `3.35.0`, thì bạn phải cài đặt package `doctrine/dbal` thông qua trình quản lý package Composer trước khi có thể sử dụng phương thức `dropColumn`. Việc xóa hoặc sửa nhiều cột trong một lần migration khi sử dụng package này sẽ không được hỗ trợ.

<a name="available-command-aliases"></a>
#### Available Command Aliases

Laravel cung cấp một số phương thức thuận tiện liên quan đến việc xoá các cột phổ biến. Mỗi phương thức này được mô tả trong bảng dưới đây:

Command  |  Description
-------  |  -----------
`$table->dropMorphs('morphable');`  |  Xoá cột `morphable_id` và cột `morphable_type`.
`$table->dropRememberToken();`  |  Xoá cột `remember_token`.
`$table->dropSoftDeletes();`  |  Xoá cột `deleted_at`.
`$table->dropSoftDeletesTz();`  |  Lối tắt của phương thức `dropSoftDeletes()`.
`$table->dropTimestamps();`  |  Xoá cột `created_at` và `updated_at`.
`$table->dropTimestampsTz();` |  Lối tắt của phương thức `dropTimestamps()`.

<a name="indexes"></a>
## Index

<a name="creating-indexes"></a>
### Tạo Index

Schema builder của Laravel có hỗ trợ một số loại index. Ví dụ sau sẽ tạo một cột `email` mới và yêu cầu rằng cột đó phải là unique. Để tạo một index, chúng ta có thể kết hợp thêm phương thức `unique` vào trong định nghĩa của cột:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->unique();
    });

Ngoài ra, bạn có thể tạo index sau khi định nghĩa cột đó. Để làm như vậy, bạn nên gọi phương thức `unique` trên schema builder blueprint. Phương thức này chấp nhận tên của cột mà sẽ được unique index:

    $table->unique('email');

Bạn thậm chí có thể truyền một mảng gồm các cột cho một phương thức index để tạo index gộp (hoặc index hỗn hợp):

    $table->index(['account_id', 'created_at']);

Khi tạo một index, Laravel sẽ tự động tạo tên index dựa trên tên bảng, tên cột và kiểu index, nhưng bạn có thể truyền thêm tham số thứ hai cho phương thức để khai báo tên index của bạn:

    $table->unique('email', 'unique_email');

<a name="available-index-types"></a>
#### Available Index Types

Class schema builder blueprint của Laravel sẽ cung cấp các phương thức khác nhau để tạo ra từng loại index mà được Laravel hỗ trợ. Mỗi phương thức của index chấp nhận một tham số thứ hai tùy chọn để chỉ định tên của index. Nếu bỏ qua tuỳ chọn này, thì tên sẽ được lấy từ tên của (các) bảng và các cột để sử dụng cho index, cũng như loại index. Các phương thức tạo index sẽ được mô tả trong bảng dưới đây:

Command  |  Description
-------  |  -----------
`$table->primary('id');`  |  Thêm một primary key.
`$table->primary(['id', 'parent_id']);`  |  Thêm key hỗn hợp.
`$table->unique('email');`  |  Thêm một unique index.
`$table->index('state');`  |  Thêm một index.
`$table->fullText('body');`  |  Thêm một full text index (MySQL/PostgreSQL).
`$table->fullText('body')->language('english');`  |  Thêm một full text index của một ngôn ngữ cụ thể (PostgreSQL).
`$table->spatialIndex('location');`  |  Thêm một spatial index. (trừ SQLite).

<a name="index-lengths-mysql-mariadb"></a>
#### Index Lengths & MySQL / MariaDB

Mặc định, Laravel sử dụng ký tự mặc định là `utf8mb4`, hỗ trợ lưu trữ cả "biểu tượng cảm xúc" trong cơ sở dữ liệu. Nếu bạn đang chạy phiên bản MySQL cũ hơn phiên bản 5.7.7 hoặc MariaDB cũ hơn phiên bản 10.2.2, bạn có thể cần phải tự cấu hình độ dài mặc định của chuỗi được tạo bởi migration, để MySQL tạo index cho chúng. Bạn có thể cấu hình độ dài mặc định của chuỗi bằng cách gọi phương thức `Schema::defaultStringLength` trong phương thức `boot` của class `AppServiceProvider` của bạn:

    use Illuminate\Support\Facades\Schema;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

Ngoài ra, bạn có thể kích hoạt tùy chọn `innodb_large_prefix` cho cơ sở dữ liệu của bạn. Tham khảo tài liệu của cơ sở dữ liệu của bạn để biết thêm hướng dẫn về cách bật tùy chọn này.

<a name="renaming-indexes"></a>
### Đổi tên Index

Để đổi tên một index, bạn có thể sử dụng phương thức `renameIndex` được cung cấp bởi schema builder blueprint. Phương thức này chấp nhận tên index hiện tại làm tham số đầu tiên và một tên làm tham số thứ hai:

    $table->renameIndex('from', 'to')

> **Warning**
> Nếu ứng dụng của bạn sử dụng cơ sở dữ liệu SQLite, bạn phải cài đặt package `doctrine/dbal` thông qua trình quản lý package Composer trước khi có thể sử dụng phương thức `renameIndex`.

<a name="dropping-indexes"></a>
### Xoá Index

Để xóa một index, bạn có thể khai báo một tên index. Mặc định, Laravel sẽ tự động gán một tên index dựa trên tên bảng và tên cột của index và loại index. Dưới đây là một số ví dụ:

Command  |  Description
-------  |  -----------
`$table->dropPrimary('users_id_primary');`  |  Xoá một primary key từ bảng "users".
`$table->dropUnique('users_email_unique');`  |  Xoá một unique index từ bảng "users".
`$table->dropIndex('geo_state_index');`  |  Xoá một index từ bảng "geo" table.
`$table->dropFullText('posts_body_fulltext');`  |  Drop a full text index from the "posts" table.
`$table->dropSpatialIndex('geo_location_spatialindex');`  |  Xoá một spatial index từ bảng "geo" (trừ SQLite).

Nếu bạn truyền một mảng gồm các cột vào trong một phương thức xoá index, thì quy ước tên index sẽ được tạo dựa trên tên bảng, tên cột, và loại index:

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // Drops index 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### Rằng buộc khoá ngoại

Laravel cũng cung cấp hỗ trợ để tạo các ràng buộc khóa ngoại, được sử dụng để đảm bảo tính toàn vẹn cho cơ sở dữ liệu. Ví dụ: hãy định nghĩa một cột `user_id` trong bảng `posts` là khoá ngoại của cột `id` trong bảng` users`:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('posts', function (Blueprint $table) {
        $table->unsignedBigInteger('user_id');

        $table->foreign('user_id')->references('id')->on('users');
    });

Vì cú pháp này khá dài dòng, nên Laravel đã cung cấp thêm các phương thức bổ sung, ngắn gọn hơn sử dụng nhiều quy ước để cung cấp trải nghiệm tốt hơn cho nhà phát triển. Khi sử dụng phương thức `foreignId` để tạo cột của bạn, ví dụ trên có thể được viết lại như sau:

    Schema::table('posts', function (Blueprint $table) {
        $table->foreignId('user_id')->constrained();
    });

Phương thức `foreignId` sẽ tạo một cột tương ứng với `UNSIGNED BIGINT`, trong khi phương thức `constrained` sẽ sử dụng các quy ước để xác định tên bảng và tên cột đang được tham chiếu. Nếu tên bảng của bạn không phù hợp với các quy ước của Laravel, bạn có thể chỉ định tên bảng bằng cách truyền nó làm một tham số cho phương thức `constrained`:

    Schema::table('posts', function (Blueprint $table) {
        $table->foreignId('user_id')->constrained('users');
    });

Bạn cũng có thể khai báo hành động mong muốn cho các thuộc tính của ràng buộc "khi xóa" hoặc "khi cập nhật":

    $table->foreignId('user_id')
          ->constrained()
          ->onUpdate('cascade')
          ->onDelete('cascade');

Một cú pháp thay thế, hàm ý cũng được cung cấp cho những hành động này:

Method  |  Description
-------  |  -----------
`$table->cascadeOnUpdate();` | Cập nhật theo.
`$table->restrictOnUpdate();`| Hạn chế cập nhật theo.
`$table->cascadeOnDelete();` | Xoá theo.
`$table->restrictOnDelete();`| Hạn chế xoá theo.
`$table->nullOnDelete();`    | Set khoá ngoại là null, nếu khoá chính bị xoá.

Bất kỳ [các sửa đổi bổ sung cho cột](#column-modifiers) sẽ đều phải được gọi trước phương thức `constrained`:

    $table->foreignId('user_id')
          ->nullable()
          ->constrained();

<a name="dropping-foreign-keys"></a>
#### Dropping Foreign Keys

Để xoá khóa ngoại, bạn có thể sử dụng phương thức `dropForeign` và truyền vào tên khóa ngoại sẽ bị xóa dưới dạng tham số. Các ràng buộc khóa ngoại sẽ được sử dụng theo quy ước đặt tên giống với các index. Nói cách khác, tên của ràng buộc khóa ngoại sẽ dựa trên tên của bảng và tên cột trong ràng buộc, theo sau là hậu tố "\_foreign":

    $table->dropForeign('posts_user_id_foreign');

Ngoài ra, bạn có thể truyền một mảng chứa tên các cột chứa khóa ngoại vào phương thức `dropForeign`. Mảng sẽ được chuyển thành tên ràng buộc khóa ngoại bằng cách sử dụng quy ước đặt tên ràng buộc của Laravel:

    $table->dropForeign(['user_id']);

<a name="toggling-foreign-key-constraints"></a>
#### Toggling Foreign Key Constraints

Bạn có thể bật hoặc tắt các ràng buộc khóa ngoại trong migration của bạn bằng cách sử dụng các phương thức sau:

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();

    Schema::withoutForeignKeyConstraints(function () {
        // Constraints disabled within this closure...
    });

> **Warning**
> Mặc định, SQLite sẽ vô hiệu hóa các ràng buộc khóa ngoại. Khi sử dụng SQLite, bạn hãy chắc chắn rằng là [đã bật hỗ trợ khóa ngoại](/docs/{{version}}/database#configuration) trong cấu hình cơ sở dữ liệu của bạn trước khi tạo chúng trong quá trình migration của bạn. Ngoài ra, SQLite chỉ hỗ trợ khóa ngoại khi tạo bảng và [không hỗ trợ khi bảng bị thay đổi](https://www.sqlite.org/omitted.html).

<a name="events"></a>
## Events

Để thuận tiện, mỗi thao tác migration sẽ gửi một [event](/docs/{{version}}/events). Tất cả các event sau đây đều được extend từ class `Illuminate\Database\Events\MigrationEvent`:

 Class | Description
-------|-------
| `Illuminate\Database\Events\MigrationsStarted` | Một tập hợp các file migration sắp được thực hiện. |
| `Illuminate\Database\Events\MigrationsEnded` | Một tập hợp các file migration đã thực hiện xong. |
| `Illuminate\Database\Events\MigrationStarted` | Một file migration sắp được thực hiện. |
| `Illuminate\Database\Events\MigrationEnded` | Một file migration đã thực hiện xong. |
| `Illuminate\Database\Events\SchemaDumped` | A database schema dump has completed. |
| `Illuminate\Database\Events\SchemaLoaded` | An existing database schema dump has loaded. |
