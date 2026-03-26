# Cài đặt

- [Lời nói đầu](#meet-laravel)
    - [Tại sao lại là Laravel?](#why-laravel)
- [Tạo một Laravel Application](#creating-a-laravel-project)
    - [Cài đặt PHP và Laravel Installer](#installing-php)
    - [Tạo một Application](#creating-an-application)
- [Cài đặt cấu hình](#initial-configuration)
    - [Cấu hình file môi trường](#environment-based-configuration)
    - [Databases và Migrations](#databases-and-migrations)
    - [Cấu hình thư mục](#directory-configuration)
- [Cài đặt bằng Herd](#installation-using-herd)
    - [Herd trên macOS](#herd-on-macos)
    - [Herd trên Windows](#herd-on-windows)
- [IDE Support](#ide-support)
- [Laravel and AI](#laravel-and-ai)
    - [Installing Laravel Boost](#installing-laravel-boost)
- [Bước tiếp theo](#next-steps)
    - [Laravel cho Full Stack](#laravel-the-fullstack-framework)
    - [Laravel cho backend api](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## Lời nói đầu

Laravel là một framework phát triển ứng dụng web với cú pháp tinh tế, hàm ý. Framework web cung cấp cấu trúc và điểm bắt đầu để tạo ứng dụng của bạn, cho phép bạn tập trung vào việc tạo ra thứ gì đó tuyệt vời trong khi chúng tôi sẽ bỏ công sức ra làm chi tiết.

Laravel cố gắng cung cấp trải nghiệm tuyệt vời nhất cho nhà phát triển đồng thời cung cấp các tính năng mạnh mẽ như tích hợp phụ thuộc, lớp abstraction hóa cơ sở dữ liệu, queue và scheduled job, unit và integration test, v.v.

Cho dù bạn là người mới làm quen với PHP web framework hay là người đã có nhiều năm kinh nghiệm, Laravel là một framework có thể phát triển cùng với bạn. Chúng tôi sẽ giúp bạn thực hiện những bước đầu tiên với tư cách là nhà phát triển web hoặc nâng cao kiến thức chuyên môn của bạn lên một tầm cao mới. Chúng tôi nóng lòng muốn xem những gì bạn xây dựng.

<a name="why-laravel"></a>
### Tại sao lại là Laravel?

Có rất nhiều công cụ và framework có sẵn cho bạn khi bạn xây dựng một ứng dụng web. Tuy nhiên, chúng tôi tin rằng Laravel là một lựa chọn tốt nhất để xây dựng các ứng dụng web full-stack hiện đại.

#### A Progressive Framework

Chúng tôi muốn gọi Laravel là một framework "tiến bộ". Bằng cách đó, chúng tôi muốn nói rằng Laravel phát triển cùng với bạn. Nếu bạn mới thực hiện những bước đầu tiên trong quá trình phát triển web, thư viện tài liệu, hướng dẫn và [video hướng dẫn](https://laracasts.com) khổng lồ của Laravel sẽ giúp bạn tìm hiểu các bước cơ bản mà không bị choáng ngợp.

Nếu bạn là nhà phát triển cấp cao, Laravel cũng cung cấp cho bạn các công cụ mạnh mẽ để [tích hợp phụ thuộc](/docs/{{version}}/container), [unit test](/docs/{{version}}/testing), [queue](/docs/{{version}}/queues), [real-time events](/docs/{{version}}/broadcasting), và nhiều hơn thế. Laravel được tinh chỉnh để xây dựng các ứng dụng web chuyên nghiệp và sẵn sàng xử lý khối lượng lớn công việc của doanh nghiệp.

#### A Scalable Framework

Laravel có khả năng mở rộng đáng kinh ngạc. Nhờ tính chất thân thiện của PHP và tính năng hỗ trợ sẵn có của Laravel dành cho các hệ thống bộ nhớ cache phân tán như Redis, việc mở rộng quy mô theo chiều ngang với Laravel thật dễ dàng. Trên thực tế, các ứng dụng Laravel đã dễ dàng mở rộng quy mô để xử lý hàng trăm triệu request mỗi tháng.

Cần mở rộng quy mô cực lớn? Các nền tảng như [Laravel Cloud](https://cloud.laravel.com) cho phép bạn chạy ứng dụng Laravel của bạn ở quy mô gần như vô hạn.

#### An Agent Ready Framework

Các quy ước đặc thù và cấu trúc được định nghĩa rõ ràng của Laravel sẽ khiến nó trở thành một framework lý tưởng cho việc [phát triển được hỗ trợ bởi AI](/docs/{{version}}/ai) thông qua các công cụ như Cursor và Claude Code. Khi bạn yêu cầu một AI agent thêm một controller, nó biết chính xác vị trí cần đặt. Khi bạn cần một migration mới, các quy ước đặt tên và vị trí file đều có thể dự đoán được. Sự nhất quán này giúp loại bỏ những phỏng đoán thường gây ra khó khăn cho các công cụ AI trong các framework khác.

Bên cạnh việc tổ chức file, cú pháp súc tích và tài liệu hướng dẫn toàn diện của Laravel cung cấp cho các AI agent bối cảnh cần thiết để tạo ra mã nguồn chính xác và đúng chuẩn. Các tính năng như quan hệ Eloquent, form request và middleware đều tuân theo các pattern mà các agent có thể hiểu và tái tạo một cách đáng tin cậy. Kết quả là code do AI tạo ra trông giống như được viết bởi một nhà phát triển Laravel chuyên nghiệp, chứ không phải được chắp vá từ các đoạn code PHP chung chung.

Để tìm hiểu thêm về lý do tại sao Laravel là lựa chọn hoàn hảo cho phát triển hỗ trợ bởi AI, hãy xem tài liệu của chúng tôi về [phát triển agentic](/docs/{{version}}/ai).

#### A Community Framework

Laravel kết hợp các package tốt nhất trong hệ sinh thái PHP để cung cấp framework mạnh mẽ và thân thiện nhất với nhà phát triển. Ngoài ra, hàng nghìn nhà phát triển tài năng từ khắp nơi trên thế giới đã [đóng góp cho framework](https://github.com/laravel/framework). Ai biết được, thậm chí có thể bạn sẽ trở thành người đóng góp cho Laravel.

<a name="creating-a-laravel-project"></a>
## Tạo một Laravel Application

<a name="installing-php"></a>
### Cài đặt PHP và Laravel Installer

Trước khi tạo application Laravel đầu tiên, bạn hãy chắc chắn là máy local của bạn đã cài đặt [PHP](https://php.net), [Composer](https://getcomposer.org), và [Laravel installer](https://github.com/laravel/installer). Ngoài ra, bạn nên cài đặt cả [Node và NPM](https://nodejs.org) hoặc [Bun](https://bun.sh/) để có thể biên dịch các asset frontend của ứng dụng.

Nếu bạn chưa cài đặt PHP và Composer trên máy local của bạn, các lệnh sau sẽ cài đặt PHP, Composer và Laravel installer trên macOS, Windows hoặc Linux:

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

```shell tab=Windows PowerShell
# Run as administrator...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

Sau khi bạn đã chạy xong một trong các lệnh trên, bạn nên khởi động lại phiên terminal của bạn. Để cập nhật PHP, Composer và Laravel installer sau khi cài đặt chúng qua `php.new`, bạn có thể chạy lại trong terminal của bạn.

Nếu bạn đã cài đặt PHP và Composer trên máy local của bạn, bạn có thể cài đặt Laravel installer thông qua Composer:

```shell
composer global require laravel/installer
```

> [!NOTE]
> Để có trải nghiệm cài đặt và quản lý PHP thuận tiện, đầy đủ tính năng, hãy xem qua [Laravel Herd](#installation-using-herd).

<a name="creating-an-application"></a>
### Tạo một Application

Sau khi bạn đã cài đặt PHP, Composer và Laravel installer xong, bạn đã sẵn sàng tạo một application Laravel mới. Laravel installer sẽ hỏi bạn chọn framework testing, cơ sở dữ liệu và bộ khởi tạo ưa thích của bạn:

```shell
laravel new example-app
```

Khi application đã được tạo, bạn có thể khởi động server local, queue worker, và server Vite development bằng lệnh `dev` của Composer:

```shell
cd example-app
npm install && npm run build
composer run dev
```

Sau khi bạn đã khởi động server, ứng dụng của bạn sẽ có thể truy cập được trong trình duyệt web của bạn bằng địa chỉ [http://localhost:8000](http://localhost:8000). Tiếp theo, bạn đã sẵn sàng [bắt đầu thực hiện các bước khác trong hệ sinh thái Laravel](#next-steps). Tất nhiên, bạn cũng có thể muốn [cấu hình cơ sở dữ liệu](#databases-and-migrations).

> [!NOTE]
> Nếu bạn muốn có một sự khởi đầu thuận tiện khi phát triển ứng dụng Laravel, thì hãy cân nhắc sử dụng một trong những [bộ khởi tạo](/docs/{{version}}/starter-kits) của chúng tôi. Bộ khởi tạo này cung cấp một nền tảng xác thực có sẵn cả backend và frontend cho ứng dụng Laravel mới của bạn.

<a name="initial-configuration"></a>
## Initial Configuration

Tất cả các file cấu hình cho Laravel framework đều được lưu trong thư mục `config`. Mỗi tùy chọn đều đã được giải thích, vì vậy bạn hãy thoải mái xem qua các file và làm quen với các tùy chọn có sẵn cho bạn.

Laravel hầu như không cần bạn cấu hình thêm bất cứ cấu hình nào khi cài đặt. Bạn có thể thoải mái bắt đầu phát triển! Tuy nhiên, bạn có thể muốn xem qua file `config/app.php` và tài liệu hướng dẫn của nó. Nó chứa một số tùy chọn như `url` và `locale` mà bạn có thể muốn thay đổi theo trạng thái ứng dụng của bạn.

<a name="environment-based-configuration"></a>
### Environment Based Configuration

Vì nhiều giá trị tùy chọn cấu hình của Laravel có thể khác nhau tùy thuộc vào việc ứng dụng của bạn đang chạy trên môi trường local hay môi trường là production, nên nhiều giá trị cấu hình quan trọng được định nghĩa trong file `.env` có ở trong thư mục root của ứng dụng.

File `.env` của bạn không nên được commit vào source control của ứng dụng, vì mỗi nhà phát triển và server của họ sẽ sử dụng ứng dụng của bạn theo nhiều yêu cầu cấu hình khác nhau. Hơn nữa, đây sẽ là rủi ro bảo mật trong trường hợp kẻ xâm nhập có quyền truy cập vào source control của bạn, vì bất kỳ thông tin xác thực nhạy cảm nào cũng sẽ bị lộ.

> [!NOTE]
> Để biết thêm thông tin về file `.env` và cấu hình theo môi trường, hãy xem [tài liệu cấu hình](/docs/{{version}}/configuration#environment-configuration).

<a name="databases-and-migrations"></a>
### Databases and Migrations

Bây giờ bạn đã tạo ứng dụng Laravel của bạn, có lẽ bạn muốn lưu một số dữ liệu vào trong cơ sở dữ liệu. Mặc định, file cấu hình `.env` của ứng dụng của bạn sẽ tương tác với cơ sở dữ liệu SQLite.

Trong quá trình tạo ứng dụng, Laravel đã tạo một file `database/database.sqlite` cho bạn và chạy các migration cần thiết để tạo các bảng cơ sở dữ liệu cho ứng dụng.

Nếu bạn muốn sử dụng một driver cơ sở dữ liệu khác như MySQL hoặc PostgreSQL, bạn có thể cập nhật file cấu hình `.env` của bạn để sử dụng cơ sở dữ liệu thích hợp. Ví dụ, nếu bạn muốn sử dụng MySQL, hãy cập nhật các biến `DB_*` trong file cấu hình `.env` của bạn như sau:

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

Nếu bạn chọn sử dụng cơ sở dữ liệu khác ngoài SQLite, bạn sẽ cần tạo một cơ sở dữ liệu và chạy [các migration cơ sở dữ liệu](/docs/{{version}}/migrations) của ứng dụng:

```shell
php artisan migrate
```

> [!NOTE]
> Nếu bạn đang phát triển trên macOS hoặc Windows và cần cài đặt MySQL, PostgreSQL hoặc Redis ở local, bạn hãy xem xét sử dụng [Herd Pro](https://herd.laravel.com/#plans) hoặc [DBngin](https://dbngin.com/).

<a name="directory-configuration"></a>
### Cấu hình thư mục

Laravel nên được chạy từ thư mục root của "web directory" đã được cấu hình trong server web của bạn. Bạn không nên cố gắng chạy ứng dụng Laravel từ thư mục con của "web directory". Cố gắng làm như vậy có thể làm lộ các file nhạy cảm có trong ứng dụng của bạn.

<a name="installation-using-herd"></a>
## Cài đặt bằng Herd

[Laravel Herd](https://herd.laravel.com) là một môi trường phát triển Laravel và PHP native cực nhanh dành cho macOS và Windows. Herd bao gồm mọi thứ mà bạn cần để bắt đầu phát triển Laravel, bao gồm cả PHP và Nginx.

Sau khi bạn cài đặt xong Herd, bạn đã sẵn sàng bắt đầu phát triển với Laravel. Herd có chứa các command line cho `php`, `composer`, `laravel`, `expose`, `node`, `npm`, và `nvm`.

> [!NOTE]
> [Herd Pro](https://herd.laravel.com/#plans) bổ sung cho Herd các tính năng mạnh mẽ khác, chẳng hạn như khả năng tạo và quản lý cơ sở dữ liệu MySQL, Postgres và Redis local, cũng như xem mail local và giám sát log.

<a name="herd-on-macos"></a>
### Herd trên macOS

Nếu bạn phát triển trên macOS, bạn có thể tải installer của Herd từ [trang web của Herd](https://herd.laravel.com). Và installer này sẽ tự động tải xuống phiên bản PHP mới nhất và cấu hình máy Mac của bạn để luôn chạy [Nginx](https://www.nginx.com/) trong background.

Herd trên macOS sử dụng [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) để hỗ trợ các thư mục "parked". Bất kỳ ứng dụng Laravel nào có trong thư mục parked sẽ tự động được Herd chạy. Mặc định, Herd sẽ tạo một thư mục parked tại `~/Herd` và bạn có thể truy cập bất kỳ ứng dụng Laravel nào có trong thư mục đó trên tên miền `.test` bằng cách sử dụng tên thư mục.

Sau khi bạn đã cài đặt Herd, cách nhanh nhất để tạo một ứng dụng Laravel mới là sử dụng Laravel CLI, được tích hợp sẵn bên trong Herd:

```shell
cd ~/Herd
laravel new my-app
cd my-app
herd open
```

Tất nhiên, bạn luôn có thể quản lý các thư mục đã được "parked" và các cài đặt PHP khác thông qua giao diện người dùng của Herd, bạn có thể được mở nó từ menu Herd ở trên tab bar system của bạn.

Bạn có thể tìm hiểu thêm về Herd bằng cách xem [tài liệu của họ](https://herd.laravel.com/docs).

<a name="herd-on-windows"></a>
### Herd trên Windows

Bạn có thể tải installer của Herd dành cho Windows trên [website của Herd](https://herd.laravel.com/windows). Sau khi quá trình cài đặt hoàn tất, bạn có thể khởi động Herd để hoàn tất quá trình giới thiệu và truy cập vào giao diện người dùng của Herd.

Giao diện người dùng của Herd có thể truy cập vào bằng cách nhấp chuột trái vào biểu tượng Herd trên tab bar của hệ thống của bạn. Nhấp chuột phải sẽ mở thanh menu nhanh với quyền truy cập vào tất cả các công cụ mà bạn cần hàng ngày.

Trong quá trình cài đặt, Herd sẽ tạo một thư mục "parked" trong thư mục home của bạn tại `%USERPROFILE%\Herd`. Bất kỳ ứng dụng Laravel nào có trong thư mục parked sẽ tự động được Herd chạy và bạn có thể truy cập bất kỳ ứng dụng Laravel nào có trong thư mục parked trên miền `.test` thông qua tên thư mục của nó.

Sau khi bạn đã cài đặt Herd, cách nhanh nhất để tạo một ứng dụng Laravel mới là sử dụng Laravel CLI, được tích hợp sẵn bên trong Herd. Để bắt đầu, hãy mở Powershell và chạy các lệnh sau:

```shell
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

Bạn có thể tìm hiểu thêm về Herd bằng cách xem [tài liệu của họ dành cho Windows](https://herd.laravel.com/docs/windows).

<a name="ide-support"></a>
## IDE Support

Bạn có thể thoải mái sử dụng bất kỳ trình code editor nào mà bạn muốn khi phát triển các ứng dụng Laravel. Nếu bạn đang tìm kiếm một editor nhẹ và có khả năng mở rộng, [VS Code](https://code.visualstudio.com) hoặc [Cursor](https://cursor.com) kết hợp với [Laravel VS Code Extension](https://marketplace.visualstudio.com/items?itemName=laravel.vscode-laravel) official cung cấp khả năng hỗ trợ Laravel tuyệt vời với các tính năng như highlight cú pháp, snippet, tích hợp lệnh artisan và tự động hoàn thành thông minh cho các Eloquent model, route, middleware, asset, config và Inertia.js.

Để có sự hỗ trợ toàn diện và mạnh mẽ cho Laravel, hãy tham khảo [PhpStorm](https://www.jetbrains.com/phpstorm/laravel/?utm_source=laravel.com&utm_medium=link&utm_campaign=laravel-2025&utm_content=partner&ref=laravel-2025), một IDE từ JetBrains. Hỗ trợ framework Laravel mặc định của PhpStorm có chứa Blade template, tự động hoàn thành thông minh cho các Eloquent model, route, view, translation và component, cùng với khả năng tạo code mạnh mẽ và điều hướng linh hoạt trong các dự án Laravel.

Đối với những người đang tìm kiếm trải nghiệm phát triển trên nền tảng cloud, [Firebase Studio](https://firebase.studio/) sẽ cung cấp khả năng truy cập tức thì ngay trong trình duyệt để xây dựng ứng dụng Laravel trực tiếp. Không cần thiết lập, Firebase Studio giúp việc bắt đầu xây dựng các ứng dụng Laravel trở nên dễ dàng từ bất kỳ thiết bị nào.

<a name="laravel-and-ai"></a>
## Laravel and AI

[Laravel Boost](https://github.com/laravel/boost) là một công cụ mạnh mẽ giúp thu hẹp khoảng cách giữa các AI coding agent và các ứng dụng Laravel. Boost cung cấp cho các AI agent cùng với các ngữ cảnh, công cụ và hướng dẫn cụ thể cho Laravel để chúng có thể tạo ra code chính xác hơn, phù hợp với phiên bản và tuân thủ các quy ước của Laravel.

Khi cài đặt Boost vào ứng dụng Laravel, các AI agent sẽ có quyền truy cập vào hơn 15 công cụ chuyên dụng, bao gồm khả năng nhận diện các package đang sử dụng, truy vấn cơ sở dữ liệu, tìm kiếm tài liệu Laravel, đọc log trình duyệt, tạo test và thực thi code thông qua Tinker.

Ngoài ra, Boost cung cấp cho các AI agent quyền truy cập vào hơn 17.000 tài liệu hệ sinh thái Laravel đã được vector hóa, dành riêng cho các phiên bản package mà bạn đã cài đặt. Điều này có nghĩa là các agent có thể cung cấp các hướng dẫn nhắm mục tiêu chính xác hơn đến các phiên bản mà dự án của bạn đang sử dụng.

Boost cũng chứa các hướng dẫn AI do Laravel phát triển nhằm giúp các agent tuân thủ các quy ước của framework, viết các test phù hợp và tránh các lỗi phổ biến khi tạo code Laravel.

<a name="installing-laravel-boost"></a>
### Installing Laravel Boost

Boost có thể được cài đặt trong các ứng dụng Laravel 10, 11, 12, và 13 chạy PHP 8.1 trở lên. Để bắt đầu, hãy cài đặt Boost như một development dependency:

```shell
composer require laravel/boost --dev
```

Sau khi cài đặt, hãy chạy installer:

```shell
php artisan boost:install
```

Installer sẽ tự động nhận diện IDE và các AI agent của bạn, cho phép bạn lựa chọn các tính năng phù hợp với dự án của mình. Boost tôn trọng các quy ước dự án hiện có và mặc định không ép buộc các quy tắc style mang tính quan điểm.

> [!NOTE]
Để tìm hiểu thêm về Boost, hãy xem [repository Laravel Boost trên GitHub](https://github.com/laravel/boost).

<a name="adding-custom-ai-guidelines"></a>
#### Adding Custom AI Guidelines

Để bổ sung các hướng dẫn AI tùy chỉnh của riêng bạn vào Laravel Boost, hãy thêm các file `.blade.php` hoặc `.md` vào thư mục `.ai/guidelines/*` của ứng dụng. Những file này sẽ tự động được thêm vào các hướng dẫn của Laravel Boost khi bạn chạy lệnh `boost:install`.

<a name="next-steps"></a>
## Bước tiếp theo

Bây giờ bạn đã tạo xong application Laravel của bạn, có thể bạn đang tự hỏi nên học gì tiếp theo. Trước tiên, chúng tôi thực sự khuyên bạn nên làm quen với cách Laravel hoạt động bằng cách đọc các tài liệu sau:

<div class="content-list" markdown="1">

- [Request Lifecycle](/docs/{{version}}/lifecycle)
- [Configuration](/docs/{{version}}/configuration)
- [Directory Structure](/docs/{{version}}/structure)
- [Frontend](/docs/{{version}}/frontend)
- [Service Container](/docs/{{version}}/container)
- [Facades](/docs/{{version}}/facades)

</div>

Cách bạn muốn sử dụng Laravel như thế nào cũng sẽ quyết định các bước tiếp theo trên hành trình của bạn. Có nhiều cách khác nhau để sử dụng Laravel và chúng ta sẽ khám phá hai trường hợp sử dụng chính của framework ở bên dưới.

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

