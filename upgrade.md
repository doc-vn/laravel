# Upgrade Guide

- [Nâng cấp đến 11.0 từ 10.x](#upgrade-11.0)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Updating Dependencies](#updating-dependencies)
- [Application Structure](#application-structure)
- [Floating-Point Types](#floating-point-types)
- [Modifying Columns](#modifying-columns)
- [SQLite Minimum Version](#sqlite-minimum-version)
- [Updating Sanctum](#updating-sanctum)

</div>

<a name="medium-impact-changes"></a>
## Những thay đổi có tác động trung bình

<div class="content-list" markdown="1">

- [Carbon 3](#carbon-3)
- [Password Rehashing](#password-rehashing)
- [Per-Second Rate Limiting](#per-second-rate-limiting)
- [Spatie Once Package](#spatie-once-package)

</div>

<a name="low-impact-changes"></a>
## Những thay đổi có tác động thấp

<div class="content-list" markdown="1">

- [Doctrine DBAL Removal](#doctrine-dbal-removal)
- [Eloquent Model `casts` Method](#eloquent-model-casts-method)
- [Spatial Types](#spatial-types)
- [The `Enumerable` Contract](#the-enumerable-contract)
- [The `UserProvider` Contract](#the-user-provider-contract)
- [The `Authenticatable` Contract](#the-authenticatable-contract)

</div>

<a name="upgrade-11.0"></a>
## Nâng cấp đến 11.0 từ 10.x

<a name="estimated-upgrade-time-30-minutes"></a>
#### Estimated Upgrade Time: 15 Minutes

> [!NOTE]
> Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này mới có thể thực sự ảnh hưởng đến application của bạn. Bạn muốn tiết kiệm thời gian? Bạn có thể sử dụng [Laravel Shift](https://laravelshift.com/) để giúp tự động hóa việc nâng cấp ứng dụng của bạn.

<a name="updating-dependencies"></a>
### Updating Dependencies

**Likelihood Of Impact: High**

#### PHP 8.2.0 Required

Laravel hiện yêu cầu PHP 8.2.0 trở lên.

#### curl 7.34.0 Required

HTTP client của Laravel hiện yêu cầu curl 7.34.0 trở lên.

#### Composer Dependencies

Bạn nên cập nhật các library sau vào file `composer.json` của ứng dụng:

<div class="content-list" markdown="1">

- `laravel/framework` to `^11.0`
- `nunomaduro/collision` to `^8.1`
- `laravel/breeze` to `^2.0` (If installed)
- `laravel/cashier` to `^15.0` (If installed)
- `laravel/dusk` to `^8.0` (If installed)
- `laravel/jetstream` to `^5.0` (If installed)
- `laravel/octane` to `^2.3` (If installed)
- `laravel/passport` to `^12.0` (If installed)
- `laravel/sanctum` to `^4.0` (If installed)
- `laravel/scout` to `^10.0` (If installed)
- `laravel/spark-stripe` to `^5.0` (If installed)
- `laravel/telescope` to `^5.0` (If installed)
- `livewire/livewire` to `^3.4` (If installed)
- `inertiajs/inertia-laravel` to `^1.0` (If installed)

</div>

Nếu ứng dụng của bạn đang sử dụng Laravel Cashier Stripe, Passport, Sanctum, Spark Stripe, hoặc Telescope, bạn sẽ cần phải export các migration của chúng vào trong ứng dụng của bạn. Cashier Stripe, Passport, Sanctum, Spark Stripe, và Telescope **sẽ không còn tự động load các migration từ thư mục migration của chúng**. Do đó, bạn nên chạy lệnh sau để export các migration của package trên vào ứng dụng của bạn:

```bash
php artisan vendor:publish --tag=cashier-migrations
php artisan vendor:publish --tag=passport-migrations
php artisan vendor:publish --tag=sanctum-migrations
php artisan vendor:publish --tag=spark-migrations
php artisan vendor:publish --tag=telescope-migrations
```

Ngoài ra, bạn nên xem thêm các hướng dẫn nâng cấp cho từng package này để đảm bảo bạn biết được hết mọi thay đổi quan trọng khác:

- [Laravel Cashier Stripe](#cashier-stripe)
- [Laravel Passport](#passport)
- [Laravel Sanctum](#sanctum)
- [Laravel Spark Stripe](#spark-stripe)
- [Laravel Telescope](#telescope)

Nếu bạn tự cài đặt Laravel Installer, thì bạn nên cập nhật Laravel Installer thông qua Composer:

```bash
composer global require laravel/installer:^5.6
```

Cuối cùng, bạn có thể gỡ bỏ dependency `doctrine/dbal` của Composer nếu trước đó bạn đã thêm nó vào ứng dụng của bạn, vì Laravel không còn phụ thuộc vào package này nữa.

<a name="application-structure"></a>
### Application Structure

Laravel 11 giới thiệu một cấu trúc file ứng dụng mặc định mới với ít file hơn. Cụ thể, các ứng dụng Laravel mới sẽ chứa ít file service provider, middleware và file cấu hình hơn.

Tuy nhiên, chúng tôi **không khuyến khích** các ứng dụng Laravel 10 khi nâng lên Laravel 11 cố gắng chuyển đổi cấu trúc ứng dụng, vì Laravel 11 đã được tinh chỉnh kỹ lưỡng để hỗ trợ cả cấu trúc ứng dụng của Laravel 10.

<a name="authentication"></a>
### Authentication

<a name="password-rehashing"></a>
#### Password Rehashing

**Likelihood Of Impact: Low**

Laravel 11 sẽ tự động hash lại mật khẩu của người dùng trong quá trình xác thực nếu "work factor" của thuật toán hashing khác với "work factor" của mật khẩu của người dùng.

Thông thường, điều này sẽ không gây ảnh hưởng đến ứng dụng của bạn; tuy nhiên, nếu trường "password" trong model `User` của bạn có tên khác, khác với tên `password`, thì bạn nên chỉ định tên của trường đó thông qua thuộc tính `authPasswordName` của model:

    protected $authPasswordName = 'custom_password_field';

Ngoài ra, bạn có thể tắt tính năng hash lại mật khẩu này bằng cách thêm tùy chọn `rehash_on_login` vào file cấu hình `config/hashing.php` của ứng dụng:

    'rehash_on_login' => false,

<a name="the-user-provider-contract"></a>
#### The `UserProvider` Contract

**Likelihood Of Impact: Low**

Contract `Illuminate\Contracts\Auth\UserProvider` đã được thêm một phương thức `rehashPasswordIfRequired` mới. Phương thức này sẽ chịu trách nhiệm hash lại và lưu mật khẩu của người dùng vào database khi work factor của thuật toán hashing trong ứng dụng được thay đổi.

Nếu ứng dụng hoặc package của bạn đang định nghĩa một class implement từ interface này, bạn nên thêm phương thức `rehashPasswordIfRequired` mới vào implementation của bạn. Bạn có thể tham khảo implementation mẫu trong class `Illuminate\Auth\EloquentUserProvider`:

```php
public function rehashPasswordIfRequired(Authenticatable $user, array $credentials, bool $force = false);
```

<a name="the-authenticatable-contract"></a>
#### The `Authenticatable` Contract

**Likelihood Of Impact: Low**

Contract `Illuminate\Contracts\Auth\Authenticatable` đã được thêm một phương thức `getAuthPasswordName` mới. Phương thức này chịu trách nhiệm trả về tên cột mật khẩu của model xác thực của bạn.

Nếu ứng dụng hoặc package của bạn đang định nghĩa một class được implement từ interface này, bạn nên thêm phương thức `getAuthPasswordName` mới vào implementation của bạn:

```php
public function getAuthPasswordName()
{
    return 'password';
}
```

Model `User` mặc định có sẵn của Laravel sẽ tự động nhận được phương thức này vì phương thức này đã có sẵn trong trait `Illuminate\Auth\Authenticatable`.

<a name="the-authentication-exception-class"></a>
#### The `AuthenticationException` Class

**Likelihood Of Impact: Very Low**

Phương thức `redirectTo` của class `Illuminate\Auth\AuthenticationException` sẽ yêu cầu một instance `Illuminate\Http\Request` làm tham số đầu tiên. Nếu bạn đang xử lý exception này và gọi phương thức `redirectTo`, thì bạn nên cập nhật code của bạn cho phù hợp:

```php
if ($e instanceof AuthenticationException) {
    $path = $e->redirectTo($request);
}
```

<a name="email-verification-notification-on-registration"></a>
#### Email Verification Notification on Registration

**Likelihood Of Impact: Very Low**

Listener `SendEmailVerificationNotification` sẽ tự động đăng ký cho event `Registered` nếu nó chưa được đăng ký bởi `EventServiceProvider` của ứng dụng. Nếu `EventServiceProvider` của ứng dụng không đăng ký listener này và bạn không muốn Laravel tự động đăng ký nó, bạn nên định nghĩa một phương thức trống `configureEmailVerification` trong `EventServiceProvider` của ứng dụng:

```php
protected function configureEmailVerification()
{
    // ...
}
```

<a name="cache"></a>
### Cache

<a name="cache-key-prefixes"></a>
#### Cache Key Prefixes

**Likelihood Of Impact: Very Low**

Trước đây, nếu một tiền tố key cache được định nghĩa cho các cache store DynamoDB, Memcached, hoặc Redis, Laravel sẽ tự động thêm một dấu `:` vào tiền tố đó. Bây giờ trong Laravel 11, tiền tố key cache sẽ không còn được thêm hậu tố `:` này nữa. Nếu bạn muốn duy trì hành vi thêm tiền tố như trước đây, bạn có thể tự thêm dấu `:` vào tiền tố key cache của bạn như bình thường.

<a name="collections"></a>
### Collections

<a name="the-enumerable-contract"></a>
#### The `Enumerable` Contract

**Likelihood Of Impact: Low**

Phương thức `dump` của contract `Illuminate\Support\Enumerable` đã được cập nhật để chấp nhận một tham số variadic `...$args`. Nếu bạn đang implement interface này, bạn nên cập nhật implementation của bạn cho phù hợp:

```php
public function dump(...$args);
```

<a name="database"></a>
### Database

<a name="sqlite-minimum-version"></a>
#### SQLite 3.26.0+

**Likelihood Of Impact: High**

Nếu ứng dụng của bạn đang sử dụng cơ sở dữ liệu SQLite, thì SQLite 3.26.0 hoặc cao hơn là bắt buộc.

<a name="eloquent-model-casts-method"></a>
#### Eloquent Model `casts` Method

**Likelihood Of Impact: Low**

Class Eloquent model base hiện đã định nghĩa một phương thức `casts` để hỗ trợ việc định nghĩa các attribute cast. Nếu một trong các model của ứng dụng của bạn đang định nghĩa một quan hệ `casts`, nó có thể xung đột với phương thức `casts` hiện đã có trong class Eloquent model base.

<a name="modifying-columns"></a>
#### Modifying Columns

**Likelihood Of Impact: High**

Khi thay đổi một cột, bây giờ bạn phải thêm tất cả các modifier mà bạn muốn giữ lại trên định nghĩa cột sau khi nó được thay đổi. Bất kỳ thuộc tính nào bị thiếu sẽ bị xoá. Ví dụ: để giữ lại các thuộc tính `unsigned`, `default`, và `comment`, bạn phải gọi từng modifier khi thay đổi cột, ngay cả khi các thuộc tính đó đã được gán cho cột bởi một migration trước đó.

Ví dụ, hãy tưởng tượng bạn có một migration tạo một cột `votes` với các thuộc tính `unsigned`, `default`, và `comment`:

```php
Schema::create('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('The vote count');
});
```

Sau đó, bạn viết một migration thay đổi cột để trở thành `nullable`:

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->nullable()->change();
});
```

Trong Laravel 10, migration này sẽ giữ lại các thuộc tính `unsigned`, `default`, và `comment` trên cột. Tuy nhiên, trong Laravel 11, migration hiện tại cũng phải chứa tất cả các thuộc tính đã được định nghĩa trước đó trên cột. Nếu không, chúng sẽ bị xóa bỏ:

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')
        ->unsigned()
        ->default(1)
        ->comment('The vote count')
        ->nullable()
        ->change();
});
```

Phương thức `change` sẽ không thay đổi các index của cột. Do đó, bạn có thể sử dụng các index modifier để thêm hoặc xóa một index khi thay đổi cột:

```php
// Add an index...
$table->bigIncrements('id')->primary()->change();

// Drop an index...
$table->char('postal_code', 10)->unique(false)->change();
```

Nếu bạn không muốn phải cập nhật hết tất cả các migration "change" hiện có, có trong ứng dụng của bạn để giữ lại các thuộc tính hiện có của cột, bạn có thể cần [squash các migration của bạn](/docs/{{version}}/migrations#squashing-migrations):

```bash
php artisan schema:dump
```

Sau khi các migration của bạn đã được squash, Laravel sẽ "migrate" database bằng cách sử dụng file schema được tạo ra trước khi chạy bất kỳ migration mới nào khác.

<a name="floating-point-types"></a>
#### Floating-Point Types

**Likelihood Of Impact: High**

Các kiểu cột migration `double` và `float` đã được viết lại để nhất quán trên tất cả các cơ sở dữ liệu.

Kiểu cột `double` hiện tại sẽ tạo ra một cột tương đương `DOUBLE` mà không có tổng các chữ số và số thập phân (các chữ số sau dấu phẩy), đây là cú pháp SQL tiêu chuẩn. Do đó, bạn có thể xoá các tham số `$total` và `$places`:

```php
$table->double('amount');
```

Kiểu cột `float` hiện tại sẽ tạo ra một cột tương đương `FLOAT` mà không có tổng các chữ số và số thập phân (các chữ số sau dấu phẩy), nhưng có một tham số `$precision` tùy chọn để xác định kích thước lưu độ chính xác là 4-byte hay chính xác là 8-byte. Do đó, bạn có thể xóa các tham số `$total` và `$places` và chỉ định tham số `$precision` tùy chọn theo giá trị mong muốn của bạn và theo tài liệu cơ sở dữ liệu của bạn:

```php
$table->float('amount', precision: 53);
```

Các phương thức `unsignedDecimal`, `unsignedDouble`, và `unsignedFloat` đã bị xóa, vì modifier `unsigned` cho các kiểu cột này đã bị MySQL ngừng hỗ trợ và chưa bao giờ được tiêu chuẩn hóa trên các hệ quản trị cơ sở dữ liệu khác. Tuy nhiên, nếu bạn vẫn muốn tiếp tục sử dụng thuộc tính `unsigned` cho các kiểu cột này, bạn có thể gọi nối phương thức `unsigned` vào định nghĩa của cột:

```php
$table->decimal('amount', total: 8, places: 2)->unsigned();
$table->double('amount')->unsigned();
$table->float('amount', precision: 53)->unsigned();
```

<a name="dedicated-mariadb-driver"></a>
#### Dedicated MariaDB Driver

**Likelihood Of Impact: Very Low**

Thay vì luôn sử dụng driver MySQL khi kết nối với cơ sở dữ liệu MariaDB, Laravel 11 đã thêm một driver cơ sở dữ liệu dành riêng cho MariaDB.

Nếu ứng dụng của bạn kết nối đến một cơ sở dữ liệu MariaDB, bạn có thể cập nhật cấu hình kết nối sang driver `mariadb` để tận dụng các tính năng dành riêng cho MariaDB trong tương lai:

    'driver' => 'mariadb',
    'url' => env('DB_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    // ...

Hiện tại, driver MariaDB hoạt động giống như driver MySQL với một ngoại lệ: phương thức schema builder `uuid` tạo ra các cột UUID native thay vì các cột `char(36)`.

Nếu các migration hiện tại của bạn đang sử dụng phương thức schema builder `uuid` và bạn chọn sử dụng driver cơ sở dữ liệu `mariadb`, thì bạn nên cập nhật các file migration của bạn gọi phương thức `uuid` thành `char` để tránh các thay đổi gây ra lỗi hoặc hành vi không mong muốn:

```php
Schema::table('users', function (Blueprint $table) {
    $table->char('uuid', 36);

    // ...
});
```

<a name="spatial-types"></a>
#### Spatial Types

**Likelihood Of Impact: Low**

Các kiểu cột không gian trong database migration cũng đã được viết lại để nhất quán trên tất cả các cơ sở dữ liệu. Do đó, bạn có thể xóa các phương thức `point`, `lineString`, `polygon`, `geometryCollection`, `multiPoint`, `multiLineString`, `multiPolygon`, và `multiPolygonZ` ra khỏi migration của bạn và sử dụng các phương thức `geometry` hoặc `geography` thay thế:

```php
$table->geometry('shapes');
$table->geography('coordinates');
```

Để giới hạn kiểu dữ liệu hoặc identifier hệ thống tham chiếu không gian cho các giá trị được lưu trong các cột trên MySQL, MariaDB và PostgreSQL, bạn có thể truyền `subtype` và `srid` vào phương thức:

```php
$table->geometry('dimension', subtype: 'polygon', srid: 0);
$table->geography('latitude', subtype: 'point', srid: 4326);
```

Các modifier cột `isGeometry` và `projection` của PostgreSQL grammar đã được loại bỏ tương ứng.

<a name="doctrine-dbal-removal"></a>
#### Doctrine DBAL Removal

**Likelihood Of Impact: Low**

Danh sách các class và phương thức liên quan đến Doctrine DBAL sau đây đã bị xóa. Laravel không còn phụ thuộc vào package này nữa và việc đăng ký các kiểu Doctrine tùy biến không còn cần thiết cho việc tạo và thay đổi các kiểu cột khác nhau mà trước đây yêu cầu các kiểu tùy biến:

<div class="content-list" markdown="1">

- `Illuminate\Database\Schema\Builder::$alwaysUsesNativeSchemaOperationsIfPossible` class property
- `Illuminate\Database\Schema\Builder::useNativeSchemaOperationsIfPossible()` method
- `Illuminate\Database\Connection::usingNativeSchemaOperations()` method
- `Illuminate\Database\Connection::isDoctrineAvailable()` method
- `Illuminate\Database\Connection::getDoctrineConnection()` method
- `Illuminate\Database\Connection::getDoctrineSchemaManager()` method
- `Illuminate\Database\Connection::getDoctrineColumn()` method
- `Illuminate\Database\Connection::registerDoctrineType()` method
- `Illuminate\Database\DatabaseManager::registerDoctrineType()` method
- `Illuminate\Database\PDO` directory
- `Illuminate\Database\DBAL\TimestampType` class
- `Illuminate\Database\Schema\Grammars\ChangeColumn` class
- `Illuminate\Database\Schema\Grammars\RenameColumn` class
- `Illuminate\Database\Schema\Grammars\Grammar::getDoctrineTableDiff()` method

</div>

Ngoài ra, việc đăng ký các kiểu Doctrine tùy biến thông qua `dbal.types` trong file cấu hình `database` của ứng dụng cũng không còn được yêu cầu.

Nếu trước đây bạn đã sử dụng Doctrine DBAL để kiểm tra cơ sở dữ liệu và các bảng liên quan, thì giờ đây bạn có thể sử dụng các phương thức schema native mới của Laravel (`Schema::getTables()`, `Schema::getColumns()`, `Schema::getIndexes()`, `Schema::getForeignKeys()`, vv...) để thay thế.

<a name="deprecated-schema-methods"></a>
#### Phương thức schema cũ

**Likelihood Of Impact: Very Low**

Các phương thức dựa trên Doctrine đã ngừng hỗ trợ như `Schema::getAllTables()`, `Schema::getAllViews()`, và `Schema::getAllTypes()` đã bị xóa để chuyển sang sử dụng các phương thức native mới của Laravel là `Schema::getTables()`, `Schema::getViews()`, và `Schema::getTypes()`.

Khi sử dụng PostgreSQL và SQL Server, không có phương thức schema mới nào chấp nhận reference gồm ba phần (ví dụ: `database.schema.table`). Do đó, bạn nên sử dụng phương thức `connection()` để khai báo cơ sở dữ liệu thay thế:

```php
Schema::connection('database')->hasTable('schema.table');
```

<a name="get-column-types"></a>
#### Schema Builder `getColumnType()` Method

**Likelihood Of Impact: Very Low**

Phương thức `Schema::getColumnType()` bây giờ sẽ luôn trả về kiểu dữ liệu thực tế của cột đã cho, thay vì kiểu tương ứng trong Doctrine DBAL.

<a name="database-connection-interface"></a>
#### Database Connection Interface

**Likelihood Of Impact: Very Low**

Interface `Illuminate\Database\ConnectionInterface` đã được bổ sung một phương thức `scalar` mới. Nếu bạn đang định nghĩa implementation cho interface này, bạn nên thêm phương thức `scalar` vào implementation của bạn:

```php
public function scalar($query, $bindings = [], $useReadPdo = true);
```

<a name="dates"></a>
### Dates

<a name="carbon-3"></a>
#### Carbon 3

**Likelihood Of Impact: Medium**

Laravel 11 hỗ trợ cả Carbon 2 và Carbon 3. Carbon là một thư viện thao tác ngày tháng năm được sử dụng rộng rãi bởi Laravel và các package trong hệ sinh thái. Nếu bạn nâng cấp lên Carbon 3, hãy lưu ý rằng các phương thức `diffIn*` bây giờ sẽ trả về một số float và có thể trả về giá trị âm để chỉ hướng thời gian, đây là một thay đổi đáng kể so với Carbon 2. Hãy xem [change log](https://github.com/briannesbitt/Carbon/releases/tag/3.0.0) và [tài liệu](https://carbon.nesbot.com/guide/getting-started/migration.html) của Carbon để biết thông tin chi tiết về cách xử lý những thay đổi này và các thay đổi khác.

<a name="mail"></a>
### Mail

<a name="the-mailer-contract"></a>
#### The `Mailer` Contract

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Mail\Mailer` đã được bổ sung thêm một phương thức `sendNow` mới. Nếu ứng dụng hoặc package của bạn đang thực hiện implement contract này, thì bạn nên thêm phương thức `sendNow` này vào implementation của bạn:

```php
public function sendNow($mailable, array $data = [], $callback = null);
```

<a name="packages"></a>
### Packages

<a name="publishing-service-providers"></a>
#### Publishing Service Providers to the Application

**Likelihood Of Impact: Very Low**

Nếu bạn đang viết một package Laravel thực hiện việc export một service provider vào thư mục `app/Providers` của ứng dụng và sửa file cấu hình `config/app.php` của ứng dụng để đăng ký service provider, thì bạn nên cập nhật package của mình để sử dụng phương thức `ServiceProvider::addProviderToBootstrapFile` mới.

Phương thức `addProviderToBootstrapFile` sẽ tự động thêm service provider mà bạn đã export vào file `bootstrap/providers.php` của ứng dụng, vì mảng `providers` sẽ không còn tồn tại trong file cấu hình `config/app.php` trong các ứng dụng Laravel 11 mới nữa.

```php
use Illuminate\Support\ServiceProvider;

ServiceProvider::addProviderToBootstrapFile(Provider::class);
```

<a name="queues"></a>
### Queues

<a name="the-batch-repository-interface"></a>
#### The `BatchRepository` Interface

**Likelihood Of Impact: Very Low**

Interface `Illuminate\Bus\BatchRepository` đã được bổ sung thêm một phương thức `rollBack` mới. Nếu bạn đang thực hiện implement interface này trong package hoặc ứng dụng của bạn, bạn nên thêm phương thức này vào implementation của bạn:

```php
public function rollBack();
```

<a name="synchronous-jobs-in-database-transactions"></a>
#### Synchronous Jobs in Database Transactions

**Likelihood Of Impact: Very Low**

Trước đây, các synchronous job (các job sử dụng queue driver `sync`) sẽ được chạy ngay lập tức, bất kể tùy chọn cấu hình `after_commit` của kết nối queue được set thành `true` hay phương thức `afterCommit` được gọi trên job đó.

Trong Laravel 11, các synchronous queue job này giờ đây sẽ tuân thủ cấu hình "after commit" của kết nối queue hoặc của job đó.

<a name="rate-limiting"></a>
### Rate Limiting

<a name="per-second-rate-limiting"></a>
#### Per-Second Rate Limiting

**Likelihood Of Impact: Medium**

Laravel 11 hiện nay đã hỗ trợ giới hạn tần suất mới theo từng giây thay vì bị giới hạn ở mức độ từng phút. Có một số thay đổi có thể gây ảnh hưởng mà bạn nên lưu ý liên quan đến sự thay đổi này.

Constructor của class `GlobalLimit` bây giờ sẽ chấp nhận số giây thay vì số phút. Class này không được ghi trong document và thường sẽ không được dùng:

```php
new GlobalLimit($attempts, 2 * 60);
```

Constructor của class `Limit` bây giờ sẽ chấp nhận số giây thay vì số phút. Tất cả các cách sử dụng đã được ghi trong document của class này đều được giới hạn ở các static constructor như `Limit::perMinute` và `Limit::perSecond`. Tuy nhiên, nếu bạn đang tự khởi tạo class này, thì bạn nên cập nhật ứng dụng của bạn để cung cấp số giây cho constructor của class:

```php
new Limit($key, $attempts, 2 * 60);
```

Thuộc tính `decayMinutes` của class `Limit` đã được đổi tên thành `decaySeconds` và hiện tại chứa số giây thay vì số phút.

Constructor của các class `Illuminate\Queue\Middleware\ThrottlesExceptions` và `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis` cũng sẽ chấp nhận số giây thay vì số phút:

```php
new ThrottlesExceptions($attempts, 2 * 60);
new ThrottlesExceptionsWithRedis($attempts, 2 * 60);
```

<a name="cashier-stripe"></a>
### Cashier Stripe

<a name="updating-cashier-stripe"></a>
#### Updating Cashier Stripe

**Likelihood Of Impact: High**

Laravel 11 không còn hỗ trợ Cashier Stripe 14.x. Do đó, bạn nên cập nhật library Cashier Stripe của ứng dụng lên `^15.0` trong file `composer.json` của bạn.

Cashier Stripe 15.0 không còn tự động load các migration từ thư mục migration của chính nó nữa. Thay vào đó, bạn nên chạy lệnh sau để export các migration của Cashier Stripe vào ứng dụng của bạn:

```shell
php artisan vendor:publish --tag=cashier-migrations
```

Vui lòng xem [hướng dẫn nâng cấp Cashier Stripe](https://github.com/laravel/cashier-stripe/blob/15.x/UPGRADE.md) để biết thêm các thay đổi có thể gây ra lỗi.

<a name="spark-stripe"></a>
### Spark (Stripe)

<a name="updating-spark-stripe"></a>
#### Updating Spark Stripe

**Likelihood Of Impact: High**

Laravel 11 không còn hỗ trợ Laravel Spark Stripe 4.x. Do đó, bạn nên cập nhật library Laravel Spark Stripe của ứng dụng lên `^5.0` trong file `composer.json` của bạn.

Spark Stripe 5.0 cũng không còn tự động load các migration từ thư mục migration của chính nó nữa. Thay vào đó, bạn nên chạy lệnh sau để export các migration của Spark Stripe vào ứng dụng của bạn:

```shell
php artisan vendor:publish --tag=spark-migrations
```

Vui lòng xem [hướng dẫn nâng cấp Spark Stripe](https://spark.laravel.com/docs/spark-stripe/upgrade.html) để biết thêm các thay đổi có thể gây ra lỗi.

<a name="passport"></a>
### Passport

<a name="updating-telescope"></a>
#### Updating Passport

**Likelihood Of Impact: High**

Laravel 11 không còn hỗ trợ Laravel Passport 11.x. Do đó, bạn nên cập nhật library Laravel Passport của ứng dụng lên `^12.0` trong file `composer.json` của bạn.

Passport 12.0 cũng không còn tự động load các migration từ thư mục migration của chính nó nữa. Thay vào đó, bạn nên chạy lệnh sau để export các migration của Passport vào ứng dụng của bạn:

```shell
php artisan vendor:publish --tag=passport-migrations
```

Ngoài ra, mặc định, password grant type sẽ bị disable. Bạn có thể enable nó bằng cách gọi phương thức `enablePasswordGrant` trong phương thức `boot` của `AppServiceProvider` trong ứng dụng của bạn:

    public function boot(): void
    {
        Passport::enablePasswordGrant();
    }

<a name="sanctum"></a>
### Sanctum

<a name="updating-sanctum"></a>
#### Updating Sanctum

**Likelihood Of Impact: High**

Laravel 11 không còn hỗ trợ Laravel Sanctum 3.x. Do đó, bạn nên cập nhật library Laravel Sanctum của ứng dụng lên `^4.0` trong file `composer.json` của bạn.

Sanctum 4.0 cũng không còn tự động load các migration từ thư mục migration của chính nó nữa. Thay vào đó, bạn nên chạy lệnh sau để export các migration của Sanctum vào ứng dụng của bạn:

```shell
php artisan vendor:publish --tag=sanctum-migrations
```

Trong file `config/sanctum.php` của ứng dụng của bạn, bạn nên cập nhật các tham chiếu đến middleware `authenticate_session`, `encrypt_cookies`, và `validate_csrf_token` thành như sau:

    'middleware' => [
        'authenticate_session' => Laravel\Sanctum\Http\Middleware\AuthenticateSession::class,
        'encrypt_cookies' => Illuminate\Cookie\Middleware\EncryptCookies::class,
        'validate_csrf_token' => Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
    ],

<a name="telescope"></a>
### Telescope

<a name="updating-telescope"></a>
#### Updating Telescope

**Likelihood Of Impact: High**

Laravel 11 không còn hỗ trợ Laravel Telescope 4.x. Do đó, bạn nên cập nhật library Laravel Telescope của ứng dụng lên `^5.0` trong file `composer.json` của bạn.

Telescope 5.0 không còn tự động load các migration từ thư mục migration của chính nó nữa. Thay vào đó, bạn nên chạy lệnh sau để export các migration của Telescope vào ứng dụng của bạn:

```shell
php artisan vendor:publish --tag=telescope-migrations
```

<a name="spatie-once-package"></a>
### Spatie Once Package

**Likelihood Of Impact: Medium**

Laravel 11 sẽ cung cấp hàm [`once`](/docs/{{version}}/helpers#method-once) riêng để đảm bảo rằng một closure nhất định chỉ được chạy một lần duy nhất. Do đó, nếu ứng dụng của bạn có phụ thuộc vào package `spatie/once`, thì bạn nên xóa nó ra khỏi file `composer.json` của ứng dụng để tránh xung đột.

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/10.x...11.x) và chọn bản cập nhật nào quan trọng với bạn.
