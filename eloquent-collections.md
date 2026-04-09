# Eloquent: Collections

- [Giới thiệu](#introduction)
- [Các phương thức có sẵn](#available-methods)
- [Tuỳ biến Collection](#custom-collections)

<a name="introduction"></a>
## Giới thiệu

Tất cả các phương thức của Eloquent sẽ trả về nhiều hơn một model và kết quả sẽ trả về các instance của đối tượng `Illuminate\Database\Eloquent\Collection`, bao gồm cả kết quả được truy xuất thông qua phương thức `get` hoặc truy vấn thông qua quan hệ. Đối tượng collection Eloquent được extend từ [base collection](/docs/{{version}}/collections), do đó, nó thừa hưởng nhiều phương thức có thể được dùng để làm việc dễ dàng hơn với mảng model Eloquent. Hãy nhớ xem lại tài liệu collection của Laravel để tìm hiểu tất cả về các phương thức hữu ích này!

Tất cả các collection này cũng có vai trò như là một vòng lặp, cho phép bạn lặp qua nó như thể nó là một mảng PHP đơn thuần:

```php
use App\Models\User;

$users = User::where('active', 1)->get();

foreach ($users as $user) {
    echo $user->name;
}
```

Tuy nhiên, như đã nói ở trên, collection mạnh mẽ hơn nhiều so với mảng và có thêm nhiều phương thức như map hoặc reduce, có thể được kết hợp lại với nhau qua một giao diện trực quan. Ví dụ: hãy xóa tất cả những người dùng không hoạt động và lấy ra tên của những người dùng còn lại:

```php
$names = User::all()->reject(function (User $user) {
    return $user->active === false;
})->map(function (User $user) {
    return $user->name;
});
```

<a name="eloquent-collection-conversion"></a>
#### Eloquent Collection Conversion

> {note} Trong khi hầu hết các phương thức Eloquent trả về một instance mới của collection Eloquent, thì các phương thức `collapse`, `flatten`, `flip`, `keys`, `pluck`, và `zip` sẽ trả về một instance [base collection](/docs/{{version}}/collections). Tương tự, nếu mà phương thức `map` trả về một collection không chứa bất kỳ model Eloquent nào, thì nó sẽ được convert thành một instance collection base.

<a name="available-methods"></a>
## Các phương thức có sẵn

Tất cả các Eloquent collection sẽ được extend từ một đối tượng [Laravel collection](/docs/{{version}}/collections#available-methods); do đó, chúng kế thừa tất cả các phương thức mạnh mẽ được cung cấp bởi class laravel collection cơ sở.

Ngoài ra, class `Illuminate\Database\Eloquent\Collection` cũng sẽ cung cấp một tập hợp các phương thức để hỗ trợ việc quản lý các model collection của bạn. Hầu hết các phương thức này đều trả về các instance `Illuminate\Database\Eloquent\Collection`; tuy nhiên, có một số phương thức, giống như `modelKeys`, sẽ trả về một instance `Illuminate\Support\Collection` cơ sở.

<style>
    .collection-method-list > p {
        columns: 14.4em 1; -moz-columns: 14.4em 1; -webkit-columns: 14.4em 1;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<div class="collection-method-list" markdown="1">

[append](#method-append)
[contains](#method-contains)
[diff](#method-diff)
[except](#method-except)
[find](#method-find)
[findOrFail](#method-find-or-fail)
[fresh](#method-fresh)
[intersect](#method-intersect)
[load](#method-load)
[loadMissing](#method-loadMissing)
[modelKeys](#method-modelKeys)
[makeVisible](#method-makeVisible)
[makeHidden](#method-makeHidden)
[mergeVisible](#method-mergeVisible)
[mergeHidden](#method-mergeHidden)
[only](#method-only)
[partition](#method-partition)
[setAppends](#method-setAppends)
[setVisible](#method-setVisible)
[setHidden](#method-setHidden)
[toQuery](#method-toquery)
[unique](#method-unique)
[withoutAppends](#method-withoutAppends)

</div>

<a name="method-append"></a>
#### `append($attributes)` {.collection-method .first-collection-method}

Phương thức `append` có thể được dùng để chỉ ra rằng một thuộc tính sẽ phải [được thêm](/docs/{{version}}/eloquent-serialization#appending-values-to-json) vào trong mọi model trong collection. Phương thức này sẽ chấp nhận một mảng các thuộc tính hoặc một thuộc tính duy nhất:

```php
$users->append('team');

$users->append(['team', 'is_admin']);
```

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)` {.collection-method}

Phương thức `contains` có thể được sử dụng để xác định xem một instance model có trong một collection hay không. Phương thức này chấp nhận một khóa chính hoặc một instance model:

```php
$users->contains(1);

$users->contains(User::find(1));
```

<a name="method-diff"></a>
#### `diff($items)` {.collection-method}

Phương thức `diff` sẽ trả về tất cả các model không có trong một collection đã cho:

```php
use App\Models\User;

$users = $users->diff(User::whereIn('id', [1, 2, 3])->get());
```

<a name="method-except"></a>
#### `except($keys)` {.collection-method}

Phương thức `except` sẽ trả về tất cả các model không chứa một mảng khóa chính đã cho:

```php
$users = $users->except([1, 2, 3]);
```

<a name="method-find"></a>
#### `find($key)` {.collection-method}

Phương thức `find` trả về model có khóa chính khớp với khóa đã cho. Nếu `$key` là một instance model, phương thức `find` sẽ cố gắng trả về một model khớp với khóa chính của model đã cho. Nếu `$key` là một mảng gồm các khóa chính, thì phương thức `find` sẽ trả về tất cả các model có một khóa chính trong mảng đã cho:

```php
$users = User::all();

$user = $users->find(1);
```

<a name="method-find-or-fail"></a>
#### `findOrFail($key)` {.collection-method}

Phương thức `findOrFail` sẽ trả về model có khóa chính khớp với khóa đã cho hoặc đưa ra exception `Illuminate\Database\Eloquent\ModelNotFoundException` nếu không tìm thấy model nào khớp có trong collection:

```php
$users = User::all();

$user = $users->findOrFail(1);
```

<a name="method-fresh"></a>
#### `fresh($with = [])` {.collection-method}

Phương thức `fresh` sẽ lấy ra lại một instance mới của mỗi model trong collection từ cơ sở dữ liệu. Ngoài ra, bất kỳ mối quan hệ được chỉ định nào cũng sẽ được eager loading lại:

```php
$users = $users->fresh();

$users = $users->fresh('comments');
```

<a name="method-intersect"></a>
#### `intersect($items)` {.collection-method}

Phương thức `intersect` sẽ trả về tất cả các model cũng có trong collection đã cho:

```php
use App\Models\User;

$users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());
```

<a name="method-load"></a>
#### `load($relations)` {.collection-method}

Phương thức `load` sẽ eager loading tất cả các quan hệ đã cho, cho tất cả các model có trong collection:

```php
$users->load(['comments', 'posts']);

$users->load('comments.author');

$users->load(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

<a name="method-loadMissing"></a>
#### `loadMissing($relations)` {.collection-method}

Phương thức `loadMissing`sẽ eager loading tất cả các quan hệ đã cho, cho tất cả các model có trong collection nếu các quan hệ đó chưa được load:

```php
$users->loadMissing(['comments', 'posts']);

$users->loadMissing('comments.author');

$users->loadMissing(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

<a name="method-modelKeys"></a>
#### `modelKeys()` {.collection-method}

Phương thức `modelKeys` sẽ trả về các khóa chính của tất cả các model có trong collection:

```php
$users->modelKeys();

// [1, 2, 3, 4, 5]
```

<a name="method-makeVisible"></a>
#### `makeVisible($attributes)` {.collection-method}

Phương thức `makeVisible` sẽ [làm cho các thuộc tính](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) bị "hidden" sẽ hiển thị trên mỗi model có trong collection:

```php
$users = $users->makeVisible(['address', 'phone_number']);
```

<a name="method-makeHidden"></a>
#### `makeHidden($attributes)` {.collection-method}

Phương thức `makeHidden` sẽ [làm cho các thuộc tính](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) được "visible" sẽ bị ẩn trên mỗi model có trong collection:

```php
$users = $users->makeHidden(['address', 'phone_number']);
```

<a name="method-mergeVisible"></a>
#### `mergeVisible($attributes)` {.collection-method}

Phương thức `mergeVisible` sẽ [làm thêm các thuộc tính hiển thị](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) trong khi vẫn giữ các thuộc tính hiển thị trước đó:

```php
$users = $users->mergeVisible(['middle_name']);
```

<a name="method-mergeHidden"></a>
#### `mergeHidden($attributes)` {.collection-method}

Phương thức `mergeHidden` sẽ [làm ẩn thêm các thuộc tính](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) trong khi vẫn giữ các thuộc tính bị ẩn trước đó:

```php
$users = $users->mergeHidden(['last_login_at']);
```

<a name="method-only"></a>
#### `only($keys)` {.collection-method}

Phương thức `only` sẽ trả về tất cả các model có khóa chính đã cho:

```php
$users = $users->only([1, 2, 3]);
```

<a name="method-partition"></a>
#### `partition` {.collection-method}

Phương thức `partition` trả về một instance của `Illuminate\Support\Collection` chứa các instance của collection `Illuminate\Database\Eloquent\Collection`:

```php
$partition = $users->partition(fn ($user) => $user->age > 18);

dump($partition::class);    // Illuminate\Support\Collection
dump($partition[0]::class); // Illuminate\Database\Eloquent\Collection
dump($partition[1]::class); // Illuminate\Database\Eloquent\Collection
```

<a name="method-setAppends"></a>
#### `setAppends($attributes)` {.collection-method}

Phương thức `setAppends` sẽ tạm thời ghi đè tất cả các [thuộc tính được append](/docs/{{version}}/eloquent-serialization#appending-values-to-json) trên mỗi model có trong collection:

```php
$users = $users->setAppends(['is_admin']);
```

<a name="method-setVisible"></a>
#### `setVisible($attributes)` {.collection-method}

Phương thức `setVisible` sẽ [tạm thời ghi đè](/docs/{{version}}/eloquent-serialization#temporarily-modifying-attribute-visibility) tất cả các thuộc tính visible có trong các model trong collection:

```php
$users = $users->setVisible(['id', 'name']);
```

<a name="method-setHidden"></a>
#### `setHidden($attributes)` {.collection-method}

Phương thức `setHidden` sẽ [tạm thời ghi đè](/docs/{{version}}/eloquent-serialization#temporarily-modifying-attribute-visibility) tất cả các thuộc tính ẩn có trong các model trong collection:

```php
$users = $users->setHidden(['email', 'password', 'remember_token']);
```

<a name="method-toquery"></a>
#### `toQuery()` {.collection-method}

Phương thức `toQuery` sẽ trả về một instance query builder của Eloquent chứa câu lệnh điều kiện `whereIn` trên các khóa chính của model collection:

```php
use App\Models\User;

$users = User::where('status', 'VIP')->get();

$users->toQuery()->update([
    'status' => 'Administrator',
]);
```

<a name="method-unique"></a>
#### `unique($key = null, $strict = false)` {.collection-method}

Phương thức `unique` sẽ trả về tất cả các unique model có trong collection. Tất cả các model có cùng khóa chính với các model khác có trong collection đều sẽ bị xóa:

```php
$users = $users->unique();
```

<a name="method-withoutAppends"></a>
#### `withoutAppends()` {.collection-method}

Phương thức `withoutAppends` sẽ tạm thời xóa tất cả các [thuộc tính được append](/docs/{{version}}/eloquent-serialization#appending-values-to-json) trên mỗi model có trong collection:

```php
$users = $users->withoutAppends();
```

<a name="custom-collections"></a>
## Tuỳ biến Collection

Nếu bạn muốn sử dụng một đối tượng `Collection` tùy biến khi tương tác với một model nhất định, bạn có thể thêm thuộc tính `CollectedBy` vào model của bạn:

```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Attributes\CollectedBy;
use Illuminate\Database\Eloquent\Model;

#[CollectedBy(UserCollection::class)]
class User extends Model
{
    // ...
}
```

Ngoài ra, bạn có thể định nghĩa một phương thức `newCollection` trên model của bạn:

```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Create a new Eloquent Collection instance.
     *
     * @param  array<int, \Illuminate\Database\Eloquent\Model>  $models
     * @return \Illuminate\Database\Eloquent\Collection<int, \Illuminate\Database\Eloquent\Model>
     */
    public function newCollection(array $models = []): Collection
    {
        $collection = new UserCollection($models);

        if (Model::isAutomaticallyEagerLoadingRelationships()) {
            $collection->withRelationshipAutoloading();
        }

        return $collection;
    }
}
```

Khi bạn đã định nghĩa một phương thức `newCollection` hoặc thêm thuộc tính `CollectedBy` vào model của bạn, bạn sẽ nhận lại một instance của collection tùy biến của bạn bất cứ lúc nào Eloquent trả về một `Illuminate\Database\Eloquent\Collection` instance.

Nếu bạn muốn sử dụng collection tùy biến cho mọi model trong application của bạn, bạn nên định nghĩa phương thức `newCollection` trên một class base model mà được tất cả các model của ứng dụng extend.
