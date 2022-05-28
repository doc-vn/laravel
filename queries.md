# Database: Query Builder

- [Giới thiệu](#introduction)
- [Lấy ra kết quả](#retrieving-results)
    - [Phân đoạn kết quả](#chunking-results)
    - [Thống kê](#aggregates)
- [Select](#selects)
- [Biểu thức raw](#raw-expressions)
- [Join](#joins)
- [Union](#unions)
- [Lệnh where](#where-clauses)
    - [Group tham số](#parameter-grouping)
    - [Lệnh where exist](#where-exists-clauses)
    - [Lệnh where cho truy vấn con](#subquery-where-clauses)
    - [Lệnh where cho JSON](#json-where-clauses)
- [Ordering, Grouping, Limit và Offset](#ordering-grouping-limit-and-offset)
- [Điều kiện cho lệnh](#conditional-clauses)
- [Insert](#inserts)
- [Update](#updates)
    - [Update JSON Column](#updating-json-columns)
    - [Increment và Decrement](#increment-and-decrement)
- [Delete](#deletes)
- [Pessimistic Locking](#pessimistic-locking)
- [Debugging](#debugging)

<a name="introduction"></a>
## Giới thiệu

Database query builder của Laravel cung cấp một interface thuận tiện, dễ dàng để tạo và chạy các query vào cơ sở dữ liệu. Nó có thể được sử dụng để thực hiện hầu hết các hành động vào cơ sở dữ liệu trong application của bạn và có thể làm việc trên tất cả các hệ thống cơ sở dữ liệu được hỗ trợ.

Query builder của Laravel sử dụng tham số PDO để bảo vệ application của bạn khỏi các cuộc tấn công SQL injection. Bạn sẽ không cần phải chuẩn hoá các chuỗi trước khi truyền vào query.

> {note} PDO không hỗ trợ truyền tên cột dưới dạng biến. Do đó, bạn không nên cho phép người dùng nhập tên cột mà truy vấn của bạn tham chiếu, bao gồm cả cột "order by", vv. Nếu bạn phải cho phép người dùng chọn một số cột nhất định để truy vấn, hãy luôn validate tên cột dựa trên một danh sách trắng gồm tên các cột được phép truy vấn.

<a name="retrieving-results"></a>
## Lấy ra kết quả

#### Retrieving All Rows From A Table

Bạn có thể sử dụng phương thức `table` trên facade `DB` để tạo một query. Phương thức `table` trả về một instance của query builder cho bảng đó, cho phép bạn kết hợp nhiều điều kiện vào trong một query và cuối cùng để nhận lại kết quả, bằng phương thức `get`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

Phương thức `get` trả về một `Illuminate\Support\Collection` chứa các kết quả trong đó, mỗi kết quả là một instance của đối tượng `stdClass` của PHP. Bạn có thể truy cập vào giá trị của từng cột bằng cách khai báo tên cột như là một tên một thuộc tính của đối tượng:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Retrieving A Single Row / Column From A Table

Nếu bạn chỉ cần lấy ra một hàng từ một bảng cơ sở dữ liệu, bạn có thể sử dụng phương thức `first`. Phương thức này sẽ trả về một đối tượng `stdClass`:

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

Nếu bạn không cần lấy ra toàn bộ giá trị của một hàng, bạn có thể lấy ra một giá trị của một bản ghi bằng cách sủ dụng phương thức `value`. Phương thức này sẽ trả về giá trị của cột mà bạn đã khai báo:

    $email = DB::table('users')->where('name', 'John')->value('email');

Để lấy một row theo giá trị cột `id` của nó, hãy sử dụng phương thức `find`:

    $user = DB::table('users')->find(3);

#### Retrieving A List Of Column Values

Nếu bạn muốn lấy ra một Collection chứa tất cả các giá trị của một cột, bạn có thể sử dụng phương thức `pluck`. Trong ví dụ này, chúng ta sẽ lấy ra một Collection tiêu đề của các role:

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

Bạn cũng có thể khai báo thêm key cho Collection được trả về là giá trị của một cột khác:

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### Phân đoạn kết quả

Nếu bạn cần làm việc với hàng ngàn bản ghi trong cơ sở dữ liệu, hãy xem xét sử dụng phương thức `chunk`. Phương thức này lấy ra một đoạn nhỏ kết quả tại một thời điểm và đưa từng đoạn đó vào một `Closure` để xử lý. Phương thức này rất hữu ích để viết [Lệnh Artisan](/docs/{{version}}/artisan) xử lý hàng ngàn bản ghi. Ví dụ: hãy làm thử với toàn bộ bảng `users` với số lượng khoảng 100 bản ghi cùng một lúc:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

Bạn có thể dừng xử lý các đoạn tiếp theo bằng cách trả về giá trị `false` từ `Closure`:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...

        return false;
    });

Nếu bạn đang cập nhật bản ghi cơ sở dữ liệu trong khi chunking kết quả, thì kết quả đang được chunking của bạn có thể bị thay đổi theo những cách mà bạn không mong muốn. Vì vậy, khi cập nhật các bản ghi cơ sở dữ liệu trong khi đang chunking, thì tốt nhất bạn nên sử dụng phương thức `chunkById`. Phương thức này sẽ tự động chunking các kết quả dựa theo khóa chính của bản ghi:

    DB::table('users')->where('active', false)
        ->chunkById(100, function ($users) {
            foreach ($users as $user) {
                DB::table('users')
                    ->where('id', $user->id)
                    ->update(['active' => true]);
            }
        });

> {note} Khi cập nhật hoặc xóa các bản ghi bên trong lệnh callback của phương thức chunk, bất kỳ thay đổi nào đối với các khóa chính hoặc khóa ngoại đều có thể ảnh hưởng đến kết quả truy vấn của phương thức chunk. Điều này có thể dẫn đến việc một số bản ghi sẽ không được đưa vào bên trong kết quả chunk.

<a name="aggregates"></a>
### Thống kê

Query builder cũng cung cấp nhiều phương thức thống kê khác nhau như `count`, `max`, `min`, `avg`, và `sum`. Bạn có thể gọi bất kỳ phương thức nào sau khi khởi tạo query của bạn:

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

Bạn có thể kết hợp các phương thức này với các câu lệnh khác:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

#### Determining If Records Exist

Thay vì sử dụng phương thức `count` để xác định xem có tồn tại bản ghi nào phù hợp với các ràng buộc ở trong truy vấn hay không, thì bạn có thể sử dụng phương thức `exists` và `doesntExist`:

    return DB::table('orders')->where('finalized', 1)->exists();

    return DB::table('orders')->where('finalized', 1)->doesntExist();

<a name="selects"></a>
## Select

#### Specifying A Select Clause

Không phải lúc nào bạn cũng muốn select tất cả các cột từ bảng cơ sở dữ liệu. Sử dụng phương thức `select`, bạn có thể khai báo một lệnh `select` tùy chỉnh cho query:

    $users = DB::table('users')->select('name', 'email as user_email')->get();

Phương thức `distinct` cho phép bạn bắt query sẽ phải trả về các kết quả khác nhau:

    $users = DB::table('users')->distinct()->get();

Nếu bạn đã có một instance query builder và bạn muốn thêm một cột vào lệnh select hiện tại của nó, bạn có thể sử dụng phương thức `addSelect`:

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## Biểu thức raw

Thỉnh thoảng bạn có thể cần sử dụng một biểu thức raw trong một query. Để tạo một biểu thức raw như vậy, bạn có thể sử dụng phương thức `DB::raw`:

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

> {note} Các câu lệnh raw sẽ được đưa vào query dưới dạng là một chuỗi, vì vậy bạn cần cực kỳ cẩn thận để không tạo ra lỗ hổng SQL injection.

<a name="raw-methods"></a>
### Raw Methods

Thay vì sử dụng `DB::raw`, bạn cũng có thể sử dụng các phương thức sau để thêm các biểu thức raw vào các phần khác nhau của query của bạn.

#### `selectRaw`

Phương thức `selectRaw` có thể được sử dụng thay cho câu lệnh `addSelect(DB::raw(...))`. Phương thức này chấp nhận một mảng các tùy chọn tham số được truyền vào làm tham số thứ hai của nó:

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();

#### `whereRaw / orWhereRaw`

Các phương thức `whereRaw` và `orWhereRaw` có thể được sử dụng để thêm câu lệnh `where` raw vào query của bạn. Các phương thức này chấp nhận một mảng các tùy chọn tham số được truyền vào làm tham số thứ hai của nó:

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();

#### `havingRaw / orHavingRaw`

Các phương thức `havingRaw` và `orHavingRaw` có thể được sử dụng để set một chuỗi raw làm giá trị của câu lệnh `having`. Phương thức này chấp nhận một mảng các tùy chọn tham số được truyền vào làm tham số thứ hai của nó:

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > ?', [2500])
                    ->get();

#### `orderByRaw`

Phương thức `orderByRaw` có thể được sử dụng để set một chuỗi raw làm giá trị của câu lệnh `order by`:

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();

### `groupByRaw`

Phương thức `groupByRaw` có thể được sử dụng để set một string raw làm giá trị của mệnh đề `group by`:

    $orders = DB::table('orders')
                    ->select('city', 'state')
                    ->groupByRaw('city, state')
                    ->get();

<a name="joins"></a>
## Join

#### Inner Join Clause

Query builder cũng có thể được sử dụng để viết các câu lệnh join. Để thực hiện một "inner join" cơ bản, bạn có thể sử dụng phương thức `join` trên một instance của query builder. Tham số đầu tiên được truyền vào cho phương thức `join` là tên của bảng mà bạn cần join, trong khi các tham số còn lại là khai báo các cột dành cho phép join. Bạn thậm chí có thể join nhiều bảng trong cùng một query duy nhất:

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Left Join Clause / Right Join Clause

Nếu bạn muốn thực hiện "left join" hoặc "right join" thay vì "inner join", hãy sử dụng phương thức `leftJoin` hoặc `rightJoin`. Những phương thức này có cùng tham số với phương thức `join`:

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

    $users = DB::table('users')
                ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cross Join Clause

Để thực hiện "cross join", hãy sử dụng phương thức `crossJoin` với tên bảng mà bạn muốn cross join. Các cross join sẽ tạo ra một bảng mới có hàng là các hàng của bảng đầu tiên và bảng thứ hai nhân chéo vào nhau:

    $sizes = DB::table('sizes')
                ->crossJoin('colors')
                ->get();

#### Advanced Join Clauses

Bạn cũng có thể khai báo các lệnh join một cách cụ thể hơn. Để bắt đầu, hãy truyền một `Closure` làm tham số thứ hai vào phương thức `join`. `Closure` sẽ nhận vào một đối tượng `JoinClause` cho phép bạn khai báo các điều kiện đối với câu lệnh `join`:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

Nếu bạn muốn sử dụng lệnh "where" trong các lệnh join của bạn, bạn có thể sử dụng các phương thức `where` và `orWhere` trong join. Thay vì so sánh hai cột, các phương thức này sẽ so sánh một cột với một giá trị:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

#### Subquery Joins

Bạn có thể sử dụng các phương thức `joinSub`, `leftJoinSub` và `rightJoinSub` để nối một truy vấn với một truy vấn phụ. Mỗi phương thức này nhận vào ba tham số: một là truy vấn phụ, hai là bí danh của nó và ba là một Closure dùng để xác định các cột liên quan:

    $latestPosts = DB::table('posts')
                       ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                       ->where('is_published', true)
                       ->groupBy('user_id');

    $users = DB::table('users')
            ->joinSub($latestPosts, 'latest_posts', function ($join) {
                $join->on('users.id', '=', 'latest_posts.user_id');
            })->get();

<a name="unions"></a>
## Union

Query builder cũng cung cấp một cách nhanh chóng để "union" hai query lại với nhau. Ví dụ, bạn có thể tạo ra một query đầu tiên và sử dụng phương thức `union` để kết hợp nó với một query thứ hai:

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} Phương thức `unionAll` cũng có sẵn và có cấu trúc giống với phương thức `union`.

<a name="where-clauses"></a>
## Lệnh where

#### Simple Where Clauses

Bạn có thể sử dụng phương thức `where` trên một instance của query builder để thêm các lệnh `where` vào query. Một phương thức `where` cơ bản thường yêu cầu ba tham số. Tham số đầu tiên là tên của cột. Tham số thứ hai là một toán tử so sánh, có thể là bất kỳ toán tử nào được hỗ trợ bởi cơ sở dữ liệu. Cuối cùng, tham số thứ ba là giá trị để so sánh với cột.

Ví dụ: đây là một truy vấn kiểm tra giá trị của cột "votes" bằng 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();

Để thuận tiện hơn, nếu bạn muốn kiểm tra giá trị một cột bằng với một giá trị nhất định, bạn có thể truyền trực tiếp giá trị đó làm tham số thứ hai cho phương thức `where`:

    $users = DB::table('users')->where('votes', 100)->get();

Bạn có thể sử dụng các toán tử khác khi viết lệnh `where`:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

Bạn cũng có thể truyền một mảng các điều kiện cho phương thức `where`:

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Or Statements

Bạn có thể kết hợp các điều kiện với nhau cũng như thêm các lệnh `or` vào query. Phương thức `orWhere` chấp nhận các tham số tương tự như phương thức `where`:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

Nếu bạn cần nhóm một điều kiện "hoặc" trong một dấu ngoặc đơn, bạn có thể truyền một Closure làm tham số đầu tiên của phương thức `orWhere`:

    $users = DB::table('users')
                ->where('votes', '>', 100)
                ->orWhere(function($query) {
                    $query->where('name', 'Abigail')
                          ->where('votes', '>', 50);
                })
                ->get();

    // SQL: select * from users where votes > 100 or (name = 'Abigail' and votes > 50)

#### Additional Where Clauses

**whereBetween / orWhereBetween**

Phương thức `whereBetween` sẽ kiểm tra giá trị của một cột nằm giữa hai giá trị đã cho:

    $users = DB::table('users')
                ->whereBetween('votes', [1, 100])
                ->get();

**whereNotBetween / orWhereNotBetween**

Phương thức `whereNotBetween` sẽ kiểm tra giá trị của một cột nằm ngoài hai giá trị đã cho:

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

Phương thức `whereIn` sẽ kiểm tra giá trị của một cột đã cho có được chứa trong mảng các giá trị đã cho hay không:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

Phương thức `whereNotIn` sẽ kiểm tra giá trị của một cột đã cho là **không** tồn tại trong mảng đã cho hay không:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

> {note} Nếu bạn đang thêm một mảng integer lớn vào truy vấn của bạn, phương thức `whereIntegerInRaw` hoặc `whereIntegerNotInRaw` có thể được sử dụng để giảm đáng kể mức sử dụng bộ nhớ của bạn.

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**

Phương thức `whereNull` sẽ kiểm tra giá trị của một cột đã cho là `NULL` hay không:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

Phương thức `whereNotNull` sẽ kiểm tra giá trị của một cột đã cho không phải là `NULL`:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear / whereTime**

Phương thức `whereDate` có thể được sử dụng để so sánh giá trị của cột với một ngày:

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

Phương thức `whereMonth` có thể được sử dụng để so sánh giá trị của một cột với một tháng cụ thể của năm:

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

Phương thức `whereDay` có thể được sử dụng để so sánh giá trị của một cột với một ngày cụ thể trong tháng:

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

Phương thức `whereYear` có thể được sử dụng để so sánh giá trị của một cột với một năm cụ thể:

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

Phương thức `whereTime` có thể được sử dụng để so sánh giá trị của một cột với thời gian cụ thể:

    $users = DB::table('users')
                    ->whereTime('created_at', '=', '11:20:45')
                    ->get();

**whereColumn / orWhereColumn**

Phương thức `whereColumn` có thể được sử dụng để kiểm tra hai cột có bằng nhau hay không:

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

Bạn cũng có thể truyền một toán tử so sánh cho phương thức:

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

Phương thức `whereColumn` cũng có thể được truyền vào một mảng gồm nhiều điều kiện. Các điều kiện này sẽ được nối bằng toán tử `and`:

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at'],
                    ])->get();

<a name="parameter-grouping"></a>
### Group tham số

Thỉnh thoảng bạn có thể cần tạo ra lệnh where nâng cao như lệnh "where exists" hoặc các nhóm tham số lồng nhau. Query builder của Laravel cũng có thể xử lý được chúng. Để bắt đầu, chúng ta hãy xem một ví dụ về các nhóm điều kiện trong ngoặc đơn:

    $users = DB::table('users')
               ->where('name', '=', 'John')
               ->where(function ($query) {
                   $query->where('votes', '>', 100)
                         ->orWhere('title', '=', 'Admin');
               })
               ->get();

Như bạn có thể thấy, việc truyền một `Closure` vào phương thức `where` sẽ làm cho query builder bắt đầu tạo ra một nhóm điều kiện. `Closure` sẽ nhận vào một instance query builder mà bạn có thể sử dụng nó để set các điều kiện cần có vào trong nhóm dấu ngoặc đơn. Ví dụ trên sẽ tạo ra SQL như sau:

    select * from users where name = 'John' and (votes > 100 or title = 'Admin')

> {tip} Bạn nên nhóm các lệnh `orWhere` lại với nhau để tránh các hành vi không mong muốn khi sử dụng global scope.

<a name="where-exists-clauses"></a>
### Lệnh where exist

Phương thức `whereExists` cho phép bạn viết các lệnh SQL `where exists`. Phương thức `whereExists` chấp nhận một tham số `Closure`, sẽ nhận vào một instance query builder cho phép bạn định nghĩa thêm query mà sẽ được set vào bên trong lệnh "exists":

    $users = DB::table('users')
               ->whereExists(function ($query) {
                   $query->select(DB::raw(1))
                         ->from('orders')
                         ->whereRaw('orders.user_id = users.id');
               })
               ->get();

Truy vấn trên sẽ tạo ra lệnh SQL như sau:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="subquery-where-clauses"></a>
### Lệnh where cho truy vấn con

Thỉnh thoảng bạn có thể cần phải xây dựng một mệnh đề where so sánh kết quả của một truy vấn con với một giá trị nhất định. Bạn có thể thực hiện điều này bằng cách truyền một Closure và một giá trị cho phương thức `where`. Ví dụ: truy vấn sau sẽ lấy ra tất cả người dùng gần đây nhất mà có "tư cách thành viên" của một loại nhất định;

    use App\User;

    $users = User::where(function ($query) {
        $query->select('type')
            ->from('membership')
            ->whereColumn('user_id', 'users.id')
            ->orderByDesc('start_date')
            ->limit(1);
    }, 'Pro')->get();

<a name="json-where-clauses"></a>
### Lệnh where cho JSON

Laravel cũng hỗ trợ truy vấn vào các cột loại JSON trên cơ sở dữ liệu. Hiện tại, các cột loại JSON đã được hỗ trợ từ MySQL 5.7, PostgreSQL, SQL Server 2016, và SQLite 3.9.0 (với [JSON1 extension](https://www.sqlite.org/json1.html)). Để truy vấn vào cột loại JSON, hãy sử dụng toán tử `->`:

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

Bạn có thể sử dụng `whereJsonContains` để truy vấn mảng JSON (không hỗ trợ trên SQLite):

    $users = DB::table('users')
                    ->whereJsonContains('options->languages', 'en')
                    ->get();

MySQL và PostgreSQL hỗ trợ `whereJsonContains` với nhiều giá trị khác nhau:

    $users = DB::table('users')
                    ->whereJsonContains('options->languages', ['en', 'de'])
                    ->get();

Bạn có thể sử dụng `whereJsonLength` để truy vấn mảng JSON theo độ dài của chúng:

    $users = DB::table('users')
                    ->whereJsonLength('options->languages', 0)
                    ->get();

    $users = DB::table('users')
                    ->whereJsonLength('options->languages', '>', 1)
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit và Offset

#### orderBy

Phương thức `orderBy` cho phép bạn sắp xếp kết quả của truy vấn theo một cột nhất định. Tham số đầu tiên cho phương thức `orderBy` phải là một tên cột mà bạn muốn sắp xếp, trong khi tham số thứ hai là chiều sắp xếp, có thể là `asc` hoặc `desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

Nếu bạn cần sắp xếp theo nhiều cột, bạn có thể gọi `orderBy` nhiều lần nếu cần:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->orderBy('email', 'asc')
                    ->get();

#### latest / oldest

Các phương thức `latest` và `oldest` cho phép bạn dễ dàng sắp xếp kết quả theo ngày. Mặc định, kết quả sẽ được sắp xếp theo cột `created_at`. Hoặc, bạn có thể truyền vào một tên cột mà bạn muốn sắp xếp theo:

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### inRandomOrder

Phương thức `inRandomOrder` có thể được sử dụng để sắp xếp các kết quả của một truy vấn theo một cách ngẫu nhiên. Ví dụ: bạn có thể sử dụng phương thức này để lấy ra một người dùng ngẫu nhiên:

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### reorder

Phương thức `reorder` cho phép bạn xóa tất cả các orderBy hiện có và một tùy chọn cho phép bạn áp dụng một orderBy mới. Ví dụ: bạn có thể xóa tất cả các orderBy hiện có:

    $query = DB::table('users')->orderBy('name');

    $unorderedUsers = $query->reorder()->get();

Để xóa tất cả các orderBy hiện có và áp dụng một orderBy mới, hãy cung cấp một cột và một chiều làm tham số cho phương thức:

    $query = DB::table('users')->orderBy('name');

    $usersOrderedByEmail = $query->reorder('email', 'desc')->get();

#### groupBy / having

Các phương thức `groupBy` và `having` có thể được sử dụng để nhóm các kết quả truy vấn. Tham số của phương thức `having` cũng tương tự như phương thức `where`:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

Bạn cũng có thể truyền nhiều tham số vào phương thức `groupBy` để nhóm theo nhiều cột:

    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();

Để biết các câu lệnh `having` nâng cao, hãy xem phương thức [`havingRaw`](#raw-methods).

#### skip / take

Để giới hạn số lượng kết quả được trả về từ một truy vấn hoặc bỏ qua một số kết quả nhất định trong truy vấn, bạn có thể sử dụng các phương thức `skip` và `take`:

    $users = DB::table('users')->skip(10)->take(5)->get();

Ngoài ra, bạn có thể sử dụng các phương thức `limit` và `offset`:

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## Điều kiện cho lệnh

Thỉnh thoảng bạn cũng có thể muốn các câu lệnh chỉ áp dụng cho một truy vấn khi điều gì đó là đúng. Chẳng hạn, bạn chỉ có thể muốn áp dụng câu lệnh `where` nếu giá trị input này có xuất hiện trong một request. Bạn có thể thực hiện điều này bằng cách sử dụng phương thức `when` như sau:

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query, $role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();

Phương thức `when` chỉ chạy Closure khi tham số đầu tiên là `true`. Nếu tham số đầu tiên là `false`, thì Closure sẽ không được chạy.

Bạn có thể truyền một Closure khác làm tham số thứ ba cho phương thức `when`. Closure này sẽ được chạy nếu tham số đầu tiên trả về giá trị là `false`. Để minh họa cách sử dụng của tính năng này, chúng ta có thể sử dụng nó để set cách sắp xếp mặc định ở trong truy vấn:

    $sortBy = null;

    $users = DB::table('users')
                    ->when($sortBy, function ($query, $sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();

<a name="inserts"></a>
## Insert

Query builder cũng cung cấp một phương thức `insert` để thêm các bản ghi vào bảng của cơ sở dữ liệu. Phương thức `insert` chấp nhận một mảng các tên cột và giá trị của chúng:

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

Bạn thậm chí có thể thêm nhiều bản ghi vào bảng của cơ sở dữ liệu với chỉ một lệnh `insert` bằng cách truyền mảng trong một mảng khác. Mỗi mảng con đại diện cho một hàng được thêm vào trong bảng đó:

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0],
    ]);

Phương thức `insertOrIgnore` sẽ bỏ qua các bản ghi trùng lặp trong khi chèn bản ghi vào cơ sở dữ liệu:

    DB::table('users')->insertOrIgnore([
        ['id' => 1, 'email' => 'taylor@example.com'],
        ['id' => 2, 'email' => 'dayle@example.com'],
    ]);

#### Auto-Incrementing IDs

Nếu bảng có set id tự động tăng, hãy sử dụng phương thức `insertGetId` để thêm bản ghi đó vào và sau đó lấy ra ID:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} Khi sử dụng PostgreSQL, phương thức `insertGetId` này sẽ giả sử tên của cột tự động tăng là cột `id`. Nếu bạn muốn lấy ID từ một "chuỗi" khác, bạn có thể truyền vào tên cột làm tham số thứ hai cho phương thức `insertGetId`.

<a name="updates"></a>
## Update

Ngoài việc thêm các bản ghi vào cơ sở dữ liệu, query builder cũng có thể cập nhật các bản ghi hiện có bằng phương thức `update`. Phương thức `update`, giống như phương thức `insert`, chấp nhận một mảng các cặp cột và giá trị để cập nhật. Bạn có thể thêm điều kiện vào lệnh `update` bằng cách sử dụng lệnh `where`:

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['votes' => 1]);

#### Cập nhật hoặc thêm

Thỉnh thoảng bạn có thể muốn cập nhật một bản ghi hiện có trong cơ sở dữ liệu hoặc tạo mới nếu không có bản ghi nào phù hợp. Trong trường hợp đó, phương thức `updateOrInsert` có thể được sử dụng. Phương thức `updateOrInsert` chấp nhận hai tham số: một là mảng các điều kiện để tìm ra bản ghi và hai là một mảng các giá trị gồm các cột và các giá trị sẽ được cập nhật.

Phương thức `updateOrInsert` trước tiên sẽ thử tìm một bản ghi trong cơ sở dữ liệu bằng cách sử dụng các cặp giá trị của tham số đầu tiên. Nếu bản ghi tồn tại, nó sẽ cập nhật các giá trị trong tham số thứ hai vào bản ghi được tìm thấy. Nếu không thể tìm thấy bản ghi, thì một bản ghi mới sẽ được thêm vào cơ sở dữ liệu, các giá trị của bản ghi này là sự kết hợp của cả hai tham số một và hai:

    DB::table('users')
        ->updateOrInsert(
            ['email' => 'john@example.com', 'name' => 'John'],
            ['votes' => '2']
        );

<a name="updating-json-columns"></a>
### Update JSON Column

Khi cập nhật một cột JSON, bạn nên sử dụng cú pháp `->` để truy cập vào key thích hợp trong đối tượng JSON. Cách này sẽ hỗ trợ trên MySQL 5.7+ và PostgreSQL 9.5+:

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### Increment và Decrement

Query builder cũng cung cấp các phương thức để tăng hoặc giảm giá trị của một cột. Đây là một cách tắt, cung cấp một cách rõ ràng và ngắn gọn hơn so với việc viết thủ công một câu lệnh `update`.

Cả hai phương thức này đều chấp nhận ít nhất một tham số là: tên cột cần sửa. tham số thứ hai có thể được truyền vào tùy ý để kiểm soát số lượng sẽ được tăng hoặc giảm đi:

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

Bạn cũng có thể khai báo thêm các cột để cập nhật trong quá trình hoạt động:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

> {note} Các model event sẽ không được kích hoạt khi sử dụng phương thức `increment` và `decrement`.

<a name="deletes"></a>
## Delete

Query builder cũng có thể được sử dụng để xóa các bản ghi ra khỏi bảng thông qua phương thức `delete`. Bạn có thể thêm điều kiện vào các câu lệnh `delete` bằng cách thêm các lệnh `where` trước khi gọi phương thức `delete`:

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

Nếu bạn muốn truncate toàn bộ bảng, điều này sẽ xóa tất cả các hàng và set lại ID tự động tăng về 0, bạn có thể sử dụng phương thức `truncate`:

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## Pessimistic Locking

Query builder cũng có chứa một vài chức năng để giúp bạn thực hiện "pessimistic locking" trên các câu lệnh `select` của bạn. Để chạy câu lệnh với một "shared lock", bạn có thể sử dụng phương thức `sharedLock` trong một query. Shared lock sẽ ngăn các hàng đang được select không bị sửa cho đến khi transaction của bạn được commit:

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

Ngoài ra, bạn có thể sử dụng phương thức `lockForUpdate`. Lock "for update" sẽ ngăn các hàng bị sửa hoặc được select bằng một shared lock khác:

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

<a name="debugging"></a>
## Debugging

Bạn có thể sử dụng các phương thức `dd` hoặc `dump` trong khi xây dựng một truy vấn để dump ra các ràng buộc truy vấn và câu lệnh SQL. Phương thức `dd` sẽ hiển thị thông tin debug rồi sau đó sẽ ngừng thực hiện request. Trong khi phương thức `dump` sẽ hiển thị thông tin debug nhưng cho phép request tiếp tục được thực thi:

    DB::table('users')->where('votes', '>', 100)->dd();

    DB::table('users')->where('votes', '>', 100)->dump();
