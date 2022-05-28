# Laravel Envoy

- [Giới thiệu](#introduction)
    - [Cài đặt](#installation)
- [Viết Task](#writing-tasks)
    - [Thiết lập](#setup)
    - [Biến](#variables)
    - [Stories](#stories)
    - [Multiple Servers](#multiple-servers)
- [Chạy Task](#running-tasks)
    - [Xác nhận Task chạy](#confirming-task-execution)
- [Thông báo](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)

<a name="introduction"></a>
## Giới thiệu

[Laravel Envoy](https://github.com/laravel/envoy) sẽ cung cấp một cú pháp rõ ràng và tối thiểu để định nghĩa các task phổ biến mà bạn hay chạy trên các server. Sử dụng cú pháp theo kiểu Blade, bạn có thể dễ dàng thiết lập các task để deploy, các lệnh Artisan và hơn thế nữa. Hiện tại, Envoy chỉ hỗ trợ trên hệ điều hành Mac và Linux.

<a name="installation"></a>
### Cài đặt

Đầu tiên, cài đặt Envoy bằng lệnh Composer `global require`:

    composer global require laravel/envoy

Vì các thư viện Composer global đôi khi có thể gây ra xung đột phiên bản package, nên bạn có thể muốn xem xét sử dụng `cgr`, đây là một thay thế cho lệnh `composer global require`. Hướng dẫn cài đặt của thư viện `cgr` này có thể [được tìm thấy trên GitHub](https://github.com/consolidation-org/cgr).

> {note} Hãy đảm bảo là bạn đã set link của thư mục `$HOME/.config/composer/vendor/bin` hoặc `$HOME/.composer/vendor/bin` vào trong PATH của bạn để có thể chạy lệnh `envoy` trong terminal của bạn.

#### Updating Envoy

Bạn cũng có thể sử dụng Composer để cập nhật Envoy. Chạy lệnh `composer global update` sẽ cập nhật tất cả các package Composer global đã được cài đặt trong máy của bạn:

    composer global update

<a name="writing-tasks"></a>
## Viết Task

Tất cả các task Envoy của bạn phải được định nghĩa trong file `Envoy.blade.php` ở trong thư mục gốc của project của bạn. Đây là một ví dụ để giúp bạn bắt đầu:

    @servers(['web' => ['user@192.168.1.1']])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

Như bạn có thể thấy, một mảng `@servers` sẽ được định nghĩa ở đầu file, cho phép bạn tham chiếu đến các server này trong tùy chọn `on` trong task của bạn. Khai báo `@server` phải luôn được viết trên một dòng. Ở trong khai báo `@task` của bạn, bạn nên viết các lệnh Bash code sẽ được chạy trên server của bạn khi task được thực thi.

Bạn có thể bắt buộc một tập lệnh phải chạy ở local bằng cách khai báo địa chỉ IP của server là `127.0.0.1`:

    @servers(['localhost' => '127.0.0.1'])

<a name="setup"></a>
### Thiết lập

Thỉnh thoảng, bạn có thể cần phải thực thi một số code PHP trước khi thực hiện các task của Envoy. Bạn có thể sử dụng lệnh `@setup` để khai báo các biến và thực hiện các công việc chung khác của PHP trước khi bất kỳ task nào của bạn được thực thi:

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

Nếu bạn cần thêm các file PHP khác trước khi task của bạn được thực thi, bạn có thể sử dụng lệnh `@include` ở đầu file `Envoy.blade.php` của bạn:

    @include('vendor/autoload.php')

    @task('foo')
        # ...
    @endtask

Bạn cũng có thể import các file Envoy khác để story và task của các file đó được thêm vào Envoy của bạn. Sau khi các file đã được import, bạn có thể thực thi các task trong các file đó như thể chúng được định nghĩa cho bạn. Bạn nên sử dụng lệnh `@import` ở đầu file `Envoy.blade.php`:

    @import('package/Envoy.blade.php')

<a name="variables"></a>
### Biến

Nếu cần, bạn có thể truyền các giá trị tùy chọn vào các task của Envoy bằng lệnh:

    envoy run deploy --branch=master

Bạn có thể truy cập đến các tùy chọn này trong task của bạn thông qua cú pháp "echo" của Blade. Bạn cũng có thể sử dụng các câu lệnh `if` và vòng lặp trong các task của bạn. Ví dụ, hãy kiểm tra sự tồn tại của biến `$branch` trước khi thực hiện lệnh `git pull`:

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site

        @if ($branch)
            git pull origin {{ $branch }}
        @endif

        php artisan migrate
    @endtask

<a name="stories"></a>
### Stories

Stories group là một nhóm các task với một tên duy nhất, cho phép bạn nhóm các task nhỏ lại với nhau thành một task lớn. Chẳng hạn như, một story `deploy` có thể chạy các task `git` và `composer` bằng cách liệt kê tên của các task trong định nghĩa của nó:

    @servers(['web' => '192.168.1.1'])

    @story('deploy')
        git
        composer
    @endstory

    @task('git')
        git pull origin master
    @endtask

    @task('composer')
        composer install
    @endtask

Khi story đã được viết xong, bạn có thể chạy nó giống như chạy một task bình thường:

    envoy run deploy

<a name="multiple-servers"></a>
### Multiple Servers

Envoy cho phép bạn dễ dàng chạy một task trên nhiều server. Đầu tiên, thêm các server vào khai báo `@servers` của bạn. Mỗi server nên được gán với một tên duy nhất. Khi bạn đã định nghĩa xong các server cần thêm, bạn hãy liệt kê từng server vào trong mảng `on` của task:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

#### Parallel Execution

Mặc định, các task sẽ được chạy tuần tự trên mỗi server. Nói cách khác, task sẽ kết thúc chạy trên server đầu tiên và sau đó nó tiến tục thực hiện trên server thứ hai. Nếu bạn muốn chạy song song một task trên nhiều server, hãy thêm tùy chọn `parallel` vào khai báo task của bạn:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="running-tasks"></a>
## Chạy Task

Để chạy một task hoặc một story đã được định nghĩa trong file `Envoy.blade.php`, bạn hãy chạy lệnh `run` của Envoy, và truyền tên của task hoặc tên của story mà bạn muốn thực hiện. Envoy sẽ chạy task và hiển thị output từ server khi task được chạy:

    envoy run deploy

<a name="confirming-task-execution"></a>
### Xác nhận Task chạy

Nếu bạn muốn được nhắc xác nhận trước khi chạy một task nào đó trên server của bạn, bạn nên thêm lệnh `confirm` vào khai báo task của bạn. Tùy chọn này đặc biệt hữu ích khi chạy các hoạt động delete:

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="notifications"></a>
## Thông báo

<a name="slack"></a>
### Slack

Envoy cũng hỗ trợ gửi thông báo tới [Slack](https://slack.com) sau khi mỗi task đã được chạy xong. Lệnh `@slack` sẽ chấp nhận một URL hook của Slack và một tên channel. Bạn có thể lấy URL webhook của bạn bằng cách tạo một "Incoming WebHooks" ở trong bảng control panel của Slack. Bạn nên truyền toàn bộ URL webhook vào lệnh `@slack`:

    @finished
        @slack('webhook-url', '#bots')
    @endfinished

Bạn có thể cung cấp thêm một trong số lựa chọn sau đây để làm tham số cho channel:

<div class="content-list" markdown="1">

- Để gửi thông báo tới một channel: `#channel`
- Để gửi thông báo cho một người dùng: `@user`

</div>

<a name="discord"></a>
### Discord

Envoy cũng hỗ trợ gửi thông báo đến [Discord](https://discord.com) sau mỗi task được chạy. Lệnh `@discord` chấp nhận một URL hook Discord và một thông báo. Bạn có thể lấy ra URL webhook của bạn bằng cách tạo "Webhook" trong Server Setting và chọn channel mà webhook sẽ đăng lên. Bạn nên truyền toàn bộ URL Webhook vào lệnh `@discord`:

    @finished
        @discord('discord-webhook-url')
    @endfinished

<a name="telegram"></a>
### Telegram

Envoy cũng hỗ trợ gửi thông báo tới [Telegram](https://telegram.org) sau mỗi task được thực thi. Lệnh `@telegram` chấp nhận một Telegram Bot ID và một Chat ID. Bạn có thể lấy ra Bot ID của bạn bằng cách tạo một bot mới bằng [BotFather](https://t.me/botfather) Bạn có thể lấy ra Chat ID bằng cách [@username_to_id_bot](https://t.me/username_to_id_bot). Bạn nên truyền toàn bộ Bot ID và Chat ID vào lệnh `@telegram`:

    @finished
        @telegram('<bot-id>','<chat-id>')
    @endfinished
