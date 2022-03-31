# Helpers

- [Giới thiệu](#introduction)
- [Các phương thức có sẵn](#available-methods)

<a name="introduction"></a>
## Giới thiệu

Laravel chứa một loạt các hàm PHP global "helper". Nhiều trong số các hàm này được sử dụng bởi chính framework; tuy nhiên, bạn có thể thoải mái sử dụng chúng trong application của bạn nếu bạn thấy chúng tiện ích.

<a name="available-methods"></a>
## Các phương thức có sẵn

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### Arrays & Objects

<div class="collection-method-list" markdown="1">

[Arr::add](#method-array-add)
[Arr::collapse](#method-array-collapse)
[Arr::divide](#method-array-divide)
[Arr::dot](#method-array-dot)
[Arr::except](#method-array-except)
[Arr::first](#method-array-first)
[Arr::flatten](#method-array-flatten)
[Arr::forget](#method-array-forget)
[Arr::get](#method-array-get)
[Arr::has](#method-array-has)
[Arr::last](#method-array-last)
[Arr::only](#method-array-only)
[Arr::pluck](#method-array-pluck)
[Arr::prepend](#method-array-prepend)
[Arr::pull](#method-array-pull)
[Arr::random](#method-array-random)
[Arr::set](#method-array-set)
[Arr::sort](#method-array-sort)
[Arr::sortRecursive](#method-array-sort-recursive)
[Arr::where](#method-array-where)
[Arr::wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[head](#method-head)
[last](#method-last)
</div>

### Paths

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### Strings

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[class_basename](#method-class-basename)
[e](#method-e)
[preg_replace_array](#method-preg-replace-array)
[Str::after](#method-str-after)
[Str::before](#method-str-before)
[Str::camel](#method-camel-case)
[Str::contains](#method-str-contains)
[Str::containsAll](#method-str-contains-all)
[Str::endsWith](#method-ends-with)
[Str::finish](#method-str-finish)
[Str::is](#method-str-is)
[Str::kebab](#method-kebab-case)
[Str::limit](#method-str-limit)
[Str::orderedUuid](#method-str-ordered-uuid)
[Str::plural](#method-str-plural)
[Str::random](#method-str-random)
[Str::replaceArray](#method-str-replace-array)
[Str::replaceFirst](#method-str-replace-first)
[Str::replaceLast](#method-str-replace-last)
[Str::singular](#method-str-singular)
[Str::slug](#method-str-slug)
[Str::snake](#method-snake-case)
[Str::start](#method-str-start)
[Str::startsWith](#method-starts-with)
[Str::studly](#method-studly-case)
[Str::title](#method-title-case)
[Str::uuid](#method-str-uuid)
[Str::words](#method-str-words)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### URLs

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[route](#method-route)
[secure_asset](#method-secure-asset)
[secure_url](#method-secure-url)
[url](#method-url)

</div>

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
[dd](#method-dd)
[decrypt](#method-decrypt)
[dispatch](#method-dispatch)
[dispatch_now](#method-dispatch-now)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
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

<a name="method-listing"></a>
## Method Listing

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## Arrays & Objects

<a name="method-array-add"></a>
#### `Arr::add()` {#collection-method .first-collection-method}

Hàm `Arr::add` sẽ thêm một cặp key / giá trị vào một mảng nếu key đó không tồn tại trong mảng hoặc giá trị trong mảng của key đó bằng `null`:

    use Illuminate\Support\Arr;

    $array = Arr::add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

    $array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]


<a name="method-array-collapse"></a>
#### `Arr::collapse()` {#collection-method}

Hàm `Arr::collapse` sẽ thu gọn một mảng gồm nhiều mảng con thành một mảng duy nhất:

    use Illuminate\Support\Arr;

    $array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `Arr::divide()` {#collection-method}

Hàm `Arr::divide` trả về hai mảng, một mảng chứa các key và một mảng chứa các giá trị của mảng đã cho:

    use Illuminate\Support\Arr;

    [$keys, $values] = Arr::divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `Arr::dot()` {#collection-method}

Hàm `Arr::dot` sẽ làm ngang hàng một mảng nhiều chiều thành một mảng một chiều sử dụng ký hiệu "dot" để biểu thị độ sâu:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = Arr::dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `Arr::except()` {#collection-method}

Hàm `Arr::except` loại bỏ các cặp key / giá trị đã cho ra khỏi một mảng:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = Arr::except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `Arr::first()` {#collection-method}

Hàm `Arr::first` trả về phần tử đầu tiên của mảng pass qua một số điều kiện đã cho:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300];

    $first = Arr::first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

Một giá trị mặc định cũng có thể được truyền làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu không có giá trị nào được pass qua điều kiện:

    use Illuminate\Support\Arr;

    $first = Arr::first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `Arr::flatten()` {#collection-method}

Hàm `Arr::flatten` làm ngang hàng một mảng nhiều chiều thành một mảng một chiều:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = Arr::flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `Arr::forget()` {#collection-method}

Hàm `Arr::forget` xóa một cặp key / giá trị đã cho ra khỏi một mảng bị lồng vào nhau bằng cách sử dụng ký hiệu "dot":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `Arr::get()` {#collection-method}

Hàm `Arr::get` lấy một giá trị từ một mảng bị lồng vào nhau bằng cách sử dụng ký hiệu "dot":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = Arr::get($array, 'products.desk.price');

    // 100

Hàm `Arr::get` cũng chấp nhận một giá trị mặc định, sẽ được trả về nếu không tìm thấy key:

    use Illuminate\Support\Arr;

    $discount = Arr::get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `Arr::has()` {#collection-method}

Hàm `Arr::has` sẽ kiểm tra xem một item hoặc các item đã cho có tồn tại trong một mảng hay không bằng cách sử dụng ký hiệu "dot":

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::has($array, 'product.name');

    // true

    $contains = Arr::has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-last"></a>
#### `Arr::last()` {#collection-method}

Hàm `Arr::last` trả về phần tử cuối cùng của mảng pass qua một số điều kiện đã cho:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300, 110];

    $last = Arr::last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

Một giá trị mặc định có thể được truyền làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu không có giá trị nào pass qua điều kiện:

    use Illuminate\Support\Arr;

    $last = Arr::last($array, $callback, $default);

<a name="method-array-only"></a>
#### `Arr::only()` {#collection-method}

Hàm `Arr::only` chỉ trả về các cặp key / giá trị được chỉ định từ mảng đã cho:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = Arr::only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `Arr::pluck()` {#collection-method}

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
#### `Arr::prepend()` {#collection-method}

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

<a name="method-array-pull"></a>
#### `Arr::pull()` {#collection-method}

Hàm `Arr::pull` trả về và xóa một cặp key / giá trị ra khỏi một mảng:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $name = Arr::pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Một giá trị mặc định có thể được truyền làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu key không tồn tại:

    use Illuminate\Support\Arr;

    $value = Arr::pull($array, $key, $default);

<a name="method-array-random"></a>
#### `Arr::random()` {#collection-method}

Hàm `Arr::random` sẽ trả về một giá trị ngẫu nhiên từ một mảng:

    use Illuminate\Support\Arr;

    $array = [1, 2, 3, 4, 5];

    $random = Arr::random($array);

    // 4 - (retrieved randomly)

Bạn cũng có thể chỉ định số lượng item sẽ được trả về làm tham số thứ hai. Lưu ý rằng việc cung cấp tham số này sẽ trả về một mảng, ngay cả khi chỉ có một item mong muốn:

    use Illuminate\Support\Arr;

    $items = Arr::random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `Arr::set()` {#collection-method}

Hàm `Arr::set` sẽ set một giá trị trong một mảng bị lồng nhau bằng cách sử dụng ký hiệu "dot":

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `Arr::sort()` {#collection-method}

Hàm `Arr::sort` sẽ sắp xếp một mảng theo các giá trị của nó:

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sort($array);

    // ['Chair', 'Desk', 'Table']

Bạn cũng có thể sắp xếp mảng theo kết quả của Closure đã cho:

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()` {#collection-method}

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

<a name="method-array-where"></a>
#### `Arr::where()` {#collection-method}

Hàm `Arr::where` sẽ lọc một mảng bằng cách sử dụng Closure:

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-wrap"></a>
#### `Arr::wrap()` {#collection-method}

Hàm `Arr::wrap` sẽ bao bọc giá trị đã cho vào trong một mảng. Nếu giá trị đã cho là một mảng, nó sẽ không bị thay đổi:

    use Illuminate\Support\Arr;

    $string = 'Laravel';

    $array = Arr::wrap($string);

    // ['Laravel']

Nếu giá trị đã cho là null, một mảng trống sẽ được trả về:

    use Illuminate\Support\Arr;

    $nothing = null;

    $array = Arr::wrap($nothing);

    // []

<a name="method-data-fill"></a>
#### `data_fill()` {#collection-method}

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
#### `data_get()` {#collection-method}

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
#### `data_set()` {#collection-method}

Hàm `data_set` sẽ set một giá trị trong một mảng hoặc một đối tượng lồng nhau bằng cách sử dụng ký hiệu "dot":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Hàm này cũng chấp nhận ký tự đại diện và để set giá trị cho mục tiêu tương ứng:

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

Mặc định, bất kỳ giá trị hiện có sẽ bị ghi đè. Nếu bạn chỉ muốn set một giá trị nếu nó không tồn tại, bạn có thể truyền `false` làm tham số thứ tư:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-head"></a>
#### `head()` {#collection-method}

Hàm `head` trả về phần tử đầu tiên trong mảng đã cho:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

Hàm `last` trả về phần tử cuối cùng trong mảng đã cho:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## Paths

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

Hàm `app_path` trả về đường dẫn đến thư mục `app`. Bạn cũng có thể sử dụng hàm `app_path` để tạo đường dẫn đến một file có bắt đầu từ thư mục app:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

Hàm `base_path` trả về đường dẫn đến thư mục gốc dự án. Bạn cũng có thể sử dụng hàm `base_path` để tạo đường dẫn đến một file đã cho có bắt đầu từ thư mục gốc của dự án:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

Hàm `config_path` trả về đường dẫn đến thư mục `config`. Bạn cũng có thể sử dụng hàm `config_path` để tạo đường dẫn đến một file đã cho trong thư mục config của application:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

Hàm `database_path` trả về đường dẫn đến thư mục `database`. Bạn cũng có thể sử dụng hàm `database_path` để tạo đường dẫn đến một file đã cho trong thư mục database:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-mix"></a>
#### `mix()` {#collection-method}

Hàm `mix` trả về đường dẫn đến [file Mix đã được version](/docs/{{version}}/mix):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

Hàm `public_path` trả về đường dẫn đến thư mục `public`. Bạn cũng có thể sử dụng hàm `public_path` để tạo đường dẫn đến một file đã cho trong thư mục public:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

Hàm `resource_path` trả về đường dẫn đến thư mục `resource`. Bạn cũng có thể sử dụng hàm `resource_path` để tạo đường dẫn đến một file đã cho trong thư mục resources:

    $path = resource_path();

    $path = resource_path('sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

Hàm `storage_path` trả về đường dẫn đến thư mục` storage`. Bạn cũng có thể sử dụng hàm `storage_path` để tạo đường dẫn đến một file đã cho trong thư mục storage:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Strings

<a name="method-__"></a>
#### `__()` {#collection-method}

Hàm `__` sẽ dịch chuỗi cần được dịch hoặc key cần được dịch đã cho bằng cách sử dụng [localization files](/docs/{{version}}/localization) của bạn:

    echo __('Welcome to our application');

    echo __('messages.welcome');

Nếu chuỗi hoặc key cần được dịch không tồn tại, hàm `__` sẽ trả về giá trị được đưa vào. Vì vậy, nếu sử dụng ví dụ mẫu trên, hàm `__` sẽ trả về `messages.welcome` nếu key cần được dịch đó không tồn tại.

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` trả về tên class đã cho với namespace của class bị xóa:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

Hàm `e` chạy hàm` htmlspecialchars` của PHP với tùy chọn `double_encode` được set mặc định thành `true`:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {#collection-method}

Hàm `preg_replace_array` sẽ thay thế một pattern vào trong một chuỗi sequentially bằng cách sử dụng một mảng:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-after"></a>
#### `Str::after()` {#collection-method}

Hàm `Str::after` trả về mọi thứ đứng sau giá trị đã cho có trong một chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::after('This is my name', 'This is');

    // ' my name'

<a name="method-str-before"></a>
#### `Str::before()` {#collection-method}

Hàm `Str::before` sẽ trả về mọi thứ đứng trước giá trị đã cho có trong một chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::before('This is my name', 'my name');

    // 'This is '

<a name="method-camel-case"></a>
#### `Str::camel()` {#collection-method}

Hàm `Str::camel` chuyển đổi chuỗi đã cho thành `camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::camel('foo_bar');

    // fooBar

<a name="method-str-contains"></a>
#### `Str::contains()` {#collection-method}

Hàm `Str::contains` xác định xem chuỗi đã cho có chứa giá trị đã cho hay không (phân biệt chữ hoa chữ thường):

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', 'my');

    // true

Bạn cũng có thể truyền vào một mảng các giá trị để xác định xem chuỗi đã cho có chứa bất kỳ giá trị nào trong mảng không:

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-contains-all"></a>
#### `Str::containsAll()` {#collection-method}

Phương thức `Str::containsAll` sẽ xác định xem string đã cho có chứa tất cả các giá trị có trong mảng hay không:

    use Illuminate\Support\Str;

    $containsAll = Str::containsAll('This is my name', ['my', 'name']);

    // true

<a name="method-ends-with"></a>
#### `Str::endsWith()` {#collection-method}

Hàm `Str::endsWith` sẽ kiểm tra chuỗi đã cho có kết thúc bằng giá trị đã cho hay không:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', 'name');

    // true

<a name="method-str-finish"></a>
#### `Str::finish()` {#collection-method}

Hàm `Str::finish` sẽ thêm một instance của giá trị đã cho vào một chuỗi nếu nó chưa kết thúc bằng giá trị đó:

    use Illuminate\Support\Str;

    $adjusted = Str::finish('this/string', '/');

    // this/string/

    $adjusted = Str::finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `Str::is()` {#collection-method}

Hàm `Str::is` sẽ xác định xem một chuỗi đã cho có khớp với pattern đã cho hay không. Dấu hoa thị có thể được sử dụng để làm ký tự đại diện:

    use Illuminate\Support\Str;

    $matches = Str::is('foo*', 'foobar');

    // true

    $matches = Str::is('baz*', 'foobar');

    // false

<a name="method-kebab-case"></a>
#### `Str::kebab()` {#collection-method}

Hàm `Str::kebab` chuyển đổi chuỗi đã cho thành `kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::kebab('fooBar');

    // foo-bar

<a name="method-str-limit"></a>
#### `Str::limit()` {#collection-method}

Hàm `Str::limit` sẽ cắt ngắn chuỗi đã cho ở độ dài nhất định:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

Bạn cũng có thể truyền một tham số thứ ba để thay đổi chuỗi sẽ được nối vào cuối chuỗi:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` {#collection-method}

Phương thức `Str::orderedUuid` sẽ tạo một UUID "timestamp first" có thể được lưu trữ tốt trong một cột được index trong cơ sở dữ liệu:

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-plural"></a>
#### `Str::plural()` {#collection-method}

Hàm `Str::plural` sẽ chuyển đổi một chuỗi thành dạng số nhiều của nó. Chức năng này hiện tại chỉ hỗ trợ ngôn ngữ tiếng Anh:

    use Illuminate\Support\Str;

    $plural = Str::plural('car');

    // cars

    $plural = Str::plural('child');

    // children

Bạn có thể cung cấp một số nguyên dưới dạng tham số thứ hai cho hàm để lấy dạng số ít hoặc số nhiều của chuỗi:

    use Illuminate\Support\Str;

    $plural = Str::plural('child', 2);

    // children

    $plural = Str::plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `Str::random()` {#collection-method}

Hàm `Str::random` sẽ tạo ra một chuỗi ngẫu nhiên có độ dài được chỉ định. Hàm này sử dụng hàm `random_bytes` của PHP:

    use Illuminate\Support\Str;

    $random = Str::random(40);

<a name="method-str-replace-array"></a>
#### `Str::replaceArray()` {#collection-method}

Hàm `Str::replaceArray` sẽ thay thế một giá trị đã cho vào trong một chuỗi sequentially bằng cách sử dụng một mảng:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `Str::replaceFirst()` {#collection-method}

Hàm `Str::replaceFirst` sẽ thay thế giá trị đầu tiên có trong chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `Str::replaceLast()` {#collection-method}

Hàm `Str::replaceLast` sẽ thay thế giá trị cuối cùng có trong chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>
#### `Str::singular()` {#collection-method}

Hàm `Str::singular` sẽ chuyển đổi một chuỗi thành dạng số ít của nó. Chức năng này hiện tại chỉ hỗ trợ ngôn ngữ tiếng Anh:

    use Illuminate\Support\Str;

    $singular = Str::singular('cars');

    // car

    $singular = Str::singular('children');

    // child

<a name="method-str-slug"></a>
#### `Str::slug()` {#collection-method}

Hàm `Str::slug` sẽ tạo ra một URL "slug" từ chuỗi đã cho:

    use Illuminate\Support\Str;

    $slug = Str::slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-snake-case"></a>
#### `Str::snake()` {#collection-method}

Hàm `Str::snake` sẽ chuyển đổi chuỗi đã cho thành `Str::snake`:

    use Illuminate\Support\Str;

    $converted = Str::snake('fooBar');

    // foo_bar

<a name="method-str-start"></a>
#### `Str::start()` {#collection-method}

Hàm `Str::start` sẽ thêm một instance của giá trị đã cho vào một chuỗi nếu nó chưa bắt đầu bằng giá trị đó:

    use Illuminate\Support\Str;

    $adjusted = Str::start('this/string', '/');

    // /this/string

    $adjusted = Str::start('/this/string', '/');

    // /this/string

<a name="method-starts-with"></a>
#### `Str::startsWith()` {#collection-method}

Hàm `started_with` sẽ kiểm tra chuỗi đã cho có bắt đầu bằng giá trị đã cho hay không:

    use Illuminate\Support\Str;

    $result = Str::startsWith('This is my name', 'This');

    // true

<a name="method-studly-case"></a>
#### `Str::studly()` {#collection-method}

Hàm `Str::studly` chuyển đổi chuỗi đã cho thành` StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::studly('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `Str::title()` {#collection-method}

Hàm `Str::title` chuyển đổi chuỗi đã cho thành` Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-uuid"></a>
#### `Str::uuid()` {#collection-method}

Phương thức `Str::uuid` sẽ tạo ra một UUID (phiên bản 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="method-str-words"></a>
#### `Str::words()` {#collection-method}

Phương thức `Str::words` sẽ giới hạn số lượng từ có trong một chuỗi:

    use Illuminate\Support\Str;

    return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

    // Perfectly balanced, as >>>

<a name="method-trans"></a>
#### `trans()` {#collection-method}

Hàm `trans` sẽ dịch các key cần dịch bằng cách sử dụng [localization files](/docs/{{version}}/localization) của bạn:

    echo trans('messages.welcome');

Nếu key cần dịch mà không tồn tại, hàm `trans` sẽ trả về key đó. Vì vậy, nếu sử dụng ví dụ trên, hàm `trans` sẽ trả về `message.welcome` nếu key cần dịch không tồn tại.

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

Hàm `trans_choice` sẽ dịch các key cần dịch đã cho với một biến số nhiều:

    echo trans_choice('messages.notifications', $unreadCount);

Nếu key cần dịch mà không tồn tại, hàm `trans_choice` sẽ trả về key đó. Vì vậy, nếu sử dụng ví dụ trên, hàm `trans_choice` sẽ trả về `messages.notifications` nếu key cần dịch không tồn tại.

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

Hàm `action` sẽ tạo ra một URL cho một action của controller đã cho. Bạn không cần phải truyền namespace của controller. Thay vào đó, hãy truyền tên class của controller liên kết đến namespace `App\Http\Controllers`:

    $url = action('HomeController@index');

    $url = action([HomeController::class, 'index']);

Nếu phương thức chấp nhận tham số cho route, bạn có thể truyền chúng làm tham số thứ hai cho phương thức:

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

Hàm `asset` sẽ tạo URL cho một asset bằng cách sử dụng scheme hiện tại của request (HTTP hoặc HTTPS):

    $url = asset('img/photo.jpg');

Bạn có thể cấu hình URL host cho asset bằng cách set biến `ASSET_URL` trong file `.env` của bạn. Điều này có thể hữu ích nếu bạn đang lưu trữ các asset của bạn trong một dịch vụ bên ngoài như Amazon S3:

    // ASSET_URL=http://example.com/assets

    $url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg

<a name="method-route"></a>
#### `route()` {#collection-method}

Hàm `route` sẽ tạo một URL cho route đã được đặt tên:

    $url = route('routeName');

Nếu route có chấp nhận tham số, bạn có thể truyền chúng làm tham số thứ hai cho phương thức:

    $url = route('routeName', ['id' => 1]);

Mặc định, hàm `route` sẽ tạo ra một URL tuyệt đối. Nếu bạn muốn tạo một URL tương đối, bạn có thể truyền `false` làm tham số thứ ba:

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

Hàm `secure_asset` sẽ tạo URL cho một asset bằng HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

Hàm `secure_url` tạo URL HTTPS cho đường dẫn đã cho:

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

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
#### `abort()` {#collection-method}

Hàm `abort` sẽ đưa ra một [exception HTTP](/docs/{{version}}/errors#http-exceptions) được tạo bởi [exception handler](/docs/{{version}}/errors#the-exception-handler):

    abort(403);

Bạn cũng có thể cung cấp response text và response header tùy biến cho exception:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

Hàm `abort_if` sẽ đưa ra một exception HTTP nếu một biểu thức boolean đã cho là `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Giống như phương thức `abort`, bạn cũng có thể cung cấp response text cho exception làm tham số thứ ba và một mảng các response header tùy biến làm tham số thứ tư.

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

Hàm `abort_unless` sẽ đưa ra một exception HTTP nếu một biểu thức boolean đã cho là `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Giống như phương thức `abort`, bạn cũng có thể cung cấp response text cho exception làm tham số thứ ba và một mảng các response header tùy biến làm tham số thứ tư.

<a name="method-app"></a>
#### `app()` {#collection-method}

Hàm `app` trả về instance [service container](/docs/{{version}}/container):

    $container = app();

Bạn có thể truyền một tên class hoặc một tên interface để resolve nó từ container:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {#collection-method}

Hàm `auth` sẽ trả về một instance [authenticator](/docs/{{version}}/authentication). Bạn có thể sử dụng nó thay vì dùng facade `Auth` cho thuận tiện:

    $user = auth()->user();

Nếu cần, bạn có thể khai báo loại instance guard mà bạn muốn truy cập:

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

Hàm `back` sẽ tạo ra một [response HTTP chuyển hướng](/docs/{{version}}/responses#redirects) đến vị trí trước đó của người dùng:

    return back($status = 302, $headers = [], $fallback = false);

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

Hàm `bcrypt` sẽ [hashes](/docs/{{version}}/hashing) giá trị đã cho bằng Bcrypt. Bạn có thể sử dụng nó như là một thay thế cho facade `Hash`:

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>
#### `blank()` {#collection-method}

Hàm `blank` sẽ trả về giá trị đã cho là "blank" hay không:

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
#### `broadcast()` {#collection-method}

Hàm `broadcast` sẽ [broadcasts](/docs/{{version}}/broadcasting) một [event](/docs/{{version}}/events) cho listener của nó:

    broadcast(new UserRegistered($user));

<a name="method-cache"></a>
#### `cache()` {#collection-method}

Hàm `cache` có thể được sử dụng để lấy các giá trị từ [cache](/docs/{{version}}/cache). Nếu key đã cho không tồn tại trong cache, giá trị mặc định sẽ được trả về:

    $value = cache('key');

    $value = cache('key', 'default');

Bạn có thể thêm các item vào cache bằng cách truyền một mảng các cặp key / giá trị cho hàm. Bạn cũng nên truyền thêm số giây hoặc thời gian mà giá trị được lưu trong bộ nhớ cache sẽ được coi là hợp lệ:

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {#collection-method}

Hàm `class_uses_recursive` sẽ trả về tất cả các trait được sử dụng bởi một class, bao gồm cả các trait được sử dụng bởi tất cả các class cha của nó:

    $traits = class_uses_recursive(App\User::class);

<a name="method-collect"></a>
#### `collect()` {#collection-method}

Hàm `collect` tạo ra một instance [collection](/docs/{{version}}/collections) từ giá trị đã cho:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

Hàm `config` sẽ lấy giá trị của biến [configuration](/docs/{{version}}/configuration). Các giá trị cấu hình có thể được truy cập bằng cú pháp "dot", bao gồm tên của file và option bạn muốn truy cập. Giá trị mặc định có thể được khai báo và được trả về nếu tùy chọn cấu hình không tồn tại:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Bạn có thể set các biến cấu hình trong thời gian chạy bằng cách truyền một mảng các cặp key / giá trị:

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()` {#collection-method}

Hàm `cookie` tạo một instance [cookie](/docs/{{version}}/requests#cookies) mới:

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

Hàm `csrf_field` sẽ tạo ra một thẻ input `hidden` HTML chứa giá trị của CSRF token. Ví dụ: sử dụng [Blade syntax](/docs/{{version}}/blade):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

Hàm `csrf_token` sẽ lấy ra giá trị của CSRF token hiện tại:

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

Hàm `dd` sẽ dump các biến đã cho và dừng thực thi lệnh:

    dd($value);

    dd($value1, $value2, $value3, ...);

Nếu bạn không muốn dừng việc thực thi lệnh của bạn, hãy sử dụng hàm [`dump`](#method-dump) để thay thế.

<a name="method-decrypt"></a>
#### `decrypt()` {#collection-method}

Hàm `decrypt` sẽ giải mã giá trị đã cho bằng cách sử dụng [encrypter](/docs/{{version}}/encryption) của Laravel:

    $decrypted = decrypt($encrypted_value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

Hàm `dispatch` sẽ tạo [job](/docs/{{version}}/queues#creating-jobs) vào Laravel [job queue](/docs/{{version}}/queues):

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-now"></a>
#### `dispatch_now()` {#collection-method}

Hàm `dispatch_now` sẽ chạy ngay lập tức [job](/docs/{{version}}/queues#creating-jobs) và trả về giá trị từ phương thức `handle` của nó:

    $result = dispatch_now(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` {#collection-method}

Hàm `dump` sẽ dump các biến đã cho:

    dump($value);

    dump($value1, $value2, $value3, ...);

Nếu bạn muốn dừng thực thi lệnh sau khi dump các biến, hãy sử dụng hàm [`dd`](#method-dd) để thay thế.

> {tip} Bạn có thể sử dụng lệnh `dump-server` của Artisan để chặn tất cả các lệnh `dump` và hiển thị chúng trong console thay vì trình duyệt của bạn.

<a name="method-encrypt"></a>
#### `encrypt()` {#collection-method}

Hàm `encrypt` sẽ mã hóa giá trị đã cho bằng cách sử dụng [encrypter](/docs/{{version}}/encryption) của Laravel:

    $encrypted = encrypt($unencrypted_value);

<a name="method-env"></a>
#### `env()` {#collection-method}

Hàm `env` sẽ lấy ra giá trị của [environment variable](/docs/{{version}}/configuration#environment-configuration) hoặc trả về giá trị mặc định:

    $env = env('APP_ENV');

    // Returns 'production' if APP_ENV is not set...
    $env = env('APP_ENV', 'production');

> {note} Nếu bạn chạy lệnh `config:cache` trong quá trình deploy của bạn, bạn nên chắc chắn rằng bạn chỉ gọi hàm `env` từ các file cấu hình của bạn. Khi các option cấu hình đã được lưu vào cached, file `.env` sẽ không được load và tất cả các lệnh gọi đến hàm `env` sẽ trả về `null`.

<a name="method-event"></a>
#### `event()` {#collection-method}

Hàm `event` sẽ dispatch [event](/docs/{{version}}/events) đến listener:

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

Hàm `factory` sẽ tạo một model factory builder cho một class, tên và số lượng nhất định. Nó có thể được sử dụng trong khi [testing](/docs/{{version}}/database-testing#writing-factories) hoặc [seeding](/docs/{{version}}/seeding#using-model-factories):

    $user = factory(App\User::class)->make();

<a name="method-filled"></a>
#### `filled()` {#collection-method}

Hàm `filled` sẽ trả về giá trị đã cho không là "blank" hay không:

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
#### `info()` {#collection-method}

Hàm `info` sẽ ghi thông tin vào [log](/docs/{{version}}/logging):

    info('Some helpful information!');

Một mảng dữ liệu theo ngữ cảnh cũng có thể được truyền cho hàm:

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

Hàm `logger` có thể được sử dụng để viết một thông báo ở mức `debug` vào [log](/docs/{{version}}/logging):

    logger('Debug message');

Một mảng dữ liệu theo ngữ cảnh cũng có thể được truyền cho hàm:

    logger('User has logged in.', ['id' => $user->id]);

Một instance [logger](/docs/{{version}}/errors#logging) sẽ được trả về nếu không có giá trị nào được truyền vào cho hàm:

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

Hàm `method_field` tạo ra thẻ input `hidden` HTML chứa giá trị HTTP action của form. Ví dụ: sử dụng [Blade syntax](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()` {#collection-method}

Hàm `now` sẽ tạo ra một instance `Illuminate\Support\Carbon` mới cho thời điểm hiện tại:

    $now = now();

<a name="method-old"></a>
#### `old()` {#collection-method}

Hàm `old` sẽ [lấy ra](/docs/{{version}}/requests#retrieving-input) một giá trị [old input](/docs/{{version}}/requests#old-input) được flash trong session :

    $value = old('value');

    $value = old('value', 'default');

<a name="method-optional"></a>
#### `optional()` {#collection-method}

Hàm `optional` nhận vào bất kỳ tham số nào và cho phép bạn truy cập vào các thuộc tính hoặc các phương thức trên đối tượng đó. Nếu đối tượng đã cho là `null`, thì các thuộc tính hoặc các phương thức đó sẽ trả về `null` thay vì gây ra lỗi:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

Phương thức `optional` cũng chấp nhận một Closure làm tham số thứ hai của nó. Closure sẽ được gọi nếu giá trị tham số đầu tiên không phải là một giá trị null:

    return optional(User::find($id), function ($user) {
        return new DummyUser;
    });

<a name="method-policy"></a>
#### `policy()` {#collection-method}

Phương thức `policy` sẽ lấy ra một instance [policy](/docs/{{version}}/authorization#creating-policies) cho một class nhất định:

    $policy = policy(App\User::class);

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

Hàm `redirect` sẽ trả về một [response HTTP chuyển hướng](/docs/{{version}}/responses#redirects) hoặc trả về instance chuyển hướng nếu không có tham số được truyền vào:

    return redirect($to = null, $status = 302, $headers = [], $secure = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

Hàm `report` sẽ report một exception bằng cách sử dụng phương thức `report` của [exception handler](/docs/{{version}}/errors#the-exception-handler) của bạn:

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

Hàm `request` trả về instance [request](/docs/{{version}}/requests) hiện tại hoặc lấy ra một input item:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` {#collection-method}

Hàm `rescue` sẽ thực thi Closure đã cho và catch bất kỳ exception nào xảy ra trong quá trình thực thi. Tất cả các exception bị catch sẽ được gửi đến phương thức `report` của [exception handler](/docs/{{version}}/errors#the-exception-handler) của bạn; tuy nhiên, request sẽ tiếp tục xử lý:

    return rescue(function () {
        return $this->method();
    });

Bạn cũng có thể truyền tham số thứ hai cho hàm `rescue`. Tham số này sẽ là giá trị "default" cần được trả về nếu có exception xảy ra trong khi thực hiện Closure:

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-resolve"></a>
#### `resolve()` {#collection-method}

Hàm `resolve` sẽ resolve một tên class hoặc một interface đã cho thành một instance của nó bằng cách sử dụng [service container](/docs/{{version}}/container):

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` {#collection-method}

Hàm `response` tạo ra một instance [response](/docs/{{version}}/responses) hoặc lấy ra một instance của response factory:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

Hàm `retry` sẽ thử thực hiện callback đã cho, cho đến khi đạt được ngưỡng thử tối đa nào đó. Nếu callback không đưa ra exception, chính giá trị trả về của nó sẽ được trả về. Nếu callback đưa ra một exception, nó sẽ tự động được thử lại. Nếu vượt quá số lần thử tối đa, exception sẽ bị đưa ra:

    return retry(5, function () {
        // Attempt 5 times while resting 100ms in between attempts...
    }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

Hàm `session` có thể được sử dụng để lấy hoặc set các giá trị [session](/docs/{{version}}/session) values:

    $value = session('key');

Bạn có thể set giá trị bằng cách truyền một mảng các cặp key / giá trị cho hàm:

    session(['chairs' => 7, 'instruments' => 3]);

Session store sẽ được trả về nếu không có giá trị nào được truyền cho hàm:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {#collection-method}

Hàm `tap` sẽ nhận vào hai tham số: một là `$value` và một Closure. `$value` sẽ được truyền đến phần Closure và sau đó được trả về bởi hàm `tap`. Giá trị trả về của Closure sẽ không liên quan:

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

Nếu không có Closure nào được truyền đến hàm `tap`, bạn có thể gọi bất kỳ phương thức nào trên `$value` đã cho. Giá trị trả về của phương thức bạn gọi sẽ luôn là `$value`, bất kể phương thức đó thực sự trả về định nghĩa gì đi chăng nữa. Ví dụ, phương thức `update` Eloquent thường trả về một số nguyên. Tuy nhiên, chúng ta có thể buộc phương thức này trả về chính model đó bằng cách gọi phương thức `update` thông qua hàm `tap`:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

Để thêm một phương thức `tap` vào một class, bạn có thể thêm trait `Illuminate\Support\Traits\Tappable` vào class. Phương thức `tap` của trait này sẽ chấp nhận một Closure làm tham số duy nhất của nó. Chính instance đối tượng sẽ được truyền đến Closure và sau đó được trả về bởi phương thức `tap`:

    return $user->tap(function ($user) {
        //
    });

<a name="method-throw-if"></a>
#### `throw_if()` {#collection-method}

Hàm `throw_if` sẽ đưa ra exception đã cho nếu một biểu thức boolean đã cho là `true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` {#collection-method}

Hàm `throw_unless` sẽ đưa ra exception đã cho nếu một biểu thức boolean đã cho là `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-today"></a>
#### `today()` {#collection-method}

Hàm `today` sẽ tạo ra một instance `Illuminate\Support\Carbon` mới cho ngày hiện tại:

    $today = today();

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {#collection-method}

Hàm `trait_uses_recursive` trả về tất cả các trait được sử dụng bởi một trait:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {#collection-method}

Hàm `transform` sẽ thực thi một `Closure` trên một giá trị đã cho nếu giá trị không [blank](#method-blank) và trả về kết quả của một `Closure`:

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

Một giá trị mặc định hoặc một `Closure` cũng có thể được truyền làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu giá trị đã cho là blank:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {#collection-method}

Hàm `validator` sẽ tạo ra một instance [validator](/docs/{{version}}/validation) mới với các tham số đã cho. Bạn có thể sử dụng nó thay vì facade `Validator` cho thuận tiện:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {#collection-method}

Hàm `value` sẽ trả về giá trị được cho. Tuy nhiên, nếu bạn truyền một `Closure` cho hàm, thì` Closure` sẽ được thực thi sau đó kết quả của nó sẽ được trả về:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

<a name="method-view"></a>
#### `view()` {#collection-method}

Hàm `view` sẽ lấy ra một instance [view](/docs/{{version}}/views):

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

Hàm `with` sẽ trả về giá trị được cho. Nếu một `Closure` được truyền làm tham số thứ hai cho hàm, thì `Closure` đó sẽ được thực thi và sau đó kết quả của nó sẽ được trả về:

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
