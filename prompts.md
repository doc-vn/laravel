# Prompts

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Hàm có sẵn](#available-prompts)
    - [Text](#text)
    - [Textarea](#textarea)
    - [Password](#password)
    - [Confirm](#confirm)
    - [Select](#select)
    - [Multi-select](#multiselect)
    - [Suggest](#suggest)
    - [Search](#search)
    - [Multi-search](#multisearch)
    - [Pause](#pause)
- [Chuyển đổi input trước khi validate](#transforming-input-before-validation)
- [Forms](#forms)
- [Thông tin messages](#informational-messages)
- [Tables](#tables)
- [Spin](#spin)
- [Progress Bar](#progress)
- [Clear terminal](#clear)
- [Cài đặt cho Terminal](#terminal-considerations)
- [Môi trường không hỗ trợ và cách dự phòng](#fallbacks)
- [Testing](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Prompts](https://github.com/laravel/prompts) là một package PHP dùng để thêm các form dễ nhìn và thân thiện với người dùng vào ứng dụng command của bạn, với các chức năng giống như trình duyệt bao gồm gợi ý văn bản và validation.

<img src="https://laravel.com/img/docs/prompts-example.png">

Laravel Prompt là package hoàn hảo cho user input trong [Artisan console commands](/docs/{{version}}/artisan#writing-commands) của bạn, nó cũng có thể được dùng cho bất kỳ PHP project command nào.

> [!NOTE]
> Laravel Prompt hỗ trợ macOS, Linux, and Windows cùng với WSL. Để biết thêm thông tin, hãy xem tài liệu của họ về [môi trường không hỗ trợ và cách dự phòngz](#fallbacks).

<a name="installation"></a>
## Cài đặt

Laravel Prompt sẽ mặc định có trong bản Laravel mới nhất.

Laravel Prompt có thể được cài đặt trong các project khác bằng cách dùng Composer package manager:

```shell
composer require laravel/prompts
```

<a name="available-prompts"></a>
## Hàm có sẵn

<a name="text"></a>
### Text

Hàm `text` sẽ hiển thị cho người dùng dưới dạng một câu hỏi, chấp nhận trả lời của họ, và trả về kết quả họ đã nhập:

```php
use function Laravel\Prompts\text;

$name = text('What is your name?');
```

Bạn có thể thêm gợi ý câu trả lời, hoặc một giá trị mặc định, và một thông tin gợi ý:

```php
$name = text(
    label: 'What is your name?',
    placeholder: 'E.g. Taylor Otwell',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="text-required"></a>
#### Required Values

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền tham số `required`:

```php
$name = text(
    label: 'What is your name?',
    required: true
);
```

Nếu bạn muốn tuỳ chỉnh một validation message, bạn cũng có thể truyền vào một string:

```php
$name = text(
    label: 'What is your name?',
    required: 'Your name is required.'
);
```

<a name="text-validation"></a>
#### Additional Validation

Cuối cùng, nếu bạn muốn thực hiện thêm các logic validation, bạn có thể truyền một closure vào tham số `validate`:

```php
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

Closure đó sẽ nhận vào giá trị mà đã được nhập vào và trả về một error message hoặc một giá trị `null` nếu validation được pass.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) trong Laravel. Để làm như vậy, hãy cung cấp một mảng gồm tên thuộc tính và các quy tắc xác thực mà bạn mong muốn cho tham số `validate`:

```php
$name = text(
    label: 'What is your name?',
    validate: ['name' => 'required|max:255|unique:users']
);
```

<a name="textarea"></a>
### Textarea

Hàm `textarea` sẽ hiển thị cho người dùng một câu hỏi, chấp nhận câu trả lời của họ thông qua một textarea gồm nhiều dòng, và sau đó trả về kết quả đó:

```php
use function Laravel\Prompts\textarea;

$story = textarea('Tell me a story.');
```

Bạn có thể gợi ý câu trả lời, hoặc một giá trị mặc định, và một thông tin gợi ý:

```php
$story = textarea(
    label: 'Tell me a story.',
    placeholder: 'This is a story about...',
    hint: 'This will be displayed on your profile.'
);
```

<a name="textarea-required"></a>
#### Required Values

Nếu bạn yêu cầu một giá trị phải được nhập vào, bạn có thể truyền tham số `required`:

```php
$story = textarea(
    label: 'Tell me a story.',
    required: true
);
```

Nếu bạn muốn tuỳ chỉnh một validation message, bạn cũng có thể truyền thêm vào một chuỗi string:

```php
$story = textarea(
    label: 'Tell me a story.',
    required: 'A story is required.'
);
```

<a name="textarea-validation"></a>
#### Additional Validation

Cuối cùng, nếu bạn muốn thực hiện thêm các logic validation, bạn có thể truyền vào một closure cho tham số `validate`:

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: fn (string $value) => match (true) {
        strlen($value) < 250 => 'The story must be at least 250 characters.',
        strlen($value) > 10000 => 'The story must not exceed 10,000 characters.',
        default => null
    }
);
```

Closure đó sẽ nhận vào giá trị mà đã được nhập vào và trả về một error message hoặc một giá trị `null` nếu validation được pass.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) trong Laravel. Để làm như vậy, hãy cung cấp một mảng gồm tên thuộc tính và các quy tắc xác thực mà bạn mong muốn cho tham số `validate`:

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: ['story' => 'required|max:10000']
);
```

<a name="password"></a>
### Password

Hàm `password` cùng giống hàm `text`, nhưng những thứ mà người dùng nhập sẽ được che bởi console. Nó sẽ hữu dụng khi họ nhập những thông tin quan trọng như là password:

```php
use function Laravel\Prompts\password;

$password = password('What is your password?');
```

Bạn cũng thể thêm text gợi ý câu trả lời và thông tin gợi ý:

```php
$password = password(
    label: 'What is your password?',
    placeholder: 'password',
    hint: 'Minimum 8 characters.'
);
```

<a name="password-required"></a>
#### Required Values

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền tham số `required`:

```php
$password = password(
    label: 'What is your password?',
    required: true
);
```

Nếu bạn muốn tuỳ chỉnh một validation message, bạn cũng có thể truyền vào một string:

```php
$password = password(
    label: 'What is your password?',
    required: 'The password is required.'
);
```

<a name="password-validation"></a>
#### Additional Validation

Cuối cùng, nếu bạn muốn thực hiện thêm các logic validation, bạn có thể truyền một closure vào tham số `validate`:

```php
$password = password(
    label: 'What is your password?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 8 => 'The password must be at least 8 characters.',
        default => null
    }
);
```

Closure đó sẽ nhận vào giá trị mà đã được nhập vào và trả về một error message hoặc một giá trị `null` nếu validation được pass.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) trong Laravel. Để làm như vậy, hãy cung cấp một mảng gồm tên thuộc tính và các quy tắc xác thực mà bạn mong muốn cho tham số `validate`:

```php
$password = password(
    label: 'What is your password?',
    validate: ['password' => 'min:8']
);
```

<a name="confirm"></a>
### Confirm

Nếu bạn cần hỏi người dùng một câu hỏi "yes hoặc no", bạn có thể dùng hàm `confirm`. Người dùng có thể sử dụng các phím mỗi tên hoặc ấn phím `y` hoặc `n` để lựa chọn cho câu trả lời của họ. Hàm này sẽ trả về giá trị `true` hoặc `false`.

```php
use function Laravel\Prompts\confirm;

$confirmed = confirm('Do you accept the terms?');
```

Bạn cũng có thể thêm một giá trị mặc định, để tuỳ chỉnh các lựa chọn "Yes" và "No" và thêm thông tin gợi ý:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    default: false,
    yes: 'I accept',
    no: 'I decline',
    hint: 'The terms must be accepted to continue.'
);
```

<a name="confirm-required"></a>
#### Requiring "Yes"

Nếu cần, bạn có thể yêu cầu người dùng chọn "Yes" bằng cách truyền tham số `required`:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: true
);
```

Nếu bạn muốn tuỳ chỉnh một validation message, bạn cũng có thể truyền vào một string:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: 'You must accept the terms to continue.'
);
```

<a name="select"></a>
### Select

Nếu bạn cần người dùng chọn một trong những lựa chọn có sẵn, bạn có thể dùng hàm `select`:

```php
use function Laravel\Prompts\select;

$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner']
);
```

Bạn cũng có thể chỉ định một giá trị mặc định và một thông tin gợi ý:

```php
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner',
    hint: 'The role may be changed at any time.'
);
```

Bạn cũng có thể truyền thêm một mảng con trong tham số `options` để trả về key thay vì giá trị được chọn:

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    default: 'owner'
);
```

Sẽ có năm lựa chọn được hiển thị trước khi danh sách lựa chọn đó bị scroll. Bạn cũng có thể tuỳ chỉnh bằng cách truyền vào tham số `scroll`:

```php
$role = select(
    label: 'Which category would you like to assign?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="select-validation"></a>
#### Additional Validation

Không giống như những hàm khác, hàm `select` sẽ không chấp nhận tham số `required` bởi vì bạn không thể không chọn gì cả. Tuy nhiên, bạn có thể truyền một closure vào tham số `validate` nếu bạn cần hiển thị một tùy chọn nhưng không muốn nó được chọn:

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    validate: fn (string $value) =>
        $value === 'owner' && User::where('role', 'owner')->exists()
            ? 'An owner already exists.'
            : null
);
```

Nếu tham số `options` là một mảng con, thì closure sẽ nhận đầu vào là giá trị khóa của mảng, nếu không, nó sẽ nhận vào giá trị mà người dùng đã chọn. Closure có thể trả về thông báo lỗi hoặc `null` nếu xác thực thành công.

<a name="multiselect"></a>
### Multi-select

Nếu bạn cần người dùng có thể chọn nhiều lựa chọn, bạn có thể sử dụng hàm `multiselect`:

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete']
);
```

Bạn cũng có thể chỉ định các lựa chọn mặc định và thông tin gợi ý:

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete'],
    default: ['Read', 'Create'],
    hint: 'Permissions may be updated at any time.'
);
```

Bạn cũng có thể truyền vào một mảng con cho tham số `options` để trả về các khóa của lựa chọn thay vì giá trị của chúng:

```php
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    default: ['read', 'create']
);
```

Sẽ có năm lựa chọn được hiển thị trước khi danh sách lựa chọn đó bị scroll. Bạn cũng có thể tuỳ chỉnh bằng cách truyền vào tham số `scroll`:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="multiselect-required"></a>
#### Requiring a Value

Mặc định, người dùng có thể không chọn gì hoặc chọn nhiều lựa chọn. Bạn có thể truyền tham số `required` để người dùng phải chọn một hoặc nhiều lựa chọn:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: true
);
```

Nếu bạn muốn tùy chỉnh các validation message, bạn có thể cung cấp một string cho tham số `required`:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: 'You must select at least one category'
);
```

<a name="multiselect-validation"></a>
#### Additional Validation

Bạn có thể truyền một closure vào tham số `validate` nếu bạn cần hiển thị một lựa chọn nhưng không muốn người dùng chọn lựa chọn đó:

```php
$permissions = multiselect(
    label: 'What permissions should the user have?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    validate: fn (array $values) => ! in_array('read', $values)
        ? 'All users require the read permission.'
        : null
);
```

Nếu tham số `options` là một mảng con thì closure sẽ nhận vào các khóa đã chọn, nếu không nó sẽ nhận là các giá trị. Closure có thể trả về thông báo lỗi hoặc `null` nếu xác thực thành công.

<a name="suggest"></a>
### Suggest

Hàm `suggest` có thể được sử dụng để auto-completion câu trả lời của người dùng. Người dùng vẫn có thể đưa ra bất kỳ câu trả lời nào cho dù auto-completion đã được hiển thị:

```php
use function Laravel\Prompts\suggest;

$name = suggest('What is your name?', ['Taylor', 'Dayle']);
```

Ngoài ra, bạn có thể truyền một closure làm tham số thứ hai cho hàm `suggest`. Closure sẽ được gọi mỗi khi người dùng nhập một ký tự vào terminal. Closure sẽ chấp nhận một tham số string chứa dữ liệu người dùng đã nhập và trả về một mảng các tùy chọn để auto-completion:

```php
$name = suggest(
    label: 'What is your name?',
    options: fn ($value) => collect(['Taylor', 'Dayle'])
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
)
```

Bạn có thể thêm gợi ý câu trả lời, hoặc một giá trị mặc định, và một thông tin gợi ý:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="suggest-required"></a>
#### Required Values

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền tham số `required`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: true
);
```

Nếu bạn muốn tuỳ chỉnh một validation message, bạn cũng có thể truyền vào một string:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: 'Your name is required.'
);
```

<a name="suggest-validation"></a>
#### Additional Validation

Cuối cùng, nếu bạn muốn thực hiện thêm các logic validation, bạn có thể truyền một closure vào tham số `validate`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

Closure đó sẽ nhận vào giá trị mà đã được nhập vào và trả về một error message hoặc một giá trị `null` nếu validation được pass.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) trong Laravel. Để làm như vậy, hãy cung cấp một mảng gồm tên thuộc tính và các quy tắc xác thực mà bạn mong muốn cho tham số `validate`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: ['name' => 'required|min:3|max:255']
);
```

<a name="search"></a>
### Search

Nếu bạn có nhiều tùy chọn để người dùng lựa chọn, thì hàm `search` sẽ cho phép người dùng nhập một truy vấn tìm kiếm để lọc ra kết quả trước khi sử dụng các phím mũi tên để chọn ra một lựa chọn tốt nhất:

```php
use function Laravel\Prompts\search;

$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

Hàm closure sẽ nhận một text đã được người dùng nhập vào và trả về một mảng các tùy chọn. Nếu bạn trả về một mảng gồm khoá và giá trị, thì khóa của tùy chọn sẽ được trả về, nếu không, giá trị của tuỳ chọn sẽ được trả về.

Khi filter một mảng mà bạn muốn trả về giá trị, thì bạn nên sử dụng hàm `array_values` hoặc phương thức `values` của Collection để đảm bảo mảng đó sẽ không bị trở thành mảng gồm key-value:

```php
$names = collect(['Taylor', 'Abigail']);

$selected = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

Bạn cũng thể thêm text gợi ý câu trả lời và thông tin gợi ý:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

Sẽ có năm lựa chọn được hiển thị trước khi danh sách lựa chọn đó bị scroll. Bạn cũng có thể tuỳ chỉnh bằng cách truyền vào tham số `scroll`:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="search-validation"></a>
#### Additional Validation

Nếu bạn muốn thực hiện thêm các logic validation, bạn có thể truyền một closure vào tham số `validate`:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (int|string $value) {
        $user = User::findOrFail($value);

        if ($user->opted_out) {
            return 'This user has opted-out of receiving mail.';
        }
    }
);
```

Nếu closure `options` trả về một mảng gồm khoá và giá trị, thì closure của `validate` sẽ nhận vào khóa đã chọn, nếu ngược lại, thì closure sẽ nhận vào giá trị đã chọn. Closure có thể trả về thông báo lỗi hoặc `null` nếu xác thực thành công.

<a name="multisearch"></a>
### Multi-search

Nếu bạn có nhiều lựa chọn tìm kiếm và cần người dùng chọn nhiều mục, hàm `multisearch` sẽ cho phép người dùng nhập truy vấn tìm kiếm để lọc ra kết quả trước khi sử dụng phím mũi tên và phím cách để chọn lựa:

```php
use function Laravel\Prompts\multisearch;

$ids = multisearch(
    'Search for the users that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

Hàm closure sẽ nhận vào text đã được người dùng nhập và trả về một mảng các tùy chọn. Nếu bạn trả về một mảng gồm khoá và giá trị, thì các khóa của tùy chọn được chọn sẽ được trả về; nếu không, thì giá trị của chúng sẽ được trả về.

Khi filter một mảng mà bạn muốn trả về giá trị, thì bạn nên sử dụng hàm `array_values` hoặc phương thức `values` của Collection để đảm bảo mảng đó sẽ không bị trở thành mảng gồm key-value:

```php
$names = collect(['Taylor', 'Abigail']);

$selected = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

Bạn cũng thể thêm text gợi ý câu trả lời và thông tin gợi ý:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

Sẽ có năm lựa chọn được hiển thị trước khi danh sách lựa chọn đó bị scroll. Bạn có thể tùy chỉnh tùy chọn này bằng cách cung cấp tham số `scroll`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="multisearch-required"></a>
#### Requiring a Value

Mặc định, người dùng có thể không chọn gì hoặc chọn nhiều lựa chọn. Bạn có thể truyền tham số `required` để người dùng phải chọn một hoặc nhiều lựa chọn:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: true
);
```

Nếu bạn muốn tuỳ chỉnh một validation message, bạn cũng có thể cung cấp một string cho tham số `required`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: 'You must select at least one user.'
);
```

<a name="multisearch-validation"></a>
#### Additional Validation

Nếu bạn muốn thực hiện thêm các logic validation, bạn có thể truyền một closure vào tham số `validate`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (array $values) {
        $optedOut = User::whereLike('name', '%a%')->findMany($values);

        if ($optedOut->isNotEmpty()) {
            return $optedOut->pluck('name')->join(', ', ', and ').' have opted out.';
        }
    }
);
```

Nếu closure `options` trả về một mảng gồm khoá và giá trị, thì closure sẽ nhận vào các khóa đã được chọn; còn lại, closure sẽ nhận vào các giá trị đã được chọn. Closure có thể trả về thông báo lỗi hoặc `null` nếu xác thực thành công.

<a name="pause"></a>
### Pause

Hàm `pause` có thể được sử dụng để hiển thị các text thông tin cho người dùng và chờ họ xác nhận tiếp tục bằng cách nhấn phím Enter hoặc Return:

```php
use function Laravel\Prompts\pause;

pause('Press ENTER to continue.');
```

<a name="transforming-input-before-validation"></a>
## Chuyển đổi input trước khi validate

Thỉnh thoảng bạn có thể muốn chuyển đổi input của prompt trước khi quá trình validate được diễn ra. Ví dụ, bạn có thể muốn loại bỏ khoảng trắng ra khỏi bất kỳ chuỗi nào được cung cấp. Để thực hiện điều này, nhiều hàm prompt cung cấp một tham số `transform`, chấp nhận một closure:

```php
$name = text(
    label: 'What is your name?',
    transform: fn (string $value) => trim($value),
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

<a name="forms"></a>
## Forms

Thông thường, bạn sẽ có nhiều prompt được hiển thị theo một trình tự nhất định để thu thập thông tin trước khi thực hiện các hành động. Bạn có thể sử dụng hàm `form` để tạo một group các prompt để người dùng nhập vào:

```php
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true)
    ->password('What is your password?', validate: ['password' => 'min:8'])
    ->confirm('Do you accept the terms?')
    ->submit();
```

Phương thức `submit` sẽ trả về một mảng được đánh số chứa tất cả các phản hồi từ các prompt của form. Tuy nhiên, bạn có thể cung cấp một tên cho mỗi prompt thông qua tham số `name`. Khi một tên được cung cấp, thì phản hồi của prompt sẽ chứa cái tên đó và có thể truy cập được thông qua tên đó:

```php
use App\Models\User;
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->password(
        label: 'What is your password?',
        validate: ['password' => 'min:8'],
        name: 'password'
    )
    ->confirm('Do you accept the terms?')
    ->submit();

User::create([
    'name' => $responses['name'],
    'password' => $responses['password'],
]);
```

Lợi ích chính của việc sử dụng hàm `form` là người dùng có thể quay lại các prompt trước đó có trong form bằng cách sử dụng `CTRL + U`. Điều này cho phép người dùng sửa hoặc thay đổi các lựa chọn mà không cần phải hủy và trả lời lại toàn bộ form.

Nếu bạn cần kiểm soát chi tiết hơn đối với một prompt có trong form, bạn có thể gọi phương thức `add` thay vì gọi trực tiếp một trong các hàm prompt. Phương thức `add` sẽ chứa tất cả các phản hồi trước đó mà người dùng đã cung cấp:

```php
use function Laravel\Prompts\form;
use function Laravel\Prompts\outro;
use function Laravel\Prompts\text;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->add(function ($responses) {
        return text("How old are you, {$responses['name']}?");
    }, name: 'age')
    ->submit();

outro("Your name is {$responses['name']} and you are {$responses['age']} years old.");
```

<a name="informational-messages"></a>
## Thông tin messages

Các hàm `note`, `info`, `warning`, `error` và `alert` có thể được sử dụng để hiển thị các thông báo theo các kiểu khác nhau:

```php
use function Laravel\Prompts\info;

info('Package installed successfully.');
```

<a name="tables"></a>
## Tables

Hàm `table` sẽ giúp bạn dễ dàng hiển thị các hàng, cột dữ liệu. Tất cả những gì bạn cần làm là cung cấp tên cột và dữ liệu cho bảng:

```php
use function Laravel\Prompts\table;

table(
    headers: ['Name', 'Email'],
    rows: User::all(['name', 'email'])->toArray()
);
```

<a name="spin"></a>
## Spin

Hàm `spin` sẽ hiển thị một spinner cùng với một thông báo tùy chọn khi đang thực hiện một lệnh callback được chỉ định. Hàm này có tác dụng chỉ ra các process đang được diễn ra và trả về kết quả của lệnh callback sau khi hoàn tất:

```php
use function Laravel\Prompts\spin;

$response = spin(
    callback: fn () => Http::get('http://example.com'),
    message: 'Fetching response...'
);
```

> [!WARNING]
> Hàm `spin` sẽ cần một PHP extension [PCNTL](https://www.php.net/manual/en/book.pcntl.php) để tạo hiệu ứng động cho spinner. Khi extension này chưa được cài đặt, thì phiên bản tĩnh của spinner sẽ được xuất hiện thay thế.

<a name="progress"></a>
## Progress Bars

Đối với các tác vụ chạy dài, việc hiển thị thanh tiến trình để thông báo cho người dùng mức độ hoàn thành của tác vụ có thể hữu ích. Sử dụng hàm `progress`, Laravel sẽ hiển thị thanh tiến trình và tăng tốc độ của nó cho mỗi lần lặp trên một giá trị lặp nhất định:

```php
use function Laravel\Prompts\progress;

$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: fn ($user) => $this->performTask($user)
);
```

Hàm `progress` hoạt động giống như một hàm map và sẽ trả về một mảng chứa giá trị được trả về của mỗi lần lặp lệnh callback của bạn.

Hàm callback cũng có thể chấp nhận instance `Laravel\Prompts\Progress`, cho phép bạn sửa label và gợi ý ở mỗi lần lặp lại:

```php
$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: function ($user, $progress) {
        $progress
            ->label("Updating {$user->name}")
            ->hint("Created on {$user->created_at}");

        return $this->performTask($user);
    },
    hint: 'This may take some time.'
);
```

Thỉnh thoảng, bạn có thể cần kiểm soát hơn về cách thanh tiến trình được chạy. Đầu tiên, hãy định nghĩa tổng số step mà process sẽ được lặp lại. Sau đó, đẩy thanh tiến trình lên bằng phương thức `advance` sau khi xử lý xong mỗi mục:

```php
$progress = progress(label: 'Updating users', steps: 10);

$users = User::all();

$progress->start();

foreach ($users as $user) {
    $this->performTask($user);

    $progress->advance();
}

$progress->finish();
```

<a name="clear"></a>
## Clear terminal

Hàm `clear` có thể được sử dụng để làm sạch terminal của người dùng:

```php
use function Laravel\Prompts\clear;

clear();
```

<a name="terminal-considerations"></a>
## Cài đặt cho Terminal

<a name="terminal-width"></a>
#### Terminal Width

Nếu độ dài của bất kỳ label, tùy chọn hoặc validation message nào vượt quá số "cột" trong terminal của người dùng, nó sẽ tự động bị cắt bớt. Vì vậy, hãy cân nhắc việc giảm độ dài của các chuỗi này vì người dùng của bạn có thể đang sử dụng terminal hẹp hơn. Độ dài tối đa an toàn thường là 74 ký tự để hỗ trợ terminal 80 ký tự.

<a name="terminal-height"></a>
#### Terminal Height

Đối với bất kỳ prompt nào chấp nhận tham số `scroll`, giá trị được cấu hình sẽ tự động được giảm xuống cho phù hợp với chiều cao của terminal của người dùng, bao gồm cả khoảng trống cho validation message.

<a name="fallbacks"></a>
## Môi trường không hỗ trợ và cách dự phòng

Laravel Prompts hỗ trợ macOS, Linux và Windows cùng với WSL. Do những hạn chế trong phiên bản PHP dành cho Windows, hiện tại không thể sử dụng Laravel Prompts trên Windows mà không có WSL.

Vì lý do này, Laravel Prompts hỗ trợ việc triển khai thay thế như [Symfony Console Question Helper](https://symfony.com/doc/current/components/console/helpers/questionhelper.html).

> [!NOTE]
> Khi sử dụng Laravel Prompts với framework Laravel, các phương án dự phòng cho từng prompt đã được cấu hình sẵn cho bạn và sẽ tự động được kích hoạt trong các môi trường không được hỗ trợ.

<a name="fallback-conditions"></a>
#### Fallback Conditions

Nếu bạn không sử dụng Laravel hoặc cần tùy chỉnh thời điểm sử dụng hành vi dự phòng, bạn có thể truyền một biến boolean vào phương thức tĩnh `fallbackWhen` trên class `Prompt`:

```php
use Laravel\Prompts\Prompt;

Prompt::fallbackWhen(
    ! $input->isInteractive() || windows_os() || app()->runningUnitTests()
);
```

<a name="fallback-behavior"></a>
#### Fallback Behavior

Nếu bạn không sử dụng Laravel hoặc cần tùy chỉnh hành vi dự phòng, bạn có thể truyền một closure vào phương thức tĩnh `fallbackUsing` trên mỗi class prompt:

```php
use Laravel\Prompts\TextPrompt;
use Symfony\Component\Console\Question\Question;
use Symfony\Component\Console\Style\SymfonyStyle;

TextPrompt::fallbackUsing(function (TextPrompt $prompt) use ($input, $output) {
    $question = (new Question($prompt->label, $prompt->default ?: null))
        ->setValidator(function ($answer) use ($prompt) {
            if ($prompt->required && $answer === null) {
                throw new \RuntimeException(
                    is_string($prompt->required) ? $prompt->required : 'Required.'
                );
            }

            if ($prompt->validate) {
                $error = ($prompt->validate)($answer ?? '');

                if ($error) {
                    throw new \RuntimeException($error);
                }
            }

            return $answer;
        });

    return (new SymfonyStyle($input, $output))
        ->askQuestion($question);
});
```

Các phương thức dự phòng phải được cấu hình riêng cho từng class prompt. Lệnh closure sẽ nhận vào một instance của class prompt và phải trả về kiểu dữ liệu phù hợp cho prompt đó.

<a name="testing"></a>
## Testing

Laravel cung cấp nhiều phương thức khác nhau để kiểm tra xem command của bạn có hiển thị các thông báo Prompt như bạn mong muốn hay không:

```php tab=Pest
test('report generation', function () {
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
public function test_report_generation(): void
{
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
}
```
