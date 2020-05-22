# Eloquent: Mutators

- [Giới thiệu](#introduction)
- [Accessor và Mutator](#accessors-and-mutators)
    - [Định nghĩa một Accessor](#defining-an-accessor)
    - [Định nghĩa một Mutator](#defining-a-mutator)
- [Date Mutator](#date-mutators)
- [Attribute Casting](#attribute-casting)
    - [Array và JSON Casting](#array-and-json-casting)

<a name="introduction"></a>
## Giới thiệu

Accessor và Mutator cho phép bạn định dạng các giá trị thuộc tính của Eloquent khi bạn lấy ra hoặc set chúng trên các instance model. Ví dụ: bạn có thể muốn sử dụng [Laravel encrypter](/docs/{{version}}/encryption) để mã hóa một giá trị trong khi lưu nó vào trong cơ sở dữ liệu, sau đó tự động giải mã nó khi bạn truy cập vào Eloquent model.

Ngoài các tuỳ biến accessors và mutator, Eloquent cũng có thể tự động chuyển các trường kiểu date thành các trường hợp kiểu [Carbon](https://github.com/briannesbitt/Carbon) hoặc thậm chí [chuyển trường text sang JSON](#attribute-casting).

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

Như bạn có thể thấy, giá trị ban đầu của cột được pass cho accessor, cho phép bạn thao tác và trả về một giá trị khác. Để truy cập vào giá trị của accessor, bạn có thể truy cập vào thuộc tính `first_name` trên một instance model:

    $user = App\User::find(1);

    $firstName = $user->first_name;

Tất nhiên, bạn cũng có thể sử dụng accessor để trả về các giá trị mới, được tính toán từ các thuộc tính hiện có:

    /**
     * Get the user's full name.
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

<a name="defining-a-mutator"></a>
### Định nghĩa một Mutator

Để định nghĩa một mutator, hãy định nghĩa một phương thức `setFooAttribute` trên model của bạn trong đó `Foo` là tên được set theo kiểu "studly" của cột mà bạn muốn truy cập. Vì thế, một lần nữa, hãy định nghĩa một mutator cho thuộc tính `first_name`. Mutator này sẽ được gọi tự động khi chúng ta set một giá trị cho thuộc tính `first_name` trên model:

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

Mutator sẽ nhận vào giá trị đang được set cho thuộc tính, cho phép bạn thao tác với giá trị đó và set một giá trị mới cho thuộc tính `$attributes` bên trong của model Eloquent. Vì thế, ví dụ, nếu chúng ta set thuộc tính `first_name` là `Sally`:

    $user = App\User::find(1);

    $user->first_name = 'Sally';

Trong ví dụ này, hàm `setFirstNameAttribute` sẽ được gọi với giá trị `Sally`. Mutator sẽ dùng hàm `strtolower` cho giá trị vào và set giá trị kết quả cho mảng `$attributes` bên trong.

<a name="date-mutators"></a>
## Date Mutator

Mặc định, Eloquent sẽ chuyển đổi các cột `created_at` và `update_at` thành các instance của [Carbon](https://github.com/briannesbitt/Carbon), đây là một class được extend từ class `DateTime` của PHP, để cung cấp các phương thức hữu ích . Bạn có thể tùy biến những date nào sẽ bị chuyển đổi hay thậm chí disable những chuyển đổi này, bằng cách ghi đè thuộc tính `$dates` trong model của bạn:

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
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }

Khi một cột được coi là một date, bạn có thể set giá trị của nó thành một UNIX timestamp hoặc một date string (`Y-m-d`) hoặc là date-time string, và tất nhiên là một instance `DateTime` / `Carbon` và các giá trị của date này sẽ tự động được lưu trữ chính xác trong cơ sở dữ liệu của bạn:

    $user = App\User::find(1);

    $user->deleted_at = now();

    $user->save();

Như đã lưu ý ở trên, khi lấy các thuộc tính được liệt kê trong thuộc tính `$dates` của bạn, chúng sẽ tự động được chuyển sang thành các instance [Carbon](https://github.com/briannesbitt/Carbon), cho phép bạn sử dụng bất kỳ phương thức nào của Carbon trên thuộc tính của bạn:

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### Date Formats

Mặc định, timestamp sẽ được định dạng là `'Y-m-d H:i:s'`. Nếu bạn cần tùy biến định dạng timestamp này, hãy set thuộc tính `$dateFormat` trên model của bạn. Thuộc tính này xác định cách mà các thuộc tính date được lưu trữ vào trong cơ sở dữ liệu, cũng như định dạng của chúng khi model được serialize thành một mảng hoặc JSON:

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

Thuộc tính `$casts` trên model của bạn cung cấp một phương thức thuận tiện để chuyển đổi các thuộc tính thành các kiểu dữ liệu phổ biến. Thuộc tính `$casts` phải là một mảng trong đó khóa là tên của thuộc tính muốn được chuyển và giá trị là loại bạn muốn chuyển đổi. Các kiểu cast được hỗ trợ là: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime`, và `timestamp`.

Ví dụ: hãy chuyển thuộc tính `is_admin`, được lưu trữ trong cơ sở dữ liệu của chúng ta dưới dạng một số integer (`0` và `1`) thành giá trị boolean:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
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

<a name="array-and-json-casting"></a>
### Array và JSON Casting

Kiểu cast `array` đặc biệt hữu ích khi làm việc với các cột được lưu trữ dưới dạng JSON được serialize. Ví dụ: nếu cơ sở dữ liệu của bạn có một trường kiểu `JSON` hoặc `TEXT` chứa JSON được serialize, việc thêm `array` vào thuộc tính đó sẽ tự động giải mã thuộc tính serialize vào mảng PHP khi bạn truy cập vào model Eloquent của bạn:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

Khi cast đã được định nghĩa, bạn có thể truy cập thuộc tính `options` và nó sẽ tự động được giải mã serialize từ JSON thành một mảng PHP. Khi bạn set giá trị cho thuộc tính `options`, thì mảng sẽ tự động được serialize trở lại thành JSON để lưu trữ:

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
