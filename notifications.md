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
    - [Tuỳ biến người nhận](#customizing-the-recipient)
    - [Tuỳ biến chủ đề](#customizing-the-subject)
    - [Tuỳ biến template](#customizing-the-templates)
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
    - [Tuỳ biến "From" Number](#customizing-the-from-number)
    - [Routing SMS Notifications](#routing-sms-notifications)
- [Slack Notifications](#slack-notifications)
    - [Yêu cầu](#slack-prerequisites)
    - [Formatting Slack Notifications](#formatting-slack-notifications)
    - [Đính kèm vào message slack](#slack-attachments)
    - [Routing Slack Notifications](#routing-slack-notifications)
- [Notification Events](#notification-events)
- [Tuỳ biến Channels](#custom-channels)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc hỗ trợ [gửi email](/docs/{{version}}/mail), Laravel cũng hỗ trợ để gửi các notification qua nhiều channel khác nhau, như mail, SMS (qua [Nexmo](https://www.nexmo.com/)) và [Slack](https://slack.com). Notification cũng có thể được lưu trữ trong cơ sở dữ liệu để có thể được hiển thị trong giao diện web của bạn.

Thông thường, notification phải ngắn gọn, nội dung của message phải notification cho người dùng biết về điều gì đó đã xảy ra trong application của bạn. Ví dụ: nếu bạn đang viết một application thanh toán, bạn có thể gửi notification "Thanh toán hóa đơn" cho người dùng của bạn biết thông qua các channel email và SMS.

<a name="creating-notifications"></a>
## Tạo Notification

Trong Laravel, mỗi notification được đại diện bởi một class duy nhất (thường được lưu trữ trong thư mục `app/Notifications`). Đừng lo lắng nếu bạn không thấy thư mục này trong application của bạn, vì nó sẽ được tạo cho bạn khi bạn chạy lệnh Artisan `make:notification`:

    php artisan make:notification InvoicePaid

Lệnh này sẽ lưu một class notification mới vào trong thư mục `app/Notifications` của bạn. Mỗi class notification chứa một phương thức `via` và một số phương thức xây dựng message khác nhau (chẳng hạn như `toMail` hoặc `toDatabase`) để chuyển đổi notification thành một message được tối ưu hóa cho một channel cụ thể nào đó.

<a name="sending-notifications"></a>
## Gửi Notification

<a name="using-the-notifiable-trait"></a>
### Dùng Notifiable Trait

Notification có thể được gửi theo hai cách: cách một là sử dụng phương thức `notify` của trait `Notifiable` hoặc cách hai là sử dụng [facade](/docs/{{version}}/facades) `Notification`. Đầu tiên, hãy xem cách sử dụng trait:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Trait này được sử dụng mặc định bởi model `App\User` và chứa một phương thức có thể được sử dụng để gửi notification: `notify`. Phương thức `notify` sẽ nhận vào một instance notification:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} Hãy nhớ rằng, bạn có thể sử dụng trait `Illuminate\Notifications\Notifiable` trên bất kỳ model nào khác của bạn. Bạn không bị giới hạn chỉ dùng nó trên model `User` của bạn.

<a name="using-the-notification-facade"></a>
### Dùng Notification Facade

Ngoài ra, bạn có thể gửi notification thông qua [facade](/docs/{{version}}/facades) `Notification`. Điều này chủ yếu hữu ích khi bạn cần gửi notification cho nhiều thực thể notifiable, chẳng hạn như một collection user. Để gửi notification bằng facade, hãy truyền tất cả các thực thể notifiable và instance notification sang phương thức `send`:

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### Chỉ định Channel sẽ được gửi

Mỗi class notification có một phương thức `via` xác định channel nào notification sẽ được gửi. Mặc định, các notification có thể được gửi trên các channel `mail`, `database`, `broadcast`, `nexmo`, và `slack`.

> {tip} Nếu bạn muốn sử dụng các channel phân phối khác như Telegram hoặc Pusher, hãy xem drive do cộng đồng phát triển [Laravel Notification Channels website](http://laravel-notification-channels.com).

Phương thức `via` nhận vào một instance `$notifiable`, đây sẽ là một instance của class mà notification sẽ được gửi đến. Bạn có thể sử dụng `$notifiable` để xác định channel nào sẽ được gửi notification:

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

> {note} Trước khi queue notification, bạn nên cấu hình queue của bạn và [start một worker](/docs/{{version}}/queues).

Gửi notification có thể mất nhiều thời gian, đặc biệt nếu channel cần call API bên ngoài để gửi notification. Để tăng tốc thời gian phản hồi của application, hãy để notification của bạn được queue bằng cách thêm interface `ShouldQueue` và trait `Queueable` vào class của bạn. Interface và trait sẽ được import cho tất cả các notification được tạo ra bằng cách sử dụng `make:notification`, vì vậy bạn có thể ngay lập tức thêm chúng vào class notification của bạn:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

Khi interface `ShouldQueue` đã được thêm vào notification của bạn, bạn có thể gửi notification như bình thường. Laravel sẽ phát hiện interface `ShouldQueue` trên class notification đó và tự động queue việc gửi notification:

    $user->notify(new InvoicePaid($invoice));

Nếu bạn muốn delay việc gửi notification, bạn có thể kết hợp với phương thức `delay` vào phần khởi tạo notification của bạn:

    $when = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($when));

<a name="on-demand-notifications"></a>
### On-Demand Notifications

Thỉnh thoảng bạn có thể cần gửi notification cho người mà chưa được lưu trữ dưới dạng một "user" trong application của bạn. Sử dụng phương thức `Notification::route`, bạn có thể chỉ định thông tin ad-hoc notification routing trước khi gửi notification:

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## Mail Notifications

<a name="formatting-mail-messages"></a>
### Formatting Mail Messages

Nếu một notification hỗ trợ gửi dưới dạng email, bạn nên định nghĩa một phương thức `toMail` trong class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Messages\MailMessage`.  Mail message có thể chứa các dòng text cũng như "call to action". Chúng ta hãy xem một ví dụ về phương thức `toMail`:

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

> {tip} Lưu ý rằng chúng ta đang sử dụng `$this->invoice->id` trong phương thức `toMail` của chúng ta. Bạn có thể truyền bất kỳ dữ liệu nào mà notification của bạn cần để tạo message cho nó bằng hàm khởi tạo của notification.

Trong ví dụ này, chúng ta đăng ký một lời chào, một dòng text, một call to action và sau đó là một dòng text khác. Các phương thức được cung cấp bởi đối tượng `MailMessage` giúp việc định dạng các email giao dịch nhỏ trở nên dễ dàng và đơn giản hơn. Sau đó, mail channel sẽ dịch các thành phần của message thành một template email HTML đẹp, phản hồi nhanh với một bản sao text đơn giản. Đây là một ví dụ về một email được tạo bởi channel `mail`:

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596">

> {tip} Khi gửi mail notification, hãy đảm bảo bạn đã set giá trị `name` trong file cấu hình `config/app.php` của bạn. Giá trị này sẽ được sử dụng trong phần header và footer của message mail notification của bạn.

#### Other Notification Formatting Options

Thay vì định nghĩa "dòng" text trong class notification, bạn có thể sử dụng phương thức `view` để khai báo một template tùy biến được sử dụng, để hiển thị email notification:

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

Một số notification thông báo cho người dùng về các lỗi, chẳng hạn như thanh toán hóa đơn không thành công. Bạn có thể tạo một mail message liên quan đến lỗi đó bằng cách gọi phương thức `error` khi xây dựng message của bạn. Khi sử dụng phương thức `error` trên mail message, button call to action sẽ có màu đỏ thay vì màu xanh nước biển:

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

<a name="customizing-the-recipient"></a>
### Tuỳ biến người nhận

Khi gửi notifications qua channel `mail`, hệ thống notification sẽ tự động tìm kiếm thuộc tính `email` trong thực thể notifiable của bạn. Bạn có thể tùy chỉnh địa chỉ email nào được sử dụng để gửi notification bằng cách định nghĩa phương thức `routeNotificationForMail` trên thực thể:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the mail channel.
         *
         * @return string
         */
        public function routeNotificationForMail()
        {
            return $this->email_address;
        }
    }

<a name="customizing-the-subject"></a>
### Tuỳ biến chủ đề

Mặc định, chủ đề của email là tên class của notification được định dạng theo dạng "title case". Vì vậy, nếu class notification của bạn được đặt tên là `InvoicePaid`, chủ đề của email sẽ là `Invoice Paid`. Nếu bạn muốn chỉ định một chủ đề rõ ràng hơn cho message, bạn có thể gọi phương thức `subject` khi xây dựng message của bạn:

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

Bạn có thể sửa HTML và template được sử dụng bởi mail notification bằng cách export resources của package notification. Sau khi chạy lệnh này, các template mail notification sẽ được lưu vào trong thư mục `resources/views/vendor/notifications`:

    php artisan vendor:publish --tag=laravel-notifications

<a name="markdown-mail-notifications"></a>
## Markdown Mail Notification

Markdown mail notification cho phép bạn tận dụng các template mail notification được xây dựng sẵn, đồng thời cho bạn tự do hơn để viết các tùy biến message dài hơn. Vì các message được viết bằng Markdown, nên Laravel có thể hiển thị các template HTML đẹp, đáp ứng cho các message, đồng thời tự động tạo một bản sao text đơn giản.

<a name="generating-the-message"></a>
### Tạo Message

Để tạo một notification với template Markdown, bạn có thể sử dụng tùy chọn `--markdown` trong lệnh Artisan `make:notification`:

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid

Giống như tất cả các mail notification khác, các notification sử dụng các template Markdown sẽ định nghĩa một phương thức `toMail` trong class notification của chúng. Tuy nhiên, thay vì sử dụng các phương thức `line` và `action` để khởi tạo notification, hãy sử dụng phương thức `markdown` để khai báo tên của template Markdown được sử dụng:

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

Markdown mail notification sử dụng kết hợp giữa các component Blade và cú pháp Markdown cho phép bạn dễ dàng khởi tạo notification trong khi tận dụng các component notification được tạo sẵn của Laravel:

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

Component button sẽ tạo một button link được đặt ở chính giữa trang. Component này chấp nhận hai tham số, một là `url` và một tùy chọn `color`. Các màu được hỗ trợ là `blue`, `green` và `red`. Bạn có thể thêm các button component vào một notification nếu muốn:

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

Bạn có thể export tất cả các component Markdown mail sang một thư mục riêng của bạn để tùy chỉnh. Để export các component, sử dụng lệnh Artisan `vendor:publish` để export thẻ nội dung `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail

Lệnh này sẽ export các component Markdown mail sang thư mục `resources/views/vendor/mail`. Thư mục `mail` sẽ chứa một thư mục `html` và `markdown`, mỗi thư mục chứa các hiển thị tương ứng cho mỗi component có sẵn. Các component trong thư mục `html` được sử dụng để tạo phiên bản HTML cho email của bạn và các bản sao của chúng trong thư mục `markdown` được sử dụng để tạo phiên bản text thuần túy. Bạn có thể tự do tùy chỉnh các component này theo cách bạn muốn.

#### Customizing The CSS

Sau khi export các component, thư mục `resources/views/vendor/mail/html/themes` sẽ chứa file `default.css`. Bạn có thể tùy chỉnh CSS trong file này và các tuỳ chỉnh của bạn sẽ tự động được nhúng vào trong các hiển thị HTML của Markdown notification của bạn.

> {tip} Nếu bạn muốn xây dựng một theme hoàn toàn mới cho các component Markdown, hãy viết một file CSS mới trong thư mục `html/themes` và thay đổi tùy chọn `theme` trong file cấu hình `mail` của bạn.

<a name="database-notifications"></a>
## Database Notifications

<a name="database-prerequisites"></a>
### Yêu cầu

Channel notification `database` lưu trữ thông tin notification trong bảng cơ sở dữ liệu. Bảng này sẽ chứa thông tin như loại notification cũng như dữ liệu JSON mô tả notification.

Bạn có thể truy vấn bảng để hiển thị các notification trong giao diện người dùng của application. Nhưng, trước khi bạn có thể làm điều đó, bạn sẽ cần tạo một bảng cơ sở dữ liệu để giữ các notification của bạn. Bạn có thể sử dụng lệnh `notifications:table` để tạo một migration với table schema thích hợp:

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### Formatting Database Notifications

Nếu một notification hỗ trợ lưu trữ trong bảng cơ sở dữ liệu, bạn nên định nghĩa một phương thức `toDatabase` hoặc `toArray` trong class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một mảng PHP. Mảng được trả về sẽ được mã hóa dưới dạng JSON và được lưu trữ vào trong cột `data` trong bảng `notifications` của bạn. Chúng ta hãy xem một ví dụ về phương thức `toArray`:

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

Phương thức `toArray` cũng được sử dụng bởi channel `broadcast` để xác định xem dữ liệu nào sẽ phát đến JavaScript client của bạn. Như nếu bạn muốn biểu thị hai mảng khác nhau cho các channel `database` và `broadcast`, bạn nên định nghĩa phương thức `toDatabase` thay vì phương thức `toArray`.

<a name="accessing-the-notifications"></a>
### Truy cập Notifications

Khi notification đã được lưu trữ trong cơ sở dữ liệu, bạn cần một cách thuận tiện để truy cập vào chúng từ các thực thể notifiable của bạn. Trait `Illuminate\Notifications\Notifiable`, được chứa trong model `App\User` mặc định của Laravel, có chứa một quan hệ Eloquent là `notifications` sẽ trả về các notification cho thực thể. Để lấy notification, bạn có thể truy cập phương thức này giống như bất kỳ quan hệ Eloquent nào khác. Mặc định, các notification sẽ được sắp xếp theo timestamp `created_at`:

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Nếu bạn chỉ muốn lấy các notification "chưa đọc", bạn có thể sử dụng quan hệ `unreadNotifications`. Một lần nữa, các notification này sẽ được sắp xếp theo timestamp `created_at`:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} Để truy cập notification của bạn từ JavaScript client, bạn nên định nghĩa một notification controller cho application của bạn để trả về notification cho một thực thể notifiable, chẳng hạn như người dùng hiện tại. Sau đó, bạn có thể tạo một HTTP request đến URI của controller đó từ JavaScript client của bạn.

<a name="marking-notifications-as-read"></a>
### Đánh dấu đã đọc cho Notification

Thông thường, bạn sẽ muốn đánh dấu một notification là "đã đọc" khi người dùng đã xem nó. Trait `Illuminate\Notifications\Notifiable` sẽ cung cấp phương thức `markAsRead`, sẽ cập nhật cột `read_at` trong record database của notification:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Tuy nhiên, thay vì lặp qua từng notification, bạn có thể sử dụng phương thức `markAsRead` trực tiếp trên collection của notification:

    $user->unreadNotifications->markAsRead();

Bạn cũng có thể sử dụng truy vấn cập nhật hàng loạt để đánh dấu tất cả các notification là đã đọc mà không lấy chúng ra cơ sở dữ liệu:

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

Tất nhiên, bạn cũng có thể 'xóa' các notification, để xóa chúng khỏi bảng bạn có thể làm như sau:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Broadcast Notifications

<a name="broadcast-prerequisites"></a>
### Yêu cầu

Trước khi broadcasting notification, bạn nên cấu hình và làm quen với các service [event broadcasting](/docs/{{version}}/broadcasting) của Laravel. Event broadcasting cung cấp một cách để phản ứng với các event Laravel do phía server tạo ra, từ JavaScript client của bạn.

<a name="formatting-broadcast-notifications"></a>
### Formatting Broadcast Notifications

Channel `broadcast` của broadcasts notification sẽ dùng các service [event broadcasting](/docs/{{version}}/broadcasting) của Laravel, cho phép JavaScript client của bạn nhận được notification theo thời gian thực. Nếu một notification hỗ trợ broadcasting, bạn nên định nghĩa phương thức `toBroadcast` trên class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `BroadcastMessage`. Dữ liệu được trả về sẽ được mã hóa dưới dạng JSON và phát đến JavaScript client của bạn. Chúng ta hãy xem một ví dụ về phương thức `toBroadcast`:

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

Tất cả các broadcast notification sẽ được queue để broadcasting. Nếu bạn muốn cấu hình queue connection hoặc tên queue được sử dụng để queue hoạt động broadcast, bạn có thể sử dụng các phương thức `onConnection` và `onQueue` của `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

> {tip} Ngoài dữ liệu bạn khai báo, broadcast notification cũng sẽ chứa một trường `type` dùng để chứa tên class của notification.

<a name="listening-for-notifications"></a>
### Listening cho Notifications

Notification sẽ được broadcast trên một private channel được định dạng theo cách sử dụng quy ước  `{notifiable}.{id}`. Vì vậy, nếu bạn đang gửi notification đến một instance `App\User` có ID là `1`, notification sẽ được broadcast trên private channel `App.User.1`. Khi sử dụng [Laravel Echo](/docs/{{version}}/broadcasting), bạn có thể dễ dàng listen cho các notification trên channel bằng phương thức helper `notification`:

    Echo.private('App.User.' + userId)Z
        .notification((notification) => {
            console.log(notification.type);
        });

#### Customizing The Notification Channel

Nếu bạn muốn tùy chỉnh channel mà nhận thực thể notifiable của broadcast notification, bạn có thể định nghĩa một phương thức `receivesBroadcastNotificationsOn` trên thực thể notifiable:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;

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

Gửi notification SMS trong Laravel được cung cấp bởi [Nexmo](https://www.nexmo.com/). Trước khi bạn có thể gửi notification qua Nexmo, bạn cần cài đặt package `nexmo/client` qua Composer và thêm một vài tùy chọn cấu hình vào file cấu hình `config/services.php` của bạn. Bạn có thể copy cấu hình mẫu ở bên dưới để bắt đầu:

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],

Tùy chọn `sms_from` là số điện thoại mà tin nhắn SMS của bạn sẽ được gửi từ đó. Bạn nên tạo số điện thoại cho application của bạn trong bảng điều khiển Nexmo.

<a name="formatting-sms-notifications"></a>
### Formatting SMS Notifications

Nếu một notification hỗ trợ gửi dưới dạng SMS, bạn nên định nghĩa phương thức `toNexmo` trên class notification. Phương thức này sẽ nhận được một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Messages\NexmoMessage`:

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

Nếu bạn muốn gửi một số notification từ một số điện thoại khác với số điện thoại được khai báo trong file `config/services.php` của bạn, bạn có thể sử dụng phương thức `from` trên một instance `NexmoMessage`:

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

Khi gửi notification qua channel `nexmo`, notification system sẽ tự động tìm thuộc tính `phone_number` trên thực thể notifiable. Nếu bạn muốn tùy chỉnh số điện thoại mà notification sẽ được gửi tới, hãy định nghĩa phương thức `routeNotificationForNexmo` trên thực thể:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Nexmo channel.
         *
         * @return string
         */
        public function routeNotificationForNexmo()
        {
            return $this->phone;
        }
    }

<a name="slack-notifications"></a>
## Slack Notifications

<a name="slack-prerequisites"></a>
### Yêu cầu

Trước khi bạn có thể gửi notification qua Slack, bạn phải cài đặt thư viện HTTP Guzzle qua Composer:

    composer require guzzlehttp/guzzle

Bạn cũng sẽ cần cấu hình ["Incoming Webhook"](https://api.slack.com/incoming-webhooks) cho group Slack của bạn. Việc cấu hình này sẽ cung cấp cho bạn một URL mà bạn có thể sử dụng khi [routing Slack notifications](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>
### Formatting Slack Notifications

Nếu một notification hỗ trợ gửi dưới dạng message Slack, bạn nên định nghĩa phương thức `toSlack` trên class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Messages\SlackMessage`. Tin nhắn Slack có thể chứa nội dung text cũng như "đính kèm" thêm một định dạng text hoặc một mảng các trường. Chúng ta hãy xem một ví dụ `toSlack` cơ bản:

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

<img src="https://laravel.com/assets/img/basic-slack-notification.png">

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
                    ->image('https://laravel.com/favicon.png')
                    ->content('This will display the Laravel logo next to the message');
    }

<a name="slack-attachments"></a>
### Đính kèm vào message slack

Bạn cũng có thể "đính kèm" thêm vào tin nhắn Slack. Đính kèm này cung cấp các tùy chọn định dạng phong phú hơn các tin nhắn text bình thường. Trong ví dụ này, chúng ta sẽ gửi notification lỗi về một ngoại lệ xảy ra trong một application, chứa một liên kết để xem thêm chi tiết về ngoại lệ:

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

<img src="https://laravel.com/assets/img/basic-slack-attachment.png">

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

<img src="https://laravel.com/assets/img/slack-fields-attachment.png">

#### Markdown Attachment Content

Nếu một số trường đính kèm của bạn chứa Markdown, bạn có thể sử dụng phương thức `markdown` để bảo Slack phân tích cú pháp và hiển thị các trường đính kèm dưới dạng văn bản được định dạng theo kiểu Markdown. Các giá trị được phương thức này chấp nhận là: `pretext`, `text` và / hoặc `fields`. Để biết thêm thông tin về định dạng đính kèm Slack, hãy xem [Tài liệu API Slack](https://api.slack.com/docs/message-formatting#message_formatting):

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

Để route Slack notification đến vị trí thích hợp, hãy định nghĩa phương thức `routeNotificationForSlack` trên thực thể notifiable của bạn. Điều này sẽ trả về URL webhook mà notification sẽ được gửi tới đó. URL webhook có thể được tạo bằng cách thêm một "Incoming Webhook" vào group Slack của bạn:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         *
         * @return string
         */
        public function routeNotificationForSlack()
        {
            return $this->slack_webhook_url;
        }
    }

<a name="notification-events"></a>
## Notification Events

Khi một notification được gửi, event `Illuminate\Notifications\Events\NotificationSent` sẽ được kích hoạt bởi notification system. Nó sẽ chứa thực thể "notifiable" và một instance notification. Bạn có thể đăng ký listener cho các event này trong `EventServiceProvider`:

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

> {tip} Sau khi đăng ký listener trong `EventServiceProvider` của bạn, hãy sử dụng lệnh Artisan `event:generate` để tạo nhanh các class listener.

Trong một event listener, bạn có thể truy cập vào các thuộc tính `notifiable`, `notification`, và `channel` trong event để biết thêm về người nhận notification hoặc chính notification đó:

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
    }

<a name="custom-channels"></a>
## Tuỳ biến Channels

Laravel có sẵn với một số notification channel, nhưng bạn có thể muốn viết driver của riêng bạn để gửi notification qua các channel khác. Laravel làm cho nó trở nên rất đơn giản. Để bắt đầu, hãy định nghĩa một class có chứa phương thức `send`. Phương thức sẽ nhận được hai tham số: một là `$notifiable` và một là `$notification`:

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

    use Illuminate\Bus\Queueable;
    use App\Channels\VoiceChannel;
    use App\Channels\Messages\VoiceMessage;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

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
