# Laravel Envoy

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Viết Task](#writing-tasks)
    - [Định nghĩa Task](#defining-tasks)
    - [Nhiều Server](#multiple-servers)
    - [Thiết lập](#setup)
    - [Biến](#variables)
    - [Stories](#stories)
    - [Hooks](#completion-hooks)
- [Chạy Task](#running-tasks)
    - [Xác nhận Task chạy](#confirming-task-execution)
- [Thông báo](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)
    - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>
## Giới thiệu

[Laravel Envoy](https://github.com/laravel/envoy) là công cụ để thực hiện các task phổ biến mà bạn hay chạy trên các server. Sử dụng cú pháp theo kiểu [Blade](/docs/{{version}}/blade), bạn có thể dễ dàng thiết lập các task để deploy, các lệnh Artisan và hơn thế nữa. Hiện tại, Envoy chỉ hỗ trợ trên hệ điều hành Mac và Linux. Tuy nhiên, có thể hỗ trợ Windows bằng cách sử dụng [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt Envoy vào project của bạn bằng trình quản lý package Composer:

    composer require laravel/envoy --dev

Khi Envoy đã được cài đặt xong, file Envoy binary sẽ ở trong thư mục `vendor/bin` của ứng dụng của bạn:

    php vendor/bin/envoy

<a name="writing-tasks"></a>
## Viết Task

<a name="defining-tasks"></a>
### Định nghĩa Task

Task là khối cơ bản của Envoy. Các task định nghĩa các shell command mà sẽ thực thi trên các server của bạn khi task được gọi. Ví dụ: bạn có thể định nghĩa một task thực thi lệnh `php artisan queue:restart` trên tất cả các queue server worker của ứng dụng của bạn.

Tất cả các task Envoy của bạn phải được định nghĩa trong file `Envoy.blade.php` ở thư mục root trong ứng dụng của bạn. Đây là một ví dụ mẫu để giúp bạn bắt đầu:

```bash
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

Như bạn có thể thấy, một mảng `@servers` sẽ được định nghĩa ở đầu file, cho phép bạn tham chiếu đến các server này thông qua tùy chọn `on` trong task của bạn. Khai báo `@server` phải luôn được viết trên một dòng. Ở trong khai báo `@task` của bạn, bạn nên viết các shell command sẽ được thực thi trên server của bạn khi task được gọi.

<a name="local-tasks"></a>
#### Local Tasks

Bạn có thể bắt buộc một tập lệnh phải chạy ở local bằng cách khai báo địa chỉ IP của server là `127.0.0.1`:

```bash
@servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>
#### Importing Envoy Tasks

Sử dụng lệnh `@import`, bạn có thể import các file Envoy khác để story và task của file đó được thêm vào story và task của bạn. Sau khi các file đã được import, bạn có thể thực thi các task trong các file đó như thể chúng được định nghĩa trong file Envoy của bạn:

```bash
@import('vendor/package/Envoy.blade.php')
```

<a name="multiple-servers"></a>
### Nhiều Server

Envoy cho phép bạn dễ dàng chạy một task trên nhiều server. Trước tiên, hãy thêm các server vào phần khai báo `@servers` của bạn. Mỗi server nên được gán một tên duy nhất. Khi bạn đã định nghĩa xong các server của bạn, bạn có thể liệt kê từng server đó trong mảng `on` của task:

```bash
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="parallel-execution"></a>
#### Parallel Execution

Mặc định, các task sẽ được thực thi theo thứ tự trên từng server. Nói cách khác, một task sẽ cần phải chạy xong trên server đầu tiên trước khi tiến hành thực thi trên server thứ hai. Nếu bạn muốn chạy song song một task trên nhiều server, thì hãy thêm tùy chọn `parallel` vào phần khai báo task của bạn:

```bash
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="setup"></a>
### Thiết lập

Thỉnh thoảng, bạn có thể cần phải thực thi một số code PHP trước khi chạy các task của Envoy. Bạn có thể sử dụng lệnh `@setup` để định nghĩa một block code PHP sẽ thực thi trước các task của bạn:

```php
@setup
    $now = new DateTime;
@endsetup
```

Nếu bạn cần thêm các file PHP khác trước khi task của bạn được thực thi, bạn có thể sử dụng lệnh `@include` ở đầu file `Envoy.blade.php` của bạn:

```bash
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

<a name="variables"></a>
### Biến

Nếu cần, bạn có thể truyền các tham số cho các task Envoy bằng cách chỉ định chúng trên command line:

    php vendor/bin/envoy run deploy --branch=master

Bạn có thể truy cập đến các tùy chọn này trong task của bạn bằng cú pháp "echo" của Blade. Bạn cũng có thể định nghĩa các câu lệnh Blade `if` và vòng lặp trong các task của bạn. Ví dụ, hãy kiểm tra sự tồn tại của biến `$branch` trước khi thực hiện lệnh `git pull`:

```bash
@servers(['web' => ['user@192.168.1.1']])

@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

<a name="stories"></a>
### Stories

Stories group là một nhóm các task với một tên duy nhất. Chẳng hạn như, một story `deploy` có thể chạy các task `update-code` và `install-dependencies` bằng cách liệt kê tên của các task trong định nghĩa của nó:

```bash
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

Khi story đã được viết xong, bạn có thể gọi nó giống như cách bạn gọi một task:

    php vendor/bin/envoy run deploy

<a name="completion-hooks"></a>
### Hooks

Khi các task và các story chạy, thì một số hook sẽ được thực thi. Các loại hook được Envoy hỗ trợ là `@before`, `@after`, `@error`, `@success`, và `@finished`. Tất cả các code trong các hook này sẽ được hiểu là code PHP và được thực thi local, không phải trên các server mà các task của bạn chạy.

Bạn có thể định nghĩa bao nhiêu hook tùy thích. Chúng sẽ được thực thi theo thứ tự xuất hiện trong script Envoy của bạn.

<a name="hook-before"></a>
#### `@before`

Trước mỗi lần thực hiện task, tất cả các hook `@before` đã được đăng ký trong script Envoy của bạn sẽ thực thi. Các hook `@before` sẽ nhận vào tên của task sẽ được thực thi:

```php
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>
#### `@after`

Sau mỗi lần thực hiện task, tất cả các hook `@after` đã được đăng ký trong script Envoy của bạn sẽ thực thi. Các hook `@after` sẽ nhận vào tên của task đã được thực thi:

```php
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>
#### `@error`

Sau mỗi lần task thất bại (exit với status code lớn hơn `0`), thì tất cả các hook `@error` đã được đăng ký trong script Envoy của bạn sẽ thực thi. Các hook `@error` sẽ nhận vào tên của task đã được thực

```php
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

<a name="completion-success"></a>
#### `@success`

Nếu tất cả các task đã được thực thi mà không có lỗi nào, thì tất cả các hook `@success` đã được đăng ký trong script Envoy của bạn sẽ được thực thi:

```bash
@success
    // ...
@endsuccess
```

<a name="completion-finished"></a>
#### `@finished`

Sau khi tất cả các task đã được thực thi (bất kể exit status nào), thì tất cả các hook `@finished` sẽ được thực thi. Hook `@finished` sẽ nhận vào status code của task đã hoàn thành, có thể là `null` hoặc `integer` lớn hơn hoặc bằng `0`:

```bash
@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

<a name="running-tasks"></a>
## Chạy Task

Để chạy một task hoặc một story đã được định nghĩa trong file `Envoy.blade.php` của application của bạn, bạn hãy chạy lệnh `run` của Envoy, và truyền tên của task hoặc tên của story mà bạn muốn thực hiện. Envoy sẽ thực hiện task và hiển thị output từ server remote của bạn khi task được chạy:

    php vendor/bin/envoy run deploy

<a name="confirming-task-execution"></a>
### Xác nhận Task chạy

Nếu bạn muốn được nhắc xác nhận trước khi chạy một task nào đó trên server của bạn, bạn nên thêm lệnh `confirm` vào khai báo task của bạn. Tùy chọn này đặc biệt hữu ích khi chạy các hoạt động delete:

```bash
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

<a name="notifications"></a>
## Thông báo

<a name="slack"></a>
### Slack

Envoy hỗ trợ gửi thông báo tới [Slack](https://slack.com) sau khi mỗi task đã được chạy xong. Lệnh `@slack` sẽ chấp nhận một URL hook của Slack và một channel / user name. Bạn có thể lấy URL webhook của bạn bằng cách tạo một "Incoming WebHooks" ở trong bảng control panel của Slack.

Bạn nên truyền toàn bộ URL webhook làm tham số đầu tiên của lệnh `@slack`. Tham số thứ hai được cung cấp cho lệnh `@slack` phải là tên channel (`#channel`) hoặc tên người dùng (`@user`):

    @finished
        @slack('webhook-url', '#bots')
    @endfinished

Mặc định, Envoy notification sẽ gửi một thông báo đến notification channel mô tả task đã được thực hiện. Tuy nhiên, bạn có thể ghi đè thông báo này bằng thông báo tùy chỉnh của riêng bạn bằng cách truyền tham số thứ ba cho lệnh `@slack`:

    @finished
        @slack('webhook-url', '#bots', 'Hello, Slack.')
    @endfinished

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
        @telegram('bot-id','chat-id')
    @endfinished

<a name="microsoft-teams"></a>
### Microsoft Teams

Envoy cũng hỗ trợ gửi thông báo tới [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) sau khi mỗi task được thực thi. Lệnh `@microsoftTeams` chấp nhận Teams Webhook (bắt buộc), thông báo, theme (success, info, warning, error) và một mảng các tùy chọn. Bạn có thể lấy Teams Webook của bạn bằng cách tạo một [webhook mới](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook). Teams API có nhiều thuộc tính khác để tùy chỉnh thông báo của bạn như tiêu đề, tổng hợp và các phần. Bạn có thể tìm thêm thông tin trên [tài liệu Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message). Bạn nên truyền toàn bộ URL Webhook vào lệnh `@microsoftTeams`:

    @finished
        @microsoftTeams('webhook-url')
    @endfinished
