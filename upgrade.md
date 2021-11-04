# Upgrade Guide

- [Upgrade đến 5.5.42 từ 5.5](#upgrade-5.5.42)
- [Upgrade đến 5.5.0 từ 5.4](#upgrade-5.5.0)

<a name="upgrade-5.5.42"></a>
## Upgrade đến 5.5.42 từ 5.5 (Release bảo mật)

Laravel 5.5.42 là một bản phát hành bảo mật riêng của Laravel và được khuyến cáo là phải nâng cấp ngay cho tất cả mọi người dùng. Laravel 5.5.42 cũng chứa một thay đổi nghiêm trọng cho việc mã hóa và serialization của cookie, vì vậy hãy đọc kỹ các lưu ý sau khi nâng cấp application của bạn.

**Lỗ hổng này chỉ có thể bị khai thác nếu khóa mã hóa của application của bạn (ở đây là biến môi trường `APP_KEY`) đã bị người khác truy cập được.** Thông thường, người dùng application của bạn không thể truy cập vào giá trị này. Tuy nhiên, nếu nhân viên cũ của bạn có thể truy cập vào khóa mã hóa này thì họ có thể sử dụng khóa này để tấn công lại vào application của bạn. Nếu bạn có một lý do nào đó để tin rằng khóa mã hóa này của bạn đang nằm trong tay của một nhận viên cũ hoặc một kẻ phá hoại, thì bạn phải **tạo** lại giá trị của khóa đó.

### Cookie Serialization

Laravel 5.5.42 sẽ vô hiệu hóa tất cả các serialization / unserialization của các giá trị cookie. Vì tất cả các cookie của Laravel đều đã được mã hóa và được ký, nên các giá trị cookie thường được coi là an toàn cho việc phân biệt các hành động giả mạo khách hàng. **Tuy nhiên, nếu khóa mã hóa của application của bạn nằm trong tay của một kẻ phá hoại, thì người đó có thể tạo các giá trị cookie bằng cách sử dụng khóa mã hóa đó và khai thác các lỗ hổng thừa kế để serialization / unserialization các đối tượng PHP, như là gọi các phương thức class tùy ý trong application của bạn.**

Vô hiệu hóa serialization trên tất cả các giá trị cookie sẽ làm mất hiệu lực tất cả các session của bạn và người dùng sẽ cần phải đăng nhập lại vào application. Ngoài ra, mọi cookie mã hóa khác mà application của bạn đang cài đặt sẽ trở lên không hợp lệ. Vì lý do này, bạn có thể muốn thêm logic vào application của bạn để validate các giá trị cookie này của bạn giống với danh sách các giá trị mà bạn mong muốn; nếu không, bạn nên xoá chúng.

#### Configuring Cookie Serialization

Vì lỗ hổng này không thể bị khai thác nếu kẻ tấn công không thể truy cập vào khóa mã hóa của bạn, nên chúng tôi đã cung cấp một cách để bạn có thể bật lại serialization cookie mã hóa trong lúc bạn làm cho application của bạn tương thích với những thay đổi này. Để bật / tắt serialization cookie, bạn có thể thay đổi thuộc tính static `serialize` của [middleware](https://github.com/laravel/laravel/blob/master/app/Http/Middleware/EncryptCookies.php) `App\Http\Middleware\EncryptCookies`:

    /**
     * Indicates if cookies should be serialized.
     *
     * @var bool
     */
    protected static $serialize = true;

> **Lưu ý:** Khi bật serialization cookie mã hóa, application của bạn có thể bị tấn công nếu khóa mã hóa của application bị truy cập bởi kẻ tấn công. Nếu bạn tin rằng khóa của bạn có thể đang nằm trong tay của kẻ tấn công, bạn nên tạo một khóa mới trước khi bật serialization cookie mã hóa.

<a name="upgrade-5.5.0"></a>
## Upgrade đến 5.5.0 từ 5.4

#### Estimated Upgrade Time: 1 Hour

> {note} Chúng tôi sẽ cố gắng ghi lại mọi thay đổi có thể xảy ra. Vì một số thay đổi này nằm trong các phần ẩn của framework, nên chỉ một phần trong những thay đổi này có thể thực sự ảnh hưởng đến application của bạn.

### PHP

Laravel 5.5 requires PHP 7.0.0 or higher.

### Updating Dependencies

Cập nhật library `laravel/framework` của bạn thành `5.5.*` trong file `composer.json` của bạn. Ngoài ra, bạn nên cập nhật library `phpunit/phpunit` của bạn thành `~6.0`. Tiếp theo, thêm package `filp/whoops` với phiên bản `~2.0` vào phần `require-dev` trong file `composer.json` của bạn. Cuối cùng, trong phần `scripts` trong file `composer.json` của bạn, hãy thêm lệnh `package:discover` vào event `post-autoload-dump`:

    "scripts": {
        ...
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover"
        ],
    }

Nếu bạn đang sử dụng package `laravel/browser-kit-testing`, bạn nên cập nhật package thành `2.*` trong file `composer.json` của bạn.

Tất nhiên, đừng quên kiểm tra lại bất kỳ package của bên thứ 3 nào mà application của bạn đang sử dụng và phiên bản của chúng cũng sẽ cần phải phù hợp với phiên bản Laravel 5.5.

#### Laravel Installer

> {tip} Nếu bạn thường sử dụng Laravel installer thông qua `laravel new`, bạn nên cập nhật package Laravel installer của bạn bằng lệnh `composer global update`.

#### Laravel Dusk

Laravel Dusk phiên bản `2.0.0` đã được phát hành để cung cấp khả năng tương thích với Laravel 5.5 và test cho Chrome.

#### Pusher

Driver cho Pusher event broadcast sẽ yêu cầu Pusher SDK phiên bản `~3.0`.

#### Swift Mailer

Laravel 5.5 sẽ yêu cầu Swift Mailer phiên bản `~6.0`.

### Artisan

#### Auto-Loading Commands

Trong Laravel 5.5, Artisan có thể tự động load các lệnh để bạn không phải tự đăng ký chúng trong kernel. Để tận dụng tính năng mới này, bạn nên thêm dòng sau vào phương thức `commands` trong class `App\Console\Kernel`:

    $this->load(__DIR__.'/Commands');

#### The `fire` Method

Bất kỳ phương thức `fire` nào có trong lệnh Artisan của bạn sẽ được đổi tên thành `handle`.

#### The `optimize` Command

Với những cải tiến gần đây đối với bộ nhớ đệm của PHP, thì lệnh `optimize` của Artisan sẽ không còn cần thiết nữa. Bạn nên xóa các khai báo có sử dụng lệnh này trong các file deploy của bạn vì nó sẽ bị xóa trong bản phát hành tiếp theo của Laravel.

### Authorization

> {note} Khi nâng cấp từ Laravel 5.4 lên 5.5, tất cả các cookie `remember_me` sẽ bị không hợp lệ và người dùng sẽ bị out ra màn hình đăng nhập trở lại.

#### The `authorizeResource` Controller Method

Khi truyền tên của một model mà có nhiều từ trong tên model đó cho phương thức `authorizeResource`, thì kết quả của route bây giờ sẽ là "snake" case, giống với kết quả của các resource controller.

#### The `basic` and `onceBasic` Methods

Các phương thức `Auth::basic` và `Auth::onceBasic` bây giờ sẽ đưa ra các `\Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException` thay vì trả về một `Response` khi authentication thất bại. Mặc định, điều này vẫn sẽ tạo ra một response 401, được gửi về client. Nhưng, nếu logic của application của bạn đang kiểm tra giá trị trả về của phương thức `Auth::basic` để tuỳ biến việc phản hồi hoặc thực hiện một hành động khác khi không authentication được, thì bây giờ bạn sẽ cần xử lý ngoại lệ `UnauthorizedHttpException` trong một lệnh `catch` hoặc trong trình xử lý ngoại lệ trong application của bạn.

#### The `before` Policy Method

Phương thức `before` của class policy sẽ không được gọi nếu class không chứa phương thức giống với tên của ability sẽ được kiểm tra.

### Cache

#### Database Driver

Nếu bạn đang sử dụng driver cache là cơ sở dữ liệu, bạn nên chạy `php artisan cache:clear` khi deploy lần đầu tiên cho application Laravel sau khi được nâng cấp lên bản 5.5.

### Eloquent

#### The `belongsToMany` Method

Nếu bạn đang ghi đè phương thức `belongsToMany` trên model Eloquent của bạn, bạn nên cập nhật phương thức của bạn để phản ánh việc thêm các tham số mới:

    /**
     * Define a many-to-many relationship.
     *
     * @param  string  $related
     * @param  string  $table
     * @param  string  $foreignPivotKey
     * @param  string  $relatedPivotKey
     * @param  string  $parentKey
     * @param  string  $relatedKey
     * @param  string  $relation
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function belongsToMany($related, $table = null, $foreignPivotKey = null,
                                  $relatedPivotKey = null, $parentKey = null,
                                  $relatedKey = null, $relation = null)
    {
        //
    }

#### BelongsToMany `getQualifiedRelatedKeyName`

Phương thức `getQualifiedRelatedKeyName` đã được đổi tên thành`getQualifiedRelatedPivotKeyName`.

#### BelongsToMany `getQualifiedForeignKeyName`

Phương thức `getQualifiedForeignKeyName` đã được đổi tên thành`getQualifiedForeignPivotKeyName`.

#### Model `is` Method

Nếu bạn đang ghi đè phương thức `is` trong model Eloquent của bạn, bạn nên xóa khai báo kiểu `Model` ra khỏi phương thức. Điều này cho phép phương thức `is` nhận được các giá trị `null` làm tham số:

    /**
     * Determine if two models have the same ID and belong to the same table.
     *
     * @param  \Illuminate\Database\Eloquent\Model|null  $model
     * @return bool
     */
    public function is($model)
    {
        //
    }

#### Model `$events` Property

Thuộc tính `$events` trong các model của bạn sẽ cần phải đổi tên thành `$dispatchesEvents`. Thay đổi này cần được thực hiện là do có nhiều người dùng cần định nghĩa quan hệ của `events`, mà điều này lại gây ra xung đột với tên thuộc tính cũ.

#### Pivot `$parent` Property

Thuộc tính protected `$parent` trong class `Illuminate\Database\Eloquent\Relations\Pivot` đã được đổi tên thành `$pivotParent`.

#### Relationship `create` Methods

Các phương thức `BelongsToMany`, `HasOneOrMany`, và `MorphOneOrMany` đã được thay đổi để cung cấp một giá trị mặc định cho tham số `$attributes`. Nếu bạn đang ghi đè phương thức này, bạn nên cập nhật phương thức của bạn để phù hợp với định nghĩa mới:

    public function create(array $attributes = [])
    {
        //
    }

#### Soft Deleted Models

Khi xóa một model bằng "soft deleted", thuộc tính `exists` trên model vẫn sẽ là `true`.

#### `withCount` Column Formatting

Khi sử dụng alias, phương thức `withCount` sẽ không còn tự động thêm `_count` vào tên cột kết quả. Ví dụ, trong Laravel 5.4, truy vấn sau đây sẽ tạo ra một cột `bar_count` được thêm vào trong kết quả:

    $users = User::withCount('foo as bar')->get();

Tuy nhiên, trong Laravel 5.5, alias sẽ được sử dụng tên như được đưa ra. Nếu bạn muốn thêm `_count` vào cột kết quả, bạn phải định nghĩa hậu tố đó khi định nghĩa alias:

    $users = User::withCount('foo as bar_count')->get();

#### Model Methods & Attribute Names

Để ngăn cho việc truy cập vào các thuộc tính private của model khi truy cập thông qua một mảng, không còn có thể có một phương thức model có cùng tên với một thuộc tính. Làm như vậy sẽ khiến một ngoại lệ được đưa ra khi truy cập vào các thuộc tính của model thông qua mảng (`$user['name']`) hoặc hàm helper `data_get`.

### Exception Format

Trong Laravel 5.5, tất cả các ngoại lệ, bao gồm các ngoại lệ validation, sẽ được chuyển thành HTTP response bởi trình xử lý ngoại lệ. Ngoài ra, định dạng mặc định cho các lỗi validation JSON cũng đã được thay đổi. Các định dạng mới phù hợp với quy ước sau:

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }

Tuy nhiên, nếu bạn muốn duy trì định dạng lỗi JSON của Laravel 5.4, bạn có thể thêm phương thức sau vào class `App\Exceptions\Handler`:

    use Illuminate\Validation\ValidationException;

    /**
     * Convert a validation exception into a JSON response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }

#### JSON Authentication Attempts

Thay đổi này cũng ảnh hưởng đến định dạng lỗi validation cho các lần authentication được thực hiện thông qua JSON. Trong Laravel 5.5, các lỗi authentication JSON sẽ trả về các thông báo lỗi theo quy ước định dạng mới được mô tả ở trên.

#### A Note On Form Requests

Nếu bạn đang tùy biến một định dạng response cho một form request riêng biệt, thì bây giờ bạn sẽ cần ghi đè phương thức `failedValidation` của form request đó và đưa ra một instance `HttpResponseException` có chứa response tùy biến của bạn:

    use Illuminate\Http\Exceptions\HttpResponseException;

    /**
     * Handle a failed validation attempt.
     *
     * @param  \Illuminate\Contracts\Validation\Validator  $validator
     * @return void
     *
     * @throws \Illuminate\Validation\ValidationException
     */
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(response()->json(..., 422));
    }

### Filesystem

#### The `files` Method

Phương thức `files` của class `Illuminate\Filesystem\Filesystem` đã được thay đổi để thêm một tham số `$hidden` và sẽ trả về một mảng đối tượng `SplFileInfo`, tương tự như phương thức `allFiles`. Trước đây, phương thức `files` sẽ trả về một mảng các tên đường dẫn. Cách khai báo mới sẽ như sau:

    public function files($directory, $hidden = false)

### Mail

#### Unused Parameters

Các tham số `$data` và `$callback` do không được sử dụng nên đã bị xóa khỏi các phương thức `queue` và `later` của contract `Illuminate\Contracts\Mail\MailQueue`:

    /**
     * Queue a new e-mail message for sending.
     *
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function queue($view, $queue = null);

    /**
     * Queue a new e-mail message for sending after (n) seconds.
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function later($delay, $view, $queue = null);

### Queues

#### The `dispatch` Helper

Nếu bạn muốn gửi một job chạy ngay lập tức và trả về một giá trị từ phương thức `handle`, bạn nên sử dụng phương thức `dispatch_now` hoặc `Bus::dispatchNow` để gửi job:

    use Illuminate\Support\Facades\Bus;

    $value = dispatch_now(new Job);

    $value = Bus::dispatchNow(new Job);

### Requests

#### The `all` Method

Nếu bạn đang ghi đè phương thức `all` của class `Illuminate\Http\Request`, bạn nên cập nhật phương thức của bạn để thêm tham số `$keys` mới:

    /**
     * Get all of the input and files for the request.
     *
     * @param  array|mixed  $keys
     * @return array
     */
    public function all($keys = null)
    {
        //
    }

#### The `has` Method

Phương thức `$request->has` sẽ trả về `true` ngay cả khi giá trị input là một chuỗi rỗng hoặc `null`. Một phương thức `$request->filled` mới cũng đã được thêm vào để cung cấp hành động giống hành động trước đó của phương thức `has`.

#### The `intersect` Method

Phương thức `intersect` đã bị xóa. Bạn có thể thực hiện hành động này bằng cách sử dụng phương thức `array_filter` với tham số là `$request->only`:

    return array_filter($request->only('foo'));

#### The `only` Method

Phương thức `only` bây giờ sẽ chỉ trả về các thuộc tính thực sự tồn tại trong request. Nếu bạn muốn thực hiện hành động cũ của phương thức `only`, bạn có thể sử dụng phương thức `all` thay thế.

    return $request->all('foo');

#### The `request()` Helper

Helper `request` sẽ không lấy ra các key lồng nhau. Nếu cần, bạn có thể sử dụng phương thức `input` của request để thực hiện hành động này:

    return request()->input('filters.date');

### Testing

#### Authentication Assertions

Một số hàm kiểm tra authentication đã được đổi tên để thống nhất với các hàm kiểm tra còn lại của framework:

<div class="content-list" markdown="1">
- `seeIsAuthenticated` đổi thành `assertAuthenticated`.
- `dontSeeIsAuthenticated` đổi thành `assertGuest`.
- `seeIsAuthenticatedAs` đổi thành `assertAuthenticatedAs`.
- `seeCredentials` đổi thành `assertCredentials`.
- `dontSeeCredentials` đổi thành `assertInvalidCredentials`.
</div>

#### Mail Fake

Nếu bạn đang sử dụng một fake `Mail` để kiểm tra xem một mail **đã được queue** hay chưa, thì bây giờ bạn có thể sử dụng `Mail::assertQueued` thay vì `Mail::assertSent`. Sự khác biệt này cho phép bạn yêu cầu mail đó phải được queue để gửi dưới background và không được gửi trong request.

#### Tinker

Laravel Tinker hiện tại đã hỗ trợ bỏ qua các namespace khi trỏ vào các class của application của bạn. Tính năng này yêu cầu Composer class-map phải được tối ưu, nên vì vậy bạn cần thêm lệnh `optimize-autoloader` vào phần `config` trong file `composer.json` của bạn:

    "config": {
        ...
        "optimize-autoloader": true
    }

### Translation

#### The `LoaderInterface`

Interface `Illuminate\Translation\LoaderInterface` đã được chuyển sang `Illuminate\Contracts\Translation\Loader`.

### Validation

#### Validator Methods

Tất cả các phương thức validation của validator đang là `public` thay vì `protected`.

### Views

#### Dynamic "With" Variable Names

Khi cho phép phương thức động `__call` chia sẻ các biến với một view, thì các biến này sẽ được tự động sử dụng theo dạng "camel" case. Ví dụ, như sau:

    return view('pool')->withMaximumVotes(100);

Biến `maximumVotes` có thể được truy cập trong template như sau:

    {{ $maximumVotes }}

#### `@php` Blade Directive

Lệnh blade `@php` không còn chấp nhận các inline tag. Thay vào đó, hãy sử dụng full form của lệnh:

    @php
        $teamMember = true;
    @endphp

### Miscellaneous

Chúng tôi cũng khuyến cáo bạn nên xem các thay đổi trong `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). Mặc dù nhiều thay đổi trong số này là không bắt buộc, nhưng có thể bạn muốn giữ các file đó được đồng bộ với ứng dụng của bạn. Một số thay đổi sẽ được đề cập trong hướng dẫn nâng cấp này, nhưng đối với những thay đổi khác, chẳng hạn như thay đổi file cấu hình hoặc comment đều sẽ không được đề cập đến. Bạn có thể dễ dàng xem các thay đổi đó bằng [Công cụ so sánh của GitHub](https://github.com/laravel/laravel/compare/5.4...5.5) và chọn bản cập nhật nào quan trọng với bạn.
