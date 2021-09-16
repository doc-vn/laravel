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
- [Xem trước Mailables trên Browser](#previewing-mailables-in-the-browser)
- [Gửi Mail](#sending-mail)
    - [Queueing Mail](#queueing-mail)
- [Mail và Local Development](#mail-and-local-development)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp API đơn giản, gọn gàng trên thư viện [SwiftMailer] (https://swiftmailer.symfony.com/) với các driver cho SMTP, Mailgun, SparkPost, Amazon SES, hàm `mail` của PHP và `sendmail`, cho phép bạn nhanh chóng bắt đầu gửi mail thông qua dịch vụ dựa trên đám mây hoặc local mà bạn chọn.

<a name="driver-prerequisites"></a>
### Yêu cầu driver

Các driver dựa trên API như Mailgun và SparkPost thường đơn giản hơn và nhanh hơn là các máy chủ SMTP. Nếu có thể, bạn nên sử dụng một trong những driver này. Tất cả các driver API yêu cầu thư viện Guzzle HTTP, có thể được cài đặt thông qua trình quản lý package Composer:

    composer require guzzlehttp/guzzle

#### Mailgun Driver

Để sử dụng driver Mailgun, trước tiên bạn hãy cài đặt Guzzle, sau đó set tùy chọn `driver` trong file cấu hình `config/mail.php` của bạn thành `mailgun`. Tiếp theo, hãy kiểm tra file cấu hình `config/services.php` của bạn đã có chứa các tùy chọn sau chưa:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPost Driver

Để sử dụng driver SparkPost, trước tiên hãy cài đặt Guzzle, sau đó set tùy chọn `driver` trong file cấu hình `config/mail.php` của bạn thành `sparkpost`. Tiếp theo, hãy kiểm tra file cấu hình `config/services.php` của bạn đã có chứa các tùy chọn sau chưa:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
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

<a name="generating-mailables"></a>
## Tạo Mailables

Trong Laravel, mỗi loại email được gửi bởi application của bạn được thể hiện dưới dạng một class "mailable". Các class này được lưu trữ trong thư mục `app/Mail`. Đừng lo lắng nếu bạn không thấy thư mục này trong application của bạn, vì nó sẽ được tạo khi bạn tạo class mailable đầu tiên bằng cách sử dụng lệnh `make:mail`:

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## Viết Mailables

Tất cả cấu hình của một class mailable được thực hiện trong phương thức `build`. Trong phương thức này, bạn có thể gọi các phương thức khác nhau, chẳng hạn như `from`, `subject`, `view`, và `attach` để cấu hình việc trình bày và gửi email.

<a name="configuring-the-sender"></a>
### Cấu hình Sender

#### Using The `from` Method

Trước tiên, hãy xem cấu hình người gửi email. Hay nói cách khác, ai sẽ gửi email "from". Có hai cách để cấu hình người gửi. Đầu tiên, bạn có thể sử dụng phương thức `from` trong phương thức `build` có sẵn trong class mailable của bạn:

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

Tuy nhiên, nếu application của bạn sử dụng cùng một địa chỉ "from" cho tất cả các email của nó, thì nó có thể trở nên cồng kềnh khi gọi phương thức `from` trong mỗi class mailable mà bạn tạo. Thay vào đó, bạn có thể khai báo một địa chỉ "from" global trong file cấu hình `config/mail.php`. Địa chỉ này sẽ được sử dụng nếu không có địa chỉ "from" nào khác được khai báo trong class mailable:

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### Cấu hình View

Trong phương thức `build` của class mailable, bạn có thể sử dụng phương thức `view` để khai báo một template nào đó sẽ được sử dụng khi hiển thị nội dung email. Vì mỗi email thường sử dụng một [Blade template](/docs/{{version}}/blade) để hiển thị nội dung của nó, bạn có toàn bộ sức mạnh và sự tiện lợi của công cụ tạo template của Blade khi xây dựng HTML cho email của bạn:

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

Nếu bạn muốn định nghĩa một phiên bản văn bản đơn giản của email của bạn, bạn có thể sử dụng phương thức `text`. Giống như phương thức `view`, phương thức `text` chấp nhận một tên template sẽ được sử dụng để hiển thị nội dung của email. Bạn có thể thoải mái định nghĩa cả phiên bản HTML và văn bản thuần của tin nhắn của bạn:

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

Thông thường, bạn sẽ muốn chuyển một số dữ liệu cho view của bạn mà bạn có thể sử dụng khi hiển thị HTML của email. Có hai cách bạn có thể cung cấp dữ liệu cho view của bạn. Đầu tiên, mọi thuộc tính công khai được định nghĩa trên class mailable của bạn sẽ tự động được cung cấp cho view. Vì thế, ví dụ, bạn có thể chuyển dữ liệu vào hàm khởi tạo của class mailable của bạn và đặt dữ liệu đó thành các thuộc tính công khai được định nghĩa trên class:

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

Khi dữ liệu đã được set thành thuộc tính công khai, nó sẽ tự động có sẵn trong view của bạn, vì vậy bạn có thể truy cập nó giống như bạn truy cập bất kỳ dữ liệu nào khác trong các template Blade của bạn:

    <div>
        Price: {{ $order->price }}
    </div>

#### Via The `with` Method:

Nếu bạn muốn tùy chỉnh định dạng dữ liệu email của bạn trước khi nó được gửi đến template, bạn có thể tự truyền dữ liệu của bạn đến view thông qua phương thức `with`. Thông thường, bạn vẫn sẽ truyền dữ liệu qua hàm khởi tạo của class mailable; tuy nhiên, bạn nên set dữ liệu này thành các thuộc tính `protected` hoặc `private` để những dữ liệu này sẽ không được tự động đưa cho template. Sau đó, khi gọi phương thức `with`, sẽ truyền một mảng dữ liệu mà bạn muốn để đưa cho template:

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

Khi dữ liệu đã được truyền đến phương thức `with`, nó sẽ tự động có sẵn trong view của bạn, vì vậy bạn có thể truy cập nó giống như bạn truy cập bất kỳ dữ liệu nào khác trong các template Blade của bạn:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### Attachments

Để thêm file đính kèm vào email, hãy sử dụng phương thức `attach` trong phương thức `build` của class mailable. Phương thức `attach` chấp nhận một đường dẫn đầy đủ đến file làm tham số đầu tiên của nó:

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

Khi đính kèm file vào một tin nhắn, bạn cũng có thể khai báo tên hiển thị và / hoặc loại MIME bằng cách chuyển một `array` làm tham số thứ hai cho phương thức `attach`:

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

#### Raw Data Attachments

Phương thức `attachData` có thể được sử dụng để đính kèm một chuỗi raw dạng byte dưới dạng file đính kèm. Ví dụ: bạn có thể sử dụng phương thức này nếu bạn đã tạo một file PDF trong bộ nhớ và muốn đính kèm nó vào email mà không ghi nó vào disk. Phương thức `attachData` chấp nhận các byte raw dữ liệu làm tham số đầu tiên của nó, tên của file làm tham số thứ hai và một mảng các tùy chọn làm tham số thứ ba của nó:

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

Nhúng hình ảnh vào trong email của bạn thường cồng kềnh; tuy nhiên, Laravel cung cấp một cách thuận tiện để đính kèm hình ảnh của bạn vào email và lấy CID của hình ảnh đó để hiển thị khi người dùng mở email. Để nhúng hình ảnh, hãy sử dụng phương thức `embed` trên biến `$message` trong template email của bạn. Laravel tự động cung cấp biến `$message` cho tất cả các template email của bạn, vì vậy bạn không cần phải lo lắng về việc truyền nó theo cách thủ công:

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

> {note} biến `$message` không thể sử dụng trong các markdown message.

#### Embedding Raw Data Attachments

Nếu bạn đã có một chuỗi raw dữ liệu mà bạn muốn nhúng vào một template email, bạn có thể sử dụng phương thức `embedData` trên biến `$message`:

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### Tuỳ biến SwiftMailer Message

Phương thức `withSwiftMessage` của class `Mailable` cho phép bạn đăng ký một callback sẽ được gọi với một instance message raw SwiftMailer trước khi gửi message. Điều này cung cấp cho bạn một cách để tùy biến message trước khi nó được gửi:

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

Để tạo một mailable với template Markdown tương ứng, bạn có thể sử dụng tùy chọn `--markdown` của lệnh Artisan `make:mail`:

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

Sau đó, khi cấu hình mailable trong phương thức `build` của nó, hãy gọi phương thức `markdown` thay vì phương thức `view`. Các phương thức `markdown` chấp nhận tên của template Markdown và một mảng dữ liệu tùy chọn để cung cấp cho template:

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

Các markdown mailable sử dụng kết hợp các component của Blade và cú pháp Markdown cho phép bạn dễ dàng tạo ra format mail trong khi vẫn tận dụng các component được tạo sẵn của Laravel:

    @component('mail::message')
    # Order Shipped

    Your order has been shipped!

    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

> {tip} Không sử dụng thụt đầu dòng khi viết email bằng Markdown. Trình phân tích cú pháp Markdown sẽ hiển thị nội dung thụt đầu dòng dưới dạng code block.

#### Button Component

Component button sẽ tạo ra một button link ở chính giữa. Component này chấp nhận hai tham số, một là `url` và một là tùy chọn `color`. Các màu được hỗ trợ là `blue`,`green` và `red`. Bạn có thể thêm các component button vào message nếu muốn:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Order
    @endcomponent

#### Panel Component

Component panel sẽ tạo một block văn bản trong một panel có màu nền hơi khác so với các phần khác của message. Điều này cho phép bạn thu hút sự chú ý của người dùng đến block văn bản:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### Table Component

Component table cho phép bạn chuyển đổi một bảng Markdown thành một bảng HTML. Component này chấp nhận nội dung như một bảng Markdown bình thường. Căn chỉnh trái phải của cột bảng cũng được mặc định hỗ trợ bởi cú pháp căn chỉnh cột bảng của Markdown:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      |      $10 |
    | Col 3 is      | Right-Aligned |      $20 |
    @endcomponent

<a name="customizing-the-components"></a>
### Tuỳ biến Components

Bạn có thể export tất cả các component Markdown mail sang thư mục riêng của bạn để tùy biến. Để export các component, sử dụng lệnh Artisan `vendor:publish` để export tag nội dung `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail

Lệnh này sẽ export các component Markdown mail sang thư mục `resources/views/vendor/mail`. Thư mục `mail` sẽ chứa một thư mục `html` và `markdown`, mỗi thư mục chứa các hiển thị tương ứng của mỗi component có sẵn. Các component trong thư mục `html` được sử dụng để tạo phiên bản HTML cho email của bạn và các bản sao của chúng trong thư mục `markdown` được sử dụng để tạo phiên bản văn bản thuần túy. Bạn có thể tự do tùy biến các component này theo cách bạn muốn.

#### Customizing The CSS

Sau khi export các component, thư mục `resources/views/vendor/mail/html/themes` sẽ chứa file `default.css`. Bạn có thể tùy biến CSS trong file này và các tuỳ biến này của bạn sẽ tự động được nhúng vào trong các hiển thị HTML cho Markdown mail của bạn.

> {tip} Nếu bạn muốn xây dựng một theme hoàn toàn mới cho các component Markdown, hãy viết một file CSS mới trong thư mục `html/themes` và thay đổi tùy chọn `theme` trong file cấu hình `mail` của bạn.

<a name="previewing-mailables-in-the-browser"></a>
## Xem trước Mailables trên Browser

Khi thiết kế một template của mailable, sẽ thật cần thiết để xem trước bản mailable được thiết kế trong trình duyệt của bạn giống như các template Blade thông thường. Vì lý do này, Laravel cho phép bạn trả lại bất kỳ mailable nào trực tiếp từ một Closure route hoặc controller. Khi một mailable được trả lại, nó sẽ được tạo và hiển thị trong trình duyệt, cho phép bạn xem trước thiết kế của nó mà không cần phải gửi nó đến một địa chỉ email thực tế:

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="sending-mail"></a>
## Gửi Mail

Để gửi một message, hãy sử dụng phương thức `to` trong [facade](/docs/{{version}}/facades) `Mail`. Phương thức `to` chấp nhận một địa chỉ email, một instance user hoặc một collection user. Nếu bạn truyền qua một đối tượng hoặc một collection các đối tượng, mailer sẽ tự động sử dụng các thuộc tính `email` và `name` của đối tượng đã được truyền khi set người nhận email, vì vậy hãy đảm bảo các thuộc tính này có tồn tại trên các đối tượng của bạn. Khi bạn đã khai báo người nhận, bạn có thể truyền một instance của class mailable của bạn tới phương thức `send`:

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

Tất nhiên, bạn không bị giới hạn chỉ trong khai báo người nhận "to" khi gửi message. Mà bạn có thể tự do set "to", "cc" và "bcc" cho người nhận, tất cả được kết hợp trong một chuỗi phương thức duy nhất:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### Queueing Mail

#### Queueing A Mail Message

Vì việc gửi email có thể mất nhiều thời gian response của application của bạn, nhiều nhà phát triển sẽ chọn cách queue email message cho background gửi. Laravel làm cho điều này trở nên dễ dàng bằng cách sử dụng [unified queue API](/docs/{{version}}/queues) được tích hợp sẵn của nó. Để queue một mail message, hãy sử dụng phương thức `queue` trên facade `Mail` sau khi khai báo người nhận thư:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

Phương thức này sẽ tự động đảm nhiệm việc tạo một job lên queue để message sẽ được gửi trong background. Tất nhiên, bạn sẽ cần [cấu hình queue](/docs/{{version}}/queues) trước khi sử dụng tính năng này.

#### Delayed Message Queueing

Nếu bạn muốn delay việc gửi thư email trong queue, bạn có thể sử dụng phương thức `later`. Với tham số đầu tiên của nó, phương thức `later` chấp nhận một instance `DateTime` cho biết khi nào thư sẽ được gửi:

    $when = now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### Pushing To Specific Queues

Vì tất cả các class mailable được tạo bằng lệnh `make:mail` đều sử dụng trait `Illuminate\Bus\Queueable`, bạn có thể gọi các phương thức `onQueue` và `onConnection` trong bất kỳ class mailable nào, cho phép bạn khai báo tên kết nối và tên queue cho mail:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### Queueing By Default

Nếu bạn có nhiều class mailable mà bạn muốn luôn được queue, bạn có thể implement contract `ShouldQueue` trên class. Bây giờ, ngay cả khi bạn gọi phương thức `send` khi gửi thư, thì mailable vẫn sẽ được queue vì nó đã implement contract:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## Mail và Local Development

Khi đang phát triển tính năng gửi email, có lẽ bạn không thực sự muốn gửi email đến các địa chỉ email thực sự. Laravel cung cấp một số cách để "vô hiệu hóa" việc gửi email trong quá trình phát triển ở local.

#### Log Driver

Thay vì gửi email của bạn, driver mail `log` sẽ ghi tất cả các email vào file log của bạn để kiểm tra. Để biết thêm thông tin về cách cấu hình application của bạn theo từng môi trường, hãy xem [tài liệu cấu hình](/docs/{{version}}/configuration#environment-configuration).

#### Universal To

Một giải pháp khác được cung cấp bởi Laravel là set một người nhận cho tất cả các email được gửi bởi framework. Bằng cách này, tất cả các email được tạo bởi application của bạn sẽ được gửi đến một địa chỉ cụ thể, thay vì địa chỉ thực sự được khai báo khi gửi mail. Điều này có thể được thực hiện thông qua tùy chọn `to` trong file cấu hình `config/mail.php` của bạn:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

Cuối cùng, bạn có thể sử dụng một service như [Mailtrap](https://mailtrap.io) và driver `smtp` để gửi email của bạn đến hộp thư "giả" nơi bạn có thể xem chúng trong một email client thực sự. Cách tiếp cận này có lợi ích là cho phép bạn thực sự kiểm tra các email cuối cùng trong hòm thư của Mailtrap.

<a name="events"></a>
## Events

Laravel sẽ bắn hai event trong quá trình gửi mail. Event `MessageSending` được kích hoạt trước khi một message được gửi, trong khi event `MessageSent` được kích hoạt sau khi message đã được gửi. Hãy nhớ rằng, những event này được kích hoạt khi thư đang được *gửi*, chứ không phải khi nó đã được queue. Bạn có thể đăng ký một listener event cho event này trong `EventServiceProvider`:

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
