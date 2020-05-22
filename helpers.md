# Helpers

- [Giới thiệu](#introduction)
- [Các phương thức có sẵn](#available-methods)

<a name="introduction"></a>
## Giới thiệu

Laravel chứa một loạt các hàm PHP global "helper". Nhiều trong số các hàm này được sử dụng bởi chính framework; tuy nhiên, bạn có thể thoải mái sử dụng chúng trong các application của bạn nếu bạn thấy chúng thuận tiện.

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

[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_last](#method-array-last)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_prepend](#method-array-prepend)
[array_pull](#method-array-pull)
[array_random](#method-array-random)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[array_wrap](#method-array-wrap)
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
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[kebab_case](#method-kebab-case)
[preg_replace_array](#method-preg-replace-array)
[snake_case](#method-snake-case)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_limit](#method-str-limit)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_replace_array](#method-str-replace-array)
[str_replace_first](#method-str-replace-first)
[str_replace_last](#method-str-replace-last)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[str_start](#method-str-start)
[studly_case](#method-studly-case)
[title_case](#method-title-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### URLs

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
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
[today](#method-today)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
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
#### `array_add()` {#collection-method .first-collection-method}

Hàm `array_add` sẽ thêm một cặp key / giá trị đã cho vào một mảng nếu key đã cho không tồn tại trong mảng:

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

Hàm `array_collapse` sẽ thu gọn một mảng các mảng con thành một mảng duy nhất:

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

Hàm `array_divide` trả về hai mảng, một mảng chứa các key và cái còn lại chứa các giá trị của mảng đã cho:

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

Hàm `array_dot` sẽ làm ngang hàng một mảng nhiều chiều thành một mảng đơn cấp sử dụng ký hiệu "dot" để biểu thị độ sâu:

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = array_dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

Hàm `array_except` loại bỏ các cặp key / giá trị đã cho ra khỏi một mảng:

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

Hàm `array_first` trả về phần tử đầu tiên của mảng pass qua một số điều kiện đã cho:

    $array = [100, 200, 300];

    $first = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

Một giá trị mặc định cũng có thể được pass làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu không có giá trị nào pass qua điều kiện:

    $first = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

Hàm `array_flatten` làm ngang hàng một mảng nhiều chiều thành một mảng đơn cấp:

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

Hàm `array_forget` xóa một cặp key / giá trị đã cho ra khỏi một mảng bị lồng vào nhau bằng cách sử dụng ký hiệu "dot":

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

Hàm `array_get` lấy một giá trị từ một mảng bị lồng vào nhau bằng cách sử dụng ký hiệu "dot":

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = array_get($array, 'products.desk.price');

    // 100

Hàm `array_get` cũng chấp nhận một giá trị mặc định, sẽ được trả về nếu không tìm thấy key:

    $discount = array_get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

Hàm `array_has` sẽ kiểm tra xem một item hoặc các item đã cho có tồn tại trong một mảng bằng cách sử dụng ký hiệu "dot" hay không:

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = array_has($array, 'product.name');

    // true

    $contains = array_has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

Hàm `array_last` trả về phần tử cuối cùng của mảng pass qua một số điều kiện đã cho:

    $array = [100, 200, 300, 110];

    $last = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

Một giá trị mặc định có thể được pass làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu không có giá trị nào pass qua điều kiện:

    $last = array_last($array, $callback, $default);

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

Hàm `array_only` chỉ trả về các cặp key / giá trị được chỉ định từ mảng đã cho:

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

Hàm `array_pluck` lấy tất cả các giá trị cho một key đã cho từ một mảng:

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

Bạn cũng có thể khai báo key cho danh sách kết quả trả về:

    $names = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

Hàm `array_prepend` sẽ thêm một item lên đầu của một mảng:

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

Nếu cần, bạn có thể khai báo key cho giá trị đó:

    $array = ['price' => 100];

    $array = array_prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

Hàm `array_pull` trả về và xóa một cặp key / giá trị ra khỏi một mảng:

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Một giá trị mặc định có thể được pass làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu key không tồn tại:

    $value = array_pull($array, $key, $default);

<a name="method-array-random"></a>
#### `array_random()` {#collection-method}

Hàm `array_random` sẽ trả về một giá trị ngẫu nhiên từ một mảng:

    $array = [1, 2, 3, 4, 5];

    $random = array_random($array);

    // 4 - (retrieved randomly)

Bạn cũng có thể chỉ định số lượng item sẽ trả về làm tham số thứ hai tùy chọn. Lưu ý rằng việc cung cấp tham số này sẽ trả về một mảng, ngay cả khi chỉ có một item mong muốn:

    $items = array_random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

Hàm `array_set` sẽ set một giá trị trong một mảng bị lông vào nhau bằng cách sử dụng ký hiệu "dot":

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

Hàm `array_sort` sẽ sắp xếp một mảng theo các giá trị của nó:

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = array_sort($array);

    // ['Chair', 'Desk', 'Table']

Bạn cũng có thể sắp xếp mảng theo kết quả của Closure đã cho:

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(array_sort($array, function ($value) {
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
#### `array_sort_recursive()` {#collection-method}

Hàm `array_sort_recursive` sẽ sắp xếp đệ quy một mảng bằng cách sử dụng hàm `sort`:

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
    ];

    $sorted = array_sort_recursive($array);

    /*
        [
            ['Li', 'Roman', 'Taylor'],
            ['JavaScript', 'PHP', 'Ruby'],
        ]
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

Hàm `array_where` sẽ lọc một mảng bằng cách sử dụng Closure:

    $array = [100, '200', 300, '400', 500];

    $filtered = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-array-wrap"></a>
#### `array_wrap()` {#collection-method}

Hàm `array_wrap` sẽ wrap giá trị đã cho vào trong một mảng. Nếu giá trị đã cho là một mảng, nó sẽ không bị thay đổi:

    $string = 'Laravel';

    $array = array_wrap($string);

    // ['Laravel']

Nếu giá trị đã cho là null, một mảng trống sẽ được trả về:

    $nothing = null;

    $array = array_wrap($nothing);

    // []

<a name="method-data-fill"></a>
#### `data_fill()` {#collection-method}

Hàm `data_fill` đặt một giá trị bị thiếu trong một mảng hoặc đối tượng lồng nhau bằng cách sử dụng ký hiệu "dot":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

Hàm này cũng chấp nhận dấu hoa thị dưới dạng như ký tự đại diện và sẽ điền vào mục tiêu tương ứng:

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

Hàm `data_get` lấy một giá trị từ một mảng hoặc đối tượng lồng nhau bằng cách sử dụng ký hiệu "dot":

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

Hàm `data_get` cũng chấp nhận một giá trị mặc định, sẽ được trả về nếu không tìm thấy key được chỉ định:

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

<a name="method-data-set"></a>
#### `data_set()` {#collection-method}

Hàm `data_set` sẽ set một giá trị trong một mảng hoặc đối tượng lồng nhau bằng cách sử dụng ký hiệu "dot":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Hàm này cũng chấp nhận ký tự đại diện và sẽ set giá trị cho mục tiêu tương ứng:

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

Mặc định, bất kỳ giá trị hiện có sẽ bị ghi đè. Nếu bạn chỉ muốn set một giá trị nếu nó không tồn tại, bạn có thể pass `false` làm tham số thứ ba:

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

Hàm `mix` trả về đường dẫn đến [file Mix được version](/docs/{{version}}/mix):

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

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

Hàm `storage_path` trả về đường dẫn đến thư mục` storage`. Bạn cũng có thể sử dụng hàm `storage_path` để tạo đường dẫn đến một file đã cho trong thư mục storage:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Strings

<a name="method-__"></a>
#### `__()` {#collection-method}

Hàm `__` sẽ translate chuỗi cần được dịch hoặc key cần được dịch đã cho bằng cách sử dụng [localization files](/docs/{{version}}/localization) của bạn:

    echo __('Welcome to our application');

    echo __('messages.welcome');

Nếu chuỗi hoặc key cần được dịch được khai báo nhưng không tồn tại, hàm `__` sẽ trả về giá trị đưa vào. Vì vậy, nếu sử dụng ví dụ mẫu trên, hàm `__` sẽ trả về `messages.welcome` nếu key cần được dịch đó không tồn tại.

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

Hàm `camel_case` chuyển đổi chuỗi đã cho thành `camelCase`:

    $converted = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` trả về tên class của class đã cho với namespace của class bị xóa:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

Hàm `e` chạy hàm` htmlspecialchars` của PHP với tùy chọn `double_encode` được set thành` false`:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

Hàm `ends_with` xác định nếu chuỗi đã cho có kết thúc bằng giá trị đã cho hay không:

    $result = ends_with('This is my name', 'name');

    // true

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

Hàm `kebab_case` chuyển đổi chuỗi đã cho thành` kebab-case`:

    $converted = kebab_case('fooBar');

    // foo-bar

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {#collection-method}

Hàm `preg_replace_array` sẽ thay thế một pattern vào trong một chuỗi sequentially bằng cách sử dụng một mảng:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

Hàm `Snake_snake_casecase` chuyển đổi chuỗi đã cho thành` snake_case`:

    $converted = snake_case('fooBar');

    // foo_bar

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

Hàm `started_with` sẽ xác định chuỗi đã cho có bắt đầu bằng giá trị đã cho hay không:

    $result = starts_with('This is my name', 'This');

    // true

<a name="method-str-after"></a>
#### `str_after()` {#collection-method}

Hàm `str_after` trả về mọi thứ đứng sau giá trị đã cho có trong một chuỗi:

    $slice = str_after('This is my name', 'This is');

    // ' my name'

<a name="method-str-before"></a>
#### `str_before()` {#collection-method}

Hàm `str_before` sẽ trả về mọi thứ đứng trước giá trị đã cho có trong một chuỗi:

    $slice = str_before('This is my name', 'my name');

    // 'This is '

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

Hàm `str_contains` xác định xem chuỗi đã cho có chứa giá trị đã cho hay không (phân biệt chữ hoa chữ thường):

    $contains = str_contains('This is my name', 'my');

    // true

Bạn cũng có thể pass một mảng các giá trị để xác định xem chuỗi đã cho có chứa bất kỳ giá trị nào trong mảng không:

    $contains = str_contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

Hàm `str_finish` thêm một instance của giá trị đã cho vào một chuỗi nếu nó chưa kết thúc bằng giá trị đó:

    $adjusted = str_finish('this/string', '/');

    // this/string/

    $adjusted = str_finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

Hàm `str_is` sẽ xác định xem một chuỗi đã cho có khớp với pattern đã cho hay không. Dấu hoa thị có thể được sử dụng để làm ký tự đại diện:

    $matches = str_is('foo*', 'foobar');

    // true

    $matches = str_is('baz*', 'foobar');

    // false

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

Hàm `str_limit` sẽ cắt ngắn chuỗi đã cho ở độ dài nhất định:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

Bạn cũng có thể pass một tham số thứ ba để thay đổi chuỗi sẽ được nối vào cuối chuỗi:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

Hàm `str_plural` chuyển đổi một chuỗi thành dạng số nhiều của nó. Chức năng này hiện chỉ hỗ trợ ngôn ngữ tiếng Anh:

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

Bạn có thể cung cấp một số nguyên dưới dạng tham số thứ hai cho hàm để lấy dạng số ít hoặc số nhiều của chuỗi:

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

Hàm `str_random` sẽ tạo ra một chuỗi ngẫu nhiên có độ dài được chỉ định. Hàm này sử dụng hàm `random_bytes` của PHP:

    $random = str_random(40);

<a name="method-str-replace-array"></a>
#### `str_replace_array()` {#collection-method}

Hàm `str_replace_array` sẽ thay thế một giá trị đã cho vào trong một chuỗi sequentially bằng cách sử dụng một mảng:

    $string = 'The event will take place between ? and ?';

    $replaced = str_replace_array('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `str_replace_first()` {#collection-method}

Hàm `str_replace_first` sẽ thay thế giá trị đầu tiên có trong chuỗi:

    $replaced = str_replace_first('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `str_replace_last()` {#collection-method}

Hàm `str_replace_last` sẽ thay thế giá trị cuối cùng có trong chuỗi:

    $replaced = str_replace_last('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

Hàm `str_singular` sẽ chuyển đổi một chuỗi thành dạng số ít của nó. Chức năng này hiện chỉ hỗ trợ ngôn ngữ tiếng Anh:

    $singular = str_singular('cars');

    // car

    $singular = str_singular('children');

    // child

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

Hàm `str_slug` sẽ tạo ra một URL "slug" từ chuỗi đã cho:

    $slug = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-str-start"></a>
#### `str_start()` {#collection-method}

Hàm `str_start` sẽ thêm một instance của giá trị đã cho vào một chuỗi nếu nó chưa bắt đầu bằng giá trị đó:

    $adjusted = str_start('this/string', '/');

    // /this/string

    $adjusted = str_start('/this/string/', '/');

    // /this/string

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

Hàm `studly_case` chuyển đổi chuỗi đã cho thành` StudlyCase`:

    $converted = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

Hàm `title_case` chuyển đổi chuỗi đã cho thành` Title Case`:

    $converted = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` {#collection-method}

Hàm `trans` sẽ translate các key cần dịch bằng cách sử dụng [localization files](/docs/{{version}}/localization) của bạn:

    echo trans('messages.welcome');

Nếu key cần dịch mà không tồn tại, hàm `trans` sẽ trả về key đó. Vì vậy, nếu sử dụng ví dụ trên, hàm `trans` sẽ trả về `message.welcome` nếu key cần dịch không tồn tại.

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

Hàm `trans_choice` sẽ translate các key cần dịch đã cho với một biến số nhiều:

    echo trans_choice('messages.notifications', $unreadCount);

Nếu key cần dịch mà không tồn tại, hàm `trans_choice` sẽ trả về key đó. Vì vậy, nếu sử dụng ví dụ trên, hàm `trans_choice` sẽ trả về `messages.notifications` nếu key cần dịch không tồn tại.

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

Hàm `action` sẽ tạo ra một URL cho hành động của controller đã cho. Bạn không cần phải pass namespace của controller. Thay vào đó, hãy pass tên class của controller liên kết đến namespace `App\Http\Controllers`:

    $url = action('HomeController@index');

Nếu phương thức nhận tham số cho route, bạn có thể pass chúng làm tham số thứ hai cho phương thức:

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

Hàm `property` tạo URL cho một asset bằng cách sử dụng scheme hiện tại của request (HTTP hoặc HTTPS):

    $url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

Hàm `secure_asset` sẽ tạo URL cho một asset bằng HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-route"></a>
#### `route()` {#collection-method}

Hàm `route` sẽ tạo một URL cho route đã được đặt tên:

    $url = route('routeName');

Nếu route có nhận tham số, bạn có thể pass chúng làm tham số thứ hai cho phương thức:

    $url = route('routeName', ['id' => 1]);

Mặc định, hàm `route` tạo ra một URL tuyệt đối. Nếu bạn muốn tạo một URL tương đối, bạn có thể pass `false` làm tham số thứ ba:

    $url = route('routeName', ['id' => 1], false);

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

Bạn cũng có thể cung cấp response text và response header tùy chỉnh của exception:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

Hàm `abort_if` sẽ đưa ra một exception HTTP nếu một biểu thức boolean đã cho là `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Giống như phương thức `abort`, bạn cũng có thể cung cấp response text của exception làm tham số thứ ba và một mảng các response header tùy chỉnh làm tham số thứ tư.

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

Hàm `abort_unless` sẽ đưa ra một exception HTTP nếu một biểu thức boolean đã cho là `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Giống như phương thức `abort`, bạn cũng có thể cung cấp response text của exception làm tham số thứ ba và một mảng các response header tùy chỉnh làm tham số thứ tư.

<a name="method-app"></a>
#### `app()` {#collection-method}

Hàm `app` trả về instance [service container](/docs/{{version}}/container):

    $container = app();

Bạn có thể pass một tên class hoặc tên interface để resolve nó từ container:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {#collection-method}

Hàm `auth` sẽ trả về một instance [authenticator](/docs/{{version}}/authentication) . Bạn có thể sử dụng nó thay vì dùng facade `Auth` cho thuận tiện:

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

<a name="method-broadcast"></a>
#### `broadcast()` {#collection-method}

Hàm `broadcast` sẽ [broadcasts](/docs/{{version}}/broadcasting) một [event](/docs/{{version}}/events) cho listener của nó:

    broadcast(new UserRegistered($user));

<a name="method-blank"></a>
#### `blank()` {#collection-method}

Hàm `blank` trả về giá trị đã cho là "blank" hay không:

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

<a name="method-cache"></a>
#### `cache()` {#collection-method}

Hàm `cache` có thể được sử dụng để lấy các giá trị từ [cache](/docs/{{version}}/cache). Nếu key đã cho không tồn tại trong cache, giá trị mặc định tùy chọn sẽ được trả về:

    $value = cache('key');

    $value = cache('key', 'default');

Bạn có thể thêm các item vào cache bằng cách pass một mảng các cặp key / giá trị cho hàm. Bạn cũng nên pass thêm số phút hoặc thời lượng mà giá trị được lưu trong bộ nhớ cache sẽ được coi là hợp lệ:

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {#collection-method}

Hàm `class_uses_recursive` sẽ trả về tất cả các trait được sử dụng bởi một class, bao gồm cả các trait được sử dụng bởi bất kỳ class con:

    $traits = class_uses_recursive(App\User::class);

<a name="method-collect"></a>
#### `collect()` {#collection-method}

Hàm `collect` tạo ra một instance [collection](/docs/{{version}}/collections)  từ giá trị đã cho:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

Hàm `config` sẽ lấy giá trị của biến [configuration](/docs/{{version}}/configuration). Các giá trị cấu hình có thể được truy cập bằng cú pháp "dot", bao gồm tên của file và option bạn muốn truy cập. Giá trị mặc định có thể được khai báo và được trả về nếu tùy chọn cấu hình không tồn tại:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Bạn có thể set các biến cấu hình trong thời gian chạy bằng cách chuyển một mảng các cặp key / giá trị:

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

Hàm `csrf_token` lấy ra giá trị của CSRF token hiện tại:

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

Hàm `info` sẽ ghi thông tin vào [log](/docs/{{version}}/errors#logging):

    info('Some helpful information!');

Một mảng dữ liệu theo ngữ cảnh cũng có thể được pass cho hàm:

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

Hàm `logger` có thể được sử dụng để viết một thông báo mức `debug` vào [log](/docs/{{version}}/errors#logging):

    logger('Debug message');

Một mảng dữ liệu theo ngữ cảnh cũng có thể được pass cho hàm:

    logger('User has logged in.', ['id' => $user->id]);

Một instance [logger](/docs/{{version}}/errors#logging) sẽ được trả về nếu không có giá trị nào được pass cho hàm:

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

Hàm `optional` nhận vào bất kỳ tham số nào và cho phép bạn truy cập các thuộc tính hoặc phương thức gọi trên đối tượng đó. Nếu đối tượng đã cho là `null`, các thuộc tính và phương thức sẽ trả về `null` thay vì gây ra lỗi:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

<a name="method-policy"></a>
#### `policy()` {#collection-method}

Phương thức `policy` sẽ lấy ra một instance [policy](/docs/{{version}}/authorization#creating-policies) cho một class nhất định:

    $policy = policy(App\User::class);

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

Hàm `redirect` sẽ trả về một [response HTTP chuyển hướng](/docs/{{version}}/responses#redirects) hoặc trả về instance chuyển hướng nếu không có tham số được pass vào:

    return redirect($to = null, $status = 302, $headers = [], $secure = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

Hàm `report` sẽ report một exception bằng cách sử dụng phương thức `report` của [exception handler](/docs/{{version}}/errors#the-exception-handler) của bạn:

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

Hàm `request` trả về instance [request](/docs/{{version}}/requests) hiện tại hoặc nhận được một input item:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` {#collection-method}

Hàm `rescue` sẽ thực thi Closure đã cho và catch bất kỳ exception nào xảy ra trong quá trình thực thi. Tất cả các exception bị catch sẽ được gửi đến phương thức `report` của [exception handler](/docs/{{version}}/errors#the-exception-handler) của bạn; tuy nhiên, request sẽ tiếp tục xử lý:

    return rescue(function () {
        return $this->method();
    });

Bạn cũng có thể pass tham số thứ hai cho hàm `rescue`. Tham số này sẽ là giá trị "default" cần được trả về nếu có exception xảy ra trong khi thực hiện Closure:

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

Hàm `resolve` sẽ resolve một tên class hoặc interface đã cho thành instance của nó bằng cách sử dụng [service container](/docs/{{version}}/container):

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` {#collection-method}

Hàm `response` tạo ra một instance [response](/docs/{{version}}/responses) hoặc lấy ra một instance của response factory:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

Hàm `retry` sẽ thử thực hiện callback đã cho, cho đến khi đạt được ngưỡng thử tối đa đã cho. Nếu callback không đưa ra exception, giá trị trả về của nó sẽ được trả về. Nếu callback đưa ra một exception, nó sẽ tự động được thử lại. Nếu vượt quá số lần thử tối đa, exception sẽ bị đưa ra:

    return retry(5, function () {
        // Attempt 5 times while resting 100ms in between attempts...
    }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

Hàm `session` có thể được sử dụng để lấy hoặc set các giá trị [session](/docs/{{version}}/session) values:

    $value = session('key');

Bạn có thể set giá trị bằng cách pass một mảng các cặp key / giá trị cho hàm:

    session(['chairs' => 7, 'instruments' => 3]);

Session store sẽ được trả về nếu không có giá trị nào được pass cho hàm:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {#collection-method}

Hàm `tap` sẽ nhận vào hai tham số: một là `$value` và một Closure. `$value` sẽ được pass đến phần Closure và sau đó được trả về bởi hàm `tap`. Giá trị trả về của Closure là không liên quan:

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

Nếu không có Closure nào được pass đến hàm `tap`, bạn có thể gọi bất kỳ phương thức nào trên `$value` đã cho. Giá trị trả về của phương thức bạn gọi sẽ luôn là `$value`, bất kể phương thức đó thực sự trả về định nghĩa của nó là gì. Ví dụ, phương thức `update` Eloquent thường trả về một số nguyên. Tuy nhiên, chúng ta có thể buộc phương thức trả về chính model bằng cách gọi phương thức `update` thông qua hàm `tap`:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

<a name="method-today"></a>
#### `today()` {#collection-method}

Hàm `today` sẽ tạo ra một instance `Illuminate\Support\Carbon` mới cho ngày hiện tại:

    $today = today();

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

Hàm `throw_unless` sẽ đưa ra exception đã cho nếu một biểu thức boolean đã cholà `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {#collection-method}

Hàm `trait_uses_recursive` trả về tất cả các trait được sử dụng bởi một trait:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {#collection-method}

Hàm `transform` sẽ thực thi một `Closure` trên một giá trị đã cho nếu giá trị không [blank](#method-blank) và trả về kết quả của `Closure`:

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

Một giá trị mặc định hoặc `Closure` cũng có thể được pass làm tham số thứ ba cho phương thức. Giá trị này sẽ được trả về nếu giá trị đã cho là blank:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {#collection-method}

Hàm `validator` sẽ tạo ra một instance [validator](/docs/{{version}}/validation) mới với các tham số đã cho. Bạn có thể sử dụng nó thay vì facade  `Validator` cho thuận tiện:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {#collection-method}

Hàm `value` sẽ trả về giá trị được cho. Tuy nhiên, nếu bạn pass một `Closure` cho hàm, thì` Closure` sẽ được thực thi sau đó kết quả của nó sẽ được trả về:

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

Hàm `with` sẽ trả về giá trị được cho. Nếu một `Closure` được pass làm tham số thứ hai cho hàm, thì` Closure` sẽ được thực thi sau đó kết quả của nó sẽ được trả về:

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
