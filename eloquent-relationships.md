# Eloquent: Relationships

- [Giới thiệu](#introduction)
- [Định nghĩa quan hệ](#defining-relationships)
    - [Một - Một](#one-to-one)
    - [Một - Nhiều](#one-to-many)
    - [Một - Nhiều (Ngược lại)](#one-to-many-inverse)
    - [Nhiều - Nhiều](#many-to-many)
    - [Quan hệ thông qua trung gian](#has-many-through)
    - [Quan hệ đa hình](#polymorphic-relations)
    - [Quan hệ đa hình Nhiều - Nhiều](#many-to-many-polymorphic-relations)
- [Query theo quan hệ](#querying-relations)
    - [Phương thức quan hệ và thuộc tính động](#relationship-methods-vs-dynamic-properties)
    - [Query quan hệ tồn tại](#querying-relationship-existence)
    - [Query quan hệ không tồn tại](#querying-relationship-absence)
    - [Đếm các bản ghi theo quan hệ model](#counting-related-models)
- [Eager Loading](#eager-loading)
    - [Rằng buộc khi eager loading](#constraining-eager-loads)
    - [Lazy Eager Loading](#lazy-eager-loading)
- [Thêm và cập nhật theo quan hệ model](#inserting-and-updating-related-models)
    - [Phương thức save](#the-save-method)
    - [Phương thức create](#the-create-method)
    - [Quan hệ thuộc về](#updating-belongs-to-relationships)
    - [Quan hệ Nhiều - Nhiều](#updating-many-to-many-relationships)
- [Sửa timestamp của model cha](#touching-parent-timestamps)

<a name="introduction"></a>
## Giới thiệu

Các bảng cơ sở dữ liệu thường được quan hệ với nhau. Ví dụ: một bài post trên một blog có thể có nhiều comment hoặc một order có thể quan hệ với người dùng đã đặt nó. Eloquent giúp quản lý và làm việc với những quan hệ này dễ dàng hơn và hỗ trợ một số loại quan hệ khác nhau:

- [Một - Một](#one-to-one)
- [Một - Nhiều](#one-to-many)
- [Nhiều - Nhiều](#many-to-many)
- [Quan hệ thông qua trung gian](#has-many-through)
- [Quan hệ đa hình](#polymorphic-relations)
- [Quan hệ đa hình Nhiều - Nhiều](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## Định nghĩa quan hệ

Các quan hệ của Eloquent là các phương thức được định nghĩa nằm trong các class của model Eloquent. Cũng giống như các model Eloquent, các quan hệ cũng đóng vai trò như là các [query builder](/docs/{{version}}/queries), nó định nghĩa các quan hệ như là các phương thức cung cấp khả năng kết hợp và truy vấn mạnh mẽ. Ví dụ, chúng ta có thể kết hợp các ràng buộc cho quan hệ `posts` như thế này:

    $user->posts()->where('active', 1)->get();

Nhưng, trước khi đi sâu vào việc sử dụng các quan hệ, hãy tìm hiểu cách định nghĩa của từng loại quan hệ.

<a name="one-to-one"></a>
### Một - Một

Một quan hệ một-một là một quan hệ rất cơ bản. Ví dụ, một model `User` có thể quan hệ với một  `Phone`. Để định nghĩa quan hệ này, chúng ta hăy set một phương thức `phone` trên model `User`. Phương thức `phone` nên gọi phương thức `hasOne` và trả về chính phương thức đó:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the phone record associated with the user.
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

Tham số đầu tiên được truyền vào cho phương thức `hasOne` là tên của model được quan hệ. Khi quan hệ đã được định nghĩa, chúng ta có thể lấy ra các bản ghi theo quan hệ bằng các thuộc tính động của Eloquent. Các thuộc tính động cho phép bạn truy cập vào các phương thức quan hệ như thể chúng là các thuộc tính được định nghĩa trên model:

    $phone = User::find(1)->phone;

Eloquent sẽ xác định khóa ngoại của quan hệ dựa trên tên model. Trong trường hợp này, model `Phone` tự động được giả định là có khóa ngoại `user_id`. Nếu bạn muốn ghi đè quy ước này, bạn có thể truyền tham số thứ hai vào phương thức `hasOne`:

    return $this->hasOne('App\Phone', 'foreign_key');

Ngoài ra, Eloquent cũng giả định rằng khóa ngoại phải có giá trị trùng khớp với giá trị cột `id` (hoặc cột tuỳ biến `$primaryKey`) của model cha. Nói cách khác, Eloquent sẽ tìm giá trị của `id` của user trong cột `user_id` trong bảng `Phone`. Nếu bạn muốn quan hệ này sử dụng một giá trị khác ngoài `id`, bạn có thể truyền vào một tham số thứ ba cho phương thức `hasOne` khai báo khóa tùy biến của bạn:

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### Defining The Inverse Of The Relationship

Vì vậy, chúng ta có thể truy cập model `Phone` từ model `User`. Bây giờ, hãy định nghĩa quan hệ trên model `Phone`, sẽ cho phép chúng ta truy cập ngược lại vào model `User` sở hữu phone đó. Chúng ta có thể định nghĩa một quan hệ ngược lại của `hasOne` bằng cách sử dụng phương thức `belongsTo`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

Trong ví dụ trên, Eloquent sẽ cố gắng tìm `user_id` từ model `Phone` với một giá trị `id` trên model `User`. Eloquent sẽ xác định tên mặc định của khóa ngoại bằng cách lấy tên của phương thức quan hệ và thêm hậu tố `_id`. Tuy nhiên, nếu khóa ngoại trên model `Phone` không phải là `user_id`, bạn có thể truyền một tên khóa khác làm tham số thứ hai cho phương thức `belongsTo`:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

Nếu model cha của bạn không sử dụng `id` làm khóa chính của nó hoặc bạn muốn join model con vào một cột khác, bạn có thể truyền một tham số thứ ba cho phương `belongsTo` khai báo khóa tùy biến của bảng cha của bạn:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="default-models"></a>
#### Default Models

Quan hệ `belongsTo` cho phép bạn định nghĩa một model mặc định sẽ được trả về nếu trong trường hợp quan hệ đó là `null`. Trường hợp này thường được gọi là [trường hợp đối tượng null](https://en.wikipedia.org/wiki/Null_Object_pattern) và có thể giúp xoá các câu lệnh kiểm tra có trong code của bạn. Trong ví dụ sau, quan hệ `user` sẽ trả về một model `App\User` trống nếu không có `user` nào đi kèm với post:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }

Để thêm các thuộc tính cho model mặc định, bạn có thể truyền vào một mảng hoặc một Closure cho phương thức `withDefault`:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => 'Guest Author',
        ]);
    }

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user) {
            $user->name = 'Guest Author';
        });
    }

<a name="one-to-many"></a>
### Một - Nhiều

Quan hệ "một-nhiều" được sử dụng để định nghĩa các quan hệ mà trong đó một model sở hữu nhiều model khác. Ví dụ, một post trên blog có thể có nhiều comment. Giống như tất cả các quan hệ Eloquent khác, quan hệ một-nhiều được định nghĩa bằng cách set một hàm trên model Eloquent của bạn:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

Hãy nhớ rằng, Eloquent sẽ tự động xác định cột khóa ngoại phù hợp trên model `Comment`. Theo quy ước, Eloquent sẽ lấy tên theo dạng "snake case" của model cha và set thêm hậu tố là `_id`. Vì vậy, trong ví dụ này, Eloquent sẽ giả định rằng khóa ngoại trên model `Comment` là `post_id`.

Khi quan hệ đã được định nghĩa, chúng ta có thể truy cập vào collection comment bằng cách truy cập thông qua thuộc tính `comments`. Hãy nhớ rằng, vì Eloquent cung cấp "các thuộc tính động", nên chúng ta có thể truy cập các phương thức quan hệ như thể chúng được định nghĩa là các thuộc tính trên model:

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

Tất nhiên, vì tất cả các quan hệ cũng đóng vai trò như là một query builder, nên bạn có thể thêm các ràng buộc cho những comment được lấy ra bằng cách gọi phương thức `comments` và tiếp tục thêm các điều kiện vào truy vấn:

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

Giống như phương thức `hasOne`, bạn cũng có thể ghi đè các khóa ngoại và khóa local bằng cách truyền thêm các tham số bổ sung cho phương thức `hasMany`:

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### Một - Nhiều (Ngược lại)

Bây giờ chúng ta có thể truy cập tất cả các comment của một post, tiếp theo hãy định nghĩa một quan hệ để cho phép từ comment có thể truy cập ngược lại vào post của chính nó. Để định nghĩa một nghịch đảo của quan hệ `hasMany`, hãy định nghĩa một hàm quan hệ trên model con gọi đến phương thức `belongsTo`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get the post that owns the comment.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Khi quan hệ đã được định nghĩa, chúng ta có thể lấy ra model `Post` từ một `Comment` bằng cách truy cập vào "thuộc tính động" `post`:

    $comment = App\Comment::find(1);

    echo $comment->post->title;

Trong ví dụ trên, Eloquent sẽ cố gắng tìm `post_id` từ model `Comment` với một giá trị `id` trên model `Post`. Eloquent sẽ xác định tên mặc định của khóa ngoại bằng cách lấy tên của phương thức quan hệ và thêm hậu tố `_id`. Tuy nhiên, nếu khóa ngoại trên model `Comment` không phải là `post_id`, bạn có thể truyền một tên khóa ngoại khác làm tham số thứ hai cho phương thức `belongsTo`:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

Nếu model cha của bạn không sử dụng `id` làm khóa chính của nó hoặc bạn muốn join model con vào một cột khác, bạn có thể truyền vào một tham số thứ ba cho phương `belongsTo` khai báo khóa tùy biến của bảng cha của bạn:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### Nhiều - Nhiều

Quan hệ nhiều-nhiều sẽ phức tạp hơn một chút so với các quan hệ `hasOne` và `hasMany`. Một ví dụ về quan hệ kiểu như vậy là một user có thể có nhiều role, trong đó các role cũng có thể được chia cho nhiều user khác. Ví dụ: nhiều user có thể có role là "Admin". Để định nghĩa quan hệ này, cần có ba bảng cơ sở dữ liệu: `users`, `roles`, và `role_user`. Tên bảng `role_user` sẽ được lấy theo thứ tự chữ cái của tên các model và có chứa các cột `user_id` và` Role_id`.

Quan hệ nhiều-nhiều được định nghĩa bằng cách viết một phương thức để trả về phương thức `belongsToMany`. Ví dụ: hãy định nghĩa phương thức `roles` trên model `User` của chúng ta:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

Khi quan hệ đã được định nghĩa, bạn có thể truy cập các role từ một user bằng cách sử dụng thuộc tính động `roles`:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

Tất nhiên, giống như tất cả các loại quan hệ khác, bạn có thể gọi phương thức `roles` để tiếp tục thêm các ràng buộc truy vấn trên quan hệ:

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

Như đã đề cập trước đó, để xác định tên của bảng join các quan hệ, Eloquent sẽ nối tên hai model có quan hệ với nhau theo thứ tự bảng chữ cái. Tuy nhiên, bạn có thể tự do ghi đè lên quy ước này. Bạn có thể làm như vậy bằng cách truyền một tham số thứ hai cho phương thức `belongsToMany`:

    return $this->belongsToMany('App\Role', 'role_user');

Ngoài việc tùy biến tên của bảng join, bạn cũng có thể tùy biến tên cột của các khóa trên bảng đó bằng cách truyền thêm các tham số bổ sung cho phương thức `belongsToMany`. Tham số thứ ba là tên khóa ngoại của model mà bạn đang định nghĩa quan hệ, trong khi tham số thứ tư là tên khóa ngoại của model mà bạn đang muốn join:

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### Defining The Inverse Of The Relationship

Để định nghĩa một nghịch đảo của quan hệ nhiều-nhiều, bạn hãy thực hiện một phương thức khác đến `belongsToMany` trên model quan hệ của bạn. Để tiếp tục ví dụ về user role của chúng ta, hãy định nghĩa phương thức `users` trên model `Role`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

Như bạn có thể thấy, quan hệ được định nghĩa sẽ giống với quan hệ trên model `User`, ngoại trừ việc tham chiếu là model `App\User`. Vì chúng ta đang sử dụng lại phương thức `belongsToMany`, nên tất cả các tùy chọn thông thường như tên bảng và khóa đều khả dụng khi định nghĩa nghịch đảo của các quan hệ nhiều-nhiều.

#### Retrieving Intermediate Table Columns

Như bạn đã biết, làm việc với các quan hệ nhiều-nhiều đòi hỏi phải có sự hiện diện của một bảng trung gian. Eloquent cung cấp một số cách tương tác rất hữu ích với bảng đó. Ví dụ: giả sử đối tượng `User` của chúng ta có nhiều đối tượng `Role` mà nó quan hệ. Sau khi truy cập quan hệ này, chúng ta có thể truy cập vào bảng trung gian bằng cách sử dụng thuộc tính `pivot` trên model:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

Lưu ý rằng mỗi model `Role` mà chúng ta lấy ra được sẽ tự động được gán một thuộc tính là `pivot`. Thuộc tính này chứa một model đại diện cho bảng trung gian và có thể được sử dụng như bất kỳ model Eloquent nào khác.

Mặc định, chỉ các khóa model sẽ xuất hiện trong đối tượng pivot`. Nếu bảng pivot của bạn chứa thêm các thuộc tính bổ sung, bạn phải khai báo chúng khi định nghĩa quan hệ:

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

Nếu bạn muốn bảng pivot của bạn tự động duy trì các cột timestamp `created_at` và `update_at`, hãy sử dụng phương thức` withTimestamps` trong định nghĩa quan hệ của bạn:

    return $this->belongsToMany('App\Role')->withTimestamps();

#### Customizing The `pivot` Attribute Name

Như đã lưu ý trước đó, các thuộc tính từ bảng trung gian có thể được truy cập trên các model sử dụng theo thuộc tính `pivot`. Tuy nhiên, bạn có thể tùy biến tên của thuộc tính này để phản ánh tốt hơn cho mục đích của nó trong application của bạn.

Ví dụ: nếu application của bạn chứa user có thể subscribe podcast, bạn có thể có một quan hệ nhiều-nhiều giữa user và podcast. Nếu đây là trường hợp đó, bạn có thể muốn đổi tên truy cập vào bảng trung gian của bạn thành `subscription` thay vì `pivot`. Điều này có thể được thực hiện bằng cách sử dụng phương thức `as` khi định nghĩa quan hệ của bạn:

    return $this->belongsToMany('App\Podcast')
                    ->as('subscription')
                    ->withTimestamps();

Khi điều này được thực hiện xong, bạn có thể truy cập vào dữ liệu của bảng trung gian bằng tên tùy biến:

    $users = User::with('podcasts')->get();

    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }

#### Filtering Relationships Via Intermediate Table Columns

Bạn cũng có thể lọc các kết quả được trả về bởi `belongsToMany` bằng cách sử dụng các phương thức `wherePivot` và `wherePivotIn` khi định nghĩa quan hệ:

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

#### Defining Custom Intermediate Table Models

Nếu bạn muốn định nghĩa một model tùy biến, để biểu diễn bảng trung gian của quan hệ của bạn, bạn có thể gọi phương thức `using` khi định nghĩa quan hệ. Tất cả các model tùy biến được sử dụng để biểu diễn các bảng quan hệ trung gian phải được extend từ class `Illuminate\Database\Eloquent\Relations\Pivot`. Ví dụ: chúng ta có thể định nghĩa một `Role` sử dụng model pivot `UserRole` tùy biến:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User')->using('App\UserRole');
        }
    }

Khi định nghĩa model `UserRole`, chúng ta sẽ extend nó từ class `Pivot`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class UserRole extends Pivot
    {
        //
    }

<a name="has-many-through"></a>
### Quan hệ thông qua trung gian

Quan hệ "trung gian" cung cấp một lối tắt thuận tiện để truy cập các quan hệ xa thông qua các quan hệ trung gian. Ví dụ, một model `Country` có thể có nhiều model `Post` thông qua một model `User` trung gian. Trong ví dụ này, bạn có thể dễ dàng thu thập tất cả các post trên blog cho một quốc gia nhất định. Hãy xem các bảng cần thiết để định nghĩa quan hệ này:

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

Mặc dù `posts` không chứa cột `country_id`, nhưng quan hệ `hasManyThrough` cung cấp quyền truy cập vào các post của một quốc gia thông qua `$country->posts`. Để thực hiện truy vấn này, Eloquent kiểm tra `country_id` trên bảng `users` trung gian. Sau khi tìm được các ID user phù hợp, chúng sẽ được sử dụng để truy vấn vào bảng `posts`.

Chúng ta đã xem qua cấu trúc bảng cho quan hệ này, bây giờ hãy định nghĩa nó trên model `Country`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

Tham số đầu tiên được truyền cho phương thức `hasManyThrough` là tên của model cuối cùng mà chúng ta muốn truy cập, trong khi tham số thứ hai là tên của model trung gian.

Các quy ước thông thường dành cho các khóa ngoại Eloquent cũng sẽ được sử dụng khi thực hiện các truy vấn quan hệ. Nếu bạn muốn tùy chỉnh các khóa của các quan hệ, bạn có thể truyền chúng làm tham số thứ ba và thứ tư cho phương thức `hasManyThrough`. Tham số thứ ba là tên của khóa ngoại trên model trung gian. Tham số thứ tư là tên của khóa ngoại trên model cuối cùng. Tham số thứ năm là khóa local, trong khi tham số thứ sáu là khóa local của model trung gian:

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post',
                'App\User',
                'country_id', // Foreign key on users table...
                'user_id', // Foreign key on posts table...
                'id', // Local key on countries table...
                'id' // Local key on users table...
            );
        }
    }

<a name="polymorphic-relations"></a>
### Quan hệ đa hình

#### Table Structure

Quan hệ đa hình cho phép một model thuộc về nhiều hơn một model khác trên một liên kết. Ví dụ: hãy tưởng tượng user trong application của bạn có thể "comment" cả post và video. Sử dụng các quan hệ đa hình, bạn có thể sử dụng một bảng `comments` cho cả hai tình huống này. Trước tiên, hãy xem cấu trúc bảng cần thiết để xây dựng quan hệ này:

    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

Hai cột quan trọng cần lưu ý ở đây là cột `commentable_id` và cột `commentable_type` trong bảng `comments`. Cột `commentable_id` sẽ chứa giá trị ID của post hoặc video, trong khi cột `commentable_type` sẽ chứa tên class của model sở hữu. Cột `commentable_type` là cách ORM xác định "loại" model nào sẽ trả về khi truy cập vào quan hệ `commentable`.

#### Model Structure

Tiếp theo, hãy xem các định nghĩa model cần thiết để xây dựng quan hệ này:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get all of the owning commentable models.
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get all of the post's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * Get all of the video's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### Retrieving Quan hệ đa hình

Khi bảng cơ sở dữ liệu và model của bạn đã được định nghĩa xong, bạn có thể truy cập vào các quan hệ thông qua các model của bạn. Ví dụ: để truy cập đến tất cả các comment cho một post, chúng ta có thể sử dụng thuộc tính động `comments`:

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

Bạn cũng có thể lấy ra chủ sở hữu của một quan hệ đa hình từ model đa hình bằng cách truy cập vào tên của phương thức thực hiện lệnh gọi tới `morphTo`. Trong trường hợp của chúng ta, đó là phương thức `commentable` trên model `Comment`. Vì vậy, chúng ta sẽ truy cập vào phương thức đó như là một thuộc tính động:

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

Quan hệ `commentable` trên model `Comment` sẽ trả về một instance `Post` hoặc `Video`, tùy thuộc vào loại model nào đang sở hữu comment đó.

#### Custom Polymorphic Types

Mặc định, Laravel sẽ sử dụng tên của class để lưu trữ loại model quan hệ. Chẳng hạn, như ví dụ mẫu ở trên, một `Comment` có thể thuộc về một `Post` hoặc là một `Video`, thì mặc định cột `commentable_type` sẽ có thể lưu giá trị là `App\Post` hoặc là `App\Video`. Tuy nhiên, nếu bạn muốn tách cơ sở dữ liệu ra khỏi cấu trúc folder bên trong của application. Trong trường hợp đó, bạn có thể định nghĩa một "morph map" quan hệ để chỉ dẫn Eloquent sử dụng một cái tên khác cho mỗi model thay vì tên class của mỗi model đó:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => 'App\Post',
        'videos' => 'App\Video',
    ]);

Bạn có thể đăng ký `morphMap` trong hàm `boot` của `AppServiceProvider` hoặc tạo một service provider riêng nếu bạn muốn.

<a name="many-to-many-polymorphic-relations"></a>
### Quan hệ đa hình Nhiều - Nhiều

#### Table Structure

Ngoài các quan hệ đa hình truyền thống, bạn cũng có thể định nghĩa các quan hệ đa hình "nhiều-nhiều". Ví dụ: model `Post` và `Video` của một blog có thể chia sẻ quan hệ đa hình với một model `Tag`. Sử dụng quan hệ đa hình nhiều-nhiều cho phép bạn có một danh sách các tag duy nhất được chia sẻ trên các post và video. Đầu tiên, hãy xem cấu trúc bảng:

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### Model Structure

Tiếp theo, chúng ta đã sẵn sàng để định nghĩa các quan hệ cho các model. Các model `Post` và `Video` đều sẽ có một phương thức `tags` gọi đến phương thức `morphToMany` trên class Eloquent:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### Defining The Inverse Of The Relationship

Tiếp theo, trên model `Tag`, bạn nên định nghĩa một phương thức cho từng model quan hệ của nó. Vì thế, trong ví dụ này, chúng ta sẽ định nghĩa một phương thức `posts` và một phương thức `videos`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### Retrieving The Relationship

Khi các bảng và model của bạn đã được định nghĩa xong, bạn có thể truy cập vào các quan hệ thông qua model của bạn. Ví dụ: để truy cập vào tất cả các tag của một post, bạn có thể sử dụng thuộc tính động `tags`:

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

Bạn cũng có thể lấy ra chủ sở hữu của một quan hệ đa hình từ model đa hình bằng cách truy cập vào tên của phương thức thực hiện lệnh gọi `morphedByMany`. Trong trường hợp của chúng ta, đó là các phương thức `posts` hoặc `videos` trên model `Tag`. Vì vậy, bạn sẽ truy cập các phương thức đó dưới dạng các thuộc tính động:

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## Query theo quan hệ

Vì tất cả các quan hệ Eloquent đều được định nghĩa thông qua phương thức, nên bạn có thể gọi phương thức đó để lấy ra các instance của quan hệ mà không cần phải thực hiện các câu lệnh truy vấn quan hệ. Ngoài ra, tất cả các quan hệ Eloquent cũng đóng vai trò như là một [query builders](/docs/{{version}}/queries), cho phép bạn tiếp tục thêm các ràng buộc vào các truy vấn trước khi được thực thi vào cơ sở dữ liệu của bạn.

Ví dụ, hãy tưởng tượng một hệ thống blog trong đó có model `User` liên kết với nhiều model `Post`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

Bạn có thể truy vấn quan hệ `post` và thêm các ràng buộc bổ sung cho quan hệ như sau:

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

Bạn cũng có thể sử dụng bất kỳ phương thức nào của [query builder](/docs/{{version}}/queries) trên quan hệ này, vì vậy hãy chắc chắn là bạn đã xem qua tài liệu của query builder để tìm hiểu về tất cả các phương thức có sẵn cho bạn.

<a name="relationship-methods-vs-dynamic-properties"></a>
### Phương thức quan hệ và thuộc tính động

Nếu bạn không cần thêm các ràng buộc bổ sung cho truy vấn quan hệ Eloquent, bạn có thể truy cập vào quan hệ như thể đó là một thuộc tính. Ví dụ: chúng ta sẽ tiếp tục sử dụng các model `User` và `Post` như ở trên, chúng ta có thể truy cập vào tất cả các post của user như sau:

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

Thuộc tính động là một "lazy loading", nghĩa là chúng sẽ chỉ load dữ liệu quan hệ của chúng khi bạn thực sự truy cập đến chúng. Do đó, các nhà phát triển thường sử dụng [eager loading](#eager-loading) để load trước các quan hệ mà họ biết là sẽ được truy cập sau khi một model được load. Eager loading sẽ cung cấp một cách hiệu quả để giảm số lượng truy vấn SQL phải được thực thi để load các quan hệ của một model.

<a name="querying-relationship-existence"></a>
### Query quan hệ tồn tại

Khi truy cập vào các bản ghi trong một model, bạn có thể muốn giới hạn kết quả của mình dựa trên sự tồn tại của một quan hệ. Ví dụ, hãy tưởng tượng bạn muốn lấy tất cả các post trên blog có ít nhất là một comment. Để làm như vậy, bạn có thể truyền tên của quan hệ cho các phương thức `has` hoặc `orHas`:

    // Retrieve all posts that have at least one comment...
    $posts = App\Post::has('comments')->get();

Bạn cũng có thể khai báo thêm toán tử và tổng số để tùy biến thêm cho các truy vấn:

    // Retrieve all posts that have three or more comments...
    $posts = App\Post::has('comments', '>=', 3)->get();

Các câu lệnh `has` lồng nhau cũng có thể được khởi tạo bằng cách sử dụng ký tự "dấu chấm". Ví dụ: bạn có thể lấy ra tất cả các post có ít nhất một comment và một vote:

    // Retrieve all posts that have at least one comment with votes...
    $posts = App\Post::has('comments.votes')->get();

Nếu bạn cần nhiều hơn thế nữa, bạn có thể sử dụng các phương thức `whereHas` hoặc `orWhereHas` để set các điều kiện "where" trên các truy vấn `has` của bạn. Các phương thức này cho phép bạn thêm các ràng buộc tùy biến vào một ràng buộc quan hệ, chẳng hạn như kiểm tra nội dung của một comment:

    // Retrieve all posts with at least one comment containing words like foo%
    $posts = App\Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="querying-relationship-absence"></a>
### Query quan hệ không tồn tại

Khi truy cập vào các bản ghi của một model, bạn có thể muốn giới hạn kết quả của bạn dựa trên việc không có bản ghi quan hệ nào. Ví dụ: hãy tưởng tượng bạn muốn lấy tất cả các post trên blog mà **không** có bất kỳ comment nào. Để làm như vậy, bạn có thể truyền tên của quan hệ cho các phương thức `doesntHave` hoặc `orDoesntHave`:

    $posts = App\Post::doesntHave('comments')->get();

Nếu bạn cần nhiều hơn thế nữa, bạn có thể sử dụng các phương thức `whereDoesntHave` và `orWhereDoesntHave` để set các điều kiện "where" vào các truy vấn `doesntHave` của bạn. Các phương thức này cho phép bạn thêm các ràng buộc tùy biến vào một ràng buộc quan hệ, chẳng hạn như kiểm tra nội dung của một comment:

    $posts = App\Post::whereDoesntHave('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="counting-related-models"></a>
### Đếm các bản ghi theo quan hệ model

Nếu bạn muốn đếm số lượng kết quả từ một quan hệ mà không thực sự load chúng, bạn có thể sử dụng phương thức `withCount`, và kết quả nhận được sẽ được set vào cột `{relation}_count` trên các model kết quả của bạn. Ví dụ:

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

Bạn cũng có thể thêm "counts" cho nhiều quan hệ cũng như thêm các ràng buộc cho các truy vấn:

    $posts = App\Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

Bạn cũng có thể thêm lối tắt cho một kết quả đếm quan hệ, cho phép nhiều lần đếm trên cùng một quan hệ:

    $posts = App\Post::withCount([
        'comments',
        'comments as pending_comments_count' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();

    echo $posts[0]->comments_count;

    echo $posts[0]->pending_comments_count;

<a name="eager-loading"></a>
## Eager Loading

Khi truy cập vào các quan hệ Eloquent dưới dạng các thuộc tính, dữ liệu quan hệ là "lazy loaded". Điều này có nghĩa là dữ liệu quan hệ không được tải cho đến khi bạn truy cập vào nó lần đầu tiên. Tuy nhiên, Eloquent có thể "eager load" quan hệ tại thời điểm bạn truy vấn vào model cha. Eager loading làm giảm bớt vấn đề truy vấn N + 1. Để minh họa vấn đề truy vấn N + 1, hãy xem một model `Book` có quan hệ đến `Author`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

Bây giờ, hãy lấy ra tất cả các sách và tác giả của chúng:

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Vòng lặp này sẽ thực hiện 1 truy vấn để lấy ra tất cả các sách trên bảng, sau đó một truy vấn khác cho mỗi cuốn sách để lấy ra tác giả. Vì vậy, nếu chúng ta có 25 cuốn sách, vòng lặp này sẽ chạy 26 truy vấn: 1 cho lấy tất cả các cuốn sách và 25 truy vấn thêm để lấy ra tác giả của mỗi cuốn sách.

Rất may, chúng ta có thể sử dụng eager loading để giảm các hành động này xuống chỉ còn 2 truy vấn. Khi truy vấn, bạn có thể khai báo những quan hệ nào sẽ được eager loading bằng phương thức `with`:

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Đối với cách làm này, chỉ có hai truy vấn duy nhất sẽ được thực hiện:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Eager Loading Multiple Relationships

Thỉnh thoảng bạn có thể cần eager load nhiều quan hệ khác nhau trong một hành động. Để làm như vậy, chỉ cần truyền thêm các tham số bổ sung cho phương thức `with`:

    $books = App\Book::with(['author', 'publisher'])->get();

#### Nested Eager Loading

Để eager load cho các quan hệ lồng nhau, bạn có thể sử dụng cú pháp "chấm". Ví dụ: hãy eager load tất cả các tác giả của một cuốn sách và tất cả các liên hệ của tác giả đó trong một lệnh Eloquent:

    $books = App\Book::with('author.contacts')->get();

#### Eager Loading Specific Columns

Bạn có thể không phải lúc nào cũng cần mọi cột từ quan hệ mà bạn đang truy xuất. Vì lý do này, Eloquent cho phép bạn khai báo các cột của quan hệ mà bạn muốn lấy ra:

    $users = App\Book::with('author:id,name')->get();

> {note} Khi sử dụng tính năng này, bạn phải luôn khai báo cột `id` trong danh sách các cột bạn muốn lấy ra.

<a name="constraining-eager-loads"></a>
### Rằng buộc khi eager loading

Thỉnh thoảng bạn có thể muốn eager load một quan hệ, nhưng cũng muốn khai báo thêm các ràng buộc truy vấn bổ sung cho quan hệ eager load đó. Và đây là một ví dụ cho điều đó:

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

Trong ví dụ này, Eloquent sẽ chỉ eager load các post mà trong đó cột `title` của post chứa từ `first`. Tất nhiên, bạn có thể gọi các phương thức [query builder](/docs/{{version}}/queries) khác để tùy biến thêm thao tác eager loading:

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

Thỉnh thoảng bạn có thể cần eager load một quan hệ sau khi model cha đã được lấy ra. Ví dụ, điều này có thể hữu ích nếu bạn cần quyết định có tự động load các model quan hệ hay không:

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

Nếu bạn cần set thêm các ràng buộc truy vấn bổ sung cho truy vấn eager loading, bạn có thể truyền một mảng có khóa là các quan hệ mà bạn muốn load. Các giá trị mảng phải là các instances `Closure` nhận vào một instances query:

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

Để load một quan hệ khi nó chưa được load, hãy sử dụng phương thức `loadMissing`:

    public function format(Book $book)
    {
        $book->loadMissing('author');

        return [
            'name' => $book->name,
            'author' => $book->author->name
        ];
    }

<a name="inserting-and-updating-related-models"></a>
## Thêm và cập nhật theo quan hệ model

<a name="the-save-method"></a>
### Phương thức save

Eloquent cung cấp các phương thức thuận tiện để thêm một model mới vào các quan hệ. Ví dụ, có lẽ bạn cần thêm một `Comment` mới cho model `Post`. Thay vì set thủ công thuộc tính `post_id` trên `Comment`, thì bạn có thể thêm trực tiếp `Comment` từ phương thức `save` của quan hệ::

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

Lưu ý rằng chúng ta đã không truy cập vào quan hệ `comments` như là một thuộc tính động. Thay vào đó, chúng ta đã gọi phương thức `comments` để lấy ra một instance của quan hệ. Phương thức `save` sẽ tự động thêm giá trị `post_id` phù hợp vào model `Comment` mới.

Nếu bạn cần lưu nhiều model quan hệ cùng một lúc, bạn có thể sử dụng phương thức `saveMany`:

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

<a name="the-create-method"></a>
### Phương thức create

Ngoài các phương thức `save` và `saveMany`, bạn cũng có thể sử dụng phương thức `create`, nó chấp nhận một mảng các thuộc tính để tạo một model và thêm nó vào cơ sở dữ liệu. Một lần nữa, sự khác biệt giữa `save` và `create` là `save` sẽ chấp nhận một instance model Eloquent trong khi `create` chấp nhận một mảng của PHP:

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

> {tip} Trước khi sử dụng phương thức `create`, bạn hãy chắc chắn là đã xem qua tài liệu về thuộc tính [mass assignment](/docs/{{version}}/eloquent#mass-assignment).

Bạn có thể sử dụng phương thức `createMany` để tạo nhiều model quan hệ:

    $post = App\Post::find(1);

    $post->comments()->createMany([
        [
            'message' => 'A new comment.',
        ],
        [
            'message' => 'Another new comment.',
        ],
    ]);

<a name="updating-belongs-to-relationships"></a>
### Quan hệ thuộc về

Khi cập nhật một quan hệ `belongsTo`, bạn có thể sử dụng phương thức `associate`. Phương thức này sẽ set khóa ngoại trên model con:

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

Khi xóa một quan hệ `belongsTo`, bạn có thể sử dụng phương thức `dissociate`. Phương thức này sẽ set khóa ngoại của quan hệ thành `null`:

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### Quan hệ Nhiều - Nhiều

#### Attaching / Detaching

Eloquent cũng cung cấp thêm một vài phương thức helper để làm việc với các model quan hệ một cách thuận tiện hơn. Ví dụ: hãy tưởng tượng một user có thể có nhiều role và một role có thể có nhiều user. Để attach một role cho một user chúng ta có thể làm bằng cách thêm một bản ghi vào trong bảng trung gian, hãy sử dụng phương thức `attach`:

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

Khi attach một quan hệ với một model, bạn cũng có thể truyền thêm một mảng dữ liệu sẽ được thêm cùng vào trong bảng trung gian:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Tất nhiên, thỉnh thoảng có thể cần phải xóa một role ra khỏi một user. Để xoá một bản ghi quan hệ nhiều-nhiều, hãy sử dụng phương thức `detach`. Phương thức `detach` sẽ xoá bản ghi phù hợp ra khỏi bảng trung gian; tuy nhiên, cả hai model vẫn sẽ còn trong cơ sở dữ liệu:

    // Detach a single role from the user...
    $user->roles()->detach($roleId);

    // Detach all roles from the user...
    $user->roles()->detach();

Để thuận tiện, các phương thức `attach` và `detach` cũng chấp nhận một mảng ID làm đầu vào:

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires]
    ]);

#### Syncing Associations

Bạn cũng có thể sử dụng phương thức `sync` để khởi tạo các liên kết nhiều-nhiều. Phương thức `sync` chấp nhận một mảng ID để set vào trong bảng trung gian. Bất kỳ ID nào không nằm trong mảng đã cho sẽ bị xóa khỏi bảng trung gian. Vì vậy, sau khi thao tác này hoàn tất, chỉ có các ID trong mảng đã cho sẽ tồn tại trong bảng trung gian:

    $user->roles()->sync([1, 2, 3]);

Bạn cũng có thể truyền thêm các giá trị cho bảng trung gian với ID:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

Nếu bạn không muốn detach ID đã tồn tại, bạn có thể sử dụng phương thức `syncWithoutDetaching`:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### Toggling Associations

Quan hệ nhiều-nhiều cũng cung cấp thêm một phương thức `toggle` để "bật" hoặc "tắt" trạng thái attach của một mảng các ID đã cho. Nếu ID trong mảng đó đã được attach, thì nó sẽ bị detached. Tương tự, nếu nó đang được detached, thì nó sẽ được attach:

    $user->roles()->toggle([1, 2, 3]);

#### Saving Additional Data On A Pivot Table

Khi làm việc với một quan hệ nhiều-nhiều, phương thức `save` sẽ chấp nhận thêm một mảng các thuộc tính cho bảng trung gian làm tham số thứ hai của nó:

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### Updating A Record On A Pivot Table

Nếu bạn cần cập nhật một bản ghi hiện có trong bảng pivot của bạn, bạn có thể sử dụng phương thức `updateExistingPivot`. Phương thức này chấp nhận khóa ngoại của bản ghi pivot và một mảng các thuộc tính để cập nhật:

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## Sửa timestamp của model chứa

Khi một model `belongsTo` hoặc `belongsToMany` tới một model khác, chẳng hạn như một `Comment` nằm trong một `Post`, đôi khi bạn sẽ cần thiết cập nhật timestamp của model cha khi model con được cập nhật. Ví dụ, khi một model `Comment` được cập nhật, bạn có thể muốn tự động "touch" vào cột timestamp `update_at` của `Post` cha của nó. Eloquent có thể làm cho hành động đó trở lên dễ dàng. Bạn chỉ cần thêm một thuộc tính `touches` chứa tên của quan hệ, vào model con của bạn:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Bây giờ, khi bạn cập nhật một `Comment`, thì `Post` cha của nó cũng sẽ được cập nhật cột `update_at`, giúp thuận tiện hơn để biết khi nào vô hiệu hoá cache của model `Post`:

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
