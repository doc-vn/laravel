# Collections

- [Giới thiệu](#introduction)
    - [Tạo collection](#creating-collections)
    - [Extend collection](#extending-collections)
- [Các phương thức có sẵn](#available-methods)
- [Higher Order Messages](#higher-order-messages)

<a name="introduction"></a>
## Giới thiệu

Class `Illuminate\Support\Collection` cung cấp wrapper dễ dàng, thuận tiện để làm việc với các mảng dữ liệu. Ví dụ, kiểm tra code sau đây. Chúng tôi sẽ sử dụng helper `collect` để tạo một instance collection mới từ mảng, chạy hàm `strtoupper` trên mỗi phần tử và sau đó xóa tất cả các phần tử trống:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

Như bạn có thể thấy, class `Collection` cho phép bạn tạo kết hợp các phương thức của nó để dễ dàng thực hiện mapping và reject phần tử khỏi mảng. Nói chung, các collection là bất biến, có nghĩa là mọi phương thức `Collection` trả về một instance `Collection` hoàn toàn mới.

<a name="creating-collections"></a>
### Tạo collection

Như đã đề cập ở trên, helper `collect` trả về một instance `Illuminate\Support\Collection`  mới cho mảng đã cho. Vì vậy, việc tạo ra một collection đơn giản như:

    $collection = collect([1, 2, 3]);

> {tip} Kết quả của các truy vấn [Eloquent](/docs/{{version}}/eloquent) luôn được trả về dưới dạng các instances `Collection`.

<a name="extending-collections"></a>
### Extend collection

Các collection là "macroable", cho phép bạn thêm các phương thức bổ sung vào class `Collection` trong thời gian chạy. Ví dụ, đoạn mã sau thêm một phương thức `toUpper` vào class `Collection`:

    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });

    $collection = collect(['first', 'second']);

    $upper = $collection->toUpper();

    // ['FIRST', 'SECOND']

Thông thường, bạn nên khai báo các collection macro trong một [service provider](/docs/{{version}}/providers).

<a name="available-methods"></a>
## Các phương thức có sẵn

Trong phần còn lại của tài liệu này, chúng ta sẽ thảo luận về từng phương thức có sẵn trên class `Collection`. Hãy nhớ rằng, tất cả các phương thức này đều có thể được kết hợp cùng nhau để xử lý dễ dàng mảng. Hơn nữa, hầu hết mọi phương thức đều trả về một instance `Collection` mới, cho phép bạn giữ bản sao gốc của collection khi cần thiết:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

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

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

Phương thức `all` trả về mảng cơ bản được biểu thị bởi collection:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {#collection-method}

Cách gọi khác cho phương thức [`avg`](#method-avg).

<a name="method-avg"></a>
#### `avg()` {#collection-method}

Phương thức `avg` trả về [giá trị trung bình](https://en.wikipedia.org/wiki/Average) của một key đã cho:

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

Phương thức `chunk` chia collection thành nhiều collection nhỏ hơn với kích thước nhất định:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

Phương thức này đặc biệt hữu ích trong [views](/docs/{{version}}/views) khi làm việc với hệ thống grid như [Bootstrap](https://getbootstrap.com/css/#grid). Hãy tưởng tượng bạn có một collection các model [Eloquent](/docs/{{version}}/eloquent) mà bạn muốn hiển thị trong một grid:

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

Phương thức `collapse` sẽ thu gọn một tập hợp các mảng nhỏ thành một collection riêng và ngang hàng:

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

Phương thức `combine` sẽ kết hợp các key của collection với các giá trị của mảng hoặc collection khác:

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-concat"></a>
#### `concat()` {#collection-method}

Phương thức `concat` sẽ nối các giá trị `array` hoặc collection đã cho vào cuối của một collection:

    $collection = collect(['John Doe']);

    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

    $concatenated->all();

    // ['John Doe', 'Jane Doe', 'Johnny Doe']

<a name="method-contains"></a>
#### `contains()` {#collection-method}

Phương thức `contains` sẽ xác định xem collection có chứa một item đã cho hay không:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

Bạn cũng có thể pass một cặp key / value cho phương thức `contains`, có sẽ xác định xem cặp đã cho có tồn tại trong collection hay không:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

Cuối cùng, bạn cũng có thể pass một callback cho phương thức `contains` để thực hiện kiểm tra truth của riêng bạn:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

Phương thức `contains` sử dụng các phép so sánh "lỏng lẻo" khi kiểm tra các giá trị item, nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng với một số integer có cùng giá trị. Sử dụng phương thức [`containsStrict`](#method-containsstrict) để lọc bằng các so sánh "nghiêm ngặt".

<a name="method-containsstrict"></a>
#### `containsStrict()` {#collection-method}

Phương thức này có cùng signature với phương thức [`contains`](#method-contains); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-count"></a>
#### `count()` {#collection-method}

Phương thức `count` trả về tổng số item trong collection:

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-crossjoin"></a>
#### `crossJoin()` {#collection-method}

Phương thức `crossJoin` sẽ join chéo các giá trị của collection vào trong các mảng hoặc các collection đã cho, trả về một Cartesian product với tất cả các hoán vị có thể có:

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b']);

    $matrix->all();

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

    $matrix->all();

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

<a name="method-dd"></a>
#### `dd()` {#collection-method}

Phương thức `dd` sẽ dump các item của collection và dừng thực thi:

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dd();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

Nếu bạn không muốn dừng thực thi, hãy sử dụng phương thức [`dump`](#method-dump) để thay thế.

<a name="method-diff"></a>
#### `diff()` {#collection-method}

Phương thức `diff` so sánh collection với collection khác hoặc PHP `array` dựa trên các giá trị của nó. Phương thức này sẽ trả về các giá trị trong collection gốc không có trong collection đã cho:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffassoc"></a>
#### `diffAssoc()` {#collection-method}

Phương thức `diffAssoc` so sánh collection với collection khác hoặc một PHP `array` dựa trên các key và value của nó. Phương thức này sẽ trả về các cặp key / value trong collection gốc không có trong collection đã cho:

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

Phương thức `diffKeys` so sánh collection với collection khác hoặc một PHP `array` dựa trên các key của nó. Phương thức này sẽ trả về các cặp key / value trong collection gốc không có trong collection đã cho:

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-dump"></a>
#### `dump()` {#collection-method}

Phương thức `dump` sẽ dump các item của collection:

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dump();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

Nếu bạn muốn dừng thực thi sau khi dump collection, hãy sử dụng phương thức [`dd`](#method-dd) để thay thế.

<a name="method-each"></a>
#### `each()` {#collection-method}

Phương thức `each` sẽ lặp lại các item trong collection và pass từng item vào một callback:

    $collection = $collection->each(function ($item, $key) {
        //
    });

Nếu bạn muốn dừng lặp qua các item, bạn có thể trả về `false` từ callback của bạn:

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-eachspread"></a>
#### `eachSpread()` {#collection-method}

Phương thức `eachSpread` lặp lại các item của collection, pass từng giá trị item bị lồng nhau vào hàm callback đã cho:

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

    $collection->eachSpread(function ($name, $age) {
        //
    });

Nếu bạn muốn dừng lặp qua các item, bạn có thể trả về `false` từ callback của bạn:

    $collection->eachSpread(function ($name, $age) {
        return false;
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

Phương thức `every` có thể được sử dụng để xác minh rằng tất cả các element của collection pass qua một số điều kiện đã cho:

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

<a name="method-except"></a>
#### `except()` {#collection-method}

Phương thức `except` trả về tất cả các item trong collection ngoại trừ các item có các key được chỉ định:

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

Đối với ngược với phương thức `except`, hãy xem phương thức [only](#method-only).

<a name="method-filter"></a>
#### `filter()` {#collection-method}

Phương thức `filter` sẽ lọc collection bằng cách dùng callback đã cho, chỉ giữ lại những item pass qua một số điều kiện đã cho:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

Nếu không có callback nào được cung cấp, tất cả các item trong collection tương đương với `false` sẽ bị xóa:

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

Đối với ngược với phương thức `filter`, hãy xem phương thức [reject](#method-reject).

<a name="method-first"></a>
#### `first()` {#collection-method}

Phương thức `first` trả về phần tử đầu tiên trong collection pass qua một số điều kiện đã cho:

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

Bạn cũng có thể gọi phương thức `first` không có tham số để lấy phần tử đầu tiên trong collection. Nếu collection trống, `null` được trả về:

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-first-where"></a>
#### `firstWhere()` {#collection-method}

Phương thức `firstWhere` trả về phần tử đầu tiên trong collection với cặp key / value đã cho:
The `firstWhere` method returns the first element in the collection with the given key / value pair:

    $collection = collect([
        ['name' => 'Regena', 'age' => 12],
        ['name' => 'Linda', 'age' => 14],
        ['name' => 'Diego', 'age' => 23],
        ['name' => 'Linda', 'age' => 84],
    ]);

    $collection->firstWhere('name', 'Linda');

    // ['name' => 'Linda', 'age' => 14]

Bạn cũng có thể gọi phương thức `firstWhere` bằng toán tử:

    $collection->firstWhere('age', '>=', 18);

    // ['name' => 'Diego', 'age' => 23]

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

Phương thức `flatMap` lặp lại qua collection và pass từng giá trị cho callback đã cho. Callback được tự do sửa đổi item và trả lại nó, do đó hình thành một collection mới với các item đã được sửa đổi. Sau đó, mảng được làm ngang hàng ở một mức:

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

Phương thức `flatten` làm ngang hàng một collection đa chiều thành một chiều duy nhất:

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

Bạn có thể tùy ý pass cho hàm với một tham số "depth":

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

Trong ví dụ này, gọi `flatten` mà không cung cấp độ sâu cũng sẽ làm ngang hàng các mảng lồng nhau, dẫn đến `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung'] `. Cung cấp độ sâu cho phép bạn hạn chế các mức của các mảng lồng nhau sẽ được làm ngang hàng.

<a name="method-flip"></a>
#### `flip()` {#collection-method}

Phương thức `flip` sẽ hoán đổi các key của collection với các giá trị tương ứng của chúng:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

Phương thức `forget` xóa một item khỏi collection bằng key của nó:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} Không giống như hầu hết các phương thức collection khác, `forget` không trả về collection đã sửa mới; mà nó sửa trực tiếp lên collection mà nó được gọi.

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

Phương thức `forPage` sẽ trả về một collection mới chứa các item sẽ xuất hiện trên một số trang nhất định. Phương thức chấp nhận số trang làm tham số đầu tiên và số item sẽ hiển thị trong mỗi trang làm tham số thứ hai:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {#collection-method}

Phương thức `get` sẽ trả về item tại một key đã cho. Nếu key không tồn tại, `null` được trả về:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

Bạn có thể tùy chọn pass một giá trị mặc định làm tham số thứ hai:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

Bạn thậm chí có thể pass một callback như là giá trị mặc định. Kết quả của callback sẽ được trả về nếu key được chỉ định không tồn tại:

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

Phương thức `groupBy` sẽ nhóm các item của collection theo một key đã cho:

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

Ngoài việc pass một chuỗi `key`, bạn cũng có thể pass một callback. Callback sẽ trả về giá trị bạn muốn key cho nhóm bằng cách:

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

Nếu bạn có bhiều tiêu chí nhóm, bạn có thể được pass dưới dạng một mảng. Mỗi phần tử mảng sẽ được áp dụng cho level tương ứng trong một mảng nhiều chiều:

    $data = new Collection([
        10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
    ]);

    $result = $data->groupBy([
        'skill',
        function ($item) {
            return $item['roles'];
        },
    ], $preserveKeys = true);

    /*
    [
        1 => [
            'Role_1' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_2' => [
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_3' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            ],
        ],
        2 => [
            'Role_1' => [
                30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
            ],
            'Role_2' => [
                40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
            ],
        ],
    ];
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

Phương thức `has` xác định nếu một key đã cho tồn tại trong collection hay không:

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('product');

    // true

<a name="method-implode"></a>
#### `implode()` {#collection-method}

Phương thức `implode` là kết hợp các item trong một collection. Tham số của nó phụ thuộc vào loại item trong collection. Nếu collection chứa các mảng hoặc đối tượng, bạn nên pass key của các thuộc tính bạn muốn join và chuỗi "glue" bạn muốn đặt vào giữa các giá trị:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

Nếu collection chứa các chuỗi hoặc giá trị số đơn giản, hãy pass "glue" làm tham số duy nhất cho phương thức:

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

Phương thức `intersect` sẽ loại bỏ bất kỳ giá trị nào khỏi collection ban đầu nếu không có trong `array` hoặc collection đã cho. Collection kết quả sẽ còn lại các key của collection gốc:

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {#collection-method}

Phương thức `intersectByKeys` loại bỏ bất kỳ key nào khỏi collection ban đầu nếu không có trong `array` hoặc collection đã cho:

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

Phương thức `isEmpty` trả về` true` nếu collection trống; ngược lại, `false` được trả về:
    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {#collection-method}

Phương thức `isNotEmpty` trả về `true` nếu collection không trống; ngược lại, `false` được trả về:

    collect([])->isNotEmpty();

    // false

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

Phương thức `keyBy` sẽ key hoá collection bằng key đã cho. Nếu nhiều item có cùng key, chỉ có item cuối cùng sẽ tạo trong collection mới:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

Bạn cũng có thể pass một callback cho phương thức. Callback sẽ trả về giá trị cho key collection bằng cách:

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>
#### `keys()` {#collection-method}

Phương thức `keys` sẽ trả về tất cả các key của collection:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

Phương thức `last` sẽ trả về phần tử cuối cùng trong collection pass qua một số điều kiện đã cho:

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2

Bạn cũng có thể gọi phương thức `last` không có tham số để lấy phần tử cuối cùng trong collection. Nếu collection trống, `null` được trả về:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-macro"></a>
#### `macro()` {#collection-method}

Phương thức tĩnh `macro` cho phép bạn thêm các phương thức vào lớp `Collection` trong thời gian chạy. Tham khảo tài liệu về [extending collections](#extending-collections) để biết thêm thông tin.

<a name="method-make"></a>
#### `make()` {#collection-method}

Phương thức tĩnh `make` sẽ tạo ra một instance collection mới. Xem phần [Tạo collection](#creating-collections).

<a name="method-map"></a>
#### `map()` {#collection-method}

Phương thức `map` lặp lại qua collection và pass từng giá trị cho hàm callback đã cho. Callback cho phép bạn tự do sửa đổi các item và trả lại nó, do đó hình thành một collection mới với các item đã được sửa:

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> {note} Giống như hầu hết các phương thức collection khác, `map` trả về một instance collection mới; nó không sửa trực tiếp vào collection mà nó được gọi. Nếu bạn muốn sửa đổi trực tiếp vào collection gốc, hãy sử dụng phương thức [`transform`](#method-transform).

<a name="method-mapinto"></a>
#### `mapInto()` {#collection-method}

Phương thức `mapInto()` lặp lại collectionp, tạo một instance mới của class đã cho bằng cách pass giá trị vào hàm khởi tạo:

    class Currency
    {
        /**
         * Create a new currency instance.
         *
         * @param  string  $code
         * @return void
         */
        function __construct(string $code)
        {
            $this->code = $code;
        }
    }

    $collection = collect(['USD', 'EUR', 'GBP']);

    $currencies = $collection->mapInto(Currency::class);

    $currencies->all();

    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>
#### `mapSpread()` {#collection-method}

Phương thức `mapSpread` lặp lại các item của collection, pass từng giá trị item bị lồng vào nhau  vào hàm callback đã cho. Callback cho phép bạn tự do sửa đổi item và trả lại nó, do đó hình thành một collection mới với các item đã được sửa:

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function ($odd, $even) {
        return $odd + $even;
    });

    $sequence->all();

    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>
#### `mapToGroups()` {#collection-method}

Phương thức `mapToGroups` sẽ nhóm các item của collection theo hàm callback đã cho. Callback sẽ trả về một mảng kết hợp có chứa một cặp key / giá trị duy nhất, do đó tạo thành một collection của các giá trị được nhóm mới:

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);

    $grouped = $collection->mapToGroups(function ($item, $key) {
        return [$item['department'] => $item['name']];
    });

    $grouped->toArray();

    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johhny Doe'],
        ]
    */

    $grouped->get('Sales')->all();

    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {#collection-method}

Phương thức `mapWithKeys` lặp qua collection và pass từng giá trị cho hàm callback đã cho. Callback sẽ trả về một mảng kết hợp có chứa một cặp key / giá trị duy nhất:

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);

    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` {#collection-method}

Phương thức `max` trả về giá trị lớn nhất của một key đã cho:

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {#collection-method}

Phương thức `median` trả về [giá trị trung vị](https://vi.wikipedia.org/wiki/S%E1%BB%91_trung_v%E1%BB%8B) của một key đã cho:

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

Phương thức `merge` sẽ merge mảng hoặc collection đã cho với collection gốc. Nếu key trong các item đã cho khớp với key trong collection gốc, giá trị của các item đã cho sẽ ghi đè lên giá trị trong collection gốc:

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

Nếu các key của các items đã cho là số, các giá trị sẽ được thêm vào cuối collection:

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

Phương thức `min` trả về giá trị nhỏ nhất của một key đã cho:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {#collection-method}

Phương thức `mode` trả về [giá trị yếu vị](https://vi.wikipedia.org/wiki/S%E1%BB%91_y%E1%BA%BFu_v%E1%BB%8B) của một key đã cho:

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>
#### `nth()` {#collection-method}

Phương thức `nth` tạo ra một collection mới chứa mọi phần tử (an) với n là vị trí muốn lấy và a là số nguyên tăng dần từ 0:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

Bạn có thể pass một phần bù làm tham số thứ hai và công thức là (an + b) với b là số bù:

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {#collection-method}

Phương thức `only` trả về các item trong collection với các key được chỉ định:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Đối với ngược với phương thức `only`, hãy xem phương thức [except](#method-except).

<a name="method-pad"></a>
#### `pad()` {#collection-method}

Phương thức `pad` sẽ fill vào mảng với giá trị đã cho cho đến khi mảng đạt kích thước đã chỉ định. Phương thức này hoạt động giống như hàm PHP [array_pad](https://secure.php.net/manual/en/function.array-pad.php).

Để pad sang bên trái, bạn nên khai báo kích thước âm. Sẽ không pad nếu giá trị tuyệt đối của kích thước đã cho nhỏ hơn hoặc bằng độ dài của mảng:

    $collection = collect(['A', 'B', 'C']);

    $filtered = $collection->pad(5, 0);

    $filtered->all();

    // ['A', 'B', 'C', 0, 0]

    $filtered = $collection->pad(-5, 0);

    $filtered->all();

    // [0, 0, 'A', 'B', 'C']

<a name="method-partition"></a>
#### `partition()` {#collection-method}

Phương thức `partition` có thể được kết hợp với hàm PHP` list` để tách các phần tử pass được điều kiện và các phần tử không pass được điều kiện:

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

<a name="method-pipe"></a>
#### `pipe()` {#collection-method}

Phương thức `pipe` pass collection đến callback đã cho và trả về kết quả:

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

Phương thức `pluck` lấy tất cả các giá trị cho một key đã cho:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

Bạn cũng có thể khai báo cách bạn muốn collection kết quả dùng key nào từ collection gốc:

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

Phương thức `pop` xóa và trả về item cuối cùng từ collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

Phương thức `prepend` thêm một item vào đầu collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

Bạn cũng có thể pass một tham số thứ hai để set key của item được prepend:

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

Phương thức `pull` loại bỏ và trả về một item từ collectionp bằng key của nó:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

Phương thức `push` nối thêm một item vào cuối collection:

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

Phương thức `put` sẽ set key và giá trị đã cho vào trong collection:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

Phương thức `Random` trả về một item ngẫu nhiên từ collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

Bạn có thể tùy chọn pass một số nguyên cho `random` để khai báo số lượng item bạn muốn lấy ngẫu nhiên. Một collection các item sẽ được trả về khi pass một số lượng item bạn muốn nhận:

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

Nếu collection có ít item hơn yêu cầu, phương thức sẽ đưa ra một exception `InvalidArgumentException`.

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

Phương thức `reduce` sẽ giảm collection thành một giá trị duy nhất, và sẽ chuyển kết quả của lần lặp trước vào lần lặp tiếp theo:

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

Giá trị cho `$carry` trong lần lặp đầu tiên là `null`; tuy nhiên, bạn có thể khai báo giá trị ban đầu của nó bằng cách pass một tham số thứ hai cho `reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

Phương thức `reject` sẽ lọc collection bằng cách sử dụng hàm callback đã cho. Callback sẽ trả về `true` nếu item bị xóa khỏi collection kết quả:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

Đối với ngược với phương thức `reject`, hãy xem phương thức [`filter`](#method-filter).

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

Phương thức `reverse` sẽ đảo ngược thứ tự của các item của collection, giữ nguyên các key gốc:

    $collection = collect(['a', 'b', 'c', 'd', 'e']);

    $reversed = $collection->reverse();

    $reversed->all();

    /*
        [
            4 => 'e',
            3 => 'd',
            2 => 'c',
            1 => 'b',
            0 => 'a',
        ]
    */

<a name="method-search"></a>
#### `search()` {#collection-method}

Phương thức `search` sẽ tìm kiếm collection cho giá trị đã cho và trả về key của nó nếu được tìm thấy. Nếu item không được tìm thấy, `false` được trả về.

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

Việc tìm kiếm được thực hiện bằng cách sử dụng so sánh "lỏng lẻo", nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng một số integer có cùng giá trị. Để sử dụng so sánh "nghiêm ngặt", hãy pass `true` làm tham số thứ hai cho phương thức:

    $collection->search('4', true);

    // false

Ngoài ra, bạn có thể pass vào một callback của chính bạn để tìm kiếm item đầu tiên pass qua điều kiện của bạn:

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

Phương thức `shift` loại bỏ và trả về item đầu tiên từ collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

Phương thức `shuffle` sẽ xáo trộn ngẫu nhiên các item trong collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

Phương thức `slice` trả về một phần của collection bắt đầu từ index đã cho:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

Nếu bạn muốn giới hạn kích thước của phần được trả về, hãy pass tham sô kích thước mong muốn làm tham số thứ hai cho phương thức:

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

Phần được trả lại sẽ mặc định giữ nguyên các key. Nếu bạn không muốn giữ các key gốc, bạn có thể sử dụng phương thức [`values`](#method-values) để reindex lại chúng.

<a name="method-sort"></a>
#### `sort()` {#collection-method}

Phương thức `sort` sẽ sắp xếp collection. Collection được sắp xếp giữ nguyên các key mảng gốc, vì vậy trong ví dụ này, chúng ta sẽ sử dụng phương thức [`values`](#method-values) để set lại các key thành các index được đánh số liên tục:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

Nếu bạn cần xắp sếp nâng cao hơn, bạn có thể pass một callback tới `sort` bằng thuật toán của riêng bạn. Tham khảo tài liệu PHP về [`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), chi tiết hơn thì đây là phương thức `sort` của collection sẽ gọi tới.

> {tip} Nếu bạn cần sắp xếp một collection các mảng hoặc object lồng nhau, hãy xem các phương thức [`sortBy`](#method-sortby) và [`sortByDesc`](#method-sortbydesc).

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

Phương thức `sortBy` sẽ sắp xếp collection theo key đã cho. Collection đã được sắp xếp sẽ giữ các key của mảng gốc, vì vậy trong ví dụ này, chúng tôi sẽ sử dụng phương thức [`values`](#method-values) để set lại các key thành các chỉ mục được đánh số liên tục:

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

Bạn cũng có thể pass qua một callback của riêng bạn để xác định cách sắp xếp các giá trị collection:

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

Phương thức này có cùng chức năng với phương thức [`sortBy`](#method-sortby), nhưng sẽ sắp xếp collection theo thứ tự ngược lại.

<a name="method-splice"></a>
#### `splice()` {#collection-method}

Phương thức `splice` loại bỏ và trả về một phần các item bắt đầu từ index được khai báo:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

Bạn có thể pass tham số thứ hai để giới hạn kích thước của kết quả:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

Ngoài ra, bạn có thể pass tham số thứ ba có chứa các item mới để thay thế các item bị xóa khỏi collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {#collection-method}

Phương thức `split` sẽ chia một collection thành một số nhóm:

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->toArray();

    // [[1, 2], [3, 4], [5]]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

Phương thức `sum` trả về tổng của tất cả các item trong collection:

    collect([1, 2, 3, 4, 5])->sum();

    // 15

Nếu collection chứa các mảng hoặc đối tượng lồng nhau, bạn nên pass key để sử dụng để xác định giá trị nào cần tính tổng:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 127

Ngoài ra, bạn có thể pass một callback của chính bạn để xác định giá trị nào của collection cần tính tổng:

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

Phương thức `Take`taketrả về một collection mới với số lượng item được chỉ định:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

Bạn cũng có thể pass một số âm để lấy số lượng item được chỉ định từ cuối collection:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-tap"></a>
#### `tap()` {#collection-method}

Phương thức `tap` sẽ pass collection đến một callback đã cho, cho phép bạn "tap" vào collection tại một điểm cụ thể và làm một cái gì đó với các item trong khi không ảnh hưởng đến chính collection:

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` {#collection-method}

Phương thức tĩnh `times` tạo ra một collection mới bằng cách gọi hàm callback một số lần nhất định:

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

Phương pháp này có thể hữu ích khi được kết hợp với các factory để tạo các model [Eloquent](/docs/{{version}}/eloquent):

    $categories = Collection::times(3, function ($number) {
        return factory(Category::class)->create(['name' => 'Category #'.$number]);
    });

    $categories->all();

    /*
        [
            ['id' => 1, 'name' => 'Category #1'],
            ['id' => 2, 'name' => 'Category #2'],
            ['id' => 3, 'name' => 'Category #3'],
        ]
    */

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

Phương thức `toArray` sẽ chuyển đổi collection thành một PHP `array`. Nếu các giá trị của collection là các model [Eloquent](/docs/{{version}}/eloquent), thì các mdoel cũng sẽ được chuyển đổi thành mảng:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} `toArray` cũng chuyển đổi tất cả các đối tượng lồng nhau của collection thành một mảng. Nếu bạn muốn lấy raw mảng cơ bản, thay vào đó, hãy sử dụng phương thức [`all`](#method-all).

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

Phương thức `toJson` chuyển đổi collection thành một chuỗi JSON:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

Phương thức `Transform` lặp lại collection và gọi hàm callback đã cho với từng item trong collection. Các item trong collection sẽ được thay thế bằng các giá trị được trả về bởi hàm callback:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> {note} Không giống như hầu hết các phương thức collection khác, `transform` tự sửa transform. Thay vào đó, nếu bạn muốn tạo một transform mới, hãy sử dụng phương thức [`map`](#method-map).

<a name="method-union"></a>
#### `union()` {#collection-method}

Phương thức `union` thêm mảng đã cho vào collection. Nếu mảng đã cho chứa các khóa đã có trong collection gốc, các giá trị của collection gốc sẽ được ưu tiên:

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

Phương thức `unique` trả về tất cả các unique item trong collection. Collection được trả về giữ nguyên các key mảng gốc, vì vậy trong ví dụ này, chúng ta sẽ sử dụng phương thức [`values`](#method-values) để set lại các key thành các index được đánh số liên tục:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

Khi xử lý các mảng hoặc đối tượng lồng nhau, bạn có thể chỉ định key được sử dụng để xác định tính duy nhất:

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

Bạn cũng có thể pass một callback của chính bạn để xác định tính duy nhất của item:

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

Phương thức `unique` sử dụng các phép so sánh "lỏng lẻo" khi kiểm tra các giá trị item, nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng một số integer có cùng giá trị. Sử dụng phương thức [`uniqueStrict`](#method-uniquestrict) để lọc bằng các so sánh "nghiêm ngặt".

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {#collection-method}

Phương thức này có cùng chức năng với phương thức [`unique`](#method-unique); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-unless"></a>
#### `unless()` {#collection-method}

Phương thức `unless` sẽ chạy hàm callback đã cho ngoại trừ tham số đầu tiên được cung cấp cho phương thức là `true`:

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->unless(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

For the inverse of `unless`, see the [`when`](#method-when) method.

<a name="method-unwrap"></a>
#### `unwrap()` {#collection-method}

Phương thức tĩnh `unwrap` sẽ trả về các item cơ bản của collection từ giá trị đã cho khi áp dụng:

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-values"></a>
#### `values()` {#collection-method}

Phương thức `value` trả về một collection mới với các key được reset thành các số nguyên liên tiếp:

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` {#collection-method}

Phương thức `when` sẽ chạy callback đã cho khi tham số đầu tiên được cung cấp cho phương thứclà `true`:

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->when(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 4]

Đối với ngược với phương thức `when`, hãy xem phương thức [`unless`](#method-unless).

<a name="method-where"></a>
#### `where()` {#collection-method}

Phương thức `where` sẽ lọc collection theo cặp key / giá trị đã cho:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

Phương thức `where` sử dụng phép so sánh "lỏng lẻo" khi kiểm tra các giá trị item, nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng một số integer có cùng giá trị. Sử dụng phương thức [`whereStrict`](#method-wherestrict) để lọc bằng các so sánh "nghiêm ngặt".

<a name="method-wherestrict"></a>
#### `whereStrict()` {#collection-method}

Phương thức này có cùng chức năng với phương thức [`where`](#method-where); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

Phương thức `whereIn` sẽ lọc collection theo một key / giá trị có trong mảng đã cho:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Desk', 'price' => 200],
        ]
    */

Phương thức `whereIn` sử dụng phép so sánh "lỏng lẻo" khi kiểm tra các giá trị item, nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng một số integer có cùng giá trị. Sử dụng phương thức [`whereInStrict`](#method-whereinstrict) để lọc bằng các so sánh "nghiêm ngặt".

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {#collection-method}

Phương thức này có cùng chức năng với phương thức [`whereIn`](#method-wherein); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-wherenotin"></a>
#### `whereNotIn()` {#collection-method}

Phương thức `whereNotIn` sẽ lọc collection theo một key / giá trị không có trong mảng đã cho:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

Phương thức `whereNotIn` sử dụng phép so sánh "lỏng lẻo" khi kiểm tra các giá trị item, nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng một số integer có cùng giá trị. Sử dụng phương thức [`whereNotInStrict`](#method-wherenotinstrict) để lọc bằng các so sánh "nghiêm ngặt".

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {#collection-method}

Phương thức này có cùng chức năng với phương thức [`whereNotIn`](#method-wherenotin); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-wrap"></a>
#### `wrap()` {#collection-method}

Phương thức tĩnh `wrap` sẽ wrap giá trị đã cho trong một collection khi áp dụng:

    $collection = Collection::wrap('John Doe');

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(['John Doe']);

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(collect('John Doe'));

    $collection->all();

    // ['John Doe']

<a name="method-zip"></a>
#### `zip()` {#collection-method}

Phương thức `zip` sẽ merge các giá trị của mảng đã cho với các giá trị của collection gốc tại index tương ứng:

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## Higher Order Messages

Collection cũng cung cấp hỗ trợ cho "higher order messages", đó là các cách rút gọn để thực hiện các hành động phổ biến trên các collection. Các phương thức collection cung cấp các higher order message hơn là: `average`, `avg`, `contains`, `each`, `every`, `filter`, `first`, `flatMap`, `map`, `partition`, `reject`, `sortBy`, `sortByDesc`, `sum`, và `unique`.

Mỗi igher order message có thể được truy cập như một thuộc tính động trên một instance của collection. Chẳng hạn, hãy sử dụng higher order message `each` để gọi một phương thức trên mỗi đối tượng trong một collection:

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

Tương tự, chúng ta có thể sử dụng higher order message `sum` để tính tổng số "votes" cho một collection user:

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;
