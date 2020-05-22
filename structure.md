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
    - [Thư mục `Console`](#the-console-directory)
    - [Thư mục `Events`](#the-events-directory)
    - [Thư mục `Exceptions`](#the-exceptions-directory)
    - [Thư mục `Http`](#the-http-directory)
    - [Thư mục `Jobs`](#the-jobs-directory)
    - [Thư mục `Listeners`](#the-listeners-directory)
    - [Thư mục `Mail`](#the-mail-directory)
    - [Thư mục `Notifications`](#the-notifications-directory)
    - [Thư mục `Policies`](#the-policies-directory)
    - [Thư mục `Providers`](#the-providers-directory)
    - [Thư mục `Rules`](#the-rules-directory)

<a name="introduction"></a>
## Giới thiệu

Cấu trúc thư mục mặc định của Laravel nhằm cung cấp một khởi đầu tốt cho cả application lớn và nhỏ. Dĩ nhiên là bạn có thể tự tổ chức theo cách bạn muốn. Laravel sẽ gần như không áp đặt một hạn chế nào về vị trí của bất kỳ class nào, miễn là Composer có thể load class của bạn.

#### Thư mục Model sẽ ở đâu?

Khi mà mới bắt đầu với Laravel, nhiều người phát triển cảm thấy bị thiếu thư mục `models`. Tuy nhiên, việc thiếu đó là chủ đích của chugns tôi. Chúng tôi thấy từ "models" rất là mơ hồ, vì nó có nhiều ý nghĩa khác nhau tuỳ theo nhà phát triển.
Một số nhà phát triển nghĩ rằng "model" sẽ là tổng hợp tất cả business logic, nhưng một số khác lại rằng "model"là một lớp tương tác với database.

Vì lý do đó, chúng tôi đã chọn nơi chứa mặc định của Eloquent models là ở trong thư mục `app`, và cho phép nhà phát triển đặt lại nơi chứa nếu họ muốn.

<a name="the-root-directory"></a>
## Thư mục gốc

<a name="the-root-app-directory"></a>
#### Thư mục App

Thư mục `app` là nơi chứa phần core code của application. Chúng tôi sẽ sớm chia thư mục này ra cho rõ ràng hơn, nhưng hầu như tất cả các class sẽ nằm trong thư mục này.

<a name="the-bootstrap-directory"></a>
#### Thư mục Bootstrap

Thư mục `bootstrap` sẽ chứa file `app.php` cái dành cho bootstraps framework. Thư mục này cũng chứa thư mục `cache` cái mà framework đã tạo ra các file để tối ưu hoá tốc độ cho route và service.

<a name="the-config-directory"></a>
#### Thư mục Config

Thư mục `config` mang ý nghĩa rất dễ hiểu, nó dùng để chứa tất cả các file config cho application của bạn. Nó là một ý tưởng tốt để đọc tất cả các file và làm quen các biến đã có sẵn trong đó.

<a name="the-database-directory"></a>
#### Thư mục Database

Thư mục `database` chứa các file migration cho database, các file factories để tạo data fake cho model, and các file seed. Nếu bạn muốn, bạn cũng có thể dùng thư mục này để chứa file SQLite database.

<a name="the-public-directory"></a>
#### Thư mục Public

Thư mục `public` chứa file `index.php`, cái file là điểm đầu vào cho mọi request gửi tới application của bạn và các file config autoloading. Thư mục này cũng chứa các file images, JavaScript, and CSS.

<a name="the-resources-directory"></a>
#### Thư mục Resources

Thư mục `resources` chứa file view của bạn cũng như file raw, và các file chưa được compiled như là LESS, SASS, or JavaScript. Thư mục này cũng chứa những file language.

<a name="the-routes-directory"></a>
#### Thư mục Routes

Thư mục `routes` chứa tất cả những file định nghĩa route cho application của bạn.
Mặc đinh, sẽ bao gồm những file sau đây: `web.php`, `api.php`, `console.php` and `channels.php`.

File `web.php` sẽ chứa những routes mà được load bởi file `RouteServiceProvider` và đặt những routes đó vào trong một group middleware có tên là `web`, middleware này cung cấp session, bảo vệ routes trước tấn CSRF và mã hoá cookie. Nếu application của bạn chỉ dùng session hoặc không dùng RESTful API, thì tất cả routes của bạn có thế được xác định trong file `web.php`.

File `api.php` chứa những routes mà được load bởi file `RouteServiceProvider` và đặt những routes đó vào trong một group middleware có tên là `api`, middleware này cung cấp tỷ lệ giới hạn. Những routes này sẽ được chủ định là không dùng session, vì vậy request đến application của bạn thông qua những routes này sẽ được authenticated thông qua tokens và không có quyền truy cập vào session.

File `console.php` là nơi bạn có thể định nghĩa tất cả các [Closure](https://www.php.net/manual/en/functions.anonymous.php) dựa trên các lệnh chạy ở console. Mỗi Closure sẽ tương ứng với một câu lệnh nên bạn sẽ dễ dàng tiếp cận các phương thức input và output của mỗi lệnh. File này sẽ không định nghĩa các Http route của application, mà nó định nghĩa các console route tới application của bạn.

File `channels.php` sẽ nơi mà bạn có thể đăng ký các tất cả các event broadcasting channels mà application bạn hỗ trợ.

<a name="the-storage-directory"></a>
#### Thư mục Storage

Thư mục `storage` sẽ chứa những file Blade đã được compile, file session, file cache, và các file khác được tạo ra bởi framework. Thư mục này sẽ chưa ra các thư mục con: `app`, `framework`, và `logs`. Thư mục `app` có thể dùng để lưu tất cả các file được tạo ra bởi application của bạn. Thư mục `framework` sẽ dùng để lưu các file được tạo ra bởi framework và cache, Cuối cùng là thư mục `logs, nó dùng để chưa các file log của application.

Thư mục `storage/app/public` có thể được dùng để lưu trữ các file mà user tạo như là: avatars, những loại mà được phép public. Bạn cũng nên tạo một link ảo là `public/storage` để trỏ vào thư mục này. Bạn có thể tạo link ảo đó bằng câu lệnh sau: `php artisan storage:link`.

<a name="the-tests-directory"></a>
#### Thư mục Tests

Thư mục `tests` sẽ chứa các file tự động test. Ví dụ [PHPUnit](https://phpunit.de/) sẽ được cung cấp out of the box. Mỗi class test nên đặt đặt với hậu tố là từ `Test`. Bạn có thể chạy test của bạn bằng câu lệnh `phpunit` hoặc `php vendor/bin/phpunit`.

<a name="the-vendor-directory"></a>
#### Thư mục Vendor

Thư mục `vendor` chứa những library mà được quản lý bằng [Composer](https://getcomposer.org).

<a name="the-app-directory"></a>
## Thư mục App

Phần lớn application của bạn sẽ được đặt trong thư mục `app`. Mặc định, thư mục này sẽ được đặt dưới tên là `App` và được autoloaded bởi Composer dùng chuẩn [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/).

Thư mục `app` sẽ chứa thêm một số thư mục bổ sung như `Console`, `Http`, và `Providers`. Hãy nghĩ các thư mục `Console` và `Http` như là các thư mục cung cấp API cho phần core của application của bạn.
Giao thức HTTP và CLI đều là các cơ chế để bên ngoài tương tác với ứng dụng của bạn, nhưng thực tế  chúng lại không hay chứa logic của ứng dụng. Nói cách khác, chúng là hai cách gọi đến ứng dụng của bạn. Thư mục `Console` chứa tất cả các lệnh Artisan của bạn, và thư mục `Http` chứa lớp controllers, middleware, và requests cảu bạn.

Các thư mục khác sẽ được tạo trong thư mục `app` khi bạn dùng lệnh Artisan `make` để tạo các class. Ví dụ, bình thường, thư mục `app/Jobs` sẽ không tồn tại cho đến khi bạn chạy lệnh Artisan `make:job` to tạo class job.

> {tip} Nhiều class trong thư mục `app` có thể được tạo thông qua lệnh Artisan. Để có thể xem các lệnh đó, bạn có chạy lệnh `php artisan list make` trên terminal của bạn.

<a name="the-console-directory"></a>
#### Thư mục Console

Thư mục `Console` sẽ chứa tất cả các câu lệnh custom Artisan của bạn. Các lệnh mới có thể được tạo bằng câu lệnh `make:command`. Thư mục đó cũng chứa những phần kernel của console, nó là nơi dành cho việc đăng ký các custom Artisan của bạn và các [scheduled tasks](/docs/{{version}}/scheduling) mà bạn đã cài đặt.

<a name="the-events-directory"></a>
#### Thư mục Events

Mặc định, thư mục này sẽ không tồn tại, nhưng nó sẽ được tạo khi bạn chạy lệnh Artisan `event:generate` và `make:event`. Thư mục `Events`, đúng như ý nghĩa của nó, nó chứa các [event classes](/docs/{{version}}/events). Events có thể được dùng thông báo cho các phần khác trong application của bạn rằng một hành động nào đó đã xảy ra, nên nó rất linh hoạt và tách biệt.

<a name="the-exceptions-directory"></a>
#### Thư mục Exceptions

Thư mục `Exceptions` chứa những file xử lý exception của application và nó cũng là một nơi tốt để đặt bất kỳ exception nào được tạo ra từ application của bạn. Nếu như bạn muốn tuỳ chỉnh cái cách mà exception được log hoặc được render, thì bạn nên sửa class `Handler` trong thư mục này.

<a name="the-http-directory"></a>
#### Thư mục Http

Thư mục `Http` chứa các file controllers, middleware và form requests. Hầu như tất cả các logic dùng để xử lý các request tới application của bạn đều được tại đây trong thư mục này.

<a name="the-jobs-directory"></a>
#### Thư  mục Jobs

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:job`. Thư mục `Jobs` sẽ chứa các [queueable jobs](/docs/{{version}}/queues) cho ứng dụng của bạn. Jobs có thể được đưa vào hàng đợi hoặc chạy đồng bộ cùng request hiện tại. Jobs cái mà được chạy đồng bộ với request hiện tại đôi khi cũng được gọi là "commands" vì chúng được triển khai theo [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
#### Thư mục Listeners

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `event:generate` hoặc `make:listener`. Thư mục `Listeners` chứa các class xử lý [events](/docs/{{version}}/events) của bạn. Sự kiện listeners nhận vào một event instance và thực hiện logic trong response tướng ứng với sự kiện được kích hoạt. Ví dụ, một event `UserRegistered` có thể được xủ lý bởi một listener `SendWelcomeEmail`.

<a name="the-mail-directory"></a>
#### Thư mục Mail

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:mail`. Thư mục `Mail` chứa tất cả các class của bạn cho việc tạo email được gửi bởi application của bạn. Đối tượng Mail cho phép bạn gói gọn tất cả logic của việc xây dựng email trong một lớp đơn giản, có thể được gửi bằng phương thức `Mail :: send`.

<a name="the-notifications-directory"></a>
#### Thư mục Notifications

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:notification`. Thư mục `Notifications` chứa tất cả các thông báo "transactional" mà được gửi bởi application của bạn, như là một thông báo đơn giản về một sự kiện xảy ra trong application của bạn. Tính năng thông báo của Laravel gửi thông báo qua nhiều driver khác nhau như email, Slack, SMS hoặc được lưu trữ trong cơ sở dữ liệu.

<a name="the-policies-directory"></a>
#### Thư mục Policies

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:policy`. Thư mục `Policies` chứa các class về quyền trong application của bạn. Policies sẽ dùng để xác định xem một user có thể thực hiện một hành động nhất định với một resource hay không. Để có thêm thông tin, hãy đọc thêm ở [authorization documentation](/docs/{{version}}/authorization).

<a name="the-providers-directory"></a>
#### Thư mục Providers

Thư mục `Providers` chứa tất cả các [service providers](/docs/{{version}}/providers) cho application của bạn.  Service providers được đăng ký trong application của bạn bởi binding services trong service container và đăng ký event hoặc thực hiện bất kỳ các task nào khác để chuẩn bị application của bạn cho những request đến.

Trong một ứng dụng mới, thì thư mục này sẽ chứa một số provider có sẵn. Bạn có thể tự do thêm các provider của riêng bạn vào thư mục này khi cần thiết.

<a name="the-rules-directory"></a>
#### Thư mục Rules

Mặc định, thư mục này không tồn tại, nhưng nó sẽ được tạo ra cho bạn, nếu bạn chạy lệnh Artisan `make:rule`. Thư mục `Rules` chứa các tuỳ chỉnh của các đối tượng validation rule cho application của bạn. Rules được sử dụng để dống gói các logic kiểm tra phức tạp trong một đối tượng đơn giản. Để có thêm thông tin, hãy đọc thêm ở [validation documentation](/docs/{{version}}/validation).
