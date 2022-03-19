# Database: Pagination

- [Giới thiệu](#introduction)
- [Cách dùng cơ bản](#basic-usage)
    - [Phân trang từ một query builder](#paginating-query-builder-results)
    - [Phân trang từ Eloquent](#paginating-eloquent-results)
    - [Tự tạo một phân trang](#manually-creating-a-paginator)
- [Hiển thị kết quả phân trang](#displaying-pagination-results)
    - [Chuyển kết quả thành JSON](#converting-results-to-json)
- [Tuỳ biến View của phân trang](#customizing-the-pagination-view)
- [Các phương thức có sẵn](#paginator-instance-methods)

<a name="introduction"></a>
## Giới thiệu

Trong các framework khác, phân trang có thể rất khổ. Mặc định, trình phân trang của Laravel được tích hợp sẵn với [query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent), cung cấp một cách thuận tiện, dễ sử dụng để phân trang kết quả từ cơ sở dữ liệu. HTML được tạo ra bởi trình phân trang tương thích với [Bootstrap CSS framework](https://getbootstrap.com/).

<a name="basic-usage"></a>
## Cách dùng cơ bản

<a name="paginating-query-builder-results"></a>
### Phân trang từ một query builder

Có một số cách để phân trang. Đơn giản nhất là sử dụng phương thức `paginate` trong một [query builder](/docs/{{version}}/queries) hoặc một [Eloquent query](/docs/{{version}}/eloquent). Phương thức `paginate` sẽ tự động đảm nhiệm việc set giới hạn và offset dựa trên trang hiện tại đang được người dùng xem. Mặc định, trang hiện tại sẽ được dò tìm giá trị của tham số `page` trong HTTP request. Giá trị này sẽ được tự động dò tìm bởi Laravel và cũng được tự động thêm vào sau các link sau quá trình phân trang.

Trong ví dụ này, chỉ có một tham số duy nhất được truyền vào phương thức `paginate` đó là số lượng dữ liệu mà bạn muốn hiển thị "trên mỗi trang". Trong trường hợp này, hãy khai báo chúng ta muốn hiển thị `15` dữ liệu trên mỗi trang:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show all of the users for the application.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} Hiện tại, các hành động phân trang mà có sử dụng câu lệnh `groupBy` thì không thể thực hiện được bởi Laravel. Nếu bạn cần sử dụng `groupBy` với một tập hợp kết quả đã được phân trang, bạn nên truy vấn vào cơ sở dữ liệu và tự tạo một trình phân trang riêng biệt dành cho bạn.

#### "Simple Pagination"

Nếu bạn chỉ cần hiển thị các link đơn giản ví dụ như "Next" và "Previous" trong view của bạn, bạn có thể sử dụng phương thức `simplePaginate` để thực hiện query hiệu quả hơn. Điều này rất hữu ích cho các bộ dữ liệu lớn khi mà bạn không cần hiển thị toàn bộ các link khi hiển thị view của bạn:

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### Phân trang từ Eloquent

Bạn cũng có thể phân trang bằng các truy vấn [Eloquent](/docs/{{version}}/eloquent). Trong ví dụ này, chúng ta sẽ phân trang model `User` với `15` dữ liệu trên mỗi trang. Như bạn có thể thấy, cú pháp này gần giống với phân trang bằng query builder:

    $users = App\User::paginate(15);

Bạn có thể gọi `paginate` sau khi set các điều kiện cho truy vấn, chẳng hạn như câu lệnh `where`:

    $users = User::where('votes', '>', 100)->paginate(15);

Bạn cũng có thể sử dụng phương thức `simplePaginate` khi phân trang trên các model Eloquent:

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### Tự tạo một phân trang

Thỉnh thoảng bạn có thể muốn tự tạo một instance phân trang riêng và truyền cho nó một mảng các dữ liệu. Bạn có thể làm như vậy bằng cách tạo một instance `Illuminate\Pagination\Paginator` hoặc `Illuminate\Pagination\LengthAwarePaginator`, tùy theo nhu cầu của bạn.

Class `Paginator` không cần biết tổng số dữ liệu có trong tập kết quả; tuy nhiên, vì điều này, mà class cũng sẽ không có các phương thức để lấy ra các index của trang cuối cùng. `LengthAwarePaginator` chấp nhận các tham số gần như tương tự `Paginator`; tuy nhiên, nó yêu cầu tổng số dữ liệu có trong tập kết quả.

Nói cách khác, `Paginator` tương ứng với phương thức `simplePaginate` trong query builder và Eloquent, trong khi `LengthAwarePaginator` tương ứng với phương thức `paginate`.

> {note} Khi tự tạo trình phân trang, bạn nên tự "phân chia" các phần tử có trong mảng kết quả mà bạn truyền nó cho trình phân trang. Nếu bạn không chắc chắn cách thực hiện việc này, hãy xem hàm [array_slice](https://secure.php.net/manual/en/function.array-slice.php).

<a name="displaying-pagination-results"></a>
## Hiển thị kết quả phân trang

Khi gọi phương thức `paginate`, bạn sẽ nhận được một instance của `Illuminate\Pagination\LengthAwarePaginator`. Khi gọi phương thức `simplePaginate`, bạn sẽ nhận được một instance của `Illuminate\Pagination\Paginator`. Các đối tượng này cung cấp một số phương thức mô tả cho tập kết quả. Ngoài các phương thức hỗ trợ này, các instances phân trang cũng là các vòng lặp và có thể được lặp như là một mảng. Vì vậy, khi bạn đã lấy được kết quả, thì bạn có thể hiển thị kết quả và hiển thị các link của các trang bằng [Blade](/docs/{{version}}/blade):

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

Phương thức `links` sẽ hiển thị các link dẫn đến các trang còn lại có trong tập kết quả. Mỗi link này sẽ chứa một biến `page` phù hợp. Hãy nhớ rằng, HTML được tạo bởi phương thức `links` tương thích với [Bootstrap CSS framework](https://getbootstrap.com).

#### Customizing The Paginator URI

Phương thức `withPath` cho phép bạn tùy biến URI cho trình phân trang khi tạo link. Ví dụ: nếu bạn muốn trình phân trang tạo ra các link như `http://example.com/custom/url?page=N`, bạn hãy truyền `custom/url` này cho phương thức `withPath`:

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->withPath('custom/url');

        //
    });

#### Appending To Pagination Links

Bạn có thể nối thêm các tham số vào các link phân trang bằng phương thức `appends`. Ví dụ: để nối `sort=votes` vào các link phân trang, bạn có thể thực hiện gọi đến phương thức `appends` như sau:

    {{ $users->appends(['sort' => 'votes'])->links() }}

Nếu bạn muốn nối thêm một "hash fragment" vào các URL của trình phân trang, bạn có thể sử dụng phương thức `fragment`. Ví dụ: để nối `#foo` vào cuối của mỗi link phân trang, hãy thực hiện gọi đến phương thức `fragment` như sau:

    {{ $users->fragment('foo')->links() }}

#### Adjusting The Pagination Link Window

Bạn có thể kiểm soát số lượng link được hiển thị ở mỗi bên của "window" URL của paginator. Mặc định, ba link sẽ được hiển thị ở mỗi bên của các link paginator chính. Tuy nhiên, bạn có thể kiểm soát số link này bằng phương thức `onEachSide`:

    {{ $users->onEachSide(5)->links() }}

<a name="converting-results-to-json"></a>
### Chuyển kết quả thành JSON

Các class kết quả phân trang của Laravel sẽ được implement một contract Interface `Illuminate\Contracts\Support\Jsonable` và có sẵn phương thức `toJson`, do đó rất dễ để chuyển đổi kết quả phân trang của bạn sang dạng JSON. Bạn cũng có thể chuyển đổi một instance phân trang thành JSON bằng cách trả nó về từ một action của một route hoặc một controller:

    Route::get('users', function () {
        return App\User::paginate();
    });

JSON từ trình phân trang sẽ chứa thông tin meta như `total`, `current_page`, `last_page`, và hơn thế nữa. Các đối tượng kết quả sẽ nằm trong key `data` trong mảng JSON. Dưới đây là một ví dụ về JSON được tạo ra bằng cách trả về instance paginator từ một route:

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
                // Result Object
            },
            {
                // Result Object
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## Tuỳ biến View của phân trang

Mặc định, các view hiển thị các link phân trang tương thích với Bootstrap CSS framework. Tuy nhiên, nếu bạn không sử dụng Bootstrap, bạn có thể thoải mái tự định nghĩa các view của riêng bạn để hiển thị các link. Khi gọi phương thức `links` trên một instance phân trang, hãy truyền tên view làm tham số đầu tiên cho phương thức:

    {{ $paginator->links('view.name') }}

    // Passing data to the view...
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}

Tuy nhiên, cách dễ nhất để tùy biến các view của phân trang là bằng cách export chúng vào thư mục `resources/views/vendor` của bạn thông qua cách sử dụng lệnh `vendor:publish`:

    php artisan vendor:publish --tag=laravel-pagination

Lệnh này sẽ lưu các view vào trong thư mục `resources/views/vendor/pagination`. File `bootstrap-4.blade.php` trong thư mục này tương ứng với view mặc định của phân trang. Bạn có thể sửa file này sẽ sửa đổi HTML của phân trang.

Nếu bạn muốn chỉ định một file khác làm pagination view mặc định, bạn có thể sử dụng phương thức `defaultView` và `defaultSimpleView` của paginator trong `AppServiceProvider` của bạn:

    use Illuminate\Pagination\Paginator;

    public function boot()
    {
        Paginator::defaultView('view-name');

        Paginator::defaultSimpleView('view-name');
    }

<a name="paginator-instance-methods"></a>
## Các phương thức có sẵn

Mỗi instance phân trang cung cấp thêm các thông tin phân trang thông qua các phương thức có sẵn sau:

Method  |  Description
-------  |  -----------
`$results->count()`  |  Lấy số lượng các item cho trang hiện tại.
`$results->currentPage()`  |  Lấy page number trong trang hiện tại.
`$results->firstItem()`  |  Lấy số lượng kết quả của item đầu tiên trong kết quả.
`$results->getOptions()`  |  Lấy các tùy chọn paginator.
`$results->getUrlRange($start, $end)`  |  Tạo một loạt các URL phân trang.
`$results->hasMorePages()`  |  Kiểm tra xem có đủ mục để chia thành nhiều trang hay không.
`$results->lastItem()`  |  Lấy số lượng kết quả của item cuối cùng trong kết quả.
`$results->lastPage()`  |  Lấy page number của trang cuối cùng có sẵn. (Không khả dụng khi sử dụng `simplePaginate`).
`$results->nextPageUrl()`  |  Lấy URL cho trang tiếp theo.
`$results->onFirstPage()`  |  Kiểm tra xem paginator có đang ở trang đầu tiên hay không.
`$results->perPage()`  |  Số lượng item được hiển thị trên mỗi trang.
`$results->previousPageUrl()`  |  Lấy URL cho trang trước đó.
`$results->total()`  |  Kiểm tra tổng số mục phù hợp trong data store. (Không khả dụng khi sử dụng `simplePaginate`).
`$results->url($page)`  |  Lấy URL cho một trang nhất định.
