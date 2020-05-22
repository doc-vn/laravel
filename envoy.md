# Envoy Task Runner

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

<a name="introduction"></a>
## Giới thiệu

[Laravel Envoy](https://github.com/laravel/envoy) cung cấp một cú pháp rõ ràng, tối thiểu để định nghĩa các tác vụ phổ biến mà bạn hay chạy trên các remote server của bạn. Sử dụng cú pháp theo kiểu Blade, bạn có thể dễ dàng thiết lập các tác vụ để deploy, các lệnh Artisan và hơn thế nữa. Hiện tại, Envoy chỉ hỗ trợ hệ điều hành Mac và Linux.

<a name="installation"></a>
### Cài đặt

Đầu tiên, cài đặt Envoy bằng lệnh Composer `global require`:

    composer global require laravel/envoy

Vì các thư viện Composer global đôi khi có thể gây ra xung đột phiên bản package, nên bạn có thể muốn xem xét sử dụng `cgr`, đây là một thay thế cho lệnh `composer global require`. Hướng dẫn cài đặt của thư viện `cgr` có thể [được tìm thấy trên GitHub](https://github.com/consolidation-org/cgr).

> {note} Đảm bảo bạn đã đặt thư mục `~/.composer/vendor/bin` trong PATH của bạn để có thể chạy lệnh `envoy` trong terminal của bạn.

#### Updating Envoy

Bạn cũng có thể sử dụng Composer để cập nhật cài đặt Envoy. Chạy lệnh `composer global update` sẽ cập nhật tất cả các package Composer global đã được cài đặt trong máy của bạn:

    composer global update

<a name="writing-tasks"></a>
## Viết Task

Tất cả các tác vụ Envoy của bạn phải được định nghĩa trong file `Envoy.blade.php` ở trong thư mục gốc của project của bạn. Đây là một ví dụ để giúp bạn bắt đầu:

    @servers(['web' => ['user@192.168.1.1']])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

Như bạn có thể thấy, một mảng `@servers` sẽ được định nghĩa ở đầu file, cho phép bạn tham chiếu đến các server này trong tùy chọn `on` của các khai báo tác vụ của bạn. Trong các khai báo `@task` của bạn, bạn có thể dùng lệnh Bash sẽ được chạy trên server của bạn khi tác vụ được thực thi.

Bạn có thể bắt buộc một tập lệnh phải chạy ở local bằng cách khai báo địa chỉ IP của server là `127.0.0.1`:

    @servers(['localhost' => '127.0.0.1'])

<a name="setup"></a>
### Thiết lập

Thỉnh thoảng, bạn có thể cần phải thực thi một số code PHP trước khi thực hiện các tác vụ của Envoy. Bạn có thể sử dụng lệnh ```@setup``` để khai báo các biến và thực hiện các công việc PHP chung khác trước khi bất kỳ tác vụ nào của bạn được thực thi:

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

Nếu bạn cần yêu cầu các file PHP khác trước khi tác vụ của bạn được thực thi, bạn có thể sử dụng lệnh `@include` ở đầu file `Envoy.blade.php` của bạn:

    @include('vendor/autoload.php')

    @task('foo')
        # ...
    @endtask

<a name="variables"></a>
### Biến

Nếu cần, bạn có thể pass các giá trị tùy chọn vào các tác vụ của Envoy bằng lệnh:

    envoy run deploy --branch=master

Bạn có thể truy cập đến các tùy chọn trong tác vụ của bạn thông qua cú pháp "echo" của Blade. Tất nhiên, bạn cũng có thể sử dụng các câu lệnh và vòng lặp `if` trong các tác vụ của bạn. Ví dụ, hãy kiểm tra sự tồn tại của biến `$branch` trước khi thực hiện lệnh `git pull`:

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

Stories group là một nhóm các tác vụ chỉ với một tên, cho phép bạn nhóm các tác vụ nhỏ, tập trung thành các tác vụ lớn. Chẳng hạn, một story `deploy` có thể chạy các tác vụ `git` và `composer` bằng cách liệt kê tên của các tác vụ trong định nghĩa của nó:

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

Khi story đã được viết xong, bạn có thể chạy nó giống như một tác vụ bình thường:

    envoy run deploy

<a name="multiple-servers"></a>
### Multiple Servers

Envoy cho phép bạn dễ dàng chạy một tác vụ trên nhiều server. Đầu tiên, thêm các server bổ sung vào khai báo `@servers` của bạn. Mỗi server nên được gán với một tên duy nhất. Khi bạn đã định nghĩa xong các server cần thêm của bạn, hãy liệt kê từng server trong mảng `on` của tác vụ:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

#### Parallel Execution

Mặc định, các tác vụ sẽ được chạy trên mỗi server. Nói cách khác, một tác vụ sẽ kết thúc chạy trên server đầu tiên trước khi tiến tục thực hiện trên server thứ hai. Nếu bạn muốn chạy một tác vụ trên nhiều server song song, hãy thêm tùy chọn `parallel` vào khai báo tác vụ của bạn:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="running-tasks"></a>
## Chạy Task

Để chạy một tác vụ hoặc story đã được định nghĩa trong file `Envoy.blade.php` của bạn, hãy chạy lệnh `run` của Envoy, pass tên của tác vụ hoặc story mà bạn muốn thực hiện. Envoy sẽ chạy tác vụ và hiển thị output từ server khi tác vụ đang chạy:

    envoy run task

<a name="confirming-task-execution"></a>
### Xác nhận Task chạy

Nếu bạn muốn được nhắc xác nhận trước khi chạy một tác vụ nào đó trên server của bạn, bạn nên thêm lệnh `confirm` vào khai báo tác vụ của bạn. Tùy chọn này đặc biệt hữu ích cho các hoạt động huỷ hoại:

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="notifications"></a>
<a name="hipchat-notifications"></a>
## Thông báo

<a name="slack"></a>
### Slack

Envoy cũng hỗ trợ gửi thông báo tới [Slack](https://slack.com) sau khi mỗi tác vụ đã được chạy. Lệnh `@slack` chấp nhận một URL hook của Slack và một tên channel. Bạn có thể lấy URL webhook của bạn bằng cách tạo tích hợp "Incoming WebHooks" trong bảng control panel của Slack của bạn. Bạn nên pass toàn bộ URL webhook vào lệnh `@slack`:

    @finished
        @slack('webhook-url', '#bots')
    @endfinished

Bạn có thể cung cấp một trong những điều sau đây làm tham số kênh:

<div class="content-list" markdown="1">
- Để gửi thông báo tới một kênh: `#channel`
- Để gửi thông báo cho một người dùng: `@user`
</div>

