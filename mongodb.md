# MongoDB

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [MongoDB Driver](#mongodb-driver)
    - [Khởi động một MongoDB Server](#starting-a-mongodb-server)
    - [Cài đặt package Laravel MongoDB](#install-the-laravel-mongodb-package)
- [Cấu hình](#configuration)
- [Chức năng](#features)

<a name="introduction"></a>
## Giới thiệu

[MongoDB](https://www.mongodb.com/resources/products/fundamentals/why-use-mongodb) là một trong những cơ sở dữ liệu NoSQL hướng document phổ biến nhất, được sử dụng nhờ khả năng chịu tải ghi cao (hữu ích cho phân tích hoặc IoT) và tính sẵn sàng cao (dễ dàng thiết lập các replica set với tính năng tự động chuyển đổi dự phòng). Nó cũng có thể phân mảnh cơ sở dữ liệu một cách dễ dàng để mở rộng quy mô theo chiều ngang và có ngôn ngữ truy vấn mạnh mẽ để thực hiện các phép tổng hợp, tìm kiếm văn bản hoặc truy vấn địa lý trong không gian.

Thay vì lưu dữ liệu trong các bảng gồm các hàng và các cột như cơ sở dữ liệu SQL truyền thống, mỗi bản ghi trong cơ sở dữ liệu MongoDB là một document được mô tả bằng BSON, đây là một kiểu nhị phân của dữ liệu. Các ứng dụng sau đó có thể truy xuất thông tin này dưới định dạng JSON. Nó hỗ trợ nhiều loại dữ liệu khác nhau, bao gồm các document, mảng, document nhúng và dữ liệu nhị phân.

Trước khi sử dụng MongoDB với Laravel, chúng tôi khuyên bạn nên cài đặt và sử dụng package `mongodb/laravel-mongodb` thông qua Composer. Package `laravel-mongodb` này được duy trì chính thức bởi MongoDB, và trong khi MongoDB được PHP gốc hỗ trợ thông qua driver MongoDB, thì package [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) cung cấp khả năng tích hợp phong phú hơn với Eloquent và các tính năng khác của Laravel:

```shell
composer require mongodb/laravel-mongodb
```

<a name="installation"></a>
## Cài đặt

<a name="mongodb-driver"></a>
### MongoDB Driver

Để kết nối với cơ sở dữ liệu MongoDB, extension PHP `mongodb` là bắt buộc. Nếu bạn đang phát triển local bằng [Laravel Herd](https://herd.laravel.com) hoặc cài đặt PHP thông qua `php.new`, bạn đã có sẵn extension này trên hệ thống của bạn. Tuy nhiên, nếu bạn muốn tự cài đặt extension này, bạn có thể thực hiện thông qua PECL:

```shell
pecl install mongodb
```

Để biết thêm thông tin về việc cài đặt extension PHP MongoDB, hãy xem [hướng dẫn cài đặt extension PHP MongoDB](https://www.php.net/manual/en/mongodb.installation.php).

<a name="starting-a-mongodb-server"></a>
### Khởi động một MongoDB Server

MongoDB Community Server có thể được sử dụng để chạy MongoDB ở local và có sẵn để cài đặt trên Windows, macOS, Linux hoặc dưới dạng Docker container. Để tìm hiểu về cách cài đặt MongoDB, vui lòng tham khảo [hướng dẫn cài đặt MongoDB Community chính thức](https://docs.mongodb.com/manual/administration/install-community/).

Chuỗi kết nối cho MongoDB server có thể được thiết lập trong file `.env` của bạn:

```ini
MONGODB_URI="mongodb://localhost:27017"
MONGODB_DATABASE="laravel_app"
```

Để chạy MongoDB trên cloud, bạn hãy cân nhắc sử dụng [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
Để truy cập vào cluster MongoDB Atlas từ ứng dụng của bạn ở local, bạn sẽ cần [thêm địa chỉ IP của bạn vào trong cài đặt mạng của cluster](https://www.mongodb.com/docs/atlas/security/add-ip-address-to-list/).

Chuỗi kết nối cho MongoDB Atlas có thể được thiết lập trong file `.env` của bạn:

```ini
MONGODB_URI="mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>?retryWrites=true&w=majority"
MONGODB_DATABASE="laravel_app"
```

<a name="install-the-laravel-mongodb-package"></a>
### Cài đặt package Laravel MongoDB

Cuối cùng, hãy sử dụng Composer để cài đặt package Laravel MongoDB:

```shell
composer require mongodb/laravel-mongodb
```

> [!NOTE]
Việc cài đặt package này sẽ bị lỗi nếu extension PHP `mongodb` chưa được cài đặt. Cấu hình PHP có thể khác nhau giữa CLI và web server, vì vậy hãy đảm bảo extension này đã được enable cho cả hai cấu hình.

<a name="configuration"></a>
## Cấu hình

Bạn có thể cấu hình kết nối MongoDB thông qua file cấu hình `config/database.php` của ứng dụng. Trong file này, hãy thêm một kết nối `mongodb` sử dụng driver `mongodb`:

```php
'connections' => [
    'mongodb' => [
        'driver' => 'mongodb',
        'dsn' => env('MONGODB_URI', 'mongodb://localhost:27017'),
        'database' => env('MONGODB_DATABASE', 'laravel_app'),
    ],
],
```

<a name="features"></a>
## Chức năng

Sau khi quá trình cấu hình hoàn tất, bạn có thể sử dụng package `mongodb` và kết nối cơ sở dữ liệu trong ứng dụng của bạn để tận dụng nhiều tính năng mạnh mẽ khác nhau:

- [Sử dụng Eloquent](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/eloquent-models/), các model có thể được lưu trong các collection của MongoDB. Ngoài các tính năng Eloquent có, package Laravel MongoDB cũng cung cấp thêm các tính năng khác như embedded relationship. Package này cũng cung cấp quyền truy cập trực tiếp vào driver MongoDB, có thể được sử dụng để thực hiện các thao tác như raw queries và aggregation pipelines.
- [Viết các truy vấn phức tạp](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/query-builder/) sử dụng query builder.
- [Cache driver](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/) `mongodb` được tối ưu hóa để sử dụng các tính năng của MongoDB như TTL index để tự động xóa các item cache đã hết hạn.
- [Gửi và xử lý các job trong queue](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/) với queue driver `mongodb`.
- [Lưu trữ file trong GridFS](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/filesystems/), thông qua [GridFS Adapter cho Flysystem](https://flysystem.thephpleague.com/docs/adapter/gridfs/).
- Hầu hết các package của bên thứ ba sử dụng kết nối cơ sở dữ liệu hoặc Eloquent đều có thể được sử dụng với MongoDB.

Để tiếp tục tìm hiểu về cách sử dụng MongoDB và Laravel, hãy tham khảo [hướng dẫn bắt đầu nhanh](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/quick-start/) của MongoDB.
