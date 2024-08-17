# Database: Query Builder

- [Giới thiệu](#introduction)
- [Chạy Database Queries](#running-database-queries)
    - [Phân đoạn kết quả](#chunking-results)
    - [Streaming Results Lazily](#streaming-results-lazily)
    - [Thống kê](#aggregates)
- [Select Statements](#select-statements)
- [Biểu thức raw](#raw-expressions)
- [Join](#joins)
- [Union](#unions)
- [Lệnh where cơ bản](#basic-where-clauses)
    - [Lệnh where](#where-clauses)
    - [Lệnh where or](#or-where-clauses)
    - [Lệnh ưhere not](#where-not-clauses)
    - [Lệnh where cho json](#json-where-clauses)
    - [Lệnh where khác](#additional-where-clauses)
    - [Logic nhóm](#logical-grouping)
- [Lệnh where nâng cao](#advanced-where-clauses)
    - [Lệnh where exist](#where-exists-clauses)
    - [Lệnh where cho truy vấn con](#subquery-where-clauses)
    - [Lệnh where full text](#full-text-where-clauses)
- [Ordering, Grouping, Limit và Offset](#ordering-grouping-limit-and-offset)
    - [Ordering](#ordering)
    - [Grouping](#grouping)
    - [Limit và Offset](#limit-and-offset)
- [Điều kiện cho lệnh](#conditional-clauses)
- [Insert Statements](#insert-statements)
    - [Upserts](#upserts)
- [Update Statements](#update-statements)
    - [Update JSON Column](#updating-json-columns)
    - [Increment và Decrement](#increment-and-decrement)
- [Delete Statements](#delete-statements)
- [Pessimistic Locking](#pessimistic-locking)
- [Debugging](#debugging)

<a name="introduction"></a>
## Giới thiệu

Database query builder của Laravel cung cấp một interface thuận tiện, dễ dàng để tạo và chạy các query vào cơ sở dữ liệu. Nó có thể được sử dụng để thực hiện hầu hết các hành động cần thiết vào cơ sở dữ liệu trong application của bạn và làm việc hoàn hảo với tất cả các hệ thống cơ sở dữ liệu được hỗ trợ.

Query builder của Laravel sử dụng tham số PDO để bảo vệ application của bạn khỏi các cuộc tấn công SQL injection. Bạn sẽ không cần phải chuẩn hoá các chuỗi trước khi truyền đến query builder dưới dạng các ràng buộc query.

> **Warning**
> PDO không hỗ trợ truyền tên cột dưới dạng biến. Do đó, bạn không nên cho phép người dùng nhập tên cột mà truy vấn của bạn tham chiếu, bao gồm cả cột "order by".

<a name="running-database-queries"></a>
## Chạy Database Queries

<a name="retrieving-all-rows-from-a-table"></a>
#### Retrieving All Rows From A Table

Bạn có thể sử dụng phương thức `table` được cung cấp bởi facade `DB` để tạo một query. Phương thức `table` trả về một instance của query builder cho bảng đó, cho phép bạn kết hợp nhiều điều kiện vào trong một query và cuối cùng để lấy ra kết quả của query bằng cách sử dụng phương thức `get`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return \Illuminate\Http\Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

Phương thức `get` trả về một instance `Illuminate\Support\Collection` chứa các kết quả của query trong đó, mỗi kết quả là một instance của đối tượng `stdClass` của PHP. Bạn có thể truy cập vào giá trị của từng cột bằng cách khai báo tên cột như là một tên một thuộc tính của đối tượng:

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')->get();

    foreach ($users as $user) {
        echo $user->name;
    }

> **Note**
> Laravel collection sẽ cung cấp nhiều phương thức cực kỳ mạnh mẽ để kết nối và giảm dữ liệu. Để biết thêm thông tin về Laravel collection, hãy xem [tài liệu về collection](/docs/{{version}}/collections).

<a name="retrieving-a-single-row-column-from-a-table"></a>
#### Retrieving A Single Row / Column From A Table

Nếu bạn chỉ cần lấy ra một hàng từ một bảng cơ sở dữ liệu, bạn có thể sử dụng phương thức `first` của facade `DB`. Phương thức này sẽ trả về một đối tượng `stdClass`:

    $user = DB::table('users')->where('name', 'John')->first();

    return $user->email;

Nếu bạn không cần lấy ra toàn bộ giá trị của một hàng, bạn có thể lấy ra một giá trị của một bản ghi bằng cách sủ dụng phương thức `value`. Phương thức này sẽ trả về giá trị của cột mà bạn đã khai báo:

    $email = DB::table('users')->where('name', 'John')->value('email');

Để lấy một row theo giá trị cột `id` của nó, hãy sử dụng phương thức `find`:

    $user = DB::table('users')->find(3);

<a name="retrieving-a-list-of-column-values"></a>
#### Retrieving A List Of Column Values

Nếu bạn muốn lấy ra một instance `Illuminate\Support\Collection` chứa tất cả các giá trị của một cột, bạn có thể sử dụng phương thức `pluck`. Trong ví dụ này, chúng ta sẽ lấy ra một collection tiêu đề của  người dùng:

    use Illuminate\Support\Facades\DB;

    $titles = DB::table('users')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

Bạn có thể chỉ định cột làm khóa cho column kết quả bằng cách cung cấp tham số thứ hai cho phương thức `pluck`:

    $titles = DB::table('users')->pluck('title', 'name');

    foreach ($titles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### Phân đoạn kết quả

Nếu bạn cần làm việc với hàng ngàn bản ghi trong cơ sở dữ liệu, hãy xem xét sử dụng phương thức `chunk` được cung cấp facade `DB`. Phương thức này lấy ra một đoạn nhỏ kết quả tại một thời điểm và đưa từng đoạn đó vào một closure để xử lý. Phương thức này rất hữu ích để viết [Lệnh Artisan](/docs/{{version}}/artisan) xử lý hàng ngàn bản ghi. Ví dụ: hãy ra toàn bộ bảng `users` với số lượng khoảng 100 bản ghi cùng một lúc:

    use Illuminate\Support\Facades\DB;

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

Bạn có thể dừng xử lý các đoạn tiếp theo bằng cách trả về giá trị `false` từ closure:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...

        return false;
    });

Nếu bạn đang cập nhật bản ghi cơ sở dữ liệu trong khi chunking kết quả, thì kết quả đang được chunking của bạn có thể bị thay đổi theo những cách mà bạn không mong muốn. Nếu bạn định cập nhật bản ghi đã lấy ra trong khi đang chunking, thì tốt nhất bạn nên sử dụng phương thức `chunkById`. Phương thức này sẽ tự động chunking các kết quả dựa theo khóa chính của bản ghi:

    DB::table('users')->where('active', false)
        ->chunkById(100, function ($users) {
            foreach ($users as $user) {
                DB::table('users')
                    ->where('id', $user->id)
                    ->update(['active' => true]);
            }
        });

> **Warning**
> Khi cập nhật hoặc xóa các bản ghi bên trong lệnh callback của phương thức chunk, bất kỳ thay đổi nào đối với các khóa chính hoặc khóa ngoại đều có thể ảnh hưởng đến kết quả truy vấn của phương thức chunk. Điều này có thể dẫn đến việc một số bản ghi sẽ không được đưa vào bên trong kết quả chunk.

<a name="streaming-results-lazily"></a>
### Streaming Results Lazily

Phương thức `lazy` hoạt động tương tự như [phương thức `chunk`](#chunking-results) có nghĩa là nó thực thi truy vấn theo từng đoạn. Tuy nhiên, thay vì truyền từng đoạn vào một lệnh callback, phương thức `lazy()` sẽ trả về một [`LazyCollection`](/docs/{{version}}/collections#lazy-collections), cho phép bạn tương tác với kết quả dưới dạng một stream duy nhất:

```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->lazy()->each(function ($user) {
    //
});
```

Một lần nữa, nếu bạn dự định cập nhật các bản ghi đã lấy ra trong khi lặp chúng, thì tốt nhất bạn nên sử dụng các phương thức `lazyById` hoặc `lazyByIdDesc`. Các phương thức này sẽ tự động phân trang kết quả dựa trên khóa chính của bản ghi:

```php
DB::table('users')->where('active', false)
    ->lazyById()->each(function ($user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```

> **Warning**
> Khi cập nhật hoặc xóa bản ghi trong khi lặp, thì mọi thay đổi đối với khóa chính hoặc khóa ngoại đều có thể ảnh hưởng đến truy vấn chunk. Điều này có thể dẫn đến việc các bản ghi sẽ thiếu trong kết quả.

<a name="aggregates"></a>
### Thống kê

Query builder cũng cung cấp nhiều phương thức để lấy các giá trị thống kê như `count`, `max`, `min`, `avg`, và `sum`. Bạn có thể gọi bất kỳ phương thức nào sau khi khởi tạo query của bạn:

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

Dĩ nhiên, bạn có thể kết hợp các phương thức này với các câu lệnh khác để tinh chỉnh cách tính giá trị thống kê của bạn:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="determining-if-records-exist"></a>
#### Determining If Records Exist

Thay vì sử dụng phương thức `count` để xác định xem có tồn tại bản ghi nào phù hợp với các ràng buộc ở trong truy vấn hay không, thì bạn có thể sử dụng phương thức `exists` và `doesntExist`:

    if (DB::table('orders')->where('finalized', 1)->exists()) {
        // ...
    }

    if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
        // ...
    }

<a name="select-statements"></a>
## Select Statements

<a name="specifying-a-select-clause"></a>
#### Specifying A Select Clause

Không phải lúc nào bạn cũng muốn select tất cả các cột từ bảng cơ sở dữ liệu. Sử dụng phương thức `select`, bạn có thể khai báo một lệnh "select" tùy chỉnh cho query:

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')
                ->select('name', 'email as user_email')
                ->get();

Phương thức `distinct` cho phép bạn bắt query sẽ phải trả về các kết quả khác nhau:

    $users = DB::table('users')->distinct()->get();

Nếu bạn đã có một instance query builder và bạn muốn thêm một cột vào lệnh select hiện tại của nó, bạn có thể sử dụng phương thức `addSelect`:

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## Biểu thức raw

Thỉnh thoảng bạn có thể cần chèn một chuỗi tùy ý vào trong một query. Để tạo một chuỗi biểu thức raw như vậy, bạn có thể sử dụng phương thức `raw` được cung cấp bởi facade `DB`:

    $users = DB::table('users')
                 ->select(DB::raw('count(*) as user_count, status'))
                 ->where('status', '<>', 1)
                 ->groupBy('status')
                 ->get();

> **Warning**
> Các câu lệnh raw sẽ được đưa vào query dưới dạng là một chuỗi, vì vậy bạn cần phải cực kỳ cẩn thận để tránh tạo ra lỗ hổng SQL injection.

<a name="raw-methods"></a>
### Raw Methods

Thay vì sử dụng phương thức `raw`, bạn cũng có thể sử dụng các phương thức sau để thêm các biểu thức raw vào các phần khác nhau của query của bạn. **Hãy nhớ rằng, Laravel không thể đảm bảo là bất kỳ truy vấn nào sử dụng biểu thức raw đều được bảo vệ khỏi các lỗ hổng SQL injection.**

<a name="selectraw"></a>
#### `selectRaw`

Phương thức `selectRaw` có thể được sử dụng thay cho câu lệnh `addSelect(DB::raw(/* ... */))`. Phương thức này chấp nhận một mảng các tùy chọn tham số được truyền vào làm tham số thứ hai của nó:

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();

<a name="whereraw-orwhereraw"></a>
#### `whereRaw / orWhereRaw`

Các phương thức `whereRaw` và `orWhereRaw` có thể được sử dụng để thêm câu lệnh "where" raw vào query của bạn. Các phương thức này chấp nhận một mảng các tùy chọn tham số được truyền vào làm tham số thứ hai của nó:

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();

<a name="havingraw-orhavingraw"></a>
#### `havingRaw / orHavingRaw`

Các phương thức `havingRaw` và `orHavingRaw` có thể được sử dụng để cung cấp một chuỗi raw làm giá trị của câu lệnh "having". Phương thức này chấp nhận một mảng các tùy chọn tham số được truyền vào làm tham số thứ hai của nó:

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > ?', [2500])
                    ->get();

<a name="orderbyraw"></a>
#### `orderByRaw`

Phương thức `orderByRaw` có thể được sử dụng để cung cấp một chuỗi raw làm giá trị của câu lệnh "order by":

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();

<a name="groupbyraw"></a>
### `groupByRaw`

Phương thức `groupByRaw` có thể được sử dụng để cung cấp một string raw làm giá trị của mệnh đề `group by`:

    $orders = DB::table('orders')
                    ->select('city', 'state')
                    ->groupByRaw('city, state')
                    ->get();

<a name="joins"></a>
## Join

<a name="inner-join-clause"></a>
#### Inner Join Clause

Query builder cũng có thể được sử dụng để thêm các câu lệnh join vào query của bạn. Để thực hiện một "inner join" cơ bản, bạn có thể sử dụng phương thức `join` trên một instance của query builder. Tham số đầu tiên được truyền vào cho phương thức `join` là tên của bảng mà bạn cần join, trong khi các tham số còn lại là khai báo các cột dành cho phép join. Bạn thậm chí có thể join nhiều bảng trong cùng một query duy nhất:

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

<a name="left-join-right-join-clause"></a>
#### Left Join Clause / Right Join Clause

Nếu bạn muốn thực hiện "left join" hoặc "right join" thay vì "inner join", hãy sử dụng phương thức `leftJoin` hoặc `rightJoin`. Những phương thức này có cùng tham số với phương thức `join`:

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

    $users = DB::table('users')
                ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

<a name="cross-join-clause"></a>
#### Cross Join Clause

Bạn có thể dùng phương thức `crossJoin` để thực hiện một "cross join". Các cross join sẽ tạo ra một bảng mới có hàng là các hàng của bảng đầu tiên và bảng thứ hai nhân chéo vào nhau:

    $sizes = DB::table('sizes')
                ->crossJoin('colors')
                ->get();

<a name="advanced-join-clauses"></a>
#### Advanced Join Clauses

Bạn cũng có thể khai báo các lệnh join một cách cụ thể hơn. Để bắt đầu, hãy truyền một closure làm tham số thứ hai vào phương thức `join`. closure sẽ nhận vào một instance `Illuminate\Database\Query\JoinClause` cho phép bạn khai báo các điều kiện đối với câu lệnh "join":

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(/* ... */);
            })
            ->get();

Nếu bạn muốn sử dụng lệnh "where" trong các lệnh join của bạn, bạn có thể sử dụng các phương thức `where` và `orWhere` được cung cấp trong instance `JoinClause`. Thay vì so sánh hai cột, các phương thức này sẽ so sánh một cột với một giá trị:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="subquery-joins"></a>
#### Subquery Joins

Bạn có thể sử dụng các phương thức `joinSub`, `leftJoinSub` và `rightJoinSub` để nối một truy vấn với một truy vấn phụ. Mỗi phương thức này nhận vào ba tham số: một là truy vấn phụ, hai là bí danh của nó và ba là một Closure dùng để xác định các cột liên quan. Trong ví dụ này, chúng ta sẽ lấy ra một tập hợp người dùng trong đó mỗi bản ghi người dùng cũng chứa một cột timestamp `created_at` của bài đăng blog gần đây nhất của người dùng:

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

Query builder cũng cung cấp một phương thức tiện lợi để "union" hai hoặc nhiều query lại với nhau. Ví dụ, bạn có thể tạo ra một query đầu tiên và sử dụng phương thức `union` để kết hợp nó với nhiều query hơn:

    use Illuminate\Support\Facades\DB;

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

Ngoài phương thức `union`, query builder còn cung cấp thêm phương thức `unionAll`. Các truy vấn được kết hợp bằng phương thức `unionAll` sẽ không bị xóa các kết quả lặp nhau. Phương thức `unionAll` có cùng định dạng với phương thức `union`.

<a name="basic-where-clauses"></a>
## Lệnh where cơ bản

<a name="where-clauses"></a>
### Lệnh where

Bạn có thể sử dụng phương thức `where` của query builder để thêm lệnh "where" vào truy vấn. Lệnh gọi cơ bản nhất của phương thức `where` yêu cầu ba tham số. Tham số đầu tiên là tên của cột. Tham số thứ hai là một toán tử, có thể là bất kỳ toán tử nào được cơ sở dữ liệu hỗ trợ. Tham số thứ ba là giá trị để so sánh với giá trị của cột mà đã nhập ở tham số một.

Ví dụ: truy vấn sau sẽ lấy ra người dùng mà trong đó giá trị của cột `votes` bằng `100` và giá trị của cột `age` lớn hơn `35`:

    $users = DB::table('users')
                    ->where('votes', '=', 100)
                    ->where('age', '>', 35)
                    ->get();

Để thuận tiện hơn, nếu bạn muốn kiểm tra giá trị một cột `=` một giá trị nhất định, bạn có thể truyền giá trị đó làm tham số thứ hai cho phương thức `where`. Laravel sẽ sử dụng toán tử `=` trong trường hợp đó:

    $users = DB::table('users')->where('votes', 100)->get();

Như đã đề ở cập trước đó, bạn có thể sử dụng bất kỳ toán tử nào mà được hệ thống cơ sở dữ liệu của bạn hỗ trợ:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

Bạn cũng có thể truyền một mảng các điều kiện cho phương thức `where`. Mỗi phần tử của mảng phải là một mảng con chứa ba tham số thường được truyền cho phương thức `where`:

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

> **Warning**
> PDO không hỗ trợ truyền tên cột dưới dạng biến. Do đó, bạn không nên cho phép người dùng nhập tên cột mà truy vấn của bạn tham chiếu, bao gồm cả cột "order by".

<a name="or-where-clauses"></a>
### Lệnh where or

Khi kết hợp phương thức `where` của query builder với nhau, thì các lệnh "where" này sẽ được nối với nhau bằng toán tử `and`. Tuy nhiên, bạn có thể sử dụng phương thức `orWhere` để nối một lệnh vào một truy vấn bằng toán tử `or`. Phương thức `orWhere` chấp nhận các tham số tương tự như phương thức `where`:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

Nếu bạn cần nhóm một điều kiện "hoặc" trong một dấu ngoặc đơn, bạn có thể truyền một closure làm tham số đầu tiên của phương thức `orWhere`:

    $users = DB::table('users')
                ->where('votes', '>', 100)
                ->orWhere(function($query) {
                    $query->where('name', 'Abigail')
                          ->where('votes', '>', 50);
                })
                ->get();

Ví dụ trên sẽ tạo ra một câu lệnh SQL như sau:

```sql
select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
```

> **Warning**
> Bạn nên nhóm các lệnh `orWhere` lại với nhau để tránh các hành vi không mong muốn khi sử dụng global scope.

<a name="where-not-clauses"></a>
### Lệnh ưhere not

Các phương thức `whereNot` và `orWhereNot` có thể được sử dụng để phủ định một nhóm các lệnh nhất định. Ví dụ, truy vấn sau đây bỏ qua các sản phẩm đang được thanh lý hoặc có giá dưới mười:

    $products = DB::table('products')
                    ->whereNot(function ($query) {
                        $query->where('clearance', true)
                              ->orWhere('price', '<', 10);
                    })
                    ->get();

<a name="json-where-clauses"></a>
### Lệnh where cho json

Laravel cũng hỗ trợ truy vấn vào các cột loại JSON trên cơ sở dữ liệu. Hiện tại, các cột loại JSON đã được hỗ trợ từ MySQL 5.7+, PostgreSQL, SQL Server 2016, và SQLite 3.39.0 (với [JSON1 extension](https://www.sqlite.org/json1.html)). Để truy vấn vào cột loại JSON, hãy sử dụng toán tử `->`:

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

Bạn có thể sử dụng `whereJsonContains` để truy vấn vào mảng JSON. Tính năng này sẽ không được hỗ trợ bởi các cơ sở dữ liệu SQLite mà có phiên bản nhỏ hơn 3.38.0:

    $users = DB::table('users')
                    ->whereJsonContains('options->languages', 'en')
                    ->get();

Nếu ứng dụng của bạn đang sử dụng cơ sở dữ liệu MySQL hoặc PostgreSQL, bạn có thể truyền một mảng giá trị cho phương thức `whereJsonContains`:

    $users = DB::table('users')
                    ->whereJsonContains('options->languages', ['en', 'de'])
                    ->get();

Bạn có thể sử dụng phương thức `whereJsonLength` để truy vấn mảng JSON theo độ dài của chúng:

    $users = DB::table('users')
                    ->whereJsonLength('options->languages', 0)
                    ->get();

    $users = DB::table('users')
                    ->whereJsonLength('options->languages', '>', 1)
                    ->get();

<a name="additional-where-clauses"></a>
### Lệnh where khác

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

**whereBetweenColumns / whereNotBetweenColumns / orWhereBetweenColumns / orWhereNotBetweenColumns**

Phương thức `whereBetweenColumns` sẽ kiểm tra giá trị của một cột nằm giữa hai giá trị của hai cột có trong cùng một hàng của một bảng:

    $patients = DB::table('patients')
                           ->whereBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
                           ->get();

Phương thức `whereNotBetweenColumns` sẽ kiểm tra giá trị của một cột nằm ngoài hai giá trị của hai cột có trong cùng một hàng của một bảng:

    $patients = DB::table('patients')
                           ->whereNotBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
                           ->get();

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

Phương thức `whereIn` sẽ kiểm tra giá trị của một cột đã cho có được chứa trong mảng các giá trị đã cho hay không:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

Phương thức `whereNotIn` sẽ kiểm tra giá trị của một cột đã cho là không tồn tại trong mảng đã cho hay không:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

Bạn cũng có thể cung cấp một đối tượng truy vấn làm tham số thứ hai của phương thức `whereIn`:

    $activeUsers = DB::table('users')->select('id')->where('is_active', 1);

    $users = DB::table('comments')
                        ->whereIn('user_id', $activeUsers)
                        ->get();

Ví dụ trên sẽ tạo ra câu lệnh SQL như sau:

```sql
select * from comments where user_id in (
    select id
    from users
    where is_active = 1
)
```

> **Warning**
> Nếu bạn đang thêm một mảng integer lớn vào truy vấn của bạn, phương thức `whereIntegerInRaw` hoặc `whereIntegerNotInRaw` có thể được sử dụng để giảm đáng kể mức sử dụng bộ nhớ của bạn.

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

Phương thức `whereMonth` có thể được sử dụng để so sánh giá trị của một cột với một tháng cụ thể:

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

Bạn cũng có thể truyền một toán tử so sánh cho phương thức `whereColumn`:

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

Bạn cũng có thể truyền một mảng gồm các cột dành cho so sánh sang phương thức `whereColumn`. Các điều kiện này sẽ được nối bằng toán tử `and`:

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at'],
                    ])->get();

<a name="logical-grouping"></a>
### Logic nhóm

Thỉnhh thoảng bạn có thể cần nhóm một số lệnh "where" vào trong một dấu ngoặc đơn để đạt được cách nhóm mà bạn mong muốn cho truy vấn của mình. Trên thực tế, bạn nên luôn nhóm các phương thức `orWhere` trong dấu ngoặc đơn để tránh hành vi truy vấn không mong muốn. Để thực hiện điều này, bạn có thể truyền một closure cho phương thức `where`:

    $users = DB::table('users')
               ->where('name', '=', 'John')
               ->where(function ($query) {
                   $query->where('votes', '>', 100)
                         ->orWhere('title', '=', 'Admin');
               })
               ->get();

Như bạn có thể thấy, việc truyền một closure vào phương thức `where` sẽ làm cho query builder bắt đầu tạo ra một nhóm điều kiện. closure sẽ nhận vào một instance query builder mà bạn có thể sử dụng nó để set các điều kiện cần có vào trong nhóm dấu ngoặc đơn. Ví dụ trên sẽ tạo ra SQL như sau:

```sql
select * from users where name = 'John' and (votes > 100 or title = 'Admin')
```

> **Warning**
> Bạn nên nhóm các lệnh `orWhere` lại với nhau để tránh các hành vi không mong muốn khi sử dụng global scope.

<a name="advanced-where-clauses"></a>
### Lệnh where nâng cao

<a name="where-exists-clauses"></a>
### Lệnh where exist

Phương thức `whereExists` cho phép bạn viết các lệnh SQL "where exists". Phương thức `whereExists` chấp nhận một closure, sẽ nhận vào một instance query builder, cho phép bạn định nghĩa thêm query mà sẽ được set vào bên trong lệnh "exists":

    $users = DB::table('users')
               ->whereExists(function ($query) {
                   $query->select(DB::raw(1))
                         ->from('orders')
                         ->whereColumn('orders.user_id', 'users.id');
               })
               ->get();

Truy vấn trên sẽ tạo ra lệnh SQL như sau:

```sql
select * from users
where exists (
    select 1
    from orders
    where orders.user_id = users.id
)
```

<a name="subquery-where-clauses"></a>
### Lệnh where cho truy vấn con

Thỉnh thoảng bạn có thể cần phải xây dựng một mệnh đề "where" so sánh kết quả của một truy vấn con với một giá trị nhất định. Bạn có thể thực hiện điều này bằng cách truyền một closure và một giá trị cho phương thức `where`. Ví dụ: truy vấn sau sẽ lấy ra tất cả người dùng gần đây nhất mà có "tư cách thành viên" của một loại nhất định;

    use App\Models\User;

    $users = User::where(function ($query) {
        $query->select('type')
            ->from('membership')
            ->whereColumn('membership.user_id', 'users.id')
            ->orderByDesc('membership.start_date')
            ->limit(1);
    }, 'Pro')->get();

Hoặc, bạn có thể cần xây dựng một lệnh "where" để so sánh một cột với kết quả của một truy vấn con. Bạn có thể thực hiện điều này bằng cách truyền một cột, một toán tử và một closure cho phương thức `where`. Ví dụ: truy vấn sau sẽ lấy ra tất cả các bản ghi mà có thu nhập nhỏ hơn mức trung bình;

    use App\Models\Income;

    $incomes = Income::where('amount', '<', function ($query) {
        $query->selectRaw('avg(i.amount)')->from('incomes as i');
    })->get();

<a name="full-text-where-clauses"></a>
### Lệnh where full text

> **Warning**
> Lệnh where full text hiện đang được MySQL và PostgreSQL hỗ trợ.

Các phương thức `whereFullText` và `orWhereFullText` có thể được sử dụng để thêm các lệnh "where" full text vào truy vấn cho các cột có [index full text](/docs/{{version}}/migrations#available-index-types). Các phương thức này sẽ được Laravel chuyển thành các câu SQL phù hợp cho hệ thống cơ sở dữ liệu. Ví dụ, một lệnh `MATCH AGAINST` sẽ được tạo cho các ứng dụng sử dụng cơ sở dữ liệu MySQL:

    $users = DB::table('users')
               ->whereFullText('bio', 'web developer')
               ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit và Offset

<a name="ordering"></a>
### Ordering

<a name="orderby"></a>
#### The `orderBy` Method

Phương thức `orderBy` cho phép bạn sắp xếp kết quả của truy vấn theo một cột nhất định. Tham số đầu tiên được chấp nhận bởi phương thức `orderBy` phải là một tên cột mà bạn muốn sắp xếp, trong khi tham số thứ hai sẽ xác định chiều sắp xếp, có thể là `asc` hoặc `desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

Để sắp xếp theo nhiều cột, bạn có thể đơn giản là gọi `orderBy` nhiều lần nếu cần thiết:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->orderBy('email', 'asc')
                    ->get();

<a name="latest-oldest"></a>
#### The `latest` & `oldest` Methods

Các phương thức `latest` và `oldest` cho phép bạn dễ dàng sắp xếp kết quả theo ngày. Mặc định, kết quả sẽ được sắp xếp theo cột `created_at` của bảng. Hoặc, bạn có thể truyền vào một tên cột mà bạn muốn sắp xếp theo:

    $user = DB::table('users')
                    ->latest()
                    ->first();

<a name="random-ordering"></a>
#### Random Ordering

Phương thức `inRandomOrder` có thể được sử dụng để sắp xếp các kết quả của một truy vấn theo một cách ngẫu nhiên. Ví dụ: bạn có thể sử dụng phương thức này để lấy ra một người dùng ngẫu nhiên:

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

<a name="removing-existing-orderings"></a>
#### Removing Existing Orderings

Phương thức `reorder` sẽ loại bỏ tất cả các lệnh "order by" đã được áp dụng trước đó cho truy vấn:

    $query = DB::table('users')->orderBy('name');

    $unorderedUsers = $query->reorder()->get();

Bạn có thể truyền một cột và hướng sắp xếp khi gọi phương thức `reorder` để xóa tất cả các lệnh "order by" hiện tại và áp dụng một thứ tự sắp xếp cột mới cho truy vấn:

    $query = DB::table('users')->orderBy('name');

    $usersOrderedByEmail = $query->reorder('email', 'desc')->get();

<a name="grouping"></a>
### Grouping

<a name="groupby-having"></a>
#### The `groupBy` & `having` Methods

Như bạn mong đợi, các phương thức `groupBy` và `having` có thể được sử dụng để nhóm các kết quả truy vấn. Tham số của phương thức `having` cũng tương tự như phương thức `where`:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

Bạn có thể sử dụng phương thức `havingBetween` để lọc kết quả trong một phạm vi nhất định:

    $report = DB::table('orders')
                    ->selectRaw('count(id) as number_of_orders, customer_id')
                    ->groupBy('customer_id')
                    ->havingBetween('number_of_orders', [5, 15])
                    ->get();

Bạn cũng có thể truyền nhiều tham số vào phương thức `groupBy` để nhóm theo nhiều cột:

    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();

Để tạo thêm nhiều các câu lệnh `having` nâng cao, hãy xem phương thức [`havingRaw`](#raw-methods).

<a name="limit-and-offset"></a>
### Limit và Offset

<a name="skip-take"></a>
#### The `skip` & `take` Methods

Bạn có thể sử dụng phương thức `skip` và `take` để giới hạn số lượng kết quả được trả về từ một truy vấn hoặc bỏ qua một số kết quả nhất định trong truy vấn, bạn có thể sử dụng các phương thức `skip` và `take`:

    $users = DB::table('users')->skip(10)->take(5)->get();

Ngoài ra, bạn có thể sử dụng các phương thức `limit` và `offset`. Các phương thức này có chức năng tương đương với các phương thức `take` và `skip`:

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## Điều kiện cho lệnh

Thỉnh thoảng bạn cũng có thể muốn một lệnh truy vấn sẽ được áp dụng cho một truy vấn dựa trên một điều kiện nhất định. Chẳng hạn, bạn chỉ có thể muốn áp dụng câu lệnh `where` nếu giá trị input này có xuất hiện trong một HTTP request. Bạn có thể thực hiện điều này bằng cách sử dụng phương thức `when` như sau:

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query, $role) {
                        $query->where('role_id', $role);
                    })
                    ->get();

Phương thức `when` chỉ chạy closure khi tham số đầu tiên là `true`. Nếu tham số đầu tiên là `false`, thì closure sẽ không được chạy. Vì vậy, trong ví dụ trên, closure mà được cung cấp cho phương thức `when` sẽ chỉ được gọi nếu field `role` có trong request đến và có giá trị là `true`.

Bạn có thể truyền một closure khác làm tham số thứ ba cho phương thức `when`. closure này sẽ được chạy nếu tham số đầu tiên trả về giá trị là `false`. Để minh họa cách sử dụng của tính năng này, chúng ta có thể sử dụng nó để set cách sắp xếp mặc định ở trong truy vấn:

    $sortByVotes = $request->input('sort_by_votes');

    $users = DB::table('users')
                    ->when($sortByVotes, function ($query, $sortByVotes) {
                        $query->orderBy('votes');
                    }, function ($query) {
                        $query->orderBy('name');
                    })
                    ->get();

<a name="insert-statements"></a>
## Insert Statements

Query builder cũng cung cấp một phương thức `insert` để thêm các bản ghi vào bảng của cơ sở dữ liệu. Phương thức `insert` chấp nhận một mảng các tên cột và giá trị của chúng:

    DB::table('users')->insert([
        'email' => 'kayla@example.com',
        'votes' => 0
    ]);

Bạn có thể thêm nhiều bản ghi vào bảng của cơ sở dữ liệu với chỉ một lệnh `insert` bằng cách truyền mảng trong một mảng khác. Mỗi mảng con đại diện cho một hàng được thêm vào trong bảng đó:

    DB::table('users')->insert([
        ['email' => 'picard@example.com', 'votes' => 0],
        ['email' => 'janeway@example.com', 'votes' => 0],
    ]);

Phương thức `insertOrIgnore` sẽ bỏ qua các lỗi có trong khi insert bản ghi vào cơ sở dữ liệu. Khi sử dụng phương thức này, bạn nên biết rằng lỗi bản ghi trùng lặp sẽ bị bỏ qua và các loại lỗi khác cũng có thể bị bỏ qua tùy thuộc vào database engine. Ví dụ, `insertOrIgnore` sẽ [bỏ qua chế độ strict của MySQL](https://dev.mysql.com/doc/refman/en/sql-mode.html#ignore-effect-on-execution):

    DB::table('users')->insertOrIgnore([
        ['id' => 1, 'email' => 'sisko@example.com'],
        ['id' => 2, 'email' => 'archer@example.com'],
    ]);

Phương thức `insertUsing` sẽ insert các bản ghi mới vào bảng trong khi vẫn sử dụng truy vấn phụ để xác định dữ liệu cần insert:

    DB::table('pruned_users')->insertUsing([
        'id', 'name', 'email', 'email_verified_at'
    ], DB::table('users')->select(
        'id', 'name', 'email', 'email_verified_at'
    )->where('updated_at', '<=', now()->subMonth()));

<a name="auto-incrementing-ids"></a>
#### Auto-Incrementing IDs

Nếu bảng có set id tự động tăng, hãy sử dụng phương thức `insertGetId` để thêm bản ghi đó vào và sau đó lấy ra ID:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> **Warning**
> Khi sử dụng PostgreSQL, phương thức `insertGetId` này sẽ giả sử tên của cột tự động tăng là cột `id`. Nếu bạn muốn lấy ID từ một "chuỗi" khác, bạn có thể truyền vào tên cột làm tham số thứ hai cho phương thức `insertGetId`.

<a name="upserts"></a>
### Upserts

Phương thức `upsert` sẽ thêm các bản ghi không tồn tại và cập nhật lại các bản ghi đã tồn tại với các giá trị mới mà bạn có thể chỉ định. Tham số đầu tiên của phương thức chứa các giá trị cần thêm hoặc cần cập nhật, trong khi tham số thứ hai liệt kê (các) cột khoá chính duy nhất trong các bản ghi để xác định cập nhật hay thêm bản ghi mới. Tham số thứ ba và cuối cùng của phương thức là một mảng các cột cần được cập nhật nếu bản ghi khớp đã tồn tại trong cơ sở dữ liệu:

    DB::table('flights')->upsert(
        [
            ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
            ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
        ],
        ['departure', 'destination'],
        ['price']
    );

Trong ví dụ trên, Laravel sẽ cố gắng thêm hai bản ghi. Nếu một bản ghi đã tồn tại với cùng giá trị cột `departure` và `destination`, thì Laravel sẽ cập nhật cột `price` của bản ghi đó.

> **Warning**
> Tất cả các cơ sở dữ liệu ngoại trừ SQL Server đều yêu cầu các cột trong tham số thứ hai của phương thức `upsert` phải ở dạng "primary" hoặc "unique". Ngoài ra, driver cơ sở dữ liệu cũng MySQL bỏ qua tham số thứ hai của phương thức `upsert` và luôn sử dụng các "primary" và "unique" của bảng để phát hiện ra các bản ghi hiện có.

<a name="update-statements"></a>
## Update Statements

Ngoài việc thêm các bản ghi vào cơ sở dữ liệu, query builder cũng có thể cập nhật các bản ghi hiện có bằng phương thức `update`. Phương thức `update`, giống như phương thức `insert`, chấp nhận một mảng các cặp cột và giá trị để biết các cột sẽ được cập nhật. Phương thức `update` trả về số lượng row bị ảnh hưởng. Bạn có thể thêm điều kiện vào lệnh `update` bằng cách sử dụng lệnh `where`:

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['votes' => 1]);

<a name="update-or-insert"></a>
#### Cập nhật hoặc thêm

Thỉnh thoảng bạn có thể muốn cập nhật một bản ghi hiện có trong cơ sở dữ liệu hoặc tạo mới nếu không có bản ghi nào phù hợp. Trong trường hợp đó, phương thức `updateOrInsert` có thể được sử dụng. Phương thức `updateOrInsert` chấp nhận hai tham số: một là mảng các điều kiện để tìm ra bản ghi và hai là một mảng các giá trị gồm các cột và các giá trị để biết các cột sẽ được cập nhật.

Phương thức `updateOrInsert` sẽ thử tìm một bản ghi trong cơ sở dữ liệu bằng cách sử dụng các cặp giá trị của tham số đầu tiên. Nếu bản ghi tồn tại, nó sẽ cập nhật các giá trị trong tham số thứ hai vào bản ghi được tìm thấy. Nếu không thể tìm thấy bản ghi, thì một bản ghi mới sẽ được thêm vào cơ sở dữ liệu, các giá trị của bản ghi này là sự kết hợp của cả hai tham số một và hai:

    DB::table('users')
        ->updateOrInsert(
            ['email' => 'john@example.com', 'name' => 'John'],
            ['votes' => '2']
        );

<a name="updating-json-columns"></a>
### Update JSON Column

Khi cập nhật một cột JSON, bạn nên sử dụng cú pháp `->` để cập nhập vào các key thích hợp trong đối tượng JSON. Cách này sẽ hỗ trợ trên MySQL 5.7+ và PostgreSQL 9.5+:

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### Increment và Decrement

Query builder cũng cung cấp các phương thức để tăng hoặc giảm giá trị của một cột. Cả hai phương thức này đều chấp nhận ít nhất một tham số là: tên cột cần sửa. Tham số thứ hai có thể cung cấp một giá trị số lượng cụ thể sẽ được tăng hoặc giảm đi:

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

Nếu cần, bạn cũng có thể khai báo thêm các cột để cập nhật trong quá trình tăng hoặc giảm:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

Ngoài ra, bạn có thể tăng hoặc giảm nhiều cột cùng lúc bằng cách sử dụng phương thức `incrementEach` và `decrementEach`:

    DB::table('users')->incrementEach([
        'votes' => 5,
        'balance' => 100,
    ]);

<a name="delete-statements"></a>
## Delete Statements

Phương thức `delete` của query builder có thể được sử dụng để xóa các bản ghi ra khỏi bảng. Phương thức `delete` sẽ trả về số hàng bị ảnh hưởng. Bạn có thể hạn chế các bản ghi bị ảnh hưởng bởi câu lệnh `delete` này bằng cách thêm mệnh đề "where" trước khi gọi phương thức `delete`:

    $deleted = DB::table('users')->delete();

    $deleted = DB::table('users')->where('votes', '>', 100)->delete();

Nếu bạn muốn truncate toàn bộ bảng, điều này sẽ xóa tất cả các hàng và set lại ID tự động tăng về 0, bạn có thể sử dụng phương thức `truncate`:

    DB::table('users')->truncate();

<a name="table-truncation-and-postgresql"></a>
#### Table Truncation & PostgreSQL

Khi truncate cơ sở dữ liệu PostgreSQL, tính năng `CASCADE` sẽ được áp dụng. Điều này có nghĩa là tất cả các bản ghi liên quan đến khóa ngoại có trong các bảng khác cũng sẽ bị xóa.

<a name="pessimistic-locking"></a>
## Pessimistic Locking

Query builder cũng có chứa một vài phương thức để giúp bạn đạt được trạng thái "pessimistic locking" khi chạy các câu lệnh `select` của bạn. Để chạy câu lệnh với một "shared lock", bạn có thể sử dụng phương thức `sharedLock` trong một query. Shared lock sẽ ngăn các hàng đang được select sẽ không bị sửa cho đến khi transaction của bạn được commit:

    DB::table('users')
            ->where('votes', '>', 100)
            ->sharedLock()
            ->get();

Ngoài ra, bạn có thể sử dụng phương thức `lockForUpdate`. Lock "for update" sẽ ngăn các hàng được select bị sửa hoặc được select bằng một shared lock khác:

    DB::table('users')
            ->where('votes', '>', 100)
            ->lockForUpdate()
            ->get();

<a name="debugging"></a>
## Debugging

Bạn có thể sử dụng các phương thức `dd` và `dump` trong khi xây dựng một truy vấn hiện tại để dump ra các ràng buộc truy vấn và câu lệnh SQL. Phương thức `dd` sẽ hiển thị thông tin debug rồi sau đó sẽ ngừng thực hiện request. Trong khi phương thức `dump` sẽ hiển thị thông tin debug nhưng cho phép request tiếp tục được thực thi:

    DB::table('users')->where('votes', '>', 100)->dd();

    DB::table('users')->where('votes', '>', 100)->dump();
