# Laravel Sail

- [Giới thiệu](#introduction)
- [Cài đặt và setup](#installation)
    - [Cài đặt Sail vào trong application hiện tại](#installing-sail-into-existing-applications)
    - [Cấu hình một shell alias](#configuring-a-shell-alias)
- [Starting và Stopping Sail](#starting-and-stopping-sail)
- [Chạy commands](#executing-sail-commands)
    - [Chạy PHP Commands](#executing-php-commands)
    - [Chạy Composer Commands](#executing-composer-commands)
    - [Chạy Artisan Commands](#executing-artisan-commands)
    - [Chạy Node và NPM Commands](#executing-node-npm-commands)
- [Tương tác với Databases](#interacting-with-sail-databases)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [Meilisearch](#meilisearch)
    - [Typesense](#typesense)
- [File Storage](#file-storage)
- [Running Tests](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [Previewing Emails](#previewing-emails)
- [Container CLI](#sail-container-cli)
- [PHP Versions](#sail-php-versions)
- [Node Versions](#sail-node-versions)
- [Sharing Your Site](#sharing-your-site)
- [Debugging With Xdebug](#debugging-with-xdebug)
  - [Xdebug CLI Usage](#xdebug-cli-usage)
  - [Xdebug Browser Usage](#xdebug-browser-usage)
- [Tuỳ chỉnh](#sail-customization)

<a name="introduction"></a>
## Giới thiệu

[Laravel Sail](https://github.com/laravel/sail) là một giao diện command-line nhẹ để tương tác với môi trường phát triển Docker của Laravel. Sail cung cấp điểm khởi đầu tuyệt vời để xây dựng ứng dụng Laravel bằng PHP, MySQL và Redis mà không yêu cầu bất cứ kinh nghiệm Docker nào.

Về cơ bản, Sail là file `docker-compose.yml` và script `sail` được lưu ở thư mục root của project của bạn. Script `sail` cung cấp một CLI với các phương thức thuận tiện để tương tác với các Docker container được định nghĩa bởi file `docker-compose.yml`.

Laravel Sail được hỗ trợ trên macOS, Linux và Windows (thông qua [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)).

<a name="installation"></a>
## Cài đặt và setup

Laravel Sail được cài đặt tự động cùng với tất cả các ứng dụng Laravel mới nên bạn có thể bắt đầu sử dụng nó ngay lập tức. Để tìm hiểu cách tạo ra một ứng dụng Laravel mới, vui lòng tham khảo [tài liệu cài đặt](/docs/{{version}}/installation#docker-installation-using-sail) của Laravel cho hệ điều hành của bạn. Trong quá trình cài đặt, bạn sẽ được yêu cầu chọn những service được Sail hỗ trợ mà ứng dụng của bạn sẽ tương tác cùng.

<a name="installing-sail-into-existing-applications"></a>
### Cài đặt Sail vào trong application hiện tại

Nếu bạn quan tâm đến việc sử dụng Sail với ứng dụng Laravel hiện có, bạn có thể chỉ cần cài đặt Sail bằng Composer package manager. Tất nhiên, các bước này giả định rằng môi trường phát triển local hiện tại của bạn cho phép bạn cài đặt các library của Composer:

```shell
composer require laravel/sail --dev
```

Sau khi Sail đã được cài đặt, bạn có thể chạy lệnh Artisan `sail:install`. Lệnh này sẽ export file `docker-compose.yml` của Sail vào thư mục root của ứng dụng của bạn và bạn có thể sửa file `.env` của bạn bằng các biến môi trường cần thiết để kết nối với các service của Docker:

```shell
php artisan sail:install
```

Cuối cùng, bạn có thể bắt đầu Sail. Để tiếp tục tìm hiểu cách sử dụng Sail, vui lòng tiếp tục đọc phần còn lại của tài liệu này:

```shell
./vendor/bin/sail up
```

> [!WARNING]
> Nếu bạn đang sử dụng Docker Desktop cho Linux, bạn nên sử dụng Docker context `default` bằng cách chạy lệnh sau: `docker context use default`.

<a name="adding-additional-services"></a>
#### Adding Additional Services

Nếu bạn muốn thêm một service bổ sung vào cài đặt Sail hiện tại của bạn, bạn có thể chạy lệnh Artisan `sail:add`:

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### Using Devcontainers

Nếu muốn phát triển trong một [Devcontainer](https://code.visualstudio.com/docs/remote/containers), bạn có thể cung cấp tùy chọn `--devcontainer` cho lệnh `sail:install`. Tùy chọn `--devcontainer` sẽ hướng dẫn lệnh `sail:install` sẽ export file `.devcontainer/devcontainer.json ` mặc định vào thư mục root của ứng dụng của bạn:

```shell
php artisan sail:install --devcontainer
```

<a name="configuring-a-shell-alias"></a>
### Cấu hình một shell alias

Mặc định, các lệnh Sail được gọi bằng cách sử dụng tập lệnh script `vendor/bin/sail` có trong tất cả các ứng dụng Laravel mới:

```shell
./vendor/bin/sail up
```

Tuy nhiên, thay vì gõ liên tục `vendor/bin/sail` để chạy các lệnh Sail, bạn có thể muốn cấu hình một shell alias cho phép bạn chạy các lệnh của Sail một cách dễ dàng hơn:

```shell
alias sail='sh $([ -f sail ] && echo sail || echo vendor/bin/sail)'
```

Để đảm bảo tính năng này luôn sẵn sàng, bạn có thể thêm lệnh này vào file cấu hình shell trong thư mục root của bạn, chẳng hạn như `~/.zshrc` hoặc `~/.bashrc`, sau đó khởi động lại shell.

Khi shell alias đã được cấu hình xong, bạn có thể chạy các lệnh Sail bằng cách chỉ cần gõ `sail`. Các ví dụ còn lại của tài liệu này sẽ giả sử là bạn đã cấu hình alias này:

```bash
sail up
```

<a name="starting-and-stopping-sail"></a>
## Starting và Stopping Sail

File `docker-compose.yml` của Laravel Sail định nghĩa nhiều container Docker hoạt động cùng nhau để giúp bạn xây dựng các ứng dụng Laravel. Mỗi container này là một mục trong cấu hình `services` của file `docker-compose.yml` của bạn. Container `laravel.test` là container ứng dụng chính sẽ chạy ứng dụng của bạn.

Trước khi khởi động Sail, bạn phải đảm bảo rằng không có máy chủ web hoặc cơ sở dữ liệu nào khác đang chạy trên máy tính local của bạn. Để khởi động tất cả các container Docker được định nghĩa trong file `docker-compose.yml` của ứng dụng, bạn nên chạy lệnh `up`:

```shell
sail up
```

Để khởi động tất cả các container Docker ở background, bạn có thể khởi động Sail ở chế độ "detached":

```shell
sail up -d
```

Khi container của ứng dụng đã được khởi động xong, bạn có thể truy cập vào dự án trong trình duyệt web của bạn tại: http://localhost.

Để dừng tất cả các container, bạn chỉ cần nhấn Control + C để dừng quá trình chạy của container. Hoặc, nếu các container đang chạy ở background, bạn có thể sử dụng lệnh `stop`:

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## Chạy commands

Khi sử dụng Laravel Sail, ứng dụng của bạn đang được chạy trong container Docker và được cách ly hoàn toàn khỏi máy tính local của bạn. Tuy nhiên, Sail cũng cung cấp một cách thuận tiện để chạy các lệnh khác nhau đối với ứng dụng của bạn, chẳng hạn như lệnh PHP, lệnh Artisan, lệnh Composer và lệnh Node/NPM.

**Khi đọc tài liệu Laravel, bạn sẽ thường thấy các lệnh chạy thẳng đến các lệnh Composer, Artisan và Node/NPM mà không chạy đến Sail.** Những ví dụ đó giả định là những công cụ này được cài đặt trên máy tính local của bạn. Nếu bạn đang sử dụng Sail cho môi trường phát triển Laravel local của bạn, bạn nên chạy các lệnh đó bằng Sail:

```shell
# Running Artisan commands locally...
php artisan queue:work

# Running Artisan commands within Laravel Sail...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### Chạy PHP Commands

Các lệnh PHP có thể được chạy bằng lệnh `php`. Tất nhiên, các lệnh này sẽ chạy bằng phiên bản PHP được cấu hình cho ứng dụng của bạn. Để tìm hiểu thêm về các phiên bản PHP có sẵn cho Laravel Sail, hãy tham khảo [tài liệu về phiên bản PHP](#sail-php-versions):

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Chạy Composer Commands

Các lệnh của Composer có thể được chạy bằng lệnh `composer`. Container ứng dụng của Laravel Sail đã có bản cài đặt Composer version 2.x:

```nothing
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### Installing Composer Dependencies For Existing Applications

Nếu bạn đang phát triển một ứng dụng với một nhóm của bạn, bạn có thể không phải là người đầu tiên tạo ra ứng dụng Laravel. Do đó, không có library Composer nào của ứng dụng, kể cả Sail, sẽ được cài đặt sau khi bạn clone repository của ứng dụng vào máy tính local của bạn.

Bạn có thể cài đặt các phần library của ứng dụng bằng cách điều hướng đến thư mục của ứng dụng và thực hiện lệnh sau. Lệnh này sử dụng một container Docker nhỏ chứa PHP và Composer để cài đặt các phần library của ứng dụng:

```shell
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php83-composer:latest \
    composer install --ignore-platform-reqs
```

Khi sử dụng image `laravelsail/phpXX-composer`, bạn nên sử dụng cùng một phiên bản PHP mà bạn đang định sử dụng cho ứng dụng của bạn (`80`, `81`, `82`, hoặc `83`).

<a name="executing-artisan-commands"></a>
### Chạy Artisan Commands

Các lệnh Laravel Artisan có thể được chạy bằng lệnh `artisan`:

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Chạy Node và NPM Commands

Các lệnh Node có thể được chạy bằng lệnh `node` trong khi các lệnh NPM có thể được chạy bằng lệnh `npm`:

```shell
sail node --version

sail npm run dev
```

Nếu muốn, bạn có thể sử dụng Yarn thay vì NPM:

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## Tương tác với Databases

<a name="mysql"></a>
### MySQL

Như bạn có thể nhận thấy, file `docker-compose.yml` của ứng dụng của bạn chứa một mục cho container MySQL. Container này sử dụng một [Docker volume](https://docs.docker.com/storage/volumes/) để lưu trữ dữ liệu trong cơ sở dữ liệu của bạn và nó sẽ được duy trì ngay cả khi dừng hoặc khởi động lại container.

Ngoài ra, lần đầu tiên container MySQL khởi động, nó sẽ tạo hai cơ sở dữ liệu cho bạn. Cơ sở dữ liệu đầu tiên được đặt tên bằng giá trị của biến môi trường `DB_DATABASE` và dành cho phát triển local của bạn. Cơ sở dữ liệu thứ hai là cơ sở dữ liệu test chuyên dụng có tên là `testing` và sẽ đảm bảo rằng các bài test của bạn không can thiệp vào dữ liệu phát triển của bạn.

Sau khi khởi động container, bạn có thể kết nối với instance MySQL trong ứng dụng của bạn bằng cách set biến môi trường `DB_HOST` trong file `.env` của ứng dụng thành `mysql`.

Để kết nối đến cơ sở dữ liệu MySQL của ứng dụng từ máy local, bạn có thể sử dụng ứng dụng quản lý cơ sở dữ liệu như [TablePlus](https://tableplus.com). Mặc định, cơ sở dữ liệu MySQL có thể truy cập được tại `localhost` cổng 3306 và thông tin xác thực truy cập tương ứng với các giá trị của biến môi trường `DB_USERNAME` và `DB_PASSWORD`. Hoặc, bạn có thể kết nối với tư cách là người dùng `root`, cũng sử dụng giá trị của biến môi trường `DB_PASSWORD` làm mật khẩu.

<a name="redis"></a>
### Redis

File `docker-compose.yml` của ứng dụng của bạn cũng chứa một mục cho container [Redis](https://redis.io). Container này sử dụng một [Docker volume](https://docs.docker.com/storage/volumes/) để lưu trữ dữ liệu Redis của bạn và nó sẽ được duy trì ngay cả khi dừng hoặc khởi động lại container của bạn. Sau khi khởi động container, bạn có thể kết nối với instance Redis trong ứng dụng của bạn bằng cách set biến môi trường `REDIS_HOST` trong file `.env` của ứng dụng thành `redis`.

Để kết nối đến cơ sở dữ liệu Redis của ứng dụng từ máy local, bạn có thể sử dụng ứng dụng quản lý cơ sở dữ liệu như [TablePlus](https://tableplus.com). Mặc định, cơ sở dữ liệu Redis có thể truy cập được tại `localhost` cổng 6379.

<a name="meilisearch"></a>
### Meilisearch

Nếu bạn chọn cài đặt service [Meilisearch](https://www.meilisearch.com) khi cài đặt Sail, file `docker-compose.yml` của ứng dụng của bạn sẽ chứa một mục cho công cụ tìm kiếm mạnh mẽ này [tương thích](https://github.com/meilisearch/meilisearch-laravel-scout) với [Laravel Scout](/docs/{{version}}/scout). Sau khi khởi động container, bạn có thể kết nối đến instance Meilisearch trong ứng dụng của bạn bằng cách set biến môi trường `MEILISEARCH_HOST` thành `http://meilisearch:7700`.

Từ máy local của bạn, bạn có thể truy cập vào trang admin dựa trên web của Meilisearch bằng cách vào `http://localhost:7700` trong trình duyệt web của bạn.

<a name="typesense"></a>
### Typesense

Nếu bạn chọn cài đặt service [Typesense](https://typesense.org) khi cài đặt Sail, file `docker-compose.yml` của ứng dụng sẽ chứa một mục cho công cụ tìm kiếm mã nguồn mở này và được tích hợp sẵn với [Laravel Scout](/docs/{{version}}/scout#typesense). Sau khi khởi động container, bạn có thể kết nối với phiên bản Typesense trong ứng dụng của bạn bằng cách set các biến môi trường sau:

```ini
TYPESENSE_HOST=typesense
TYPESENSE_PORT=8108
TYPESENSE_PROTOCOL=http
TYPESENSE_API_KEY=xyz
```

Từ máy local của bạn, bạn có thể truy cập API của Typesense qua `http://localhost:8108`.

<a name="file-storage"></a>
## File Storage

Nếu bạn dự định sử dụng Amazon S3 để lưu trữ file trong khi chạy ứng dụng của bạn trong môi trường production, bạn có thể muốn cài đặt service [MinIO](https://min.io) khi cài đặt Sail. MinIO cung cấp API tương thích với S3 mà bạn có thể sử dụng để phát triển local bằng driver storage file `s3` của Laravel mà không cần tạo bucket lưu trữ "thử nghiệm" trong môi trường S3 production của bạn. Nếu bạn muốn chọn cài đặt MinIO trong khi cài đặt Sail, phần cấu hình MinIO sẽ được thêm vào file `docker-compose.yml` của ứng dụng của bạn.

Mặc định, file cấu hình `filesystems` của ứng dụng của bạn đã chứa cấu hình disk cho disk `s3`. Ngoài việc sử dụng disk này để tương tác với Amazon S3, bạn có thể sử dụng disk này để tương tác với bất kỳ dịch vụ lưu trữ file nào mà tương thích với S3 chẳng hạn như MinIO bằng cách sửa các biến môi trường liên quan đến kiểm soát cấu hình của nó. Ví dụ: khi sử dụng MinIO, cấu hình biến môi trường filesystem của bạn phải được định nghĩa như sau:

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

Để tích hợp Flysystem của Laravel vào để tạo ra các URL phù hợp khi sử dụng MinIO, bạn phải định nghĩa biến môi trường `AWS_URL` sao cho nó khớp với URL local của ứng dụng và chứa tên bucket trong đường dẫn URL:

```ini
AWS_URL=http://localhost:9000/local
```

Bạn có thể tạo bucket thông qua bảng điều khiển của MinIO tại `http://localhost:8900`. Tên người dùng mặc định cho bảng điều khiển MinIO là `sail` và mật khẩu mặc định là `password`.

> [!WARNING]
> Việc tạo URL tạm thời thông qua phương thức `temporaryUrl` sẽ không được hỗ trợ khi sử dụng MinIO.

<a name="running-tests"></a>
## Running Tests

Laravel mặc định cung cấp khả năng testing tuyệt vời và bạn có thể sử dụng lệnh `test` của Sail để chạy các [bài kiểm tra tính năng hoặc unit test](/docs/{{version}}/testing) cho ứng dụng của bạn. Bất kỳ tùy chọn CLI nào mà được PHPUnit chấp nhận cũng có thể được truyền cho lệnh `test`:

```shell
sail test

sail test --group orders
```

Lệnh Sail `test` tương đương với việc chạy lệnh Artisan `test`:

```shell
sail artisan test
```

Mặc định, Sail sẽ tạo một cơ sở dữ liệu `testing` chuyên dụng để các bài test của bạn không can thiệp vào trạng thái hiện tại của cơ sở dữ liệu. Trong cài đặt mặc định của Laravel, Sail cũng sẽ cấu hình file `phpunit.xml` của bạn để sử dụng cơ sở dữ liệu này khi thực hiện các bài test của bạn:

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) cung cấp API testing và automation browser dễ sử dụng và mang tính hàm ý. Nhờ Sail, bạn có thể chạy các bài test này mà không cần cài đặt Selenium hoặc các công cụ khác trên máy tính local của bạn. Để bắt đầu, hãy uncomment service Selenium trong file `docker-compose.yml` của ứng dụng của bạn:

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

Tiếp theo, hãy đảm bảo là service `laravel.test` trong file `docker-compose.yml` trong ứng dụng của bạn có một mục `depends_on` cho `selenium`:

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

Cuối cùng, bạn có thể chạy bài test Dusk bằng cách khởi động Sail và chạy lệnh `dusk`:

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Selenium On Apple Silicon

Nếu máy local của bạn dùng chip Apple Silicon, service `selenium` của bạn phải được sử dụng image `seleniarm/standalone-chromium`:

```yaml
selenium:
    image: 'seleniarm/standalone-chromium'
    extra_hosts:
        - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

<a name="previewing-emails"></a>
## Previewing Emails

File `docker-compose.yml` mặc định của Laravel Sail có chứa một mục cho service [Mailpit](https://github.com/axllent/mailpit). Mailpit sẽ chặn các email được gửi đi bởi ứng dụng của bạn trong quá trình phát triển local và cung cấp giao diện web thuận tiện để bạn có thể xem các email đã được gửi trong trình duyệt của bạn. Khi sử dụng Sail, máy chủ mặc định của Mailpit là `mailpit` và trên cổng 1025:

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

Khi Sail đang chạy, bạn có thể truy cập vào giao diện web Mailpit tại: http://localhost:8025

<a name="sail-container-cli"></a>
## Container CLI

Thỉnh thoảng bạn có thể muốn bắt đầu một Bash session trong container ứng dụng của bạn. Bạn có thể sử dụng lệnh `shell` để kết nối với container ứng dụng của bạn, cho phép bạn kiểm tra các file của nó và các service đã cài đặt cũng như chạy các lệnh shell tùy ý trong container:

```shell
sail shell

sail root-shell
```

Để bắt đầu một session [Laravel Tinker](https://github.com/laravel/tinker) mới, bạn có thể chạy lệnh `tinker`:

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## PHP Versions

Sail hiện hỗ trợ chạy ứng dụng của bạn thông qua PHP 8.3, 8.2, 8.1, hoặc PHP 8.0. Phiên bản PHP mặc định được Sail sử dụng hiện tại là PHP 8.3. Để thay đổi phiên bản PHP được sử dụng để chạy ứng dụng của bạn, bạn nên cập nhật định nghĩa `build` của container `laravel.test` trong file `docker-compose.yml` của ứng dụng:

```yaml
# PHP 8.3
context: ./vendor/laravel/sail/runtimes/8.3

# PHP 8.2
context: ./vendor/laravel/sail/runtimes/8.2

# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0
```

Ngoài ra, bạn có thể muốn cập nhật tên `image` của bạn để phản ánh phiên bản PHP đang được ứng dụng của bạn sử dụng. Tùy chọn này cũng được định nghĩa trong file `docker-compose.yml` trong ứng dụng của bạn:

```yaml
image: sail-8.1/app
```

Sau khi cập nhật file `docker-compose.yml` của ứng dụng, bạn nên build lại image container của bạn:

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Node Versions

Mặc định, Sail cài đặt Node 20. Để thay đổi phiên bản Node được cài đặt khi build image của bạn, bạn có thể cập nhật định nghĩa `build.args` của service `laravel.test` trong file `docker-compose.yml` của ứng dụng của bạn:

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '18'
```

Sau khi cập nhật file `docker-compose.yml` của ứng dụng, bạn nên build lại image container của bạn:

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## Sharing Your Site

Thỉnh thoảng, bạn có thể cần share trang web của bạn để cho người khác có thể xem qua trang web của bạn hoặc để test khả năng tích hợp webhook với ứng dụng của bạn. Để share trang web của bạn, bạn có thể sử dụng lệnh `share`. Sau khi thực hiện lệnh này, bạn sẽ được nhận được một URL `laravel-sail.site` ngẫu nhiên mà bạn có thể sử dụng để truy cập vào ứng dụng của bạn:

```shell
sail share
```

Khi chia sẻ trang web của bạn thông qua lệnh `share`, bạn nên cấu hình các proxy đáng tin cậy của ứng dụng của bạn trong middleware `TrustProxies`. Nếu không, các helper tạo URL như `url` và `route` sẽ không thể xác định HTTP host chính xác sẽ được sử dụng trong quá trình tạo URL:

    /**
     * The trusted proxies for this application.
     *
     * @var array|string|null
     */
    protected $proxies = '*';

Nếu bạn muốn chọn subdomain cho trang web được chia sẻ của bạn, bạn có thể cung cấp tùy chọn `subdomain` khi chạy lệnh `share`:

```shell
sail share --subdomain=my-sail-site
```

> [!NOTE]
> Lệnh `share` được hỗ trợ bởi [Expose](https://github.com/beyondcode/expose), một service nguồn mở của [BeyondCode](https://beyondco.de).

<a name="debugging-with-xdebug"></a>
## Debugging With Xdebug

Cấu hình Docker của Laravel Sail có hỗ trợ cho [Xdebug](https://xdebug.org/), một trình debug phổ biến và mạnh mẽ cho PHP. Để bật Xdebug, bạn cần thêm một vài biến vào file `.env` của ứng dụng để [cấu hình Xdebug](https://xdebug.org/docs/step_debug#mode). Để bật Xdebug, bạn phải set (các) chế độ thích hợp trước khi khởi động Sail:

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

#### Linux Host IP Configuration

Bên trong, biến môi trường `XDEBUG_CONFIG` sẽ được định nghĩa là `client_host=host.docker.internal` để Xdebug sẽ được cấu hình đúng cho Mac và Windows (WSL2). Nếu máy local của bạn đang chạy Linux, bạn nên đảm bảo là bạn đang chạy Docker Engine 17.06.0+ và Compose 1.16.0+. Nếu không, bạn sẽ cần định nghĩa thủ công biến môi trường này như ở bên dưới.

Trước tiên, bạn nên xác định chính xác địa chỉ IP máy chủ để thêm vào biến môi trường bằng cách chạy lệnh sau. Thông thường, `<container-name>` phải là tên của container chạy ứng dụng của bạn và thường kết thúc bằng `_laravel.test_1`:

```shell
docker inspect -f {{range.NetworkSettings.Networks}}{{.Gateway}}{{end}} <container-name>
```

Khi bạn đã có được chính xác địa chỉ IP máy chủ, bạn nên định nghĩa biến `SAIL_XDEBUG_CONFIG` trong file `.env` của ứng dụng của bạn:

```ini
SAIL_XDEBUG_CONFIG="client_host=<host-ip-address>"
```

<a name="xdebug-cli-usage"></a>
### Xdebug CLI Usage

Lệnh `sail debug` có thể được sử dụng để bắt đầu session debug khi chạy lệnh Artisan:

```shell
# Run an Artisan command without Xdebug...
sail artisan migrate

# Run an Artisan command with Xdebug...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Xdebug Browser Usage

Để debug ứng dụng trong khi tương tác với ứng dụng đó qua trình duyệt web, bạn hãy làm theo [hướng dẫn do Xdebug cung cấp](https://xdebug.org/docs/step_debug#web-application) để bắt đầu session Xdebug từ trình duyệt web.

Nếu bạn đang sử dụng PhpStorm, thì vui lòng xem lại tài liệu của JetBrain về [debug không cần cấu hình](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html).

> [!WARNING]
> Laravel Sail dựa vào `artisan Serve` để chạy ứng dụng của bạn. Lệnh `artisan Serve` chỉ chấp nhận các biến `XDEBUG_CONFIG` và `XDEBUG_MODE` kể từ phiên bản Laravel 8.53.0. Các phiên bản cũ hơn của Laravel (8.52.0 trở xuống) sẽ không hỗ trợ các biến này và sẽ không chấp nhận khi kết nối debug.

<a name="sail-customization"></a>
## Tuỳ chỉnh

Vì Sail chỉ là Docker nên bạn có thể tự do tùy chỉnh hầu hết mọi thứ về nó. Để export Dockerfiles của Sail, bạn có thể chạy lệnh `sail:publish`:

```shell
sail artisan sail:publish
```

Sau khi chạy lệnh này, Dockerfiles và các file cấu hình khác được Laravel Sail sử dụng sẽ được lưu vào trong thư mục `docker` trong thư mục root của ứng dụng của bạn. Sau khi tùy chỉnh cài đặt của Sail, bạn có thể muốn thay đổi tên image cho container ứng dụng trong file `docker-compose.yml` của ứng dụng. Sau khi làm như vậy, hãy build lại container ứng dụng của bạn bằng lệnh `build`. Gán một tên duy nhất cho image ứng dụng sẽ đặc biệt quan trọng nếu bạn đang sử dụng Sail để phát triển nhiều ứng dụng Laravel trên một máy local:

```shell
sail build --no-cache
```
