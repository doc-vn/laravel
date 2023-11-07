# Cài đặt

- [Lời nói đầu](#meet-laravel)
    - [Tại sao lại là Laravel?](#why-laravel)
- [Project Laravel đầu tiên](#your-first-laravel-project)
    - [Bắt đầu với macOS](#getting-started-on-macos)
    - [Bắt đầu với Windows](#getting-started-on-windows)
    - [Bắt đầu với Linux](#getting-started-on-linux)
    - [Chọn service Sail bạn dùng](#choosing-your-sail-services)
    - [Cài đặt qua Composer](#installation-via-composer)
- [Cài đặt cấu hình](#initial-configuration)
    - [Cấu hình file môi trường](#environment-based-configuration)
    - [Cấu hình thư mục](#directory-configuration)
- [Bước tiếp theo](#next-steps)
    - [Laravel cho Full Stack](#laravel-the-fullstack-framework)
    - [Laravel cho backend api](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## Lời nói đầu

Laravel là một framework phát triển ứng dụng web với cú pháp tinh tế, hàm ý. Framework web cung cấp cấu trúc và điểm bắt đầu để tạo ứng dụng của bạn, cho phép bạn tập trung vào việc tạo ra thứ gì đó tuyệt vời trong khi chúng tôi sẽ bỏ công sức ra làm chi tiết.

Laravel cố gắng cung cấp trải nghiệm tuyệt vời nhất cho nhà phát triển đồng thời cung cấp các tính năng mạnh mẽ như tích hợp phụ thuộc, lớp abstraction hóa cơ sở dữ liệu, queue và scheduled job, unit và integration test, v.v.

Cho dù bạn là người mới làm quen với PHP hay các framework web hay đã có nhiều năm kinh nghiệm, Laravel là một framework có thể phát triển cùng với bạn. Chúng tôi sẽ giúp bạn thực hiện những bước đầu tiên với tư cách là nhà phát triển web hoặc nâng cao kiến thức chuyên môn của bạn lên một tầm cao mới. Chúng tôi nóng lòng muốn xem những gì bạn xây dựng.

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

<a name="your-first-laravel-project"></a>
## Project Laravel đầu tiên

Chúng tôi muốn việc bắt đầu với Laravel trở nên dễ dàng nhất có thể. Có nhiều tùy chọn để phát triển và chạy dự án Laravel trên máy tính của bạn. Mặc dù bạn có thể muốn khám phá các tùy chọn này sau, nhưng Laravel cung cấp [Sail](/docs/{{version}}/sail), một giải pháp sẵn có để chạy các project Laravel của bạn bằng [Docker](https://www.docker.com).

Docker là một công cụ để chạy các ứng dụng và service trong các "containers" nhỏ, nhẹ, không can thiệp vào cấu hình hoặc phần mềm được cài đặt trên máy tính local của bạn. Điều này có nghĩa là bạn không phải lo lắng về việc cấu hình hoặc thiết lập các công cụ phát triển phức tạp như web server và cơ sở dữ liệu trên máy tính cá nhân của bạn. Để bắt đầu, bạn chỉ cần cài đặt [Docker Desktop](https://www.docker.com/products/docker-desktop).

Laravel Sail là giao diện command-line nhẹ để tương tác với cấu hình Docker mặc định của Laravel. Sail cung cấp điểm khởi đầu tuyệt vời để xây dựng ứng dụng Laravel bằng PHP, MySQL và Redis mà không cần yêu cầu kinh nghiệm về Docker trước đó.

> {tip} Bạn đã là chuyên gia về Docker? Đừng lo lắng! Mọi thứ về Sail có thể được tùy chỉnh bằng cách sử dụng file `docker-compose.yml` có trong Laravel.

<a name="getting-started-on-macos"></a>
### Bắt đầu với macOS

Nếu bạn đang phát triển trên máy Mac và [Docker Desktop](https://www.docker.com/products/docker-desktop) đã được cài đặt, bạn có thể sử dụng một lệnh terminal đơn giản để tạo project Laravel mới. Ví dụ: để tạo một ứng dụng Laravel mới trong thư mục có tên "example-app", bạn có thể chạy lệnh sau trong terminal của bạn:

```nothing
curl -s "https://laravel.build/example-app" | bash
```

Tất nhiên, bạn có thể thay đổi "example-app trong URL này thành bất kỳ thứ gì bạn thích. Thư mục của ứng dụng Laravel sẽ được tạo trong thư mục mà bạn đang chạy lệnh.

Sau khi project được tạo, bạn có thể di chuyển đến thư mục ứng dụng và khởi động Laravel Sail. Laravel Sail cung cấp một giao diện command-line đơn giản để tương tác với cấu hình Docker mặc định của Laravel:

```nothing
cd example-app

./vendor/bin/sail up
```

Lần đầu tiên bạn chạy lệnh Sail `up`, các container ứng dụng của Sail sẽ được built trên máy của bạn. Việc này có thể mất vài phút. **Đừng lo lắng, những lần khởi động Sail tiếp theo sẽ nhanh hơn nhiều.**

Khi container Docker của ứng dụng đã được khởi động xong, bạn có thể truy cập vào ứng dụng trong trình duyệt web của bạn tại: http://localhost.

> {tip} Để tiếp tục tìm hiểu thêm về Laravel Sail, hãy xem lại [tài liệu đầy đủ](/docs/{{version}}/sail).

<a name="getting-started-on-windows"></a>
### Bắt đầu với Windows

Trước khi chúng ta tạo một ứng dụng Laravel mới trên máy Windows của bạn, hãy đảm bảo bạn đã cài đặt [Docker Desktop](https://www.docker.com/products/docker-desktop). Tiếp theo, bạn nên đảm bảo là Windows Subsystem cho Linux 2 (WSL2) đã được cài đặt và được kích hoạt trên máy của bạn. WSL cho phép bạn chạy các tệp lệnh nhị phân Linux nguyên bản trên Windows 10. Bạn có thể tìm thấy thông tin về cách cài đặt và kích hoạt WSL2 trong [tài liệu về môi trường dành cho nhà phát triển](https://docs.microsoft.com/en-us/windows/wsl/install-win10) của Microsoft.

> {tip} Sau khi cài đặt và kích hoạt WSL2 xong, bạn nên đảm bảo rằng Docker Desktop đã được [cấu hình để sử dụng WSL2](https://docs.docker.com/docker-for-windows/wsl/).

Tiếp theo, bạn đã sẵn sàng tạo project Laravel đầu tiên của bạn. Chạy [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab) và bắt đầu phiên terminal mới cho hệ điều hành WSL2 Linux của bạn. Tiếp theo, bạn có thể sử dụng lệnh terminal đơn giản để tạo project Laravel mới. Ví dụ: để tạo một ứng dụng Laravel mới trong thư mục có tên là "example-app", bạn có thể chạy lệnh sau trong terminal của bạn:

```nothing
curl -s https://laravel.build/example-app | bash
```

Tất nhiên, bạn có thể thay đổi "example-app trong URL này thành bất kỳ thứ gì bạn thích. Thư mục của ứng dụng Laravel sẽ được tạo trong thư mục mà bạn đang chạy lệnh.

Sau khi project được tạo, bạn có thể di chuyển đến thư mục ứng dụng và khởi động Laravel Sail. Laravel Sail cung cấp một giao diện command-line đơn giản để tương tác với cấu hình Docker mặc định của Laravel:

```nothing
cd example-app

./vendor/bin/sail up
```

Lần đầu tiên bạn chạy lệnh Sail `up`, các container ứng dụng của Sail sẽ được built trên máy của bạn. Việc này có thể mất vài phút. **Đừng lo lắng, những lần khởi động Sail tiếp theo sẽ nhanh hơn nhiều.**

Khi container Docker của ứng dụng đã được khởi động xong, bạn có thể truy cập vào ứng dụng trong trình duyệt web của bạn tại: http://localhost.

> {tip} Để tiếp tục tìm hiểu thêm về Laravel Sail, hãy xem lại [tài liệu đầy đủ](/docs/{{version}}/sail).

#### Developing Within WSL2

Tất nhiên, bạn cũng sẽ cần có khả năng thay đổi các file ứng dụng Laravel đã được tạo trong quá trình cài đặt WSL2. Để thực hiện điều này, chúng tôi khuyên bạn nên sử dụng IDE [Visual Studio Code](https://code.visualstudio.com) của Microsoft và extension của nhà phát triển cho [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack).

Sau khi cài đặt các công cụ này, bạn có thể mở bất kỳ dự án Laravel nào bằng cách chạy lệnh `code .` từ thư mục gốc của ứng dụng bằng Windows Terminal.

<a name="getting-started-on-linux"></a>
### Bắt đầu với Linux

Nếu bạn đang phát triển trên Linux và [Docker Compose](https://docs.docker.com/compose/install/) đã được cài đặt, bạn có thể sử dụng lệnh terminal đơn giản để tạo dự án Laravel mới. Ví dụ: để tạo một ứng dụng Laravel mới trong thư mục có tên là "example-app", bạn có thể chạy lệnh sau trong terminal của bạn:

```nothing
curl -s https://laravel.build/example-app | bash
```

Tất nhiên, bạn có thể thay đổi "example-app trong URL này thành bất kỳ thứ gì bạn thích. Thư mục của ứng dụng Laravel sẽ được tạo trong thư mục mà bạn đang chạy lệnh.

Sau khi project được tạo, bạn có thể di chuyển đến thư mục ứng dụng và khởi động Laravel Sail. Laravel Sail cung cấp một giao diện command-line đơn giản để tương tác với cấu hình Docker mặc định của Laravel:

```nothing
cd example-app

./vendor/bin/sail up
```

Lần đầu tiên bạn chạy lệnh Sail `up`, các container ứng dụng của Sail sẽ được built trên máy của bạn. Việc này có thể mất vài phút. **Đừng lo lắng, những lần khởi động Sail tiếp theo sẽ nhanh hơn nhiều.**

Khi container Docker của ứng dụng đã được khởi động xong, bạn có thể truy cập vào ứng dụng trong trình duyệt web của bạn tại: http://localhost.

> {tip} Để tiếp tục tìm hiểu thêm về Laravel Sail, hãy xem lại [tài liệu đầy đủ](/docs/{{version}}/sail).

<a name="choosing-your-sail-services"></a>
### Chọn service Sail bạn dùng

Khi tạo một ứng dụng Laravel mới thông qua Sail, bạn có thể sử dụng biến `with` để chọn service nào sẽ được cấu hình trong file `docker-compose.yml` của ứng dụng mới của bạn. Các service có sẵn là `mysql`, `pgsql`, `mariadb`, `redis`, `memcached`, `meilisearch`, `minio`, `selenium` và `mailhog`:

```nothing
curl -s "https://laravel.build/example-app?with=mysql,redis" | bash
```

Nếu bạn không chỉ định service nào mà bạn muốn cấu hình, một stackp mặc định gồm có `mysql`, `redis`, `meilisearch`, `mailhog` và `selenium` sẽ được cấu hình mặc định.

<a name="installation-via-composer"></a>
### Cài đặt qua Composer

Nếu máy tính của bạn đã cài đặt PHP và Composer, bạn có thể tạo một dự án Laravel mới bằng cách sử dụng trực tiếp Composer. Sau khi ứng dụng được tạo, bạn có thể khởi động server phát triển local của Laravel bằng lệnh `serve` của Artisan CLI:

    composer create-project laravel/laravel:^8.0 example-app

    cd example-app

    php artisan serve

<a name="the-laravel-installer"></a>
#### The Laravel Installer

Hoặc, bạn có thể cài đặt Laravel Installer làm library global của Composer:

```nothing
composer global require laravel/installer

laravel new example-app

cd example-app

php artisan serve
```

Hãy chắc chắn rằng thư mục bin của Composer đã có trong `$PATH` của bạn để hệ thống của bạn có thể định vị file lệnh `laravel`. Thư mục này tồn tại ở các vị trí khác nhau tùy theo hệ điều hành của bạn; tuy nhiên, một số vị trí phổ biến bao gồm:

<div class="content-list" markdown="1">

- macOS: `$HOME/.composer/vendor/bin`
- Windows: `%USERPROFILE%\AppData\Roaming\Composer\vendor\bin`
- GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin` or `$HOME/.composer/vendor/bin`

</div>

Để thuận tiện, Laravel installer cũng có thể tạo Git repository cho dự án mới của bạn. Để cho biết rằng bạn muốn tạo một Git repository cho dự án, hãy truyền flag `--git` khi tạo một dự án mới:

```bash
laravel new example-app --git
```

Lệnh này sẽ khởi tạo một Git repository mới cho dự án của bạn và tự động commit cho source mới được tạo đó. Flag `git` sẽ giả sử bạn đã cài và cấu hình Git đúng cách. Bạn cũng có thể sử dụng flag `--branch` để set tên nhánh ban đầu:

```bash
laravel new example-app --git --branch="main"
```

Thay vì sử dụng flag `--git`, bạn cũng có thể sử dụng flag `--github` để tạo một Git repository và cũng có thể tạo một private repository tương ứng trên GitHub:

```bash
laravel new example-app --github
```

Sau đó, repository đã tạo sẽ có sẵn tại `https://github.com/<your-account>/example-app`. Flag `github` sẽ giả định rằng bạn đã cài đúng [GitHub CLI](https://cli.github.com) và được xác thực bằng GitHub. Ngoài ra, bạn nên cài `git` và cấu hình đúng cách. Nếu cần, bạn có thể truyền thêm các flag bổ sung được GitHub CLI hỗ trợ:

```bash
laravel new example-app --github="--public"
```

Bạn có thể sử dụng flag `--organization` để tạo repository trong một tổ chức GitHub cụ thể:

```bash
laravel new example-app --github="--public" --organization="laravel"
```

<a name="initial-configuration"></a>
## Cài đặt cấu hình

Tất cả các file cấu hình cho framework Laravel được lưu trong thư mục `config`. Mỗi tùy chọn đều được mô tả, vì vậy, bạn có thể thoải mái xem qua các file và làm quen với các tùy chọn có sẵn cho bạn.

Laravel hầu như không cần cấu hình thêm. Bạn có thể tự do bắt đầu phát triển! Tuy nhiên, bạn có thể muốn xem lại file `config/app.php` và tài liệu của nó. Nó chứa một số tùy chọn như `timezone` và `locale` mà bạn có thể muốn thay đổi tùy theo ứng dụng của bạn.

<a name="environment-based-configuration"></a>
### Cấu hình file môi trường

Vì nhiều giá trị tùy chọn cấu hình của Laravel có thể khác nhau tùy thuộc vào việc ứng dụng của bạn đang chạy trên máy tính local hay là trên máy chủ web production, nhiều giá trị cấu hình quan trọng được định nghĩa bằng cách sử dụng file `.env` tồn tại ở thư mục gốc của ứng dụng.

File `.env` của bạn không nên được commit vào source code ứng dụng của bạn, vì mỗi nhà phát triển hoặc máy chủ có thể yêu cầu một cấu hình môi trường khác nhau. Hơn nữa, đây sẽ là một rủi ro bảo mật trong trường hợp kẻ xâm nhập có được quyền truy cập vào repository source code của bạn, vì mọi thông tin xác thực nhạy cảm sẽ có thể bị lộ.

> {tip} Để biết thêm thông tin về file `.env` và cấu hình trên môi trường, hãy xem [tài liệu cấu hình](/docs/{{version}}/configuration#environment-configuration).

<a name="directory-configuration"></a>
### Cấu hình thư mục

Laravel phải luôn được chạy từ thư mục gốc của "thư mục web" được cấu hình cho web server của bạn. Bạn không nên cố gắng chạy ứng dụng Laravel ngoài thư mục con của "thư mục web". Cố gắng làm như vậy sẽ có thể làm lộ các file nhạy cảm đã tồn tại trong ứng dụng của bạn.

<a name="next-steps"></a>
## Bước tiếp theo

Bây giờ bạn đã tạo xong project Laravel của bạn, có thể bạn đang tự hỏi nên học gì tiếp theo. Trước tiên, chúng tôi thực sự khuyên bạn nên làm quen với cách Laravel hoạt động bằng cách đọc các tài liệu sau:

<div class="content-list" markdown="1">

- [Request Lifecycle](/docs/{{version}}/lifecycle)
- [Configuration](/docs/{{version}}/configuration)
- [Directory Structure](/docs/{{version}}/structure)
- [Service Container](/docs/{{version}}/container)
- [Facades](/docs/{{version}}/facades)

</div>

Cách bạn muốn sử dụng Laravel như thế nào cũng sẽ quyết định các bước tiếp theo trên hành trình của bạn. Có nhiều cách khác nhau để sử dụng Laravel và chúng ta sẽ khám phá hai trường hợp sử dụng chính của framework ở bên dưới.

<a name="laravel-the-fullstack-framework"></a>
### Laravel cho Full Stack

Laravel có thể phục vụ như một full stack framework. "Full stack" framework, ý chúng tôi muốn nói là bạn sẽ sử dụng Laravel để route các request đến ứng dụng của bạn và hiển thị giao diện người dùng của bạn thông qua [Blade templates](/docs/{{version}}/blade) hoặc sử dụng ứng dụng kết hợp với một single-page application như [Inertia.js](https://inertiajs.com). Đây là cách phổ biến nhất để sử dụng framework Laravel.

Nếu đây là cách mà bạn định sử dụng Laravel, bạn có thể muốn xem tài liệu của chúng tôi về [routing](/docs/{{version}}/routing), [views](/docs/{{version}}/views) hoặc [Eloquent ORM](/docs/{{version}}/eloquent). Ngoài ra, bạn có thể muốn tìm hiểu về các package cộng đồng như [Livewire](https://laravel-livewire.com) và [Inertia.js](https://inertiajs.com). Các package này cho phép bạn vẫn sử dụng Laravel làm full-stack framework trong khi vẫn tận hưởng nhiều lợi ích về giao diện người dùng được cung cấp bởi các ứng dụng JavaScript single-page.

Nếu bạn đang sử dụng Laravel làm full stack framework, chúng tôi cũng đặc biệt khuyến khích bạn tìm hiểu cách biên dịch CSS và JavaScript cho ứng dụng của bạn bằng cách sử dụng [Laravel Mix](/docs/{{version}}/mix).

> {tip} Nếu bạn muốn bắt đầu xây dựng ứng dụng của bạn một cách thuận lợi, hãy xem một trong các [bộ công cụ tạo nhanh ứng dụng](/docs/{{version}}/starter-kits) chính thức của chúng tôi.

<a name="laravel-the-api-backend"></a>
### Laravel cho backend api

Laravel cũng có thể đóng vai trò là backend API cho mộpt ứng dụng single-page JavaScript application hoặc ứng dụng di động. Ví dụ: bạn có thể sử dụng Laravel làm backend API cho ứng dụng [Next.js](https://nextjs.org) của bạn. Trong trường hợp này, bạn có thể sử dụng Laravel để cung cấp [xác thực](/docs/{{version}}/sanctum) và lưu trữ, truy xuất dữ liệu cho ứng dụng của bạn, đồng thời tận dụng các service mạnh mẽ của Laravel như queue, email, thông báo, và nhiều hơn thế nữa.

Nếu đây là cách bạn dự định sử dụng Laravel, bạn có thể muốn xem tài liệu của chúng tôi về [routing](/docs/{{version}}/routing), [Laravel Sanctum](/docs/{{version}}/sanctum) và [Eloquent ORM](/docs/{{version}}/eloquent).

> {tip} Bạn cần bắt đầu xây dựng backend là Laravel và frontend là Next.js? Laravel Breeze sẽ cung cấp [API stack](/docs/{{version}}/starter-kits#breeze-and-next) cũng như [triển khai frontend Next.js](https://github.com/laravel/breeze-next) để bạn có thể bắt đầu sau vài phút.
