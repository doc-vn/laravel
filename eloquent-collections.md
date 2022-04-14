# Eloquent: Collections

- [Giới thiệu](#introduction)
- [Các phương thức có sẵn](#available-methods)
- [Tuỳ biến Collection](#custom-collections)

<a name="introduction"></a>
## Giới thiệu

Tất cả các kết quả được trả về từ Eloquent đều là các instance của đối tượng `Illuminate\Database\Eloquent\Collection`, bao gồm cả kết quả được truy xuất thông qua phương thức `get` hoặc truy vấn thông qua quan hệ. Đối tượng collection Eloquent được extend từ [base collection](/docs/{{version}}/collections), do đó, nó thừa hưởng nhiều phương thức có thể được dùng để làm việc dễ dàng hơn với mảng model Eloquent.

Tất cả các collection này cũng có vai trò như là một vòng lặp, cho phép bạn lặp qua nó như thể nó là một mảng PHP đơn thuần:

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

Tuy nhiên, collection mạnh mẽ hơn nhiều so với mảng và có thêm nhiều phương thức như map hoặc reduce, có thể được kết hợp lại với nhau qua một giao diện trực quan. Ví dụ: hãy xóa tất cả những người dùng không hoạt động và lấy ra tên của những người dùng còn lại:

    $users = App\User::all();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} Trong khi hầu hết các phương thức Eloquent trả về một instance mới của collection Eloquent, thì các phương thức `pluck`, `keys`, `zip`, `collapse`, `flatten` và `flip` sẽ trả về một instance [base collection](/docs/{{version}}/collections). Tương tự, nếu mà phương thức `map` trả về một collection không chứa bất kỳ model Eloquent nào, thì nó sẽ tự động được chuyển đổi thành một base collection.

<a name="available-methods"></a>
## Các phương thức có sẵn

Tất cả các Eloquent collection sẽ được extend từ một đối tượng [Laravel collection](/docs/{{version}}/collections#available-methods); do đó, chúng kế thừa tất cả các phương thức mạnh mẽ được cung cấp bởi class laravel collection cơ sở.

Ngoài ra, class `Illuminate\Database\Eloquent\Collection` cũng sẽ cung cấp một tập hợp các phương thức để hỗ trợ việc quản lý các model collection của bạn. Hầu hết các phương thức này đều trả về các instance `Illuminate\Database\Eloquent\Collection`; tuy nhiên, có một số phương thức sẽ trả về một instance `Illuminate\Support\Collection` cơ sở.

<style>
    #collection-method-list > p {
        column-count: 1; -moz-column-count: 1; -webkit-column-count: 1;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[contains](#method-contains)
[diff](#method-diff)
[except](#method-except)
[find](#method-find)
[fresh](#method-fresh)
[intersect](#method-intersect)
[load](#method-load)
[loadMissing](#method-loadMissing)
[modelKeys](#method-modelKeys)
[makeVisible](#method-makeVisible)
[makeHidden](#method-makeHidden)
[only](#method-only)
[unique](#method-unique)

</div>

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)`

Phương thức `contains` có thể được sử dụng để xác định xem một instance model có trong một collection hay không. Phương thức này chấp nhận một khóa chính hoặc một instance model:

    $users->contains(1);

    $users->contains(User::find(1));

<a name="method-diff"></a>
#### `diff($items)`

Phương thức `diff` sẽ trả về tất cả các model không có trong một collection đã cho:

    use App\User;

    $users = $users->diff(User::whereIn('id', [1, 2, 3])->get());

<a name="method-except"></a>
#### `except($keys)`

Phương thức `except` sẽ trả về tất cả các model không chứa một mảng khóa chính đã cho:

    $users = $users->except([1, 2, 3]);

<a name="method-find"></a>
#### `find($key)` {#collection-method .first-collection-method}

Phương thức `find` sẽ tìm một model bầng một khóa chính cho trước. Nếu `$key` là một instance model, phương thức `find` sẽ cố gắng trả về một model khớp với khóa chính của model đã cho. Nếu `$key` là một mảng gồm các khóa chính, thì phương thức `find` sẽ trả về tất cả các model mà khớp với `$key` bằng cách sử dụng phương thức `whereIn()`:

    $users = User::all();

    $user = $users->find(1);

<a name="method-fresh"></a>
#### `fresh($with = [])`

Phương thức `fresh` sẽ lấy ra lại một instance mới của mỗi model trong collection từ cơ sở dữ liệu. Ngoài ra, bất kỳ mối quan hệ được chỉ định nào cũng sẽ được eager loading lại:

    $users = $users->fresh();

    $users = $users->fresh('comments');

<a name="method-intersect"></a>
#### `intersect($items)`

Phương thức `intersect` sẽ trả về tất cả các model cũng có trong collection đã cho:

    use App\User;

    $users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());

<a name="method-load"></a>
#### `load($relations)`

Phương thức `load` sẽ eager loading tất cả các quan hệ đã cho, cho tất cả các model có trong collection:

    $users->load('comments', 'posts');

    $users->load('comments.author');

<a name="method-loadMissing"></a>
#### `loadMissing($relations)`

Phương thức `loadMissing`sẽ eager loading tất cả các quan hệ đã cho, cho tất cả các model có trong collection nếu các quan hệ đó chưa được load:

    $users->loadMissing('comments', 'posts');

    $users->loadMissing('comments.author');

<a name="method-modelKeys"></a>
#### `modelKeys()`

Phương thức `modelKeys` sẽ trả về các khóa chính của tất cả các model có trong collection:

    $users->modelKeys();

    // [1, 2, 3, 4, 5]

<a name="method-makeVisible"></a>
#### `makeVisible($attributes)`

Phương thức `makeVisible` sẽ làm cho các thuộc tính bị "hidden" sẽ hiển thị trên mỗi model có trong collection:

    $users = $users->makeVisible(['address', 'phone_number']);

<a name="method-makeHidden"></a>
#### `makeHidden($attributes)`

Phương thức `makeHidden` sẽ làm cho các thuộc tính được "visible" sẽ bị ẩn trên mỗi model có trong collection:

    $users = $users->makeHidden(['address', 'phone_number']);

<a name="method-only"></a>
#### `only($keys)`

Phương thức `only` sẽ trả về tất cả các model có khóa chính đã cho:

    $users = $users->only([1, 2, 3]);

<a name="method-unique"></a>
#### `unique($key = null, $strict = false)`

Phương thức `unique` sẽ trả về tất cả các unique model có trong collection. Tất cả các model có cùng khóa chính với các model khác có trong collection đều sẽ bị xóa.

    $users = $users->unique();

<a name="custom-collections"></a>
## Tuỳ biến Collection

Nếu bạn cần sử dụng một đối tượng `Collection` tùy biến cho các phương thức dành riêng cho bạn, bạn có thể ghi đè phương thức `newCollection` trên model của bạn:

    <?php

    namespace App;

    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Create a new Eloquent Collection instance.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

Khi bạn đã định nghĩa một phương thức `newCollection`, bạn sẽ nhận lại một instance của collection tùy biến của bạn bất cứ khi nào Eloquent trả về một instance `Collection` cho model đó. Nếu bạn muốn sử dụng collection tùy biến cho mọi model trong application của bạn, bạn nên ghi đè phương thức `newCollection` trên một class base model mà được tất cả các model khác extend.
