# Laravel Valet

- [Giới thiệu](#introduction)
    - [Valet hoặc Homestead](#valet-or-homestead)
- [Cài đặt](#installation)
    - [Nâng cấp](#upgrading)
- [Tạo sites](#serving-sites)
    - [Lệnh "Park"](#the-park-command)
    - [Lệnh "Link"](#the-link-command)
    - [Bảo vệ site với TLS](#securing-sites)
    - [Chạy một site mặc định](#serving-a-default-site)
- [Chia sẻ site](#sharing-sites)
- [Các biến môi trường cho trang web](#site-specific-environment-variables)
- [Proxying Services](#proxying-services)
- [Tuỳ chỉnh Valet Driver](#custom-valet-drivers)
    - [Local Driver](#local-drivers)
- [Các lệnh Valet khác](#other-valet-commands)
- [Thư mục và file valet](#valet-directories-and-files)

<a name="introduction"></a>
## Giới thiệu

Valet là một môi trường phát triển Laravel cho người dùng Mac. Không Vagrant, không file `/etc/hosts`. Bạn có thể công khai site của bạn bằng local tunnel. _Vâng, chúng tôi cũng thích nó._

Laravel Valet sẽ cấu hình máy Mac của bạn chạy [Nginx](https://www.nginx.com/) ở background mỗi khi máy khởi động. Sau đó, dùng [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet sẽ chuyển tất cả các request đến domain `*.test` vào site mà bạn đã cài đặt ở local.

Cụ thể là, bạn sẽ có một môi trường phát triển Laravel nhanh chóng chỉ dùng có khoảng 7 MB RAM. Valet không phải là một sự thay thế hoàn toàn cho Vagrant hoặc Homestead, nhưng cung cấp một sự thay thế tuyệt vời nếu bạn muốn những điều cơ bản, linh hoạt, thích tốc độ cực cao hoặc đang làm việc trên một máy có dung lượng RAM thấp.

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

<a name="valet-or-homestead"></a>
### Valet hoặc Homestead

Như bạn đã biết, Laravel cung cấp [Homestead](/docs/{{version}}/homestead), và các môi trường phát triển Laravel khác. Homestead và Valet sẽ khác nhau về đối tượng người dùng, và cách tiếp cận với môi trường phát triển ở local. Homestead cung cấp một máy ảo Ubuntu cùng với cấu hình Nginx. Homestead sẽ là lựa chọn tốt nếu bạn muốn có một môi trường phát triển Linux được ảo hóa hoàn toàn trên Windows / Linux.

Còn Valet chỉ hỗ trợ Mac và yêu cầu bạn cài đặt PHP và database server trực tiếp vào máy local của bạn. Điều này có thể dễ dàng đạt được bằng cách sử dụng [Homebrew](https://brew.sh/) với các lệnh như `brew install php` và `brew install mysql`. Valet cung cấp một môi trường phát triển local nhanh chóng với mức tiêu thụ tài nguyên tối thiểu, vì vậy thật tuyệt vời cho các nhà phát triển nếu chỉ cần PHP / MySQL và không cần môi trường phát triển ảo hóa.

Cả Valet và Homestead đều là những lựa chọn tuyệt vời để cấu hình môi trường phát triển Laravel cho bạn. Bạn chọn cái nào đều sẽ phụ thuộc vào sở thích cá nhân hoặc nhu cầu của nhóm phát triển của bạn.

<a name="installation"></a>
## Cài đặt

**Valet yêu cầu macOS và [Homebrew](https://brew.sh/). Trước khi cài đặt, bạn nên đảm bảo rằng không có chương trình nào như Apache hoặc Nginx đang chạy ở cổng 80 trên máy local của bạn.**

<div class="content-list" markdown="1">

- Cài đặt hoặc cập nhật [Homebrew](http://brew.sh/) mới nhất bằng cách dùng lệnh `brew update`.
- Cài đặt PHP 7.4 bằng cách dùng lệnh `brew install php` thông qua Homebrew.
- Cài đặt [Composer](https://getcomposer.org).
- Cài đặt Valet bằng Composer thông qua lệnh `composer global require laravel/valet`. Và chắc chắn là thư mục `~/.composer/vendor/bin` này đã có trong "PATH" của máy bạn.
- Chạy lệnh `valet install`. Lệnh này sẽ cấu hình và cài đặt Valet cùng DnsMasq, ngoài ra cũng sẽ đăng ký Valet's daemon chạy mỗi khi máy bạn khởi động.

</div>

Khi Valet đã được cài đặt xong, hãy thử ping đến bất kỳ tên miền nào có đuôi là `* .test` trên terminal của bạn bằng cách sử dụng một lệnh như sau `ping foobar.test`. Nếu Valet được cài đặt chính xác, bạn sẽ thấy tên miền này phản hồi trên `127.0.0.1`.

Valet sẽ tự động khởi động daemon của nó mỗi khi máy bạn khởi động. Bạn sẽ không cần phải chạy `valet start` hoặc `valet install` một lần nào nữa sau khi quá trình cài đặt Valet hoàn tất.

#### Dùng tên miền khác

Mặc định, Valet sẽ cung cấp project của bạn dưới tên miền `.test`. Nếu bạn muốn sử dụng một tên miền khác, bạn có thể sử dụng lệnh `valet tld tld-name`.

Ví dụ: nếu bạn muốn sử dụng tên miền `.app` thay vì tên miền `.test`, hãy chạy `valet tld app` và Valet sẽ tự động thay đổi tên miền project của bạn sang `*.app`.

#### Database

Nếu bạn cần một cơ sở dữ liệu, hãy thử dùng MySQL bằng cách chạy `brew install mysql@5.7` trên terminal. Khi MySQL đã được cài đặt, bạn có thể khởi động nó bằng lệnh `brew services start mysql@5.7`. Sau đó, bạn có thể kết nối đến cơ sở dữ liệu tại địa chỉ `127.0.0.1` với username là `root` và mật khẩu là trống.

#### PHP Versions

Valet cho phép bạn chuyển đổi các phiên bản PHP khác nhau bằng lệnh `valet use php@version`. Valet sẽ cài đặt phiên bản PHP được chỉ định thông qua Brew nếu nó chưa được cài đặt:

    valet use php@7.2

    valet use php

> {note} Valet chỉ cung cấp một phiên bản PHP tại một thời điểm, kể cả khi bạn đã cài đặt nhiều phiên bản PHP.

#### Resetting Your Installation

Nếu bạn gặp khó khăn trong việc cài đặt Valet của bạn chạy đúng cách, hãy chạy lệnh `composer global update`, theo sau là `valet install` để reset lại cài đặt của bạn và nó có thể giải quyết nhiều vấn đề. Trong một số trường hợp hiếm hoi, có thể cần phải "hard reset" Valet bằng cách chạy `valet uninstall --force` và sau đó là `valet install`.

<a name="upgrading"></a>
### Cập nhật

Bạn có thể cập nhật cài đặt Valet của bạn bằng lệnh `composer global update` trong terminal của bạn. Sau khi cập nhật, bạn nên chạy lệnh `valet install` để Valet có thể nâng cấp bổ sung thêm các file cấu hình nếu cần.

<a name="serving-sites"></a>
## Tạo Site

Sau khi Valet được cài đặt xong, bạn có thể bắt đầu tạo site của bạn. Valet cung cấp hai lệnh để giúp bạn tạo các trang web: `park` và `link`.

<a name="the-park-command"></a>
#### The `park` Command

<div class="content-list" markdown="1">

- Tạo một thư mục mới trên máy Mac của bạn, ví dụ như `mkdir ~/Sites`. Tiếp theo, chạy lệnh `cd ~/Sites` và `valet park`. Lệnh `valet park` sẽ đăng ký thư mục hiện tại thành một đường dẫn, mà Valet sẽ tìm kiếm cho site.
- Tiếp theo, tạo môt project Laravel mới vào thư mục mà bạn vừa tạo bằng lệnh `laravel new blog`.
- Và mở trang `http://blog.test` trên web browser của bạn.

</div>

**Đó là tất cả** Bây giờ, bất kỳ project Laravel nào bạn mà được tạo trong thư mục mà đã được park thì nó sẽ tự động được tạo một site tương ứng theo quy tắc là `http://folder-name.test`.

<a name="the-link-command"></a>
#### The `link` Command

Lệnh `link` cũng được dùng để tạo site cho bạn. Lệnh này hữu ích nếu bạn muốn tạo một site trong một thư mục chứ không phải là toàn bộ thư mục.

<div class="content-list" markdown="1">

- Để dùng lệnh này, bạn cần trỏ vào project mà bạn đang muốn tạo site, và chạy lệnh `valet link app-name` trong terminal. Valet sẽ tạo một link ảo trong thư mục `~/.config/valet/Sites`, và nó sẽ trỏ đến thư mục mà bạn đang chạy lệnh ở trên.
- Sau khi bạn đã chạy lệnh `link`, bạn có thể truy cập vào site của bạn trên web browser với link là `http://app-name.test`.

</div>

Để xem danh sách tất cả các thư mục đã được tạo link, hãy chạy lệnh `valet links`. Bạn có thể sử dụng lệnh `valet unlink app-name` để hủy link đó.

> {tip} Bạn có thể sử dụng `valet link` để tạo nhiều sub domain trong cùng một project. Để thêm một sub domain hoặc một tên miền khác vào project của bạn, hãy chạy `valet link domainomain.app-name` từ thư mục project.

<a name="securing-sites"></a>
#### Securing Sites With TLS

Mặc định, Valet sẽ tạo site trên HTTP. Tuy nhiên, nếu bạn muốn tạo một trang web được mã hoá TLS bằng HTTP/2, hãy sử dụng lệnh `secure`. Ví dụ: nếu trang web của bạn đang được Valet tạo trên tên miền là `laravel.test`, thì bạn nên chạy lệnh sau để bảo vệ trang web này:

    valet secure laravel

Để bỏ lớp bảo mật và quay lại dùng HTTP, thì hãy dùng lệnh `unsecure`. Giống như lệnh `secure`, nó chấp nhận host name là bạn không muốn bảo mật:

    valet unsecure laravel

<a name="serving-a-default-site"></a>
### Serving A Default Site

Thỉnh thoảng, bạn có thể muốn cấu hình Valet chạy một trang web "mặc định" thay vì trang `404` khi truy cập vào tên miền `test` không có. Để thực hiện điều này, bạn có thể thêm một tùy chọn `default` vào file cấu hình `~/.config/valet/config.json` của bạn để chứa đường dẫn đến trang web sẽ đóng vai trò là trang web mặc định của bạn:

    "default": "/Users/Sally/Sites/foo",

<a name="sharing-sites"></a>
## Chia sẻ site

Valet đã chứa một lệnh để chia sẻ các trang web ở local của bạn với thế giới, cung cấp một cách dễ dàng để kiểm tra trang web của bạn trên các thiết bị di động hoặc chia sẻ nó với các thành viên trong team của bạn hoặc khách hàng. Không cần phải cài đặt thêm các phần mềm sau khi Valet được cài đặt.

### Sharing Sites Via Ngrok

Để chia sẻ một trang web, hãy trỏ đến thư mục chứa trang web đó trong terminal của bạn và chạy lệnh `valet share`. Một URL sẽ được chèn vào clipboard của bạn và sẵn sàng paste bất kỳ đâu, ví dụ như vào trong trình duyệt của bạn hoặc chia sẻ với team của bạn.

Để ngừng chia sẻ trang web của bạn, hãy nhấn `Control + C` để hủy quá trình.

> {tip} Bạn có thể truyền thêm các tham số cho lệnh chia sẻ, chẳng hạn như `valet share --region=eu`. Để biết thêm thông tin, hãy tham khảo [tài liệu ngrok](https://ngrok.com/docs).

### Sharing Sites Via Expose

Nếu bạn đã cài đặt [Expose](https://beyondco.de/docs/expose), bạn có thể chia sẻ trang web của bạn bằng cách di chuyển đến thư mục chứa trang web của bạn trong terminal và chạy lệnh `expose`. Tham khảo tài liệu expose để biết thêm các tham số command-line mà nó hỗ trợ. Sau khi chia sẻ trang web, Expose sẽ hiển thị một sharable URL mà bạn có thể sử dụng trên các thiết bị khác của bạn hoặc giữa các thành viên trong team.

Để dừng chia sẻ trang web của bạn, hãy nhấn `Control + C` để huỷ process.

### Sharing Sites On Your Local Network

Mặc định, Valet sẽ hạn chế lưu lượng đến địa chỉ IP `127.0.0.1`. Bằng cách này, máy local của bạn sẽ không bị ảnh hưởng bởi các rủi ro bảo mật từ Internet.

Nếu bạn muốn cho phép các thiết bị khác trong mạng nội bộ của mình truy cập được vào các trang web Valet trên máy của bạn thông qua địa chỉ IP của máy (ví dụ: `192.168.1.10/app-name.test`), bạn sẽ cần phải chỉnh sửa file cấu hình Nginx cho trang web của bạn để loại bỏ các hạn chế đối với các lệnh `listen` bằng cách xóa tiền tố `127.0.0.1:` trên lệnh cho các cổng 80 và 443.

Nếu bạn chưa chạy `valet secure` trong project, bạn có thể mở quyền truy cập mạng cho tất cả các trang web không phải HTTPS bằng cách chỉnh sửa file `/usr/local/etc/nginx/valet/valet.conf`. Tuy nhiên, nếu bạn đang chạy trang web của project thông qua HTTPS (bạn đã chạy lệnh `valet secure` cho trang web) thì bạn nên chỉnh sửa file `~/.config/valet/Nginx/app-name.test`.

Khi bạn đã cập nhật cấu hình Nginx của bạn, hãy chạy lệnh `valet restart` để áp dụng các thay đổi cấu hình.

<a name="site-specific-environment-variables"></a>
## Các biến môi trường cho trang web

Một số ứng dụng sử dụng các framework khác có thể phụ thuộc vào các biến môi trường trên server nhưng lại không cung cấp cách thức để các biến đó được cấu hình trong project của bạn. Valet cho phép bạn cấu hình các biến môi trường cho trang web bằng cách thêm một file `.valet-env.php` vào trong thư mục gốc của project của bạn. Các biến này sẽ được thêm vào trong mảng global `$_SERVER`:

    <?php

    // Set $_SERVER['key'] to "value" for the foo.test site...
    return [
        'foo' => [
            'key' => 'value',
        ],
    ];

    // Set $_SERVER['key'] to "value" for all sites...
    return [
        '*' => [
            'key' => 'value',
        ],
    ];

<a name="proxying-services"></a>
## Proxying Services

Thỉnh thoảng bạn có thể muốn proxy một tên miền Valet cho một service khác trên máy local của bạn. Ví dụ: đôi khi bạn có thể cần chạy Valet trong khi cũng cần chạy một trang web khác trong Docker; tuy nhiên, cả Valet và Docker không thể cùng liên kết đến cổng 80 cùng một lúc.

Để giải quyết vấn đề này, bạn có thể sử dụng lệnh `proxy` để tạo proxy. Ví dụ: bạn có thể proxy tất cả các lưu lượng truy cập từ `http://elasticsearch.test` đến `http://127.0.0.1:9200`:

    valet proxy elasticsearch http://127.0.0.1:9200

Bạn có thể xóa proxy đó bằng lệnh `unproxy`:

    valet unproxy elasticsearch

Bạn có thể sử dụng lệnh `proxies` để hiển thị tất cả cấu hình trang web mà đang được proxy:

    valet proxies

<a name="custom-valet-drivers"></a>
## Tuỳ chỉnh Valet Drivers

Bạn có thể viết Valet "driver" của riêng bạn để tạo các application PHP chạy trên framework khác hoặc CMS khác mà không được Valet hỗ trợ. Khi bạn cài đặt Valet, một thư mục `~/.config/valet/Drivers` sẽ được tạo và chứa file `SampleValetDriver.php`. File này sẽ chứa một driver mẫu để trình bày cách viết một driver tuỳ chỉnh. Để viết một driver tuỳ chỉnh thì nó chỉ yêu cầu bạn kế thừa 3 phương thức: `serves`, `isStaticFile`, và `frontControllerPath`.

Tất cả 3 phương thức này đều nhận các giá trị là `$sitePath`, `$siteName`, và `$uri` làm tham số của chúng. `$sitePath` là đường dẫn đến trang web mà đã được tạo trên máy của bạn, chẳng hạn như `/Users/Lisa/Sites/my-project`. `$siteName` là phần "host" hoặc phần "site name" của tên miền(`my-project`). `$uri` là request URI (`/foo/bar`).

Khi mà bạn đã tuỳ chỉnh xong Valet driver, hãy lưu nó vào trong thư mục `~/.config/valet/Drivers` bằng cách sử dụng quy ước đặt tên như sau `FrameworkValetDriver.php`. Ví dụ: nếu bạn đang viết valet driver cho WordPress, thì nên đặt tên file của bạn phải là `WordPressValetDriver.php`.

Hãy xem cách làm mẫu của từng phương thức mà driver Valet của bạn nên làm.

#### Phương thức `serves`

Phương thức `serves` sẽ trả về `true` nếu driver của bạn sẽ xử lý request. Ngược lại, phương thức sẽ trả về `false`. Vì vậy, trong phương thức này, bạn nên xác định xem `$sitePath` đã cho có chứa loại dự án mà bạn đang tạo hay không.

Ví dụ: giả sử chúng ta đang viết một driver `WordPressValetDriver`. Phương thức `serves` của chúng ta có thể trông giống như thế này:

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

#### Phương thức `frontControllerPath`

Phương thức `frontControllPath` sẽ trả về đường dẫn "front controller" của application, thường là tệp "index.php" của bạn hoặc tương đương:

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

Nếu bạn muốn định nghĩa một Valet driver tùy chỉnh cho một application, hãy tạo một `LocalValetDriver.php` trong thư mục gốc của application. Valet driver tùy chỉnh của bạn có thể extent từ class `ValetDriver` hoặc extent từ một driver nào đó của một application hiện có, chẳng hạn như` LaravelValetDriver`:

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
`valet trust` | Thêm quyền sudoer cho Brew và Valet để chạy các lệnh Valet mà không cần xác nhận password.
`valet uninstall` | Gỡ cài đặt Valet: Hiển thị hướng dẫn gỡ cài đặt; hoặc truyền tham số `--force` để bắt xóa tất cả các Valet.

<a name="valet-directories-and-files"></a>
## Thư mục và file valet

Các thông tin về thư mục và các file sau đây có thể hữu ích cho bạn, trong khi bạn khắc phục sự cố với môi trường Valet của bạn:

File / Path | Description
--------- | -----------
`~/.config/valet/` | Chứa tất cả cấu hình của Valet. Bạn có thể muốn tạo ra một bản sao lưu của thư mục này.
`~/.config/valet/dnsmasq.d/` | Chứa cấu hình của DNSMasq.
`~/.config/valet/Drivers/` | Chứa trình các driver Valet tùy chỉnh.
`~/.config/valet/Extensions/` | Chứa các tùy chỉnh extension hoặc command cho Valet.
`~/.config/valet/Nginx/` | Chứa tất cả các cấu hình trang web Nginx do Valet tạo ra. Các file này được tạo lại khi chạy các lệnh `install`, `secure` và `tld`.
`~/.config/valet/Sites/` | Chứa tất cả các link ảo trỏ đến các project của bạn.
`~/.config/valet/config.json` | File cấu hình chính của Valet
`~/.config/valet/valet.sock` | Socket PHP-FPM được sử dụng bởi cấu hình Nginx của Valet. Điều này sẽ chỉ tồn tại nếu PHP đang chạy đúng.
`~/.config/valet/Log/fpm-php.www.log` | User log cho các lỗi của PHP.
`~/.config/valet/Log/nginx-error.log` | User log cho các lỗi của Nginx.
`/usr/local/var/log/php-fpm.log` | System log cho các lỗi của PHP-FPM.
`/usr/local/var/log/nginx` | Chứa Nginx access và error log.
`/usr/local/etc/php/X.X/conf.d` | Chứa các file `*.ini` dành cho các cài đặt cấu hình PHP khác nhau.
`/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf` | File cấu hình PHP-FPM pool.
`~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf` | Cấu hình Nginx mặc định được sử dụng để tạo chứng chỉ trang web.
