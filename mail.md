# Mail

- [Giới thiệu](#introduction)
    - [Cấu hình](#configuration)
    - [Yêu cầu driver](#driver-prerequisites)
    - [Cấu hình dự phòng](#failover-configuration)
- [Tạo Mailables](#generating-mailables)
- [Viết Mailables](#writing-mailables)
    - [Cấu hình Sender](#configuring-the-sender)
    - [Cấu hình View](#configuring-the-view)
    - [View Data](#view-data)
    - [Attachments](#attachments)
    - [Inline Attachments](#inline-attachments)
    - [Tuỳ biến SwiftMailer Message](#customizing-the-swiftmailer-message)
- [Markdown Mailables](#markdown-mailables)
    - [Tạo Markdown Mailables](#generating-markdown-mailables)
    - [Viết Markdown Messages](#writing-markdown-messages)
    - [Tuỳ biến Components](#customizing-the-components)
- [Gửi Mail](#sending-mail)
    - [Queueing Mail](#queueing-mail)
- [Hiển thị Mailable](#rendering-mailables)
    - [Xem trước Mailable trên trình duyệt](#previewing-mailables-in-the-browser)
- [Ngôn ngữ trong Mailable](#localizing-mailables)
- [Test Mail](#testing-mailables)
- [Mail và Local Development](#mail-and-local-development)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Gửi email không cần phải phức tạp. Laravel cung cấp một API đơn giản, gọn gàng dựa trên thư viện [SwiftMailer](https://swiftmailer.symfony.com/). Laravel và SwiftMailer cung cấp các driver cho việc gửi email như SMTP, Mailgun, Postmark, Amazon SES và `sendmail`, cho phép bạn nhanh chóng bắt đầu gửi mail thông qua dịch vụ trên đám mây hoặc local mà bạn chọn.

<a name="configuration"></a>
### Cấu hình

Các email service của Laravel có thể được cấu hình thông qua file cấu hình `config/mail.php` trong application cảu bạn. Mỗi mailer được cấu hình trong file này có thể có các cấu hình riêng duy nhất và thậm chí là "transport" của riêng nó, cho phép ứng dụng của bạn sử dụng các email service khác nhau để gửi một số email message nhất định. Ví dụ: ứng dụng của bạn có thể sử dụng Postmark để gửi email giao dịch trong khi sử dụng Amazon SES để gửi email hàng loạt.

Trong file cấu hình `mail`, bạn sẽ tìm thấy mảng cấu hình `mail`. Mảng này chứa các mục cấu hình mẫu cho từng loại driver và transport mà được Laravel hỗ trợ, trong khi giá trị cấu hình `default` sẽ định nghĩa xem driver nào sẽ được sử dụng khi ứng dụng của bạn cần gửi email.

<a name="driver-prerequisites"></a>
### Yêu cầu driver / transport

Các driver dựa trên API như Mailgun, và Postmark thường đơn giản và nhanh hơn là việc gửi mailthông qua các máy chủ SMTP. Bất cứ khi nào có thể, chúng tôi khuyên bạn nên sử dụng một trong những driver này. Tất cả các driver API đều yêu cầu thư viện Guzzle HTTP, thư viện này có thể được cài đặt thông qua trình quản lý package Composer:

    composer require guzzlehttp/guzzle

<a name="mailgun-driver"></a>
#### Mailgun Driver

Để sử dụng driver Mailgun, trước tiên bạn hãy cài đặt thư viện HTTP Guzzle. Sau đó set tùy chọn `default` trong file cấu hình `config/mail.php` của bạn thành `mailgun`. Tiếp theo, hãy kiểm tra file cấu hình `config/services.php` của bạn đã có chứa các tùy chọn sau chưa:

    'mailgun' => [
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
    ],

Nếu bạn không sử dụng [Mailgun khu vực](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions) "Hoa Kỳ", thì bạn có thể cần định nghĩa endpoint khu vực của bạn trong file cấu hình `services`:

    'mailgun' => [
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
        'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
    ],

<a name="postmark-driver"></a>
#### Postmark Driver

Để sử dụng driver Postmark, hãy cài đặt SwiftMailer transport của Postmark qua Composer:

    composer require wildbit/swiftmailer-postmark

Tiếp theo, cài đặt library HTTP Guzzle và set tùy chọn `default` trong file cấu hình `config/mail.php` của bạn thành `postmark`. Cuối cùng, hãy đảm bảo rằng file cấu hình `config/services.php` của bạn đã chứa các tùy chọn sau:

    'postmark' => [
        'token' => env('POSTMARK_TOKEN'),
    ],

Nếu bạn muốn chỉ định một Postmark message stream sẽ được sử dụng cho một mail, bạn có thể thêm tùy chọn cấu hình `message_stream_id` vào mảng cấu hình mail. Mảng cấu hình này có thể được tìm thấy trong file cấu hình `config/mail.php` của ứng dụng của bạn:

    'postmark' => [
        'transport' => 'postmark',
        'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
    ],

Bằng cách này, bạn cũng có thể thiết lập nhiều Postmark mail với các stream message khác nhau.

<a name="ses-driver"></a>
#### SES Driver

Để sử dụng driver Amazon SES, trước tiên bạn phải cài đặt SDK Amazon AWS cho PHP. Bạn có thể cài đặt thư viện này thông qua Composer package manager:

```bash
composer require aws/aws-sdk-php
```

Tiếp theo hãy set tùy chọn `default` trong file cấu hình `config/mail.php` của bạn thành `ses`. Tiếp theo, hãy kiểm tra file cấu hình `config/services.php` của bạn đã có chứa các tùy chọn sau chưa:

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],

Để sử dụng [thông tin xác thực tạm thời](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) thông qua một session token của AWS, bạn có thể thêm key `token` vào cấu hình SES của ứng dụng :

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'token' => env('AWS_SESSION_TOKEN'),
    ],

Nếu bạn muốn định nghĩa thêm [các tùy chọn](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-email-2010-12-01.html#sendrawemail) thì Laravel sẽ truyền các tuỳ chọn đó cho phương thức `SendRawEmail` của AWS SDK khi gửi email, bạn có thể định nghĩa mảng `options` trong cấu hình `ses` của bạn:

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'options' => [
            'ConfigurationSetName' => 'MyConfigurationSet',
            'Tags' => [
                ['Name' => 'foo', 'Value' => 'bar'],
            ],
        ],
    ],

<a name="failover-configuration"></a>
### Cấu hình dự phòng

Thỉnh thoảng, một service bên ngoài mà bạn đã cấu hình để gửi mail cho ứng dụng của bạn có thể bị không hoạt động. Trong những trường hợp như thế này, có thể hữu ích nếu bạn định nghĩa thêm một hoặc nhiều cấu hình gửi mail dự phòng và nó sẽ được sử dụng trong trường hợp driver gửi mail chính của bạn không hoạt động.

Để thực hiện điều này, bạn nên định nghĩa thêm một mailer trong file cấu hình `mail` của bạn và sử dụng transport `failover`. Mảng cấu hình cho mailer `failover` của bạn phải chứa một mảng các `mailers` sẽ tham chiếu đến thứ tự mà các driver mail sẽ được lựa chọn để gửi đi:

    'mailers' => [
        'failover' => [
            'transport' => 'failover',
            'mailers' => [
                'postmark',
                'mailgun',
                'sendmail',
            ],
        ],

        // ...
    ],

Khi mailer dự phòng của bạn đã được định nghĩa xong, bạn nên set mailer này làm mailer mặc định cho ứng dụng của bạn và chỉ định tên của nó làm giá trị cho key cấu hình `default` trong file cấu hình `mail` của ứng dụng của bạn:

    'default' => env('MAIL_MAILER', 'failover'),

<a name="generating-mailables"></a>
## Tạo Mailables

Trong Laravel, mỗi loại email được gửi bởi application của bạn được thể hiện dưới dạng một class "mailable". Các class này được lưu trữ trong thư mục `app/Mail`. Đừng lo lắng nếu bạn không thấy thư mục này trong application của bạn, vì nó sẽ được tạo khi bạn tạo class mailable đầu tiên bằng cách sử dụng lệnh `make:mail`:

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## Viết Mailables

Khi bạn đã tạo ra một class mailable, hãy mở nó ra để chúng ta có thể khám phá nội dung của nó. First, note that tất cả các cấu hình dành cho một class mailable đều được thực hiện trong phương thức `build`. Trong phương thức này, bạn có thể gọi các phương thức khác nhau, chẳng hạn như `from`, `subject`, `view`, và `attach` để cấu hình việc trình bày và gửi email.

> {tip} Bạn có thể khai báo các phụ thuộc vào phương thức `build` của mailable. Laravel [service container](/docs/{{version}}/container) sẽ tự động thêm các phần phụ thuộc này vào class của bạn.

<a name="configuring-the-sender"></a>
### Cấu hình Sender

<a name="using-the-from-method"></a>
#### Using The `from` Method

Trước tiên, hãy xem cấu hình người gửi email. Hay nói cách khác, ai sẽ gửi email "from". Có hai cách để cấu hình người gửi. Đầu tiên, bạn có thể sử dụng phương thức `from` trong phương thức `build` trong class mailable của bạn:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com', 'Example')
                    ->view('emails.orders.shipped');
    }

<a name="using-a-global-from-address"></a>
#### Using A Global `from` Address

Tuy nhiên, nếu application của bạn sử dụng cùng một địa chỉ "from" cho tất cả các email, thì nó có thể trở nên cồng kềnh khi gọi phương thức `from` trong mỗi class mailable mà bạn tạo. Thay vào đó, bạn có thể khai báo một địa chỉ "from" global trong file cấu hình `config/mail.php`. Địa chỉ này sẽ được sử dụng nếu không có địa chỉ "from" nào được khai báo trong class mailable:

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

Ngoài ra, bạn có thể cần định nghĩa một địa chỉ "reply_to" global trong file cấu hình `config/mail.php` của bạn:

    'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### Cấu hình View

Trong phương thức `build` của class mailable, bạn có thể sử dụng phương thức `view` để khai báo một template sẽ được sử dụng khi hiển thị nội dung email. Vì mỗi email thường sử dụng một [Blade template](/docs/{{version}}/blade) để hiển thị nội dung của nó, bạn có thể có toàn bộ sức mạnh và sự tiện lợi của công cụ tạo template của Blade khi xây dựng HTML cho email của bạn:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} Bạn có thể muốn tạo một thư mục `resources/views/emails` để chứa tất cả các template email của bạn; tuy nhiên, bạn có thể thoải mái lưu chúng ở bất cứ nơi nào bạn muốn trong thư mục `resources/views` của bạn.

<a name="plain-text-emails"></a>
#### Plain Text Emails

Nếu bạn muốn định nghĩa một văn bản thuần cho email của bạn, bạn có thể sử dụng phương thức `text`. Giống như phương thức `view`, phương thức `text` chấp nhận một tên template sẽ được sử dụng để hiển thị nội dung của email. Bạn có thể thoải mái định nghĩa cả phiên bản HTML và phiên bản văn bản thuần cho nội dung của bạn:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### View Data

<a name="via-public-properties"></a>
#### Via Public Properties

Thông thường, bạn sẽ muốn chuyển một số dữ liệu cho view mà bạn có thể sử dụng khi hiển thị HTML. Có hai cách để bạn có thể cung cấp dữ liệu cho view của bạn. Đầu tiên, mọi thuộc tính công khai được định nghĩa trong class mailable của bạn sẽ tự động được cung cấp cho view. Vì thế, ví dụ, bạn có thể chuyển dữ liệu vào hàm khởi tạo của class mailable của bạn và set dữ liệu đó thành thuộc tính công khai được định nghĩa trong class:

    <?php

    namespace App\Mail;

    use App\Models\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var \App\Models\Order
         */
        public $order;

        /**
         * Create a new message instance.
         *
         * @param  \App\Models\Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

Khi dữ liệu đã được set thành thuộc tính công khai, nó sẽ được tự động truyền vào trong view, vì vậy bạn có thể truy cập vào nó giống như bạn truy cập vào bất kỳ dữ liệu nào khác có trong template Blade của bạn:

    <div>
        Price: {{ $order->price }}
    </div>

<a name="via-the-with-method"></a>
#### Via The `with` Method:

Nếu bạn muốn tùy chỉnh định dạng dữ liệu email của bạn trước khi nó được gửi đến template, bạn có thể tự truyền dữ liệu của bạn đến view thông qua phương thức `with`. Thông thường, bạn vẫn sẽ truyền dữ liệu qua hàm khởi tạo của class mailable; tuy nhiên, bạn nên set dữ liệu này thành thuộc tính `protected` hoặc `private` để những dữ liệu này sẽ không được tự động đưa cho template. Sau đó, khi gọi phương thức `with`, để truyền những dữ liệu mà bạn muốn đưa vào cho template:

    <?php

    namespace App\Mail;

    use App\Models\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var \App\Models\Order
         */
        protected $order;

        /**
         * Create a new message instance.
         *
         * @param  \App\Models\Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

Khi dữ liệu đã được truyền đến phương thức `with`, nó sẽ tự động đưa vào trong view của bạn, vì vậy bạn có thể truy cập vào nó giống như bạn truy cập vào bất kỳ dữ liệu nào khác có trong template Blade của bạn:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### Attachments

Để thêm một file đính kèm vào email, hãy sử dụng phương thức `attach` trong phương thức `build` của class mailable. Phương thức `attach` chấp nhận một đường dẫn đầy đủ đến file làm tham số đầu tiên của nó:

    /**
        * Build the message.
        *
        * @return $this
        */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attach('/path/to/file');
    }

Khi đính kèm file một vào một email, bạn cũng có thể khai báo tên hiển thị hoặc loại MIME bằng cách truyền một `array` làm tham số thứ hai cho phương thức `attach`:

    /**
        * Build the message.
        *
        * @return $this
        */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="attaching-files-from-disk"></a>
#### Đính kèm file từ disk

Nếu bạn đã lưu một file trên một trong các [filesystem disk](/docs/{{version}}/filesystem), thì bạn có thể đính kèm file đó vào email bằng phương thức `attachFromStorage`:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('emails.orders.shipped')
                   ->attachFromStorage('/path/to/file');
    }

Nếu cần, bạn có thể chỉ định tên file đính kèm và các tùy chọn bổ sung bằng cách sử dụng tham số thứ hai và thứ ba cho phương thức `attachFromStorage`:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('emails.orders.shipped')
                   ->attachFromStorage('/path/to/file', 'name.pdf', [
                       'mime' => 'application/pdf'
                   ]);
    }

Phương thức `attachFromStorageDisk` có thể được sử dụng nếu bạn muốn chỉ định một disk khác, khác với disk mặc định của bạn:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('emails.orders.shipped')
                   ->attachFromStorageDisk('s3', '/path/to/file');
    }

<a name="raw-data-attachments"></a>
#### Raw Data Attachments

Phương thức `attachData` có thể được sử dụng để đính kèm một chuỗi raw byte dưới dạng file đính kèm. Ví dụ: bạn có thể sử dụng phương thức này nếu bạn đã tạo một file PDF trong bộ nhớ và muốn đính kèm file đó vào email mà không muốn ghi nó ra disk. Phương thức `attachData` chấp nhận các raw byte dữ liệu làm tham số đầu tiên và tên của file làm tham số thứ hai ngoài ra một mảng các tùy chọn làm tham số thứ ba của nó:

    /**
        * Build the message.
        *
        * @return $this
        */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="inline-attachments"></a>
### Inline Attachments

Nhúng hình ảnh vào trong email của bạn thường rất cồng kềnh; tuy nhiên, Laravel cung cấp một cách xử lý thuận tiện để đính kèm hình ảnh của bạn vào email. Để nhúng hình ảnh, hãy sử dụng phương thức `embed` trên biến `$message` trong template email của bạn. Laravel tự động cung cấp biến `$message` cho tất cả các template email của bạn, vì vậy bạn không cần phải lo lắng về việc truyền nó:

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToImage) }}">
    </body>

> {note} Biến `$message` sẽ không có sẵn trong phiên bản văn bản thuần vì phiên bản văn bản thuần không sử dụng file đính kèm nội dung.

<a name="embedding-raw-data-attachments"></a>
#### Embedding Raw Data Attachments

Nếu bạn đã có một chuỗi dữ liệu image raw mà bạn muốn nhúng vào một template email, bạn có thể gọi phương thức `embedData` trên biến `$message`. Khi gọi phương thức `embedData` này, bạn sẽ cần cung cấp tên file sẽ được gán cho image raw đó:

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### Tuỳ biến SwiftMailer Message

Phương thức `withSwiftMessage` của class `Mailable` cho phép bạn đăng ký một closure sẽ được gọi với một instance message SwiftMailer trước khi gửi message. Điều này cung cấp cho bạn một cách để tùy biến sâu vào message trước khi nó được gửi đi:

    /**
        * Build the message.
        *
        * @return $this
        */
    public function build()
    {
        $this->view('emails.orders.shipped');

        $this->withSwiftMessage(function ($message) {
            $message->getHeaders()->addTextHeader(
                'Custom-Header', 'Header Value'
            );
        });

        return $this;
    }

<a name="markdown-mailables"></a>
## Markdown Mailables

Markdown mailable message cho phép bạn tận dụng lợi thế của các template xây dụng sẵn và các component của [mail notification](/docs/{{version}}/notifications#mail-notifications) có trong các mailable của bạn. Vì các message được viết bằng Markdown, nên Laravel có thể hiển thị các template HTML đẹp, đáp ứng cho các message đồng thời tự động tạo một bản sao đơn giản.

<a name="generating-markdown-mailables"></a>
### Tạo Markdown Mailables

Để tạo một mailable với template Markdown tương ứng, bạn có thể sử dụng tùy chọn `--markdown` trong lệnh Artisan `make:mail`:

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

Sau đó, khi cấu hình mailable trong phương thức `build`, hãy gọi phương thức `markdown` thay vì phương thức `view`. Các phương thức `markdown` chấp nhận tên của template Markdown và một mảng dữ liệu để cung cấp cho template:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped', [
                        'url' => $this->orderUrl,
                    ]);
    }

<a name="writing-markdown-messages"></a>
### Viết Markdown Messages

Các markdown mailable sử dụng kết hợp các component của Blade và cú pháp Markdown cho phép bạn dễ dàng tạo ra format mail trong khi vẫn tận dụng được các component email UI đã được tạo sẵn của Laravel:

    @component('mail::message')
    # Order Shipped

    Your order has been shipped!

    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

> {tip} Đừng sử dụng thụt đầu dòng khi viết email bằng Markdown. Vì theo tiêu chuẩn Markdown, trình phân tích cú pháp sẽ hiển thị nội dung thụt đầu dòng dưới dạng một code block.

<a name="button-component"></a>
#### Button Component

Component button sẽ tạo ra một link button ở chính giữa. Component này chấp nhận hai tham số, một là `url` và một là tùy chọn `color`. Các màu được hỗ trợ là `primary`, `success` và `error`. Bạn có thể thêm các component button vào message nếu muốn:

    @component('mail::button', ['url' => $url, 'color' => 'success'])
    View Order
    @endcomponent

<a name="panel-component"></a>
#### Panel Component

Component panel sẽ tạo một block văn bản trong một panel có màu nền hơi khác so với các phần khác của message. Điều này cho phép bạn thu hút sự chú ý của người dùng đến block văn bản:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

<a name="table-component"></a>
#### Table Component

Component table cho phép bạn chuyển đổi một bảng Markdown thành một bảng HTML. Component này chấp nhận nội dung như một bảng Markdown bình thường. Căn chỉnh trái phải của cột cũng được mặc định hỗ trợ bởi cú pháp căn chỉnh cột của Markdown:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### Tuỳ biến Components

Bạn có thể export ra tất cả các component mail Markdown sang thư mục riêng của bạn để tùy biến. Để export các component, hãy sử dụng lệnh Artisan `vendor:publish` để export tag nội dung `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail

Lệnh này sẽ export các component mail Markdown sang thư mục `resources/views/vendor/mail`. Thư mục `mail` sẽ chứa một thư mục `html` và một thư mục `text`, mỗi thư mục chứa các hiển thị tương ứng của mỗi component. Bạn có thể tự do tùy biến các component này theo cách bạn muốn.

<a name="customizing-the-css"></a>
#### Customizing The CSS

Sau khi export các component, thư mục `resources/views/vendor/mail/html/themes` sẽ chứa một file `default.css`. Bạn có thể tùy biến CSS trong file này và các tuỳ biến này của bạn sẽ tự động được chuyển thành inline CSS trong các hiển thị HTML cho mail Markdown của bạn.

Nếu bạn muốn xây dựng một theme mới cho các component Markdown của Laravel, bạn có thể tạo một file CSS mới trong thư mục `html/themes`. Sau khi tạo tên và lưu file CSS của bạn, hãy cập nhật tùy chọn `theme` trong file cấu hình `config/mail.php` trong application của bạn để khớp với tên theme mới của bạn.

Để tùy chỉnh theme cho một mail riêng lẻ, bạn có thể set thuộc tính `$theme` trong class mailable thành tên theme mà bạn muốn sử dụng khi gửi mailable đó.

<a name="sending-mail"></a>
## Gửi Mail

Để gửi một message, hãy sử dụng phương thức `to` trong [facade](/docs/{{version}}/facades) `Mail`. Phương thức `to` chấp nhận một địa chỉ email, một instance user hoặc một collection user. Nếu bạn truyền qua một đối tượng hoặc một collection các đối tượng, mailer sẽ tự động sử dụng các thuộc tính `email` và `name` trong các đối tượng đã được truyền vào khi xác định người nhận của email, vì vậy hãy đảm bảo các thuộc tính này có tồn tại trong các đối tượng của bạn. Khi bạn đã khai báo người nhận, bạn có thể truyền một instance của class mailable của bạn tới phương thức `send`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Mail\OrderShipped;
    use App\Models\Order;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;

    class OrderShipmentController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $order = Order::findOrFail($request->order_id);

            // Ship the order...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

Bạn không bị giới hạn chỉ trong khai báo người nhận "to" khi gửi message. Mà bạn có thể tự do set thêm "to", "cc" và "bcc" cho nhiều người nhận bằng cách nối thêm các phương thức tương ứng của chúng lại với nhau:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="looping-over-recipients"></a>
#### Looping Over Recipients

Đôi khi, bạn có thể cần gửi một mailable đến một danh sách người nhận thông qua cách lặp một mảng gồm những người nhận và địa chỉ email của họ. Tuy nhiên, vì phương thức `to` sẽ gắn các địa chỉ email vào danh sách người nhận của mailable, nên mỗi lần lặp nó sẽ gửi một email khác đến những người đã gửi ở trước đó. Vì thế, nên bạn luôn phải tạo lại instance mailable cho từng người nhận:

    foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
        Mail::to($recipient)->send(new OrderShipped($order));
    }

<a name="sending-mail-via-a-specific-mailer"></a>
#### Sending Mail Via A Specific Mailer

Mặc định, Laravel sẽ gửi email bằng cách sử dụng mailer mà được cấu hình làm mailer `default` trong file cấu hình` mail` trong application của bạn. Tuy nhiên, bạn có thể sử dụng phương thức `mailer` để gửi một message với một cấu hình mailer cụ thể:

    Mail::mailer('postmark')
            ->to($request->user())
            ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### Queueing Mail

<a name="queueing-a-mail-message"></a>
#### Queueing A Mail Message

Vì việc gửi email có thể ảnh hưởng tiêu cực đến thời gian response của application, nên nhiều nhà phát triển sẽ chọn cách queue email message để gửi dưới background. Laravel làm cho điều này trở nên dễ dàng bằng cách sử dụng [unified queue API](/docs/{{version}}/queues) được tích hợp sẵn trong facade `Mail`. Để queue một mail message, hãy sử dụng phương thức `queue` trong facade `Mail` sau khi khai báo người nhận thư:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

Phương thức này sẽ tự động đảm nhận việc tạo một job lên queue để message sẽ được gửi trong background. Bạn sẽ cần [cấu hình queue](/docs/{{version}}/queues) trước khi sử dụng tính năng này.

<a name="delayed-message-queueing"></a>
#### Delayed Message Queueing

Nếu bạn muốn delay việc gửi thư email trong queue, bạn có thể sử dụng phương thức `later`. Với tham số đầu tiên của phương thức `later` là một instance `DateTime` cho biết khi nào mail sẽ được gửi:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later(now()->addMinutes(10), new OrderShipped($order));

<a name="pushing-to-specific-queues"></a>
#### Pushing To Specific Queues

Vì tất cả các class mailable mà được tạo bằng lệnh `make:mail` đều có sử dụng trait `Illuminate\Bus\Queueable`, nên bạn có thể gọi các phương thức `onQueue` và `onConnection` trong bất kỳ class mailable nào, cho phép bạn khai báo tên kết nối và tên queue sẽ cần khi gửi mail:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

<a name="queueing-by-default"></a>
#### Queueing By Default

Nếu bạn có class mailable mà luôn muốn sử dụng queue, bạn có thể implement contract `ShouldQueue` trên class. Bây giờ, ngay cả khi bạn gọi phương thức `send` khi gửi thư, thì mailable vẫn sẽ được queue vì nó đã được implement contract:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="queued-mailables-and-database-transactions"></a>
#### Queued Mailables & Database Transactions

Khi các queued mailable được gửi đi trong một database transaction, chúng có thể được xử lý bởi queue trước khi database transaction được commit. Khi điều này xảy ra, bất kỳ các cập nhật nào của bạn đã thực hiện đối với model hoặc một record nào đó trong quá trình database transaction có thể chưa được lưu vào trong cơ sở dữ liệu. Ngoài ra, bất kỳ model hoặc record nào được tạo ra trong transaction cũng có thể chưa tồn tại thực tế trong database. Nếu mailable của bạn phụ thuộc vào các model này, thì các lỗi không mong muốn có thể xảy ra khi job xử lý các queued mailable.

Nếu tùy chọn cấu hình `after_commit` của connection queue của bạn được set thành `false`, bạn vẫn có thể set một queued mailable cụ thể sẽ được gửi đi sau khi tất cả các open database transaction được commit bằng cách gọi phương thức `afterCommit` khi gửi mail:

    Mail::to($request->user())->send(
        (new OrderShipped($order))->afterCommit()
    );

Ngoài ra, bạn có thể gọi phương thức `afterCommit` từ hàm khởi tạo của mailable của bạn:

    <?php

    namespace App\Mail;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        use Queueable, SerializesModels;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->afterCommit();
        }
    }

> {tip} Để tìm hiểu thêm về cách giải quyết những vấn đề này, vui lòng xem lại tài liệu về [queued jobs và database transactions](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="rendering-mailables"></a>
## Rendering Mailables

Thỉnh thoảng bạn có thể muốn xem nội dung HTML của một mailable mà không cần thiết phải gửi đi. Để thực hiện điều này, bạn có thể gọi phương thức `render` của mailable. Phương thức này sẽ trả về nội dung HTML của mailable dưới dạng một chuỗi:

    use App\Mail\InvoicePaid;
    use App\Models\Invoice;

    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))->render();

<a name="previewing-mailables-in-the-browser"></a>
### Previewing Mailables In The Browser

Khi thiết kế một template của một mailable, sẽ thật thuận tiện nếu có thể nhanh chóng xem trước được mailable trong trình duyệt của bạn như là một dạng template Blade điển hình. Vì lý do này, Laravel cho phép bạn trả về bất kỳ mailable nào trực tiếp từ một route closure hoặc một controller. Khi một mailable được trả về, nó sẽ được tạo và hiển thị trong trình duyệt của bạn, cho phép bạn nhanh chóng xem trước các thiết kế của nó mà không cần phải gửi nó đến một địa chỉ email cụ thể:

    Route::get('/mailable', function () {
        $invoice = App\Models\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

> {note} [Inline attachments](#inline-attachments) sẽ không được render khi preview một mailable trong trình duyệt của bạn. Để preview những mailable như thế này, bạn nên gửi chúng đến một ứng dụng kiểm tra email chẳng hạn như [MailHog](https://github.com/mailhog/MailHog) hoặc [HELO](https://usehelo.com).

<a name="localizing-mailables"></a>
## Ngôn ngữ trong Mailable

Laravel cho phép bạn gửi mailable bằng ngôn ngữ khác, khác với ngôn ngữ hiện tại của request và thậm chí sẽ nhớ ngôn ngữ này nếu mail đang được queue.

Để thực hiện điều này, facade `Mail` có cung cấp một phương thức `locale` để set ngôn ngữ mà bạn mong muốn. Application sẽ chuyển sang ngôn ngữ này khi template của mailable đang được kiểm tra và sau đó quay lại về ngôn ngữ trước đó khi quá trình kiểm tra này hoàn tất:

    Mail::to($request->user())->locale('es')->send(
        new OrderShipped($order)
    );

<a name="user-preferred-locales"></a>
### User Preferred Locales

Thỉnh thoảng, các application sẽ lưu lại ngôn ngữ ưa thích của mỗi người dùng. Bằng cách implement contract `HasLocalePreference` trên một hoặc nhiều model của bạn, bạn có thể hướng dẫn Laravel sử dụng ngôn ngữ này khi gửi mail:

    use Illuminate\Contracts\Translation\HasLocalePreference;

    class User extends Model implements HasLocalePreference
    {
        /**
         * Get the user's preferred locale.
         *
         * @return string
         */
        public function preferredLocale()
        {
            return $this->locale;
        }
    }

Khi bạn đã implement xong interface này, Laravel sẽ tự động sử dụng ngôn ngữ này khi gửi mailable và notification tới model. Do đó, không cần phải gọi phương thức `locale` khi bạn sử dụng interface này:

    Mail::to($request->user())->send(new OrderShipped($order));

<a name="testing-mailables"></a>
## Test Mail

Laravel cung cấp một số phương thức thuận tiện để kiểm tra xem mailable của bạn có chứa nội dung mà bạn mong đợi hay không. Các phương thức này là: `assertSeeInHtml`, `assertDontSeeInHtml`, `assertSeeInText` và `assertDontSeeInText`.

Như bạn có thể mong đợi, các kiểm tra "HTML" sẽ yêu cầu phiên bản HTML của mailable có thể chứa một chuỗi nhất định, trong khi các kiểm tra "text" sẽ yêu cầu phiên bản text của mailable phải chứa một chuỗi nhất định:

    use App\Mail\InvoicePaid;
    use App\Models\User;

    public function test_mailable_content()
    {
        $user = User::factory()->create();

        $mailable = new InvoicePaid($user);

        $mailable->assertSeeInHtml($user->email);
        $mailable->assertSeeInHtml('Invoice Paid');

        $mailable->assertSeeInText($user->email);
        $mailable->assertSeeInText('Invoice Paid');
    }

<a name="testing-mailable-sending"></a>
#### Testing Mailable Sending

Chúng tôi khuyên bạn nên kiểm tra nội dung mailable một cách riêng biệt với các kiểm tra một mailable đã được "gửi" đến một người dùng cụ thể hay chưa. Để tìm hiểu thêm về cách kiểm tra xem mailable đã được gửi hay chưa, hãy xem tài liệu của chúng tôi về [Mail fake](/docs/{{version}}/mocking#mail-fake).

<a name="mail-and-local-development"></a>
## Mail và Local Development

Khi bạn đang phát triển một tính năng gửi email, có thể bạn không muốn gửi email đến các địa chỉ email thực sự. Laravel cung cấp một số cách để "vô hiệu hóa" việc gửi email trong quá trình phát triển ở local.

<a name="log-driver"></a>
#### Log Driver

Thay vì gửi email, driver mail `log` sẽ viết tất cả các email vào file log của bạn để bạn tiện kiểm tra. Thông thường, driver này sẽ chỉ được sử dụng trong quá trình phát triển ở local. Để biết thêm thông tin về cách cấu hình application của bạn theo từng môi trường, hãy xem [tài liệu cấu hình](/docs/{{version}}/configuration#environment-configuration).

<a name="mailtrap"></a>
#### HELO / Mailtrap / MailHog

Ngoài ra, bạn có thể sử dụng dịch vụ như [HELO](https://usehelo.com) hoặc [Mailtrap](https://mailtrap.io) và driver `smtp` để gửi email của bạn đến một mailbox"giả", nơi mà bạn có thể xem chúng trong một ứng dụng email thực sự. Cách tiếp cận này có lợi ích là cho phép bạn thực sự kiểm tra các email sau cùng sau quá trình phát triển.

Nếu đang sử dụng [Laravel Sail](/docs/{{version}}/sail), bạn có thể preview message của bạn bằng [MailHog](https://github.com/mailhog/MailHog). Khi Sail đang được chạy, bạn có thể truy cập vào giao diện MailHog tại: `http://localhost:8025`.

<a name="using-a-global-to-address"></a>
#### Using A Global `to` Address

Cuối cùng, bạn có thể chỉ định một địa chỉ "to" global bằng cách gọi phương thức `alwaysTo` được cung cấp bởi facade `Mail`. Thông thường, phương thức này phải được gọi từ phương thức `boot` của một trong các service provider trong ứng dụng của bạn:

    use Illuminate\Support\Facades\Mail;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->environment('local')) {
            Mail::alwaysTo('taylor@example.com');
        }
    }

<a name="events"></a>
## Events

Laravel sẽ tạo hai event trong quá trình gửi mail. Event `MessageSending` sẽ được kích hoạt trước khi một message được gửi, trong khi event `MessageSent` sẽ được kích hoạt sau khi message đã được gửi. Hãy nhớ rằng, những event này được kích hoạt khi thư đang được *gửi*, chứ không phải là khi nó đã được queue. Bạn có thể đăng ký nhiều listener event cho event này trong service provider `App\Providers\EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSendingMessage',
        ],
        'Illuminate\Mail\Events\MessageSent' => [
            'App\Listeners\LogSentMessage',
        ],
    ];
