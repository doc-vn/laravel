# Laravel Homestead

- [Giới thiệu](#introduction)
- [Cài đặt](#installation-and-setup)
    - [Bước đầu tiên](#first-steps)
    - [Cấu hình Homestead](#configuring-homestead)
    - [Chạy The Vagrant Box](#launching-the-vagrant-box)
    - [Per Project Installation](#per-project-installation)
    - [Cài đặt các chức năng tuỳ chọn](#installing-optional-features)
    - [Lối tắt](#aliases)
- [Daily Usage](#daily-usage)
    - [Accessing Homestead Globally](#accessing-homestead-globally)
    - [Kết nối thông qua SSH](#connecting-via-ssh)
    - [Kết nối tới Databases](#connecting-to-databases)
    - [Backup Database](#database-backups)
    - [Database Snapshots](#database-snapshots)
    - [Adding Additional Sites](#adding-additional-sites)
    - [Biến environment](#environment-variables)
    - [Wildcard SSL](#wildcard-ssl)
    - [Cấu hình Cron Schedules](#configuring-cron-schedules)
    - [Cấu hình Mailhog](#configuring-mailhog)
    - [Cấu hình Minio](#configuring-minio)
    - [Cổng](#ports)
    - [Chia sẻ biến environment của bạn](#sharing-your-environment)
    - [Multiple PHP Versions](#multiple-php-versions)
    - [Web Servers](#web-servers)
    - [Mail](#mail)
- [Debugging và Profiling](#debugging-and-profiling)
    - [Debug request với Xdebug](#debugging-web-requests)
    - [Debug CLI Application](#debugging-cli-applications)
    - [Profiling Applications với Blackfire](#profiling-applications-with-blackfire)
- [Network Interfaces](#network-interfaces)
- [Mở rộng Homestead](#extending-homestead)
- [Updating Homestead](#updating-homestead)
- [Provider Specific Settings](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Giới thiệu

Laravel cố gắng làm cho toàn bộ trải nghiệm phát triển PHP của bạn trở nên thú vị, bao gồm cả môi trường phát triển local của bạn. [Vagrant](https://www.vagrantup.com) cung cấp một cách đơn giản, dễ hiểu để quản lý và cung cấp Virtual Machines.

Laravel Homestead là một Vagrant box đóng gói sẵn, chính thức, cung cấp cho bạn một môi trường phát triển tuyệt vời mà không yêu cầu bạn phải cài đặt PHP, server web hoặc bất kỳ phần mềm server nào khác trên máy local của bạn. Bạn sẽ không cần lo lắng về việc làm rối tung hệ điều hành của bạn! Vagrant box hoàn toàn sẵn sàng để dùng. Nếu xảy ra sự cố, bạn có thể xoá và tạo lại một box mới trong vài phút!

Homestead có thể chạy trên nhiều hệ điều hành Windows, Mac, hoặc Linux, và chứa Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node, và những công cụ tuyệt vời khác để giúp bạn phát triển application của bạn.

> {note} Nếu bạn đang dùng Windows, bạn có thể cần bật hardware virtualization (VT-x). Nó có thể được bật thông qua BIOS của bạn. Nếu bạn đang dùng Hyper-V trên hệ thống UEFI, bạn có thể cần phải tắt Hyper-V để có thể truy cập vào VT-x.

<a name="included-software"></a>
### Software cài đặt sẵn

<style>
    #software-list > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Ubuntu 18.04
- Git
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL
- lmm for MySQL or MariaDB database snapshots
- Sqlite3
- PostgreSQL (9.6, 10, 11, 12)
- Composer
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli

</div>

<a name="optional-software"></a>
### Optional Software

<style>
    #software-list > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Crystal & Lucky Framework
- Docker
- Elasticsearch
- Gearman
- Go
- Grafana
- InfluxDB
- MariaDB
- MinIO
- MongoDB
- MySQL 8
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- RabbitMQ
- Solr
- Webdriver & Laravel Dusk Utilities

</div>

<a name="installation-and-setup"></a>
## Cài đặt

<a name="first-steps"></a>
### Bước đầu tiên

Trước khi chạy môi trường Homestead của bạn, bạn cần phải cài đặt [VirtualBox 6.x](https://www.virtualbox.org/wiki/Downloads), [VMWare](https://www.vmware.com), [Parallels](https://www.parallels.com/products/desktop/) hoặc [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) tốt nhất là [Vagrant](https://www.vagrantup.com/downloads.html). Tất cả các phần mềm này đều cung cấp một cài đặt dễ dàng dành cho các hệ điều hành phổ biến hiện nay.

Để dùng phần mềm VMware, bạn sẽ cần mua bản quyền cả VMware Fusion / Workstation và [VMware Vagrant plug-in](https://www.vagrantup.com/vmware). Mặc dù nó không miễn phí, nhưng VMware có thể cung cấp những hiệu năng tốt khi chia sẻ thư mục nhanh hơn.

Để dùng phần mềm Parallels, bạn sẽ cần cài đặt [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels). Nó miễn phí.

Do [Vagrant limitations](https://www.vagrantup.com/docs/hyperv/limitations.html), phần mềm Hyper-V sẽ chặn tất cả các cài đặt về networking.

#### Cài đặt Homestead Vagrant

Khi mà VirtualBox / VMware và Vagrant đã được cài đặt, bạn nên thêm môi trường phát triển `laravel/homestead` vào trong phần cài đặt của Vagrant thông qua câu lệnh trong terminal của bạn. Nó có thể mất vài phút cho download tuỳ thuộc vào tốc độ Internet của bạn.

    vagrant box add laravel/homestead

Nếu câu lệnh trên không chạy được, thì hãy chắc chắn rằng bạn đang sử dụng Vagrant phiên bản mới nhất.

> {note} Homestead sẽ định kỳ phát hành box "alpha" / "beta" để testing, điều này có thể ảnh hưởng đến lệnh `vagrant box add`. Nếu bạn gặp sự cố khi chạy lệnh `vagrant box add`, bạn có thể chạy lệnh `vagrant up` và box đúng sẽ được tải xuống khi Vagrant thử khởi động máy ảo.

#### Cài đặt Homestead

Bạn có thể cài đặt Homestead bằng cách clone repository vào trong máy local của bạn. Sau đó lưu thư mục đã clone đó vào trong thư mục cso tên là `Homestead` trong thư mục "home" của bạn, vì Homestead sẽ đóng vai trò là server lưu trữ cho tất cả các dự án Laravel của bạn:

    git clone https://github.com/laravel/homestead.git ~/Homestead

Bạn nên check out một tagged version của Homestead, vì branch `master` không phải lúc nào cũng ổn định. Bạn có thể tìm thấy phiên bản mới nhất ổn định ở [GitHub Release Page](https://github.com/laravel/homestead/releases). Ngoài ra, bạn có thể checkout branch `release` để luôn có bản phát hành mới nhất ổn định:

    cd ~/Homestead

    git checkout release

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
        - map: ~/code/project1
          to: /home/vagrant/project1

> {note} Người dùng Windows không nên sử dụng cú pháp đường dẫn `~/` và thay vào đó nên sử dụng đường dẫn đầy đủ đến project của họ, chẳng hạn như `C:\Users\user\Code\project1`.

Bạn nên map các project nhỏ thành các thư mục riêng của chúng thay vì map toàn bộ thư mục `~/code` của bạn. Khi bạn map một thư mục, máy ảo phải theo dõi tất cả disk IO  cho *mọi file* có trong thư mục đó. Điều đó sẽ dẫn đến các vấn đề về hiệu suất nếu bạn có một lượng lớn các file trong một thư mục.

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1

        - map: ~/code/project2
          to: /home/vagrant/project2

> {note} Bạn không nên mount `.` (thư mục hiện tại) khi sử dụng Homestead. Điều này khiến Vagrant không map được thư mục hiện tại thành `/vagrant` và sẽ phá vỡ các tính năng tùy chọn và gây ra kết quả không mong muốn khi cấp phép.

Để bật [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), bạn đơn giản chỉ cần là thêm một flag vào trong thuộc tính `folders` của bạn:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "nfs"

> {note} Khi dùng NFS trên Windows, bạn nên cân nhắc cài đặt plug-in [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd). Plug-in này duy trì chính xác các quyền user và group cho các file và các thư mục trong Homestead.

Bạn cũng có thể thêm vào các option khác được hỗ trợ bởi Vagrant [Synced Folders](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) bằng cách liệt kê chúng dưới từ khoá `options`:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

#### Cấu hình Nginx Site

Nếu bạn chưa biết Nginx? Không sao cả. Thuộc tính `sites` cho phép bạn dễ dàng map một "domain" tới một thư mục trên môi trường phát triển Homestead. Một cấu hình site mẫu đã được chứa trong file `Homestead.yaml`. Một lần nữa, bạn có thể thêm nhiều site vào môi trường phát triển Homestead của bạn nếu cần. Homestead có thể phục vụ như một môi trường ảo hóa, thuận tiện cho mọi dự án Laravel mà bạn đang làm việc:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public

Nếu bạn muốn thay đổi thuộc tính `sites` khi Homestead đã được cài đặt, thì bạn nên chạy lại lệnh `vagrant reload --provision` để cập nhật cấu hình Nginx trên máy ảo.

> {note} Các script Homestead được xây dựng sao cho phù hợp nhất có thể. Tuy nhiên, nếu bạn đang gặp sự cố trong khi cấp phép, bạn nên xoá và build lại một máy ảo thông qua `vagrant destroy && vagrant up`.

#### Bật hoặc tắt service

Homestead mặc định chạy một số service; tuy nhiên, bạn có thể tùy chỉnh các service nào sẽ được bật hoặc tắt trong quá trình provisioning. Ví dụ: bạn có thể bật PostgreSQL và tắt MySQL:

    services:
        - enabled:
            - "postgresql@12-main"
        - disabled:
            - "mysql"

Các service cụ thể sẽ được chạy hoặc dừng lại dựa theo thứ tự của chúng trong lệnh `enabled` và `disabled`.

<a name="hostname-resolution"></a>
#### Hostname Resolution

Homestead sẽ publish hostname trên `mDNS` để tự động phân giải host. Nếu bạn set `hostname: homestead` trong file `Homestead.yaml` của bạn, thì host của bạn sẽ ở tại địa chỉ `homestead.local`. Mặc định, MacOS, iOS, và các bản phân phối máy tính để bàn Linux sẽ hỗ trợ `mDNS`. Còn Windows sẽ yêu cầu cài đặt [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US).

Sử dụng hostname tự động sẽ hoạt động tốt nhất cho cài đặt "từng dự án" của Homestead. Nếu bạn lưu nhiều trang web trên cùng một phiên bản Homestead, bạn có thể thêm "tên miền" cho các trang web Nginx của bạn vào file `hosts` trên máy của bạn. File `hosts` sẽ chuyển hướng các request đến các trang Homestead của bạn vào máy ảo Homestead của bạn. Trên Mac và Linux, file này được lưu tại `/etc/hosts`. Trên Windows, file đó được lưu tại `C:\Windows\System32\drivers\etc\hosts`. Các dòng mà bạn cần thêm vào trong file này sẽ trông như sau:

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

Tiếp theo, chạy lệnh `vagrant up` trong terminal để có thể truy cập vào project của bạn trên địa chỉ `http://homestead.test` trong web browser. Hãy nhớ rằng, bạn cần phải thêm domain `homestead.test` vào file `/etc/hosts` hoặc tên domain mà bạn thích nếu bạn không muốn sử dụng [hostname resolution](#hostname-resolution)

<a name="installing-optional-features"></a>
### Cài đặt các chức năng tuỳ chọn

Các phần mềm tùy chọn sẽ được cài đặt bằng cách sử dụng cài đặt "features" trong file cấu hình Homestead của bạn. Hầu hết các chức năng đều có thể được bật hoặc tắt bằng giá trị boolean, trong khi đó có một số chức năng cho phép nhiều tùy chọn cấu hình:

    features:
        - blackfire:
            server_id: "server_id"
            server_token: "server_value"
            client_id: "client_id"
            client_token: "client_value"
        - cassandra: true
        - chronograf: true
        - couchdb: true
        - crystal: true
        - docker: true
        - elasticsearch:
            version: 7.9.0
        - gearman: true
        - golang: true
        - grafana: true
        - influxdb: true
        - mariadb: true
        - minio: true
        - mongodb: true
        - mysql8: true
        - neo4j: true
        - ohmyzsh: true
        - openresty: true
        - pm2: true
        - python: true
        - rabbitmq: true
        - solr: true
        - webdriver: true

#### MariaDB

Hãy bật MariaDB rồi xóa MySQL và cài đặt MariaDB. MariaDB server sẽ đóng vai trò thay thế cho MySQL, vì vậy bạn vẫn nên sử dụng driver cơ sở dữ liệu là `mysql` trong cấu hình cơ sở dữ liệu trong ứng dụng của bạn.

#### MongoDB

Mặc định, cài đặt của MongoDB sẽ lưu tên người dùng cơ sở dữ liệu là `homestead` và mật khẩu là `secret`.

#### Elasticsearch

Bạn có thể chỉ định phiên bản được hỗ trợ cho Elasticsearch, có thể là phiên bản chính thức hoặc phiên bản chính xác (major.minor.patch). Cài đặt mặc định sẽ tạo ra một cụm có tên là 'homestead'. Bạn đừng nên cho Elasticsearch nhiều hơn một nửa bộ nhớ của máy ảo, vì vậy hãy đảm bảo rằng máy ảo Homestead của bạn có ít nhất gấp đôi lượng được phân bổ cho Elasticsearch.

> {tip} Hãy xem [tài liệu về Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current) để tìm hiểu cách tùy chỉnh cấu hình của bạn.

#### Neo4j

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

Một database `homestead` sẽ được cấu hình cho cả MySQL và PostgreSQL. Để kết nối đến database MySQL hoặc PostgreSQL từ máy thật của bạn, bạn cần kết nối tới địa chỉ `127.0.0.1` và cổng là `33060` (MySQL) hoặc `54320` (PostgreSQL). Username và Password sẽ là `homestead` và `secret`.

> {note} Bạn chỉ nên sử dụng các cổng không mặc định này khi kết nối với cơ sở dữ liệu từ máy thật của bạn. Bạn sẽ dùng các cổng mặc định 3306 và 5432 trong file cấu hình database vì Laravel đang chạy từ máy ảo chứ không phải máy thật của bạn.

<a name="database-backups"></a>
### Backup Database

Homestead có thể tự động backup cơ sở dữ liệu của bạn khi Vagrant box của bạn bị phá hủy. Để sử dụng tính năng này, bạn sẽ phải sử dụng Vagrant 2.1.0 trở lên. Hoặc, nếu bạn đang sử dụng phiên bản Vagrant cũ hơn, bạn phải cài đặt plug-in `vagrant-triggers`. Để bật backup cơ sở dữ liệu tự động, hãy thêm dòng sau vào file `Homestead.yaml` của bạn:

    backup: true

Sau khi đã được cấu hình, Homestead sẽ export cơ sở dữ liệu của bạn sang các thư mục `mysql_backup` và `postgres_backup` khi lệnh `vagrant destroy` được thực thi. Bạn có thể tìm thấy những thư mục này trong thư mục mà bạn đã clone Homestead hoặc trong thư mục gốc của project nếu bạn đang sử dụng phương thức [per project installation](#per-project-installation).

<a name="database-snapshots"></a>
### Database Snapshots

Homestead hỗ trợ tạo snapshot cho trạng thái của cơ sở dữ liệu MySQL và MariaDB và phân nhánh giữa chúng bằng cách sử dụng [Logical MySQL Manager](https://github.com/Lullabot/lmm). Ví dụ: hãy tưởng tượng bạn đang làm việc trên một trang web có cơ sở dữ liệu nhiều gigabyte. Bạn có thể import cơ sở dữ liệu vào và tạo ra một bản snapshot. Sau khi thực hiện một số công việc và tạo một số nội dung test local, bạn có thể nhanh chóng khôi phục lại trạng thái ban đầu.

Mặc định, LMM sẽ sử dụng chức năng snapshot thin của LVM với hỗ trợ sao chép và ghi. Trên thực tế, điều này có nghĩa là việc thay đổi một hàng trong bảng cơ sở dữ liệu sẽ chỉ khiến cho những thay đổi mà bạn đã thực hiện được ghi vào disk, điều này sẽ giúp tiết kiệm đáng kể thời gian và dung lượng disk trong quá trình khôi phục.

Vì `lmm` tương tác với LVM nên nó phải được chạy dưới quyền `root`. Để xem tất cả các lệnh có sẵn, hãy chạy `sudo lmm` bên trong Vagrant box của bạn. Quy trình làm việc sẽ trông giống như sau:

1. Import cơ sở dữ liệu vào nhánh `master` lmm mặc định.
1. Lưu một snapshot của cơ sở dữ liệu chưa thay đổi bằng cách sử dụng `sudo lmm branch prod-YYYY-MM-DD`.
1. Sửa cơ sở dữ liệu.
1. Chạy `sudo lmm merge prod-YYYY-MM-DD` để undo tất cả các thay đổi.
1. Chạy `sudo lmm delete <branch>` để xóa các nhánh không cần thiết.

<a name="adding-additional-sites"></a>
### Thêm một site mới

Khi môi trường Homestead của bạn đã được cài đặt và chạy xong, bạn có thể muốn thêm một site mới cho Laravel application của bạn. Bạn có thể chạy nhiều Laravel application nếu muốn trên một môi trường Homestead duy nhất. Để thêm những thông tin site mới này, hăy thêm site vào file `Homestead.yaml` của bạn:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
        - map: another.test
          to: /home/vagrant/project2/public

Nếu Vagrant không tự động quản lý file "hosts" cho bạn, thì bạn có thể cần thêm thông tin site mới vào file host như sau:

    192.168.10.10  homestead.test
    192.168.10.10  another.test

Khi mà một site mới đã được thêm vào, hãy chạy lại lệnh `vagrant reload --provision` từ thư mục Homestead của bạn.

<a name="site-types"></a>
#### Loại Site

Homestead hỗ trợ nhiều loại site khác nhau, cho phép bạn dễ dàng chạy các project khác nhau mà không cần phải xây dựng từ Laravel. Ví dụ, chúng ta có thể dễ dàng thêm một Symfony application vào Homestead chỉ cần dùng loại site là `symfony2`:

    sites:
        - map: symfony2.test
          to: /home/vagrant/my-symfony-project/web
          type: "symfony2"

Các loại site được hỗ trợ là: `apache`, `laravel` (mặc định), `proxy`, `silverstripe`, `statamic`, `symfony2`, và `symfony4`.

<a name="site-parameters"></a>
#### Site Parameters

Bạn có thể thêm các giá trị Nginx `fastcgi_param` vào site của bạn thông qua `params` trong site. Ví dụ, chúng ta có thể thêm một `FOO` parameter với giá trị là `BAR`:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
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

<a name="wildcard-ssl"></a>
### Wildcard SSL

Homestead cấu hình chứng chỉ SSL self-signed cho mỗi trang web được định nghĩa trong phần `sites:` của file `Homestead.yaml` của bạn. Nếu bạn muốn tạo một wildcard SSL cho một trang web, bạn có thể thêm một tùy chọn `wildcard` vào cấu hình của trang web đó. Mặc định, trang web đó sẽ sử dụng chứng chỉ wildcard *thay vì* một chứng chỉ cho một tên miền cụ thể:

    - map: foo.domain.test
      to: /home/vagrant/domain
      wildcard: "yes"

Nếu tùy chọn `use_wildcard` được set thành `no`, thì chứng chỉ wildcard sẽ được tạo nhưng sẽ không được sử dụng:

    - map: foo.domain.test
      to: /home/vagrant/domain
      wildcard: "yes"
      use_wildcard: "no"

<a name="configuring-cron-schedules"></a>
### Cấu hình schedule cho Cron

Laravel cung cấp một cách thuận tiện để [schedule Cron jobs](/docs/{{version}}/scheduling) bằng lệnh Artisan `schedule:run`, sẽ được chạy mỗi phút. Lệnh `schedule:run` sẽ kiểm tra danh sách job đã được cài đặt trong class `App\Console\Kernel` của bạn để xác định xem job nào sẽ được chạy.

Nếu bạn muốn lệnh `schedule:run` sẽ chạy cho một site trong Homestead, bạn có thể thiết lập option `schedule` với giá trị `true` khi thiết lập thông tin site:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          schedule: true

Cron job cho site sẽ được thiết lập trong thư mục `/etc/cron.d` trong máy ảo.

<a name="configuring-mailhog"></a>
### Cấu hình Mailhog

Mailhog cho phép bạn dễ dàng gửi email đi và thực hiện việc kiểm tra quá trình gửi mail đó mà không cần phải thực sự gửi mail đến người dùng. Để bắt đầu, hãy cập nhật file `.env` của bạn sử dụng các cài đặt như sau:

    MAIL_MAILER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

Khi Mailhog đã được cấu hình xong, bạn có thể truy cập vào bảng điều khiển của Mailhog tại `http://localhost:8025`.

<a name="configuring-minio"></a>
### Cấu hình Minio

Minio là một server lưu trữ đối tượng mã nguồn mở có API tương thích với Amazon S3. Để cài đặt Minio, hãy cập nhật file `Homestead.yaml` của bạn với tùy chọn cấu hình sau trong phần [features](#installing-optional-features):

    minio: true

Mặc định, Minio có sẵn trên cổng 9600. Bạn có thể truy cập vào bảng điều khiển của Minio bằng cách truy cập vào `http://localhost:9600/`. Khóa truy cập mặc định là `homestead`, trong khi khóa bí mật mặc định là `secretkey`. Khi truy cập vào Minio, bạn nên sử dụng region `us-east-1`.

Để sử dụng Minio, bạn sẽ cần điều chỉnh cấu hình S3 disk trong file cấu hình `config/filesystems.php` của bạn. Bạn sẽ cần thêm tùy chọn `use_path_style_endpoint` vào cấu hình disk, cũng như thay đổi `url` thành `endpoint`:

    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'endpoint' => env('AWS_URL'),
        'use_path_style_endpoint' => true,
    ]

Cuối cùng, hãy đảm bảo file `.env` của bạn đã có các tùy chọn sau:

    AWS_ACCESS_KEY_ID=homestead
    AWS_SECRET_ACCESS_KEY=secretkey
    AWS_DEFAULT_REGION=us-east-1
    AWS_URL=http://localhost:9600

Để cung cấp các bucket, hãy thêm tuỳ chọn `buckets` vào trong file cấu hình Homestead của bạn:

    buckets:
        - name: your-bucket
          policy: public
        - name: your-private-bucket
          policy: none

Các giá trị `policy` được hỗ trợ là: `none`, `download`, `upload`, và `public`.

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

Để giải quyết vấn đề này, Homestead chứa một lệnh `share` của riêng nó. Để bắt đầu, hãy SSH vào máy ảo Homestead của bạn thông qua `vagrant ssh` và chạy `share homestead.test`. Điều này sẽ chia sẻ trang web `homestead.test` từ file cấu hình` Homestead.yaml` của bạn. Bạn có thể thay thế `homestead.test` thành bất kỳ trang web nào mà bạn muốn:

    share homestead.test

Sau khi chạy lệnh, bạn sẽ thấy một màn hình Ngrok xuất hiện chứa nhật ký hoạt động của URL và có thể truy cập được từ internet cho trang web vừa được chia sẻ. Nếu bạn muốn chỉ định một region hoặc một subdomain hoặc một tùy chọn thời gian chạy Ngrok, bạn có thể thêm chúng vào lệnh `share` của bạn:

    share homestead.test -region=eu -subdomain=laravel

> {note} Hãy nhớ rằng, Vagrant vốn không an toàn và bạn đang công khai máy ảo của bạn với Internet khi bạn chạy lệnh `share`.

<a name="multiple-php-versions"></a>
### Chạy nhiều phiên bản PHP

Homestead 6 đã giới thiệu chức năng hỗ trợ cho nhiều phiên bản PHP trên cùng một máy ảo. Bạn có thể chỉ định phiên bản PHP nào sẽ sử dụng cho một trang web nhất định trong file `Homestead.yaml` của bạn. Các phiên bản PHP có sẵn là: "5.6", "7.0", "7.1", "7.2", "7.3", và "7.4" (mặc định):

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          php: "7.1"

Thêm vào đó, bạn có thể sử dụng bất kỳ phiên bản PHP nào được hỗ trợ thông qua CLI:

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list
    php7.3 artisan list
    php7.4 artisan list

Bạn cũng có thể cập nhật phiên bản CLI mặc định bằng cách chạy các lệnh sau từ bên trong máy ảo Homestead của bạn:

    php56
    php70
    php71
    php72
    php73
    php74

<a name="web-servers"></a>
### Server Web

Mặc đinh, Homestead sẽ dùng server web là Nginx, Tuy nhiên, nó có thể cài đặt thêm cả Apache, nếu `apache` được thiết lập cho một site nào đó. Mặc dù cả hai web server này đều có thể cài đặt cùng nhau, nhưng chúng ta sẽ không thể chạy cả hai cùng một lúc. Nên lệnh shell `flip` sẽ giảm bớt quá trình chuyển đổi giữa các server web. Lệnh `flip` sẽ tự động xác định web server nào đang chạy, tắt nó đi và sau đó khởi động web server còn lại. Để sử dụng lệnh này, SSH vào máy ảo Homestead của bạn và chạy lệnh đó trong terminal của bạn:

    flip

<a name="mail"></a>
### Mail

Mặc định, homestead có chứa hộp thư Postfix đang lắng nghe trên cổng `1025`. Vì vậy, bạn có thể hướng dẫn ứng dụng của bạn sử dụng `smtp` mail driver trên `localhost` với cổng `1025`. Sau đó, tất cả các mail đã gửi sẽ được xử lý bởi Postfix và được Mailhog bắt lấy. Để xem các email đã gửi của bạn, hãy mở [http://localhost:8025](http://localhost:8025) trên trình duyệt web của bạn.

<a name="debugging-and-profiling"></a>
## Debugging và Profiling

<a name="debugging-web-requests"></a>
### Debug request với Xdebug

Homestead có hỗ trợ debug bằng [Xdebug](https://xdebug.org). Ví dụ: bạn có thể load một trang web trên trình duyệt và PHP sẽ kết nối đến IDE của bạn để cho phép kiểm tra và sửa lỗi code đang chạy.

Mặc định, Xdebug đã được chạy và sẵn sàng cho viêc kết nối. Nếu bạn cần kích hoạt Xdebug trên CLI, hãy chạy lệnh `sudo phpenmod xdebug` trong Vagrant box của bạn. Tiếp theo, làm theo hướng dẫn của IDE để kích hoạt debug. Cuối cùng, cấu hình trình duyệt mà bạn muốn chạy trang web để kích hoạt Xdebug, bạn có thể kích hoạt Xdebug bằng một extension hoặc [bookmarklet](https://www.jetbrains.com/phpstorm/marklets/).

> {note} Xdebug sẽ khiến PHP chạy chậm hơn đáng kể. Để tắt Xdebug, hãy chạy `sudo phpdismod xdebug` trong Vagrant box của bạn và khởi động lại FPM service.

<a name="debugging-cli-applications"></a>
### Debug CLI Application

Để debug một ứng dụng PHP CLI, hãy sử dụng alias shell `xphp` trong Vagrant box của bạn:

    xphp path/to/script

#### Autostarting Xdebug

Khi debug các bài test chức năng thực hiện request đến web server, việc tự động debug sẽ dễ dàng hơn việc sửa các bài test để truyền qua một header hoặc một cookie tuỳ biến để kích hoạt debug. Để yêu cầu Xdebug tự khởi động, hãy sửa `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` bên trong Vagrant box của bạn và thêm cấu hình sau:

    ; If Homestead.yaml contains a different subnet for the IP address, this address may be different...
    xdebug.remote_host = 192.168.10.1
    xdebug.remote_autostart = 1

<a name="profiling-applications-with-blackfire"></a>
### Profiling Applications với Blackfire

[Blackfire](https://blackfire.io/docs/introduction) là một dịch vụ SaaS để lập hồ sơ các web request và ứng dụng CLI và viết các kiểm tra về hiệu suất. Nó cung cấp một giao diện người dùng tương tác hiển thị dữ liệu trong các biểu đồ đường đi và dòng thời gian. Nó được xây dựng để sử dụng trong quá trình phát triển, dàn dựng và production mà không đòi hỏi chi phí cao cho người dùng cuối. Nó cung cấp các kiểm tra hiệu suất, chất lượng và bảo mật trên code và cài đặt cấu hình `php.ini`.

[Blackfire Player](https://blackfire.io/docs/player/index) là một ứng dụng mã nguồn mở có thể dùng cho các việc Web Crawling, Web Testing và Web Scraping, nó có thể hoạt động chung với Blackfire để viết các script cho các kịch bản.

Để bật Blackfire, hãy sử dụng cài đặt "features" trong file cấu hình Homestead của bạn:

    features:
        - blackfire:
            server_id: "server_id"
            server_token: "server_value"
            client_id: "client_id"
            client_token: "client_value"

Thông tin xác thực server và thông tin xác thực client của Blackfire [yêu cầu một tài khoản người dùng](https://blackfire.io/signup). Blackfire cung cấp các tùy chọn khác nhau để lập hồ sơ cho ứng dụng, bao gồm một công cụ CLI và một extension trình duyệt. Vui lòng [xem tài liệu Blackfire để biết thêm chi tiết](https://blackfire.io/docs/cookbooks/index).

### Profiling PHP Performance Using XHGui

[XHGui](https://www.github.com/perftools/xhgui) là một giao diện người dùng để khám phá hiệu suất các ứng dụng PHP của bạn. Để bật XHGui, hãy thêm `xhgui: 'true'` vào cấu hình trang web của bạn:

    sites:
        -
            map: your-site.test
            to: /home/vagrant/your-site/public
            type: "apache"
            xhgui: 'true'

Nếu trang web đã tồn tại, hãy chắc chắn là bạn đã chạy `vagrant provision` sau khi cập nhật cấu hình của bạn.

Để lập hồ sơ một web request, hãy thêm `xhgui=on` làm tham số truy vấn cho một request. XHGui sẽ tự động gắn cookie vào response để các request tiếp theo không cần đến giá trị chuỗi truy vấn nữa. Bạn có thể xem kết quả hồ sơ ứng dụng của bạn bằng mở trang tại `http://your-site.test/xhgui`.

Để lập hồ sơ cho một CLI request bằng XHGui, hãy set tiền tố command bằng `XHGUI=on`:

    XHGUI=on path/to/script

Kết quả hồ sơ CLI có thể được xem giống như kết quả hồ sơ trên web.

Lưu ý rằng hành động lập hồ sơ sẽ làm chậm quá trình thực thi script và thời gian chờ có thể nhiều gấp đôi so với request trong thực tế. Do đó, hãy luôn so sánh các nâng cấp theo tỷ lệ phần trăm chứ không phải một con số tuyệt đối. Ngoài ra, hãy lưu ý rằng thời gian thực thi sẽ bao gồm bất kỳ thời gian nào bị tạm dừng trong trình debug.

Vì hồ sơ hiệu suất sẽ chiếm dung lượng lớn disk, nên chúng sẽ bị xóa sau một vài ngày.

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

<a name="extending-homestead"></a>
## Mở rộng Homestead

Bạn có thể mở rộng Homestead bằng cách sử dụng script `after.sh` trong thư mục gốc của thư mục Homestead. Trong file này, bạn có thể thêm bất kỳ lệnh shell nào mà bạn muốn để cấu hình và tùy chỉnh máy ảo của bạn.

Khi đang tùy chỉnh Homestead, Ubuntu có thể hỏi bạn muốn giữ cấu hình ban đầu của package hay ghi đè cấu hình đó bằng file cấu hình mới. Để tránh điều này, bạn nên sử dụng lệnh sau khi cài đặt các package để tránh ghi đè lên bất kỳ cấu hình nào đã được Homestead viết trước đó:

    sudo apt-get -y \
        -o Dpkg::Options::="--force-confdef" \
        -o Dpkg::Options::="--force-confold" \
        install your-package

### User Customizations

Khi sử dụng Homestead trong một team setting, bạn có thể muốn điều chỉnh Homestead để phù hợp với phong cách phát triển của từng cá nhân. Bạn có thể tạo file `user-customizations.sh` trong thư mục gốc của thư mục Homestead (cùng thư mục chứa `Homestead.yaml` của bạn). Trong file này, bạn có thể thực hiện bất kỳ tùy chỉnh nào mà bạn muốn; tuy nhiên, không nên version controll file `user-customizations.sh` này.

<a name="updating-homestead"></a>
## Cập nhật Homestead

Trước khi bạn bắt đầu cập nhật Homestead, hãy đảm bảo là bạn đã xóa máy ảo hiện tại của bạn bằng cách chạy lệnh sau trong thư mục Homestead của bạn:

    vagrant destroy

Tiếp theo, bạn cần cập nhật mã nguồn Homestead. Nếu bạn clone từ repository Homestead, bạn có thể chạy những lệnh dưới đây từ thư mục mà bạn đã clone để cập nhật.

    git fetch

    git pull origin release

Các lệnh pull code này sẽ lấy source code Homestead mới nhất từ GitHub repository, nó sẽ tìm các tag mới nhất và sau đó kiểm tra phiên bản release mà được gắn với tag mới nhất. Bạn có thể tìm thấy phiên bản release mới nhất trên [trang release GitHub](https://github.com/laravel/homestead/releases).

Nếu bạn cài đặt Homestead thông qua file `composer.json` trong project của bạn, bạn nên sửa version Homestead thành `"laravel/homestead": "^11"` trong file đó, rồi sau đó bạn chỉ cần cập nhật thư viện thông qua composer:

    composer update

Sau đó, bạn nên cập nhật Vagrant box bằng lệnh `vagrant box update`:

    vagrant box update

Tiếp theo, bạn nên chạy lệnh `bash init.sh` từ thư mục Homestead để cập nhật thêm một số file cấu hình. Bạn sẽ có thể bị hỏi là có muốn ghi đè lên các file `Homestead.yaml`, `after.sh` và `aliases` hiện có của bạn hay không:

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

Cuối cùng, bạn sẽ cần tạo lại Homestead box của bạn để sử dụng cài đặt Vagrant mới:

    vagrant up

<a name="provider-specific-settings"></a>
## Cài đặt Provider cụ thể

<a name="provider-specific-virtualbox"></a>
### VirtualBox

#### `natdnshostresolver`

Mặc định, Homestead cấu hình `natdnshostresolver` là `on`. Điều này cho phép Homestead sử dụng DNS của hệ điều hành server. Nếu bạn muốn ghi đè hành vi này, hãy thêm các dòng sau vào file `Homestead.yaml` của bạn:

    provider: virtualbox
    natdnshostresolver: 'off'

#### Link ảo trên Windows

Nếu như các link ảo không hoạt động đúng trên máy Windows của bạn, thì bạn có thể cần thêm lệnh sau vào `Vagrantfile`:

    config.vm.provider "virtualbox" do |v|
        v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end
