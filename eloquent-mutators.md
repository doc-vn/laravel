# Eloquent: Mutators & Casting

- [Giới thiệu](#introduction)
- [Accessor và Mutator](#accessors-and-mutators)
    - [Định nghĩa một Accessor](#defining-an-accessor)
    - [Định nghĩa một Mutator](#defining-a-mutator)
- [Attribute Casting](#attribute-casting)
    - [Array và JSON Casting](#array-and-json-casting)
    - [Date Casting](#date-casting)
    - [Enum Casting](#enum-casting)
    - [Encrypted Casting](#encrypted-casting)
    - [Query Time Casting](#query-time-casting)
- [Custom Casts](#custom-casts)
    - [Value Object Casting](#value-object-casting)
    - [Array / JSON Serialization](#array-json-serialization)
    - [Inbound Casting](#inbound-casting)
    - [Cast Parameters](#cast-parameters)
    - [Castables](#castables)

<a name="introduction"></a>
## Giới thiệu

Accessors, mutators, và attribute casting cho phép bạn chuyển đổi các thuộc tính Eloquent khi bạn lấy ra hoặc set lại trên một instance model. Ví dụ: bạn có thể muốn sử dụng [Laravel encrypter](/docs/{{version}}/encryption) để mã hóa một giá trị trong khi lưu nó vào trong cơ sở dữ liệu, sau đó tự động giải mã khi bạn truy cập vào Eloquent model đó. Hoặc, bạn có thể muốn chuyển đổi một chuỗi JSON được lưu trữ trong cơ sở dữ liệu của bạn thành một mảng khi nó được lấy ra thông qua model Eloquent của bạn.

<a name="accessors-and-mutators"></a>
## Accessor và Mutator

<a name="defining-an-accessor"></a>
### Định nghĩa một Accessor

Accessor sẽ biến đổi một giá trị thuộc tính Eloquent khi nó được truy cập. Để định nghĩa một accessor, hãy tạo một phương thức `get{Attribute}Attribute` trên model của bạn trong đó `{Attribute}` là tên được đặt theo kiểu "studly" của cột mà bạn muốn truy cập.

Trong ví dụ này, chúng ta sẽ định nghĩa một accessor cho thuộc tính `first_name`. Accessor sẽ tự động được gọi bởi Eloquent khi bạn truy xuất vào thuộc tính `first_name`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

Như bạn có thể thấy, giá trị ban đầu của cột được truyền vào accessor, cho phép bạn thao tác và trả về một giá trị khác. Để truy cập vào giá trị của accessor, bạn có thể truy cập dễ dàng vào thuộc tính `first_name` trên một instance model:

    use App\Models\User;

    $user = User::find(1);

    $firstName = $user->first_name;

Bạn không bị giới hạn trong việc tương tác với một thuộc tính trong accessor của bạn. Bạn cũng có thể sử dụng accessor để trả về các giá trị mới, được tính toán từ các thuộc tính hiện có:

    /**
     * Get the user's full name.
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

> {tip} Nếu bạn muốn thêm các giá trị đã được tính toán này vào mảng hoặc JSON được chuyển đổi của model, [bạn sẽ cần thêm chúng vào](/docs/{{version}}/eloquent-serialization#appending-values-to-json).

<a name="defining-a-mutator"></a>
### Định nghĩa một Mutator

Một mutator sẽ biến đổi một giá trị của một thuộc tính Eloquent khi nó được set. Để định nghĩa một mutator, hãy định nghĩa một phương thức `set{Attribute}Attribute` trên model của bạn trong đó `{Attribute}` là tên được set theo kiểu "studly" của cột mà bạn muốn truy cập.

Hãy định nghĩa một mutator cho thuộc tính `first_name`. Mutator này sẽ được gọi tự động khi bạn set một giá trị cho thuộc tính `first_name` trên model:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Set the user's first name.
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

Mutator sẽ nhận vào giá trị mà đang được set cho thuộc tính đó, và cho phép bạn thao tác với giá trị đó rồi set một giá trị mới cho thuộc tính `$attributes` bên trong của model Eloquent. Để sử dụng mutator, chúng ta chỉ cần set thuộc tính `first_name` trên Eloquent model:

    use App\Models\User;

    $user = User::find(1);

    $user->first_name = 'Sally';

Trong ví dụ này, hàm `setFirstNameAttribute` sẽ được gọi với giá trị `Sally`. Mutator sẽ sử dụng hàm `strtolower` cho giá trị được đưa vào và set giá trị kết quả cho mảng `$attributes`.

<a name="attribute-casting"></a>
## Attribute Casting

Casting thuộc tính cung cấp chức năng tương tự như accessor và mutator mà không yêu cầu bạn phải định nghĩa thêm bất kỳ phương thức nào trên model của bạn. Thay vào đó, thuộc tính `$casts` của model của bạn phải cung cấp một phương thức để chuyển đổi các thuộc tính thành các loại dữ liệu phổ biến.

Thuộc tính `$casts` phải là một mảng trong đó khóa là tên của thuộc tính được cast và giá trị là loại mà bạn muốn cast. Các loại cast được hỗ trợ là:

<div class="content-list" markdown="1">

- `array`
- `AsStringable::class`
- `boolean`
- `collection`
- `date`
- `datetime`
- `immutable_date`
- `immutable_datetime`
- `decimal:`<code>&lt;digits&gt;</code>
- `double`
- `encrypted`
- `encrypted:array`
- `encrypted:collection`
- `encrypted:object`
- `float`
- `integer`
- `object`
- `real`
- `string`
- `timestamp`

</div>

Để minh họa việc cast thuộc tính, hãy truyền thuộc tính `is_admin`, được lưu trong cơ sở dữ liệu của chúng ta là dưới dạng một số nguyên (`0` hoặc `1`) thành một giá trị boolean:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

Sau khi định nghĩa cast xong, thuộc tính `is_admin` sẽ luôn được cast thành boolean khi bạn truy cập vào nó, ngay cả khi giá trị của nó được lưu trong cơ sở dữ liệu là dưới dạng số nguyên:

    $user = App\Models\User::find(1);

    if ($user->is_admin) {
        //
    }

Nếu bạn cần thêm một cast mới, tạm thời trong khi chạy, bạn có thể sử dụng phương thức `mergeCasts`. Các định nghĩa cast này sẽ được thêm vào bất kỳ cast nào đã được định nghĩa trên model:

    $user->mergeCasts([
        'is_admin' => 'integer',
        'options' => 'object',
    ]);

> {note} Các thuộc tính `null` sẽ không được cast. Ngoài ra, bạn cũng đừng định nghĩa một cast (hoặc một thuộc tính) có cùng tên với một quan hệ.

<a name="stringable-casting"></a>
#### Stringable Casting

Bạn có thể sử dụng class cast `Illuminate\Database\Eloquent\Casts\AsStringable` để cast model cho [đối tượng fluent `Illuminate\Support\Stringable`](/docs/{{version}}/helpers#fluent-strings-method-list):

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\AsStringable;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'directory' => AsStringable::class,
        ];
    }

<a name="array-and-json-casting"></a>
### Array & JSON Casting

Cast `array` đặc biệt hữu ích khi làm việc với các cột được lưu dưới dạng JSON. Ví dụ: nếu cơ sở dữ liệu của bạn có một loại trường `JSON` hoặc `TEXT` chứa một chuỗi dưới dạng JSON, thì việc thêm `array` cast vào thuộc tính đó sẽ tự động phân giải thuộc tính đó thành một mảng PHP và khi bạn truy cập vào nó trên model Eloquent của bạn:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

Sau khi đã định nghĩa xong cast, bạn có thể truy cập vào thuộc tính `options` và nó sẽ tự động được phân giải hoá hóa từ JSON thành một mảng PHP. Khi bạn set giá trị cho thuộc tính `options`, thì mảng đã cho sẽ tự động được chuyển hóa trở lại thành JSON để lưu trữ:

    use App\Models\User;

    $user = User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();

Để cập nhật một trường của thuộc tính JSON bằng một cú pháp ngắn gọn hơn, bạn có thể sử dụng toán tử `->` khi gọi phương thức `update`:

    $user = User::find(1);

    $user->update(['options->key' => 'value']);

<a name="array-object-and-collection-casting"></a>
#### Array Object & Collection Casting

Mặc dù cast `array` là đủ cho nhiều ứng dụng, nhưng nó có một số nhược điểm. Vì cast `array` sẽ yêu cầu key phải là kiểu nguyên, nên nó sẽ không thể làm việc với key khác kiểu nguyên. Ví dụ: đoạn code sau sẽ gây ra lỗi PHP:

    $user = User::find(1);

    $user->options['key'] = $value;

Để giải quyết vấn đề này, Laravel cung cấp một `AsArrayObject` để cast thuộc tính JSON của bạn sang một class [ArrayObject](https://www.php.net/manual/en/class.arrayobject.php). Tính năng này được làm bằng cách sử dụng implementation [custom cast](#custom-casts) của Laravel, cho phép Laravel lưu vào bộ nhớ cache một cách thông minh và chuyển đối tượng bị thay đổi sao cho các offset riêng có thể được sửa mà không gây ra lỗi PHP. Để sử dụng cast `AsArrayObject`, bạn chỉ cần gán nó cho một thuộc tính:

    use Illuminate\Database\Eloquent\Casts\AsArrayObject;

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'options' => AsArrayObject::class,
    ];

Tương tự, Laravel cũng cung cấp một cast `AsCollection` để cast các thuộc tính JSON của bạn thành một instance [Collection](/docs/{{version}}/collections) của Laravel:

    use Illuminate\Database\Eloquent\Casts\AsCollection;

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'options' => AsCollection::class,
    ];

<a name="date-casting"></a>
### Date Casting

Mặc định, Eloquent sẽ cast các field `created_at` và `updated_at` sang các instance của [Carbon](https://github.com/briannesbitt/Carbon), được extend từ class `DateTime` của PHP và cung cấp nhiều phương thức hữu ích. Bạn có thể cast thêm các thuộc tính date này bằng cách định nghĩa thêm các cast date bổ sung vào trong mảng thuộc tính `$casts` của model của bạn. Thông thường, date nên được cast bằng cách sử dụng các loại cast là: `datetime` hoặc `immutable_datetime`.

Khi định nghĩa các kiểu cast `date` hoặc `datetime`, bạn cũng có thể chỉ định định dạng của date đó. Định dạng này sẽ được sử dụng khi [model được chuyển đổi thành mảng hoặc JSON](/docs/{{version}}/eloquent-serialization):

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];

Khi một cột được cast dưới dạng là một ngày, bạn có set đặt giá trị của thuộc tính model tương ứng thành các kiểu: một UNIX timestamp, chuỗi ngày (`Y-m-d`), chuỗi ngày và giờ hoặc instance `DateTime` hoặc `Carbon`. Các giá trị của ngày sẽ được chuyển đổi và lưu trữ chính xác trong cơ sở dữ liệu của bạn.

Bạn có thể tùy chỉnh định dạng mặc định của việc chuyển đổi này cho tất cả các ngày có trong model của bạn bằng cách định nghĩa phương thức `serializeDate` trên model của bạn. Phương thức này không ảnh hưởng đến cách định dạng ngày của bạn trong lưu trữ trong cơ sở dữ liệu:

    /**
     * Prepare a date for array / JSON serialization.
     *
     * @param  \DateTimeInterface  $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date)
    {
        return $date->format('Y-m-d');
    }

Để chỉ định định dạng sẽ được sử dụng khi thực sự lưu trữ ngày tháng của model trong cơ sở dữ liệu của bạn, bạn nên định nghĩa một thuộc tính `$dateFormat` trên model của bạn:

    /**
     * The storage format of the model's date columns.
     *
     * @var string
     */
    protected $dateFormat = 'U';

<a name="date-casting-and-timezones"></a>
#### Date Casting, Serialization, & Timezones

Mặc định, các cast `date` và `datetime` sẽ chuyển đổi các ngày thành các chuỗi ngày UTC ISO-8601 (`1986-05-28T21:05:54.000000Z`), bất kể múi giờ được chỉ định trong tùy chọn cấu hình `timezone` của ứng dụng của bạn là gì. Bạn được khuyến khích luôn sử dụng định dạng chuyển đổi này, và cũng như cách lưu trữ ngày trong cơ sở dữ liệu bằng cách không thay đổi tùy chọn cấu hình `timezone` của ứng dụng mặc định là giá trị `UTC`. Việc sử dụng múi giờ UTC một cách nhất quán trong toàn bộ ứng dụng của bạn sẽ mang lại mức độ tương tác tối đa với các thư viện thao tác ngày khác được viết bằng PHP hoặc JavaScript.

Nếu một định dạng tùy chỉnh được áp dụng cho kiểu `date` hoặc `datetime`, chẳng hạn như `datetime:Y-m-d H:i:s`, thì múi giờ bên trong instance Carbon sẽ được sử dụng trong quá trình chuyển đổi ngày. Thông thường, đây sẽ là múi giờ được chỉ định trong tùy chọn cấu hình `timezone` của ứng dụng của bạn.

<a name="enum-casting"></a>
### Enum Casting

> {note} Casting Enum chỉ khả dụng cho PHP 8.1 trở lên.

Eloquent cũng cho phép bạn cast các giá trị thuộc tính của bạn sang PHP enums. Để thực hiện điều này, bạn có thể chỉ định thuộc tính và enum mà bạn muốn truyền vào trong mảng thuộc tính `$casts` của model:

    use App\Enums\ServerStatus;

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'status' => ServerStatus::class,
    ];

Khi bạn đã định nghĩa xong kiểu cast trong model của bạn, thuộc tính được chỉ định sẽ tự động được cast đến một enum hoặc một enum chuyển qua khi bạn tương tác với thuộc tính:

    if ($server->status == ServerStatus::provisioned) {
        $server->status = ServerStatus::ready;

        $server->save();
    }

<a name="encrypted-casting"></a>
### Encrypted Casting

Cast `encrypted` sẽ mã hóa giá trị thuộc tính của model bằng cách sử dụng các tính năng [mã hóa](/docs/{{version}}/encryption) được tích hợp sẵn trong Laravel. Ngoài ra, các cast `encrypted:array`, `encrypted:collection`, `encrypted:object`, `AsEncryptedArrayObject` và `AsEncryptedCollection` hoạt động cũng giống như các đối tượng không được mã hóa của chúng; tuy nhiên, như bạn có thể mong đợi, giá trị sẽ được mã hóa trở lại khi được lưu vào trong cơ sở dữ liệu của bạn.

Vì độ dài của văn bản được mã hóa không thể dự đoán được hoặc có thể dài hơn so với văn bản gốc của nó, nên hãy đảm bảo cột cơ sở dữ liệu sẽ được gán với loại `TEXT` hoặc có độ dài lớn hơn. Ngoài ra, vì các giá trị đã được mã hóa trong cơ sở dữ liệu nên bạn sẽ không thể truy vấn hoặc tìm kiếm các giá trị của thuộc tính đã được mã hóa.

<a name="query-time-casting"></a>
### Query Time Casting

Thỉnh thoảng, bạn có thể cần áp dụng cast kiểu trong khi thực hiện truy vấn, chẳng hạn như khi select một giá trị raw từ một bảng. Ví dụ: hãy xem xét truy vấn sau:

    use App\Models\Post;
    use App\Models\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

Thuộc tính `last_posted_at` trên kết quả của truy vấn này sẽ là một chuỗi. Và sẽ thật tuyệt nếu chúng ta có thể áp dụng kiểu cast `datetime` cho thuộc tính này khi thực hiện truy vấn. Rất may, chúng ta có thể thực hiện việc này bằng cách sử dụng phương thức `withCasts`:

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'datetime'
    ])->get();

<a name="custom-casts"></a>
## Custom Casts

Laravel có nhiều kiểu cast tích hợp, hữu ích; tuy nhiên, đôi khi bạn có thể cần phải định nghĩa kiểu cast của riêng bạn. Bạn có thể thực hiện điều này bằng cách định nghĩa một class implement interface `CastsAttributes`.

Các class implement interface này phải định nghĩa một phương thức `get` và một phương thức `set`. Phương thức `get` chịu trách nhiệm chuyển đổi một giá trị thô từ cơ sở dữ liệu thành một giá trị cast, trong khi phương thức `set` sẽ chuyển đổi một giá trị cast thành một giá trị thô có thể được lưu được vào trong cơ sở dữ liệu. Ví dụ: chúng ta sẽ implement lại kiểu cast `json` có sẵn dưới dạng là một kiểu cast tùy chỉnh:

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Json implements CastsAttributes
    {
        /**
         * Cast the given value.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return array
         */
        public function get($model, $key, $value, $attributes)
        {
            return json_decode($value, true);
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return json_encode($value);
        }
    }

Khi bạn đã định nghĩa xong một kiểu cast tùy chỉnh, bạn có thể gắn nó vào một thuộc tính model bằng cách sử dụng tên class của nó:

    <?php

    namespace App\Models;

    use App\Casts\Json;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'options' => Json::class,
        ];
    }

<a name="value-object-casting"></a>
### Value Object Casting

Bạn không bị giới hạn trong việc cast giá trị cho các kiểu nguyên thủy. Bạn cũng có thể cast một giá trị cho các đối tượng. Việc định nghĩa các cast tùy chỉnh để cast một giá trị cho các đối tượng rất giống với việc cast kiểu nguyên thủy; tuy nhiên, phương thức `set` sẽ trả về một mảng các cặp khóa và giá trị sẽ được sử dụng để set các giá trị thô, và lưu trữ vào model.

Ví dụ, chúng ta sẽ định nghĩa một class cast tùy chỉnh truyền nhiều giá trị model vào một đối tượng giá trị `Address`. Chúng ta sẽ giả sử giá trị `Address` có hai thuộc tính công khai là : `lineOne` và `lineTwo`:

    <?php

    namespace App\Casts;

    use App\Models\Address as AddressModel;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
    use InvalidArgumentException;

    class Address implements CastsAttributes
    {
        /**
         * Cast the given value.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return \App\Models\Address
         */
        public function get($model, $key, $value, $attributes)
        {
            return new AddressModel(
                $attributes['address_line_one'],
                $attributes['address_line_two']
            );
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  \App\Models\Address  $value
         * @param  array  $attributes
         * @return array
         */
        public function set($model, $key, $value, $attributes)
        {
            if (! $value instanceof AddressModel) {
                throw new InvalidArgumentException('The given value is not an Address instance.');
            }

            return [
                'address_line_one' => $value->lineOne,
                'address_line_two' => $value->lineTwo,
            ];
        }
    }

Khi cast các giá trị của đối tượng, mọi thay đổi được thực hiện đối với giá trị của đối tượng sẽ được tự động đồng bộ trở lại model trước khi model được lưu:

    use App\Models\User;

    $user = User::find(1);

    $user->address->lineOne = 'Updated Address Value';

    $user->save();

> {tip} Nếu bạn muốn chuyển đổi các model Eloquent của bạn chứa các giá trị của đối tượng thành JSON hoặc một mảng, bạn nên implement interface `Illuminate\Contracts\Support\Arrayable` và interface `JsonSerializable` trên giá trị của đối tượng.

<a name="array-json-serialization"></a>
### Array / JSON Serialization

Khi một model Eloquent được chuyển thành một mảng hoặc chuỗi JSON thông qua các phương thức `toArray` hoặc `toJson`, giá trị của các thuộc tính cast tùy chỉnh của bạn thường sẽ được đánh số thứ tự miễn là chúng implement các interface `Illuminate\Contracts\Support\Arrayable` và `JsonSerializable`. Tuy nhiên, khi sử dụng giá trị của các đối tượng do thư viện bên thứ ba cung cấp, bạn có thể không có khả năng thêm các interface này vào cho các đối tượng.

Do đó, bạn có thể chỉ định class cast tùy chỉnh của bạn sẽ chịu trách nhiệm chuyển đổi giá trị của đối tượng. Để làm như vậy, class cast tùy chỉnh của bạn phải implement interface `Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes`. Interface này sẽ yêu cầu class của bạn phải chứa phương thức `serialize` và sẽ trả về định dạng chuyển đổi mà giá trị của đối tượng mà bạn muốn tuỳ chỉnh:

    /**
     * Get the serialized representation of the value.
     *
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @param  string  $key
     * @param  mixed  $value
     * @param  array  $attributes
     * @return mixed
     */
    public function serialize($model, string $key, $value, array $attributes)
    {
        return (string) $value;
    }

<a name="inbound-casting"></a>
### Inbound Casting

Đôi khi, bạn có thể cần viết một cast tùy chỉnh chỉ biến đổi các giá trị khi được set vào trong model và không thực hiện bất kỳ hoạt động nào khi các thuộc tính đó được lấy ra từ model. Một ví dụ cổ điển về inbound cast này là cast một thuộc tính "hashing". Một inbound cast tuỳ chỉnh nên được implement interface `CastsInboundAttributes`, interface này chỉ yêu cầu phương thức `set` phải được định nghĩa trên class implement.

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;

    class Hash implements CastsInboundAttributes
    {
        /**
         * The hashing algorithm.
         *
         * @var string
         */
        protected $algorithm;

        /**
         * Create a new cast class instance.
         *
         * @param  string|null  $algorithm
         * @return void
         */
        public function __construct($algorithm = null)
        {
            $this->algorithm = $algorithm;
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return is_null($this->algorithm)
                        ? bcrypt($value)
                        : hash($this->algorithm, $value);
        }
    }

<a name="cast-parameters"></a>
### Cast Parameters

Khi gắn một cast tùy chỉnh vào một model, các tham số cast có thể được chỉ định bằng cách tách chúng ra khỏi tên class bằng ký tự `:` và phân cách bằng dấu phẩy cho nhiều tham số khác nhau. Các tham số này sẽ được truyền cho hàm khởi tạo của class cast:

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'secret' => Hash::class.':sha256',
    ];

<a name="castables"></a>
### Castables

Bạn có thể muốn cho phép các giá trị của đối tượng trong ứng dụng của bạn được định nghĩa trong các class cast tùy chỉnh của riêng chúng. Thay vì gán class cast tùy chỉnh vào model của bạn, bạn có thể gán một class giá trị đối tượng implement interface `Illuminate\Contracts\Database\Eloquent\Castable`:

    use App\Models\Address;

    protected $casts = [
        'address' => Address::class,
    ];

Các đối tượng implement interface `Castable` phải định nghĩa một phương thức `castUsing` trả về tên class của class caster tùy chỉnh chịu trách nhiệm cast đến và đi từ class `Castable`:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use App\Casts\Address as AddressCast;

    class Address implements Castable
    {
        /**
         * Get the name of the caster class to use when casting from / to this cast target.
         *
         * @param  array  $arguments
         * @return string
         */
        public static function castUsing(array $arguments)
        {
            return AddressCast::class;
        }
    }

Khi sử dụng các class `Castable`, bạn vẫn có thể truyền các tham số trong định nghĩa `$casts`. Các tham số này sẽ được truyền đến phương thức `castUsing`:

    use App\Models\Address;

    protected $casts = [
        'address' => Address::class.':argument',
    ];

<a name="anonymous-cast-classes"></a>
#### Castables & Anonymous Cast Classes

Bằng cách kết hợp "castables" và [anonymous class](https://www.php.net/manual/en/language.oop5.anonymous.php) của PHP, bạn có thể định nghĩa một đối tượng giá trị và logic cast của nó dưới dạng một đối tượng castable. Để thực hiện điều này, hãy trả về một anonymous class từ phương thức `castUsing` của đối tượng giá trị của bạn. Anonymous class sẽ triển khai interface `CastsAttributes`:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Address implements Castable
    {
        // ...

        /**
         * Get the caster class to use when casting from / to this cast target.
         *
         * @param  array  $arguments
         * @return object|string
         */
        public static function castUsing(array $arguments)
        {
            return new class implements CastsAttributes
            {
                public function get($model, $key, $value, $attributes)
                {
                    return new Address(
                        $attributes['address_line_one'],
                        $attributes['address_line_two']
                    );
                }

                public function set($model, $key, $value, $attributes)
                {
                    return [
                        'address_line_one' => $value->lineOne,
                        'address_line_two' => $value->lineTwo,
                    ];
                }
            };
        }
    }
