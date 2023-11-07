# Encryption

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Dùng Encrypter](#using-the-encrypter)

<a name="introduction"></a>
## Giới thiệu

Các service mã hóa của Laravel cung cấp một giao diện đơn giản, thuận tiện để mã hóa và giải mã text thông qua OpenSSL sử dụng chuẩn mã hóa AES-256 và AES-128. Tất cả các giá trị được mã hóa bởi Laravel đều được ký bằng message authentication code (MAC) để không thể bị sửa đổi hoặc giả mạo giá trị của chúng sau khi đã được mã hóa.

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng encrypter của Laravel, bạn phải set tùy chọn cấu hình `key` trong file cấu hình `config/app.php` của bạn. Giá trị cấu hình này được điều khiển bởi biến môi trường `APP_KEY`. Bạn nên sử dụng lệnh `php artisan key:generate` để tạo giá trị cho biến này vì lệnh `key:generate` sẽ sử dụng hàm tạo byte ngẫu nhiên an toàn của PHP để tạo khóa an toàn bằng mật mã cho ứng dụng của bạn. Thông thường, giá trị của biến môi trường `APP_KEY` sẽ được tạo cho bạn trong quá trình [cài đặt Laravel](/docs/{{version}}/installation).

<a name="using-the-encrypter"></a>
## Dùng Encrypter

<a name="encrypting-a-value"></a>
#### Encrypting A Value

Bạn có thể mã hóa một giá trị bằng cách sử dụng phương thức `encryptString` được cung cấp bởi facade `Crypt`. Tất cả các giá trị mã hóa đều được mã hóa bằng OpenSSL và mật mã `AES-256-CBC`. Hơn nữa, tất cả các giá trị mã hóa mà được ký bằng message authentication code (MAC). MAC được tích hợp sẵn sẽ ngăn chặn việc giải mã bất kỳ giá trị nào đã bị giả mạo bởi người dùng:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class DigitalOceanTokenController extends Controller
    {
        /**
         * Store a DigitalOcean API token for the user.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function storeSecret(Request $request)
        {
            $request->user()->fill([
                'token' => Crypt::encryptString($request->token),
            ])->save();
        }
    }

<a name="decrypting-a-value"></a>
#### Decrypting A Value

Bạn có thể giải mã các giá trị bằng cách sử dụng phương thức `decryptString` được cung cấp bởi facade `Crypt`. Nếu giá trị không thể được giải mã chính xác, chẳng hạn như khi message authentication code (MAC) không hợp lệ, một `Illuminate\Contracts\Encryption\DecryptException` sẽ được tạo ra:

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
