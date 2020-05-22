# Laravel Homestead

- [Giới thiệu](#introduction)
- [Cài đặt](#installation-and-setup)
    - [Bước đầu tiên](#first-steps)
    - [Cấu hình Homestead](#configuring-homestead)
    - [Chạy The Vagrant Box](#launching-the-vagrant-box)
    - [Per Project Installation](#per-project-installation)
    - [Cài đặt MariaDB](#installing-mariadb)
    - [Cài đặt Elasticsearch](#installing-elasticsearch)
    - [Lối tắt](#aliases)
- [Daily Usage](#daily-usage)
    - [Accessing Homestead Globally](#accessing-homestead-globally)
    - [Kết nối thông qua SSH](#connecting-via-ssh)
    - [Kết nối tới Databases](#connecting-to-databases)
    - [Adding Additional Sites](#adding-additional-sites)
    - [Biến environment](#environment-variables)
    - [Cấu hình Cron Schedules](#configuring-cron-schedules)
    - [Cấu hình Mailhog](#configuring-mailhog)
    - [Cổng](#ports)
    - [Chia sẻ biến environment của bạn](#sharing-your-environment)
    - [Multiple PHP Versions](#multiple-php-versions)
    - [Web Servers](#web-servers)
- [Network Interfaces](#network-interfaces)
- [Updating Homestead](#updating-homestead)
- [Provider Specific Settings](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Giới thiệu


Laravel strives to make the entire PHP development experience delightful, including your local development environment. [Vagrant](https://www.vagrantup.com) provides a simple, elegant way to manage and provision Virtual Machines.

Laravel Homestead is an official, pre-packaged Vagrant box that provides you a wonderful development environment without requiring you to install PHP, a web server, and any other server software on your local machine. No more worrying about messing up your operating system! Vagrant boxes are completely disposable. If something goes wrong, you can destroy and re-create the box in minutes!

Homestead có thể chạy nhiều hệ điều hành Windows, Mac, hoặc Linux, và chứa server web Nginx, PHP 7.2, PHP 7.1, PHP 7.0, PHP 5.6, MySQL, PostgreSQL, Redis, Memcached, Node, và những công cụ tuyệt vời khác để giúp bạn phát triển application của bạn.

> {note} Nếu bạn đang dùng Windows, bạn có thể cần bật hardware virtualization (VT-x). Nó có thể bật thông qua BIOS của bạn. Nếu bạn đang dùng Hyper-V trên hệ thống UEFI, bạn có thể cần phải tắt Hyper-V để có thể truy cập vào VT-x.

<a name="included-software"></a>
### Software cài đặt sẵn

<div class="content-list" markdown="1">
- Ubuntu 16.04
- Git
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- Apache (Optional)
- MySQL
- MariaDB (Optional)
- Sqlite3
- PostgreSQL
- Composer
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- Elasticsearch (Optional)
- ngrok
</div>

<a name="installation-and-setup"></a>
## Cài đặt

<a name="first-steps"></a>
### Bước đầu tiên

Trước khi chạy môi trường Homestead của bạn, bạn cần phải cài đặt [VirtualBox 5.2](https://www.virtualbox.org/wiki/Downloads), [VMWare](https://www.vmware.com), [Parallels](https://www.parallels.com/products/desktop/) hoặc [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) tốt nhất là [Vagrant](https://www.vagrantup.com/downloads.html). Tất cả các phần mềm này đều cung cấp một các cài đặt dễ dàng cho các hệ điều hành phổ biến hiện nay.

Để dùng phần mềm VMware, bạn sẽ cần mua bản quyền cả VMware Fusion / Workstation và [VMware Vagrant plug-in](https://www.vagrantup.com/vmware). Mặc dù nó không miễn phí, nhưng VMware có thể cung cấp hiệu năng chia sẽ thư mục nhanh hơn.

Để dùng phần mềm Parallels, bạn sẽ cần cài đặt [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels). Nó miễn phí.

Do [Vagrant limitations](https://www.vagrantup.com/docs/hyperv/limitations.html), phần mềm Hyper-V sẽ chặn tất cả các cài đặt về networking.

#### Cài đặt Homestead Vagrant

Khi mà VirtualBox / VMware và Vagrant đã được cài đặt, bạn nên thêm môi trường phát triển `laravel/homestead` vào trong phần cài đặt của Vagrant thông qua câu lệnh trong terminal của bạn. Nó có thể mất vài phút cho download tuỳ thuộc vào tốc độ Internet của bạn.

    vagrant box add laravel/homestead

Nếu câu lệnh trên không chạy được, thì hãy chắc chắn rằng bạn đang sử dụng Vagrant phiên bản mới nhất.

#### Cài đặt Homestead

Bạn có thể cài đặt Homestead bằng cách clone repository. Và xem xét clone repository đó vào một thư mục `Homestead` trong thư mục "home" của bạn, vì Homestead sẽ dống vai trò là máy chủ lưu trữ cho tất cả các dự án Laravel của bạn:

    git clone https://github.com/laravel/homestead.git ~/Homestead

Bạn nên check out một tagged version của Homestead, vì branch `master` không phải lúc nào cũng ổn định. Bạn có thể tìm phiên bản mới nhất ổn định ở [GitHub Release Page](https://github.com/laravel/homestead/releases):

    cd ~/Homestead

    // Clone một bản release cụ thể...
    git checkout v7.1.2

Khi mà bạn đã clone xong Homestead repository, hãy chạy lệnh `bash init.sh` từ trong thư mục Homestead để tạo file configuration `Homestead.yaml`, File `Homestead.yaml` sẽ được đặt trong thư mục Homestead:

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### Cấu hình Homestead

#### Cài đặt Provider

Từ khoá `provider` trong file `Homestead.yaml` của bạn sẽ chỉ ra loại Vagrant nào sẽ được sủ dụng: `virtualbox`, `vmware_fusion`, `vmware_workstation`, `parallels` hoặc `hyperv`. Bạn có thể cài đặt nó đến loại bạn mong muốn:

    provider: virtualbox

#### Cài đặt thư mục chia sẻ

Thuộc tính `folders` trong file `Homestead.yaml` sẽ liệt kê tất cả các thư mục bạn muốn chia sẻ với môi trường Homestead của bạn. Khi các file trong các thư mục này được thay đổi, chúng sẽ được giữ đồng bộ giữa local của bạn và môi trường Homestead. Bạn có thể định cấu hình nhiều thư mục dùng chung nếu cần:

    folders:
        - map: ~/code
          to: /home/vagrant/code

Nếu bạn chỉ tạo một vài trang web, ánh xạ chung này sẽ hoạt động tốt. Tuy nhiên, khi số lượng trang web tiếp tục tăng lên, bạn có thể bắt đầu gặp vấn đề về hiệu suất. Vấn đề này có thể rất rõ ràng trên các máy đời thấp hoặc các dự án có chứa số lượng tệp rất lớn. Nếu bạn đang gặp vấn đề này, hãy thử mapping mọi dự án vào thư mục Vagrant của riêng nó:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/code/project1

        - map: ~/code/project2
          to: /home/vagrant/code/project2

Để bật [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), bạn chỉ cần đơn giản là thêm một flag vào trong thuộc tính `folders` của bạn:

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "nfs"

> {note} Khi dùng NFS, bạn nên cân nhắc cài đặt plug-in [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs). Plug-in này duy trì chính xác các quyền user và group cho các file và thư mục trong  Homestead.

Bạn cũng có thể thêm vào các option khác được hỗ trợ bởi Vagrant [Synced Folders](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) bằng cách liệt kê chúng dưới từ khoá `options`:

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

#### Cấu hình Nginx Site

Bạn chưa biết Nginx? Không sao cả. Thuộc tính `sites` cho phép bạn dễ dàng map một "domain" tới một thư mục trên môi trường Homestead. Một cấu hình site được chứa trong file `Homestead.yaml`. Một lần nữa, bạn có thể thêm nhiều site vào môi trường Homestead của bạn nếu cần. Homestead có thể phục vụ như một môi trường ảo hóa, thuận tiện cho mọi dự án Laravel mà bạn đang làm việc:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public

Nếu bạn muốn thay đổi thuộc tính `sites` khi Homestead đã được cài đặt, thì bạn nên chạy lại lệnh `vagrant reload --provision` để cập nhật cấu hình Nginx trên máy ảo.

#### File Hosts

Bạn phải thêm "domains cho các trang web Nginx của mình vào tệp `hosts` trên máy của bạn. File `hosts` sẽ chuyển hướng các request cho các trang Homestead của bạn vào máy Homestead của bạn. Trên Mac và Linux, tệp này được đặt tại `/etc/hosts`. Trên Windows, nó được đặt tại `CC:\Windows\System32\drivers\etc\hosts`. Các dòng bạn thêm vào tệp này sẽ trông như sau:

    192.168.10.10  homestead.test

Hãy chắc chắn rằng địa chỉ IP được liệt kê là địa chỉ được cài đặt trong tệp `Homestead.yaml` của bạn. Khi bạn đã thêm tên miền vào tệp `hosts` của mình và khởi chạy Vagrant, bạn sẽ có thể truy cập trang web thông qua trình duyệt web của mình:

    http://homestead.test

<a name="launching-the-vagrant-box"></a>
### Chạy Vagrant

Khi bạn đã chỉnh sửa xong `Homestead.yaml` theo ý thích của mình, hãy chạy lệnh` vagrant up` trong thư mục Homestead của bạn. Vagrant sẽ khởi động máy ảo và tự động cấu hình các thư mục và trang web Nginx của bạn.

Để xoá máy ảo, bạn có dùng lệnh `vagrant destroy --force`.

<a name="per-project-installation"></a>
### Cài đặt cho mỗi Project

Thay vì cài đặt Homestead globally và chia sẻ Homestead đó cho tất cả các project của bạn, thay vào đó bạn có thể cấu hình một phiên bản Homestead riêng cho mỗi project bạn quản lý. Cài đặt Homestead cho mỗi project có thể có một lợi thế là nếu bạn gửi một file `Vagrantfile` cùng với dự án của bạn, thì những người khác có thể làm việc với project của bạn chỉ bằng một câu lệnh đơn giản `vagrant up`.

Để cài đặt Homestead vào project của bạn, thì nó cần bạn phải cài Composer:

    composer require laravel/homestead --dev

Khi mà Homestead đã cài đặt xong, thì chạy lệnh `make` để tạo ra 2 file `Vagrantfile` và `Homestead.yaml` ở thư mục root trong project của bạn. Lệnh `make` sẽ tự động cấu hình các thuộc tính `sites` và `folders` trong file `Homestead.yaml`.

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\\bin\\homestead make

Tiếp theo, chạy lệnh `vagrant up` trong terminal để có thể truy cập project của bạn trên địa chỉ `http://homestead.test` trong web browser của bạn. Nhớ rằng, bạn vẫn phải thêm domain `homestead.test` vào file `/etc/hosts` hoặc tên domain khác mà bạn thích.

<a name="installing-mariadb"></a>
### Cài đặt MariaDB

Nếu bạn thích sử dụng MariaDB thay cho MySQL, bạn có thể thêm option `mariadb` vào file `Homestead.yaml`. Cái option này sẽ xoá MySQL và cài đặt MariaDB, MariaDB được phát triển là nhằm thay thế cho cơ sở dữ liệu MySQL nên vì thế bạn vẫn có thể dùng database driver `mysql` trong cấu hình database trong application của bạn.

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="installing-elasticsearch"></a>
### Cài đặt Elasticsearch

Để cài đặt Elasticsearch, thì bạn cần thêm option `elasticsearch` vào file `Homestead.yaml` và phiên bản được support. Mặc định thì nso sẽ tạo ra một cluster với tên là 'homestead'. Bạn đừng nên tạo Elaticsearch hơn một nửa bộ nhớ của hệ điều hành, hãy đảm bảo rằng máy Homestead của bạn có ít nhất là gấp hai lần số mà đã được phân bổ Elaticsearch:

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 4096
    cpus: 4
    provider: virtualbox
    elasticsearch: 6

> {tip} Hăy đọc [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current) để biết làm sao có thể tuỳ chỉnh cho cấu hình của bạn.

<a name="aliases"></a>
### Aliases (Tên viết tắt)

Bạn có thể thêm alias của Bash vào máy Homestead của mình bằng cách sửa đổi file `aliases` trong thư mục Homestead của bạn:

    alias c='clear'
    alias ..='cd ..'

Sau khi bạn đã update file `aliases`, bạn nên chạy lại Homestead bằng lệnh` vagrant reload --provision`. Điều này sẽ đảm bảo rằng các alias mới của bạn đă được cài đặt vào máy ảo.

<a name="daily-usage"></a>
## Hướng dẫn sử dụng

<a name="accessing-homestead-globally"></a>
### Truy cập vào Homestead Globally

Thỉnh thoảng bạn muốn chạy `vagrant up` cho Homestead của bạn từ bất cứ đâu trên máy thật của bạn. Bạn có thể làm điều này trên các máy thật Mac hoặc Linux bằng cách thêm function vào Bash profile của bạn. Trên Windows, bạn có thể thực hiện điều này bằng cách thêm file "batch" vào `PATH` của bạn. Các lệnh này sẽ cho phép bạn chạy bất kỳ lệnh Vagrant từ bất kỳ đâu trên máy thật của bạn và sẽ tự động trỏ lệnh đó vào Homestead mà bạn đã cài đặt:

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

Hãy chắc chắn cái path `~/Homestead` ở câu lệnh trên trỏ vào đúng vị trí mà bạn đã cài Homestead. Khi mà cái function đã được cài đặt, thì bạn có thể chạy lệnh như `homestead up` or `homestead ssh` ở bất cứ đâu mà bạn muốn.

#### Windows

Tạo một file batch `homestead.bat` ở bất kỳ trên máy thật của bạn và hãy làm theo code ở dưới:

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

Hãy chắc chắn cái path `C:\Homestead` ở câu lệnh trên trỏ vào đúng vị trí mà bạn đã cài Homestead trên máy thật. Sau khi bạn đã tạo xong file ở trên, thì bạn thêm vị trí của nó vào `PATH` của bạn, và sau đó bạn có thể chạy lệnh như `homestead up` or `homestead ssh` ở bất cứ đâu mà bạn muốn.

<a name="connecting-via-ssh"></a>
### Kết nối thông qua SSH

Bạn có thế thông qua SSH để truy cập vào máy ảo của bạn bằng cách chạy lệnh `vagrant ssh` trên terminal từ thư mục Homestead.

Nhưng, vì có thể bạn sẽ cần SSH vào máy ảo Homestead thường xuyên, nên hãy cân nhắc thêm "function" được mô tả ở trên vào máy thật của bạn để có thể nhanh chóng truy cập vào Homestead thông qua SSH.

<a name="connecting-to-databases"></a>
### Kết nối đến Databases

Một database `homestead` sẽ được cấu hình cả MySQL và PostgreSQL. Và để thuận tiện hơn thì file cấu hình `.env` của Laravel framework sẽ dùng database này.

Để kết nối đến database MySQL hoặc PostgreSQL từ máy thật của bạn, bạn nên kết nối tới địa chỉ `127.0.0.1` và cổng là `33060` (MySQL) hoặc `54320` (PostgreSQL). Username và Password sẽ là `homestead` và `secret`.

> {note} Bạn chỉ nên sử dụng các cổng không chuẩn này khi kết nối với cơ sở dữ liệu từ máy thật của bạn. Bạn sẽ dùng các cổng mặc định 3306 và 5432 trong file cấu hình database vì Laravel đang chạy trong máy ảo chứ không phải máy thật.

<a name="adding-additional-sites"></a>
### Thêm một site mới

Khi môi trường Homestead của bạn đã được cài đặt và chạy, bạn có thể muốn thêm một site mới cho Laravel application của bạn. Bạn có thể chạy nhiều Laravel application, như bạn mong muốn trên một môi trường Homestead duy nhất. Để thêm những thông tin site mới, hăy thêm site vào file `Homestead.yaml` của bạn:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
        - map: another.test
          to: /home/vagrant/code/another/public

Nếu Vagrant không tự động quản lý file "hosts" của bạn, bạn có thể cần thêm thông tin site mới vào file đó như sau:

    192.168.10.10  homestead.test
    192.168.10.10  another.test

Khi mà một site mới đã được thêm vào, hãy chạy lệnh `vagrant reload --provision` từ thư mục Homestead của bạn.

<a name="site-types"></a>
#### Loại Site

Homestead hỗ trợ nhiều loại site khác nhau, cái mà cho phép bạn dễ dàng chạy các project khác nhau mà không được xây dựng từ Laravel. Ví dụ, chúng ta có thế dễ dàng thêm một Symfony application vào Homestead chỉ cần dùng loại site `symfony2`:

    sites:
        - map: symfony2.test
          to: /home/vagrant/code/Symfony/web
          type: "symfony2"

Các loại site được hỗ trợ là: `apache`, `laravel` (mặc định), `proxy`, `silverstripe`, `statamic`, `symfony2`, và `symfony4`.

<a name="site-parameters"></a>
#### Site Parameters

Bạn có thể thêm các giá trị Nginx `fastcgi_param` vào site của bạn thông qua `params` site. Ví ddụ, chúng ta có thể thêm một `FOO` parameter với giá trị là `BAR`:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          params:
              - key: FOO
                value: BAR

<a name="environment-variables"></a>
### Biến môi trường

Bạn có thể set biến môi trường global bằng cách thêm chúng vào file `Homestead.yaml` của bạn:

    variables:
        - key: APP_ENV
          value: local
        - key: FOO
          value: bar

Sau khi cập nhật file `Homestead.yaml`, hãy tải lại provision cho máy bằng lệnh `vagrant reload --provision`. Điều này sẽ cập nhật cấu hình PHP-FPM cho tất cả các phiên bản PHP đã cài đặt và cũng cập nhật môi trường cho người dùng `vagrant`.

<a name="configuring-cron-schedules"></a>
### Cấu hình schedule cho Cron

Laravel cung cấp một cách thuận tiện để [schedule Cron jobs](/docs/{{version}}/scheduling) bằng lệnh Artisan `schedule:run` để chạy mỗi phút. Lệnh `schedule:run` sẽ kiểm tra các danh sách job đã được cài đặt trong class `App\Console\Kernel` của bạn để xác định các job sẽ được chạy.

Nếu bạn muốn lệnh `schedule:run` được chạy cho một site trong Homestead, bạn có thể thiết lập option `schedule` về giá trị `true` khi thiết lập thông tin site:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          schedule: true

Cron job cho site sẽ được thiết lập trong thư mục `/etc/cron.d` của máy ảo.

<a name="configuring-mailhog"></a>
### Cấu hình Mailhog

Mailhog cho phép bạn dễ dàng gửi email đi và thực hiện việc kiểm tra việc gửi đó mà không cần thực sự gửi mail đến người dùng thật. Để bắt đầu, hãy cập nhật file `.env` của bạn để sử dụng các cài đặt như sau:

    MAIL_DRIVER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

<a name="ports"></a>
### Ports

Mặc định, các cổng dưới đây sẽ được thiết lập để chuyển tiếp tới môi trường Homestead của bạn:

- **SSH:** 2222 &rarr; Forwards To 22
- **ngrok UI:** 4040 &rarr; Forwards To 4040
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **PostgreSQL:** 54320 &rarr; Forwards To 5432
- **Mailhog:** 8025 &rarr; Forwards To 8025

#### Thêm Port

Nếu bạn muốn, bạn có thể thêm port vào Vagrant, và cũng có thể thiết lập giao thức cho chúng:

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

<a name="sharing-your-environment"></a>
### Chia sẻ môi trường của bạn

Đôi khi bạn có thể muốn chia sẻ những gì bạn đang làm việc với đồng nghiệp hoặc khách hàng. Vagrant có một cách tích hợp để hỗ trợ điều này thông qua `share vagrant`; tuy nhiên, điều này sẽ không hoạt động nếu bạn có nhiều trang được cấu hình trong file `Homestead.yaml`.

Để giải quyết vấn đề này, Homestead chứa một lệnh `share` của riêng nó. Để bắt đầu, SSH vào máy Homestead của bạn thông qua `vagrant ssh` và chạy` share homestead.test`. Điều này sẽ chia sẻ trang web `homestead.test` từ file cấu hình` Homestead.yaml` của bạn. Tất nhiên, bạn có thể thay thế bất kỳ trang web được định cấu hình nào khác của bạn cho `homestead.test`:

    share homestead.test

Sau khi chạy lệnh, bạn sẽ thấy màn hình Ngrok xuất hiện chứa nhật ký hoạt động của các URL có thể truy cập công khai cho trang web được chia sẻ. Nếu bạn muốn chỉ định một region, subdomain hoặc tùy chọn thời gian chạy Ngrok khác, bạn có thể thêm chúng vào lệnh `share` của mình:

    share homestead.test -region=eu -subdomain=laravel

> {note} Hãy nhớ rằng, Vagrant vốn không an toàn và bạn đang công khai máy ảo của mình với Internet khi chạy lệnh `share`.

<a name="multiple-php-versions"></a>
### Chạy nhiều phiên bản PHP

> {note} Tính năng này chỉ tương thích với Nginx.

Homestead 6 đã giới thiệu hỗ trợ cho nhiều phiên bản PHP trên cùng một máy ảo. Bạn có thể chỉ định phiên bản PHP nào sẽ sử dụng cho một trang web nhất định trong tệp `Homestead.yaml` của bạn. Các phiên bản PHP có sẵn là: "5.6", "7.0", "7.1" và "7.2" (mặc định):

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          php: "5.6"

Thêm vào đó, bạn có thể sử dụng bất kỳ phiên bản PHP nào được hỗ trợ thông qua CLI:

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list

<a name="web-servers"></a>
### Máy chủ Web

Mặc đinh, Homestead sẽ dùng máy chủ web là Nginx, Tuy nhiên, nó có thể cài đặt thêm Apache, nếu `apache` được thiết lập cho một site nào đó, Mặc dù cả hai máy chủ web đó có thể cài đặt cùng nhau, nhưng mà chúng ta không thể chạy cả hai cùng lúc. Nên lệnh shell `flip` để giảm bớt quá trình chuyển đổi giữa các máy chủ web. Lệnh `flip` tự động xác định máy chủ web nào đang chạy, tắt nó đi và sau đó khởi động máy chủ khác. Để sử dụng lệnh này, SSH vào máy Homestead của bạn và chạy lệnh trong terminal của bạn:

    flip

<a name="network-interfaces"></a>
## Network Interfaces

Thuộc tính `networks` của` Homestead.yaml` cấu hình các network interface cho môi trường Homestead của bạn. Bạn có thể định cấu hình nhiều network interface nếu cần:

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

Để bật một [bridged](https://www.vagrantup.com/docs/networking/public_network.html) interface, hãy cấu hình một `bridge` và đổi loại network sang `public_network`:

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

Để bật một [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), chỉ cần xoá option `ip` từ file cấu hình của bạn:

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="updating-homestead"></a>
## Cập nhật Homestead

Bạn có thể cập nhật Homestead qua 2 bước đơn giản, Đầu tiên, bạn nên cập nhật Vagrant bằng cách dùng lệnh `vagrant box update`:

    vagrant box update

Tiếp theo, bạn cần cập nhật mã nguồn Homestead. Nếu bạn clone từ repository Homestead, bạn có thể chạy lệnh `git pull origin master` từ thư mục bạn đã clone để cập nhật.

Nếu bạn đã cài đặt Homestead thông qua file `composer.json` trong project của bạn, bạn nên sửa version Homestead thành `"laravel/homestead": "^7"` trong file đó, sau đó bạn chỉ cần cập nhật thư viện thông qua composer:

    composer update

<a name="provider-specific-settings"></a>
## Cài đặt Provider cụ thể

<a name="provider-specific-virtualbox"></a>
### VirtualBox

#### `natdnshostresolver`

Mặc định, Homestead định cấu hình cài đặt `natdnshostresolver` thành` on`. Điều này cho phép Homestead sử dụng cài đặt DNS của hệ điều hành máy chủ của bạn. Nếu bạn muốn ghi đè hành vi này, hãy thêm các dòng sau vào file `Homestead.yaml` của bạn:

    provider: virtualbox
    natdnshostresolver: off

#### Link ảo trên Windows

Nếu như các link ảo không hoạt động đúng trên máy Windows của bạn, thì bạn có thể cần thêm khối sau vào `Vagrantfile`:

    config.vm.provider "virtualbox" do |v|
        v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end
