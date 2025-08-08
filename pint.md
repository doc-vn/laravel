# Laravel Pint

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Chạy Pint](#running-pint)
- [Cấu hình Pint](#configuring-pint)
    - [Presets](#presets)
    - [Rules](#rules)
    - [Loại trừ file hoặc folder](#excluding-files-or-folders)

<a name="introduction"></a>
## Giới thiệu

[Laravel Pint](https://github.com/laravel/pint) là một trình sửa lỗi code cho PHP dành cho những người theo chủ nghĩa tối giản. Pint được xây dựng trên PHP-CS-Fixer và giúp đảm bảo code của bạn luôn sạch sẽ và nhất quán.

Pint được tự động cài đặt với tất cả các ứng dụng Laravel mới để bạn có thể bắt đầu sử dụng ngay lập tức. Mặc định, Pint không yêu cầu bất kỳ cấu hình nào và sẽ khắc phục các lỗi về code style của bạn bằng cách tuân theo coding style ​​của Laravel.

<a name="installation"></a>
## Cài đặt

Pint được có trong các bản phát hành gần đây của Laravel framework, do đó việc cài đặt thường không cần thiết. Tuy nhiên, đối với các ứng dụng cũ hơn, bạn có thể cài đặt Laravel Pint thông qua Composer:

```shell
composer require laravel/pint --dev
```

<a name="running-pint"></a>
## Chạy Pint

Bạn có thể hướng dẫn Pint sửa các lỗi về code style bằng cách gọi file binary `pint` có trong thư mục `vendor/bin` của project của bạn:

```shell
./vendor/bin/pint
```

Bạn cũng có thể chạy Pint cho các file hoặc một thư mục cụ thể:

```shell
./vendor/bin/pint app/Models

./vendor/bin/pint app/Models/User.php
```

Pint sẽ hiển thị một danh sách đầy đủ tất cả các file mà nó cập nhật. Bạn có thể xem chi tiết hơn về các thay đổi mà Pint thực hiện bằng cách cung cấp tùy chọn `-v` khi gọi Pint:

```shell
./vendor/bin/pint -v
```

Nếu bạn muốn Pint chỉ kiểm tra các lỗi code style của bạn mà không muốn thay đổi các file, bạn có thể sử dụng tùy chọn `--test`:

```shell
./vendor/bin/pint --test
```

Nếu bạn muốn Pint chỉ sửa các file có thay đổi chưa được commit vào Git, bạn có thể sử dụng tùy chọn `--dirty`:

```shell
./vendor/bin/pint --dirty
```

<a name="configuring-pint"></a>
## Cấu hình Pint

Như đã đề cập trước đó, Pint không yêu cầu bất kỳ cấu hình nào. Tuy nhiên, nếu bạn muốn tùy chỉnh các cài đặt có sẵn, quy tắc hoặc thư mục đã kiểm tra, bạn có thể thực hiện bằng cách tạo ra file `pint.json` trong thư mục root của dự án:

```json
{
    "preset": "laravel"
}
```

Ngoài ra, nếu bạn muốn sử dụng `pint.json` từ một thư mục cụ thể, bạn có thể cung cấp tùy chọn `--config` khi gọi Pint:

```shell
pint --config vendor/my-company/coding-style/pint.json
```

<a name="presets"></a>
### Presets

Presets định nghĩa một tập hợp các quy tắc có thể được sử dụng để sửa các vấn đề về coding style trong code của bạn. Mặc định, Pint sử dụng cài đặt có sẵn `laravel`, cài đặt này sửa các vấn đề bằng cách tuân theo coding style ​​của Laravel. Tuy nhiên, bạn cũng có thể chỉ định một cài đặt có sẵn khác bằng cách cung cấp tùy chọn `--preset` cho Pint:

```shell
pint --preset psr12
```

Nếu bạn muốn, bạn cũng có thể thiết lập cài đặt có sẵn vào trong file `pint.json` của project:

```json
{
    "preset": "psr12"
}
```

Các cài đặt có sẵn hiện được Pint hỗ trợ là: `laravel`, `per`, `psr12` và `symfony`.

<a name="rules"></a>
### Rules

Quy tắc là hướng dẫn về style mà Pint sẽ sử dụng để sửa các vấn đề về code style trong code của bạn. Như đã đề cập ở trên, các cài đặt có sẵn là các nhóm quy tắc được định nghĩa trước, hoàn hảo cho hầu hết các dự án PHP, vì vậy bạn thường không cần phải lo lắng về các quy tắc riêng mà chúng chứa.

Tuy nhiên, nếu bạn muốn, bạn có thể enable hoặc disable các quy tắc riêng đó trong file `pint.json` của bạn:

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "braces": false,
        "new_with_braces": {
            "anonymous_class": false,
            "named_class": false
        }
    }
}
```
Pint được xây dựng dựa trên [PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer). Do đó, bạn có thể sử dụng bất kỳ quy tắc nào của PHP-CS-Fixer để khắc phục các lỗi code style trong project của bạn: [PHP-CS-Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator).

<a name="excluding-files-or-folders"></a>
### Loại trừ file hoặc folder

Mặc định, Pint sẽ kiểm tra tất cả các file `.php` có trong dự án của bạn ngoại trừ các file có trong thư mục `vendor`. Nếu bạn muốn bỏ qua nhiều thư mục hơn, bạn có thể thực hiện điều này bằng tùy chọn cấu hình `exclude`:

```json
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

Nếu bạn muốn bỏ qua tất cả các file có chứa pattern tên đã cho, bạn có thể thực hiện việc này bằng cách sử dụng tùy chọn cấu hình `notName`:

```json
{
    "notName": [
        "*-my-file.php"
    ]
}
```

Nếu bạn muốn bỏ qua một file bằng cách cung cấp đường dẫn đến file đó, bạn có thể thực hiện điều này bằng cách sử dụng tùy chọn cấu hình `notPath`:

```json
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```
