# Upgrade Guide

- [Nâng cấp đến 9.0 từ 8.x](#upgrade-9.0)

<a name="high-impact-changes"></a>
## Những thay đổi có tác động lớn

<div class="content-list" markdown="1">

- [Updating Dependencies](#updating-dependencies)
- [Flysystem 3.x](#flysystem-3)
- [Symfony Mailer](#symfony-mailer)

</div>

<a name="medium-impact-changes"></a>
## Những thay đổi có tác động trung bình

<div class="content-list" markdown="1">

- [Belongs To Many `firstOrNew`, `firstOrCreate`, and `updateOrCreate` methods](#belongs-to-many-first-or-new)
- [Custom Casts & `null`](#custom-casts-and-null)
- [Default HTTP Client Timeout](#http-client-default-timeout)
- [PHP Return Types](#php-return-types)
- [Postgres "Schema" Configuration](#postgres-schema-configuration)
- [The `assertDeleted` Method](#the-assert-deleted-method)
- [The `lang` Directory](#the-lang-directory)
- [The `password` Rule](#the-password-rule)
- [The `when` / `unless` Methods](#when-and-unless-methods)
- [Unvalidated Array Keys](#unvalidated-array-keys)

</div>

<a name="upgrade-9.0"></a>
## Nâng cấp đến 9.0 từ 8.x

<a name="estimated-upgrade-time-30-minutes"></a>
#### Estimated Upgrade Time: 30 Minutes

> **Note**
> Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này mới có thể thực sự ảnh hưởng đến application của bạn. Bạn muốn tiết kiệm thời gian? Bạn có thể sử dụng [Laravel Shift](https://laravelshift.com/) để giúp tự động hóa việc nâng cấp ứng dụng của bạn.

<a name="updating-dependencies"></a>
### Updating Dependencies

**Likelihood Of Impact: High**

#### PHP 8.0.2 Required

Laravel hiện yêu cầu PHP 8.0.2 trở lên.

#### Composer Dependencies

Bạn nên cập nhật các library sau vào file `composer.json` của ứng dụng:

<div class="content-list" markdown="1">

- `laravel/framework` to `^9.0`
- `nunomaduro/collision` to `^6.1`

</div>

Ngoài ra, hãy thay thế library `facade/ignition` bằng `"spatie/laravel-ignition": "^1.0"` và `pusher/pusher-php-server` (nếu có) bằng `"pusher/pusher-php-server": "^5.0"` trong file `composer.json` của ứng dụng.

Ngoài ra, các package của bên thứ nhất sau đây cũng đã nhận được bản phát hành chính thức mới để hỗ trợ cho Laravel 9.x. Nếu có thể, bạn nên đọc hướng dẫn nâng cấp riêng của từng package trước khi nâng cấp:

<div class="content-list" markdown="1">

- [Vonage Notification Channel (v3.0)](https://github.com/laravel/vonage-notification-channel/blob/3.x/UPGRADE.md) (Thay thế Nexmo)

</div>

Cuối cùng, hãy kiểm tra tất cả các package của bên thứ ba mà ứng dụng của bạn sử dụng và xác nhận rằng bạn đang sử dụng phiên bản phù hợp để hỗ trợ Laravel 9.

<a name="php-return-types"></a>
#### PHP Return Types

PHP đang bắt đầu chuyển sang là yêu cầu bạn sẽ cần phải định nghĩa kiểu trả về trên các phương thức PHP như `offsetGet`, `offsetSet`, vv. Theo quan điểm này, Laravel 9 cũng đã triển khai các kiểu trả về này trong code base của nó. Thông thường, điều này sẽ không ảnh hưởng đến code do bạn viết; tuy nhiên, nếu bạn đang ghi đè một trong những phương thức này bằng cách extend các class core của Laravel, bạn sẽ cần thêm các kiểu trả về vào trong code ứng dụng hoặc package của bạn:

<div class="content-list" markdown="1">

- `count(): int`
- `getIterator(): Traversable`
- `getSize(): int`
- `jsonSerialize(): array`
- `offsetExists($key): bool`
- `offsetGet($key): mixed`
- `offsetSet($key, $value): void`
- `offsetUnset($key): void`

</div>

Ngoài ra, các kiểu trả về đã được thêm vào các phương thức implement `SessionHandlerInterface` của PHP. Một lần nữa, vẫn có khả năng thay đổi này ảnh hưởng đến ứng dụng hoặc code package của bạn:

<div class="content-list" markdown="1">

- `open($savePath, $sessionName): bool`
- `close(): bool`
- `read($sessionId): string|false`
- `write($sessionId, $data): bool`
- `destroy($sessionId): bool`
- `gc($lifetime): int`

</div>

<a name="application"></a>
### Application

<a name="the-application-contract"></a>
#### The `Application` Contract

**Likelihood Of Impact: Low**

Phương thức `storagePath` của interface `Illuminate\Contracts\Foundation\Application` đã được cập nhật để chấp nhận thêm tham số `$path`. Nếu bạn đang implement interface này, bạn nên cập nhật implement của bạn cho phù hợp:

    public function storagePath($path = '');

Tương tự như vậy, phương thức `langPath` của class `Illuminate\Foundation\Application` đã được cập nhật để chấp nhận thêm tham số `$path`:

    public function langPath($path = '');

#### Exception Handler `ignore` Method

**Likelihood Of Impact: Low**

Phương thức `ignore` của trình xử lý ngoại lệ sẽ là `public` thay vì `protected`. Phương thức này sẽ không có bên framework ứng dụng mặc định; tuy nhiên, nếu bạn đã định nghĩa phương thức này, bạn nên cập nhật chế độ của nó là thành `public`:

```php
public function ignore(string $class);
```

#### Exception Handler Contract Binding

**Likelihood Of Impact: Very Low**

Trước đây, để ghi đè trình xử lý ngoại lệ mặc định của Laravel, các implementation tùy chỉnh sẽ được liên kết vào service container bằng cách sử dụng type `\App\Exceptions\Handler::class`. Tuy nhiên, bây giờ bạn sẽ cần liên kết các implementation tùy chỉnh của bạn bằng cách sử dụng type `\Illuminate\Contracts\Debug\ExceptionHandler::class`.

### Blade

#### Lazy Collections & The `$loop` Variable

**Likelihood Of Impact: Low**

Khi lặp qua một instance `LazyCollection` trong template Blade, biến `$loop` sẽ không còn tồn tại vì việc truy cập vào biến này sẽ khiến toàn bộ `LazyCollection` sẽ được load vào bộ nhớ, do đó việc sử dụng lazy collection sẽ trở nên vô nghĩa trong trường hợp này.

#### Checked / Disabled / Selected Blade Directives

**Likelihood Of Impact: Low**

Các lệnh Blade `@checked`, `@disabled` và `@selected` mới có thể xung đột với các event Vue cùng tên. Bạn có thể sử dụng `@@` để chỉ rõ các lệnh javascript và tránh xung đột: `@@selected`.

### Collections

#### The `Enumerable` Contract

**Likelihood Of Impact: Low**

Contract `Illuminate\Support\Enumerable` hiện đã định nghĩa thêm một phương thức `sole`. Nếu bạn đang implement interface này, thì bạn nên cập nhật implement của bạn để phản ánh phương thức mới này:

```php
public function sole($key = null, $operator = null, $value = null);
```

#### The `reduceWithKeys` Method

Phương thức `reduceWithKeys` đã bị xóa vì phương thức `reduce` cũng cung cấp một chức năng tương tự. Bạn chỉ cần cập nhật code của bạn gọi phương thức `reduce` thay vì phương thức `reduceWithKeys`.

#### The `reduceMany` Method

Phương thức `reduceMany` đã được đổi tên thành `reduceSpread` để đặt tên thống nhất với các phương thức tương tự khác.

### Container

#### The `Container` Contract

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Container\Container` đã nhận thêm hai định nghĩa phương thức mới: `scoped` và `scopedIf`. Nếu bạn đang implement contract này, bạn nên cập nhật implement của bạn để phản ánh các phương thức mới này.


#### The `ContextualBindingBuilder` Contract

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Container\ContextualBindingBuilder` hiện đã định nghĩa thêm một phương thức `giveConfig`. Nếu bạn đang implement interface này, thì bạn nên cập nhật implement của bạn để phản ánh phương thức mới này:

```php
public function giveConfig($key, $default = null);
```

### Database

<a name="postgres-schema-configuration"></a>
#### Postgres "Schema" Configuration

**Likelihood Of Impact: Medium**

Tùy chọn cấu hình `schema` được sử dụng để cấu hình đường dẫn tìm kiếm kết nối đến cơ sở dữ liệu Postgres trong file cấu hình `config/database.php` trong ứng dụng của bạn phải được đổi tên thành `search_path`.

<a name="schema-builder-doctrine-method"></a>
#### Schema Builder `registerCustomDoctrineType` Method

**Likelihood Of Impact: Low**

Phương thức `registerCustomDoctrineType` đã bị xóa khỏi class `Illuminate\Database\Schema\Builder`. Bạn có thể sử dụng phương thức `registerDoctrineType` trên facade `DB` thay thế hoặc đăng ký các kiểu Doctrine tùy chỉnh trong file cấu hình `config/database.php`.

### Eloquent

<a name="custom-casts-and-null"></a>
#### Custom Casts & `null`

**Likelihood Of Impact: Medium**

Trong các bản phát hành trước của Laravel, phương thức `set` của các class cast tùy chỉnh sẽ không được gọi nếu thuộc tính cast được set thành `null`. Tuy nhiên, hành vi này không nhất quán với tài liệu Laravel. Trong Laravel 9.x, phương thức `set` của class cast sẽ được gọi với giá trị là `null` thông qua tham số `$value` được cung cấp. Do đó, bạn nên đảm bảo các cast tùy chỉnh của bạn có thể xử lý đủ cho tình huống này:

```php
/**
 * Prepare the given value for storage.
 *
 * @param  \Illuminate\Database\Eloquent\Model  $model
 * @param  string  $key
 * @param  AddressModel  $value
 * @param  array  $attributes
 * @return array
 */
public function set($model, $key, $value, $attributes)
{
    if (! $value instanceof AddressModel) {
        throw new InvalidArgumentException('The given value is not an Address instance.');
    }

    return [
        'address_line_one' => $value->lineOne,
        'address_line_two' => $value->lineTwo,
    ];
}
```

<a name="belongs-to-many-first-or-new"></a>
#### Belongs To Many `firstOrNew`, `firstOrCreate`, and `updateOrCreate` Methods

**Likelihood Of Impact: Medium**

Các phương thức `firstOrNew`, `firstOrCreate` và `updateOrCreate` của quan hệ `belongsToMany` đều chấp nhận một mảng các thuộc tính làm tham số đầu tiên của chúng. Trong các bản phát hành trước của Laravel, mảng các thuộc tính này được so sánh với bảng "pivot" hoặc bảng trung gian cho các bản ghi hiện tại.

Tuy nhiên, đây là hành vi không được mong đợi và thường không mong muốn. Thay vào đó, các phương thức này hiện so sánh mảng thuộc tính với bảng của model được quan hệ:

```php
$user->roles()->updateOrCreate([
    'name' => 'Administrator',
]);
```

Ngoài ra, phương thức `firstOrCreate` hiện chấp nhận mảng `$values` làm tham số thứ hai của nó. Mảng này sẽ được merge vào tham số đầu tiên của phương thức (`$attributes`) khi tạo model quan hệ nếu model đó chưa được tạo. Thay đổi này sẽ làm cho phương thức này nhất quán với các phương thức `firstOrCreate` do các quan hệ khác cung cấp:

```php
$user->roles()->firstOrCreate([
    'name' => 'Administrator',
], [
    'created_by' => $user->id,
]);
```

#### The `touch` Method

**Likelihood Of Impact: Low**

Phương thức `touch` bây giờ sẽ chấp nhận thêm một thuộc tính. Nếu trước đây bạn đã ghi đè phương thức này, bạn nên cập nhật lại mẫu phương thức của bạn để phản ánh tham số mới này:

```php
public function touch($attribute = null);
```

### Encryption

#### The Encrypter Contract

**Likelihood Of Impact: Low**

Contract `Illuminate\Contracts\Encryption\Encrypter` bây giờ sẽ định nghĩa thêm phương thức `getKey`. Nếu bạn đang implement interface này, bạn nên cập nhật implement của bạn cho phù hợp:

```php
public function getKey();
```

### Facades

#### The `getFacadeAccessor` Method

**Likelihood Of Impact: Low**

Phương thức `getFacadeAccessor` phải luôn trả về một container binding key. Trong các bản phát hành trước của Laravel, phương thức này có thể trả về một object instance; tuy nhiên, hành vi này không còn được hỗ trợ. Nếu bạn đã viết các facade của riêng bạn, bạn nên đảm bảo rằng phương thức này sẽ trả về một chuỗi container binding:

```php
/**
 * Get the registered name of the component.
 *
 * @return string
 */
protected static function getFacadeAccessor()
{
    return Example::class;
}
```

### Filesystem

#### The `FILESYSTEM_DRIVER` Environment Variable

**Likelihood Of Impact: Low**

Biến môi trường `FILESYSTEM_DRIVER` đã được đổi tên thành `FILESYSTEM_DISK` để phản ánh chính xác hơn cách sử dụng của nó. Thay đổi này chỉ ảnh hưởng đến bên framework ứng dụng; tuy nhiên, bạn có thể cập nhật các biến môi trường của ứng dụng của bạn để phản ánh thay đổi này nếu bạn muốn.

#### The "Cloud" Disk

**Likelihood Of Impact: Low**

Tùy chọn cấu hình disk `cloud` mặc định đã bị xóa khỏi bên framework ứng dụng vào tháng 11 năm 2020. Thay đổi này chỉ ảnh hưởng đến bên framework ứng dụng. Nếu bạn đang sử dụng disk `cloud` trong ứng dụng của bạn, bạn nên để giá trị cấu hình này trong framework ứng dụng của riêng bạn.

<a name="flysystem-3"></a>
### Flysystem 3.x

**Likelihood Of Impact: High**

Laravel 9.x đã chuyển đổi từ [Flysystem](https://flysystem.thephpleague.com/v2/docs/) 1.x sang 3.x. Về cơ bản, Flysystem cung cấp tất cả các phương thức thao tác file do facade `Storage` cung cấp. Theo đó, một số thay đổi có thể cần thiết trong ứng dụng của bạn; tuy nhiên, chúng tôi đã cố gắng thực hiện quá trình chuyển đổi này một cách liền mạch nhất có thể.

#### Driver Prerequisites

Trước khi sử dụng driver S3, FTP hoặc SFTP, bạn sẽ cần cài đặt package phù hợp thông qua trình quản lý package Composer:

- Amazon S3: `composer require -W league/flysystem-aws-s3-v3 "^3.0"`
- FTP: `composer require league/flysystem-ftp "^3.0"`
- SFTP: `composer require league/flysystem-sftp-v3 "^3.0"`

#### Overwriting Existing Files

Các thao tác ghi như `put`, `write` và `writeStream` bây giờ sẽ mặc định ghi đè lên các file hiện có. Nếu bạn không muốn ghi đè lên các file này, bạn nên check xem file đó đã tồn tại hay chưa trước khi thực hiện thao tác ghi.

#### Write Exceptions

Các thao tác ghi như `put`, `write` và `writeStream` không còn đưa ra ngoại lệ khi thao tác ghi không thành công. Thay vào đó, `false` sẽ được trả về. Nếu bạn muốn giữ nguyên hành vi trước đó để đưa ra ngoại lệ, bạn có thể định nghĩa tùy chọn `throw` trong mảng cấu hình của filesystem disk:

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],
```

#### Reading Missing Files

Việc đọc một file không tồn tại bây giờ sẽ trả về `null`. Trong các bản phát hành trước của Laravel, `Illuminate\Contracts\Filesystem\FileNotFoundException` sẽ được đưa ra.

#### Deleting Missing Files

Việc `delete` một file không tồn tại bây giờ sẽ trả về `true`.

#### Cached Adapters

Flysystem không còn hỗ trợ "cached adapters". Do đó, chúng đã bị xóa ra khỏi Laravel và bất kỳ cấu hình liên quan nào khác (như khóa `cache` trong cấu hình disk) đều có thể bị xóa.

#### Custom Filesystems

Đã có một số thay đổi nhỏ đối với các bước cần thiết để đăng ký driver filesystem tùy chỉnh. Do đó, nếu bạn đang định nghĩa driver filesystem tùy chỉnh của riêng bạn hoặc sử dụng các package có định nghĩa driver tùy chỉnh, bạn nên cập nhật code và các library của bạn.

Ví dụ, trong Laravel 8.x, driver filesystem tùy chỉnh có thể được đăng ký như sau:

```php
use Illuminate\Support\Facades\Storage;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

Storage::extend('dropbox', function ($app, $config) {
    $client = new DropboxClient(
        $config['authorization_token']
    );

    return new Filesystem(new DropboxAdapter($client));
});
```

Tuy nhiên, trong Laravel 9.x, callback được cung cấp cho phương thức `Storage::extend` sẽ trả về trực tiếp một instance của `Illuminate\Filesystem\FilesystemAdapter`:

```php
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

Storage::extend('dropbox', function ($app, $config) {
    $adapter = new DropboxAdapter(
        new DropboxClient($config['authorization_token'])
    );

    return new FilesystemAdapter(
        new Filesystem($adapter, $config),
        $adapter,
        $config
    );
});
```

#### SFTP Private-Public Key Passphrase

Nếu ứng dụng của bạn đang sử dụng SFTP adapter và xác thực private-public key của Flysystem, thì mục cấu hình `password` được sử dụng để giải mã khóa private phải được đổi tên thành `passphrase`.

### Helpers

<a name="data-get-function"></a>
#### The `data_get` Helper & Iterable Objects

**Likelihood Of Impact: Very Low**

Trước đây, helper `data_get` có thể được sử dụng để lấy ra dữ liệu lồng nhau trên các mảng và các instance `Collection`; tuy nhiên, helper này bây giờ có thể lấy ra dữ liệu lồng nhau trên cả các đối tượng có thể lặp.

<a name="str-function"></a>
#### The `str` Helper

**Likelihood Of Impact: Very Low**

Laravel 9.x bây giờ sẽ chứa [hàm global helper `str`](/docs/{{version}}/helpers#method-str). Nếu bạn đang định nghĩa hàm global helper `str` trong ứng dụng của bạn, bạn nên đổi tên hoặc xóa hàm đó để nó không xung đột với hàm helper `str` của Laravel.

<a name="when-and-unless-methods"></a>
#### The `when` / `unless` Methods

**Likelihood Of Impact: Medium**

Như bạn có thể biết, các phương thức `when` và `unless` được cung cấp bởi nhiều class khác nhau trong toàn bộ framework. Các phương thức này có thể được sử dụng để thực hiện một hành động có điều kiện nếu giá trị boolean của tham số đầu tiên của phương thức được trả về là `true` hoặc `false`:

```php
$collection->when(true, function ($collection) {
    $collection->merge([1, 2, 3]);
});
```

Do đó, trong các phiên bản trước của Laravel, việc truyền closure cho các phương thức `when` hoặc `unless` có nghĩa là thao tác đó sẽ luôn được thực thi, vì phép so sánh "lỏng lẻo" đối với đối tượng closure (hoặc bất kỳ đối tượng nào khác) sẽ luôn trả về là `true`. Điều này thường dẫn đến kết quả không mong muốn vì các nhà phát triển mong đợi **kết quả** của closure được sử dụng làm giá trị boolean để xác định xem hành động đó có được thực thi hay không.

Vì vậy, trong Laravel 9.x, bất kỳ closure nào được truyền vào cho phương thức `when` hoặc `unless` sẽ luôn được thực thi và giá trị trả về bởi closure sẽ được coi là giá trị boolean để phương thức `when` và `unless` sử dụng để xem có chạy phương thức tiếp theo hay không:

```php
$collection->when(function ($collection) {
    // This closure is executed...
    return false;
}, function ($collection) {
    // Not executed since first closure returned "false"...
    $collection->merge([1, 2, 3]);
});
```

### HTTP Client

<a name="http-client-default-timeout"></a>
#### Default Timeout

**Likelihood Of Impact: Medium**

[HTTP client](/docs/{{version}}/http-client) bây giờ sẽ có thời gian chờ đợi mặc định là 30 giây. Nói cách khác, nếu server không phản hồi trong vòng 30 giây, một ngoại lệ sẽ được đưa ra. Trước đây, không có thời gian chờ mặc định nào được cấu hình cho HTTP client, do đó, sẽ khiến các request thỉnh thoảng bị "treo" vô thời hạn.

Nếu bạn muốn chỉ định một thời gian chờ dài hơn cho một request nhất định, bạn có thể thực hiện bằng phương thức `timeout`:

    $response = Http::timeout(120)->get(/* ... */);

#### HTTP Fake & Middleware

**Likelihood Of Impact: Low**

Trước đây, Laravel sẽ không thực thi bất kỳ Guzzle HTTP middleware nào được cung cấp khi [HTTP client](/docs/{{version}}/http-client) bị "faked". Tuy nhiên, trong Laravel 9.x, Guzzle HTTP middleware sẽ được thực thi ngay cả khi HTTP client bị faked.

#### HTTP Fake & Dependency Injection

**Likelihood Of Impact: Low**

Trong các phiên bản trước của Laravel, việc gọi phương thức `Http::fake()` sẽ không ảnh hưởng đến các instance của `Illuminate\Http\Client\Factory` mà được tích hợp vào các hàm constructor của class. Tuy nhiên, trong Laravel 9.x, `Http::fake()` sẽ phải đảm bảo các fake response sẽ được trả về bởi các HTTP client mà được tích hợp vào các service khác thông qua dependency injection. Hành vi này sẽ phù hợp hơn với các hành vi của facades và fakes khác.

<a name="symfony-mailer"></a>
### Symfony Mailer

**Likelihood Of Impact: High**

Một trong những thay đổi lớn nhất trong Laravel 9.x là quá trình chuyển đổi từ SwiftMailer, không còn được bảo trì từ tháng 12 năm 2021, sang Symfony Mailer. Tuy nhiên, chúng tôi đã cố gắng thực hiện quá trình chuyển đổi này một cách liền mạch nhất có thể cho các ứng dụng của bạn. Tuy nhiên, vui lòng xem xét kỹ danh sách các thay đổi ở bên dưới để đảm bảo rằng ứng dụng của bạn hoàn toàn tương thích.

#### Driver Prerequisites

Để tiếp tục sử dụng Mailgun transport, ứng dụng của bạn phải yêu cầu các package Composer `symfony/mailgun-mailer` và `symfony/http-client`:

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

Package Composer `wildbit/swiftmailer-postmark` phải được xóa ra khỏi ứng dụng của bạn. Thay vào đó, ứng dụng của bạn phải yêu cầu các package Composer `symfony/postmark-mailer` và `symfony/http-client`:

```shell
composer require symfony/postmark-mailer symfony/http-client
```

#### Updated Return Types

Các phương thức `send`, `html`, `raw` và `plain` trên `Illuminate\Mail\Mailer` sẽ không còn trả về `void` nữa. Thay vào đó, một instance của `Illuminate\Mail\SentMessage` sẽ được trả về. Đối tượng này sẽ chứa một instance của `Symfony\Component\Mailer\SentMessage` có thể truy cập thông qua phương thức `getSymfonySentMessage` hoặc bằng cách gọi phương thức dynamically trên đối tượng.

#### Renamed "Swift" Methods

Nhiều phương thức liên quan đến SwiftMailer, một số trong đó không được ghi lại, đã được đổi tên thành các phương thức Symfony Mailer tương ứng. Ví dụ, phương thức `withSwiftMessage` đã được đổi tên thành `withSymfonyMessage`:

    // Laravel 8.x...
    $this->withSwiftMessage(function ($message) {
        $message->getHeaders()->addTextHeader(
            'Custom-Header', 'Header Value'
        );
    });

    // Laravel 9.x...
    use Symfony\Component\Mime\Email;

    $this->withSymfonyMessage(function (Email $message) {
        $message->getHeaders()->addTextHeader(
            'Custom-Header', 'Header Value'
        );
    });

> **Warning**
> Vui lòng xem kỹ [tài liệu Symfony Mailer](https://symfony.com/doc/6.0/mailer.html#creating-sending-messages) để biết tất cả các tương tác có thể có với đối tượng `Symfony\Component\Mime\Email`.

Danh sách bên dưới sẽ chi tiết hơn về các phương thức đã được đổi tên. Nhiều phương thức trong số này là các phương thức cấp thấp được sử dụng để tương tác trực tiếp với SwiftMailer / Symfony Mailer, do đó có thể không được sử dụng phổ biến trong hầu hết các ứng dụng Laravel:

    Message::getSwiftMessage();
    Message::getSymfonyMessage();

    Mailable::withSwiftMessage($callback);
    Mailable::withSymfonyMessage($callback);

    MailMessage::withSwiftMessage($callback);
    MailMessage::withSymfonyMessage($callback);

    Mailer::getSwiftMailer();
    Mailer::getSymfonyTransport();

    Mailer::setSwiftMailer($swift);
    Mailer::setSymfonyTransport(TransportInterface $transport);

    MailManager::createTransport($config);
    MailManager::createSymfonyTransport($config);

#### Proxied `Illuminate\Mail\Message` Methods

`Illuminate\Mail\Message` thường sẽ chuyển hướng các phương thức bị thiếu cho instance `Swift_Message`. Tuy nhiên, các phương thức bị thiếu bây giờ sẽ được chuyển cho instance `Symfony\Component\Mime\Email`. Vì vậy, bất kỳ code nào trước đây dựa vào các phương thức bị thiếu để được chuyển hướng cho SwiftMailer đều phải được cập nhật thành các Symfony Mailer tương ứng của chúng.

Một lần nữa, nhiều ứng dụng có thể không tương tác với các phương thức này vì chúng không được ghi trong tài liệu Laravel:

    // Laravel 8.x...
    $message
        ->setFrom('taylor@laravel.com')
        ->setTo('example@example.org')
        ->setSubject('Order Shipped')
        ->setBody('<h1>HTML</h1>', 'text/html')
        ->addPart('Plain Text', 'text/plain');

    // Laravel 9.x...
    $message
        ->from('taylor@laravel.com')
        ->to('example@example.org')
        ->subject('Order Shipped')
        ->html('<h1>HTML</h1>')
        ->text('Plain Text');

#### Generated Messages IDs

SwiftMailer cung cấp khả năng xác định tên miền tùy chỉnh để đưa vào ID message được tạo thông qua tùy chọn cấu hình `mime.idgenerator.idright`. Tính năng này không được Symfony Mailer hỗ trợ. Thay vào đó, Symfony Mailer sẽ tự động tạo ID message dựa trên người gửi.

#### `MessageSent` Event Changes

Thuộc tính `message` của event `Illuminate\Mail\Events\MessageSent` bây giờ sẽ chứa một instance của `Symfony\Component\Mime\Email` thay vì một instance của `Swift_Message`. Message này đại diện cho email **trước khi** được gửi.

Ngoài ra, một thuộc tính `sent` mới đã được thêm vào event `MessageSent`. Thuộc tính này chứa một instance của `Illuminate\Mail\SentMessage` và chứa thông tin về email đã gửi, chẳng hạn như ID message.

#### Forced Reconnections

Bây giờ, sẽ không còn có thể bắt buộc kết nối lại với transport (ví dụ khi mailer đang chạy qua tiến trình daemon). Thay vào đó, Symfony Mailer sẽ tự động kết nối lại với transport và đưa ngoại lệ nếu như kết nối lại bị lỗi.

#### SMTP Stream Options

Việc định nghĩa các tùy chọn stream cho SMTP transport không còn được hỗ trợ. Thay vào đó, bạn phải định nghĩa các tùy chọn có liên quan trực tiếp trong cấu hình nếu chúng được hỗ trợ. Ví dụ: để disable verification TLS:

    'smtp' => [
        // Laravel 8.x...
        'stream' => [
            'ssl' => [
                'verify_peer' => false,
            ],
        ],

        // Laravel 9.x...
        'verify_peer' => false,
    ],

Để tìm hiểu thêm về các tùy chọn cấu hình có sẵn, vui lòng xem [tài liệu Symfony Mailer](https://symfony.com/doc/6.0/mailer.html#transport-setup).

> **Warning**
> Bất chấp ví dụ trên, bạn thường không được khuyên disable SSL verification vì nó có thể dẫn đến khả năng xảy ra các cuộc tấn công "trung gian".

#### SMTP `auth_mode`

Bây giờ, sẽ không còn cần phải định nghĩa SMTP `auth_mode` trong file cấu hình `mail`. Chế độ xác thực sẽ được tự động giữa Symfony Mailer và máy chủ SMTP.

#### Failed Recipients

Sẽ không còn có thể lấy ra danh sách người nhận bị thất bại sau khi gửi một tin nhắn. Thay vào đó, ngoại lệ `Symfony\Component\Mailer\Exception\TransportExceptionInterface` sẽ được đưa ra nếu tin nhắn không gửi được. Thay vì dựa vào việc lấy ra địa chỉ email không hợp lệ sau khi gửi một tin nhắn, chúng tôi khuyên bạn nên xác thực địa chỉ email trước khi gửi tin nhắn.

### Packages

<a name="the-lang-directory"></a>
#### The `lang` Directory

**Likelihood Of Impact: Medium**

Trong các ứng dụng Laravel mới, thư mục `resources/lang` bây giờ sẽ nằm trong thư mục root của dự án (`lang`). Nếu package của bạn đang export các file ngôn ngữ vào thư mục này, bạn nên đảm bảo rằng package của bạn đang export vào `app()->langPath()` thay vì đường dẫn được hard-code.

<a name="queue"></a>
### Queue

<a name="the-opis-closure-library"></a>
#### The `opis/closure` Library

**Likelihood Of Impact: Low**

Sự phụ thuộc của Laravel vào `opis/closure` đã được thay thế bằng `laravel/serializable-closure`. Điều này sẽ không gây ra bất kỳ thay đổi nghiêm trọng nào trong ứng dụng của bạn trừ khi bạn đang tương tác trực tiếp với thư viện `opis/closure`. Ngoài ra, các class `Illuminate\Queue\SerializableClosureFactory` và `Illuminate\Queue\SerializableClosure` đã bị loại bỏ trước đó, đã bị xoá. Nếu bạn đang tương tác trực tiếp với thư viện `opis/closure` hoặc sử dụng bất kỳ class nào đã bị xoá, bạn có thể sử dụng [Laravel Serializable Closure](https://github.com/laravel/serializable-closure) thay thế.

#### The Failed Job Provider `flush` Method

**Likelihood Of Impact: Low**

Phương thức `flush` được định nghĩa bởi interface `Illuminate\Queue\Failed\FailedJobProviderInterface` bây giờ sẽ chấp nhận thêm tham số `$hours` dùng để xác định thời gian của một job bị lỗi (tính bằng giờ) trước khi nó được xóa bằng lệnh `queue:flush`. Nếu bạn đang implement `FailedJobProviderInterface`, thì bạn nên đảm bảo là implement của bạn được cập nhật để phản ánh tham số mới này:

```php
public function flush($hours = null);
```

### Session

#### The `getSession` Method

**Likelihood Of Impact: Low**

Class `Symfony\Component\HttpFoundaton\Request` được extend bởi class `Illuminate\Http\Request` của Laravel đã cung cấp một phương thức `getSession` để lấy ra session storage hiện tại. Laravel không ghi lại phương thức này vì hầu hết các ứng dụng Laravel sẽ tương tác với session thông qua phương thức `session` của Laravel.

Phương thức `getSession` trước đây sẽ trả về một instance của `Illuminate\Session\Store` hoặc `null`; tuy nhiên, do bản phát hành Symfony 6.x áp dụng kiểu trả về là `Symfony\Component\HttpFoundation\Session\SessionInterface`, nên `getSession` hiện sẽ trả về đúng một implementation của `SessionInterface` hoặc đưa ra một ngoại lệ `\Symfony\Component\HttpFoundation\Exception\SessionNotFoundException` khi không có session nào tồn tại.

### Testing

<a name="the-assert-deleted-method"></a>
#### The `assertDeleted` Method

**Likelihood Of Impact: Medium**

Tất cả các lệnh gọi đến phương thức `assertDeleted` phải được cập nhật thành `assertModelMissing`.

### Trusted Proxies

**Likelihood Of Impact: Low**

Nếu bạn đang nâng cấp project Laravel 8 của bạn lên Laravel 9 bằng cách import code ứng dụng hiện có vào một framework ứng dụng Laravel 9 hoàn toàn mới, bạn có thể cần cập nhật middleware  "trusted proxy của ứng dụng.

Trong file `app/Http/Middleware/TrustProxies.php` của bạn, hãy cập nhật `use Fideloper\Proxy\TrustProxies as Middleware` thành `use Illuminate\Http\Middleware\TrustProxies as Middleware`.

Tiếp theo, trong `app/Http/Middleware/TrustProxies.php`, bạn nên cập nhật định nghĩa thuộc tính `$headers`:

```php
// Before...
protected $headers = Request::HEADER_X_FORWARDED_ALL;

// After...
protected $headers =
    Request::HEADER_X_FORWARDED_FOR |
    Request::HEADER_X_FORWARDED_HOST |
    Request::HEADER_X_FORWARDED_PORT |
    Request::HEADER_X_FORWARDED_PROTO |
    Request::HEADER_X_FORWARDED_AWS_ELB;
```

Cuối cùng, bạn có thể xóa library Composer `fideloper/proxy` ra khỏi ứng dụng của bạn:

```shell
composer remove fideloper/proxy
```

### Validation

#### Form Request `validated` Method

**Likelihood Of Impact: Low**

Phương thức `validated` được cung cấp bởi các form request bây giờ sẽ chấp nhận các tham số `$key` và `$default`. Nếu bạn ghi đè định nghĩa của phương thức này, bạn nên cập nhật mẫu của phương thức để phản ánh các tham số mới này:

```php
public function validated($key = null, $default = null)
```

<a name="the-password-rule"></a>
#### The `password` Rule

**Likelihood Of Impact: Medium**

Quy tắc `password`, cái mà validate giá trị input có khớp với mật khẩu hiện tại của người dùng đã xác thực hay không, đã được đổi tên thành `current_password`.

<a name="unvalidated-array-keys"></a>
#### Unvalidated Array Keys

**Likelihood Of Impact: Medium**

Trong các phiên bản Laravel trước, bạn phải tự hướng dẫn validator của Laravel loại bỏ các khóa mảng chưa được validate ra khỏi dữ liệu "đã được validate" mà nó trả về, đặc biệt là khi kết hợp với quy tắc `array` không chỉ định một list các khóa được phép.

Tuy nhiên, trong Laravel 9.x, các khóa mảng mà chưa được validate sẽ luôn bị loại bỏ ra khỏi dữ liệu "đã được validate" ngay cả khi không có khóa nào được phép được chỉ định thông qua quy tắc `array`. Thông thường, hành vi này là hành vi được mong đợi nhất và phương thức `excludeUnvalidatedArrayKeys` trước đó chỉ được thêm vào Laravel 8.x như một biện pháp tạm thời để duy trì khả năng tương thích ngược.

Mặc dù không được khuyến khích, bạn có thể chọn sử dụng hành vi Laravel 8.x trước đó bằng cách gọi phương thức `includeUnvalidatedArrayKeys` mới trong phương thức `boot` của một trong các service provider của ứng dụng:

```php
use Illuminate\Support\Facades\Validator;

/**
 * Register any application services.
 *
 * @return void
 */
public function boot()
{
    Validator::includeUnvalidatedArrayKeys();
}
```

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [Công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/8.x...9.x) và chọn bản cập nhật nào quan trọng với bạn.
