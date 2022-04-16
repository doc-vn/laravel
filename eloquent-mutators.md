# Eloquent: Mutators

- [Giới thiệu](#introduction)
- [Accessor và Mutator](#accessors-and-mutators)
    - [Định nghĩa một Accessor](#defining-an-accessor)
    - [Định nghĩa một Mutator](#defining-a-mutator)
- [Date Mutator](#date-mutators)
- [Attribute Casting](#attribute-casting)
    - [Custom Casts](#custom-casts)
    - [Array và JSON Casting](#array-and-json-casting)
    - [Date Casting](#date-casting)
    - [Query Time Casting](#query-time-casting)

<a name="introduction"></a>
## Giới thiệu

Accessor và Mutator cho phép bạn định dạng các giá trị của thuộc tính Eloquent khi bạn lấy ra hoặc set trên một instance model. Ví dụ: bạn có thể muốn sử dụng [Laravel encrypter](/docs/{{version}}/encryption) để mã hóa một giá trị trong khi lưu nó vào trong cơ sở dữ liệu, sau đó tự động giải mã khi bạn truy cập vào Eloquent model đó.

Ngoài các tuỳ biến accessors và mutator, Eloquent cũng có thể tự chuyển các trường kiểu date thành các trường hợp kiểu [Carbon](https://github.com/briannesbitt/Carbon) hoặc thậm chí [chuyển trường từ text sang JSON](#attribute-casting).

<a name="accessors-and-mutators"></a>
## Accessor và Mutator

<a name="defining-an-accessor"></a>
### Định nghĩa một Accessor

Để định nghĩa một accessor, hãy tạo một phương thức `getFooAttribute` trên model của bạn trong đó `Foo` là tên được đặt theo kiểu "studly" của cột mà bạn muốn truy cập. Trong ví dụ này, chúng ta sẽ định nghĩa một accessor cho thuộc tính `first_name`. Accessor sẽ tự động được gọi bởi Eloquent khi bạn truy xuất vào thuộc tính `first_name`:

    <?php

    namespace App;

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

Như bạn có thể thấy, giá trị ban đầu của cột được truyền vào accessor, cho phép bạn thao tác và trả về một giá trị khác. Để truy cập vào giá trị của accessor, bạn có thể truy cập vào thuộc tính `first_name` trên một instance model:

    $user = App\User::find(1);

    $firstName = $user->first_name;

Bạn cũng có thể sử dụng accessor để trả về các giá trị mới, được tính toán từ các thuộc tính hiện có:

    /**
     * Get the user's full name.
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

> {tip} Nếu bạn muốn thêm các giá trị đã được tính toán này vào mảng hoặc JSON được chuyển đổi của model, [bạn sẽ cần thêm chúng vào](https://laravel.com/docs/{{version}}/eloquent-serialization#appending-values-to-json).

<a name="defining-a-mutator"></a>
### Định nghĩa một Mutator

Để định nghĩa một mutator, hãy định nghĩa một phương thức `setFooAttribute` trên model của bạn trong đó `Foo` là tên được set theo kiểu "studly" của cột mà bạn muốn truy cập. Vì thế, một lần nữa, hãy định nghĩa một mutator cho thuộc tính `first_name`. Mutator này sẽ được gọi tự động khi bạn set một giá trị cho thuộc tính `first_name` trên model:

    <?php

    namespace App;

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

Mutator sẽ nhận vào giá trị mà đang được set cho thuộc tính đó, và cho phép bạn thao tác với giá trị đó rồi set một giá trị mới cho thuộc tính `$attributes` bên trong của model Eloquent. Vì vậy, ví dụ, nếu chúng ta đang set một thuộc tính `first_name` là `Sally`:

    $user = App\User::find(1);

    $user->first_name = 'Sally';

Trong ví dụ này, hàm `setFirstNameAttribute` sẽ được gọi với giá trị `Sally`. Mutator sẽ sử dụng hàm `strtolower` cho giá trị được đưa vào và set giá trị kết quả cho mảng `$attributes`.

<a name="date-mutators"></a>
## Date Mutator

Mặc định, Eloquent sẽ chuyển đổi các cột `created_at` và `update_at` thành các instance của [Carbon](https://github.com/briannesbitt/Carbon), đây là một class được extend từ class `DateTime` của PHP, cung cấp rất nhiều phương thức hữu ích. Bạn có thể thêm các thuộc tính date bằng cách set thuộc tính `$dates` trong model của bạn:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = [
            'seen_at',
        ];
    }

> {tip} Bạn có thể vô hiệu hóa các timestamp mặc định `create_at` và `updated_at` bằng cách set thuộc tính public `$timestamps` của model thành `false`.

Khi một cột được coi là một date, thì bạn có thể set giá trị của nó thành một UNIX timestamp, một date string (`Y-m-d`), date-time string, hoặc là một instance `DateTime` / `Carbon`. Các giá trị của date này sẽ được chuyển đổi và lưu chính xác vào trong cơ sở dữ liệu của bạn:

    $user = App\User::find(1);

    $user->deleted_at = now();

    $user->save();

Như đã lưu ý ở trên, khi lấy các thuộc tính được liệt kê trong thuộc tính `$dates` của bạn, chúng sẽ tự động được chuyển sang thành các instance [Carbon](https://github.com/briannesbitt/Carbon), cho phép bạn sử dụng bất kỳ phương thức nào của Carbon trên thuộc tính của model:

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### Date Formats

Mặc định, timestamp sẽ được định dạng là `'Y-m-d H:i:s'`. Nếu bạn cần tùy biến định dạng timestamp này, hãy set thuộc tính `$dateFormat` trên model của bạn. Thuộc tính này sẽ cho biết cách mà các thuộc tính date được lưu vào trong cơ sở dữ liệu:

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

<a name="attribute-casting"></a>
## Attribute Casting

Thuộc tính `$casts` trên model của bạn cung cấp một phương thức thuận tiện để chuyển đổi các thuộc tính thành các kiểu dữ liệu phổ biến. Thuộc tính `$casts` phải là một mảng trong đó khóa là tên của thuộc tính mà bạn muốn được chuyển và giá trị là loại bạn muốn chuyển đổi đến. Các kiểu cast được hỗ trợ là: `integer`, `real`, `float`, `double`, `decimal:<digits>`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime`, và `timestamp`. Khi chuyển sang loại `decimal`, bạn phải định nghĩa số ký tự, ví dụ như: (`decimal:2`).

Để ví dụ cho việc cast kiểu, hãy chuyển thuộc tính `is_admin`, được lưu trong cơ sở dữ liệu của chúng ta dưới dạng một số integer (`0` và `1`) thành giá trị boolean:

    <?php

    namespace App;

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

Bây giờ, thuộc tính `is_admin` sẽ luôn được chuyển thành giá trị boolean khi bạn truy cập đến nó, ngay cả khi giá trị được lưu trữ trong cơ sở dữ liệu dưới dạng một số integer:

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

> {note} Các thuộc tính `null` sẽ không được cast. Ngoài ra, bạn đừng bao giờ định nghĩa một cast (hoặc một thuộc tính) có cùng tên với tên của một quan hệ.

<a name="custom-casts"></a>
### Custom Casts

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

    namespace App;

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

#### Value Object Casting

Bạn không bị giới hạn trong việc cast giá trị cho các kiểu nguyên thủy. Bạn cũng có thể cast một giá trị cho các đối tượng. Việc định nghĩa các cast tùy chỉnh để cast một giá trị cho các đối tượng rất giống với việc cast kiểu nguyên thủy; tuy nhiên, phương thức `set` sẽ trả về một mảng các cặp khóa và giá trị sẽ được sử dụng để set các giá trị thô, và lưu trữ vào model.

Ví dụ, chúng ta sẽ định nghĩa một class cast tùy chỉnh truyền nhiều giá trị model vào một đối tượng giá trị `Address`. Chúng ta sẽ giả sử giá trị `Address` có hai thuộc tính công khai là : `lineOne` và `lineTwo`:

    <?php

    namespace App\Casts;

    use App\Address;
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
         * @return \App\Address
         */
        public function get($model, $key, $value, $attributes)
        {
            return new Address(
                $attributes['address_line_one'],
                $attributes['address_line_two']
            );
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  \App\Address  $value
         * @param  array  $attributes
         * @return array
         */
        public function set($model, $key, $value, $attributes)
        {
            if (! $value instanceof Address) {
                throw new InvalidArgumentException('The given value is not an Address instance.');
            }

            return [
                'address_line_one' => $value->lineOne,
                'address_line_two' => $value->lineTwo,
            ];
        }
    }

Khi cast các giá trị của đối tượng, mọi thay đổi được thực hiện đối với giá trị của đối tượng sẽ được tự động đồng bộ trở lại model trước khi model được lưu:

    $user = App\User::find(1);

    $user->address->lineOne = 'Updated Address Value';

    $user->save();

> {tip} Nếu bạn muốn chuyển đổi các model Eloquent của bạn chứa các giá trị của đối tượng thành JSON hoặc một mảng, bạn nên implement interface `Illuminate\Contracts\Support\Arrayable` và interface `JsonSerializable` trên giá trị của đối tượng.

#### Inbound Casting

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

#### Cast Parameters

Khi gắn một cast tùy chỉnh vào một model, các tham số cast có thể được chỉ định bằng cách tách chúng ra khỏi tên class bằng ký tự `:` và phân cách bằng dấu phẩy cho nhiều tham số khác nhau. Các tham số này sẽ được truyền cho hàm khởi tạo của class cast:

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'secret' => Hash::class.':sha256',
    ];

#### Castables

Thay vì gắn một cast tùy chỉnh vào model của bạn, bạn có thể gắn theo một cách khác là gắn vào một class implement interface `Illuminate\Contracts\Database\Eloquent\Castable`:

    protected $casts = [
        'address' => \App\Address::class,
    ];

Các đối tượng implement interface `Castable` phải định nghĩa một phương thức `castUsing` trả về tên class của class caster tùy chỉnh chịu trách nhiệm cast đến và đi từ class `Castable`:

    <?php

    namespace App;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use App\Casts\Address as AddressCast;

    class Address implements Castable
    {
        /**
         * Get the name of the caster class to use when casting from / to this cast target.
         *
         * @return string
         */
        public static function castUsing()
        {
            return AddressCast::class;
        }
    }

Khi sử dụng các class `Castable`, bạn vẫn có thể truyền các tham số trong định nghĩa `$casts`. Các tham số này sẽ được truyền trực tiếp đến class caster:

    protected $casts = [
        'address' => \App\Address::class.':argument',
    ];

<a name="array-and-json-casting"></a>
### Array và JSON Casting

Kiểu cast `array` đặc biệt hữu ích khi làm việc với các cột được lưu trữ dưới dạng JSON. Ví dụ: nếu cơ sở dữ liệu của bạn có một cột kiểu `JSON` hoặc `TEXT` chứa một chuỗi JSON, thì việc thêm `array` vào thuộc tính đó sẽ tự động giải mã các thuộc tính JSON đó thành mảng PHP khi bạn truy cập vào thuộc tính đó trên model Eloquent của bạn:

    <?php

    namespace App;

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

Khi cast đã được định nghĩa xong, bạn có thể truy cập vào thuộc tính `options` và nó sẽ tự động được giải mã thuộc tính từ JSON vào một mảng PHP. Khi bạn set giá trị cho thuộc tính `options`, thì mảng sẽ tự động được chuyển đổi trở lại thành JSON để lưu trữ:

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();

<a name="date-casting"></a>
### Date Casting

Khi sử dụng kiểu `date` hoặc `datetime`, bạn có thể chỉ định định dạng của date đó. Định dạng này sẽ được sử dụng khi [model được chuyển hóa thành một mảng hoặc một chuỗi JSON](/docs/{{version}}/eloquent-serialization):

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];

<a name="query-time-casting"></a>
### Query Time Casting

Thỉnh thoảng bạn có thể cần phải áp dụng các cast trong khi thực hiện một query, chẳng hạn như khi chọn một giá trị thô từ một bảng. Ví dụ: hãy xem xét query sau:

    use App\Post;
    use App\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

Thuộc tính `last_posted_at` trên kết quả của query này sẽ là một chuỗi thô. Sẽ rất tiện lợi nếu chúng ta có thể áp dụng một cast `date` cho thuộc tính này khi thực hiện query. Để thực hiện điều này, chúng ta có thể sử dụng phương thức `withCasts`:

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'datetime'
    ])->get();
