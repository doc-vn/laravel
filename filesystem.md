# File Storage

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Public Disk](#the-public-disk)
    - [Local Driver](#the-local-driver)
    - [Yêu cầu Driver](#driver-prerequisites)
    - [Caching](#caching)
- [Lấy Disk Instance](#obtaining-disk-instances)
- [Lấy File](#retrieving-files)
    - [Tải File](#downloading-files)
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

Laravel cung cấp một abstraction filesystem mạnh mẽ nhờ package PHP [Flysystem](https://github.com/thephpleague/flysystem) tuyệt vời của Frank de Jonge. Laravel Flysystem integration cung cấp các driver đơn giản để sử dụng và làm việc với các local filesystems và Amazon S3. Thậm chí, nó cũng rất đơn giản để chuyển đổi giữa các tùy chọn lưu trữ này vì API vẫn giống nhau cho mỗi hệ thống.

<a name="configuration"></a>
## Cấu hình

File cấu hình của filesystem được lưu tại `config/filesystems.php`. Trong file này, bạn có thể cấu hình tất cả các "disks" của bạn. Mỗi disk sẽ được đại diện cho một driver lưu trữ với một vị trí lưu trữ cụ thể. Các cấu hình mẫu cho các driver được hỗ trợ cũng đã được khai báo sẵn vào trong file cấu hình. Vì vậy, bạn có thể sửa cấu hình để đúng với tuỳ chọn lưu trữ của bạn và thông tin của chúng.

Bạn có thể cấu hình bao nhiêu disk tùy ý của bạn và thậm chí có thể có nhiều disk sử dụng cùng một driver.

<a name="the-public-disk"></a>
### Public Disk

`public` disk dành cho các file có thể truy cập ở dạng công khai. Mặc định, `public` disk sẽ sử dụng driver `local` và lưu trữ các file này trong `storage/app/public`. Để làm cho các file này có thể truy cập từ web, bạn nên tạo một link liên kết ảo từ `public/storage` đến `storage/app/public`. Quy ước này sẽ giúp cho các file có thể truy cập công khai của bạn ở trong một thư mục có thể dễ dàng chia sẻ qua mỗi lần deploy khi sử dụng các hệ thống deploy zero down-time như [Envoyer](https://envoyer.io).

Để tạo link liên kết ảo, bạn có thể sử dụng lệnh Artisan `storage:link`:

    php artisan storage:link

Một khi một file đã được lưu trữ và link liên kết ảo đã được tạo xong, bạn có thể tạo URL tới các file này bằng cách sử dụng helper `asset`:

    echo asset('storage/file.txt');

Bạn có thể cấu hình thêm các link ảo trong file cấu hình `filesystems` của bạn. Mỗi link được cấu hình sẽ được tạo khi bạn chạy lệnh `storage:link`:

    'links' => [
        public_path('storage') => storage_path('app/public'),
        public_path('images') => storage_path('app/images'),
    ],

<a name="the-local-driver"></a>
### Local Driver

Khi sử dụng driver `local`, tất cả các hoạt động của các file đều liên quan đến thư mục `root` sẽ được định nghĩa trong file cấu hình `filesystems` của bạn. Mặc định, giá trị này sẽ được đặt là thư mục `storage/app`. Vì thế, phương thức sau đây sẽ lưu trữ một file vào trong `storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');

#### Permissions

Thư mục `public` [visibility](#file-visibility) sẽ được chuyển thành `0755` cho thư mục và `0644` cho file. Bạn có thể sửa các quyền này trong file cấu hình `filesystems` của bạn:

    'local' => [
        'driver' => 'local',
        'root' => storage_path('app'),
        'permissions' => [
            'file' => [
                'public' => 0664,
                'private' => 0600,
            ],
            'dir' => [
                'public' => 0775,
                'private' => 0700,
            ],
        ],
    ],

<a name="driver-prerequisites"></a>
### Yêu cầu Driver

#### Composer Packages

Trước khi sử dụng driver SFTP hoặc S3, bạn sẽ cần cài đặt các package thích hợp thông qua Composer:

- SFTP: `league/flysystem-sftp ~1.0`
- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`

Để tăng hiệu suất, bạn cần phải dùng một cached adapter. Bạn có thể thêm một package cho việc này:

- CachedAdapter: `league/flysystem-cached-adapter ~1.0`

#### S3 Driver Configuration

Thông tin cấu hình driver S3 nằm trong file cấu hình `config/filesystems.php` của bạn. File này chứa một mảng cấu hình mẫu cho driver S3. Bạn có thể tự do sửa mảng này với thông tin và cấu hình S3 của riêng bạn. Để thuận tiện, các biến môi trường đã được đặt tên khớp với quy ước đặt tên được sử dụng bởi AWS CLI.

#### FTP Driver Configuration

Flysystem integration của Laravel hoạt động tốt với FTP; tuy nhiên, mặc định, một cấu hình mẫu không được thêm vào trong file cấu hình `filesystems.php` của framework. Nếu bạn cần cấu hình một hệ thống file FTP, bạn có thể sử dụng cấu hình mẫu ở bên dưới:

    'ftp' => [
        'driver' => 'ftp',
        'host' => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port' => 21,
        // 'root' => '',
        // 'passive' => true,
        // 'ssl' => true,
        // 'timeout' => 30,
    ],

#### SFTP Driver Configuration

Flysystem tích hợp trong Laravel hoạt động tốt với SFTP; tuy nhiên, mặc định một cấu hình mẫu sẽ không có trong file cấu hình `filesystems.php` của framework. Nếu bạn cần cấu hình một hệ thống filesystem SFTP, bạn có thể sử dụng cấu hình ví dụ ở bên dưới:

    'sftp' => [
        'driver' => 'sftp',
        'host' => 'example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Settings for SSH key based authentication...
        // 'privateKey' => '/path/to/privateKey',
        // 'password' => 'encryption-password',

        // Optional SFTP Settings...
        // 'port' => 22,
        // 'root' => '',
        // 'timeout' => 30,
    ],

<a name="caching"></a>
### Caching

Để kích hoạt bộ nhớ cache cho một disk nhất định, bạn có thể thêm tuỳ chọn `cache` vào các tùy chọn cấu hình của disk. Tùy chọn `cache` sẽ phải là một mảng gồm các tùy chọn là tên `disk`, thời gian hết hạn `expire` tính bằng giây và tiền tố `prefix`:

    's3' => [
        'driver' => 's3',

        // Other Disk Options...

        'cache' => [
            'store' => 'memcached',
            'expire' => 600,
            'prefix' => 'cache-prefix',
        ],
    ],

<a name="obtaining-disk-instances"></a>
## Lấy Disk Instance

Facade `Storage` có thể được sử dụng để tương tác với bất kỳ disk nào mà bạn cấu hình. Ví dụ, bạn có thể sử dụng phương thức `put` trong facade này để lưu trữ hình đại diện cho một disk mà bạn muốn. Nếu bạn gọi các phương thức này trong facade `Storage` mà không khai báo thêm phương thức `disk`, thì câu lệnh này sẽ tự động được chuyển file đó đến disk mặc định:

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

Nếu application của bạn tương tác với nhiều disk, thì bạn có thể sử dụng phương thức `disk` trên facade `Storage` để làm việc với các file cho một disk cụ thể:

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## Lấy File

Phương thức `get` có thể được sử dụng để lấy nội dung của file. Một chuỗi raw của nội dung file sẽ được phương thức trả về. Hãy nhớ rằng, tất cả các đường dẫn đến file phải được khai báo liên kết đến vị trí "root" mà đã được cấu hình cho disk:

    $contents = Storage::get('file.jpg');

Phương thức `exists` có thể được sử dụng để xác định xem một file có tồn tại trên disk hay không:

    $exists = Storage::disk('s3')->exists('file.jpg');

Phương thức `missing` có thể được sử dụng để xác định xem file có bị thiếu trong disk hay không:

    $missing = Storage::disk('s3')->missing('file.jpg');

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

> {note} Hãy nhớ rằng, nếu bạn đang sử dụng driver `local`, tất cả các file mà có thể truy cập ở dạng công khai thì nên được lưu trong thư mục `storage/app/public`. Hơn nữa, bạn nên [tạo một link liên kết ảo](#the-public-disk) ở thư mục `public/storage` để trỏ đến thư mục `storage/app/public`.

#### Temporary URLs

Đối với các file đã được lưu trữ bằng driver `s3`, bạn có thể tạo một URL tạm thời cho một file bằng cách sử dụng phương thức `temporaryUrl`. Phương thức này chấp nhận một đường dẫn và một instance `DateTime` để định nghĩa khi URL sẽ hết hạn:

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

Nếu bạn cần chỉ định thêm một [S3 request parameters](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests), bạn có thể truyền một mảng request parameter làm tham số thứ ba cho phương thức `temporaryUrl`:

    $url = Storage::temporaryUrl(
        'file.jpg',
        now()->addMinutes(5),
        ['ResponseContentType' => 'application/octet-stream']
    );

#### URL Host Customization

Nếu như bạn muốn định nghĩa thêm host cho các URL mà được tạo ra khi đang dùng facade `Storage`, bạn có thể thêm tùy chọn `url` vào mảng cấu hình của disk:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### File Metadata

Ngoài việc đọc và ghi file, Laravel cũng cung cấp thông tin về các file đó. Ví dụ, phương thức `size` có thể được sử dụng để lấy ra kích thước của file theo đơn vị byte:

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

Phương thức `lastModified` trả về một UNIX timestamp về lần cuối cùng mà file được sửa:

    $time = Storage::lastModified('file.jpg');

<a name="storing-files"></a>
## Lưu File

Phương thức `put` có thể được sử dụng để lưu trữ một nội dung raw của file lên disk. Bạn cũng có thể truyền một PHP `resource` đến phương thức `put`, phương thức này sẽ sử dụng support stream của Flysystem. Hãy nhớ rằng, tất cả các đường dẫn đến file phải được khai báo liên kết đến vị trí "root" mà đã được cấu hình cho disk:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### Automatic Streaming

Nếu bạn muốn Laravel tự động quản lý việc streaming một file đã cho đến một vị trí lưu trữ của bạn, bạn có thể sử dụng phương thức `putFile` hoặc `putFileAs`. Phương thức này chấp nhận một instance `Illuminate\Http\File` hoặc một `Illuminate\Http\UploadedFile` và sẽ tự động stream file đó đến vị trí mong muốn của bạn:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // Automatically generate a unique ID for file name...
    Storage::putFile('photos', new File('/path/to/photo'));

    // Manually specify a file name...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

Có một vài điều quan trọng cần phải lưu ý về phương thức `putFile`. Hãy lưu ý rằng chúng ta chỉ khai báo đến của tên thư mục, không phải khai báo đến tên của file. Mặc định, phương thức `putFile` sẽ tạo một unique ID để làm tên file. Phần đuôi mở rộng của file sẽ được xác định bằng cách kiểm tra kiểu MIME của file. Đường dẫn đến file sẽ được trả về bởi phương thức `putFile` để bạn có thể lưu trữ đường dẫn đó vào trong cơ sở dữ liệu của bạn, đường dẫn này cũng chứa cả tên file đã được tạo.

Các phương thức `putFile` và `putFileAs` cũng chấp nhận một than số để khai báo "visibility" của file được lưu trữ. Điều này đặc biệt hữu ích nếu bạn đang lưu trữ file trên một cloud disk như S3 và muốn file này có thể truy cập công khai:

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### Prepending & Appending To Files

Các phương thức `prepend` và `append` cho phép bạn ghi vào đầu dòng hoặc cuối dòng của một file:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### Copying & Moving Files

Phương thức `copy` có thể được sử dụng để sao chép một file hiện có sang một vị trí mới trên disk, trong khi phương thức `move` có thể được sử dụng để đổi tên hoặc di chuyển một file hiện có sang một vị trí mới:

    Storage::copy('old/file.jpg', 'new/file.jpg');

    Storage::move('old/file.jpg', 'new/file.jpg');

<a name="file-uploads"></a>
### File Uploads

Trong các application web, một trong những trường hợp hay sử dụng nhất cho lưu trữ file là lưu trữ các file được upload từ người dùng như profile picture, photo và document. Laravel giúp lưu trữ dễ dàng các file được upload bằng cách sử dụng phương thức `store` trên một instance file upload. Gọi phương thức `store` với đường dẫn mà bạn muốn lưu trữ file vào:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

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

Có một vài điều quan trọng cần lưu ý về ví dụ này. Hãy lưu ý rằng chúng ta chỉ khai báo đến tên thư mục, không phải đến tên file. Mặc định, phương thức `store` sẽ tạo một unique ID để làm tên file. Phần đuôi mở rộng của file sẽ được xác định bằng cách kiểm tra kiểu MIME của file. Đường dẫn đến file sẽ được trả về từ phương thức `store`, để bạn có thể lưu trữ đường dẫn đó vào trong cơ sở dữ liệu của bạn, đường dẫn này cũng chứa cả tên file đã được tạo.

Bạn cũng có thể gọi phương thức `putFile` trên facade `Storage` để thực hiện thao tác với file giống như ví dụ trên:

    $path = Storage::putFile('avatars', $request->file('avatar'));

#### Specifying A File Name

Nếu bạn không muốn tên file được tự động gán cho file, bạn có thể sử dụng phương thức `storeAs`, nhận vào một đường dẫn, một tên file và một tên disk (tùy chọn) làm tham số của nó:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

Bạn cũng có thể sử dụng phương thức `putFileAs` trên facade `Storage`, sẽ thực hiện thao tác với file tương tự như ví dụ trên:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

> {note} Các ký tự unicode không in được hoặc không hợp lệ sẽ bị tự động xóa khỏi đường dẫn đến file. Vì vậy, bạn có thể muốn làm sạch đường dẫn đến file của bạn trước khi truyền chúng đến các phương thức lưu trữ file của Laravel. Đường dẫn đến file có thể được chuẩn hóa bằng phương thức `League\Flysystem\Util::normalizePath`.

#### Specifying A Disk

Mặc định, phương thức này sẽ sử dụng disk mặc định. Nếu bạn muốn chỉ định một disk khác, hãy truyền tên disk làm tham số thứ hai cho phương thức `store`:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

Nếu bạn đang sử dụng phương thức `storeAs`, bạn có thể truyền tên disk làm tham số thứ ba cho phương thức:

    $path = $request->file('avatar')->storeAs(
        'avatars',
        $request->user()->id,
        's3'
    );

#### Other File Information

Nếu bạn muốn lấy tên gốc của file đã được tải lên, bạn có thể làm như vậy bằng cách sử dụng phương thức `getClientOriginalName`:

    $name = $request->file('avatar')->getClientOriginalName();

Phương thức `extension` có thể được sử dụng để lấy phần mở rộng của file đã được tải lên:

    $extension = $request->file('avatar')->extension();

<a name="file-visibility"></a>
### File Visibility

Trong Flysystem integration của Laravel, "visibility" là một trừu tượng về các quyền của file trên nhiều nền tảng. Các file có thể được khai báo `public` hoặc `private`. Khi một file được khai báo `public`, bạn đã cho phép file đó có thể truy cập được từ người dùng khác. Ví dụ: khi sử dụng driver S3, bạn có thể lấy được URL cho các file `public`.

Bạn có thể set visibility khi cài đặt file thông qua phương thức `put`:

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

<a name="deleting-files"></a>
## Xoá File

Phương thức `delete` chấp nhận một tên file hoặc một mảng các tên file để xóa khỏi disk:

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file.jpg', 'file2.jpg']);

Nếu cần, bạn có thể khai báo disk mà file đó sẽ bị xóa:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');

<a name="directories"></a>
## Thư mục

#### Get All Files Within A Directory

Phương thức `files` sẽ trả về một mảng của tất cả các file có trong một thư mục nhất định. Nếu bạn muốn lấy danh sách tất cả các file có trong một thư mục bao gồm cả các thư mục con, bạn có thể sử dụng phương thức `allFiles`:

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### Get All Directories Within A Directory

Phương thức `directories` trả về một mảng gồm tất cả các thư mục có trong một thư mục đã cho. Ngoài ra, bạn có thể sử dụng phương thức `allDirectories` để lấy danh sách tất cả các thư mục có trong một thư mục đã cho và cả các thư mục con của nó:

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

#### Create A Directory

Phương thức `makeDirectory` sẽ tạo mới một thư mục, bao gồm cả thư mục con nếu cần thiết:

    Storage::makeDirectory($directory);

#### Delete A Directory

Cuối cùng, phương thức `deleteDirectory` có thể được sử dụng để xóa một thư mục và tất cả các file trong nó:

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## Tuỳ chỉnh Filesystem

Flysystem integrationcủa Laravel cung cấp nhiều driver cho một số "driver" mặc đinh; tuy nhiên, Flysystem không chỉ giới hạn ở những điều này mà còn có bộ chuyển đổi cho nhiều hệ thống lưu trữ khác. Bạn có thể tạo driver tùy biến nếu bạn muốn sử dụng một trong những bộ chuyển đổi đó vào trong ứng dụng Laravel của bạn.

Để thiết lập một tuỳ biến filesystem, bạn sẽ cần một bộ chuyển đổi Flysystem. Hãy thêm một bộ chuyển đổi Dropbox được cộng đồng phát triển vào trong dự án của bạn:

    composer require spatie/flysystem-dropbox

Tiếp theo, bạn nên tạo một [service provider](/docs/{{version}}/providers), chẳng hạn như `DropboxServiceProvider`. Trong phương thức `boot` của provider, bạn có thể sử dụng phương thức `extend` của facade `Storage` để định nghĩa tuỳ biến driver:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Storage;
    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Filesystem;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorization_token']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }
    }

Tham số đầu tiên của phương thức `extend` là tên của driver và tham số thứ hai là một Closure nhận các biến `$app` và `$config`. Closure này cần trả về một instance của `League\Flysystem\Filesystem`. Biến `$config` sẽ chứa các giá trị được định nghĩa trong file `config/filesystems.php` cho disk mà bạn đang khai báo.

Next, register the service provider in your `config/app.php` configuration file:

    'providers' => [
        // ...
        App\Providers\DropboxServiceProvider::class,
    ];

Khi bạn đã tạo và đăng ký xong service provider, bạn có thể sử dụng driver `dropbox` trong file cấu hình `config/filesystems.php`.
