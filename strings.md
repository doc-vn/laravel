# Strings

- [Giới thiệu](#introduction)
- [Các phương thức có sẵn](#available-methods)

<a name="introduction"></a>
## Giới thiệu

Laravel có chứa nhiều hàm khác nhau để thao tác với các giá trị string. Nhiều hàm trong số này được chính framework sử dụng; tuy nhiên, bạn có thể tự do sử dụng chúng trong ứng dụng của bạn nếu bạn thấy thuận tiện.

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

<a name="strings-method-list"></a>
### Strings

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[class_basename](#method-class-basename)
[e](#method-e)
[preg_replace_array](#method-preg-replace-array)
[Str::after](#method-str-after)
[Str::afterLast](#method-str-after-last)
[Str::apa](#method-str-apa)
[Str::ascii](#method-str-ascii)
[Str::before](#method-str-before)
[Str::beforeLast](#method-str-before-last)
[Str::between](#method-str-between)
[Str::betweenFirst](#method-str-between-first)
[Str::camel](#method-camel-case)
[Str::charAt](#method-char-at)
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
[Str::isUrl](#method-str-is-url)
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
[Str::password](#method-str-password)
[Str::plural](#method-str-plural)
[Str::pluralStudly](#method-str-plural-studly)
[Str::position](#method-str-position)
[Str::random](#method-str-random)
[Str::remove](#method-str-remove)
[Str::repeat](#method-str-repeat)
[Str::replace](#method-str-replace)
[Str::replaceArray](#method-str-replace-array)
[Str::replaceFirst](#method-str-replace-first)
[Str::replaceLast](#method-str-replace-last)
[Str::replaceMatches](#method-str-replace-matches)
[Str::replaceStart](#method-str-replace-start)
[Str::replaceEnd](#method-str-replace-end)
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
[Str::take](#method-take)
[Str::title](#method-title-case)
[Str::toBase64](#method-str-to-base64)
[Str::toHtmlString](#method-str-to-html-string)
[Str::ucfirst](#method-str-ucfirst)
[Str::ucsplit](#method-str-ucsplit)
[Str::upper](#method-str-upper)
[Str::ulid](#method-str-ulid)
[Str::unwrap](#method-str-unwrap)
[Str::uuid](#method-str-uuid)
[Str::wordCount](#method-str-word-count)
[Str::wordWrap](#method-str-word-wrap)
[Str::words](#method-str-words)
[Str::wrap](#method-str-wrap)
[str](#method-str)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

<a name="fluent-strings-method-list"></a>
### Fluent Strings

<div class="collection-method-list" markdown="1">

[after](#method-fluent-str-after)
[afterLast](#method-fluent-str-after-last)
[apa](#method-fluent-str-apa)
[append](#method-fluent-str-append)
[ascii](#method-fluent-str-ascii)
[basename](#method-fluent-str-basename)
[before](#method-fluent-str-before)
[beforeLast](#method-fluent-str-before-last)
[between](#method-fluent-str-between)
[betweenFirst](#method-fluent-str-between-first)
[camel](#method-fluent-str-camel)
[charAt](#method-fluent-str-char-at)
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
[isUrl](#method-fluent-str-is-url)
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
[isMatch](#method-fluent-str-is-match)
[newLine](#method-fluent-str-new-line)
[padBoth](#method-fluent-str-padboth)
[padLeft](#method-fluent-str-padleft)
[padRight](#method-fluent-str-padright)
[pipe](#method-fluent-str-pipe)
[plural](#method-fluent-str-plural)
[position](#method-fluent-str-position)
[prepend](#method-fluent-str-prepend)
[remove](#method-fluent-str-remove)
[repeat](#method-fluent-str-repeat)
[replace](#method-fluent-str-replace)
[replaceArray](#method-fluent-str-replace-array)
[replaceFirst](#method-fluent-str-replace-first)
[replaceLast](#method-fluent-str-replace-last)
[replaceMatches](#method-fluent-str-replace-matches)
[replaceStart](#method-fluent-str-replace-start)
[replaceEnd](#method-fluent-str-replace-end)
[rtrim](#method-fluent-str-rtrim)
[scan](#method-fluent-str-scan)
[singular](#method-fluent-str-singular)
[slug](#method-fluent-str-slug)
[snake](#method-fluent-str-snake)
[split](#method-fluent-str-split)
[squish](#method-fluent-str-squish)
[start](#method-fluent-str-start)
[startsWith](#method-fluent-str-starts-with)
[stripTags](#method-fluent-str-strip-tags)
[studly](#method-fluent-str-studly)
[substr](#method-fluent-str-substr)
[substrReplace](#method-fluent-str-substrreplace)
[swap](#method-fluent-str-swap)
[take](#method-fluent-str-take)
[tap](#method-fluent-str-tap)
[test](#method-fluent-str-test)
[title](#method-fluent-str-title)
[toBase64](#method-fluent-str-to-base64)
[trim](#method-fluent-str-trim)
[ucfirst](#method-fluent-str-ucfirst)
[ucsplit](#method-fluent-str-ucsplit)
[unwrap](#method-fluent-str-unwrap)
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

<a name="strings"></a>
## Strings

<a name="method-__"></a>
#### `__()` {.collection-method}

Hàm `__` sẽ dịch chuỗi cần được dịch hoặc key cần được dịch đã cho bằng cách sử dụng [language files](/docs/{{version}}/localization) của bạn:

    echo __('Welcome to our application');

    echo __('messages.welcome');

Nếu chuỗi hoặc key cần được dịch không tồn tại, hàm `__` sẽ trả về giá trị được đưa vào. Vì vậy, nếu sử dụng ví dụ mẫu trên, hàm `__` sẽ trả về `messages.welcome` nếu key cần được dịch đó không tồn tại.

<a name="method-class-basename"></a>
#### `class_basename()` {.collection-method}

Hàm `class_basename` sẽ trả về tên class đã cho với namespace của class bị xóa:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {.collection-method}

Hàm `e` sẽ chạy hàm `htmlspecialchars` của PHP với tùy chọn `double_encode` được set mặc định thành `true`:

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

<a name="method-str-apa"></a>
#### `Str::apa()` {.collection-method}

Hàm `Str::apa` sẽ chuyển chuỗi đã cho thành chữ viết hoa đầu dòng theo [hướng dẫn của APA](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case):

    use Illuminate\Support\Str;

    $title = Str::apa('Creating A Project');

    // 'Creating a Project'

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

Hàm `Str::camel` sẽ chuyển chuỗi đã cho thành `camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::camel('foo_bar');

    // 'fooBar'

<a name="method-char-at"></a>
#### `Str::charAt()` {.collection-method}

Hàm `Str::charAt` sẽ trả về một ký tự tại vị trí được chỉ định. Nếu vị trí chỉ định nằm ngoài chuỗi, thì `false` sẽ được trả về:

    use Illuminate\Support\Str;

    $character = Str::charAt('This is my name.', 6);

    // 's'

<a name="method-str-contains"></a>
#### `Str::contains()` {.collection-method}

Hàm `Str::contains` sẽ xác định xem chuỗi đã cho có chứa giá trị đã cho hay không. Phương thức này phân biệt chữ hoa và chữ thường:

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

#### Markdown Security

Mặc định, Markdown hỗ trợ HTML raw, điều này sẽ tạo ra lỗ hổng Cross-Site Scripting (XSS) khi sử dụng với dữ liệu input raw của người dùng. Theo [tài liệu bảo mật CommonMark](https://commonmark.thephpleague.com/security/), bạn có thể sử dụng tùy chọn `html_input` để loại bỏ ký tự đặc biệt hoặc loại bỏ thẻ HTML, và tùy chọn `allow_unsafe_links` để chỉ định có cho phép các unsafe link hay không. Nếu bạn cần cho phép một số HTML raw, bạn nên truyền Markdown đã biên dịch của bạn qua HTML Purifier:

    use Illuminate\Support\Str;

    Str::inlineMarkdown('Inject: <script>alert("Hello XSS!");</script>', [
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);

    // Inject: alert(&quot;Hello XSS!&quot;);

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

<a name="method-str-is-url"></a>
#### `Str::isUrl()` {.collection-method}

Hàm `Str::isUrl` sẽ xác định xem chuỗi đã cho có phải là URL hợp lệ hay không:

    use Illuminate\Support\Str;

    $isUrl = Str::isUrl('http://example.com');

    // true

    $isUrl = Str::isUrl('laravel');

    // false

Hàm `isUrl` chấp nhận nhiều giao thức. Tuy nhiên, bạn có thể chỉ định các giao thức được coi là hợp lệ bằng cách cung cấp chúng cho hàm `isUrl`:

    $isUrl = Str::isUrl('http://example.com', ['http', 'https']);

<a name="method-str-is-ulid"></a>
#### `Str::isUlid()` {.collection-method}

Hàm `Str::isUlid` sẽ kiểm tra xem chuỗi đã cho có phải là một ULID hợp lệ hay không:

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

Hàm `Str::kebab` chuyển chuỗi đã cho thành `kebab-case`:

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

#### Markdown Security

Mặc định, Markdown hỗ trợ HTML raw, điều này sẽ tạo ra lỗ hổng Cross-Site Scripting (XSS) khi sử dụng với dữ liệu input raw của người dùng. Theo [tài liệu bảo mật CommonMark](https://commonmark.thephpleague.com/security/), bạn có thể sử dụng tùy chọn `html_input` để loại bỏ ký tự đặc biệt hoặc loại bỏ thẻ HTML, và tùy chọn `allow_unsafe_links` để chỉ định có cho phép các unsafe link hay không. Nếu bạn cần cho phép một số HTML raw, bạn nên truyền Markdown đã biên dịch của bạn qua HTML Purifier:

    use Illuminate\Support\Str;

    Str::markdown('Inject: <script>alert("Hello XSS!");</script>', [
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);

    // <p>Inject: alert(&quot;Hello XSS!&quot;);</p>

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

<a name="method-str-password"></a>
#### `Str::password()` {.collection-method}

Hàm `Str::password` sẽ có thể được sử dụng để tạo ra một mật khẩu an toàn với độ dài nhất định. Mật khẩu sẽ chứa một tổ hợp các chữ cái, chữ số, ký hiệu và khoảng trắng. Mặc định, mật khẩu sẽ dài 32 ký tự:

    use Illuminate\Support\Str;

    $password = Str::password();

    // 'EbJo2vE-AS:U,$%_gkrV4n,q~1xy/-_4'

    $password = Str::password(12);

    // 'qwuar>#V|i]N'

<a name="method-str-plural"></a>
#### `Str::plural()` {.collection-method}

Hàm `Str::plural` sẽ chuyển một chuỗi đơn thành dạng số nhiều của nó. Chức năng này hỗ trợ [bất kỳ ngôn ngữ nào được hỗ trợ bộ quy tắc số nhiều của Laravel](/docs/{{version}}/localization#pluralization-language):

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

<a name="method-str-position"></a>
#### `Str::position()` {.collection-method}

Hàm `Str::position` sẽ trả về vị trí xuất hiện đầu tiên của một chuỗi trong một chuỗi khác. Nếu chuỗi cần tìm không tồn tại trong chuỗi đã cho, thì giá trị `false` sẽ được trả về:

    use Illuminate\Support\Str;

    $position = Str::position('Hello, World!', 'Hello');

    // 0

    $position = Str::position('Hello, World!', 'W');

    // 7

<a name="method-str-random"></a>
#### `Str::random()` {.collection-method}

Hàm `Str::random` sẽ tạo ra một chuỗi ngẫu nhiên có độ dài được chỉ định. Hàm này sử dụng hàm `random_bytes` của PHP:

    use Illuminate\Support\Str;

    $random = Str::random(40);

Trong quá trình testing, việc "fake" các giá trị được trả về bởi phương thức `Str::random` có thể hữu ích. Để thực hiện việc này, bạn có thể sử dụng phương thức `createRandomStringsUsing`:

    Str::createRandomStringsUsing(function () {
        return 'fake-random-string';
    });

Để bảo phương thức `random` quay lại tạo một chuỗi ngẫu nhiên một cách bình thường, bạn có thể gọi phương thức `createRandomStringsNormally`:

    Str::createRandomStringsNormally();

<a name="method-str-remove"></a>
#### `Str::remove()` {.collection-method}

Hàm `Str::remove` sẽ xoá các giá trị hoặc một mảng các giá trị ra khỏi chuỗi:

    use Illuminate\Support\Str;

    $string = 'Peter Piper picked a peck of pickled peppers.';

    $removed = Str::remove('e', $string);

    // Ptr Pipr pickd a pck of pickld ppprs.

Bạn cũng có thể truyền một tham số `false` làm tham số thứ ba cho phương thức `remove` để xoá cả chữ hoa chữ thường.

<a name="method-str-repeat"></a>
#### `Str::repeat()` {.collection-method}

Hàm `Str::repeat` sẽ lặp lại chuỗi đã cho:

```php
use Illuminate\Support\Str;

$string = 'a';

$repeat = Str::repeat($string, 5);

// aaaaa
```

<a name="method-str-replace"></a>
#### `Str::replace()` {.collection-method}

Hàm `Str::replace` sẽ thay thế một chuỗi trong chuỗi:

    use Illuminate\Support\Str;

    $string = 'Laravel 8.x';

    $replaced = Str::replace('8.x', '9.x', $string);

    // Laravel 9.x

Phương thức `replace` cũng chấp nhận tham số `caseSensitive`. Mặc định, phương thức `replace` sẽ phân biệt chữ hoa và chữ thường:

    Str::replace('Framework', 'Laravel', caseSensitive: false);

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

<a name="method-str-replace-matches"></a>
#### `Str::replaceMatches()` {.collection-method}

Hàm `Str::replaceMatches` sẽ thay thế tất cả các phần của chuỗi mà khớp với pattern bằng một chuỗi thay thế đã cho:

    use Illuminate\Support\Str;

    $replaced = Str::replaceMatches(
        pattern: '/[^A-Za-z0-9]++/',
        replace: '',
        subject: '(+1) 501-555-1000'
    )

    // '15015551000'

Phương thức `replaceMatches` cũng chấp nhận một closure sẽ được gọi với mỗi phần của chuỗi khớp với pattern đã cho, cho phép bạn thực hiện logic thay thế trong closure và trả về giá trị muốn thay thế:

    use Illuminate\Support\Str;

    $replaced = Str::replaceMatches('/\d/', function (array $matches) {
        return '['.$matches[0].']';
    }, '123');

    // '[1][2][3]'

<a name="method-str-replace-start"></a>
#### `Str::replaceStart()` {.collection-method}

Hàm `Str::replaceStart` sẽ chỉ thay thế phần tử đầu tiên của chuỗi được chỉ định nếu giá trị cần thay thế xuất hiện ở đầu chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::replaceStart('Hello', 'Laravel', 'Hello World');

    // Laravel World

    $replaced = Str::replaceStart('World', 'Laravel', 'Hello World');

    // Hello World

<a name="method-str-replace-end"></a>
#### `Str::replaceEnd()` {.collection-method}

Hàm `Str::replaceEnd` sẽ chỉ thay thế phần tử cuối cùng của chuỗi đã cho nếu giá trị cần thay thế xuất hiện ở cuối chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::replaceEnd('World', 'Laravel', 'Hello World');

    // Hello Laravel

    $replaced = Str::replaceEnd('Hello', 'Laravel', 'Hello World');

    // Hello World

<a name="method-str-reverse"></a>
#### `Str::reverse()` {.collection-method}

Hàm `Str::reverse` sẽ đảo ngược chuỗi đã cho:

    use Illuminate\Support\Str;

    $reversed = Str::reverse('Hello World');

    // dlroW olleH

<a name="method-str-singular"></a>
#### `Str::singular()` {.collection-method}

Hàm `Str::singular` sẽ chuyển một chuỗi thành dạng số ít của nó. Chức năng này hỗ trợ [bất kỳ ngôn ngữ nào được hỗ trợ bộ quy tắc số nhiều của Laravel](/docs/{{version}}/localization#pluralization-language):

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

Hàm `Str::snake` sẽ chuyển chuỗi đã cho thành `Str::snake`:

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

Hàm `Str::studly` chuyển chuỗi đã cho thành` StudlyCase`:

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

<a name="method-take"></a>
#### `Str::take()` {.collection-method}

Hàm `Str::take` sẽ trả về các ký tự được chỉ định tính từ đầu chuỗi:

    use Illuminate\Support\Str;

    $taken = Str::take('Build something amazing!', 5);

    // Build

<a name="method-title-case"></a>
#### `Str::title()` {.collection-method}

Hàm `Str::title` chuyển chuỗi đã cho thành` Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-to-base64"></a>
#### `Str::toBase64()` {.collection-method}

Hàm `Str::toBase64` sẽ chuyển chuỗi đã cho thành Base64:

    use Illuminate\Support\Str;

    $base64 = Str::toBase64('Laravel');

    // TGFyYXZlbA==

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

Hàm `Str::upper` sẽ chuyển chuỗi đã cho thành chữ hoa toàn bộ chuỗi:

    use Illuminate\Support\Str;

    $string = Str::upper('laravel');

    // LARAVEL

<a name="method-str-ulid"></a>
#### `Str::ulid()` {.collection-method}

Hàm `Str::ulid` sẽ tạo một mã ULID nhỏ gọn kết hợp với thời gian và có thể sắp xếp được:

    use Illuminate\Support\Str;

    return (string) Str::ulid();

    // 01gd6r360bp37zj17nxb55yv40

Nếu bạn muốn lấy ra một instance `Illuminate\Support\Carbon` theo kiểu ngày và giờ từ một ULID nhất định, bạn có thể sử dụng phương thức `createFromId` do Laravel cung cấp được tích hợp sẵn trong class Carbon:

```php
use Illuminate\Support\Carbon;
use Illuminate\Support\Str;

$date = Carbon::createFromId((string) Str::ulid());
```

Trong quá trình testing, việc "fake" các giá trị được trả về bởi phương thức `Str::ulid` có thể hữu ích. Để thực hiện việc này, bạn có thể sử dụng phương thức `createUlidsUsing`:

    use Symfony\Component\Uid\Ulid;

    Str::createUlidsUsing(function () {
        return new Ulid('01HRDBNHHCKNW2AK4Z29SN82T9');
    });

Để bảo phương thức `ulid` quay lại tạo một ULID theo cách bình thường, bạn có thể gọi phương thức `createUlidsNormally`:

    Str::createUlidsNormally();

<a name="method-str-unwrap"></a>
#### `Str::unwrap()` {.collection-method}

Hàm `Str::unwrap` sẽ xóa các chuỗi được chỉ định ra khỏi đầu hoặc cuối của một chuỗi nhất định:

    use Illuminate\Support\Str;

    Str::unwrap('-Laravel-', '-');

    // Laravel

    Str::unwrap('{framework: "Laravel"}', '{', '}');

    // framework: "Laravel"

<a name="method-str-uuid"></a>
#### `Str::uuid()` {.collection-method}

Hàm `Str::uuid` sẽ tạo ra một UUID (phiên bản 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

Trong quá trình testing, việc "fake" các giá trị được trả về bởi phương thức `Str::uuid` có thể hữu ích. Để thực hiện việc này, bạn có thể sử dụng phương thức `createUuidsUsing`:

    use Ramsey\Uuid\Uuid;

    Str::createUuidsUsing(function () {
        return Uuid::fromString('eadbfeac-5258-45c2-bab7-ccb9b5ef74f9');
    });

Để bảo phương thức `uuid` quay lại tạo một UUID theo cách bình thường, bạn có thể gọi phương thức `createUuidsNormally`:

    Str::createUuidsNormally();

<a name="method-str-word-count"></a>
#### `Str::wordCount()` {.collection-method}

Hàm `Str::wordCount` sẽ trả về số lượng từ mà một chuỗi chứa:

```php
use Illuminate\Support\Str;

Str::wordCount('Hello, world!'); // 2
```

<a name="method-str-word-wrap"></a>
#### `Str::wordWrap()` {.collection-method}

Hàm `Str::wordWrap` sẽ thêm một chuỗi vào trong một chuỗi khác ở một số vị trí nhất định:

    use Illuminate\Support\Str;

    $text = "The quick brown fox jumped over the lazy dog."

    Str::wordWrap($text, characters: 20, break: "<br />\n");

    /*
    The quick brown fox<br />
    jumped over the lazy<br />
    dog.
    */

<a name="method-str-words"></a>
#### `Str::words()` {.collection-method}

Hàm `Str::words` sẽ giới hạn số lượng từ có trong một chuỗi. Một chuỗi bổ sung có thể được truyền cho phương thức này thông qua tham số thứ ba của nó để chỉ định chuỗi nào sẽ được thêm vào cuối chuỗi bị cắt ngắn:

    use Illuminate\Support\Str;

    return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

    // Perfectly balanced, as >>>

<a name="method-str-wrap"></a>
#### `Str::wrap()` {.collection-method}

Hàm `Str::wrap` sẽ thêm vào một chuỗi đã cho bằng một chuỗi khác hoặc một cặp chuỗi khác:

    use Illuminate\Support\Str;

    Str::wrap('Laravel', '"');

    // "Laravel"

    Str::wrap('is', before: 'This ', after: ' Laravel!');

    // This is Laravel!

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

Hàm `trans` sẽ dịch các key cần dịch bằng cách sử dụng [language files](/docs/{{version}}/localization) của bạn:

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

<a name="method-fluent-str-apa"></a>
#### `apa` {.collection-method}

Hàm `apa` sẽ chuyển chuỗi đã cho thành chữ viết hoa đầu dòng theo [hướng dẫn của APA](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case):

    use Illuminate\Support\Str;

    $converted = Str::of('a nice title uses the correct case')->apa();

    // A Nice Title Uses the Correct Case

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

    // 'fooBar'

<a name="method-fluent-str-char-at"></a>
#### `charAt` {.collection-method}

Hàm `charAt` sẽ trả về một ký tự tại vị trí được chỉ định. Nếu vị trí chỉ định nằm ngoài chuỗi, thì `false` sẽ được trả về:

    use Illuminate\Support\Str;

    $character = Str::of('This is my name.')->charAt(6);

    // 's'

<a name="method-fluent-str-class-basename"></a>
#### `classBasename` {.collection-method}

Hàm `classBasename` sẽ trả về tên class của class đã cho mà namespace của class đó đã bị xóa bỏ:

    use Illuminate\Support\Str;

    $class = Str::of('Foo\Bar\Baz')->classBasename();

    // 'Baz'

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

#### Markdown Security

Mặc định, Markdown hỗ trợ HTML raw, điều này sẽ tạo ra lỗ hổng Cross-Site Scripting (XSS) khi sử dụng với dữ liệu input raw của người dùng. Theo [tài liệu bảo mật CommonMark](https://commonmark.thephpleague.com/security/), bạn có thể sử dụng tùy chọn `html_input` để loại bỏ ký tự đặc biệt hoặc loại bỏ thẻ HTML, và tùy chọn `allow_unsafe_links` để chỉ định có cho phép các unsafe link hay không. Nếu bạn cần cho phép một số HTML raw, bạn nên truyền Markdown đã biên dịch của bạn qua HTML Purifier:

    use Illuminate\Support\Str;

    Str::of('Inject: <script>alert("Hello XSS!");</script>')->inlineMarkdown([
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);

    // Inject: alert(&quot;Hello XSS!&quot;);

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

<a name="method-fluent-str-is-url"></a>
#### `isUrl` {.collection-method}

Hàm `isUrl` sẽ xác định xem một chuỗi nhất định có phải là một URL hay không:

    use Illuminate\Support\Str;

    $result = Str::of('http://example.com')->isUrl();

    // true

    $result = Str::of('Taylor')->isUrl();

    // false

Hàm `isUrl` chấp nhận nhiều giao thức. Tuy nhiên, bạn có thể chỉ định các giao thức được coi là hợp lệ bằng cách cung cấp chúng cho hàm `isUrl`:

    $result = Str::of('http://example.com')->isUrl(['http', 'https']);

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

#### Markdown Security

Mặc định, Markdown hỗ trợ HTML raw, điều này sẽ tạo ra lỗ hổng Cross-Site Scripting (XSS) khi sử dụng với dữ liệu input raw của người dùng. Theo [tài liệu bảo mật CommonMark](https://commonmark.thephpleague.com/security/), bạn có thể sử dụng tùy chọn `html_input` để loại bỏ ký tự đặc biệt hoặc loại bỏ thẻ HTML, và tùy chọn `allow_unsafe_links` để chỉ định có cho phép các unsafe link hay không. Nếu bạn cần cho phép một số HTML raw, bạn nên truyền Markdown đã biên dịch của bạn qua HTML Purifier:

    use Illuminate\Support\Str;

    Str::of('Inject: <script>alert("Hello XSS!");</script>')->markdown([
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);

    // <p>Inject: alert(&quot;Hello XSS!&quot;);</p>

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

<a name="method-fluent-str-is-match"></a>
#### `isMatch` {.collection-method}

Hàm `isMatch` sẽ trả về giá trị `true` nếu chuỗi khớp với một biểu thức chính quy nhất định:

    use Illuminate\Support\Str;

    $result = Str::of('foo bar')->isMatch('/foo (.*)/');

    // true

    $result = Str::of('laravel')->isMatch('/foo (.*)/');

    // false

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
    use Illuminate\Support\Stringable;

    $hash = Str::of('Laravel')->pipe('md5')->prepend('Checksum: ');

    // 'Checksum: a5c95b86291ea299fcbe64458ed12702'

    $closure = Str::of('foo')->pipe(function (Stringable $str) {
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

<a name="method-fluent-str-position"></a>
#### `position` {.collection-method}

Hàm `position` sẽ trả về vị trí xuất hiện đầu tiên của một chuỗi trong một chuỗi khác. Nếu chuỗi cần tìm không tồn tại trong chuỗi đã cho, thì giá trị `false` sẽ được trả về:

    use Illuminate\Support\Str;

    $position = Str::of('Hello, World!')->position('Hello');

    // 0

    $position = Str::of('Hello, World!')->position('W');

    // 7

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

Bạn cũng có thể truyền một tham số `false` làm tham số thứ hai cho phương thức `remove` để xoá cả chữ hoa chữ thường.

<a name="method-fluent-str-repeat"></a>
#### `repeat` {.collection-method}

Hàm `repeat` sẽ lặp lại chuỗi đã cho:

```php
use Illuminate\Support\Str;

$repeated = Str::of('a')->repeat(5);

// aaaaa
```

<a name="method-fluent-str-replace"></a>
#### `replace` {.collection-method}

Hàm `replace` sẽ thay thế một chuỗi đã cho trong chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::of('Laravel 6.x')->replace('6.x', '7.x');

    // Laravel 7.x

Phương thức `replace` cũng chấp nhận tham số `caseSensitive`. Mặc định, phương thức `replace` sẽ phân biệt chữ hoa và chữ thường:

    $replaced = Str::of('macOS 13.x')->replace(
        'macOS', 'iOS', caseSensitive: false
    );

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

    $replaced = Str::of('123')->replaceMatches('/\d/', function (array $matches) {
        return '['.$matches[0].']';
    });

    // '[1][2][3]'

<a name="method-fluent-str-replace-start"></a>
#### `replaceStart` {.collection-method}

Hàm `replaceStart` sẽ chỉ thay thế phần tử đầu tiên của chuỗi được chỉ định nếu giá trị cần thay thế xuất hiện ở đầu chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::of('Hello World')->replaceStart('Hello', 'Laravel');

    // Laravel World

    $replaced = Str::of('Hello World')->replaceStart('World', 'Laravel');

    // Hello World

<a name="method-fluent-str-replace-end"></a>
#### `replaceEnd` {.collection-method}

Hàm `replaceEnd` sẽ chỉ thay thế phần tử cuối cùng của chuỗi đã cho nếu giá trị cần thay thế xuất hiện ở cuối chuỗi:

    use Illuminate\Support\Str;

    $replaced = Str::of('Hello World')->replaceEnd('World', 'Laravel');

    // Hello Laravel

    $replaced = Str::of('Hello World')->replaceEnd('Hello', 'Laravel');

    // Hello World

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

<a name="method-fluent-str-strip-tags"></a>
#### `stripTags` {.collection-method}

Hàm `stripTags` sẽ xóa tất cả các tag HTML và tag PHP ra khỏi một chuỗi:

    use Illuminate\Support\Str;

    $result = Str::of('<a href="https://laravel.com">Taylor <b>Otwell</b></a>')->stripTags();

    // Taylor Otwell

    $result = Str::of('<a href="https://laravel.com">Taylor <b>Otwell</b></a>')->stripTags('<b>');

    // Taylor <b>Otwell</b>

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

<a name="method-fluent-str-take"></a>
#### `take` {.collection-method}

Hàm `take` sẽ trả về các ký tự được chỉ định tính từ đầu chuỗi:

    use Illuminate\Support\Str;

    $taken = Str::of('Build something amazing!')->take(5);

    // Build

<a name="method-fluent-str-tap"></a>
#### `tap` {.collection-method}

Hàm `tap` sẽ truyền chuỗi đến một closure đã cho, cho phép bạn kiểm tra và tương tác với chuỗi trong khi không ảnh hưởng đến chính chuỗi đó. Chuỗi ban đầu sẽ được trả về bởi phương thức `tap` bất kể giá trị trả về của closure là thế nào:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('Laravel')
        ->append(' Framework')
        ->tap(function (Stringable $string) {
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

<a name="method-fluent-str-to-base64"></a>
#### `toBase64()` {.collection-method}

Hàm `toBase64` sẽ chuyển chuỗi đã cho thành Base64:

    use Illuminate\Support\Str;

    $base64 = Str::of('Laravel')->toBase64();

    // TGFyYXZlbA==

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

<a name="method-fluent-str-unwrap"></a>
#### `unwrap` {.collection-method}

Hàm `unwrap` sẽ xóa các chuỗi được chỉ định ra khỏi đầu hoặc cuối của một chuỗi nhất định:

    use Illuminate\Support\Str;

    Str::of('-Laravel-')->unwrap('-');

    // Laravel

    Str::of('{framework: "Laravel"}')->unwrap('{', '}');

    // framework: "Laravel"

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
    use Illuminate\Support\Stringable;

    $string = Str::of('Taylor')
                    ->when(true, function (Stringable $string) {
                        return $string->append(' Otwell');
                    });

    // 'Taylor Otwell'

Nếu cần thiết, bạn có thể truyền một Closure khác làm tham số thứ ba cho phương thức `when`. Closure này sẽ được thực thi nếu tham số điều kiện là `false`.

<a name="method-fluent-str-when-contains"></a>
#### `whenContains` {.collection-method}

Hàm `whenContains` sẽ gọi closure đã cho nếu chuỗi chứa giá trị đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                ->whenContains('tony', function (Stringable $string) {
                    return $string->title();
                });

    // 'Tony Stark'

Nếu cần, bạn có thể truyền một closure khác làm tham số thứ ba cho phương thức `when`. Closure này sẽ thực hiện nếu chuỗi không chứa giá trị đã cho.

Bạn cũng có thể truyền một mảng các giá trị để xác định xem chuỗi đã cho có chứa bất kỳ giá trị nào có trong mảng hay không:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                ->whenContains(['tony', 'hulk'], function (Stringable $string) {
                    return $string->title();
                });

    // Tony Stark

<a name="method-fluent-str-when-contains-all"></a>
#### `whenContainsAll` {.collection-method}

Hàm `whenContainsAll` sẽ gọi closure nếu chuỗi chứa tất cả các chuỗi con đã cho. Closure này sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                    ->whenContainsAll(['tony', 'stark'], function (Stringable $string) {
                        return $string->title();
                    });

    // 'Tony Stark'

Nếu cần thiết, bạn có thể truyền một closure khác làm tham số thứ ba cho phương thức `when`. Closure này sẽ được thực thi nếu tham số điều kiện là `false`.

<a name="method-fluent-str-when-empty"></a>
#### `whenEmpty` {.collection-method}

Hàm `whenEmpty` sẽ gọi một closure nếu chuỗi là trống. Nếu closure trả về một giá trị, thì giá trị đó cũng sẽ được trả về từ phương thức `whenEmpty`. Nếu closure không trả về giá trị nào, thì instance string sẽ được trả về:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('  ')->whenEmpty(function (Stringable $string) {
        return $string->trim()->prepend('Laravel');
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-empty"></a>
#### `whenNotEmpty` {.collection-method}

Hàm `whenNotEmpty` sẽ gọi closure đã cho nếu chuỗi đã cho có giá trị. Nếu closure trả về một giá trị, thì giá trị đó cũng sẽ được trả về bởi phương thức `whenNotEmpty`. Nếu closure không trả về giá trị gì, thì instance chuỗi ban đầu sẽ được trả về:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('Framework')->whenNotEmpty(function (Stringable $string) {
        return $string->prepend('Laravel ');
    });

    // 'Laravel Framework'

<a name="method-fluent-str-when-starts-with"></a>
#### `whenStartsWith` {.collection-method}

Hàm `whenStartsWith` sẽ gọi closure đã cho nếu chuỗi bắt đầu bằng chuỗi con đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('disney world')->whenStartsWith('disney', function (Stringable $string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-ends-with"></a>
#### `whenEndsWith` {.collection-method}

Hàm `whenEndsWith` sẽ gọi closure đã cho nếu chuỗi kết thúc bằng chuỗi con đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('disney world')->whenEndsWith('world', function (Stringable $string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-exactly"></a>
#### `whenExactly` {.collection-method}

Hàm `whenExactly` sẽ gọi closure đã cho nếu chuỗi đúng bằng chuỗi đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel')->whenExactly('laravel', function (Stringable $string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-exactly"></a>
#### `whenNotExactly` {.collection-method}

Hàm `whenNotExactly` sẽ gọi một closure đã cho nếu chuỗi đó không giống với chuỗi đã cho. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('framework')->whenNotExactly('laravel', function (Stringable $string) {
        return $string->title();
    });

    // 'Framework'

<a name="method-fluent-str-when-is"></a>
#### `whenIs` {.collection-method}

Hàm `whenIs` sẽ gọi closure đã cho nếu chuỗi khớp với một mẫu nhất định. Dấu hoa thị có thể được sử dụng làm một ký tự đại diện. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('foo/bar')->whenIs('foo/*', function (Stringable $string) {
        return $string->append('/baz');
    });

    // 'foo/bar/baz'

<a name="method-fluent-str-when-is-ascii"></a>
#### `whenIsAscii` {.collection-method}

Hàm `whenIsAscii` sẽ gọi closure đã cho nếu chuỗi là một dạng ASCII 7 bit. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel')->whenIsAscii(function (Stringable $string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-is-ulid"></a>
#### `whenIsUlid` {.collection-method}

Hàm `whenIsUlid` sẽ gọi một closure nếu chuỗi đã cho là một ULID hợp lệ. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;

    $string = Str::of('01gd6r360bp37zj17nxb55yv40')->whenIsUlid(function (Stringable $string) {
        return $string->substr(0, 8);
    });

    // '01gd6r36'

<a name="method-fluent-str-when-is-uuid"></a>
#### `whenIsUuid` {.collection-method}

Hàm `whenIsUuid` sẽ gọi closure đã cho nếu chuỗi là một UUID hợp lệ. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('a0a2a2d2-0b87-4a18-83f2-2529882be2de')->whenIsUuid(function (Stringable $string) {
        return $string->substr(0, 8);
    });

    // 'a0a2a2d2'

<a name="method-fluent-str-when-test"></a>
#### `whenTest` {.collection-method}

Hàm `whenTest` sẽ gọi closure đã cho nếu chuỗi khớp với một biểu thức chính quy. Closure sẽ nhận vào instance chuỗi:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel framework')->whenTest('/laravel/', function (Stringable $string) {
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
