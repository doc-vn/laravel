# Notifications

- [Giới thiệu](#introduction)
- [Tạo Notification](#generating-notifications)
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
    - [Tuỳ biến Mailer](#customizing-the-mailer)
    - [Tuỳ biến template](#customizing-the-templates)
    - [Đính kèm](#mail-attachments)
    - [Thêm tags và metadata](#adding-tags-metadata)
    - [Tuỳ biến Symfony Message](#customizing-the-symfony-message)
    - [Dùng Mail](#using-mailables)
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
    - [Unicode Content](#unicode-content)
    - [Tuỳ biến "From" Number](#customizing-the-from-number)
    - [Thêm Client Reference](#adding-a-client-reference)
    - [Routing SMS Notifications](#routing-sms-notifications)
- [Slack Notifications](#slack-notifications)
    - [Yêu cầu](#slack-prerequisites)
    - [Formatting Slack Notifications](#formatting-slack-notifications)
    - [Tương tác với Slack](#slack-interactivity)
    - [Routing Slack Notifications](#routing-slack-notifications)
    - [Thông báo đến một External Slack Workspaces](#notifying-external-slack-workspaces)
- [Ngôn ngữ trong Notifications](#localizing-notifications)
- [Testing](#testing)
- [Notification Events](#notification-events)
- [Tuỳ biến Channels](#custom-channels)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc hỗ trợ [gửi email](/docs/{{version}}/mail), Laravel cũng hỗ trợ để gửi các notification qua nhiều channel khác nhau, như mail, SMS (qua [Vonage](https://www.vonage.com/communications-apis/), trước đây được gọi là Nexmo) và [Slack](https://slack.com). Ngoài ra, có nhiều [channel notification do cộng đồng xây dựng](https://laravel-notification-channels.com/about/#suggesting-a-new-channel) sẽ giúp gửi thông báo qua hàng chục channel khác nhau! Notification cũng có thể được lưu vào trong cơ sở dữ liệu của bạn để có thể được hiển thị trong giao diện của người dùng.

Thông thường, notification phải ngắn gọn, nội dung của message phải thông báo cho người dùng biết về điều gì đó đã xảy ra trong application của bạn. Ví dụ: nếu bạn đang viết một application thanh toán, bạn có thể gửi một notification "Thanh toán hóa đơn" cho người dùng của bạn thông qua các channel email và SMS.

<a name="generating-notifications"></a>
## Tạo Notification

Trong Laravel, các notification được đại diện bởi duy nhất một class thường được lưu trong thư mục `app/Notifications`. Bạn đừng lo lắng nếu bạn không thấy thư mục đó trong application của bạn - vì nó sẽ được tạo khi bạn chạy lệnh Artisan `make:notification`:

```shell
php artisan make:notification InvoicePaid
```

Lệnh này sẽ tạo một class notification mới vào trong thư mục `app/Notifications` của bạn. Mỗi class notification chứa một phương thức `via` và một số phương thức xây dựng message khác, chẳng hạn như `toMail` hoặc `toDatabase`, để chuyển đổi notification thành một message phù hợp cho một channel cụ thể.

<a name="sending-notifications"></a>
## Gửi Notification

<a name="using-the-notifiable-trait"></a>
### Dùng Notifiable Trait

Notification có thể được gửi theo hai cách: cách một là sử dụng phương thức `notify` trong trait `Notifiable` hoặc cách hai là sử dụng [facade](/docs/{{version}}/facades) `Notification`. Mặc định, trait `Notifiable` đã được thêm vào trong model `App\Models\User` trong ứng dụng của bạn:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Phương thức `notify` được cung cấp bởi trait này sẽ nhận vào một instance notification:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> [!NOTE]
> Hãy nhớ rằng, bạn có thể sử dụng trait `Notifiable` trên bất kỳ model nào mà bạn muốn. Bạn không bị giới hạn dùng nó trên model `User` của bạn.

<a name="using-the-notification-facade"></a>
### Dùng Notification Facade

Ngoài ra, bạn có thể gửi notification thông qua [facade](/docs/{{version}}/facades) `Notification`. Cách tiếp cận sẽ hữu ích khi bạn cần gửi notification cho nhiều thực thể notifiable, chẳng hạn như một collection user. Để gửi notification bằng facade, hãy truyền tất cả các thực thể notifiable và instance notification sang phương thức `send`:

    use Illuminate\Support\Facades\Notification;

    Notification::send($users, new InvoicePaid($invoice));

Bạn cũng có thể gửi một notification ngay lập tức bằng phương thức `sendNow`. Phương thức này sẽ gửi notification ngay lập tức ngay cả khi notification implement interface `ShouldQueue`:

    Notification::sendNow($developers, new DeploymentCompleted($deployment));

<a name="specifying-delivery-channels"></a>
### Chỉ định Channel sẽ được gửi

Mỗi class notification có một phương thức `via` định nghĩa channel nào của notification sẽ được gửi. Các notification có thể được gửi trên các channel `mail`, `database`, `broadcast`, `vonage`, và `slack`.

> [!NOTE]
> Nếu bạn muốn sử dụng các channel khác như Telegram hoặc Pusher, hãy xem drive do cộng đồng phát triển [Laravel Notification Channels website](http://laravel-notification-channels.com).

Phương thức `via` nhận vào một instance `$notifiable`, đây sẽ là một instance của class mà notification sẽ gửi đến. Bạn có thể sử dụng `$notifiable` để xác định channel nào sẽ gửi notification:

    /**
     * Get the notification's delivery channels.
     *
     * @return array<int, string>
     */
    public function via(object $notifiable): array
    {
        return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### Queue Notification

> [!WARNING]
> Trước khi queue notification, bạn nên cấu hình queue và [chạy một worker](/docs/{{version}}/queues#running-the-queue-worker).

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

Khi queue thông báo, một queued job sẽ được tạo cho mỗi kết hợp giữa người nhận và channel. Ví dụ, sáu job sẽ được gửi đến queue nếu thông báo của bạn có ba người nhận và hai channel.

<a name="delaying-notifications"></a>
#### Delaying Notifications

Nếu bạn muốn delay việc gửi notification, bạn có thể kết hợp với phương thức `delay` vào phần khởi tạo notification của bạn:

    $delay = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($delay));

<a name="delaying-notifications-per-channel"></a>
#### Delaying Notifications Per Channel

Bạn có thể truyền một mảng cho phương thức `delay` để chỉ định độ trễ cho các channel cụ thể:

    $user->notify((new InvoicePaid($invoice))->delay([
        'mail' => now()->addMinutes(5),
        'sms' => now()->addMinutes(10),
    ]));

Ngoài ra, bạn có thể định nghĩa phương thức `withDelay` trên chính class notification. Phương thức `withDelay` sẽ trả về một mảng gồm tên channel và giá trị độ trễ:

    /**
     * Determine the notification's delivery delay.
     *
     * @return array<string, \Illuminate\Support\Carbon>
     */
    public function withDelay(object $notifiable): array
    {
        return [
            'mail' => now()->addMinutes(5),
            'sms' => now()->addMinutes(10),
        ];
    }

<a name="customizing-the-notification-queue-connection"></a>
#### Customizing The Notification Queue Connection

Mặc định, các queued notification sẽ được queue bằng kết nối queue mặc định trong ứng dụng của bạn. Nếu bạn muốn chỉ định một kết nối khác sẽ được sử dụng cho một notification cụ thể, bạn có thể gọi phương thức `onConnection` từ hàm constructor của notification của bạn:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        /**
         * Create a new notification instance.
         */
        public function __construct()
        {
            $this->onConnection('redis');
        }
    }

Hoặc, nếu bạn muốn chỉ định cụ thể một kết nối queue sẽ được sử dụng cho mỗi channel notification mà được notification của bạn hỗ trợ, bạn có thể định nghĩa phương thức `viaConnections` trên notification của bạn. Phương thức này sẽ trả về một mảng gồm các cặp tên channel và tên kết nối queue:

    /**
     * Determine which connections should be used for each notification channel.
     *
     * @return array<string, string>
     */
    public function viaConnections(): array
    {
        return [
            'mail' => 'redis',
            'database' => 'sync',
        ];
    }

<a name="customizing-notification-channel-queues"></a>
#### Customizing Notification Channel Queues

Nếu bạn muốn chỉ định một queue cụ thể được sử dụng cho mỗi loại notification channel hỗ trợ, bạn có thể định nghĩa một phương thức `viaQueues` trong notification của bạn. Phương thức này sẽ trả về một mảng gồm các cặp tên channel và tên queue:

    /**
     * Determine which queues should be used for each notification channel.
     *
     * @return array<string, string>
     */
    public function viaQueues(): array
    {
        return [
            'mail' => 'mail-queue',
            'slack' => 'slack-queue',
        ];
    }

<a name="queued-notifications-and-database-transactions"></a>
#### Queued Notifications và Database Transactions

Khi các queued notification được gửi đi trong các database transaction, chúng có thể bị queue xử lý trước khi database transaction được commit. Khi điều này xảy ra, mọi cập nhật bạn đã commit đối với model hoặc bản ghi cơ sở dữ liệu trong quá trình database transaction có thể chưa được lưu vào trong cơ sở dữ liệu. Ngoài ra, mọi model hoặc bản ghi cơ sở dữ liệu được tạo trong transaction có thể không tồn tại trong cơ sở dữ liệu. Nếu notification của bạn phụ thuộc vào các trường hợp như thế này thì các lỗi không mong muốn có thể xảy ra khi job xử lý queued notification.

Nếu tùy chọn cấu hình `after_commit` trong queue connection của bạn được set thành `false`, thì bạn vẫn có thể chỉ định một queued notification cụ thể sẽ được gửi đi sau khi tất cả các database transaction được commit bằng cách gọi phương thức `afterCommit` khi gửi notification:

    use App\Notifications\InvoicePaid;

    $user->notify((new InvoicePaid($invoice))->afterCommit());

Ngoài ra, bạn có thể gọi phương thức `afterCommit` từ hàm khởi tạo của notification:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        /**
         * Create a new notification instance.
         */
        public function __construct()
        {
            $this->afterCommit();
        }
    }

> [!NOTE]
> Để tìm hiểu thêm về cách giải quyết những vấn đề này, vui lòng xem lại tài liệu về [queued job và database transaction](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="determining-if-the-queued-notification-should-be-sent"></a>
#### Determining If A Queued Notification Should Be Sent

Sau khi một queued notification được gửi đến queue để xử lý background, nó thường sẽ được queue worker chấp nhận và gửi đến người nhận.

Tuy nhiên, nếu bạn muốn đưa ra một kiểm tra cuối cùng về việc có nên gửi queued notification khi nó đang được queue worker xử lý hay không, bạn có thể định nghĩa một phương thức `shouldSend` trên class notification. Nếu phương thức này trả về `false`, notification sẽ không được gửi:

    /**
     * Determine if the notification should be sent.
     */
    public function shouldSend(object $notifiable, string $channel): bool
    {
        return $this->invoice->isPaid();
    }

<a name="on-demand-notifications"></a>
### On-Demand Notifications

Thỉnh thoảng bạn có thể cần gửi notification cho người mà chưa được lưu trong cơ sở dữ liệu dưới dạng một "user". Sử dụng phương thức `route` của facade `Notification`, bạn có thể chỉ định thông tin ad-hoc notification routing trước khi gửi notification:

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Support\Facades\Notification;

    Notification::route('mail', 'taylor@example.com')
                ->route('vonage', '5555555555')
                ->route('slack', '#slack-channel')
                ->route('broadcast', [new Channel('channel-name')])
                ->notify(new InvoicePaid($invoice));

Nếu bạn muốn thêm tên người nhận khi gửi notification tới route `mail`, bạn có thể thêm một mảng chứa các địa chỉ email làm khóa và tên người nhận làm giá trị cho tham số đầu tiên trong mảng:

    Notification::route('mail', [
        'barrett@example.com' => 'Barrett Blair',
    ])->notify(new InvoicePaid($invoice));

Bằng cách sử dụng phương thức `routes`, bạn có thể cung cấp thông tin tùy ý cho nhiều channel thông báo cùng một lúc:

    Notification::routes([
        'mail' => ['barrett@example.com' => 'Barrett Blair'],
        'vonage' => '5555555555',
    ])->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## Mail Notifications

<a name="formatting-mail-messages"></a>
### Formatting Mail Messages

Nếu một notification hỗ trợ gửi dưới dạng email, bạn nên định nghĩa một phương thức `toMail` trong class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Messages\MailMessage`.

Class `MailMessage` có chứa một số phương thức đơn giản để giúp bạn xây dựng các email. Mail message có thể chứa các dòng text cũng như các "call to action". Chúng ta hãy xem một ví dụ về phương thức `toMail`:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->lineIf($this->amount > 0, "Amount paid: {$this->amount}")
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> [!NOTE]
> Lưu ý rằng chúng ta đang sử dụng `$this->invoice->id` trong phương thức `toMail`. Bạn có thể truyền bất kỳ dữ liệu nào mà notification của bạn cần để tạo message cho nó bằng hàm khởi tạo của notification.

Trong ví dụ này, chúng ta đã đăng ký một lời chào, một dòng text, một call to action và sau đó là một dòng text khác. Các phương thức này được cung cấp bởi đối tượng `MailMessage` giúp cho việc định dạng các email giao dịch nhỏ trở nên dễ dàng và đơn giản hơn. Sau đó, mail channel sẽ dịch các thành phần của message này thành một template email HTML đẹp có phản hồi nhanh với một bản sao text đơn giản. Đây là một ví dụ mẫu về email được tạo bởi channel `mail`:

<img src="https://laravel.com/img/docs/notification-example-2.png">

> [!NOTE]
> Khi gửi mail notification, hãy đảm bảo là bạn đã set tuỳ chọn cấu hình `name` trong file cấu hình `config/app.php` của bạn. Giá trị này sẽ được sử dụng trong phần header và footer của message mail notification của bạn.

<a name="error-messages"></a>
#### Error Messages

Một số notification sẽ thông báo cho người dùng về lỗi, chẳng hạn như thanh toán hóa đơn không thành công. Bạn có thể chỉ định một tin nhắn email lỗi bằng cách gọi phương thức `error` khi build tin nhắn của bạn. Khi sử dụng phương thức `error` trên một tin nhắn email, thì nút action sẽ có màu đỏ thay vì màu đen:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Invoice Payment Failed')
                    ->line('...');
    }

<a name="other-mail-notification-formatting-options"></a>
#### Other Mail Notification Formatting Options

Thay vì định nghĩa "các dòng" văn bản trong class thông báo, bạn có thể sử dụng phương thức `view` để chỉ định một template tùy biến sẽ được sử dụng để hiển thị email thông báo:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)->view(
            'mail.invoice.paid', ['invoice' => $this->invoice]
        );
    }

Bạn có thể chỉ định thêm chế độ plain-text view cho tin nhắn email bằng cách truyền tên view làm phần tử thứ hai của mảng được cung cấp cho phương thức `view`:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)->view(
            ['mail.invoice.paid', 'mail.invoice.paid-text'],
            ['invoice' => $this->invoice]
        );
    }

Hoặc, nếu tin nhắn của bạn chỉ có dạng plain-text, bạn có thể sử dụng phương thức `text`:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)->text(
            'mail.invoice.paid-text', ['invoice' => $this->invoice]
        );
    }

<a name="customizing-the-sender"></a>
### Tuỳ biến người gửi

Mặc định, địa chỉ người gửi hoặc từ địa chỉ email được định nghĩa trong file cấu hình `config/mail.php`. Tuy nhiên, bạn có thể chỉ định một địa chỉ from cho một notification cụ thể bằng cách sử dụng phương thức `from`:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->from('barrett@example.com', 'Barrett Blair')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### Tuỳ biến người nhận

Khi gửi notifications qua channel `mail`, hệ thống notification sẽ tự động tìm kiếm thuộc tính `email` trong thực thể notifiable của bạn. Bạn có thể tùy biến địa chỉ email nào sẽ được sử dụng để gửi notification bằng cách định nghĩa phương thức `routeNotificationForMail` trên thực thể notifiable đó:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the mail channel.
         *
         * @return  array<string, string>|string
         */
        public function routeNotificationForMail(Notification $notification): array|string
        {
            // Return email address only...
            return $this->email_address;

            // Return email address and name...
            return [$this->email_address => $this->name];
        }
    }

<a name="customizing-the-subject"></a>
### Tuỳ biến chủ đề

Mặc định, chủ đề của email là tên class của notification được định dạng theo dạng "Title Case". Vì vậy, nếu class notification của bạn được đặt tên là `InvoicePaid`, thì chủ đề của email sẽ là `Invoice Paid`. Nếu bạn muốn chỉ định một chủ đề khác cho message, bạn có thể gọi phương thức `subject` khi xây dựng message của bạn:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-mailer"></a>
### Tuỳ biến Mailer

Mặc định, email notification sẽ được gửi bằng mailer mặc định được định nghĩa trong file cấu hình `config/mail.php`. Tuy nhiên, bạn có thể chỉ định một mailer khác trong lúc runtime bằng cách gọi phương thức `mailer` khi tạo message của bạn:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->mailer('postmark')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### Tuỳ biến template

Bạn có thể sửa HTML và template được sử dụng bởi mail notification bằng cách export resources của package notification. Sau khi chạy lệnh này, các template mail notification sẽ được lưu ở trong thư mục `resources/views/vendor/notifications`:

```shell
php artisan vendor:publish --tag=laravel-notifications
```

<a name="mail-attachments"></a>
### Đính kèm

Để thêm một file đính kèm vào mail notification, hãy sử dụng phương thức `attach` trong khi tạo mail của bạn. Phương thức `attach` sẽ chấp nhận một đường dẫn tuyệt đối đến file làm tham số đầu tiên:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file');
    }

> [!NOTE]
> Phương thức `attach` được cung cấp bởi các tin nhắn thông báo email cũng chấp nhận các [attachable object](/docs/{{version}}/mail#attachable-objects). Vui lòng tham khảo tài liệu cụ thể về các [attachable object](/docs/{{version}}/mail#attachable-objects) để hiểu thêm về chúng.

Khi đính kèm file vào tin nhắn, bạn cũng có thể chỉ định thêm tên hiển thị hoặc loại MIME bằng cách truyền một `array` làm tham số thứ hai cho phương thức `attach`:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',
                        'mime' => 'application/pdf',
                    ]);
    }

Không giống như đính kèm file trong các đối tượng mail, bạn không được đính kèm file trực tiếp từ storage disk bằng cách sử dụng `attachFromStorage`. Thay vào đó, bạn nên sử dụng phương thức `attach` với đường dẫn tuyệt đối đến file trên storage disk. Ngoài ra, bạn có thể trả về [mailable](/docs/{{version}}/mail#generating-mailables) từ phương thức `toMail`:

    use App\Mail\InvoicePaid as InvoicePaidMailable;

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): Mailable
    {
        return (new InvoicePaidMailable($this->invoice))
                    ->to($notifiable->email)
                    ->attachFromStorage('/path/to/file');
    }

Khi cần thiết, có thể đính kèm nhiều file vào một tin nhắn bằng phương thức `attachMany`:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attachMany([
                        '/path/to/forge.svg',
                        '/path/to/vapor.svg' => [
                            'as' => 'Logo.svg',
                            'mime' => 'image/svg+xml',
                        ],
                    ]);
    }

<a name="raw-data-attachments"></a>
#### Raw Data Attachments

Phương thức `attachData` có thể được sử dụng để đính kèm một chuỗi raw byte dưới dạng file đính kèm. Khi gọi phương thức `attachData`, bạn nên cung cấp tên file sẽ được gán cho file đính kèm đó:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="adding-tags-metadata"></a>
### Thêm tags và metadata

Một số nhà cung cấp dịch vụ email của bên thứ ba như Mailgun và Postmark hỗ trợ "tags" và "metadata" cho tin nhắn, có thể được sử dụng để nhóm và theo dõi email được gửi bởi ứng dụng của bạn. Bạn có thể thêm tags và metadata vào tin nhắn email thông qua phương thức `tag` and `metadata`:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Comment Upvoted!')
                    ->tag('upvote')
                    ->metadata('comment_id', $this->comment->id);
    }

Nếu ứng dụng của bạn đang sử dụng driver Mailgun, bạn có thể tham khảo tài liệu của Mailgun để biết thêm thông tin về [tags](https://documentation.mailgun.com/en/latest/user_manual.html#tagging-1) và [metadata](https://documentation.mailgun.com/en/latest/user_manual.html#attaching-data-to-messages). Tương tự như vậy, bạn cũng có thể tham khảo tài liệu của Postmark để biết thêm thông tin về hỗ trợ của họ đối với [tags](https://postmarkapp.com/blog/tags-support-for-smtp) và [metadata](https://postmarkapp.com/support/article/1125-custom-metadata-faq).

Nếu ứng dụng của bạn sử dụng Amazon SES để gửi email, bạn nên sử dụng phương thức `metadata` để đính kèm ["tags" SES](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) vào tin nhắn.

<a name="customizing-the-symfony-message"></a>
### Tuỳ biến Symfony Message

Phương thức `withSymfonyMessage` của class `MailMessage` cho phép bạn đăng ký một closure sẽ được gọi cùng với instance Symfony Message trước khi tin nhắn được gửi. Điều này cho bạn có cơ hội tùy chỉnh sâu vào tin nhắn trước khi nó được gửi:

    use Symfony\Component\Mime\Email;

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->withSymfonyMessage(function (Email $message) {
                        $message->getHeaders()->addTextHeader(
                            'Custom-Header', 'Header Value'
                        );
                    });
    }

<a name="using-mailables"></a>
### Dùng Mail

Nếu cần, bạn có thể trả về một [mailable object](/docs/{{version}}/mail) từ phương thức `toMail` của notification. Khi trả về `Mailable` thay vì `MailMessage`, bạn sẽ cần chỉ định người nhận mail bằng phương thức `to` của đối tượng mailable:

    use App\Mail\InvoicePaid as InvoicePaidMailable;
    use Illuminate\Mail\Mailable;

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): Mailable
    {
        return (new InvoicePaidMailable($this->invoice))
                    ->to($notifiable->email);
    }

<a name="mailables-and-on-demand-notifications"></a>
#### Mailables & On-Demand Notifications

Nếu bạn đang gửi [notification theo yêu cầu](#on-demand-notifications), instance `$notificable` được cung cấp cho phương thức `toMail` sẽ là một instance của `Illuminate\Notifications\AnonymousNotifiable`, cung cấp một phương thức `routeNotificationFor` có thể được sử dụng để lấy ra địa chỉ email mà notification theo yêu cầu sẽ được gửi tới:

    use App\Mail\InvoicePaid as InvoicePaidMailable;
    use Illuminate\Notifications\AnonymousNotifiable;
    use Illuminate\Mail\Mailable;

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): Mailable
    {
        $address = $notifiable instanceof AnonymousNotifiable
                ? $notifiable->routeNotificationFor('mail')
                : $notifiable->email;

        return (new InvoicePaidMailable($this->invoice))
                    ->to($address);
    }

<a name="previewing-mail-notifications"></a>
### Xem trước Mail Notification

Khi thiết kế một template mail notification, sẽ tiện lợi hơn nếu xem trước được một mail notification được tạo ra trong trình duyệt của bạn giống như một template Blade điển hình. Vì lý do này, Laravel cho phép bạn trả về bất kỳ mail notification nào được tạo ra bởi mail notification trực tiếp từ một route closure hoặc một controller. Khi một `MailMessage` được trả về, nó sẽ được tạo và hiển thị trong trình duyệt, cho phép bạn xem trước các thiết kế mà không cần gửi nó đến một địa chỉ email thực tế:

    use App\Models\Invoice;
    use App\Notifications\InvoicePaid;

    Route::get('/notification', function () {
        $invoice = Invoice::find(1);

        return (new InvoicePaid($invoice))
                    ->toMail($invoice->user);
    });

<a name="markdown-mail-notifications"></a>
## Markdown Mail Notification

Markdown mail notification cho phép bạn tận dụng các template mail notification được xây dựng sẵn, đồng thời cho bạn tự do hơn để viết các tùy biến message dài hơn. Vì các message được viết bằng Markdown, nên Laravel có thể hiển thị các template HTML đẹp đáp ứng cho các message, đồng thời tự động tạo một bản sao text đơn giản.

<a name="generating-the-message"></a>
### Tạo Message

Để tạo một notification với một template Markdown, bạn có thể sử dụng tùy chọn `--markdown` trong lệnh Artisan `make:notification`:

```shell
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

Giống như tất cả các mail notification khác, các notification sử dụng bởi các template Markdown sẽ định nghĩa một phương thức `toMail` trong class notification của chúng. Tuy nhiên, thay vì sử dụng các phương thức `line` và `action` để khởi tạo cho notification, hãy sử dụng phương thức `markdown` để khai báo tên của template Markdown được sử dụng. Một mảng dữ liệu bạn muốn truyền cho template có thể truyề vào làm tham số thứ hai của phương thức:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### Viết Message

Markdown mail notification sử dụng kết hợp giữa các component Blade và cú pháp Markdown cho phép bạn dễ dàng khởi tạo notification trong khi vẫn tận dụng được các component notification được tạo sẵn của Laravel:

```blade
<x-mail::message>
# Invoice Paid

Your invoice has been paid!

<x-mail::button :url="$url">
View Invoice
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

<a name="button-component"></a>
#### Button Component

Component button sẽ tạo một button được đặt ở chính giữa của trang. Component này chấp nhận hai tham số, một là `url` và một là tùy chọn `color`. Các màu được hỗ trợ là `primary`, `green` và `red`. Bạn có thể thêm các button component vào một notification nếu muốn:

```blade
<x-mail::button :url="$url" color="green">
View Invoice
</x-mail::button>
```

<a name="panel-component"></a>
#### Panel Component

Component panel sẽ tạo một block text trong một panel có màu nền hơi khác so với các phần khác của notification. Điều này cho phép bạn thu hút sự chú ý của người dùng đến block text:

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>
#### Table Component

Component table cho phép bạn chuyển đổi một bảng Markdown thành một bảng HTML. Component này chấp nhận nội dung như một bảng Markdown bình thường. Căn chỉnh trái phải của cột cũng được mặc định hỗ trợ bởi cú pháp căn chỉnh cột của Markdown:

```blade
<x-mail::table>
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### Tuỳ biến The Compoents

Bạn có thể export tất cả các component Markdown mail sang một thư mục riêng của bạn để tùy chỉnh. Để export các component này, hãy sử dụng lệnh Artisan `vendor:publish` để export với nội dung tag `laravel-mail`:

```shell
php artisan vendor:publish --tag=laravel-mail
```

Lệnh này sẽ export các component Markdown mail sang thư mục `resources/views/vendor/mail`. Thư mục `mail` sẽ chứa một thư mục là `html` và một thư mục là `text`, mỗi thư mục chứa các hiển thị tương ứng cho mỗi component có sẵn. Các component trong thư mục `html` được sử dụng để tạo phiên bản HTML cho email và các bản sao của chúng còn trong thư mục `text` được sử dụng để tạo các phiên bản text thuần túy. Bạn có thể tự do tùy chỉnh các component này theo cách mà bạn muốn.

<a name="customizing-the-css"></a>
#### Customizing The CSS

Sau khi export các component, thư mục `resources/views/vendor/mail/html/themes` sẽ chứa một file `default.css`. Bạn có thể tùy chỉnh CSS trong file này và các tuỳ chỉnh của bạn sẽ tự động được nhúng vào trong các hiển thị HTML của Markdown notification của bạn.

Nếu bạn muốn xây dựng một theme mới cho các component Markdown của Laravel, bạn có thể tạo một file CSS mới trong thư mục `html/themes`. Sau khi tạo tên và lưu file CSS của bạn, hãy cập nhật tùy chọn `theme` trong file cấu hình `mail` để khớp với tên theme mới của bạn.

Để tùy chỉnh theme cho một notification riêng lẻ, bạn có thể gọi phương thức `theme` trong khi xây dựng mail message notification. Phương thức `theme` chấp nhận tên của theme sẽ được sử dụng khi gửi notification:

    /**
     * Get the mail representation of the notification.
     */
    public function toMail(object $notifiable): MailMessage
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

Channel notification `database` sẽ lưu trữ thông tin notification vào trong bảng của cơ sở dữ liệu. Bảng này sẽ chứa thông tin như loại notification cũng như cấu trúc dữ liệu JSON mô tả của notification đó.

Bạn có thể truy vấn vào bảng để hiển thị các notification trong giao diện người dùng của application. Nhưng, trước khi bạn có thể làm điều đó, bạn sẽ cần phải tạo một bảng để lưu các notification của bạn. Bạn có thể sử dụng lệnh `notifications:table` để tạo một [migration](/docs/{{version}}/migrations) với một table schema thích hợp:

```shell
php artisan notifications:table

php artisan migrate
```

> [!NOTE]
> Nếu các model notifiable của bạn đang sử dụng [khóa chính UUID hoặc ULID](/docs/{{version}}/eloquent#uuid-and-ulid-keys), bạn nên thay phương thức `morphs` bằng [`uuidMorphs`](/docs/{{version}}/migrations#column-method-uuidMorphs) hoặc [`ulidMorphs`](/docs/{{version}}/migrations#column-method-ulidMorphs) trong bảng notification migration.

<a name="formatting-database-notifications"></a>
### Formatting Database Notifications

Nếu một notification hỗ trợ lưu trữ trong bảng cơ sở dữ liệu, thì bạn nên định nghĩa một phương thức `toDatabase` hoặc `toArray` trong class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một mảng PHP. Mảng được trả về sẽ được mã hóa dưới dạng JSON và được lưu trữ vào trong cột `data` trong bảng `notifications`. Bạn hãy xem một ví dụ về phương thức `toArray`:

    /**
     * Get the array representation of the notification.
     *
     * @return array<string, mixed>
     */
    public function toArray(object $notifiable): array
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

Khi thông báo được lưu vào trong cơ sở dữ liệu của ứng dụng, cột `type` sẽ được điền bằng tên class của thông báo. Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách định nghĩa phương thức `databaseType` trên class thông báo của bạn:

    /**
     * Get the notification's database type.
     *
     * @return string
     */
    public function databaseType(object $notifiable): string
    {
        return 'invoice-paid';
    }

<a name="todatabase-vs-toarray"></a>
#### `toDatabase` và `toArray`

Phương thức `toArray` cũng được sử dụng bởi channel `broadcast` để xác định xem dữ liệu nào sẽ được phát đến JavaScript mà được cung cấp bởi frontend. Nhưng nếu bạn muốn biểu thị hai mảng khác nhau cho hai channel `database` và `broadcast` khác nhau, thì bạn nên định nghĩa bằng phương thức `toDatabase` thay vì phương thức `toArray`.

<a name="accessing-the-notifications"></a>
### Truy cập Notifications

Khi notification đã được lưu vào trong cơ sở dữ liệu, bạn cần một cách để truy cập vào các notification từ các thực thể notifiable của bạn. Mặc định trait `Illuminate\Notifications\Notifiable` đã được khai báo trong model mặc định `App\Models\User` đi kèm với Laravel, nó có chứa một [quan hệ Eloquent](/docs/{{version}}/eloquent-relationships) là `notifications` sẽ trả về các notification cho các thực thể. Để lấy notification, bạn có thể truy cập phương thức này giống như bất kỳ phương thức quan hệ Eloquent nào khác. Mặc định, các notification sẽ được sắp xếp theo thời gian `created_at` các thông báo gần nhất sẽ nằm ở đầu collection:

    $user = App\Models\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Nếu bạn chỉ muốn lấy các notification "chưa đọc", bạn có thể sử dụng quan hệ `unreadNotifications`. Một lần nữa, các notification này sẽ được sắp xếp theo thời gian timestamp `created_at` các thông báo gần nhất sẽ nằm ở đầu collection:

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> [!NOTE]
> Để truy cập vào notification của bạn từ JavaScript client, bạn nên định nghĩa một notification controller riêng cho application của bạn để trả về notification cho một thực thể notifiable, chẳng hạn như người dùng hiện tại. Sau đó, bạn hãy tạo một HTTP request đến URL của controller đó từ JavaScript client của bạn.

<a name="marking-notifications-as-read"></a>
### Đánh dấu đã đọc cho Notification

Thông thường, bạn sẽ muốn đánh dấu một notification là "đã đọc" khi người dùng đã xem nó. Trait `Illuminate\Notifications\Notifiable` cũng sẽ cung cấp một phương thức `markAsRead` để cập nhật cột `read_at` trong record database của notification đó:

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Tuy nhiên, thay vì lặp từng notification, bạn có thể sử dụng phương thức `markAsRead` trực tiếp trên một collection của notification:

    $user->unreadNotifications->markAsRead();

Bạn cũng có thể sử dụng cập nhật hàng loạt để đánh dấu tất cả các notification là đã đọc mà không cần phải lấy chúng ra khỏi cơ sở dữ liệu:

    $user = App\Models\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

Bạn cũng có thể 'xóa' các notification này, để xóa chúng ra khỏi bảng bạn có thể làm như sau:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Broadcast Notifications

<a name="broadcast-prerequisites"></a>
### Yêu cầu

Trước khi broadcasting notification, bạn nên cấu hình và làm quen với các service [event broadcasting](/docs/{{version}}/broadcasting) của Laravel. Event broadcasting cung cấp một cách phù hợp để tương tác với các event Laravel bên phía server từ JavaScript mà được cung cấp bởi frontend của bạn.

<a name="formatting-broadcast-notifications"></a>
### Formatting Broadcast Notifications

Channel `broadcast` của broadcasts notification sẽ dùng các service [event broadcasting](/docs/{{version}}/broadcasting) của Laravel, cho phép JavaScript mà được cung cấp bởi frontend của bạn nhận được các notification theo thời gian thực. Nếu một notification hỗ trợ broadcasting, bạn có thể định nghĩa phương thức `toBroadcast` trên class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `BroadcastMessage`. Nếu phương thức `toBroadcast` không tồn tại, phương thức `toArray` sẽ được sử dụng để thu thập dữ liệu cần được broadcast. Dữ liệu được trả về sẽ được mã hóa dưới dạng JSON và broadcast đến JavaScript mà được cung cấp bởi frontend của bạn. Chúng ta hãy xem một ví dụ về phương thức `toBroadcast`:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Get the broadcastable representation of the notification.
     */
    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

<a name="broadcast-queue-configuration"></a>
#### Broadcast Queue Configuration

Tất cả các broadcast notification sẽ được queue lại để broadcasting. Nếu bạn muốn cấu hình queue connection hoặc tên queue được sử dụng để queue lại broadcast, bạn có thể sử dụng các phương thức `onConnection` và `onQueue` của `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

<a name="customizing-the-notification-type"></a>
#### Customizing The Notification Type

Ngoài dữ liệu bạn chỉ định, tất cả các broadcast notification cũng có thêm một trường `type` để chứa tên class của notification. Nếu bạn muốn tùy chỉnh `type` của notification, bạn có thể định nghĩa phương thức` broadcastType` trên class notification đó:

    /**
     * Get the type of the notification being broadcast.
     */
    public function broadcastType(): string
    {
        return 'broadcast.message';
    }

<a name="listening-for-notifications"></a>
### Listening cho Notifications

Notification sẽ được broadcast trên các private channel được định danh theo cách sử dụng quy ước `{notifiable}.{id}`. Vì vậy, nếu bạn đang gửi notification đến một instance `App\Models\User` có ID là `1`, thì notification sẽ được broadcast trên private channel là `App.Models.User.1`. Khi sử dụng [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation), bạn có thể dễ dàng listen cho các notification trên channel này bằng phương thức helper `notification`:

    Echo.private('App.Models.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

<a name="customizing-the-notification-channel"></a>
#### Customizing The Notification Channel

Nếu bạn muốn tùy chỉnh channel mà một broadcast notification của thực thể được phát sóng trên đó, bạn có thể định nghĩa một phương thức `receivesBroadcastNotificationsOn` trên thực thể notifiable:

    <?php

    namespace App\Models;

    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The channels the user receives notification broadcasts on.
         */
        public function receivesBroadcastNotificationsOn(): string
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## SMS Notifications

<a name="sms-prerequisites"></a>
### Yêu cầu

Gửi notification SMS trong Laravel được mặc định cung cấp bởi [Vonage](https://www.vonage.com/) (trước đây được gọi là Nexmo). Trước khi bạn có thể gửi notification qua Vonage, bạn cần phải cài đặt package `laravel/vonage-notification-channel` và package `guzzlehttp/guzzle` thông qua Composer.

    composer require laravel/vonage-notification-channel guzzlehttp/guzzle

Package đã chứa một [file cấu hình](https://github.com/laravel/vonage-notification-channel/blob/3.x/config/vonage.php). Tuy nhiên, bạn không bắt buộc phải export file cấu hình này sang ứng dụng của riêng bạn. Bạn có thể đơn giản là sử dụng các biến môi trường `VONAGE_KEY` và` VONAGE_SECRET` để định nghĩa khóa bí mật và công khai Vonage của bạn.

Sau khi định nghĩa key của bạn, bạn nên set một biến môi trường `VONAGE_SMS_FROM` để định nghĩa số điện thoại mà tin nhắn SMS của bạn sẽ được gửi theo mặc định. Bạn có thể tạo số điện thoại này trong bảng điều khiển Vonage:

    VONAGE_SMS_FROM=15556666666

<a name="formatting-sms-notifications"></a>
### Formatting SMS Notifications

Nếu một notification hỗ trợ gửi dưới dạng SMS, bạn nên định nghĩa phương thức `toVonage` trên class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Messages\VonageMessage`:

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * Get the Vonage / SMS representation of the notification.
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your SMS message content');
    }

<a name="unicode-content"></a>
#### Unicode Content

Nếu tin nhắn SMS của bạn chứa các ký tự unicode, bạn nên gọi phương thức `unicode` khi khởi tạo instance `VonageMessage`:

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * Get the Vonage / SMS representation of the notification.
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### Customizing The "From" Number

Nếu bạn muốn gửi một số thông báo từ một số điện thoại khác, khác với số điện thoại được chỉ định bằng biến môi trường `VONAGE_SMS_FROM` của bạn, bạn có thể gọi phương thức `from` trên instance `VonageMessage`:

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * Get the Vonage / SMS representation of the notification.
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="adding-a-client-reference"></a>
### Thêm Client Reference

Nếu bạn muốn theo dõi chi phí cho mỗi người dùng, một nhóm hoặc khách hàng của bạn, bạn có thể thêm một "client reference" vào thông báo. Vonage sẽ cho phép bạn tạo báo cáo bằng cách sử dụng client reference này để bạn có thể hiểu rõ hơn về việc sử dụng SMS của một khách hàng cụ thể. Client reference có thể là bất kỳ chuỗi nào mà bạn muốn và có tối đa 40 ký tự:

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * Get the Vonage / SMS representation of the notification.
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->clientReference((string) $notifiable->id)
                    ->content('Your SMS message content');
    }

<a name="routing-sms-notifications"></a>
### Routing SMS Notifications

Để route thông báo Vonage đến một số điện thoại thích hợp, hãy định nghĩa phương thức `routeNotificationForVonage` trên model notifiable của bạn:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Vonage channel.
         */
        public function routeNotificationForVonage(Notification $notification): string
        {
            return $this->phone_number;
        }
    }

<a name="slack-notifications"></a>
## Slack Notifications

<a name="slack-prerequisites"></a>
### Yêu cầu

Trước khi bạn gửi một Slack notification, bạn cần cài đặt notification channel Slack thông qua Composer:

```shell
composer require laravel/slack-notification-channel
```

Ngoài ra, bạn phải tạo một [Slack App](https://api.slack.com/apps?new_app=1) cho Slack workspace của bạn.

Nếu bạn chỉ cần gửi thông báo đến cùng một Slack workspace mà App được tạo ra, bạn nên đảm bảo là App của bạn có quyền `chat:write`, `chat:write.public` và `chat:write.customize`. Các quyền này có thể được thêm vào từ tab quản lý App "OAuth & Permissions" trong Slack.

Tiếp theo, copy "Bot User OAuth Token" của App và set nó vào mảng cấu hình `slack` trong file cấu hình `services.php` của ứng dụng. Token này có thể được tìm thấy trên tab "OAuth & Permissions" trong Slack:

    'slack' => [
        'notifications' => [
            'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
            'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
        ],
    ],

<a name="slack-app-distribution"></a>
#### App Distribution

Nếu ứng dụng của bạn cần gửi thông báo đến các Slack workspace bên ngoài do người dùng ứng dụng của bạn sở hữu, bạn sẽ cần phải "phân phối" App của bạn qua Slack. Việc phân phối App có thể được quản lý từ tab "Manage Distribution" của App trong Slack. Sau khi App của bạn đã được phân phối, bạn có thể sử dụng [Socialite](/docs/{{version}}/socialite) để [lấy token Slack Bot](/docs/{{version}}/socialite#slack-bot-scopes) thay cho người dùng ứng dụng của bạn.

<a name="formatting-slack-notifications"></a>
### Formatting Slack Notifications

Nếu một notification hỗ trợ gửi dưới dạng message của Slack, bạn nên định nghĩa phương thức `toSlack` trên class notification. Phương thức này sẽ nhận vào một thực thể `$notifiable` và sẽ trả về một instance `Illuminate\Notifications\Slack\SlackMessage`. Bạn có thể xây dựng thông báo bằng cách sử dụng [Slack's Block Kit API](https://api.slack.com/block-kit). Ví dụ sau có thể được preview trong [Slack's Block Kit builder](https://app.slack.com/block-kit-builder/T01KWS6K23Z#%7B%22blocks%22:%5B%7B%22type%22:%22header%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Invoice%20Paid%22%7D%7D,%7B%22type%22:%22context%22,%22elements%22:%5B%7B%22type%22:%22plain_text%22,%22text%22:%22Customer%20%231234%22%7D%5D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22An%20invoice%20has%20been%20paid.%22%7D,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20No:*%5Cn1000%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20Recipient:*%5Cntaylor@laravel.com%22%7D%5D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Congratulations!%22%7D%7D%5D%7D):

    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * Get the Slack representation of the notification.
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                    $block->field("*Invoice No:*\n1000")->markdown();
                    $block->field("*Invoice Recipient:*\ntaylor@laravel.com")->markdown();
                })
                ->dividerBlock()
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('Congratulations!');
                });
    }

<a name="slack-interactivity"></a>
### Tương tác với Slack

Hệ thống Block Kit notification system của Slack cung cấp các tính năng mạnh mẽ để [xử lý tương tác với người dùng](https://api.slack.com/interactivity/handling). Để sử dụng các tính năng này, Slack App của bạn phải được bật chế độ "tương tác" và cấu hình "Request URL" trỏ đến URL do ứng dụng của bạn cung cấp. Các cài đặt này có thể được quản lý từ tab quản lý App "Interactivity & Shortcuts" trong Slack.

Trong ví dụ sau sẽ sử dụng phương thức `actionsBlock`, Slack sẽ gửi một request `POST` đến "Request URL" của bạn với một data chứa người dùng Slack đã ấn vào nút, ID của nút đã được ấn và nhiều thông tin khác. Sau đó, ứng dụng của bạn có thể xác định hành động cần thực hiện dựa trên data đó. Và bạn cũng nên [verify request](https://api.slack.com/authentication/verifying-requests-from-slack) được thực hiện bởi Slack:

    use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * Get the Slack representation of the notification.
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                })
                ->actionsBlock(function (ActionsBlock $block) {
                     // ID defaults to "button_acknowledge_invoice"...
                    $block->button('Acknowledge Invoice')->primary();

                    // Manually configure the ID...
                    $block->button('Deny')->danger()->id('deny_invoice');
                });
    }

<a name="slack-confirmation-modals"></a>
#### Confirmation Modals

Nếu bạn muốn người dùng cần xác nhận một hành động trước khi thực hiện, bạn có thể gọi phương thức `confirm` khi định nghĩa nút của bạn. Phương thức `confirm` chấp nhận một thông báo và một closure nhận vào một instance `ConfirmObject`:

    use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * Get the Slack representation of the notification.
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                })
                ->actionsBlock(function (ActionsBlock $block) {
                    $block->button('Acknowledge Invoice')
                        ->primary()
                        ->confirm(
                            'Acknowledge the payment and send a thank you email?',
                            function (ConfirmObject $dialog) {
                                $dialog->confirm('Yes');
                                $dialog->deny('No');
                            }
                        );
                });
    }

<a name="inspecting-slack-blocks"></a>
#### Inspecting Slack Blocks

Nếu bạn muốn kiểm tra qua các block mà bạn đã build, bạn có thể gọi phương thức `dd` trên instance `SlackMessage`. Phương thức `dd` sẽ tạo và dump một URL đến [Block Kit Builder](https://app.slack.com/block-kit-builder/) của Slack, hiển thị bản preview của data và thông báo trong trình duyệt của bạn. Bạn có thể truyền `true` cho phương thức `dd` để dump ra raw data:

    return (new SlackMessage)
            ->text('One of your invoices has been paid!')
            ->headerBlock('Invoice Paid')
            ->dd();

<a name="routing-slack-notifications"></a>
### Routing Slack Notifications

Để chuyển hướng thông báo Slack đến một nhóm và channel Slack phù hợp, hãy định nghĩa phương thức `routeNotificationForSlack` trên model notifiable của bạn. Phương thức này có thể trả về một trong ba giá trị:

- `null` - sẽ giao phó việc chuyển hướng này đến channel được cấu hình trong chính thông báo đó. Bạn có thể sử dụng phương thức `to` khi xây dựng `SlackMessage` để cấu hình channel được chuyển hướng trong thông báo.
- Một string chỉ định channel Slack sẽ nhận thông báo, ví dụ: `#support-channel`.
- Một instance `SlackRoute`, cho phép bạn chỉ định mã token OAuth và tên channel, ví dụ: `SlackRoute::make($this->slack_channel, $this->slack_token)`. Phương thức này nên được sử dụng để gửi thông báo đến các workspace bên ngoài Slack.

Ví dụ, việc trả về `#support-channel` từ phương thức `routeNotificationForSlack` sẽ gửi thông báo đến channel `#support-channel` trong workspace được liên kết với mã token Bot User OAuth nằm trong file cấu hình `services.php` của ứng dụng của bạn:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         */
        public function routeNotificationForSlack(Notification $notification): mixed
        {
            return '#support-channel';
        }
    }

<a name="notifying-external-slack-workspaces"></a>
### Thông báo đến một External Slack Workspaces

> [!NOTE]
> Trước khi gửi thông báo đến các workspace bên ngoài Slack, Slack App của bạn phải được [phân phối](#slack-app-distribution).

Tất nhiên, bạn thường muốn gửi một thông báo đến workspace bên trong Slack do người dùng ứng dụng của bạn sở hữu. Để làm như vậy, trước tiên bạn cần lấy mã token Slack OAuth cho người dùng. Rất may, [Laravel Socialite](/docs/{{version}}/socialite) đã chứa một driver Slack cho phép bạn dễ dàng xác thực người dùng ứng dụng của bạn bằng Slack và [lấy mã token bot](/docs/{{version}}/socialite#slack-bot-scopes) đó một cách dễ dàng.

Sau khi bạn đã có được mã token bot và lưu nó vào trong cơ sở dữ liệu của ứng dụng, bạn có thể sử dụng phương thức `SlackRoute::make` để chuyển thông báo đó đến workspace của người dùng. Ngoài ra, ứng dụng của bạn có thể cần cung cấp cho người dùng cách chỉ định channel thông báo nào sẽ được gửi đến:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Notifications\Slack\SlackRoute;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         */
        public function routeNotificationForSlack(Notification $notification): mixed
        {
            return SlackRoute::make($this->slack_channel, $this->slack_token);
        }
    }

<a name="localizing-notifications"></a>
## Ngôn ngữ trong Notifications

Laravel cho phép bạn gửi các notification bằng ngôn ngữ khác, khác với ngôn ngữ hiện tại của HTTP request và thậm chí là sẽ nhớ ngôn ngữ này nếu notification đang được queue.

Để thực hiện điều này, class `Illuminate\Notifications\Notification` có cung cấp một phương thức `locale` để set ngôn ngữ mà bạn mong muốn. Application sẽ chuyển thành ngôn ngữ này khi định dạng notification và sau đó quay lại về ngôn ngữ trước đó khi quá trình định dạng này hoàn tất:

    $user->notify((new InvoicePaid($invoice))->locale('es'));

Set ngôn ngữ cho nhiều notification cũng có thể đạt được thông qua facade `Notification`:

    Notification::locale('es')->send(
        $users, new InvoicePaid($invoice)
    );

<a name="user-preferred-locales"></a>
### User Preferred Locales

Thỉnh thoảng, các application sẽ lưu lại ngôn ngữ ưa thích của mỗi người dùng. Bằng cách implement contract `HasLocalePreference` trên một model notifiable của bạn, bạn có thể hướng dẫn Laravel sử dụng ngôn ngữ này khi gửi notification:

    use Illuminate\Contracts\Translation\HasLocalePreference;

    class User extends Model implements HasLocalePreference
    {
        /**
         * Get the user's preferred locale.
         */
        public function preferredLocale(): string
        {
            return $this->locale;
        }
    }

Khi bạn đã implement xong interface này, Laravel sẽ tự động sử dụng ngôn ngữ này khi gửi notification và mailable tới model. Do đó, không cần phải gọi phương thức `locale` khi bạn sử dụng interface này:

    $user->notify(new InvoicePaid($invoice));

<a name="testing"></a>
## Testing

Bạn có thể sử dụng phương thức `fake` của facade `Notification` để chặn một thông báo được gửi đi. Thông thường, việc gửi thông báo không liên quan đến code mà bạn đang kiểm tra. Nhiều khả năng, chỉ cần kiểm tra là Laravel đã gửi một thông báo đi là đủ.

Sau khi gọi phương thức `fake` của facade `Notification`, bạn có thể kiểm tra các thông báo đã được gửi đến một người dùng hoặc thậm chí là kiểm tra dữ liệu mà các thông báo đã gửi:

    <?php

    namespace Tests\Feature;

    use App\Notifications\OrderShipped;
    use Illuminate\Support\Facades\Notification;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped(): void
        {
            Notification::fake();

            // Perform order shipping...

            // Assert that no notifications were sent...
            Notification::assertNothingSent();

            // Assert a notification was sent to the given users...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // Assert a notification was not sent...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );

            // Assert that a given number of notifications were sent...
            Notification::assertCount(3);
        }
    }

Bạn có thể truyền một closure cho các phương thức `assertSentTo` hoặc `assertNotSentTo` để kiểm tra một thông báo đã được gửi đi và pass qua được "truth test" đã cho. Nếu có ít nhất một thông báo đã được gửi đi và pass được bài kiểm tra truth test đã cho thì kiểm tra đó sẽ thành công:

    Notification::assertSentTo(
        $user,
        function (OrderShipped $notification, array $channels) use ($order) {
            return $notification->order->id === $order->id;
        }
    );

<a name="on-demand-notifications"></a>
#### On-Demand Notifications

Nếu code bạn đang kiểm tra gửi một [thông báo theo yêu cầu](#on-demand-notifications), bạn có thể kiểm tra xem thông báo đó có được gửi đi hay không qua phương thức `assertSentOnDemand`:

    Notification::assertSentOnDemand(OrderShipped::class);

Bằng cách truyền vào một closure làm tham số thứ hai cho phương thức `assertSentOnDemand`, bạn có thể xác định xem thông báo đó có được gửi đến đúng địa chỉ "route" hay không:

    Notification::assertSentOnDemand(
        OrderShipped::class,
        function (OrderShipped $notification, array $channels, object $notifiable) use ($user) {
            return $notifiable->routes['mail'] === $user->email;
        }
    );

<a name="notification-events"></a>
## Notification Events

<a name="notification-sending-event"></a>
#### Notification Sending Event

Khi một thông báo đang được gửi, [event](/docs/{{version}}/events) `Illuminate\Notifications\Events\NotificationSending` sẽ được gửi bởi notification system. Nó sẽ chứa thực thể "notifiable" và một instance notification. Bạn có thể đăng ký listener cho các event này trong `EventServiceProvider`:

    use App\Listeners\CheckNotificationStatus;
    use Illuminate\Notifications\Events\NotificationSending;

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        NotificationSending::class => [
            CheckNotificationStatus::class,
        ],
    ];

Thông báo sẽ không được gửi nếu event listener cho event `NotificationSending` này trả về `false` từ phương thức `handle` của nó:

    use Illuminate\Notifications\Events\NotificationSending;

    /**
     * Handle the event.
     */
    public function handle(NotificationSending $event): bool
    {
        return false;
    }

Trong event listener này, bạn có thể truy cập vào các thuộc tính `notifiable`, `notification` và `channel` trên event để biết thêm thông tin về người nhận notification hoặc chính notification đó:

    /**
     * Handle the event.
     */
    public function handle(NotificationSending $event): void
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="notification-sent-event"></a>
#### Notification Sent Event

Khi một notification đã được gửi, thì một [event](/docs/{{version}}/events) `Illuminate\Notifications\Events\NotificationSent` sẽ được gửi bởi notification system. Nó sẽ chứa thực thể "notifiable" và một instance notification. Bạn có thể đăng ký listener cho các event này trong `EventServiceProvider`:

    use App\Listeners\LogNotification;
    use Illuminate\Notifications\Events\NotificationSent;

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        NotificationSent::class => [
            LogNotification::class,
        ],
    ];

> [!NOTE]
> Sau khi đăng ký listener trong `EventServiceProvider`, hãy sử dụng lệnh Artisan `event:generate` để tạo ra các class listener.

Trong một event listener, bạn có thể truy cập vào các thuộc tính `notifiable`, `notification`, `channel`, và `response` trong event để biết thêm thông tin về người nhận notification hoặc chính notification đó:

    /**
     * Handle the event.
     */
    public function handle(NotificationSent $event): void
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
        // $event->response
    }

<a name="custom-channels"></a>
## Tuỳ biến Channels

Laravel có sẵn với một số notification channel, nhưng bạn có thể muốn viết thêm các driver khác để gửi notification qua các channel riêng của bạn. Laravel làm cho nó trở nên rất đơn giản. Để bắt đầu, hãy định nghĩa một class có chứa phương thức `send`. Phương thức sẽ nhận vào hai tham số: một là `$notifiable` và một là `$notification`.

Trong phương thức `send`, bạn có thể gọi các phương thức trên notification để lấy ra đối tượng message bằng channel của bạn và gửi notification đó đến instance `$notifiable` theo cách bạn muốn:

    <?php

    namespace App\Notifications;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Send the given notification.
         */
        public function send(object $notifiable, Notification $notification): void
        {
            $message = $notification->toVoice($notifiable);

            // Send notification to the $notifiable instance...
        }
    }

Khi class notification channel của bạn đã được định nghĩa, bạn có thể trả về tên class từ phương thức `via` của bất kỳ notifications nào của bạn. Trong ví dụ này, phương thức `toVoice` của notification của bạn có thể trả về bất kỳ đối tượng nào mà bạn chọn để thể hiện tin nhắn. Ví dụ: bạn có thể định nghĩa class `VoiceMessage` của riêng bạn để thể hiện những notification này:

    <?php

    namespace App\Notifications;

    use App\Notifications\Messages\VoiceMessage;
    use App\Notifications\VoiceChannel;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Get the notification channels.
         */
        public function via(object $notifiable): string
        {
            return VoiceChannel::class;
        }

        /**
         * Get the voice representation of the notification.
         */
        public function toVoice(object $notifiable): VoiceMessage
        {
            // ...
        }
    }
