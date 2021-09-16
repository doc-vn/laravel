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

Khi xây dựng API JSON, bạn thường sẽ cần phải chuyển đổi các model và quan hệ của bạn thành các mảng hoặc JSON. Eloquent có chứa các phương thức thuận tiện để thực hiện các chuyển đổi này, cũng như kiểm soát các thuộc tính nào sẽ được chứa trong các chuyển đổi của bạn.

<a name="serializing-models-and-collections"></a>
## Serialize Model và Collection

<a name="serializing-to-arrays"></a>
### Serialize vào Array

Để chuyển đổi một model và [quan hệ](/docs/{{version}}/eloquent-relationships) của nó thành một mảng, bạn nên sử dụng phương thức `toArray`. Phương thức này là đệ quy, vì vậy tất cả các thuộc tính và tất cả các quan hệ (bao gồm cả quan hệ của quan hệ) cũng sẽ được chuyển đổi thành mảng:

    $user = App\User::with('roles')->first();

    return $user->toArray();

Bạn cũng có thể chuyển đổi toàn bộ [collection](/docs/{{version}}/eloquent-collections) của các model thành mảng:

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### Serialize vào JSON

Để chuyển đổi một model thành JSON, bạn nên sử dụng phương thức `toJson`. Giống như `toArray`, phương thức` toJson` cũng là đệ quy, vì vậy tất cả các thuộc tính và quan hệ sẽ được chuyển đổi thành JSON:

    $user = App\User::find(1);

    return $user->toJson();

Ngoài ra, bạn có thể cast một model hoặc một collection thành một chuỗi, nó sẽ tự động gọi phương thức `toJson` trên model hoặc collection của bạn:

    $user = App\User::find(1);

    return (string) $user;

Vì các model và collection sẽ được chuyển đổi thành JSON khi cast thành một chuỗi, nên bạn có thể trả về các đối tượng Eloquent trực tiếp từ các route hoặc controller của application của bạn:

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## Ẩn thuộc tính từ JSON

Thỉnh thoảng bạn có thể muốn giới hạn các thuộc tính, chẳng hạn như mật khẩu, được chứa trong mảng hoặc JSON của model của bạn. Để làm như vậy, hãy thêm một thuộc tính `$hidden` vào model của bạn:

    <?php

    namespace App;

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

> {note} Khi ẩn các quan hệ, thì sử dụng tên phương thức của các quan hệ đó.

Ngoài ra, bạn có thể sử dụng thuộc tính `visible` để định nghĩa một danh sách các thuộc tính có thể có trong mảng hoặc JSON của bạn. Tất cả các thuộc tính khác sẽ bị ẩn khi model được chuyển đổi thành một mảng hoặc một JSON:

    <?php

    namespace App;

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

#### Temporarily Modifying Attribute Visibility

Nếu bạn muốn làm cho một số thuộc tính bị ẩn được hiển thị trên một instance model, bạn có thể sử dụng phương thức `makeVisible`. Phương thức `makeVisible` sẽ trả về một instance model giúp cho việc kết hợp với các phương thức khác thuận tiện hơn:

    return $user->makeVisible('attribute')->toArray();

Tương tự, nếu bạn muốn làm cho một số thuộc tính thường được hiển thị bị ẩn trong một instance model, bạn có thể sử dụng phương thức `makeHidden`.

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## Thêm giá trị vào JSON

Đôi khi, khi cast một model thành một mảng hoặc một JSON, bạn có thể muốn thêm các thuộc tính mà không có cột tương ứng trong cơ sở dữ liệu của bạn. Để làm như vậy, trước tiên, hãy định nghĩa một [accessor](/docs/{{version}}/eloquent-mutators) cho thuộc tính đó:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the administrator flag for the user.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

Sau khi tạo accessor xong, hãy thêm tên thuộc tính đó vào thuộc tính `appends` trên model. Lưu ý rằng tên thuộc tính thường được tham chiếu theo kiểu "snake case", mặc dù accessor được định nghĩa theo kiểu "camel case":

    <?php

    namespace App;

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

Khi thuộc tính đã được thêm vào mảng `appends`, nó sẽ được hiển thị trong JSON hoặc mảng của model. Các thuộc tính trong mảng `appends` cũng sẽ sử dụng các cài đặt `visible` và `hidden` đã được cấu hình trên model.

#### Appending At Run Time

Bạn cũng có thể chỉ dẫn instance model thêm các thuộc tính bằng cách sử dụng phương thức `append`. Hoặc, bạn có thể sử dụng phương thức `setAppends` để ghi đè toàn bộ mảng thuộc tính sẽ được thêm vào một instance model đã cho:

    return $user->append('is_admin')->toArray();

    return $user->setAppends(['is_admin'])->toArray();

<a name="date-serialization"></a>
## Date Serialization

Laravel extend thư viện [Carbon] (https://github.com/briannesbitt/Carbon) date để cung cấp các tùy biến thuận tiện cho định dạng chuyển đổi JSON . Để tùy biến cách chuyển đổi của tất cả các Carbon date trong application của bạn, hãy sử dụng phương thức `Carbon::serializeUsing`. Phương thức `serializeUsing` chấp nhận một Closure trả về một chuỗi hiển thị cho một date để chuyển nó sang JSON:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Carbon;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Carbon::serializeUsing(function ($carbon) {
                return $carbon->format('U');
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
