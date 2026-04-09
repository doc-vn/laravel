# Laravel Pint

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Chạy Pint](#running-pint)
- [Cấu hình Pint](#configuring-pint)
    - [Presets](#presets)
    - [Rules](#rules)
    - [Loại trừ file hoặc folder](#excluding-files-or-folders)
- [Continuous Integration](#continuous-integration)
    - [GitHub Actions](#running-tests-on-github-actions)

<a name="introduction"></a>
## Giới thiệu

[Laravel Pint](https://github.com/laravel/pint) là một trình sửa lỗi code cho PHP dành cho những người theo chủ nghĩa tối giản. Pint được xây dựng trên [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) và giúp đảm bảo code của bạn luôn sạch sẽ và nhất quán.

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

Nếu bạn muốn Pint chạy ở chế độ song song (thử nghiệm) để cải thiện hiệu suất, bạn có thể sử dụng tùy chọn `--parallel`:

```shell
./vendor/bin/pint --parallel
```

Chế độ song song cũng cho phép bạn chỉ định số lượng process tối đa được chạy thông qua tùy chọn `--max-processes`. Nếu tùy chọn này không được cung cấp, Pint sẽ sử dụng tất cả các core có sẵn trên máy của bạn:

```shell
./vendor/bin/pint --parallel --max-processes=4
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

Nếu bạn muốn Pint chỉ kiểm tra các lỗi code style của bạn mà không muốn thay đổi các file, bạn có thể sử dụng tùy chọn `--test`. Pint sẽ trả về exit code không phải là 0 nếu có lỗi code style nào được tìm thấy:

```shell
./vendor/bin/pint --test
```

Nếu bạn muốn Pint chỉ sửa các file khác với branch được cung cấp trong Git, bạn có thể sử dụng tùy chọn `--diff=[branch]`. Điều này có thể được sử dụng hiệu quả trong môi trường CI của bạn (như GitHub actions) để tiết kiệm thời gian bằng cách chỉ kiểm tra các file mới hoặc đã sửa trước đó:

```shell
./vendor/bin/pint --diff=main
```

Nếu bạn muốn Pint chỉ sửa các file có thay đổi chưa được commit vào Git, bạn có thể sử dụng tùy chọn `--dirty`:

```shell
./vendor/bin/pint --dirty
```

Nếu bạn muốn Pint sửa bất kỳ file nào có lỗi code style nhưng vẫn trả về exit code khác 0 nếu có bất kỳ lỗi nào được sửa, bạn có thể sử dụng tùy chọn `--repair`:

```shell
./vendor/bin/pint --repair
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
./vendor/bin/pint --config vendor/my-company/coding-style/pint.json
```

<a name="presets"></a>
### Presets

Presets định nghĩa một tập hợp các quy tắc có thể được sử dụng để sửa các vấn đề về coding style trong code của bạn. Mặc định, Pint sử dụng cài đặt có sẵn `laravel`, cài đặt này sửa các vấn đề bằng cách tuân theo coding style ​​của Laravel. Tuy nhiên, bạn cũng có thể chỉ định một cài đặt có sẵn khác bằng cách cung cấp tùy chọn `--preset` cho Pint:

```shell
./vendor/bin/pint --preset psr12
```

Nếu bạn muốn, bạn cũng có thể thiết lập cài đặt có sẵn vào trong file `pint.json` của project:

```json
{
    "preset": "psr12"
}
```

Các cài đặt có sẵn hiện được Pint hỗ trợ là: `laravel`, `per`, `psr12`, `symfony`, và `empty`.

<a name="rules"></a>
### Rules

Quy tắc là hướng dẫn về style mà Pint sẽ sử dụng để sửa các vấn đề về code style trong code của bạn. Như đã đề cập ở trên, các cài đặt có sẵn là các nhóm quy tắc được định nghĩa trước, hoàn hảo cho hầu hết các dự án PHP, vì vậy bạn thường không cần phải lo lắng về các quy tắc riêng mà chúng chứa.

Tuy nhiên, nếu bạn muốn, bạn có thể enable hoặc disable các quy tắc riêng đó trong file `pint.json` của bạn hoặc sử dụng preset `empty` và định nghĩa các quy tắc từ đầu:

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "array_indentation": false,
        "new_with_parentheses": {
            "anonymous_class": true,
            "named_class": true
        }
    }
}
```

Pint được xây dựng dựa trên [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer). Do đó, bạn có thể sử dụng bất kỳ quy tắc nào của PHP-CS-Fixer để khắc phục các lỗi code style trong project của bạn: [PHP CS Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator).

<a name="custom-rules"></a>
#### Custom Rules

Ngoài các rules của PHP CS Fixer, Pint còn cung cấp thêm các rule tùy chỉnh có prefix là `Pint/`. Các rule này mặc định không được enable, nhưng bạn có thể enable chúng trong file `pint.json` của bạn.

<a name="phpdoc-type-annotations-only"></a>
##### `Pint/phpdoc_type_annotations_only`

Rule này sẽ xóa tất cả các comment và docblock ra khỏi code của bạn, chỉ giữ lại các dòng có chứa các annotation `@` như `@param`, `@return`, `@var`, `@phpstan-type`, vv...:

```php
/**
 * Get the posts for the user. [tl! remove]
 * [tl! remove]
 * @return HasMany<Post, $this>
 */
public function posts(): HasMany
```

Các comment một dòng và comment block mà không có annotation `@` sẽ bị xóa. Nếu bạn muốn giữ lại một comment cụ thể, bạn có thể thêm prefix `@note`, `@warning`, hoặc `@todo` vào trước comment đó:

```php
// @note This comment will be preserved.
```

Để enable rule này, hãy thêm nó vào file `pint.json` của bạn:

```json
{
    "preset": "laravel",
    "rules": {
        "Pint/phpdoc_type_annotations_only": true
    }
}
```

> [!NOTE]
> Rule này sẽ tự động bỏ qua các file trong thư mục `config`, vì các file cấu hình thường dựa vào các comment để làm tài liệu hướng dẫn.

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


<a name="continuous-integration"></a>
## Continuous Integration

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

Để tự động sửa project của bạn bằng Laravel Pint, bạn có thể cấu hình [GitHub Actions](https://github.com/features/actions) để chạy Pint bất cứ khi nào có code mới được push lên GitHub. Đầu tiên, hãy đảm bảo cấp "quyền Read và write" cho các workflow trong GitHub tại **Settings > Actions > General > Workflow permissions**. Sau đó, bạn hãy tạo một file `.github/workflows/lint.yml` với nội dung sau:

```yaml
name: Fix Code Style

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
      matrix:
        php: [8.4]

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          tools: pint

      - name: Run Pint
        run: pint

      - name: Commit linted files
        uses: stefanzweifel/git-auto-commit-action@v6
```
