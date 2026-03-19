# Hashing

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Cách dùng cơ bản](#basic-usage)
    - [Hashing một mật khẩu](#hashing-passwords)
    - [Kiểm tra một mật khẩu khớp với một hashing](#verifying-that-a-password-matches-a-hash)
    - [Kiểm tra một mật khẩu cần re-hashing](#determining-if-a-password-needs-to-be-rehashed)
- [Verify thuật toán hash](#hash-algorithm-verification)

<a name="introduction"></a>
## Giới thiệu

[Facade](/docs/{{version}}/facades) `Hash` của Laravel cung cấp hashing Bcrypt và hashing Argon2 để lưu trữ mật khẩu của người dùng. Nếu bạn đang sử dụng một trong những [bộ dụng cụ khởi động ứng dụng Laravel](/docs/{{version}}/starter-kits), Bcrypt sẽ được sử dụng để đăng ký và authentication cho bạn.

Bcrypt là một lựa chọn tuyệt vời để hashing mật khẩu vì "work factor" của nó có thể điều chỉnh được, điều đó có nghĩa là thời gian cần thiết để tạo ra một chuỗi hash có thể tăng lên khi sức mạnh phần cứng tăng lên. Khi hashing mật khẩu, chậm là tốt. Thuật toán càng mất nhiều thời gian để hashing mật khẩu, thì người dùng xấu lại càng mất nhiều thời gian để tạo một "rainbow tables" của tất cả các giá trị hashing chuỗi có thể được sử dụng trong các cuộc tấn công brute force chống lại các ứng dụng.

<a name="configuration"></a>
## Cấu hình

Mặc định, Laravel sử dụng driver hashing `bcrypt` khi hashing dữ liệu. Tuy nhiên, một số driver hashing khác cũng được hỗ trợ, có cả [argon](https://en.wikipedia.org/wiki/Argon2) và [argon2id](https://en.wikipedia.org/wiki/Argon2).

Bạn có thể chỉ định driver hashing cho ứng dụng của bạn bằng cách sử dụng biến môi trường `HASH_DRIVER`. Tuy nhiên, nếu bạn muốn tùy chỉnh tất cả các tùy chọn cấu hình của driver hashing có trong Laravel, bạn nên export file cấu hình `hashing` bằng lệnh Artisan `config:publish`:

```shell
php artisan config:publish hashing
```

<a name="basic-usage"></a>
## Cách dùng cơ bản

<a name="hashing-passwords"></a>
### Hashing một mật khẩu

Bạn có thể hash một mật khẩu bằng cách gọi phương thức `make` trên facade `Hash`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class PasswordController extends Controller
{
    /**
     * Update the password for the user.
     */
    public function update(Request $request): RedirectResponse
    {
        // Validate the new password length...

        $request->user()->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();

        return redirect('/profile');
    }
}
```

<a name="adjusting-the-bcrypt-work-factor"></a>
#### Adjusting The Bcrypt Work Factor

Nếu bạn đang dùng thuật toán Bcrypt, thì phương thức `make` cũng cho phép bạn quản lý work factor của thuật toán bcrypt hashing bằng cách sử dụng tùy chọn `rounds`; tuy nhiên, giá trị work factor mặc định của Laravel được chấp nhận cho hầu hết các application:

```php
$hashed = Hash::make('password', [
    'rounds' => 12,
]);
```

<a name="adjusting-the-argon2-work-factor"></a>
#### Adjusting The Argon2 Work Factor

Nếu bạn đang sử dụng thuật toán Argon2, phương thức `make` cho phép bạn quản lý work factor của thuật toán bằng cách sử dụng các tùy chọn `memory`, `time`, and `threads`; tuy nhiên, giá trị mặc định của Laravel được chấp nhận cho hầu hết các application:

```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

> [!NOTE]
> Để biết thêm thông tin về các tùy chọn này, xin vui lòng tham khảo [tài liệu chính thức của PHP về Argon hashing](https://secure.php.net/manual/en/function.password-hash.php).

<a name="verifying-that-a-password-matches-a-hash"></a>
### Kiểm tra một mật khẩu khớp với một hashing

Phương thức `check` được cung cấp facade `Hash` cho phép bạn xác minh một chuỗi plain-text có tương ứng với một chuỗi đã được hash hay không:

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // The passwords match...
}
```

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### Kiểm tra một mật khẩu cần re-hashing

Hàm `needsRehash` được cung cấp facade `Hash` cho phép bạn kiểm tra xem work factor đã bị thay đổi kể từ sau khi mật khẩu được hash hay chưa. Một số ứng dụng chọn thực hiện kiểm tra này trong quá trình xác thực của ứng dụng:

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```

<a name="hash-algorithm-verification"></a>
## Verify thuật toán hash

Để ngăn chặn việc thao túng thuật toán hash, phương thức `Hash::check` của Laravel sẽ kiểm tra xem chuỗi hash đã cho có được tạo bằng thuật toán hashing đã chọn của ứng dụng hay không. Nếu thuật toán khác với thuật toán đã chọn, thì một ngoại lệ `RuntimeException` sẽ được đưa ra.

Đây là hành vi mong đợi đối với hầu hết tất cả các ứng dụng, nơi mà thuật toán hashing bị thay đổi và sự khác biệt trong thuật toán hashing cũng có thể là một dấu hiệu của một cuộc tấn công. Tuy nhiên, nếu bạn cần hỗ trợ nhiều thuật toán hashing trong ứng dụng của bạn, chẳng hạn như khi migrating từ thuật toán này sang một thuật toán khác, bạn có thể tắt xác minh thuật toán hash bằng cách set biến môi trường `HASH_VERIFY` thành `false`:

```ini
HASH_VERIFY=false
```
