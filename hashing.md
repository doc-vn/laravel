# Hashing

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Cách dùng cơ bản](#basic-usage)
    - [Hashing một mật khẩu](#hashing-passwords)
    - [Kiểm tra một mật khẩu khớp với một hashing](#verifying-that-a-password-matches-a-hash)
    - [Kiểm tra một mật khẩu cần re-hashing](#determining-if-a-password-needs-to-be-rehashed)

<a name="introduction"></a>
## Giới thiệu

[Facade](/docs/{{version}}/facades) `Hash` của Laravel cung cấp hashing Bcrypt và hashing Argon2 để lưu trữ mật khẩu của người dùng. Nếu bạn đang sử dụng một trong những [bộ dụng cụ khởi động ứng dụng Laravel](/docs/{{version}}/starter-kits), Bcrypt sẽ được sử dụng để đăng ký và authentication cho bạn.

Bcrypt là một lựa chọn tuyệt vời để hashing mật khẩu vì "work factor" của nó có thể điều chỉnh được, điều đó có nghĩa là thời gian cần thiết để tạo ra một chuỗi hash có thể tăng lên khi sức mạnh phần cứng tăng lên. Khi hashing mật khẩu, chậm là tốt. Thuật toán càng mất nhiều thời gian để hashing mật khẩu, thì người dùng xấu lại càng mất nhiều thời gian để tạo một "rainbow tables" của tất cả các giá trị hashing chuỗi có thể được sử dụng trong các cuộc tấn công brute force chống lại các ứng dụng.

<a name="configuration"></a>
## Cấu hình

Driver hashing mặc định cho ứng dụng của bạn sẽ được cấu hình trong file cấu hình `config/hashing.php` của ứng dụng của bạn. Hiện tại có nhiều driver được hỗ trợ: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) và [Argon2](https://en.wikipedia.org/wiki/Argon2) (Argon2i và biến thể Argon2id).

> {note} Driver Argon2i yêu cầu PHP 7.2.0 hoặc hơn và Driver Argon2id yêu cầu PHP 7.3.0 hoặc hơn.

<a name="basic-usage"></a>
## Cách dùng cơ bản

<a name="hashing-passwords"></a>
### Hashing một mật khẩu

Bạn có thể hash một mật khẩu bằng cách gọi phương thức `make` trên facade `Hash`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class PasswordController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request)
        {
            // Validate the new password length...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

<a name="adjusting-the-bcrypt-work-factor"></a>
#### Adjusting The Bcrypt Work Factor

Nếu bạn đang dùng thuật toán Bcrypt, thì phương thức `make` cũng cho phép bạn quản lý work factor của thuật toán bcrypt hashing bằng cách sử dụng tùy chọn `rounds`; tuy nhiên, giá trị work factor mặc định của Laravel được chấp nhận cho hầu hết các application:

    $hashed = Hash::make('password', [
        'rounds' => 12,
    ]);

<a name="adjusting-the-argon2-work-factor"></a>
#### Adjusting The Argon2 Work Factor

Nếu bạn đang sử dụng thuật toán Argon2, phương thức `make` cho phép bạn quản lý work factor của thuật toán bằng cách sử dụng các tùy chọn `memory`, `time`, and `threads`; tuy nhiên, giá trị mặc định của Laravel được chấp nhận cho hầu hết các application:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> {tip} Để biết thêm thông tin về các tùy chọn này, xin vui lòng tham khảo [tài liệu chính thức của PHP về Argon hashing](https://secure.php.net/manual/en/function.password-hash.php).

<a name="verifying-that-a-password-matches-a-hash"></a>
### Kiểm tra một mật khẩu khớp với một hashing

Phương thức `check` được cung cấp facade `Hash` cho phép bạn xác minh một chuỗi plain-text có tương ứng với một chuỗi đã được hash hay không:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### Kiểm tra một mật khẩu cần re-hashing

Hàm `needsRehash` được cung cấp facade `Hash` cho phép bạn kiểm tra xem work factor đã bị thay đổi kể từ sau khi mật khẩu được hash hay chưa. Một số ứng dụng chọn thực hiện kiểm tra này trong quá trình xác thực của ứng dụng:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
