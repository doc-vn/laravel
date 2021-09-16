# Hashing

- [Giới thiệu](#introduction)
- [Cách dùng cơ bản](#basic-usage)

<a name="introduction"></a>
## Giới thiệu

[Facade](/docs/{{version}}/facades) `Hash` của Laravel cung cấp hashing Bcrypt an toàn để lưu trữ mật khẩu người dùng. Nếu bạn đang sử dụng các class `LoginController` và `RegisterController` đi kèm với application Laravel, thì nó sẽ tự động sử dụng Bcrypt để đăng ký và authentication cho bạn.

> {tip} Bcrypt là một lựa chọn tuyệt vời để hashing mật khẩu vì "work factor" của nó có thể điều chỉnh được, điều đó có nghĩa là thời gian cần thiết để tạo ra một chuỗi hash có thể tăng lên khi sức mạnh phần cứng tăng lên.

<a name="basic-usage"></a>
## Cách dùng cơ bản

Bạn có thể hash một mật khẩu bằng cách gọi phương thức `make` trên facade `Hash`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

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

Phương thức `make` cũng cho phép bạn quản lý work factor của thuật toán bcrypt hashing bằng cách sử dụng tùy chọn `rounds`; tuy nhiên, giá trị mặc định được chấp nhận cho hầu hết các application:

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);

#### Verifying A Password Against A Hash

Phương thức `check` cho phép bạn xác minh rằng một chuỗi plain-text có tương ứng với một chuỗi đã được hash hay không. Tuy nhiên, nếu bạn đang sử dụng `LoginController` [đi kèm với Laravel](/docs/{{version}}/authentication), có lẽ bạn sẽ không cần phải sử dụng trực tiếp, vì controller này đã tự động gọi phương thức này:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

#### Checking If A Password Needs To Be Rehashed

Hàm `needsRehash` cho phép bạn xác định xem work factor đã bị thay đổi kể từ sau khi mật khẩu được hash hay chưa:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
