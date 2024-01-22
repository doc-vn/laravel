# Laravel Homestead

- [Giới thiệu](#introduction)
- [Cài đặt](#installation-and-setup)
    - [Bước đầu tiên](#first-steps)
    - [Cấu hình Homestead](#configuring-homestead)
    - [Cấu hình Nginx Site](#configuring-nginx-sites)
    - [Cấu hình Services](#configuring-services)
    - [Chạy The Vagrant Box](#launching-the-vagrant-box)
    - [Per Project Installation](#per-project-installation)
    - [Cài đặt các chức năng tuỳ chọn](#installing-optional-features)
    - [Lối tắt](#aliases)
- [Cập nhật Homestead](#updating-homestead)
- [Daily Usage](#daily-usage)
    - [Kết nối thông qua SSH](#connecting-via-ssh)
    - [Adding Additional Sites](#adding-additional-sites)
    - [Biến environment](#environment-variables)
    - [Ports](#ports)
    - [Phiên bản PHP](#php-versions)
    - [Kết nối đến database](#connecting-to-databases)
    - [Sao lưu database](#database-backups)
    - [Cấu hình Cron Schedules](#configuring-cron-schedules)
    - [Cấu hình MailHog](#configuring-mailhog)
    - [Cấu hình Minio](#configuring-minio)
    - [Laravel Dusk](#laravel-dusk)
    - [Chia sẻ biến environment của bạn](#sharing-your-environment)
- [Debugging và Profiling](#debugging-and-profiling)
    - [Debug request với Xdebug](#debugging-web-requests)
    - [Debug CLI Application](#debugging-cli-applications)
    - [Profiling Applications với Blackfire](#profiling-applications-with-blackfire)
- [Network Interfaces](#network-interfaces)
- [Mở rộng Homestead](#extending-homestead)
- [Provider Specific Settings](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Giới thiệu

Laravel cố gắng làm cho toàn bộ trải nghiệm phát triển PHP của bạn trở nên thú vị, bao gồm cả môi trường phát triển local của bạn. [Laravel Homestead](https://github.com/laravel/homestead) là một Vagrant box đóng gói sẵn, chính thức, cung cấp cho bạn một môi trường phát triển tuyệt vời mà không yêu cầu bạn phải cài đặt PHP, server web hoặc bất kỳ phần mềm server nào khác trên máy local của bạn.

[Vagrant](https://www.vagrantup.com) cung cấp một cách thức đơn giản, nhẹ nhàng để quản lý và cung cấp máy ảo. Vagrant box hoàn toàn sẵn sàng để dùng. Nếu xảy ra sự cố, bạn có thể xoá và tạo lại một box mới trong vài phút!

Homestead có thể chạy trên nhiều hệ điều hành Windows, macOS, hoặc Linux, và chứa Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node, và những software tuyệt vời khác để giúp bạn phát triển application của bạn.

> {note} Nếu bạn đang dùng Windows, bạn có thể cần bật hardware virtualization (VT-x). Nó có thể được bật thông qua BIOS của bạn. Nếu bạn đang dùng Hyper-V trên hệ thống UEFI, bạn có thể cần phải tắt Hyper-V để có thể truy cập vào VT-x.

<a name="included-software"></a>
### Software cài đặt sẵn

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">
- Ubuntu 20.04
- Git
- PHP 8.1
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL 8.0
- lmm
- Sqlite3
- PostgreSQL 13
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
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
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
- EventStoreDB
- Gearman
- Go
- Grafana
- InfluxDB
- MariaDB
- Meilisearch
- MinIO
- MongoDB
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- R
- RabbitMQ
- RVM (Ruby Version Manager)
- Solr
- TimescaleDB
- Trader <small>(PHP extension)</small>
- Webdriver & Laravel Dusk Utilities
</div>

<a name="installation-and-setup"></a>
## Cài đặt

<a name="first-steps"></a>
### Bước đầu tiên

Trước khi chạy môi trường Homestead của bạn, bạn cần phải cài đặt [Vagrant](https://www.vagrantup.com/downloads.html) hoặc một trong những nhà cung cấp được hỗ trợ sau:

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Downloads)
- [Parallels](https://www.parallels.com/products/desktop/)

Tất cả các package phần mềm này đều có cách cài đặt trực quan dễ sử dụng cho tất cả các hệ điều hành phổ biến.

Để dùng phần mềm Parallels, bạn sẽ cần cài đặt [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels). Nó miễn phí.

<a name="installing-homestead"></a>
#### Cài đặt Homestead

Bạn có thể cài đặt Homestead bằng cách clone repository của Homestead vào trong máy local của bạn. Sau đó lưu thư mục đã clone đó vào trong thư mục có tên là `Homestead` trong thư mục "home" của bạn, vì máy ảo Homestead sẽ đóng vai trò là server lưu trữ cho tất cả các application Laravel của bạn. Nên trong suốt tài liệu này, chúng ta sẽ gọi thư mục này là "thư mục Homestead" của bạn:

```bash
git clone https://github.com/laravel/homestead.git ~/Homestead
```

Sau khi clone repository Laravel Homestead, bạn nên checkout ra branch `release`. Branch này luôn chứa bản phát hành mới nhất của Homestead:

    cd ~/Homestead

    git checkout release

Tiếp theo, chạy lệnh `bash init.sh` từ trong thư mục Homestead để tạo file configuration `Homestead.yaml`, File `Homestead.yaml` này là nơi bạn sẽ chứa cấu hình tất cả các cài đặt cho Homestead của bạn. File này sẽ được lưu trong thư mục Homestead:

    // macOS / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### Cấu hình Homestead

<a name="setting-your-provider"></a>
#### Cài đặt Provider

Từ khoá `provider` trong file `Homestead.yaml` của bạn sẽ chỉ ra loại Vagrant nào sẽ được sủ dụng: `virtualbox` hoặc `parallels`:

    provider: virtualbox

> {note} Nếu bạn đang sử dụng Apple Silicon, bạn nên thêm `box: laravel/homestead-arm` vào file `Homestead.yaml` của bạn. Apple Silicon yêu cầu nhà cung cấp Parallels.

<a name="configuring-shared-folders"></a>
#### Cài đặt thư mục chia sẻ

Thuộc tính `folders` trong file `Homestead.yaml` sẽ liệt kê tất cả các thư mục mà bạn muốn chia sẻ với môi trường Homestead của bạn. Khi các file trong các thư mục này thay đổi, chúng sẽ được giữ đồng bộ giữa local của bạn và môi trường phát triển Homestead. Bạn có thể cấu hình nhiều thư mục dùng chung nếu cần:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> {note} Người dùng Windows không nên sử dụng cú pháp đường dẫn `~/` và thay vào đó nên sử dụng đường dẫn đầy đủ đến project của họ, chẳng hạn như `C:\Users\user\Code\project1`.

Bạn nên map các project nhỏ thành các thư mục riêng của chúng thay vì map toàn bộ thư mục `~/code` của bạn. Khi bạn map một thư mục, máy ảo phải theo dõi tất cả disk IO  cho *mọi file* có trong thư mục đó. Điều đó sẽ dẫn đến các vấn đề về hiệu suất nếu bạn có một lượng lớn các file trong một thư mục.

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> {note} Bạn không nên mount `.` (thư mục hiện tại) khi sử dụng Homestead. Điều này khiến Vagrant không map được thư mục hiện tại thành `/vagrant` và sẽ phá vỡ các tính năng tùy chọn và gây ra kết quả không mong muốn khi cấp phép.

Để bật [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), bạn có thể thêm tùy chọn `type` vào thư mục mapping của bạn:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "nfs"

> {note} Khi dùng NFS trên Windows, bạn nên cân nhắc cài đặt plug-in [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd). Plug-in này duy trì chính xác các quyền user và group cho các file và các thư mục trong máy ảo Homestead.

Bạn cũng có thể thêm vào các option khác được hỗ trợ bởi Vagrant [Synced Folders](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) bằng cách liệt kê chúng dưới từ khoá `options`:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

<a name="configuring-nginx-sites"></a>
### Cấu hình Nginx Site

Nếu bạn chưa biết Nginx? Không sao cả. Thuộc tính `sites` của file `Homestead.yaml` của bạn cho phép bạn dễ dàng map một "domain" tới một thư mục trên môi trường phát triển Homestead. Một cấu hình site mẫu đã được chứa trong file `Homestead.yaml`. Một lần nữa, bạn có thể thêm nhiều site vào môi trường phát triển Homestead của bạn nếu cần. Homestead có thể phục vụ như một môi trường ảo hóa, thuận tiện cho mọi application Laravel mà bạn đang làm việc:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public

Nếu bạn muốn thay đổi thuộc tính `sites` khi máy ảo Homestead đã được cài đặt, bạn nên chạy lệnh `vagrant reload --provision` trong terminal của bạn để cập nhật cấu hình Nginx trên máy ảo.

> {note} Các script Homestead được xây dựng sao cho phù hợp nhất có thể. Tuy nhiên, nếu bạn đang gặp sự cố trong khi cấp phép, bạn nên xoá và build lại một máy ảo bằng cách chạy lệnh `vagrant destroy && vagrant up`.

<a name="hostname-resolution"></a>
#### Hostname Resolution

Homestead sẽ publish hostname dùng `mDNS` để tự động phân giải host. Nếu bạn set `hostname: homestead` trong file `Homestead.yaml` của bạn, thì host của bạn sẽ ở tại địa chỉ `homestead.local`. Mặc định, m,macOS, iOS, và các bản phân phối máy tính để bàn Linux sẽ hỗ trợ `mDNS`. Nếu bạn đang sử dụng Windows, bạn phải cài đặt [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US).

Sử dụng hostname tự động sẽ hoạt động tốt nhất cho [cài đặt từng dự án](#per-project-installation) của Homestead. Nếu bạn lưu nhiều trang web trên cùng một phiên bản Homestead, bạn có thể thêm "tên miền" cho các trang web Nginx của bạn vào file `hosts` trên máy của bạn. File `hosts` sẽ chuyển hướng các request đến các trang Homestead của bạn vào máy ảo Homestead của bạn. Trên macOS và Linux, file này được lưu tại `/etc/hosts`. Trên Windows, file đó được lưu tại `C:\Windows\System32\drivers\etc\hosts`. Các dòng mà bạn cần thêm vào trong file này sẽ trông như sau:

    192.168.56.56  homestead.test

Hãy chắc chắn rằng địa chỉ IP được liệt kê là địa chỉ được cài đặt trong file `Homestead.yaml` của bạn. Khi bạn đã thêm tên miền vào file `hosts` của bạn và khởi chạy Vagrant, bạn có thể truy cập trang web thông qua trình duyệt web của bạn:

```bash
http://homestead.test
```

<a name="configuring-services"></a>
### Cấu hình Services

Homestead mặc định chạy một số service; tuy nhiên, bạn có thể tùy chỉnh các service nào sẽ được bật hoặc tắt trong quá trình provisioning. Ví dụ: bạn có thể bật PostgreSQL và tắt MySQL bằng cách sửa đổi tùy chọn `services` trong file `Homestead.yaml` của bạn:

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

The specified services will be started or stopped based on their order in the `enabled` and `disabled` directives.

<a name="launching-the-vagrant-box"></a>
### Chạy Vagrant

Khi bạn đã chỉnh sửa xong file `Homestead.yaml` theo cách bạn muốn, hãy chạy lệnh `vagrant up` trong thư mục Homestead của bạn. Vagrant sẽ khởi động máy ảo và tự động cấu hình các thư mục và trang web Nginx cho bạn.

Để xoá máy ảo, bạn có dùng lệnh `vagrant destroy`.

<a name="per-project-installation"></a>
### Cài đặt cho mỗi Project

Thay vì cài đặt Homestead globally và chia sẻ máy ảo Homestead đó cho tất cả các project của bạn, thì bạn có thể cấu hình một phiên bản Homestead dành riêng cho mỗi project mà bạn quản lý. Cài đặt Homestead cho mỗi project có thể có một lợi thế là nếu bạn gửi một file `Vagrantfile` cùng với code trong dự án của bạn, thì những người khác có thể làm việc ngay với project của bạn với chỉ bằng một câu lệnh đơn giản là `vagrant up` sau khi clone repository của dự án về.

Để cài đặt Homestead vào project của bạn, thì bạn phải cài nó thông qua Composer:

```bash
composer require laravel/homestead --dev
```

Khi mà Homestead đã cài đặt xong, gọi lệnh `make` của Homestead để tạo ra 2 file `Vagrantfile` và `Homestead.yaml` cho project của bạn. Những file này sẽ được lưu trong thư mục gốc của dự án của bạn. Lệnh `make` sẽ tự động cấu hình các thuộc tính `sites` và `folders` vào trong file `Homestead.yaml`.

    // macOS / Linux...
    php vendor/bin/homestead make

    // Windows...
    vendor\\bin\\homestead make

Tiếp theo, chạy lệnh `vagrant up` trong terminal để có thể truy cập vào project của bạn trên địa chỉ `http://homestead.test` trong web browser. Hãy nhớ rằng, bạn cần phải thêm domain `homestead.test` vào file `/etc/hosts` hoặc tên domain mà bạn thích nếu bạn không muốn sử dụng [hostname resolution](#hostname-resolution)

<a name="installing-optional-features"></a>
### Cài đặt các chức năng tuỳ chọn

Các phần mềm tùy chọn sẽ được cài đặt bằng cách sử dụng tuỳ chọn `features` trong file `Homestead.yaml` của bạn. Hầu hết các chức năng đều có thể được bật hoặc tắt bằng giá trị boolean, trong khi đó có một số chức năng cho phép nhiều tùy chọn cấu hình:

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
        - eventstore: true
            version: 21.2.0
        - gearman: true
        - golang: true
        - grafana: true
        - influxdb: true
        - mariadb: true
        - meilisearch: true
        - minio: true
        - mongodb: true
        - neo4j: true
        - ohmyzsh: true
        - openresty: true
        - pm2: true
        - python: true
        - r-base: true
        - rabbitmq: true
        - rvm: true
        - solr: true
        - timescaledb: true
        - trader: true
        - webdriver: true

<a name="elasticsearch"></a>
#### Elasticsearch

Bạn có thể chỉ định phiên bản được hỗ trợ cho Elasticsearch, nó phải là phiên bản chính xác (major.minor.patch). Cài đặt mặc định sẽ tạo ra một cụm có tên là 'homestead'. Bạn đừng nên cho Elasticsearch nhiều hơn một nửa bộ nhớ của máy ảo, vì vậy hãy đảm bảo rằng máy ảo Homestead của bạn có ít nhất gấp đôi lượng được phân bổ cho Elasticsearch.

> {tip} Hãy xem [tài liệu về Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current) để tìm hiểu cách tùy chỉnh cấu hình của bạn.

<a name="mariadb"></a>
#### MariaDB

Hãy bật MariaDB rồi xóa MySQL và cài đặt MariaDB. MariaDB server thường sẽ đóng vai trò thay thế cho MySQL, vì vậy bạn vẫn nên sử dụng driver cơ sở dữ liệu là `mysql` trong cấu hình cơ sở dữ liệu trong ứng dụng của bạn.

<a name="mongodb"></a>
#### MongoDB

Mặc định, cài đặt của MongoDB sẽ lưu tên người dùng cơ sở dữ liệu là `homestead` và mật khẩu là `secret`.

<a name="neo4j"></a>
#### Neo4j

Mặc định, cài đặt của Neo4j sẽ lưu tên người dùng trong cơ sở dữ liệu là `homestead` và mật khẩu là `secret`. Để truy cập vào Neo4j, hãy truy cập vào `http://homestead.test:7474` trên trình duyệt web của bạn. Các cổng `7687` (Bolt), `7474` (HTTP), và `7473` (HTTPS) đã cài đặt sẵn để phục vụ các request từ Neo4j client.

<a name="aliases"></a>
### Aliases (Tên viết tắt)

Bạn có thể thêm các alias của Bash vào trong máy ảo Homestead của bạn bằng cách sửa file `aliases` trong thư mục Homestead của bạn:

    alias c='clear'
    alias ..='cd ..'

Sau khi bạn đã update file `aliases`, bạn nên chạy lại máy ảo Homestead bằng lệnh `vagrant reload --provision`. Điều này sẽ đảm bảo rằng các alias mới của bạn sẽ được cài đặt vào máy ảo.

<a name="updating-homestead"></a>
## Cập nhật Homestead

Trước khi bạn bắt đầu cập nhật Homestead, hãy đảm bảo là bạn đã xóa máy ảo hiện tại của bạn bằng cách chạy lệnh sau trong thư mục Homestead của bạn:

    vagrant destroy

Tiếp theo, bạn cần cập nhật mã nguồn của Homestead. Nếu bạn clone từ repository Homestead, bạn có thể chạy những lệnh dưới đây từ thư mục mà bạn đã clone để cập nhật.

    git fetch

    git pull origin release

Các lệnh pull code này sẽ lấy source code Homestead mới nhất từ GitHub repository, nó sẽ tìm các tag mới nhất và sau đó kiểm tra phiên bản release mà được gắn với tag mới nhất. Bạn có thể tìm thấy phiên bản release mới nhất trên [trang release GitHub](https://github.com/laravel/homestead/releases).

Nếu bạn cài đặt Homestead thông qua file `composer.json` trong project của bạn, bạn nên sửa version Homestead thành `"laravel/homestead": "^12"` trong file đó, rồi sau đó bạn chỉ cần cập nhật thư viện thông qua composer:

    composer update

Sau đó, bạn nên cập nhật Vagrant box bằng lệnh `vagrant box update`:

    vagrant box update

Tiếp theo, bạn nên chạy lệnh `bash init.sh` từ thư mục Homestead để cập nhật thêm một số file cấu hình. Bạn sẽ có thể bị hỏi là có muốn ghi đè lên các file `Homestead.yaml`, `after.sh` và `aliases` hiện có của bạn hay không:

    // macOS / Linux...
    bash init.sh

    // Windows...
    init.bat

Cuối cùng, bạn sẽ cần tạo lại Homestead box của bạn để sử dụng cài đặt Vagrant mới:

    vagrant up

<a name="daily-usage"></a>
## Daily Usage

<a name="connecting-via-ssh"></a>
### Connecting Via SSH

Bạn có thể SSH vào máy ảo của bạn bằng cách chạy lệnh terminal `vagrant ssh` từ thư mục Homestead của bạn.

<a name="adding-additional-sites"></a>
### Thêm một site mới

Khi môi trường Homestead của bạn đã được cài đặt và chạy xong, bạn có thể muốn thêm một site mới cho  những dự án Laravel khác của bạn. Bạn có thể chạy nhiều dự án Laravel tùy thích trên một môi trường Homestead duy nhất. Để thêm những thông tin site mới này, hăy thêm site vào file `Homestead.yaml` của bạn:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
        - map: another.test
          to: /home/vagrant/project2/public

> {note} Bạn nên đảm bảo là bạn đã cấu hình [folder mapping](#configuring-shared-folders) cho thư mục của dự án trước khi thêm site.

Nếu Vagrant không tự động quản lý file "hosts" cho bạn, thì bạn có thể cần thêm thông tin site mới vào file host như sau. Trên Mac và Linux, file này được lưu tại `/etc/hosts`. Trên Windows, file đó được lưu tại `C:\Windows\System32\drivers\etc\hosts`.

    192.168.56.56  homestead.test
    192.168.56.56  another.test

Khi mà một site mới đã được thêm vào, hãy chạy lại lệnh terminal `vagrant reload --provision` từ thư mục Homestead của bạn.

<a name="site-types"></a>
#### Loại Site

Homestead hỗ trợ nhiều "loại" site khác nhau, cho phép bạn dễ dàng chạy các project khác nhau mà không cần phải xây dựng từ Laravel. Ví dụ, chúng ta có thể dễ dàng thêm một Statamic application vào Homestead chỉ cần dùng loại site là `statamic`:

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```

Các loại site được hỗ trợ là: `apache`, `laravel` (mặc định), `proxy`, `silverstripe`, `statamic`, `symfony2`, và `symfony4`.

<a name="site-parameters"></a>
#### Site Parameters

Bạn có thể thêm các giá trị Nginx `fastcgi_param` vào site của bạn thông qua `params` trong site:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          params:
              - key: FOO
                value: BAR

<a name="environment-variables"></a>
### Biến môi trường

Bạn có thể định nghĩa biến môi trường global bằng cách thêm chúng vào file `Homestead.yaml` của bạn:

    variables:
        - key: APP_ENV
          value: local
        - key: FOO
          value: bar

Sau khi đã cập nhật xong file `Homestead.yaml`, hãy tải lại provision cho máy ảo bằng cách chạy lệnh `vagrant reload --provision`. Lệnh này sẽ cập nhật cấu hình PHP-FPM cho tất cả các phiên bản PHP đã cài đặt và cũng cập nhật môi trường cho user `vagrant`.

<a name="ports"></a>
### Ports

Mặc định, các cổng dưới đây sẽ được thiết lập để chuyển tiếp tới môi trường Homestead của bạn:

<div class="content-list" markdown="1">

- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443

</div>

<a name="forwarding-additional-ports"></a>
#### Forwarding Additional Ports

Nếu muốn, bạn có thể chuyển tiếp các cổng bổ sung tới Vagrant box bằng cách định nghĩa một cấu hình `ports` trong file `Homestead.yaml` của bạn. Sau khi cập nhật file `Homestead.yaml` này, hãy đảm bảo là bạn đã provision lại bằng cách thực hiện lệnh `vagrant reload --provision`:

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

Dưới đây là danh sách các cổng service Homestead mà bạn có thể muốn map từ host của bạn tới Vagrant box:

<div class="content-list" markdown="1">

- **SSH:** 2222 &rarr; To 22
- **ngrok UI:** 4040 &rarr; To 4040
- **MySQL:** 33060 &rarr; To 3306
- **PostgreSQL:** 54320 &rarr; To 5432
- **MongoDB:** 27017 &rarr; To 27017
- **Mailhog:** 8025 &rarr; To 8025
- **Minio:** 9600 &rarr; To 9600

</div>

<a name="php-versions"></a>
### Phiên bản PHP

Homestead 6 đã giới thiệu chức năng hỗ trợ cho nhiều phiên bản PHP trên cùng một máy ảo. Bạn có thể chỉ định phiên bản PHP nào sẽ sử dụng cho một trang web nhất định trong file `Homestead.yaml` của bạn. Các phiên bản PHP có sẵn là: "5.6", "7.0", "7.1", "7.2", "7.3", "7.4", "8.0" (mặc định), và "8.1":

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          php: "7.1"

[Trong máy ảo Homestead của bạn](#connecting-via-ssh), bạn có thể sử dụng bất kỳ phiên bản PHP nào được hỗ trợ thông qua CLI:

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list
    php7.3 artisan list
    php7.4 artisan list
    php8.0 artisan list
    php8.1 artisan list

Bạn cũng có thể cập nhật phiên bản mặc định của PHP mà CLI sử dụng bằng cách chạy các lệnh sau từ bên trong máy ảo Homestead của bạn:

    php56
    php70
    php71
    php72
    php73
    php74
    php80
    php81

<a name="connecting-to-databases"></a>
### Kết nối đến database

Một database `homestead` sẽ được cấu hình cho cả MySQL và PostgreSQL. Để kết nối đến database MySQL hoặc PostgreSQL từ máy thật của bạn, bạn cần kết nối tới địa chỉ `127.0.0.1` và cổng là `33060` (MySQL) hoặc `54320` (PostgreSQL). Với username và password sẽ là `homestead` và `secret`.

> {note} Bạn chỉ nên sử dụng các cổng không mặc định này khi kết nối với cơ sở dữ liệu từ máy thật của bạn. Bạn sẽ dùng các cổng mặc định 3306 và 5432 trong file cấu hình `database` trong application của bạn vì Laravel đang chạy _trong_ máy ảo chứ không phải máy thật của bạn.

<a name="database-backups"></a>
### Sao lưu database

Homestead có thể tự động backup cơ sở dữ liệu của bạn khi Vagrant box của bạn bị destroy. Để sử dụng tính năng này, bạn sẽ phải sử dụng Vagrant 2.1.0 trở lên. Hoặc, nếu bạn đang sử dụng phiên bản Vagrant cũ hơn, bạn phải cài đặt plug-in `vagrant-triggers`. Để bật backup cơ sở dữ liệu tự động, hãy thêm dòng sau vào file `Homestead.yaml` của bạn:

    backup: true

Sau khi đã được cấu hình, Homestead sẽ export cơ sở dữ liệu của bạn sang các thư mục `mysql_backup` và `postgres_backup` khi lệnh `vagrant destroy` được thực thi. Bạn có thể tìm thấy những thư mục này trong thư mục mà bạn đã clone Homestead hoặc trong thư mục gốc của project nếu bạn đang sử dụng phương thức [cài đặt Homestead cho từng dự án](#per-project-installation).

<a name="configuring-cron-schedules"></a>
### Cấu hình schedule cho Cron

Laravel cung cấp một cách thuận tiện để [schedule Cron jobs](/docs/{{version}}/scheduling) bằng lệnh Artisan `schedule:run`, sẽ chạy mỗi phút. Lệnh `schedule:run` sẽ kiểm tra danh sách job đã được cài đặt trong class `App\Console\Kernel` của bạn để xác định xem task nào sẽ được chạy.

Nếu bạn muốn lệnh `schedule:run` sẽ chạy cho một site trong Homestead, bạn có thể thiết lập option `schedule` với giá trị `true` khi thiết lập thông tin site:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

Cron job cho site sẽ được thiết lập trong thư mục `/etc/cron.d` trong máy ảo Homestead.

<a name="configuring-mailhog"></a>
### Cấu hình MailHog

[MailHog](https://github.com/mailhog/MailHog) cho phép bạn chặn việc gửi email và thực hiện việc kiểm tra quá trình gửi mail đó mà không cần phải thực sự gửi mail đến người dùng. Để bắt đầu, hãy cập nhật file `.env` trong application của bạn sử dụng các cài đặt như sau:

    MAIL_MAILER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

Khi MailHog đã được cấu hình xong, bạn có thể truy cập vào bảng điều khiển của MailHog tại `http://localhost:8025`.

<a name="configuring-minio"></a>
### Cấu hình Minio

[Minio](https://github.com/minio/minio) là một server lưu trữ đối tượng mã nguồn mở có API tương thích với Amazon S3. Để cài đặt Minio, hãy cập nhật file `Homestead.yaml` của bạn với tùy chọn cấu hình sau trong phần [features](#installing-optional-features):

    minio: true

Mặc định, Minio có sẵn trên cổng 9600. Bạn có thể truy cập vào bảng điều khiển của Minio bằng cách truy cập vào `http://localhost:9600`. Khóa truy cập mặc định là `homestead`, trong khi khóa bí mật mặc định là `secretkey`. Khi truy cập vào Minio, bạn nên sử dụng region `us-east-1`.

Để sử dụng Minio, bạn sẽ cần điều chỉnh cấu hình S3 disk trong file cấu hình `config/filesystems.php` trong application của bạn. Bạn sẽ cần thêm tùy chọn `use_path_style_endpoint` vào cấu hình disk cũng như thay đổi `url` thành `endpoint`:

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

```bash
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
AWS_URL=http://localhost:9600
```

Để cung cấp các bucket "S3" được hỗ trợ bởi Minio, hãy thêm lệnh `buckets` vào file `Homestead.yaml` của bạn. Sau khi định nghĩa xong bucket của bạn, bạn nên chạy lại lệnh `vagrant reload --provision` trong terminal của bạn:

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

Các giá trị `policy` được hỗ trợ là: `none`, `download`, `upload`, và `public`.

<a name="laravel-dusk"></a>
### Laravel Dusk

Để chạy test [Laravel Dusk](/docs/{{version}}/dusk) trong Homestead, bạn nên bật [feature `webdriver`](#installing-optional-features) trong file cấu hình Homestead của bạn:

```yaml
features:
    - webdriver: true
```

Sau khi bật feature `webdriver`, bạn nên chạy lại lệnh `vagrant reload --provision` trong terminal của bạn.

<a name="sharing-your-environment"></a>
### Chia sẻ môi trường của bạn

Đôi khi bạn có thể muốn chia sẻ những gì bạn đang làm việc với đồng nghiệp hoặc khách hàng. Vagrant có một cách để hỗ trợ cho điều này thông qua lệnh `share vagrant`; tuy nhiên, điều này sẽ không hoạt động nếu bạn có nhiều trang đang được cấu hình trong file `Homestead.yaml`.

Để giải quyết vấn đề này, Homestead chứa một lệnh `share` của riêng nó. Để bắt đầu, [hãy SSH vào máy ảo Homestead của bạn](#connecting-via-ssh) thông qua `vagrant ssh` và chạy lệnh `share homestead.test`. Lệnh này sẽ chia sẻ trang web `homestead.test` từ file cấu hình` Homestead.yaml` của bạn. Bạn có thể thay thế `homestead.test` thành bất kỳ trang web nào mà bạn muốn:

    share homestead.test

Sau khi chạy lệnh, bạn sẽ thấy một màn hình Ngrok xuất hiện chứa nhật ký hoạt động của URL và có thể truy cập được từ internet cho trang web vừa được chia sẻ. Nếu bạn muốn chỉ định một region hoặc một subdomain hoặc một tùy chọn thời gian chạy Ngrok, bạn có thể thêm chúng vào lệnh `share` của bạn:

    share homestead.test -region=eu -subdomain=laravel

> {note} Hãy nhớ rằng, Vagrant vốn không an toàn và bạn đang công khai máy ảo của bạn với Internet khi bạn chạy lệnh `share`.

<a name="debugging-and-profiling"></a>
## Debugging và Profiling

<a name="debugging-web-requests"></a>
### Debug request với Xdebug

Homestead có hỗ trợ debug bằng [Xdebug](https://xdebug.org). Ví dụ: bạn có thể truy cập vào một trang web trên trình duyệt của bạn và PHP sẽ kết nối đến IDE của bạn để cho phép kiểm tra và sửa lỗi code đang chạy.

Mặc định, Xdebug đã được chạy và sẵn sàng cho viêc kết nối. Nếu bạn cần kích hoạt Xdebug trên CLI, hãy chạy lệnh `sudo phpenmod xdebug` trong máy ảo Homestead của bạn. Tiếp theo, làm theo hướng dẫn của IDE để kích hoạt debug. Cuối cùng, cấu hình trình duyệt mà bạn muốn chạy trang web để kích hoạt Xdebug, bạn có thể kích hoạt Xdebug bằng một extension hoặc [bookmarklet](https://www.jetbrains.com/phpstorm/marklets/).

> {note} Xdebug sẽ khiến PHP chạy chậm hơn đáng kể. Để tắt Xdebug, hãy chạy `sudo phpdismod xdebug` trong máy ảo Homestead của bạn và khởi động lại FPM service.

<a name="autostarting-xdebug"></a>
#### Autostarting Xdebug

Khi debug các bài test chức năng thực hiện request đến web server, việc tự động debug sẽ dễ dàng hơn việc sửa các bài test để truyền qua một header hoặc một cookie tuỳ biến để kích hoạt debug. Để yêu cầu Xdebug tự khởi động, hãy sửa file `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` bên trong máy ảo Homestead của bạn và thêm cấu hình sau:

```ini
; If Homestead.yaml contains a different subnet for the IP address, this address may be different...
xdebug.remote_host = 192.168.10.1
xdebug.remote_autostart = 1
```

<a name="debugging-cli-applications"></a>
### Debug CLI Application

Để debug một ứng dụng PHP CLI, hãy sử dụng alias shell `xphp` trong máy ảo Homestead của bạn:

    xphp /path/to/script

<a name="profiling-applications-with-blackfire"></a>
### Profiling Applications với Blackfire

[Blackfire](https://blackfire.io/docs/introduction) là một service để lập hồ sơ các web request và ứng dụng CLI. Nó cung cấp một giao diện người dùng tương tác hiển thị dữ liệu trong các biểu đồ đường đi và dòng thời gian. Nó được xây dựng để sử dụng trong quá trình phát triển, dàn dựng và production mà không đòi hỏi chi phí cao cho người dùng cuối. Ngoài ra, Blackfire cũng cung cấp các kiểm tra hiệu suất, chất lượng và bảo mật trên code và cài đặt cấu hình `php.ini`.

[Blackfire Player](https://blackfire.io/docs/player/index) là một ứng dụng mã nguồn mở có thể dùng cho các việc Web Crawling, Web Testing, và Web Scraping, nó có thể hoạt động chung với Blackfire để viết các script cho các kịch bản.

Để bật Blackfire, hãy sử dụng cài đặt "features" trong file cấu hình Homestead của bạn:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Thông tin xác thực server và thông tin xác thực client của Blackfire [yêu cầu một tài khoản Blackfire](https://blackfire.io/signup). Blackfire cung cấp các tùy chọn khác nhau để lập hồ sơ cho ứng dụng, bao gồm một công cụ CLI và một extension trình duyệt. Vui lòng [xem tài liệu Blackfire để biết thêm chi tiết](https://blackfire.io/docs/cookbooks/index).

<a name="network-interfaces"></a>
## Network Interfaces

Thuộc tính `networks` trong file `Homestead.yaml` sẽ cấu hình các network interface cho máy ảo Homestead của bạn. Bạn có thể cấu hình nhiều network interface nếu cần:

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

Để bật một [bridged](https://www.vagrantup.com/docs/networking/public_network.html) interface, hãy cấu hình một `bridge` cho network và đổi loại của network sang `public_network`:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

Để bật một [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), chỉ cần xoá tuỳ chọn `ip` từ file cấu hình của bạn:

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

<a name="extending-homestead"></a>
## Mở rộng Homestead

Bạn có thể mở rộng Homestead bằng cách sử dụng script `after.sh` trong thư mục gốc của thư mục Homestead. Trong file này, bạn có thể thêm bất kỳ lệnh shell nào mà bạn muốn để cấu hình và tùy chỉnh máy ảo của bạn.

Khi đang tùy chỉnh Homestead, Ubuntu có thể hỏi bạn muốn giữ lại cấu hình ban đầu của package hay ghi đè cấu hình đó bằng file cấu hình mới. Để tránh điều này, bạn nên sử dụng lệnh sau đây để khi cài đặt các package để tránh ghi đè lên bất kỳ cấu hình nào đã được Homestead viết trước đó:

    sudo apt-get -y \
        -o Dpkg::Options::="--force-confdef" \
        -o Dpkg::Options::="--force-confold" \
        install package-name

<a name="user-customizations"></a>
### User Customizations

Khi sử dụng Homestead trong một team của bạn, bạn có thể muốn điều chỉnh Homestead để phù hợp với phong cách phát triển của từng cá nhân. Để thực hiện được điều này, bạn có thể tạo file `user-customizations.sh` trong thư mục gốc của thư mục Homestead (cùng thư mục chứa file `Homestead.yaml` của bạn). Trong file này, bạn có thể thực hiện bất kỳ tùy chỉnh nào mà bạn muốn; tuy nhiên, không nên version controll file `user-customizations.sh` này.

<a name="provider-specific-settings"></a>
## Cài đặt Provider cụ thể

<a name="provider-specific-virtualbox"></a>
### VirtualBox

<a name="natdnshostresolver"></a>
#### `natdnshostresolver`

Mặc định, Homestead cấu hình `natdnshostresolver` là `on`. Điều này cho phép Homestead sử dụng DNS của hệ điều hành server. Nếu bạn muốn ghi đè hành vi này, hãy thêm các tuỳ chọn cấu hình sau vào file `Homestead.yaml` của bạn:

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```

<a name="symbolic-links-on-windows"></a>
#### Link ảo trên Windows

Nếu như các link ảo không hoạt động đúng trên máy Windows của bạn, thì bạn có thể cần thêm lệnh sau vào `Vagrantfile`:

```ruby
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```
