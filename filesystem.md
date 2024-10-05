# File Storage

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Local Driver](#the-local-driver)
    - [Public Disk](#the-public-disk)
    - [Yêu cầu Driver](#driver-prerequisites)
    - [Scoped và Read-Only Filesystems](#scoped-and-read-only-filesystems)
    - [Filesystem tương thích Amazon S3](#amazon-s3-compatible-filesystems)
- [Lấy Disk Instance](#obtaining-disk-instances)
    - [Disk theo yêu cầu](#on-demand-disks)
- [Lấy File](#retrieving-files)
    - [Tải File](#downloading-files)
    - [File URL](#file-urls)
    - [Temporary URLs](#temporary-urls)
    - [File Metadata](#file-metadata)
- [Lưu File](#storing-files)
    - [Ghi vào đầu hoặc cuối file](#prepending-appending-to-files)
    - [Copy và di chuyển Files](#copying-moving-files)
    - [Automatic Streaming](#automatic-streaming)
    - [File Uploads](#file-uploads)
    - [File Visibility](#file-visibility)
- [Xoá File](#deleting-files)
- [Thư mục](#directories)
- [Testing](#testing)
- [Tuỳ chỉnh Filesystem](#custom-filesystems)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một abstraction filesystem mạnh mẽ nhờ package PHP [Flysystem](https://github.com/thephpleague/flysystem) tuyệt vời của Frank de Jonge. Laravel Flysystem integration cung cấp các driver đơn giản để làm việc với các local filesystems, SFTP, và Amazon S3. Thậm chí, nó cũng rất đơn giản để chuyển đổi giữa các tùy chọn lưu trữ giữa máy phát triển local của bạn và máy chủ production vì API vẫn giống nhau cho mỗi hệ thống.

<a name="configuration"></a>
## Cấu hình

File cấu hình filesystem của Laravel được lưu tại `config/filesystems.php`. Trong file này, bạn có thể cấu hình tất cả các filesystem "disks" của bạn. Mỗi disk sẽ được đại diện cho một driver lưu trữ với một vị trí lưu trữ cụ thể. Các cấu hình mẫu cho các driver được hỗ trợ cũng đã được khai báo sẵn vào trong file cấu hình vì vậy bạn có thể sửa cấu hình để đúng với tuỳ chọn lưu trữ của bạn và thông tin của chúng.

Driver `local` tương tác với các file được lưu trữ local trên máy chủ đang chạy ứng dụng Laravel trong khi driver `s3` sẽ được sử dụng để write vào dịch vụ lưu trữ đám mây S3 của Amazon.

> [!NOTE]
> Bạn có thể cấu hình bao nhiêu disk tùy ý của bạn và thậm chí có thể có nhiều disk sử dụng cùng một driver.

<a name="the-local-driver"></a>
### The Local Driver

Khi sử dụng driver `local`, tất cả các hoạt động của file đều liên quan đến thư mục `root` được định nghĩa trong file cấu hình `filesystems` của bạn. Mặc định, giá trị này được set cho thư mục `storage/app`. Do đó, phương thức sau sẽ write vào `storage/app/example.txt`:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('local')->put('example.txt', 'Contents');

<a name="the-public-disk"></a>
### Public Disk

Disk `public` có trong file cấu hình `filesystems` của ứng dụng của bạn là dành cho các file có thể truy cập ở dạng công khai. Mặc định, `public` disk sẽ sử dụng driver `local` và lưu trữ các file này trong `storage/app/public`.

Để làm cho các file này có thể truy cập từ web, bạn nên tạo một link liên kết ảo từ `public/storage` đến `storage/app/public`. Việc sử dụng quy ước thư mục này sẽ giúp cho các file có thể truy cập công khai của bạn ở trong một thư mục có thể dễ dàng chia sẻ qua mỗi lần deploy khi sử dụng các hệ thống deploy zero down-time như [Envoyer](https://envoyer.io).

Để tạo link liên kết ảo, bạn có thể sử dụng lệnh Artisan `storage:link`:

```shell
php artisan storage:link
```

Một khi một file đã được lưu trữ và link liên kết ảo đã được tạo xong, bạn có thể tạo URL tới các file này bằng cách sử dụng helper `asset`:

    echo asset('storage/file.txt');

Bạn có thể cấu hình thêm các link ảo trong file cấu hình `filesystems` của bạn. Mỗi link được cấu hình sẽ được tạo khi bạn chạy lệnh `storage:link`:

    'links' => [
        public_path('storage') => storage_path('app/public'),
        public_path('images') => storage_path('app/images'),
    ],

Lệnh `storage:unlink` có thể được sử dụng để hủy các link liên kết ảo đã cấu hình của bạn:

```shell
php artisan storage:unlink
```

<a name="driver-prerequisites"></a>
### Yêu cầu Driver

<a name="s3-driver-configuration"></a>
#### S3 Driver Configuration

Trước khi sử dụng driver S3, bạn cần cài đặt package Flysystem S3 thông qua trình quản lý package Composer:

```shell
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

Thông tin cấu hình driver S3 nằm trong file cấu hình `config/filesystems.php` của bạn. File này chứa một mảng cấu hình mẫu cho driver S3. Bạn có thể tự do sửa mảng này với thông tin và cấu hình S3 của riêng bạn. Để thuận tiện, các biến môi trường đã được đặt tên khớp với quy ước đặt tên được sử dụng bởi AWS CLI.

<a name="ftp-driver-configuration"></a>
#### FTP Driver Configuration

Trước khi sử dụng driver FTP, bạn cần cài đặt package Flysystem FTP thông qua trình quản lý package Composer:

```shell
composer require league/flysystem-ftp "^3.0"
```

Flysystem integration của Laravel hoạt động tốt với FTP; tuy nhiên, mặc định, một cấu hình mẫu không được thêm vào trong file cấu hình `filesystems.php` của framework. Nếu bạn cần cấu hình một hệ thống file FTP, bạn có thể sử dụng cấu hình mẫu ở bên dưới:

    'ftp' => [
        'driver' => 'ftp',
        'host' => env('FTP_HOST'),
        'username' => env('FTP_USERNAME'),
        'password' => env('FTP_PASSWORD'),

        // Optional FTP Settings...
        // 'port' => env('FTP_PORT', 21),
        // 'root' => env('FTP_ROOT'),
        // 'passive' => true,
        // 'ssl' => true,
        // 'timeout' => 30,
    ],

<a name="sftp-driver-configuration"></a>
#### SFTP Driver Configuration

Trước khi sử dụng driver SFTP, bạn cần cài đặt package Flysystem SFTP thông qua trình quản lý package Composer:

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

Flysystem tích hợp trong Laravel hoạt động tốt với SFTP; tuy nhiên, mặc định một cấu hình mẫu sẽ không có trong file cấu hình `filesystems.php` của framework. Nếu bạn cần cấu hình một hệ thống filesystem SFTP, bạn có thể sử dụng cấu hình ví dụ ở bên dưới:

    'sftp' => [
        'driver' => 'sftp',
        'host' => env('SFTP_HOST'),

        // Settings for basic authentication...
        'username' => env('SFTP_USERNAME'),
        'password' => env('SFTP_PASSWORD'),

        // Settings for SSH key based authentication with encryption password...
        'privateKey' => env('SFTP_PRIVATE_KEY'),
        'passphrase' => env('SFTP_PASSPHRASE'),

         // Settings for file / directory permissions...
        'visibility' => 'private', // `private` = 0600, `public` = 0644
        'directory_visibility' => 'private', // `private` = 0700, `public` = 0755

        // Optional SFTP Settings...
        // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
        // 'maxTries' => 4,
        // 'passphrase' => env('SFTP_PASSPHRASE'),
        // 'port' => env('SFTP_PORT', 22),
        // 'root' => env('SFTP_ROOT', ''),
        // 'timeout' => 30,
        // 'useAgent' => true,
    ],

<a name="scoped-and-read-only-filesystems"></a>
### Scoped và Read-Only Filesystems

Scoped disk cho phép bạn định nghĩa một hệ thống filesystem mà trong đó tất cả các đường dẫn được tự động thêm một tiền tố nhất định. Trước khi tạo một scoped filesystem disk, bạn sẽ cần cài đặt thêm package Flysystem thông qua trình quản lý package Composer:

```shell
composer require league/flysystem-path-prefixing "^3.0"
```

Bạn có thể tạo một instance scoped đường dẫn của bất kỳ filesystem disk nào hiện có bằng cách định nghĩa một disk sử dụng driver `scoped`. Ví dụ: bạn có thể tạo một disk scope cho disk `s3` hiện có với một tiền tố đường dẫn cụ thể, sau đó mọi thao tác với file bằng disk scope này sẽ sử dụng tiền tố đã được chỉ định:

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

Disk "read-only" cho phép bạn tạo ra các disk filesystem không cho phép quyền ghi. Trước khi sử dụng tùy chọn cấu hình `read-only`, bạn sẽ cần cài đặt thêm package Flysystem thông qua trình quản lý package Composer:

```shell
composer require league/flysystem-read-only "^3.0"
```

Tiếp theo, bạn có thể thêm tùy chọn cấu hình `read-only` vào trong một hoặc nhiều mảng cấu hình disk của bạn:

```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

<a name="amazon-s3-compatible-filesystems"></a>
### Filesystem tương thích Amazon S3

Mặc định, file cấu hình `filesystems` của ứng dụng sẽ chứa một cấu hình disk cho disk `s3`. Ngoài việc sử dụng disk này để tương tác với Amazon S3, bạn cũng có thể sử dụng nó để tương tác với bất kỳ dịch vụ lưu trữ file nào tương thích S3 nào, chẳng hạn như [MinIO](https://github.com/minio/minio) hoặc [DigitalOcean Spaces](https://www.digitalocean.com/products/spaces/).

Thông thường, sau khi cập nhật thông tin đăng nhập của disk để khớp với thông tin đăng nhập của dịch vụ mà bạn đang sử dụng, bạn chỉ cần cập nhật giá trị của tùy chọn của cấu hình `endpoint`. Giá trị tùy chọn này thường được định nghĩa thông qua biến môi trường `AWS_ENDPOINT`:

    'endpoint' => env('AWS_ENDPOINT', 'https://minio:9000'),

<a name="minio"></a>
#### MinIO

Để tích hợp Flysystem của Laravel vào việc tạo các URL khi sử dụng MinIO, bạn nên định nghĩa một biến môi trường là `AWS_URL` sao cho nó khớp với URL local của ứng dụng của bạn và chứa tên bucket trong đường dẫn URL:

```ini
AWS_URL=http://localhost:9000/local
```

> [!WARNING]
> Việc tạo URL tạm thời thông qua phương thức `temporaryUrl` không được hỗ trợ khi sử dụng MinIO.

<a name="obtaining-disk-instances"></a>
## Lấy Disk Instance

Facade `Storage` có thể được sử dụng để tương tác với bất kỳ disk nào mà bạn cấu hình. Ví dụ, bạn có thể sử dụng phương thức `put` trong facade này để lưu trữ hình đại diện cho một disk mà bạn muốn. Nếu bạn gọi các phương thức này trong facade `Storage` mà không khai báo thêm phương thức `disk`, thì câu lệnh này sẽ tự động được chuyển file đó đến disk mặc định:

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $content);

Nếu application của bạn tương tác với nhiều disk, thì bạn có thể sử dụng phương thức `disk` trên facade `Storage` để làm việc với các file cho một disk cụ thể:

    Storage::disk('s3')->put('avatars/1', $content);

<a name="on-demand-disks"></a>
### Disk theo yêu cầu

Thỉnh thoảng bạn có thể muốn tạo một disk trong khi ứng dụng chạy bằng cách sử dụng một cấu hình đã cho mà không có cấu hình đó ở trong file cấu hình `filesystems` của ứng dụng của bạn. Để thực hiện điều này, bạn có thể truyền một mảng cấu hình cho phương thức `build` của facade `Storage`:

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

<a name="retrieving-files"></a>
## Lấy File

Phương thức `get` có thể được sử dụng để lấy nội dung của file. Một chuỗi raw của nội dung file sẽ được phương thức trả về. Hãy nhớ rằng, tất cả các đường dẫn đến file phải được khai báo liên kết đến vị trí "root" của disk:

    $contents = Storage::get('file.jpg');

Nếu file bạn đang lấy ra có chứa chuỗi JSON, bạn có thể sử dụng phương thức `json` để lấy ra file đó và giải mã nội dung của file:

    $orders = Storage::json('orders.json');

Phương thức `exists` có thể được sử dụng để xác định xem một file có tồn tại trên disk hay không:

    if (Storage::disk('s3')->exists('file.jpg')) {
        // ...
    }

Phương thức `missing` có thể được sử dụng để xác định xem file có bị thiếu trong disk hay không:

    if (Storage::disk('s3')->missing('file.jpg')) {
        // ...
    }

<a name="downloading-files"></a>
### Tải File

Phương thức `download` có thể được sử dụng để tạo một response buộc trình duyệt của người dùng tải xuống một file theo đường dẫn đã cho. Phương thức `download` chấp nhận một tên file làm tham số thứ hai cho phương thức, tên file này sẽ hiển thị khi người dùng tải xuống. Cuối cùng, bạn có thể truyền một mảng HTTP header làm tham số thứ ba cho phương thức:

    return Storage::download('file.jpg');

    return Storage::download('file.jpg', $name, $headers);

<a name="file-urls"></a>
### File URL

Bạn có thể sử dụng phương thức `url` để lấy ra URL đã cho cho một file. Nếu bạn đang sử dụng driver `local`, điều này sẽ chỉ cần thêm `/storage` vào đường dẫn đã cho và trả về một URL tương đối cho file. Nếu bạn đang sử dụng driver `s3`, remote URL sẽ được trả về:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file.jpg');

Khi sử dụng driver `local`, tất cả các file mà có thể truy cập ở dạng công khai thì nên được lưu trong thư mục `storage/app/public`. Hơn nữa, bạn nên [tạo một link liên kết ảo](#the-public-disk) ở thư mục `public/storage` để trỏ đến thư mục `storage/app/public`.

> [!WARNING]
> Khi sử dụng driver `local`, giá trị trả về của `url` không phải là URL đã được encoded. Vì lý do này, mà chúng tôi khuyên bạn nên lưu trữ các file của bạn bằng các tên mà sẽ tạo ra URL hợp lệ.

<a name="url-host-customization"></a>
#### URL Host Customization

Nếu bạn muốn định nghĩa trước host cho các URL được tạo ra bằng cách sử dụng facade `Storage`, bạn có thể thêm tùy chọn `url` vào mảng cấu hình của disk:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="temporary-urls"></a>
### Temporary URLs

Sử dụng phương thức `temporaryUrl`, bạn có thể tạo ra các URL tạm cho các file được lưu trữ bằng driver `s3`. Phương thức này chấp nhận một đường dẫn và một instance `DateTime` để định nghĩa khi URL sẽ hết hạn:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

Nếu bạn cần chỉ định thêm một [S3 request parameters](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests), bạn có thể truyền một mảng request parameter làm tham số thứ ba cho phương thức `temporaryUrl`:

    $url = Storage::temporaryUrl(
        'file.jpg',
        now()->addMinutes(5),
        [
            'ResponseContentType' => 'application/octet-stream',
            'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
        ]
    );

Nếu bạn cần tùy chỉnh cách mà các URL tạm được tạo ra cho một disk lưu trữ cụ thể, bạn có thể sử dụng phương thức `buildTemporaryUrlsUsing`. Ví dụ: điều này có thể hữu ích nếu bạn có một controller cho phép người dùng tải xuống các file được lưu trữ thông qua disk mà thường không hỗ trợ URL tạm. Thông thường, phương thức này nên được gọi từ phương thức `boot` của một service provider:

    <?php

    namespace App\Providers;

    use DateTime;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Support\Facades\URL;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Storage::disk('local')->buildTemporaryUrlsUsing(
                function (string $path, DateTime $expiration, array $options) {
                    return URL::temporarySignedRoute(
                        'files.download',
                        $expiration,
                        array_merge($options, ['path' => $path])
                    );
                }
            );
        }
    }

<a name="temporary-upload-urls"></a>
#### Temporary Upload URLs

> [!WARNING]
> Chức năng URL temporary upload chỉ được hỗ trợ bởi driver `s3`.

Nếu bạn cần tạo một URL temporary có thể được sử dụng để upload file trực tiếp từ ứng dụng client-side của bạn, bạn có thể sử dụng phương thức `temporaryUploadUrl`. Phương thức này chấp nhận một đường dẫn và một instance `DateTime` chỉ định thời điểm URL sẽ hết hạn. Phương thức `temporaryUploadUrl` sẽ trả về một mảng có cấu trúc là một URL upload và các header cần được chứa trong upload request:

    use Illuminate\Support\Facades\Storage;

    ['url' => $url, 'headers' => $headers] = Storage::temporaryUploadUrl(
        'file.jpg', now()->addMinutes(5)
    );

Phương thức này chủ yếu hữu ích trong môi trường serverless yêu cầu ứng dụng client-side phải trực tiếp upload file lên hệ thống lưu trữ đám mây như Amazon S3.

<a name="file-metadata"></a>
### File Metadata

Ngoài việc đọc và ghi file, Laravel cũng cung cấp thông tin về các file đó. Ví dụ, phương thức `size` có thể được sử dụng để lấy ra kích thước của một file theo đơn vị byte:

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

Phương thức `lastModified` trả về một UNIX timestamp về lần cuối cùng mà file được sửa:

    $time = Storage::lastModified('file.jpg');

Loại MIME của một file nhất định có thể được lấy thông qua phương thức `mimeType`:

    $mime = Storage::mimeType('file.jpg');

<a name="file-paths"></a>
#### File Paths

Bạn có thể sử dụng phương thức `path` để lấy ra một đường dẫn cho một file nhất định. Nếu bạn đang sử dụng driver `local`, điều này sẽ trả về đường dẫn tuyệt đối tới file. Nếu bạn đang sử dụng driver `s3`, phương thức này sẽ trả về đường dẫn tương đối tới file trong bộ chứa S3:

    use Illuminate\Support\Facades\Storage;

    $path = Storage::path('file.jpg');

<a name="storing-files"></a>
## Lưu File

Phương thức `put` có thể được sử dụng để lưu trữ một nội dung của file lên disk. Bạn cũng có thể truyền một PHP `resource` đến phương thức `put`, phương thức này sẽ sử dụng support stream của Flysystem. Hãy nhớ rằng, tất cả các đường dẫn đến file phải được khai báo liên kết đến vị trí "root" mà đã được cấu hình cho disk:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

<a name="failed-writes"></a>
#### Failed Writes

Nếu phương thức `put` (hoặc các thao tác "ghi" khác) không thể ghi file vào disk, thì `false` sẽ được trả về:

    if (! Storage::put('file.jpg', $contents)) {
        // The file could not be written to disk...
    }

Nếu muốn, bạn có thể định nghĩa một tùy chọn `throw` trong mảng cấu hình của filesystem disk của bạn. Khi tùy chọn đó đã được định nghĩa là `true`, các phương thức "write" chẳng hạn như `put` sẽ throw ra một instance của `League\Flysystem\UnableToWriteFile` khi thao tác ghi không thành công:

    'public' => [
        'driver' => 'local',
        // ...
        'throw' => true,
    ],

<a name="prepending-appending-to-files"></a>
### Ghi vào đầu hoặc cuối file

Các phương thức `prepend` và `append` cho phép bạn ghi vào đầu hoặc cuối file:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

<a name="copying-moving-files"></a>
### Copy và di chuyển Files

Phương thức `copy` có thể được sử dụng để sao chép một file hiện có sang một vị trí mới trên disk, trong khi phương thức `move` có thể được sử dụng để đổi tên hoặc di chuyển file hiện có sang vị trí mới:

    Storage::copy('old/file.jpg', 'new/file.jpg');

    Storage::move('old/file.jpg', 'new/file.jpg');

<a name="automatic-streaming"></a>
### Automatic Streaming

Streaming một file đến storage giúp giảm đáng kể mức sử dụng bộ nhớ. Nếu bạn muốn Laravel tự động quản lý việc streaming một file đã cho đến một vị trí lưu trữ của bạn, bạn có thể sử dụng phương thức `putFile` hoặc `putFileAs`. Phương thức này chấp nhận một instance `Illuminate\Http\File` hoặc một `Illuminate\Http\UploadedFile` và sẽ tự động stream file đó đến vị trí mong muốn của bạn:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // Automatically generate a unique ID for filename...
    $path = Storage::putFile('photos', new File('/path/to/photo'));

    // Manually specify a filename...
    $path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

Có một vài điều quan trọng cần phải lưu ý về phương thức `putFile`. Hãy lưu ý rằng chúng ta chỉ khai báo đến của tên thư mục và không phải khai báo đến tên của file. Mặc định, phương thức `putFile` sẽ tạo một unique ID để làm tên file. Phần đuôi mở rộng của file sẽ được xác định bằng cách kiểm tra kiểu MIME của file. Đường dẫn đến file sẽ được trả về bởi phương thức `putFile` để bạn có thể lưu trữ đường dẫn đó vào trong cơ sở dữ liệu của bạn, đường dẫn này cũng chứa cả tên file đã được tạo.

Các phương thức `putFile` và `putFileAs` cũng chấp nhận một than số để khai báo "visibility" của file được lưu trữ. Điều này đặc biệt hữu ích nếu bạn đang lưu trữ file trên một cloud disk như Amazon S3 và muốn file này có thể truy cập công khai thông qua URL được generate :

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

<a name="file-uploads"></a>
### File Uploads

Trong các application web, một trong những trường hợp hay sử dụng nhất cho lưu trữ file là lưu trữ các file được upload từ người dùng như photo và document. Laravel giúp lưu trữ dễ dàng các file được upload bằng cách sử dụng phương thức `store` trên một instance file upload. Gọi phương thức `store` với đường dẫn mà bạn muốn lưu trữ file vào:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UserAvatarController extends Controller
    {
        /**
         * Update the avatar for the user.
         */
        public function update(Request $request): string
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

Có một vài điều quan trọng cần lưu ý về ví dụ này. Hãy lưu ý rằng chúng ta chỉ khai báo đến tên thư mục, không phải đến tên file. Mặc định, phương thức `store` sẽ tạo một unique ID để làm tên file. Phần đuôi mở rộng của file sẽ được xác định bằng cách kiểm tra kiểu MIME của file. Đường dẫn đến file sẽ được trả về từ phương thức `store`, để bạn có thể lưu trữ đường dẫn đó vào trong cơ sở dữ liệu của bạn, đường dẫn này cũng chứa cả tên file đã được tạo.

Bạn cũng có thể gọi phương thức `putFile` trên facade `Storage` để thực hiện thao tác lưu trữ file giống như ví dụ trên:

    $path = Storage::putFile('avatars', $request->file('avatar'));

<a name="specifying-a-file-name"></a>
#### Specifying A File Name

Nếu bạn không muốn tên file được tự động gán cho file, bạn có thể sử dụng phương thức `storeAs`, nhận vào một đường dẫn, một tên file và một tên disk (tùy chọn) làm tham số của nó:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

Bạn cũng có thể sử dụng phương thức `putFileAs` trên facade `Storage`, để thực hiện thao tác lưu trữ file giống như ví dụ trên:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

> [!WARNING]
> Các ký tự unicode không in được hoặc không hợp lệ sẽ bị tự động xóa khỏi đường dẫn đến file. Vì vậy, bạn có thể muốn làm sạch đường dẫn đến file của bạn trước khi truyền chúng đến các phương thức lưu trữ file của Laravel. Đường dẫn đến file có thể được chuẩn hóa bằng phương thức `League\Flysystem\WhitespacePathNormalizer::normalizePath`.

<a name="specifying-a-disk"></a>
#### Specifying A Disk

Mặc định, phương thức `store` của file được upload sẽ sử dụng disk mặc định. Nếu bạn muốn chỉ định một disk khác, hãy truyền tên disk làm tham số thứ hai cho phương thức `store`:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

Nếu bạn đang sử dụng phương thức `storeAs`, bạn có thể truyền tên disk làm tham số thứ ba cho phương thức:

    $path = $request->file('avatar')->storeAs(
        'avatars',
        $request->user()->id,
        's3'
    );

<a name="other-uploaded-file-information"></a>
#### Other Uploaded File Information

Nếu bạn muốn lấy tên gốc và phần mở rộng của một file đã được tải lên, bạn có thể làm như sau bằng cách sử dụng các phương thức `getClientOriginalName` và `getClientOriginalExtension`:

    $file = $request->file('avatar');

    $name = $file->getClientOriginalName();
    $extension = $file->getClientOriginalExtension();

Tuy nhiên, hãy nhớ rằng các phương thức `getClientOriginalName` và `getClientOriginalExtension` sẽ được coi là không an toàn vì tên file và phần mở rộng có thể bị người dùng ác ý giả mạo. Vì lý do này, bạn nên sử dụng các phương thức `hashName` và `extension` để lấy tên và phần mở rộng cho file đã được tải lên:

    $file = $request->file('avatar');

    $name = $file->hashName(); // Generate a unique, random name...
    $extension = $file->extension(); // Determine the file's extension based on the file's MIME type...

<a name="file-visibility"></a>
### File Visibility

Trong Flysystem integration của Laravel, "visibility" là một trừu tượng về các quyền của file trên nhiều nền tảng. Các file có thể được khai báo `public` hoặc `private`. Khi một file được khai báo `public`, bạn đã cho phép file đó có thể truy cập được từ người dùng khác. Ví dụ: khi sử dụng driver S3, bạn có thể lấy được URL cho các file `public`.

Bạn có thể set visibility khi viết file thông qua phương thức `put`:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

Nếu file đã được lưu trữ, visibility của nó có thể được lấy ra và thiết lập thông qua các phương thức `getVisibility` và `setVisibility`:

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public');

Khi tương tác với các file được upload, bạn có thể sử dụng các phương thức `storePublicly` hoặc `storePubliclyAs` để lưu trữ các file đã được upload với chế độ `public`:

    $path = $request->file('avatar')->storePublicly('avatars', 's3');

    $path = $request->file('avatar')->storePubliclyAs(
        'avatars',
        $request->user()->id,
        's3'
    );

<a name="local-files-and-visibility"></a>
#### Local Files và Visibility

Khi sử dụng driver `local`, thư mục `public` [visibility](#file-visibility) sẽ được chuyển thành `0755` cho thư mục và `0644` cho file. Bạn có thể sửa các quyền này trong file cấu hình `filesystems` của bạn:

    'local' => [
        'driver' => 'local',
        'root' => storage_path('app'),
        'permissions' => [
            'file' => [
                'public' => 0644,
                'private' => 0600,
            ],
            'dir' => [
                'public' => 0755,
                'private' => 0700,
            ],
        ],
    ],

<a name="deleting-files"></a>
## Xoá File

Phương thức `delete` chấp nhận một tên file hoặc một mảng các tên file để xóa:

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file.jpg', 'file2.jpg']);

Nếu cần, bạn có thể khai báo disk mà file đó sẽ bị xóa:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('path/file.jpg');

<a name="directories"></a>
## Thư mục

<a name="get-all-files-within-a-directory"></a>
#### Get All Files Within A Directory

Phương thức `files` sẽ trả về một mảng của tất cả các file có trong một thư mục nhất định. Nếu bạn muốn lấy danh sách tất cả các file có trong một thư mục bao gồm cả các thư mục con, bạn có thể sử dụng phương thức `allFiles`:

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

<a name="get-all-directories-within-a-directory"></a>
#### Get All Directories Within A Directory

Phương thức `directories` trả về một mảng gồm tất cả các thư mục có trong một thư mục đã cho. Ngoài ra, bạn có thể sử dụng phương thức `allDirectories` để lấy danh sách tất cả các thư mục có trong một thư mục đã cho và cả các thư mục con của nó:

    $directories = Storage::directories($directory);

    $directories = Storage::allDirectories($directory);

<a name="create-a-directory"></a>
#### Create A Directory

Phương thức `makeDirectory` sẽ tạo mới một thư mục, bao gồm cả thư mục con nếu cần thiết:

    Storage::makeDirectory($directory);

<a name="delete-a-directory"></a>
#### Delete A Directory

Cuối cùng, phương thức `deleteDirectory` có thể được sử dụng để xóa một thư mục và tất cả các file trong nó:

    Storage::deleteDirectory($directory);

<a name="testing"></a>
## Testing

Phương thức `fake` của facade `Storage` cho phép bạn dễ dàng tạo ra một disk giả, kết hợp với các tiện ích tạo file của class `Illuminate\Http\UploadedFile`, giúp bạn đơn giản hóa đáng kể việc kiểm tra các file upload. Ví dụ:

    <?php

    namespace Tests\Feature;

    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_albums_can_be_uploaded(): void
        {
            Storage::fake('photos');

            $response = $this->json('POST', '/photos', [
                UploadedFile::fake()->image('photo1.jpg'),
                UploadedFile::fake()->image('photo2.jpg')
            ]);

            // Assert one or more files were stored...
            Storage::disk('photos')->assertExists('photo1.jpg');
            Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

            // Assert one or more files were not stored...
            Storage::disk('photos')->assertMissing('missing.jpg');
            Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

            // Assert that a given directory is empty...
            Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
        }
    }

Mặc định, phương thức `fake` sẽ xóa tất cả các file có trong thư mục tạm thời của nó. Nếu bạn muốn giữ lại các file này, bạn có thể sử dụng phương thức "persistentFake" thay thế. Để biết thêm thông tin về việc thử nghiệm file upload, bạn có thể tham khảo [thông tin về file upload của tài liệu HTTP testing](/docs/{{version}}/http-tests#testing-file-uploads).

> [!WARNING]
> Phương thức `image` yêu cầu [GD extension](https://www.php.net/manual/en/book.image.php).

<a name="custom-filesystems"></a>
## Tuỳ chỉnh Filesystem

Flysystem tích hợp của Laravel cung cấp hỗ trợ cho một số "driver" mặc đinh; tuy nhiên, Flysystem không chỉ giới hạn ở những điều này mà còn có bộ chuyển đổi cho nhiều hệ thống lưu trữ khác. Bạn có thể tạo driver tùy biến nếu bạn muốn sử dụng một trong những bộ chuyển đổi đó vào trong ứng dụng Laravel của bạn.

Để định nghĩa một tuỳ biến filesystem, bạn sẽ cần một bộ chuyển đổi Flysystem. Hãy thêm một bộ chuyển đổi Dropbox được cộng đồng phát triển vào trong dự án của bạn:

```shell
composer require spatie/flysystem-dropbox
```

Tiếp theo, bạn có thể đăng ký driver trong phương thức `boot` của một trong những [service providers](/docs/{{version}}/providers) trong ứng dụng của bạn. Để thực hiện điều này, bạn nên sử dụng phương thức `extend` của facade `Storage`:

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Filesystem\FilesystemAdapter;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Filesystem;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Storage::extend('dropbox', function (Application $app, array $config) {
                $adapter = new DropboxAdapter(new DropboxClient(
                    $config['authorization_token']
                ));

                return new FilesystemAdapter(
                    new Filesystem($adapter, $config),
                    $adapter,
                    $config
                );
            });
        }
    }

Tham số đầu tiên của phương thức `extend` là tên của driver và tham số thứ hai là một closure nhận các biến `$app` và `$config`. Closure này cần trả về một instance của `Illuminate\Filesystem\FilesystemAdapter`. Biến `$config` sẽ chứa các giá trị được định nghĩa trong file `config/filesystems.php` cho disk mà bạn đang khai báo.

Khi bạn đã tạo và đăng ký xong service provider, bạn có thể sử dụng driver `dropbox` trong file cấu hình `config/filesystems.php`.
