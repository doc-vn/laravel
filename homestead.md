# Laravel Homestead

- [Giới thiệu](#introduction)
- [Cài đặt](#installation-and-setup)
    - [Bước đầu tiên](#first-steps)
    - [Cấu hình Homestead](#configuring-homestead)
    - [Chạy The Vagrant Box](#launching-the-vagrant-box)
    - [Per Project Installation](#per-project-installation)
    - [Cài đặt MariaDB](#installing-mariadb)
    - [Cài đặt MongoDB](#installing-mongodb)
    - [Cài đặt Elasticsearch](#installing-elasticsearch)
    - [Cài đặt Neo4j](#installing-neo4j)
    - [Lối tắt](#aliases)
- [Daily Usage](#daily-usage)
    - [Accessing Homestead Globally](#accessing-homestead-globally)
    - [Kết nối thông qua SSH](#connecting-via-ssh)
    - [Kết nối tới Databases](#connecting-to-databases)
    - [Backup Database](#database-backups)
    - [Adding Additional Sites](#adding-additional-sites)
    - [Biến environment](#environment-variables)
    - [Cấu hình Cron Schedules](#configuring-cron-schedules)
    - [Cấu hình Mailhog](#configuring-mailhog)
    - [Cấu hình Minio](#configuring-minio)
    - [Cổng](#ports)
    - [Chia sẻ biến environment của bạn](#sharing-your-environment)
    - [Multiple PHP Versions](#multiple-php-versions)
    - [Web Servers](#web-servers)
    - [Mail](#mail)
- [Network Interfaces](#network-interfaces)
- [Updating Homestead](#updating-homestead)
- [Provider Specific Settings](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Giới thiệu

Laravel cố gắng làm cho toàn bộ trải nghiệm phát triển PHP của bạn trở nên thú vị, bao gồm cả môi trường phát triển local của bạn. [Vagrant] (https://www.vagrantup.com) cung cấp một cách đơn giản, dễ hiểu để quản lý và cung cấp Virtual Machines.

Laravel Homestead là một Vagrant box đóng gói sẵn, chính thức, cung cấp cho bạn một môi trường phát triển tuyệt vời mà không yêu cầu bạn phải cài đặt PHP, server web hoặc bất kỳ phần mềm server nào khác trên máy local của bạn. Bạn sẽ không cần lo lắng về việc làm rối tung hệ điều hành của bạn! Vagrant box hoàn toàn sẵn sàng để dùng. Nếu xảy ra sự cố, bạn có thể xoá và tạo lại một box mới trong vài phút!

Homestead có thể chạy nhiều hệ điều hành Windows, Mac, hoặc Linux, và chứa server web Nginx, PHP 7.2, PHP 7.1, PHP 7.0, PHP 5.6, MySQL, PostgreSQL, Redis, Memcached, Node, và những công cụ tuyệt vời khác để giúp bạn phát triển application của bạn.

> {note} Nếu bạn đang dùng Windows, bạn có thể cần bật hardware virtualization (VT-x). Nó có thể được bật thông qua BIOS của bạn. Nếu bạn đang dùng Hyper-V trên hệ thống UEFI, bạn có thể cần phải tắt Hyper-V để có thể truy cập vào VT-x.

<a name="included-software"></a>
### Software cài đặt sẵn

<div class="content-list" markdown="1">
- Ubuntu 18.04
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
- Neo4j (Optional)
- MongoDB (Optional)
- Elasticsearch (Optional)
- ngrok
- wp-cli
- Zend Z-Ray
- Go
- Minio
</div>

<a name="installation-and-setup"></a>
## Cài đặt

<a name="first-steps"></a>
### Bước đầu tiên

Trước khi chạy môi trường Homestead của bạn, bạn cần phải cài đặt [VirtualBox 5.2](https://www.virtualbox.org/wiki/Downloads), [VMWare](https://www.vmware.com), [Parallels](https://www.parallels.com/products/desktop/) hoặc [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) tốt nhất là [Vagrant](https://www.vagrantup.com/downloads.html). Tất cả các phần mềm này đều cung cấp một cài đặt dễ dàng dành cho các hệ điều hành phổ biến hiện nay.

Để dùng phần mềm VMware, bạn sẽ cần mua bản quyền cả VMware Fusion / Workstation và [VMware Vagrant plug-in](https://www.vagrantup.com/vmware). Mặc dù nó không miễn phí, nhưng VMware có thể cung cấp những hiệu năng tốt khi chia sẻ thư mục nhanh hơn.

Để dùng phần mềm Parallels, bạn sẽ cần cài đặt [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels). Nó miễn phí.

Do [Vagrant limitations](https://www.vagrantup.com/docs/hyperv/limitations.html), phần mềm Hyper-V sẽ chặn tất cả các cài đặt về networking.

#### Cài đặt Homestead Vagrant

Khi mà VirtualBox / VMware và Vagrant đã được cài đặt, bạn nên thêm môi trường phát triển `laravel/homestead` vào trong phần cài đặt của Vagrant thông qua câu lệnh trong terminal của bạn. Nó có thể mất vài phút cho download tuỳ thuộc vào tốc độ Internet của bạn.

    vagrant box add laravel/homestead

Nếu câu lệnh trên không chạy được, thì hãy chắc chắn rằng bạn đang sử dụng Vagrant phiên bản mới nhất.

#### Cài đặt Homestead

Bạn có thể cài đặt Homestead bằng cách clone repository. Sau đó lưu thư mục đã clone repository vào trong thư mục `Homestead` trong thư mục "home" của bạn, vì Homestead sẽ đóng vai trò là server lưu trữ cho tất cả các dự án Laravel của bạn:

    git clone https://github.com/laravel/homestead.git ~/Homestead

Bạn nên check out một tagged version của Homestead, vì branch `master` không phải lúc nào cũng ổn định. Bạn có thể tìm phiên bản mới nhất ổn định ở [GitHub Release Page](https://github.com/laravel/homestead/releases):

    cd ~/Homestead

    // Clone một bản release cụ thể...
    git checkout v7.16.1

Khi mà bạn đã clone xong Homestead repository, hãy chạy lệnh `bash init.sh` từ trong thư mục Homestead để tạo file configuration `Homestead.yaml`, File `Homestead.yaml` này sẽ được lưu vào trong thư mục Homestead:

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### Cấu hình Homestead

#### Cài đặt Provider

Từ khoá `provider` trong file `Homestead.yaml` của bạn sẽ chỉ ra loại Vagrant nào sẽ được sủ dụng: `virtualbox`, `vmware_fusion`, `vmware_workstation`, `parallels` hoặc `hyperv`. Bạn có thể set nó đến loại mà bạn mong muốn:

    provider: virtualbox

#### Cài đặt thư mục chia sẻ

Thuộc tính `folders` trong file `Homestead.yaml` sẽ liệt kê tất cả các thư mục mà bạn muốn chia sẻ với môi trường Homestead của bạn. Khi các file trong các thư mục này thay đổi, chúng sẽ được giữ đồng bộ giữa local của bạn và môi trường phát triển Homestead. Bạn có thể cấu hình nhiều thư mục dùng chung nếu cần:

    folders:
        - map: ~/code
          to: /home/vagrant/code

Nếu bạn chỉ cần tạo một vài trang web, thì ánh xạ chung này sẽ hoạt động tốt. Tuy nhiên, khi số lượng trang web tiếp tục tăng, bạn có thể bắt đầu gặp vấn đề về hiệu suất. Vấn đề này có thể rất rõ ràng trên các máy đời cũ hoặc các dự án có chứa nhiều file. Nếu bạn đang gặp vấn đề này, hãy thử mapping mọi dự án vào mỗi thư mục Vagrant của riêng nó:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/code/project1

        - map: ~/code/project2
          to: /home/vagrant/code/project2

Để bật [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), bạn đơn giản chỉ cần là thêm một flag vào trong thuộc tính `folders` của bạn:

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "nfs"

> {note} Khi dùng NFS, bạn nên cân nhắc cài đặt plug-in [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs). Plug-in này duy trì chính xác các quyền user và group cho các file và các thư mục trong Homestead.

Bạn cũng có thể thêm vào các option khác được hỗ trợ bởi Vagrant [Synced Folders](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) bằng cách liệt kê chúng dưới từ khoá `options`:

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

#### Cấu hình Nginx Site

Nếu bạn chưa biết Nginx? Không sao cả. Thuộc tính `sites` cho phép bạn dễ dàng map một "domain" tới một thư mục trên môi trường phát triển Homestead. Một cấu hình site mẫu đã được chứa trong file `Homestead.yaml`. Một lần nữa, bạn có thể thêm nhiều site vào môi trường phát triển Homestead của bạn nếu cần. Homestead có thể phục vụ như một môi trường ảo hóa, thuận tiện cho mọi dự án Laravel mà bạn đang làm việc:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public

Nếu bạn muốn thay đổi thuộc tính `sites` khi Homestead đã được cài đặt, thì bạn nên chạy lại lệnh `vagrant reload --provision` để cập nhật cấu hình Nginx trên máy ảo.

#### File Hosts

Bạn phải thêm "domains" cho các trang web Nginx của bạn vào file `hosts` trên máy của bạn. File `hosts` sẽ chuyển hướng các request đến các trang Homestead của bạn vào máy ảo Homestead của bạn. Trên Mac và Linux, file này được lưu tại `/etc/hosts`. Trên Windows, file đó được lưu tại `C:\Windows\System32\drivers\etc\hosts`. Các dòng mà bạn cần thêm vào trong file này sẽ trông như sau:

    192.168.10.10  homestead.test

Hãy chắc chắn rằng địa chỉ IP được liệt kê là địa chỉ được cài đặt trong file `Homestead.yaml` của bạn. Khi bạn đã thêm tên miền vào file `hosts` của bạn và khởi chạy Vagrant, bạn có thể truy cập trang web thông qua trình duyệt web của bạn:

    http://homestead.test

<a name="launching-the-vagrant-box"></a>
### Chạy Vagrant

Khi bạn đã chỉnh sửa xong file `Homestead.yaml` theo cách bạn muốn, hãy chạy lệnh `vagrant up` trong thư mục Homestead của bạn. Vagrant sẽ khởi động máy ảo và tự động cấu hình các thư mục và trang web Nginx cho bạn.

Để xoá máy ảo, bạn có dùng lệnh `vagrant destroy --force`.

<a name="per-project-installation"></a>
### Cài đặt cho mỗi Project

Thay vì cài đặt Homestead globally và chia sẻ Homestead đó cho tất cả các project của bạn, thì bạn có thể cấu hình một phiên bản Homestead dành riêng cho mỗi project mà bạn quản lý. Cài đặt Homestead cho mỗi project có thể có một lợi thế là nếu bạn gửi một file `Vagrantfile` cùng với code trong dự án của bạn, thì những người khác có thể làm việc với project của bạn chỉ bằng một câu lệnh đơn giản là `vagrant up`.

Để cài đặt Homestead vào project của bạn, thì bạn phải cài nó thông qua Composer:

    composer require laravel/homestead --dev

Khi mà Homestead đã cài đặt xong, thì hãy chạy lệnh `make` để tạo ra 2 file `Vagrantfile` và `Homestead.yaml` ở thư mục root trong project của bạn. Lệnh `make` sẽ tự động cấu hình các thuộc tính `sites` và `folders` vào trong file `Homestead.yaml`.

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\\bin\\homestead make

Tiếp theo, chạy lệnh `vagrant up` trong terminal để có thể truy cập vào project của bạn trên địa chỉ `http://homestead.test` trên web browser. Hãy nhớ rằng, bạn cần phải thêm domain `homestead.test` vào file `/etc/hosts` hoặc tên domain mà bạn thích.

<a name="installing-mariadb"></a>
### Cài đặt MariaDB

Nếu bạn thích sử dụng MariaDB thay cho MySQL, bạn có thể thêm option `mariadb` vào file `Homestead.yaml`. Option này sẽ xoá MySQL và cài đặt MariaDB, MariaDB được phát triển nhằm thay thế cho cơ sở dữ liệu MySQL nên vì thế bạn vẫn có thể sử dụng database driver `mysql` trong cấu hình database trong application của bạn.

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="installing-mongodb"></a>
### Cài đặt MongoDB

Để cài đặt MongoDB Community Edition, hãy cập nhật file `Homestead.yaml` của bạn với tùy chọn cấu hình sau:

    mongodb: true

Mặc định, cài đặt của MongoDB sẽ lưu tên người dùng cơ sở dữ liệu là `homestead` và mật khẩu là `secret`.

<a name="installing-elasticsearch"></a>
### Cài đặt Elasticsearch

Để cài đặt Elasticsearch, bạn cần thêm option `elasticsearch` vào file `Homestead.yaml` và phiên bản được support, phiên bản này có thể phiên bản chính thức hoặc một phiên bản vụ thể (major.minor.patch). Mặc định thì nó sẽ tạo ra một cluster với tên là 'homestead'. Bạn đừng nên tạo Elaticsearch hơn một nửa bộ nhớ của hệ điều hành, hãy đảm bảo rằng máy ảo Homestead của bạn có ít nhất là gấp hai lần số mà đã được phân bổ cho Elaticsearch:

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 4096
    cpus: 4
    provider: virtualbox
    elasticsearch: 6

> {tip} Hăy đọc [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current) để biết làm thế nào để có thể tuỳ biến nó.

<a name="installing-neo4j"></a>
### Cài đặt Neo4j

[Neo4j](https://neo4j.com/) là một hệ thống quản lý cơ sở dữ liệu theo đồ thị. Để cài đặt Neo4j Community Edition, hãy cập nhật file `Homestead.yaml` của bạn với tùy chọn cấu hình sau:

    neo4j: true

Mặc định, cài đặt của Neo4j sẽ lưu tên người dùng trong cơ sở dữ liệu là `homestead` và mật khẩu là `secret`. Để truy cập vào Neo4j, hãy truy cập vào `http://homestead.test:7474` trên trình duyệt web của bạn. Các cổng `7687` (Bolt), `7474` (HTTP), và `7473` (HTTPS) đã cài đặt sẵn để phục vụ các request từ Neo4j client.

<a name="aliases"></a>
### Aliases (Tên viết tắt)

Bạn có thể thêm các alias của Bash vào trong máy ảo Homestead của bạn bằng cách sửa đổi file `aliases` trong thư mục Homestead của bạn:

    alias c='clear'
    alias ..='cd ..'

Sau khi bạn đã update file `aliases`, bạn nên chạy lại Homestead bằng lệnh `vagrant reload --provision`. Điều này sẽ đảm bảo rằng các alias mới của bạn sẽ được cài đặt vào máy ảo.

<a name="daily-usage"></a>
## Hướng dẫn sử dụng

<a name="accessing-homestead-globally"></a>
### Truy cập vào Homestead Globally

Thỉnh thoảng bạn có thể muốn chạy lệnh `vagrant up` cho Homestead từ bất cứ đâu trên máy thật của bạn. Bạn có thể làm điều này trên các loại máy Mac hoặc Linux bằng cách thêm function vào Bash profile của bạn. Trên Windows, bạn có thể thực hiện điều này bằng cách thêm file "batch" vào `PATH` của bạn. Các lệnh này sẽ cho phép bạn chạy bất kỳ lệnh Vagrant nào từ bất kỳ đâu trên máy thật của bạn và sẽ tự động trỏ lệnh đó vào Homestead mà bạn đã cài đặt:

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

Bạn hãy chắc chắn là path `~/Homestead` ở câu lệnh trên đang trỏ vào đúng vị trí mà bạn đã cài Homestead. Sau khi bạn đã thêm function đó vào file bash profile của bạn rồi, thì bạn có thể chạy lệnh như `homestead up` or `homestead ssh` ở bất cứ đâu mà bạn muốn.

#### Windows

Bạn hãy tạo một file batch `homestead.bat` ở trên máy thật của bạn và làm theo code ở dưới đây:

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

Bạn hãy chắc chắn là path `C:\Homestead` ở câu lệnh trên đang trỏ vào đúng vị trí mà bạn đã cài Homestead. Sau khi bạn đã tạo xong file ở trên, bạn có thể thêm vị trí của file đó vào trong `PATH` của bạn, và sau đó bạn có thể chạy các lệnh như `homestead up` hoặc `homestead ssh` ở bất cứ đâu mà bạn muốn.

<a name="connecting-via-ssh"></a>
### Kết nối thông qua SSH

Bạn có thể thông qua SSH để truy cập vào máy ảo của bạn bằng cách chạy lệnh `vagrant ssh` trên terminal từ thư mục Homestead.

Nhưng, nếu bạn cần SSH vào máy ảo Homestead thường xuyên hơn, thì bạn hãy cân nhắc việc thêm "function" đã được mô tả ở trên vào máy thật của bạn để có thể nhanh chóng truy cập vào Homestead thông qua SSH.

<a name="connecting-to-databases"></a>
### Kết nối đến Databases

Một database `homestead` sẽ được cấu hình cho cả MySQL và PostgreSQL. Để thuận tiện hơn thì file cấu hình `.env` của Laravel framework sẽ dùng database này.

Để kết nối đến database MySQL hoặc PostgreSQL từ máy thật của bạn, bạn cần kết nối tới địa chỉ `127.0.0.1` và cổng là `33060` (MySQL) hoặc `54320` (PostgreSQL). Username và Password sẽ là `homestead` và `secret`.

> {note} Bạn chỉ nên sử dụng các cổng không mặc định này khi kết nối với cơ sở dữ liệu từ máy thật của bạn. Bạn sẽ dùng các cổng mặc định 3306 và 5432 trong file cấu hình database vì Laravel đang chạy từ máy ảo chứ không phải máy thật của bạn.

<a name="database-backups"></a>
### Backup Database

Homestead có thể tự động backup cơ sở dữ liệu của bạn khi Vagrant box của bạn bị phá hủy. Để sử dụng tính năng này, bạn sẽ phải sử dụng Vagrant 2.1.0 trở lên. Hoặc, nếu bạn đang sử dụng phiên bản Vagrant cũ hơn, bạn phải cài đặt plug-in `vagrant-triggers`. Để bật backup cơ sở dữ liệu tự động, hãy thêm dòng sau vào file `Homestead.yaml` của bạn:

    backup: true

Sau khi đã được cấu hình, Homestead sẽ export cơ sở dữ liệu của bạn sang các thư mục `mysql_backup` và `postgres_backup` khi lệnh `vagrant destroy` được thực thi. Bạn có thể tìm thấy những thư mục này trong thư mục mà bạn đã clone Homestead hoặc trong thư mục gốc của project nếu bạn đang sử dụng phương thức [per project installation](#per-project-installation).

<a name="adding-additional-sites"></a>
### Thêm một site mới

Khi môi trường Homestead của bạn đã được cài đặt và chạy xong, bạn có thể muốn thêm một site mới cho Laravel application của bạn. Bạn có thể chạy nhiều Laravel application nếu muốn trên một môi trường Homestead duy nhất. Để thêm những thông tin site mới này, hăy thêm site vào file `Homestead.yaml` của bạn:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
        - map: another.test
          to: /home/vagrant/code/another/public

Nếu Vagrant không tự động quản lý file "hosts" cho bạn, thì bạn có thể cần thêm thông tin site mới vào file host như sau:

    192.168.10.10  homestead.test
    192.168.10.10  another.test

Khi mà một site mới đã được thêm vào, hãy chạy lại lệnh `vagrant reload --provision` từ thư mục Homestead của bạn.

<a name="site-types"></a>
#### Loại Site

Homestead hỗ trợ nhiều loại site khác nhau, cho phép bạn dễ dàng chạy các project khác nhau mà không cần phải xây dựng từ Laravel. Ví dụ, chúng ta có thể dễ dàng thêm một Symfony application vào Homestead chỉ cần dùng loại site là `symfony2`:

    sites:
        - map: symfony2.test
          to: /home/vagrant/code/Symfony/web
          type: "symfony2"

Các loại site được hỗ trợ là: `apache`, `laravel` (mặc định), `proxy`, `silverstripe`, `statamic`, `symfony2`, và `symfony4`.

<a name="site-parameters"></a>
#### Site Parameters

Bạn có thể thêm các giá trị Nginx `fastcgi_param` vào site của bạn thông qua `params` trong site. Ví dụ, chúng ta có thể thêm một `FOO` parameter với giá trị là `BAR`:

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

Sau khi đã cập nhật xong file `Homestead.yaml`, hãy tải lại provision cho máy ảo bằng lệnh `vagrant reload --provision`. Lệnh này sẽ cập nhật cấu hình PHP-FPM cho tất cả các phiên bản PHP đã cài đặt và cũng cập nhật môi trường cho user `vagrant`.

<a name="configuring-cron-schedules"></a>
### Cấu hình schedule cho Cron

Laravel cung cấp một cách thuận tiện để [schedule Cron jobs](/docs/{{version}}/scheduling) bằng lệnh Artisan `schedule:run`, sẽ được chạy mỗi phút. Lệnh `schedule:run` sẽ kiểm tra danh sách job đã được cài đặt trong class `App\Console\Kernel` của bạn để xác định xem job nào sẽ được chạy.

Nếu bạn muốn lệnh `schedule:run` sẽ chạy cho một site trong Homestead, bạn có thể thiết lập option `schedule` với giá trị `true` khi thiết lập thông tin site:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          schedule: true

Cron job cho site sẽ được thiết lập trong thư mục `/etc/cron.d` trong máy ảo.

<a name="configuring-mailhog"></a>
### Cấu hình Mailhog

Mailhog cho phép bạn dễ dàng gửi email đi và thực hiện việc kiểm tra quá trình gửi mail đó mà không cần phải thực sự gửi mail đến người dùng. Để bắt đầu, hãy cập nhật file `.env` của bạn sử dụng các cài đặt như sau:

    MAIL_DRIVER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

<a name="configuring-minio"></a>
### Cấu hình Minio

Minio là một server lưu trữ đối tượng mã nguồn mở có API tương thích với Amazon S3. Để cài đặt Minio, hãy cập nhật file `Homestead.yaml` của bạn với tùy chọn cấu hình sau:

    minio: true

Mặc định, Minio có sẵn trên cổng 9600. Bạn có thể truy cập vào bảng điều khiển của Minio bằng cách truy cập vào `http://homestead:9600/`. Khóa truy cập mặc định là `homestead`, trong khi khóa bí mật mặc định là `secretkey`. Khi truy cập vào Minio, bạn nên sử dụng region `us-east-1`.

Để sử dụng Minio, bạn sẽ cần điều chỉnh cấu hình S3 disk trong file cấu hình `config/filesystems.php` của bạn. Bạn sẽ cần thêm tùy chọn `use_path_style_endpoint` vào cấu hình disk, cũng như thay đổi `url` thành `endpoint`:

    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'endpoint' => env('AWS_URL'),
        'use_path_style_endpoint' => true
    ]

Cuối cùng, hãy đảm bảo file `.env` của bạn đã có các tùy chọn sau:

    AWS_ACCESS_KEY_ID=homestead
    AWS_SECRET_ACCESS_KEY=secretkey
    AWS_DEFAULT_REGION=us-east-1
    AWS_URL=http://homestead:9600

<a name="ports"></a>
### Ports

Mặc định, các cổng dưới đây sẽ được thiết lập để chuyển tiếp tới môi trường Homestead của bạn:

<div class="content-list" markdown="1">
- **SSH:** 2222 &rarr; Chuyển tới 22
- **ngrok UI:** 4040 &rarr; Chuyển tới 4040
- **HTTP:** 8000 &rarr; Chuyển tới 80
- **HTTPS:** 44300 &rarr; Chuyển tới 443
- **MySQL:** 33060 &rarr; Chuyển tới 3306
- **PostgreSQL:** 54320 &rarr; Chuyển tới 5432
- **MongoDB:** 27017 &rarr; Chuyển tới 27017
- **Mailhog:** 8025 &rarr; Chuyển tới 8025
- **Minio:** 9600 &rarr; Chuyển tới 9600
</div>

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

Đôi khi bạn có thể muốn chia sẻ những gì bạn đang làm việc với đồng nghiệp hoặc khách hàng. Vagrant có một cách để hỗ trợ điều này thông qua `share vagrant`; tuy nhiên, điều này sẽ không hoạt động nếu bạn có nhiều trang đang được cấu hình trong file `Homestead.yaml`.

Để giải quyết vấn đề này, Homestead chứa một lệnh `share` của riêng nó. Để bắt đầu, hãy SSH vào máy ảo Homestead của bạn thông qua `vagrant ssh` và chạy `share homestead.test`. Điều này sẽ chia sẻ trang web `homestead.test` từ file cấu hình` Homestead.yaml` của bạn. Tất nhiên, bạn có thể thay thế `homestead.test` thành bất kỳ trang web nào mà bạn muốn:

    share homestead.test

Sau khi chạy lệnh, bạn sẽ thấy một màn hình Ngrok xuất hiện chứa nhật ký hoạt động của URL và có thể truy cập được từ internet cho trang web vừa được chia sẻ. Nếu bạn muốn chỉ định một region hoặc một subdomain hoặc một tùy chọn thời gian chạy Ngrok, bạn có thể thêm chúng vào lệnh `share` của bạn:

    share homestead.test -region=eu -subdomain=laravel

> {note} Hãy nhớ rằng, Vagrant vốn không an toàn và bạn đang công khai máy ảo của bạn với Internet khi bạn chạy lệnh `share`.

<a name="multiple-php-versions"></a>
### Chạy nhiều phiên bản PHP

> {note} Tính năng này chỉ tương thích với Nginx.

Homestead 6 đã giới thiệu chức năng hỗ trợ cho nhiều phiên bản PHP trên cùng một máy ảo. Bạn có thể chỉ định phiên bản PHP nào sẽ sử dụng cho một trang web nhất định trong file `Homestead.yaml` của bạn. Các phiên bản PHP có sẵn là: "5.6", "7.0", "7.1" và "7.2" (mặc định):

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
### Server Web

Mặc đinh, Homestead sẽ dùng server web là Nginx, Tuy nhiên, nó có thể cài đặt thêm cả Apache, nếu `apache` được thiết lập cho một site nào đó. Mặc dù cả hai web server này đều có thể cài đặt cùng nhau, nhưng chúng ta sẽ không thể chạy cả hai cùng một lúc. Nên lệnh shell `flip` sẽ giảm bớt quá trình chuyển đổi giữa các server web. Lệnh `flip` sẽ tự động xác định web server nào đang chạy, tắt nó đi và sau đó khởi động web server còn lại. Để sử dụng lệnh này, SSH vào máy ảo Homestead của bạn và chạy lệnh đó trong terminal của bạn:

    flip

<a name="mail"></a>
### Mail

Mặc định, homestead có chứa hộp thư Postfix đang lắng nghe trên cổng `1025`. Vì vậy, bạn có thể hướng dẫn ứng dụng của bạn sử dụng `smtp` mail driver trên `localhost` với cổng `1025`. Sau đó, tất cả các mail đã gửi sẽ được xử lý bởi Postfix và được Mailhog bắt lấy. Để xem các email đã gửi của bạn, hãy mở [http://localhost:8025](http://localhost:8025) trên trình duyệt web của bạn.

<a name="network-interfaces"></a>
## Network Interfaces

Thuộc tính `networks` của `Homestead.yaml` sẽ cấu hình các network interface cho môi trường Homestead của bạn. Bạn có thể cấu hình nhiều network interface nếu cần:

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

Để bật một [bridged](https://www.vagrantup.com/docs/networking/public_network.html) interface, hãy cấu hình một `bridge` và đổi loại network sang `public_network`:

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

Để bật một [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), chỉ cần xoá tuỳ chọn `ip` từ file cấu hình của bạn:

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="updating-homestead"></a>
## Cập nhật Homestead

Bạn có thể cập nhật Homestead qua 2 bước đơn giản như sau, Đầu tiên, bạn nên cập nhật Vagrant bằng cách dùng lệnh `vagrant box update`:

    vagrant box update

Tiếp theo, bạn cần cập nhật mã nguồn Homestead. Nếu bạn clone từ repository Homestead, bạn có thể chạy lệnh `git pull origin master` từ thư mục mà bạn đã clone để cập nhật.

Nếu bạn cài đặt Homestead thông qua file `composer.json` trong project của bạn, bạn nên sửa version Homestead thành `"laravel/homestead": "^7"` trong file đó, rồi sau đó bạn chỉ cần cập nhật thư viện thông qua composer:

    composer update

<a name="provider-specific-settings"></a>
## Cài đặt Provider cụ thể

<a name="provider-specific-virtualbox"></a>
### VirtualBox

#### `natdnshostresolver`

Mặc định, Homestead cấu hình `natdnshostresolver` là `on`. Điều này cho phép Homestead sử dụng DNS của hệ điều hành server. Nếu bạn muốn ghi đè hành vi này, hãy thêm các dòng sau vào file `Homestead.yaml` của bạn:

    provider: virtualbox
    natdnshostresolver: off

#### Link ảo trên Windows

Nếu như các link ảo không hoạt động đúng trên máy Windows của bạn, thì bạn có thể cần thêm lệnh sau vào `Vagrantfile`:

    config.vm.provider "virtualbox" do |v|
        v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end
