# Helpers

- [Giới thiệu](#introduction)
- [Các phương thức có sẵn](#available-methods)
- [Các class hữu ích khác](#other-utilities)
    - [Benchmarking](#benchmarking)
    - [Dates](#dates)
    - [Lottery](#lottery)
    - [Pipeline](#pipeline)
    - [Sleep](#sleep)

<a name="introduction"></a>
## Giới thiệu

Laravel chứa một loạt các hàm PHP global "helper". Nhiều trong số các hàm này được sử dụng bởi chính framework; tuy nhiên, bạn có thể thoải mái sử dụng chúng trong application của bạn nếu bạn thấy chúng tiện ích.

<a name="available-methods"></a>
## Các phương thức có sẵn

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<a name="arrays-and-objects-method-list"></a>
### Arrays & Objects

<div class="collection-method-list" markdown="1">

[Arr::accessible](#method-array-accessible)
[Arr::add](#method-array-add)
[Arr::collapse](#method-array-collapse)
[Arr::crossJoin](#method-array-crossjoin)
[Arr::divide](#method-array-divide)
[Arr::dot](#method-array-dot)
[Arr::except](#method-array-except)
[Arr::exists](#method-array-exists)
[Arr::first](#method-array-first)
[Arr::flatten](#method-array-flatten)
[Arr::forget](#method-array-forget)
[Arr::get](#method-array-get)
[Arr::has](#method-array-has)
[Arr::hasAny](#method-array-hasany)
[Arr::isAssoc](#method-array-isassoc)
[Arr::isList](#method-array-islist)
[Arr::join](#method-array-join)
[Arr::keyBy](#method-array-keyby)
[Arr::last](#method-array-last)
[Arr::map](#method-array-map)
[Arr::mapWithKeys](#method-array-map-with-keys)
[Arr::only](#method-array-only)
[Arr::pluck](#method-array-pluck)
[Arr::prepend](#method-array-prepend)
[Arr::prependKeysWith](#method-array-prependkeyswith)
[Arr::pull](#method-array-pull)
[Arr::query](#method-array-query)
[Arr::random](#method-array-random)
[Arr::set](#method-array-set)
[Arr::shuffle](#method-array-shuffle)
[Arr::sort](#method-array-sort)
[Arr::sortDesc](#method-array-sort-desc)
[Arr::sortRecursive](#method-array-sort-recursive)
[Arr::sortRecursiveDesc](#method-array-sort-recursive-desc)
[Arr::take](#method-array-take)
[Arr::toCssClasses](#method-array-to-css-classes)
[Arr::toCssStyles](#method-array-to-css-styles)
[Arr::undot](#method-array-undot)
[Arr::where](#method-array-where)
[Arr::whereNotNull](#method-array-where-not-null)
[Arr::wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[data_forget](#method-data-forget)
[head](#method-head)
[last](#method-last)
</div>

<a name="numbers-method-list"></a>
### Numbers

<div class="collection-method-list" markdown="1">

[Number::abbreviate](#method-number-abbreviate)
[Number::clamp](#method-number-clamp)
[Number::currency](#method-number-currency)
[Number::fileSize](#method-number-file-size)
[Number::forHumans](#method-number-for-humans)
[Number::format](#method-number-format)
[Number::ordinal](#method-number-ordinal)
[Number::percentage](#method-number-percentage)
[Number::spell](#method-number-spell)
[Number::useLocale](#method-number-use-locale)
[Number::withLocale](#method-number-with-locale)

</div>


<a name="paths-method-list"></a>
### Paths

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[lang_path](#method-lang-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

<a name="urls-method-list"></a>
### URLs

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[route](#method-route)
[secure_asset](#method-secure-asset)
[secure_url](#method-secure-url)
[to_route](#method-to-route)
[url](#method-url)

</div>

<a name="miscellaneous-method-list"></a>
### Miscellaneous

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[decrypt](#method-decrypt)
[dd](#method-dd)
[dispatch](#method-dispatch)
[dispatch_sync](#method-dispatch-sync)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[fake](#method-fake)
[filled](#method-filled)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[report_if](#method-report-if)
[report_unless](#method-report-unless)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[today](#method-today)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)

</div>

<a name="arrays"></a>
## Arrays & Objects

<a name="method-array-accessible"></a>
#### `Arr::accessible()` {.collection-method .first-collection-method}

Hàm `Arr::accessible` sẽ xác định xem giá trị đã cho có phải là mảng có thể truy cập được hay không:

    use Illuminate\Support\Arr;
    use Illuminate\Support\Collection;

    $isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

    // true

    $isAccessible = Arr::accessible(new Collection);

    // true

    $isAccessible = Arr::accessible('abc');

    // false

    $isAccessible = Arr::accessible(new stdClass);

    // false

<a name="method-array-add"></a>
#### `Arr::add()` {.collection-method .first-collection-method}

Hàm `Arr::add` sẽ thêm một cặp key / giá trị vào một mảng nếu key đó không tồn tại trong mảng hoặc giá trị trong mảng của key đó bằng `null`:

    use Illuminate\Support\Arr;

    $array = Arr::add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

    $array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]


<a name="method-array-collapse"></a>
#### `Arr::collapse()` {.collection-method}

Hàm `Arr::collapse` sẽ thu gọn một mảng gồm nhiều mảng con thành một mảng duy nhất:

    use Illuminate\Support\Arr;

    $array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-crossjoin"></a>
#### `Arr::crossJoin()` {.collection-method}

Hàm `Arr::crossJoin` sẽ join chéo các giá trị của mảng đã cho, và trả về một tích chéo với tất cả các hoán vị có thể có:

    use Illuminate\Support\Arr;

    $matrix = Arr::crossJoin([1, 2], ['a', 'b']);

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-array-divide"></a>
#### `Arr::divide()` {.collection-method}

Hàm `Arr::divide` trả về hai mảng: một mảng chứa các key và một mảng chứa các giá trị của mảng đã cho:

    use Illuminate\Support\Arr;

    [$keys, $values] = Arr::divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `Arr::dot()` {.collection-method}

Hàm `Arr::dot` sẽ làm ngang hàng một mảng nhiều chiều thành một mảng một chiều sử dụng ký hiệu "dot" để biểu thị độ sâu:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = Arr::dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `Arr::except()` {.collection-method}

Hàm `Arr::except` loại bỏ các cặp key / giá trị đã cho ra khỏi một mảng:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = Arr::except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-exists"></a>
#### `Arr::exists()` {.collection-method}

Hàm `Arr::exists` sẽ kiểm tra xem khóa đã cho có tồn tại trong mảng đã cho hay không:

    use Illuminate\Support\Arr;

    $array = ['name' => 'John Doe', 'age' => 17];

    $exists = Arr::exists($array, 'name');

    // true

    $exists = Arr::exists($array, 'salary');

    // false

<a name="method-array-first"></a>
#### `Arr::first()` {.collection-method}

Hàm `Arr::first` trả về phần tử đầu tiên của mảng pass qua một số điều kiện đã cho:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300];

    $first = Arr::first($array, function (int $value, int $key) {
        return $value >= 150;
    });

    // 200

Một giá trị mặc định cũng có thể được truyền làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu không có giá trị nào được pass qua điều kiện:

    use Illuminate\Support\Arr;

    $first = Arr::first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `Arr::flatten()` {.collection-method}

Hàm `Arr::flatten` làm ngang hàng một mảng nhiều chiều thành một mảng một chiều:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = Arr::flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `Arr::forget()` {.collection-method}

Hàm `Arr::forget` xóa một cặp key / giá trị đã cho ra khỏi một mảng bị lồng vào nhau bằng cách sử dụng ký hiệu "dot":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `Arr::get()` {.collection-method}

Hàm `Arr::get` lấy một giá trị từ một mảng bị lồng vào nhau bằng cách sử dụng ký hiệu "dot":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = Arr::get($array, 'products.desk.price');

    // 100

Hàm `Arr::get` cũng chấp nhận một giá trị mặc định, sẽ được trả về nếu khóa được chỉ định không có trong mảng:

    use Illuminate\Support\Arr;

    $discount = Arr::get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `Arr::has()` {.collection-method}

Hàm `Arr::has` sẽ kiểm tra xem một item hoặc các item đã cho có tồn tại trong một mảng hay không bằng cách sử dụng ký hiệu "dot":

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::has($array, 'product.name');

    // true

    $contains = Arr::has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-hasany"></a>
#### `Arr::hasAny()` {.collection-method}

Hàm `Arr::hasAny` sẽ kiểm tra xem có bất kỳ item nào có trong một mảng hay không bằng cách sử dụng ký tự "chấm":

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::hasAny($array, 'product.name');

    // true

    $contains = Arr::hasAny($array, ['product.name', 'product.discount']);

    // true

    $contains = Arr::hasAny($array, ['category', 'product.discount']);

    // false

<a name="method-array-isassoc"></a>
#### `Arr::isAssoc()` {.collection-method}

Hàm `Arr::isAssoc` sẽ trả về `true` nếu mảng đã cho là một mảng associative. Một mảng được coi là "associative" nếu nó không có khóa bắt đầu từ 0:

    use Illuminate\Support\Arr;

    $isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

    // true

    $isAssoc = Arr::isAssoc([1, 2, 3]);

    // false

<a name="method-array-islist"></a>
#### `Arr::isList()` {.collection-method}

Hàm `Arr::isList` sẽ trả về `true` nếu khóa của mảng đã cho là các số nguyên theo thứ tự bắt đầu từ 0:

    use Illuminate\Support\Arr;

    $isList = Arr::isList(['foo', 'bar', 'baz']);

    // true

    $isList = Arr::isList(['product' => ['name' => 'Desk', 'price' => 100]]);

    // false

<a name="method-array-join"></a>
#### `Arr::join()` {.collection-method}

Hàm `Arr::join` sẽ nối các phần tử của mảng vào với nhau bằng một string. Sử dụng tham số thứ hai của phương thức này, bạn cũng có thể chỉ định string mà bạn muốn nối cho phần tử cuối cùng của mảng:

    use Illuminate\Support\Arr;

    $array = ['Tailwind', 'Alpine', 'Laravel', 'Livewire'];

    $joined = Arr::join($array, ', ');

    // Tailwind, Alpine, Laravel, Livewire

    $joined = Arr::join($array, ', ', ' and ');

    // Tailwind, Alpine, Laravel and Livewire

<a name="method-array-keyby"></a>
#### `Arr::keyBy()` {.collection-method}

Hàm `Arr::keyBy` sẽ tạo khóa cho mảng bằng khóa đã cho. Nếu nhiều item có cùng một khóa, thì item cuối cùng sẽ được cho vào trong mảng mới:

    use Illuminate\Support\Arr;

    $array = [
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ];

    $keyed = Arr::keyBy($array, 'product_id');

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-array-last"></a>
#### `Arr::last()` {.collection-method}

Hàm `Arr::last` trả về phần tử cuối cùng của mảng pass qua một số điều kiện đã cho:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300, 110];

    $last = Arr::last($array, function (int $value, int $key) {
        return $value >= 150;
    });

    // 300

Một giá trị mặc định có thể được truyền làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu không có giá trị nào pass qua điều kiện:

    use Illuminate\Support\Arr;

    $last = Arr::last($array, $callback, $default);

<a name="method-array-map"></a>
#### `Arr::map()` {.collection-method}

Hàm `Arr::map` sẽ lặp từng phần tử của mảng và chuyển từng giá trị cũng như khóa của nó cho một callback đã cho. Giá trị mảng sẽ được thay thế bằng giá trị được trả về bởi callback:

    use Illuminate\Support\Arr;

    $array = ['first' => 'james', 'last' => 'kirk'];

    $mapped = Arr::map($array, function (string $value, string $key) {
        return ucfirst($value);
    });

    // ['first' => 'James', 'last' => 'Kirk']

<a name="method-array-map-with-keys"></a>
#### `Arr::mapWithKeys()` {.collection-method}

Hàm `Arr::mapWithKeys` sẽ lặp qua mảng và chuyển từng giá trị cho lệnh callback đã cho. Lệnh callback sẽ trả về một mảng kết hợp giữa một khóa và giá trị:

    use Illuminate\Support\Arr;

    $array = [
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com',
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com',
        ]
    ];

    $mapped = Arr::mapWithKeys($array, function (array $item, int $key) {
        return [$item['email'] => $item['name']];
    });

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-array-only"></a>
#### `Arr::only()` {.collection-method}

Hàm `Arr::only` chỉ trả về các cặp key / giá trị được chỉ định từ mảng đã cho:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = Arr::only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `Arr::pluck()` {.collection-method}

Hàm `Arr::pluck` lấy tất cả các giá trị cho một key đã cho từ một mảng:

    use Illuminate\Support\Arr;

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = Arr::pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

Bạn cũng có thể khai báo thêm key cho mảng đó:

    use Illuminate\Support\Arr;

    $names = Arr::pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `Arr::prepend()` {.collection-method}

Hàm `Arr::prepend` sẽ thêm một item lên đầu của một mảng:

    use Illuminate\Support\Arr;

    $array = ['one', 'two', 'three', 'four'];

    $array = Arr::prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

Nếu cần, bạn có thể khai báo key cho giá trị đó:

    use Illuminate\Support\Arr;

    $array = ['price' => 100];

    $array = Arr::prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-prependkeyswith"></a>
#### `Arr::prependKeysWith()` {.collection-method}

Hàm `Arr::prependKeysWith` sẽ nối một tiền tố đã cho vào trước tất cả các khóa của một mảng:

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Desk',
        'price' => 100,
    ];

    $keyed = Arr::prependKeysWith($array, 'product.');

    /*
        [
            'product.name' => 'Desk',
            'product.price' => 100,
        ]
    */

<a name="method-array-pull"></a>
#### `Arr::pull()` {.collection-method}

Hàm `Arr::pull` trả về và xóa một cặp key / giá trị ra khỏi một mảng:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $name = Arr::pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Một giá trị mặc định có thể được truyền làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu key không tồn tại:

    use Illuminate\Support\Arr;

    $value = Arr::pull($array, $key, $default);

<a name="method-array-query"></a>
#### `Arr::query()` {.collection-method}

Hàm `Arr::query` sẽ chuyển đổi một mảng thành một chuỗi query:

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Taylor',
        'order' => [
            'column' => 'created_at',
            'direction' => 'desc'
        ]
    ];

    Arr::query($array);

    // name=Taylor&order[column]=created_at&order[direction]=desc

<a name="method-array-random"></a>
#### `Arr::random()` {.collection-method}

Hàm `Arr::random` sẽ trả về một giá trị ngẫu nhiên từ một mảng:

    use Illuminate\Support\Arr;

    $array = [1, 2, 3, 4, 5];

    $random = Arr::random($array);

    // 4 - (retrieved randomly)

Bạn cũng có thể chỉ định số lượng item sẽ được trả về làm tham số thứ hai. Lưu ý rằng việc cung cấp tham số này sẽ trả về một mảng ngay cả khi chỉ có một item mong muốn:

    use Illuminate\Support\Arr;

    $items = Arr::random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `Arr::set()` {.collection-method}

Hàm `Arr::set` sẽ set một giá trị trong một mảng bị lồng nhau bằng cách sử dụng ký hiệu "dot":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-shuffle"></a>
#### `Arr::shuffle()` {.collection-method}

Hàm `Arr::shuffle` sẽ trộn ngẫu nhiên các item có trong mảng:

    use Illuminate\Support\Arr;

    $array = Arr::shuffle([1, 2, 3, 4, 5]);

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-array-sort"></a>
#### `Arr::sort()` {.collection-method}

Hàm `Arr::sort` sẽ sắp xếp một mảng theo các giá trị của nó:

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sort($array);

    // ['Chair', 'Desk', 'Table']

Bạn cũng có thể sắp xếp mảng theo kết quả của closure đã cho:

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sort($array, function (array $value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-desc"></a>
#### `Arr::sortDesc()` {.collection-method}

Hàm `Arr::sortDesc` sẽ sắp xếp một mảng theo thứ tự giảm dần bằng các giá trị của chính nó:

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sortDesc($array);

    // ['Table', 'Desk', 'Chair']

Bạn cũng có thể sắp xếp một mảng theo kết quả của một closure đã cho:

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sortDesc($array, function (array $value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Table'],
            ['name' => 'Desk'],
            ['name' => 'Chair'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()` {.collection-method}

Hàm `Arr::sortRecursive` sẽ sắp xếp đệ quy một mảng bằng cách sử dụng hàm `sort` cho mảng không có key, còn nếu mảng đó có key thì sẽ dùng hàm `ksort`:

    use Illuminate\Support\Arr;

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
        ['one' => 1, 'two' => 2, 'three' => 3],
    ];

    $sorted = Arr::sortRecursive($array);

    /*
        [
            ['JavaScript', 'PHP', 'Ruby'],
            ['one' => 1, 'three' => 3, 'two' => 2],
            ['Li', 'Roman', 'Taylor'],
        ]
    */

Nếu bạn muốn kết quả được sắp xếp theo thứ tự giảm dần, bạn có thể sử dụng phương thức `Arr::sortRecursiveDesc`.

    $sorted = Arr::sortRecursiveDesc($array);

<a name="method-array-take"></a>
#### `Arr::take()` {.collection-method}

Hàm `Arr::take` sẽ trả về một mảng mới với số lượng item được chỉ định:

    use Illuminate\Support\Arr;

    $array = [0, 1, 2, 3, 4, 5];

    $chunk = Arr::take($array, 3);

    // [0, 1, 2]

Bạn cũng có thể truyền một số âm để lấy số phần tử được chỉ định từ cuối mảng trở về:

    $array = [0, 1, 2, 3, 4, 5];

    $chunk = Arr::take($array, -2);

    // [4, 5]

<a name="method-array-to-css-classes"></a>
#### `Arr::toCssClasses()` {.collection-method}

Hàm `Arr::toCssClasses` sẽ compile ra một chuỗi class CSS theo một điều kiện. Phương thức chấp nhận một mảng gồm các class trong đó khóa mảng sẽ chứa class hoặc các class mà bạn muốn thêm vào, trong khi giá trị là một biểu thức boolean. Nếu một phần tử mảng có một khóa là dạng số, thì nó sẽ luôn được đưa vào danh sách class được tạo:

    use Illuminate\Support\Arr;

    $isActive = false;
    $hasError = true;

    $array = ['p-4', 'font-bold' => $isActive, 'bg-red' => $hasError];

    $classes = Arr::toCssClasses($array);

    /*
        'p-4 bg-red'
    */

<a name="method-array-to-css-styles"></a>
#### `Arr::toCssStyles()` {.collection-method}

Hàm `Arr::toCssStyles` sẽ compile có điều kiện một chuỗi CSS style. Phương thức này chấp nhận một mảng các class trong đó khóa của mảng đó sẽ chứa class hoặc các class bạn muốn thêm, trong khi giá trị là một biểu thức boolean. Nếu phần tử của mảng đó có khóa là một số, thì nó sẽ luôn được thêm vào trong danh sách class được render:

```php
use Illuminate\Support\Arr;

$hasColor = true;

$array = ['background-color: blue', 'color: blue' => $hasColor];

$classes = Arr::toCssStyles($array);

/*
    'background-color: blue; color: blue;'
*/
```

Phương thức này sẽ hỗ trợ chức năng của Laravel cho phép [nối các class với các attribute bag của Blade component](/docs/{{version}}/blade#conditionally-merge-classes) cũng như [lệnh Blade](/docs/{{version}}/blade#conditional-classes) `@class`.

<a name="method-array-undot"></a>
#### `Arr::undot()` {.collection-method}

Hàm `Arr::undot` mở rộng một mảng một chiều sử dụng ký tự "chấm" thành một mảng nhiều chiều:

    use Illuminate\Support\Arr;

    $array = [
        'user.name' => 'Kevin Malone',
        'user.occupation' => 'Accountant',
    ];

    $array = Arr::undot($array);

    // ['user' => ['name' => 'Kevin Malone', 'occupation' => 'Accountant']]

<a name="method-array-where"></a>
#### `Arr::where()` {.collection-method}

Hàm `Arr::where` sẽ lọc một mảng bằng cách sử dụng closure:

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::where($array, function (string|int $value, int $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-where-not-null"></a>
#### `Arr::whereNotNull()` {.collection-method}

Hàm `Arr::whereNotNull` sẽ loại bỏ tất cả các giá trị `null` ra khỏi mảng đã cho:

    use Illuminate\Support\Arr;

    $array = [0, null];

    $filtered = Arr::whereNotNull($array);

    // [0 => 0]

<a name="method-array-wrap"></a>
#### `Arr::wrap()` {.collection-method}

Hàm `Arr::wrap` sẽ bao bọc giá trị đã cho vào trong một mảng. Nếu giá trị đã cho là một mảng, nó sẽ được trả về mà không cần sửa đổi gì:

    use Illuminate\Support\Arr;

    $string = 'Laravel';

    $array = Arr::wrap($string);

    // ['Laravel']

Nếu giá trị đã cho là `null`, một mảng trống sẽ được trả về:

    use Illuminate\Support\Arr;

    $array = Arr::wrap(null);

    // []

<a name="method-data-fill"></a>
#### `data_fill()` {.collection-method}

Hàm `data_fill` sẽ set một giá trị bị thiếu trong một mảng hoặc một đối tượng lồng nhau bằng cách sử dụng ký hiệu "dot":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

Hàm này cũng chấp nhận dấu hoa thị dưới dạng như một ký tự đại diện và sẽ điền vào mục tiêu tương ứng:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>
#### `data_get()` {.collection-method}

Hàm `data_get` lấy một giá trị từ một mảng hoặc một đối tượng lồng nhau bằng cách sử dụng ký hiệu "dot":

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

Hàm `data_get` cũng chấp nhận một giá trị mặc định, sẽ được trả về nếu không tìm thấy key được chỉ định:

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

Phương thức cũng chấp nhận các ký tự đại diện sử dụng bằng dấu hoa thị để có thể lấy ra bất kỳ khóa nào có trong một mảng hoặc một đối tượng:

    $data = [
        'product-one' => ['name' => 'Desk 1', 'price' => 100],
        'product-two' => ['name' => 'Desk 2', 'price' => 150],
    ];

    data_get($data, '*.name');

    // ['Desk 1', 'Desk 2'];

<a name="method-data-set"></a>
#### `data_set()` {.collection-method}

Hàm `data_set` sẽ set một giá trị trong một mảng hoặc một đối tượng lồng nhau bằng cách sử dụng ký hiệu "dot":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Hàm này cũng chấp nhận ký tự đại diện hoa thị và để set giá trị cho mục tiêu tương ứng:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

Mặc định, bất kỳ giá trị hiện có sẽ bị ghi đè. Nếu bạn chỉ muốn set một giá trị nếu nó không tồn tại, bạn có thể truyền `false` làm tham số thứ tư cho hàm:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, overwrite: false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-data-forget"></a>
#### `data_forget()` {.collection-method}

Hàm `data_forget` sẽ xóa một giá trị trong một mảng hoặc một đối tượng lồng nhau bằng cách sử dụng ký hiệu "dot":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_forget($data, 'products.desk.price');

    // ['products' => ['desk' => []]]

Hàm này cũng chấp nhận ký tự đại diện sử dụng dấu hoa thị và sẽ xóa các giá trị tương ứng:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_forget($data, 'products.*.price');

    /*
        [
            'products' => [
                ['name' => 'Desk 1'],
                ['name' => 'Desk 2'],
            ],
        ]
    */

<a name="method-head"></a>
#### `head()` {.collection-method}

Hàm `head` trả về phần tử đầu tiên trong mảng đã cho:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {.collection-method}

Hàm `last` trả về phần tử cuối cùng trong mảng đã cho:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="numbers"></a>
## Numbers

<a name="method-number-abbreviate"></a>
#### `Number::abbreviate()` {.collection-method}

Hàm `Number::abbreviate` sẽ trả về định dạng dễ đọc hơn cho giá trị số được cung cấp, với hàng đơn vị được viết tắt:

    use Illuminate\Support\Number;

    $number = Number::abbreviate(1000);

    // 1K

    $number = Number::abbreviate(489939);

    // 490K

    $number = Number::abbreviate(1230000, precision: 2);

    // 1.23M

<a name="method-number-clamp"></a>
#### `Number::clamp()` {.collection-method}

Hàm `Number::clamp` sẽ đảm bảo là một số nhất định sẽ nằm trong một phạm vi nhất định. Nếu số đó thấp hơn giá trị tối thiểu, thì giá trị tối thiểu sẽ được trả về. Nếu số đó cao hơn giá trị tối đa, thì giá trị tối đa sẽ được trả về:

    use Illuminate\Support\Number;

    $number = Number::clamp(105, min: 10, max: 100);

    // 100

    $number = Number::clamp(5, min: 10, max: 100);

    // 10

    $number = Number::clamp(10, min: 10, max: 100);

    // 10

    $number = Number::clamp(20, min: 10, max: 100);

    // 20

<a name="method-number-currency"></a>
#### `Number::currency()` {.collection-method}

Hàm `Number::currency` sẽ trả về giá trị tiền tệ của giá trị đã cho dưới dạng chuỗi:

    use Illuminate\Support\Number;

    $currency = Number::currency(1000);

    // $1,000

    $currency = Number::currency(1000, in: 'EUR');

    // €1,000

    $currency = Number::currency(1000, in: 'EUR', locale: 'de');

    // 1.000 €

<a name="method-number-file-size"></a>
#### `Number::fileSize()` {.collection-method}

Hàm `Number::fileSize` sẽ trả về giá trị kích thước file của một giá trị byte đã cho dưới dạng chuỗi:

    use Illuminate\Support\Number;

    $size = Number::fileSize(1024);

    // 1 KB

    $size = Number::fileSize(1024 * 1024);

    // 1 MB

    $size = Number::fileSize(1024, precision: 2);

    // 1.00 KB

<a name="method-number-for-humans"></a>
#### `Number::forHumans()` {.collection-method}

Hàm `Number::forHumans` sẽ trả về định dạng có thể đọc của một giá trị số được cung cấp:

    use Illuminate\Support\Number;

    $number = Number::forHumans(1000);

    // 1 thousand

    $number = Number::forHumans(489939);

    // 490 thousand

    $number = Number::forHumans(1230000, precision: 2);

    // 1.23 million

<a name="method-number-format"></a>
#### `Number::format()` {.collection-method}

Hàm `Number::format` sẽ định dạng số đã cho thành chuỗi ký tự cụ thể theo ngôn ngữ:

    use Illuminate\Support\Number;

    $number = Number::format(100000);

    // 100,000

    $number = Number::format(100000, precision: 2);

    // 100,000.00

    $number = Number::format(100000.123, maxPrecision: 2);

    // 100,000.12

    $number = Number::format(100000, locale: 'de');

    // 100.000

<a name="method-number-ordinal"></a>
#### `Number::ordinal()` {.collection-method}

Hàm `Number::ordinal` sẽ trả về số thứ tự của một số:

    use Illuminate\Support\Number;

    $number = Number::ordinal(1);

    // 1st

    $number = Number::ordinal(2);

    // 2nd

    $number = Number::ordinal(21);

    // 21st

<a name="method-number-percentage"></a>
#### `Number::percentage()` {.collection-method}

Hàm `Number::percentage` sẽ trả về phần trăm của giá trị đã cho dưới dạng chuỗi:

    use Illuminate\Support\Number;

    $percentage = Number::percentage(10);

    // 10%

    $percentage = Number::percentage(10, precision: 2);

    // 10.00%

    $percentage = Number::percentage(10.123, maxPrecision: 2);

    // 10.12%

    $percentage = Number::percentage(10, precision: 2, locale: 'de');

    // 10,00%

<a name="method-number-spell"></a>
#### `Number::spell()` {.collection-method}

Hàm `Number::spell` sẽ chuyển số đã cho thành một chuỗi các từ:

    use Illuminate\Support\Number;

    $number = Number::spell(102);

    // one hundred and two

    $number = Number::spell(88, locale: 'fr');

    // quatre-vingt-huit


Tham số `after` cho phép bạn chỉ định một giá trị mà nhỏ hơn số đã được nhập vào sẽ được viết ra:

    $number = Number::spell(10, after: 10);

    // 10

    $number = Number::spell(11, after: 10);

    // eleven

Tham số `until` cho phép bạn chỉ định một giá trị mà lớn hơn số đã được nhập vào sẽ được viết ra:

    $number = Number::spell(5, until: 10);

    // five

    $number = Number::spell(10, until: 10);

    // 10

<a name="method-number-use-locale"></a>
#### `Number::useLocale()` {.collection-method}

Hàm `Number::useLocale` sẽ thiết lập ngôn ngữ global mặc định cho số, điều này sẽ ảnh hưởng đến cách định dạng số và tiền tệ trong các lần gọi tiếp theo tới các phương thức của class `Number`:

    use Illuminate\Support\Number;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Number::useLocale('de');
    }

<a name="method-number-with-locale"></a>
#### `Number::withLocale()` {.collection-method}

Hàm `Number::withLocale` sẽ chạy lệnh closure đã cho bằng cách sử dụng ngôn ngữ được truyền vào cho hàm và sau đó khôi phục ngôn ngữ trước đó sau khi lệnh callback đã được chạy xong:

    use Illuminate\Support\Number;

    $number = Number::withLocale('de', function () {
        return Number::format(1500);
    });

<a name="paths"></a>
## Paths

<a name="method-app-path"></a>
#### `app_path()` {.collection-method}

Hàm `app_path` sẽ trả về đường dẫn đến thư mục `app` của ứng dụng. Bạn cũng có thể sử dụng hàm `app_path` để tạo đường dẫn đến file trong thư mục app:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {.collection-method}

Hàm `base_path` sẽ trả về đường dẫn đến thư mục root của ứng dụng. Bạn cũng có thể sử dụng hàm `base_path` để tạo đường dẫn đến file trong thư mục root của project:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {.collection-method}

Hàm `config_path` sẽ trả về đường dẫn đến thư mục `config` của ứng dụng. Bạn cũng có thể sử dụng hàm `config_path` để tạo đường dẫn đến file trong thư mục config:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {.collection-method}

Hàm `database_path` sẽ trả về đường dẫn đến thư mục `database` của ứng dụng. Bạn cũng có thể sử dụng hàm `database_path` để tạo đường dẫn đến file trong thư mục database:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-lang-path"></a>
#### `lang_path()` {.collection-method}

Hàm `lang_path` sẽ trả về đường dẫn đến thư mục `lang` của ứng dụng. Bạn cũng có thể sử dụng hàm `lang_path` để tạo đường dẫn đến file trong thư mục:

    $path = lang_path();

    $path = lang_path('en/messages.php');

> [!NOTE]
> Mặc định, Laravel không chứa thư mục `lang`. Nếu bạn muốn tùy chỉnh các file ngôn ngữ của Laravel, bạn có thể publish các file đó thông qua lệnh Artisan `lang:publish`.

<a name="method-mix"></a>
#### `mix()` {.collection-method}

Hàm `mix` sẽ trả về đường dẫn đến [file Mix đã được phiên bản hoá](/docs/{{version}}/mix):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` {.collection-method}

Hàm `public_path` sẽ trả về đường dẫn đến thư mục `public` của ứng dụng. Bạn cũng có thể sử dụng hàm `public_path` để tạo đường dẫn đến file trong thư mục public:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` {.collection-method}

Hàm `resource_path` sẽ trả về đường dẫn đến thư mục `resources` của ứng dụng. Bạn cũng có thể sử dụng hàm `resource_path` để tạo đường dẫn đến file trong thư mục resources:

    $path = resource_path();

    $path = resource_path('sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {.collection-method}

Hàm `storage_path` sẽ trả về đường dẫn đến thư mục `storage` của ứng dụng. Bạn cũng có thể sử dụng hàm `storage_path` để tạo đường dẫn đến file trong thư mục storage:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {.collection-method}

Hàm `action` sẽ tạo ra một URL cho một action của controller đã cho:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Nếu phương thức chấp nhận tham số cho route, bạn có thể truyền chúng làm tham số thứ hai cho phương thức:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {.collection-method}

Hàm `asset` sẽ tạo URL cho một asset bằng cách sử dụng scheme hiện tại của request (HTTP hoặc HTTPS):

    $url = asset('img/photo.jpg');

Bạn có thể cấu hình URL host cho asset bằng cách set biến `ASSET_URL` trong file `.env` của bạn. Điều này có thể hữu ích nếu bạn đang lưu trữ các asset của bạn trong một dịch vụ bên ngoài như Amazon S3 hoặc một dịch vụ CDN khác:

    // ASSET_URL=http://example.com/assets

    $url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg

<a name="method-route"></a>
#### `route()` {.collection-method}

Hàm `route` sẽ tạo một URL cho một [route đã được đặt tên](/docs/{{version}}/routing#named-routes):

    $url = route('route.name');

Nếu route có chấp nhận tham số, bạn có thể truyền chúng làm tham số thứ hai cho phương thức:

    $url = route('route.name', ['id' => 1]);

Mặc định, hàm `route` sẽ tạo ra một URL tuyệt đối. Nếu bạn muốn tạo một URL tương đối, bạn có thể truyền `false` làm tham số thứ ba cho phương thức:

    $url = route('route.name', ['id' => 1], false);

<a name="method-secure-asset"></a>
#### `secure_asset()` {.collection-method}

Hàm `secure_asset` sẽ tạo URL cho một asset bằng HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-secure-url"></a>
#### `secure_url()` {.collection-method}

Hàm `secure_url` sẽ tạo URL HTTPS cho đường dẫn đã cho. Các parameter của URL có thể được truyền vào thông qua tham số thứ hai của phương thức:

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-to-route"></a>
#### `to_route()` {.collection-method}

Hàm `to_route` sẽ tạo ra một [response HTTP chuyển hướng](/docs/{{version}}/responses#redirects) cho một [route đã được đặt tên](/docs/{{version}}/routing#named-routes):

    return to_route('users.show', ['user' => 1]);

Nếu cần, bạn cũng có thể truyền thêm một HTTP status code được gán cho chuyển hướng và thêm các response headers làm tham số thứ ba và thứ tư của phương thức `to_route`:

    return to_route('users.show', ['user' => 1], 302, ['X-Framework' => 'Laravel']);

<a name="method-url"></a>
#### `url()` {.collection-method}

Hàm `url` tạo ra một URL cho đường dẫn đã cho:

    $url = url('user/profile');

    $url = url('user/profile', [1]);

Nếu không có đường dẫn nào được cung cấp, một instance `Illuminate\Routing\UrlGenerator` sẽ được trả về:

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## Miscellaneous

<a name="method-abort"></a>
#### `abort()` {.collection-method}

Hàm `abort` sẽ đưa ra một [exception HTTP](/docs/{{version}}/errors#http-exceptions) được tạo bởi [exception handler](/docs/{{version}}/errors#the-exception-handler):

    abort(403);

Bạn cũng có thể cung cấp message và response header tùy biến của exception mà sẽ được gửi về trình duyệt:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {.collection-method}

Hàm `abort_if` sẽ đưa ra một exception HTTP nếu một biểu thức boolean đã cho là `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Giống như phương thức `abort`, bạn cũng có thể cung cấp response text cho exception làm tham số thứ ba và một mảng các response header tùy biến làm tham số thứ tư cho phương thức.

<a name="method-abort-unless"></a>
#### `abort_unless()` {.collection-method}

Hàm `abort_unless` sẽ đưa ra một exception HTTP nếu một biểu thức boolean đã cho là `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Giống như phương thức `abort`, bạn cũng có thể cung cấp response text cho exception làm tham số thứ ba và một mảng các response header tùy biến làm tham số thứ tư cho phương thức.

<a name="method-app"></a>
#### `app()` {.collection-method}

Hàm `app` trả về instance [service container](/docs/{{version}}/container):

    $container = app();

Bạn có thể truyền một tên class hoặc một tên interface để resolve nó từ container:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {.collection-method}

Hàm `auth` sẽ trả về một instance [authenticator](/docs/{{version}}/authentication). Bạn có thể sử dụng nó như là một thay thế cho facade `Auth`:

    $user = auth()->user();

Nếu cần, bạn có thể khai báo loại instance guard mà bạn muốn truy cập:

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {.collection-method}

Hàm `back` sẽ tạo ra một [response HTTP chuyển hướng](/docs/{{version}}/responses#redirects) đến vị trí trước đó của người dùng:

    return back($status = 302, $headers = [], $fallback = '/');

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {.collection-method}

Hàm `bcrypt` sẽ [hashes](/docs/{{version}}/hashing) giá trị đã cho bằng Bcrypt. Bạn có thể sử dụng phương thức này như là một thay thế cho facade `Hash`:

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>
#### `blank()` {.collection-method}

Hàm `blank` sẽ xác định xem giá trị đã cho là "blank" hay không:

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

Để tìm trái ngược của `blank`, hãy xem phương thức [`filled`](#method-filled).

<a name="method-broadcast"></a>
#### `broadcast()` {.collection-method}

Hàm `broadcast` sẽ [broadcasts](/docs/{{version}}/broadcasting) một [event](/docs/{{version}}/events) cho listener của nó:

    broadcast(new UserRegistered($user));

    broadcast(new UserRegistered($user))->toOthers();

<a name="method-cache"></a>
#### `cache()` {.collection-method}

Hàm `cache` có thể được sử dụng để lấy các giá trị từ [cache](/docs/{{version}}/cache). Nếu key đã cho không tồn tại trong cache, giá trị mặc định sẽ được trả về:

    $value = cache('key');

    $value = cache('key', 'default');

Bạn có thể thêm các item vào cache bằng cách truyền một mảng các cặp key / giá trị cho hàm. Bạn cũng nên truyền thêm số giây hoặc thời gian mà giá trị được lưu trong bộ nhớ cache sẽ được coi là hợp lệ:

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {.collection-method}

Hàm `class_uses_recursive` sẽ trả về tất cả các trait được sử dụng bởi một class, bao gồm cả các trait được sử dụng bởi tất cả các class cha của nó:

    $traits = class_uses_recursive(App\Models\User::class);

<a name="method-collect"></a>
#### `collect()` {.collection-method}

Hàm `collect` tạo ra một instance [collection](/docs/{{version}}/collections) từ giá trị đã cho:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {.collection-method}

Hàm `config` sẽ lấy giá trị của biến [configuration](/docs/{{version}}/configuration). Các giá trị cấu hình có thể được truy cập bằng cú pháp "dot", bao gồm tên của file và option bạn muốn truy cập. Giá trị mặc định có thể được khai báo và được trả về nếu tùy chọn cấu hình không tồn tại:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Bạn có thể set các biến cấu hình trong thời gian chạy bằng cách truyền một mảng các cặp key / giá trị. Tuy nhiên, lưu ý rằng chức năng này chỉ ảnh hưởng đến các giá trị cấu hình cho request hiện tại và không cập nhật giá trị cấu hình thực tế của bạn:

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()` {.collection-method}

Hàm `cookie` tạo một instance [cookie](/docs/{{version}}/requests#cookies) mới:

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()` {.collection-method}

Hàm `csrf_field` sẽ tạo ra một thẻ input `hidden` HTML chứa giá trị của CSRF token. Ví dụ: sử dụng [Blade syntax](/docs/{{version}}/blade):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {.collection-method}

Hàm `csrf_token` sẽ lấy ra giá trị của CSRF token hiện tại:

    $token = csrf_token();

<a name="method-decrypt"></a>
#### `decrypt()` {.collection-method}

Hàm `decrypt` sẽ [giải mã](/docs/{{version}}/encryption) giá trị đã cho. Bạn có thể sử dụng hàm này thay cho facade `Crypt`:

    $password = decrypt($value);

<a name="method-dd"></a>
#### `dd()` {.collection-method}

Hàm `dd` sẽ dump các biến đã cho và dừng thực thi lệnh:

    dd($value);

    dd($value1, $value2, $value3, ...);

Nếu bạn không muốn dừng việc thực thi lệnh của bạn, hãy sử dụng hàm [`dump`](#method-dump) để thay thế.

<a name="method-dispatch"></a>
#### `dispatch()` {.collection-method}

Hàm `dispatch` sẽ tạo [job](/docs/{{version}}/queues#creating-jobs) vào Laravel [job queue](/docs/{{version}}/queues):

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-sync"></a>
#### `dispatch_sync()` {.collection-method}

Hàm `dispatch_sync` sẽ gửi job đã cho vào [sync](/docs/{{version}}/queues#synchronous-dispatching) queue để job đó được xử lý ngay lập tức:

    dispatch_sync(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` {.collection-method}

Hàm `dump` sẽ dump các biến đã cho:

    dump($value);

    dump($value1, $value2, $value3, ...);

Nếu bạn muốn dừng thực thi lệnh sau khi dump các biến, hãy sử dụng hàm [`dd`](#method-dd) để thay thế.

<a name="method-encrypt"></a>
#### `encrypt()` {.collection-method}

Hàm `encrypt` sẽ [mã hóa](/docs/{{version}}/encryption) giá trị đã cho. Bạn có thể sử dụng hàm này thay cho facade `Crypt`:

    $secret = encrypt('my-secret-value');

<a name="method-env"></a>
#### `env()` {.collection-method}

Hàm `env` sẽ lấy ra giá trị của [environment variable](/docs/{{version}}/configuration#environment-configuration) hoặc trả về giá trị mặc định:

    $env = env('APP_ENV');

    $env = env('APP_ENV', 'production');

> [!WARNING]
> Nếu bạn chạy lệnh `config:cache` trong quá trình deploy của bạn, bạn nên chắc chắn rằng bạn chỉ gọi hàm `env` từ các file cấu hình của bạn. Khi các option cấu hình đã được lưu vào cached, file `.env` sẽ không được load và tất cả các lệnh gọi đến hàm `env` sẽ trả về `null`.

<a name="method-event"></a>
#### `event()` {.collection-method}

Hàm `event` sẽ dispatch [event](/docs/{{version}}/events) đến listener:

    event(new UserRegistered($user));

<a name="method-fake"></a>
#### `fake()` {.collection-method}

Hàm `fake` sẽ resolve một [Faker](https://github.com/FakerPHP/Faker) từ container, và có thể hữu ích khi tạo dữ liệu giả trong các model factory, database seeding, test và xem thử view:

```blade
@for($i = 0; $i < 10; $i++)
    <dl>
        <dt>Name</dt>
        <dd>{{ fake()->name() }}</dd>

        <dt>Email</dt>
        <dd>{{ fake()->unique()->safeEmail() }}</dd>
    </dl>
@endfor
```

Mặc định, hàm `fake` sẽ sử dụng tùy chọn cấu hình `app.faker_locale` trong file cấu hình `config/app.php` của bạn; tuy nhiên, bạn cũng có thể chỉ định ngôn ngữ này bằng cách truyền nó tới hàm `fake`. Mỗi ngôn ngữ sẽ được resolve ra một instance riêng biệt:

    fake('nl_NL')->name()

<a name="method-filled"></a>
#### `filled()` {.collection-method}

Hàm `filled` sẽ xác định xem giá trị đã cho không là "blank" hay không:

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

Để tìm trái ngược của `filled`, hãy xem phương thức [`blank`](#method-blank).

<a name="method-info"></a>
#### `info()` {.collection-method}

Hàm `info` sẽ ghi thông tin vào [log](/docs/{{version}}/logging) của application của bạn:

    info('Some helpful information!');

Một mảng dữ liệu theo ngữ cảnh cũng có thể được truyền cho hàm:

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {.collection-method}

Hàm `logger` có thể được sử dụng để viết một thông báo ở mức `debug` vào [log](/docs/{{version}}/logging):

    logger('Debug message');

Một mảng dữ liệu theo ngữ cảnh cũng có thể được truyền cho hàm:

    logger('User has logged in.', ['id' => $user->id]);

Một instance [logger](/docs/{{version}}/errors#logging) sẽ được trả về nếu không có giá trị nào được truyền vào cho hàm:

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()` {.collection-method}

Hàm `method_field` tạo ra thẻ input `hidden` HTML chứa giá trị HTTP action của form. Ví dụ: sử dụng [Blade syntax](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()` {.collection-method}

Hàm `now` sẽ tạo ra một instance `Illuminate\Support\Carbon` mới cho thời điểm hiện tại:

    $now = now();

<a name="method-old"></a>
#### `old()` {.collection-method}

Hàm `old` sẽ [lấy ra](/docs/{{version}}/requests#retrieving-input) một giá trị [old input](/docs/{{version}}/requests#old-input) được flash trong session :

    $value = old('value');

    $value = old('value', 'default');

Vì "giá trị mặc định" được cung cấp làm tham số thứ hai cho hàm `old` thường là một thuộc tính của model Eloquent, nên Laravel cho phép bạn chỉ cần truyền toàn bộ model Eloquent làm tham số thứ hai cho hàm `old`. Khi làm như vậy, Laravel sẽ coi tham số đầu tiên được cung cấp cho hàm `old` là tên của thuộc tính của Eloquent và cũng coi giá trị của thuộc tính đó trong Eloquent là "giá trị mặc định" nếu không tìm thấy giá trị đó trong session:

    {{ old('name', $user->name) }}

    // Is equivalent to...

    {{ old('name', $user) }}

<a name="method-optional"></a>
#### `optional()` {.collection-method}

Hàm `optional` nhận vào bất kỳ tham số nào và cho phép bạn truy cập vào các thuộc tính hoặc các phương thức trên đối tượng đó. Nếu đối tượng đã cho là `null`, thì các thuộc tính hoặc các phương thức đó sẽ trả về `null` thay vì gây ra lỗi:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

Hàm `optional` cũng chấp nhận một closure làm tham số thứ hai của nó. Closure sẽ được gọi nếu giá trị tham số đầu tiên không phải là một giá trị null:

    return optional(User::find($id), function (User $user) {
        return $user->name;
    });

<a name="method-policy"></a>
#### `policy()` {.collection-method}

Hàm `policy` sẽ lấy ra một instance [policy](/docs/{{version}}/authorization#creating-policies) cho một class nhất định:

    $policy = policy(App\Models\User::class);

<a name="method-redirect"></a>
#### `redirect()` {.collection-method}

Hàm `redirect` sẽ trả về một [response HTTP chuyển hướng](/docs/{{version}}/responses#redirects) hoặc trả về instance chuyển hướng nếu không có tham số được truyền vào:

    return redirect($to = null, $status = 302, $headers = [], $https = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {.collection-method}

Hàm `report` sẽ report một exception bằng cách sử dụng [exception handler](/docs/{{version}}/errors#the-exception-handler) của bạn:

    report($e);

Hàm `report` cũng sẽ chấp nhận một chuỗi làm tham số đầu vào. Khi một chuỗi được cấp cho hàm, hàm sẽ tạo ra một ngoại lệ với chuỗi đã cho dưới dạng một thông báo của nó:

    report('Something went wrong.');

<a name="method-report-if"></a>
#### `report_if()` {.collection-method}

Hàm `report_if` sẽ report ra một ngoại lệ bằng cách sử dụng [exception handler](/docs/{{version}}/errors#the-exception-handler) của bạn nếu điều kiện đã cho là `true`:

    report_if($shouldReport, $e);

    report_if($shouldReport, 'Something went wrong.');

<a name="method-report-unless"></a>
#### `report_unless()` {.collection-method}

Hàm `report_unless` sẽ report ra một ngoại lệ bằng cách sử dụng [exception handler](/docs/{{version}}/errors#the-exception-handler) của bạn nếu điều kiện đã cho là `false`:

    report_unless($reportingDisabled, $e);

    report_unless($reportingDisabled, 'Something went wrong.');

<a name="method-request"></a>
#### `request()` {.collection-method}

Hàm `request` trả về instance [request](/docs/{{version}}/requests) hiện tại hoặc lấy ra một giá trị của trường input từ request hiện tại:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` {.collection-method}

Hàm `rescue` sẽ thực thi closure đã cho và catch bất kỳ exception nào xảy ra trong quá trình thực thi. Tất cả các exception bị catch sẽ được gửi đến [exception handler](/docs/{{version}}/errors#the-exception-handler) của bạn; tuy nhiên, request sẽ tiếp tục xử lý:

    return rescue(function () {
        return $this->method();
    });

Bạn cũng có thể truyền tham số thứ hai cho hàm `rescue`. Tham số này sẽ là giá trị "default" cần được trả về nếu có exception xảy ra trong khi thực hiện closure:

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

Có thể cung cấp tham số `report` cho hàm `rescue` để xác định xem ngoại lệ có được report thông qua hàm `report` hay không:

    return rescue(function () {
        return $this->method();
    }, report: function (Throwable $throwable) {
        return $throwable instanceof InvalidArgumentException;
    });

<a name="method-resolve"></a>
#### `resolve()` {.collection-method}

Hàm `resolve` sẽ resolve một tên class hoặc một interface đã cho thành một instance bằng cách sử dụng [service container](/docs/{{version}}/container):

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` {.collection-method}

Hàm `response` tạo ra một instance [response](/docs/{{version}}/responses) hoặc lấy ra một instance của response factory:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {.collection-method}

Hàm `retry` sẽ thử thực hiện callback đã cho, cho đến khi đạt được ngưỡng thử tối đa nào đó. Nếu callback không đưa ra exception, chính giá trị trả về của nó sẽ được trả về. Nếu callback đưa ra một exception, nó sẽ tự động được thử lại. Nếu vượt quá số lần thử tối đa, exception sẽ bị đưa ra:

    return retry(5, function () {
        // Attempt 5 times while resting 100ms between attempts...
    }, 100);

Nếu bạn muốn đưa vào một số lượng mili giây để ngủ giữa các lần thử, bạn có thể truyền một closure làm tham số thứ ba cho hàm `retry`:

    use Exception;

    return retry(5, function () {
        // ...
    }, function (int $attempt, Exception $exception) {
        return $attempt * 100;
    });

Để thuận tiện, bạn cũng có thể cung cấp một mảng làm tham số đầu tiên cho hàm `retry`. Mảng này sẽ được sử dụng để xác định số mili giây sẽ ngủ giữa các lần thử tiếp theo:

    return retry([100, 200], function () {
        // Sleep for 100ms on first retry, 200ms on second retry...
    });

Để chỉ thử lại trong một điều kiện cụ thể, bạn có thể truyền một closure làm tham số thứ tư cho hàm `retry`:

    use Exception;

    return retry(5, function () {
        // ...
    }, 100, function (Exception $exception) {
        return $exception instanceof RetryException;
    });

<a name="method-session"></a>
#### `session()` {.collection-method}

Hàm `session` có thể được sử dụng để lấy hoặc set các giá trị [session](/docs/{{version}}/session) values:

    $value = session('key');

Bạn có thể set giá trị bằng cách truyền một mảng các cặp key / giá trị cho hàm:

    session(['chairs' => 7, 'instruments' => 3]);

Session store sẽ được trả về nếu không có giá trị nào được truyền cho hàm:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {.collection-method}

Hàm `tap` sẽ nhận vào hai tham số: một là `$value` và một closure. `$value` sẽ được truyền đến phần closure và sau đó được trả về bởi hàm `tap`. Giá trị trả về của closure sẽ không liên quan:

    $user = tap(User::first(), function (User $user) {
        $user->name = 'taylor';

        $user->save();
    });

Nếu không có closure nào được truyền đến hàm `tap`, bạn có thể gọi bất kỳ phương thức nào trên `$value` đã cho. Giá trị trả về của phương thức bạn gọi sẽ luôn là `$value`, bất kể phương thức đó thực sự trả về định nghĩa gì đi chăng nữa. Ví dụ, phương thức `update` Eloquent thường trả về một số nguyên. Tuy nhiên, chúng ta có thể buộc phương thức này trả về chính model đó bằng cách gọi phương thức `update` thông qua hàm `tap`:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

Để thêm một phương thức `tap` vào một class, bạn có thể thêm trait `Illuminate\Support\Traits\Tappable` vào class. Hàm `tap` của trait này sẽ chấp nhận một Closure làm tham số duy nhất của nó. Chính instance đối tượng sẽ được truyền đến Closure và sau đó được trả về bởi phương thức `tap`:

    return $user->tap(function (User $user) {
        // ...
    });

<a name="method-throw-if"></a>
#### `throw_if()` {.collection-method}

Hàm `throw_if` sẽ đưa ra exception đã cho nếu một biểu thức boolean đã cho là `true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page.'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` {.collection-method}

Hàm `throw_unless` sẽ đưa ra exception đã cho nếu một biểu thức boolean đã cho là `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page.'
    );

<a name="method-today"></a>
#### `today()` {.collection-method}

Hàm `today` sẽ tạo ra một instance `Illuminate\Support\Carbon` mới cho ngày hiện tại:

    $today = today();

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {.collection-method}

Hàm `trait_uses_recursive` trả về tất cả các trait được sử dụng bởi một trait:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {.collection-method}

Hàm `transform` sẽ thực thi một closure trên một giá trị đã cho nếu giá trị không [blank](#method-blank) và sau đó trả về giá trị trả về của một closure:

    $callback = function (int $value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

Một giá trị mặc định hoặc một closure có thể được truyền làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu giá trị đã cho là blank:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {.collection-method}

Hàm `validator` sẽ tạo ra một instance [validator](/docs/{{version}}/validation) mới với các tham số đã cho. Bạn có thể sử dụng nó như là một thay thế cho facade `Auth`:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {.collection-method}

Hàm `value` sẽ trả về giá trị được cho. Tuy nhiên, nếu bạn truyền một closure cho hàm, thì closure sẽ được thực thi và giá trị trả về của nó sẽ được trả về:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

Các tham số bổ sung khác cũng có thể được truyền đến hàm `value`. Nếu tham số đầu tiên là một closure thì các tham số bổ sung tiếp theo sẽ được truyền đến closure dưới dạng các tham số, nếu không chúng sẽ bị bỏ qua:

    $result = value(function (string $name) {
        return $name;
    }, 'Taylor');

    // 'Taylor'

<a name="method-view"></a>
#### `view()` {.collection-method}

Hàm `view` sẽ lấy ra một instance [view](/docs/{{version}}/views):

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {.collection-method}

Hàm `with` sẽ trả về giá trị được cho. Nếu một closure được truyền làm tham số thứ hai cho hàm, thì closure đó sẽ được thực thi và giá trị trả về của nó sẽ được trả về:

    $callback = function (mixed $value) {
        return is_numeric($value) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5

<a name="other-utilities"></a>
## Các class hữu ích khác

<a name="benchmarking"></a>
### Benchmarking

Thỉnh thoảng bạn có thể muốn kiểm tra nhanh hiệu suất của một số phần nhất định trong ứng dụng của bạn. Trong những trường hợp đó, bạn có thể sử dụng class hỗ trợ `Benchmark` để đo số mili giây cần thiết để hoàn thành các callback nhất định:

    <?php

    use App\Models\User;
    use Illuminate\Support\Benchmark;

    Benchmark::dd(fn () => User::find(1)); // 0.1 ms

    Benchmark::dd([
        'Scenario 1' => fn () => User::count(), // 0.5 ms
        'Scenario 2' => fn () => User::all()->count(), // 20.0 ms
    ]);

Mặc định, các callback đã cho sẽ được thực hiện một lần và thời gian thực hiện của chúng sẽ được hiển thị trong trình duyệt hoặc console.

Để gọi một callback nhiều lần, bạn có thể chỉ định số lần lặp mà callback sẽ được gọi làm tham số thứ hai cho phương thức. Khi thực hiện callback nhiều lần, class `Benchmark` sẽ trả về lượng mili giây trung bình cần thiết để thực hiện callback trên tất cả các lần lặp:

    Benchmark::dd(fn () => User::count(), iterations: 10); // 0.5 ms

Thỉnh thoảng, bạn có thể muốn đánh giá việc thực hiện lệnh callback trong khi vẫn lấy ra giá trị trả về của lệnh callback. Phương thức `value` sẽ trả về một giá trị trả về của lệnh callback và số mili giây cần thiết để thực hiện lệnh callback:

    [$count, $duration] = Benchmark::value(fn () => User::count());

<a name="dates"></a>
### Dates

Laravel có chứa [Carbon](https://carbon.nesbot.com/docs/), một thư viện xử lý ngày và giờ mạnh mẽ. Để tạo một instance `Carbon` mới, bạn có thể gọi hàm `now`. Hàm này có sẵn trong toàn bộ ứng dụng Laravel của bạn:

```php
$now = now();
```

Hoặc, bạn có thể tạo một instance `Carbon` mới bằng cách sử dụng class `Illuminate\Support\Carbon`:

```php
use Illuminate\Support\Carbon;

$now = Carbon::now();
```

Để thảo luận kỹ hơn về Carbon và các tính năng của nó, vui lòng tham khảo [tài liệu chính thức của Carbon](https://carbon.nesbot.com/docs/).

<a name="lottery"></a>
### Lottery

Class lottery của Laravel có thể được sử dụng để thực hiện lệnh callback dựa trên một tập hợp tỷ lệ nhất định. Điều này có thể đặc biệt hữu ích khi bạn chỉ muốn thực hiện code trên một tỷ lệ phần trăm các request được gửi đến của bạn:

    use Illuminate\Support\Lottery;

    Lottery::odds(1, 20)
        ->winner(fn () => $user->won())
        ->loser(fn () => $user->lost())
        ->choose();

Bạn có thể kết hợp class lottery của Laravel với các tính năng khác của Laravel. Ví dụ: bạn có thể chỉ muốn report một tỷ lệ nhỏ các truy vấn chậm trong exception handler của bạn. Và, vì class lottery là một callable được nên chúng ta có thể truyền một instance của class đó vào bất kỳ phương thức nào mà chấp nhận một callable:

    use Carbon\CarbonInterval;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Lottery;

    DB::whenQueryingForLongerThan(
        CarbonInterval::seconds(2),
        Lottery::odds(1, 100)->winner(fn () => report('Querying > 2 seconds.')),
    );

<a name="testing-lotteries"></a>
#### Testing Lotteries

Laravel cung cấp một số phương thức đơn giản để cho phép bạn dễ dàng kiểm tra các lottery trong ứng dụng của bạn:

    // Lottery will always win...
    Lottery::alwaysWin();

    // Lottery will always lose...
    Lottery::alwaysLose();

    // Lottery will win then lose, and finally return to normal behavior...
    Lottery::fix([true, false]);

    // Lottery will return to normal behavior...
    Lottery::determineResultsNormally();

<a name="pipeline"></a>
### Pipeline

Facade `Pipeline` của Laravel cung cấp một cách thuận tiện để "dẫn" một input nhất định qua một loạt các invokable class, closure hoặc callable, cung cấp cho mỗi invokable class cơ hội kiểm tra hoặc sửa input và gọi callable tiếp theo trong pipeline:

```php
use Closure;
use App\Models\User;
use Illuminate\Support\Facades\Pipeline;

$user = Pipeline::send($user)
            ->through([
                function (User $user, Closure $next) {
                    // ...

                    return $next($user);
                },
                function (User $user, Closure $next) {
                    // ...

                    return $next($user);
                },
            ])
            ->then(fn (User $user) => $user);
```

Như bạn có thể thấy, mỗi invokable class hoặc closure có thể được gọi trong pipeline cùng với việc được cung cấp input cho các invokable class hoặc closure đó và cuối cùng là closure `$next`. Việc gọi closure `$next` sẽ gọi callable tiếp theo trong pipeline. Như bạn có thể thấy, điều này rất giống với [middleware](/docs/{{version}}/middleware).

Khi callable cuối cùng trong pipeline gọi closure `$next`, callable được cung cấp cho phương thức `then` sẽ được gọi. Thông thường, callable này sẽ chỉ trả về input đã cho.

Tất nhiên, như đã thảo luận trước đó, bạn không bị giới hạn trong việc cung cấp closure cho pipeline của bạn. Bạn cũng có thể cung cấp các invokable class. Nếu tên class được truyền vào, thì class đó sẽ được khởi tạo thông qua [service container](/docs/{{version}}/container) của Laravel, cho phép các dependency được inject vào invokable class:

```php
$user = Pipeline::send($user)
            ->through([
                GenerateProfilePhoto::class,
                ActivateSubscription::class,
                SendWelcomeEmail::class,
            ])
            ->then(fn (User $user) => $user);
```

<a name="sleep"></a>
### Sleep

Class `Sleep` của Laravel là một class wrapper nhẹ cho các hàm `sleep` và `usleep` của PHP, cung cấp khả năng kiểm tra tốt hơn đồng thời cung cấp API thân thiện hơn cho nhà phát triển để làm việc với thời gian:

    use Illuminate\Support\Sleep;

    $waiting = true;

    while ($waiting) {
        Sleep::for(1)->second();

        $waiting = /* ... */;
    }

Class `Sleep` cung cấp nhiều phương thức khác nhau cho phép bạn làm việc với các đơn vị thời gian khác nhau:

    // Pause execution for 90 seconds...
    Sleep::for(1.5)->minutes();

    // Pause execution for 2 seconds...
    Sleep::for(2)->seconds();

    // Pause execution for 500 milliseconds...
    Sleep::for(500)->milliseconds();

    // Pause execution for 5,000 microseconds...
    Sleep::for(5000)->microseconds();

    // Pause execution until a given time...
    Sleep::until(now()->addMinute());

    // Alias of PHP's native "sleep" function...
    Sleep::sleep(2);

    // Alias of PHP's native "usleep" function...
    Sleep::usleep(5000);

Để dễ dàng kết hợp với các đơn vị thời gian khác, bạn có thể sử dụng phương thức `and`:

    Sleep::for(1)->second()->and(10)->milliseconds();

<a name="testing-sleep"></a>
#### Testing Sleep

Khi kiểm tra các code mà sử dụng class `Sleep` hoặc các hàm sleep gốc của PHP, bài kiểm tra của bạn sẽ phải tạm dừng thực hiện khi chạy vào hàm sleep. Như bạn có thể thấy, điều này làm cho bài kiểm tra của bạn chậm hơn đáng kể. Ví dụ, hãy tưởng tượng bạn đang kiểm tra code sau:

    $waiting = /* ... */;

    $seconds = 1;

    while ($waiting) {
        Sleep::for($seconds++)->seconds();

        $waiting = /* ... */;
    }

Thông thường, việc kiểm tra code này sẽ mất _ít nhất_ một giây để chờ sleep. May mắn thay, class `Sleep` cho phép chúng ta "fake" thời gian sleep để bài kiểm tra của bạn vẫn chạy được nhanh:

    public function test_it_waits_until_ready()
    {
        Sleep::fake();

        // ...
    }

Khi fake class `Sleep`, việc tạm dừng để sleep thực tế sẽ bị bỏ qua, dẫn đến bài kiểm tra của chúng ta nhanh hơn đáng kể.

Sau khi class `Sleep` đã được fake, bạn có thể đưa ra các kiểm tra cho các "sleep" dự kiến ​​đáng lẽ phải xảy ra. Để minh họa điều này, hãy tưởng tượng chúng ta đang thử nghiệm code tạm dừng thực thi ba lần, với mỗi lần tạm dừng tăng thêm một giây. Sử dụng phương thức `assertSequence`, chúng ta có thể kiểm tra code của chúng ta đã "sleep" trong khoảng thời gian thích hợp trong khi vẫn giữ cho bài kiểm tra của chúng ta được nhanh:

    public function test_it_checks_if_ready_four_times()
    {
        Sleep::fake();

        // ...

        Sleep::assertSequence([
            Sleep::for(1)->second(),
            Sleep::for(2)->seconds(),
            Sleep::for(3)->seconds(),
        ]);
    }

Tất nhiên, class `Sleep` cung cấp nhiều kiểm tra khác mà bạn có thể sử dụng khi testing:

    use Carbon\CarbonInterval as Duration;
    use Illuminate\Support\Sleep;

    // Assert that sleep was called 3 times...
    Sleep::assertSleptTimes(3);

    // Assert against the duration of sleep...
    Sleep::assertSlept(function (Duration $duration): bool {
        return /* ... */;
    }, times: 1);

    // Assert that the Sleep class was never invoked...
    Sleep::assertNeverSlept();

    // Assert that, even if Sleep was called, no execution paused occurred...
    Sleep::assertInsomniac();

Thỉnh thoảng, có thể hữu ích khi thực hiện một hành động nào đó khi một fake sleep xảy ra trong code ứng dụng của bạn. Để đạt được điều này, bạn có thể cung cấp một lệnh callback cho phương thức `whenFakingSleep`. Trong ví dụ sau, chúng ta sử dụng [helper tương tác với time](/docs/{{version}}/mocking#interacting-with-time) của Laravel để đưa thời gian hiện tại đến luôn thời gian sau mỗi lần sleep:

```php
use Carbon\CarbonInterval as Duration;

$this->freezeTime();

Sleep::fake();

Sleep::whenFakingSleep(function (Duration $duration) {
    // Progress time when faking sleep...
    $this->travel($duration->totalMilliseconds)->milliseconds();
});
```

Laravel sử dụng class `Sleep` ở bên trong bất cứ khi nào code cần tạm dừng thực thi. Ví dụ, helper [`retry`](#method-retry) sử dụng class `Sleep` để chờ cho đến khi một hành động nào đó được thực hiện lại, cho phép cải thiện khả năng kiểm tra khi sử dụng helper này.
