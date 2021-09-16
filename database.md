# Database: Getting Started

- [Giới thiệu](#introduction)
    - [Cấu hình](#configuration)
    - [Đọc và viết thông qua Connection](#read-and-write-connections)
    - [Dùng Multiple Database Connection](#using-multiple-database-connections)
- [Chạy Raw SQL Query](#running-queries)
    - [Listen cho Query Event](#listening-for-query-events)
- [Database Transaction](#database-transactions)

<a name="introduction"></a>
## Giới thiệu

Laravel làm cho việc tương tác với cơ sở dữ liệu cực kỳ đơn giản với nhiều loại cơ sở dữ liệu bằng cách sử dụng raw SQL, [fluent query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent). Hiện tại, Laravel hỗ trợ bốn loại cơ sở dữ liệu:

<div class="content-list" markdown="1">
- MySQL
- PostgreSQL
- SQLite
- SQL Server
</div>

<a name="configuration"></a>
### Cấu hình

Cấu hình cơ sở dữ liệu cho application của bạn được đặt trong file `config/database.php`. Trong file này, bạn có thể định nghĩa tất cả các connection đến các cơ sở dữ liệu của bạn, cũng như khai báo connection nào sẽ là mặc định được sử dụng. Ví dụ mẫu cho các cơ sở dữ liệu này cũng được cung cấp sẵn trong file này.

Mặc định, Laravel đã cài đặt sẵn một [cấu hình môi trường](/docs/{{version}}/configuration#environment-configuration) mẫu cho việc sử dụng với [Laravel Homestead](/docs/{{version}}/homestead), đây là một máy ảo thuận tiện để thực hiện phát triển Laravel trên local của bạn. Tất nhiên, bạn có thể tự do sửa lại cấu hình này cho phù hợp với cơ sở dữ liệu trên máy local của bạn.

#### SQLite Configuration

Sau khi tạo cơ sở dữ liệu SQLite mới bằng cách sử dụng câu lệnh như `touch database/database.sqlite`, bạn có thể dễ dàng cấu hình các biến môi trường của bạn để trỏ đến cơ sở dữ liệu mới này bằng cách sử dụng một đường dẫn tuyệt đối của cơ sở dữ liệu:

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

<a name="read-and-write-connections"></a>
### Đọc và viết thông qua Connection

Thỉnh thoảng, bạn có thể muốn sử dụng một kết nối cơ sở dữ liệu cho các câu lệnh SELECT và một kết nối khác cho các câu lệnh INSERT, UPDATE và DELETE. Laravel làm cho điều này trở nên dễ dàng và các kết nối thích hợp sẽ luôn được sử dụng cho dù bạn đang sử dụng bất kỳ loại nào: raw query, query builder hoặc Eloquent ORM.

Để xem cách cấu hình các kết nối đọc và ghi, hãy xem ví dụ sau:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

Lưu ý rằng có ba key đã được thêm vào trong mảng cấu hình là: `read`, `write` và `stick`. Các key `read` và `write` có thể có một mảng các giá trị chứa một key duy nhất là: `host`. Còn lại các tùy chọn cơ sở dữ liệu cho các kết nối `read` và `write` sẽ được lấy từ mảng `mysql` chính.

 Nếu bạn muốn ghi đè các giá trị trong mảng chính, thì bạn chỉ cần set các item đó vào trong mảng `read` và `write`. Vì thế, trong trường hợp này, `192.168.1.1` sẽ được sử dụng làm máy chủ cho kết nối "read", trong khi `192.168.1.2` sẽ được sử dụng cho kết nối "write". Thông tin cho cơ sở dữ liệu, tiền tố, bộ ký tự và tất cả các tùy chọn khác trong mảng `mysql` chính sẽ được chia sẻ cho cả hai kết nối.

#### The `sticky` Option

Tùy chọn `sticky` là một giá trị *tùy chọn* có thể được sử dụng để cho phép đọc ngay các bản ghi đã được ghi vào cơ sở dữ liệu ngay trong request hiện tại. Nếu tùy chọn `stick` được enable và các thao tác "write" được thực hiện trong cơ sở dữ liệu trong request hiện tại, thì mọi thao tác "read" tiếp theo sẽ sử dụng kết nối "write". Điều này đảm bảo rằng mọi dữ liệu được ghi trong request hiện tại có thể được đọc lại ngay lập tức từ cơ sở dữ liệu trong cùng request đó. Tùy thuộc vào bạn, để quyết định xem đây có phải là một hành động mong muốn cho application của bạn hay không.

<a name="using-multiple-database-connections"></a>
### Dùng Multiple Database Connection

Khi sử dụng nhiều kết nối, bạn có thể truy cập từng kết nối thông qua phương thức `connection` trên facade `DB`. `name` mà sẽ được truyền vào cho phương thức `connection` phải tương ứng với một trong các kết nối đã được tạo trong file cấu hình `config/database.php` của bạn:

    $users = DB::connection('foo')->select(...);

Bạn cũng có thể truy cập vào instance PDO raw bằng cách sử dụng phương thức `getPdo` trong một instance connection:

    $pdo = DB::connection()->getPdo();

<a name="running-queries"></a>
## Chạy Raw SQL Query

Khi bạn đã cấu hình các kết nối cơ sở dữ liệu của bạn, bạn có thể chạy các truy vấn bằng cách sử dụng facade `DB`. Facade `DB` sẽ cung cấp các phương thức cho từng loại truy vấn như: `select`, `update`, `insert`, `delete`, và `statement`.

#### Running A Select Query

Để chạy một truy vấn cơ bản, bạn có thể sử dụng phương thức `select` trên facade `DB`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

Tham số đầu tiên được truyền cho phương thức `select` là một truy vấn SQL raw, trong khi tham số thứ hai là bất kỳ tham số nào cần thiết cho truy vấn. Thông thường, đây là các giá trị của các mệnh đề `where`. Rằng buộc tham số này sẽ được bảo vệ để chống lại các SQL injection.

Phương thức `select` sẽ luôn trả về một `array` kết quả. Mỗi kết quả trong mảng sẽ là một đối tượng `StdClass` của PHP, cho phép bạn truy cập các giá trị của kết quả đó:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Using Named Bindings

Thay vì sử dụng `?` để biểu thị cho các tham số của bạn, bạn có thể thực hiện truy vấn bằng các tham số có tên:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### Running An Insert Statement

Để thực hiện một câu lệnh `insert`, bạn có thể sử dụng phương thức `insert` trên facade `DB`. Giống như `select`, phương thức này lấy truy vấn SQL raw làm tham số đầu tiên và các tham số làm tham số thứ hai:

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### Running An Update Statement

Phương thức `update` nên được sử dụng để cập nhật các bản ghi hiện có trong cơ sở dữ liệu. Số lượng các hàng bị cập nhật bởi câu lệnh sẽ được trả về:

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### Running A Delete Statement

Phương thức `delete` sẽ được sử dụng để xóa các bản ghi ra khỏi cơ sở dữ liệu. Giống như `update`, Số lượng các hàng bị xoá sẽ được trả về:

    $deleted = DB::delete('delete from users');

#### Running A General Statement

Có một số lệnh cơ sở dữ liệu không trả về bất kỳ giá trị nào. Đối với các loại lệnh này, bạn có thể sử dụng phương thức `statement` trên facade `DB`:

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### Listen cho Query Event

Nếu bạn muốn nhận về từng câu lệnh truy vấn SQL được thực hiện bởi application, bạn có thể sử dụng phương thức `listen`. Phương thức này rất hữu ích để ghi log các truy vấn hoặc để debug. Bạn có thể đăng ký listener query của bạn trong [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            DB::listen(function ($query) {
                // $query->sql
                // $query->bindings
                // $query->time
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="database-transactions"></a>
## Database Transaction

Bạn có thể sử dụng phương thức `transaction` trên facade `DB` để chạy một tập hợp các lệnh trong một transaction cơ sở dữ liệu. Nếu một ngoại lệ được đưa ra trong transaction `Closure`, transaction sẽ được tự động khôi phục lại. Nếu `Closure` thực hiện thành công, transaction sẽ được tự động thực hiện. Bạn không cần phải lo lắng về việc rollback hoặc commit thủ công trong khi sử dụng phương thức `transaction`:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### Handling Deadlocks

Phương thức `transaction` chấp nhận một tham số thứ hai là một tùy chọn để định nghĩa số lần transaction sẽ được thử lại khi xảy ra lỗi. Sau khi thử lại hết số lần thử, thì một ngoại lệ sẽ được đưa ra:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    }, 5);

#### Manually Using Transactions

Nếu bạn muốn bắt đầu một transaction theo cách thủ công và có toàn quyền kiểm soát các rollback và commit, bạn có thể sử dụng phương thức `beginTransaction` trên facade `DB`:

    DB::beginTransaction();

Bạn có thể rollback transaction thông qua phương thức `rollBack`:

    DB::rollBack();

Cuối cùng, bạn có thể commit một transaction thông qua phương thức `commit`:

    DB::commit();

> {tip} Các phương thức transaction của facade `DB` sẽ kiểm soát các transaction cho cả [query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent).
