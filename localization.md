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

Các tính năng localization của Laravel cung cấp một cách thuận tiện để lấy ra các chuỗi bằng nhiều ngôn ngữ khác nhau, cho phép bạn dễ dàng hỗ trợ nhiều ngôn ngữ trong application của bạn. Các chuỗi ngôn ngữ được lưu trữ trong các file ở thư mục `resources/lang`. Trong thư mục này cần có thư mục con cho mỗi ngôn ngữ được application của bạn hỗ trợ:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

Tất cả các file ngôn ngữ đều trả về một mảng của các chuỗi đã được đặt key. Ví dụ:

    <?php

    return [
        'welcome' => 'Welcome to our application',
    ];

> {note} Đối với các ngôn ngữ khác nhau theo lãnh thổ, bạn nên set tên cho các thư mục ngôn ngữ đó theo chuẩn ISO 15897. Ví dụ: "en_GB" nên được sử dụng cho tiếng Anh-Anh thay vì "en-gb".

<a name="configuring-the-locale"></a>
### Cấu hình ngôn ngữ

Ngôn ngữ mặc định cho application của bạn được lưu trữ trong file cấu hình `config/app.php`. Bạn có thể sửa đổi giá trị này cho phù hợp với nhu cầu application của bạn. Bạn cũng có thể thay đổi ngôn ngữ hoạt động trong lúc chạy bằng cách sử dụng phương thức `setLocale` trên facade `App`:

    Route::get('welcome/{locale}', function ($locale) {
        if (! in_array($locale, ['en', 'es', 'fr'])) {
            abort(400);
        }

        App::setLocale($locale);

        //
    });

Bạn có thể cấu hình "fallback language", ngôn ngữ này sẽ được sử dụng khi ngôn ngữ đang hoạt động không chứa chuỗi đang cần dịch. Giống như ngôn ngữ mặc định, fallback language cũng được cấu hình trong file cấu hình `config/app.php`:

    'fallback_locale' => 'en',

#### Xác định ngôn ngữ hiện tại

Bạn có thể sử dụng các phương thức `getLocale` và `isLocale` trên facade `App` để xác định ngôn ngữ hiện tại hoặc kiểm tra xem ngôn ngữ có phải là một giá trị nào đó hay không:

    $locale = App::getLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="defining-translation-strings"></a>
## Định nghĩa một chuỗi translation

<a name="using-short-keys"></a>
### Sử dụng short key

Thông thường, các chuỗi translation được lưu trữ trong các file trong thư mục `resources/lang`. Trong thư mục này cần có thư mục con cho mỗi ngôn ngữ được application hỗ trợ:

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
        'welcome' => 'Welcome to our application',
    ];

<a name="using-translation-strings-as-keys"></a>
### Sử dụng chuỗi translation như key

Đối với các application có yêu cầu dịch thuật nặng, việc xác định mọi chuỗi bằng "short key" có thể nhanh chóng gây nhầm lẫn khi tham chiếu chúng trong các view của bạn. Vì lý do này, Laravel cũng cung cấp một hỗ trợ để xác định chuỗi translation bằng cách sử dụng bản translation "default" của chuỗi làm khóa.

Các file translation sử dụng chuỗi translation làm khóa được lưu trữ dưới dạng file JSON trong thư mục `resources/lang`. Ví dụ: nếu ứng dụng của bạn có bản translation tiếng Tây Ban Nha, bạn nên tạo file `resources/lang/es.json`:

    {
        "I love programming.": "Me encanta programar."
    }

<a name="retrieving-translation-strings"></a>
## Lấy chuỗi translation

Bạn có thể lấy các chuỗi đã được dịch từ các file ngôn ngữ bằng cách sử dụng hàm helper `__`. Phương thức `__` chấp nhận tên file và khóa của chuỗi đã dịch làm tham số đầu tiên của nó. Ví dụ: hãy lấy chuỗi đã được  đượcdịch `welcome` từ file ngôn ngữ `resources/lang/messages.php`:

    echo __('messages.welcome');

    echo __('I love programming.');

Nếu bạn đang sử dụng [Blade templating engine](/docs/{{version}}/blade), bạn có thể sử dụng cú pháp `{{ }}` để echo một chuỗi đã được dịch hoặc sử dụng lệnh `@lang`:

    {{ __('messages.welcome') }}

    @lang('messages.welcome')

Nếu chuỗi cần dịch được chỉ định không tồn tại, hàm `__` sẽ trả về khóa của chuỗi cần dịch. Vì vậy, nếu sử dụng ví dụ trên, thì hàm `__` sẽ trả về `messages.welcome` nếu chuỗi cần dịch không tồn tại.

> {note} Lệnh `@lang` không loại bỏ các ký tự đặc biệt ra khỏi output. Bạn cần phải **chịu trách nhiệm** về việc loại bỏ các ký tự đặc biệt ra khỏi output của bạn khi sử dụng lệnh này.

<a name="replacing-parameters-in-translation-strings"></a>
### Thay thế parameter trong chuỗi translation

Nếu bạn muốn, bạn có thể định nghĩa một thuộc tính thay thế trong các chuỗi translation của bạn. Tất cả những thuộc tính thay thế đều có tiền tố là `:`. Ví dụ: bạn có thể định nghĩa thông báo chào mừng với một thuộc tính thay thế name:

    'welcome' => 'Welcome, :name',

Để thay đổi các thuộc tính thay thế khi lấy chuỗi translation, hãy truyền một mảng các thay thế làm tham số thứ hai cho hàm `__`:

    echo __('messages.welcome', ['name' => 'dayle']);

Nếu biến thay của bạn đều là chữ in hoa hoặc chỉ viết hoa chữ cái đầu tiên, giá trị translation cũng sẽ được viết hoa tương ứng:

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle

<a name="pluralization"></a>
### Số nhiều

Số nhiều là một vấn đề phức tạp, vì các ngôn ngữ khác nhau có nhiều quy tắc phức tạp cho số nhiều. Bằng cách sử dụng ký tự "|", bạn có thể phân biệt các dạng số ít và số nhiều của chuỗi:

    'apples' => 'There is one apple|There are many apples',

Bạn thậm chí có thể tạo các quy tắc số nhiều phức tạp hơn, bằng cách chỉ định các chuỗi translation cho nhiều đoạn trong dãy số:

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

Sau khi định nghĩa chuỗi translation có nhiều tùy chọn về số nhiều, bạn có thể sử dụng hàm `trans_choice` để lấy ra chuỗi cho một "count" đã cho. Trong ví dụ này, vì count lớn hơn một, dạng số nhiều của chuỗi translation sẽ được trả về:

    echo trans_choice('messages.apples', 10);

Bạn cũng có thể định nghĩa các thuộc tính thay thế trong các chuỗi số nhiều. Những thuộc tính thay thế này có thể được thay thế bằng cách truyền một mảng làm tham số thứ ba cho hàm `trans_choice`:

    'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

    echo trans_choice('time.minutes_ago', 5, ['value' => 5]);

Nếu bạn muốn hiển thị giá trị integer đã được truyền vào hàm `trans_choice`, bạn có thể sử dụng thuộc tính thay thế `:count`:

    'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',

<a name="overriding-package-language-files"></a>
## Ghi đè package file language

Một số package có thể đi cùng với các file ngôn ngữ riêng của họ. Thay vì sửa vào các file core của package để thay đổi các chuổi translation, bạn có thể ghi đè chúng bằng cách lưu các file trong thư mục `resources/lang/vendor/{package}/{locale}`.

Vậy, ví dụ, nếu bạn cần ghi đè các chuỗi translation tiếng Anh trong file `messages.php` của package có tên là `skyrim/hearthfire`, thì bạn cần lưu một file ngôn ngữ có path như sau: `resources/lang/vendor/hearthfire/en/messages.php`. Trong file này, bạn chỉ cần định nghĩa chuỗi translation mà bạn muốn ghi đè. Bất kỳ chuỗi translation nào mà bạn không muốn ghi đè sẽ vẫn được tải từ các file ngôn ngữ gốc của package.
