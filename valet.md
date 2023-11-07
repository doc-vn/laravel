# Laravel Valet

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Nâng cấp Valet](#upgrading-valet)
- [Tạo sites](#serving-sites)
    - [Lệnh "Park"](#the-park-command)
    - [Lệnh "Link"](#the-link-command)
    - [Bảo vệ site với TLS](#securing-sites)
    - [Chạy một site mặc định](#serving-a-default-site)
- [Chia sẻ site](#sharing-sites)
    - [Chia sẻ site thông qua Ngrok](#sharing-sites-via-ngrok)
    - [Chia sẻ site thông qua Expose](#sharing-sites-via-expose)
    - [Chia sẻ site trên mạng local](#sharing-sites-on-your-local-network)
- [Các biến môi trường cho trang web](#site-specific-environment-variables)
- [Proxying Services](#proxying-services)
- [Tuỳ chỉnh Valet Driver](#custom-valet-drivers)
    - [Local Driver](#local-drivers)
- [Các lệnh Valet khác](#other-valet-commands)
- [Thư mục và file valet](#valet-directories-and-files)

<a name="introduction"></a>
## Giới thiệu

[Laravel Valet](https://github.com/laravel/valet) là một môi trường phát triển cho người dùng macOS. Laravel Valet sẽ cấu hình máy Mac của bạn chạy [Nginx](https://www.nginx.com/) ở background mỗi khi máy khởi động. Sau đó, dùng [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet sẽ chuyển tất cả các request đến domain `*.test` vào site mà bạn đã cài đặt ở local.

Cụ thể là, Valet là một môi trường phát triển Laravel nhanh chóng chỉ dùng có khoảng 7 MB RAM. Valet không phải là một sự thay thế hoàn toàn cho [Sail](/docs/{{version}}/sail) hoặc [Homestead](/docs/{{version}}/homestead), nhưng cung cấp một sự thay thế tuyệt vời nếu bạn muốn những điều cơ bản, linh hoạt, thích tốc độ cực cao hoặc đang làm việc trên một máy có dung lượng RAM thấp.

Mặc định, Valet hỗ trợ những phần sau, nhưng không giới hạn:

<style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1">

- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [Concrete5](https://www.concrete5.org/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [ExpressionEngine](https://www.expressionengine.com/)
- [Jigsaw](https://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)

</div>

Tuy nhiên, bạn có thể mở rộng Valet với [custom drivers](#custom-valet-drivers).

<a name="installation"></a>
## Cài đặt

> {note} Valet yêu cầu macOS và [Homebrew](https://brew.sh/). Trước khi cài đặt, bạn nên đảm bảo rằng không có chương trình nào như Apache hoặc Nginx đang chạy ở cổng 80 trên máy local của bạn.

Để bắt đầu, trước tiên bạn cần đảm bảo là Homebrew đã được cập nhật bằng lệnh `update`:

    brew update

Tiếp theo, bạn nên sử dụng Homebrew để cài đặt PHP:

    brew install php

Sau khi cài đặt PHP xong, bạn đã sẵn sàng cài đặt [Composer package manager](https://getcomposer.org). Ngoài ra, bạn nên đảm bảo là thư mục `~/.composer/vendor/bin` nằm trong "PATH" của hệ thống. Sau khi cài đặt Composer xong, bạn có thể cài đặt Laravel Valet dưới dạng package Composer global:

    composer global require laravel/valet

Cuối cùng, bạn có thể chạy lệnh `install` của Valet. Điều này sẽ cấu hình và cài đặt Valet và DnsMasq. Ngoài ra, các daemon mà Valet phụ thuộc cũng sẽ được cấu hình để khởi chạy khi hệ thống của bạn khởi động:

    valet install

Sau khi cài đặt Valet xong, hãy thử ping đến bất kỳ domain `*.test` nào trên terminal của bạn bằng lệnh như `ping foobar.test`. Nếu Valet được cài đặt chính xác, bạn sẽ thấy domain này phản hồi trên `127.0.0.1`.

Valet sẽ tự động khởi động các service cần thiết mỗi khi máy của bạn khởi động.

<a name="php-versions"></a>
#### PHP Versions

Valet cho phép bạn chuyển đổi các phiên bản PHP khác nhau bằng lệnh `valet use php@version`. Valet sẽ cài đặt phiên bản PHP được chỉ định thông qua Homebrew nếu nó chưa được cài đặt:

    valet use php@7.2

    valet use php

Bạn cũng có thể tạo file `.valetphprc` trong thư mục root của dự án. File `.valetphprc` phải chứa phiên bản PHP mà trang web của bạn sử dụng:

    php@7.2

Khi file này đã được tạo, bạn có thể chỉ cần chạy lệnh `valet use` và lệnh này sẽ xác định phiên bản PHP mặc định của trang web bằng cách đọc file trên.

> {note} Valet chỉ cung cấp một phiên bản PHP tại một thời điểm, kể cả khi bạn đã cài đặt nhiều phiên bản PHP.

<a name="database"></a>
#### Database

Nếu ứng dụng của bạn cần cơ sở dữ liệu, hãy xem [DBngin](https://dbngin.com). DBngin cung cấp công cụ quản lý cơ sở dữ liệu tất cả trong một phần mền miễn phí bao gồm MySQL, PostgreSQL và Redis. Sau khi DBngin đã được cài đặt, bạn có thể kết nối đến cơ sở dữ liệu của bạn tại `127.0.0.1` bằng username là `root` và một empty password.

<a name="resetting-your-installation"></a>
#### Resetting Your Installation

Nếu bạn gặp khó khăn trong việc cài đặt Valet của bạn chạy đúng cách, hãy chạy lệnh `composer global update`, theo sau là `valet install` để reset lại cài đặt của bạn và nó có thể giải quyết nhiều vấn đề. Trong một số trường hợp hiếm hoi, có thể cần phải "hard reset" Valet bằng cách chạy `valet uninstall --force` và sau đó là `valet install`.

<a name="upgrading-valet"></a>
### Nâng cấp Valet

Bạn có thể cập nhật cài đặt Valet của bạn bằng cách chạy lệnh `composer global update` trong terminal của bạn. Sau khi cập nhật, bạn nên chạy lệnh `valet install` để Valet có thể nâng cấp bổ sung thêm các file cấu hình nếu cần.

<a name="serving-sites"></a>
## Tạo Site

Sau khi Valet được cài đặt xong, bạn có thể bắt đầu tạo application Laravel của bạn. Valet cung cấp hai lệnh để giúp bạn tạo application: `park` và `link`.

<a name="the-park-command"></a>
### The `park` Command

Lệnh `park` sẽ đăng ký một thư mục trên máy của bạn để chứa các ứng dụng của bạn. Khi thư mục đã được "parked" vào Valet, tất cả các thư mục con có trong thư mục đó sẽ có thể truy cập được trong trình duyệt web của bạn tại địa chỉ `http://<directory-name>.test`:

    cd ~/Sites

    valet park

Đó là tất cả. Bây giờ, bất kỳ application nào được tạo trong thư mục mà đã được park thì nó sẽ tự động được tạo một site tương ứng theo quy tắc là `http://<directory-name>.test`. Vì vậy, nếu thư mục parked của bạn chứa một thư mục có tên là "laravel", ứng dụng trong thư mục đó sẽ có thể truy cập được tại địa chỉ `http://laravel.test`. Ngoài ra, Valet còn tự động cho phép bạn truy cập trang web bằng wildcard subdomain (`http://foo.laravel.test`).

<a name="the-link-command"></a>
### The `link` Command

Lệnh `link` cũng có thể được dùng để tạo application Laravel cho bạn. Lệnh này hữu ích nếu bạn muốn tạo một site trong một thư mục chứ không phải là toàn bộ thư mục:

    cd ~/Sites/laravel

    valet link

Khi một ứng dụng đã được liên kết với Valet bằng lệnh `link`, bạn có thể truy cập vào ứng dụng bằng tên thư mục của nó. Vì vậy, trang web được liên kết trong ví dụ trên có thể được truy cập tại địa chỉ `http://laravel.test`. Ngoài ra, Valet còn tự động cho phép bạn truy cập trang web bằng cách sử dụng wildcard subdomain (`http://foo.laravel.test`).

Nếu bạn muốn chạy ứng dụng ở một hostname khác, bạn có thể truyền hostname đó cho lệnh `link`. Ví dụ: bạn có thể chạy lệnh sau để tạo ứng dụng tại địa chỉ `http://application.test`:

    cd ~/Sites/laravel

    valet link application

Bạn có thể chạy lệnh `links` để hiển thị danh sách tất cả các thư mục đã được liên kết của bạn:

    valet links

Lệnh `unlink` có thể được sử dụng để hủy liên kết cho một trang web:

    cd ~/Sites/laravel

    valet unlink

<a name="securing-sites"></a>
### Securing Sites With TLS

Mặc định, Valet sẽ tạo site trên HTTP. Tuy nhiên, nếu bạn muốn tạo một trang web được mã hoá TLS bằng HTTP/2, bạn có thể sử dụng lệnh `secure`. Ví dụ: nếu trang web của bạn đang được Valet tạo trên tên miền là `laravel.test`, thì bạn nên chạy lệnh sau để bảo vệ trang web này:

    valet secure laravel

Để bỏ lớp bảo mật và quay lại dùng HTTP, thì hãy dùng lệnh `unsecure`. Giống như lệnh `secure`, nó chấp nhận host name là bạn không muốn bảo mật:

    valet unsecure laravel

<a name="serving-a-default-site"></a>
### Serving A Default Site

Thỉnh thoảng, bạn có thể muốn cấu hình Valet chạy một trang web "mặc định" thay vì trang `404` khi truy cập vào tên miền `test` không có. Để thực hiện điều này, bạn có thể thêm một tùy chọn `default` vào file cấu hình `~/.config/valet/config.json` của bạn để chứa đường dẫn đến trang web sẽ đóng vai trò là trang web mặc định của bạn:

    "default": "/Users/Sally/Sites/foo",

<a name="sharing-sites"></a>
## Chia sẻ site

Valet đã chứa một lệnh để chia sẻ các trang web ở local của bạn với thế giới, cung cấp một cách dễ dàng để kiểm tra trang web của bạn trên các thiết bị di động hoặc chia sẻ nó với các thành viên trong team của bạn hoặc khách hàng.

<a name="sharing-sites-via-ngrok"></a>
### Chia sẻ site thông qua Ngrok

Để chia sẻ một trang web, hãy trỏ đến thư mục chứa trang web đó trong terminal của bạn và chạy lệnh `share` của Valet. Một URL sẽ được chèn vào clipboard của bạn và sẵn sàng paste bất kỳ đâu, ví dụ như vào trong trình duyệt của bạn hoặc chia sẻ với team của bạn:

    cd ~/Sites/laravel

    valet share

Để ngừng chia sẻ trang web của bạn, bạn có thể nhấn `Control + C`. Việc chia sẻ trang web của bạn bằng Ngrok sẽ yêu cầu bạn [tạo tài khoản Ngrok](https://dashboard.ngrok.com/signup) và [thiết lập một authentication token](https://dashboard.ngrok.com/get-started/your-authtoken).

> {tip} Bạn có thể truyền thêm các tham số Ngrok cho lệnh chia sẻ, chẳng hạn như `valet share --region=eu`. Để biết thêm thông tin, hãy tham khảo [tài liệu ngrok](https://ngrok.com/docs).

<a name="sharing-sites-via-expose"></a>
### Chia sẻ site thông qua Expose

Nếu bạn đã cài đặt [Expose](https://expose.dev), bạn có thể chia sẻ trang web của bạn bằng cách di chuyển đến thư mục chứa trang web của bạn trong terminal và chạy lệnh `expose`. Tham khảo [tài liệu Expose](https://expose.dev/docs) để biết thêm thông tin về các tham số command-line mà nó hỗ trợ. Sau khi chia sẻ trang web, Expose sẽ hiển thị một sharable URL mà bạn có thể sử dụng trên các thiết bị khác của bạn hoặc giữa các thành viên trong team:

    cd ~/Sites/laravel

    expose

Để dừng chia sẻ trang web của bạn, bạn có thể nhấn `Control + C`.

<a name="sharing-sites-on-your-local-network"></a>
### Chia sẻ site trên mạng local

Mặc định, Valet sẽ hạn chế lưu lượng đến địa chỉ IP `127.0.0.1` nên máy local của bạn sẽ không bị ảnh hưởng bởi các rủi ro bảo mật từ Internet.

Nếu bạn muốn cho phép các thiết bị khác trong mạng nội bộ của mình truy cập được vào các trang web Valet trên máy của bạn thông qua địa chỉ IP của máy (ví dụ: `192.168.1.10/application.test`), bạn sẽ cần phải chỉnh sửa file cấu hình Nginx cho trang web của bạn để loại bỏ các hạn chế đối với các lệnh `listen`. Bạn có thể làm điều đó bằng cách xóa tiền tố `127.0.0.1:` trên lệnh `listen` cho các cổng 80 và 443.

Nếu bạn chưa chạy `valet secure` trong project, bạn có thể mở quyền truy cập mạng cho tất cả các trang web không phải HTTPS bằng cách chỉnh sửa file `/usr/local/etc/nginx/valet/valet.conf`. Tuy nhiên, nếu bạn đang chạy trang web của project thông qua HTTPS (bạn đã chạy lệnh `valet secure` cho trang web) thì bạn nên chỉnh sửa file `~/.config/valet/Nginx/app-name.test`.

Khi bạn đã cập nhật cấu hình Nginx của bạn, hãy chạy lệnh `valet restart` để áp dụng các thay đổi cấu hình.

<a name="site-specific-environment-variables"></a>
## Các biến môi trường cho trang web

Một số ứng dụng sử dụng các framework khác có thể phụ thuộc vào các biến môi trường trên server nhưng lại không cung cấp cách thức để các biến đó được cấu hình trong project của bạn. Valet cho phép bạn cấu hình các biến môi trường cho trang web bằng cách thêm một file `.valet-env.php` vào trong thư mục gốc của project của bạn. File này sẽ trả về một mảng trang web gồm các cặp biến môi trường sẽ được thêm vào mảng global `$_SERVER` cho mỗi trang web được chỉ định trong mảng:

    <?php

    return [
        // Set $_SERVER['key'] to "value" for the laravel.test site...
        'laravel' => [
            'key' => 'value',
        ],

        // Set $_SERVER['key'] to "value" for all sites...
        '*' => [
            'key' => 'value',
        ],
    ];

<a name="proxying-services"></a>
## Proxying Services

Thỉnh thoảng bạn có thể muốn proxy một tên miền Valet cho một service khác trên máy local của bạn. Ví dụ: đôi khi bạn có thể cần chạy Valet trong khi cũng cần chạy một trang web khác trong Docker; tuy nhiên, cả Valet và Docker không thể cùng liên kết đến cổng 80 cùng một lúc.

Để giải quyết vấn đề này, bạn có thể sử dụng lệnh `proxy` để tạo proxy. Ví dụ: bạn có thể proxy tất cả các lưu lượng truy cập từ `http://elasticsearch.test` đến `http://127.0.0.1:9200`:

```bash
// Proxy over HTTP...
valet proxy elasticsearch http://127.0.0.1:9200

// Proxy over TLS + HTTP/2...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

Bạn có thể xóa proxy đó bằng lệnh `unproxy`:

    valet unproxy elasticsearch

Bạn có thể sử dụng lệnh `proxies` để hiển thị tất cả cấu hình trang web mà đang được proxy:

    valet proxies

<a name="custom-valet-drivers"></a>
## Tuỳ chỉnh Valet Drivers

Bạn có thể viết Valet "driver" của riêng bạn để tạo các application PHP chạy trên framework hoặc CMS mà không được Valet hỗ trợ. Khi bạn cài đặt Valet, một thư mục `~/.config/valet/Drivers` sẽ được tạo và chứa file `SampleValetDriver.php`. File này sẽ chứa một driver mẫu để trình bày cách viết một driver tuỳ chỉnh. Để viết một driver tuỳ chỉnh thì nó chỉ yêu cầu bạn kế thừa 3 phương thức: `serves`, `isStaticFile`, và `frontControllerPath`.

Tất cả 3 phương thức này đều nhận các giá trị là `$sitePath`, `$siteName`, và `$uri` làm tham số của chúng. `$sitePath` là đường dẫn đến trang web mà đã được tạo trên máy của bạn, chẳng hạn như `/Users/Lisa/Sites/my-project`. `$siteName` là phần "host" hoặc phần "site name" của tên miền(`my-project`). `$uri` là request URI (`/foo/bar`).

Khi mà bạn đã tuỳ chỉnh xong Valet driver, hãy lưu nó vào trong thư mục `~/.config/valet/Drivers` bằng cách sử dụng quy ước đặt tên như sau `FrameworkValetDriver.php`. Ví dụ: nếu bạn đang viết valet driver cho WordPress, thì nên đặt tên file của bạn phải là `WordPressValetDriver.php`.

Hãy xem cách làm mẫu của từng phương thức mà driver Valet của bạn nên làm.

<a name="the-serves-method"></a>
#### Phương thức `serves`

Phương thức `serves` sẽ trả về `true` nếu driver của bạn sẽ xử lý request. Ngược lại, phương thức sẽ trả về `false`. Vì vậy, trong phương thức này, bạn nên xác định xem `$sitePath` đã cho có chứa loại dự án mà bạn đang tạo hay không.

Ví dụ: hãy nghĩ rằng, chúng ta đang viết một driver `WordPressValetDriver`. Phương thức `serves` của chúng ta có thể trông giống như thế này:

    /**
     * Determine if the driver serves the request.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

<a name="the-isstaticfile-method"></a>
#### Phương thức `isStaticFile`

`IsStaticFile` sẽ xác định xem request đến có phải là file "static" hay không, chẳng hạn như hình ảnh hoặc stylesheet. Nếu file là static, phương thức sẽ trả về đường dẫn đến file static trên disk. Nếu request đến không phải cho file static, phương thức sẽ trả về `false`:

    /**
     * Determine if the incoming request is for a static file.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {note} phương thức `isStaticFile` sẽ chỉ được gọi nếu phương thức `serves` trả về `true` và request URI không phải là `/`.

<a name="the-frontcontrollerpath-method"></a>
#### Phương thức `frontControllerPath`

Phương thức `frontControllPath` sẽ trả về đường dẫn "front controller" của application, thường là file "index.php" hoặc tương đương:

    /**
     * Get the fully resolved path to the application's front controller.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### Local Drivers

Nếu bạn muốn định nghĩa một Valet driver tùy chỉnh cho một application, hãy tạo một file `LocalValetDriver.php` trong thư mục gốc của application. Valet driver tùy chỉnh của bạn có thể extent từ class `ValetDriver` hoặc extent từ một driver nào đó của một application hiện có, chẳng hạn như` LaravelValetDriver`:

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * Determine if the driver serves the request.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }

        /**
         * Get the fully resolved path to the application's front controller.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="other-valet-commands"></a>
## Các lệnh Valet khác

Lệnh  | Mô tả
------------- | -------------
`valet forget` | Chạy lệnh này từ một thư mục đã được park để xóa thư mục đó ra khỏi danh sách thư mục đã được park.
`valet log` | Xem danh sách các file log được ghi bởi các service của Valet.
`valet paths` | Xem tất cả các đường dẫn đã được park.
`valet restart` | Khởi động lại daemon Valet.
`valet start` | Khởi động daemon Valet.
`valet stop` | Dừng daemon Valet.
`valet trust` | Thêm quyền sudoer cho Brew và Valet để chạy các lệnh Valet mà không cần hỏi password của bạn.
`valet uninstall` | Gỡ cài đặt Valet: hiển thị hướng dẫn gỡ cài đặt. Truyền thêm tuỳ chọn `--force` để bắt xóa tất cả các resource của Valet.

<a name="valet-directories-and-files"></a>
## Thư mục và file valet

Các thông tin về thư mục và các file sau đây có thể hữu ích cho bạn, trong khi bạn khắc phục sự cố với môi trường Valet của bạn:

#### `~/.config/valet`

Chứa tất cả cấu hình của Valet. Bạn có thể muốn tạo một bản sao của thư mục này.

#### `~/.config/valet/dnsmasq.d/`

Thư mục này chứa cấu hình của DNSMasq.

#### `~/.config/valet/Drivers/`

Thư mục này chứa driver của Valet. Driver xác định cách chạy của một framework hoặc một CMS cụ thể.

#### `~/.config/valet/Extensions/`

Thư mục này chứa các extension và lệnh Valet tùy chỉnh.

#### `~/.config/valet/Nginx/`

Thư mục này chứa tất cả các cấu hình trang Nginx của Valet. Các file này sẽ được built lại khi chạy các lệnh `install`, `secure` và `tld`.

#### `~/.config/valet/Sites/`

Thư mục này chứa tất cả các liên kết ảo cho [các project đã được liên kết](#the-link-command) của bạn.

#### `~/.config/valet/config.json`

File này là file cấu hình chính của Valet.

#### `~/.config/valet/valet.sock`

File này là socket PHP-FPM được sử dụng bởi quá trình cài đặt Nginx của Valet. Nó sẽ chỉ tồn tại nếu PHP chạy đúng cách.

#### `~/.config/valet/Log/fpm-php.www.log`

File này là file user log cho các lỗi PHP.

#### `~/.config/valet/Log/nginx-error.log`

File này là file user log cho các lỗi Nginx.

#### `/usr/local/var/log/php-fpm.log`

File này là file system log cho các lỗi PHP-FPM.

#### `/usr/local/var/log/nginx`

Thư mục này chứa file error log và file log Nginx access.

#### `/usr/local/etc/php/X.X/conf.d`

Thư mục này sẽ chứa các file `*.ini` cho các cài đặt cấu hình PHP khác nhau.

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

File này là file cấu hình PHP-FPM pool.

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

File này là file cấu hình Nginx mặc định được sử dụng để tạo chứng chỉ SSL cho trang web của bạn.
