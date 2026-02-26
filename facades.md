# Facades

- [Giới thiệu](#introduction)
- [Khi nào dùng Facade](#when-to-use-facades)
    - [Facade và khai báo phụ thuộc](#facades-vs-dependency-injection)
    - [Facade và Helper Functions](#facades-vs-helper-functions)
- [Facade làm việc như thế nào](#how-facades-work)
- [Real-Time Facades](#real-time-facades)
- [Facade Class tham khảo](#facade-class-reference)

<a name="introduction"></a>
## Giới thiệu

Trong suốt tài liệu Laravel, bạn sẽ thấy các ví dụ về code tương tác với các tính năng của Laravel thông qua "facades". Facade cung cấp một "static" interface cho các class có trong [service container](/docs/{{version}}/container) của application. Laravel có sẵn rất nhiều facade cung cấp các quyền truy cập vào hầu hết các tính năng của Laravel.

Facade của Laravel đóng vai trò như là một "static proxies" cho các class cơ bản nằm trong service container, mang lại lợi ích của một cú pháp ngắn gọn, hàm ý trong khi vẫn duy trì khả năng kiểm thử và tính linh hoạt cao so với các phương thức static truyền thống. Sẽ hoàn toàn ổn nếu bạn không hoàn toàn hiểu về cách mà facade hoạt động - chỉ cần tiếp tục và tiếp tục tìm hiểu về Laravel.

Tất cả các facade của Laravel được định nghĩa trong namespace `Illuminate\Support\Facades`. Vì vậy, chúng ta có thể dễ dàng truy cập vào một facade như sau:

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

Trong suốt tài liệu của Laravel, nhiều ví dụ sẽ sử dụng các facade để thực hiện các tính năng khác nhau của framework.

<a name="helper-functions"></a>
#### Helper Functions

Để bổ sung cho các facade, Laravel cung cấp nhiều "helper functions" global giúp cho việc tương tác với các tính năng cơ bản của Laravel trở nên dễ dàng hơn. Một số hàm helper phổ biến mà bạn có thể tương tác là `view`, `response`, `url`, `config`, v.v. Mỗi hàm helper được cung cấp bởi Laravel đều được ghi lại với tính năng tương ứng của chúng; tuy nhiên, có một danh sách đầy đủ có sẵn trong [tài liệu helper](/docs/{{version}}/helpers) chuyên dụng.

Ví dụ: thay vì sử dụng facade `Illuminate\Support\Facades\Response` để tạo một JSON response, chúng ta có thể chỉ cần sử dụng hàm `response`. Vì các hàm helper này là các hàm global, nên bạn không cần khai báo bất kỳ class nào để sử dụng chúng:

```php
use Illuminate\Support\Facades\Response;

Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});

Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

<a name="when-to-use-facades"></a>
## Khi nào dùng Facade

Facade có nhiều lợi ích. Chúng cung cấp một cú pháp ngắn gọn, dễ nhớ cho phép bạn sử dụng các tính năng của Laravel mà không cần phải nhớ các tên class dài sẽ phải khai báo hoặc phải tự cấu hình. Hơn nữa, do cách sử dụng độc đáo của các phương thức động của PHP, chúng rất dễ để test.

Tuy nhiên, một số lưu ý phải được thực hiện khi sử dụng facade. Mối nguy hiểm chính của facade là class bị quá giới hạn. Vì facade rất dễ sử dụng và không cần phải khai báo, nên nó rất dễ để các class của bạn lớn lên và sử dụng nhiều facade trong một class. Việc dùng nhiều khai báo phụ thuộc, làm cho việc phát triển các dòng code trong class của bạn ngày càng lớn hơn. Và vì vậy, khi sử dụng facade, đặc biệt chú ý đến giới hạn của class của bạn để giới hạn của nó ở trong giới hạn cho phép. Nếu class của bạn quá lớn, hãy xem xét việc chia nó thành nhiều class nhỏ hơn.

<a name="facades-vs-dependency-injection"></a>
### Facades và khai báo phụ thuộc

Một trong những lợi ích chính của khai báo phụ thuộc là khả năng để hoán đổi implementation của class được khai báo. Điều này rất hữu ích trong quá trình testing vì bạn có thể tích hợp một giả lập hoặc một khai báo và kiểm tra các hàm khác trên giả lập đó.

Thông thường, không thể giả lập hoặc khai báo một phương thức static class thực sự. Tuy nhiên, vì các facade sử dụng các phương thức động để gọi các phương thức proxy đến các đối tượng được resolve từ service container, nên chúng ta có thể kiểm thử các facade giống như chúng ta đang kiểm tra một instance class đã được tích hợp. Ví dụ: hãy xem route sau:

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

Để sử dụng các phương thức kiểm tra facade của Laravel, chúng ta có thể viết test sau để kiểm tra phương thức `Cache::get` đã được gọi với tham số mà chúng ta mong muốn hay chưa:

```php tab=Pest
use Illuminate\Support\Facades\Cache;

test('basic example', function () {
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
});
```

```php tab=PHPUnit
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="facades-vs-helper-functions"></a>
### Facades và Helper Functions

Ngoài facade, Laravel còn chứa nhiều hàm "helper" để có thể thực hiện các task phổ biến như tạo views, kích hoạt event, gửi job hoặc gửi response HTTP. Nhiều hàm của helper này thực hiện giống với facade tương ứng. Ví dụ: facade này và helper này là tương đương:

```php
return Illuminate\Support\Facades\View::make('profile');

return view('profile');
```

Hoàn toàn không có sự khác biệt giữa facade và helper. Khi sử dụng các helper, bạn vẫn có thể kiểm tra chúng như là bạn làm với facade. Ví dụ, hãy xem route sau:

```php
Route::get('/cache', function () {
    return cache('key');
});
```

Hàm helper `cache` sẽ gọi phương thức `get` trong class facade `Cache`. Vì vậy, mặc dù chúng ta đang sử dụng hàm helper, nhưng chúng ta có thể viết bài kiểm tra như ở dưới để kiểm tra phương thức đã được gọi với tham số mà chúng ta mong muốn hay chưa:

```php
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="how-facades-work"></a>
## Facade làm việc như thế nào

Trong một application Laravel, facade là một class chuyên cung cấp các quyền truy cập các một đối tượng từ container. Điều này được thực hiện bởi cơ chế trong class Facade. Và các Facade của Laravel hay bất kỳ facade nào mà bạn đã tạo sẽ phải extend từ class  `Illuminate\Support\Facades\Facade`.

Class `Facade` sử dụng phương thức magic `__callStatic()` để trì hoãn gọi từ facade của bạn đến một đối tượng được resolve từ container. Trong ví dụ ở dưới đây, sẽ thực hiện gọi đến cache system của Laravel. Nếu bạn chỉ liếc qua, bạn có thể nghĩ rằng phương thức static `get` đang được gọi trong class` Cache`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function showProfile(string $id): View
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

Nhưng hãy lưu ý rằng, ở đầu file chúng ta đã "importing" một facade `Cache`. Facade này đóng vai trò như là một proxy để truy cập vào implementation của interface `Illuminate\Contracts\Cache\Factory`. Bất kỳ hàm gọi nào mà chúng ta thực hiện bằng cách sử dụng facade đều sẽ được chuyển đến instance cache service của Laravel.

Nếu chúng ta nhìn vào class `Illuminate\Support\Facades\Cache`, bạn sẽ thấy rằng không có một phương thức static `get` nào cả:

```php
class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

Thay vào đó, facade `Cache` sẽ được extend từ class `Facade` và định nghĩa một phương thức là `getFacadeAccessor()`. Công việc của phương thức này là trả về tên của một liên kết đã có trong service container. Khi người dùng tham chiếu bất kỳ phương thức tĩnh nào trên facade `Cache`, Laravel sẽ resolve một liên kết có tên là `cache` từ trong [service container](/docs/{{version}}/container) và chạy phương thức được gọi (trong trường hợp này là `get`) trên đối tượng đó.

<a name="real-time-facades"></a>
## Real-Time Facades

Sử dụng real-time facade, bạn có thể coi bất kỳ class nào trong ứng dụng của bạn như là một facade. Để minh họa cách sử dụng này, đầu tiên chúng ta hãy xem một số code không sử dụng facade thời gian thực. Ví dụ: giả sử model `Podcast` của chúng ta có một phương thức là `publish`. Tuy nhiên, để publish một podcast, chúng ta cần khai báo một instance `Publisher`:

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

Việc khai báo một implementation của publisher vào trong phương thức này cho phép chúng ta dễ dàng kiểm tra phương thức này một cách độc lập vì chúng ta có thể giả định publisher đã được khai báo. Tuy nhiên, nó đòi hỏi chúng ta phải luôn truyền vào một instance publisher cho mỗi lần chúng ta gọi phương thức `publish`. Sử dụng các real-time facade, chúng ta có thể duy trì khả năng kiểm tra tương ứng trong khi không bắt buộc phải truyền vào instance của `Publisher`. Để tạo real-time facade, thêm tiền tố namespace của class được import với `Facades`:

```php
<?php

namespace App\Models;

use App\Contracts\Publisher; // [tl! remove]
use Facades\App\Contracts\Publisher; // [tl! add]
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void // [tl! remove]
    public function publish(): void // [tl! add]
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this); // [tl! remove]
        Publisher::publish($this); // [tl! add]
    }
}
```

Khi real-time facade được sử dụng, việc implementation của publisher sẽ được resolve từ service container bằng cách sử dụng phần interface hoặc tên class xuất hiện phía sau tiền tố `Facades`. Khi testing, chúng ta có thể sử dụng helper built-in facade testing của Laravel để giả lập phương thức này được gọi:

```php tab=Pest
<?php

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('podcast can be published', function () {
    $podcast = Podcast::factory()->create();

    Publisher::shouldReceive('publish')->once()->with($podcast);

    $podcast->publish();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A test example.
     */
    public function test_podcast_can_be_published(): void
    {
        $podcast = Podcast::factory()->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

<a name="facade-class-reference"></a>
## Tham khảo Class Facade

Dưới đây bạn có thể tìm thấy mọi facade và class cơ sở nó. Đây là một công cụ hữu ích để nhanh chóng để đào sâu vào tài liệu API cho một facade gốc. [Service container binding](/docs/{{version}}/container) key cũng được kèm theo nếu trong trường hợp bạn cần dùng.

<div class="overflow-auto">

| Facade | Class | Service Container Binding |
| --- | --- | --- |
| App | [Illuminate\Foundation\Application](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Application.html) | `app` |
| Artisan | [Illuminate\Contracts\Console\Kernel](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Console/Kernel.html) | `artisan` |
| Auth (Instance) | [Illuminate\Contracts\Auth\Guard](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Auth/Guard.html) | `auth.driver` |
| Auth | [Illuminate\Auth\AuthManager](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/AuthManager.html) | `auth` |
| Blade | [Illuminate\View\Compilers\BladeCompiler](https://api.laravel.com/docs/{{version}}/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler` |
| Broadcast (Instance) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html) | &nbsp; |
| Broadcast | [Illuminate\Contracts\Broadcasting\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html) | &nbsp; |
| Bus | [Illuminate\Contracts\Bus\Dispatcher](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html) | &nbsp; |
| Cache (Instance) | [Illuminate\Cache\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/Repository.html) | `cache.store` |
| Cache | [Illuminate\Cache\CacheManager](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/CacheManager.html) | `cache` |
| Config | [Illuminate\Config\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Config/Repository.html) | `config` |
| Context | [Illuminate\Log\Context\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Log/Context/Repository.html) | &nbsp; |
| Cookie | [Illuminate\Cookie\CookieJar](https://api.laravel.com/docs/{{version}}/Illuminate/Cookie/CookieJar.html) | `cookie` |
| Crypt | [Illuminate\Encryption\Encrypter](https://api.laravel.com/docs/{{version}}/Illuminate/Encryption/Encrypter.html) | `encrypter` |
| Date | [Illuminate\Support\DateFactory](https://api.laravel.com/docs/{{version}}/Illuminate/Support/DateFactory.html) | `date` |
| DB (Instance) | [Illuminate\Database\Connection](https://api.laravel.com/docs/{{version}}/Illuminate/Database/Connection.html) | `db.connection` |
| DB | [Illuminate\Database\DatabaseManager](https://api.laravel.com/docs/{{version}}/Illuminate/Database/DatabaseManager.html) | `db` |
| Event | [Illuminate\Events\Dispatcher](https://api.laravel.com/docs/{{version}}/Illuminate/Events/Dispatcher.html) | `events` |
| Exceptions (Instance) | [Illuminate\Contracts\Debug\ExceptionHandler](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Debug/ExceptionHandler.html) | &nbsp; |
| Exceptions | [Illuminate\Foundation\Exceptions\Handler](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Exceptions/Handler.html) | &nbsp; |
| File | [Illuminate\Filesystem\Filesystem](https://api.laravel.com/docs/{{version}}/Illuminate/Filesystem/Filesystem.html) | `files` |
| Gate | [Illuminate\Contracts\Auth\Access\Gate](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html) | &nbsp; |
| Hash | [Illuminate\Contracts\Hashing\Hasher](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Hashing/Hasher.html) | `hash` |
| Http | [Illuminate\Http\Client\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Client/Factory.html) | &nbsp; |
| Lang | [Illuminate\Translation\Translator](https://api.laravel.com/docs/{{version}}/Illuminate/Translation/Translator.html) | `translator` |
| Log | [Illuminate\Log\LogManager](https://api.laravel.com/docs/{{version}}/Illuminate/Log/LogManager.html) | `log` |
| Mail | [Illuminate\Mail\Mailer](https://api.laravel.com/docs/{{version}}/Illuminate/Mail/Mailer.html) | `mailer` |
| Notification | [Illuminate\Notifications\ChannelManager](https://api.laravel.com/docs/{{version}}/Illuminate/Notifications/ChannelManager.html) | &nbsp; |
| Password (Instance) | [Illuminate\Auth\Passwords\PasswordBroker](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html) | `auth.password.broker` |
| Password | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password` |
| Pipeline (Instance) | [Illuminate\Pipeline\Pipeline](https://api.laravel.com/docs/{{version}}/Illuminate/Pipeline/Pipeline.html) | &nbsp; |
| Process | [Illuminate\Process\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Process/Factory.html) | &nbsp; |
| Queue (Base Class) | [Illuminate\Queue\Queue](https://api.laravel.com/docs/{{version}}/Illuminate/Queue/Queue.html) | &nbsp; |
| Queue (Instance) | [Illuminate\Contracts\Queue\Queue](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Queue/Queue.html) | `queue.connection` |
| Queue | [Illuminate\Queue\QueueManager](https://api.laravel.com/docs/{{version}}/Illuminate/Queue/QueueManager.html) | `queue` |
| RateLimiter | [Illuminate\Cache\RateLimiter](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/RateLimiter.html) | &nbsp; |
| Redirect | [Illuminate\Routing\Redirector](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Redirector.html) | `redirect` |
| Redis (Instance) | [Illuminate\Redis\Connections\Connection](https://api.laravel.com/docs/{{version}}/Illuminate/Redis/Connections/Connection.html) | `redis.connection` |
| Redis | [Illuminate\Redis\RedisManager](https://api.laravel.com/docs/{{version}}/Illuminate/Redis/RedisManager.html) | `redis` |
| Request | [Illuminate\Http\Request](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Request.html) | `request` |
| Response (Instance) | [Illuminate\Http\Response](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Response.html) | &nbsp; |
| Response | [Illuminate\Contracts\Routing\ResponseFactory](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html) | &nbsp; |
| Route | [Illuminate\Routing\Router](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Router.html) | `router` |
| Schedule | [Illuminate\Console\Scheduling\Schedule](https://api.laravel.com/docs/{{version}}/Illuminate/Console/Scheduling/Schedule.html) | &nbsp; |
| Schema | [Illuminate\Database\Schema\Builder](https://api.laravel.com/docs/{{version}}/Illuminate/Database/Schema/Builder.html) | &nbsp; |
| Session (Instance) | [Illuminate\Session\Store](https://api.laravel.com/docs/{{version}}/Illuminate/Session/Store.html) | `session.store` |
| Session | [Illuminate\Session\SessionManager](https://api.laravel.com/docs/{{version}}/Illuminate/Session/SessionManager.html) | `session` |
| Storage (Instance) | [Illuminate\Contracts\Filesystem\Filesystem](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html) | `filesystem.disk` |
| Storage | [Illuminate\Filesystem\FilesystemManager](https://api.laravel.com/docs/{{version}}/Illuminate/Filesystem/FilesystemManager.html) | `filesystem` |
| URL | [Illuminate\Routing\UrlGenerator](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/UrlGenerator.html) | `url` |
| Validator (Instance) | [Illuminate\Validation\Validator](https://api.laravel.com/docs/{{version}}/Illuminate/Validation/Validator.html) | &nbsp; |
| Validator | [Illuminate\Validation\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Validation/Factory.html) | `validator` |
| View (Instance) | [Illuminate\View\View](https://api.laravel.com/docs/{{version}}/Illuminate/View/View.html) | &nbsp; |
| View | [Illuminate\View\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/View/Factory.html) | `view` |
| Vite | [Illuminate\Foundation\Vite](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Vite.html) | &nbsp; |

</div>
