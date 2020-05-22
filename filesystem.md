# File Storage

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Public Disk](#the-public-disk)
    - [Local Driver](#the-local-driver)
    - [Yêu cầu Driver](#driver-prerequisites)
- [Lấy Disk Instance](#obtaining-disk-instances)
- [Lấy File](#retrieving-files)
    - [File URL](#file-urls)
    - [File Metadata](#file-metadata)
- [Lưu File](#storing-files)
    - [File Uploads](#file-uploads)
    - [File Visibility](#file-visibility)
- [Xoá File](#deleting-files)
- [Thư mục](#directories)
- [Tuỳ chỉnh Filesystem](#custom-filesystems)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một filesystem abstraction mạnh mẽ nhờ package PHP [Flysystem](https://github.com/thephpleague/flysystem) tuyệt vời của Frank de Jonge. Laravel Flysystem integration cung cấp các driver đơn giản để sử dụng để làm việc với các local filesystems, Amazon S3 và Rackspace Cloud Storage. Thậm chí, nó thật đơn giản để chuyển đổi giữa các tùy chọn lưu trữ này vì API vẫn giống nhau cho từng hệ thống.

<a name="configuration"></a>
## Cấu hình

File cấu hình filesystem được đặt tại `config/filesystems.php`. Trong file này, bạn có thể cấu hình tất cả các "disks" của bạn. Mỗi disk đại diện cho một driver lưu trữ và vị trí lưu trữ cụ thể. Các cấu hình mẫu cho mỗi driver được hỗ trợ đã được khai báo sẵn trong file cấu hình. Vì vậy, bạn có thể sửa cấu hình để đúng với tuỳ chọn lưu trữ của bạn và thông tin của chúng.

Tất nhiên, bạn có thể cấu hình bao nhiêu disk tùy thích và thậm chí có thể có nhiều disk sử dụng cùng một driver.

<a name="the-public-disk"></a>
### Public Disk

`public` disk dành cho các file sẽ có thể truy cập công khai. Mặc định, disk `public` sử dụng driver `local` và lưu trữ các file này trong `storage/app/public`. Để làm cho chúng có thể truy cập từ web, bạn nên tạo một liên kết ảo từ `public/storage` đến `storage/app/public`. Quy ước này sẽ giữ các file có thể truy cập công khai của bạn trong một thư mục có thể dễ dàng chia sẻ qua các mỗi lần deploy khi sử dụng các hệ thống deploy zero down-time như [Envoyer](https://envoyer.io).

Để tạo liên kết ảo, bạn có thể sử dụng lệnh Artisan `storage:link`:

    php artisan storage:link

Tất nhiên, một khi một file đã được lưu trữ và liên kết ảo đã được tạo, bạn có thể tạo một URL tới các file bằng cách sử dụng helper `asset`:

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### Local Driver

Khi sử dụng driver `local`, tất cả các file operation đều liên quan đến thư mục `root` được định nghĩa trong file cấu hình của bạn. Mặc định, giá trị này được set là thư mục `storage/app`. Vì thế, phương thức sau đây sẽ lưu trữ một file trong `storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### Yêu cầu Driver

#### Composer Packages

Trước khi sử dụng driver S3 hoặc Rackspace, bạn sẽ cần cài đặt package thích hợp thông qua Composer:

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### S3 Driver Configuration

Thông tin cấu hình driver S3 nằm trong file cấu hình `config/filesystems.php` của bạn. File này chứa một mảng cấu hình mẫu cho driver S3. Bạn có thể tự do sửa mảng này với thông tin và cấu hình S3 của riêng bạn. Để thuận tiện, các biến môi trường khớp với quy ước đặt tên được sử dụng bởi AWS CLI.

#### FTP Driver Configuration

Flysystem integration của Laravel hoạt động tốt với FTP; tuy nhiên, một cấu hình mẫu mặc định không được thêm vào trong file cấu hình `filesystems.php` của framework. Nếu bạn cần cấu hình hệ thống file FTP, bạn có thể sử dụng cấu hình mẫu bên dưới:

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Rackspace Driver Configuration

Flysystem integration của Laravel hoạt động tốt với Rackspace; tuy nhiên, một cấu hình mẫu mặc định không được thêm vào trong file cấu hình `filesystems.php` của framework. Nếu bạn cần cấu hình hệ thống file Rackspace, bạn có thể sử dụng cấu hình mẫu bên dưới:

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="obtaining-disk-instances"></a>
## Lấy Disk Instance

Facade `Storage` có thể được sử dụng để tương tác với bất kỳ disk nào được cấu hình của bạn. Ví dụ, bạn có thể sử dụng phương thức `put` trên facade để lưu trữ hình đại diện trên disk mặc định. Nếu bạn gọi các phương thức trên facade `Storage` mà không khai báo phương thức` disk`, thì câu lệnh sẽ tự động được chuyển sang disk mặc định:

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

Nếu application của bạn tương tác với nhiều disk, bạn có thể sử dụng phương thức `disk` trên facade `Storage` để làm việc với các file trên một disk cụ thể:

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## Lấy File

Phương thức `get` có thể được sử dụng để lấy nội dung của file. Một chuỗi raw nội dung của file sẽ được phương thức trả về. Hãy nhớ rằng, tất cả các đường dẫn file phải được khai báo đều phải liên kết đến vị trí "root" được cấu hình cho disk:

    $contents = Storage::get('file.jpg');

Phương thức `exists` có thể được sử dụng để xác định xem một file có tồn tại trên disk hay không:

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="file-urls"></a>
### File URL

Bạn có thể sử dụng phương thức `url` để lấy URL cho một file đã cho. Nếu bạn đang sử dụng driver `local`, điều này thường sẽ chỉ cần thêm `/storage` vào đường dẫn đã cho và trả về một URL tương đối cho file. Nếu bạn đang sử dụng driver `s3` hoặc `rackspace`, remote URL sẽ được trả về:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file1.jpg');

> {note} Hãy nhớ rằng, nếu bạn đang sử dụng driver `local`, tất cả các file có thể truy cập công khai nên được đặt trong thư mục `storage/app/public`. Hơn nữa, bạn nên [tạo một liên kết ảo](#the-public-disk) ở `public/storage` trỏ đến thư mục `storage/app/public`.

#### Temporary URLs

Đối với các file đã được lưu trữ bằng driver `s3` hoặc `rackspace`, bạn có thể tạo một URL tạm thời cho một file đã cho bằng cách sử dụng phương thức `temporaryUrl`. Phương thức này chấp nhận một đường dẫn và một instance `DateTime` xác định khi URL sẽ hết hạn:

    $url = Storage::temporaryUrl(
        'file1.jpg', now()->addMinutes(5)
    );

#### Local URL Host Customization

Nếu như bạn muốn xác định host trước, cho các file được lưu trữ trên một disk đang dùng driver `local`, bạn có thể thêm tùy chọn `url` vào mảng cấu hình của disk:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### File Metadata

Ngoài việc đọc và ghi file, Laravel cũng có thể cung cấp thông tin về các file đó. Ví dụ, phương thức `size` có thể được sử dụng để lấy kích thước của file theo byte:

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file1.jpg');

Phương thức `lastModified` trả về UNIX timestamp về lần cuối cùng file được sửa:

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
## Lưu File

Phương thức `put` có thể được sử dụng để lưu trữ nội dung raw của file lên disk. Bạn cũng có thể pass một PHP `resource` đến phương thức `put`, phương thức này sẽ sử dụng stream support của Flysystem. Sử dụng stream rất được khuyến khích khi xử lý các file lớn:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### Automatic Streaming

Nếu bạn muốn Laravel tự động quản lý việc streaming một file đã cho đến vị trí lưu trữ của bạn, bạn có thể sử dụng phương thức `putFile` hoặc `putFileAs`. Phương thức này chấp nhận một instance `Illuminate\Http\File` hoặc `Illuminate\Http\UploadedFile` và sẽ tự động stream file đến vị trí mong muốn của bạn:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // Automatically generate a unique ID for file name...
    Storage::putFile('photos', new File('/path/to/photo'));

    // Manually specify a file name...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

Có một vài điều quan trọng cần lưu ý về phương thức `putFile`. Hãy lưu ý rằng chúng ta chỉ khai báo đến tên thư mục, không phải đến tên file. Mặc định, phương thức `putFile` sẽ tạo một unique ID để làm tên file. Đường dẫn đến file sẽ được trả về bởi phương thức `putFile` để bạn có thể lưu trữ đường dẫn vào trong cơ sở dữ liệu của bạn, đường dẫn đó cũng chứa cả tên file đã được tạo.

Các phương thức `putFile` và `putFileAs` cũng chấp nhận một than số để khai báo "visibility" của file được lưu trữ. Điều này đặc biệt hữu ích nếu bạn đang lưu trữ file trên một cloud disk như S3 và muốn file này có thể truy cập công khai:

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### Prepending & Appending To Files

Các phương thức `prepend` và `append` cho phép bạn ghi vào đầu hoặc cuối file:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### Copying & Moving Files

Phương thức `copy` có thể được sử dụng để sao chép một file hiện có sang một vị trí mới trên disk, trong khi phương thức `move` có thể được sử dụng để đổi tên hoặc di chuyển một file hiện có sang một vị trí mới:

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

    Storage::move('old/file1.jpg', 'new/file1.jpg');

<a name="file-uploads"></a>
### File Uploads

Trong các application web, một trong những trường hợp sử dụng hay sử dụng nhất cho lưu trữ file là lưu trữ các uploaded file của người dùng như profile picture, photo và document. Laravel giúp lưu trữ dễ dàng các uploaded file bằng phương thức `store` trên một instance uploaded file. Gọi phương thức `store` với đường dẫn mà bạn muốn lưu trữ uploaded file:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserAvatarController extends Controller
    {
        /**
         * Update the avatar for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

Có một vài điều quan trọng cần lưu ý về ví dụ này. Hãy lưu ý rằng chúng ta chỉ khai báo đến tên thư mục, không phải đến tên file. Mặc định, phương thức `store` sẽ tạo một unique ID để làm tên file. Đường dẫn đến file sẽ được trả về bởi phương thức `store` để bạn có thể lưu trữ đường dẫn vào trong cơ sở dữ liệu của bạn, đường dẫn đó cũng chứa cả tên file đã được tạo.

Bạn cũng có thể gọi phương thức `putFile` trên facade `Storage` để thực hiện thao tác với file giống như ví dụ trên:

    $path = Storage::putFile('avatars', $request->file('avatar'));

#### Specifying A File Name

Nếu bạn không muốn tên file được tự động gán cho file lưu trữ của bạn, bạn có thể sử dụng phương thức `storeAs`, nhận một đường dẫn, một tên file và tên disk (tùy chọn) làm tham số của nó:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

Tất nhiên, bạn cũng có thể sử dụng phương thức `putFileAs` trên facade `Storage`, sẽ thực hiện thao tác với file tương tự như ví dụ trên:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### Specifying A Disk

Mặc định, phương pháp này sẽ sử dụng disk mặc định của bạn. Nếu bạn muốn chỉ định một disk khác, hãy pass tên disk làm tham số thứ hai cho phương thức `store`:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### File Visibility

Trong Flysystem integration của Laravel, "visibility" là một trừu tượng về các quyền của file trên nhiều nền tảng. Các file có thể được khai báo `public` hoặc `private`. Khi một file được khai báo `public`, bạn đang cho phép file đó có thể truy cập được từ người khác. Ví dụ: khi sử dụng driver S3, bạn có thể lấy được URL cho các file `public`.

Bạn có thể set visibility khi cài đặt file thông qua phương thức `put`:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

Nếu file đã được lưu trữ, visibility của nó có thể được lấy ra và thiết lập thông qua các phương thức `getVisibility` và `setVisibility`:

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## Xoá File

Phương thức `delete` chấp nhận một tên file hoặc một mảng các tên file để xóa khỏi disk:

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

Nếu cần, bạn có thể khai báo disk mà file có sẽ bị xóa:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');

<a name="directories"></a>
## Thư mục

#### Get All Files Within A Directory

Phương thức `files` trả về một mảng của tất cả các file có trong một thư mục nhất định. Nếu bạn muốn lấy danh sách tất cả các file có trong một thư mục bao gồm tất cả các thư mục con, bạn có thể sử dụng phương thức `allFiles`:

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### Get All Directories Within A Directory

Phương thức `directories` trả về một mảng của tất cả các thư mục có trong một thư mục đã cho. Ngoài ra, bạn có thể sử dụng phương thức `allDirectories` để lấy danh sách tất cả các thư mục có trong một thư mục đã cho và tất cả các thư mục con của nó:

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

#### Create A Directory

Phương thức `makeDirectory` sẽ tạo mới thư mục, bao gồm mọi thư mục con cần thiết:

    Storage::makeDirectory($directory);

#### Delete A Directory

Cuối cùng, `deleteDirectory` có thể được sử dụng để xóa một thư mục và tất cả các file của nó:

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## Tuỳ chỉnh Filesystem

Flysystem integrationcủa Laravel cung cấp nhiều driver cho một số "driver" mặc đinh; tuy nhiên, Flysystem không chỉ giới hạn ở những điều này mà còn có bộ chuyển đổi cho nhiều hệ thống lưu trữ khác. Bạn có thể tạo driver tùy chỉnh nếu bạn muốn sử dụng một trong những bộ chuyển đổi bổ sung này vào trong ứng dụng Laravel của bạn.

Để thiết lập custom filesystem, bạn sẽ cần một bộ chuyển đổi Flysystem. Hãy thêm một bộ chuyển đổi Dropbox được cộng đồng phát triển vào dự án của chúng ta:

    composer require spatie/flysystem-dropbox

Tiếp theo, bạn nên tạo một [service provider](/docs/{{version}}/providers), chẳng hạn như `DropboxServiceProvider`. Trong phương thức `boot` của provider, bạn có thể sử dụng phương thức `extend` của facade `Storage` để định nghĩa custom driver:

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Illuminate\Support\ServiceProvider;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorizationToken']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Tham số đầu tiên của phương thức `extend` là tên của driver và tham số thứ hai là một Closure nhận các biến `$app` và `$config`. Resolver Closure cần phải trả về một instance của `League\Flysystem\Filesystem`. Biến `$config` chứa các giá trị được định nghĩa trong file `config/filesystems.php` cho disk được khai báo.

Khi bạn đã tạo service provider để đăng ký extension, bạn có thể sử dụng driver `dropbox` trong file cấu hình `config/filesystems.php`.
