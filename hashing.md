# Hashing

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Cách dùng cơ bản](#basic-usage)

<a name="introduction"></a>
## Giới thiệu

[Facade](/docs/{{version}}/facades) `Hash` của Laravel cung cấp hashing Bcrypt và hashing Argon2 để lưu trữ mật khẩu của người dùng. Nếu bạn đang sử dụng các class `LoginController` và `RegisterController` mà đi kèm với application Laravel, thì mặc định nó sẽ sử dụng Bcrypt để đăng ký và authentication cho bạn.

> {tip} Bcrypt là một lựa chọn tuyệt vời để hashing mật khẩu vì "work factor" của nó có thể điều chỉnh được, điều đó có nghĩa là thời gian cần thiết để tạo ra một chuỗi hash có thể tăng lên khi sức mạnh phần cứng tăng lên.

<a name="configuration"></a>
## Cấu hình

Driver hashing mặc định cho ứng dụng của bạn sẽ được cấu hình trong file cấu hình `config/hashing.php`. Hiện tại có hai driver được hỗ trợ: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) và [Argon2](https://en.wikipedia.org/wiki/Argon2) (Argon2i và biến thể Argon2id).

> {note} Driver Argon2i yêu cầu PHP 7.2.0 hoặc hơn và Driver Argon2id yêu cầu PHP 7.3.0 hoặc hơn.

<a name="basic-usage"></a>
## Cách dùng cơ bản

Bạn có thể hash một mật khẩu bằng cách gọi phương thức `make` trên facade `Hash`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class UpdatePasswordController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Validate the new password length...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

#### Adjusting The Bcrypt Work Factor

Nếu bạn đang thuật toán Bcrypt, thì phương thức `make` cũng cho phép bạn quản lý work factor của thuật toán bcrypt hashing bằng cách sử dụng tùy chọn `rounds`; tuy nhiên, giá trị mặc định được chấp nhận cho hầu hết các application:

    $hashed = Hash::make('password', [
        'rounds' => 12,
    ]);

#### Adjusting The Argon2 Work Factor

Nếu bạn đang sử dụng thuật toán Argon2, phương thức `make` cho phép bạn quản lý work factor của thuật toán bằng cách sử dụng các tùy chọn `memory`, `time`, and `threads`; tuy nhiên, giá trị mặc định được chấp nhận cho hầu hết các application:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> {tip} Để biết thêm thông tin về các tùy chọn này, hãy xem [tài liệu PHP chính thức](https://secure.php.net/manual/en/function.password-hash.php).

#### Verifying A Password Against A Hash

Phương thức `check` cho phép bạn xác minh một chuỗi plain-text có tương ứng với một chuỗi đã được hash hay không. Tuy nhiên, nếu bạn đang sử dụng `LoginController` [đi kèm với Laravel](/docs/{{version}}/authentication), có lẽ bạn sẽ không cần phải sử dụng trực tiếp phương thức này, vì controller này đã tự động gọi phương thức đó:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

#### Checking If A Password Needs To Be Rehashed

Hàm `needsRehash` cho phép bạn kiểm tra xem work factor đã bị thay đổi kể từ sau khi mật khẩu được hash hay chưa:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
