# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 7](#laravel-7)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Laravel và các package khác của nó tuân theo [Phiên bản Semantic](https://semver.org). Các phiên bản được phát hành chính thức của framework được phát hành sáu tháng một lần (khoảng tháng 2 và khoảng tháng 8), trong khi các bản phát hành nhỏ hơn và các bản sửa lỗi có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ và các bản sửa lỗi sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `^7.0`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với các bản phát hành hỗ trợ dài hạn, chẳng hạn như Laravel 6, các bản sửa lỗi được cung cấp trong 2 năm và các bản sửa lỗi bảo mật được cung cấp trong 3 năm. Những bản phát hành này cung cấp các hỗ trợ và bảo trì dài nhất. Đối với các bản phát hành bình thường, các bản sửa lỗi được cung cấp trong 6 tháng và các bản sửa lỗi bảo mật được cung cấp trong 1 năm. Đối với tất cả các thư viện, bao gồm cả Lumen, chỉ bản phát hành mới nhất mới nhận được các bản sửa lỗi. Ngoài ra, hãy xem các phiên bản cơ sở dữ liệu [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

| Version | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- |
| 6 (LTS) | ngày 3 tháng 9 năm 2019 | ngày 3 tháng 9 năm 2021 | ngày 3 tháng 9 năm 2022 |
| 7 | March 3rd, 2020 | September 10th, 2020 | March 3rd, 2021 |
| 8 | September 8th, 2020 | March 8th, 2021 | September 8th, 2021 |

<a name="laravel-7"></a>
## Laravel 7

Laravel 7 tiếp tục những cải tiến được thực hiện trong Laravel 6.x bằng cách giới thiệu thêm Laravel Sanctum, cải tiến tốc độ routing, tùy chỉnh Eloquent cast, Blade component tag, xử lý chuỗi, HTTP client sẽ tập trung vào nhà phát triển, hỗ trợ CORS của bên thứ nhất, cải thiện scoping cho các route model binding, tùy chỉnh stub, cải tiến database queue, multiple mail driver, truy vấn cast thời gian, lệnh `artisan test` mới, và một loạt các bản sửa lỗi và cải tiến khả năng sử dụng khác.

### Laravel Sanctum

_Laravel Sanctum được xây dựng bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel Sanctum cung cấp một hệ thống xác thực nhẹ cho các SPAs (các ứng dụng single page), ứng dụng di động và các API đơn giản dựa trên token. Sanctum cho phép mỗi người dùng ứng dụng của bạn tạo ra nhiều API token cho tài khoản của họ. Các token này có thể được cấp các quyền / phạm vi cụ thể cho các hành động mà token được phép thực hiện.

Để biết thêm thông tin về Laravel Sanctum, hãy tham khảo [tài liệu về Sanctum](/docs/{{version}}/sanctum).

### Custom Eloquent Casts

_Tuỳ chỉnh Eloquent cast được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel có nhiều kiểu cast có sẵn và hữu ích; tuy nhiên, đôi khi bạn có thể cần phải định nghĩa thêm kiểu cast của riêng bạn. Giờ đây, bạn có thể thực hiện điều này bằng cách định nghĩa thêm một class implement interface `CastsAttributes`.

Các class implement interface này phải định nghĩa hai phương thức `get` và `set`. Phương thức `get` chịu trách nhiệm chuyển đổi một giá trị từ cơ sở dữ liệu thành một giá trị theo kiểu cast, trong khi phương thức `set` sẽ biến đổi một giá trị theo kiểu cast thành một giá trị có thể được lưu trữ được vào trong cơ sở dữ liệu. Ví dụ: chúng ta sẽ implement lại kiểu cast `json` có sẵn trong laravel dưới dạng một kiểu cast tùy chỉnh:

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Json implements CastsAttributes
    {
        /**
         * Cast the given value.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return array
         */
        public function get($model, $key, $value, $attributes)
        {
            return json_decode($value, true);
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return json_encode($value);
        }
    }

Khi bạn đã định nghĩa xpng một kiểu cast tùy chỉnh, bạn có thể gắn nó vào một thuộc tính của model bằng cách sử dụng tên class của nó:

    <?php

    namespace App;

    use App\Casts\Json;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'options' => Json::class,
        ];
    }

Để tìm hiểu thêm về cách viết các Eloquent cast tùy chỉnh, bao gồm cả các cast tùy chỉnh cho các giá trị đối tượng, vui lòng tham khảo [tài liệu về Eloquent](/docs/{{version}}/eloquent-mutators#custom-casts).

### Blade Component Tags & Improvements

_Blade component tag được đóng góp bởi [Spatie](https://spatie.be/), [Marcel Pociot](https://twitter.com/marcelpociot), [Caleb Porzio](https://twitter.com/calebporzio), [Dries Vints](https://twitter.com/driesvints), và [Taylor Otwell](https://github.com/taylorotwell)_.

> {tip} Các blade component đã được làm lại để cho phép việc hiển thị dựa trên tag, quản lý thuộc tính, các component class, các component inline view và hơn thế nữa. Vì việc làm lại các blade component rất rộng, nên vui lòng tham khảo [tài liệu đầy đủ về component blade](/docs/{{version}}/blade#components) để tìm hiểu về các tính năng này.

Tóm lại, một component bây giờ có thể có một class được liên kết với một loạt dữ liệu chỉ định mà nó chấp nhận. Tất cả các thuộc tính và phương thức công khai được định nghĩa trên class component sẽ tự động được cung cấp cho view component. Bất kỳ thuộc tính HTML thêm mới nào được chỉ định trong component đều có thể được quản lý bằng biến `$attributes` có sẵn trong component, là một instance chứa nhiều thuộc tính.

Trong ví dụ này, chúng ta sẽ giả định rằng component `App\View\Components\Alert` đã được định nghĩa như sau:

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * The alert type.
         *
         * @var string
         */
        public $type;

        /**
         * Create the component instance.
         *
         * @param  string  $type
         * @return void
         */
        public function __construct($type)
        {
            $this->type = $type;
        }

        /**
         * Get the class for the given alert type.
         *
         * @return string
         */
        public function classForType()
        {
            return $this->type == 'danger' ? 'alert-danger' : 'alert-warning';
        }

        /**
         * Get the view / contents that represent the component.
         *
         * @return \Illuminate\View\View|string
         */
        public function render()
        {
            return view('components.alert');
        }
    }

Và, giả sử template blade của component đã được định nghĩa như sau:

    <!-- /resources/views/components/alert.blade.php -->

    <div class="alert {{ $classForType }}" {{ $attributes }}>
        {{ $heading }}

        {{ $slot }}
    </div>

Component có thể được tạo trong view blade khác bằng cách sử dụng tag của component:

    <x-alert type="error" class="mb-4">
        <x-slot name="heading">
            Alert content...
        </x-slot>

        Default slot content...
    </x-alert>

Như đã đề cập, đây chỉ là một ví dụ rất nhỏ về chức năng của component blade trong Laravel 7 và không thể hiện các component ẩn, component inline view và nhiều tính năng khác. Vui lòng tham khảo thêm [tài liệu đầy đủ về component blade](/docs/{{version}}/blade#components) để tìm hiểu thêm về các tính năng này.

> {note} Cú pháp `@component` trước đó của component blade sẽ chưa và sẽ không bị xóa.

### HTTP Client

_HTTP client là một wrapper của thư viện Guzzle và được đóng góp bởi [Adam Wathan](https://twitter.com/adamwathan), [Jason McCreary](https://twitter.com/gonedark), và [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel cung cấp một API nhỏ, rõ ràng dựa trên thư viện [Guzzle HTTP client](http://docs.guzzlephp.org/en/stable/), cho phép bạn nhanh chóng thực hiện các HTTP request giao tiếp với các ứng dụng web khác. API này tập trung vào các trường hợp sử dụng phổ biến và giúp tăng trải nghiệm tuyệt vời dành cho nhà phát triển. Ví dụ: client có thể dễ dàng tạo ra một `POST` request và thêm dữ liệu JSON:

    use Illuminate\Support\Facades\Http;

    $response = Http::withHeaders([
        'X-First' => 'foo',
        'X-Second' => 'bar'
    ])->post('http://test.com/users', [
        'name' => 'Taylor',
    ]);

    return $response['id'];

Ngoài ra, HTTP client cũng cung cấp các chức năng testing tuyệt vời, tiện dụng:

    Http::fake([
        // Stub a JSON response for GitHub endpoints...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Stub a string response for Google endpoints...
        'google.com/*' => Http::response('Hello World', 200, ['Headers']),

        // Stub a series of responses for Facebook endpoints...
        'facebook.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->pushStatus(404),
    ]);

Để tìm hiểu thêm về tất cả các tính năng của HTTP client, vui lòng tham khảo thêm [tài liệu về HTTP client](/docs/{{version}}/http-client).

### Fluent String Operations

_Xử lý chuỗi được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Có thể bạn đã quen thuộc với class `Illuminate\Support\Str` hiện có của Laravel, class này cung cấp nhiều chức năng xử lý chuỗi hữu ích. Laravel 7 bây giờ cung cấp một thư viện xử lý chuỗi hoàn thiện hơn, hướng đối tượng hơn được xây dựng dựa trên các hàm này. Bạn có thể tạo ra một đối tượng `Illuminate\Support\Stringable` bằng cách sử dụng phương thức `Str::of`. Sau đó, nhiều phương thức có thể được kết hợp vào trong đối tượng để xử lý chuỗi:

    return (string) Str::of('  Laravel Framework 6.x ')
                        ->trim()
                        ->replace('6.x', '7.x')
                        ->slug();

Để biết thêm thông tin về các phương thức có sẵn để xử lý chuỗi, vui lòng tham khảo [tài liệu đầy đủ](/docs/{{version}}/helpers#fluent-strings) của nó.

### Route Model Binding Improvements

_Route model binding đã được cải tiến và được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

#### Key Customization

Thỉnh thoảng bạn có thể muốn resolve các model Eloquent ra bằng cách sử dụng một cột khác, khác với cột `id`. Để làm như thế, Laravel 7 cho phép bạn chỉ định cột trong định nghĩa tham số route:

    Route::get('api/posts/{post:slug}', function (App\Post $post) {
        return $post;
    });

#### Automatic Scoping

Thỉnh thoảng, khi liên kết ngầm nhiều model Eloquent trong một định nghĩa route, bạn có thể muốn scope model Eloquent thứ hai sao cho nó phải là con của model Eloquent thứ nhất. Ví dụ: hãy xem tình huống sau lấy ra một bài đăng trong blog bằng slug cho một user cụ thể:

    use App\Post;
    use App\User;

    Route::get('api/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

Khi sử dụng liên kết ngầm có key tùy biến làm một tham số route lồng nhau, Laravel 7 sẽ tự động scope truy vấn để lấy ra các model lồng nhau thông qua cha của nó bằng cách sử dụng các quy ước để đặt tên quan hệ trên cha. Trong trường hợp này, sẽ giả định rằng model `User` có một quan hệ có tên là `posts` (số nhiều của tên tham số route) có thể được sử dụng để lấy ra model `Post`.

Để biết thêm thông tin về route model binding, vui lòng tham khảo [tài liệu định route](/docs/{{version}}/routing#route-model-binding).

### Multiple Mail Drivers

_Multiple mail driver đã được hỗ trợ và được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel 7 allows the confi
Laravel 7 cho phép cấu hình nhiều "mailers" trong một ứng dụng. Mỗi mailers được cấu hình trong file cấu hình `mail` có thể có thêm các tùy chọn riêng nó hoặc thậm chí là một "transport" của riêng nó, cho phép ứng dụng của bạn sử dụng các dịch vụ email khác nhau để gửi một số email nhất định. Ví dụ: ứng dụng của bạn có thể sử dụng Postmark để gửi mail giao dịch trong khi sử dụng Amazon SES để gửi các mail hàng loạt.

Mặc định, Laravel sẽ sử dụng mailer được cấu hình làm mailer `default` trong file cấu hình` mail` của bạn. Tuy nhiên, bạn có thể sử dụng phương thức `mailer` để gửi một message với một cấu hình mailer cụ thể:

    Mail::mailer('postmark')
            ->to($request->user())
            ->send(new OrderShipped($order));

### Route Caching Speed Improvements

_Route caching đã được cải thiện về tốc độ và được đóng góp bởi cộng đồng [Symfony](https://symfony.com) và [Dries Vints](https://twitter.com/driesvints)_.

Laravel 7 có chứa một phương thức mới để tìm các route đã được biên dịch, được lưu trong bộ nhớ cache bằng cách sử dụng lệnh Artisan `route:cache`. Trên các ứng dụng lớn (ví dụ: ứng dụng có 800 route trở lên), những cải thiện này có thể dẫn đến cải thiện về tốc độ nhanh hơn **2 lần** trong các request trên mỗi giây với một ví dụ "Hello World" đơn giản. Và không cần thay đổi ứng dụng của bạn.

### CORS Support

_Hỗ trợ CORS được đóng góp bởi [Barry vd. Heuvel](https://twitter.com/barryvdh)_.

Laravel 7 có hỗ trợ cấu hình phản hồi request Cross-Origin Resource Sharing (CORS) `OPTIONS` bằng cách tích hợp package Laravel CORS được viết bởi Barry vd. Heuvel. Một cấu hình `cors` mới đã được thêm vào trong [framework laravel mặc định](https://github.com/laravel/laravel/blob/develop/config/cors.php).

Để biết thêm thông tin về hỗ trợ CORS trong Laravel 7.x, vui lòng tham khảo thêm [tài liệu CORS](/docs/{{version}}/routing#cors).

### Query Time Casts

_Query time casting được đóng góp bởi [Matt Barlow](https://github.com/mpbarlow)_.

Thỉnh thoảng bạn có thể cần phải áp dụng các cast trong khi thực hiện một query, chẳng hạn như khi chọn một giá trị thô từ một bảng. Ví dụ: hãy xem xét query sau:

    use App\Post;
    use App\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

Thuộc tính `last_posted_at` trên kết quả của query này sẽ là một chuỗi thô. Sẽ rất tiện lợi nếu chúng ta có thể áp dụng một cast `date` cho thuộc tính này khi thực hiện query. Để thực hiện điều này, chúng ta có thể sử dụng phương thức `withCasts` được cung cấp bởi Laravel 7:

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'date'
    ])->get();

### MySQL 8+ Database Queue Improvements

_MySQL database queue được cải tiến và được đóng góp bởi [Mohamed Said](https://github.com/themsaid)_.

Trong các bản phát hành trước của Laravel, queue `database` không được coi là đủ mạnh để sử dụng trong production, do deadlock. Tuy nhiên, Laravel 7 đã cung cấp các cải tiến cho các ứng dụng sử dụng MySQL 8+ dưới dạng queue được hỗ trợ cơ sở dữ liệu. Bằng cách sử dụng mệnh đề `FOR UPDATE SKIP LOCKED` và các cải tiến SQL khác, driver `database` bây giờ có thể được sử dụng một cách an toàn trong các ứng dụng production lớn.

### Artisan `test` Command

_Lệnh `test` được đóng góp bởi [Nuno Maduro](https://twitter.com/enunomaduro)_.

Ngoài lệnh `phpunit`, bây giờ bạn có thể sử dụng lệnh Artisan `test` để chạy các bài test của bạn. Trình chạy test của Artisan cung cấp một giao diện console đẹp mắt và nhiều thông tin hơn về các bài test đang được thực hiện. Ngoài ra, trình chạy này sẽ tự động dừng lại ở lần kiểm thử đầu tiên mà bị thất bại:

    php artisan test

<p align="center">
<img src="https://laravel.com/img/docs/7x-release-notes-artisan-test-preview.png">
</p>

Bất kỳ tham số nào mà có thể được truyền vào cho lệnh `phpunit` thì cũng có thể được truyền vào cho lệnh Artisan `test`:

    php artisan test --group=feature

### Markdown Mail Template Improvements

_Template markdown mail đã được cải tiến và được đóng góp bởi [Taylor Otwell](https://twitter.com/taylorotwell)_.

Template markdown mail mặc định đã nhận được một thiết kế mới, hiện đại hơn dựa trên bảng màu của Tailwind CSS. Tất nhiên, template này có thể được export và tùy chỉnh theo nhu cầu ứng dụng của bạn:

<p align="center">
<img src="https://laravel.com/img/docs/7x-release-notes-notification-preview.png">
</p>

Để biết thêm thông tin về markdown mail, vui lòng tham khảo [tài liệu về mail](/docs/{{version}}/mail#markdown-mailables).

### Stub Customization

_Tuỳ chỉnh stub được đóng góp bởi [Taylor Otwell](https://twitter.com/taylorotwell)_.

Lệnh `make` của Artisan console sẽ được sử dụng để tạo nhiều class khác nhau, chẳng hạn như controller, job, migration và các bài test. Các class này được tạo ra bằng cách sử dụng các file "stub" được điền sẵn các giá trị dựa trên đầu vào mà bạn đưa vào. Tuy nhiên, thỉnh thoảng bạn có thể muốn thực hiện các thay đổi nhỏ đối với các file do Artisan tạo ra. Để thực hiện điều này, Laravel 7 cung cấp một lệnh `stub:publish` để export ra các stub cơ bản nhất để tùy chỉnh:

    php artisan stub:publish

Các file stub đã được export sẽ nằm trong thư mục `stubs` trong thư mục gốc của ứng dụng của bạn. Bất kỳ thay đổi nào mà bạn thực hiện đối với các file stub này sẽ được phản ánh khi bạn tạo các class tương ứng khi sử dụng lệnh Artisan `make`.

### Queue `maxExceptions` Configuration

_Thuộc tính `maxExceptions` được đóng góp bởi [Mohamed Said](https://twitter.com/themsaid)_.

Thỉnh thoảng bạn có thể muốn chỉ định một job có thể được thử lại nhiều lần, nhưng sẽ thất bại nếu trong các lần thử lại được kích hoạt bởi một số lượng exception nhất định. Trong Laravel 7, bạn có thể định nghĩa một thuộc tính `maxExceptions` trên class job của bạn:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of times the job may be attempted.
         *
         * @var int
         */
        public $tries = 25;

        /**
         * The maximum number of exceptions to allow before failing.
         *
         * @var int
         */
        public $maxExceptions = 3;

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            Redis::throttle('key')->allow(10)->every(60)->then(function () {
                // Lock obtained, process the podcast...
            }, function () {
                // Unable to obtain lock...
                return $this->release(10);
            });
        }
    }

Trong ví dụ này, job sẽ được giải phóng trong 10 giây nếu ứng dụng không thể lấy được Redis lock và sẽ tiếp tục được thử lại tối đa 25 lần. Tuy nhiên, job sẽ thất bại nếu job đưa ra quá ba exception.
