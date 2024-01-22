# Cấu trúc thư mục

- [Giới thiệu](#introduction)
- [Thư mục gốc](#the-root-directory)
    - [Thư mục `app`](#the-root-app-directory)
    - [Thư mục `bootstrap`](#the-bootstrap-directory)
    - [Thư mục `config`](#the-config-directory)
    - [Thư mục `database`](#the-database-directory)
    - [Thư mục `public`](#the-public-directory)
    - [Thư mục `resources`](#the-resources-directory)
    - [Thư mục `routes`](#the-routes-directory)
    - [Thư mục `storage`](#the-storage-directory)
    - [Thư mục `tests`](#the-tests-directory)
    - [Thư mục `vendor`](#the-vendor-directory)
- [Thư mục App](#the-app-directory)
    - [Thư mục `Broadcasting`](#the-broadcasting-directory)
    - [Thư mục `Console`](#the-console-directory)
    - [Thư mục `Events`](#the-events-directory)
    - [Thư mục `Exceptions`](#the-exceptions-directory)
    - [Thư mục `Http`](#the-http-directory)
    - [Thư mục `Jobs`](#the-jobs-directory)
    - [Thư mục `Listeners`](#the-listeners-directory)
    - [Thư mục `Mail`](#the-mail-directory)
    - [Thư mục `Models`](#the-models-directory)
    - [Thư mục `Notifications`](#the-notifications-directory)
    - [Thư mục `Policies`](#the-policies-directory)
    - [Thư mục `Providers`](#the-providers-directory)
    - [Thư mục `Rules`](#the-rules-directory)

<a name="introduction"></a>
## Giới thiệu

Cấu trúc thư mục mặc định của Laravel nhằm cung cấp một khởi đầu tốt cho tất cả các application lớn và nhỏ. Nhưng bạn có thể tự tổ chức theo cách mà bạn muốn. Laravel sẽ gần như không áp đặt một hạn chế nào về mặt vị trí cho bất cứ class nào, miễn là Composer có thể load class đó.

<a name="the-root-directory"></a>
## Thư mục gốc

<a name="the-root-app-directory"></a>
#### Thư mục App

Thư mục `app` là nơi chứa phần core code của application. Chúng tôi sẽ sớm chia thư mục này ra, cho rõ ràng hơn, nhưng hầu như tất cả các class sẽ nằm trong thư mục này.

<a name="the-bootstrap-directory"></a>
#### Thư mục Bootstrap

Thư mục `bootstrap` sẽ chứa file `app.php` dành cho khởi động framework. Thư mục này cũng chứa thư mục `cache` dành cho framework, tạo ra các file để tối ưu tốc độ cho route và service. Bạn thường không cần phải sửa bất kỳ file nào trong thư mục này.

<a name="the-config-directory"></a>
#### Thư mục Config

Thư mục `config` mang ý nghĩa rất dễ hiểu, nó dùng để chứa tất cả các file config cho application của bạn. Nó là một ý tưởng tốt để đọc tất cả các file và làm quen với các biến có sẵn trong đó.

<a name="the-database-directory"></a>
#### Thư mục Database

Thư mục `database` chứa các file migration cho database, các file factories để tạo fake data cho model, và các file seed. Nếu bạn muốn, bạn cũng có thể dùng thư mục này để chứa các file SQLite database.

<a name="the-public-directory"></a>
#### Thư mục Public

Thư mục `public` chứa file `index.php` là điểm khởi đầu vào cho mọi request gửi tới application của bạn và các file config autoloading. Thư mục này cũng chứa các file images, JavaScript, và CSS.

<a name="the-resources-directory"></a>
#### Thư mục Resources

Thư mục `resources` chứa file [view](/docs/{{version}}/views) cũng như file raw, và các file chưa được biên dịch như là CSS hoặc JavaScript. Thư mục này cũng chứa những file language.

<a name="the-routes-directory"></a>
#### Thư mục Routes

Thư mục `routes` chứa tất cả các file định nghĩa route cho application của bạn. Mặc đinh, sẽ bao gồm những file sau đây: `web.php`, `api.php`, `console.php`, và `channels.php`.

File `web.php` sẽ chứa những route mà được load bởi file `RouteServiceProvider` và lưu trữ những route đó vào trong một group middleware có tên là `web`, middleware này cung cấp session, bảo vệ route trước các cuộc tấn công CSRF và mã hoá cookie. Nếu application của bạn chỉ dùng session và không dùng RESTful API thì có khả năng là tất cả route của bạn có thế được định nghĩa trong file `web.php` duy nhất.

File `api.php` chứa những route mà được load bởi file `RouteServiceProvider` và lưu trữ những route đó vào trong một group middleware có tên là `api`. Những route này sẽ được chủ đích là không dùng session, vì vậy request đến application của bạn thông qua những route này sẽ được authenticated [thông qua token](/docs/{{version}}/sanctum) và không có quyền truy cập vào session.

File `console.php` là nơi bạn có thể định nghĩa tất cả các [closure](https://www.php.net/manual/en/functions.anonymous.php) dựa trên các lệnh chạy ở console. Mỗi closure sẽ tương ứng với một câu lệnh, nên bạn sẽ dễ dàng tiếp cận được với các phương thức input và output của mỗi câu lệnh. File này sẽ không định nghĩa các Http route của application, mà nó chỉ định nghĩa các console route tới application của bạn.

File `channels.php` sẽ là nơi mà bạn có thể đăng ký các tất cả các [event broadcasting](/docs/{{version}}/broadcasting) channel mà application bạn hỗ trợ.

<a name="the-storage-directory"></a>
#### Thư mục Storage

Thư mục `storage` sẽ chứa những file log, Blade đã được biên dịch, file session, file cache, và các file khác được tạo ra bởi framework. Thư mục này sẽ chứa các thư mục con là: `app`, `framework`, và `logs`. Thư mục `app` có thể dùng để lưu tất cả các file được tạo ra bởi application của bạn. Thư mục `framework` sẽ dùng để lưu các file được tạo ra bởi framework và cache, Cuối cùng là thư mục `logs`, được dùng để chứa các file log của application.

Thư mục `storage/app/public` có thể được dùng để lưu trữ các file mà user tạo như là: avatars, những loại mà được phép public. Bạn cũng nên tạo một link ảo `public/storage` để trỏ vào thư mục này. Bạn có thể tạo link ảo đó bằng câu lệnh Artisan sau: `php artisan storage:link`.

<a name="the-tests-directory"></a>
#### Thư mục Tests

Thư mục `tests` sẽ chứa các file test tự động. Mặc định, một ví dụ [PHPUnit](https://phpunit.de/) unit test và một bài test chức năng đã được cung cấp sẵn trong project. Mỗi class test nên lưu lại với hậu tố là `Test`. Bạn có thể chạy test của bạn bằng câu lệnh `phpunit` hoặc `php vendor/bin/phpunit`. Hoặc, nếu bạn muốn hiển thị chi tiết hơn và đẹp mắt hơn về kết quả test của bạn, bạn có thể chạy test của bạn bằng lệnh Artisan `php artisan test`.

<a name="the-vendor-directory"></a>
#### Thư mục Vendor

Thư mục `vendor` chứa những library mà được quản lý bởi [Composer](https://getcomposer.org).

<a name="the-app-directory"></a>
## Thư mục App

Phần lớn application của bạn sẽ được lưu trong thư mục `app`. Mặc định, thư mục này sẽ được lưu dưới tên là `App` và được autoloaded bởi Composer dùng chuẩn [PSR-4 autoloading standard](https://www.php-fig.org/psr/psr-4/).

Thư mục `app` sẽ chứa một số thư mục bổ sung như `Console`, `Http`, và `Providers`. Hãy nghĩ các thư mục `Console` và `Http` như là các thư mục cung cấp API cho phần core của application của bạn. Giao thức HTTP và CLI đều là các cơ chế để bên ngoài tương tác với application của bạn, nhưng thực tế chúng lại không hay chứa logic của application. Nói cách khác, chúng là hai cách để gọi đến application của bạn. Thư mục `Console` chứa tất cả các lệnh Artisan của bạn, và thư mục `Http` chứa các class controllers, middleware, và requests của bạn.

Các thư mục khác sẽ được tạo trong thư mục `app` khi bạn dùng lệnh Artisan `make` để tạo các class tương ứng với thư mục đó. Ví dụ, bình thường, thư mục `app/Jobs` sẽ không tồn tại cho đến khi bạn chạy lệnh Artisan `make:job` để tạo class job.

> {tip} Nhiều class trong thư mục `app` có thể được tạo ra thông qua lệnh Artisan. Để có thể xem các lệnh đó, bạn có chạy lệnh `php artisan list make` trên terminal của bạn.

<a name="the-broadcasting-directory"></a>
#### Thư mục `Broadcasting`

Thư mục `Broadcasting` chứa tất cả các class broadcast channel cho ứng dụng của bạn. Các class này được tạo ra bằng lệnh `make:channel`. Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn khi bạn tạo channel đầu tiên. Để tìm hiểu thêm về các channel, hãy xem tài liệu về [event broadcasting](/docs/{{version}}/broadcasting).

<a name="the-console-directory"></a>
#### Thư mục Console

Thư mục `Console` sẽ chứa tất cả các câu lệnh Artisan của bạn. Các câu lệnh mới có thể được tạo ra bằng câu lệnh `make:command`. Thư mục đó cũng chứa những phần kernel của console, nó là nơi dành cho việc đăng ký các câu lệnh Artisan của bạn và các [scheduled tasks](/docs/{{version}}/scheduling) mà bạn đã cài đặt.

<a name="the-events-directory"></a>
#### Thư mục Events

Mặc định, thư mục này sẽ không tồn tại, nhưng nó sẽ được tạo khi bạn chạy lệnh Artisan `event:generate` và `make:event`. Thư mục `Events` sẽ chứa các [event classes](/docs/{{version}}/events). Events có thể được dùng thông báo cho các phần khác trong application của bạn rằng một hành động nào đó đã xảy ra, vì vậy nó rất linh hoạt và tách biệt.

<a name="the-exceptions-directory"></a>
#### Thư mục Exceptions

Thư mục `Exceptions` chứa những file xử lý exception cho application và nó cũng là một nơi tốt để lưu bất kỳ exception nào được tạo ra từ application của bạn. Nếu như bạn muốn tuỳ biến cách mà các exception được log hoặc được render, thì bạn nên sửa class `Handler` trong thư mục này.

<a name="the-http-directory"></a>
#### Thư mục Http

Thư mục `Http` chứa các file controllers, middleware và form request. Hầu như tất cả các logic dùng để xử lý các request tới application của bạn đều được lưu tại đây trong thư mục này.

<a name="the-jobs-directory"></a>
#### Thư  mục Jobs

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:job`. Thư mục `Jobs` sẽ chứa các [queueable jobs](/docs/{{version}}/queues) cho application của bạn. Jobs có thể được đưa vào hàng đợi hoặc chạy đồng bộ cùng với request hiện tại. Jobs mà được chạy đồng bộ cùng với request hiện tại đôi khi cũng có thể được gọi là "commands" vì chúng được triển khai theo [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
#### Thư mục Listeners

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `event:generate` hoặc `make:listener`. Thư mục `Listeners` sẽ chứa các class xử lý [events](/docs/{{version}}/events) cho bạn. Event listeners nhận vào một event instance và thực hiện logic phản hồi tướng ứng với sự kiện đã được kích hoạt. Ví dụ, một event `UserRegistered` có thể được xủ lý bởi một listener `SendWelcomeEmail`.

<a name="the-mail-directory"></a>
#### Thư mục Mail

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:mail`. Thư mục `Mail` chứa tất cả các [class cho việc tạo email](/docs/{{version}}/mail) được gửi đi bởi application của bạn. Đối tượng Mail cho phép bạn gói gọn tất cả logic của việc xây dựng email vào trong một class đơn giản, có thể được gửi bằng phương thức `Mail::send`.

<a name="the-models-directory"></a>
#### Thư mục Models

Thư mục `Models` sẽ chứa tất cả [các class model Eloquent](/docs/{{version}}/eloquent) của bạn. Eloquent ORM đi kèm với Laravel đã cung cấp một implement ActiveRecord đơn giản, đẹp mắt để làm việc với cơ sở dữ liệu của bạn. Mỗi bảng cơ sở dữ liệu có một "Model" tương ứng để sử dụng để tương tác với bảng đó. Model cho phép bạn truy vấn vào dữ liệu trong bảng của bạn, cũng như chèn thêm các bản ghi mới vào bảng.

<a name="the-notifications-directory"></a>
#### Thư mục Notifications

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:notification`. Thư mục `Notifications` chứa tất cả các [thông báo](/docs/{{version}}/notifications) "transactional" mà được gửi bởi application của bạn, như là một thông báo về một sự kiện xảy ra trong application của bạn. Tính năng thông báo của Laravel sẽ gửi thông báo đi qua nhiều driver khác nhau như email, Slack, SMS hoặc được lưu trữ trong cơ sở dữ liệu.

<a name="the-policies-directory"></a>
#### Thư mục Policies

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:policy`. Thư mục `Policies` chứa các [class về quyền](/docs/{{version}}/authorization) trong application của bạn. Policies sẽ dùng để xác định xem một user có thể thực hiện một hành động nhất định đối với một resource hay không. Để có thêm thông tin, hãy đọc thêm ở [authorization documentation](/docs/{{version}}/authorization).

<a name="the-providers-directory"></a>
#### Thư mục Providers

Thư mục `Providers` chứa tất cả các [service providers](/docs/{{version}}/providers) cho application của bạn. Service providers sẽ khởi động application của bạn bằng binding services trong service container và đăng ký các event hoặc thực hiện bất kỳ các task nào khác để chuẩn bị application xử lý những request đến.

Trong một application mới, thư mục này sẽ chứa sẵn một số provider. Bạn có thể tự do thêm các provider mà bạn muốn vào thư mục này khi cần thiết.

<a name="the-rules-directory"></a>
#### Thư mục Rules

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:rule`. Thư mục `Rules` chứa các tuỳ biến của các đối tượng validation rule. Rules được sử dụng để đóng gói các logic kiểm tra phức tạp trong một đối tượng đơn giản. Để có thêm thông tin, hãy đọc thêm ở [validation documentation](/docs/{{version}}/validation).
