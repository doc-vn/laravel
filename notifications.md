# Notifications

- [Giới thiệu](#introduction)
- [Tạo Notification](#creating-notifications)
- [Gửi Notification](#sending-notifications)
    - [Dùng Notifiable Trait](#using-the-notifiable-trait)
    - [Dùng Notification Facade](#using-the-notification-facade)
    - [Chỉ định Channel sẽ được gửi](#specifying-delivery-channels)
    - [Queue Notification](#queueing-notifications)
    - [On-Demand Notifications](#on-demand-notifications)
- [Mail Notifications](#mail-notifications)
    - [Formatting Mail Messages](#formatting-mail-messages)
    - [Tuỳ biến người gửi](#customizing-the-sender)
    - [Tuỳ biến người nhận](#customizing-the-recipient)
    - [Tuỳ biến chủ đề](#customizing-the-subject)
    - [Tuỳ biến template](#customizing-the-templates)
    - [Xem trước Mail Notification](#previewing-mail-notifications)
- [Markdown Mail Notification](#markdown-mail-notifications)
    - [Tạo Message](#generating-the-message)
    - [Viết Message](#writing-the-message)
    - [Tuỳ biến Component](#customizing-the-components)
- [Database Notifications](#database-notifications)
    - [Yêu cầu](#database-prerequisites)
    - [Formatting Database Notifications](#formatting-database-notifications)
    - [Truy cập Notifications](#accessing-the-notifications)
    - [Đánh dấu đã đọc cho Notification](#marking-notifications-as-read)
- [Broadcast Notifications](#broadcast-notifications)
    - [Yêu cầu](#broadcast-prerequisites)
    - [Formatting Broadcast Notifications](#formatting-broadcast-notifications)
    - [Listening cho Notifications](#listening-for-notifications)
- [SMS Notifications](#sms-notifications)
    - [Yêu cầu](#sms-prerequisites)
    - [Formatting SMS Notifications](#formatting-sms-notifications)
    - [Formatting Shortcode Notifications](#formatting-shortcode-notifications)
    - [Tuỳ biến "From" Number](#customizing-the-from-number)
    - [Routing SMS Notifications](#routing-sms-notifications)
- [Slack Notifications](#slack-notifications)
    - [Yêu cầu](#slack-prerequisites)
    - [Formatting Slack Notifications](#formatting-slack-notifications)
    - [Đính kèm vào message slack](#slack-attachments)
    - [Routing Slack Notifications](#routing-slack-notifications)
- [Ngôn ngữ trong Notifications](#localizing-notifications)
- [Notification Events](#notification-events)
- [Tuỳ biến Channels](#custom-channels)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc hỗ trợ [gửi email](/docs/{{version}}/mail), Laravel cũng hỗ trợ để gửi các notification qua nhiều channel khác nhau, như mail, SMS (qua [Nexmo](https://www.nexmo.com/)) và [Slack](https://slack.com). Notification cũng có thể được lưu trữ vào trong cơ sở dữ liệu của bạn để có thể được hiển thị trong giao diện của người dùng.

Thông thường, notification phải ngắn gọn, nội dung của message phải thông báo cho người dùng biết về điều gì đó đã xảy ra trong application của bạn. Ví dụ: nếu bạn đang viết một application thanh toán, bạn có thể gửi một notification "Thanh toán hóa đơn" cho người dùng của bạn thông qua các channel email và SMS.

<a name="creating-notifications"></a>
## Tạo Notification

Trong Laravel, các notification được đại diện bởi duy nhất một class (thường được lưu trong thư mục `app/Notifications`). Bạn đừng lo lắng nếu bạn không thấy thư mục đó trong application của bạn, vì nó sẽ được tạo khi bạn chạy lệnh Artisan `make:notification`:

    php artisan make:notification InvoicePaid

Lệnh này sẽ tạo một class notification mới vào trong thư mục `app/Notifications` của bạn. Mỗi class notification chứa một phương thức `via` và một số phương thức xây dựng message khác (chẳng hạn như `toMail` hoặc `toDatabase`) để chuyển đổi notification thành một message được tối ưu hóa cho một channel cụ thể.

<a name="sending-notifications"></a>
## Gửi Notification

<a name="using-the-notifiable-trait"></a>
### Dùng Notifiable Trait

Notification có thể được gửi theo hai cách: cách một là sử dụng phương thức `notify` trong trait `Notifiable` hoặc cách hai là sử dụng [facade](/docs/{{version}}/facades) `Notification`. Đầu tiên, hãy xem cách sử dụng trait:

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Trait này được sử dụng mặc định bởi model `App\User` và chứa một phương thức có thể được sử dụng để gửi notification: `notify`. Phương thức `notify` sẽ nhận vào một instance notification:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} Hãy nhớ rằng, bạn có thể sử dụng trait `Illuminate\Notifications\Notifiable` trên bất kỳ model nào mà bạn muốn. Bạn không bị giới hạn dùng nó trên model `User` của bạn.

<a name="using-the-notification-facade"></a>
### Dùng Notification Facade

Ngoài ra, bạn có thể gửi notification thông qua [facade](/docs/{{version}}/facades) `Notification`. Điều này sẽ hữu ích khi bạn cần gửi notification cho nhiều thực thể notifiable, chẳng hạn như một collection user. Để gửi notification bằng facade, hãy truyền tất cả các thực thể notifiable và instance notification sang phương thức `send`:

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### Chỉ định Channel sẽ được gửi

Mỗi class notification có một phương thức `via` định nghĩa channel nào của notification sẽ được gửi. Các notification có thể được gửi trên các channel `mail`, `database`, `broadcast`, `nexmo`, và `slack`.

> {tip} Nếu bạn muốn sử dụng các channel khác như Telegram hoặc Pusher, hãy xem drive do cộng đồng phát triển [Laravel Notification Channels website](http://laravel-notification-channels.com).

Phương thức `via` nhận vào một instance `$notifiable`, đây sẽ là một instance của class mà notification sẽ gửi đến. Bạn có thể sử dụng `$notifiable` để xác định channel nào sẽ gửi notification:

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### Queue Notification

> {note} Trước khi queue notification, bạn nên cấu hình queue và [chạy một worker](/docs/{{version}}/queues).

Gửi notification có thể mất nhiều thời gian, đặc biệt nếu channel cần gọi API bên ngoài để gửi notification. Để tăng tốc độ thời gian phản hồi của application, hãy queue notification của bạn bằng cách thêm interface `ShouldQueue` và trait `Queueable` vào class của bạn. Interface và trait này sẽ mặc định được import cho các notification được tạo ra bằng lệnh `make:notification`, vì vậy bạn có thể ngay lập tức thêm chúng vào trong class notification của bạn:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

Khi interface `ShouldQueue` đã được thêm vào, bạn có thể gửi notification như bình thường. Laravel sẽ phát hiện interface `ShouldQueue` này và tự động queue việc gửi notification:

    $user->notify(new InvoicePaid($invoice));

Nếu bạn muốn delay việc gửi notification, bạn có thể kết hợp với phương thức `delay` vào phần khởi tạo notification của bạn:

    $when = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($when));

<a name="on-demand-notifications"></a>
### On-Demand Notifications

Thỉnh thoảng bạn có thể cần gửi notification cho người mà chưa được lưu trong cơ sở dữ liệu dưới dạng một "user". Sử dụng phương thức `Notification::route`, bạn có thể chỉ định thông tin ad-hoc notification routing trước khi gửi notification:

    Notification::route('mail', 'taylor@example.com')
                ->route('nexmo', '5555555555')
                ->route('slack', 'https://hooks.slack.com/services/...')
                ->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## Mail Notifications

<a name="formatting-mail-messages"></a>
### Formatting Mail Messages

Nếu một notification hỗ trợ gửi dưới dạng email, bạn nên định nghĩa một phương thức `toMail` trong class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Messages\MailMessage`.  Mail message có thể chứa các dòng text cũng như các "call to action". Chúng ta hãy xem một ví dụ về phương thức `toMail`:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} Lưu ý rằng chúng ta đang sử dụng `$this->invoice->id` trong phương thức `toMail`. Bạn có thể truyền bất kỳ dữ liệu nào mà notification của bạn cần để tạo message cho nó bằng hàm khởi tạo của notification.

Trong ví dụ này, chúng ta đã đăng ký một lời chào, một dòng text, một call to action và sau đó là một dòng text khác. Các phương thức này được cung cấp bởi đối tượng `MailMessage` giúp cho việc định dạng các email giao dịch nhỏ trở nên dễ dàng và đơn giản hơn. Sau đó, mail channel sẽ dịch các thành phần của message này thành một template email HTML đẹp có phản hồi nhanh với một bản sao text đơn giản. Đây là một ví dụ mẫu về email được tạo bởi channel `mail`:

<img src="https://laravel.com/img/docs/notification-example.png" width="551" height="596">

> {tip} Khi gửi mail notification, hãy đảm bảo là bạn đã set giá trị `name` trong file cấu hình `config/app.php` của bạn. Giá trị này sẽ được sử dụng trong phần header và footer của message mail notification của bạn.

#### Other Notification Formatting Options

Thay vì định nghĩa "dòng" text trong class notification, bạn có thể sử dụng phương thức `view` để khai báo một template tùy biến để hiển thị email notification:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }

Ngoài ra, bạn có thể trả về một [đối tượng mailable](/docs/{{version}}/mail) từ phương thức `toMail`:

    use App\Mail\InvoicePaid as Mailable;

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($this->user->email);
    }

<a name="error-messages"></a>
#### Error Messages

Một số notification thông báo cho người dùng về các lỗi, chẳng hạn như thanh toán hóa đơn không thành công. Bạn có thể tạo một mail message liên quan đến lỗi đó bằng cách gọi phương thức `error` khi xây dựng message của bạn. Khi sử dụng phương thức `error` trên mail message, button call to action sẽ có màu đỏ thay vì màu xanh nước biển như thường lệ:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-sender"></a>
### Tuỳ biến người gửi

Mặc định, địa chỉ người gửi hoặc từ địa chỉ email được định nghĩa trong file cấu hình `config/mail.php`. Tuy nhiên, bạn có thể chỉ định một địa chỉ from cho một notification cụ thể bằng cách sử dụng phương thức `from`:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->from('test@example.com', 'Example')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### Tuỳ biến người nhận

Khi gửi notifications qua channel `mail`, hệ thống notification sẽ tự động tìm kiếm thuộc tính `email` trong thực thể notifiable của bạn. Bạn có thể tùy biến địa chỉ email nào sẽ được sử dụng để gửi notification bằng cách định nghĩa phương thức `routeNotificationForMail` trên thực thể đó:

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the mail channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return array|string
         */
        public function routeNotificationForMail($notification)
        {
            // Return email address only...
            return $this->email_address;

            // Return name and email address...
            return [$this->email_address => $this->name];
        }
    }

<a name="customizing-the-subject"></a>
### Tuỳ biến chủ đề

Mặc định, chủ đề của email là tên class của notification được định dạng theo dạng "title case". Vì vậy, nếu class notification của bạn được đặt tên là `InvoicePaid`, thì chủ đề của email sẽ là `Invoice Paid`. Nếu bạn muốn chỉ định một chủ đề rõ ràng hơn cho message, bạn có thể gọi phương thức `subject` khi xây dựng message của bạn:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### Tuỳ biến template

Bạn có thể sửa HTML và template được sử dụng bởi mail notification bằng cách export resources của package notification. Sau khi chạy lệnh này, các template mail notification sẽ được lưu ở trong thư mục `resources/views/vendor/notifications`:

    php artisan vendor:publish --tag=laravel-notifications

<a name="previewing-mail-notifications"></a>
### Xem trước Mail Notification

Khi thiết kế một template mail notification, sẽ tiện lợi hơn nếu xem trước được một mail notification được tạo ra trong trình duyệt của bạn giống như một template Blade điển hình. Vì lý do này, Laravel cho phép bạn trả về bất kỳ mail notification nào được tạo ra bởi mail notification trực tiếp từ một route Closure hoặc một controller. Khi một `MailMessage` được trả về, nó sẽ được tạo và hiển thị trong trình duyệt, cho phép bạn xem trước các thiết kế mà không cần gửi nó đến một địa chỉ email thực tế:

    Route::get('mail', function () {
        $invoice = App\Invoice::find(1);

        return (new App\Notifications\InvoicePaid($invoice))
                    ->toMail($invoice->user);
    });

<a name="markdown-mail-notifications"></a>
## Markdown Mail Notification

Markdown mail notification cho phép bạn tận dụng các template mail notification được xây dựng sẵn, đồng thời cho bạn tự do hơn để viết các tùy biến message dài hơn. Vì các message được viết bằng Markdown, nên Laravel có thể hiển thị các template HTML đẹp đáp ứng cho các message, đồng thời tự động tạo một bản sao text đơn giản.

<a name="generating-the-message"></a>
### Tạo Message

Để tạo một notification với một template Markdown, bạn có thể sử dụng tùy chọn `--markdown` trong lệnh Artisan `make:notification`:

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid

Giống như tất cả các mail notification khác, các notification sử dụng bởi các template Markdown sẽ định nghĩa một phương thức `toMail` trong class notification của chúng. Tuy nhiên, thay vì sử dụng các phương thức `line` và `action` để khởi tạo cho notification, hãy sử dụng phương thức `markdown` để khai báo tên của template Markdown được sử dụng:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### Viết Message

Markdown mail notification sử dụng kết hợp giữa các component Blade và cú pháp Markdown cho phép bạn dễ dàng khởi tạo notification trong khi vẫn tận dụng được các component notification được tạo sẵn của Laravel:

    @component('mail::message')
    # Invoice Paid

    Your invoice has been paid!

    @component('mail::button', ['url' => $url])
    View Invoice
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

#### Button Component

Component button sẽ tạo một button được đặt ở chính giữa của trang. Component này chấp nhận hai tham số, một là `url` và một là tùy chọn `color`. Các màu được hỗ trợ là `blue`, `green` và `red`. Bạn có thể thêm các button component vào một notification nếu muốn:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Invoice
    @endcomponent

#### Panel Component

Component panel sẽ tạo một block text trong một panel có màu nền hơi khác so với các phần khác của notification. Điều này cho phép bạn thu hút sự chú ý của người dùng đến block text:

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
### Tuỳ biến The Compoents

Bạn có thể export tất cả các component Markdown mail sang một thư mục riêng của bạn để tùy chỉnh. Để export các component này, hãy sử dụng lệnh Artisan `vendor:publish` để export với nội dung tag `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail

Lệnh này sẽ export các component Markdown mail sang thư mục `resources/views/vendor/mail`. Thư mục `mail` sẽ chứa một thư mục là `html` và một thư mục là `text`, mỗi thư mục chứa các hiển thị tương ứng cho mỗi component có sẵn. Các component trong thư mục `html` được sử dụng để tạo phiên bản HTML cho email và các bản sao của chúng còn trong thư mục `text` được sử dụng để tạo các phiên bản text thuần túy. Bạn có thể tự do tùy chỉnh các component này theo cách mà bạn muốn.

#### Customizing The CSS

Sau khi export các component, thư mục `resources/views/vendor/mail/html/themes` sẽ chứa một file `default.css`. Bạn có thể tùy chỉnh CSS trong file này và các tuỳ chỉnh của bạn sẽ tự động được nhúng vào trong các hiển thị HTML của Markdown notification của bạn.

Nếu bạn muốn xây dựng một theme mới cho các component Markdown của Laravel, bạn có thể tạo một file CSS mới trong thư mục `html/themes`. Sau khi tạo tên và lưu file CSS của bạn, hãy cập nhật tùy chọn `theme` trong file cấu hình `mail` để khớp với tên theme mới của bạn.

Để tùy chỉnh theme cho một notification riêng lẻ, bạn có thể gọi phương thức `theme` trong khi xây dựng mail message notification. Phương thức `theme` chấp nhận tên của theme sẽ được sử dụng khi gửi notification:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->theme('invoice')
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="database-notifications"></a>
## Database Notifications

<a name="database-prerequisites"></a>
### Yêu cầu

Channel notification `database` sẽ lưu trữ thông tin notification vào trong bảng của cơ sở dữ liệu. Bảng này sẽ chứa thông tin như loại notification cũng như dữ liệu JSON mô tả của notification đó.

Bạn có thể truy vấn vào bảng để hiển thị các notification trong giao diện người dùng của application. Nhưng, trước khi bạn có thể làm điều đó, bạn sẽ cần phải tạo một bảng để lưu các notification của bạn. Bạn có thể sử dụng lệnh `notifications:table` để tạo một migration với một table schema thích hợp:

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### Formatting Database Notifications

Nếu một notification hỗ trợ lưu trữ trong bảng cơ sở dữ liệu, thì bạn nên định nghĩa một phương thức `toDatabase` hoặc `toArray` trong class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một mảng PHP. Mảng được trả về sẽ được mã hóa dưới dạng JSON và được lưu trữ vào trong cột `data` trong bảng `notifications`. Bạn hãy xem một ví dụ về phương thức `toArray`:

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

#### `toDatabase` và `toArray`

Phương thức `toArray` cũng được sử dụng bởi channel `broadcast` để xác định xem dữ liệu nào sẽ phát đến JavaScript client. Nhưng nếu bạn muốn biểu thị hai mảng khác nhau cho hai channel `database` và `broadcast` khác nhau, thì bạn nên định nghĩa bằng phương thức `toDatabase` thay vì phương thức `toArray`.

<a name="accessing-the-notifications"></a>
### Truy cập Notifications

Khi notification đã được lưu vào trong cơ sở dữ liệu, bạn cần một cách để truy cập vào các notification từ các thực thể notifiable của bạn. Mặc định trait `Illuminate\Notifications\Notifiable` đã được khai báo trong model `App\User` mặc định đi kèm với Laravel, nó có chứa một quan hệ Eloquent là `notifications` sẽ trả về các notification cho các thực thể. Để lấy notification, bạn có thể truy cập phương thức này giống như bất kỳ phương thức quan hệ Eloquent nào khác. Mặc định, các notification sẽ được sắp xếp theo thời gian timestamp `created_at`:

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Nếu bạn chỉ muốn lấy các notification "chưa đọc", bạn có thể sử dụng quan hệ `unreadNotifications`. Một lần nữa, các notification này sẽ được sắp xếp theo thời gian timestamp `created_at`:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} Để truy cập vào notification của bạn từ JavaScript client, bạn nên định nghĩa một notification controller riêng cho application của bạn để trả về notification cho một thực thể notifiable, chẳng hạn như người dùng hiện tại. Sau đó, bạn hãy tạo một HTTP request đến URI của controller đó từ JavaScript client của bạn.

<a name="marking-notifications-as-read"></a>
### Đánh dấu đã đọc cho Notification

Thông thường, bạn sẽ muốn đánh dấu một notification là "đã đọc" khi người dùng đã xem nó. Trait `Illuminate\Notifications\Notifiable` cũng sẽ cung cấp một phương thức `markAsRead` để cập nhật cột `read_at` trong record database của notification đó:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Tuy nhiên, thay vì lặp từng notification, bạn có thể sử dụng phương thức `markAsRead` trực tiếp trên một collection của notification:

    $user->unreadNotifications->markAsRead();

Bạn cũng có thể sử dụng cập nhật hàng loạt để đánh dấu tất cả các notification là đã đọc mà không cần phải lấy chúng ra khỏi cơ sở dữ liệu:

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

Bạn cũng có thể 'xóa' các notification này, để xóa chúng ra khỏi bảng bạn có thể làm như sau:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Broadcast Notifications

<a name="broadcast-prerequisites"></a>
### Yêu cầu

Trước khi broadcasting notification, bạn nên cấu hình và làm quen với các service [event broadcasting](/docs/{{version}}/broadcasting) của Laravel. Event broadcasting cung cấp một cách phù hợp để tương tác với các event Laravel do phía server tạo ra, từ JavaScript client của bạn.

<a name="formatting-broadcast-notifications"></a>
### Formatting Broadcast Notifications

Channel `broadcast` của broadcasts notification sẽ dùng các service [event broadcasting](/docs/{{version}}/broadcasting) của Laravel, cho phép JavaScript client của bạn nhận được các notification theo thời gian thực. Nếu một notification hỗ trợ broadcasting, bạn có thể định nghĩa phương thức `toBroadcast` trên class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `BroadcastMessage`. Nếu phương thức `toBroadcast` không tồn tại, phương thức `toArray` sẽ được sử dụng để thu thập dữ liệu cần được broadcast. Dữ liệu được trả về sẽ được mã hóa dưới dạng JSON và broadcast đến JavaScript client của bạn. Chúng ta hãy xem một ví dụ về phương thức `toBroadcast`:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Get the broadcastable representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

#### Broadcast Queue Configuration

Tất cả các broadcast notification sẽ được queue lại để broadcasting. Nếu bạn muốn cấu hình queue connection hoặc tên queue được sử dụng để queue lại broadcast, bạn có thể sử dụng các phương thức `onConnection` và `onQueue` của `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

> {tip} Ngoài dữ liệu bạn khai báo, broadcast notification cũng sẽ chứa một trường `type` sẽ được để chứa tên class của notification.

<a name="listening-for-notifications"></a>
### Listening cho Notifications

Notification sẽ được broadcast trên các private channel được định danh theo cách sử dụng quy ước `{notifiable}.{id}`. Vì vậy, nếu bạn đang gửi notification đến một instance `App\User` có ID là `1`, thì notification sẽ được broadcast trên private channel là `App.User.1`. Khi sử dụng [Laravel Echo](/docs/{{version}}/broadcasting), bạn có thể dễ dàng listen cho các notification trên channel này bằng phương thức helper `notification`:

    Echo.private('App.User.' + userId)Z
        .notification((notification) => {
            console.log(notification.type);
        });

#### Customizing The Notification Channel

Nếu bạn muốn tùy chỉnh channel nhận thực thể notifiable của broadcast notification, bạn có thể định nghĩa một phương thức `receivesBroadcastNotificationsOn` trên thực thể notifiable:

    <?php

    namespace App;

    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The channels the user receives notification broadcasts on.
         *
         * @return string
         */
        public function receivesBroadcastNotificationsOn()
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## SMS Notifications

<a name="sms-prerequisites"></a>
### Yêu cầu

Gửi notification SMS trong Laravel được mặc định cung cấp bởi [Nexmo](https://www.nexmo.com/). Trước khi bạn có thể gửi notification qua Nexmo, bạn cần phải cài đặt package `laravel/nexmo-notification-channel` thông qua Composer.

    composer require laravel/nexmo-notification-channel

Thao tác này cũng sẽ cài đặt package [`nexmo/laravel`](https://github.com/Nexmo/nexmo-laravel). Package này cũng chứa một [file cấu hình của riêng nó](https://github.com/Nexmo/nexmo-laravel/blob/master/config/nexmo.php). Bạn có thể sử dụng các biến môi trường `NEXMO_KEY` và` NEXMO_SECRET` để set khóa bí mật và công khai Nexmo của bạn.

Tiếp theo, bạn sẽ cần thêm một tùy chọn cấu hình vào file cấu hình `config/services.php` của bạn. Bạn có thể copy cấu hình mẫu ở bên dưới để bắt đầu:

    'nexmo' => [
        'sms_from' => '15556666666',
    ],

Tùy chọn `sms_from` là số điện thoại mà tin nhắn SMS của bạn sẽ được gửi. Bạn nên tạo số điện thoại cho application của bạn trong bảng điều khiển Nexmo.

<a name="formatting-sms-notifications"></a>
### Formatting SMS Notifications

Nếu một notification hỗ trợ gửi dưới dạng SMS, bạn nên định nghĩa phương thức `toNexmo` trên class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Messages\NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

<a name="formatting-shortcode-notifications"></a>
### Formatting Shortcode Notifications

Laravel cũng hỗ trợ gửi các mã thông báo ngắn, là các template tin nhắn được định nghĩa trước trong tài khoản Nexmo của bạn. Bạn có thể chỉ định loại thông báo (`alert`, `2fa` hoặc `marketing`), cũng như các giá trị tùy chỉnh sẽ được điền vào template:

    /**
     * Get the Nexmo / Shortcode representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toShortcode($notifiable)
    {
        return [
            'type' => 'alert',
            'custom' => [
                'code' => 'ABC123',
            ];
        ];
    }

> {tip} Giống như [routing thông báo SMS](#routing-sms-notifications), bạn nên implement phương thức `routeNotificationForShortcode` trên model thông báo của bạn.

#### Unicode Content

Nếu tin nhắn SMS của bạn sẽ chứa các ký tự unicode, bạn nên gọi phương thức `unicode` khi khởi tạo instance `NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### Tuỳ biến "From" Number

Nếu bạn muốn gửi một số notification từ một số điện thoại khác với số điện thoại đã được khai báo trong file `config/services.php` của bạn, bạn có thể sử dụng phương thức `from` trên một instance `NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>
### Routing SMS Notifications

Để route thông báo Nexmo đến một số điện thoại thích hợp, hãy định nghĩa phương thức `routeNotificationForNexmo` trên model notifiable của bạn:

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Nexmo channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForNexmo($notification)
        {
            return $this->phone_number;
        }
    }

<a name="slack-notifications"></a>
## Slack Notifications

<a name="slack-prerequisites"></a>
### Yêu cầu

Trước khi bạn có thể gửi notification qua Slack, bạn phải cài đặt notification channel thông qua Composer:

    composer require laravel/slack-notification-channel

Bạn cũng sẽ cần cấu hình ["Incoming Webhook"](https://api.slack.com/incoming-webhooks) cho group Slack của bạn. Việc cấu hình này sẽ cung cấp cho bạn một URL mà bạn có thể sử dụng khi [routing Slack notifications](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>
### Formatting Slack Notifications

Nếu một notification hỗ trợ gửi dưới dạng message của Slack, bạn nên định nghĩa phương thức `toSlack` trên class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Messages\SlackMessage`. Message Slack có thể có chứa nội dung text cũng như "đính kèm" thêm một định dạng text hoặc một mảng các trường. Chúng ta hãy xem một ví dụ `toSlack` cơ bản:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

Trong ví dụ này, chúng ta chỉ gửi một dòng text tới Slack, điều này sẽ tạo ra một notification giống như sau:

<img src="https://laravel.com/img/docs/basic-slack-notification.png">

#### Customizing The Sender & Recipient

Bạn có thể sử dụng các phương thức `from` và `to` để tùy chỉnh người gửi và người nhận. Phương thức `from` chấp nhận tên người dùng và biểu tượng cảm xúc, trong khi phương thức `to` chấp nhận tên một channel hoặc tên người dùng:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('This will be sent to #other');
    }

Bạn cũng có thể sử dụng hình ảnh làm logo thay vì biểu tượng cảm xúc:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Laravel')
                    ->image('https://laravel.com/img/favicon/favicon.ico')
                    ->content('This will display the Laravel logo next to the message');
    }

<a name="slack-attachments"></a>
### Đính kèm vào message slack

Bạn cũng có thể "đính kèm" thêm thông tin vào tin nhắn Slack. Đính kèm này cung cấp các tùy chọn định dạng phong phú hơn các tin nhắn text bình thường. Trong ví dụ này, chúng ta sẽ gửi notification lỗi về một ngoại lệ xảy ra trong application, chứa một link liên kết để xem chi tiết hơn về ngoại lệ:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

Ví dụ trên sẽ tạo ra một notification Slack trông giống như sau:

<img src="https://laravel.com/img/docs/basic-slack-attachment.png">

Đính kèm này cũng cho phép bạn khai báo một mảng dữ liệu sẽ được hiển thị cho người dùng. Dữ liệu này sẽ được hiển thị theo định dạng bảng để dễ đọc hơn:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

Ví dụ trên sẽ tạo ra một notification Slack trông giống như sau:

<img src="https://laravel.com/img/docs/slack-fields-attachment.png">

#### Markdown Attachment Content

Nếu một số trường đính kèm của bạn chứa Markdown, bạn có thể sử dụng phương thức `markdown` để bảo Slack phân tích cú pháp và hiển thị các trường đính kèm dưới dạng văn bản được định dạng theo kiểu Markdown. Các giá trị được phương thức này chấp nhận là: `pretext`, `text` hoặc `fields`. Để biết thêm thông tin về định dạng đính kèm Slack, hãy xem [Tài liệu API Slack](https://api.slack.com/docs/message-formatting#message_formatting):

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was *not found*.')
                                   ->markdown(['text']);
                    });
    }

<a name="routing-slack-notifications"></a>
### Routing Slack Notifications

Để route Slack notification đến một vị trí, hãy định nghĩa phương thức `routeNotificationForSlack` trên thực thể notifiable của bạn. Điều này sẽ trả về một URL webhook mà notification sẽ được gửi tới đó. URL webhook có thể được tạo ra bằng cách thêm một "Incoming Webhook" vào group Slack của bạn:

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForSlack($notification)
        {
            return 'https://hooks.slack.com/services/...';
        }
    }

<a name="localizing-notifications"></a>
## Ngôn ngữ trong Notifications

Laravel cho phép bạn gửi các notification bằng ngôn ngữ khác, khác với ngôn ngữ hiện tại của application và thậm chí sẽ nhớ ngôn ngữ này nếu notification đang được queue.

Để thực hiện điều này, class `Illuminate\Notifications\Notification` có cung cấp một phương thức `locale` để set ngôn ngữ mà bạn mong muốn. Application sẽ chuyển đổi thành ngôn ngữ này khi định dạng notification và sau đó quay lại về ngôn ngữ trước đó khi quá trình định dạng này hoàn tất:

    $user->notify((new InvoicePaid($invoice))->locale('es'));

Set ngôn ngữ cho nhiều notification cũng có thể đạt được thông qua facade `Notification`:

    Notification::locale('es')->send($users, new InvoicePaid($invoice));

### User Preferred Locales

Thỉnh thoảng, các application sẽ lưu lại ngôn ngữ ưa thích của mỗi người dùng. Bằng cách implement contract `HasLocalePreference` trên một model notifiable của bạn, bạn có thể hướng dẫn Laravel sử dụng ngôn ngữ này khi gửi notification:

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

Khi bạn đã implement xong interface này, Laravel sẽ tự động sử dụng ngôn ngữ này khi gửi notification và mailable tới model. Do đó, không cần phải gọi phương thức `locale` khi bạn sử dụng interface này:

    $user->notify(new InvoicePaid($invoice));

<a name="notification-events"></a>
## Notification Events

Khi một notification đã được gửi, event `Illuminate\Notifications\Events\NotificationSent` sẽ được kích hoạt bởi notification system. Nó sẽ chứa thực thể "notifiable" và một instance notification. Bạn có thể đăng ký listener cho các event này trong `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} Sau khi đăng ký listener trong `EventServiceProvider`, hãy sử dụng lệnh Artisan `event:generate` để tạo ra các class listener.

Trong một event listener, bạn có thể truy cập vào các thuộc tính `notifiable`, `notification` và `channel` trong event để biết thêm thông tin về người nhận notification hoặc chính notification đó:

    /**
     * Handle the event.
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
        // $event->response
    }

<a name="custom-channels"></a>
## Tuỳ biến Channels

Laravel có sẵn với một số notification channel, nhưng bạn có thể muốn viết thêm các driver khác để gửi notification qua các channel riêng của bạn. Laravel làm cho nó trở nên rất đơn giản. Để bắt đầu, hãy định nghĩa một class có chứa phương thức `send`. Phương thức sẽ nhận vào hai tham số: một là `$notifiable` và một là `$notification`:

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Send the given notification.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // Send notification to the $notifiable instance...
        }
    }

Khi class notification channel của bạn đã được định nghĩa, bạn có thể trả về tên class từ phương thức `via` của bất kỳ notifications nào của bạn:

    <?php

    namespace App\Notifications;

    use App\Channels\Messages\VoiceMessage;
    use App\Channels\VoiceChannel;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Get the notification channels.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * Get the voice representation of the notification.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
