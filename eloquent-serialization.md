# Eloquent: Serialization

- [Giới thiệu](#introduction)
- [Serialize Model và Collection](#serializing-models-and-collections)
    - [Serialize vào Array](#serializing-to-arrays)
    - [Serialize vào JSON](#serializing-to-json)
- [Ẩn thuộc tính từ JSON](#hiding-attributes-from-json)
- [Thêm giá trị vào JSON](#appending-values-to-json)
- [Date Serialization](#date-serialization)

<a name="introduction"></a>
## Giới thiệu

Khi xây dựng một API mà dùng Laravel, bạn thường sẽ cần phải chuyển đổi các model và các quan hệ của bạn thành các mảng hoặc JSON. Eloquent có chứa các phương thức thuận tiện để thực hiện các chuyển đổi này, cũng như kiểm soát các thuộc tính nào sẽ được thêm vào trong các chuyển đổi representation of your models.

> **Note**
> Để biết cách xử lý chuyển hóa JSON của collection và model Eloquent hiệu quả hơn nữa, hãy xem tài liệu về [resource API Eloquent](/docs/{{version}}/eloquent-resources).

<a name="serializing-models-and-collections"></a>
## Serialize Model và Collection

<a name="serializing-to-arrays"></a>
### Serialize vào Array

Để chuyển đổi một model và [quan hệ](/docs/{{version}}/eloquent-relationships) của nó thành một mảng, bạn nên sử dụng phương thức `toArray`. Phương thức này là phương thức đệ quy, vì vậy tất cả các thuộc tính và các quan hệ (bao gồm cả quan hệ của quan hệ) cũng sẽ được chuyển đổi thành mảng:

    use App\Models\User;

    $user = User::with('roles')->first();

    return $user->toArray();

Phương thức `attributesToArray` có thể được sử dụng để chuyển đổi các thuộc tính của model thành một mảng không có các quan hệ của nó:

    $user = User::first();

    return $user->attributesToArray();

Bạn cũng có thể chuyển đổi toàn bộ [collection](/docs/{{version}}/eloquent-collections) của các model thành một mảng bằng cách gọi phương thức `toArray` trong instance collection:

    $users = User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### Serialize vào JSON

Để chuyển đổi một model thành dạng JSON, bạn có thể sử dụng phương thức `toJson`. Giống như phương thức `toArray`, phương thức` toJson` cũng là phương thức đệ quy, nên tất cả các thuộc tính và các quan hệ sẽ được chuyển đổi thành dạng JSON. Ngoài ra, bạn cũng có thể chỉ định thêm bất kỳ tùy chọn mã hóa JSON nào mà [được hỗ trợ bởi PHP](https://secure.php.net/manual/en/function.json-encode.php):

    use App\Models\User;

    $user = User::find(1);

    return $user->toJson();

    return $user->toJson(JSON_PRETTY_PRINT);

Ngoài ra, bạn có thể cast một model hoặc một collection thành một chuỗi, nó cũng sẽ được tự động gọi phương thức `toJson` trên các model hoặc các collection đó của bạn:

    return (string) User::find(1);

Vì các model và các collection sẽ được chuyển đổi thành dạng JSON khi bị cast thành một chuỗi, nên bạn có thể trả về các đối tượng Eloquent trực tiếp từ các route hoặc các controller của application của bạn. Laravel sẽ tự động chuyển hóa các model và collection Eloquent của bạn thành JSON khi chúng được trả về từ các route hoặc controller:

    Route::get('users', function () {
        return User::all();
    });

<a name="relationships"></a>
#### Relationships

Khi một model Eloquent được chuyển đổi thành JSON, các quan hệ mà đã được load của model đó cũng sẽ bị tự động chuyển đổi và đưa vào làm một thuộc tính trên đối tượng JSON. Ngoài ra, mặc dù các phương thức quan hệ của Eloquent được định nghĩa bằng quy tắc đặt tên "camel case", nhưng khi chuyển đổi thuộc tính JSON của quan hệ sẽ bị chuyển thành "snake case".

<a name="hiding-attributes-from-json"></a>
## Ẩn thuộc tính từ JSON

Thỉnh thoảng bạn cũng có thể muốn giới hạn các thuộc tính, chẳng hạn như mật khẩu, được chứa trong mảng hoặc JSON của model của bạn. Để làm như vậy, hãy thêm một thuộc tính `$hidden` vào model của bạn. Các thuộc tính được liệt kê trong mảng của thuộc tính `$hidden` sẽ không được chuyển đổi khi model của bạn được chuyển đổi:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> **Note**
> Để ẩn các quan hệ, hãy thêm tên phương thức của quan hệ đó vào thuộc tính `$hidden` của model Eloquent của bạn.

Ngoài ra, bạn có thể sử dụng thuộc tính `visible` để định nghĩa một danh sách các thuộc tính có thể hiển thị trong mảng hoặc JSON của bạn. Tất cả các thuộc tính không có mặt trong mảng `$visible` sẽ bị ẩn khi model được chuyển đổi thành một mảng hoặc một JSON:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be visible in arrays.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

<a name="temporarily-modifying-attribute-visibility"></a>
#### Temporarily Modifying Attribute Visibility

Nếu bạn muốn một số thuộc tính bị ẩn được hiển thị trên một instance model, bạn có thể sử dụng phương thức `makeVisible`. Phương thức `makeVisible` sẽ trả về instance của model:

    return $user->makeVisible('attribute')->toArray();

Tương tự, nếu bạn muốn ẩn một số thuộc tính thường được hiển thị, bạn có thể sử dụng phương thức `makeHidden`.

    return $user->makeHidden('attribute')->toArray();

Nếu bạn muốn tạm thời ghi đè tất cả các thuộc tính ẩn hoặc hiển thị, bạn có thể sử dụng các phương thức `setVisible` và `setHidden` tương ứng:

    return $user->setVisible(['id', 'name'])->toArray();

    return $user->setHidden(['email', 'password', 'remember_token'])->toArray();

<a name="appending-values-to-json"></a>
## Thêm giá trị vào JSON

Đôi khi, khi chuyển đổi một model thành một mảng hoặc một JSON, bạn có thể muốn thêm các thuộc tính mà không có cột tương ứng trong cơ sở dữ liệu. Để làm như vậy, trước tiên, hãy định nghĩa một [accessor](/docs/{{version}}/eloquent-mutators) cho thuộc tính đó:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Determine if the user is an administrator.
         *
         * @return \Illuminate\Database\Eloquent\Casts\Attribute
         */
        protected function isAdmin(): Attribute
        {
            return new Attribute(
                get: fn () => 'yes',
            );
        }
    }

Sau khi tạo accessor xong, hãy thêm tên thuộc tính đó vào thuộc tính `appends` của model của bạn. Lưu ý rằng tên thuộc tính thường được tham chiếu thường sử dụng quy ước "snake case", mặc dù phương phức PHP của accessor được định nghĩa theo kiểu "camel case":

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The accessors to append to the model's array form.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

Khi thuộc tính đã được thêm vào mảng `appends` xong, nó sẽ được hiển thị trong JSON hoặc mảng của model. Các thuộc tính trong mảng `appends` cũng sẽ sử dụng được các cài đặt `visible` và `hidden` trên model.

<a name="appending-at-run-time"></a>
#### Appending At Run Time

Trong lúc chạy, bạn cũng có thể chỉ dẫn instance model thêm các thuộc tính bổ sung bằng cách sử dụng phương thức `append`. Hoặc, bạn có thể sử dụng phương thức `setAppends` để ghi đè toàn bộ mảng thuộc tính sẽ được thêm vào một instance model đã cho:

    return $user->append('is_admin')->toArray();

    return $user->setAppends(['is_admin'])->toArray();

<a name="date-serialization"></a>
## Date Serialization

<a name="customizing-the-default-date-format"></a>
#### Customizing The Default Date Format

Bạn có thể tùy chỉnh định dạng chuyển đổi mặc định bằng cách ghi đè phương thức `serializeDate`. Phương thức này không ảnh hưởng đến cách định dạng ngày của bạn khi lưu trong cơ sở dữ liệu:

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

<a name="customizing-the-date-format-per-attribute"></a>
#### Customizing The Date Format Per Attribute

Bạn có thể tùy chỉnh định dạng chuyển đổi của từng thuộc tính date trong Eloquent bằng cách chỉ định định dạng date trong [khai báo](/docs/{{version}}/eloquent-mutators#attribute-casting) của model đó:

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];
