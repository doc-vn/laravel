# Encryption

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Dùng Encrypter](#using-the-encrypter)

<a name="introduction"></a>
## Giới thiệu

Bộ mã hóa của Laravel sử dụng OpenSSL để cung cấp các loại mã hóa AES-256 và AES-128. Bạn được khuyến khích sử dụng các hệ thống mã hóa tích hợp của Laravel và không cố gắng tạo ra các thuật toán mã hóa "home grown" của riêng bạn. Tất cả các giá trị được mã hóa của Laravel được ký bằng  message authentication code (MAC) để không thể sửa đổi giá trị của chúng sau khi được mã hóa.

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng bộ mã hóa của Laravel, bạn phải cài đặt tùy chọn `key` trong file cấu hình `config/app.php` của bạn. Bạn nên sử dụng lệnh `php artisan key:generate` để tạo key này vì lệnh Artisan này sẽ sử dụng trình tạo byte an toàn ngẫu nhiên của PHP để xây dựng key của bạn. Nếu giá trị này không được cài đặt đúng cách, tất cả các giá trị được mã hóa bởi Laravel sẽ không an toàn.

<a name="using-the-encrypter"></a>
## Dùng Encrypter

#### Encrypting A Value

Bạn có thể mã hóa một giá trị bằng cách sử dụng helper `encrypt`. Tất cả các giá trị đã được mã hóa sẽ được mã hóa bằng OpenSSL và mật mã `AES-256-CBC`. Hơn nữa, tất cả các giá trị được mã hóa được ký bằng message authentication code (MAC) để phát hiện bất kỳ sửa đổi nào đối với chuỗi được mã hóa:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

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
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }

#### Encrypting Without Serialization

Các giá trị đã được mã hóa sẽ được pass qua `serialize` trong quá trình mã hóa, cho phép mã hóa các đối tượng và mảng. Do đó, các client không phải PHP nhận về các giá trị được mã hóa sẽ cần phải `unserialize` dữ liệu. Nếu bạn muốn mã hóa và giải mã các giá trị mà không cần serialization, bạn có thể sử dụng các phương thức `encryptString` và `decryptString` của facade `Crypt`:

    use Illuminate\Support\Facades\Crypt;

    $encrypted = Crypt::encryptString('Hello world.');

    $decrypted = Crypt::decryptString($encrypted);

#### Decrypting A Value

Bạn có thể giải mã các giá trị bằng cách sử dụng helper `decrypt`. Nếu giá trị không thể được giải mã chính xác, chẳng hạn như khi MAC không hợp lệ, một `Illuminate\Contracts\Encryption\DecryptException` sẽ bị tạo:

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
