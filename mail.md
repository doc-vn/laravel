# Mail

- [Giới thiệu](#introduction)
    - [Yêu cầu driver](#driver-prerequisites)
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
- [Mail và Local Development](#mail-and-local-development)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một API đơn giản, gọn gàng trên thư viện [SwiftMailer](https://swiftmailer.symfony.com/) với các driver như SMTP, Mailgun, Postmark, SparkPost, Amazon SES và `sendmail`, cho phép bạn nhanh chóng bắt đầu gửi mail thông qua dịch vụ trên đám mây hoặc local mà bạn chọn.

<a name="driver-prerequisites"></a>
### Yêu cầu driver

Các driver dựa trên API như Mailgun, SparkPost, và Postmark thường đơn giản hơn và nhanh hơn là các máy chủ SMTP. Nếu có thể, bạn nên sử dụng một trong những driver này. Tất cả các driver API đều yêu cầu thư viện Guzzle HTTP, thư viện này có thể được cài đặt thông qua trình quản lý package Composer:

    composer require guzzlehttp/guzzle

#### Mailgun Driver

Để sử dụng driver Mailgun, trước tiên bạn hãy cài đặt Guzzle, sau đó set tùy chọn `driver` trong file cấu hình `config/mail.php` của bạn thành `mailgun`. Tiếp theo, hãy kiểm tra file cấu hình `config/services.php` của bạn đã có chứa các tùy chọn sau chưa:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

Nếu bạn không sử dụng [Mailgun khu vực](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions) "Hoa Kỳ", thì bạn có thể cần định nghĩa endpoint khu vực của bạn trong file cấu hình `services`:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
        'endpoint' => 'api.eu.mailgun.net',
    ],

#### Postmark Driver

Để sử dụng driver Postmark, hãy cài đặt SwiftMailer transport của Postmark qua Composer:

    composer require wildbit/swiftmailer-postmark

Tiếp theo, cài đặt Guzzle và set tùy chọn `driver` trong file cấu hình `config/mail.php` của bạn thành `postmark`. Cuối cùng, hãy đảm bảo rằng file cấu hình `config/services.php` của bạn đã chứa các tùy chọn sau:

    'postmark' => [
        'token' => 'your-postmark-token',
    ],

#### SparkPost Driver

Để sử dụng driver SparkPost, trước tiên hãy cài đặt Guzzle, sau đó set tùy chọn `driver` trong file cấu hình `config/mail.php` của bạn thành `sparkpost`. Tiếp theo, hãy kiểm tra file cấu hình `config/services.php` của bạn đã có chứa các tùy chọn sau chưa:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

Nếu cần, bạn cũng có thể cấu hình [API endpoint](https://developers.sparkpost.com/api/#header-endpoints) nào sẽ được sử dụng:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
        'options' => [
            'endpoint' => 'https://api.eu.sparkpost.com/api/v1/transmissions',
        ],
    ],

#### SES Driver

Để sử dụng driver Amazon SES, trước tiên bạn phải cài đặt SDK Amazon AWS cho PHP. Bạn có thể cài đặt thư viện này bằng cách thêm dòng lệnh sau vào phần `require` của file `composer.json` của bạn và chạy lệnh `composer update`:

    "aws/aws-sdk-php": "~3.0"

Tiếp theo hãy set tùy chọn `driver` trong file cấu hình `config/mail.php` của bạn thành `ses`. Tiếp theo, hãy kiểm tra file cấu hình `config/services.php` của bạn đã có chứa các tùy chọn sau chưa:

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

Nếu bạn cần thêm [một số tùy chọn bổ sung](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-email-2010-12-01.html#sendrawemail) khi thực hiện request SES `SendRawEmail`, bạn có thể cần định nghĩa thêm một mảng `options` trong cấu hình `ses` của bạn:

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
        'options' => [
            'ConfigurationSetName' => 'MyConfigurationSet',
            'Tags' => [
                [
                    'Name' => 'foo',
                    'Value' => 'bar',
                ],
            ],
        ],
    ],

<a name="generating-mailables"></a>
## Tạo Mailables

Trong Laravel, mỗi loại email được gửi bởi application của bạn được thể hiện dưới dạng một class "mailable". Các class này được lưu trữ trong thư mục `app/Mail`. Đừng lo lắng nếu bạn không thấy thư mục này trong application của bạn, vì nó sẽ được tạo khi bạn tạo class mailable đầu tiên bằng cách sử dụng lệnh `make:mail`:

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## Viết Mailables

Tất cả các cấu hình dành cho một class mailable đều được thực hiện trong phương thức `build`. Trong phương thức này, bạn có thể gọi các phương thức khác nhau, chẳng hạn như `from`, `subject`, `view`, và `attach` để cấu hình việc trình bày và gửi email.

<a name="configuring-the-sender"></a>
### Cấu hình Sender

#### Using The `from` Method

Trước tiên, hãy xem cấu hình người gửi email. Hay nói cách khác, ai sẽ gửi email "from". Có hai cách để cấu hình người gửi. Đầu tiên, bạn có thể sử dụng phương thức `from` trong phương thức `build` trong class mailable của bạn:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

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

#### Via Public Properties

Thông thường, bạn sẽ muốn chuyển một số dữ liệu cho view mà bạn có thể sử dụng khi hiển thị HTML. Có hai cách để bạn có thể cung cấp dữ liệu cho view của bạn. Đầu tiên, mọi thuộc tính công khai được định nghĩa trong class mailable của bạn sẽ tự động được cung cấp cho view. Vì thế, ví dụ, bạn có thể chuyển dữ liệu vào hàm khởi tạo của class mailable của bạn và set dữ liệu đó thành thuộc tính công khai được định nghĩa trong class:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        public $order;

        /**
         * Create a new message instance.
         *
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

#### Via The `with` Method:

Nếu bạn muốn tùy chỉnh định dạng dữ liệu email của bạn trước khi nó được gửi đến template, bạn có thể tự truyền dữ liệu của bạn đến view thông qua phương thức `with`. Thông thường, bạn vẫn sẽ truyền dữ liệu qua hàm khởi tạo của class mailable; tuy nhiên, bạn nên set dữ liệu này thành thuộc tính `protected` hoặc `private` để những dữ liệu này sẽ không được tự động đưa cho template. Sau đó, khi gọi phương thức `with`, để truyền những dữ liệu mà bạn muốn đưa vào cho template:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        protected $order;

        /**
         * Create a new message instance.
         *
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

#### Đính kèm file từ disk

Nếu bạn đã lưu một file trên một trong các [filesystem disk](/docs/{{version}}/filesystem), thì bạn có thể đính kèm file đó vào email bằng phương thức `attachFromStorage`:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('email.orders.shipped')
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
       return $this->view('email.orders.shipped')
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
       return $this->view('email.orders.shipped')
                   ->attachFromStorageDisk('s3', '/path/to/file');
    }

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

Nhúng hình ảnh vào trong email của bạn thường rất cồng kềnh; tuy nhiên, Laravel cung cấp một cách xử lý thuận tiện để đính kèm hình ảnh của bạn vào email và lấy CID của hình ảnh đó để hiển thị khi người dùng mở email. Để nhúng hình ảnh, hãy sử dụng phương thức `embed` trên biến `$message` trong template email của bạn. Laravel tự động cung cấp biến `$message` cho tất cả các template email của bạn, vì vậy bạn không cần phải lo lắng về việc truyền nó:

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToImage) }}">
    </body>

> {note} Biến `$message` sẽ không sẵn trong phiên bản văn bản thuần vì phiên bản văn bản thuần không sử dụng file đính kèm nội dung.

#### Embedding Raw Data Attachments

Nếu bạn đã có một chuỗi raw dữ liệu mà bạn muốn nhúng vào một template email, bạn có thể sử dụng phương thức `embedData` trên biến `$message`:

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### Tuỳ biến SwiftMailer Message

Phương thức `withSwiftMessage` của class `Mailable` cho phép bạn đăng ký một callback sẽ được gọi với một instance message raw SwiftMailer trước khi gửi message. Điều này cung cấp cho bạn một cách để tùy biến message trước khi nó được gửi đi:

    /**
        * Build the message.
        *
        * @return $this
        */
    public function build()
    {
        $this->view('emails.orders.shipped');

        $this->withSwiftMessage(function ($message) {
            $message->getHeaders()
                    ->addTextHeader('Custom-Header', 'HeaderValue');
        });
    }

<a name="markdown-mailables"></a>
## Markdown Mailables

Markdown mailable message cho phép bạn tận dụng lợi thế của các template xây dụng sẵn và các component của mail notification có trong các mailable của bạn. Vì các message được viết bằng Markdown, nên Laravel có thể hiển thị các template HTML đẹp, đáp ứng cho các message đồng thời tự động tạo một bản sao đơn giản.

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
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### Viết Markdown Messages

Các markdown mailable sử dụng kết hợp các component của Blade và cú pháp Markdown cho phép bạn dễ dàng tạo ra format mail trong khi vẫn tận dụng được các component đã được tạo sẵn của Laravel:

    @component('mail::message')
    # Order Shipped

    Your order has been shipped!

    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

> {tip} Đừng sử dụng thụt đầu dòng khi viết email bằng Markdown. Trình phân tích cú pháp Markdown sẽ hiển thị nội dung thụt đầu dòng dưới dạng code block.

#### Button Component

Component button sẽ tạo ra một link button ở chính giữa. Component này chấp nhận hai tham số, một là `url` và một là tùy chọn `color`. Các màu được hỗ trợ là `primary`, `success` và `error`. Bạn có thể thêm các component button vào message nếu muốn:

    @component('mail::button', ['url' => $url, 'color' => 'success'])
    View Order
    @endcomponent

#### Panel Component

Component panel sẽ tạo một block văn bản trong một panel có màu nền hơi khác so với các phần khác của message. Điều này cho phép bạn thu hút sự chú ý của người dùng đến block văn bản:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

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

#### Customizing The CSS

Sau khi export các component, thư mục `resources/views/vendor/mail/html/themes` sẽ chứa một file `default.css`. Bạn có thể tùy biến CSS trong file này và các tuỳ biến này của bạn sẽ tự động được nhúng vào trong các hiển thị HTML cho mail Markdown của bạn.

Nếu bạn muốn xây dựng một theme mới cho các component Markdown của Laravel, bạn có thể tạo một file CSS mới trong thư mục `html/themes`. Sau khi tạo tên và lưu file CSS của bạn, hãy cập nhật tùy chọn `theme` trong file cấu hình `mail` để khớp với tên theme mới của bạn.

Để tùy chỉnh theme cho một mail riêng lẻ, bạn có thể set thuộc tính `$theme` trong class mailable thành tên theme mà bạn muốn sử dụng khi gửi mailable đó.

<a name="sending-mail"></a>
## Gửi Mail

Để gửi một message, hãy sử dụng phương thức `to` trong [facade](/docs/{{version}}/facades) `Mail`. Phương thức `to` chấp nhận một địa chỉ email, một instance user hoặc một collection user. Nếu bạn truyền qua một đối tượng hoặc một collection các đối tượng, mailer sẽ tự động sử dụng các thuộc tính `email` và `name` trong các đối tượng đã được truyền vào khi set người nhận email, vì vậy hãy đảm bảo các thuộc tính này có tồn tại trong các đối tượng của bạn. Khi bạn đã khai báo người nhận, bạn có thể truyền một instance của class mailable của bạn tới phương thức `send`:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // Ship order...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

Bạn không bị giới hạn chỉ trong khai báo người nhận "to" khi gửi message. Mà bạn có thể tự do set "to", "cc" và "bcc" cho người nhận, tất cả có thể được kết hợp trong một chuỗi phương thức duy nhất:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="rendering-mailables"></a>
## Hiển thị Mailable

Thỉnh thoảng bạn có thể muốn xem nội dung HTML của một mailable mà không cần phải gửi. Để thực hiện điều này, bạn có thể gọi phương thức `render` của mailable. Phương thức này sẽ trả về nội dung của mailable dưới dạng một chuỗi:

    $invoice = App\Invoice::find(1);

    return (new App\Mail\InvoicePaid($invoice))->render();

<a name="previewing-mailables-in-the-browser"></a>
### Xem trước Mailable trên trình duyệt

Khi thiết kế một template của một mailable, sẽ rất tiện lợi, nếu xem được mailable đó trong trình duyệt web của bạn giống như một template Blade. Vì lý do này, Laravel cho phép bạn trả về một mailable bất kỳ từ một route Closure hoặc controller. Khi một mailable được trả về, nó sẽ được tạo và hiển thị trong trình duyệt, cho phép bạn nhanh chóng xem trước thiết kế của nó mà không cần phải gửi nó đến một địa chỉ email thực tế:

    Route::get('mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="queueing-mail"></a>
### Queueing Mail

#### Queueing A Mail Message

Vì việc gửi email có thể mất nhiều thời gian của application, nên nhiều nhà phát triển sẽ chọn cách queue email message để gửi dưới background. Laravel làm cho điều này trở nên dễ dàng bằng cách sử dụng [unified queue API](/docs/{{version}}/queues) được tích hợp sẵn trong facade `Mail`. Để queue một mail message, hãy sử dụng phương thức `queue` trong facade `Mail` sau khi khai báo người nhận thư:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

Phương thức này sẽ tự động đảm nhận việc tạo một job lên queue để message sẽ được gửi trong background. Bạn sẽ cần [cấu hình queue](/docs/{{version}}/queues) trước khi sử dụng tính năng này.

#### Delayed Message Queueing

Nếu bạn muốn delay việc gửi thư email trong queue, bạn có thể sử dụng phương thức `later`. Với tham số đầu tiên của phương thức `later` là một instance `DateTime` cho biết khi nào mail sẽ được gửi:

    $when = now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### Pushing To Specific Queues

Vì tất cả các class mailable mà được tạo bằng lệnh `make:mail` đều có sử dụng trait `Illuminate\Bus\Queueable`, nên bạn có thể gọi các phương thức `onQueue` và `onConnection` trong bất kỳ class mailable nào, cho phép bạn khai báo tên kết nối và tên queue sẽ cần khi gửi mail:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### Queueing By Default

Nếu bạn có class mailable mà luôn muốn sử dụng queue, bạn có thể implement contract `ShouldQueue` trên class. Bây giờ, ngay cả khi bạn gọi phương thức `send` khi gửi thư, thì mailable vẫn sẽ được queue vì nó đã được implement contract:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="localizing-mailables"></a>
## Ngôn ngữ trong Mailable

Laravel cho phép bạn gửi mailable bằng ngôn ngữ khác, khác với ngôn ngữ hiện tại của application và thậm chí sẽ nhớ ngôn ngữ này nếu mail đang được queue.

Để thực hiện điều này, facade `Mail` có cung cấp một phương thức `locale` để set ngôn ngữ mà bạn mong muốn. Application sẽ chuyển đổi thành ngôn ngữ này khi định dạng mailable và sau đó quay lại về ngôn ngữ trước đó khi quá trình định dạng này hoàn tất:

    Mail::to($request->user())->locale('es')->send(
        new OrderShipped($order)
    );

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

<a name="mail-and-local-development"></a>
## Mail và Local Development

Khi bạn đang phát triển một tính năng gửi email, có thể bạn không muốn gửi email đến các địa chỉ email thực sự. Laravel cung cấp một số cách để "vô hiệu hóa" việc gửi email trong quá trình phát triển ở local.

#### Log Driver

Thay vì gửi email, driver mail `log` sẽ viết tất cả các email vào file log của bạn để bạn tiện kiểm tra. Để biết thêm thông tin về cách cấu hình application của bạn theo từng môi trường, hãy xem [tài liệu cấu hình](/docs/{{version}}/configuration#environment-configuration).

#### Universal To

Một giải pháp khác được cung cấp bởi Laravel là set một người nhận cho tất cả các email được gửi bởi framework. Bằng cách này, tất cả các email được tạo bởi application của bạn sẽ được gửi đến một địa chỉ cụ thể, thay vì địa chỉ thực sự được khai báo khi gửi mail. Điều này có thể được thực hiện thông qua tùy chọn `to` trong file cấu hình `config/mail.php` của bạn:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

Cuối cùng, bạn có thể sử dụng một service như [Mailtrap](https://mailtrap.io) và driver `smtp` để gửi email của bạn đến hộp thư "giả" nơi mà bạn có thể xem các email trong một hòm thư thực sự. Cách tiếp cận này có lợi ích là cho phép bạn thực sự kiểm tra các email cuối cùng trong hòm thư của Mailtrap.

<a name="events"></a>
## Events

Laravel sẽ tạo hai event trong quá trình gửi mail. Event `MessageSending` sẽ được kích hoạt trước khi một message được gửi, trong khi event `MessageSent` sẽ được kích hoạt sau khi message đã được gửi. Hãy nhớ rằng, những event này được kích hoạt khi thư đang được *gửi*, chứ không phải là khi nó đã được queue. Bạn có thể đăng ký một listener event cho những event này trong `EventServiceProvider`:

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
