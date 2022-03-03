# Database: Migrations

- [Giới thiệu](#introduction)
- [Tạo Migration](#generating-migrations)
- [Cấu trúc Migration](#migration-structure)
- [Chạy Migration](#running-migrations)
    - [Rollback Migration](#rolling-back-migrations)
- [Table](#tables)
    - [Tạo Tables](#creating-tables)
    - [Đổi tên / Xoá Table](#renaming-and-dropping-tables)
- [Column](#columns)
    - [Tạo Column](#creating-columns)
    - [Column Modifiers](#column-modifiers)
    - [Sửa Column](#modifying-columns)
    - [Xoá Column](#dropping-columns)
- [Index](#indexes)
    - [Tạo Index](#creating-indexes)
    - [Đổi tên Index](#renaming-indexes)
    - [Xoá Index](#dropping-indexes)
    - [Rằng buộc khoá ngoại](#foreign-key-constraints)

<a name="introduction"></a>
## Giới thiệu

Migration giống như là một version control cho cơ sở dữ liệu, cho phép team của bạn dễ dàng sửa và chia sẻ các database schema của application. Migration thường được kết hợp với schema builder của Laravel để xây dựng database schema cho application của bạn. Nếu bạn đã từng phải nói với các thành viên trong team của bạn là tự thêm một cột vào database schema ở local của họ, thì bạn đã từng phải gặp phải vấn đề về migration cơ sở dữ liệu.

Laravel [facade](/docs/{{version}}/facades) `Schema` cung cấp một cách để tạo và thao tác với các bảng trên tất cả các hệ thống cơ sở dữ liệu được hỗ trợ bởi Laravel mà không cần quan tâm về loại database.

<a name="generating-migrations"></a>
## Tạo Migration

Để tạo một migration, hãy sử dụng [Lệnh Artisan](/docs/{{version}}/artisan) `make:migration`:

    php artisan make:migration create_users_table

File migration mới sẽ được lưu trong thư mục `database/migrations` của bạn. Mỗi tên file migration đều có chứa một dấu mốc timestamp cho phép Laravel xác định thứ tự migration.

Các tùy chọn `--table` và `--create` cũng có thể được sử dụng để chỉ định tên bảng và migration có tạo một bảng mới hay không. Những tùy chọn này sẽ được điền trước vào file migration sẽ được tạo cùng với tên bảng đã được chỉ định:

    php artisan make:migration create_users_table --create=users

    php artisan make:migration add_votes_to_users_table --table=users

Nếu bạn muốn khai báo một đường dẫn output tùy biến cho file migration được tạo, bạn có thể sử dụng tùy chọn `--path` khi chạy lệnh `make:migration`. Đường dẫn tuỳ biến này phải được bắt đầu từ đường dẫn root của application của bạn.

<a name="migration-structure"></a>
## Cấu trúc Migration

Một class migration sẽ chứa hai phương thức: `up` và `down`. Phương thức `up` sẽ được sử dụng để thêm một bảng, một cột hoặc một index mới vào cơ sở dữ liệu của bạn, trong khi phương thức `down` sẽ quay ngược lại các hành động mà được thực hiện bởi phương thức `up`.

Trong cả hai phương thức này, bạn đều có thể sử dụng schema builder của Laravel để tạo và sửa các bảng một cách rõ ràng. Để tìm hiểu về tất cả các phương thức có sẵn trong `Schema` builder, [hãy xem tài liệu về nó](#creating-tables). Ví dụ, migration mẫu này sẽ tạo ra một bảng `flights`:

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
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
    }

<a name="running-migrations"></a>
## Chạy Migration

Để chạy tất cả các migration của bạn, hãy chạy lệnh Artisan `migrate`:

    php artisan migrate

> {note} Nếu bạn đang sử dụng [máy ảo Homestead](/docs/{{version}}/homestead), bạn nên chạy lệnh này từ bên trong máy ảo của bạn.

#### Forcing Migrations To Run In Production

Một số hành động migration có thể là nguy hiểm, có nghĩa là chúng có thể khiến bạn mất dữ liệu. Để bảo vệ bạn khỏi việc chạy các lệnh này đối với cơ sở dữ liệu production, bạn sẽ được nhắc xác nhận trước khi chạy các lệnh được này. Để bắt các lệnh này chạy mà không nhắc xác nhận, hãy sử dụng cờ `--force`:

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### Rollback Migration

Để rollback về migration mới nhất, bạn có thể sử dụng lệnh `rollback`. Lệnh này sẽ rollback lại "batch" migration cuối cùng mà bạn dùng, nó có thể có chứa nhiều file migration:

    php artisan migrate:rollback

Bạn có thể muốn rollback lại một số migration cần thiết bằng cách cung cấp thêm tùy chọn `step` cho lệnh `rollback`. Ví dụ: lệnh sau sẽ rollback lại năm lần trước khi đến batch cuối cùng:

    php artisan migrate:rollback --step=5

Lệnh `migrate:reset` sẽ rollback lại tất cả các migration của application của bạn:

    php artisan migrate:reset

#### Rollback & Migrate In Single Command

Lệnh `migrate:refresh` sẽ rollback lại tất cả các migration của bạn và sau đó thực hiện lại lệnh `migrate`. Lệnh này sẽ tạo lại toàn bộ cơ sở dữ liệu của bạn:

    php artisan migrate:refresh

    // Refresh the database and run all database seeds...
    php artisan migrate:refresh --seed

Bạn có thể rollback và migrate lại một số migration cần thiết bằng cách cung cấp tùy chọn `step` cho lệnh `refresh`. Ví dụ: lệnh sau sẽ rollback và migrate lại năm lần trước so với migration gần nhất:

    php artisan migrate:refresh --step=5

#### Drop All Tables & Migrate

Lệnh `migrate:fresh` sẽ xóa tất cả các bảng ra khỏi cơ sở dữ liệu và sau đó thực thi lại lệnh `migrate`:

    php artisan migrate:fresh

    php artisan migrate:fresh --seed

<a name="tables"></a>
## Table

<a name="creating-tables"></a>
### Tạo Tables

Để tạo một bảng cơ sở dữ liệu mới, hãy sử dụng phương thức `create` trên facade `Schema`. Phương thức `create` chấp nhận hai tham số.Tham số đầu tiên là tên của bảng, trong khi tham số thứ hai là một `Closure` nhận vào một đối tượng `Blueprint` có thể được sử dụng để định nghĩa một bảng mới:

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Khi tạo bảng, bạn có thể sử dụng bất kỳ [column methods](#creating-columns) nào của schema builder để định nghĩa các cột của bảng.

#### Checking For Table / Column Existence

Bạn có thể dễ dàng kiểm tra sự tồn tại của một bảng hoặc một cột bằng các phương thức `hasTable` và `hasColumn`:

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### Database Connection & Table Options

Nếu bạn muốn thực hiện một schema trên một kết nối cơ sở dữ liệu không phải là kết nối mặc định của bạn, hãy sử dụng phương thức `connection`:

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Bạn có thể sử dụng các lệnh sau trên schema builder để định nghĩa các tùy chọn cho bảng:

Command  |  Description
-------  |  -----------
`$table->engine = 'InnoDB';`  |  Khai báo engine cho table storage (MySQL).
`$table->charset = 'utf8';`  |  Khai báo character mặc định cho table (MySQL).
`$table->collation = 'utf8_unicode_ci';`  |  Khai báo collation mặc định cho table (MySQL).
`$table->temporary();`  |  Tạo một table tạm thời (ngoại trừ SQL Server).

<a name="renaming-and-dropping-tables"></a>
### Đổi tên / Xoá Table

Để đổi tên một bảng đã tồn tại trong cơ sở dữ liệu, hãy sử dụng phương thức `rename`:

    Schema::rename($from, $to);

Để xóa một bảng hiện có, bạn có thể sử dụng các phương thức `drop` hoặc `dropIfExists`:

    Schema::drop('users');

    Schema::dropIfExists('users');

#### Renaming Tables With Foreign Keys

Trước khi đổi tên một bảng, bạn nên kiểm tra khóa ngoại trỏ đến bảng đó đã có trong file migration thay vì để Laravel tự gán tên dựa trên quy ước của nó. Nếu không, tên khóa ngoại đó sẽ tham chiếu đến tên bảng cũ.

<a name="columns"></a>
## Column

<a name="creating-columns"></a>
### Tạo Column

Phương thức `table` trên facade `Schema` có thể được sử dụng để cập nhật các bảng đã tồn tại. Giống như phương thức `create`, phương thức `table` chấp nhận hai tham số: một là tên một bảng và hai là một `Closure` nhận vào một instance `Blueprint` mà bạn có thể sử dụng nó để thêm các cột vào trong bảng:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });

#### Available Column Types

Schema builder cũng sẽ chứa nhiều loại cột mà bạn có thể khai báo khi xây dựng các bảng cho bạn:

Lệnh  |  Mô tả
-------  |  -----------
`$table->bigIncrements('id');`  |  Tương đương với cột (primary key) tự động tăng và là số dương kiểu BIGINT.
`$table->bigInteger('votes');`  |  Tương đương với cột kiểu BIGINT.
`$table->binary('data');`  |  Tương đương với cột kiểu BLOB.
`$table->boolean('confirmed');`  |  Tương đương với cột kiểu BOOLEAN.
`$table->char('name', 100);`  |  Tương đương với cột kiểu CHAR.
`$table->date('created_at');`  |  Tương đương với cột kiểu DATE.
`$table->dateTime('created_at');`  |  Tương đương với cột kiểu DATETIME.
`$table->dateTimeTz('created_at');`  |  Tương đương với cột kiểu DATETIME (cùng timezone).
`$table->decimal('amount', 8, 2);`  |  Tương đương với cột kiểu DECIMAL với tổng số ký tự và độ dài sau dấu phẩy.
`$table->double('amount', 8, 2);`  |  Tương đương với cột kiểu DDOUBLE với tổng số ký tự và độ dài sau dấu phẩy.
`$table->enum('level', ['easy', 'hard']);`  |  Tương đương với cột kiểu ENUM.
`$table->float('amount', 8, 2);`  |  Tương đương với cột kiểu FLOAT với tổng số ký tự và độ dài sau dấu phẩy.
`$table->geometry('positions');`  |  Tương đương với cột kiểu GEOMETRY.
`$table->geometryCollection('positions');`  |  Tương đương với cột kiểu GEOMETRYCOLLECTION.
`$table->increments('id');`  |  Tương đương với cột (primary key) tự động tăng và là số dương kiểu INTEGER.
`$table->integer('votes');`  |  Tương đương với cột kiểu INTEGER.
`$table->ipAddress('visitor');`  |  Tương đương với cột kiểu IP address.
`$table->json('options');`  |  Tương đương với cột kiểu JSON.
`$table->jsonb('options');`  |  Tương đương với cột kiểu JSONB.
`$table->lineString('positions');`  |  Tương đương với cột kiểu LINESTRING.
`$table->longText('description');`  |  Tương đương với cột kiểu LONGTEXT.
`$table->macAddress('device');`  |  Tương đương với cột kiểu MAC address.
`$table->mediumIncrements('id');`  |  Tương đương với cột (primary key) tự động tăng và là số dương kiểu MEDIUMINT.
`$table->mediumInteger('votes');`  |  Tương đương với cột kiểu MEDIUMINT.
`$table->mediumText('description');`  |  Tương đương với cột kiểu MEDIUMTEXT.
`$table->morphs('taggable');`  |  Thêm cột `taggable_id` kiểu BIGINT luôn dương và cột `taggable_type` kiểu VARCHARs.
`$table->multiLineString('positions');`  |  Tương đương với cột kiểu MULTILINESTRING.
`$table->multiPoint('positions');`  |  Tương đương với cột kiểu MULTIPOINT.
`$table->multiPolygon('positions');`  |  Tương đương với cột kiểu MULTIPOLYGON.
`$table->nullableMorphs('taggable');`  |  Phiên bản thêm giá trị nullable vào cột `morphs()`.
`$table->nullableTimestamps();`  |  Lối tắt của phương thức `timestamps()`.
`$table->point('position');`  |  Tương đương với cột kiểu POINT.
`$table->polygon('positions');`  |  Tương đương với cột kiểu POLYGON.
`$table->rememberToken();`  |  Thêm cột `remember_token` VARCHAR(100) cho phép nullable.
`$table->smallIncrements('id');`  |  Tương đương với cột (primary key) tự động tăng và là số dương kiểu SMALLINT.
`$table->smallInteger('votes');`  |  Tương đương với cột kiểu SMALLINT.
`$table->softDeletes();`  |  Thêm cột `deleted_at` kiểu TIMESTAMP cho phép nullable cho soft deletes.
`$table->softDeletesTz();`  |  Thêm cột `deleted_at` kiểu TIMESTAMP (cùng timezone) cho phép nullable cho soft deletes.
`$table->string('name', 100);`  |  Tương đương với cột kiểu VARCHAR cùng một tuỳ chọn độ dài.
`$table->text('description');`  |  Tương đương với cột kiểu TEXT.
`$table->time('sunrise');`  |  Tương đương với cột kiểu TIME.
`$table->timeTz('sunrise');`  |  Tương đương với cột kiểu TIME (cùng timezone).
`$table->timestamp('added_on');`  |  Tương đương với cột kiểu TIMESTAMP.
`$table->timestampTz('added_on');`  |  Tương đương với cột kiểu TIMESTAMP (cùng timezone).
`$table->timestamps();`  |  Thêm cột nullable `created_at` và `updated_at` kiểu TIMESTAMP.
`$table->timestampsTz();`  |  Thêm cột nullable `created_at` và `updated_at` kiểu TIMESTAMP (cùng timezone).
`$table->tinyIncrements('id');`  |  Tương đương với cột (primary key) tự động tăng và là số dương kiểu TINYINT.
`$table->tinyInteger('votes');`  |  Tương đương với cột kiểu TINYINT.
`$table->unsignedBigInteger('votes');`  |  Tương đương với cột kiểu BIGINT luôn dương.
`$table->unsignedDecimal('amount', 8, 2);`  |  Tương đương với cột kiểu DECIMAL luôn dương với tổng số ký tự và độ dài sau dấu phẩy.
`$table->unsignedInteger('votes');`  |  Tương đương với cột kiểu INTEGER luôn dương.
`$table->unsignedMediumInteger('votes');`  |  Tương đương với cột kiểu MEDIUMINT luôn dương.
`$table->unsignedSmallInteger('votes');`  |  Tương đương với cột kiểu SMALLINT luôn dương.
`$table->unsignedTinyInteger('votes');`  |  Tương đương với cột kiểu TINYINT luôn dương.
`$table->uuid('id');`  |  Tương đương với cột kiểu UUID.
`$table->year('birth_year');`  |  Tương đương với cột kiểu YEAR.

<a name="column-modifiers"></a>
### Column Modifiers

Ngoài các loại cột được liệt kê ở trên, có một số "modifiers" cột mà bạn có thể sử dụng trong khi thêm một cột vào trong bảng cơ sở dữ liệu. Ví dụ, để tạo một cột chấp nhận "nullable", bạn có thể sử dụng phương thức `nullable`:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

Dưới đây là danh sách tất cả các column modifier có sẵn. Danh sách này không bao gồm [index modifiers](#creating-indexes):

Modifier  |  Mô tả
--------  |  -----------
`->after('column')`  |  Set một column vào "sau" một column khác (MySQL)
`->autoIncrement()`  |  Set một cột kiểu INTEGER là tự động tăng (primary key)
`->charset('utf8')`  |  Khai báo character set cho cột (MySQL)
`->collation('utf8_unicode_ci')`  |  Khai báo collation cho cột (MySQL/SQL Server)
`->comment('my comment')`  |  Thêm comment vào một column (MySQL/PostgreSQL)
`->default($value)`  |  Khai báo giá trị "default" cho cột
`->first()`  |  Set một column vào vị trí "đầu tiên" trong table (MySQL)
`->nullable($value = true)`  |  Cho phép giá trị mặc định là NULL khi tạo bản ghi mới
`->storedAs($expression)`  |  Tạo một cột lấy data từ cột khác lưu vào chính nó (MySQL)
`->unsigned()`  |  Set một cột kiểu INTEGER là luôn dương (MySQL)
`->useCurrent()`  |  Set cột TIMESTAMP dùng CURRENT_TIMESTAMP làm giá trị mặc định
`->virtualAs($expression)`  |  Tạo một cột lấy data từ cột khác nhưng không được lưu trữ (MySQL)
`->generatedAs($expression)`  |  Tạo một cột identity với tùy chọn tăng dần được chỉ định (PostgreSQL)
`->always()`  |  Định nghĩa mức độ ưu tiên của các giá trị tăng dần so với giá trị đầu vào cho một cột identity (PostgreSQL)

<a name="modifying-columns"></a>
### Sửa Column

#### Prerequisites

Trước khi sửa một cột, bạn hãy chắc chắn là bạn đã thêm library `doctrine/dbal` vào trong file `composer.json` của bạn. Thư viện Doctrine DBAL được sử dụng để xác định trạng thái hiện tại của cột và tạo ra các truy vấn SQL cần thiết để thực hiện các khai báo điều chỉnh cho các cột đó:

    composer require doctrine/dbal

#### Updating Column Attributes

Phương thức `change` cho phép bạn sửa một số loại cột hiện có thành một loại mới hoặc sửa các thuộc tính của cột. Ví dụ: bạn có thể muốn tăng kích thước của cột string. Để xem phương thức `change` hoạt động như thế nào, hãy thử tăng kích thước của cột `name` từ 25 lên 50:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

Chúng ta cũng có thể sửa một cột thành nullable:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} Chỉ có thể "thay đổi" các loại cột sau: bigInteger, binary, boolean, date, dateTime, dateTimeTz, decimal, integer, json, longText, mediumText, smallInteger, string, text, time, unsignedBigInteger, unsignedInteger và unsignedSmallInteger.

#### Renaming Columns

Để đổi tên một cột, bạn có thể sử dụng phương thức `renameColumn` trong Schema builder. Trước khi đổi tên một cột, hãy nhớ thêm library `doctrine/dbal` vào trong file `composer.json` của bạn:

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

> {note} Việc đổi tên cột trong một bảng mà có chứa một cột khác kiểu `enum` thì sẽ không thể đổi được.

<a name="dropping-columns"></a>
### Xoá Column

Để xóa một cột, hãy sử dụng phương thức `dropColumn` trong Schema builder. Trước khi xóa một cột ra khỏi cơ sở dữ liệu SQLite, bạn sẽ cần thêm library `doctrine/dbal` vào trong file `composer.json` của bạn và chạy lệnh `composer update` trong terminal để cài đặt thư viện:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

Bạn có thể xóa nhiều cột từ một bảng bằng cách truyền một mảng gồm tên các cột vào trong phương thức `dropColumn`:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> {note} Nếu bạn đang sử dụng cơ sở dữ liệu SQLite thì việc xóa hoặc sửa nhiều cột trong một file migration sẽ không được hỗ trợ.

#### Available Command Aliases

Command  |  Description
-------  |  -----------
`$table->dropRememberToken();`  |  Xoá cột `remember_token`.
`$table->dropSoftDeletes();`  |  Xoá cột `deleted_at`.
`$table->dropSoftDeletesTz();`  |  Lối tắt của phương thức `dropSoftDeletes()`.
`$table->dropTimestamps();`  |  Xoá cột `created_at` và `updated_at`.
`$table->dropTimestampsTz();` |  Lối tắt của phương thức `dropTimestamps()`.

<a name="indexes"></a>
## Index

<a name="creating-indexes"></a>
### Tạo Index

Schema builder có hỗ trợ một số loại cindex. Trước tiên, hãy xem một ví dụ khai báo giá trị của một cột phải là unique. Để tạo một index, chúng ta có thể kết hợp thêm phương thức `unique` vào trong định nghĩa của cột:

    $table->string('email')->unique();

Ngoài ra, bạn có thể tạo index sau khi định nghĩa cột. Ví dụ:

    $table->unique('email');

Bạn thậm chí có thể truyền một mảng gồm các cột cho một phương thức index để tạo index gộp (hoặc index hỗn hợp):

    $table->index(['account_id', 'created_at']);

Laravel sẽ tự động tạo một tên index phù hợp, nhưng bạn có thể truyền thêm tham số thứ hai cho phương thức để khai báo tên đó:

    $table->unique('email', 'unique_email');

#### Available Index Types

Mỗi phương thức của index chấp nhận một đối số thứ hai tùy chọn để chỉ định tên của index. Nếu bỏ qua tuỳ chọn này, thì tên sẽ được lấy từ tên của (các) bảng và cột.

Command  |  Description
-------  |  -----------
`$table->primary('id');`  |  Thêm một primary key.
`$table->primary(['id', 'parent_id']);`  |  Thêm key hỗn hợp.
`$table->unique('email');`  |  Thêm một unique index.
`$table->index('state');`  |  Thêm một index.
`$table->spatialIndex('location');`  |  Thêm một spatial index. (trừ SQLite)

#### Index Lengths & MySQL / MariaDB

Laravel sử dụng ký tự mặc định là `utf8mb4`, hỗ trợ lưu trữ cả "biểu tượng cảm xúc" trong cơ sở dữ liệu. Nếu bạn đang chạy phiên bản MySQL cũ hơn phiên bản 5.7.7 hoặc MariaDB cũ hơn phiên bản 10.2.2, bạn có thể cần phải tự cấu hình độ dài mặc định của chuỗi được tạo bởi migration, để MySQL tạo index cho chúng. Bạn có thể cấu hình điều này bằng cách gọi phương thức `Schema::defaultStringLength` trong `AppServiceProvider` của bạn:

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

Để đổi tên một index, bạn có thể sử dụng phương thức `renameIndex`. Phương thức này chấp nhận tên index hiện tại làm đối số đầu tiên và tên mong muốn làm đối số thứ hai:

    $table->renameIndex('from', 'to')

<a name="dropping-indexes"></a>
### Xoá Index

Để xóa một index, bạn có thể khai báo một tên index. Mặc định, Laravel sẽ tự động gán một tên phù hợp cho các index. Chỉ cần nối tên bảng và tên cột index và loại index. Dưới đây là một số ví dụ:

Command  |  Description
-------  |  -----------
`$table->dropPrimary('users_id_primary');`  |  Xoá một primary key từ bảng "users".
`$table->dropUnique('users_email_unique');`  |  Xoá một unique index từ bảng "users".
`$table->dropIndex('geo_state_index');`  |  Xoá một index từ bảng "geo" table.
`$table->dropSpatialIndex('geo_location_spatialindex');`  |  Xoá một spatial index từ bảng "geo" (trừ SQLite).

Nếu bạn truyền một mảng gồm các cột vào trong một phương thức xoá index, thì quy ước tên index sẽ được tạo dựa trên tên bảng, tên cột và loại khóa:

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // Drops index 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### Rằng buộc khoá ngoại

Laravel cũng cung cấp hỗ trợ để tạo các ràng buộc khóa ngoại, được sử dụng để đảm bảo tính toàn vẹn cho cơ sở dữ liệu. Ví dụ: hãy định nghĩa một cột `user_id` trong bảng `posts` là khoá ngoại của cột `id` trong bảng` users`:

    Schema::table('posts', function (Blueprint $table) {
        $table->unsignedInteger('user_id');

        $table->foreign('user_id')->references('id')->on('users');
    });

Bạn cũng có thể khai báo hành động mong muốn cho các thuộc tính của ràng buộc "khi xóa" hoặc "khi cập nhật":

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

Để xoá khóa ngoại, bạn có thể sử dụng phương thức `dropForeign`. Các ràng buộc khóa ngoại sẽ được sử dụng theo quy ước đặt tên giống với các index. Vì vậy, chúng ta sẽ nối tên bảng và tên cột trong ràng buộc, sau đó thêm hậu tố "\_foreign":

    $table->dropForeign('posts_user_id_foreign');

Hoặc, bạn có thể truyền một mảng các giá trị sẽ tự động sử dụng quy ước tên ràng buộc khi xoá:

    $table->dropForeign(['user_id']);

Bạn có thể bật hoặc tắt các ràng buộc khóa ngoại trong migration của bạn bằng cách sử dụng các phương thức sau:

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();

> {note} Mặc định, SQLite sẽ vô hiệu hóa các ràng buộc khóa ngoại. Khi sử dụng SQLite, bạn hãy chắc chắn rằng là [đã bật hỗ trợ khóa ngoại](/docs/{{version}}/database#configuration) trong cấu hình cơ sở dữ liệu của bạn trước khi tạo chúng trong quá trình migration của bạn.
