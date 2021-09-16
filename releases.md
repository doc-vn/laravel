# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 5.5](#laravel-5.5)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Cấu trúc của phiên bản của Laravel được duy trì theo quy ước như sau: `paradigm.major.minor`. Các phiên bản được phát hành chính thức của framework được phát hành sáu tháng một lần (tháng 2 và tháng 8), trong khi các bản phát hành nhỏ có thể được phát hành thường xuyên hơn có thể cho mỗi tuần. Các bản phát hành nhỏ sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application hoặc package của bạn, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như `5.5.*`, Vì các bản phát hành chính thức của Laravel  có thể chứa các thay đổi mà làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng bạn có thể cập nhật lên bản phát hành chính thức mới trong một ngày hoặc ít hơn.

Các bản phát hành thay đổi Paradigm được phân tách qua nhiều năm và đại diện cho những thay đổi căn bản trong kiến trúc và quy ước của framework. Hiện tại, chưa có bản phát hành thay đổi Paradigm nào được phát triển hiện tại.

#### Tại sao Laravel không sử dụng phiên bản Semantic?

Ngoài ra, tất cả các component tùy chọn của Laravel như (Cashier, Dusk, Valet, Socialite, vv...) **đều** sử dụng phiên bản Semantic. Tuy nhiên, bản thân framework Laravel thì không. Lý do cho điều này là bởi vì phiên bản Semantic là một cách "tiếp cận" để xác định xem hai đoạn mã có tương thích hay không. Ngay cả khi sử dụng phiên bản Semantic, bạn vẫn phải cài đặt các package nâng cấp và phải chạy lại bộ test suite của bạn để biết được rằng liệu có gì *thực sự* không tương thích với code của bạn không.

Vì vậy, thay vào đó, framework Laravel sử dụng cấu trúc phiên bản có tính truyền đạt cao hơn về phạm vi phát hành thực tế. Hơn nữa, vì các bản phát hành nhỏ sẽ **không bao giờ** chứa các thay đổi mà dẫn đến hệ thống của bạn bị lỗi, nên bạn sẽ không bao giờ nhận được các thay đổi nào mà có thể gây lỗi, chừng nào các ràng buộc phiên bản của bạn vẫn tuân theo quy ước `paradigm.major.*`.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với các bản phát hành hỗ trợ dài hạn, chẳng hạn như Laravel 5.5, các bản sửa lỗi được cung cấp trong 2 năm và các bản sửa lỗi bảo mật được cung cấp trong 3 năm. Những bản phát hành này cung cấp các hỗ trợ và bảo trì dài nhất. Đối với các bản phát hành bình thường, các bản sửa lỗi được cung cấp trong 6 tháng và các bản sửa lỗi bảo mật được cung cấp trong 1 năm.

<a name="laravel-5.5"></a>
## Laravel 5.5 (LTS)

Laravel 5.5 tiếp tục các cải tiến được thực hiện trong Laravel 5.4 bằng cách thêm tính năng tự động phát hiện package, API resources / chuyển đổi, tự động đăng ký các lệnh console, kết hợp nhiều queued job trong một lần thực hiện, giới hạn queued job, thời gian job được chạy, tạo mailable, tạo và báo cáo các exception, xử lý exception sẽ phù hợp hơn, cải thiện việc test cơ sở dữ liệu, dễ dàng chỉnh sửa validation rule hơn, cài đặt được React, các phương thức `Route::view` và `Route::redirect`, "locks" cho các driver bộ nhớ cache như Memcached và Redis, thông báo, hỗ trợ Chrome trong Dusk, các shortcut trong Blade sẽ tiện lợi, hỗ trợ trust proxy, và nhiều hơn thế nữa.

Ngoài ra, Laravel 5.5 cũng trùng với việc phát hành [Laravel Horizon](https://horizon.laravel.com), một bảng điều khiển queue và cấu hình hệ thống mới và đẹp cho các queue của Laravel mà được dựa trên Redis của bạn.

> {tip} Tài liệu này sẽ chỉ tóm tắt những cải tiến cần chú ý nhất đối với framework; nhưng, nếu bạn cần xem các thay đổi một cách kỹ lưỡng hơn thì bạn có thể xem [trên GitHub](https://github.com/laravel/framework/blob/5.5/CHANGELOG-5.5.md).

### Laravel Horizon

Horizon cung cấp một bảng điều khiển đẹp mắt và cấu hình code-driven cho queue Redis được hỗ trợ bởi Laravel của bạn. Horizon cho phép bạn dễ dàng theo dõi các số liệu chính của hệ thống queue của bạn như job được thông qua, thời gian chạy và job bị thất bại.

Tất cả các cấu hình worker của bạn được lưu trữ trong một file cấu hình đơn giản, cho phép cấu hình của bạn ở trong source control nơi mà toàn bộ nhóm của bạn có thể làm việc cùng nhau.

Để biết thêm thông tin về Horizon, hãy xem [tài liệu đầy đủ của Horizon](/docs/{{version}}/horizon)

### Package Discovery

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/5) miễn phí cho tính năng này ở trên Laracasts.

Trong các phiên bản trước của Laravel, việc cài đặt package thường yêu cầu một số bước bổ sung như thêm service provider vào file cấu hình `app` của bạn và đăng ký các facade liên quan. Tuy nhiên, với Laravel 5.5, Laravel có thể tự động phát hiện và tự đăng ký service provider và các facade cho bạn.

Ví dụ, bạn có thể trải nghiệm điều này bằng cách cài đặt một package rất phổ biến là `barryvdh/laravel-debugbar` vào application Laravel của bạn. Khi package đã được cài đặt qua Composer, debug bar sẽ tự động thêm vào cho application của bạn mà không cần cấu hình thêm bất cứ thứ gì khác:

    composer require barryvdh/laravel-debugbar

Các nhà phát triển package cũng chỉ cần thêm các service provider và các facade của họ vào file `composer.json` trong package của họ:

    "extra": {
        "laravel": {
            "providers": [
                "Laravel\\Tinker\\TinkerServiceProvider"
            ]
        }
    },

Để biết thêm thông tin về việc tự thêm service provider và facade cho các package mà bạn đã phát triển, hãy xem tài liệu đầy đủ về [phát triển package](/docs/{{version}}/packages).

### API Resources

Khi xây dựng API, bạn có thể cần một lớp chuyển đổi nằm giữa các model Eloquent của bạn và các response JSON được trả về cho người dùng application. Các class resource của Laravel cho phép bạn chuyển đổi một cách rõ ràng và dễ hiểu các model và các collection model của bạn thành JSON.

Một class resource đại diện cho một model cần được chuyển đổi thành cấu trúc JSON. Ví dụ, đây là một class `UserResource` đơn giản:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Tất nhiên, đây chỉ là ví dụ cơ bản nhất về API resource. Laravel cũng cung cấp nhiều phương thức để giúp bạn khi xây dựng các resources và resource collection của bạn. Để biết thêm thông tin, hãy xem [tài liệu đầy đủ](/docs/{{version}}/eloquent-resources) trên API resource.

### Console Command Auto-Registration

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/12) miễn phí cho tính năng này ở trên Laracasts.

Khi tạo các lệnh console mới, bạn sẽ không còn phải bắt buộc liệt kê chúng theo cách thủ công trong thuộc tính `$commands` trong Console kernel của bạn. Thay vào đó, một phương thức `load` mới sẽ được gọi từ phương thức `commands` từ kernel của bạn, nó sẽ quét một thư mục nhất định và sẽ tự động đăng ký bất kỳ lệnh console nào có trong thư mục đó:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');

        // ...
    }

### New Frontend Presets

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/4) miễn phí cho tính năng này ở trên Laracasts.

Mặc dù Vue scaffolding vẫn được có sẵn trong Laravel 5.5, nhưng cũng có thêm một số tùy chọn cài đặt frontend mới mà bạn có thể sử dụng. Trong một application Laravel mới, bạn có thể thay đổi Vue scaffolding thành React scaffolding bằng lệnh `preset`:

    php artisan preset react

Hoặc, bạn có thể xoá hoàn toàn JavaScript và CSS framework scaffolding bằng cách sử dụng cài đặt `none`. Cài đặt này sẽ để lại application của bạn, một file Sass và một vài tiện ích JavaScript đơn giản:

    php artisan preset none

> {note} Các lệnh này chỉ nên chạy trên các bản cài đặt Laravel mới. Chúng không nên được sử dụng trên các application đã tồn tại.

### Queued Job Chaining

Kết hợp job cho phép bạn khai báo một danh sách các queued job nên được chạy theo trình tự. Nếu một job trong danh sách bị thất bại, thì các job còn lại sẽ không được chạy. Để thực thi danh sách queued job, bạn có thể sử dụng phương thức `withChain` trên bất kỳ dispatchable job nào của bạn:

    ProvisionServer::withChain([
        new InstallNginx,
        new InstallPhp
    ])->dispatch();

### Queued Job Rate Limiting

Nếu application của bạn tương tác với Redis, bạn có thể điều tiết các queued job theo thời gian hoặc đồng thời. Tính năng này có thể hỗ trợ khi các queued job của bạn đang tương tác với các API mà cũng bị giới hạn về tỷ lệ chạy. Ví dụ, bạn có thể điều tiết một loại job nhất định chỉ chạy 10 lần trong 60 giây.

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

> {tip} Trong ví dụ trên, `key` có thể là bất kỳ chuỗi nào xác định một loại job mà bạn muốn giới hạn tỷ lệ chạy. Ví dụ, bạn có thể muốn khởi tạo một key dựa trên tên class của job và ID của các model Eloquent mà nó hoạt động.

Ngoài ra, bạn có thể khai báo số lượng worker tối đa có thể xử lý đồng thời một job nhất định. Điều này có thể hữu ích khi một queued job đang sửa một resource chỉ được sửa bởi một job tại một thời điểm. Ví dụ, bạn có thể giới hạn các job thuộc loại đã cho chỉ được xử lý bởi một worker tại một thời điểm:

    Redis::funnel('key')->limit(1)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

### Time Based Job Attempts

Thay thế cho việc định nghĩa số lần một job có thể được thử trước khi nó thất bại, bạn có thể định nghĩa thời gian mà job đó sẽ hết thời gian. Điều này cho phép một job được thử thoải mái trong một khoảng thời gian nhất định. Để định nghĩa thời gian mà một job sẽ hết thời gian, hãy thêm phương thức `retryUntil` vào class job của bạn:

    /**
     * Determine the time at which the job should timeout.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} Bạn cũng có thể định nghĩa phương thức `retryUntil` trên các queued event listener của bạn.

### Validation Rule Objects

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/7) miễn phí cho tính năng này ở trên Laracasts.

Các đối tượng validation rule cung cấp một cách mới, nhỏ gọn để thêm các validation rule tùy biến vào application của bạn. Trong các phiên bản trước của Laravel, phương thức `Validator::extend` đã được sử dụng để thêm các validation rule tùy biến thông qua Closures. Tuy nhiên, điều này có thể trở lên rất cồng kềnh. Trong Laravel 5.5, lệnh Artisan `make:rule` mới sẽ tạo ra một file validation rule mới trong thư mục `app/Rules`:

    php artisan make:rule ValidName

Một đối tượng validation rule chỉ có hai phương thức: `passes` và `message`. Phương thức `passes` nhận vào giá trị và tên của thuộc tính và sẽ trả về `true` hoặc `false` tùy thuộc vào giá trị thuộc tính có hợp lệ hay không. Phương thức `message` sẽ trả về thông báo lỗi của validation, nó sẽ được sử dụng khi validation thất bại:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class ValidName implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strlen($value) === 6;
        }

        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The name must be six characters long.';
        }
    }

Khi rule đã được định nghĩa, bạn có thể sử dụng rule đó bằng cách truyền một instance của đối tượng rule đó với các rule validation khác của bạn:

    use App\Rules\ValidName;

    $request->validate([
        'name' => ['required', new ValidName],
    ]);

### Trusted Proxy Integration

Khi application của bạn đang chạy sau một hệ thống load balancer, mà không dùng chứng chỉ TLS / SSL để connect đến server của bạn. Đôi khi bạn sẽ cảm thấy rằng application của bạn không trả về liên kết HTTPS. Thông thường, điều này là do application của bạn đang bị chuyển tiếp lưu lượng truy cập từ load balancer của bạn vào cổng 80 và không biết rằng nó đang tạo ra các liên kết không an toàn.

Để giải quyết vấn đề này, nhiều người dùng Laravel đã cài đặt package [Trusted Proxies](https://github.com/fideloper/TrustedProxy) của Chris Fidao. Vì đây là trường hợp phổ biến, nên package của Chris cũng đã được sử dụng mặc định cùng với Laravel 5.5.

Một middleware mới `App\Http\Middleware\TrustProxies` đã có mặc định trong application Laravel 5.5. Middleware này cho phép bạn tùy chỉnh các proxy sẽ được application của bạn trust:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var array
         */
        protected $proxies;

        /**
         * The current proxy header mappings.
         *
         * @var array
         */
        protected $headers = [
            Request::HEADER_FORWARDED => 'FORWARDED',
            Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
            Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
            Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
            Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        ];
    }

### On-Demand Notifications

Thỉnh thoảng bạn có thể cần gửi thông báo cho người mà thông tin người đó chưa được lưu vào trong bảng cơ sở dữ liệu. Sử dụng phương thức `Notification::route`, bạn có thể chỉ định thông tin ad-hoc notification routing trước khi gửi thông báo:

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->send(new InvoicePaid($invoice));

### Renderable Mailables

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/6) miễn phí cho tính năng này ở trên Laracasts.

Mailable có thể được trả lại trực tiếp từ route, cho phép bạn xem xét các thiết kế mailable của bạn trên trình duyệt:

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

### Renderable & Reportable Exceptions

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/18) miễn phí cho tính năng này ở trên Laracasts.

Trong các phiên bản trước của Laravel, bạn có thể bạn đã phải dùng đến "kiểm tra kiểu" trong xử lý ngoại lệ của bạn để hiển thị các response tùy biến cho một ngoại lệ nhất định. Chẳng hạn, bạn có thể đã viết code như thế này trong phương thức `render` của xử lý ngoại lệ của bạn:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof SpecialException) {
            return response(...);
        }

        return parent::render($request, $exception);
    }

Trong Laravel 5.5, bây giờ bạn có thể định nghĩa một phương thức `render` trực tiếp trên các ngoại lệ của bạn. Điều này cho phép bạn set các logic tạo response tùy biến trực tiếp lên trên ngoại lệ, giúp tránh cồng kềnh các logic có nhiều điều kiện trong trình xử lý ngoại lệ của bạn. Nếu bạn cũng muốn tùy biến logic report cho ngoại lệ, bạn có thể định nghĩa một phương thức `report` trong class này:

    <?php

    namespace App\Exceptions;

    use Exception;

    class SpecialException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * Report the exception.
         *
         * @param  \Illuminate\Http\Request
         * @return void
         */
        public function render($request)
        {
            return response(...);
        }
    }

### Request Validation

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/2) miễn phí cho tính năng này ở trên Laracasts.

Hiện tại, đối tượng `Illuminate\Http\Request` cung cấp phương thức `validate`, cho phép bạn validate một request đến từ một route Closure hoặc một controller:

    use Illuminate\Http\Request;

    Route::get('/comment', function (Request $request) {
        $request->validate([
            'title' => 'required|string',
            'body' => 'required|string',
        ]);

        // ...
    });

### Consistent Exception Handling

Xử lý ngoại lệ validation hiện đã nhất quán trong toàn bộ framework. Trước đây, có nhiều vị trí trong framework yêu cầu phải được tùy chỉnh để thay đổi format mặc định cho các phản hồi lỗi JSON validation. Ngoài ra, hiện tại, các format mặc định cho các phản hồi JSON validation trong Laravel 5.5 cũng đã tuân thủ quy ước sau:

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }

Tất cả định dạng lỗi JSON validation có thể được kiểm soát bằng cách định nghĩa một phương thức trên class `App\Exceptions\Handler` của bạn. Ví dụ, tùy chỉnh sau đây sẽ định dạng các phản hồi JSON validation sử dụng theo quy ước của bản Laravel 5.4.

    use Illuminate\Validation\ValidationException;

    /**
     * Convert a validation exception into a JSON response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }

### Cache Locks

Các driver bộ nhớ cache như Redis và Memcached hiện có hỗ trợ để lấy và giải phóng các "locks". Điều này cung cấp một phương thức đơn giản để có được các locks tùy ý mà không phải lo lắng khi nhiều threads cùng truy cập và cùng lúc muốn thay đổi dữ liệu. Ví dụ: trước khi thực hiện một tác vụ, bạn có thể muốn có được một lock để không có bất kỳ một process nào khác có thể thực hiện cùng một tác vụ đang diễn ra:

    if (Cache::lock('lock-name', 60)->get()) {
        // Lock obtained for 60 seconds, continue processing...

        Cache::lock('lock-name')->release();
    } else {
        // Lock was not able to be obtained...
    }

Hoặc, bạn có thể truyền vào một Closure cho phương thức `get`. Closure sẽ chỉ được thực hiện nếu có thể lấy được lock và lock sẽ tự động được giải phóng sau khi thực hiện xong Closure:

    Cache::lock('lock-name', 60)->get(function () {
        // Lock obtained for 60 seconds...
    });

Ngoài ra, bạn có thể "chặn" cho đến khi lock đó sẵn sàng trở lại:

    if (Cache::lock('lock-name', 60)->block(10)) {
        // Wait for a maximum of 10 seconds for the lock to become available...
    }

### Blade Improvements

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/10) miễn phí cho tính năng này ở trên Laracasts.

Lập trình một lệnh tùy biến đôi khi phức tạp hơn là khi định nghĩa các câu lệnh điều kiện tùy biến đơn giản. Vì lý do đó, Blade hiện cung cấp một phương thức `Blade::if` cho phép bạn định nghĩa các câu lệnh điều kiện tùy biến bằng cách sử dụng Closures. Ví dụ: hãy định nghĩa một câu lệnh điều kiện tùy biến kiểm tra môi trường của application hiện tại. Chúng ta có thể làm điều này trong phương thức `boot` của `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

Khi câu lệnh điều kiện tùy biến này đã được định nghĩa xong, chúng ta có thể dễ dàng sử dụng nó trên các template của bạn:

    @env('local')
        // The application is in the local environment...
    @else
        // The application is not in the local environment...
    @endenv

Ngoài khả năng dễ dàng định nghĩa các câu lệnh điều kiện Blade tùy biến, các shortcut mới cũng đã được thêm vào để nhanh chóng kiểm tra trạng thái xác thực của người dùng hiện tại:

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

### New Routing Methods

> {video} Có một [video hướng dẫn](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/16) miễn phí cho tính năng này ở trên Laracasts.

Nếu bạn đang định nghĩa một route để chuyển hướng đến một URI khác, bây giờ bạn có thể sử dụng phương thức `Route::redirect`. Phương thức này cung cấp một shortcut thuận tiện để bạn không phải định nghĩa lại một route hoặc một controller mới để thực hiện chuyển hướng:

    Route::redirect('/here', '/there', 301);

Nếu route của bạn chỉ cần trả về một view, thì bây giờ bạn có thể sử dụng phương thức `Route::view`. Giống như phương thức `redirect`, phương thức này cung cấp một shortcut đơn giản để bạn không phải xác định một route hoặc một controller. Phương thức `view` chấp nhận URI làm tham số đầu tiên và tên view làm tham số thứ hai. Ngoài ra, bạn có thể cung cấp một mảng dữ liệu để truyền đến view dưới dạng tham số thứ ba tùy chọn:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

### "Sticky" Database Connections

#### The `sticky` Option

Khi cấu hình đọc / ghi cho các kết nối cơ sở dữ liệu, một tùy chọn cấu hình `stick` mới cũng có thể được sử dụng:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

Tùy chọn `stick` là một giá trị *tùy chọn* có thể được sử dụng để cho phép đọc ngay các bản ghi đã được ghi vào cơ sở dữ liệu trong request hiện tại. Nếu tùy chọn `stick` được bật và các thao tác "ghi" đã được thực hiện đối với cơ sở dữ liệu trong request hiện tại, thì mọi thao tác "đọc" tiếp theo đó sẽ sử dụng kết nối của thao tác "ghi" vừa được gọi. Điều này đảm bảo rằng mọi dữ liệu được ghi trong cùng một request có thể được đọc lại ngay lập tức từ cơ sở dữ liệu trong cùng một request đó. Tùy thuộc vào application của bạn, mà bạn có thể quyết định xem đây có phải là một hành động mong muốn cho application của bạn hay không.
