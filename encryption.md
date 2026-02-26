# Encryption

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Thay đổi key encrytion](#gracefully-rotating-encryption-keys)
- [Dùng Encrypter](#using-the-encrypter)

<a name="introduction"></a>
## Giới thiệu

Các service mã hóa của Laravel cung cấp một giao diện đơn giản, thuận tiện để mã hóa và giải mã text thông qua OpenSSL sử dụng chuẩn mã hóa AES-256 và AES-128. Tất cả các giá trị được mã hóa bởi Laravel đều được ký bằng message authentication code (MAC) để không thể bị sửa đổi hoặc giả mạo giá trị của chúng sau khi đã được mã hóa.

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng encrypter của Laravel, bạn phải set tùy chọn cấu hình `key` trong file cấu hình `config/app.php` của bạn. Giá trị cấu hình này được điều khiển bởi biến môi trường `APP_KEY`. Bạn nên sử dụng lệnh `php artisan key:generate` để tạo giá trị cho biến này vì lệnh `key:generate` sẽ sử dụng hàm tạo byte ngẫu nhiên an toàn của PHP để tạo khóa an toàn bằng mật mã cho ứng dụng của bạn. Thông thường, giá trị của biến môi trường `APP_KEY` sẽ được tạo cho bạn trong quá trình [cài đặt Laravel](/docs/{{version}}/installation).

<a name="gracefully-rotating-encryption-keys"></a>
### Thay đổi key encrytion

Nếu bạn thay đổi key encrytion của ứng dụng, tất cả các session người dùng đã xác thực sẽ bị đăng xuất khỏi ứng dụng. Điều này là do mọi cookie, bao gồm cả cookie session, đều được Laravel mã hóa. Ngoài ra, bạn cũng sẽ không thể giải mã bất kỳ dữ liệu nào đã được mã hóa bằng key encrytion trước đó nữa.

Để giảm thiểu vấn đề này, Laravel cho phép bạn liệt kê các key encrytion trước đó trong biến môi trường `APP_PREVIOUS_KEYS` của ứng dụng. Biến này có thể chứa danh sách tất cả các key encrytion trước đó của bạn, phân tách bằng dấu phẩy:

```ini
APP_KEY="base64:J63qRTDLub5NuZvP+kb8YIorGS6qFYHKVo6u7179stY="
APP_PREVIOUS_KEYS="base64:2nLsGFGzyoae2ax3EF2Lyq/hH6QghBGLIq5uL+Gp8/w="
```

Khi bạn thiết lập biến môi trường này, Laravel sẽ luôn sử dụng key encrytion "hiện tại" khi mã hóa giá trị. Tuy nhiên, khi giải mã giá trị, thì Laravel sẽ thử khóa hiện tại trước, và nếu giải mã không thành công bằng khóa hiện tại, thì Laravel sẽ thử tất cả các khóa trước đó cho đến khi một trong các khóa đó có thể giải mã giá trị.

Phương án giải mã an toàn này cho phép người dùng tiếp tục sử dụng ứng dụng mà không bị gián đoạn ngay cả khi key encrytion của bạn bị thay đổi.

<a name="using-the-encrypter"></a>
## Dùng Encrypter

<a name="encrypting-a-value"></a>
#### Encrypting A Value

Bạn có thể mã hóa một giá trị bằng cách sử dụng phương thức `encryptString` được cung cấp bởi facade `Crypt`. Tất cả các giá trị mã hóa đều được mã hóa bằng OpenSSL và mật mã `AES-256-CBC`. Hơn nữa, tất cả các giá trị mã hóa mà được ký bằng message authentication code (MAC). MAC được tích hợp sẵn sẽ ngăn chặn việc giải mã bất kỳ giá trị nào đã bị giả mạo bởi người dùng:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Crypt;

class DigitalOceanTokenController extends Controller
{
    /**
     * Store a DigitalOcean API token for the user.
     */
    public function store(Request $request): RedirectResponse
    {
        $request->user()->fill([
            'token' => Crypt::encryptString($request->token),
        ])->save();

        return redirect('/secrets');
    }
}
```

<a name="decrypting-a-value"></a>
#### Decrypting A Value

Bạn có thể giải mã các giá trị bằng cách sử dụng phương thức `decryptString` được cung cấp bởi facade `Crypt`. Nếu giá trị không thể được giải mã chính xác, chẳng hạn như khi message authentication code (MAC) không hợp lệ, một `Illuminate\Contracts\Encryption\DecryptException` sẽ được tạo ra:

```php
use Illuminate\Contracts\Encryption\DecryptException;
use Illuminate\Support\Facades\Crypt;

try {
    $decrypted = Crypt::decryptString($encryptedValue);
} catch (DecryptException $e) {
    // ...
}
```
