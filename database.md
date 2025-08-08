# Database: Getting Started

- [Giới thiệu](#introduction)
    - [Cấu hình](#configuration)
    - [Đọc và viết thông qua Connection](#read-and-write-connections)
- [Chạy SQL Query](#running-queries)
    - [Dùng Multiple Database Connection](#using-multiple-database-connections)
    - [Listen cho Query Event](#listening-for-query-events)
    - [Giám sát thời gian truy vấn](#monitoring-cumulative-query-time)
- [Database Transaction](#database-transactions)
- [Kết nối đến database cli](#connecting-to-the-database-cli)
- [Kiểm tra cơ sở dữ liệu của bạn](#inspecting-your-databases)
- [Giám sát cơ sở dữ liệu của bạn](#monitoring-your-databases)

<a name="introduction"></a>
## Giới thiệu

Hầu hết các ứng dụng web hiện đại đều tương tác với cơ sở dữ liệu. Laravel làm cho việc tương tác với cơ sở dữ liệu được hỗ trợ trở nên cực kỳ đơn giản bằng cách sử dụng raw SQL, [fluent query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent). Hiện tại, Laravel cung cấp hỗ trợ cho năm cơ sở dữ liệu:

<div class="content-list" markdown="1">

- MariaDB 10.10+ ([Version Policy](https://mariadb.org/about/#maintenance-policy))
- MySQL 5.7+ ([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 11.0+ ([Version Policy](https://www.postgresql.org/support/versioning/))
- SQLite 3.8.8+
- SQL Server 2017+ ([Version Policy](https://docs.microsoft.com/en-us/lifecycle/products/?products=sql-server))

</div>

<a name="configuration"></a>
### Cấu hình

Cấu hình cho các service cơ sở dữ liệu của Laravel nằm trong file cấu hình `config/database.php` của ứng dụng của bạn. Trong file này, bạn có thể định nghĩa tất cả các kết nối cơ sở dữ liệu của bạn, cũng như chỉ định các kết nối nào sẽ được sử dụng mặc định. Hầu hết các tùy chọn cấu hình trong file này đều được điều khiển bởi các giá trị của các biến môi trường trong ứng dụng của bạn. Các ví dụ cho hầu hết các hệ thống cơ sở dữ liệu được hỗ trợ của Laravel đã được cung cấp trong file này.

Mặc định, [cấu hình môi trường](/docs/{{version}}/configuration#environment-configuration) mẫu của Laravel đã sẵn sàng để sử dụng với [Laravel Sail](/docs/{{version}}/sail), đây là một cấu hình Docker để phát triển các ứng dụng Laravel trên máy local của bạn. Tuy nhiên, bạn có thể tự do sửa đổi cấu hình cơ sở dữ liệu nếu cần cho cơ sở dữ liệu local riêng của bạn.

<a name="sqlite-configuration"></a>
#### SQLite Configuration

Cơ sở dữ liệu SQLite được chứa trong một file duy nhất trên filesystem của bạn. Bạn có thể tạo cơ sở dữ liệu SQLite mới bằng cách sử dụng lệnh `touch` trong terminal của bạn: `touch database/database.sqlite`. Sau khi cơ sở dữ liệu đã được tạo, bạn có thể dễ dàng cấu hình các biến môi trường của bạn để trỏ đến cơ sở dữ liệu này bằng cách set đường dẫn tuyệt đối tới file cơ sở dữ liệu trong biến môi trường `DB_DATABASE`:

```ini
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

Để enable các ràng buộc khóa ngoại cho các kết nối SQLite, bạn nên set biến môi trường `DB_FOREIGN_KEYS` thành `true`:

```ini
DB_FOREIGN_KEYS=true
```

<a name="mssql-configuration"></a>
#### Microsoft SQL Server Configuration

Để sử dụng cơ sở dữ liệu Microsoft SQL Server, bạn phải đảm bảo rằng bạn đã cài đặt các extension PHP `sqlsrv` và `pdo_sqlsrv` cũng như bất kỳ library nào mà chúng có thể yêu cầu, chẳng hạn như Microsoft SQL ODBC driver.

<a name="configuration-using-urls"></a>
#### Configuration Using URLs

Thông thường, các kết nối cơ sở dữ liệu được cấu hình bằng nhiều giá trị cấu hình như `host`, `database`, `username`, `password`, vv. Mỗi giá trị cấu hình này đều có một biến môi trường tương ứng. Điều này có nghĩa là khi cấu hình thông tin kết nối cơ sở dữ liệu của bạn trên production server, bạn sẽ cần quản lý một số lượng nhiều biến môi trường.

Một số nhà cung cấp cơ sở dữ liệu như AWS và Heroku cung cấp một "URL" cơ sở dữ liệu để chứa tất cả thông tin kết nối cho một cơ sở dữ liệu trong một string duy nhất. URL cơ sở dữ liệu mẫu có thể trông giống như sau:

```html
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

Các URL này thường tuân theo quy ước như sau:

```html
driver://username:password@host:port/database?options
```

Để thuận tiện, Laravel cũng hỗ trợ các URL này như là một giải pháp thay thế cho việc cấu hình cơ sở dữ liệu của bạn với nhiều tùy chọn cấu hình. Nếu có tùy chọn cấu hình `url` (hoặc biến môi trường `DATABASE_URL`), nó sẽ được sử dụng để kết nối cơ sở dữ liệu và thông tin xác thực.

<a name="read-and-write-connections"></a>
### Đọc và viết thông qua Connection

Thỉnh thoảng, bạn cũng có thể muốn sử dụng một kết nối riêng cho các câu lệnh SELECT và một kết nối riêng khác cho các câu lệnh INSERT, UPDATE và DELETE. Laravel làm cho điều này trở nên dễ dàng và các kết nối thích hợp sẽ luôn được sử dụng cho dù bạn đang sử dụng bất kỳ loại nào: raw query, query builder hoặc Eloquent ORM.

Để xem cách cấu hình các kết nối đọc và ghi này, hãy xem ví dụ sau:

    'mysql' => [
        'read' => [
            'host' => [
                '192.168.1.1',
                '196.168.1.2',
            ],
        ],
        'write' => [
            'host' => [
                '196.168.1.3',
            ],
        ],
        'sticky' => true,
        'driver' => 'mysql',
        'database' => 'database',
        'username' => 'root',
        'password' => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix' => '',
    ],

Lưu ý rằng có ba key đã được thêm vào trong mảng cấu hình là: `read`, `write` và `stick`. Các key `read` và `write` có thể có một mảng các giá trị chứa key duy nhất là: `host`. Còn lại các tùy chọn cơ sở dữ liệu khác cho các kết nối `read` và `write` sẽ được lấy từ trong mảng cấu hình `mysql`.

 Nếu bạn muốn ghi đè các giá trị trong mảng `mysql`, thì bạn chỉ cần set các item đó vào trong mảng `read` và `write`. Vì vậy, trong trường hợp này, `192.168.1.1` sẽ được sử dụng để làm máy chủ cho kết nối "read", trong khi `192.168.1.3` sẽ được sử dụng cho kết nối "write". Các thông tin cho cơ sở dữ liệu, tiền tố, bộ ký tự và tất cả các tùy chọn còn lại trong mảng `mysql` sẽ được chia sẻ cho cả hai kết nối này. Khi có nhiều giá trị trong mảng cấu hình `host`, một máy chủ cơ sở dữ liệu sẽ được chọn ngẫu nhiên cho mỗi yêu cầu.

<a name="the-sticky-option"></a>
#### The `sticky` Option

Tùy chọn `sticky` là một giá trị *tùy chọn* có thể được sử dụng để cho phép đọc các bản ghi đã được ghi vào trong cơ sở dữ liệu ngay trong request hiện tại. Nếu tùy chọn `stick` được bật và các thao tác "write" đã được thực hiện trong cơ sở dữ liệu ở trong request hiện tại, thì các thao tác "read" tiếp theo sẽ được sử dụng kết nối "write". Điều này giúp đảm bảo rằng mọi dữ liệu được ghi vào trong request hiện tại có thể được đọc lại ngay lập tức từ cơ sở dữ liệu trong cùng một request đó. Tùy thuộc vào loại yêu cầu, mà bạn sẽ quyết định xem đây có phải là một hành động mong muốn cho application của bạn hay không.

<a name="running-queries"></a>
## Chạy SQL Query

Khi bạn đã cấu hình các kết nối cơ sở dữ liệu của bạn, bạn có thể chạy các truy vấn bằng cách sử dụng facade `DB`. Facade `DB` sẽ cung cấp các phương thức cho từng loại truy vấn như: `select`, `update`, `insert`, `delete`, và `statement`.

<a name="running-a-select-query"></a>
#### Running a Select Query

Để chạy một truy vấn SELECT cơ bản, bạn có thể sử dụng phương thức `select` trên facade `DB`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         */
        public function index(): View
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

Tham số đầu tiên được truyền cho phương thức `select` là một truy vấn SQL, còn tham số thứ hai là bất kỳ tham số nào cần thiết cho truy vấn đó thông thường là các giá trị của các mệnh đề `where`. Rằng buộc tham số này sẽ được bảo vệ để chống lại các SQL injection.

Phương thức `select` sẽ luôn trả về một kết quả là một `array`. Mỗi kết quả trong mảng sẽ là một đối tượng `stdClass` của PHP, đại diện cho một record trong cơ sở dữ liệu:

    use Illuminate\Support\Facades\DB;

    $users = DB::select('select * from users');

    foreach ($users as $user) {
        echo $user->name;
    }

<a name="selecting-scalar-values"></a>
#### Selecting Scalar Values

Thỉnh thoảng truy vấn cơ sở dữ liệu của bạn có thể dẫn đến một giá trị duy nhất. Thay vì được yêu cầu lấy ra kết quả của truy vấn từ một record object, Laravel cho phép bạn lấy ra trực tiếp giá trị này bằng phương thức `scalar`:

    $burgers = DB::scalar(
        "select count(case when food = 'burger' then 1 end) as burgers from menu"
    );

<a name="selecting-multiple-result-sets"></a>
#### Selecting Multiple Result Sets

Nếu ứng dụng của bạn gọi các procedure của sql và trả về nhiều kết quả, bạn có thể sử dụng phương thức `selectResultSets` để lấy ra tất cả các kết quả được trả về bởi procedure đó:

    [$options, $notifications] = DB::selectResultSets(
        "CALL get_user_options_and_notifications(?)", $request->user()->id
    );

<a name="using-named-bindings"></a>
#### Using Named Bindings

Thay vì sử dụng `?` để biểu thị cho các tham số của bạn, bạn có thể thực hiện truy vấn bằng các tham số có tên:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

<a name="running-an-insert-statement"></a>
#### Running an Insert Statement

Để thực hiện một câu lệnh `insert`, bạn có thể sử dụng phương thức `insert` trên facade `DB`. Giống như `select`, phương thức này chấp nhận truy vấn SQL làm tham số đầu tiên và các tham số còn lại làm tham số thứ hai:

    use Illuminate\Support\Facades\DB;

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);

<a name="running-an-update-statement"></a>
#### Running an Update Statement

Phương thức `update` sẽ được sử dụng để cập nhật các bản ghi hiện có trong cơ sở dữ liệu. Số lượng các hàng bị cập nhật bởi câu lệnh này sẽ được trả về từ phương thức:

    use Illuminate\Support\Facades\DB;

    $affected = DB::update(
        'update users set votes = 100 where name = ?',
        ['Anita']
    );

<a name="running-a-delete-statement"></a>
#### Running a Delete Statement

Phương thức `delete` sẽ được sử dụng để xóa các bản ghi ra khỏi cơ sở dữ liệu. Giống như `update`, Số lượng các hàng bị xoá sẽ được trả về từ phương thức:

    use Illuminate\Support\Facades\DB;

    $deleted = DB::delete('delete from users');

<a name="running-a-general-statement"></a>
#### Running a General Statement

Có một số lệnh cơ sở dữ liệu không trả về bất kỳ giá trị nào. Đối với các loại lệnh như thế này, bạn có thể sử dụng phương thức `statement` trên facade `DB`:

    DB::statement('drop table users');

<a name="running-an-unprepared-statement"></a>
#### Running an Unprepared Statement

Thỉnh thoảng bạn có thể muốn thực hiện một câu lệnh SQL mà không có liên kết với bất kỳ vào giá trị nào. Bạn có thể sử dụng phương pháp `unprepared` của facade `DB` để thực hiện việc này:

    DB::unprepared('update users set votes = 100 where name = "Dries"');

> [!WARNING]
> Vì các câu lệnh unprepared không liên kết với bất kỳ tham số nên chúng có thể dễ bị tấn công bởi SQL injection. Bạn đừng bao giờ cho phép các giá trị do người dùng kiểm soát thực hiện trong câu lệnh unprepared .

<a name="implicit-commits-in-transactions"></a>
#### Implicit Commits

Khi sử dụng các phương thức `statement` và `unprepared` của facade `DB` trong các transaction, bạn phải cẩn thận để tránh các câu lệnh gây ra các [commit ngầm](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html). Các câu lệnh này sẽ khiến database engine gián tiếp commit toàn bộ transaction, khiến Laravel không biết về mức độ transaction của cơ sở dữ liệu. Một ví dụ về câu lệnh như vậy là tạo một bảng cơ sở dữ liệu:

    DB::unprepared('create table a (col varchar(1) null)');

Vui lòng tham khảo hướng dẫn sử dụng MySQL để biết thêm về [danh sách tất cả các câu lệnh](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html) kích hoạt các commit ngầm.

<a name="using-multiple-database-connections"></a>
### Using Multiple Database Connections

Nếu ứng dụng của bạn định nghĩa nhiều kết nối trong file cấu hình `config/database.php`, thì bạn có thể truy cập từng kết nối thông qua phương thức `connection` do facade `DB` cung cấp. Tên kết nối được truyền cho phương thức `connection` phải tương ứng với một trong các kết nối được liệt kê trong file cấu hình `config/database.php` hoặc được cấu hình trong lúc chạy thực bằng helper `config`:

    use Illuminate\Support\Facades\DB;

    $users = DB::connection('sqlite')->select(/* ... */);

Bạn có thể truy cập vào instance PDO raw, cơ bản của kết nối bằng cách sử dụng phương thức `getPdo` trên instance kết nối:

    $pdo = DB::connection()->getPdo();

<a name="listening-for-query-events"></a>
### Listen cho Query Event

Nếu bạn muốn chỉ định một closure được gọi cho mỗi truy vấn SQL được thực thi bởi ứng dụng của bạn, bạn có thể sử dụng phương thức `listen` của facade `DB`. Phương thức này có thể hữu ích để ghi log truy vấn hoặc để debug. Bạn có thể đăng ký closure listener truy vấn này trong phương thức `boot` của [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Database\Events\QueryExecuted;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            DB::listen(function (QueryExecuted $query) {
                // $query->sql;
                // $query->bindings;
                // $query->time;
            });
        }
    }

<a name="monitoring-cumulative-query-time"></a>
### Giám sát thời gian truy vấn

Lỗi hiệu suất phổ biến của các ứng dụng web hiện đại là lượng thời gian chúng dành để truy vấn cơ sở dữ liệu. Rất may, Laravel có thể gọi một closure hoặc một callback theo lựa chọn của bạn khi nó dành quá nhiều thời gian để truy vấn cơ sở dữ liệu trong một single request. Để bắt đầu, hãy cung cấp ngưỡng thời gian truy vấn (tính bằng mili giây) và closure cho phương thức `whenQueryingForLongerThan`. Bạn có thể gọi phương thức này trong phương thức `boot` của một [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Database\Connection;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Database\Events\QueryExecuted;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
                // Notify development team...
            });
        }
    }

<a name="database-transactions"></a>
## Database Transaction

Bạn có thể sử dụng phương thức `transaction` được cung cấp bởi facade `DB` để chạy một tập hợp các lệnh trong một transaction cơ sở dữ liệu. Nếu một ngoại lệ được đưa ra trong transaction closure này, transaction sẽ được tự động khôi phục lại trạng thái trước khi chạy và ngoại lệ đó sẽ được đưa ra. Nếu closure thực hiện thành công, transaction sẽ được tự động thực hiện. Bạn không cần phải lo lắng về việc bạn phải tự rollback hay commit trong khi sử dụng phương thức `transaction`:

    use Illuminate\Support\Facades\DB;

    DB::transaction(function () {
        DB::update('update users set votes = 1');

        DB::delete('delete from posts');
    });

<a name="handling-deadlocks"></a>
#### Handling Deadlocks

Phương thức `transaction` chấp nhận một tham số thứ hai làm một tùy chọn để định nghĩa số lần transaction sẽ được thử lại khi xảy ra lỗi. Sau khi thử lại hết số lần thử, thì một ngoại lệ sẽ được đưa ra:

    use Illuminate\Support\Facades\DB;

    DB::transaction(function () {
        DB::update('update users set votes = 1');

        DB::delete('delete from posts');
    }, 5);

<a name="manually-using-transactions"></a>
#### Manually Using Transactions

Nếu bạn muốn chạy một transaction theo cách thủ công và có toàn quyền kiểm soát với các rollback và commit, bạn có thể sử dụng phương thức `beginTransaction` được cung cấp bởi facade `DB`:

    use Illuminate\Support\Facades\DB;

    DB::beginTransaction();

Bạn có thể rollback transaction thông qua phương thức `rollBack`:

    DB::rollBack();

Cuối cùng, bạn có thể commit một transaction thông qua phương thức `commit`:

    DB::commit();

> [!NOTE]
> Các phương thức transaction của facade `DB` sẽ kiểm soát các transaction cho cả [query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent).

<a name="connecting-to-the-database-cli"></a>
## Kết nối đến database cli

Nếu bạn muốn kết nối đến CLI của cơ sở dữ liệu của bạn, bạn có thể sử dụng lệnh Artisan `db`:

```shell
php artisan db
```

Nếu cần, bạn có thể chỉ định tên kết nối để kết nối đến cơ sở dữ liệu mà không phải là kết nối mặc định:

```shell
php artisan db mysql
```

<a name="inspecting-your-databases"></a>
## Kiểm tra cơ sở dữ liệu của bạn

Bằng cách sử dụng các lệnh Artisan `db:show` và `db:table`, bạn sẽ có thể có được thông tin chi tiết về cơ sở dữ liệu của bạn và các bảng liên kết. Để xem tổng quan về cơ sở dữ liệu của bạn, bao gồm cả kích thước, loại, số lượng connection đang kết nối và bản tổng quan các table, bạn có thể sử dụng lệnh `db:show`:

```shell
php artisan db:show
```

Bạn có thể chỉ định kết nối cơ sở dữ liệu nào sẽ được kiểm tra bằng cách cung cấp tên kết nối cơ sở dữ liệu cho lệnh thông qua tùy chọn `--database`:

```shell
php artisan db:show --database=pgsql
```

Nếu bạn muốn chứa thêm số lượng record và view của cơ sở dữ liệu trong output của command, bạn có thể cung cấp các tùy chọn `--counts` và `--views` tương ứng. Trên những cơ sở dữ liệu lớn, việc lấy ra số lượng record và view có thể bị chậm:

```shell
php artisan db:show --counts --views
```

<a name="table-overview"></a>
#### Table Overview

Nếu bạn muốn có cái nhìn tổng quan về một bảng riêng lẻ trong cơ sở dữ liệu của bạn, bạn có thể thực thi lệnh Artisan `db:table`. Lệnh này cung cấp cái nhìn tổng quan chung về bảng cơ sở dữ liệu, bao gồm các cột, loại, thuộc tính, khóa và các index của nó:

```shell
php artisan db:table users
```

<a name="monitoring-your-databases"></a>
## Giám sát cơ sở dữ liệu của bạn

Sử dụng lệnh `db:monitor` Artisan, bạn có thể hướng dẫn Laravel gửi một event `Illuminate\Database\Events\DatabaseBusy` nếu cơ sở dữ liệu của bạn đang làm việc nhiều hơn số kết nối được cài đặt.

Để bắt đầu, bạn nên tạo schedule cho lệnh `db:monitor` để [chạy mỗi phút](/docs/{{version}}/scheduling). Lệnh này chấp nhận một tên kết nối cơ sở dữ liệu đã được cấu hình mà bạn muốn theo dõi cũng như số lượng kết nối tối đa cần được chấp nhận trước khi gửi event:

```shell
php artisan db:monitor --databases=mysql,pgsql --max=100
```

Schedule cho lệnh này là không đủ để kích hoạt một thông báo cảnh báo bạn về số lượng kết nối đang được mở bị vượt quá. Khi lệnh gặp cơ sở dữ liệu có số lượng kết nối mở vượt quá ngưỡng của bạn, sự kiện `DatabaseBusy` sẽ được gửi đi. Bạn nên lắng nghe sự kiện này trong `EventServiceProvider` của ứng dụng để gửi thông báo cho bạn hoặc nhóm phát triển của bạn:

```php
use App\Notifications\DatabaseApproachingMaxConnections;
use Illuminate\Database\Events\DatabaseBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Register any other events for your application.
 */
public function boot(): void
{
    Event::listen(function (DatabaseBusy $event) {
        Notification::route('mail', 'dev@example.com')
                ->notify(new DatabaseApproachingMaxConnections(
                    $event->connectionName,
                    $event->connections
                ));
    });
}
```
