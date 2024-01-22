# Localization

- [Giới thiệu](#introduction)
    - [Cấu hình ngôn ngữ](#configuring-the-locale)
- [Định nghĩa chuỗi translation](#defining-translation-strings)
    - [Sử dụng short key](#using-short-keys)
    - [Sử dụng chuỗi translation như key](#using-translation-strings-as-keys)
- [Lấy chuỗi translation](#retrieving-translation-strings)
    - [Thay thế parameter trong chuỗi translation](#replacing-parameters-in-translation-strings)
    - [Số nhiều](#pluralization)
- [Ghi đè package file language](#overriding-package-language-files)

<a name="introduction"></a>
## Giới thiệu

Các tính năng localization của Laravel cung cấp một cách thuận tiện để lấy ra các chuỗi bằng nhiều ngôn ngữ khác nhau, cho phép bạn dễ dàng hỗ trợ nhiều ngôn ngữ trong application của bạn.

Laravel cung cấp hai cách để quản lý chuỗi được dịch. Đầu tiên, các chuỗi ngôn ngữ có thể được lưu trữ trong các file ở thư mục `resources/lang`. Trong thư mục này, có thể có các thư mục con cho mỗi ngôn ngữ được application của bạn hỗ trợ. Đây là cách tiếp cận mà Laravel sử dụng để quản lý các chuỗi dịch cho các tính năng được tích hợp sẵn của Laravel, chẳng hạn như thông báo lỗi validation:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

Hoặc, các chuỗi dịch có thể được định nghĩa trong các file JSON được lưu trong thư mục `resources/lang`. Khi thực hiện cách này, mỗi ngôn ngữ được ứng dụng của bạn hỗ trợ sẽ có một file JSON tương ứng trong thư mục này. Cách tiếp cận này được khuyến cáo cho các ứng dụng có số lượng lớn chuỗi cần phải dịch:

    /resources
        /lang
            en.json
            es.json

Chúng ta sẽ thảo luận về từng cách quản lý chuỗi dịch này trong tài liệu dưới.

<a name="configuring-the-locale"></a>
### Cấu hình ngôn ngữ

Ngôn ngữ mặc định cho application của bạn được lưu trữ trong tuỳ chọn cấu hình `locale` của file cấu hình `config/app.php`. Bạn hãy thoải mái sửa giá trị này cho phù hợp với nhu cầu application của bạn.

Bạn có thể sửa ngôn ngữ mặc định cho một HTTP request khi đang chạy bằng cách sử dụng phương thức `setLocale` được cung cấp bởi facade `App`:

    use Illuminate\Support\Facades\App;

    Route::get('/greeting/{locale}', function ($locale) {
        if (! in_array($locale, ['en', 'es', 'fr'])) {
            abort(400);
        }

        App::setLocale($locale);

        //
    });

Bạn có thể cấu hình "fallback language", ngôn ngữ này sẽ được sử dụng khi ngôn ngữ đang hoạt động không chứa chuỗi đang cần dịch. Giống như ngôn ngữ mặc định, fallback language cũng được cấu hình trong file cấu hình `config/app.php`:

    'fallback_locale' => 'en',

<a name="determining-the-current-locale"></a>
#### Xác định ngôn ngữ hiện tại

Bạn có thể sử dụng các phương thức `currentLocale` và `isLocale` trên facade `App` để xác định ngôn ngữ hiện tại hoặc kiểm tra xem ngôn ngữ có phải là một giá trị nào đó hay không:

    use Illuminate\Support\Facades\App;

    $locale = App::currentLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="defining-translation-strings"></a>
## Định nghĩa một chuỗi translation

<a name="using-short-keys"></a>
### Sử dụng short key

Thông thường, các chuỗi dịch được lưu trữ trong các file trong thư mục `resources/lang`. Trong thư mục này, cần có thư mục con cho mỗi ngôn ngữ được application của bạn hỗ trợ. Đây là cách tiếp cận mà Laravel sử dụng để quản lý các chuỗi dịch cho các tính năng được tích hợp sẵn của Laravel, chẳng hạn như thông báo lỗi validation:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

Tất cả các file ngôn ngữ đều trả về một mảng của các chuỗi đã được đặt key. Ví dụ:

    <?php

    // resources/lang/en/messages.php

    return [
        'welcome' => 'Welcome to our application!',
    ];

> {note} Đối với các ngôn ngữ khác nhau theo lãnh thổ, bạn nên set tên thư mục của ngôn ngữ theo tiêu chuẩn ISO 15897. Ví dụ: "en_GB" nên được sử dụng cho tiếng Anh của nước Anh thay vì "en-gb".

<a name="using-translation-strings-as-keys"></a>
### Sử dụng chuỗi translation như key

Đối với các application có một số lượng lớn các chuỗi cần phải dịch, việc định nghĩa mọi chuỗi bằng "short key" có thể nhanh chóng gây nhầm lẫn khi tham chiếu các key đó vào trong các file view của bạn và thật khó khăn khi liên tục phải tạo ra các khóa cho mọi chuỗi được ứng dụng của bạn hỗ trợ.

Vì lý do này, Laravel cũng cung cấp hỗ trợ cho việc định nghĩa chuỗi dịch bằng cách sử dụng bản dịch "mặc định" của chuỗi làm khóa. Các file translation sử dụng chuỗi translation làm khóa được lưu dưới dạng file JSON trong thư mục `resources/lang`. Ví dụ: nếu ứng dụng của bạn có bản translation tiếng Tây Ban Nha, bạn nên tạo file `resources/lang/es.json`:

```js
{
    "I love programming.": "Me encanta programar."
}
```

#### Key / File Conflicts

Bạn không nên định nghĩa các khóa chuỗi dịch xung đột với các tên file dịch khác. Ví dụ: dịch `__('Action')` cho ngôn ngữ "NL" trong khi file `nl/action.php` tồn tại nhưng file `nl.json` không tồn tại sẽ dẫn đến việc translator sẽ hiểu nhầm và trả về nội dung của `nl/action.php`.

<a name="retrieving-translation-strings"></a>
## Lấy chuỗi translation

Bạn có thể lấy chuỗi dịch từ các file ngôn ngữ của bạn bằng hàm helper `__`. Nếu bạn đang sử dụng "short keys" để định nghĩa các chuỗi dịch của bạn, bạn nên truyền file chứa khóa và chính khóa của nó cho hàm `__` bằng cú pháp "chấm". Ví dụ: có thể lấy chuỗi đã được dịch `welcome` từ file ngôn ngữ `resources/lang/en/messages.php`:

    echo __('messages.welcome');

Nếu chuỗi dịch được chỉ định không tồn tại, hàm `__` sẽ trả về khóa của chuỗi dịch. Vì vậy, theo ví dụ trên, hàm `__` sẽ trả về `messages.welcome` nếu chuỗi dịch đó không tồn tại.

Nếu bạn đang sử dụng [chuỗi dịch mặc định làm khóa dịch](#using-translation-strings-as-keys), bạn nên truyền bản dịch mặc định của chuỗi đó cho hàm `__`;

    echo __('I love programming.');

Một lần nữa, nếu chuỗi dịch đó không tồn tại, hàm `__` sẽ trả về khóa của chuỗi dịch và nó đã được cung cấp.

Nếu đang sử dụng [Blade templating engine](/docs/{{version}}/blade), bạn có thể sử dụng cú pháp echo `{{ }}` để hiển thị chuỗi dịch:

    {{ __('messages.welcome') }}

<a name="replacing-parameters-in-translation-strings"></a>
### Thay thế parameter trong chuỗi translation

Nếu bạn muốn, bạn có thể định nghĩa một thuộc tính thay thế trong các chuỗi translation của bạn. Tất cả những thuộc tính thay thế đều có tiền tố là `:`. Ví dụ: bạn có thể định nghĩa thông báo chào mừng với một thuộc tính thay thế name:

    'welcome' => 'Welcome, :name',

Để thay đổi các thuộc tính thay thế khi lấy chuỗi translation, bạn có thể truyền một mảng các thay thế làm tham số thứ hai cho hàm `__`:

    echo __('messages.welcome', ['name' => 'dayle']);

Nếu biến thay của bạn đều là chữ in hoa hoặc chỉ viết hoa chữ cái đầu tiên, giá trị translation cũng sẽ được viết hoa tương ứng:

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle

<a name="pluralization"></a>
### Số nhiều

Số nhiều là một vấn đề phức tạp, vì các ngôn ngữ khác nhau lại có nhiều quy tắc phức tạp cho số nhiều; Tuy nhiên, Laravel có thể giúp bạn dịch các chuỗi khác nhau dựa trên các quy tắc số nhiều mà bạn đã định nghĩa. Sử dụng một ký tự "|", bạn có thể phân biệt các dạng số ít và số nhiều của chuỗi:

    'apples' => 'There is one apple|There are many apples',

Tất nhiên, số nhiều cũng được hỗ trợ khi sử dụng [chuỗi dịch làm khóa](#using-translation-strings-as-keys):

```js
{
    "There is one apple|There are many apples": "Hay una manzana|Hay muchas manzanas"
}
```

Bạn thậm chí có thể tạo ra các quy tắc số nhiều phức tạp hơn, bằng cách chỉ định các chuỗi translation cho nhiều phạm vi giá trị khác nhau:

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

Sau khi định nghĩa chuỗi translation có nhiều tùy chọn về số nhiều, bạn có thể sử dụng hàm `trans_choice` để lấy ra chuỗi cho một "count" đã cho. Trong ví dụ này, vì count lớn hơn một, dạng số nhiều của chuỗi translation sẽ được trả về:

    echo trans_choice('messages.apples', 10);

Bạn cũng có thể định nghĩa các thuộc tính thay thế trong các chuỗi số nhiều. Những thuộc tính thay thế này có thể được thay thế bằng cách truyền một mảng làm tham số thứ ba cho hàm `trans_choice`:

    'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

    echo trans_choice('time.minutes_ago', 5, ['value' => 5]);

Nếu bạn muốn hiển thị giá trị integer đã được truyền vào hàm `trans_choice`, bạn có thể sử dụng thuộc tính thay thế `:count` có sẵn:

    'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',

<a name="overriding-package-language-files"></a>
## Ghi đè package file language

Một số package có thể đi cùng với các file ngôn ngữ riêng của họ. Thay vì sửa vào các file core của package để thay đổi các chuổi translation, bạn có thể ghi đè chúng bằng cách lưu các file trong thư mục `resources/lang/vendor/{package}/{locale}`.

Vậy, ví dụ, nếu bạn cần ghi đè các chuỗi translation tiếng Anh trong file `messages.php` của package có tên là `skyrim/hearthfire`, thì bạn cần lưu một file ngôn ngữ có path như sau: `resources/lang/vendor/hearthfire/en/messages.php`. Trong file này, bạn chỉ cần định nghĩa chuỗi translation mà bạn muốn ghi đè. Bất kỳ chuỗi translation nào mà bạn không muốn ghi đè sẽ vẫn được tải từ các file ngôn ngữ gốc của package.
