# Helpers

- [Giới thiệu](#introduction)
- [Các phương thức có sẵn](#available-methods)
- [Các class hữu ích khác](#other-utilities)
    - [Benchmarking](#benchmarking)
    - [Lottery](#lottery)

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
[Arr::toCssClasses](#method-array-to-css-classes)
[Arr::undot](#method-array-undot)
[Arr::where](#method-array-where)
[Arr::whereNotNull](#method-array-where-not-null)
[Arr::wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[head](#method-head)
[last](#method-last)
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

<a name="strings-method-list"></a>
### Strings

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[class_basename](#method-class-basename)
[e](#method-e)
[preg_replace_array](#method-preg-replace-array)
[Str::after](#method-str-after)
[Str::afterLast](#method-str-after-last)
[Str::ascii](#method-str-ascii)
[Str::before](#method-str-before)
[Str::beforeLast](#method-str-before-last)
[Str::between](#method-str-between)
[Str::betweenFirst](#method-str-between-first)
[Str::camel](#method-camel-case)
[Str::contains](#method-str-contains)
[Str::containsAll](#method-str-contains-all)
[Str::endsWith](#method-ends-with)
[Str::excerpt](#method-excerpt)
[Str::finish](#method-str-finish)
[Str::headline](#method-str-headline)
[Str::inlineMarkdown](#method-str-inline-markdown)
[Str::is](#method-str-is)
[Str::isAscii](#method-str-is-ascii)
[Str::isJson](#method-str-is-json)
[Str::isUlid](#method-str-is-ulid)
[Str::isUuid](#method-str-is-uuid)
[Str::kebab](#method-kebab-case)
[Str::lcfirst](#method-str-lcfirst)
[Str::length](#method-str-length)
[Str::limit](#method-str-limit)
[Str::lower](#method-str-lower)
[Str::markdown](#method-str-markdown)
[Str::mask](#method-str-mask)
[Str::orderedUuid](#method-str-ordered-uuid)
[Str::padBoth](#method-str-padboth)
[Str::padLeft](#method-str-padleft)
[Str::padRight](#method-str-padright)
[Str::plural](#method-str-plural)
[Str::pluralStudly](#method-str-plural-studly)
[Str::random](#method-str-random)
[Str::remove](#method-str-remove)
[Str::replace](#method-str-replace)
[Str::replaceArray](#method-str-replace-array)
[Str::replaceFirst](#method-str-replace-first)
[Str::replaceLast](#method-str-replace-last)
[Str::reverse](#method-str-reverse)
[Str::singular](#method-str-singular)
[Str::slug](#method-str-slug)
[Str::snake](#method-snake-case)
[Str::squish](#method-str-squish)
[Str::start](#method-str-start)
[Str::startsWith](#method-starts-with)
[Str::studly](#method-studly-case)
[Str::substr](#method-str-substr)
[Str::substrCount](#method-str-substrcount)
[Str::substrReplace](#method-str-substrreplace)
[Str::swap](#method-str-swap)
[Str::title](#method-title-case)
[Str::toHtmlString](#method-str-to-html-string)
[Str::ucfirst](#method-str-ucfirst)
[Str::ucsplit](#method-str-ucsplit)
[Str::upper](#method-str-upper)
[Str::ulid](#method-str-ulid)
[Str::uuid](#method-str-uuid)
[Str::wordCount](#method-str-word-count)
[Str::words](#method-str-words)
[str](#method-str)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

<a name="fluent-strings-method-list"></a>
### Fluent Strings

<div class="collection-method-list" markdown="1">

[after](#method-fluent-str-after)
[afterLast](#method-fluent-str-after-last)
[append](#method-fluent-str-append)
[ascii](#method-fluent-str-ascii)
[basename](#method-fluent-str-basename)
[before](#method-fluent-str-before)
[beforeLast](#method-fluent-str-before-last)
[between](#method-fluent-str-between)
[betweenFirst](#method-fluent-str-between-first)
[camel](#method-fluent-str-camel)
[classBasename](#method-fluent-str-class-basename)
[contains](#method-fluent-str-contains)
[containsAll](#method-fluent-str-contains-all)
[dirname](#method-fluent-str-dirname)
[endsWith](#method-fluent-str-ends-with)
[excerpt](#method-fluent-str-excerpt)
[exactly](#method-fluent-str-exactly)
[explode](#method-fluent-str-explode)
[finish](#method-fluent-str-finish)
[headline](#method-fluent-str-headline)
[inlineMarkdown](#method-fluent-str-inline-markdown)
[is](#method-fluent-str-is)
[isAscii](#method-fluent-str-is-ascii)
[isEmpty](#method-fluent-str-is-empty)
[isNotEmpty](#method-fluent-str-is-not-empty)
[isJson](#method-fluent-str-is-json)
[isUlid](#method-fluent-str-is-ulid)
[isUuid](#method-fluent-str-is-uuid)
[kebab](#method-fluent-str-kebab)
[lcfirst](#method-fluent-str-lcfirst)
[length](#method-fluent-str-length)
[limit](#method-fluent-str-limit)
[lower](#method-fluent-str-lower)
[ltrim](#method-fluent-str-ltrim)
[markdown](#method-fluent-str-markdown)
[mask](#method-fluent-str-mask)
[match](#method-fluent-str-match)
[matchAll](#method-fluent-str-match-all)
[newLine](#method-fluent-str-new-line)
[padBoth](#method-fluent-str-padboth)
[padLeft](#method-fluent-str-padleft)
[padRight](#method-fluent-str-padright)
[pipe](#method-fluent-str-pipe)
[plural](#method-fluent-str-plural)
[prepend](#method-fluent-str-prepend)
[remove](#method-fluent-str-remove)
[replace](#method-fluent-str-replace)
[replaceArray](#method-fluent-str-replace-array)
[replaceFirst](#method-fluent-str-replace-first)
[replaceLast](#method-fluent-str-replace-last)
[replaceMatches](#method-fluent-str-replace-matches)
[rtrim](#method-fluent-str-rtrim)
[scan](#method-fluent-str-scan)
[singular](#method-fluent-str-singular)
[slug](#method-fluent-str-slug)
[snake](#method-fluent-str-snake)
[split](#method-fluent-str-split)
[squish](#method-fluent-str-squish)
[start](#method-fluent-str-start)
[startsWith](#method-fluent-str-starts-with)
[studly](#method-fluent-str-studly)
[substr](#method-fluent-str-substr)
[substrReplace](#method-fluent-str-substrreplace)
[swap](#method-fluent-str-swap)
[tap](#method-fluent-str-tap)
[test](#method-fluent-str-test)
[title](#method-fluent-str-title)
[trim](#method-fluent-str-trim)
[ucfirst](#method-fluent-str-ucfirst)
[ucsplit](#method-fluent-str-ucsplit)
[upper](#method-fluent-str-upper)
[when](#method-fluent-str-when)
[whenContains](#method-fluent-str-when-contains)
[whenContainsAll](#method-fluent-str-when-contains-all)
[whenEmpty](#method-fluent-str-when-empty)
[whenNotEmpty](#method-fluent-str-when-not-empty)
[whenStartsWith](#method-fluent-str-when-starts-with)
[whenEndsWith](#method-fluent-str-when-ends-with)
[whenExactly](#method-fluent-str-when-exactly)
[whenNotExactly](#method-fluent-str-when-not-exactly)
[whenIs](#method-fluent-str-when-is)
[whenIsAscii](#method-fluent-str-when-is-ascii)
[whenIsUlid](#method-fluent-str-when-is-ulid)
[whenIsUuid](#method-fluent-str-when-is-uuid)
[whenTest](#method-fluent-str-when-test)
[wordCount](#method-fluent-str-word-count)
[words](#method-fluent-str-words)

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

    $first = Arr::first($array, function ($value, $key) {
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

    $last = Arr::last($array, function ($value, $key) {
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

    $mapped = Arr::map($array, function ($value, $key) {
        return ucfirst($value);
    });

    // ['first' => 'James', 'last' => 'Kirk']

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

    $sorted = array_values(Arr::sortDesc($array, function ($value) {
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

    $filtered = Arr::where($array, function ($value, $key) {
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

<a name="paths"></a>
## Paths

<a name="method-app-path"></a>
#### `app_path()` {.collection-method}

Hàm `app_path` trả về đường dẫn đến thư mục `app` của application của bạn. Bạn cũng có thể sử dụng hàm `app_path` để tạo đường dẫn đến một file có bắt đầu từ thư mục app:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {.collection-method}

Hàm `base_path` trả về đường dẫn đến thư mục root  của application của bạn. Bạn cũng có thể sử dụng hàm `base_path` để tạo đường dẫn đến một file đã cho có bắt đầu từ thư mục gốc của dự án:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {.collection-method}

Hàm `config_path` trả về đường dẫn đến thư mục `config` của application của bạn. Bạn cũng có thể sử dụng hàm `config_path` để tạo đường dẫn đến một file đã cho trong thư mục config của application:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {.collection-method}

Hàm `database_path` trả về đường dẫn đến thư mục `database` của application của bạn. Bạn cũng có thể sử dụng hàm `database_path` để tạo đường dẫn đến một file đã cho trong thư mục database:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-lang-path"></a>
#### `lang_path()` {.collection-method}

Hàm `lang_path` sẽ trả về một đường dẫn đầy đủ tới thư mục `lang` của ứng dụng của bạn. Bạn cũng có thể sử dụng hàm `lang_path` để một tạo đường dẫn đầy đủ đến một file nhất định trong thư mục:

    $path = lang_path();

    $path = lang_path('en/messages.php');

<a name="method-mix"></a>
#### `mix()` {.collection-method}

Hàm `mix` trả về đường dẫn đến [file Mix đã được version](/docs/{{version}}/mix):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` {.collection-method}

Hàm `public_path` trả về đường dẫn đến thư mục `public` của application của bạn. Bạn cũng có thể sử dụng hàm `public_path` để tạo đường dẫn đến một file đã cho trong thư mục public:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` {.collection-method}

Hàm `resource_path` trả về đường dẫn đến thư mục `resource` của application của bạn. Bạn cũng có thể sử dụng hàm `resource_path` để tạo đường dẫn đến một file đã cho trong thư mục resources:

    $path = resource_path();

    $path = resource_path('sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {.collection-method}

Hàm `storage_path` trả về đường dẫn đến thư mục` storage` của application của bạn. Bạn cũng có thể sử dụng hàm `storage_path` để tạo đường dẫn đến một file đã cho trong thư mục storage:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Strings

<a name="method-__"></a>
#### `__()` {.collection-method}

Hàm `__` sẽ dịch chuỗi cần được dịch hoặc key cần được dịch đã cho bằng cách sử dụng [localization files](/docs/{{version}}/localization) của bạn:

    echo __('Welcome to our application');

    echo __('messages.welcome');

Nếu chuỗi hoặc key cần được dịch không tồn tại, hàm `__` sẽ trả về giá trị được đưa vào. Vì vậy, nếu sử dụng ví dụ mẫu trên, hàm `__` sẽ trả về `messages.welcome` nếu key cần được dịch đó không tồn tại.

<a name="method-class-basename"></a>
#### `class_basename()` {.collection-method}

`class_basename` trả về tên class đã cho với namespace của class bị xóa:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {.collection-method}

Hàm `e` chạy hàm` htmlspecialchars` của PHP với tùy chọn `double_encode` được set mặc định thành `true`:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {.collection-method}

Hàm `preg_replace_array` sẽ thay thế một pattern vào trong một chuỗi sequentially bằng cách sử dụng một mảng:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-after"></a>
#### `Str::after()` {.collection-method}

Hàm `Str::after` sẽ trả về mọi thứ đứng sau giá trị đã cho có trong một chuỗi. Toàn bộ chuỗi sẽ được trả về nếu giá trị đó không tồn tại trong chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::after('This is my name', 'This is');

    // ' my name'

<a name="method-str-after-last"></a>
#### `Str::afterLast()` {.collection-method}

Hàm `Str::afterLast` sẽ trả về mọi thứ đứng đằng sau, sau lần xuất hiện cuối cùng của giá trị đã cho trong một chuỗi. Toàn bộ chuỗi sẽ được trả về nếu giá trị đó không tồn tại trong chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

    // 'Controller'

<a name="method-str-ascii"></a>
#### `Str::ascii()` {.collection-method}

Hàm `Str::ascii` sẽ cố thử chuyển một chuỗi thành một giá trị ASCII:

    use Illuminate\Support\Str;

    $slice = Str::ascii('û');

    // 'u'

<a name="method-str-before"></a>
#### `Str::before()` {.collection-method}

Hàm `Str::before` sẽ trả về mọi thứ đứng trước giá trị đã cho có trong một chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::before('This is my name', 'my name');

    // 'This is '

<a name="method-str-before-last"></a>
#### `Str::beforeLast()` {.collection-method}

Hàm `Str::beforeLast` sẽ trả về mọi thứ đứng đằng trước, trước lần xuất hiện cuối cùng của giá trị đã cho trong một chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::beforeLast('This is my name', 'is');

    // 'This '

<a name="method-str-between"></a>
#### `Str::between()` {.collection-method}

Hàm `Str::between` sẽ trả về một phần của chuỗi nằm giữa hai giá trị đã cho:

    use Illuminate\Support\Str;

    $slice = Str::between('This is my name', 'This', 'name');

    // ' is my '

<a name="method-str-between-first"></a>
#### `Str::betweenFirst()` {.collection-method}

Hàm `Str::betweenFirst` sẽ trả về phần nhỏ nhất của một chuỗi nằm giữa hai giá trị:

    use Illuminate\Support\Str;

    $slice = Str::betweenFirst('[a] bc [d]', '[', ']');

    // 'a'

<a name="method-camel-case"></a>
#### `Str::camel()` {.collection-method}

Hàm `Str::camel` chuyển đổi chuỗi đã cho thành `camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::camel('foo_bar');

    // fooBar

<a name="method-str-contains"></a>
#### `Str::contains()` {.collection-method}

Hàm `Str::contains` xác định xem chuỗi đã cho có chứa giá trị đã cho hay không. Phương thức này phân biệt chữ hoa và chữ thường:

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', 'my');

    // true

Bạn cũng có thể truyền vào một mảng các giá trị để xác định xem chuỗi đã cho có chứa bất kỳ giá trị nào trong mảng không:

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-contains-all"></a>
#### `Str::containsAll()` {.collection-method}

Hàm `Str::containsAll` sẽ xác định xem string đã cho có chứa tất cả các giá trị có trong mảng hay không:

    use Illuminate\Support\Str;

    $containsAll = Str::containsAll('This is my name', ['my', 'name']);

    // true

<a name="method-ends-with"></a>
#### `Str::endsWith()` {.collection-method}

Hàm `Str::endsWith` sẽ kiểm tra chuỗi đã cho có kết thúc bằng giá trị đã cho hay không:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', 'name');

    // true


Bạn cũng có thể truyền một mảng các giá trị để kiểm tra xem chuỗi đã cho có kết thúc bằng các giá trị có trong số các giá trị đã cho hay không in the array:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', ['name', 'foo']);

    // true

    $result = Str::endsWith('This is my name', ['this', 'foo']);

    // false

<a name="method-excerpt"></a>
#### `Str::excerpt()` {.collection-method}

Hàm `Str::excerpt` sẽ lấy ra một đoạn đầu tiên từ một chuỗi mà khớp với chuỗi đã cho:

    use Illuminate\Support\Str;

    $excerpt = Str::excerpt('This is my name', 'my', [
        'radius' => 3
    ]);

    // '...is my na...'

Tùy chọn `radius` có giá trị mặc định là `100`, cho phép bạn định nghĩa số lượng ký tự sẽ xuất hiện ở mỗi bên của chuỗi đã được lấy ra.

Ngoài ra, bạn có thể sử dụng tùy chọn `omission` để định nghĩa chuỗi sẽ được thêm vào trước hoặc sau chuỗi đã được lấy ra:

    use Illuminate\Support\Str;

    $excerpt = Str::excerpt('This is my name', 'name', [
        'radius' => 3,
        'omission' => '(...) '
    ]);

    // '(...) my name'

<a name="method-str-finish"></a>
#### `Str::finish()` {.collection-method}

Hàm `Str::finish` sẽ thêm một instance của giá trị đã cho vào một chuỗi nếu nó chưa kết thúc bằng giá trị đó:

    use Illuminate\Support\Str;

    $adjusted = Str::finish('this/string', '/');

    // this/string/

    $adjusted = Str::finish('this/string/', '/');

    // this/string/

<a name="method-str-headline"></a>
#### `Str::headline()` {.collection-method}

Hàm `Str::headline` sẽ chuyển các chuỗi được phân tách bằng cách viết hoa chữ cái đầu, dấu gạch ngang hoặc dấu gạch dưới thành một chuỗi được phân cách bằng dấu cách và chữ cái đầu tiên của mỗi từ được viết hoa:

    use Illuminate\Support\Str;

    $headline = Str::headline('steve_jobs');

    // Steve Jobs

    $headline = Str::headline('EmailNotificationSent');

    // Email Notification Sent

<a name="method-str-inline-markdown"></a>
#### `Str::inlineMarkdown()` {.collection-method}

Hàm `Str::inlineMarkdown` sẽ chuyển đổi Markdown định dạng theo chuẩn GitHub thành HTML bằng cách sử dụng [CommonMark](https://commonmark.thephpleague.com/). Tuy nhiên, không giống như phương thức `markdown`, nó không wrap tất cả code HTML được tạo vào trong một phần tử ở mức độ block:

    use Illuminate\Support\Str;

    $html = Str::inlineMarkdown('**Laravel**');

    // <strong>Laravel</strong>

<a name="method-str-is"></a>
#### `Str::is()` {.collection-method}

Hàm `Str::is` sẽ xác định xem một chuỗi đã cho có khớp với mẫu đã cho hay không. Dấu hoa thị có thể được sử dụng để làm giá trị đại diện:

    use Illuminate\Support\Str;

    $matches = Str::is('foo*', 'foobar');

    // true

    $matches = Str::is('baz*', 'foobar');

    // false

<a name="method-str-is-ascii"></a>
#### `Str::isAscii()` {.collection-method}

Hàm `Str::isAscii` sẽ xác định xem một chuỗi đã cho có phải là dạng ASCII 7 bit hay không:

    use Illuminate\Support\Str;

    $isAscii = Str::isAscii('Taylor');

    // true

    $isAscii = Str::isAscii('ü');

    // false

<a name="method-str-is-json"></a>
#### `Str::isJson()` {.collection-method}

Hàm `Str::isJson` sẽ xác định xem chuỗi đã cho có phải là một định dạng JSON hợp lệ hay không:

    use Illuminate\Support\Str;

    $result = Str::isJson('[1,2,3]');

    // true

    $result = Str::isJson('{"first": "John", "last": "Doe"}');

    // true

    $result = Str::isJson('{first: "John", last: "Doe"}');

    // false

<a name="method-str-is-ulid"></a>
#### `Str::isUlid()` {.collection-method}

Phương thức `Str::isUlid` sẽ xác định xem chuỗi đã cho có phải là một ULID hợp lệ hay không:

    use Illuminate\Support\Str;

    $isUlid = Str::isUlid('01gd6r360bp37zj17nxb55yv40');

    // true

    $isUlid = Str::isUlid('laravel');

    // false

<a name="method-str-is-uuid"></a>
#### `Str::isUuid()` {.collection-method}

Hàm `Str::isUuid` sẽ kiểm tra xem chuỗi đã cho có phải là một UUID hợp lệ hay không:

    use Illuminate\Support\Str;

    $isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

    // true

    $isUuid = Str::isUuid('laravel');

    // false

<a name="method-kebab-case"></a>
#### `Str::kebab()` {.collection-method}

Hàm `Str::kebab` chuyển đổi chuỗi đã cho thành `kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::kebab('fooBar');

    // foo-bar

<a name="method-str-lcfirst"></a>
#### `Str::lcfirst()` {.collection-method}

Hàm `Str::lcfirst` sẽ trả về chuỗi đã cho với ký tự đầu tiên được viết thường:

    use Illuminate\Support\Str;

    $string = Str::lcfirst('Foo Bar');

    // foo Bar

<a name="method-str-length"></a>
#### `Str::length()` {.collection-method}

Hàm `Str::length` sẽ trả về độ dài của chuỗi đã cho:

    use Illuminate\Support\Str;

    $length = Str::length('Laravel');

    // 7

<a name="method-str-limit"></a>
#### `Str::limit()` {.collection-method}

Hàm `Str::limit` sẽ cắt ngắn chuỗi đã cho đến độ dài nhất định:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

Bạn có thể truyền một tham số thứ ba vào phương thức để thay đổi chuỗi sẽ được nối vào cuối chuỗi bị cắt ngắn:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-lower"></a>
#### `Str::lower()` {.collection-method}

Hàm `Str::lower` sẽ chuyển một chuỗi đã cho thành chữ thường:

    use Illuminate\Support\Str;

    $converted = Str::lower('LARAVEL');

    // laravel

<a name="method-str-markdown"></a>
#### `Str::markdown()` {.collection-method}

Hàm `Str::markdown` sẽ chuyển đổi Markdown định dạng theo chuẩn GitHub thành HTML dùng [CommonMark](https://commonmark.thephpleague.com/):

    use Illuminate\Support\Str;

    $html = Str::markdown('# Laravel');

    // <h1>Laravel</h1>

    $html = Str::markdown('# Taylor <b>Otwell</b>', [
        'html_input' => 'strip',
    ]);

    // <h1>Taylor Otwell</h1>

<a name="method-str-mask"></a>
#### `Str::mask()` {.collection-method}

Hàm `Str::mask` sẽ che giấu một phần của chuỗi với một ký tự lặp lại và có thể được sử dụng để làm xáo trộn các phân đoạn của chuỗi như địa chỉ email hoặc các số điện thoại:

    use Illuminate\Support\Str;

    $string = Str::mask('taylor@example.com', '*', 3);

    // tay***************

Nếu cần, bạn cũng có thể cung cấp một số âm làm tham số thứ ba cho phương thức `mask`, điều này sẽ hướng dẫn phương thức bắt đầu tạo chuỗi ở khoảng cách nhất định tính từ cuối chuỗi trở về:

    $string = Str::mask('taylor@example.com', '*', -15, 3);

    // tay***@example.com

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` {.collection-method}

Hàm `Str::orderedUuid` sẽ tạo một UUID "timestamp first" có thể được lưu trữ tốt trong một cột được index trong cơ sở dữ liệu. Mỗi UUID được tạo ra bằng phương thức này sẽ được sắp xếp sau các UUID đã được tạo ra trước đó:

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-padboth"></a>
#### `Str::padBoth()` {.collection-method}

Hàm `Str::padBoth` sẽ wrap hàm `str_pad` của PHP, sẽ thêm vào cả hai bên của một chuỗi để thành một chuỗi khác cho đến khi chuỗi cuối cùng đạt đến độ dài mong muốn:

    use Illuminate\Support\Str;

    $padded = Str::padBoth('James', 10, '_');

    // '__James___'

    $padded = Str::padBoth('James', 10);

    // '  James   '

<a name="method-str-padleft"></a>
#### `Str::padLeft()` {.collection-method}

Hàm `Str::padLeft` sẽ wrap hàm `str_pad` của PHP, sẽ thêm vào phía bên trái của một chuỗi để thành một chuỗi khác cho đến khi chuỗi cuối cùng đạt đến độ dài mong muốn:

    use Illuminate\Support\Str;

    $padded = Str::padLeft('James', 10, '-=');

    // '-=-=-James'

    $padded = Str::padLeft('James', 10);

    // '     James'

<a name="method-str-padright"></a>
#### `Str::padRight()` {.collection-method}

Hàm `Str::padRight` sẽ wrap hàm `str_pad` của PHP, sẽ thêm vào phía bên phải của một chuỗi để thành một chuỗi khác cho đến khi chuỗi cuối cùng đạt đến độ dài mong muốn:

    use Illuminate\Support\Str;

    $padded = Str::padRight('James', 10, '-');

    // 'James-----'

    $padded = Str::padRight('James', 10);

    // 'James     '

<a name="method-str-plural"></a>
#### `Str::plural()` {.collection-method}

Hàm `Str::plural` sẽ chuyển đổi một chuỗi đơn thành dạng số nhiều của nó. Chức năng này hỗ trợ [bất kỳ ngôn ngữ nào được hỗ trợ bộ quy tắc số nhiều của Laravel](/docs/{{version}}/localization#pluralization-language):

    use Illuminate\Support\Str;

    $plural = Str::plural('car');

    // cars

    $plural = Str::plural('child');

    // children

Bạn có thể cung cấp một số nguyên dưới dạng tham số thứ hai cho hàm để lấy dạng số ít hoặc số nhiều của chuỗi:

    use Illuminate\Support\Str;

    $plural = Str::plural('child', 2);

    // children

    $singular = Str::plural('child', 1);

    // child

<a name="method-str-plural-studly"></a>
#### `Str::pluralStudly()` {.collection-method}

Hàm `Str::pluralStudly` sẽ chuyển một chuỗi từ số ít sang số nhiều. Chức năng này hỗ trợ [bất kỳ ngôn ngữ nào được hỗ trợ bộ quy tắc số nhiều của Laravel](/docs/{{version}}/localization#pluralization-language):

    use Illuminate\Support\Str;

    $plural = Str::pluralStudly('VerifiedHuman');

    // VerifiedHumans

    $plural = Str::pluralStudly('UserFeedback');

    // UserFeedback

Bạn có thể cung cấp một số nguyên làm tham số thứ hai cho phương thức để trả về dạng số ít hoặc số nhiều của chuỗi:

    use Illuminate\Support\Str;

    $plural = Str::pluralStudly('VerifiedHuman', 2);

    // VerifiedHumans

    $singular = Str::pluralStudly('VerifiedHuman', 1);

    // VerifiedHuman

<a name="method-str-random"></a>
#### `Str::random()` {.collection-method}

Hàm `Str::random` sẽ tạo ra một chuỗi ngẫu nhiên có độ dài được chỉ định. Hàm này sử dụng hàm `random_bytes` của PHP:

    use Illuminate\Support\Str;

    $random = Str::random(40);

<a name="method-str-remove"></a>
#### `Str::remove()` {.collection-method}

Hàm `Str::remove` sẽ xoá các giá trị hoặc một mảng các giá trị ra khỏi chuỗi:

    use Illuminate\Support\Str;

    $string = 'Peter Piper picked a peck of pickled peppers.';

    $removed = Str::remove('e', $string);

    // Ptr Pipr pickd a pck of pickld ppprs.

Bạn cũng có thể truyền một tham số `false` làm tham số thứ ba cho phương thức `remove` để xoá cả chữ hoa chữ thường.

<a name="method-str-replace"></a>
#### `Str::replace()` {.collection-method}

Hàm `Str::replace` sẽ thay thế một chuỗi trong chuỗi:

    use Illuminate\Support\Str;

    $string = 'Laravel 8.x';

    $replaced = Str::replace('8.x', '9.x', $string);

    // Laravel 9.x

<a name="method-str-replace-array"></a>
#### `Str::replaceArray()` {.collection-method}

Hàm `Str::replaceArray` sẽ thay thế một giá trị đã cho vào trong một chuỗi sequentially bằng cách sử dụng một mảng:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `Str::replaceFirst()` {.collection-method}

Hàm `Str::replaceFirst` sẽ thay thế giá trị đầu tiên có trong chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `Str::replaceLast()` {.collection-method}

Hàm `Str::replaceLast` sẽ thay thế giá trị cuối cùng có trong chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog


<a name="method-str-reverse"></a>
#### `Str::reverse()` {.collection-method}

Hàm `Str::reverse` sẽ đảo ngược chuỗi đã cho:

    use Illuminate\Support\Str;

    $reversed = Str::reverse('Hello World');

    // dlroW olleH

<a name="method-str-singular"></a>
#### `Str::singular()` {.collection-method}

Hàm `Str::singular` sẽ chuyển đổi một chuỗi thành dạng số ít của nó. Chức năng này hỗ trợ [bất kỳ ngôn ngữ nào được hỗ trợ bộ quy tắc số nhiều của Laravel](/docs/{{version}}/localization#pluralization-language):

    use Illuminate\Support\Str;

    $singular = Str::singular('cars');

    // car

    $singular = Str::singular('children');

    // child

<a name="method-str-slug"></a>
#### `Str::slug()` {.collection-method}

Hàm `Str::slug` sẽ tạo ra một URL "slug" từ chuỗi đã cho:

    use Illuminate\Support\Str;

    $slug = Str::slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-snake-case"></a>
#### `Str::snake()` {.collection-method}

Hàm `Str::snake` sẽ chuyển đổi chuỗi đã cho thành `Str::snake`:

    use Illuminate\Support\Str;

    $converted = Str::snake('fooBar');

    // foo_bar

    $converted = Str::snake('fooBar', '-');

    // foo-bar

<a name="method-str-squish"></a>
#### `Str::squish()` {.collection-method}

Hàm `Str::squish` sẽ loại bỏ tất cả các khoảng trắng không cần thiết có trong một chuỗi, bao gồm cả các khoảng trắng không liên quan giữa các từ đó:

    use Illuminate\Support\Str;

    $string = Str::squish('    laravel    framework    ');

    // laravel framework

<a name="method-str-start"></a>
#### `Str::start()` {.collection-method}

Hàm `Str::start` sẽ thêm một instance của giá trị đã cho vào một chuỗi nếu nó chưa bắt đầu bằng giá trị đó:

    use Illuminate\Support\Str;

    $adjusted = Str::start('this/string', '/');

    // /this/string

    $adjusted = Str::start('/this/string', '/');

    // /this/string

<a name="method-starts-with"></a>
#### `Str::startsWith()` {.collection-method}

Hàm `started_with` sẽ kiểm tra chuỗi đã cho có bắt đầu bằng giá trị đã cho hay không:

    use Illuminate\Support\Str;

    $result = Str::startsWith('This is my name', 'This');

    // true

Nếu một mảng các giá trị được truyền, thì phương thức `startsWith` sẽ trả về `true` nếu chuỗi bắt đầu bằng một trong các giá trị đã cho:

    $result = Str::startsWith('This is my name', ['This', 'That', 'There']);

    // true

<a name="method-studly-case"></a>
#### `Str::studly()` {.collection-method}

Hàm `Str::studly` chuyển đổi chuỗi đã cho thành` StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::studly('foo_bar');

    // FooBar

<a name="method-str-substr"></a>
#### `Str::substr()` {.collection-method}

Hàm `Str::substr` sẽ trả lại phần chuỗi được chỉ định bởi các tham số bắt đầu và độ dài của chuỗi cần lấy:

    use Illuminate\Support\Str;

    $converted = Str::substr('The Laravel Framework', 4, 7);

    // Laravel

<a name="method-str-substrcount"></a>
#### `Str::substrCount()` {.collection-method}

Hàm `Str::substrCount` sẽ trả về số lần xuất hiện của một giá trị trong một chuỗi đã cho:

    use Illuminate\Support\Str;

    $count = Str::substrCount('If you like ice cream, you will like snow cones.', 'like');

    // 2

<a name="method-str-substrreplace"></a>
#### `Str::substrReplace()` {.collection-method}

Hàm `Str::substrReplace` sẽ thay thế text có trong một phần của chuỗi, bắt đầu từ vị trí được chỉ định bởi tham số thứ ba và thay thế số ký tự được chỉ định bởi tham số thứ tư. Truyền tham số thứ tư là `0` nếu muốn chèn chuỗi vào vị trí đã chỉ định mà không thay thế bất kỳ ký tự nào có trong chuỗi:

    use Illuminate\Support\Str;

    $result = Str::substrReplace('1300', ':', 2);
    // 13:

    $result = Str::substrReplace('1300', ':', 2, 0);
    // 13:00

<a name="method-str-swap"></a>
#### `Str::swap()` {.collection-method}

Hàm `Str::swap` sẽ thay thế nhiều giá trị trong chuỗi đã cho bằng hàm `strtr` của PHP:

    use Illuminate\Support\Str;

    $string = Str::swap([
        'Tacos' => 'Burritos',
        'great' => 'fantastic',
    ], 'Tacos are great!');

    // Burritos are fantastic!

<a name="method-title-case"></a>
#### `Str::title()` {.collection-method}

Hàm `Str::title` chuyển đổi chuỗi đã cho thành` Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-to-html-string"></a>
#### `Str::toHtmlString()` {.collection-method}

Hàm `Str::toHtmlString` sẽ chuyển một instance chuỗi thành một instance của `Illuminate\Support\HtmlString`, để có thể được hiển thị trong các template Blade:

    use Illuminate\Support\Str;

    $htmlString = Str::of('Nuno Maduro')->toHtmlString();

<a name="method-str-ucfirst"></a>
#### `Str::ucfirst()` {.collection-method}

Hàm `Str::ucfirst` sẽ trả lại chuỗi đã cho với ký tự đầu tiên được viết hoa:

    use Illuminate\Support\Str;

    $string = Str::ucfirst('foo bar');

    // Foo bar

<a name="method-str-ucsplit"></a>
#### `Str::ucsplit()` {.collection-method}

Hàm `Str::ucsplit` sẽ chia chuỗi đã cho thành một mảng theo các ký tự được viết hoa:

    use Illuminate\Support\Str;

    $segments = Str::ucsplit('FooBar');

    // [0 => 'Foo', 1 => 'Bar']

<a name="method-str-upper"></a>
#### `Str::upper()` {.collection-method}

Hàm `Str::upper` sẽ chuyển đổi chuỗi đã cho thành chữ hoa toàn bộ chuỗi:

    use Illuminate\Support\Str;

    $string = Str::upper('laravel');

    // LARAVEL

<a name="method-str-ulid"></a>
#### `Str::ulid()` {.collection-method}

Hàm `Str::ulid` sẽ tạo một ULID:

    use Illuminate\Support\Str;

    return (string) Str::ulid();

    // 01gd6r360bp37zj17nxb55yv40

<a name="method-str-uuid"></a>
#### `Str::uuid()` {.collection-method}

Hàm `Str::uuid` sẽ tạo ra một UUID (phiên bản 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="method-str-word-count"></a>
#### `Str::wordCount()` {.collection-method}

Hàm `Str::wordCount` sẽ trả về số lượng từ mà một chuỗi chứa:

```php
use Illuminate\Support\Str;

Str::wordCount('Hello, world!'); // 2
```

<a name="method-str-words"></a>
#### `Str::words()` {.collection-method}

Hàm `Str::words` sẽ giới hạn số lượng từ có trong một chuỗi. Một chuỗi bổ sung có thể được truyền cho phương thức này thông qua tham số thứ ba của nó để chỉ định chuỗi nào sẽ được thêm vào cuối chuỗi bị cắt ngắn:

    use Illuminate\Support\Str;

    return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

    // Perfectly balanced, as >>>

<a name="method-str"></a>
#### `str()` {.collection-method}

Hàm `str` sẽ trả về một instance `Illuminate\Support\Stringable` mới của một chuỗi đã cho. Hàm này giống với hàm `Str::of`:

    $string = str('Taylor')->append(' Otwell');

    // 'Taylor Otwell'

Nếu không có tham số nào được truyền vào cho hàm `str`, thì hàm này trả về một instance của `Illuminate\Support\Str`:

    $snake = str()->snake('FooBar');

    // 'foo_bar'

<a name="method-trans"></a>
#### `trans()` {.collection-method}

Hàm `trans` sẽ dịch các key cần dịch bằng cách sử dụng [localization files](/docs/{{version}}/localization) của bạn:

    echo trans('messages.welcome');

Nếu key cần dịch mà không tồn tại, hàm `trans` sẽ trả về key đó. Vì vậy, nếu sử dụng ví dụ trên, hàm `trans` sẽ trả về `message.welcome` nếu key cần dịch không tồn tại.

<a name="method-trans-choice"></a>
#### `trans_choice()` {.collection-method}

Hàm `trans_choice` sẽ dịch các key cần dịch đã cho với một biến số nhiều:

    echo trans_choice('messages.notifications', $unreadCount);

Nếu key cần dịch mà không tồn tại, hàm `trans_choice` sẽ trả về key đó. Vì vậy, nếu sử dụng ví dụ trên, hàm `trans_choice` sẽ trả về `messages.notifications` nếu key cần dịch không tồn tại.

<a name="fluent-strings"></a>
## Fluent Strings

Fluent string cung cấp một interface hướng đối tượng, trôi chảy hơn để làm việc với các giá trị chuỗi, cho phép bạn kết hợp nhiều xử lý chuỗi lại với nhau bằng cách sử dụng cú pháp dễ đọc hơn so với các xử lý chuỗi truyền thống.

<a name="method-fluent-str-after"></a>
#### `after` {.collection-method}

Hàm `after` sẽ trả về mọi thứ nằm sau giá trị đã cho trong một chuỗi. Toàn bộ chuỗi sẽ được trả về nếu giá trị truyền vào không tồn tại trong chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->after('This is');

    // ' my name'

<a name="method-fluent-str-after-last"></a>
#### `afterLast` {.collection-method}

Hàm `afterLast` sẽ trả về mọi thứ sau lần xuất hiện cuối cùng của giá trị đã cho trong một chuỗi. Toàn bộ chuỗi sẽ được trả về nếu giá trị truyền vào không tồn tại trong chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::of('App\Http\Controllers\Controller')->afterLast('\\');

    // 'Controller'

<a name="method-fluent-str-append"></a>
#### `append` {.collection-method}

Hàm `append` sẽ nối các giá trị đã cho vào chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('Taylor')->append(' Otwell');

    // 'Taylor Otwell'

<a name="method-fluent-str-ascii"></a>
#### `ascii` {.collection-method}

Hàm `ascii` sẽ thử chuyển một chuỗi thành giá trị ASCII:

    use Illuminate\Support\Str;

    $string = Str::of('ü')->ascii();

    // 'u'

<a name="method-fluent-str-basename"></a>
#### `basename` {.collection-method}

Hàm `basename` sẽ trả về phần cuối cùng của chuỗi đã cho:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->basename();

    // 'baz'

Nếu cần, bạn có thể cung cấp một "extension" sẽ bị xóa ra khỏi phần cuối cùng:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz.jpg')->basename('.jpg');

    // 'baz'

<a name="method-fluent-str-before"></a>
#### `before` {.collection-method}

Hàm `before` trả về mọi thứ đứng trước giá trị đã cho trong một chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->before('my name');

    // 'This is '

<a name="method-fluent-str-before-last"></a>
#### `beforeLast` {.collection-method}

Hàm `beforeLast` trả về mọi thứ đứng trước, trước lần xuất hiện cuối cùng của giá trị đã cho trong một chuỗi:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->beforeLast('is');

    // 'This '

<a name="method-fluent-str-between"></a>
#### `between` {.collection-method}

Hàm `between` sẽ trả về một phần của chuỗi nằm giữa hai giá trị:

    use Illuminate\Support\Str;

    $converted = Str::of('This is my name')->between('This', 'name');

    // ' is my '

<a name="method-fluent-str-between-first"></a>
#### `betweenFirst` {.collection-method}

Hàm `betweenFirst` sẽ trả về phần nhỏ nhất của một chuỗi nằm giữa hai giá trị:

    use Illuminate\Support\Str;

    $converted = Str::of('[a] bc [d]')->betweenFirst('[', ']');

    // 'a'

<a name="method-fluent-str-camel"></a>
#### `camel` {.collection-method}

Hàm `camel` sẽ chuyển đổi chuỗi đã cho thành `camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->camel();

    // fooBar

<a name="method-fluent-str-class-basename"></a>
#### `classBasename` {.collection-method}

Hàm `classBasename` sẽ trả về tên class của class đã cho mà namespace của class đó đã bị xóa bỏ:

    use Illuminate\Support\Str;

    $class = Str::of('Foo\Bar\Baz')->classBasename();

    // Baz

<a name="method-fluent-str-contains"></a>
#### `contains` {.collection-method}

Hàm `contains` sẽ xác định xem chuỗi đã cho có chứa giá trị đã cho hay không. Phương thức này sẽ phân biệt chữ hoa chữ thường:

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains('my');

    // true

Bạn cũng có thể truyền một mảng giá trị để xác định xem chuỗi đã cho có chứa bất kỳ giá trị nào trong mảng đó hay không:

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains(['my', 'foo']);

    // true

<a name="method-fluent-str-contains-all"></a>
#### `containsAll` {.collection-method}

Hàm `containsAll` sẽ xác định xem chuỗi đã cho có chứa tất cả các giá trị của một mảng đã cho hay không:

    use Illuminate\Support\Str;

    $containsAll = Str::of('This is my name')->containsAll(['my', 'name']);

    // true

<a name="method-fluent-str-dirname"></a>
#### `dirname` {.collection-method}

Hàm `dirname` sẽ trả về phần thư mục cha của chuỗi đã cho:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname();

    // '/foo/bar'

Nếu cần, bạn có thể chỉ định thêm số lượng cấp của thư mục mà bạn muốn cắt ra khỏi chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname(2);

    // '/foo'

<a name="method-fluent-str-excerpt"></a>
#### `excerpt` {.collection-method}

Hàm `excerpt` sẽ lấy ra một đoạn đầu tiên từ một chuỗi mà khớp với chuỗi đã cho:

    use Illuminate\Support\Str;

    $excerpt = Str::of('This is my name')->excerpt('my', [
        'radius' => 3
    ]);

    // '...is my na...'

Tùy chọn `radius` có giá trị mặc định là `100`, cho phép bạn định nghĩa số lượng ký tự sẽ xuất hiện ở mỗi bên của chuỗi đã được lấy ra.

Ngoài ra, bạn có thể sử dụng tùy chọn `omission` để thay đổi chuỗi sẽ được thêm vào trước hoặc sau chuỗi đã được lấy ra:

    use Illuminate\Support\Str;

    $excerpt = Str::of('This is my name')->excerpt('name', [
        'radius' => 3,
        'omission' => '(...) '
    ]);

    // '(...) my name'

<a name="method-fluent-str-ends-with"></a>
#### `endsWith` {.collection-method}

Hàm `endsWith` sẽ xác định xem chuỗi đã cho có kết thúc bằng giá trị đã cho hay không:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith('name');

    // true

Bạn cũng có thể truyền một mảng giá trị để xác định xem chuỗi đã cho có kết thúc bằng một giá trị trong số các giá trị đã cho hay không:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith(['name', 'foo']);

    // true

    $result = Str::of('This is my name')->endsWith(['this', 'foo']);

    // false

<a name="method-fluent-str-exactly"></a>
#### `exactly` {.collection-method}

Hàm `exactly` sẽ xác định xem chuỗi đã cho có khớp với một chuỗi khác hay không:

    use Illuminate\Support\Str;

    $result = Str::of('Laravel')->exactly('Laravel');

    // true

<a name="method-fluent-str-explode"></a>
#### `explode` {.collection-method}

Hàm `explode` sẽ chia chuỗi ra theo dấu phân cách đã cho và trả về một collection chứa từng chuỗi nhỏ của chuỗi đã cho:

    use Illuminate\Support\Str;

    $collection = Str::of('foo bar baz')->explode(' ');

    // collect(['foo', 'bar', 'baz'])

<a name="method-fluent-str-finish"></a>
#### `finish` {.collection-method}

Hàm `finish` sẽ thêm một giá trị đã cho vào sau một chuỗi nếu nó chưa kết thúc bằng giá trị đó:

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->finish('/');

    // this/string/

    $adjusted = Str::of('this/string/')->finish('/');

    // this/string/

<a name="method-fluent-str-headline"></a>
#### `headline` {.collection-method}

Phương thức `headline` sẽ chuyển đổi các chuỗi được phân cách bằng cách viết hoa hoặc gạch nối giữa các từ và hoặc dấu gạch dưới thành chuỗi được phân cách bằng dấu cách với chữ cái đầu tiên của mỗi từ sẽ được viết hoa:

    use Illuminate\Support\Str;

    $headline = Str::of('taylor_otwell')->headline();

    // Taylor Otwell

    $headline = Str::of('EmailNotificationSent')->headline();

    // Email Notification Sent

<a name="method-fluent-str-inline-markdown"></a>
#### `inlineMarkdown` {.collection-method}

Hàm `inlineMarkdown` sẽ chuyển đổi Markdown định dạng theo chuẩn GitHub thành HTML bằng cách sử dụng [CommonMark](https://commonmark.thephpleague.com/). Tuy nhiên, không giống như phương thức `markdown`, nó không wrap tất cả code HTML được tạo vào trong một phần tử ở mức độ block:

    use Illuminate\Support\Str;

    $html = Str::of('**Laravel**')->inlineMarkdown();

    // <strong>Laravel</strong>

<a name="method-fluent-str-is"></a>
#### `is` {.collection-method}

Hàm `is` sẽ xác định xem một chuỗi đã cho có khớp với một pattern nhất định hay không. Dấu hoa thị có thể được sử dụng để biểu thị cho giá trị đại diện:

    use Illuminate\Support\Str;

    $matches = Str::of('foobar')->is('foo*');

    // true

    $matches = Str::of('foobar')->is('baz*');

    // false

<a name="method-fluent-str-is-ascii"></a>
#### `isAscii` {.collection-method}

Hàm `isAscii` sẽ xác định xem một chuỗi đã cho có phải là chuỗi ASCII hay không:

    use Illuminate\Support\Str;

    $result = Str::of('Taylor')->isAscii();

    // true

    $result = Str::of('ü')->isAscii();

    // false

<a name="method-fluent-str-is-empty"></a>
#### `isEmpty` {.collection-method}

Hàm `isEmpty` sẽ xác định xem chuỗi đã cho có trống hay không:

    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isEmpty();

    // true

    $result = Str::of('Laravel')->trim()->isEmpty();

    // false

<a name="method-fluent-str-is-not-empty"></a>
#### `isNotEmpty` {.collection-method}

Hàm `isNotEmpty` sẽ xác định xem chuỗi đã cho không trống đúng không:


    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isNotEmpty();

    // false

    $result = Str::of('Laravel')->trim()->isNotEmpty();

    // true

<a name="method-fluent-str-is-json"></a>
#### `isJson` {.collection-method}

Hàm `isJson` sẽ xác định xem một chuỗi đã cho có phải là dạng JSON hợp lệ hay không:

    use Illuminate\Support\Str;

    $result = Str::of('[1,2,3]')->isJson();

    // true

    $result = Str::of('{"first": "John", "last": "Doe"}')->isJson();

    // true

    $result = Str::of('{first: "John", last: "Doe"}')->isJson();

    // false

<a name="method-fluent-str-is-ulid"></a>
#### `isUlid` {.collection-method}

Hàm `isUlid` sẽ xác định xem một chuỗi đã cho có phải là một dạng ULID hay không:

    use Illuminate\Support\Str;

    $result = Str::of('01gd6r360bp37zj17nxb55yv40')->isUlid();

    // true

    $result = Str::of('Taylor')->isUlid();

    // false

<a name="method-fluent-str-is-uuid"></a>
#### `isUuid` {.collection-method}

Hàm `isUuid` sẽ xác định xem một chuỗi đã cho có phải là dạng UUID hay không:

    use Illuminate\Support\Str;

    $result = Str::of('5ace9ab9-e9cf-4ec6-a19d-5881212a452c')->isUuid();

    // true

    $result = Str::of('Taylor')->isUuid();

    // false

<a name="method-fluent-str-kebab"></a>
#### `kebab` {.collection-method}

Hàm `kebab` sẽ chuyển đổi một chuỗi đã cho thành một dạng `kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->kebab();

    // foo-bar

<a name="method-fluent-str-lcfirst"></a>
#### `lcfirst` {.collection-method}

Hàm `lcfirst` sẽ trả về chuỗi đã cho với ký tự đầu tiên được viết thường:

    use Illuminate\Support\Str;

    $string = Str::of('Foo Bar')->lcfirst();

    // foo Bar


<a name="method-fluent-str-length"></a>
#### `length` {.collection-method}

Hàm `length` sẽ trả về độ dài của chuỗi đã cho:

    use Illuminate\Support\Str;

    $length = Str::of('Laravel')->length();

    // 7

<a name="method-fluent-str-limit"></a>
#### `limit` {.collection-method}

Hàm `limit` sẽ cắt chuỗi đã cho đến một độ dài nhất định:

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20);

    // The quick brown fox...

Bạn cũng có thể truyền thêm một tham số thứ hai để nối vào cuối chuỗi đã bị cắt:

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20, ' (...)');

    // The quick brown fox (...)

<a name="method-fluent-str-lower"></a>
#### `lower` {.collection-method}

Hàm `lower` sẽ chuyển đổi chuỗi đã cho thành chữ thường:

    use Illuminate\Support\Str;

    $result = Str::of('LARAVEL')->lower();

    // 'laravel'

<a name="method-fluent-str-ltrim"></a>
#### `ltrim` {.collection-method}

Hàm `ltrim` sẽ cắt bên trái của chuỗi đã cho:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->ltrim();

    // 'Laravel  '

    $string = Str::of('/Laravel/')->ltrim('/');

    // 'Laravel/'

<a name="method-fluent-str-markdown"></a>
#### `markdown` {.collection-method}

Hàm `markdown` sẽ chuyển đổi Markdown định dạng theo chuẩn GitHub thành HTML:

    use Illuminate\Support\Str;

    $html = Str::of('# Laravel')->markdown();

    // <h1>Laravel</h1>

    $html = Str::of('# Taylor <b>Otwell</b>')->markdown([
        'html_input' => 'strip',
    ]);

    // <h1>Taylor Otwell</h1>

<a name="method-fluent-str-mask"></a>
#### `mask` {.collection-method}

Hàm `mask` sẽ che giấu một phần của chuỗi với một ký tự lặp lại và có thể được sử dụng để làm xáo trộn các phân đoạn của chuỗi như địa chỉ email hoặc các số điện thoại:

    use Illuminate\Support\Str;

    $string = Str::of('taylor@example.com')->mask('*', 3);

    // tay***************

Nếu cần, bạn cũng có thể cung cấp một số âm làm tham số thứ ba hoặc tham số thứ tư cho phương thức `mask`, điều này sẽ hướng dẫn phương thức bắt đầu tạo chuỗi ở khoảng cách nhất định tính từ cuối chuỗi trở về:

    $string = Str::of('taylor@example.com')->mask('*', -15, 3);

    // tay***@example.com

    $string = Str::of('taylor@example.com')->mask('*', 4, -4);

    // tayl**********.com

<a name="method-fluent-str-match"></a>
#### `match` {.collection-method}

Hàm `match` sẽ trả về một phần của chuỗi khớp với một biểu thức chính quy đã cho:

    use Illuminate\Support\Str;

    $result = Str::of('foo bar')->match('/bar/');

    // 'bar'

    $result = Str::of('foo bar')->match('/foo (.*)/');

    // 'bar'

<a name="method-fluent-str-match-all"></a>
#### `matchAll` {.collection-method}

Hàm `matchAll` sẽ trả về một collection chứa các phần của một chuỗi khớp với một biểu thức chính quy đã cho:

    use Illuminate\Support\Str;

    $result = Str::of('bar foo bar')->matchAll('/bar/');

    // collect(['bar', 'bar'])

Nếu bạn chỉ định một nhóm vào trong biểu thức, Laravel sẽ trả về một collection phù hợp của nhóm đó:

    use Illuminate\Support\Str;

    $result = Str::of('bar fun bar fly')->matchAll('/f(\w*)/');

    // collect(['un', 'ly']);

Nếu không tìm thấy kết quả phù hợp, một collection trống sẽ được trả về.

<a name="method-fluent-str-new-line"></a>
#### `newLine` {.collection-method}

Hàm `newLine` sẽ thêm một dòng mới vào chuỗi:

    use Illuminate\Support\Str;

    $padded = Str::of('Laravel')->newLine()->append('Framework');

    // 'Laravel
    //  Framework'

<a name="method-fluent-str-padboth"></a>
#### `padBoth` {.collection-method}

Hàm `padBoth` sẽ wrap phương thức `str_pad` của PHP, sẽ thêm vào cả hai bên của một chuỗi để thành một chuỗi khác cho đến khi chuỗi cuối cùng đạt đến độ dài mong muốn:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padBoth(10, '_');

    // '__James___'

    $padded = Str::of('James')->padBoth(10);

    // '  James   '

<a name="method-fluent-str-padleft"></a>
#### `padLeft` {.collection-method}

Hàm `padLeft` sẽ wrap phương thức `str_pad` của PHP, sẽ thêm vào bên trái của một chuỗi để thành một chuỗi khác cho đến khi chuỗi cuối cùng đạt đến độ dài mong muốn:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padLeft(10, '-=');

    // '-=-=-James'

    $padded = Str::of('James')->padLeft(10);

    // '     James'

<a name="method-fluent-str-padright"></a>
#### `padRight` {.collection-method}

Hàm `padRight` sẽ wrap phương thức `str_pad` của PHP, sẽ thêm vào bên phải của một chuỗi để thành một chuỗi khác cho đến khi chuỗi cuối cùng đạt đến độ dài mong muốn:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padRight(10, '-');

    // 'James-----'

    $padded = Str::of('James')->padRight(10);

    // 'James     '

<a name="method-fluent-str-pipe"></a>
#### `pipe` {.collection-method}

Hàm `pipe` cho phép bạn chuyển đổi chuỗi bằng cách truyền giá trị hiện tại của nó sang một hàm gọi lại:

    use Illuminate\Support\Str;

    $hash = Str::of('Laravel')->pipe('md5')->prepend('Checksum: ');

    // 'Checksum: a5c95b86291ea299fcbe64458ed12702'

    $closure = Str::of('foo')->pipe(function ($str) {
        return 'bar';
    });

    // 'bar'

<a name="method-fluent-str-plural"></a>
#### `plural` {.collection-method}

Hàm `plural` sẽ chuyển một chuỗi từ dạng số ít sang dạng số nhiều của nó. Chức năng này hỗ trợ [bất kỳ ngôn ngữ nào được hỗ trợ bộ quy tắc số nhiều của Laravel](/docs/{{version}}/localization#pluralization-language):

    use Illuminate\Support\Str;

    $plural = Str::of('car')->plural();

    // cars

    $plural = Str::of('child')->plural();

    // children

Bạn có thể cung cấp một số nguyên làm tham số thứ hai cho phương thức để lấy ra dạng số ít hoặc số nhiều của chuỗi:

    use Illuminate\Support\Str;

    $plural = Str::of('child')->plural(2);

    // children

    $plural = Str::of('child')->plural(1);

    // child

<a name="method-fluent-str-prepend"></a>
#### `prepend` {.collection-method}

Hàm `prepend` sẽ thêm các giá trị đã cho vào chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('Framework')->prepend('Laravel ');

    // Laravel Framework

<a name="method-fluent-str-remove"></a>
#### `remove` {.collection-method}

Hàm `remove` sẽ xoá các giá trị hoặc một mảng các giá trị ra khỏi chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('Arkansas is quite beautiful!')->remove('quite');

    // Arkansas is beautiful!

You may also pass `false` as a second parameter to ignore case when removing strings.

<a name="method-fluent-str-replace"></a>
#### `replace` {.collection-method}

Hàm `replace` sẽ thay thế một chuỗi đã cho trong chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::of('Laravel 6.x')->replace('6.x', '7.x');

    // Laravel 7.x

<a name="method-fluent-str-replace-array"></a>
#### `replaceArray` {.collection-method}

Hàm `replaceArray` sẽ thay thế từng giá trị một vào trong chuỗi bằng cách sử dụng một mảng:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::of($string)->replaceArray('?', ['8:30', '9:00']);

    // The event will take place between 8:30 and 9:00

<a name="method-fluent-str-replace-first"></a>
#### `replaceFirst` {.collection-method}

Hàm `replaceFirst` sẽ thay vào chỗ xuất hiện đầu tiên của một giá trị đã cho vào trong một chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceFirst('the', 'a');

    // a quick brown fox jumps over the lazy dog

<a name="method-fluent-str-replace-last"></a>
#### `replaceLast` {.collection-method}

Hàm `replaceLast` sẽ thay vào chỗ xuất hiện cuối cùng của một giá trị đã cho vào trong một chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceLast('the', 'a');

    // the quick brown fox jumps over a lazy dog

<a name="method-fluent-str-replace-matches"></a>
#### `replaceMatches` {.collection-method}

Hàm `replaceMatches` sẽ thay thế tất cả các phần của một chuỗi mà khớp với một mẫu:

    use Illuminate\Support\Str;

    $replaced = Str::of('(+1) 501-555-1000')->replaceMatches('/[^A-Za-z0-9]++/', '')

    // '15015551000'

Hàm `replaceMatches` cũng chấp nhận một Closure sẽ được gọi với từng phần của chuỗi mà khớp với mẫu đã cho, cho phép bạn thực hiện các logic chi tiết trong closure và trả về giá trị đã thay thế:

    use Illuminate\Support\Str;

    $replaced = Str::of('123')->replaceMatches('/\d/', function ($match) {
        return '['.$match[0].']';
    });

    // '[1][2][3]'

<a name="method-fluent-str-rtrim"></a>
#### `rtrim` {.collection-method}

Hàm `rtrim` sẽ cắt bên phải của chuỗi đã cho:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->rtrim();

    // '  Laravel'

    $string = Str::of('/Laravel/')->rtrim('/');

    // '/Laravel'

<a name="method-fluent-str-scan"></a>
#### `scan` {.collection-method}

Hàm `scan` sẽ phân tích cú pháp đầu vào của một chuỗi thành một collection theo định dạng được hỗ trợ bởi [hàm `sscanf` của PHP](https://www.php.net/manual/en/function.sscanf.php):

    use Illuminate\Support\Str;

    $collection = Str::of('filename.jpg')->scan('%[^.].%s');

    // collect(['filename', 'jpg'])

<a name="method-fluent-str-singular"></a>
#### `singular` {.collection-method}

Hàm `singular` sẽ chuyển một chuỗi thành dạng số ít của nó. Chức năng này hỗ trợ [bất kỳ ngôn ngữ nào được hỗ trợ bộ quy tắc số nhiều của Laravel](/docs/{{version}}/localization#pluralization-language):

    use Illuminate\Support\Str;

    $singular = Str::of('cars')->singular();

    // car

    $singular = Str::of('children')->singular();

    // child

<a name="method-fluent-str-slug"></a>
#### `slug` {.collection-method}

Hàm `slug` sẽ tạo ra một "slug" thân thiện với URL từ một chuỗi đã cho:

    use Illuminate\Support\Str;

    $slug = Str::of('Laravel Framework')->slug('-');

    // laravel-framework

<a name="method-fluent-str-snake"></a>
#### `snake` {.collection-method}

Hàm `snake` sẽ chuyển chuỗi đã cho thành `snake_case`:

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->snake();

    // foo_bar

<a name="method-fluent-str-split"></a>
#### `split` {.collection-method}

Hàm `split` sẽ cắt một chuỗi thành một collection bằng cách sử dụng một biểu thức chính quy:

    use Illuminate\Support\Str;

    $segments = Str::of('one, two, three')->split('/[\s,]+/');

    // collect(["one", "two", "three"])

<a name="method-fluent-str-squish"></a>
#### `squish` {.collection-method}

Hàm `squish` sẽ loại bỏ tất cả các khoảng trắng không cần thiết có trong một chuỗi, bao gồm cả các khoảng trắng không liên quan giữa các từ đó:

    use Illuminate\Support\Str;

    $string = Str::of('    laravel    framework    ')->squish();

    // laravel framework

<a name="method-fluent-str-start"></a>
#### `start` {.collection-method}

Hàm `start` sẽ thêm một giá trị đã cho vào một chuỗi nếu nó chưa bắt đầu bằng giá trị đó:

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->start('/');

    // /this/string

    $adjusted = Str::of('/this/string')->start('/');

    // /this/string

<a name="method-fluent-str-starts-with"></a>
#### `startsWith` {.collection-method}

Hàm `startsWith` sẽ xác định xem chuỗi đã cho có bắt đầu bằng giá trị đã cho hay không:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->startsWith('This');

    // true

<a name="method-fluent-str-studly"></a>
#### `studly` {.collection-method}

Hàm `studly` sẽ chuyển chuỗi đã cho thành dạng `StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->studly();

    // FooBar

<a name="method-fluent-str-substr"></a>
#### `substr` {.collection-method}

Hàm `substr` sẽ trả lại phần chuỗi được chỉ định bởi các tham số bắt đầu và độ dài của chuỗi cần lấy:

    use Illuminate\Support\Str;

    $string = Str::of('Laravel Framework')->substr(8);

    // Framework

    $string = Str::of('Laravel Framework')->substr(8, 5);

    // Frame

<a name="method-fluent-str-substrreplace"></a>
#### `substrReplace` {.collection-method}

Hàm `substrReplace` sẽ thay thế text có trong một phần của chuỗi, bắt đầu từ vị trí được chỉ định bởi tham số thứ hai và thay thế số ký tự được chỉ định bởi tham số thứ ba. Truyền tham số thứ ba là `0` nếu muốn chèn chuỗi vào vị trí đã chỉ định mà không thay thế bất kỳ ký tự nào có trong chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('1300')->substrReplace(':', 2);

    // 13:

    $string = Str::of('The Framework')->substrReplace(' Laravel', 3, 0);

    // The Laravel Framework

<a name="method-fluent-str-swap"></a>
#### `swap` {.collection-method}

Hàm `swap` sẽ thay thế nhiều giá trị trong chuỗi đã cho bằng hàm `strtr` của PHP:

    use Illuminate\Support\Str;

    $string = Str::of('Tacos are great!')
        ->swap([
            'Tacos' => 'Burritos',
            'great' => 'fantastic',
        ]);

    // Burritos are fantastic!

<a name="method-fluent-str-tap"></a>
#### `tap` {.collection-method}

Hàm `tap` sẽ truyền chuỗi đến một closure đã cho, cho phép bạn kiểm tra và tương tác với chuỗi trong khi không ảnh hưởng đến chính chuỗi đó. Chuỗi ban đầu sẽ được trả về bởi phương thức `tap` bất kể giá trị trả về của closure là thế nào:

    use Illuminate\Support\Str;

    $string = Str::of('Laravel')
        ->append(' Framework')
        ->tap(function ($string) {
            dump('String after append: '.$string);
        })
        ->upper();

    // LARAVEL FRAMEWORK

<a name="method-fluent-str-test"></a>
#### `test` {.collection-method}

Hàm `test` sẽ xác định xem một chuỗi có khớp với một biểu thức chính quy đã cho hay không:

    use Illuminate\Support\Str;

    $result = Str::of('Laravel Framework')->test('/Laravel/');

    // true

<a name="method-fluent-str-title"></a>
#### `title` {.collection-method}

Hàm `title` sẽ chuyển một chuỗi đã cho thành dạng `Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::of('a nice title uses the correct case')->title();

    // A Nice Title Uses The Correct Case

<a name="method-fluent-str-trim"></a>
#### `trim` {.collection-method}

Hàm `trim` sẽ cắt chuỗi đã cho:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->trim();

    // 'Laravel'

    $string = Str::of('/Laravel/')->trim('/');

    // 'Laravel'

<a name="method-fluent-str-ucfirst"></a>
#### `ucfirst` {.collection-method}

Hàm `ucfirst` sẽ trả về chuỗi đã cho với ký tự đầu tiên được viết hoa:

    use Illuminate\Support\Str;

    $string = Str::of('foo bar')->ucfirst();

    // Foo bar

<a name="method-fluent-str-ucsplit"></a>
#### `ucsplit` {.collection-method}

Hàm `ucsplit` sẽ chia chuỗi đã cho thành một collection theo các ký tự được viết hoa:

    use Illuminate\Support\Str;

    $string = Str::of('Foo Bar')->ucsplit();

    // collect(['Foo', 'Bar'])

<a name="method-fluent-str-upper"></a>
#### `upper` {.collection-method}

Hàm `upper` sẽ chuyển một chuỗi đã cho thành viết chữ hoa toàn bộ chuỗi:

    use Illuminate\Support\Str;

    $adjusted = Str::of('laravel')->upper();

    // LARAVEL

<a name="method-fluent-str-when"></a>
#### `when` {.collection-method}

Hàm `when` sẽ gọi Closure nếu một điều kiện đã cho là `đúng`. Closure sẽ nhận vào một instance chuỗi ban đầu:

    use Illuminate\Support\Str;

    $string = Str::of('Taylor')
                    ->when(true, function ($string) {
                        return $string->append(' Otwell');
                    });

    // 'Taylor Otwell'

Nếu cần thiết, bạn có thể truyền một Closure khác làm tham số thứ ba cho phương thức `when`. Closure này sẽ được thực thi nếu tham số điều kiện là `false`.

<a name="method-fluent-str-when-contains"></a>
#### `whenContains` {.collection-method}

Hàm `whenContains` sẽ gọi closure đã cho nếu chuỗi chứa giá trị đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('tony stark')
                ->whenContains('tony', function ($string) {
                    return $string->title();
                });

    // 'Tony Stark'

Nếu cần, bạn có thể truyền một closure khác làm tham số thứ ba cho phương thức `when`. Closure này sẽ thực hiện nếu chuỗi không chứa giá trị đã cho.

Bạn cũng có thể truyền một mảng các giá trị để xác định xem chuỗi đã cho có chứa bất kỳ giá trị nào có trong mảng hay không:

    use Illuminate\Support\Str;

    $string = Str::of('tony stark')
                ->whenContains(['tony', 'hulk'], function ($string) {
                    return $string->title();
                });

    // Tony Stark

<a name="method-fluent-str-when-contains-all"></a>
#### `whenContainsAll` {.collection-method}

Hàm `whenContainsAll` sẽ gọi closure nếu chuỗi chứa tất cả các chuỗi con đã cho. Closure này sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('tony stark')
                    ->whenContainsAll(['tony', 'stark'], function ($string) {
                        return $string->title();
                    });

    // 'Tony Stark'

Nếu cần thiết, bạn có thể truyền một closure khác làm tham số thứ ba cho phương thức `when`. Closure này sẽ được thực thi nếu tham số điều kiện là `false`.

<a name="method-fluent-str-when-empty"></a>
#### `whenEmpty` {.collection-method}

Hàm `whenEmpty` sẽ gọi một closure nếu chuỗi là trống. Nếu closure trả về một giá trị, thì giá trị đó cũng sẽ được trả về từ phương thức `whenEmpty`. Nếu closure không trả về giá trị nào, thì instance string sẽ được trả về:

    use Illuminate\Support\Str;

    $string = Str::of('  ')->whenEmpty(function ($string) {
        return $string->trim()->prepend('Laravel');
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-empty"></a>
#### `whenNotEmpty` {.collection-method}

Hàm `whenNotEmpty` sẽ gọi closure đã cho nếu chuỗi đã cho có giá trị. Nếu closure trả về một giá trị, thì giá trị đó cũng sẽ được trả về bởi phương thức `whenNotEmpty`. Nếu closure không trả về giá trị gì, thì instance chuỗi ban đầu sẽ được trả về:

    use Illuminate\Support\Str;

    $string = Str::of('Framework')->whenNotEmpty(function ($string) {
        return $string->prepend('Laravel ');
    });

    // 'Laravel Framework'

<a name="method-fluent-str-when-starts-with"></a>
#### `whenStartsWith` {.collection-method}

Hàm `whenStartsWith` sẽ gọi closure đã cho nếu chuỗi bắt đầu bằng chuỗi con đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('disney world')->whenStartsWith('disney', function ($string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-ends-with"></a>
#### `whenEndsWith` {.collection-method}

Hàm `whenEndsWith` sẽ gọi closure đã cho nếu chuỗi kết thúc bằng chuỗi con đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('disney world')->whenEndsWith('world', function ($string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-exactly"></a>
#### `whenExactly` {.collection-method}

Hàm `whenExactly` sẽ gọi closure đã cho nếu chuỗi đúng bằng chuỗi đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('laravel')->whenExactly('laravel', function ($string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-exactly"></a>
#### `whenNotExactly` {.collection-method}

Hàm `whenNotExactly` sẽ gọi một closure đã cho nếu chuỗi đó không giống với chuỗi đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('framework')->whenNotExactly('laravel', function ($string) {
        return $string->title();
    });

    // 'Framework'

<a name="method-fluent-str-when-is"></a>
#### `whenIs` {.collection-method}

Hàm `whenIs` sẽ gọi closure đã cho nếu chuỗi khớp với một mẫu nhất định. Dấu hoa thị có thể được sử dụng làm một ký tự đại diện. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('foo/bar')->whenIs('foo/*', function ($string) {
        return $string->append('/baz');
    });

    // 'foo/bar/baz'

<a name="method-fluent-str-when-is-ascii"></a>
#### `whenIsAscii` {.collection-method}

Hàm `whenIsAscii` sẽ gọi closure đã cho nếu chuỗi là một dạng ASCII 7 bit. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('laravel')->whenIsAscii(function ($string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-is-ulid"></a>
#### `whenIsUlid` {.collection-method}

Hàm `whenIsUlid` sẽ gọi một closure nếu chuỗi đã cho là một ULID hợp lệ. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('01gd6r360bp37zj17nxb55yv40')->whenIsUlid(function ($string) {
        return $string->substr(0, 8);
    });

    // '01gd6r36'

<a name="method-fluent-str-when-is-uuid"></a>
#### `whenIsUuid` {.collection-method}

Hàm `whenIsUuid` sẽ gọi closure đã cho nếu chuỗi là một UUID hợp lệ. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('a0a2a2d2-0b87-4a18-83f2-2529882be2de')->whenIsUuid(function ($string) {
        return $string->substr(0, 8);
    });

    // 'a0a2a2d2'

<a name="method-fluent-str-when-test"></a>
#### `whenTest` {.collection-method}

Hàm `whenTest` sẽ gọi closure đã cho nếu chuỗi khớp với một biểu thức chính quy. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('laravel framework')->whenTest('/laravel/', function ($string) {
        return $string->title();
    });

    // 'Laravel Framework'

<a name="method-fluent-str-word-count"></a>
#### `wordCount` {.collection-method}

Hàm `wordCount` trả về số lượng từ mà một chuỗi đó chứa:

```php
use Illuminate\Support\Str;

Str::of('Hello, world!')->wordCount(); // 2
```

<a name="method-fluent-str-words"></a>
#### `words` {.collection-method}

Hàm `words` sẽ giới hạn số lượng từ trong một chuỗi. Nếu cần, bạn có thể chỉ định một chuỗi bổ sung sẽ được thêm vào chuỗi bị cắt ngắn:

    use Illuminate\Support\Str;

    $string = Str::of('Perfectly balanced, as all things should be.')->words(3, ' >>>');

    // Perfectly balanced, as >>>

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

> **Warning**
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

    return optional(User::find($id), function ($user) {
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

    return retry(5, function () {
        // ...
    }, function ($attempt, $exception) {
        return $attempt * 100;
    });

Để thuận tiện, bạn cũng có thể cung cấp một mảng làm tham số đầu tiên cho hàm `retry`. Mảng này sẽ được sử dụng để xác định số mili giây sẽ ngủ giữa các lần thử tiếp theo:

    return retry([100, 200], function () {
        // Sleep for 100ms on first retry, 200ms on second retry...
    });

Để chỉ thử lại trong một điều kiện cụ thể, bạn có thể truyền một closure làm tham số thứ tư cho hàm `retry`:

    return retry(5, function () {
        // ...
    }, 100, function ($exception) {
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

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

Nếu không có closure nào được truyền đến hàm `tap`, bạn có thể gọi bất kỳ phương thức nào trên `$value` đã cho. Giá trị trả về của phương thức bạn gọi sẽ luôn là `$value`, bất kể phương thức đó thực sự trả về định nghĩa gì đi chăng nữa. Ví dụ, phương thức `update` Eloquent thường trả về một số nguyên. Tuy nhiên, chúng ta có thể buộc phương thức này trả về chính model đó bằng cách gọi phương thức `update` thông qua hàm `tap`:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

Để thêm một phương thức `tap` vào một class, bạn có thể thêm trait `Illuminate\Support\Traits\Tappable` vào class. Hàm `tap` của trait này sẽ chấp nhận một Closure làm tham số duy nhất của nó. Chính instance đối tượng sẽ được truyền đến Closure và sau đó được trả về bởi phương thức `tap`:

    return $user->tap(function ($user) {
        //
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

    $callback = function ($value) {
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

    $result = value(function ($name) {
        return $parameter;
    }, 'Taylor');

    // 'Taylor'

<a name="method-view"></a>
#### `view()` {.collection-method}

Hàm `view` sẽ lấy ra một instance [view](/docs/{{version}}/views):

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {.collection-method}

Hàm `with` sẽ trả về giá trị được cho. Nếu một closure được truyền làm tham số thứ hai cho hàm, thì closure đó sẽ được thực thi và giá trị trả về của nó sẽ được trả về:

    $callback = function ($value) {
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
