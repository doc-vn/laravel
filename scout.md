# Laravel Scout

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Queueing](#queueing)
    - [Driver Prerequisites](#driver-prerequisites)
- [Configuration](#configuration)
    - [Configuring Model Indexes](#configuring-model-indexes)
    - [Configuring Searchable Data](#configuring-searchable-data)
    - [Configuring The Model ID](#configuring-the-model-id)
    - [Identifying Users](#identifying-users)
- [Indexing](#indexing)
    - [Batch Import](#batch-import)
    - [Adding Records](#adding-records)
    - [Updating Records](#updating-records)
    - [Removing Records](#removing-records)
    - [Pausing Indexing](#pausing-indexing)
    - [Điều kiện Model Searchable](#conditionally-searchable-model-instances)
- [Searching](#searching)
    - [Where Clauses](#where-clauses)
    - [Pagination](#pagination)
    - [Soft Deleting](#soft-deleting)
    - [Tuỳ chỉnh Engine Search](#customizing-engine-searches)
- [Custom Engines](#custom-engines)
- [Builder Macros](#builder-macros)

<a name="introduction"></a>
## Giới thiệu

Laravel Scout cung cấp một giải pháp dựa trên driver đơn giản để thêm chức năng tìm kiếm full-text vào [các model Eloquent](/docs/{{version}}/eloquent). Sử dụng model observer, Scout sẽ tự động giữ các index tìm kiếm và đồng bộ nó với các bản ghi trong Eloquent của bạn.

Hiện tại, Scout đang làm việc cùng driver [Algolia](https://www.algolia.com/); tuy nhiên, nếu viết một driver tùy biến mới, thì cũng rất đơn giản, bạn có thể tự do mở rộng Scout với việc tạo một tìm kiếm của riêng bạn.

<a name="installation"></a>
## Cài đặt

Đầu tiên, hãy cài đặt Scout thông qua package manager Composer:

    composer require laravel/scout:^8.0

Sau khi cài đặt Scout xong, bạn nên export cấu hình của Scout bằng lệnh Artisan `vendor:publish`. Lệnh này sẽ export file cấu hình `scout.php` vào thư mục `config` của bạn:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

Cuối cùng, thêm trait `Laravel\Scout\Searchable` vào model mà bạn muốn thêm chức năng tìm kiếm. Trait này sẽ đăng ký một model observer để giữ cho các model được đồng bộ với driver tìm kiếm của bạn:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### Queueing

Mặc dù không bắt buộc phải sử dụng Scout, nhưng bạn nên cân nhắc cấu hình một [queue driver](/docs/{{version}}/queues) trước khi sử dụng thư viện. Chạy một queue worker sẽ cho phép Scout sẽ queue tất cả các hoạt động đồng bộ thông tin model của bạn với các index tìm kiếm của bạn lại, cung cấp thời gian phản hồi tốt hơn cho giao diện ứng dụng web của bạn.

Khi bạn đã cấu hình xong queue driver, hãy set giá trị của tùy chọn `queue` trong file cấu hình `config/scout.php` của bạn là `true`:

    'queue' => true,

<a name="driver-prerequisites"></a>
### Driver Prerequisites

#### Algolia

Khi sử dụng driver Algolia, bạn nên cấu hình thông tin đăng nhập `id` và `secret` trong file cấu hình `config/scout.php` của bạn. Khi thông tin đăng nhập của bạn đã được cấu hình xong, bạn cũng sẽ cần cài đặt thêm SDK PHP Algolia thông qua package manager Composer:

    composer require algolia/algoliasearch-client-php:^2.2

<a name="configuration"></a>
## Configuration

<a name="configuring-model-indexes"></a>
### Configuring Model Indexes

Mỗi một model Eloquent được đồng bộ với một "index" tìm kiếm nhất định, nó sẽ chứa tất cả các bản ghi có thể được tìm kiếm cho model đó. Nói cách khác, bạn có thể nghĩ mỗi index giống như là một bảng trong MySQL. Mặc định, mỗi model sẽ được lưu trữ theo một index khớp với tên "bảng" của model. Thông thường, là dạng số nhiều của tên model; tuy nhiên, bạn có thể tùy biến index của model bằng cách ghi đè phương thức `searchableAs` trên model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

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

Mặc định, toàn bộ form `toArray` của một model sẽ được lưu theo index tìm kiếm của nó. Nếu bạn muốn tùy biến dữ liệu được đồng bộ với index tìm kiếm, bạn có thể ghi đè phương thức `toSearchableArray` trên model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

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

<a name="configuring-the-model-id"></a>
### Configuring The Model ID

Mặc định, Scout sẽ sử dụng khóa chính của model làm ID được lưu trữ trong search index. Nếu bạn cần tùy chỉnh hành vi này, bạn có thể ghi đè phương thức `getScoutKey` và phương thức `getScoutKeyName` trên model đó:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * Get the value used to index the model.
         *
         * @return mixed
         */
        public function getScoutKey()
        {
            return $this->email;
        }

        /**
         * Get the key name used to index the model.
         *
         * @return mixed
         */
        public function getScoutKeyName()
        {
            return 'email';
        }
    }

<a name="identifying-users"></a>
### Identifying Users

Scout cũng cho phép bạn tự động xác định người dùng khi sử dụng Algolia. Việc liên kết người dùng đã authenticate với các thao tác tìm kiếm có thể hữu ích khi xem bảng phân tích tìm kiếm trong bảng điều khiển của Algolia. Bạn có thể bật nhận dạng người dùng bằng cách set `SCOUT_IDENTIFY` thành `true` trong file `.env` của bạn:

    SCOUT_IDENTIFY=true

Bật tính năng này, nó cũng sẽ truyền địa chỉ IP của request và khoá chính của người dùng đã authenticate của bạn tới Algolia để dữ liệu này được liên kết với bất kỳ request tìm kiếm nào được người dùng thực hiện.

<a name="indexing"></a>
## Indexing

<a name="batch-import"></a>
### Batch Import

Nếu bạn đang cài đặt Scout cho một project đã tồn tại, có thể bạn đã có các bản ghi trong cơ sở dữ liệu và bạn cần import nó vào driver tìm kiếm của bạn. Scout cung cấp một lệnh Artisan `import` mà bạn có thể sử dụng để import tất cả các bản ghi hiện có vào các index tìm kiếm của bạn:

    php artisan scout:import "App\Post"

Lệnh `flush` có thể được sử dụng để xóa tất cả các bản ghi của model ra khỏi các search index của bạn:

    php artisan scout:flush "App\Post"

<a name="adding-records"></a>
### Adding Records

Khi mà bạn đã thêm trait `Laravel\Scout\Searchable` vào một model, tất cả những gì bạn cần làm là `save` một instance model và nó sẽ tự động được thêm vào index tìm kiếm cho bạn. Nếu bạn đã cấu hình Scout để [sử dụng queue](#queueing) thì thao tác này sẽ được thực hiện dưới background bởi queue worker của bạn:

    $order = new App\Order;

    // ...

    $order->save();

#### Adding Via Query

Nếu bạn muốn thêm một collection model vào index tìm kiếm của bạn thông qua truy vấn của Eloquent, bạn có thể kết hợp thêm phương thức `searchable` vào truy vấn của Eloquent. Phương thức `searchable` sẽ [chunk các kết quả](/docs/{{version}}/eloquent#chunking-results) của truy vấn và thêm các bản ghi vào index tìm kiếm của bạn. Một lần nữa, nếu bạn đã cấu hình Scout để sử dụng queue, tất cả các chunk sẽ được thêm vào dưới background bởi các queue worker của bạn:

    // Adding via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also add records via relationships...
    $user->orders()->searchable();

    // You may also add records via collections...
    $orders->searchable();

Phương thức `searchable` có thể được coi như là một hành động "updateOrCreate". Nói cách khác, nếu bản ghi model đã có trong index của bạn, nó sẽ được cập nhật. Nếu nó không tồn tại trong index, nó sẽ được thêm vào index.

<a name="updating-records"></a>
### Updating Records

Để cập nhật một model mà có thể tìm kiếm, bạn chỉ cần cập nhật các thuộc tính của instance model và `save` model đó vào cơ sở dữ liệu của bạn. Scout sẽ tự động lưu các thay đổi đối với index tìm kiếm của bạn:

    $order = App\Order::find(1);

    // Update the order...

    $order->save();

Bạn cũng có thể sử dụng phương thức `searchable` trên truy vấn của Eloquent để cập nhật một collection model. Nếu model không tồn tại trong index tìm kiếm của bạn, chúng sẽ được tạo mới:

    // Updating via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also update via relationships...
    $user->orders()->searchable();

    // You may also update via collections...
    $orders->searchable();

<a name="removing-records"></a>
### Removing Records

Để xóa một bản ghi ra khỏi index của bạn, hãy xóa model đó ra khỏi cơ sở dữ liệu. Hình thức xoá này thậm chí tương thích với các model đang dùng [soft deleted](/docs/{{version}}/eloquent#soft-deleting):

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

Thỉnh thoảng bạn có thể cần thực hiện một loạt các hành động Eloquent trên một model mà không muốn đồng bộ dữ liệu của model đó với index tìm kiếm. Bạn có thể làm điều này bằng cách sử dụng phương thức `withoutSyncingToSearch`. Phương thức này sẽ chấp nhận một callback sẽ được thực hiện ngay lập tức. Bất kỳ hành động model nào xảy ra trong callback này đều sẽ không được đồng bộ với index của model:

    App\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });

<a name="conditionally-searchable-model-instances"></a>
### Điều kiện Model Searchable

Thỉnh thoảng bạn có thể muốn tìm kiếm trong model searchable có thêm một số điều kiện nhất định. Ví dụ: hãy tưởng tượng bạn có model `App\Post` có thể ở một trong hai trạng thái: "draft" và "published". Bạn có thể chỉ muốn tìm kiếm các bài đăng đã được "published". Để thực hiện điều này, bạn có thể định nghĩa một phương thức `shouldBeSearchable` trên model của bạn:

    public function shouldBeSearchable()
    {
        return $this->isPublished();
    }

Phương thức `shouldBeSearchable` chỉ được áp dụng khi bạn thao tác với model thông qua phương thức `save` hoặc các câu lệnh truy vấn hoặc các quan hệ. Bạn gọi trực tiếp phương thức `searchable` qua model hoặc qua các collection searchable, thì nó sẽ ghi đè kết quả của phương thức `shouldBeSearchable`:

    // Will respect "shouldBeSearchable"...
    App\Order::where('price', '>', 100)->searchable();

    $user->orders()->searchable();

    $order->save();

    // Will override "shouldBeSearchable"...
    $orders->searchable();

    $order->searchable();

<a name="searching"></a>
## Searching

Bạn có thể bắt đầu tìm kiếm một model bằng phương thức `search`. Phương thức search chấp nhận một chuỗi string để tìm kiếm model của bạn. Sau đó, bạn nên kết hợp thêm phương thức `get` vào truy vấn tìm kiếm để lấy ra các model Eloquent phù hợp với truy vấn tìm kiếm đã cho:

    $orders = App\Order::search('Star Trek')->get();

Vì các tìm kiếm Scout trả về một collection của model Eloquent, nên bạn thậm chí có thể trả về kết quả trực tiếp từ một route hoặc một controller và chúng sẽ tự động được chuyển thành dạng JSON:

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

Nếu bạn muốn nhận kết quả raw trước khi chúng được chuyển đổi thành các model Eloquent, bạn nên sử dụng phương thức `raw`:

    $orders = App\Order::search('Star Trek')->raw();

Các câu lệnh truy vấn tìm kiếm thường sẽ được thực hiện trên index mà được chỉ định bởi phương thức [`searchableAs`](#configuring-model-indexes) của model. Tuy nhiên, bạn có thể sử dụng phương thức `within` để chỉ định một index khác sẽ được tìm kiếm:

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Where Clauses

Scout cho phép bạn thêm các điều kiện "where" vào các truy vấn tìm kiếm của bạn. Hiện tại, các điều kiện này chỉ hỗ trợ so sánh số cơ bản và chủ yếu sử dụng cho việc query tìm kiếm theo ID đối tượng. Vì các index tìm kiếm không phải là cơ sở dữ liệu quan hệ, nên các điều kiện "where" nâng cao hơn sẽ chưa được hỗ trợ:

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### Pagination

Ngoài việc lấy ra một collection của model, bạn có thể phân trang kết quả tìm kiếm của bạn bằng phương thức `paginate`. Phương thức này sẽ trả về một instance `Paginator` giống như bạn đã đọc ở [tài liệu phân trang truy vấn Eloquent](/docs/{{version}}/pagination) trong các phần trước:

    $orders = App\Order::search('Star Trek')->paginate();

Bạn có thể chỉ định số lượng model được trả về trên mỗi trang bằng cách truyền vào số lượng đó làm tham số đầu tiên cho phương thức `paginate`:

    $orders = App\Order::search('Star Trek')->paginate(15);

Khi bạn đã lấy ra được kết quả, bạn có thể hiển thị kết quả và tạo ra các page links bằng [Blade](/docs/{{version}}/blade) giống như khi bạn thực hiện phân trang truy vấn Eloquent:

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="soft-deleting"></a>
### Soft Deleting

Nếu các model index của bạn là loại có thể [soft deleting](/docs/{{version}}/eloquent#soft-deleting) và bạn cần tìm kiếm các model đã bị soft delete của bạn, hãy set tùy chọn `soft_delete` vào trong file cấu hình `config/scout.php` của bạn thành `true`:

    'soft_delete' => true,

Khi tùy chọn cấu hình này thành `true`, Scout sẽ không xóa các model đó ra khỏi search index. Thay vào đó, nó sẽ set thuộc tính ẩn `__soft_deleted` trên bản ghi đó. Và sau đó, bạn có thể sử dụng phương thức `withTrashed` hoặc `onlyTrashed` để lấy ra các bản ghi đã soft delete khi tìm kiếm:

    // Include trashed records when retrieving results...
    $orders = App\Order::search('Star Trek')->withTrashed()->get();

    // Only include trashed records when retrieving results...
    $orders = App\Order::search('Star Trek')->onlyTrashed()->get();

> {tip} Khi một model đã bị xóa vĩnh viễn bằng cách sử dụng `forceDelete`, Scout sẽ tự động xóa model đó ra khỏi search index.

<a name="customizing-engine-searches"></a>
### Tuỳ chỉnh Engine Search

Nếu bạn cần tùy chỉnh hành động tìm kiếm của một engine, bạn có thể truyền một lệnh callback làm tham số thứ hai cho phương thức `search`. Ví dụ: bạn có thể sử dụng lệnh callback này để thêm dữ liệu vị trí vào các tùy chọn tìm kiếm trước khi câu lệnh tìm kiếm được truyền đến Algolia:

    use Algolia\AlgoliaSearch\SearchIndex;

    App\Order::search('Star Trek', function (SearchIndex $algolia, string $query, array $options) {
        $options['body']['query']['bool']['filter']['geo_distance'] = [
            'distance' => '1000km',
            'location' => ['lat' => 36, 'lon' => 111],
        ];

        return $algolia->search($query, $options);
    })->get();

<a name="custom-engines"></a>
## Custom Engines

#### Writing The Engine

Nếu một trong những engine tìm kiếm của Scout không phù hợp với nhu cầu của bạn, bạn có thể viết một engine mới của riêng bạn và đăng ký nó với Scout. Engine của bạn sẽ được extend từ abstract class `Laravel\Scout\Engines\Engine`. Abstract class này chứa tám phương thức mà engine mới của bạn phải implement:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map(Builder $builder, $results, $model);
    abstract public function getTotalCount($results);
    abstract public function flush($model);

Bạn có thể tham khảo class `Laravel\Scout\Engines\AlgoliaEngine` để biết thêm cách bạn nên implement các phương thức đó như thế nào. Class này sẽ cung cấp cho bạn một điểm khởi đầu tốt để bạn có thể học cách implement từng phương thức này trong engine của riêng bạn.

#### Registering The Engine

Khi bạn đã viết xong engine mới của bạn, bạn có thể đăng ký nó với Scout bằng phương thức `extend` trong engine manager của Scout. Bạn nên gọi phương thức `extend` từ phương thức `boot` của `AppServiceProvider` hoặc bất kỳ service provider nào khác được application của bạn sử dụng. Ví dụ: nếu bạn đã viết một `MySqlSearchEngine`, bạn có thể đăng ký nó như sau:

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

<a name="builder-macros"></a>
## Builder Macros

Nếu bạn muốn định nghĩa một phương thức builder tùy chỉnh, bạn có thể sử dụng phương thức `macro` trên class `Laravel\Scout\Builder`. Thông thường, "macro" phải được định nghĩa trong phương thức `boot` của một [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;
    use Laravel\Scout\Builder;

    class ScoutMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's scout macros.
         *
         * @return void
         */
        public function boot()
        {
            Builder::macro('count', function () {
                return $this->engine->getTotalCount(
                    $this->engine()->search($this)
                );
            });
        }
    }

Phương thức `macro` chấp nhận một tên làm tham số đầu tiên và một Closure làm tham số thứ hai của nó. Closure của macro sẽ được chạy khi bạn gọi tên macro này từ một implementation `Laravel\Scout\Builder`:

    App\Order::search('Star Trek')->count();
