# Encryption

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Dùng Encrypter](#using-the-encrypter)

<a name="introduction"></a>
## Giới thiệu

Bộ mã hóa Laravel sử dụng là OpenSSL, cung cấp các loại mã hóa AES-256 và AES-128. Bạn được khuyến khích sử dụng các hệ thống mã hóa đi kèm với Laravel và đừng cố gắng tạo ra các thuật toán mã hóa "home grown" của riêng bạn. Tất cả các giá trị được mã hóa bởi Laravel đều được ký bằng message authentication code (MAC) để không thể bị sửa đổi giá trị của chúng sau khi đã được mã hóa.

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng bộ mã hóa của Laravel, bạn cần phải cài đặt tùy chọn `key` trong file cấu hình `config/app.php` của bạn. Bạn nên sử dụng lệnh `php artisan key:generate` để tạo key này vì lệnh Artisan này sẽ sử dụng trình tạo byte ngẫu nhiên an toàn của PHP để tạo key cho bạn. Nếu giá trị này không được cài đặt đúng cách, tất cả các giá trị được mã hóa bởi Laravel sẽ không an toàn.

<a name="using-the-encrypter"></a>
## Dùng Encrypter

#### Encrypting A Value

Bạn có thể mã hóa một giá trị bằng cách sử dụng phương thức `encryptString` của facade `Crypt`. Tất cả các giá trị mã hóa đều được mã hóa bằng OpenSSL và mật mã `AES-256-CBC`. Hơn nữa, tất cả các giá trị mã hóa mà được ký bằng message authentication code (MAC) đều có thể phát hiện bất kỳ sửa đổi nào đối với giá trị đã được mã hóa:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => Crypt::encryptString($request->secret),
            ])->save();
        }
    }

#### Decrypting A Value

Bạn có thể giải mã các giá trị bằng cách sử dụng phương thức `decryptString` của facade `Crypt`. Nếu giá trị không thể được giải mã chính xác, chẳng hạn như khi MAC không hợp lệ, một `Illuminate\Contracts\Encryption\DecryptException` sẽ được tạo ra:

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
