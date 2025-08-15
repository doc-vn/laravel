# Cài đặt

- [Lời nói đầu](#meet-laravel)
    - [Tại sao lại là Laravel?](#why-laravel)
- [Creating a Laravel Project](#creating-a-laravel-project)
- [Cài đặt cấu hình](#initial-configuration)
    - [Cấu hình file môi trường](#environment-based-configuration)
    - [Databases và Migrations](#databases-and-migrations)
    - [Cấu hình thư mục](#directory-configuration)
- [Cài đặt Docker bằng Sail](#docker-installation-using-sail)
    - [Sail trên macOS](#sail-on-macos)
    - [Sail trên Windows](#sail-on-windows)
    - [Sail trên Linux](#sail-on-linux)
    - [Chọn service Sail bạn dùng](#choosing-your-sail-services)
- [IDE Support](#ide-support)
- [Bước tiếp theo](#next-steps)
    - [Laravel cho Full Stack](#laravel-the-fullstack-framework)
    - [Laravel cho backend api](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## Lời nói đầu

Laravel là một framework phát triển ứng dụng web với cú pháp tinh tế, hàm ý. Framework web cung cấp cấu trúc và điểm bắt đầu để tạo ứng dụng của bạn, cho phép bạn tập trung vào việc tạo ra thứ gì đó tuyệt vời trong khi chúng tôi sẽ bỏ công sức ra làm chi tiết.

Laravel cố gắng cung cấp trải nghiệm tuyệt vời nhất cho nhà phát triển đồng thời cung cấp các tính năng mạnh mẽ như tích hợp phụ thuộc, lớp abstraction hóa cơ sở dữ liệu, queue và scheduled job, unit và integration test, v.v.

Cho dù bạn là người mới làm quen với PHP web framework hay là người đã có nhiều năm kinh nghiệm, Laravel là một framework có thể phát triển cùng với bạn. Chúng tôi sẽ giúp bạn thực hiện những bước đầu tiên với tư cách là nhà phát triển web hoặc nâng cao kiến thức chuyên môn của bạn lên một tầm cao mới. Chúng tôi nóng lòng muốn xem những gì bạn xây dựng.

> [!NOTE]
> Bạn mới sử dụng Laravel? Hãy xem [Laravel Bootcamp](https://bootcamp.laravel.com) để có thể tham quan thực tế về framework và chúng tôi sẽ hướng dẫn bạn về cách xây dựng ứng dụng Laravel đầu tiên của bạn.

<a name="why-laravel"></a>
### Tại sao lại là Laravel?

Có rất nhiều công cụ và framework có sẵn cho bạn khi bạn xây dựng một ứng dụng web. Tuy nhiên, chúng tôi tin rằng Laravel là một lựa chọn tốt nhất để xây dựng các ứng dụng web full-stack hiện đại.

#### A Progressive Framework

Chúng tôi muốn gọi Laravel là một framework "tiến bộ". Bằng cách đó, chúng tôi muốn nói rằng Laravel phát triển cùng với bạn. Nếu bạn mới thực hiện những bước đầu tiên trong quá trình phát triển web, thư viện tài liệu, hướng dẫn và [video hướng dẫn](https://laracasts.com) khổng lồ của Laravel sẽ giúp bạn tìm hiểu các bước cơ bản mà không bị choáng ngợp.

Nếu bạn là nhà phát triển cấp cao, Laravel cũng cung cấp cho bạn các công cụ mạnh mẽ để [tích hợp phụ thuộc](/docs/{{version}}/container), [unit test](/docs/{{version}}/testing), [queue](/docs/{{version}}/queues), [real-time events](/docs/{{version}}/broadcasting), và nhiều hơn thế. Laravel được tinh chỉnh để xây dựng các ứng dụng web chuyên nghiệp và sẵn sàng xử lý khối lượng lớn công việc của doanh nghiệp.

#### A Scalable Framework

Laravel có khả năng mở rộng đáng kinh ngạc. Nhờ tính chất thân thiện của PHP và tính năng hỗ trợ sẵn có của Laravel dành cho các hệ thống bộ nhớ cache phân tán như Redis, việc mở rộng quy mô theo chiều ngang với Laravel thật dễ dàng. Trên thực tế, các ứng dụng Laravel đã dễ dàng mở rộng quy mô để xử lý hàng trăm triệu request mỗi tháng.

Cần mở rộng quy mô cực lớn? Các nền tảng như [Laravel Vapor](https://vapor.laravel.com) cho phép bạn chạy ứng dụng Laravel của bạn ở quy mô gần như vô hạn trên công nghệ serverless mới nhất của AWS.

#### A Community Framework

Laravel kết hợp các package tốt nhất trong hệ sinh thái PHP để cung cấp framework mạnh mẽ và thân thiện nhất với nhà phát triển. Ngoài ra, hàng nghìn nhà phát triển tài năng từ khắp nơi trên thế giới đã [đóng góp cho framework](https://github.com/laravel/framework). Ai biết được, thậm chí có thể bạn sẽ trở thành người đóng góp cho Laravel.

<a name="creating-a-laravel-project"></a>
## Creating a Laravel Project

Trước khi tạo project Laravel đầu tiên, bạn hãy chắc chắn là máy local của bạn đã cài đặt PHP và [Composer](https://getcomposer.org). Nếu bạn đang phát triển trên môi trường macOS, PHP và Composer, thì bạn có thể cài đặt được chúng trong vài phút thông qua [Laravel Herd](https://herd.laravel.com). Ngoài ra, chúng tôi cũng khuyên bạn nên [cài đặt Node và NPM](https://nodejs.org).

Sau khi bạn đã hoàn thành việc cài đặt PHP và Composer, bạn có thể tạo một project Laravel mới thông qua lệnh `create-project` của Composer:

```nothing
composer create-project "laravel/laravel:^10.0" example-app
```

Hoặc, bạn có thể tạo project Laravel mới bằng cách dùng qua [Laravel installer](https://github.com/laravel/installer) đang được cài đặt global thông qua Composer:

```nothing
composer global require laravel/installer

laravel new example-app
```

Khi project đã được tạo, hãy khởi động máy chủ local của Laravel bằng lệnh `serve` của Laravel Artisan:

```nothing
cd example-app

php artisan serve
```

Sau khi bạn đã khởi động máy chủ bằng Artisan, ứng dụng của bạn sẽ có thể truy cập được trong trình duyệt web của bạn bằng địa chỉ [http://localhost:8000](http://localhost:8000). Tiếp theo, bạn đã sẵn sàng [bắt đầu thực hiện các bước khác trong hệ sinh thái Laravel](#next-steps). Tất nhiên, bạn cũng có thể muốn [cấu hình cơ sở dữ liệu](#databases-and-migrations).

> [!NOTE]
> Nếu bạn muốn có một sự khởi đầu thuận tiện khi phát triển ứng dụng Laravel, thì hãy cân nhắc sử dụng một trong những [bộ khởi tạo](/docs/{{version}}/starter-kits) của chúng tôi. Bộ khởi tạo này cung cấp một nền tảng xác thực có sẵn cả backend và frontend cho ứng dụng Laravel mới của bạn.

<a name="initial-configuration"></a>
## Initial Configuration

Tất cả các file cấu hình cho Laravel framework đều được lưu trong thư mục `config`. Mỗi tùy chọn đều đã được giải thích, vì vậy bạn hãy thoải mái xem qua các file và làm quen với các tùy chọn có sẵn cho bạn.

Laravel hầu như không cần bạn cấu hình thêm bất cứ cấu hình nào khi cài đặt. Bạn có thể thoải mái bắt đầu phát triển! Tuy nhiên, bạn có thể muốn xem qua file `config/app.php` và tài liệu hướng dẫn của nó. Nó chứa một số tùy chọn như `timezone` và `locale` mà bạn có thể muốn thay đổi theo trạng thái ứng dụng của bạn.

<a name="environment-based-configuration"></a>
### Environment Based Configuration

Vì nhiều giá trị tùy chọn cấu hình của Laravel có thể khác nhau tùy thuộc vào việc ứng dụng của bạn đang chạy trên môi trường local hay môi trường là production, nên nhiều giá trị cấu hình quan trọng được định nghĩa trong file `.env` có ở trong thư mục root của ứng dụng.

File `.env` của bạn không nên được commit vào source control của ứng dụng, vì mỗi nhà phát triển và server của họ sẽ sử dụng ứng dụng của bạn theo nhiều yêu cầu cấu hình khác nhau. Hơn nữa, đây sẽ là rủi ro bảo mật trong trường hợp kẻ xâm nhập có quyền truy cập vào source control của bạn, vì bất kỳ thông tin xác thực nhạy cảm nào cũng sẽ bị lộ.

> [!NOTE]
> Để biết thêm thông tin về file `.env` và cấu hình theo môi trường, hãy xem [tài liệu cấu hình](/docs/{{version}}/configuration#environment-configuration).

<a name="databases-and-migrations"></a>
### Databases and Migrations

Bây giờ bạn đã tạo ứng dụng Laravel của bạn, có lẽ bạn muốn lưu một số dữ liệu vào trong cơ sở dữ liệu. Mặc định, file cấu hình `.env` của ứng dụng của bạn sẽ tương tác với cơ sở dữ liệu MySQL và sẽ truy cập cơ sở dữ liệu ở địa chỉ `127.0.0.1`.

> [!NOTE]
> Nếu bạn đang phát triển trên macOS và cần cài đặt MySQL, Postgres hoặc Redis ở local, hãy cân nhắc sử dụng [DBngin](https://dbngin.com/).

Nếu bạn không muốn cài đặt MySQL hoặc Postgres trên máy local của bạn, bạn có thể sử dụng cơ sở dữ liệu [SQLite](https://www.sqlite.org/index.html). SQLite là một loại cơ sở dữ liệu nhỏ, nhanh và độc lập. Để bắt đầu, hãy cập nhật file cấu hình `.env` của bạn để sử dụng driver cơ sở dữ liệu `sqlite` của Laravel. Bạn có thể xóa các tùy chọn cấu hình cơ sở dữ liệu khác:

```ini
DB_CONNECTION=sqlite # [tl! add]
DB_CONNECTION=mysql # [tl! remove]
DB_HOST=127.0.0.1 # [tl! remove]
DB_PORT=3306 # [tl! remove]
DB_DATABASE=laravel # [tl! remove]
DB_USERNAME=root # [tl! remove]
DB_PASSWORD= # [tl! remove]
```

Sau khi cấu hình cơ sở dữ liệu SQLite, bạn có thể chạy [database migrations](/docs/{{version}}/migrations) của ứng dụng, lệnh này sẽ tạo ra các bảng cơ sở dữ liệu cho ứng dụng:

```shell
php artisan migrate
```

Nếu cơ sở dữ liệu SQLite chưa được tạo cho ứng dụng của bạn, Laravel sẽ hỏi bạn có muốn tạo cơ sở dữ liệu không. Thông thường, file cơ sở dữ liệu SQLite sẽ được tạo tại `database/database.sqlite`.

<a name="directory-configuration"></a>
### Cấu hình thư mục

Laravel nên được chạy từ thư mục root của "web directory" đã được cấu hình trong server web của bạn. Bạn không nên cố gắng chạy ứng dụng Laravel từ thư mục con của "web directory". Cố gắng làm như vậy có thể làm lộ các file nhạy cảm có trong ứng dụng của bạn.

<a name="docker-installation-using-sail"></a>
## Cài đặt Docker bằng Sail

Chúng tôi muốn việc bắt đầu với Laravel trở nên dễ dàng nhất có thể với bất kỳ hệ điều hành nào mà bạn thích. Vì vậy, có nhiều tùy chọn để phát triển và chạy dự án Laravel trên máy local của bạn. Mặc dù bạn có thể muốn khám phá các tùy chọn này sau, nhưng Laravel cung cấp [Sail](/docs/{{version}}/sail), một giải pháp sẵn có để chạy các project Laravel của bạn bằng [Docker](https://www.docker.com).

Docker là một công cụ để chạy các ứng dụng và service trong các "containers" nhỏ, nhẹ, không can thiệp vào cấu hình hoặc phần mềm được cài đặt trên máy local của bạn. Điều này có nghĩa là bạn không phải lo lắng về việc cấu hình hoặc thiết lập các công cụ phát triển phức tạp như web server và cơ sở dữ liệu trên máy local của bạn. Để bắt đầu, bạn chỉ cần cài đặt [Docker Desktop](https://www.docker.com/products/docker-desktop).

Laravel Sail là giao diện command-line nhẹ để tương tác với cấu hình Docker mặc định của Laravel. Sail cung cấp điểm khởi đầu tuyệt vời để xây dựng ứng dụng Laravel bằng PHP, MySQL và Redis mà không cần yêu cầu kinh nghiệm về Docker trước đó.

> [!NOTE]
> Bạn đã là chuyên gia về Docker? Đừng lo lắng! Mọi thứ về Sail có thể được tùy chỉnh bằng cách sử dụng file `docker-compose.yml` có trong Laravel.

<a name="sail-on-macos"></a>
### Sail trên macOS

Nếu bạn đang phát triển trên máy Mac và [Docker Desktop](https://www.docker.com/products/docker-desktop) đã được cài đặt, bạn có thể sử dụng một lệnh terminal đơn giản để tạo project Laravel mới. Ví dụ: để tạo một ứng dụng Laravel mới trong thư mục có tên "example-app", bạn có thể chạy lệnh sau trong terminal của bạn:

```shell
curl -s "https://laravel.build/example-app" | bash
```

Tất nhiên, bạn có thể thay đổi "example-app" trong URL này thành bất kỳ thứ gì bạn thích - chỉ cần đảm bảo tên ứng dụng của bạn chỉ chứa các ký tự chữ, số, dấu gạch ngang và dấu gạch dưới. Thư mục của ứng dụng Laravel sẽ được tạo trong thư mục mà bạn đang chạy lệnh.

Việc cài đặt Sail có thể mất vài phút khi các container ứng dụng của Sail được cấu trúc vào máy local của bạn.

Sau khi project được tạo, bạn có thể di chuyển đến thư mục ứng dụng và khởi động Laravel Sail. Laravel Sail cung cấp một giao diện command-line đơn giản để tương tác với cấu hình Docker mặc định của Laravel:

```shell
cd example-app

./vendor/bin/sail up
```

Khi container Docker của ứng dụng đã được khởi động xong, bạn có thể truy cập vào ứng dụng trong trình duyệt web của bạn tại: http://localhost.

> [!NOTE]
> Để tiếp tục tìm hiểu thêm về Laravel Sail, hãy xem lại [tài liệu đầy đủ](/docs/{{version}}/sail).

<a name="sail-on-windows"></a>
### Sail trên Windows

Trước khi chúng ta tạo một ứng dụng Laravel mới trên máy Windows của bạn, hãy đảm bảo bạn đã cài đặt [Docker Desktop](https://www.docker.com/products/docker-desktop). Tiếp theo, bạn nên đảm bảo là Windows Subsystem cho Linux 2 (WSL2) đã được cài đặt và được kích hoạt trên máy của bạn. WSL cho phép bạn chạy các tệp lệnh nhị phân Linux nguyên bản trên Windows 10. Bạn có thể tìm thấy thông tin về cách cài đặt và kích hoạt WSL2 trong [tài liệu về môi trường dành cho nhà phát triển](https://docs.microsoft.com/en-us/windows/wsl/install-win10) của Microsoft.

> [!NOTE]
> Sau khi cài đặt và kích hoạt WSL2 xong, bạn nên đảm bảo rằng Docker Desktop đã được [cấu hình để sử dụng WSL2](https://docs.docker.com/docker-for-windows/wsl/).

Tiếp theo, bạn đã sẵn sàng tạo project Laravel đầu tiên của bạn. Chạy [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab) và bắt đầu phiên terminal mới cho hệ điều hành WSL2 Linux của bạn. Tiếp theo, bạn có thể sử dụng lệnh terminal đơn giản để tạo project Laravel mới. Ví dụ: để tạo một ứng dụng Laravel mới trong thư mục có tên là "example-app", bạn có thể chạy lệnh sau trong terminal của bạn:

```shell
curl -s https://laravel.build/example-app | bash
```

Tất nhiên, bạn có thể thay đổi "example-app" trong URL này thành bất kỳ thứ gì bạn thích - chỉ cần đảm bảo tên ứng dụng của bạn chỉ chứa các ký tự chữ, số, dấu gạch ngang và dấu gạch dưới. Thư mục của ứng dụng Laravel sẽ được tạo trong thư mục mà bạn đang chạy lệnh.

Việc cài đặt Sail có thể mất vài phút khi các container ứng dụng của Sail được cấu trúc vào máy local của bạn.

Sau khi project được tạo, bạn có thể di chuyển đến thư mục ứng dụng và khởi động Laravel Sail. Laravel Sail cung cấp một giao diện command-line đơn giản để tương tác với cấu hình Docker mặc định của Laravel:

```shell
cd example-app

./vendor/bin/sail up
```

Khi container Docker của ứng dụng đã được khởi động xong, bạn có thể truy cập vào ứng dụng trong trình duyệt web của bạn tại: http://localhost.

> [!NOTE]
> Để tiếp tục tìm hiểu thêm về Laravel Sail, hãy xem lại [tài liệu đầy đủ](/docs/{{version}}/sail).

#### Developing Within WSL2

Tất nhiên, bạn cũng sẽ cần có khả năng thay đổi các file ứng dụng Laravel đã được tạo trong quá trình cài đặt WSL2. Để thực hiện điều này, chúng tôi khuyên bạn nên sử dụng IDE [Visual Studio Code](https://code.visualstudio.com) của Microsoft và extension của nhà phát triển cho [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack).

Sau khi cài đặt các công cụ này, bạn có thể mở bất kỳ dự án Laravel nào bằng cách chạy lệnh `code .` từ thư mục gốc của ứng dụng bằng Windows Terminal.

<a name="sail-on-linux"></a>
### Sail trên Linux

Nếu bạn đang phát triển trên Linux và [Docker Compose](https://docs.docker.com/compose/install/) đã được cài đặt, bạn có thể sử dụng lệnh terminal đơn giản để tạo dự án Laravel mới.

Đầu tiên, nếu bạn đang sử dụng Docker Desktop cho Linux, thì bạn nên thực hiện lệnh sau. Nếu bạn không sử dụng Docker Desktop cho Linux, bạn có thể bỏ qua bước này:

```shell
docker context use default
```

Sau đó, để tạo một ứng dụng Laravel mới trong thư mục có tên là "example-app", bạn có thể chạy lệnh sau trong terminal của bạn:

```shell
curl -s https://laravel.build/example-app | bash
```

Tất nhiên, bạn có thể thay đổi "example-app" trong URL này thành bất kỳ thứ gì bạn thích - chỉ cần đảm bảo tên ứng dụng của bạn chỉ chứa các ký tự chữ, số, dấu gạch ngang và dấu gạch dưới. Thư mục của ứng dụng Laravel sẽ được tạo trong thư mục mà bạn đang chạy lệnh.

Việc cài đặt Sail có thể mất vài phút khi các container ứng dụng của Sail được cấu trúc vào máy local của bạn.

Sau khi project được tạo, bạn có thể di chuyển đến thư mục ứng dụng và khởi động Laravel Sail. Laravel Sail cung cấp một giao diện command-line đơn giản để tương tác với cấu hình Docker mặc định của Laravel:

```shell
cd example-app

./vendor/bin/sail up
```

Khi container Docker của ứng dụng đã được khởi động xong, bạn có thể truy cập vào ứng dụng trong trình duyệt web của bạn tại: http://localhost.

> [!NOTE]
> Để tiếp tục tìm hiểu thêm về Laravel Sail, hãy xem lại [tài liệu đầy đủ](/docs/{{version}}/sail).

<a name="choosing-your-sail-services"></a>
### Chọn service Sail bạn dùng

Khi tạo một ứng dụng Laravel mới thông qua Sail, bạn có thể sử dụng biến `with` để chọn service nào sẽ được cấu hình trong file `docker-compose.yml` của ứng dụng mới của bạn. Các service có sẵn là `mysql`, `pgsql`, `mariadb`, `redis`, `memcached`, `meilisearch`, `typesense`, `minio`, `selenium` và `mailpit`:

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis" | bash
```

Nếu bạn không chỉ định service nào mà bạn muốn cấu hình, một stackp mặc định gồm có `mysql`, `redis`, `meilisearch`, `mailpit` và `selenium` sẽ được cấu hình mặc định.

Bạn cũng có thể chỉ định Sail cài đặt một [Devcontainer](/docs/{{version}}/sail#using-devcontainers) mặc định vào bằng cách thêm tham số `devcontainer` vào URL:

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis&devcontainer" | bash
```

<a name="ide-support"></a>
## IDE Support

Bạn có thể thoải mái sử dụng bất kỳ trình code editor nào mà bạn muốn khi phát triển các ứng dụng Laravel; tuy nhiên, [PhpStorm](https://www.jetbrains.com/phpstorm/laravel/) cung cấp hỗ trợ toàn diện cho Laravel và hệ sinh thái của nó, bao gồm cả [Laravel Pint](https://www.jetbrains.com/help/phpstorm/using-laravel-pint.html).

Ngoài ra, cộng đồng cũng duy trì [Laravel Idea](https://laravel-idea.com/), một Plugin PhpStorm cung cấp nhiều tiện ích bổ sung cho IDE, bao gồm việc tạo code, gợi ý cú pháp Eloquent, gợi ý rule validation...

<a name="next-steps"></a>
## Bước tiếp theo

Bây giờ bạn đã tạo xong project Laravel của bạn, có thể bạn đang tự hỏi nên học gì tiếp theo. Trước tiên, chúng tôi thực sự khuyên bạn nên làm quen với cách Laravel hoạt động bằng cách đọc các tài liệu sau:

<div class="content-list" markdown="1">

- [Request Lifecycle](/docs/{{version}}/lifecycle)
- [Configuration](/docs/{{version}}/configuration)
- [Directory Structure](/docs/{{version}}/structure)
- [Frontend](/docs/{{version}}/frontend)
- [Service Container](/docs/{{version}}/container)
- [Facades](/docs/{{version}}/facades)

</div>

Cách bạn muốn sử dụng Laravel như thế nào cũng sẽ quyết định các bước tiếp theo trên hành trình của bạn. Có nhiều cách khác nhau để sử dụng Laravel và chúng ta sẽ khám phá hai trường hợp sử dụng chính của framework ở bên dưới.

> [!NOTE]
> Bạn mới sử dụng Laravel? Hãy xem [Laravel Bootcamp](https://bootcamp.laravel.com) để có thể tham quan thực tế về framework và chúng tôi sẽ hướng dẫn bạn về cách xây dựng ứng dụng Laravel đầu tiên của bạn.

<a name="laravel-the-fullstack-framework"></a>
### Laravel cho Full Stack

Laravel có thể phục vụ như một full stack framework. "Full stack" framework, ý chúng tôi muốn nói là bạn sẽ sử dụng Laravel để route các request đến ứng dụng của bạn và hiển thị giao diện người dùng của bạn thông qua [Blade templates](/docs/{{version}}/blade) hoặc kết hợp với một single-page application như [Inertia](https://inertiajs.com). Đây là cách phổ biến nhất để sử dụng framework Laravel và theo chúng tôi, đây là cách sử dụng Laravel hiệu quả nhất.

Nếu đây là cách mà bạn định sử dụng Laravel, bạn có thể muốn xem tài liệu của chúng tôi về [frontend development](/docs/{{version}}/frontend), [routing](/docs/{{version}}/routing), [views](/docs/{{version}}/views) hoặc [Eloquent ORM](/docs/{{version}}/eloquent). Ngoài ra, bạn có thể muốn tìm hiểu về các package cộng đồng như [Livewire](https://livewire.laravel.com) và [Inertia](https://inertiajs.com). Các package này cho phép bạn vẫn sử dụng Laravel làm full-stack framework trong khi vẫn tận hưởng nhiều lợi ích về giao diện người dùng được cung cấp bởi các ứng dụng JavaScript single-page.

Nếu bạn đang sử dụng Laravel làm full stack framework, chúng tôi cũng đặc biệt khuyến khích bạn tìm hiểu cách biên dịch CSS và JavaScript cho ứng dụng của bạn bằng cách sử dụng [Vite](/docs/{{version}}/vite).

> [!NOTE]
> Nếu bạn muốn bắt đầu xây dựng ứng dụng của bạn một cách thuận lợi, hãy xem một trong các [bộ công cụ tạo nhanh ứng dụng](/docs/{{version}}/starter-kits) chính thức của chúng tôi.

<a name="laravel-the-api-backend"></a>
### Laravel cho backend api

Laravel cũng có thể đóng vai trò là backend API cho mộpt ứng dụng single-page JavaScript application hoặc ứng dụng di động. Ví dụ: bạn có thể sử dụng Laravel làm backend API cho ứng dụng [Next.js](https://nextjs.org) của bạn. Trong trường hợp này, bạn có thể sử dụng Laravel để cung cấp [xác thực](/docs/{{version}}/sanctum) và lưu trữ, truy xuất dữ liệu cho ứng dụng của bạn, đồng thời tận dụng các service mạnh mẽ của Laravel như queue, email, thông báo, và nhiều hơn thế nữa.

Nếu đây là cách bạn dự định sử dụng Laravel, bạn có thể muốn xem tài liệu của chúng tôi về [routing](/docs/{{version}}/routing), [Laravel Sanctum](/docs/{{version}}/sanctum) và [Eloquent ORM](/docs/{{version}}/eloquent).

> [!NOTE]
> Bạn cần bắt đầu xây dựng backend là Laravel và frontend là Next.js? Laravel Breeze sẽ cung cấp [API stack](/docs/{{version}}/starter-kits#breeze-and-next) cũng như [triển khai frontend Next.js](https://github.com/laravel/breeze-next) để bạn có thể bắt đầu sau vài phút.
