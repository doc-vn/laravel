# Eloquent: Relationships

- [Giới thiệu](#introduction)
- [Định nghĩa quan hệ](#defining-relationships)
    - [Một - Một](#one-to-one)
    - [Một - Nhiều](#one-to-many)
    - [Một - Nhiều (Ngược lại) / Belongs To](#one-to-many-inverse)
    - [Một trong nhiều](#has-one-of-many)
    - [Quan hệ thông qua liên kết một](#has-one-through)
    - [Quan hệ thông qua liên kết nhiều](#has-many-through)
- [Nhiều - Nhiều](#many-to-many)
    - [Lấy cột trong bảng trung gian](#retrieving-intermediate-table-columns)
    - [Lọc bảng trung gian](#filtering-queries-via-intermediate-table-columns)
    - [Sắp xếp thông qua bảng trung gian](#ordering-queries-via-intermediate-table-columns)
    - [Định nghĩa model trung gian](#defining-custom-intermediate-table-models)
- [Quan hệ đa hình](#polymorphic-relationships)
    - [Một - Một](#one-to-one-polymorphic-relations)
    - [Một - Nhiều](#one-to-many-polymorphic-relations)
    - [Một trong nhiều](#one-of-many-polymorphic-relations)
    - [Nhiều - Nhiều](#many-to-many-polymorphic-relations)
    - [Tuỳ biến quan hệ đa hình](#custom-polymorphic-types)
    - [Quan hệ động](#dynamic-relationships)
- [Query theo quan hệ](#querying-relations)
    - [Phương thức quan hệ và thuộc tính động](#relationship-methods-vs-dynamic-properties)
    - [Query quan hệ tồn tại](#querying-relationship-existence)
    - [Query quan hệ không tồn tại](#querying-relationship-absence)
    - [Query quan hệ đa hình](#querying-morph-to-relationships)
- [Tính toán model quan hệ](#aggregating-related-models)
    - [Đếm các bản ghi theo quan hệ model](#counting-related-models)
    - [Các hàm tính toán khác](#other-aggregate-functions)
    - [Đếm model quan hệ trên quan hệ đa hình](#counting-related-models-on-morph-to-relationships)
- [Eager Loading](#eager-loading)
    - [Rằng buộc khi eager loading](#constraining-eager-loads)
    - [Lazy Eager Loading](#lazy-eager-loading)
    - [Chặn Lazy Loading](#preventing-lazy-loading)
- [Thêm và cập nhật theo quan hệ model](#inserting-and-updating-related-models)
    - [Phương thức save](#the-save-method)
    - [Phương thức create](#the-create-method)
    - [Quan hệ thuộc về](#updating-belongs-to-relationships)
    - [Quan hệ Nhiều - Nhiều](#updating-many-to-many-relationships)
- [Sửa timestamp của model cha](#touching-parent-timestamps)

<a name="introduction"></a>
## Giới thiệu

Các bảng cơ sở dữ liệu thường được quan hệ với nhau. Ví dụ: một bài post trên một blog có thể có nhiều comment hoặc một order có thể có quan hệ với người dùng đã đặt nó. Eloquent giúp quản lý và làm việc với những quan hệ này một cách dễ dàng hơn và hỗ trợ nhiều quan hệ phổ biến:

<div class="content-list" markdown="1">

- [Một - Một](#one-to-one)
- [Một - Nhiều](#one-to-many)
- [Nhiều - Nhiều](#many-to-many)
- [Quan hệ thông qua liên kết một](#has-one-through)
- [Quan hệ thông qua liên kết nhiều](#has-many-through)
- [Một - Một (đa hình)](#one-to-one-polymorphic-relations)
- [Một - Nhiều (đa hình)](#one-to-many-polymorphic-relations)
- [Nhiều - Nhiều (đa hình)](#many-to-many-polymorphic-relations)

</div>

<a name="defining-relationships"></a>
## Định nghĩa quan hệ

Các quan hệ của Eloquent là các phương thức được định nghĩa nằm trong các class của model Eloquent. Cũng giống như các quan hệ cũng đóng vai trò như là các [query builder](/docs/{{version}}/queries), nó định nghĩa các quan hệ như là các phương thức cung cấp khả năng kết hợp và truy vấn mạnh mẽ. Ví dụ, chúng ta có thể kết hợp các ràng buộc cho quan hệ `posts` như thế này:

    $user->posts()->where('active', 1)->get();

Nhưng, trước khi đi sâu vào việc sử dụng các quan hệ, hãy tìm hiểu cách định nghĩa cho từng loại quan hệ được hỗ trợ bởi Eloquent.

<a name="one-to-one"></a>
### Một - Một

Một quan hệ một-một là một type of database relationship rất cơ bản. Ví dụ, một model `User` có thể có quan hệ với một model `Phone`. Để định nghĩa quan hệ này, chúng ta sẽ set một phương thức `phone` trên model `User`. Phương thức `phone` này sẽ gọi phương thức `hasOne` và trả về chính phương thức đó. Phương thức `hasOne` sẽ có sẵn trong model của bạn thông qua class `Illuminate\Database\Eloquent\Model` của model:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasOne;

    class User extends Model
    {
        /**
         * Get the phone associated with the user.
         */
        public function phone(): HasOne
        {
            return $this->hasOne(Phone::class);
        }
    }

Tham số đầu tiên được truyền vào cho phương thức `hasOne` là tên class của model được quan hệ. Khi quan hệ đã được định nghĩa, chúng ta có thể lấy ra các bản ghi theo quan hệ bằng các thuộc tính động của Eloquent. Các thuộc tính động cho phép bạn truy cập vào các phương thức quan hệ như thể chúng là các thuộc tính được định nghĩa trên model:

    $phone = User::find(1)->phone;

Eloquent sẽ xác định khóa ngoại của quan hệ dựa trên tên model cha. Trong trường hợp này, model `Phone` tự động được giả định là có khóa ngoại `user_id`. Nếu bạn muốn ghi đè quy ước này, bạn có thể truyền tham số thứ hai vào phương thức `hasOne`:

    return $this->hasOne(Phone::class, 'foreign_key');

Ngoài ra, Eloquent cũng giả định rằng khóa ngoại này phải có giá trị trùng với giá trị cột primary key của model cha. Nói cách khác, Eloquent sẽ tìm giá trị `id` của user trong cột `user_id` trong bảng `Phone`. Nếu bạn muốn quan hệ này sử dụng một giá trị primary key khác ngoài `id` hoặc thuộc tính `$primaryKey` của model của bạn, bạn có thể truyền vào một tham số thứ ba cho phương thức `hasOne`:

    return $this->hasOne(Phone::class, 'foreign_key', 'local_key');

<a name="one-to-one-defining-the-inverse-of-the-relationship"></a>
#### Defining The Inverse Of The Relationship

Vì vậy, chúng ta có thể truy cập vào model `Phone` từ model `User`. Tiếp theo, hãy định nghĩa quan hệ trong model `Phone`, sẽ cho phép chúng ta truy cập ngược lại vào model `User` sở hữu chiếc phone đó. Chúng ta có thể định nghĩa một quan hệ ngược lại của `hasOne` bằng cách sử dụng phương thức `belongsTo`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsTo;

    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user(): BelongsTo
        {
            return $this->belongsTo(User::class);
        }
    }

Khi gọi phương thức `user`, Eloquent sẽ cố gắng tìm một model `User` có `id` khớp với cột `user_id` trên model `Phone`.

Eloquent sẽ xác định tên mặc định của khóa ngoại bằng cách lấy tên của phương thức quan hệ và thêm hậu tố `_id`. Vì vậy, trong trường hợp này, Eloquent giả định rằng model `Phone` sẽ có cột `user_id`. Tuy nhiên, nếu khóa ngoại trên model `Phone` không phải là `user_id`, bạn có thể truyền một tên khóa khác làm tham số thứ hai cho phương thức `belongsTo`:

    /**
     * Get the user that owns the phone.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class, 'foreign_key');
    }

Nếu model cha không sử dụng cột `id` làm khóa chính hoặc bạn muốn tìm model hiện tại bằng cách sử dụng một cột khác không phải cột mặc định, bạn có thể truyền một tham số thứ ba cho phương thức `belongsTo` khai báo khóa chính tùy biến trong bảng cha:

    /**
     * Get the user that owns the phone.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
    }

<a name="one-to-many"></a>
### Một - Nhiều

Quan hệ một-nhiều có thể được sử dụng để định nghĩa các quan hệ mà trong đó một model cha là cha của một hoặc nhiều model con. Ví dụ, một post trên blog có thể có nhiều comment. Giống như tất cả các quan hệ Eloquent khác, quan hệ một-nhiều được định nghĩa bằng cách định nghĩa một phương thức trên model Eloquent của bạn:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasMany;

    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments(): HasMany
        {
            return $this->hasMany(Comment::class);
        }
    }

Hãy nhớ rằng, Eloquent sẽ tự động xác định cột khóa ngoại phù hợp cho model `Comment`. Theo quy ước, Eloquent sẽ lấy tên theo dạng "snake case" của model cha và set thêm hậu tố là `_id`. Vì vậy, trong ví dụ này, Eloquent sẽ giả định rằng cột khóa ngoại trong model `Comment` là `post_id`.

Khi phương thức quan hệ đã được định nghĩa xong, bạn có thể truy cập vào một [collection](/docs/{{version}}/eloquent-collections) comment bằng cách truy cập thông qua thuộc tính `comments`. Hãy nhớ rằng, vì Eloquent sẽ cung cấp "các thuộc tính động" cho quan hệ, nên bạn có thể truy cập vào các phương thức quan hệ như thể chúng được định nghĩa là các thuộc tính trong model:

    use App\Models\Post;

    $comments = Post::find(1)->comments;

    foreach ($comments as $comment) {
        // ...
    }

Vì tất cả các quan hệ cũng đóng vai trò như là một query builder, nên bạn có thể thêm các ràng buộc cho the relationship query bằng cách gọi phương thức `comments` và tiếp tục thêm các điều kiện vào trong truy vấn:

    $comment = Post::find(1)->comments()
                        ->where('title', 'foo')
                        ->first();

Giống như phương thức `hasOne`, bạn cũng có thể ghi đè các khóa ngoại và khóa chính bằng cách truyền thêm các tham số bổ sung cho phương thức `hasMany`:

    return $this->hasMany(Comment::class, 'foreign_key');

    return $this->hasMany(Comment::class, 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### Một - Nhiều (Ngược lại) / Belongs To

Bây giờ chúng ta có thể truy cập vào tất cả các comment của một post, tiếp theo hãy định nghĩa một quan hệ để cho phép từ một comment có thể truy cập ngược lại vào một post của chính nó. Để định nghĩa một nghịch đảo của quan hệ `hasMany`, hãy định nghĩa một phương thức quan hệ trên model con gọi đến phương thức `belongsTo`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsTo;

    class Comment extends Model
    {
        /**
         * Get the post that owns the comment.
         */
        public function post(): BelongsTo
        {
            return $this->belongsTo(Post::class);
        }
    }

Khi quan hệ đã được định nghĩa, chúng ta có thể lấy ra một post từ một comment cha bằng cách truy cập vào "thuộc tính quan hệ động" `post`:

    use App\Models\Comment;

    $comment = Comment::find(1);

    return $comment->post->title;

Trong ví dụ trên, Eloquent sẽ thử tìm model `Post` có `id` khớp với cột `post_id` trên model `Comment`.

Eloquent sẽ xác định tên mặc định của khóa ngoại bằng cách lấy tên của phương thức quan hệ và thêm hậu tố `_` cùng với tên của cột khóa chính của model cha. Vì vậy, trong ví dụ này, Eloquent sẽ giả sử khóa ngoại của model `Post` trên bảng `comments` sẽ là `post_id`.

Tuy nhiên, nếu khóa ngoại cho quan hệ của bạn không tuân theo các quy ước này, thì bạn có thể truyền một tên khóa ngoại khác làm tham số thứ hai cho phương thức `belongsTo`:

    /**
     * Get the post that owns the comment.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class, 'foreign_key');
    }

Nếu model cha của bạn không sử dụng `id` làm khóa chính của nó hoặc bạn muốn tìm model hiện tại bằng cách sử dụng một cột khác, bạn có thể truyền vào một tham số thứ ba cho phương `belongsTo` khai báo khóa chính tùy biến của bảng cha của bạn:

    /**
     * Get the post that owns the comment.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class, 'foreign_key', 'owner_key');
    }

<a name="default-models"></a>
#### Default Models

Quan hệ `belongsTo`, `hasOne`, `hasOneThrough` và `morphOne` cho phép bạn định nghĩa một model mặc định sẽ được trả về nếu quan hệ đã cho là `null`. Trường hợp này thường được gọi là [trường hợp đối tượng null](https://en.wikipedia.org/wiki/Null_Object_pattern) và có thể giúp loại bỏ các kiểm tra có điều kiện trong code của bạn. Trong ví dụ sau, quan hệ `user` sẽ trả về một model `App\Models\User` trống nếu không có user nào được gán với model `Post`:

    /**
     * Get the author of the post.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class)->withDefault();
    }

Để thêm các thuộc tính vào model mặc định, bạn có thể truyền một mảng hoặc một closure cho phương thức `withDefault`:

    /**
     * Get the author of the post.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class)->withDefault([
            'name' => 'Guest Author',
        ]);
    }

    /**
     * Get the author of the post.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
            $user->name = 'Guest Author';
        });
    }

<a name="querying-belongs-to-relationships"></a>
#### Querying Belongs To Relationships

Khi truy vấn vào các phần tử con trong quan hệ "belongs to", bạn có thể xây dựng thêm điều kiện `where` để lấy ra các model Eloquent tương ứng:

    use App\Models\Post;

    $posts = Post::where('user_id', $user->id)->get();

Tuy nhiên, bạn có thể cảm thấy thuận tiện hơn khi sử dụng phương thức `whereBelongsTo`, phương thức này sẽ tự động xác định quan hệ thích hợp và khóa ngoại cho model đã cho:

    $posts = Post::whereBelongsTo($user)->get();

Bạn cũng có thể cung cấp một instance [collection](/docs/{{version}}/eloquent-collections) cho phương thức `whereBelongsTo`. Khi làm như vậy, Laravel sẽ lấy ra tất cả các model mà thuộc về bất kỳ model gốc nào có trong collection:

    $users = User::where('vip', true)->get();

    $posts = Post::whereBelongsTo($users)->get();

Mặc định, Laravel sẽ xác định quan hệ được liên kết với model đã cho dựa trên tên class của model; tuy nhiên, bạn có thể chỉ định tên quan hệ bằng cách cung cấp nó làm tham số thứ hai cho phương thức `whereBelongsTo`:

    $posts = Post::whereBelongsTo($user, 'author')->get();

<a name="has-one-of-many"></a>
### Một trong nhiều

Đôi khi một model có thể có nhiều model quan hệ, nhưng bạn muốn dễ dàng lấy ra một model quan hệ "mới nhất" hoặc "cũ nhất" của quan hệ đó. Ví dụ: model `User` có thể có quan hệ đến nhiều model `Order`, nhưng bạn muốn định nghĩa một cách để tương tác với đơn hàng gần đây nhất mà người dùng đã đặt. Bạn có thể thực hiện việc này bằng cách sử dụng quan hệ `hasOne` kết hợp với các phương thức `ofMany`:

```php
/**
 * Get the user's most recent order.
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

Tương tự như vậy, bạn có thể định nghĩa một phương thức để lấy ra model quan hệ "cũ nhất" hoặc đầu tiên của một quan hệ:

```php
/**
 * Get the user's oldest order.
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

Mặc định, các phương thức `latestOfMany` và `oldestOfMany` sẽ lấy ra một model quan hệ mới nhất hoặc cũ nhất dựa trên khóa chính của model, khóa này phải có thể sắp xếp được. Tuy nhiên, đôi khi bạn có thể muốn lấy ra một model từ một quan hệ bằng cách sử dụng một tiêu chí khác.

Ví dụ: sử dụng phương thức `ofMany`, bạn có thể lấy ra đơn đặt hàng đắt nhất của người dùng. Phương thức `ofMany` chấp nhận cột có thể sắp xếp làm tham số đầu tiên của nó và hàm tính toán nào (`min` hoặc `max`) được sử dụng khi truy vấn model quan hệ:

```php
/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

> [!WARNING]
> Bởi vì PostgreSQL không hỗ trợ thực thi các hàm `MAX` đối với các cột UUID, nên hiện tại không thể sử dụng quan hệ một trong nhiều kết hợp với các cột UUID của PostgreSQL.

<a name="converting-many-relationships-to-has-one-relationships"></a>
#### Converting "Many" Relationships to Has One Relationships

Thông thường, khi lấy ra một model duy nhất bằng các phương thức `latestOfMany`, `oldestOfMany` hoặc `ofMany`, bạn đã có quan hệ "has many" được định nghĩa cho cùng một model. Để thuận tiện, Laravel cho phép bạn dễ dàng chuyển quan hệ này thành quan hệ "has one" bằng cách gọi phương thức `one` trên quan hệ:

```php
/**
 * Get the user's orders.
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```

<a name="advanced-has-one-of-many-relationships"></a>
#### Advanced Has One Of Many Relationships

Có thể xây dựng một quan hệ "một trong nhiều" nâng cao hơn. Ví dụ: Một model `Product` có thể có nhiều model `Price` được gán cho model sản phẩm đó và được giữ lại trong hệ thống ngay cả sau khi giá mới được công bố. Ngoài ra, giá mới cho sản phẩm dó có thể được thêm vào trước đó để có thể có hiệu lực vào một ngày nhất định trong tương lai thông qua cột `published_at`.

Vì vậy, tóm lại, chúng ta cần lấy ra giá được công bố mới nhất khi mà ngày công bố không ở trong tương lai. Ngoài ra, nếu có hai giá có cùng ngày công bố, chúng ta sẽ ưu tiên giá mà có ID được tạo sau. Để thực hiện điều này, chúng ta phải truyền một mảng cho phương thức `ofMany` có chứa các cột có thể sắp xếp để xác định giá mới nhất. Ngoài ra, một closure sẽ được cung cấp làm tham số thứ hai cho phương thức `ofMany`. Closure này sẽ chịu trách nhiệm thêm các ràng buộc ngày công bố vào truy vấn quan hệ:

```php
/**
 * Get the current pricing for the product.
 */
public function currentPricing(): HasOne
{
    return $this->hasOne(Price::class)->ofMany([
        'published_at' => 'max',
        'id' => 'max',
    ], function (Builder $query) {
        $query->where('published_at', '<', now());
    });
}
```

<a name="has-one-through"></a>
### Has One Through

Quan hệ "has-one-through" định nghĩa quan hệ một-một với một model khác. Tuy nhiên, quan hệ này chỉ ra rằng model khai báo có thể được khớp với một instance của model khác bằng cách thực hiện _thông qua_ model thứ ba.

Ví dụ: trong ứng dụng cửa hàng sửa chữa xe, mỗi model `Mechanic` có thể được liên kết với một model `Car` và mỗi model `Car` lại có thể được liên kết với một model `Owner`. Mặc dù mechanic và owner không có quan hệ trực tiếp trong cơ sở dữ liệu, nhưng mechanic lại có thể truy cập vào owner _thông qua_ model `Car`. Hãy xem các bảng cần thiết để định nghĩa quan hệ này như sau:

    mechanics
        id - integer
        name - string

    cars
        id - integer
        model - string
        mechanic_id - integer

    owners
        id - integer
        name - string
        car_id - integer

Bây giờ chúng ta đã xem qua cấu trúc bảng cho quan hệ, hãy định nghĩa quan hệ trên model `Mechanic`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasOneThrough;

    class Mechanic extends Model
    {
        /**
         * Get the car's owner.
         */
        public function carOwner(): HasOneThrough
        {
            return $this->hasOneThrough(Owner::class, Car::class);
        }
    }

Tham số đầu tiên được truyền cho phương thức `hasOneThrough` là tên của model cuối cùng mà chúng ta muốn lấy ra, trong khi tham số thứ hai là tên của model trung gian.

Hoặc, nếu các quan hệ liên quan đã được định nghĩa trong tất cả các model khác, bạn có thể định nghĩa một cách dễ dàng quan hệ "has-one-through" bằng cách gọi phương thức `through` và cung cấp tên các quan hệ đó. Ví dụ: nếu model `Mechanic` có một quan hệ là `cars` và model `Car` có một quan hệ là `owner`, bạn có thể định nghĩa một quan hệ "has-one-through" kết nối model mechanic với owner như sau:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-one-through-key-conventions"></a>
#### Key Conventions

Các quy ước khóa ngoại mặc định của Eloquent sẽ được sử dụng khi thực hiện các truy vấn của quan hệ. Nếu bạn muốn tùy chỉnh các khóa của quan hệ, bạn có thể truyền chúng dưới dạng là các tham số thứ ba và thứ tư cho phương thức `hasOneThrough`. Tham số thứ ba là tên của khóa ngoại trên model trung gian. Tham số thứ tư là tên của khóa ngoại trên model cuối cùng. Tham số thứ năm là khóa local, trong khi tham số thứ sáu là khóa local của model trung gian:

    class Mechanic extends Model
    {
        /**
         * Get the car's owner.
         */
        public function carOwner(): HasOneThrough
        {
            return $this->hasOneThrough(
                Owner::class,
                Car::class,
                'mechanic_id', // Foreign key on the cars table...
                'car_id', // Foreign key on the owners table...
                'id', // Local key on the mechanics table...
                'id' // Local key on the cars table...
            );
        }
    }

Hoặc, như đã thảo luận trước đó, nếu các quan hệ liên quan đã được định nghĩa trên tất cả các model khác, bạn có thể định nghĩa một cách dễ dàng quan hệ "has-one-through" bằng cách gọi phương thức `through` và cung cấp tên của những quan hệ đó. Cách tiếp cận này mang lại lợi ích là sử dụng lại các quy ước chính đã được định nghĩa trên các quan hệ hiện có:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-many-through"></a>
### Has Many Through

Quan hệ "has-many-through" cung cấp một cách thuận tiện để lấy ra các quan hệ ở xa thông qua một quan hệ trung gian. Ví dụ: giả sử chúng ta đang xây dựng một nền tảng triển khai như [Laravel Vapor](https://vapor.laravel.com). Model `Project` có thể lấy ra nhiều model `Deployment` thông qua model `Environment` trung gian. Sử dụng ví dụ này, bạn có thể dễ dàng lấy ra tất cả các triển khai cho một dự án nhất định. Hãy xem các bảng cần thiết để định nghĩa quan hệ này:

    projects
        id - integer
        name - string

    environments
        id - integer
        project_id - integer
        name - string

    deployments
        id - integer
        environment_id - integer
        commit_hash - string

Bây giờ chúng ta đã xem qua cấu trúc bảng cho quan hệ, hãy định nghĩa quan hệ trên model `Project`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasManyThrough;

    class Project extends Model
    {
        /**
         * Get all of the deployments for the project.
         */
        public function deployments(): HasManyThrough
        {
            return $this->hasManyThrough(Deployment::class, Environment::class);
        }
    }

Tham số đầu tiên được truyền cho phương thức `hasManyThrough` là tên của model cuối cùng mà chúng ta muốn truy cập, trong khi tham số thứ hai là tên của model trung gian.

Hoặc, nếu các quan hệ liên quan đã được định nghĩa trên tất cả các model khác, bạn có thể định nghĩa một cách dễ dàng quan hệ "has-many-through" bằng cách gọi phương thức `through` và cung cấp tên của các quan hệ đó. Ví dụ: nếu model `Project` có một quan hệ `environments` và model `Environment` có một quan hệ `deployments`, bạn có thể định nghĩa một quan hệ "has-many-through" kết nối dự án và các deployment như sau:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

Mặc dù bảng của model `Deployment` không chứa cột `project_id`, nhưng quan hệ `hasManyThrough` cung cấp quyền truy cập vào các deployment của dự án thông qua `$project->deployments`. Để lấy ra các model này, Eloquent sẽ kiểm tra cột `project_id` trên bảng của model `Environment` trung gian. Sau khi tìm thấy ID environment có quan hệ, chúng sẽ được sử dụng để truy vấn vào bảng của model `Deployment`.

<a name="has-many-through-key-conventions"></a>
#### Key Conventions

Các quy ước khóa ngoại mặc định của Eloquent sẽ được sử dụng khi thực hiện các truy vấn của quan hệ. Nếu bạn muốn tùy chỉnh các khóa của quan hệ, bạn có thể truyền chúng dưới dạng các tham số thứ ba và thứ tư cho phương thức `hasManyThrough`. Tham số thứ ba là tên của khóa ngoại trên model trung gian. Tham số thứ tư là tên của khóa ngoại trên model cuối cùng. Tham số thứ năm là khóa local, trong khi Tham số thứ sáu là khóa local của model trung gian:

    class Project extends Model
    {
        public function deployments(): HasManyThrough
        {
            return $this->hasManyThrough(
                Deployment::class,
                Environment::class,
                'project_id', // Foreign key on the environments table...
                'environment_id', // Foreign key on the deployments table...
                'id', // Local key on the projects table...
                'id' // Local key on the environments table...
            );
        }
    }

Hoặc, như đã thảo luận trước đó, nếu các quan hệ liên quan đã được định nghĩa trên tất cả các model khác, bạn có thể định nghĩa một cách dễ dàng quan hệ "has-many-through" bằng cách gọi phương thức `through` và cung cấp tên của những quan hệ đó. Cách tiếp cận này mang lại lợi ích là sử dụng lại các quy ước chính đã được định nghĩa trên các quan hệ hiện có:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

<a name="many-to-many"></a>
## Nhiều - Nhiều

Quan hệ nhiều-nhiều có thể sẽ phức tạp hơn một chút so với các quan hệ `hasOne` và `hasMany`. Một ví dụ về quan hệ kiểu như vậy là một user có thể có nhiều role, trong đó các role cũng có thể được chia cho nhiều user khác nhau trong ứng dụng. Ví dụ: user có thể được với role "Author" và "Editor"; tuy nhiên, những role đó cũng có thể được gán cho những user khác. Vì vậy, một user có nhiều role và một role có thể có nhiều user.

<a name="many-to-many-table-structure"></a>
#### Table Structure

Để định nghĩa quan hệ này, cần có ba bảng cơ sở dữ liệu: `users`, `roles`, và `role_user`. Tên bảng `role_user` sẽ được lấy theo thứ tự chữ cái của tên các model và có chứa các cột `user_id` và `role_id`. Bảng này được sử dụng làm bảng trung gian liên kết giữa user và role.

Hãy nhớ rằng, vì một role có thể thuộc về nhiều user, nên chúng ta không thể chỉ tạo một cột `user_id` trên bảng `roles`. Điều này có nghĩa là một role chỉ có thể thuộc về một user. Để cung cấp hỗ trợ cho các role được gán cho nhiều user, cần có bảng `role_user`. Chúng ta có thể tóm tắt cấu trúc bảng của quan hệ như sau:

    users
        id - integer
        name - string

    roles
        id - integer
        name - string

    role_user
        user_id - integer
        role_id - integer

<a name="many-to-many-model-structure"></a>
#### Model Structure

Quan hệ nhiều-nhiều được định nghĩa bằng cách viết một phương thức trả về kết quả của phương thức `belongsToMany`. Phương thức `belongsToMany` được cung cấp bởi class `Illuminate\Database\Eloquent\Model` được sử dụng bởi tất cả các model Eloquent trong ứng dụng của bạn. Ví dụ: hãy định nghĩa một phương thức `roles` trên model `User` của chúng ta. Tham số đầu tiên được truyền cho phương thức này là tên của class model quan hệ:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsToMany;

    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles(): BelongsToMany
        {
            return $this->belongsToMany(Role::class);
        }
    }

Khi quan hệ này được định nghĩa xong, bạn có thể truy cập role của user bằng cách sử dụng thuộc tính quan hệ `roles`:

    use App\Models\User;

    $user = User::find(1);

    foreach ($user->roles as $role) {
        // ...
    }

Vì tất cả các quan hệ cũng đóng vai trò là một query builder, nên bạn có thể thêm các ràng buộc khác vào truy vấn bằng cách gọi phương thức `roles` và tiếp tục thêm các điều kiện vào truy vấn:

    $roles = User::find(1)->roles()->orderBy('name')->get();

Để xác định tên bảng của bảng trung gian của quan hệ, Eloquent sẽ nối tên của hai mode có quan hệ lại với nhay theo thứ tự bảng chữ cái. Tuy nhiên, bạn có thể tự do ghi đè quy ước này. Bạn có thể làm như vậy bằng cách truyền tham số thứ hai cho phương thức `belongsToMany`:

    return $this->belongsToMany(Role::class, 'role_user');

Ngoài việc tùy chỉnh tên của bảng trung gian, bạn cũng có thể tùy chỉnh tên cột của các khóa trên bảng trung gian bằng cách truyền thêm các tham số cho phương thức `belongsToMany`. Tham số thứ ba là tên khóa ngoại của model mà bạn đang định nghĩa quan hệ, trong khi tham số thứ tư là tên khóa ngoại của model mà bạn đang nối đến:

    return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');

<a name="many-to-many-defining-the-inverse-of-the-relationship"></a>
#### Defining The Inverse Of The Relationship

Để định nghĩa "nghịch đảo" của quan hệ nhiều-nhiều, bạn nên định nghĩa một phương thức trên model quan hệ, phương thức này cũng trả về kết quả của phương thức `belongsToMany`. Để hoàn thành ví dụ về user và role của chúng ta, hãy định nghĩa phương thức `users` trên model `Role`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsToMany;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users(): BelongsToMany
        {
            return $this->belongsToMany(User::class);
        }
    }

Như bạn có thể thấy, quan hệ được định nghĩa chính xác giống y hệt model `User` của nó, ngoại trừ việc tham chiếu model là `App\Models\User`. Vì chúng ta đang sử dụng lại phương thức `belongsToMany`, nên tất cả các tùy chọn tùy chỉnh khóa và bảng thông thường khác đều khả dụng khi định nghĩa "nghịch đảo" của quan hệ nhiều-nhiều.

<a name="retrieving-intermediate-table-columns"></a>
### Lấy cột trong bảng trung gian

Như bạn đã biết, làm việc với quan hệ nhiều-nhiều yêu cầu phải có bảng trung gian. Eloquent cung cấp một số cách rất hữu ích để tương tác với bảng này. Ví dụ: giả sử model `User` của chúng ta có nhiều model `Role` quan hệ đến model đó. Sau khi truy cập vào quan hệ này, chúng ta có thể truy cập vào bảng trung gian bằng thuộc tính `pivot` trên các model:

    use App\Models\User;

    $user = User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

Lưu ý rằng mỗi model `Role` mà chúng ta lấy ra được sẽ tự động được gán một thuộc tính là `pivot`. Thuộc tính này chứa model đại diện cho bảng trung gian.

Mặc định, chỉ những khóa của model mới có thể xuất hiện trong đối tượng `pivot`. Nếu bảng pivot của bạn có chứa thêm các thuộc tính khác, bạn phải khai báo chúng khi định nghĩa quan hệ:

    return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');

Nếu bạn muốn bảng pivot của bạn tự động duy trì các cột timestamp `created_at` và `update_at`, hãy sử dụng phương thức` withTimestamps` trong định nghĩa quan hệ của bạn:

    return $this->belongsToMany(Role::class)->withTimestamps();

> [!WARNING]
> Các bảng trung gian sử dụng timestamp được duy trì tự động của Eloquent bắt buộc phải có cả hai cột timestamp `created_at` và `updated_at`.

<a name="customizing-the-pivot-attribute-name"></a>
#### Customizing The `pivot` Attribute Name

Như đã lưu ý trước đó, các thuộc tính từ bảng trung gian có thể được truy cập trên model bằng cách sử dụng theo thuộc tính `pivot`. Tuy nhiên, bạn có thể tùy biến tên của thuộc tính này để phản ánh tốt hơn cho mục đích của bạn trong application.

Ví dụ: nếu application của bạn chứa user có thể subscribe podcast, bạn có thể có một quan hệ nhiều-nhiều giữa user và podcast. Nếu đây là trường hợp đó, bạn có thể muốn đổi tên truy cập vào bảng trung gian của bạn thành `subscription` thay vì `pivot`. Điều này có thể được thực hiện bằng cách sử dụng phương thức `as` khi định nghĩa quan hệ của bạn:

    return $this->belongsToMany(Podcast::class)
                    ->as('subscription')
                    ->withTimestamps();

Khi điều này được thực hiện xong, bạn có thể truy cập vào dữ liệu của bảng trung gian bằng tên tùy biến:

    $users = User::with('podcasts')->get();

    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }

<a name="filtering-queries-via-intermediate-table-columns"></a>
### Lọc bảng trung gian

Bạn cũng có thể lọc các kết quả được trả về bởi `belongsToMany` bằng cách sử dụng các phương thức `wherePivot`, `wherePivotIn`, `wherePivotNotIn`, `wherePivotBetween`, `wherePivotNotBetween`, `wherePivotNull`, và `wherePivotNotNull khi định nghĩa quan hệ:

    return $this->belongsToMany(Role::class)
                    ->wherePivot('approved', 1);

    return $this->belongsToMany(Role::class)
                    ->wherePivotIn('priority', [1, 2]);

    return $this->belongsToMany(Role::class)
                    ->wherePivotNotIn('priority', [1, 2]);

    return $this->belongsToMany(Podcast::class)
                    ->as('subscriptions')
                    ->wherePivotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

    return $this->belongsToMany(Podcast::class)
                    ->as('subscriptions')
                    ->wherePivotNotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

    return $this->belongsToMany(Podcast::class)
                    ->as('subscriptions')
                    ->wherePivotNull('expired_at');

    return $this->belongsToMany(Podcast::class)
                    ->as('subscriptions')
                    ->wherePivotNotNull('expired_at');

<a name="ordering-queries-via-intermediate-table-columns"></a>
### Sắp xếp thông qua bảng trung gian

Bạn có thể sắp xếp các kết quả được trả về bởi quan hệ `belongsToMany` bằng cách sử dụng phương thức `orderByPivot`. Trong ví dụ sau, chúng ta sẽ lấy ra tất cả các huy hiệu mới nhất của người dùng:

    return $this->belongsToMany(Badge::class)
                    ->where('rank', 'gold')
                    ->orderByPivot('created_at', 'desc');

<a name="defining-custom-intermediate-table-models"></a>
### Định nghĩa model trung gian

Nếu bạn muốn định nghĩa một model tùy biến, để biểu diễn bảng trung gian của quan hệ của bạn, bạn có thể gọi phương thức `using` khi định nghĩa quan hệ. Các model trung gian này cho bạn cơ hội để định nghĩa thêm các hành động trên model trung gian, như thêm phương thức và cast.

Để tuỳ biến một model pivot nhiều-nhiều bạn cần extend từ class `Illuminate\Database\Eloquent\Relations\Pivot`, còn nếu bạn muốn tuỳ biến model theo đa hình nhiều-nhiều, thì bạn cần extend từ class `Illuminate\Database\Eloquent\Relations\MorphPivot`. Ví dụ: chúng ta có thể định nghĩa một `Role` sử dụng model pivot `RoleUser` tùy biến như sau:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsToMany;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users(): BelongsToMany
        {
            return $this->belongsToMany(User::class)->using(RoleUser::class);
        }
    }

Khi định nghĩa model `RoleUser`, chúng ta sẽ extend nó từ class `Illuminate\Database\Eloquent\Relations\Pivot`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class RoleUser extends Pivot
    {
        // ...
    }

> [!WARNING]
> Các model pivot có thể không sử dụng trait `SoftDeletes`. Nếu bạn cần soft delete các bản ghi của model pivot, hãy xem xét chuyển đổi model pivot của bạn thành một model Eloquent thực tế.

<a name="custom-pivot-models-and-incrementing-ids"></a>
#### Custom Pivot Models And Incrementing IDs

Nếu bạn đã định nghĩa một quan hệ nhiều-nhiều sử dụng model pivot tùy chỉnh và model pivot đó có khóa chính tự động tăng, bạn nên đảm bảo là class model pivot tùy chỉnh của bạn đã định nghĩa thuộc tính `incrementing` là `true `.

    /**
     * Indicates if the IDs are auto-incrementing.
     *
     * @var bool
     */
    public $incrementing = true;

<a name="polymorphic-relationships"></a>
## Quan hệ đa hình

Quan hệ đa hình cho phép một model con thuộc về nhiều loại model khác nhau bằng cách sử dụng một kết nối duy nhất. Ví dụ: hãy tưởng tượng bạn đang xây dựng một ứng dụng cho phép người dùng chia sẻ các bài đăng và video trên blog. Trong một ứng dụng như vậy, model `Comment` có thể thuộc về cả model `Post` và `Video`.

<a name="one-to-one-polymorphic-relations"></a>
### Một - Một (đa hình)

<a name="one-to-one-polymorphic-table-structure"></a>
#### Table Structure

Quan hệ đa hình một - một tương tự như quan hệ một - một cơ bản; tuy nhiên, model con có thể thuộc về nhiều loại model khác nhau bằng một liên kết duy nhất. Ví dụ: một blog `Post` và một `User` có thể chia sẻ quan hệ với model `Image`. Sử dụng quan hệ đa hình 1-1 cho phép bạn có một table các hình ảnh duy nhất có thể liên kết với post trên blog và tài khoản user. Đầu tiên, hãy xem cấu trúc bảng sau:

    posts
        id - integer
        name - string

    users
        id - integer
        name - string

    images
        id - integer
        url - string
        imageable_id - integer
        imageable_type - string

Hãy chú ý đến các cột `imageable_id` và `imageable_type` trong bảng `images`. Cột `imageable_id` sẽ chứa giá trị ID của post hoặc user, trong khi cột `imageable_type` sẽ chứa tên class của model được kết nối. Cột `imageable_type` sẽ được sử dụng bởi Eloquent để xác định xem "loại" model nào sẽ trả về khi truy xuất quan hệ `imageable`. Trong trường hợp này, cột sẽ chứa `App\Models\Post` hoặc `App\Models\User`.

<a name="one-to-one-polymorphic-model-structure"></a>
#### Cấu trúc Model

Tiếp theo, hãy xem xét đến các định nghĩa model cần thiết để xây dựng quan hệ này:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphTo;

    class Image extends Model
    {
        /**
         * Get the parent imageable model (user or post).
         */
        public function imageable(): MorphTo
        {
            return $this->morphTo();
        }
    }

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphOne;

    class Post extends Model
    {
        /**
         * Get the post's image.
         */
        public function image(): MorphOne
        {
            return $this->morphOne(Image::class, 'imageable');
        }
    }

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphOne;

    class User extends Model
    {
        /**
         * Get the user's image.
         */
        public function image(): MorphOne
        {
            return $this->morphOne(Image::class, 'imageable');
        }
    }

<a name="one-to-one-polymorphic-retrieving-the-relationship"></a>
#### Lấy qua quan hệ

Khi bảng cơ sở dữ liệu và các model của bạn đã được xác định xong, bạn có thể lấy ra các quan hệ của bạn thông qua các model. Ví dụ, để lấy ra image cho một post, chúng ta có thể truy cập vào thuộc tính quan hệ động `image`:

    use App\Models\Post;

    $post = Post::find(1);

    $image = $post->image;

Bạn có thể lấy ra model cha của model đa hình bằng cách truy cập vào tên của phương thức mà thực hiện lệnh gọi đến `morphTo`. Trong trường hợp này, đó là phương thức `imageable` trên model `Image`. Vì vậy, chúng ta sẽ truy cập vào phương thức đó dưới dạng một thuộc tính quan hệ động như sau:

    use App\Models\Image;

    $image = Image::find(1);

    $imageable = $image->imageable;

Quan hệ `imageable` trên model `Image` sẽ trả về một instance `Post` hoặc `User`, tùy thuộc vào loại model mà sở hữu image đó.

<a name="morph-one-to-one-key-conventions"></a>
#### Key Conventions

Nếu cần, bạn có thể chỉ định tên cho cột "id" và cột "type" được model con đa hình của bạn sử dụng. Nếu bạn làm như vậy, hãy đảm bảo là bạn đã truyền tên của quan hệ làm tham số đầu tiên cho phương thức `morphTo`. Thông thường, giá trị này phải khớp với tên phương thức, vì vậy bạn có thể sử dụng hằng số `__FUNCTION__` của PHP:

    /**
     * Get the model that the image belongs to.
     */
    public function imageable(): MorphTo
    {
        return $this->morphTo(__FUNCTION__, 'imageable_type', 'imageable_id');
    }

<a name="one-to-many-polymorphic-relations"></a>
### Một - Nhiều (đa hình)

<a name="one-to-many-polymorphic-table-structure"></a>
#### Table Structure

Quan hệ đa hình một-nhiều sẽ giống với quan hệ một-nhiều bình thường; tuy nhiên model con cho phép nhiều hơn một model bằng một liên kết. Ví dụ: hãy tưởng tượng user trong application của bạn có thể "comment" cả post và video. Sử dụng các quan hệ đa hình, bạn có thể sử dụng bảng `comments` để chứa comment cho cả post và video. Trước tiên, hãy xem cấu trúc bảng cần thiết để xây dựng quan hệ này:

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

<a name="one-to-many-polymorphic-model-structure"></a>
#### Cấu trúc Model

Tiếp theo, hãy xem các định nghĩa model cần thiết để xây dựng quan hệ này:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphTo;

    class Comment extends Model
    {
        /**
         * Get the parent commentable model (post or video).
         */
        public function commentable(): MorphTo
        {
            return $this->morphTo();
        }
    }

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphMany;

    class Post extends Model
    {
        /**
         * Get all of the post's comments.
         */
        public function comments(): MorphMany
        {
            return $this->morphMany(Comment::class, 'commentable');
        }
    }

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphMany;

    class Video extends Model
    {
        /**
         * Get all of the video's comments.
         */
        public function comments(): MorphMany
        {
            return $this->morphMany(Comment::class, 'commentable');
        }
    }

<a name="one-to-many-polymorphic-retrieving-the-relationship"></a>
#### Lấy quan hệ

Khi bảng cơ sở dữ liệu và model của bạn đã được định nghĩa xong, bạn có thể truy cập vào quan hệ này thông qua các thuộc tính quan hệ của model. Ví dụ: để truy cập đến tất cả các comment cho một post, chúng ta có thể sử dụng thuộc tính động `comments`:

    use App\Models\Post;

    $post = Post::find(1);

    foreach ($post->comments as $comment) {
        // ...
    }

Bạn cũng có thể lấy ra cha của một quan hệ đa hình từ một model đa hình con bằng cách truy cập vào tên của phương thức mà thực hiện lệnh gọi tới `morphTo`. Trong trường hợp này, đó là phương thức `commentable` trên model `Comment`. Vì vậy, chúng ta sẽ truy cập vào phương thức đó như là một thuộc tính quan hệ động để truy cập model cha của comment:

    use App\Models\Comment;

    $comment = Comment::find(1);

    $commentable = $comment->commentable;

Quan hệ `commentable` trên model `Comment` sẽ trả về một instance `Post` hoặc `Video`, tùy thuộc vào loại model là cha của comment.

<a name="one-of-many-polymorphic-relations"></a>
### Một trong nhiều (Polymorphic)

Thỉnh thoảng một model có thể có nhiều model quan hệ, nhưng bạn chỉ muốn lấy ra model quan hệ "mới nhất" hoặc "cũ nhất" của quan hệ đó. Ví dụ: model `User` có thể quan hệ đến nhiều model `Image`, nhưng bạn muốn định nghĩa một cách tương tác với những hình ảnh gần đây nhất mà người dùng đã tải lên. Bạn có thể thực hiện việc này bằng cách sử dụng quan hệ `morphOne` kết hợp với các phương thức `ofMany`:

```php
/**
 * Get the user's most recent image.
 */
public function latestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->latestOfMany();
}
```

Tương tự như vậy, bạn có thể định nghĩa một phương thức để lấy ra model quan hệ "cũ nhất" hoặc đầu tiên của một quan hệ:

```php
/**
 * Get the user's oldest image.
 */
public function oldestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->oldestOfMany();
}
```

By default, the `latestOfMany` and `oldestOfMany` methods will retrieve the latest or oldest related model based on the model's primary key, which must be sortable. However, sometimes you may wish to retrieve a single model from a larger relationship using a different sorting criteria.

Ví dụ: sử dụng phương thức `ofMany`, bạn có thể lấy ra hình ảnh nhiều like nhất của người dùng. Phương thức `ofMany` chấp nhận cột có thể sắp xếp làm tham số đầu tiên của nó và hàm tính toán nào (`min` hoặc `max`) được sử dụng khi truy vấn model quan hệ:

```php
/**
 * Get the user's most popular image.
 */
public function bestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->ofMany('likes', 'max');
}
```

> [!NOTE]
> Có thể xây dựng các quan hệ "một trong nhiều" nâng cao. Để biết thêm thông tin, vui lòng tham khảo [tài liệu một trong nhiều](#advanced-has-one-of-many-relationships).

<a name="many-to-many-polymorphic-relations"></a>
### Nhiều - Nhiều (đa hình)

<a name="many-to-many-polymorphic-table-structure"></a>
#### Table Structure

Quan hệ đa hình nhiều-nhiều phức tạp hơn một chút so với quan hệ "morph one" và "morph many". Ví dụ: một model `Post` và một model `Video` có thể dùng quan hệ đa hình với một model `Tag`. Sử dụng quan hệ đa hình nhiều-nhiều trong tình huống này sẽ cho phép ứng dụng của chúng ta có một bảng các tag duy nhất có thể được liên kết với post hoặc video. Đầu tiên, hãy xem qua cấu trúc bảng cần thiết để xây dựng quan hệ này:

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

> [!NOTE]
> Trước khi đi sâu hơn vào quan hệ nhiều-nhiều đa hình, bạn có thể đọc tài liệu về [quan hệ nhiều-nhiều](#many-to-many).

<a name="many-to-many-polymorphic-model-structure"></a>
#### Cấu trúc Model

Tiếp theo, chúng ta đã sẵn sàng để định nghĩa các quan hệ cho các model. Các model `Post` và `Video` đều sẽ chứa một phương thức `tags` gọi đến phương thức `morphToMany` được cung cấp bởi class model Eloquent.

Phương thức `morphToMany` chấp nhận tên của model quan hệ cũng như "tên quan hệ". Dựa vào tên mà chúng ta đã đặt cho tên bảng trung gian và các khóa chứa trong nó, chúng ta sẽ coi quan hệ là "taggable":

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphToMany;

    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags(): MorphToMany
        {
            return $this->morphToMany(Tag::class, 'taggable');
        }
    }

<a name="many-to-many-polymorphic-defining-the-inverse-of-the-relationship"></a>
#### Defining The Inverse Of The Relationship

Tiếp theo, trên model `Tag`, bạn sẽ định nghĩa một phương thức cho từng model cha của nó. Vì vậy, trong ví dụ này, chúng ta sẽ định nghĩa một phương thức `posts` và một phương thức `videos`. Cả hai phương thức này sẽ trả về kết quả của phương thức `morphedByMany`.

Phương thức `morphedByMany` chấp nhận tên của model quan hệ cũng như "tên quan hệ". Dựa vào tên mà chúng ta đã đặt cho tên bảng trung gian và các khóa chứa trong nó, chúng ta sẽ coi quan hệ là "taggable":

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphToMany;

    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts(): MorphToMany
        {
            return $this->morphedByMany(Post::class, 'taggable');
        }

        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos(): MorphToMany
        {
            return $this->morphedByMany(Video::class, 'taggable');
        }
    }

<a name="many-to-many-polymorphic-retrieving-the-relationship"></a>
#### Lấy qua quan hệ

Khi các bảng và các model của bạn đã được định nghĩa xong, bạn có thể truy cập vào các quan hệ này thông qua model của bạn. Ví dụ: để truy cập vào tất cả các tag của một post, bạn có thể sử dụng thuộc tính quan hệ động `tags`:

    use App\Models\Post;

    $post = Post::find(1);

    foreach ($post->tags as $tag) {
        // ...
    }

Bạn có thể lấy ra cha của một quan hệ đa hình từ model đa hình con bằng cách truy cập vào tên của phương thức mà thực hiện lệnh gọi tới `morphedByMany`. Trong trường hợp này, đó là các phương thức `posts` hoặc `videos` trên model `Tag`:

    use App\Models\Tag;

    $tag = Tag::find(1);

    foreach ($tag->posts as $post) {
        // ...
    }

    foreach ($tag->videos as $video) {
        // ...
    }

<a name="custom-polymorphic-types"></a>
### Custom Polymorphic Types

Mặc định, Laravel sẽ sử dụng tên của class để lưu vào "loại" model cho quan hệ này. Chẳng hạn, như ví dụ như quan hệ một-nhiều ở trên, một model `Comment` có thể thuộc về một model `Post` hoặc một model `Video`, mặc định cột `commentable_type` sẽ phải lưu một giá trị là `App\Models\Post` hoặc `App\Models\Video`. Tuy nhiên, nếu bạn muốn tách những giá trị này ra khỏi cấu trúc thư mục trong application của bạn.

Ví dụ: thay vì sử dụng tên model như "type", chúng ta có thể sử dụng các string đơn giản như `post` và `video`. Bằng cách đó, các giá trị cột "type" đa hình trong cơ sở dữ liệu của chúng ta sẽ vẫn hợp lệ ngay cả khi các model đã được đổi tên:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::enforceMorphMap([
        'post' => 'App\Models\Post',
        'video' => 'App\Models\Video',
    ]);

Bạn có thể gọi phương thức `enforceMorphMap` trong phương thức `boot` của class `App\Providers\AppServiceProvider` hoặc tạo một service provider riêng nếu bạn muốn.

Bạn có thể xác định bí danh morph của một model trong khi ứng dụng đang chạy bằng cách sử dụng phương thức `getMorphClass` của model. Ngược lại, bạn cũng có thể xác định tên class được liên kết với bí danh morph bằng cách sử dụng phương thức `Relation::getMorphedModel`:

    use Illuminate\Database\Eloquent\Relations\Relation;

    $alias = $post->getMorphClass();

    $class = Relation::getMorphedModel($alias);

> [!WARNING]
> Khi thêm một "morph map" vào ứng dụng hiện có của bạn, mọi giá trị của cột morphable `*_type` trong cơ sở dữ liệu của bạn vẫn sẽ chứa tên đầy đủ của class đó và nó sẽ cần được chuyển đổi thành tên "map" của nó.

<a name="dynamic-relationships"></a>
### Quan hệ động

Bạn có thể sử dụng phương thức `resolveRelationUsing` để định nghĩa các quan hệ giữa các model Eloquent trong khi ứng dụng đang chạy. Mặc dù phương thức này thường không được khuyến nghị khi phát triển ứng dụng thông thường, nhưng điều này đôi khi có thể hữu ích khi phát triển các package Laravel.

Phương thức `resolveRelationUsing` chấp nhận tên quan hệ mong muốn làm tham số đầu tiên của nó. Tham số thứ hai được truyền cho phương thức phải là một closure chấp nhận instance của model đó và trả về một định nghĩa quan hệ Eloquent hợp lệ. Thông thường, bạn nên cấu hình các quan hệ động trong phương thức boot của [service provider](/docs/{{version}}/providers):

    use App\Models\Order;
    use App\Models\Customer;

    Order::resolveRelationUsing('customer', function (Order $orderModel) {
        return $orderModel->belongsTo(Customer::class, 'customer_id');
    });

> [!WARNING]
> Khi định nghĩa quan hệ động, hãy luôn đảm bảo là bạn đã cung cấp các tham số tên khóa cho các phương thức quan hệ Eloquent.

<a name="querying-relations"></a>
## Query theo quan hệ

Vì tất cả các quan hệ Eloquent đều được định nghĩa thông qua phương thức, nên bạn có thể gọi các phương thức đó để lấy ra các instance của quan hệ mà không cần phải thực hiện một query to load the related models. Ngoài ra, tất cả các quan hệ Eloquent cũng đóng vai trò như là một [query builders](/docs/{{version}}/queries), cho phép bạn tiếp tục thêm các ràng buộc cho các truy vấn trước khi được thực thi truy vấn vào trong cơ sở dữ liệu của bạn.

Ví dụ, hãy tưởng tượng một application blog trong đó có model `User` liên kết với nhiều model `Post`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasMany;

    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts(): HasMany
        {
            return $this->hasMany(Post::class);
        }
    }

Bạn có thể truy vấn quan hệ `post` và thêm các ràng buộc cho quan hệ này như sau:

    use App\Models\User;

    $user = User::find(1);

    $user->posts()->where('active', 1)->get();

Bạn cũng có thể sử dụng bất kỳ phương thức nào của [query builder](/docs/{{version}}/queries) của Laravel trên quan hệ này, vì vậy hãy chắc chắn là bạn đã xem qua tài liệu của query builder để tìm hiểu về tất cả các phương thức có sẵn cho bạn.

<a name="chaining-orwhere-clauses-after-relationships"></a>
#### Chaining `orWhere` Clauses After Relationships

Như đã trình bày trong ví dụ trên, bạn có thể thoải mái thêm các ràng buộc cho các quan hệ khi truy vấn chúng. Tuy nhiên, hãy cẩn trọng khi kết hợp các mệnh đề `orWhere` vào một quan hệ, vì các mệnh đề` orWhere` sẽ được nhóm ở cùng cấp với ràng buộc quan hệ:

    $user->posts()
            ->where('active', 1)
            ->orWhere('votes', '>=', 100)
            ->get();

Ví dụ trên sẽ tạo SQL sau. Như bạn có thể thấy, mệnh đề `or` sẽ hướng dẫn truy vấn trả về _bất kỳ_ post nào có hơn 100 vote. Truy vấn không còn bị ràng buộc với một user cụ thể:

```sql
select *
from posts
where user_id = ? and active = 1 or votes >= 100
```

Trong hầu hết các trường hợp, bạn nên sử dụng [nhóm logic](/docs/{{version}}/queries#logical-grouping) để nhóm các kiểm tra có điều kiện giữa các dấu ngoặc đơn:

    use Illuminate\Database\Eloquent\Builder;

    $user->posts()
            ->where(function (Builder $query) {
                return $query->where('active', 1)
                             ->orWhere('votes', '>=', 100);
            })
            ->get();

Ví dụ trên sẽ tạo ra SQL sau. Lưu ý rằng nhóm logic đã nhóm các ràng buộc và truy vấn tới một người dùng cụ thể:

```sql
select *
from posts
where user_id = ? and (active = 1 or votes >= 100)
```

<a name="relationship-methods-vs-dynamic-properties"></a>
### Phương thức quan hệ và thuộc tính động

Nếu bạn không cần thêm các ràng buộc cho truy vấn quan hệ Eloquent, bạn có thể truy cập vào quan hệ như thể đó là một thuộc tính. Ví dụ: chúng ta sẽ tiếp tục sử dụng các model `User` và `Post` như ở trên, chúng ta có thể truy cập vào tất cả các post của một user như sau:

    use App\Models\User;

    $user = User::find(1);

    foreach ($user->posts as $post) {
        // ...
    }

Thuộc tính quan hệ động thực hiện một "lazy loading", nghĩa là chúng sẽ chỉ load dữ liệu quan hệ khi bạn thực sự truy cập đến chúng. Do đó, các nhà phát triển thường sử dụng [eager loading](#eager-loading) để load trước các quan hệ mà họ biết là sẽ được truy cập vào sau khi một model được load. Eager loading sẽ cung cấp một cách hiệu quả để giảm số lượng truy vấn SQL phải được thực thi để load các quan hệ của một model.

<a name="querying-relationship-existence"></a>
### Query quan hệ tồn tại

Khi retrieving model records, bạn có thể muốn giới hạn kết quả của bạn nhận được dựa trên sự tồn tại của một quan hệ. Ví dụ, hãy tưởng tượng bạn muốn lấy tất cả các post trên blog có ít nhất là một comment. Để làm như vậy, bạn có thể truyền tên của quan hệ cho các phương thức `has` hoặc `orHas`:

    use App\Models\Post;

    // Retrieve all posts that have at least one comment...
    $posts = Post::has('comments')->get();

Bạn cũng có thể khai báo thêm các toán tử và số lượng để tùy biến thêm cho các truy vấn này:

    // Retrieve all posts that have three or more comments...
    $posts = Post::has('comments', '>=', 3)->get();

Các câu lệnh `has` lồng nhau có thể được khởi tạo bằng cách sử dụng ký tự "chấm". Ví dụ: bạn có thể lấy ra tất cả các post có ít nhất một comment mà có ít nhất một hình ảnh:

    // Retrieve posts that have at least one comment with images...
    $posts = Post::has('comments.images')->get();

Nếu bạn cần nhiều hơn thế nữa, bạn có thể sử dụng các phương thức `whereHas` hoặc `orWhereHas` để định nghĩa thêm các ràng buộc truy vấn đối với các truy vấn `has` của bạn, chẳng hạn như kiểm tra nội dung của một comment:

    use Illuminate\Database\Eloquent\Builder;

    // Retrieve posts with at least one comment containing words like code%...
    $posts = Post::whereHas('comments', function (Builder $query) {
        $query->where('content', 'like', 'code%');
    })->get();

    // Retrieve posts with at least ten comments containing words like code%...
    $posts = Post::whereHas('comments', function (Builder $query) {
        $query->where('content', 'like', 'code%');
    }, '>=', 10)->get();

> [!WARNING]
> Eloquent hiện không hỗ trợ truy vấn quan hệ có tồn tại trong các cơ sở dữ liệu hay không. Các quan hệ phải tồn tại trong cùng một cơ sở dữ liệu.

<a name="inline-relationship-existence-queries"></a>
#### Inline Relationship Existence Queries

Nếu bạn muốn truy vấn sự tồn tại của một quan hệ bằng một điều kiện where đơn giản, thì bạn có thể thấy thuận tiện hơn khi sử dụng các phương thức `whereRelation`, `orWhereRelation`, `whereMorphRelation`, và `orWhereMorphRelation`. Ví dụ: chúng ta có thể truy vấn tất cả các post có comment chưa được chấp nhận:

    use App\Models\Post;

    $posts = Post::whereRelation('comments', 'is_approved', false)->get();

Tất nhiên, giống như tất cả các lệnh gọi đến phương thức `where` của query builder, bạn cũng có thể chỉ định một toán tử:

    $posts = Post::whereRelation(
        'comments', 'created_at', '>=', now()->subHour()
    )->get();

<a name="querying-relationship-absence"></a>
### Query quan hệ không tồn tại

Khi retrieving model records, bạn có thể muốn giới hạn kết quả nhận được dựa trên việc có hoặc không có bản ghi quan hệ. Ví dụ: hãy tưởng tượng bạn muốn lấy tất cả các post trên một blog mà **không** có bất kỳ comment nào. Để làm như vậy, bạn có thể truyền tên của quan hệ cho các phương thức `doesntHave` hoặc `orDoesntHave`:

    use App\Models\Post;

    $posts = Post::doesntHave('comments')->get();

Nếu bạn cần nhiều hơn thế nữa, bạn có thể sử dụng các phương thức `whereDoesntHave` và `orWhereDoesntHave` để thêm các ràng buộc truy vấn vào các truy vấn `doesntHave` của bạn, chẳng hạn như kiểm tra nội dung của một comment:

    use Illuminate\Database\Eloquent\Builder;

    $posts = Post::whereDoesntHave('comments', function (Builder $query) {
        $query->where('content', 'like', 'code%');
    })->get();

Bạn có thể sử dụng ký hiệu "dấu chấm" để thực hiện truy vấn các quan hệ lồng nhau. Ví dụ: truy vấn sau sẽ lấy ra tất cả các bài đăng không có nhận xét; tuy nhiên, các bài đăng đó có các nhận xét từ các tác giả không bị cấm sẽ được đưa vào kết quả:

    use Illuminate\Database\Eloquent\Builder;

    $posts = Post::whereDoesntHave('comments.author', function (Builder $query) {
        $query->where('banned', 0);
    })->get();

<a name="querying-morph-to-relationships"></a>
### Query quan hệ đa hình

Để truy vấn sự tồn tại của quan hệ "morph to", bạn có thể sử dụng phương thức `whereHasMorph` and `whereDoesntHaveMorph`. Các phương thức này chấp nhận tên của quan hệ làm tham số đầu tiên của chúng. Tiếp theo, các phương thức chấp nhận tên của các model quan hệ mà bạn muốn đưa vào truy vấn. Cuối cùng, bạn có thể cung cấp một closure để tùy chỉnh truy vấn quan hệ:

    use App\Models\Comment;
    use App\Models\Post;
    use App\Models\Video;
    use Illuminate\Database\Eloquent\Builder;

    // Retrieve comments associated to posts or videos with a title like code%...
    $comments = Comment::whereHasMorph(
        'commentable',
        [Post::class, Video::class],
        function (Builder $query) {
            $query->where('title', 'like', 'code%');
        }
    )->get();

    // Retrieve comments associated to posts with a title not like code%...
    $comments = Comment::whereDoesntHaveMorph(
        'commentable',
        Post::class,
        function (Builder $query) {
            $query->where('title', 'like', 'code%');
        }
    )->get();

Đôi khi, bạn có thể cần thêm các ràng buộc truy vấn dựa trên "loại" của model đa hình quan hệ. Closure được truyền vào phương thức `whereHasMorph` có thể nhận giá trị `$type` làm tham số thứ hai. Tham số này cho phép bạn kiểm tra "loại" truy vấn đang được tạo:

    use Illuminate\Database\Eloquent\Builder;

    $comments = Comment::whereHasMorph(
        'commentable',
        [Post::class, Video::class],
        function (Builder $query, string $type) {
            $column = $type === Post::class ? 'content' : 'title';

            $query->where($column, 'like', 'code%');
        }
    )->get();

<a name="querying-all-morph-to-related-models"></a>
#### Querying All Related Models

Thay vì phải truyền một mảng các model đa hình, bạn có thể cung cấp ký tự `*` làm giá trị đại diện. Điều này sẽ hướng dẫn Laravel lấy ra tất cả model đa hình có thể có từ cơ sở dữ liệu. Laravel sẽ thực hiện thêm một truy vấn bổ sung để thực hiện thao tác này:

    use Illuminate\Database\Eloquent\Builder;

   $comments = Comment::whereHasMorph('commentable', '*', function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    })->get();

<a name="aggregating-related-models"></a>
## Tính toán model quan hệ

<a name="counting-related-models"></a>
### Đếm các bản ghi theo quan hệ model

Thỉnh thoảng bạn có thể muốn đếm số lượng model quan hệ cho một quan hệ nhất định mà không cần load ra các model. Để thực hiện điều này, bạn có thể sử dụng phương thức `withCount`. Phương thức `withCount` sẽ set thuộc tính `{relation}_count` trong các model kết quả:

    use App\Models\Post;

    $posts = Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

Bằng cách truyền một mảng cho phương thức `withCount`, bạn cũng có thể thêm "counts" cho nhiều quan hệ khác cũng như thêm các ràng buộc bổ sung cho các câu lệnh truy vấn:

    use Illuminate\Database\Eloquent\Builder;

    $posts = Post::withCount(['votes', 'comments' => function (Builder $query) {
        $query->where('content', 'like', 'code%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

Bạn cũng có thể thêm tên gọi khác cho một kết quả đếm quan hệ, cho phép bạn thực hiện nhiều lần đếm trên cùng một quan hệ:

    use Illuminate\Database\Eloquent\Builder;

    $posts = Post::withCount([
        'comments',
        'comments as pending_comments_count' => function (Builder $query) {
            $query->where('approved', false);
        },
    ])->get();

    echo $posts[0]->comments_count;
    echo $posts[0]->pending_comments_count;

<a name="deferred-count-loading"></a>
#### Deferred Count Loading

Bằng cách sử dụng phương thức `loadCount`, bạn có thể đếm số lượng bản ghi trong các quan hệ sau khi model cha đã được lấy ra:

    $book = Book::first();

    $book->loadCount('genres');

Nếu bạn cần set thêm các ràng buộc truy vấn cho các truy vấn đếm số lượng, bạn có thể truyền một mảng gồm các khóa là các tên của các quan hệ mà bạn muốn đếm. Các giá trị của mảng phải là các instance closure nhận vào một instance query builder:

    $book->loadCount(['reviews' => function (Builder $query) {
        $query->where('rating', 5);
    }])

<a name="relationship-counting-and-custom-select-statements"></a>
#### Relationship Counting & Custom Select Statements

Nếu bạn đang kết hợp `withCount` với câu lệnh `select`, hãy đảm bảo là bạn đang gọi phương thức `withCount` sau phương thức `select`:

    $posts = Post::select(['title', 'body'])
                    ->withCount('comments')
                    ->get();

<a name="other-aggregate-functions"></a>
### Các hàm tính toán khác

Ngoài phương thức `withCount`, Eloquent còn cung cấp các phương thức `withMin`, `withMax`, `withAvg`, `withSum` và `withExists`. Các phương thức này sẽ set thuộc tính `{relation}_{function}_{column}` trong các model kết quả của bạn:

    use App\Models\Post;

    $posts = Post::withSum('comments', 'votes')->get();

    foreach ($posts as $post) {
        echo $post->comments_sum_votes;
    }

Nếu bạn muốn truy cập vào kết quả của hàm tính toán bằng các tên khác, bạn có thể chỉ định bí danh của riêng bạn:

    $posts = Post::withSum('comments as total_comments', 'votes')->get();

    foreach ($posts as $post) {
        echo $post->total_comments;
    }

Giống như phương thức `loadCount`, các phiên bản load sau của các phương thức này cũng có sẵn. Các tính toán tổng hợp bổ sung này có thể được thực hiện trên các model Eloquent đã có sẵn:

    $post = Post::first();

    $post->loadSum('comments', 'votes');

Nếu bạn đang kết hợp các phương thức tính toán này với câu lệnh `select`, hãy đảm bảo là bạ đang gọi các phương thức tính toán này sau phương thức `select`:

    $posts = Post::select(['title', 'body'])
                    ->withExists('comments')
                    ->get();

<a name="counting-related-models-on-morph-to-relationships"></a>
### Đếm model quan hệ trên quan hệ đa hình

Nếu bạn muốn eager load một quan hệ "morph to", cũng như đếm số lượng quan hệ của model đó cho các thực thể khác nhau có thể được trả về bởi quan hệ đó, bạn có thể sử dụng phương thức `with` kết hợp với phương thức `morphWithCount` của quan hệ `morphTo`.

Trong ví dụ này, giả sử rằng model `Photo` và `Post` có thể tạo model `ActivityFeed`. Chúng ta sẽ giả sử rằng model `ActivityFeed` định nghĩa một quan hệ "morph to" có tên là `parentable` cho phép chúng ta lấy ra model `Photo` hoặc `Post` cho một instance `ActivityFeed` nhất định.  Ngoài ra, giả sử rằng model `Photo` "có nhiều" model `Tag` và model `Post` cũng "có nhiều" model `Comment`.

Bây giờ, hãy tưởng tượng chúng ta muốn lấy ra các instances `ActivityFeed` và eager load các model cha `parentable` cho mỗi instances `ActivityFeed`. Ngoài ra, chúng ta muốn lấy ra số lượng các tag được liên kết với mỗi photo và số lượng comment được liên kết với mỗi bài post:

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $activities = ActivityFeed::with([
        'parentable' => function (MorphTo $morphTo) {
            $morphTo->morphWithCount([
                Photo::class => ['tags'],
                Post::class => ['comments'],
            ]);
        }])->get();

<a name="morph-to-deferred-count-loading"></a>
#### Deferred Count Loading

Giả sử là chúng ta đã lấy ra được một collection các model `ActivityFeed` và bây giờ chúng ta muốn load số lượng quan hệ lồng nhau cho các model `parentable` khác nhau được liên kết đến các activity feed. Bạn có thể sử dụng phương thức `loadMorphCount` để thực hiện điều này:

    $activities = ActivityFeed::with('parentable')->get();

    $activities->loadMorphCount('parentable', [
        Photo::class => ['tags'],
        Post::class => ['comments'],
    ]);

<a name="eager-loading"></a>
## Eager Loading

Khi truy cập vào các quan hệ Eloquent dưới dạng các thuộc tính, các model quan hệ là "lazy loaded". Điều này có nghĩa là dữ liệu quan hệ sẽ không được load cho đến khi bạn truy cập vào chúng lần đầu tiên. Tuy nhiên, Eloquent có thể "eager load" quan hệ tại thời điểm bạn truy vấn vào model cha. Eager loading sẽ làm giảm bớt vấn đề truy vấn "N + 1". Để minh họa cho vấn đề truy vấn N + 1, hãy xem một model `Book` mà "belongs to" đến một model `Author`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsTo;

    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author(): BelongsTo
        {
            return $this->belongsTo(Author::class);
        }
    }

Bây giờ, hãy lấy ra tất cả các sách và tác giả của chúng:

    use App\Models\Book;

    $books = Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Vòng lặp này sẽ thực hiện một truy vấn để lấy ra tất cả các sách có trong bảng cơ sở dữ liệu, rồi sau đó thực hiện một truy vấn khác cho mỗi cuốn sách để lấy ra tác giả của cuốn sách đó. Vì vậy, nếu chúng ta có 25 cuốn sách, thì code trên sẽ chạy 26 truy vấn: một truy vấn sẽ lấy ra tất cả các cuốn sách và 25 truy vấn còn lại để lấy ra tác giả của mỗi cuốn sách.

Rất may, chúng ta có thể sử dụng eager loading để giảm các hành động truy vấn này xuống chỉ còn hai truy vấn. Khi tạo một truy vấn, bạn có thể khai báo những quan hệ nào sẽ được eager loading bằng phương thức `with`:

    $books = Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Đối với cách làm này, chỉ có hai truy vấn sẽ được thực hiện - một truy vấn để lấy ra tất cả các sách và một truy vấn để lấy ra tất cả các tác giả của tất cả các sách đó:

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

<a name="eager-loading-multiple-relationships"></a>
#### Eager Loading Multiple Relationships

Thỉnh thoảng bạn có thể cần eager load nhiều quan hệ khác nhau. Để làm như vậy, chỉ cần truyền thêm một mảng quan hệ cho phương thức `with`:

    $books = Book::with(['author', 'publisher'])->get();

<a name="nested-eager-loading"></a>
#### Nested Eager Loading

Để eager load một quan hệ trong một quan hệ, bạn có thể sử dụng cú pháp "chấm". Ví dụ: hãy eager load tất cả các tác giả của một cuốn sách và tất cả các liên hệ của tác giả đó:

    $books = App\Book::with('author.contacts')->get();

Ngoài ra, bạn có thể chỉ định các quan hệ sẽ được eager loading lồng nhau bằng cách cung cấp một mảng lồng nhau cho phương thức `with`, điều này có thể thuận tiện khi eager load nhiều quan hệ lồng nhau:

    $books = Book::with([
        'author' => [
            'contacts',
            'publisher',
        ],
    ])->get();

<a name="nested-eager-loading-morphto-relationships"></a>
#### Nested Eager Loading `morphTo` Relationships

Nếu bạn muốn eager loading một quan hệ `morphTo`, cũng như các quan hệ lồng nhau trên các thực thể khác nhau có thể được trả về bởi quan hệ đó, bạn có thể sử dụng phương thức `with` kết hợp với phương thức `morphWith` của quan hệ `morphTo`. Để giúp minh họa cho phương thức này, chúng ta hãy xem model sau:

    <?php

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphTo;

    class ActivityFeed extends Model
    {
        /**
         * Get the parent of the activity feed record.
         */
        public function parentable(): MorphTo
        {
            return $this->morphTo();
        }
    }

Trong ví dụ này, giả sử các model `Event`, `Photo`, và `Post` có thể tạo model `ActivityFeed`. Ngoài ra, giả sử rằng model `Event` thuộc một model `Calendar`, model `Photo` được liên kết với model `Tag` và model `Post` thuộc model `Author`.

Sử dụng các định nghĩa và quan hệ của model này, chúng ta có thể truy xuất các instance model của `ActivityFeed` và eager loading tất cả các model `parentable` và các quan hệ lồng nhau tương ứng của chúng:

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $activities = ActivityFeed::query()
        ->with(['parentable' => function (MorphTo $morphTo) {
            $morphTo->morphWith([
                Event::class => ['calendar'],
                Photo::class => ['tags'],
                Post::class => ['author'],
            ]);
        }])->get();

<a name="eager-loading-specific-columns"></a>
#### Eager Loading Specific Columns

Bạn có thể không phải lúc nào cũng cần mọi cột của quan hệ mà bạn đang truy xuất. Vì lý do này, Eloquent cho phép bạn khai báo các cột của quan hệ mà bạn muốn lấy ra:

    $books = Book::with('author:id,name,book_id')->get();

> [!WARNING]
> Khi sử dụng tính năng này, bạn phải luôn thêm cột `id` và bất kỳ cột khóa ngoại nào có liên quan trong danh sách các cột mà bạn muốn truy xuất.

<a name="eager-loading-by-default"></a>
#### Eager Loading By Default

Thỉnh thoảng bạn có thể muốn luôn load một số quan hệ khi truy xuất vào một model. Để thực hiện điều này, bạn có thể định nghĩa một thuộc tính `$with` trong model:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsTo;

    class Book extends Model
    {
        /**
         * The relationships that should always be loaded.
         *
         * @var array
         */
        protected $with = ['author'];

        /**
         * Get the author that wrote the book.
         */
        public function author(): BelongsTo
        {
            return $this->belongsTo(Author::class);
        }

        /**
         * Get the genre of the book.
         */
        public function genre(): BelongsTo
        {
            return $this->belongsTo(Genre::class);
        }
    }

Nếu bạn muốn xóa một quan hệ ra khỏi thuộc tính `$with` cho một truy vấn nhất định, bạn có thể sử dụng phương thức `without`:

    $books = Book::without('author')->get();

Nếu bạn muốn ghi đè tất cả các item có trong thuộc tính `$with` cho một truy vấn nhất định, bạn có thể sử dụng phương thức `withOnly`:

    $books = Book::withOnly('genre')->get();

<a name="constraining-eager-loads"></a>
### Rằng buộc khi eager loading

Thỉnh thoảng bạn có thể muốn eager load một quan hệ, nhưng cũng muốn khai báo thêm các điều kiện truy vấn cho quan hệ eager load đó. Bạn có thể thực hiện điều này bằng cách truyền một mảng các quan hệ cho phương thức `with` trong đó khóa mảng là tên quan hệ và giá trị mảng là một closure có thêm các ràng buộc bổ sung cho truy vấn eager loading:

    use App\Models\User;
    use Illuminate\Contracts\Database\Eloquent\Builder;

    $users = User::with(['posts' => function (Builder $query) {
        $query->where('title', 'like', '%code%');
    }])->get();

Trong ví dụ này, Eloquent sẽ chỉ eager load các post mà trong đó cột `title` của post sẽ chứa từ `code`. Bạn có thể gọi các phương thức [query builder](/docs/{{version}}/queries) khác để tùy biến thêm cho thao tác eager loading:

    $users = User::with(['posts' => function (Builder $query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

> [!WARNING]
> Phương thức query builder `limit` và `take` có thể không sử dụng được khi bạn đang eager loading.

<a name="constraining-eager-loading-of-morph-to-relationships"></a>
#### Constraining Eager Loading Of `morphTo` Relationships

Nếu bạn muốn eager loading một quan hệ `morphTo`, Eloquent sẽ chạy nhiều truy vấn để tìm nạp từng loại model quan hệ. Bạn có thể thêm các ràng buộc bổ sung cho từng truy vấn này bằng cách sử dụng phương thức `constrain` của quan hệ `MorphTo`:

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $comments = Comment::with(['commentable' => function (MorphTo $morphTo) {
        $morphTo->constrain([
            Post::class => function ($query) {
                $query->whereNull('hidden_at');
            },
            Video::class => function ($query) {
                $query->where('type', 'educational');
            },
        ]);
    }])->get();

Trong ví dụ này, Eloquent sẽ chỉ eager load các bài post chưa bị ẩn và video mà có giá trị `type` là "educational".

<a name="constraining-eager-loads-with-relationship-existence"></a>
#### Constraining Eager Loads With Relationship Existence

Thỉnh thoảng bạn có thể thấy mình cần phải kiểm tra sự tồn tại của một quan hệ đồng thời load quan hệ dựa trên các điều kiện giống nhau. Ví dụ: bạn có thể chỉ muốn lấy ra các model `User` mà có các model `Post` phù hợp với một điều kiện truy vấn nhất định trong khi cũng mong muốn eager loading các bài post đó. Bạn có thể thực hiện việc này bằng phương thức `withWhereHas`:

    use App\Models\User;

    $users = User::withWhereHas('posts', function ($query) {
        $query->where('featured', true);
    })->get();

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

Thỉnh thoảng bạn có thể cần eager load một quan hệ sau khi một model cha đã được lấy ra. Ví dụ, điều này có thể hữu ích nếu bạn cần một cách linh động để load các model quan hệ:

    use App\Models\Book;

    $books = Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

Nếu bạn cần set thêm các ràng buộc truy vấn cho các truy vấn eager loading, bạn có thể truyền vào một mảng có khóa là các quan hệ mà bạn muốn load. Các giá trị mảng phải là các instances closure nhận vào một instances query:

    $author->load(['books' => function (Builder $query) {
        $query->orderBy('published_date', 'asc');
    }]);

Để load một quan hệ khi nó chưa được load, hãy sử dụng phương thức `loadMissing`:

    $book->loadMissing('author');

<a name="nested-lazy-eager-loading-morphto"></a>
#### Nested Lazy Eager Loading và `morphTo`

Nếu bạn muốn eager loading một quan hệ `morphTo`, cũng như các quan hệ lồng nhau trên các thực thể khác nhau có thể được trả về bởi quan hệ đó, bạn có thể sử dụng phương thức `loadMorph`.

Phương thức này chấp nhận tên của quan hệ `morphTo` làm tham số đầu tiên và một mảng các cặp model / quan hệ làm tham số thứ hai của nó. Để giúp minh họa phương thức này, chúng ta hãy xem một model sau:

    <?php

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphTo;

    class ActivityFeed extends Model
    {
        /**
         * Get the parent of the activity feed record.
         */
        public function parentable(): MorphTo
        {
            return $this->morphTo();
        }
    }

Trong ví dụ này, giả sử các model `Event`, `Photo`, và `Post` có thể tạo model `ActivityFeed`. Ngoài ra, giả sử rằng model `Event` thuộc một model `Calendar`, model `Photo` được liên kết với model `Tag` và model `Post` thuộc model `Author`.

Sử dụng các định nghĩa và quan hệ của model này, chúng ta có thể truy xuất các instance model của `ActivityFeed` và eager loading tất cả các model `parentable` và các quan hệ lồng nhau tương ứng của chúng:

    $activities = ActivityFeed::with('parentable')
        ->get()
        ->loadMorph('parentable', [
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);

<a name="preventing-lazy-loading"></a>
### Chặn Lazy Loading

Như đã thảo luận trước đó, các quan hệ eager loading thường có thể mang lại lợi ích hiệu suất đáng kể cho ứng dụng của bạn. Do đó, nếu muốn, bạn có thể hướng dẫn Laravel luôn chặn quá trình lazy loading của các quan hệ. Để thực hiện điều này, bạn có thể gọi phương thức `preventLazyLoading` do class model Eloquent cung cấp. Thông thường, bạn nên gọi phương thức này trong phương thức `boot` của class `AppServiceProvider` của ứng dụng của bạn.

Phương thức `preventLazyLoading` chấp nhận một tham số boolean tùy chọn cho biết liệu có nên chặn lazy loading hay không. Ví dụ: bạn có thể chỉ muốn disable tính năng lazy loading trong môi trường không phải production để môi trường production của bạn sẽ tiếp tục hoạt động bình thường ngay cả khi quan hệ lazy loading có xuất hiện trong code production:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

Sau khi chặn lazy loading, Eloquent sẽ đưa ra một exception `Illuminate\Database\LazyLoadingViolationException` khi ứng dụng của bạn thử lazy load bất kỳ quan hệ Eloquent nào.

Bạn có thể tùy chỉnh hành vi lazy loading bằng phương thức `handleLazyLoadingViolationsUsing`. Ví dụ: sử dụng phương thức này, bạn có thể hướng dẫn các lazy loading chỉ được ghi log thay vì làm gián đoạn quá trình thực thi của ứng dụng với các exceptions:

```php
Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
    $class = $model::class;

    info("Attempted to lazy load [{$relation}] on model [{$class}].");
});
```

<a name="inserting-and-updating-related-models"></a>
## Thêm và cập nhật theo quan hệ model

<a name="the-save-method"></a>
### Phương thức `save`

Eloquent cung cấp các phương thức thuận tiện để thêm các model mới vào các quan hệ. Ví dụ: bạn cần thêm một comment mới vào bài post. Thay vì set thủ công thuộc tính `post_id` trên model `Comment`, bạn có thể thêm một comment bằng phương thức `save` của quan hệ:

    use App\Models\Comment;
    use App\Models\Post;

    $comment = new Comment(['message' => 'A new comment.']);

    $post = Post::find(1);

    $post->comments()->save($comment);

Lưu ý rằng chúng ta đã không truy cập vào quan hệ `comments` như là một thuộc tính động. Thay vào đó, chúng ta đã gọi phương thức `comments` để lấy ra một instance của quan hệ. Và phương thức `save` sẽ tự động thêm giá trị `post_id` phù hợp cho model `Comment` mới.

Nếu bạn cần lưu nhiều model quan hệ trong cùng một lúc, bạn có thể sử dụng phương thức `saveMany`:

    $post = Post::find(1);

    $post->comments()->saveMany([
        new Comment(['message' => 'A new comment.']),
        new Comment(['message' => 'Another new comment.']),
    ]);

Các phương thức `save` và `saveMany` sẽ lưu các instance model mới cơ sở dữ liệu, nhưng sẽ không thêm các model mới đó lưu vào bất kỳ quan hệ nào đã được load vào trong bộ nhớ. Nếu bạn định truy cập vào quan hệ sau khi sử dụng phương thức `save` hoặc `saveMany`, bạn có thể sử dụng phương thức `refresh` để load lại model và các quan hệ của nó:

    $post->comments()->save($comment);

    $post->refresh();

    // All comments, including the newly saved comment...
    $post->comments;

<a name="the-push-method"></a>
#### Lưu đệ quy quan hệ và model

Nếu bạn muốn `save` model của bạn và tất cả các quan hệ liên quan đến nó, bạn có thể sử dụng phương thức `push`. Trong ví dụ này, model `Post` sẽ được lưu cũng như các comment của nó và tác giả của các comment đó:

    $post = Post::find(1);

    $post->comments[0]->message = 'Message';
    $post->comments[0]->author->name = 'Author Name';

    $post->push();

Phương thức `pushQuietly` có thể được sử dụng để lưu model và các quan hệ liên quan của nó mà không cần đưa ra bất kỳ event nào:

    $post->pushQuietly();

<a name="the-create-method"></a>
### Phương thức `create`

Ngoài các phương thức `save` và `saveMany`, bạn cũng có thể sử dụng phương thức `create`, phương thức này chấp nhận một mảng các thuộc tính, tạo một model và thêm nó vào cơ sở dữ liệu. Sự khác biệt giữa `save` và `create` là `save` chấp nhận một instance model Eloquent đầy đủ trong khi `create` chấp nhận một `array` PHP đơn giản. Model mới được tạo sẽ được trả về bằng phương thức `create`:

    use App\Models\Post;

    $post = Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

Bạn có thể sử dụng phương thức `createMany` để tạo nhiều model quan hệ:

    $post = Post::find(1);

    $post->comments()->createMany([
        ['message' => 'A new comment.'],
        ['message' => 'Another new comment.'],
    ]);

Các phương thức `createQuietly` và `createManyQuietly` có thể được sử dụng để tạo một hoặc là nhiều model mà không gửi bất kỳ event nào:

    $user = User::find(1);

    $user->posts()->createQuietly([
        'title' => 'Post title.',
    ]);

    $user->posts()->createManyQuietly([
        ['title' => 'First post.'],
        ['title' => 'Second post.'],
    ]);

Bạn cũng có thể sử dụng các phương thức `findOrNew`, `firstOrNew`, `firstOrCreate`, và `updateOrCreate` để [tạo và cập nhật model trên các quan hệ](https://laravel.com/docs/{{version}}/eloquent#other-creation-methods).

> [!NOTE]
> Trước khi sử dụng phương thức `create`, bạn hãy chắc chắn là đã xem qua tài liệu về thuộc tính [mass assignment](/docs/{{version}}/eloquent#mass-assignment).

<a name="updating-belongs-to-relationships"></a>
### Quan hệ thuộc về

Nếu bạn muốn gán một model con cho một model cha mới, bạn có thể sử dụng phương thức `associate`. Trong ví dụ này, model `User` định nghĩa một quan hệ `belongsTo` với model `Account`. Phương thức `associate` này sẽ set khóa ngoại trên model con:

    use App\Models\Account;

    $account = Account::find(10);

    $user->account()->associate($account);

    $user->save();

Để xóa model gốc ra khỏi model con, bạn có thể sử dụng phương thức `dissociate`. Phương thức này sẽ set khóa ngoại của quan hệ thành `null`:

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### Quan hệ Nhiều - Nhiều

<a name="attaching-detaching"></a>
#### Attaching / Detaching

Eloquent cũng cung cấp thêm những phương thức để làm việc với các quan hệ nhiều-nhiều một cách thuận tiện hơn. Ví dụ: hãy tưởng tượng một user có thể có nhiều role và một role có thể có nhiều user. Bạn có thể sử dụng phương thức `attach`để attach một role cho một user, chúng ta có thể làm bằng cách thêm một bản ghi vào trong bảng trung gian của quan hệ:

    use App\Models\User;

    $user = User::find(1);

    $user->roles()->attach($roleId);

Khi attach một quan hệ với một model, bạn cũng có thể truyền thêm một mảng dữ liệu sẽ được thêm cùng vào trong bảng trung gian:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Tất nhiên, thỉnh thoảng bạn cũng có thể cần phải xóa một role ra khỏi một user. Để xoá một bản ghi quan hệ nhiều-nhiều, hãy sử dụng phương thức `detach`. Phương thức `detach` sẽ xoá bản ghi phù hợp ra khỏi bảng trung gian; tuy nhiên, cả hai model vẫn sẽ còn trong cơ sở dữ liệu:

    // Detach a single role from the user...
    $user->roles()->detach($roleId);

    // Detach all roles from the user...
    $user->roles()->detach();

Để thuận tiện, các phương thức `attach` và `detach` cũng chấp nhận một mảng các ID làm đầu vào:

    $user = User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires],
    ]);

<a name="syncing-associations"></a>
#### Syncing Associations

Bạn cũng có thể sử dụng phương thức `sync` để khởi tạo các liên kết nhiều-nhiều. Phương thức `sync` chấp nhận một mảng ID để set vào trong bảng trung gian. Bất kỳ ID nào không nằm trong mảng đã cho sẽ bị xóa khỏi bảng trung gian. Vì vậy, sau khi thao tác này hoàn tất, chỉ có các ID trong mảng đã cho sẽ tồn tại trong bảng trung gian:

    $user->roles()->sync([1, 2, 3]);

Bạn cũng có thể truyền thêm các giá trị cho bảng trung gian với ID:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

Nếu bạn muốn thêm các giá trị cho bảng trung gian giống nhau với từng ID model được sync, bạn có thể sử dụng phương thức `syncWithPivotValues`:

    $user->roles()->syncWithPivotValues([1, 2, 3], ['active' => true]);

Nếu bạn không muốn detach những ID đã tồn tại bị thiếu trong mảng đã cho, bạn có thể sử dụng phương thức `syncWithoutDetaching`:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

<a name="toggling-associations"></a>
#### Toggling Associations

Quan hệ nhiều-nhiều cũng cung cấp thêm một phương thức `toggle` để "bật" hoặc "tắt" trạng thái attach của một mảng các ID của model quan hệ. Nếu ID trong mảng đó đã được attach, thì nó sẽ bị detached. Tương tự, nếu nó đang được detached, thì nó sẽ được attach:

    $user->roles()->toggle([1, 2, 3]);

Bạn cũng có thể chuyển thêm các giá trị cho bảng trung gian bằng ID:

    $user->roles()->toggle([
        1 => ['expires' => true],
        2 => ['expires' => true],
    ]);

<a name="updating-a-record-on-the-intermediate-table"></a>
#### Updating A Record On The Intermediate Table

Nếu bạn cần cập nhật một bản ghi hiện có trong bảng quan hệ trung gian của bạn, bạn có thể sử dụng phương thức `updateExistingPivot`. Phương thức này chấp nhận khóa ngoại của bản ghi trung gian và một mảng các thuộc tính để cập nhật:

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, [
        'active' => false,
    ]);

<a name="touching-parent-timestamps"></a>
## Sửa timestamp của model chứa

Khi một model định nghĩa một quan hệ `belongsTo` hoặc `belongsToMany` tới một model khác, chẳng hạn như một `Comment` nằm trong một `Post`, đôi khi bạn sẽ cần cập nhật timestamp của model cha khi model con được cập nhật.

Ví dụ, khi một model `Comment` được cập nhật, bạn có thể muốn tự động "touch" vào cột timestamp `update_at` của model `Post` để nó được set thành ngày và giờ hiện tại. Để thực hiện điều này, bạn có thể thêm một thuộc tính `touches` vào model con của bạn và chứa tên của các quan hệ sẽ được cập nhật timestamp `updated_at` của chúng khi model con được cập nhật:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsTo;

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
        public function post(): BelongsTo
        {
            return $this->belongsTo(Post::class);
        }
    }

> [!WARNING]
> Timestamp của model gốc sẽ chỉ được cập nhật nếu model con được cập nhật bằng phương thức `save` của Eloquent.
