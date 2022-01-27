# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 5.6](#laravel-5.6)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Cấu trúc của phiên bản của Laravel được duy trì theo quy ước như sau: `paradigm.major.minor`. Các phiên bản được phát hành chính thức của framework được phát hành sáu tháng một lần (tháng 2 và tháng 8), trong khi các bản phát hành nhỏ hơn có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `5.5.*`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

Các bản phát hành thay đổi Paradigm được phân tách qua nhiều năm và đại diện cho những thay đổi căn bản trong kiến trúc và quy ước của framework. Hiện tại, chưa có bản phát hành nào mà cần phải thay đổi Paradigm được phát triển trong hiện tại.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với các bản phát hành hỗ trợ dài hạn, chẳng hạn như Laravel 5.5, các bản sửa lỗi được cung cấp trong 2 năm và các bản sửa lỗi bảo mật được cung cấp trong 3 năm. Những bản phát hành này cung cấp các hỗ trợ và bảo trì dài nhất. Đối với các bản phát hành bình thường, các bản sửa lỗi được cung cấp trong 6 tháng và các bản sửa lỗi bảo mật được cung cấp trong 1 năm.

| Version | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- |
| 5.0 | ngày 4 tháng 2 năm 2015 | ngày 4 tháng 8 năm 2015 | ngày 4 tháng 2 năm 2016 |
| 5.1 (LTS) | ngày 9 tháng 6 năm 2015 | ngày 9 tháng 6 năm 2017 | ngày 9 tháng 6 năm 2018 |
| 5.2 | ngày 21 tháng 12 năm 2015 | ngày 6 tháng 12 năm 2016 | ngày 21 tháng 12 năm 2016 |
| 5.3 | ngày 23 tháng 8 năm 2016 | ngày 23 tháng 2 năm 2016 | ngày 23 tháng 8 năm 2017 |
| 5.4 | ngày 24 tháng 1 năm 2017 | ngày 24 tháng 7 năm 2017| ngày 24 tháng 1 năm 2018 |
| 5.5 (LTS) | ngày 30 tháng 8 năm 2017 | ngày 30 tháng 8 năm 2019 | ngày 30 tháng 8 năm 2020 |
| 5.6 | ngày 7 tháng 2 năm 2018 | ngày 7 tháng 8 năm 2018 | ngày 7 tháng 2 năm 2019 |

<a name="laravel-5.6"></a>
## Laravel 5.6

Laravel 5.6 tiếp tục những cải tiến đã được thực hiện trong Laravel 5.5 bằng cách nâng cấp hệ thống ghi log, task schedule trên một server, cải tiến chuyển hoá model, giới hạn số lượt truy cập, các class broadcast channel, API resource controller, cải tiến định dạng ngày tháng của Eloquent, bí danh Blade component, hỗ trợ hàm hash Argon2, thêm package Collision và hơn thế nữa. Ngoài ra, tất cả các front-end đã được nâng cấp lên Bootstrap 4.

Tất cả các component của Symfony mà được Laravel sử dụng đều đã được nâng cấp lên phiên bản Symfony `~4.0`.

Việc phát hành Laravel 5.6 trùng với việc phát hành [Spark 6.0](https://spark.laravel.com), đây là bản nâng cấp lớn đầu tiên của Laravel Spark kể từ khi phát hành. Spark 6.0 giới thiệu giá per-seat cho Stripe và Braintree, localization, Bootstrap 4, giao diện người dùng nâng cao và hỗ trợ Stripe Elements.

> {tip} Tài liệu này sẽ chỉ tóm tắt những cải tiến cần chú ý nhất đối với framework; nhưng, nếu bạn cần xem các thay đổi một cách kỹ lưỡng hơn thì bạn có thể xem [trên GitHub](https://github.com/laravel/framework/blob/5.6/CHANGELOG-5.6.md)..

### Logging Improvements

Laravel 5.6 mang đến những cải tiến lớn cho hệ thống ghi log. Tất cả cấu hình ghi log được lưu trong file cấu hình `config/logging.php` mới. Giờ đây, bạn có thể dễ dàng xây dựng các "stack" logging để gửi tlog message đến nhiều trình xử lý log khác nhau. Ví dụ: bạn có thể gửi tất cả các message cấp độ `debug` vào log hệ thống trong khi gửi các message cấp độ `error` vào Slack để team của bạn có thể nhanh chóng xử lý các lỗi:

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],
    ],

Ngoài ra, giờ đây việc tùy chỉnh các channel log cũng dễ dàng hơn bằng cách sử dụng chức năng "tap" mới của hệ thống ghi log. Để biết thêm thông tin, hãy xem [tài liệu đầy đủ về ghi log](/docs/{{version}}/logging).

### Single Server Task Scheduling

> {note} Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng driver cache `memcached` hoặc `redis` làm driver cache mặc định của ứng dụng của bạn. Ngoài ra, tất cả các server phải được giao tiếp với cùng một server cache trung tâm.

Nếu ứng dụng của bạn đang chạy trên nhiều server, bạn có thể giới hạn schedule job chỉ được chạy trên một server duy nhất. Ví dụ: giả sử bạn đang có một task schedule là tạo một báo cáo vào mỗi tối thứ Sáu. Nếu schedule của bạn đang chạy trên ba server worker, thì task schedule sẽ được chạy trên cả ba server và tạo báo cáo ba lần. Không tốt!

Để yêu cầu task chỉ được chạy trên một server, hãy sử dụng phương thức `onOneServer` khi bạn định nghĩa task schedule. Server đầu tiên nhận được task sẽ dùng một atomic lock trên job đó để ngăn các server khác chạy cùng một task vào cùng một thời điểm:

    $schedule->command('report:generate')
             ->fridays()
             ->at('17:00')
             ->onOneServer();

### Dynamic Rate Limiting

Khi chỉ định [giới hạn số lượt truy cập](/docs/{{version}}/routing#rate-limiting) trên một nhóm các route trong các bản phát hành trước của Laravel, bạn buộc phải cung cấp số lượng request tối đa:

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

Trong Laravel 5.6, bạn có thể linh hoạt chỉ định số lượng request tối đa dựa vào một thuộc tính của model `User`. Ví dụ: nếu model `User` của bạn chứa thuộc tính `rate_limit`, bạn có thể truyền tên của thuộc tính đó vào middleware `throttle` để nó sử dụng để tính toán số lượng request tối đa:

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

### Broadcast Channel Classes

Nếu ứng dụng của bạn sử dụng nhiều channel khác nhau, thì file `routes/channels.php` của bạn có thể trở nên rất cồng kềnh. Vì vậy, thay vì sử dụng Closure để cấp quyền cho các channel, bạn có thể sử dụng các class channel. Để tạo một class channel mới, hãy sử dụng lệnh Artisan `make:channel`. Lệnh này sẽ lưu một class channel mới vào trong thư mục `App/Broadcasting`.

    php artisan make:channel OrderChannel

Tiếp theo, đăng ký channel của bạn vào trong file `routes/channels.php`:

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('order.{order}', OrderChannel::class);

Cuối cùng, bạn có thể viết các logic cấp quyền cho channel của bạn vào trong phương thức `join` của class channel. Phương thức `join` sẽ chứa cùng một logic với code mà bạn thường viết trong Closure cấp quyền channel của bạn. Tất nhiên, bạn cũng có thể tận dụng lợi thế của liên kết model channel:

    <?php

    namespace App\Broadcasting;

    use App\User;
    use App\Order;

    class OrderChannel
    {
        /**
         * Create a new channel instance.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Authenticate the user's access to the channel.
         *
         * @param  \App\User  $user
         * @param  \App\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

### API Controller Generation

Khi khai báo một resource route mà sẽ được sử dụng bởi các API, bạn sẽ muốn loại bỏ các route mà phải nhập form HTML như `create` và` edit`. Để tạo một API resource controller mà không chứa các phương thức đó, bây giờ bạn có thể sử dụng switch `--api` khi chạy lệnh `make:controller`:

    php artisan make:controller API/PhotoController --api

### Model Serialization Improvements

Trong các bản phát hành trước của Laravel, các model queue sẽ không khôi phục lại được các quan hệ mà đã được load. Trong Laravel 5.6, các quan hệ mà đã được load trên model thì khi queue model, nó sẽ được tự động load lại khi job được queue xử lý.

### Eloquent Date Casting

Giờ đây, bạn có thể tùy biến định dạng của cột date trong Eloquent. Để bắt đầu, hãy chỉ định định dạng date mà bạn mong muốn trong khai báo cast. Sau khi đã được chỉ định, định dạng đó sẽ được sử dụng khi chuyển hóa model thành một mảng hoặc JSON:

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];

### Blade Component Aliases

Nếu các component Blade của bạn được lưu trữ trong một thư mục con, bạn có thể muốn đặt tên bí danh cho chúng để dễ dàng truy cập hơn. Ví dụ, hãy tưởng tượng một component Blade được lưu trữ trong thư mục `resources/views/components/alert.blade.php`. Bạn có thể sử dụng phương thức `component` để đặt tên bí danh cho component từ `components.alert` thành `alert`.

    Blade::component('components.alert', 'alert');

Khi component đã được đặt tên bí danh, bạn có thể render nó bằng cách sử dụng lệnh sau:

    @alert('alert', ['type' => 'danger'])
        You are not allowed to access this resource!
    @endalert

Bạn có thể bỏ qua các parameter của component nếu nó không có thêm slot:

    @alert
        You are not allowed to access this resource!
    @endalert

### Argon2 Password Hashing

Nếu bạn đang xây dựng ứng dụng từ PHP 7.2.0 trở lên, Laravel hiện hỗ trợ hàm hash mật khẩu thông qua thuật toán Argon2. Driver hash mặc định cho ứng dụng của bạn được lưu trong file cấu hình `config/hashing.php` mới.

### UUID Methods

Laravel 5.6 giới thiệu hai phương thức mới để tạo UUID: `Str::uuid` và `Str::orderedUuid`. Phương thức `orderedUuid` sẽ tạo một timestamp vào đầu UUID để lập index dễ dàng và hiệu quả hơn cho các cơ sở dữ liệu như MySQL. Mỗi phương thức này trả về một đối tượng `Ramsey\Uuid\Uuid`:

    use Illuminate\Support\Str;

    return (string) Str::uuid();

    return (string) Str::orderedUuid();

### Collision

Từ bây giờ, mặc định ứng dụng `laravel/laravel` chứa một phụ thuộc Composer `dev` cho package [Collision](https://github.com/nunomaduro/collision) do Nuno Maduro tạo ra. Package này cung cấp các báo cáo lỗi tuyệt vời khi tương tác với ứng dụng Laravel của bạn trên dòng lệnh:

<img src="https://raw.githubusercontent.com/nunomaduro/collision/stable/docs/example.png" width="600" height="388">

### Bootstrap 4

Tất cả các giao diện người dùng như phần authentication và Vue component mẫu đều đã được nâng cấp lên [Bootstrap 4](https://blog.getbootstrap.com/2018/01/18/bootstrap-4/). Mặc định, việc tạo các link phân trang cũng được set mặc định là Bootstrap 4.
