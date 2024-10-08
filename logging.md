# Logging

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Channel Driver có sẵn](#available-channel-drivers)
    - [Channel Prerequisites](#channel-prerequisites)
    - [Cảnh báo Logging Deprecation](#logging-deprecation-warnings)
- [Tạo Log Stack](#building-log-stacks)
- [Viết Log Messages](#writing-log-messages)
    - [Contextual Information](#contextual-information)
    - [Viết cho một Channel cụ thể](#writing-to-specific-channels)
- [Tuỳ biến Monolog Channel](#monolog-channel-customization)
    - [Tuỳ biến Monolog cho Channel](#customizing-monolog-for-channels)
    - [Tạo Monolog xử lý Channel](#creating-monolog-handler-channels)
    - [Tạo một channel tuỳ biến thông qua Factory](#creating-custom-channels-via-factories)

<a name="introduction"></a>
## Giới thiệu

Để giúp bạn hiểu thêm về những gì đang xảy ra trong ứng dụng của bạn, Laravel cung cấp các dịch vụ ghi log mạnh mẽ cho phép bạn log các message vào file, log lỗi hệ thống hoặc thậm chí là cả Slack để thông báo cho toàn bộ team của bạn.

Laravel logging được dựa trên "channel". Mỗi channel sẽ thể hiện một cách ghi log cụ thể. Ví dụ: channel `single` sẽ ghi các file log vào một file log duy nhất, trong khi channel `slack` sẽ gửi thông báo log đến Slack. Thông báo log có thể được ghi vào nhiều channel dựa trên mức độ nghiêm trọng của chúng.

Laravel sử dụng thư viện [Monolog](https://github.com/Seldaek/monolog) để cung cấp và hỗ trợ nhiều cách xử lý log mạnh mẽ. Laravel giúp bạn dễ dàng cấu hình các cách xử lý này, cho phép bạn pha trộn và kết hợp chúng lại để tùy chỉnh việc xử lý log trong ứng dụng của bạn.

<a name="configuration"></a>
## Cấu hình

Tất cả các tuỳ chọn cấu hình cho các hành động của hệ thống ghi log của ứng dụng của bạn được lưu trong file cấu hình `config/logging.php`. File này cho phép bạn cấu hình các channel log, vì vậy hãy đảm bảo là bạn đã xem qua các channel hiện có và các tùy chọn của chúng. Chúng ta cũng sẽ xem xét một số tùy chọn phổ biến ở bên dưới.

Mặc định, Laravel sẽ sử dụng channel `stack` để ghi log. Channel `stack` có thể được sử dụng để tổng hợp nhiều channel log thành một channel. Để biết thêm thông tin về cách xây dựng stack, hãy xem [tài liệu ở bên dưới](#building-log-stacks).

<a name="configuring-the-channel-name"></a>
#### Configuring The Channel Name

Mặc định, Monolog được khởi tạo bởi "tên channel" phù hợp với môi trường hiện tại đang chạy ứng dụng, chẳng hạn như `production` hoặc `local`. Để thay đổi giá trị này, hãy thêm tùy chọn `name` vào cấu hình channel của bạn:

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

<a name="available-channel-drivers"></a>
### Driver Channel có sẵn

Mỗi log channel được cung cấp bởi một "driver". Driver sẽ xác định xem cách thức và vị trí của log message sẽ được ghi lại. Những Driver log channel sau đây sẽ có sẵn trong mọi ứng dụng Laravel. Một mục dành riêng cho các driver này đã có sẵn trong file cấu hình `config/logging.php` của ứng dụng của bạn, vì vậy hãy nhớ xem lại file này để làm quen với nội dung của nó:

Name | Description
------------- | -------------
`custom` | Một driver gọi một factory cụ thể để tạo ra một channel
`daily` | Một driver Monolog dựa trên `RotatingFileHandler` xoay vòng theo ngày
`errorlog` | Một driver Monolog dựa trên `ErrorLogHandler`
`monolog` | Một driver Monolog factory có thể sử dụng bất kỳ Monolog handler nào được hỗ trợ
`null` | Một driver sẽ loại bỏ tất cả các message log
`papertrail` | Một driver Monolog dựa trên `SyslogUdpHandler`
`single` | Một file hoặc một channel ghi log dựa trên đường dẫn (`StreamHandler`)
`slack` | Một driver Monolog dựa trên `SlackWebhookHandler`
`stack` | Một wrapper để tạo điều kiện thuận lợi cho việc tạo một channel với "nhiều channel"
`syslog` | Một driver Monolog dựa trên `SyslogHandler`

> **Note**
> Xem tài liệu về [tùy chỉnh channel nâng cao](#monolog-channel-customization) để tìm hiểu thêm về driver `monolog` và `custom`.

<a name="channel-prerequisites"></a>
### Channel Prerequisites

<a name="configuring-the-single-and-daily-channels"></a>
#### Configuring The Single and Daily Channels

Các channel `single` và `daily` có thêm ba tùy chọn cấu hình khác: `bubble`, `permission`, và `locking`.

Name | Description | Default
------------- | ------------- | -------------
`bubble` | Cho biết messages đang được xử lý có được gửi sang channel khác sau khi xử lý xong hay không | `true`
`locking` | Cố gắng khóa file log trước khi ghi vào nó | `false`
`permission` | Quyền của file log | `0644`

Ngoài ra, thời hạn lưu giữ file log cho channel `daily` có thể được cấu hình thông qua tùy chọn `days`:

Name | Description                                                       | Default
------------- |-------------------------------------------------------------------| -------------
`days` | Số ngày mà các file daily log phải được lưu giữ | `7`

<a name="configuring-the-papertrail-channel"></a>
#### Configuring The Papertrail Channel

Channel `papertrail` sẽ yêu cầu các tùy chọn cấu hình `host` và `port`. Bạn có thể lấy các giá trị này từ [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app).

<a name="configuring-the-slack-channel"></a>
#### Configuring The Slack Channel

Channel `slack` yêu cầu một cấu hình `url`. URL này phải khớp với một URL đã cho của một [webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) mà bạn đã cấu hình trong nhóm Slack của bạn.

Mặc định, Slack sẽ chỉ nhận các log ở cấp độ `critical` trở lên; tuy nhiên, bạn có thể điều chỉnh điều này trong file cấu hình `config/logging.php` của bạn bằng cách sửa tùy chọn cấu hình `level` trong mảng cấu hình của channel log Slack của bạn.

<a name="logging-deprecation-warnings"></a>
### Cảnh báo Logging Deprecation

PHP, Laravel và các thư viện khác thường thông báo cho người dùng biết một số tính năng của php, laravel hoặc của một thư viện khác sẽ không dùng được nữa và sẽ bị loại bỏ trong những phiên bản khác. Nếu muốn ghi lại những cảnh báo này, bạn có thể chỉ định log channel `deprecations` trong file cấu hình `config/logging.php` của ứng dụng:

    'deprecations' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),

    'channels' => [
        ...
    ]

Hoặc, bạn có thể định nghĩa một log channel có tên là `deprecations`. Nếu log channel có tên này tồn tại, nó sẽ luôn được sử dụng để ghi lại các trường hợp ngừng sử dụng như thế này:

    'channels' => [
        'deprecations' => [
            'driver' => 'single',
            'path' => storage_path('logs/php-deprecation-warnings.log'),
        ],
    ],

<a name="building-log-stacks"></a>
## Tạo Log Stack

Như đã đề cập trước đây, driver `stack` cho phép bạn kết hợp nhiều channel thành một channel duy nhất để thuận tiện. Để minh họa cho cách sử dụng stack log, hãy xem một cấu hình mẫu sau mà bạn có thể thấy trong ứng dụng thực tế:

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],

        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],

Hãy cùng xem xét cấu hình này. Đầu tiên, hãy để ý đến channel `stack` của chúng ta, nó được tổng hợp từ hai channel khác nhau thông qua tùy chọn `channels` của nó: `syslog` và `slack`. Vì vậy, khi ghi log message, cả hai channel này đều sẽ được ghi log message. Tuy nhiên, như chúng ta sẽ thấy, việc các channel này có thực sự ghi log hay không có thể được xác định bởi mức độ nghiêm trọng của tin nhắn.

<a name="log-levels"></a>
#### Log Levels

Hãy lưu ý tùy chọn cấu hình `level` có trong cấu hình channel `syslog` và `slack` có trong ví dụ ở trên. Tùy chọn này sẽ xác định xem "mức độ" tối thiểu mà một message phải có để có thể được channel ghi log. Monolog hỗ trợ các service ghi log của Laravel, cung cấp tất cả các cấp độ log được định nghĩa trong [đặc tả RFC 5424](https://tools.ietf.org/html/rfc5424). Theo thứ tự giảm dần về mức độ nghiêm trọng, các mức log này là: **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, và **debug**.

Vì thế, code dưới đây là chúng ta đang ghi log một message bằng phương thức `debug`:

    Log::debug('An informational message.');

Dựa vào cấu hình ở phía trên, thì channel `syslog` sẽ ghi message này vào trong system log; tuy nhiên, vì message này không phải là loại `critical` hoặc lớn hơn, nên nó sẽ không được gửi đến Slack. Tuy nhiên, nếu chúng ta ghi lại một log message là `emergency`, thì nó sẽ được gửi đến cả system log và Slack vì mức độ `emergency` sẽ cao hơn mức độ mà chúng ta đã cài đặt cho cả hai channel:

    Log::emergency('The system is down!');

<a name="writing-log-messages"></a>
## Viết Log Messages

Bạn có thể ghi thêm thông tin vào log bằng cách sử dụng [facade](/docs/{{version}}/facades) `Log`. Như đã đề cập ở trên, log sẽ cung cấp tám cấp độ ghi log được định nghĩa trong [đặc tả RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** và **debug**:

    use Illuminate\Support\Facades\Log;

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

Vì vậy, bạn có thể gọi bất kỳ phương thức nào trong các phương thức này để ghi log một message cho một cấp độ tương ứng. Mặc định, message sẽ được ghi vào channel log như được cấu hình bởi file cấu hình `logging` của bạn:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Support\Facades\Log;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */
        public function show($id)
        {
            Log::info('Showing the user profile for user: '.$id);

            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

<a name="contextual-information"></a>
### Contextual Information

Một mảng dữ liệu có thể được truyền vào cho các phương thức log. Các dữ liệu này sẽ được định dạng và hiển thị cùng với thông báo log:

    use Illuminate\Support\Facades\Log;

    Log::info('User failed to login.', ['id' => $user->id]);

Đôi khi, bạn có thể muốn chỉ định một số thông tin ngữ cảnh cần được đưa vào log. Ví dụ: bạn có thể muốn ghi lại ID request được liên kết với từng request được gửi đến ứng dụng của bạn trong một channel cụ thể. Để thực hiện điều này, bạn có thể gọi phương thức `withContext` của facade `Log`:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Str;

    class AssignRequestId
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            $requestId = (string) Str::uuid();

            Log::withContext([
                'request-id' => $requestId
            ]);

            return $next($request)->header('Request-Id', $requestId);
        }
    }

Nếu bạn muốn chia sẻ thông tin ngữ cảnh trên _tất cả_ các channel log, bạn có thể gọi phương thức `Log::shareContext()`. Phương thức này sẽ cung cấp thông tin ngữ cảnh cho tất cả các channel đã tạo và bất kỳ channel nào được tạo sau đó. Thông thường, phương thức `shareContext` phải được gọi từ phương thức `boot` của một application service provider:

    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Str;

    class AppServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Log::shareContext([
                'invocation-id' => (string) Str::uuid(),
            ]);
        }
    }

<a name="writing-to-specific-channels"></a>
### Viết cho một Channel cụ thể

Đôi khi bạn cũng có thể muốn ghi log một message vào một channel khác, khác với channel mặc định của ứng dụng của bạn. Bạn có thể sử dụng phương thức `channel` trên facade `Log` để lấy ra và log message vào bất kỳ channel nào mà đã được định nghĩa trong file cấu hình của bạn:

    use Illuminate\Support\Facades\Log;

    Log::channel('slack')->info('Something happened!');

Nếu bạn muốn tạo một stack để ghi log ở nhiều channel khác nhau, bạn có thể sử dụng phương thức `stack`:

    Log::stack(['single', 'slack'])->info('Something happened!');

<a name="on-demand-channels"></a>
#### On-Demand Channels

Cũng có thể tạo một channel theo yêu cầu bằng cách cung cấp cấu hình trong khi chạy thời gian thực mà không cần có cấu hình đó trong file cấu hình `logging` của ứng dụng của bạn. Để thực hiện điều này, bạn có thể truyền một mảng cấu hình tới phương thức `build` của facade `Log`:

    use Illuminate\Support\Facades\Log;

    Log::build([
      'driver' => 'single',
      'path' => storage_path('logs/custom.log'),
    ])->info('Something happened!');

Bạn cũng có thể muốn đưa channel theo yêu cầu này vào stack logging. Điều này có thể đạt được bằng cách đưa instance channel theo yêu cầu của bạn vào mảng được truyền cho phương thức `stack`:

    use Illuminate\Support\Facades\Log;

    $channel = Log::build([
      'driver' => 'single',
      'path' => storage_path('logs/custom.log'),
    ]);

    Log::stack(['slack', $channel])->info('Something happened!');

<a name="monolog-channel-customization"></a>
## Tuỳ biến Monolog Channel

<a name="customizing-monolog-for-channels"></a>
### Tuỳ biến Monolog cho Channel

Thỉnh thoảng bạn có thể cần kiểm soát cách cấu hình Monolog cho một channel. Ví dụ: bạn có thể muốn cấu hình một implementation Monolog `FormatterInterface` tùy biến cho channel `single` có sẵn của Laravel .

Để bắt đầu, hãy định nghĩa một mảng `tap` trong cấu hình channel của bạn. Mảng `tap` phải chứa một danh sách các class để tùy biến (hoặc "sửa") instance Monolog sau khi nó được tạo ra. Không có một vị trí mặc định nào để lưu các class này, vì vậy bạn có thể thoải mái tạo một thư mục trong ứng dụng của bạn để chứa các class này:

    'single' => [
        'driver' => 'single',
        'tap' => [App\Logging\CustomizeFormatter::class],
        'path' => storage_path('logs/laravel.log'),
        'level' => 'debug',
    ],

Sau khi bạn đã cấu hình tùy chọn `tap` trong file cấu hình channel của bạn, bạn đã sẵn sàng để định nghĩa class sẽ tùy biến instance Monolog. Class này chỉ cần một phương thức duy nhất: `__invoke` phương thức này nhận vào một instance `Illuminate\Log\Logger`. Instance `Illuminate\Log\Logger` sẽ chuyển hướng tất cả các cuộc gọi phương thức đến trực tiếp instance Monolog để thực hiện:

    <?php

    namespace App\Logging;

    use Monolog\Formatter\LineFormatter;

    class CustomizeFormatter
    {
        /**
         * Customize the given logger instance.
         *
         * @param  \Illuminate\Log\Logger  $logger
         * @return void
         */
        public function __invoke($logger)
        {
            foreach ($logger->getHandlers() as $handler) {
                $handler->setFormatter(new LineFormatter(
                    '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
                ));
            }
        }
    }

> **Note**
> Tất cả các class "tap" của bạn đều được [service container](/docs/{{version}}/container) resolve, vì vậy mọi phụ thuộc trong hàm constructor sẽ tự động được đưa vào.

<a name="creating-monolog-handler-channels"></a>
### Tạo Monolog xử lý Channel

Monolog có nhiều [xử lý có sẵn](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Handler) và Laravel không chứa bất kỳ channel nào cho mỗi xử lý đó. Trong một số trường hợp, bạn có thể muốn tạo ra một channel tùy biến là một instance của Monolog handler cụ thể tương ứng. Thì các channel này có thể dễ dàng được tạo ra bằng cách dùng driver `monolog`.

Khi sử dụng driver `monolog`, tùy chọn cấu hình `handler` sẽ chỉ định xử lý nào sẽ được khởi tạo. Còn các tham số khác thì đều có thể được chỉ định thông qua cách sử dụng tùy chọn cấu hình `with`:

    'logentries' => [
        'driver'  => 'monolog',
        'handler' => Monolog\Handler\SyslogUdpHandler::class,
        'with' => [
            'host' => 'my.logentries.internal.datahubhost.company.com',
            'port' => '10000',
        ],
    ],

<a name="monolog-formatters"></a>
#### Monolog Formatters

Khi sử dụng driver `monolog`, Monolog `LineFormatter` sẽ được sử dụng làm định dạng mặc định. Tuy nhiên, bạn có thể tùy chỉnh loại định dạng mà bạn muốn bằng cách sử dụng tùy chọn cấu hình `formatter` và `formatter_with`:

    'browser' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\BrowserConsoleHandler::class,
        'formatter' => Monolog\Formatter\HtmlFormatter::class,
        'formatter_with' => [
            'dateFormat' => 'Y-m-d',
        ],
    ],

Nếu bạn đang sử dụng xử lý Monolog mà có khả năng cung cấp một định dạng của riêng nó, thì bạn có thể set giá trị của tùy chọn cấu hình `formatter` thành `default`:

    'newrelic' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\NewRelicHandler::class,
        'formatter' => 'default',
    ],

<a name="creating-custom-channels-via-factories"></a>
### Tạo một channel tuỳ biến thông qua Factory

Nếu bạn muốn định nghĩa một channel tùy biến, trong đó bạn có toàn quyền kiểm soát về việc khởi tạo và cấu hình Monolog, bạn có thể chỉ định loại driver `custom` trong file cấu hình `config/logging.php` của bạn. Cấu hình của bạn nên chứa một tùy chọn `via` để chứa tên của class factory sẽ được gọi để tạo instance Monolog:

    'channels' => [
        'example-custom-channel' => [
            'driver' => 'custom',
            'via' => App\Logging\CreateCustomLogger::class,
        ],
    ],

Sau khi bạn đã cấu hình xong driver channel `custom`, bạn đã sẵn sàng để định nghĩa class sẽ tạo instance Monolog của bạn. Class này chỉ cần một phương thức `__invoke` duy nhất sẽ trả về instance logger Monolog. Phương thức này sẽ nhận một mảng cấu hình channel làm tham số duy nhất của nó:

    <?php

    namespace App\Logging;

    use Monolog\Logger;

    class CreateCustomLogger
    {
        /**
         * Create a custom Monolog instance.
         *
         * @param  array  $config
         * @return \Monolog\Logger
         */
        public function __invoke(array $config)
        {
            return new Logger(/* ... */);
        }
    }
