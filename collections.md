# Collections

- [Giới thiệu](#introduction)
    - [Tạo collection](#creating-collections)
    - [Extend collection](#extending-collections)
- [Các phương thức có sẵn](#available-methods)
- [Higher Order Messages](#higher-order-messages)
- [Lazy Collections](#lazy-collections)
    - [Giới thiệu](#lazy-collection-introduction)
    - [Tạo Lazy Collections](#creating-lazy-collections)
    - [The Enumerable Contract](#the-enumerable-contract)
    - [Các phương thức của Lazy Collection](#lazy-collection-methods)

<a name="introduction"></a>
## Giới thiệu

Class `Illuminate\Support\Collection` cung cấp một wrapper dễ dàng, thuận tiện để làm việc với các mảng dữ liệu. Ví dụ, hãy xem code sau đây. Chúng ta sẽ sử dụng helper `collect` để tạo ra một instance collection mới từ một mảng, và chạy hàm `strtoupper` cho mỗi phần tử và xóa đi tất cả các phần tử trống:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })->reject(function ($name) {
        return empty($name);
    });

Như các bạn có thể thấy, class `Collection` cho phép bạn kết hợp các phương thức của nó lại với nhau cho phép bạn dễ dàng thực hiện mapping và reject các phần tử ra khỏi mảng. Nói chung, các collection là bất biến, điều này có nghĩa là mọi phương thức của `Collection` đều trả về một instance `Collection` mới.

<a name="creating-collections"></a>
### Tạo collection

Như đã đề cập ở trên, helper `collect` sẽ trả về một instance `Illuminate\Support\Collection` cho mảng đã được cho. Vì vậy, việc tạo một collection rất đơn giản như sau:

    $collection = collect([1, 2, 3]);

> **Note**
> Kết quả của các truy vấn [Eloquent](/docs/{{version}}/eloquent) cũng luôn được trả về dưới dạng các instances `Collection`.

<a name="extending-collections"></a>
### Extend collection

Các collection là các "macroable", nên nó cho phép bạn bổ sung các phương thức vào các class `Collection` trong thời gian chạy. Phương thức `macro` của class `Illuminate\Support\Collection` cũng chấp nhận một closure sẽ được thực thi khi macro của bạn được gọi. Closure macro có thể truy cập vào các phương thức khác của collection thông qua `$this`, giống như nó là một phương thức thực sự của class collection. Ví dụ, đoạn code sau sẽ thêm một phương thức `toUpper` vào class `Collection`:

    use Illuminate\Support\Collection;
    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });

    $collection = collect(['first', 'second']);

    $upper = $collection->toUpper();

    // ['FIRST', 'SECOND']

Thông thường, bạn nên khai báo các collection macro trong phương thức `boot` của một [service provider](/docs/{{version}}/providers).

<a name="macro-arguments"></a>
#### Macro Arguments

Nếu cần, bạn có thể định nghĩa các macro chấp nhận các thêm các tham số bổ sung:

    use Illuminate\Support\Collection;
    use Illuminate\Support\Facades\Lang;

    Collection::macro('toLocale', function ($locale) {
        return $this->map(function ($value) use ($locale) {
            return Lang::get($value, [], $locale);
        });
    });

    $collection = collect(['first', 'second']);

    $translated = $collection->toLocale('es');

<a name="available-methods"></a>
## Các phương thức có sẵn

Trong phần lớn tài liệu collection còn lại này, chúng ta sẽ thảo luận về các phương thức có sẵn trên class `Collection`. Hãy nhớ rằng, tất cả các phương thức này đều có thể được kết hợp lại với nhau để xử lý cho một mảng dễ dàng. Hơn nữa, hầu hết mọi phương thức đều sẽ trả về một instance `Collection` mới, giúp bạn giữ bản gốc của collection đó khi cần thiết:

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

<div class="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsOneItem](#method-containsoneitem)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[doesntContain](#method-doesntcontain)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstOrFail](#method-first-or-fail)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[hasAny](#method-hasany)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[lazy](#method-lazy)
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
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pipeInto](#method-pipeinto)
[pipeThrough](#method-pipethrough)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[range](#method-range)
[reduce](#method-reduce)
[reduceSpread](#method-reduce-spread)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[skip](#method-skip)
[skipUntil](#method-skipuntil)
[skipWhile](#method-skipwhile)
[slice](#method-slice)
[sliding](#method-sliding)
[sole](#method-sole)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortDesc](#method-sortdesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[sortKeysUsing](#method-sortkeysusing)
[splice](#method-splice)
[split](#method-split)
[splitIn](#method-splitin)
[sum](#method-sum)
[take](#method-take)
[takeUntil](#method-takeuntil)
[takeWhile](#method-takewhile)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[undot](#method-undot)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[value](#method-value)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[whereNotNull](#method-wherenotnull)
[whereNull](#method-wherenull)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

<a name="method-listing"></a>
## Method Listing

<style>
    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="method-all"></a>
#### `all()` {.collection-method .first-collection-method}

Phương thức `all` sẽ trả về một mảng được biểu thị bởi collection:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {.collection-method}

Cách gọi khác của phương thức [`avg`](#method-avg).

<a name="method-avg"></a>
#### `avg()` {.collection-method}

Phương thức `avg` trả về [giá trị trung bình](https://en.wikipedia.org/wiki/Average) của một key đã cho:

    $average = collect([
        ['foo' => 10],
        ['foo' => 10],
        ['foo' => 20],
        ['foo' => 40]
    ])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` {.collection-method}

Phương thức `chunk` chia collection thành nhiều collection nhỏ hơn với kích thước nhất định:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->all();

    // [[1, 2, 3, 4], [5, 6, 7]]

Phương thức này đặc biệt hữu ích trong [views](/docs/{{version}}/views) khi làm việc với các hệ thống grid như [Bootstrap](https://getbootstrap.com/docs/4.1/layout/grid/). Ví dụ, hãy tưởng tượng bạn có một collection các model [Eloquent](/docs/{{version}}/eloquent) mà bạn muốn hiển thị trong một grid:

```blade
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

<a name="method-chunkwhile"></a>
#### `chunkWhile()` {.collection-method}

Phương thức `chunkWhile` sẽ chia collection ra thành nhiều collection nhỏ dựa trên kết quả của callback đã cho. Biến `$chunk` được truyền cho closure có thể được sử dụng để kiểm tra phần tử trước đó:

    $collection = collect(str_split('AABBCCCD'));

    $chunks = $collection->chunkWhile(function ($value, $key, $chunk) {
        return $value === $chunk->last();
    });

    $chunks->all();

    // [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]

<a name="method-collapse"></a>
#### `collapse()` {.collection-method}

Phương thức `collapse` sẽ thu gọn một tập hợp các mảng nhỏ thành một collection chung và ngang hàng với nhau:

    $collection = collect([
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],
    ]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-collect"></a>
#### `collect()` {.collection-method}

Phương thức `collect` sẽ trả về một instance `Collection` mới với các item hiện có có trong collection:

    $collectionA = collect([1, 2, 3]);

    $collectionB = $collectionA->collect();

    $collectionB->all();

    // [1, 2, 3]

Phương thức `collect` chủ yếu hữu ích để chuyển đổi các [lazy collections](#lazy-collections) thành các instance `Collection` cơ bản:

    $lazyCollection = LazyCollection::make(function () {
        yield 1;
        yield 2;
        yield 3;
    });

    $collection = $lazyCollection->collect();

    get_class($collection);

    // 'Illuminate\Support\Collection'

    $collection->all();

    // [1, 2, 3]

> **Note**
> Phương thức `collect` đặc biệt hữu ích khi bạn có một instance của `Enumerable` và cần một instance collection non-lazy. Vì `collect()` là một phần của contract `Enumerable` nên bạn có thể sử dụng nó một cách an toàn để lấy ra một instance `Collection`.

<a name="method-combine"></a>
#### `combine()` {.collection-method}

Phương thức `combine` sẽ lấy các value của collection để làm key và các giá trị sẽ lấy từ mảng hoặc một collection khác:

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-concat"></a>
#### `concat()` {.collection-method}

Phương thức `concat` sẽ gắn thêm các giá trị của một `array` hoặc các giá trị của collection vào cuối của một collection khác:

    $collection = collect(['John Doe']);

    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

    $concatenated->all();

    // ['John Doe', 'Jane Doe', 'Johnny Doe']

Phương thức `concat` sẽ lập lại index các khóa cho các item được thêm vào collection gốc. Để giữ nguyên các khóa trong collection, hãy xem phương thức [merge](#method-merge).

<a name="method-contains"></a>
#### `contains()` {.collection-method}

Phương thức `contains` sẽ xác định xem trong collection đó có chứa item đã cho hay không. Bạn có thể truyền một closure cho phương thức `contains` để xác định xem có tồn tại một phần tử nào trong collection khớp với một kiểm tra đã cho hay không:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

Ngoài ra, bạn có thể truyền một chuỗi vào phương thức `contains` để xác định xem collection có chứa một giá trị đã cho hay không:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

Bạn cũng có thể truyền vào một cặp key và value cho phương thức `contains`, nó sẽ xác định xem cặp key value đó có tồn tại trong collection hay không:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

Phương thức `contains` sử dụng các phép so sánh "lỏng lẻo" khi kiểm tra các giá trị của item, nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng với một số integer có cùng giá trị. Sử dụng phương thức [`containsStrict`](#method-containsstrict) để so sánh "nghiêm ngặt".

Đối ngược với phương thức `contains`, hãy xem phương thức [doesntContain](#method-doesntcontain).

<a name="method-containsoneitem"></a>
#### `containsOneItem()` {.collection-method}

Phương thức `containsOneItem` sẽ xác định xem collection có chứa một item hay không:

    collect([])->containsOneItem();

    // false

    collect(['1'])->containsOneItem();

    // true

    collect(['1', '2'])->containsOneItem();

    // false

<a name="method-containsstrict"></a>
#### `containsStrict()` {.collection-method}

Phương thức này có cùng dạng với phương thức [`contains`](#method-contains); tuy nhiên, tất cả các giá trị được so sánh đều sử dụng phép so sánh "nghiêm ngặt".

> **Note**
> Hành vi của phương thức này được thay đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-contains).

<a name="method-count"></a>
#### `count()` {.collection-method}

Phương thức `count` sẽ trả về tổng số các item trong collection:

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-countBy"></a>
#### `countBy()` {.collection-method}

Phương thức `countBy` sẽ đếm số lần xuất hiện của các giá trị trong một collection. Mặc định, phương thức này sẽ đếm số lần xuất hiện của tất cả các phần tử, cho phép bạn đếm các "loại" phần tử trong collection:

    $collection = collect([1, 2, 2, 2, 3]);

    $counted = $collection->countBy();

    $counted->all();

    // [1 => 1, 2 => 3, 3 => 1]

Nếu bạn truyền vào một closure cho phương thức `countBy`, thì nó sẽ đếm tất cả các item theo quy luật tùy chỉnh của bạn:

    $collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

    $counted = $collection->countBy(function ($email) {
        return substr(strrchr($email, "@"), 1);
    });

    $counted->all();

    // ['gmail.com' => 2, 'yahoo.com' => 1]

<a name="method-crossjoin"></a>
#### `crossJoin()` {.collection-method}

Phương thức `crossJoin` sẽ join chéo các giá trị của collection vào trong các mảng hoặc các collection đã cho, trả về một tích chéo với tất cả các hoán vị có thể có:

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
#### `dd()` {.collection-method}

Phương thức `dd` sẽ hiển thi các item có trong collection và dừng thực thi lệnh ngay tại đó:

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

Nếu bạn không muốn dừng thực thi lệnh, thì hãy sử dụng phương thức [`dump`](#method-dump) để thay thế.

<a name="method-diff"></a>
#### `diff()` {.collection-method}

Phương thức `diff` so sánh collection với một collection khác hoặc một PHP `array` dựa trên các giá trị của nó. Phương thức này sẽ trả về các giá trị trong collection gốc mà không có giá trị trong collection đã cho:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

> **Note**
> Hành vi của phương thức này được thay đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-diff).

<a name="method-diffassoc"></a>
#### `diffAssoc()` {.collection-method}

Phương thức `diffAssoc` so sánh collection với một collection khác hoặc một PHP `array` dựa trên các key và value của nó. Phương thức này sẽ trả về các cặp key và value trong collection gốc không có trong collection đã cho:

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6,
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6,
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>
#### `diffKeys()` {.collection-method}

Phương thức `diffKeys` so sánh collection với một collection khác hoặc một PHP `array` dựa trên các key của nó. Phương thức này sẽ trả về các cặp key và value trong collection gốc không có trong collection đã cho:

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

<a name="method-doesntcontain"></a>
#### `doesntContain()` {.collection-method}

Phương thức `doesntContain` sẽ xác định xem collection sẽ không chứa một item đúng hay không. Bạn có thể truyền một closure cho phương thức `doesntContain` để xác định xem một phần tử mà không tồn tại trong collection và khớp với một kiểm tra đã cho hay không:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->doesntContain(function ($value, $key) {
        return $value < 5;
    });

    // false

Ngoài ra, bạn có thể truyền một chuỗi vào trong phương thức `doesntContain` để xác định xem collection có chứa giá trị item đã cho hay không:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->doesntContain('Table');

    // true

    $collection->doesntContain('Desk');

    // false

Bạn cũng có thể truyền một cặp key và giá trị cho phương thức `doesntContain`, phương thức này sẽ xác định xem cặp key, value đó không tồn tại trong collection hay không:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->doesntContain('product', 'Bookcase');

    // true

Phương thức `doesntContain` sử dụng phép so sánh "lỏng lẻo", nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng với một số integer có cùng giá trị.

<a name="method-dump"></a>
#### `dump()` {.collection-method}

Phương thức `dump` sẽ hiển thị các item của collection:

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

Nếu bạn muốn dừng thực thi lệnh sau khi dump collection, hãy sử dụng phương thức [`dd`](#method-dd) để thay thế.

<a name="method-duplicates"></a>
#### `duplicates()` {.collection-method}

Phương thức `duplicates` sẽ lấy ra và trả về các giá trị trùng lặp từ collection:

    $collection = collect(['a', 'b', 'a', 'c', 'b']);

    $collection->duplicates();

    // [2 => 'a', 4 => 'b']

Nếu collection chứa các mảng hoặc các đối tượng, bạn có thể truyền vào khóa của thuộc tính mà bạn muốn kiểm tra:

    $employees = collect([
        ['email' => 'abigail@example.com', 'position' => 'Developer'],
        ['email' => 'james@example.com', 'position' => 'Designer'],
        ['email' => 'victoria@example.com', 'position' => 'Developer'],
    ]);

    $employees->duplicates('position');

    // [2 => 'Developer']

<a name="method-duplicatesstrict"></a>
#### `duplicatesStrict()` {.collection-method}

Phương thức này có cùng chức năng với phương thức [`duplicates`](#method-duplicates); tuy nhiên, tất cả các giá trị đều được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-each"></a>
#### `each()` {.collection-method}

Phương thức `each` sẽ lặp lại các item trong collection và truyền vào từng item đó một closure:

    $collection->each(function ($item, $key) {
        //
    });

Nếu bạn muốn dừng lặp qua các item, bạn có thể trả về `false` từ closure của bạn:

    $collection->each(function ($item, $key) {
        if (/* condition */) {
            return false;
        }
    });

<a name="method-eachspread"></a>
#### `eachSpread()` {.collection-method}

Phương thức `eachSpread` sẽ lặp lại các item của collection, và truyền vào từng giá trị item lồng nhau đó một hàm callback đã cho:

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

    $collection->eachSpread(function ($name, $age) {
        //
    });

Nếu bạn muốn dừng lặp qua các item còn lại, bạn có thể trả về `false` từ callback của bạn:

    $collection->eachSpread(function ($name, $age) {
        return false;
    });

<a name="method-every"></a>
#### `every()` {.collection-method}

Phương thức `every` có thể được sử dụng để xác minh rằng tất cả các element của một collection có pass qua một số điều kiện đã cho hay không:

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

Nếu một collection là trống, thì phương thức `every` sẽ trả về true:

    $collection = collect([]);

    $collection->every(function ($value, $key) {
        return $value > 2;
    });

    // true

<a name="method-except"></a>
#### `except()` {.collection-method}

Phương thức `except` trả về tất cả các item trong collection ngoại trừ các item có các key được chỉ định:

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

Đối ngược với phương thức `except`, hãy xem phương thức [only](#method-only).

> **Note**
> Hành vi của phương thức này được thay đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-except).

<a name="method-filter"></a>
#### `filter()` {.collection-method}

Phương thức `filter` sẽ lọc các collection bằng cách dùng một callback đã cho, và chỉ giữ lại những item mà đã pass qua một số điều kiện đã cho:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

Nếu không có callback nào được cung cấp, tất cả các item trong collection tương đương với giá trị `false` sẽ bị xóa:

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

Đối ngược với phương thức `filter`, hãy xem phương thức [reject](#method-reject).

<a name="method-first"></a>
#### `first()` {.collection-method}

Phương thức `first` sẽ trả về phần tử đầu tiên có trong collection mà đã pass qua một số điều kiện đã cho:

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

Bạn cũng có thể gọi phương thức `first` mà không có tham số để lấy ra phần tử đầu tiên có trong collection. Nếu collection là trống, thì `null` sẽ được trả về:

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-first-or-fail"></a>
#### `firstOrFail()` {.collection-method}

Phương thức `firstOrFail` giống hệt với phương thức `first`; tuy nhiên, nếu không tìm thấy kết quả nào thì một exception `Illuminate\Support\ItemNotFoundException` sẽ được đưa ra:

    collect([1, 2, 3, 4])->firstOrFail(function ($value, $key) {
        return $value > 5;
    });

    // Throws ItemNotFoundException...

Bạn cũng có thể gọi phương thức `firstOrFail` mà không cần phải truyền tham số vào, để lấy ra phần tử đầu tiên trong collection. Nếu collection trống, thì một exception `Illuminate\Support\ItemNotFoundException` sẽ được đưa ra:

    collect([])->firstOrFail();

    // Throws ItemNotFoundException...

<a name="method-first-where"></a>
#### `firstWhere()` {.collection-method}

Phương thức `firstWhere` sẽ trả về phần tử đầu tiên có trong collection với cặp key value đã cho:

    $collection = collect([
        ['name' => 'Regena', 'age' => null],
        ['name' => 'Linda', 'age' => 14],
        ['name' => 'Diego', 'age' => 23],
        ['name' => 'Linda', 'age' => 84],
    ]);

    $collection->firstWhere('name', 'Linda');

    // ['name' => 'Linda', 'age' => 14]

Bạn cũng có thể gọi phương thức `firstWhere` bằng toán tử so sánh:

    $collection->firstWhere('age', '>=', 18);

    // ['name' => 'Diego', 'age' => 23]

Giống như phương thức [where](#method-where), bạn có thể truyền một tham số cho phương thức `firstWhere`. Trong trường hợp này, phương thức `firstWhere` sẽ trả về item đầu tiên mà có giá trị của khóa đã cho là "truthy":

    $collection->firstWhere('age');

    // ['name' => 'Linda', 'age' => 14]

<a name="method-flatmap"></a>
#### `flatMap()` {.collection-method}

Phương thức `flatMap` lặp qua collection và truyền từng value vào trong một closure đã cho. Closure sẽ sửa đổi value đó và trả về value mới, do đó sẽ tạo nên một collection mới với các value đã được sửa đổi. Sau đó, mảng được trả về sẽ được làm ngang hàng ở một mức:

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
#### `flatten()` {.collection-method}

Phương thức `flatten` sẽ làm phẳng hàng một collection đa chiều thành một chiều duy nhất:

    $collection = collect([
        'name' => 'taylor',
        'languages' => [
            'php', 'javascript'
        ]
    ]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

Nếu cần thiết, bạn có thể truyền một tham số "depth" vào hàm `flatten`:

    $collection = collect([
        'Apple' => [
            [
                'name' => 'iPhone 6S',
                'brand' => 'Apple'
            ],
        ],
        'Samsung' => [
            [
                'name' => 'Galaxy S7',
                'brand' => 'Samsung'
            ],
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

Trong ví dụ trên, nếu gọi `flatten` mà không cung cấp "depth" thì nó cũng sẽ làm phẳng hàng đến cả mảng ở trong và kết quả sẽ là `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Cung cấp một giá trị "depth" sẽ cho phép bạn chỉ định số lượng mà các mảng nằm ở trong sẽ bị làm phẳng.

<a name="method-flip"></a>
#### `flip()` {.collection-method}

Phương thức `flip` sẽ hoán đổi các key của collection với các giá trị value tương ứng của chúng:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {.collection-method}

Phương thức `forget` sẽ xóa một item ra khỏi collection bằng key của nó:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> **Warning**
> Không giống như hầu hết các phương thức collection khác, `forget` không trả về một collection mới; mà nó sẽ sửa trực tiếp lên collection mà nó được gọi.

<a name="method-forpage"></a>
#### `forPage()` {.collection-method}

Phương thức `forPage` sẽ trả về một collection mới chứa các item sẽ xuất hiện trên một số trang nhất định. Phương thức chấp nhận số trang làm tham số đầu tiên và số item sẽ được hiển thị trong mỗi trang làm tham số thứ hai:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {.collection-method}

Phương thức `get` sẽ trả về item tại một key đã cho. Nếu key không tồn tại, `null` được trả về:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

Bạn có thể tùy chọn truyền vào một giá trị mặc định làm tham số thứ hai:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('age', 34);

    // 34

Bạn thậm chí cũng có thể truyền vào một callback như là một giá trị mặc định của phương thức. Và kết quả của callback sẽ được trả về nếu key được chỉ định không tồn tại:

    $collection->get('email', function () {
        return 'taylor@example.com';
    });

    // taylor@example.com

<a name="method-groupby"></a>
#### `groupBy()` {.collection-method}

Phương thức `groupBy` sẽ nhóm các item của collection theo một key đã cho:

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->all();

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

Thay vì truyền vào một chuỗi `key`, bạn có thể truyền vào một callback. Callback sẽ trả về giá trị key mà bạn muốn nhóm bằng cách như sau:

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->all();

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

Nếu bạn có nhiều tiêu chí nhóm, bạn có thể truyền vào dưới dạng một mảng. Mỗi phần tử mảng sẽ được áp dụng cho một level tương ứng trong một mảng nhiều chiều:

    $data = new Collection([
        10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
    ]);

    $result = $data->groupBy(['skill', function ($item) {
        return $item['roles'];
    }], preserveKeys: true);

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
#### `has()` {.collection-method}

Phương thức `has` sẽ xác định nếu một key đã cho có tồn tại trong collection hay không:

    $collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

    $collection->has('product');

    // true

    $collection->has(['product', 'amount']);

    // true

    $collection->has(['amount', 'price']);

    // false

<a name="method-hasany"></a>
#### `hasAny()` {.collection-method}

Phương thức `hasAny` sẽ xác định xem có bất kỳ khóa nào đã tồn tại trong collection hay không:

    $collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

    $collection->hasAny(['product', 'price']);

    // true

    $collection->hasAny(['name', 'price']);

    // false

<a name="method-implode"></a>
#### `implode()` {.collection-method}

Phương thức `implode` là kết hợp các item trong một collection. Tham số của nó phụ thuộc vào loại item trong collection. Nếu collection chứa các mảng hoặc các đối tượng, bạn nên truyền key của các thuộc tính mà bạn muốn join và chuỗi "glue" bạn muốn đặt vào giữa các giá trị:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

Nếu collection chứa nhiều chuỗi hoặc các số đơn giản, bạn nên truyền chuỗi "glue" làm tham số duy nhất cho phương thức:

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

Bạn có thể truyền một closure cho phương thức `implode` nếu bạn muốn định dạng các giá trị được xuất ra:

    $collection->implode(function ($item, $key) {
        return strtoupper($item['product']);
    }, ', ');

    // DESK, CHAIR

<a name="method-intersect"></a>
#### `intersect()` {.collection-method}

Phương thức `intersect` sẽ loại bỏ bất kỳ value nào ra khỏi collection ban đầu nếu không có trong `array` hoặc collection đã cho. Collection kết quả sẽ còn lại các key của collection gốc:

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

> **Note**
> Hành vi của phương thức này được thay đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-intersect).

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {.collection-method}

Phương thức `intersectByKeys` loại bỏ bất kỳ key nào và các giá trị tương ứng của chúng ra khỏi collection ban đầu nếu không có trong `array` hoặc collection đã cho:

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009,
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011,
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {.collection-method}

Phương thức `isEmpty` sẽ trả về` true` nếu collection trống; ngược lại, `false` được trả về:

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {.collection-method}

Phương thức `isNotEmpty` sẽ trả về `true` nếu collection không trống; ngược lại, `false` được trả về:

    collect([])->isNotEmpty();

    // false

<a name="method-join"></a>
#### `join()` {.collection-method}

Phương thức `join` sẽ nối các giá trị của collection với một string. Sử dụng tham số thứ hai của phương thức này, bạn cũng có thể chỉ định cách mà phần tử cuối cùng sẽ được thêm vào chuỗi:

    collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
    collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
    collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
    collect(['a'])->join(', ', ' and '); // 'a'
    collect([])->join(', ', ' and '); // ''

<a name="method-keyby"></a>
#### `keyBy()` {.collection-method}

Phương thức `keyBy` sẽ key hoá collection bằng key đã cho. Nếu nhiều item có cùng key, thì chỉ item cuối cùng sẽ được tạo trong collection mới:

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

Bạn cũng có thể truyền vào một callback cho phương thức. Callback sẽ trả về giá trị cho key collection bằng cách như sau:

    $keyed = $collection->keyBy(function ($item, $key) {
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
#### `keys()` {.collection-method}

Phương thức `keys` sẽ trả về tất cả các key của collection:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {.collection-method}

Phương thức `last` sẽ trả về phần tử cuối cùng có trong collection nếu pass qua một số điều kiện đã cho:

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2

Bạn cũng có thể gọi phương thức `last` không có tham số để lấy phần tử cuối cùng có trong collection. Nếu collection trống, `null` được trả về:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-lazy"></a>
#### `lazy()` {.collection-method}

Phương thức `lazy` sẽ trả về một instance [`LazyCollection`](#lazy-collections) mới từ mảng các mục:

    $lazyCollection = collect([1, 2, 3, 4])->lazy();

    get_class($lazyCollection);

    // Illuminate\Support\LazyCollection

    $lazyCollection->all();

    // [1, 2, 3, 4]

Điều này đặc biệt hữu ích khi bạn cần thực hiện các phép biến đổi trên một `Collection` khổng lồ chứa nhiều mục:

    $count = $hugeCollection
        ->lazy()
        ->where('country', 'FR')
        ->where('balance', '>', '100')
        ->count();

Bằng cách chuyển collection thành `LazyCollection`, chúng ta có thể tránh phải phân bổ thêm quá nhiều bộ nhớ vào collection. Mặc dù collection gốc vẫn giữ các giá trị _its_ trong bộ nhớ nhưng các filter tiếp theo thì không. Do đó, hầu như không có thêm bộ nhớ nào được phân bổ khi lọc kết quả của collection.

<a name="method-macro"></a>
#### `macro()` {.collection-method}

Phương thức tĩnh `macro` cho phép bạn thêm các phương thức mới vào lớp `Collection` trong thời gian chạy. Tham khảo tài liệu về [extending collections](#extending-collections) để biết thêm thông tin chi tiết.

<a name="method-make"></a>
#### `make()` {.collection-method}

Phương thức tĩnh `make` sẽ tạo ra một instance collection mới. Xem phần [Tạo collection](#creating-collections).

<a name="method-map"></a>
#### `map()` {.collection-method}

Phương thức `map` sẽ lặp qua collection và truyền từng item trong collection vào hàm callback đã cho. Hàm callback cho phép bạn có thể sửa đổi các item và trả về item mới, do đó sẽ tạo nên một collection mới với các item đã được sửa:

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> **Warning**
> Giống như hầu hết các phương thức collection khác, `map` trả về một instance collection mới; nó không sửa trực tiếp vào collection mà nó được gọi. Nếu bạn muốn sửa đổi trực tiếp vào collection gốc, hãy sử dụng phương thức [`transform`](#method-transform).

<a name="method-mapinto"></a>
#### `mapInto()` {.collection-method}

Phương thức `mapInto()` sẽ lặp qua collectionp, và tạo một instance mới của một class đã cho bằng cách truyền các giá trị đó vào một hàm khởi tạo:

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
#### `mapSpread()` {.collection-method}

Phương thức `mapSpread` sẽ lặp qua các item của collection, và truyền từng giá trị item bị lồng vào nhau vào hàm closure đã cho. Closure cho phép bạn sửa đổi item và trả về item mới, do đó sẽ tạo thành một collection mới với các item đã được sửa:

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function ($even, $odd) {
        return $even + $odd;
    });

    $sequence->all();

    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>
#### `mapToGroups()` {.collection-method}

Phương thức `mapToGroups` sẽ nhóm các item của collection theo hàm closure đã cho. Closure sẽ trả về một mảng kết hợp có chứa một cặp key giá trị duy nhất, do đó tạo thành một collection gồm các giá trị được nhóm mới:

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

    $grouped->all();

    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johnny Doe'],
        ]
    */

    $grouped->get('Sales')->all();

    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {.collection-method}

Phương thức `mapWithKeys` sẽ lặp qua collection và truyền từng item vào hàm callback đã cho. Hàm callback sẽ trả về một mảng kết hợp có chứa một cặp key value duy nhất:

    $collection = collect([
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
    ]);

    $keyed = $collection->mapWithKeys(function ($item, $key) {
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
#### `max()` {.collection-method}

Phương thức `max` sẽ trả về giá trị lớn nhất của một key đã cho:

    $max = collect([
        ['foo' => 10],
        ['foo' => 20]
    ])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {.collection-method}

Phương thức `median` sẽ trả về [giá trị trung vị](https://vi.wikipedia.org/wiki/S%E1%BB%91_trung_v%E1%BB%8B) của một key đã cho:

    $median = collect([
        ['foo' => 10],
        ['foo' => 10],
        ['foo' => 20],
        ['foo' => 40]
    ])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {.collection-method}

Phương thức `merge` sẽ merge một mảng hoặc một collection đã cho với một collection gốc. Nếu một key trong các item đã cho, khớp với một key trong collection gốc, thì giá trị của item đã cho đó sẽ ghi đè lên giá trị có trong collection gốc:

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

Nếu các key của các item đã cho là số, thì các giá trị sẽ được thêm vào cuối collection:

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-mergerecursive"></a>
#### `mergeRecursive()` {.collection-method}

Phương thức `mergeRecursive` sẽ hợp nhất một mảng hoặc một collection đã cho với một collection ban đầu theo cách đệ quy. Nếu một khóa string trong các item đã cho khớp với khóa string trong collection ban đầu, thì các giá trị cho các khóa này sẽ được hợp nhất với nhau thành một mảng và điều này được thực hiện theo cách đệ quy:

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->mergeRecursive([
        'product_id' => 2,
        'price' => 200,
        'discount' => false
    ]);

    $merged->all();

    // ['product_id' => [1, 2], 'price' => [100, 200], 'discount' => false]

<a name="method-min"></a>
#### `min()` {.collection-method}

Phương thức `min` sẽ trả về giá trị nhỏ nhất của một key đã cho:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {.collection-method}

Phương thức `mode` sẽ trả về [giá trị yếu vị](https://vi.wikipedia.org/wiki/S%E1%BB%91_y%E1%BA%BFu_v%E1%BB%8B) của một key đã cho:

    $mode = collect([
        ['foo' => 10],
        ['foo' => 10],
        ['foo' => 20],
        ['foo' => 40]
    ])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

    $mode = collect([1, 1, 2, 2])->mode();

    // [1, 2]

<a name="method-nth"></a>
#### `nth()` {.collection-method}

Phương thức `nth` sẽ tạo ra một collection mới để chứa các phần tử nằm ở những vị trí (an) với n là khoảng cách muốn lấy của bạn và a là một số nguyên tố tăng dần đều từ 0:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

Bạn có thể truyền vào một phần bù làm tham số thứ hai và công thức sẽ là (an + b) với b là số bù:

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {.collection-method}

Phương thức `only` trả về các item có trong collection với một key được chỉ định:

    $collection = collect([
        'product_id' => 1,
        'name' => 'Desk',
        'price' => 100,
        'discount' => false
    ]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Đối ngược với phương thức `only`, hãy xem phương thức [except](#method-except).

> **Note**
> Hành vi của phương thức này được thay đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-only).

<a name="method-pad"></a>
#### `pad()` {.collection-method}

Phương thức `pad` sẽ thêm vào mảng các giá trị đã cho cho đến khi mảng đạt được kích thước đã chỉ định. Phương thức này hoạt động giống như hàm PHP [array_pad](https://secure.php.net/manual/en/function.array-pad.php).

Để thêm vào bên trái, bạn có thể khai báo một kích thước âm. Và sẽ không thêm nếu giá trị tuyệt đối của kích thước đã cho nhỏ hơn hoặc bằng với độ dài của mảng:

    $collection = collect(['A', 'B', 'C']);

    $filtered = $collection->pad(5, 0);

    $filtered->all();

    // ['A', 'B', 'C', 0, 0]

    $filtered = $collection->pad(-5, 0);

    $filtered->all();

    // [0, 0, 'A', 'B', 'C']

<a name="method-partition"></a>
#### `partition()` {.collection-method}

Phương thức `partition` có thể được kết hợp với mảng của PHP để tách ra các phần tử mà đã pass được qua điều kiện và các phần tử không pass được điều kiện:

    $collection = collect([1, 2, 3, 4, 5, 6]);

    [$underThree, $equalOrAboveThree] = $collection->partition(function ($i) {
        return $i < 3;
    });

    $underThree->all();

    // [1, 2]

    $equalOrAboveThree->all();

    // [3, 4, 5, 6]

<a name="method-pipe"></a>
#### `pipe()` {.collection-method}

Phương thức `pipe` sẽ truyền collection đến một closure đã cho và trả về kết quả chạy của closure đó:

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pipeinto"></a>
#### `pipeInto()` {.collection-method}

Phương thức `pipeInto` sẽ tạo một instance mới của class đã cho và truyền toàn bộ collection vào hàm khởi tạo:

    class ResourceCollection
    {
        /**
         * The Collection instance.
         */
        public $collection;

        /**
         * Create a new ResourceCollection instance.
         *
         * @param  Collection  $collection
         * @return void
         */
        public function __construct(Collection $collection)
        {
            $this->collection = $collection;
        }
    }

    $collection = collect([1, 2, 3]);

    $resource = $collection->pipeInto(ResourceCollection::class);

    $resource->collection->all();

    // [1, 2, 3]

<a name="method-pipethrough"></a>
#### `pipeThrough()` {.collection-method}

Phương thức `pipeThrough` sẽ truyền toàn collection tới một mảng các closure đã cho và trả về kết quả cuối cùng sau khi thông qua các closure đó:

    $collection = collect([1, 2, 3]);

    $result = $collection->pipeThrough([
        function ($collection) {
            return $collection->merge([4, 5]);
        },
        function ($collection) {
            return $collection->sum();
        },
    ]);

    // 15

<a name="method-pluck"></a>
#### `pluck()` {.collection-method}

Phương thức `pluck` sẽ lấy ra tất cả các giá trị của một key đã cho:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

Bạn cũng có thể khai báo thêm key mà bạn muốn dùng từ collection gốc:

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

Phương thức `pluck` cũng hỗ trợ lấy ra các giá trị lồng nhau bằng ký tự "chấm":

    $collection = collect([
        [
            'name' => 'Laracon',
            'speakers' => [
                'first_day' => ['Rosa', 'Judith'],
            ],
        ],
        [
            'name' => 'VueConf',
            'speakers' => [
                'first_day' => ['Abigail', 'Joey'],
            ],
        ],
    ]);

    $plucked = $collection->pluck('speakers.first_day');

    $plucked->all();

    // [['Rosa', 'Judith'], ['Abigail', 'Joey']]

Nếu bị trùng khoá, thì phần tử cuối cùng của khoá đó sẽ được thêm vào collection kết quả:

    $collection = collect([
        ['brand' => 'Tesla',  'color' => 'red'],
        ['brand' => 'Pagani', 'color' => 'white'],
        ['brand' => 'Tesla',  'color' => 'black'],
        ['brand' => 'Pagani', 'color' => 'orange'],
    ]);

    $plucked = $collection->pluck('color', 'brand');

    $plucked->all();

    // ['Tesla' => 'black', 'Pagani' => 'orange']

<a name="method-pop"></a>
#### `pop()` {.collection-method}

Phương thức `pop` sẽ xóa và trả về item cuối cùng từ collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

Bạn có thể truyền một số nguyên cho phương thức `pop` để xóa và trả lại các item từ phần cuối của một collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop(3);

    // collect([5, 4, 3])

    $collection->all();

    // [1, 2]

<a name="method-prepend"></a>
#### `prepend()` {.collection-method}

Phương thức `prepend` sẽ thêm một item vào đầu collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

Bạn cũng có thể truyền vào một tham số thứ hai để chỉ định key của item được prepend:

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {.collection-method}

Phương thức `pull` sẽ loại bỏ và trả về một item từ collectionp bằng key của nó:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {.collection-method}

Phương thức `push` sẽ nối thêm một item vào cuối collection:

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {.collection-method}

Phương thức `put` sẽ set key và giá trị của nó vào trong collection:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {.collection-method}

Phương thức `Random` sẽ trả về một item ngẫu nhiên từ collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

Bạn có thể truyền một số nguyên tố cho hàm `random` để khai báo số lượng item mà bạn muốn lấy ngẫu nhiên. Một collection các item sẽ được trả về khi bạn truyền vào một số nguyên tố mà bạn muốn nhận:

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

Nếu instance collection có ít item hơn yêu cầu, thì phương thức `random` sẽ đưa ra một exception `InvalidArgumentException`.

Phương thức `random` cũng chấp nhận một closure, sẽ nhận vào instance collection hiện tại:

    $random = $collection->random(fn ($items) => min(10, count($items)));

    $random->all();

    // [1, 2, 3, 4, 5] - (retrieved randomly)

<a name="method-range"></a>
#### `range()` {.collection-method}

Phương thức `range` sẽ trả về một collection chứa các số nguyên nằm trong phạm vi đã chỉ định:

    $collection = collect()->range(3, 6);

    $collection->all();

    // [3, 4, 5, 6]

<a name="method-reduce"></a>
#### `reduce()` {.collection-method}

Phương thức `reduce` sẽ biến một collection thành một giá trị duy nhất, và nó sẽ chuyển kết quả của lần lặp trước vào trong lần lặp tiếp theo:

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

Giá trị cho `$carry` trong lần lặp đầu tiên là `null`; tuy nhiên, bạn có thể khai báo giá trị ban đầu của nó bằng cách truyền vào một tham số thứ hai cho `reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

Phương thức `reduce` cũng sẽ truyền một mảng khoá trong collection kết hợp với hàm gọi lại đã cho:

    $collection = collect([
        'usd' => 1400,
        'gbp' => 1200,
        'eur' => 1000,
    ]);

    $ratio = [
        'usd' => 1,
        'gbp' => 1.37,
        'eur' => 1.22,
    ];

    $collection->reduce(function ($carry, $value, $key) use ($ratio) {
        return $carry + ($value * $ratio[$key]);
    });

    // 4264

<a name="method-reduce-spread"></a>
#### `reduceSpread()` {.collection-method}

Phương thức `reduceSpread` sẽ rút gọn collection thành một mảng các giá trị, truyền kết quả của mỗi lần lặp vào lần lặp tiếp theo. Phương thức này tương tự như phương thức `reduce`; tuy nhiên, nó có thể chấp nhận nhiều giá trị khởi tạo:

    [$creditsRemaining, $batch] = Image::where('status', 'unprocessed')
        ->get()
        ->reduceSpread(function ($creditsRemaining, $batch, $image) {
            if ($creditsRemaining >= $image->creditsRequired()) {
                $batch->push($image);

                $creditsRemaining -= $image->creditsRequired();
            }

            return [$creditsRemaining, $batch];
        }, $creditsAvailable, collect());

<a name="method-reject"></a>
#### `reject()` {.collection-method}

Phương thức `reject` sẽ lọc một collection bằng cách sử dụng hàm closure đã cho. Hàm closure sẽ trả về `true` nếu item đó sẽ bị xóa bỏ khỏi collection kết quả:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

Đối ngược với phương thức `reject`, hãy xem phương thức [`filter`](#method-filter).

<a name="method-replace"></a>
#### `replace()` {.collection-method}

Phương thức `replace` hoạt động tương tự như `merge`; tuy nhiên, ngoài việc ghi đè các item có khóa string phù hợp, phương thức `replace` cũng sẽ ghi đè các item trong collection mà có các khóa numeric phù hợp:

    $collection = collect(['Taylor', 'Abigail', 'James']);

    $replaced = $collection->replace([1 => 'Victoria', 3 => 'Finn']);

    $replaced->all();

    // ['Taylor', 'Victoria', 'James', 'Finn']

<a name="method-replacerecursive"></a>
#### `replaceRecursive()` {.collection-method}

Phương thức này hoạt động giống như `replace`, nhưng nó sẽ đệ quy vào trong các mảng và áp dụng quy trình replace tương tự cho các giá trị ở bên trong:

    $collection = collect([
        'Taylor',
        'Abigail',
        [
            'James',
            'Victoria',
            'Finn'
        ]
    ]);

    $replaced = $collection->replaceRecursive([
        'Charlie',
        2 => [1 => 'King']
    ]);

    $replaced->all();

    // ['Charlie', 'Abigail', ['James', 'King', 'Finn']]

<a name="method-reverse"></a>
#### `reverse()` {.collection-method}

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
#### `search()` {.collection-method}

Phương thức `search` sẽ tìm kiếm trong collection với một giá trị đã cho và trả về key của nó nếu được tìm thấy. Nếu item không được tìm thấy, `false` được trả về:

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

Việc tìm kiếm được thực hiện bằng cách sử dụng so sánh "lỏng lẻo", nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng với một số integer có cùng giá trị. Để sử dụng so sánh "nghiêm ngặt", hãy truyền vào một giá trị `true` làm tham số thứ hai cho phương thức:

    collect([2, 4, 6, 8])->search('4', $strict = true);

    // false

Ngoài ra, bạn có thể cung cấp một closure của chính bạn để tìm kiếm item đầu tiên mà thoả mãn một điều kiện cụ thể:

    collect([2, 4, 6, 8])->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {.collection-method}

Phương thức `shift` sẽ loại bỏ và trả về item đầu tiên của collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

Bạn có thể truyền một số nguyên cho phương thức `shift` để xóa đi và trả về nhiều item theo thứ tự đầu của collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift(3);

    // collect([1, 2, 3])

    $collection->all();

    // [4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {.collection-method}

Phương thức `shuffle` sẽ xáo trộn ngẫu nhiên các item trong collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-skip"></a>
#### `skip()` {.collection-method}

Phương thức `skip` sẽ trả về một collection mới, với số lượng phần tử sẽ bị xóa khỏi phần đầu của collection:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $collection = $collection->skip(4);

    $collection->all();

    // [5, 6, 7, 8, 9, 10]

<a name="method-skipuntil"></a>
#### `skipUntil()` {.collection-method}

Phương thức `skipUntil` sẽ bỏ qua các item từ collection cho đến khi lệnh callback trả về giá trị `true` và sau đó nó sẽ trả về các item còn lại có trong collection như một instance collection mới:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipUntil(function ($item) {
        return $item >= 3;
    });

    $subset->all();

    // [3, 4]

Bạn cũng có thể truyền một giá trị đơn giản cho phương thức `skipUntil` để bỏ qua tất cả các item cho đến khi nó tìm thấy giá trị đã cho và trả vể các item còn lại:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipUntil(3);

    $subset->all();

    // [3, 4]

> **Warning**
> Nếu giá trị đã cho không được tìm thấy hoặc lệnh callback không trả về giá trị `true`, thì phương thức `skipUntil` sẽ trả về một collection trống.

<a name="method-skipwhile"></a>
#### `skipWhile()` {.collection-method}

Phương thức `skipWhile` sẽ bỏ qua các item từ collection cho đến khi lệnh callback trả về giá trị `true` và thậm chí cả giá trị true đó, sau đó trả về các item còn lại có trong collection như một instance collection mới:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipWhile(function ($item) {
        return $item <= 3;
    });

    $subset->all();

    // [4]

> **Warning**
> Nếu lệnh callback của bạn không trả về giá trị `false`, thì phương thức `skipWhile` sẽ trả về một collection trống.

<a name="method-slice"></a>
#### `slice()` {.collection-method}

Phương thức `slice` sẽ trả về các phần của collection bắt đầu từ index đã cho:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

Nếu bạn muốn giới hạn kích thước của phần được trả về, hãy truyền một tham số kích thước mong muốn làm tham số thứ hai cho phương thức:

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

Phần được trả lại sẽ mặc định giữ nguyên các key. Nếu bạn không muốn giữ các key gốc, bạn có thể sử dụng phương thức [`values`](#method-values) để reindex lại chúng.

<a name="method-sliding"></a>
#### `sliding()` {.collection-method}

Phương thức `sliding` sẽ trả về một collection với các đoạn mới theo một "sliding window" của các item trong bộ collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunks = $collection->sliding(2);

    $chunks->toArray();

    // [[1, 2], [2, 3], [3, 4], [4, 5]]

Điều này đặc biệt hữu ích khi kết hợp với phương thức [`eachSpread`](#method-eachspread):

    $transactions->sliding(2)->eachSpread(function ($previous, $current) {
        $current->total = $previous->total + $current->amount;
    });

Bạn có thể tùy chọn truyền một giá trị "step" thứ hai vào phương thức và giá trị này sẽ xác định xem khoảng cách giữa các item đầu tiên của mỗi đoạn:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunks = $collection->sliding(3, step: 2);

    $chunks->toArray();

    // [[1, 2, 3], [3, 4, 5]]

<a name="method-sole"></a>
#### `sole()` {.collection-method}

Phương thức `sole` sẽ trả về phần tử đầu tiên trong collection mà đã pass qua một kiểm tra giá trị nhất định, nhưng chỉ khi kiểm tra giá trị đó đúng hoàn toàn với một phần tử đã cho:

    collect([1, 2, 3, 4])->sole(function ($value, $key) {
        return $value === 2;
    });

    // 2

Bạn cũng có thể truyền một cặp key và value cho phương thức `sole`, phương thức này sẽ trả về phần tử đầu tiên trong collection khớp với cặp đã cho, nhưng chỉ khi nó khớp hoàn toàn với một phần tử:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->sole('product', 'Chair');

    // ['product' => 'Chair', 'price' => 100]

Ngoài ra, bạn cũng có thể gọi phương thức `sole` với không tham số truyền vào để lấy ra phần tử đầu tiên trong collection nếu chỉ có một phần tử:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
    ]);

    $collection->sole();

    // ['product' => 'Desk', 'price' => 200]

Nếu không có phần tử nào trong collection cần được trả về bằng phương thức `sole`, thì một exception `\Illuminate\Collections\ItemNotFoundException` sẽ được đưa ra. Nếu có nhiều hơn một phần tử cần được trả về, thì một exception `\Illuminate\Collections\MultipleItemsFoundException` sẽ được đưa ra.

<a name="method-some"></a>
#### `some()` {.collection-method}

Bí danh cho phương thức [`contains`](#method-contains).

<a name="method-sort"></a>
#### `sort()` {.collection-method}

Phương thức `sort` sẽ giúp sắp xếp collection. Collection được sắp xếp sẽ giữ nguyên các key gốc, vì vậy trong ví dụ dưới đây, chúng ta sẽ sử dụng phương thức [`values`](#method-values) để set lại các key thành các index được đánh số theo thứ tự:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

Nếu bạn cần xắp sếp nâng cao hơn, bạn có thể truyền vào một callback tới `sort` bằng một thuật toán của riêng bạn. Tham khảo tài liệu PHP về [`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), chi tiết hơn thì đây là phương thức mà phương thức `sort` của collection sẽ gọi tới trong nội bộ.

> **Note**
> Nếu bạn cần sắp xếp một collection là các mảng hoặc các object lồng nhau, hãy xem thêm các phương thức [`sortBy`](#method-sortby) và [`sortByDesc`](#method-sortbydesc).

<a name="method-sortby"></a>
#### `sortBy()` {.collection-method}

Phương thức `sortBy` sẽ sắp xếp collection theo một key đã cho. Collection đã được sắp xếp sẽ giữ các key gốc, vì vậy trong ví dụ dưới đây, chúng tôi sẽ sử dụng phương thức [`values`](#method-values) để set lại các key thành các chỉ mục được đánh số theo thứ tự:

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

Phương thức `sortBy` chấp nhận vào một [flag sắp xếp](https://www.php.net/manual/en/function.sort.php) làm tham số thứ hai:

    $collection = collect([
        ['title' => 'Item 1'],
        ['title' => 'Item 12'],
        ['title' => 'Item 3'],
    ]);

    $sorted = $collection->sortBy('title', SORT_NATURAL);

    $sorted->values()->all();

    /*
        [
            ['title' => 'Item 1'],
            ['title' => 'Item 3'],
            ['title' => 'Item 12'],
        ]
    */

Ngoài ra, bạn có thể truyền vào một closure của bạn để xác định xem cách sắp xếp các giá trị có trong collection:

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

Nếu bạn muốn sắp xếp collection của bạn theo nhiều thuộc tính, bạn có thể truyền một mảng các thao tác sắp xếp vào phương thức `sortBy`. Mỗi thao tác sắp xếp phải là một mảng chứa các thuộc tính mà bạn muốn sắp xếp và hướng sắp xếp mà bạn mong muốn:

    $collection = collect([
        ['name' => 'Taylor Otwell', 'age' => 34],
        ['name' => 'Abigail Otwell', 'age' => 30],
        ['name' => 'Taylor Otwell', 'age' => 36],
        ['name' => 'Abigail Otwell', 'age' => 32],
    ]);

    $sorted = $collection->sortBy([
        ['name', 'asc'],
        ['age', 'desc'],
    ]);

    $sorted->values()->all();

    /*
        [
            ['name' => 'Abigail Otwell', 'age' => 32],
            ['name' => 'Abigail Otwell', 'age' => 30],
            ['name' => 'Taylor Otwell', 'age' => 36],
            ['name' => 'Taylor Otwell', 'age' => 34],
        ]
    */

Khi sắp xếp một collection theo nhiều thuộc tính, bạn cũng có thể cung cấp các closure để định nghĩa từng thao tác sắp xếp:

    $collection = collect([
        ['name' => 'Taylor Otwell', 'age' => 34],
        ['name' => 'Abigail Otwell', 'age' => 30],
        ['name' => 'Taylor Otwell', 'age' => 36],
        ['name' => 'Abigail Otwell', 'age' => 32],
    ]);

    $sorted = $collection->sortBy([
        fn ($a, $b) => $a['name'] <=> $b['name'],
        fn ($a, $b) => $b['age'] <=> $a['age'],
    ]);

    $sorted->values()->all();

    /*
        [
            ['name' => 'Abigail Otwell', 'age' => 32],
            ['name' => 'Abigail Otwell', 'age' => 30],
            ['name' => 'Taylor Otwell', 'age' => 36],
            ['name' => 'Taylor Otwell', 'age' => 34],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {.collection-method}

Phương thức này có cùng chức năng với phương thức [`sortBy`](#method-sortby), nhưng sẽ sắp xếp collection theo thứ tự ngược lại.

<a name="method-sortdesc"></a>
#### `sortDesc()` {.collection-method}

Phương thức này sẽ sắp xếp collection theo thứ tự ngược lại với phương thức [`sort`](#method-sort):

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sortDesc();

    $sorted->values()->all();

    // [5, 4, 3, 2, 1]

Không giống như `sort`, bạn không thể truyền một lệnh closure đến `sortDesc`. Thay vào đó, bạn có thể sử dụng phương thức [`sort`](#method-sort) và đảo ngược phép so sánh của bạn.

<a name="method-sortkeys"></a>
#### `sortKeys()` {.collection-method}

Phương thức `sortKeys` sẽ sắp xếp một collection theo các khóa của mảng:

    $collection = collect([
        'id' => 22345,
        'first' => 'John',
        'last' => 'Doe',
    ]);

    $sorted = $collection->sortKeys();

    $sorted->all();

    /*
        [
            'first' => 'John',
            'id' => 22345,
            'last' => 'Doe',
        ]
    */

<a name="method-sortkeysdesc"></a>
#### `sortKeysDesc()` {.collection-method}

Phương thức này có cùng dạng với phương thức [`sortKeys`](#method-sortkeys), nhưng nó sẽ sắp xếp collection theo thứ tự ngược lại.

<a name="method-sortkeysusing"></a>
#### `sortKeysUsing()` {.collection-method}

Phương thức `sortKeysUsing` sẽ sắp xếp collection theo khóa của mảng bằng cách sử dụng hàm callback:

    $collection = collect([
        'ID' => 22345,
        'first' => 'John',
        'last' => 'Doe',
    ]);

    $sorted = $collection->sortKeysUsing('strnatcasecmp');

    $sorted->all();

    /*
        [
            'first' => 'John',
            'ID' => 22345,
            'last' => 'Doe',
        ]
    */

Hàm callback phải là một hàm so sánh trả về một số nguyên nhỏ hơn, hoặc bằng hoặc lớn hơn 0. Để biết thêm thông tin, hãy tham khảo tài liệu PHP trên [`uksort`](https://www.php.net/manual/en/function.uksort.php#refsect1-function.uksort-parameters), là hàm PHP được phương thức `sortKeysUsing` sử dụng ở bên trong.

<a name="method-splice"></a>
#### `splice()` {.collection-method}

Phương thức `splice` sẽ loại bỏ và trả về một phần các item bắt đầu từ index được khai báo:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

Bạn có thể truyền vào tham số thứ hai để giới hạn kích thước của collection kết quả:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

Ngoài ra, bạn có thể truyền vào tham số thứ ba chứa các item mới để thay thế các item bị xóa khỏi collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {.collection-method}

Phương thức `split` sẽ chia một collection thành một số nhóm:

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->all();

    // [[1, 2], [3, 4], [5]]

<a name="method-splitin"></a>
#### `splitIn()` {.collection-method}

Phương thức `splitIn` sẽ chia một collection thành một số nhóm nhất định, lấp đầy các nhóm ở đầu trước, trước khi phân bổ phần các element còn lại cho nhóm cuối cùng:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $groups = $collection->splitIn(3);

    $groups->all();

    // [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10]]

<a name="method-sum"></a>
#### `sum()` {.collection-method}

Phương thức `sum` sẽ trả về tổng của tất cả các item trong collection:

    collect([1, 2, 3, 4, 5])->sum();

    // 15

Nếu collection chứa các mảng hoặc các đối tượng lồng nhau, bạn có thể truyền vào một key sẽ được sử dụng để xác định giá trị nào sẽ cần tính tổng:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 127

Ngoài ra, bạn có thể truyền vào một closure của chính bạn để xác định giá trị nào của collection cần tính tổng:

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
#### `take()` {.collection-method}

Phương thức `Take` sẽ trả về một collection mới với một số lượng item được chỉ định:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

Bạn cũng có thể truyền vào một số âm để lấy số lượng item được chỉ định từ cuối collection trở về:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-takeuntil"></a>
#### `takeUntil()` {.collection-method}

Phương thức `takeUntil` sẽ trả về các item có trong collection cho đến khi lệnh callback của bạn trả về giá trị `true`:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeUntil(function ($item) {
        return $item >= 3;
    });

    $subset->all();

    // [1, 2]

Bạn cũng có thể truyền vào một giá trị đơn giản cho phương thức `takeUntil` để lấy ra các item cho đến khi tìm thấy giá trị đã cho:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeUntil(3);

    $subset->all();

    // [1, 2]

> **Warning**
> Nếu giá trị đã cho không được tìm thấy hoặc lệnh callback không trả về giá trị `true`, thì phương thức `takeUntil` sẽ trả về một collection trống.

<a name="method-takewhile"></a>
#### `takeWhile()` {.collection-method}

Phương thức `takeWhile` sẽ trả về các item có trong collection cho đến khi lệnh callback của bạn trả về `false`:

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeWhile(function ($item) {
        return $item < 3;
    });

    $subset->all();

    // [1, 2]

> **Warning**
> Nếu lệnh callback của bạn không trả về `false`, thì phương thức `takeWhile` sẽ trả về tất cả các item có trong collection đó.

<a name="method-tap"></a>
#### `tap()` {.collection-method}

Phương thức `tap` sẽ truyền collection đến một callback đã cho, cho phép bạn "tap" vào collection tại một điểm cụ thể và làm một cái gì đó với các item trong khi không ảnh hưởng đến chính collection. Sau đó, collection sẽ được trả về bằng phương thức `tap`:

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->all());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` {.collection-method}

Phương thức tĩnh `times` sẽ tạo ra một collection mới bằng cách gọi hàm closure đã cho với một số lần đã được chỉ định:

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

<a name="method-toarray"></a>
#### `toArray()` {.collection-method}

Phương thức `toArray` sẽ chuyển đổi collection thành một PHP `array`. Nếu các giá trị của collection là các model [Eloquent](/docs/{{version}}/eloquent), thì các mdoel này cũng sẽ được chuyển đổi thành mảng:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> **Warning**
> `toArray` cũng sẽ chuyển đổi tất cả các đối tượng `Arrayable` có trong collection thành một mảng kể cả các đối tượng nằm sâu bên trong mảng. Nếu bạn muốn lấy một mảng thô của collection, bạn có thể sử dụng phương thức [`all`](#method-all).

<a name="method-tojson"></a>
#### `toJson()` {.collection-method}

Phương thức `toJson` sẽ chuyển đổi một collection thành một chuỗi JSON:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {.collection-method}

Phương thức `Transform` sẽ lặp collection và gọi hàm callback đã cho với từng item có trong collection. Các item có trong collection sẽ được thay thế bằng một giá trị mới được trả về bởi hàm callback:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> **Warning**
> Không giống như hầu hết các phương thức collection khác, `transform` sẽ trực tiếp sửa vào collection. Nếu bạn muốn tạo một collection mới, hãy sử dụng phương thức [`map`](#method-map).

<a name="method-undot"></a>
#### `undot()` {.collection-method}

Phương thức `undot` sẽ thay đổi collection một chiều sử dụng ký hiệu "chấm" thành một collection đa chiều:

    $person = collect([
        'name.first_name' => 'Marie',
        'name.last_name' => 'Valentine',
        'address.line_1' => '2992 Eagle Drive',
        'address.line_2' => '',
        'address.suburb' => 'Detroit',
        'address.state' => 'MI',
        'address.postcode' => '48219'
    ]);

    $person = $person->undot();

    $person->toArray();

    /*
        [
            "name" => [
                "first_name" => "Marie",
                "last_name" => "Valentine",
            ],
            "address" => [
                "line_1" => "2992 Eagle Drive",
                "line_2" => "",
                "suburb" => "Detroit",
                "state" => "MI",
                "postcode" => "48219",
            ],
        ]
    */

<a name="method-union"></a>
#### `union()` {.collection-method}

Phương thức `union` sẽ thêm một mảng đã cho vào collection. Nếu mảng đã cho có chứa các khóa đã có trong collection gốc, thì các giá trị của collection gốc sẽ được ưu tiên:

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['d']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {.collection-method}

Phương thức `unique` sẽ trả về tất cả các item duy nhất có trong collection. Collection được trả về sẽ giữ nguyên các key gốc, vì vậy trong ví dụ dưới đây, chúng ta sẽ sử dụng phương thức [`values`](#method-values) để set lại các key gốc đó thành các index được đánh số theo thứ tự:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

Khi xử lý các mảng hoặc các đối tượng lồng nhau, bạn có thể chỉ định key mà được sử dụng để xác định tính duy nhất:

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

Cuối cùng, bạn cũng có thể truyền một closure của bạn cho phương thức `unique` để chỉ định xem giá trị nào sẽ được định nghĩa là tính duy nhất của một item:

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

Phương thức `unique` sử dụng các phép so sánh "lỏng lẻo" khi kiểm tra các giá trị item, nghĩa là một chuỗi có giá trị integer sẽ được coi là bằng với một số integer có cùng giá trị. Sử dụng phương thức [`uniqueStrict`](#method-uniquestrict) để lọc bằng các so sánh "nghiêm ngặt".

> **Note**
> Hành vi của phương thức này được thay đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-unique).

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {.collection-method}

Phương thức này có cùng chức năng với phương thức [`unique`](#method-unique); tuy nhiên, tất cả các giá trị được so sánh sẽ sử dụng so sánh "nghiêm ngặt".

<a name="method-unless"></a>
#### `unless()` {.collection-method}

Phương thức `unless` sẽ chạy hàm callback đã cho nếu như tham số đầu tiên được cung cấp cho phương thức này là khác `true`:

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->unless(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

Callback thứ hai có thể được truyền đến phương thức `unless`. Callback thứ hai này sẽ được thực thi khi tham số đầu tiên được cung cấp cho phương thức `unless` trả về giá trị là `true`:

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    }, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

Đối ngược với phương thức `unless`, hãy xem phương thức [`when`](#method-when).

<a name="method-unlessempty"></a>
#### `unlessEmpty()` {.collection-method}

Bí danh cho phương thức [`whenNotEmpty`](#method-whennotempty).

<a name="method-unlessnotempty"></a>
#### `unlessNotEmpty()` {.collection-method}

Bí danh cho phương thức [`whenEmpty`](#method-whenempty).

<a name="method-unwrap"></a>
#### `unwrap()` {.collection-method}

Phương thức tĩnh `unwrap` sẽ trả về các item mà không được bao bọc trong collection:

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-value"></a>
#### `value()` {.collection-method}

Phương thức `value` sẽ lấy ra một giá trị nhất định từ phần tử đầu tiên của collection:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Speaker', 'price' => 400],
    ]);

    $value = $collection->value('price');

    // 200

<a name="method-values"></a>
#### `values()` {.collection-method}

Phương thức `value` trả về một collection mới với các key đã được sắp xếp theo thứ tự:

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200],
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
#### `when()` {.collection-method}

Phương thức `when` sẽ chạy callback đã cho khi mà tham số đầu tiên trả về giá trị `true`. Instance collection và tham số đầu tiên được cung cấp cho phương thức `when` sẽ được cung cấp cho closure:

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection, $value) {
        return $collection->push(4);
    });

    $collection->when(false, function ($collection, $value) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 4]

Callback thứ hai có thể được truyền đến phương thức `when`. Callback thứ hai này sẽ được thực thi khi tham số đầu tiên được cung cấp cho phương thức `when` trả về giá trị là `false`:

    $collection = collect([1, 2, 3]);

    $collection->when(false, function ($collection, $value) {
        return $collection->push(4);
    }, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

Đối ngược với phương thức `when`, hãy xem phương thức [`unless`](#method-unless).

<a name="method-whenempty"></a>
#### `whenEmpty()` {.collection-method}

Phương thức `whenEmpty` sẽ thực hiện lệnh callback đã cho khi collection là trống:

    $collection = collect(['Michael', 'Tom']);

    $collection->whenEmpty(function ($collection) {
        return $collection->push('Adam');
    });

    $collection->all();

    // ['Michael', 'Tom']


    $collection = collect();

    $collection->whenEmpty(function ($collection) {
        return $collection->push('Adam');
    });

    $collection->all();

    // ['Adam']

Closure thứ hai có thể được truyền đến phương thức `whenEmpty` và sẽ được thực thi khi collection không trống:

    $collection = collect(['Michael', 'Tom']);

    $collection->whenEmpty(function ($collection) {
        return $collection->push('Adam');
    }, function ($collection) {
        return $collection->push('Taylor');
    });

    $collection->all();

    // ['Michael', 'Tom', 'Taylor']

Đối ngược với phương thức `whenEmpty`, hãy xem phương thức [`whenNotEmpty`](#method-whennotempty).

<a name="method-whennotempty"></a>
#### `whenNotEmpty()` {.collection-method}

Phương thức `whenNotEmpty` sẽ thực hiện lệnh callback đã cho khi collection không trống:

    $collection = collect(['michael', 'tom']);

    $collection->whenNotEmpty(function ($collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // ['michael', 'tom', 'adam']


    $collection = collect();

    $collection->whenNotEmpty(function ($collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // []

Closure thứ hai có thể được truyền đến phương thức `whenNotEmpty` và sẽ được thực thi khi collection trống:

    $collection = collect();

    $collection->whenNotEmpty(function ($collection) {
        return $collection->push('adam');
    }, function ($collection) {
        return $collection->push('taylor');
    });

    $collection->all();

    // ['taylor']

Đối ngược với phương thức `whenNotEmpty`, hãy xem phương thức [`whenEmpty`](#method-whenempty).

<a name="method-where"></a>
#### `where()` {.collection-method}

Phương thức `where` sẽ lọc collection theo giá trị của cặp key và value:

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

Phương thức `where` sẽ sử dụng phép so sánh "lỏng lẻo" khi kiểm tra các giá trị của item, nghĩa là một chuỗi có giá trị integer sẽ bằng với một số integer có cùng giá trị. Sử dụng phương thức [`whereStrict`](#method-wherestrict) để lọc collection bằng các so sánh "nghiêm ngặt".

Bạn có thể tùy chọn truyền thêm vào một toán tử so sánh làm tham số thứ hai cho phương thức. Các toán tử được hỗ trợ là: '===', '!==', '!=', '==', '=', '<>', '>', '<', '>=', và '<=':

    $collection = collect([
        ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
        ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
        ['name' => 'Sue', 'deleted_at' => null],
    ]);

    $filtered = $collection->where('deleted_at', '!=', null);

    $filtered->all();

    /*
        [
            ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
            ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
        ]
    */

<a name="method-wherestrict"></a>
#### `whereStrict()` {.collection-method}

Phương thức này có cùng chức năng với phương thức [`where`](#method-where); tuy nhiên, tất cả các giá trị đều được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-wherebetween"></a>
#### `whereBetween()` {.collection-method}

Phương thức `whereBetween` sẽ lọc collection bằng cách xác định xem một item nào đó nằm trong một phạm vi nhất định:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Pencil', 'price' => 30],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereBetween('price', [100, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Desk', 'price' => 200],
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Door', 'price' => 100],
        ]
    */

<a name="method-wherein"></a>
#### `whereIn()` {.collection-method}

Phương thức `whereIn` sẽ loại bỏ tất cả các phần tử khỏi collection nếu giá trị không có trong trong mảng đã cho:

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
            ['product' => 'Desk', 'price' => 200],
            ['product' => 'Bookcase', 'price' => 150],
        ]
    */

Phương thức `whereIn` sử dụng phép so sánh "lỏng lẻo" khi kiểm tra các giá trị của item, nghĩa là một chuỗi có giá trị integer sẽ bằng với một số integer có cùng giá trị. Sử dụng phương thức [`whereInStrict`](#method-whereinstrict) để lọc collection bằng các so sánh "nghiêm ngặt".

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {.collection-method}

Phương thức này có cùng chức năng với phương thức [`whereIn`](#method-wherein); tuy nhiên, tất cả các giá trị đều được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-whereinstanceof"></a>
#### `whereInstanceOf()` {.collection-method}

Phương thức `whereInstanceOf` sẽ lọc collection theo một loại class nhất định:

    use App\Models\User;
    use App\Models\Post;

    $collection = collect([
        new User,
        new User,
        new Post,
    ]);

    $filtered = $collection->whereInstanceOf(User::class);

    $filtered->all();

    // [App\Models\User, App\Models\User]

<a name="method-wherenotbetween"></a>
#### `whereNotBetween()` {.collection-method}

Phương thức `whereBetween` sẽ lọc collection bằng cách xác định xem một item nào đó nằm ngoài một phạm vi nhất định:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Pencil', 'price' => 30],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotBetween('price', [100, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 80],
            ['product' => 'Pencil', 'price' => 30],
        ]
    */

<a name="method-wherenotin"></a>
#### `whereNotIn()` {.collection-method}

Phương thức `whereIn` sẽ loại bỏ tất cả các phần tử khỏi collection nếu giá trị có trong trong mảng đã cho:

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

Phương thức `whereNotIn` sử dụng phép so sánh "lỏng lẻo" khi kiểm tra các giá trị item, nghĩa là một chuỗi có giá trị integer sẽ bằng với một số integer có cùng giá trị. Sử dụng phương thức [`whereNotInStrict`](#method-wherenotinstrict) để lọc collection bằng các so sánh "nghiêm ngặt".

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {.collection-method}

Phương thức này có cùng chức năng với phương thức [`whereNotIn`](#method-wherenotin); tuy nhiên, tất cả các giá trị đều được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-wherenotnull"></a>
#### `whereNotNull()` {.collection-method}

Phương thức `whereNotNull` sẽ trả về các item từ collection với một khóa đã cho không phải là `null`:

    $collection = collect([
        ['name' => 'Desk'],
        ['name' => null],
        ['name' => 'Bookcase'],
    ]);

    $filtered = $collection->whereNotNull('name');

    $filtered->all();

    /*
        [
            ['name' => 'Desk'],
            ['name' => 'Bookcase'],
        ]
    */

<a name="method-wherenull"></a>
#### `whereNull()` {.collection-method}

Phương thức `whereNull` sẽ trả về các item từ collection với một khóa đã cho phải là `null`:

    $collection = collect([
        ['name' => 'Desk'],
        ['name' => null],
        ['name' => 'Bookcase'],
    ]);

    $filtered = $collection->whereNull('name');

    $filtered->all();

    /*
        [
            ['name' => null],
        ]
    */


<a name="method-wrap"></a>
#### `wrap()` {.collection-method}

Phương thức tĩnh `wrap` sẽ bao bọc giá trị đã cho trong một collection khi được áp dụng:

    use Illuminate\Support\Collection;

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
#### `zip()` {.collection-method}

Phương thức `zip` sẽ nối các giá trị của mảng đã cho với các giá trị của collection tại index tương ứng của chúng:

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## Higher Order Messages

Collection cũng cung cấp hỗ trợ cho "higher order messages", đó là các cách rút gọn để thực hiện các hành động phổ biến có trên các collection. Các phương thức collection cung cấp các higher order message như sau: [`average`](#method-average), [`avg`](#method-avg), [`contains`](#method-contains), [`each`](#method-each), [`every`](#method-every), [`filter`](#method-filter), [`first`](#method-first), [`flatMap`](#method-flatmap), [`groupBy`](#method-groupby), [`keyBy`](#method-keyby), [`map`](#method-map), [`max`](#method-max), [`min`](#method-min), [`partition`](#method-partition), [`reject`](#method-reject), [`skipUntil`](#method-skipuntil), [`skipWhile`](#method-skipwhile), [`some`](#method-some), [`sortBy`](#method-sortby), [`sortByDesc`](#method-sortbydesc), [`sum`](#method-sum), [`takeUntil`](#method-takeuntil), [`takeWhile`](#method-takewhile), và [`unique`](#method-unique).

Mỗi higher order message có thể được truy cập giống như một thuộc tính động có trên một instance của collection. Chẳng hạn, hãy sử dụng higher order message `each` để gọi một phương thức ở trên mỗi đối tượng có trong một collection:

    use App\Models\User;

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

Tương tự, chúng ta có thể sử dụng higher order message `sum` để tính tổng số "votes" cho một collection user:

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;

<a name="lazy-collections"></a>
## Lazy Collections

<a name="lazy-collection-introduction"></a>
### Giới thiệu

> **Warning**
> Trước khi tìm hiểu thêm về lazy collection của Laravel, hãy dành chút thời gian để làm quen với [PHP generators](https://www.php.net/manual/en/language.generators.overview.php).

Để bổ sung cho class `Collection` vốn đã mạnh mẽ, class `LazyCollection` sử dụng [generators](https://www.php.net/manual/en/language.generators.overview.php) của PHP để cho phép bạn làm việc với bộ dữ liệu rất lớn trong khi vẫn giữ mức sử dụng bộ nhớ thấp.

Ví dụ: hãy tưởng tượng ứng dụng của bạn cần xử lý file log nhiều gigabyte trong khi tận dụng các phương thức của Laravel để phân tích cú pháp log. Thay vì đọc toàn bộ file vào bộ nhớ cùng một lúc, lazy collection có thể được sử dụng để chỉ giữ một phần nhỏ của file vào trong bộ nhớ tại một thời điểm nhất định:

    use App\Models\LogEntry;
    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    })->chunk(4)->map(function ($lines) {
        return LogEntry::fromLines($lines);
    })->each(function (LogEntry $logEntry) {
        // Process the log entry...
    });

Hoặc, hãy tưởng tượng bạn cần lặp 10.000 model Eloquent. Khi sử dụng collection truyền thống của Laravel, tất cả 10.000 model Eloquent sẽ được load vào trong bộ nhớ cùng một lúc:

    use App\Models\User;

    $users = User::all()->filter(function ($user) {
        return $user->id > 500;
    });

Tuy nhiên, phương thức `cursor` của query builder sẽ trả về một instance `LazyCollection`. Điều này cho phép bạn vẫn chạy một truy vấn duy nhất đối với cơ sở dữ liệu nhưng cũng chỉ giữ một model Eloquent được load vào trong bộ nhớ tại một thời điểm. Trong ví dụ này, lệnh callback `filter` không được thực thi cho đến khi chúng ta thực sự lặp từng user, cho phép giảm đáng kể mức sử dụng bộ nhớ:

    use App\Models\User;

    $users = User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

<a name="creating-lazy-collections"></a>
### Tạo Lazy Collections

Để tạo một instance lazy collection, bạn nên truyền một hàm của generator PHP vào phương thức `make` của collection:

    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    });

<a name="the-enumerable-contract"></a>
### The Enumerable Contract

Hầu như tất cả các phương thức có sẵn trên class `Collection` cũng có sẵn trên class `LazyCollection`. Cả hai class này đều implement contract `Illuminate\Support\Enumerable` định nghĩa các phương thức sau:

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

<div class="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstOrFail](#method-first-or-fail)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
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
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[shuffle](#method-shuffle)
[skip](#method-skip)
[slice](#method-slice)
[sole](#method-sole)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

> **Warning**
> Các phương thức làm thay đổi collection (chẳng hạn như `shift`,` pop`, `prepend`, vv.) **không** có sẵn trên class `LazyCollection`.

<a name="lazy-collection-methods"></a>
### Các phương thức của Lazy Collection

Ngoài các phương thức được định nghĩa trong contract `Enumerable`, class `LazyCollection` cũng chứa thêm các phương thức sau:

<a name="method-takeUntilTimeout"></a>
#### `takeUntilTimeout()` {.collection-method}

Phương thức `takeUntilTimeout` sẽ trả về một lazy collection mới và sẽ thực hiện các giá trị cho đến thời điểm đã chỉ định. Sau thời gian đó, collection sẽ ngừng thực hiện:

    $lazyCollection = LazyCollection::times(INF)
        ->takeUntilTimeout(now()->addMinute());

    $lazyCollection->each(function ($number) {
        dump($number);

        sleep(1);
    });

    // 1
    // 2
    // ...
    // 58
    // 59

Để minh họa cách sử dụng phương thức này, hãy tưởng tượng một ứng dụng gửi hóa đơn từ cơ sở dữ liệu bằng cursor. Bạn có thể định nghĩa một [scheduled task](/docs/{{version}}/scheduling) chạy 15 phút một lần và chỉ xử lý hóa đơn tối đa trong 14 phút:

    use App\Models\Invoice;
    use Illuminate\Support\Carbon;

    Invoice::pending()->cursor()
        ->takeUntilTimeout(
            Carbon::createFromTimestamp(LARAVEL_START)->add(14, 'minutes')
        )
        ->each(fn ($invoice) => $invoice->submit());

<a name="method-tapEach"></a>
#### `tapEach()` {.collection-method}

Trong khi phương thức `each` gọi lệnh callback đã cho cho từng item có trong collection ngay lập tức, thì phương thức` tapEach` chỉ gọi lệnh callback đã cho cho một item được lấy ra khỏi danh sách:

    // Nothing has been dumped so far...
    $lazyCollection = LazyCollection::times(INF)->tapEach(function ($value) {
        dump($value);
    });

    // Three items are dumped...
    $array = $lazyCollection->take(3)->all();

    // 1
    // 2
    // 3

<a name="method-remember"></a>
#### `remember()` {.collection-method}

Phương thức `remember` sẽ trả về một lazy collection mới sẽ remember bất kỳ giá trị nào đã được lấy ra rồi và sẽ không lấy lại các giá trị đó khi thực hiện lệnh kế tiếp của collection:

    // No query has been executed yet...
    $users = User::cursor()->remember();

    // The query is executed...
    // The first 5 users are hydrated from the database...
    $users->take(5)->all();

    // First 5 users come from the collection's cache...
    // The rest are hydrated from the database...
    $users->take(20)->all();
