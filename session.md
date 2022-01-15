# HTTP Session

- [Giới thiệu](#introduction)
    - [Cấu hình](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Dùng Session](#using-the-session)
    - [Lấy dữ liệu](#retrieving-data)
    - [Lưu dữ liệu](#storing-data)
    - [Flash dữ liệu](#flash-data)
    - [Xoá dữ liệu](#deleting-data)
    - [Tạo Session ID](#regenerating-the-session-id)
- [Thêm tuỳ chỉnh Session Drivers](#adding-custom-session-drivers)
    - [Implementing Driver](#implementing-the-driver)
    - [Đăng ký Driver](#registering-the-driver)

<a name="introduction"></a>
## Giới thiệu

Vì các HTTP driven sẽ không lưu trạng thái của người dùng qua mỗi request, nên các session sẽ cung cấp một cách dễ dàng để lưu trữ thông tin về người dùng qua mỗi request. Laravel có nhiều session backend khác nhau và có thể được truy cập thông qua một API thống nhất, rõ ràng. Laravel hỗ trợ cho các backend phổ biến như [Memcached](https://memcached.org), [Redis](https://redis.io) và có cả database.

<a name="configuration"></a>
### Cấu hình

File cấu hình cho session sẽ được lưu trữ tại `config/session.php`. Bạn hãy xem qua các tùy chọn có sẵn cho bạn trong file này. Mặc định, Laravel sẽ cấu hình sử dụng session driver `file`, nó sẽ hoạt động tốt cho nhiều application. Khi mà application của bạn ở trạng thái production, thì bạn có thể cân nhắc sử dụng driver `memcached` hoặc `redis` để có được hiệu suất session nhanh hơn.

Tham số `driver` sẽ khai báo nơi mà dữ liệu của session sẽ được lưu trữ cho mỗi request. Mặc định, Laravel đã có sẵn một số driver:

<div class="content-list" markdown="1">
- `file` - sessions được lưu ở file `storage/framework/sessions`.
- `cookie` - sessions được lưu trữ trong cookie và đã được mã hóa.
- `database` - sessions được lưu trữ trong cơ sở dữ liệu.
- `memcached` / `redis` - sessions được lưu trữ ở trong những cache base này.
- `array` - sessions được lưu trữ trong một mảng PHP và sẽ không được duy trì.
</div>

> {tip} Array driver sẽ được sử dụng trong các [testing](/docs/{{version}}/testing) để ngăn việc dữ liệu được lưu trữ trong session.

<a name="driver-prerequisites"></a>
### Điều kiện dùng Driver

#### Database

Khi sử dụng session driver `database`, bạn sẽ cần tạo một bảng để chứa các session. Dưới đây là một ví dụ khai báo `Schema` cho bảng:

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->unsignedInteger('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

Bạn có thể dùng lệnh Artisan `session:table` để tạo file migration đó:

    php artisan session:table

    php artisan migrate

#### Redis

Trước khi sử dụng session Redis cùng với Laravel, bạn sẽ cần phải cài đặt một package `predis/predis` (~ 1.0) thông qua Composer. Bạn có thể cấu hình các kết nối Redis của bạn trong file cấu hình `database`. Trong file cấu hình `session` sẽ có tùy chọn `connection` để có thể được sử dụng để định nghĩa kết nối Redis nào mà có thể được sử dụng bởi session.

<a name="using-the-session"></a>
## Dùng Session

<a name="retrieving-data"></a>
### Lấy dữ liệu

Có hai cách chính để truy cập vào dữ liệu session trong Laravel: global helper `session` và thông qua một instance `Request`. Đầu tiên, chúng ta hãy xem xét việc truy cập session thông qua một instance `Request`, nó có thể được khai báo dưới dạng kiểu trong một phương thức controller. Hãy nhớ rằng, các phụ thuộc của phương thức controller được tự động đưa vào thông qua [service container](/docs/{{version}}/container) của Laravel:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

Khi bạn muốn lấy một item từ session, bạn có thể truyền vào một giá trị mặc định làm tham số thứ hai cho phương thức `get`. Giá trị mặc định này sẽ được trả về nếu key bạn muốn lấy không tồn tại trong session. Nếu bạn truyền vào một `Closure` làm giá trị mặc định cho phương thức `get` nếu key được yêu cầu không tồn tại, thì `Closure` sẽ được thực thi và kết quả của nó sẽ được trả về:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

#### Global helper session

Bạn cũng có thể sử dụng qua hàm PHP global `session` để lấy và lưu trữ dữ liệu vào trong session. Khi helper `session` được gọi với một tham số là key, nó sẽ trả về giá trị của key đó trong session. Khi helper được gọi với một loạt các cặp key / value, thì các giá trị đó sẽ được lưu trữ vào trong session:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');

        // Specifying a default value...
        $value = session('key', 'default');

        // Store a piece of data in the session...
        session(['key' => 'value']);
    });

> {tip} Có rất ít sự khác biệt giữa việc sử dụng session thông qua instance request HTTP và sử dụng thông qua global helper `session`. Cả hai phương thức đều có thể [test](/docs/{{version}}/testing) thông qua phương thức `assertSessionHas` có sẵn trong tất cả các test case của bạn.

#### Lấy tất cả dữ liệu trong session

Nếu bạn muốn lấy tất cả dữ liệu trong session, bạn có thể sử dụng phương thức `all`:

    $data = $request->session()->all();

#### Xác định một item có tồn tại trong session hay không

Để xác định xem một item có trong session hay không, bạn có thể sử dụng phương thức `has`. Phương thức `has` sẽ trả về `true` nếu item đó tồn tại và khác giá trị `null`:

    if ($request->session()->has('users')) {
        //
    }

Để xác định xem một item có trong session hay không, ngay cả khi giá trị của nó là `null`, thì bạn có thể sử dụng phương thức `exists`. Phương thức `exists` trả về `true` nếu item đó tồn tại:

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### Lưu dữ liệu

Để lưu dữ liệu vào trong session, bạn có thể sử dụng phương thức `put` hoặc helper `session`:

    // Via a request instance...
    $request->session()->put('key', 'value');

    // Via the global helper...
    session(['key' => 'value']);

#### Push một giá trị mới vào một mảng value trong session

Phương thức `push` có thể được sử dụng để đẩy một giá trị mới vào trong một mảng value session. Ví dụ: nếu key `user.teams` chứa một mảng gồm các tên team, bạn có thể đẩy thêm một giá trị mới vào mảng đó như sau:

    $request->session()->push('user.teams', 'developers');

#### Lấy và xoá một item

Phương thức `pull` sẽ lấy ra và xóa đi một item ra khỏi session chỉ với một câu lệnh:

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### Flash dữ liệu

Thỉnh thoảng bạn chỉ muốn lưu trữ các item trong session cho đến request kế tiếp. Bạn có thể làm như vậy bằng cách sử dụng phương thức `flash`. Dữ liệu được lưu trữ trong session sử dụng phương thức này sẽ chỉ được lưu đến request HTTP kế tiếp và sau đó nó sẽ bị xóa. Dữ liệu flash chủ yếu hữu ích khi dùng cho các thông báo trạng thái ngắn:

    $request->session()->flash('status', 'Task was successful!');

Nếu bạn cần lưu dữ liệu flash của mình cho một số request, bạn có thể sử dụng phương thức `reflash`, phương thức này sẽ lưu tất cả dữ liệu đã được flash thêm một request nữa. Nếu bạn chỉ cần lưu một dữ liệu flash cụ thể nào đó, bạn có thể sử dụng phương thức `keep`:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### Xoá dữ liệu

Phương thức `forget` sẽ xóa một phần dữ liệu ra khỏi session. Nếu bạn muốn xóa tất cả dữ liệu ra khỏi session, bạn có thể sử dụng phương thức `flush`:

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### Tạo Session ID

Việc tạo lại session ID thường được thực hiện để ngăn kẻ xấu khai thác lỗ hỏng bảo mật [session fixation](https://en.wikipedia.org/wiki/Session_fixation) trên application của bạn.

 Nếu bạn đang sử dụng `LoginController`, thì Laravel sẽ tự động tạo lại session ID trong khi xác thực; tuy nhiên, nếu bạn cần tự tạo lại session ID, bạn có thể sử dụng phương thức `regenerate`.

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## Thêm tuỳ chỉnh Session Drivers

<a name="implementing-the-driver"></a>
#### Implementing Driver

Driver session tùy chỉnh của bạn sẽ cần phải implement từ `SessionHandlerInterface`. Interface này chỉ chứa một vài phương thức đơn giản mà bạn cần implement. Một implementation MongoDB đơn giản sẽ trông giống như thế này:

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> {tip} Laravel sẽ không định nghĩa một thư mục để chứa các extension cho bạn. Bạn có thể tự do lưu extension của bạn vào bất kỳ nơi nào mà bạn thích. Trong ví dụ này, chúng tôi đã tạo một thư mục `Extensions` để chứa `MongoSessionHandler`.

Vì mục đích của những phương thức này là không dễ hiểu, chúng ta hãy nhanh chóng xem những gì mà mỗi phương thức làm:

<div class="content-list" markdown="1">
- Phương thức `open` thường được sử dụng trong các hệ thống lưu trữ session dựa trên file. Vì Laravel đã định nghĩa driver session `file`, nên bạn sẽ không cần phải lưu bất cứ thứ gì vào trong phương thức này. Bạn có thể để nó trống. Đó là một thực tế của interface design kém (mà chúng ta sẽ thảo luận sau) nhưng PHP yêu cầu chúng ta implement phương thức này.
- Phương thức `close`, giống như phương thức` open`, thường có thể bị bỏ qua. Đối với hầu hết các driver, nó là không cần thiết.
- Phương thức `read` sẽ trả về string của dữ liệu session được liên kết với `$sessionId` đã cho. Bạn sẽ không cần thực hiện bất kỳ việc chuyển đổi hoặc encoding nào khác khi truy xuất hoặc lưu trữ dữ liệu session vào trong driver của bạn, vì Laravel sẽ thực hiện việc chuyển đổi cho bạn.
- Phương thức `write` sẽ viết chuỗi `$data` đã cho liên kết với một `$sessionId` vào trong một số hệ thống lưu trữ, chẳng hạn như MongoDB, Dynamo, vv. Một lần nữa, bạn không nên thực hiện bất kỳ chuyển đổi nào - Laravel sẽ xử lý điều đó cho bạn.
- Phương thức `destroy` sẽ xóa dữ liệu được liên kết với `$sessionId` ra khỏi bộ lưu trữ.
- Phương thức `gc` sẽ hủy tất cả dữ liệu session cũ hơn so với `$lifetime` đã cho, đó là UNIX timestamp. Đối với các hệ thống tự hết hạn như Memcached và Redis, phương thức này có thể bị bỏ trống.
</div>

<a name="registering-the-driver"></a>
#### Đăng ký Driver

Khi driver của bạn đã được thực hiện xong, bạn đã sẵn sàng để đăng ký nó với framework. Để thêm driver vào backend session của Laravel, bạn có thể sử dụng phương thức `extend` trong [facade](/docs/{{version}}/facades) `Session`. Bạn nên gọi phương thức `extend` từ phương thức `boot` của một [service provider](/docs/{{version}}/providers). Bạn có thể làm điều này từ `AppServiceProvider` hoặc tạo một provider mới hoàn toàn:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function ($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionHandler;
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Khi driver session đã được đăng ký, bạn có thể sử dụng driver `mongo` trong file cấu hình `config/session.php` của bạn.
