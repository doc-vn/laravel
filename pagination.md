# Database: Pagination

- [Giới thiệu](#introduction)
- [Cách dùng cơ bản](#basic-usage)
    - [Phân trang từ một query builder](#paginating-query-builder-results)
    - [Phân trang từ Eloquent](#paginating-eloquent-results)
    - [Phân trang từ con trỏ](#cursor-pagination)
    - [Tự tạo một phân trang](#manually-creating-a-paginator)
    - [Tuỳ biến Pagination URLs](#customizing-pagination-urls)
- [Hiển thị kết quả phân trang](#displaying-pagination-results)
    - [Điều chỉnh Pagination Link Window](#adjusting-the-pagination-link-window)
    - [Chuyển kết quả thành JSON](#converting-results-to-json)
- [Tuỳ biến View của phân trang](#customizing-the-pagination-view)
    - [Dùng Bootstrap](#using-bootstrap)
- [Paginator và LengthAwarePaginator Instance Methods](#paginator-instance-methods)
- [Cursor Paginator Instance Methods](#cursor-paginator-instance-methods)

<a name="introduction"></a>
## Giới thiệu

Trong các framework khác, phân trang có thể rất khổ. Chúng tôi hy vọng cách tiếp cận phân trang của Laravel sẽ là một luồng gió mới. Trình phân trang của Laravel được tích hợp sẵn với [query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent), cung cấp một cách thuận tiện, dễ sử dụng để phân trang các bản ghi cơ sở dữ liệu với zero cấu hình.

Mặc định, HTML được tạo ra bởi trình phân trang tương thích với [Tailwind CSS framework](https://tailwindcss.com/); tuy nhiên, phân trang với Bootstrap cũng có sẵn.

<a name="tailwind-jit"></a>
#### Tailwind JIT

Nếu bạn đang sử dụng view phân trang Tailwind mặc định của Laravel và engine Tailwind JIT, bạn nên đảm bảo là khóa `content` của file `tailwind.config.js` trong ứng dụng của bạn tham chiếu đến các view phân trang của Laravel để các class Tailwind của chúng không bị xóa:

```js
content: [
    './resources/**/*.blade.php',
    './resources/**/*.js',
    './resources/**/*.vue',
    './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
],
```

<a name="basic-usage"></a>
## Cách dùng cơ bản

<a name="paginating-query-builder-results"></a>
### Phân trang từ một query builder

Có một số cách để phân trang. Đơn giản nhất là sử dụng phương thức `paginate` trong một [query builder](/docs/{{version}}/queries) hoặc một [Eloquent query](/docs/{{version}}/eloquent). Phương thức `paginate` sẽ tự động đảm nhiệm việc set "limit" và "offset" của query dựa trên trang hiện tại đang được người dùng xem. Mặc định, trang hiện tại sẽ được dò tìm giá trị của tham số `page` trong HTTP request. Giá trị này sẽ được tự động dò tìm bởi Laravel và cũng được tự động thêm vào sau các link sau quá trình phân trang.

Trong ví dụ này, chỉ có một tham số duy nhất được truyền vào phương thức `paginate` đó là số lượng dữ liệu mà bạn muốn hiển thị "trên mỗi trang". Trong trường hợp này, hãy khai báo chúng ta muốn hiển thị `15` dữ liệu trên mỗi trang:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class UserController extends Controller
    {
        /**
         * Show all application users.
         *
         * @return \Illuminate\Http\Response
         */
        public function index()
        {
            return view('user.index', [
                'users' => DB::table('users')->paginate(15)
            ]);
        }
    }

<a name="simple-pagination"></a>
#### Simple Pagination

Phương thức `paginate` sẽ đếm tổng số bản ghi khớp với truy vấn trước khi lấy ra các bản ghi từ cơ sở dữ liệu. Điều này được thực hiện để paginator biết tổng cộng có bao nhiêu trang. Tuy nhiên, nếu bạn không định hiển thị tổng số trang trong giao diện người dùng của ứng dụng thì truy vấn số lượng bản ghi là không cần thiết.

Vì vậy, nếu bạn chỉ cần hiển thị các link đơn giản ví dụ như "Next" và "Previous" trong UI của application của bạn, bạn có thể sử dụng phương thức `simplePaginate` để thực hiện query đơn giản, hiệu quả.

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### Phân trang từ Eloquent

Bạn cũng có thể phân trang bằng các truy vấn [Eloquent](/docs/{{version}}/eloquent). Trong ví dụ này, chúng ta sẽ phân trang model `App\Models\User` và cho biết chúng ta sẽ dự định hiển thị 15 bản ghi trên mỗi trang. Như bạn có thể thấy, cú pháp này gần giống với phân trang bằng query builder:

    use App\Models\User;

    $users = User::paginate(15);

Mặc định, bạn có thể gọi phương thức `paginate` sau khi set các điều kiện cho truy vấn, chẳng hạn như câu lệnh `where`:

    $users = User::where('votes', '>', 100)->paginate(15);

Bạn cũng có thể sử dụng phương thức `simplePaginate` khi phân trang trên các model Eloquent:

    $users = User::where('votes', '>', 100)->simplePaginate(15);

Tương tự, bạn có thể sử dụng phương thức `cursorPaginate` để phân trang từ con trỏ trong các model Eloquent:

    $users = User::where('votes', '>', 100)->cursorPaginate(15);

<a name="multiple-paginator-instances-per-page"></a>
#### Multiple Paginator Instances Per Page

Thỉnh thoảng, bạn có thể cần hiển thị hai phân trang khác nhau trong một màn hình do ứng dụng của bạn hiển thị. Tuy nhiên, nếu cả hai instance phân trang đều sử dụng tham số query `page` để lưu lại trang hiện tại thì hai phân trang sẽ xung đột với nhau. Để giải quyết xung đột này, bạn có thể truyền tên của tham số query mà bạn muốn sử dụng để lưu lại trang hiện tại của phân trang thông qua tham số thứ ba được cung cấp cho các phương thức `paginate`, `simplePaginate` và `cursorPaginate`:

    use App\Models\User;

    $users = User::where('votes', '>', 100)->paginate(
        $perPage = 15, $columns = ['*'], $pageName = 'users'
    );

<a name="cursor-pagination"></a>
### Phân trang từ con trỏ

Trong khi `paginate` và `simplePaginate` tạo các truy vấn bằng cách sử dụng mệnh đề "offset" của SQL, thì việc phân trang từ con trỏ sẽ hoạt động bằng cách tạo ra các mệnh đề "where" để so sánh các giá trị của các cột có trong truy vấn, mang lại hiệu suất cơ sở dữ liệu hiệu quả nhất trong số tất cả các các phương thức phân trang của Laravel. Phương thức phân trang này đặc biệt phù hợp với các tập dữ liệu lớn và giao diện người dùng có scroll "vô hạn".

Không giống như phân trang dựa trên offset, sẽ chứa trang số bao nhiêu trong chuỗi truy vấn của các URL do phân trang tạo ra, phân trang từ con trỏ sẽ lưu một chuỗi "con trỏ" vào chuỗi truy vấn. "Con trỏ" là một chuỗi được mã hóa chứa vị trí tiếp theo sẽ bắt đầu phân trang và hướng mà nó sẽ phân trang:

```nothing
http://localhost/users?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

Bạn có thể tạo một instance phân trang từ con trỏ thông qua phương thức `cursorPaginate` do query builder cung cấp. Phương thức này sẽ trả về một instance của `Illuminate\Pagination\CursorPaginator`:

    $users = DB::table('users')->orderBy('id')->cursorPaginate(15);

Khi bạn đã lấy ra được một instance phân trang từ con trỏ, bạn có thể [hiển thị kết quả phân trang](#displaying-pagination-results) như bạn thường làm khi sử dụng các phương thức `paginate` và `simplePaginate`. Để biết thêm thông tin về các phương thức instance do phân trang từ con trỏ cung cấp, vui lòng tham khảo [tài liệu về phương thức instance phân trang từ con trỏ](#cursor-paginator-instance-methods).

> {note} Truy vấn của bạn phải chứa lệnh "order by" để tận dụng khả năng phân trang bằng con trỏ.

<a name="cursor-vs-offset-pagination"></a>
#### Cursor vs. Offset Pagination

Để minh họa sự khác biệt giữa phân trang offset và phân trang con trỏ, hãy xem xét một số truy vấn SQL mẫu. Cả hai truy vấn sau đây đều sẽ hiển thị "trang thứ hai" của kết quả cho bảng `users` được sắp xếp theo `id`:

```sql
# Offset Pagination...
select * from users order by id asc limit 15 offset 15;

# Cursor Pagination...
select * from users where id > 15 order by id asc limit 15;
```

Truy vấn phân trang con trỏ cung cấp các ưu điểm sau so với phân trang offset:

- Đối với các tập dữ liệu lớn, việc phân trang con trỏ sẽ mang lại hiệu suất tốt hơn nếu các cột "order by" được lập index. Điều này là do lệnh "offset" sẽ quét qua tất cả dữ liệu khớp trước đó.
- Đối với các tập dữ liệu mà bị ghi thường xuyên, phân trang offset có thể cập nhật thiếu bản ghi.

Tuy nhiên, phân trang con trỏ cũng có những hạn chế sau:

- Giống như `simplePaginate`, phân trang con trỏ chỉ có thể được sử dụng để hiển thị các link "Next" và "Previous" và không hỗ trợ tạo link với một số trang bất kỳ.
- Nó yêu cầu order by phải dựa trên ít nhất một cột unique hoặc sự kết hợp của các cột unique. Các cột có giá trị `null` sẽ không được hỗ trợ.
- Biểu thức truy vấn trong lệnh "order by" chỉ được hỗ trợ nếu chúng được đặt alias và được thêm vào lệnh "select".

<a name="manually-creating-a-paginator"></a>
### Tự tạo một phân trang

Thỉnh thoảng bạn có thể muốn tự tạo một instance phân trang riêng và truyền cho nó một mảng các dữ liệu mà bạn đã có trong bộ nhớ. Bạn có thể làm như vậy bằng cách tạo một instance `Illuminate\Pagination\Paginator`, `Illuminate\Pagination\LengthAwarePaginator` hoặc `Illuminate\Pagination\CursorPaginator` tùy theo nhu cầu của bạn.

Class `Paginator` và class `CursorPaginator` sẽ không cần biết tổng số dữ liệu có trong tập kết quả; tuy nhiên, vì điều này, mà các class này cũng sẽ không có các phương thức để lấy ra các index của trang cuối cùng. `LengthAwarePaginator` chấp nhận các tham số gần như tương tự `Paginator`; tuy nhiên, nó yêu cầu tổng số dữ liệu có trong tập kết quả.

Nói cách khác, `Paginator` tương ứng với phương thức `simplePaginate` trong query builder, `CursorPaginator` tương ứng với phương thức `cursorPaginate`, và `LengthAwarePaginator` tương ứng với phương thức `paginate`.

> {note} Khi tự tạo trình phân trang, bạn nên tự "phân chia" các phần tử có trong mảng kết quả mà bạn truyền nó cho trình phân trang. Nếu bạn không chắc chắn cách thực hiện việc này, hãy xem hàm [array_slice](https://secure.php.net/manual/en/function.array-slice.php).

<a name="customizing-pagination-urls"></a>
### Tuỳ biến Pagination URLs

Mặc định, các link do phân trang được tạo ra sẽ giống với URI của request hiện tại. Tuy nhiên, phương thức `withPath` của phân trang cho phép bạn tùy chỉnh các URI mà được trình phân trang sử dụng khi tạo link. Ví dụ: nếu bạn muốn trình phân trang tạo các link như `http://example.com/admin/users?page=N`, thì bạn nên truyền `/admin/users` cho phương thức `withPath`:

    use App\Models\User;

    Route::get('/users', function () {
        $users = User::paginate(15);

        $users->withPath('/admin/users');

        //
    });

<a name="appending-query-string-values"></a>
#### Appending Query String Values

Bạn có thể nối thêm các tham số vào các link phân trang bằng phương thức `appends`. Ví dụ: để nối `sort=votes` vào các link phân trang, bạn có thể thực hiện gọi đến phương thức `appends` như sau:

    use App\Models\User;

    Route::get('/users', function () {
        $users = User::paginate(15);

        $users->appends(['sort' => 'votes']);

        //
    });

Nếu bạn muốn nối tất cả các giá trị query string hiện tại vào các link phân trang, bạn có thể sử dụng phương thức `withQueryString`:

    $users = User::paginate(15)->withQueryString();

<a name="appending-hash-fragments"></a>
#### Appending Hash Fragments

Nếu bạn muốn nối thêm một "hash fragment" vào các URL của trình phân trang, bạn có thể sử dụng phương thức `fragment`. Ví dụ: để nối `#users` vào cuối của mỗi link phân trang, hãy thực hiện gọi đến phương thức `fragment` như sau:

    $users = User::paginate(15)->fragment('users');

<a name="displaying-pagination-results"></a>
## Displaying Pagination Results

Khi gọi phương thức `paginate`, bạn sẽ nhận được một instance của `Illuminate\Pagination\LengthAwarePaginator`, trong khi gọi phương thức `simplePaginate` sẽ trả về một instance của `Illuminate\Pagination\Paginator`. Và cuối cùng, gọi phương thức `cursorPaginate` sẽ trả về một instance của `Illuminate\Pagination\CursorPaginator`.

Các đối tượng này cung cấp một số phương thức để hiển thị kết quả. Ngoài các phương thức helper này, các instance của phân trang còn là các vòng lặp và có thể lặp như một mảng. Vì vậy, sau khi lấy ra được kết quả, bạn có thể hiển thị kết quả và hiển thị các link của trang bằng cách sử dụng [Blade](/docs/{{version}}/blade):

```html
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```

Phương thức `links` sẽ hiển thị các link đến các trang còn lại trongs kết quả. Mỗi link này sẽ chứa sẵn một biến `page` thích hợp. Hãy nhớ rằng HTML được tạo bằng phương thức `links` sẽ tương thích với [framework Tailwind CSS](https://tailwindcss.com).

<a name="adjusting-the-pagination-link-window"></a>
### Điều chỉnh Pagination Link Window

Khi trình phân trang hiển thị các link phân trang, số trang hiện tại cũng được hiển thị cùng với các link cho ba trang trước và sau của trang hiện tại. Sử dụng phương thức `onEachSide`, bạn có thể kiểm soát được số lượng link sẽ được hiển thị cho mỗi bên của trang hiện tại:

    {{ $users->onEachSide(5)->links() }}

<a name="converting-results-to-json"></a>
### Chuyển kết quả thành JSON

Các class phân trang của Laravel sẽ được implement một contract Interface `Illuminate\Contracts\Support\Jsonable` và có sẵn phương thức `toJson`, do đó rất dễ để chuyển đổi kết quả phân trang của bạn sang dạng JSON. Bạn cũng có thể chuyển đổi một instance phân trang thành JSON bằng cách trả nó về từ một action của một route hoặc một controller:

    use App\Models\User;

    Route::get('/users', function () {
        return User::paginate();
    });

JSON từ trình phân trang sẽ chứa thông tin meta như `total`, `current_page`, `last_page`, và hơn thế nữa. Các record kết quả sẽ nằm trong key `data` trong mảng JSON. Dưới đây là một ví dụ về JSON được tạo ra bằng cách trả về instance paginator từ một route:

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "first_page_url": "http://laravel.app?page=1",
       "last_page_url": "http://laravel.app?page=4",
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "path": "http://laravel.app",
       "from": 1,
       "to": 15,
       "data":[
            {
                // Record...
            },
            {
                // Record...
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## Tuỳ biến View của phân trang

Mặc định, các view hiển thị các link phân trang tương thích với [Tailwind CSS](https://tailwindcss.com) framework. Tuy nhiên, nếu bạn không sử dụng Tailwind, bạn có thể thoải mái tự định nghĩa các view của riêng bạn để hiển thị các link. Khi gọi phương thức `links` trên một instance phân trang, bạn có thể truyền tên view làm tham số đầu tiên cho phương thức:

    {{ $paginator->links('view.name') }}

    // Passing additional data to the view...
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}

Tuy nhiên, cách dễ nhất để tùy biến các view của phân trang là bằng cách export chúng vào thư mục `resources/views/vendor` của bạn thông qua cách sử dụng lệnh `vendor:publish`:

    php artisan vendor:publish --tag=laravel-pagination

Lệnh này sẽ lưu các view vào trong thư mục `resources/views/vendor/pasgination` của application của bạn. File `tailwind.blade.php` trong thư mục này tương ứng với view mặc định của phân trang. Bạn có thể sửa file này sẽ sửa đổi HTML của phân trang.

Nếu bạn muốn chỉ định một file khác làm pagination view mặc định, bạn có thể gọi phương thức `defaultView` và `defaultSimpleView` của paginator trong phương thức `boot` của class `App\Providers\AppServiceProvider` của bạn:

    <?php

    namespace App\Providers;

    use Illuminate\Pagination\Paginator;
    use Illuminate\Support\Facades\Blade;
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
            Paginator::defaultView('view-name');

            Paginator::defaultSimpleView('view-name');
        }
    }

<a name="using-bootstrap"></a>
### Dùng Bootstrap

Laravel có chứa các view phân trang được xây dựng bằng [Bootstrap CSS](https://getbootstrap.com/). Để sử dụng các view này thay vì các view Tailwind mặc định, bạn có thể gọi phương thức `useBootstrap` của paginator trong phương thức `boot` của class `App\Providers\AppServiceProvider` của bạn:

    use Illuminate\Pagination\Paginator;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Paginator::useBootstrap();
    }

<a name="paginator-instance-methods"></a>
## Paginator / LengthAwarePaginator Instance Methods

Mỗi instance phân trang cung cấp thêm các thông tin phân trang thông qua các phương thức có sẵn sau:

Method  |  Description
-------  |  -----------
`$paginator->count()`  |  Lấy số lượng các item cho trang hiện tại.
`$paginator->currentPage()`  |  Lấy page number trong trang hiện tại.
`$paginator->firstItem()`  |  Lấy số lượng kết quả của item đầu tiên trong kết quả.
`$paginator->getOptions()`  |  Lấy các tùy chọn paginator.
`$paginator->getUrlRange($start, $end)`  |  Tạo một loạt các URL phân trang.
`$paginator->hasPages()`  |  Kiểm tra xem có đủ item để chia thành nhiều trang hay không.
`$paginator->hasMorePages()`  |  Kiểm tra xem có nhiều item hơn trong data store hay không.
`$paginator->items()`  |  Lấy các item cho trang hiện tại.
`$paginator->lastItem()`  |  Lấy số lượng kết quả của item cuối cùng trong kết quả.
`$paginator->lastPage()`  |  Lấy page number của trang cuối cùng có sẵn. (Không khả dụng khi sử dụng `simplePaginate`).
`$paginator->nextPageUrl()`  |  Lấy URL cho trang tiếp theo.
`$paginator->onFirstPage()`  |  Kiểm tra xem paginator có đang ở trang đầu tiên hay không.
`$paginator->perPage()`  |  Số lượng item được hiển thị trên mỗi trang.
`$paginator->previousPageUrl()`  |  Lấy URL cho trang trước đó.
`$paginator->total()`  |  Kiểm tra tổng số item phù hợp trong data store. (Không khả dụng khi sử dụng `simplePaginate`).
`$paginator->url($page)`  |  Lấy URL cho một trang nhất định.
`$paginator->getPageName()`  |  Lấy biến query string được sử dụng để lưu trữ trang.
`$paginator->setPageName($name)`  |  Set biến query string được sử dụng để lưu trữ trang.

<a name="cursor-paginator-instance-methods"></a>
## Cursor Paginator Instance Methods

Mỗi instance phân trang con trỏ cung cấp thêm các thông tin phân trang thông qua các phương thức có sẵn sau:

Method  |  Description
-------  |  -----------
`$paginator->count()`  |  Lấy số lượng các item cho trang hiện tại.
`$paginator->cursor()`  |  Lấy instance con trỏ hiện tại.
`$paginator->getOptions()`  |  Lấy các tùy chọn paginator.
`$paginator->hasPages()`  |  Kiểm tra xem có đủ item để chia thành nhiều trang hay không.
`$paginator->hasMorePages()`  |  Kiểm tra xem có nhiều item hơn trong data store hay không.
`$paginator->getCursorName()`  |  Lấy biến truy vấn được sử dụng để lưu trữ con trỏ.
`$paginator->items()`  |  Lấy các item cho trang hiện tại.
`$paginator->nextCursor()`  |  Lấy instance con trỏ cho set item tiếp theo.
`$paginator->nextPageUrl()`  |  Lấy URL cho trang tiếp theo.
`$paginator->onFirstPage()`  |  Kiểm tra xem paginator có đang ở trang đầu tiên hay không.
`$paginator->perPage()`  |  Số lượng item được hiển thị trên mỗi trang.
`$paginator->previousCursor()`  |  Lấy instance con trỏ cho set item trước đó.
`$paginator->previousPageUrl()`  |  Lấy URL cho trang trước đó.
`$paginator->setCursorName()`  |  Set biến truy vấn sẽ được sử dụng để lưu trữ con trỏ.
`$paginator->url($cursor)`  |  Lấy URL cho một instance con trỏ nhất định.
