# Laravel Scout

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Queueing](#queueing)
    - [Driver Prerequisites](#driver-prerequisites)
- [Configuration](#configuration)
    - [Configuring Model Indexes](#configuring-model-indexes)
    - [Configuring Searchable Data](#configuring-searchable-data)
- [Indexing](#indexing)
    - [Batch Import](#batch-import)
    - [Adding Records](#adding-records)
    - [Updating Records](#updating-records)
    - [Removing Records](#removing-records)
    - [Pausing Indexing](#pausing-indexing)
- [Searching](#searching)
    - [Where Clauses](#where-clauses)
    - [Pagination](#pagination)
- [Custom Engines](#custom-engines)

<a name="introduction"></a>
## Giới thiệu

Laravel Scout cung cấp một giải pháp dựa trên driver đơn giản để thêm tìm kiếm full-text vào [các model Eloquent](/docs/{{version}}/eloquent). Sử dụng model observer, Scout sẽ tự động giữ các index tìm kiếm của bạn và đồng bộ với các bản ghi Eloquent của bạn.

Hiện tại, Scout đang làm việc cùng driver [Algolia](https://www.algolia.com/); tuy nhiên, viết driver tùy biến rất đơn giản và bạn có thể tự do mở rộng Scout với việc triển khai tìm kiếm của riêng bạn.

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt Scout thông qua package manager Composer:

    composer require laravel/scout

Sau khi cài đặt Scout, bạn nên export cấu hình của Scout bằng lệnh Artisan `vendor:publish`. Lệnh này sẽ export file cấu hình `scout.php` vào thư mục `config` của bạn:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

Cuối cùng, thêm trait `Laravel\Scout\Searchable` vào model mà bạn muốn thêm chức năng tìm kiếm. Trait này sẽ đăng ký một model observer để giữ cho model được đồng bộ với driver tìm kiếm của bạn:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### Queueing

Mặc dù không bắt buộc phải sử dụng Scout, nhưng bạn nên cân nhắc cấu hình một [queue driver](/docs/{{version}}/queues) trước khi sử dụng thư viện. Chạy một queue worker sẽ cho phép Scout sẽ queue tất cả các hoạt động đồng bộ thông tin model của bạn với các index tìm kiếm của bạn, cung cấp thời gian phản hồi tốt hơn nhiều cho giao diện web của ứng dụng.

Khi bạn đã cấu hình xong một queue driver, hãy set giá trị của tùy chọn `queue` trong file cấu hình `config/scout.php` của bạn là `true`:

    'queue' => true,

<a name="driver-prerequisites"></a>
### Driver Prerequisites

#### Algolia

Khi sử dụng driver Algolia, bạn nên cấu hình thông tin đăng nhập `id` và `secret` trong file cấu hình `config/scout.php` của bạn. Khi mà thông tin đăng nhập của bạn đã được cấu hình xong, bạn cũng sẽ cần cài đặt SDK PHP Algolia thông qua package manager Composer:

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## Configuration

<a name="configuring-model-indexes"></a>
### Configuring Model Indexes

Mỗi một model Eloquent được đồng bộ với một "index" tìm kiếm nhất định, nó sẽ chứa tất cả các bản ghi có thể được tìm kiếm cho model đó. Nói cách khác, bạn có thể nghĩ về mỗi index giống như một bảng trong MySQL. Mặc định, mỗi model sẽ được lưu trữ theo một index khớp với tên "bảng" của model. Thông thường, là dạng số nhiều của tên model; tuy nhiên, bạn có thể tùy biến index của model bằng cách ghi đè phương thức `searchableAs` trên model:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the index name for the model.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### Configuring Searchable Data

Mặc định, toàn bộ form `toArray` của một model sẽ được lưu trữ theo index tìm kiếm của nó. Nếu bạn muốn tùy biến dữ liệu được đồng bộ với index tìm kiếm, bạn có thể ghi đè phương thức `toSearchableArray` trên model:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the indexable data array for the model.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize array...

            return $array;
        }
    }

<a name="indexing"></a>
## Indexing

<a name="batch-import"></a>
### Batch Import

Nếu bạn đang cài đặt Scout vào một project đã tồn tại, bạn có thể đã có sẵn các bản ghi cơ sở dữ liệu mà bạn cần import vào driver tìm kiếm của bạn. Scout cung cấp lệnh Artisan `import` mà bạn có thể sử dụng để import tất cả các bản ghi hiện có vào các index tìm kiếm của bạn:

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### Adding Records

Khi mà bạn đã thêm trait `Laravel\Scout\Searchable` vào một model, tất cả những gì bạn cần làm là `save` một instance model và nó sẽ tự động được thêm vào index tìm kiếm của bạn. Nếu bạn đã cấu hình Scout để [sử dụng queue](#queueing) thì thao tác này sẽ được thực hiện dưới background bởi queue worker của bạn:

    $order = new App\Order;

    // ...

    $order->save();

#### Adding Via Query

Nếu bạn muốn thêm một collection các model vào index tìm kiếm của bạn thông qua truy vấn của Eloquent, bạn có thể kết hợp thêm phương thức `searchable` vào truy vấn của Eloquent. Phương thức `searchable` sẽ [chunk các kết quả](/docs/{{version}}/eloquent#chunking-results) của truy vấn và thêm các bản ghi vào index tìm kiếm của bạn. Một lần nữa, nếu bạn đã cấu hình Scout để sử dụng queue, tất cả các chunk sẽ được thêm vào background bởi các queue worker của bạn:

    // Adding via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also add records via relationships...
    $user->orders()->searchable();

    // You may also add records via collections...
    $orders->searchable();

Phương thức `searchable` có thể được coi là một hoạt động "updateOrCreate". Nói cách khác, nếu bản ghi model đã có trong index của bạn, nó sẽ được cập nhật. Nếu nó không tồn tại trong index tìm kiếm, nó sẽ được thêm vào index.

<a name="updating-records"></a>
### Updating Records

Để cập nhật một model mà có thể tìm kiếm, bạn chỉ cần cập nhật các thuộc tính của instance model đó và `save` model vào cơ sở dữ liệu của bạn. Scout sẽ tự động lưu các thay đổi đối với index tìm kiếm của bạn:

    $order = App\Order::find(1);

    // Update the order...

    $order->save();

Bạn cũng có thể sử dụng phương thức `searchable` trên truy vấn của Eloquent để cập nhật một collection của model. Nếu model không tồn tại trong index tìm kiếm của bạn, chúng sẽ được tạo:

    // Updating via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also update via relationships...
    $user->orders()->searchable();

    // You may also update via collections...
    $orders->searchable();

<a name="removing-records"></a>
### Removing Records

Để xóa một bản ghi ra khỏi index của bạn, hãy xóa model ra khỏi cơ sở dữ liệu. Hình thức xoá này thậm chí tương thích với các model đang dùng [soft deleted](/docs/{{version}}/eloquent#soft-deleting):

    $order = App\Order::find(1);

    $order->delete();

Nếu bạn không muốn lấy ra model trước khi xóa nó, bạn có thể sử dụng phương thức `unsearchable` trên một instance hoặc collection của truy vấn Eloquent:

    // Removing via Eloquent query...
    App\Order::where('price', '>', 100)->unsearchable();

    // You may also remove via relationships...
    $user->orders()->unsearchable();

    // You may also remove via collections...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### Pausing Indexing

Thỉnh thoảng bạn có thể cần thực hiện một loạt các hoạt động Eloquent trên một model mà không đồng bộ dữ liệu của model với index tìm kiếm của bạn. Bạn có thể làm điều này bằng cách sử dụng phương thức `withoutSyncingToSearch`. Phương thức này chấp nhận một callback sẽ được thực hiện ngay lập tức. Bất kỳ hoạt động model nào xảy ra trong callback sẽ không được đồng bộ với index của model:

    App\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });

<a name="searching"></a>
## Searching

Bạn có thể bắt đầu tìm kiếm một model bằng phương thức `search`. Phương thức search chấp nhận một chuỗi string để tìm kiếm model của bạn. Sau đó, bạn nên kết hợp thêm phương thức `get` vào truy vấn tìm kiếm để lấy ra các model Eloquent phù hợp với truy vấn tìm kiếm đã cho:

    $orders = App\Order::search('Star Trek')->get();

Vì các tìm kiếm Scout trả về một collection của model Eloquent, nên bạn thậm chí có thể trả về kết quả trực tiếp từ một route hoặc controller và chúng sẽ tự động được chuyển đổi thành JSON:

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

Nếu bạn muốn nhận kết quả raw trước khi chúng được chuyển đổi thành các model Eloquent, bạn nên sử dụng phương thức `raw`:

    $orders = App\Order::search('Star Trek')->raw();

Các truy vấn tìm kiếm thường sẽ được thực hiện trên index được chỉ định bởi phương thức [`searchableAs`](#configuring-model-indexes) của model. Tuy nhiên, bạn có thể sử dụng phương thức `within` để chỉ định một index tùy biến sẽ được tìm kiếm thay thế:

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Where Clauses

Scout cho phép bạn thêm các điều kiện "where" vào các truy vấn tìm kiếm của bạn. Hiện tại, các điều kiện này chỉ hỗ trợ so sánh số cơ bản và chủ yếu sử dụng cho việc query tìm kiếm theo ID đối tượng. Vì các index tìm kiếm không phải là cơ sở dữ liệu quan hệ với nhau, nên các điều kiện "where" nâng cao hơn sẽ chưa không được hỗ trợ:

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### Pagination

Ngoài việc lấy ra một collection của model, bạn có thể phân trang kết quả tìm kiếm được của bạn bằng phương thức `paginate`. Phương thức này sẽ trả về một instance `Paginator` giống như bạn đã đọc [tài liệu phân trang truy vấn Eloquent](/docs/{{version}}/pagination) ở các phần trước:

    $orders = App\Order::search('Star Trek')->paginate();

Bạn có thể chỉ định số lượng model được trả về trên mỗi trang bằng cách pass số lượng đó làm tham số đầu tiên cho phương thức `paginate`:

    $orders = App\Order::search('Star Trek')->paginate(15);

Khi bạn đã lấy ra được kết quả, bạn có thể hiển thị kết quả và tạo ra các page links bằng [Blade](/docs/{{version}}/blade) giống như khi bạn đã phân trang truy vấn Eloquent:

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="custom-engines"></a>
## Custom Engines

#### Writing The Engine

Nếu một trong những engine tìm kiếm của Scout không phù hợp với nhu cầu của bạn, bạn có thể viết engine tùy biến của riêng bạn và đăng ký với Scout. Engine của bạn sẽ được extend từ abstract class `Laravel\Scout\Engines\Engine`. Abstract class này chứa bảy phương thức mà engine tùy biến của bạn phải implement:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map($results, $model);
    abstract public function getTotalCount($results);

Bạn có thể tham khảo class `Laravel\Scout\Engines\AlgoliaEngine` để biết bạn nên implement các phương thức đó như thế nào. Class này sẽ cung cấp cho bạn một điểm khởi đầu tốt để học cách implement từng phương thức này trong engine của riêng bạn.

#### Registering The Engine

Khi bạn đã viết xong engine tùy biến của bạn, bạn có thể đăng ký nó với Scout bằng phương thức `extend` của engine manager của Scout. Bạn nên gọi phương thức `extend` từ phương thức` boot` của `AppServiceProvider` hoặc bất kỳ service provider nào khác được application của bạn sử dụng. Ví dụ: nếu bạn đã viết một `MySqlSearchEngine`, bạn có thể đăng ký nó như sau:

    use Laravel\Scout\EngineManager;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

Khi engine của bạn đã được đăng ký, bạn có thể khai báo nó làm Scout `driver` mặc định trong file cấu hình `config/scout.php` của bạn:

    'driver' => 'mysql',
