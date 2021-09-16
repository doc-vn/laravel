# Eloquent: Getting Started

- [Giới thiệu](#introduction)
- [Định nghĩa Model](#defining-models)
    - [Quy ước tên Eloquent Model](#eloquent-model-conventions)
- [Lấy ra Model](#retrieving-models)
    - [Collection](#collections)
    - [Phân kết quả](#chunking-results)
- [Lấy ra một Model / một thống kê](#retrieving-single-models)
    - [Lấy ra một thống kê](#retrieving-aggregates)
- [Thêm và cập nhật Model](#inserting-and-updating-models)
    - [Thêm](#inserts)
    - [Cập nhật](#updates)
    - [Mass Assignment](#mass-assignment)
    - [Các phương thức tạo khác](#other-creation-methods)
- [Xoá Model](#deleting-models)
    - [Soft Delete](#soft-deleting)
    - [Query Model Soft Deleted](#querying-soft-deleted-models)
- [Query Scope](#query-scopes)
    - [Global Scope](#global-scopes)
    - [Local Scope](#local-scopes)
- [Event](#events)
    - [Observer](#observers)

<a name="introduction"></a>
## Giới thiệu

ORM Eloquent được đi kèm với Laravel cung cấp một implementation ActiveRecord đơn giản và đẹp mắt để làm việc với cơ sở dữ liệu của bạn. Mỗi bảng cơ sở dữ liệu có một "Model" tương ứng được sử dụng để tương tác với bảng đó. Các model cho phép bạn truy vấn dữ liệu trong các bảng, và cũng như thêm các bản ghi mới vào bảng.

Trước khi bắt đầu, bạn hãy chắc chắn là đã cấu hình kết nối cơ sở dữ liệu trong file `config/database.php`. Để biết thêm thông tin về cách cấu hình cơ sở dữ liệu của bạn, hãy xem [tài liệu](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>
## Định nghĩa Model

Để bắt đầu, bạn hãy tạo một model Eloquent. Các model thường được lưu trong thư mục `app`, nhưng bạn có thể tự do lưu chúng ở bất cứ đâu, mà có thể được auto-loaded theo file `composer.json` của bạn. Tất cả các model Eloquent đều được extend từ class `Illuminate\Database\Eloquent\Model`.

Cách dễ nhất để tạo một instance model là sử dụng [lệnh Artisan](/docs/{{version}}/artisan) `make: model`:

    php artisan make:model User

Nếu bạn muốn tạo [migration cho cơ sở dữ liệu](/docs/{{version}}/migrations) khi bạn tạo model, bạn có thể sử dụng tùy chọn `--migration` hoặc `-m`:

    php artisan make:model User --migration

    php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Quy ước tên Eloquent Model

Bây giờ, chúng ta hãy xem một model `Flight` mẫu, sẽ sử dụng để lấy và lưu trữ dữ liệu từ bảng `flights` từ cơ sở dữ liệu`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }

#### Table Names

Lưu ý rằng chúng ta đã không khai báo gì cho Eloquent biết rằng nó sẽ phải sử dụng bảng nào cho model `Flight`. Mặc định, "snake case" cộng với tên số nhiều của class sẽ được sử dụng làm tên bảng trừ khi bạn khai báo một tên khác. Vì vậy, trong trường hợp này, Eloquent sẽ giả định rằng: model `Flight` sẽ lưu các bản ghi vào trong bảng `flights`. Bạn có thể khai báo tuỳ biến tên bảng khác bằng cách định nghĩa thuộc tính `table` trên model của bạn:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### Primary Keys

Eloquent cũng sẽ giả định rằng mỗi bảng có một cột khóa chính có tên là `id`. Bạn có thể định nghĩa một thuộc tính protected `$primaryKey` để ghi đè quy ước này.

Ngoài ra, Eloquent cũng giả định rằng khóa chính là một giá trị integer tăng dần, có nghĩa là mặc định, khóa chính sẽ được tự động chuyển thành một `int`. Nếu bạn muốn sử dụng khóa chính không tăng dần hoặc không phải dạng integer, thì khoá chính của bạn phải định nghĩa thuộc tính public `$incrementing` trên model của bạn là `false`. Nếu khóa chính của bạn không phải là dạng integer, bạn nên định nghĩa thuộc tính protected `$keyType` trên model của bạn là `string`.

#### Timestamps

Mặc định, Eloquent sẽ yêu cầu các cột `created_at` và `updated_at` tồn tại trong các bảng của bạn. Nếu bạn không muốn các cột này được tự động quản lý bởi Eloquent, hãy định nghĩa thuộc tính `$timestamps` trên model của bạn là `false`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }

Nếu bạn cần tùy biến định dạng timestamp của bạn, hãy định nghĩa thuộc tính `$dateFormat` trong model của bạn. Thuộc tính này sẽ định nghĩa cái cách mà các thuộc tính date được lưu trữ vào trong cơ sở dữ liệu, cũng như định dạng của chúng khi model được chuyển đổi thành một mảng hoặc JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

Nếu bạn cần tùy biến tên của các cột được sử dụng để lưu trữ timestamp, bạn có thể set các hằng số `CREATED_AT` và `UPDATED_AT` trong model của bạn:

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

#### Database Connection

Mặc định, tất cả các model Eloquent sẽ sử dụng kết nối cơ sở dữ liệu mặc định được cấu hình trong application của bạn. Nếu bạn muốn khai báo một kết nối khác cho model, hãy sử dụng thuộc tính `$connection`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The connection name for the model.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="retrieving-models"></a>
## Lấy ra Model

Khi bạn đã tạo một model và [bảng cơ sở dữ liệu được liên kết với model đó](/docs/{{version}}/migrations#writing-migrations), bạn có thể bắt đầu truy xuất dữ liệu từ cơ sở dữ liệu của bạn. Hãy nghĩ về mỗi model Eloquent như là một [query builder](/docs/{{version}}/queries) cho phép bạn truy vấn vào bảng cơ sở dữ liệu được liên kết với model. Ví dụ:

    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### Adding Additional Constraints

Phương thức `all` của Eloquent sẽ trả về tất cả các bản ghi có trong bảng của model. Vì mỗi model Eloquent đóng vai trò là một [query builder](/docs/{{version}}/queries), nên bạn cũng có thể thêm các ràng buộc cho các truy vấn, sau đó sử dụng phương thức `get` để lấy ra kết quả:

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} Vì các model Eloquent là các query builder, nên bạn nên xem lại tất cả các phương thức có sẵn trên [query builder](/docs/{{version}}/queries). Bạn có thể sử dụng bất kỳ phương thức nào có trong các truy vấn Eloquent của bạn.

<a name="collections"></a>
### Collection

Đối với các phương thức Eloquent như `all` và `get` để lấy nhiều bản ghi, một instance của `Illuminate\Database\Eloquent\Collection` sẽ được trả về. Class `Collection` cung cấp [nhiều phương thức hữu ích](/docs/{{version}}/eloquent-collections#available-methods) để làm việc với các bản ghi Eloquent của bạn:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

Tất nhiên, bạn cũng có thể chạy vòng lặp cho collection giống như một mảng:

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### Phân kết quả

Nếu bạn cần xử lý hàng ngàn bản ghi Eloquent, hãy sử dụng phương thức `chunk`. Phương thức `chunk` sẽ lấy ra một đoạn "chunk" của model Eloquent, đưa chúng đến một `Closure` để xử lý. Sử dụng phương thức `chunk` sẽ bảo toàn bộ nhớ khi làm việc với các truy vấn mà có nhiều bản ghi:

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

Tham số đầu tiên được truyền cho phương thức là số lượng bản ghi mà bạn muốn nhận vào cho mỗi đoạn "chunk". Closure sẽ được truyền thông qua tham số thứ hai, và nó sẽ được gọi cho mỗi đoạn được lấy từ cơ sở dữ liệu. Một truy vấn cơ sở dữ liệu sẽ được thực hiện để truy xuất từng đoạn của bản ghi và được chuyển đến cho Closure.

#### Using Cursors

Phương thức `cursor` cho phép bạn lặp qua các bản ghi cơ sở dữ liệu của bạn bằng một con trỏ, nó sẽ chỉ thực hiện một truy vấn duy nhất. Khi xử lý lượng lớn dữ liệu, phương thức `cursor` có thể được sử dụng để giảm đáng kể việc sử dụng bộ nhớ của bạn:

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

<a name="retrieving-single-models"></a>
## Lấy ra một Model / một thống kê

Tất nhiên, ngoài việc truy xuất tất cả các bản ghi có trong một bảng đã cho, bạn cũng có thể truy xuất một bản ghi bằng cách sử dụng `find` hoặc `first`. Thay vì trả về một tập hợp các model, thì các phương thức này trả về một instance model duy nhất:

    // Retrieve a model by its primary key...
    $flight = App\Flight::find(1);

    // Retrieve the first model matching the query constraints...
    $flight = App\Flight::where('active', 1)->first();

Bạn cũng có thể gọi phương thức `find` với một mảng các khóa chính, sẽ trả về một tập hợp các bản ghi khớp với mảng khoá chính đó:

    $flights = App\Flight::find([1, 2, 3]);

#### Not Found Exceptions

Thỉnh thoảng bạn có thể muốn đưa ra một ngoại lệ nếu không tìm thấy model. Điều này đặc biệt hữu ích trong các route hoặc controller. Các phương thức `findOrFail` và `firstOrFail` sẽ lấy ra kết quả đầu tiên của truy vấn; tuy nhiên, nếu không tìm thấy kết quả nào, một `Illuminate\Database\Eloquent\ModelNotFoundException` sẽ được đưa ra:

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

Nếu ngoại lệ không được xử lý, một HTTP response `404` sẽ tự động được gửi đến cho người dùng. Bạn sẽ không cần thiết phải viết code kiểm tra để trả về các response `404` khi sử dụng các phương thức này:

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### Lấy ra một thống kê

Bạn cũng có thể sử dụng các phương thức `count`, `sum`, `max`, và [các phương thức thống kê](/docs/{{version}}/queries#aggregates) khác được cung cấp bởi [query builder](/docs/{{version}}/queries). Các phương thức này trả về giá trị thay vì một instance model:

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Thêm và cập nhật Model

<a name="inserts"></a>
### Thêm

Để tạo một bản ghi mới vào trong cơ sở dữ liệu, hãy tạo một instance model mới, set các thuộc tính có trên model đó, và sau đó gọi phương thức `save`:

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * Create a new flight instance.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate the request...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

Trong ví dụ này, chúng ta đã gán tham số `name` từ HTTP request đến cho thuộc tính `name` của instance model `App\Flight`. Khi chúng ta gọi phương thức `save`, một bản ghi sẽ được thêm vào cơ sở dữ liệu. Các timestamp `created_at` và` update_at` cũng sẽ tự động được set khi phương thức `save` được gọi, do đó bạn không cần phải set chúng.

<a name="updates"></a>
### Cập nhật

Phương thức `save` cũng có thể được sử dụng để cập nhật các model đã tồn tại trong cơ sở dữ liệu. Để cập nhật một model, bạn nên truy xuất nó ra, set lại bất kỳ thuộc tính nào bạn muốn cập nhật và sau đó gọi phương thức `save`. Timestamp `update_at` sẽ tự động được cập nhật lại, do đó bạn không cần phải tự phải set giá trị cho nó:

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### Mass Updates

Cập nhật cũng có thể được thực hiện đối với một số lượng lớn model tương ứng với một truy vấn nhất định. Trong ví dụ này, tất cả các flight 'đang hoạt động' và có 'điểm đến' là `San Diego` sẽ bị đánh dấu là delay:

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

Phương thức `update` yêu cầu một mảng các cặp gồm: tên cột và giá trị cần được cập nhật.

> {note} Khi chạy một mass update thông qua Eloquent, thì các event của model như là `saved` và `updated` sẽ không được kích hoạt. Điều này là do các model không thực sự được lấy ra khi chạy một mass update.

<a name="mass-assignment"></a>
### Mass Assignment

Bạn cũng có thể sử dụng phương thức `create` để lưu một model mới với chỉ một dòng code duy nhất. Model được thêm vào rồi sẽ được trả về cho bạn từ phương thức đó. Tuy nhiên, trước khi làm như vậy, bạn sẽ cần khai báo thuộc tính `fillable` hoặc `guarded` trên model, vì mặc định, tất cả các model Eloquent đều được bảo vệ để chống lại việc mass-assignment.

Lỗ hổng mass-assignment xảy ra khi người dùng truyền một tham số HTTP mà bạn không yêu cầu thông qua một request và tham số đó thay đổi giá trị một cột trong cơ sở dữ liệu của bạn mà bạn không mong muốn. Ví dụ: kẻ xấu có thể gửi một tham số `is_admin` thông qua một request HTTP, sau đó được truyền vào phương thức `create` của model của bạn, cho phép người đó có thể nâng quyền lên đến quyền admin.

Vì vậy, để bắt đầu, bạn nên định nghĩa thuộc tính của bạn mà bạn muốn mass assignable. Bạn có thể làm điều này bằng cách sử dụng thuộc tính `$fillable` trên model. Ví dụ: hãy tạo thuộc tính `name` trong model `Flight` có thể được sử dụng để mass assignable:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Khi chúng ta đã tạo cho các thuộc tính có thể được sử dụng để mass assignable, thì chúng ta có thể sử dụng phương thức `create` để thêm một bản ghi mới vào cơ sở dữ liệu. Phương thức `create` sẽ trả về instance model đã được lưu:

    $flight = App\Flight::create(['name' => 'Flight 10']);

Nếu bạn đã có một instance model, bạn có thể sử dụng phương thức `fill` để thêm vào model một mảng các thuộc tính:

    $flight->fill(['name' => 'Flight 22']);

#### Guarding Attributes

Trong khi `$fillable` đóng vai trò là một "danh sách trắng" cho các thuộc tính có thể được sử dụng để mass assignable, bạn cũng có thể chọn sử dụng `$guarded`. Thuộc tính `$guarded` nên chứa một loạt các thuộc tính mà bạn không muốn được sử dụng cho mass assignable. Tất cả các thuộc tính khác không có trong mảng sẽ được sử dụng mass assignable. Vì vậy, thuộc tính `$guarded` giống như là một "danh sách đen". Tất nhiên, bạn nên sử dụng `$fillable` hoặc `$guarded` - nhưng không phải cả hai. Trong ví dụ dưới đây, tất cả các thuộc tính **ngoại trừ `price`** sẽ được sử dụng mass assignable:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that aren't mass assignable.
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

Nếu bạn muốn làm cho tất cả các thuộc tính có thể được sử dụng mass assignable, bạn có thể định nghĩa thuộc tính `$guarded` là một mảng trống:

    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### Các phương thức tạo khác

#### `firstOrCreate`/ `firstOrNew`

Có hai phương thức khác mà bạn có thể sử dụng để tạo các model bằng cách mass assigning thuộc tính: `firstOrCreate` và `firstOrNew`. Phương thức `firstOrCreate` sẽ thử xác định bản ghi có trong cơ sở dữ liệu hay chưa bằng cách sử dụng các cặp giá trị gồm: cột và giá trị. Nếu không thể tìm thấy model trong cơ sở dữ liệu, một bản ghi sẽ được thêm với các thuộc tính từ tham số đầu tiên, và các thuộc tính tùy chọn trong tham số thứ hai.

Phương thức `firstOrNew`, giống như phương thức `firstOrCreate` cũng sẽ cố gắng thử xác định một bản ghi có trong cơ sở dữ liệu có khớp với các thuộc tính đã cho hay không. Tuy nhiên, nếu không tìm thấy model, một instance model mới sẽ được trả về. Lưu ý rằng model được trả về bởi `firstOrNew` vẫn chưa được lưu vào trong cơ sở dữ liệu. Bạn sẽ cần gọi `save`  để lưu nó:

    // Retrieve flight by name, or create it if it doesn't exist...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // Retrieve flight by name, or create it with the name and delayed attributes...
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

    // Retrieve by name, or instantiate...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

    // Retrieve by name, or instantiate with the name and delayed attributes...
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

#### `updateOrCreate`

Bạn cũng có thể gặp các tình huống mà bạn muốn cập nhật một model nếu có hoặc tạo một model mới nếu không tồn tại. Laravel cung cấp một phương thức `updateOrCreate` để thực hiện điều này với chỉ trong một bước. Giống như phương thức `firstOrCreate`, `updateOrCreate` sẽ lưu model vào cơ sở dữ liệu, do đó bạn sẽ không cần phải gọi `save()`:

    // If there's a flight from Oakland to San Diego, set the price to $99.
    // If no matching model exists, create one.
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    );

<a name="deleting-models"></a>
## Xoá Model

Để xóa một model, hãy gọi phương thức `delete` trên một instance model:

    $flight = App\Flight::find(1);

    $flight->delete();

#### Deleting An Existing Model By Key

Trong ví dụ trên, chúng ta đang lấy một model từ cơ sở dữ liệu trước khi gọi phương thức `delete`. Tuy nhiên, nếu bạn biết khóa chính của model, bạn có thể xóa model mà không cần phải truy xuất nó ra. Để làm như vậy, hãy gọi phương thức `destroy`:

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### Deleting Models By Query

Dĩ nhiên, bạn cũng có thể chạy một câu lệnh xóa trên một tập các model. Trong ví dụ này, chúng ta sẽ xóa tất cả các flight có đánh dấu là không hoạt động. Giống như mass update, mass delete sẽ không kích hoạt bất kỳ event nào của model khi các model bị xóa:

    $deletedRows = App\Flight::where('active', 0)->delete();

> {note} Khi thực hiện câu lệnh mass delete thông qua Eloquent, các event model như là `deleting` và `deleted` sẽ không được kích hoạt cho các model đã xóa. Điều này là do các model không thực sự được lấy ra khi thực hiện câu lệnh xóa.

<a name="soft-deleting"></a>
### Soft Delete

Ngoài việc thực sự xóa các bản ghi ra khỏi cơ sở dữ liệu của bạn, Eloquent cũng có thể "soft delete" các model. Khi các model bị soft delete, thì chúng không thực sự bị xóa ra khỏi cơ sở dữ liệu của bạn. Thay vào đó, một thuộc tính `deleted_at` sẽ được set vào model và được thêm vào cơ sở dữ liệu. Nếu một model có giá trị `deleted_at` không null, model đó đã bị soft delete. Để bật soft delete cho một model, hãy sử dụng trait `Illuminate\Database\Eloquent\SoftDeletes` trên model và thêm cột `deleted_at` vào thuộc tính `$dates` của bạn:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;

        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

Tất nhiên, bạn cần thêm cột `deleted_at` vào bảng cơ sở dữ liệu của bạn. [Schema builder](/docs/{{version}}/migrations) của Laravel có chứa một phương thức helper để tạo cột này:

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

Bây giờ, khi bạn gọi phương thức `delete` trên model, cột `deleted_at` sẽ được set thành ngày và giờ của hiện tại. Và khi truy vấn một model mà có sử dụng soft delete, thì các model mà đã bị soft delete thì sẽ bị tự động loại khỏi tất cả các kết quả truy vấn.

Để xác định xem một instance model đã cho có bị soft delete hay chưa, hãy sử dụng phương thức `trashed`:

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### Query Model Soft Deleted

#### Including Soft Deleted Models

Như đã lưu ý ở trên, các model bị soft delete sẽ bị tự động loại ra khỏi kết quả truy vấn. Tuy nhiên, bạn có thể bắt buộc các model bị soft delete xuất hiện trong tập kết quả bằng cách sử dụng phương thức `withTrashed` trong câu truy vấn:

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

Phương thức `withTrashed` cũng có thể được sử dụng cho các query [quan hệ](/docs/{{version}}/eloquent-relationships):

    $flight->history()->withTrashed()->get();

#### Retrieving Only Soft Deleted Models

Phương thức `onlyTrashed` sẽ **chỉ** truy xuất các model đã bị soft deleted:

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### Restoring Soft Deleted Models

Thỉnh thoảng bạn có thể muốn "un-delete" một model đã bị soft deleted. Để khôi phục model đã bị soft deleted thành trạng thái hoạt động, hãy sử dụng phương thức `restore` trên một instance model:

    $flight->restore();

Bạn cũng có thể sử dụng phương thức `restore` trong một truy vấn để nhanh chóng khôi phục nhiều model. Một lần nữa, giống như các hoạt động "mass" khác, điều này sẽ không kích hoạt bất kỳ event nào của model khi các model được khôi phục:

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Giống như phương thức `withTrashed`, phương thức` restore` cũng có thể được sử dụng trên [quan hệ](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();

#### Permanently Deleting Models

Thỉnh thoảng bạn có thể cần phải thực sự loại bỏ vĩnh viến một model ra khỏi cơ sở dữ liệu của bạn. Để xóa vĩnh viễn một model bị soft deleted ra khỏi cơ sở dữ liệu, hãy sử dụng phương thức `forceDelete`:

    // Force deleting a single model instance...
    $flight->forceDelete();

    // Force deleting all related models...
    $flight->history()->forceDelete();

<a name="query-scopes"></a>
## Query Scope

<a name="global-scopes"></a>
### Global Scope

Global scope cho phép bạn thêm các ràng buộc cho tất cả các truy vấn cho một model nhất định. Chức năng [soft delete](#soft-deleting) của Laravel sử dụng global scope để chỉ lấy các model "non-deleted" ra khỏi cơ sở dữ liệu. Viết global scope của riêng bạn có thể cung cấp một cách thuận tiện và dễ dàng để đảm bảo mọi truy vấn cho một model nhất định đều nhận được các ràng buộc nhất định.

#### Writing Global Scopes

Viết một global scope rất là đơn giản. Định nghĩa một class implement từ interface `Illuminate\Database\Eloquent\Scope`. Interface này yêu cầu bạn implement một phương thức: `apply`. Phương thức `apply` có thể thêm các ràng buộc `where` cho truy vấn khi cần:

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class AgeScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }

> {tip} Nếu global scope của bạn đang thêm các cột vào câu lệnh select, thì bạn nên sử dụng phương thức `addSelect` thay vì `select`. Điều này sẽ ngăn việc vô tình thay thế lệnh select hiện tại của truy vấn.

#### Applying Global Scopes

Để gán một global scope cho một model, bạn nên ghi đè phương thức `boot` của một model và sử dụng phương thức `addGlobalScope`:

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope(new AgeScope);
        }
    }

Sau khi thêm scope, một truy vấn tới `User::all()` sẽ tạo ra SQL như sau:

    select * from `users` where `age` > 200

#### Anonymous Global Scopes

Eloquent cũng cho phép bạn định nghĩa global scope bằng cách sử dụng Closures, điều này đặc biệt hữu ích cho các scope đơn giản không cần thiết tạo một class riêng biệt:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

#### Removing Global Scopes

Nếu bạn muốn xóa một global trong cho một truy vấn, bạn có thể sử dụng phương thức `withoutGlobalScope`. Phương thức chấp nhận tên class của global scope làm tham số duy nhất của nó:

    User::withoutGlobalScope(AgeScope::class)->get();

Nếu bạn muốn xóa một vài hoặc thậm chí tất cả các global scope, bạn cũng có thể sử dụng phương thức `withoutGlobalScopes`:

    // Remove all of the global scopes...
    User::withoutGlobalScopes()->get();

    // Remove some of the global scopes...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### Local Scope

Local scope cho phép bạn xác định các nhóm ràng buộc chung mà bạn có thể dễ dàng sử dụng lại trong suốt application của bạn. Ví dụ: bạn có thể cần thường xuyên truy xuất tất cả người dùng được coi là "popular". Để định nghĩa một scope, hãy set tiền tố `scope` cho tên phương thức trong model Eloquent.

Scope phải luôn trả về một instance query builder:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * Scope a query to only include active users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### Utilizing A Local Scope

Khi mà scope đã được định nghĩa xong, bạn có thể gọi phương thức scope khi truy vấn model. Tuy nhiên, bạn không cần ghi tiền tố `scope` khi gọi phương thức. Bạn thậm chí có thể kết hợp nó với các scope khác, ví dụ:

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### Dynamic Scopes

Thỉnh thoảng bạn có thể muốn định nghĩa một scope chấp nhận các tham số. Để bắt đầu, chỉ cần thêm các tham số bổ sung vào scope của bạn. Các tham số scope cần được định nghĩa sau tham số `$query`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @param mixed $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

Bây giờ, bạn có thể truyền tham số khi gọi scope:

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## Event

Các eloquent model sẽ kích hoạt một số vent, cho phép bạn hook đến các chỗ khác trong vòng đời của một model: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. Event cho phép bạn dễ dàng thực thi code mỗi khi một class model cụ thể nào đó được lưu hoặc được cập nhật trong cơ sở dữ liệu.

Event `retrieved` sẽ được kích hoạt khi một model đã tồn tại được lấy ra từ cơ sở dữ liệu. Khi một model mới được lưu vào lần đầu tiên, các event `creating` và `created` sẽ kích hoạt. Nếu một model đã tồn tại trong cơ sở dữ liệu và phương thức `save` được gọi, các event `updating` và `updated` sẽ kích hoạt. Tuy nhiên, trong cả hai trường hợp trên, thì các event `saving` / `saved` cũng sẽ kích hoạt.

Để bắt đầu, hãy định nghĩa một thuộc tính `$dispatchesEvents` trên model Eloquent của bạn để map các thời điểm khác nhau trong vòng đời của model Eloquent vào [event classes](/docs/{{version}}/events) của bạn:

    <?php

    namespace App;

    use App\Events\UserSaved;
    use App\Events\UserDeleted;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The event map for the model.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

<a name="observers"></a>
### Observer

Nếu bạn đang listen nhiều event trên một model nhất định, bạn có thể sử dụng các observer để nhóm tất cả những listen của bạn vào một class duy nhất. Các class observer có tên phương thức chính là tên các event Eloquent mà bạn muốn listen. Mỗi phương thức này nhận vào model làm tham số duy nhất của chúng. Laravel không chứa một thư mục mặc định cho observer, vì vậy bạn có thể tạo bất kỳ thư mục nào bạn muốn để chứa các class observer của bạn:

    <?php

    namespace App\Observers;

    use App\User;

    class UserObserver
    {
        /**
         * Listen to the User created event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * Listen to the User deleting event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function deleting(User $user)
        {
            //
        }
    }

Để đăng ký một observer, hãy sử dụng phương thức `observe` trên model mà bạn muốn observe. Bạn có thể đăng ký observer trong phương thức `boot` của một trong những service provider của bạn. Trong ví dụ này, chúng ta sẽ đăng ký observer trong `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use App\User;
    use App\Observers\UserObserver;
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
            User::observe(UserObserver::class);
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
