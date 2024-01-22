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

    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\Facades\Route;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Trong suốt tài liệu của Laravel, nhiều ví dụ sẽ sử dụng các facade để thực hiện các tính năng khác nhau của framework.

<a name="helper-functions"></a>
#### Helper Functions

Để bổ sung cho các facade, Laravel cung cấp nhiều "helper functions" global giúp cho việc tương tác với các tính năng cơ bản của Laravel trở nên dễ dàng hơn. Một số hàm helper phổ biến mà bạn có thể tương tác là `view`, `response`, `url`, `config`, v.v. Mỗi hàm helper được cung cấp bởi Laravel đều được ghi lại với tính năng tương ứng của chúng; tuy nhiên, có một danh sách đầy đủ có sẵn trong [tài liệu helper](/docs/{{version}}/helpers) chuyên dụng.

Ví dụ: thay vì sử dụng facade `Illuminate\Support\Facades\Response` để tạo một JSON response, chúng ta có thể chỉ cần sử dụng hàm `response`. Vì các hàm helper này là các hàm global, nên bạn không cần khai báo bất kỳ class nào để sử dụng chúng:

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

<a name="when-to-use-facades"></a>
## Khi nào dùng Facade

Facade có nhiều lợi ích. Chúng cung cấp một cú pháp ngắn gọn, dễ nhớ cho phép bạn sử dụng các tính năng của Laravel mà không cần phải nhớ các tên class dài sẽ phải khai báo hoặc phải tự cấu hình. Hơn nữa, do cách sử dụng độc đáo của các phương thức động của PHP, chúng rất dễ để test.

Tuy nhiên, một số lưu ý phải được thực hiện khi sử dụng facade. Mối nguy hiểm chính của facade là class bị quá giới hạn. Vì facade rất dễ sử dụng và không cần phải khai báo, nên nó rất dễ để các class của bạn lớn lên và sử dụng nhiều facade trong một class. Việc dùng nhiều khai báo phụ thuộc, làm cho việc phát triển các dòng code trong class của bạn ngày càng lớn hơn. Và vì vậy, khi sử dụng facade, đặc biệt chú ý đến giới hạn của class của bạn để giới hạn của nó ở trong giới hạn cho phép. Nếu class của bạn quá lớn, hãy xem xét việc chia nó thành nhiều class nhỏ hơn.

<a name="facades-vs-dependency-injection"></a>
### Facades và khai báo phụ thuộc

Một trong những lợi ích chính của khai báo phụ thuộc là khả năng để hoán đổi implementation của class được khai báo. Điều này rất hữu ích trong quá trình testing vì bạn có thể tích hợp một giả lập hoặc một khai báo và kiểm tra các hàm khác trên giả lập đó.

Thông thường, không thể giả lập hoặc khai báo một phương thức static class thực sự. Tuy nhiên, vì các facade sử dụng các phương thức động để gọi các phương thức proxy đến các đối tượng được resolve từ service container, nên chúng ta có thể kiểm thử các facade giống như chúng ta đang kiểm tra một instance class đã được tích hợp. Ví dụ: hãy xem route sau:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Để sử dụng các phương thức kiểm tra facade của Laravel, chúng ta có thể viết test sau để kiểm tra phương thức `Cache::get` đã được gọi với tham số mà chúng ta mong muốn hay chưa:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="facades-vs-helper-functions"></a>
### Facades và Helper Functions

Ngoài facade, Laravel còn chứa nhiều hàm "helper" để có thể thực hiện các task phổ biến như tạo views, kích hoạt event, gửi job hoặc gửi response HTTP. Nhiều hàm của helper này thực hiện giống với facade tương ứng. Ví dụ: facade này và helper này là tương đương:

    return Illuminate\Support\Facades\View::make('profile');

    return view('profile');

Hoàn toàn không có sự khác biệt giữa facade và helper. Khi sử dụng các helper, bạn vẫn có thể kiểm tra chúng như là bạn làm với facade. Ví dụ, hãy xem route sau:

    Route::get('/cache', function () {
        return cache('key');
    });

Ở trong route cache ở trên, hàm helper `cache` sẽ gọi phương thức `get` trong class facade `Cache`. Vì vậy, mặc dù chúng ta đang sử dụng hàm helper, nhưng chúng ta có thể viết bài kiểm tra như ở dưới để kiểm tra phương thức đã được gọi với tham số mà chúng ta mong muốn hay chưa:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="how-facades-work"></a>
## Facade làm việc như thế nào

Trong một application Laravel, facade là một class chuyên cung cấp các quyền truy cập các một đối tượng từ container. Điều này được thực hiện bởi cơ chế trong class Facade. Và các Facade của Laravel hay bất kỳ facade nào mà bạn đã tạo sẽ phải extend từ class  `Illuminate\Support\Facades\Facade`.

Class `Facade` sử dụng phương thức magic `__callStatic()` để trì hoãn gọi từ facade của bạn đến một đối tượng được resolve từ container. Trong ví dụ ở dưới đây, sẽ thực hiện gọi đến cache system của Laravel. Nếu bạn chỉ liếc qua, bạn có thể nghĩ rằng phương thức static `get` đang được gọi trong class` Cache`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

Nhưng hãy lưu ý rằng, ở đầu file chúng ta đã "importing" một facade `Cache`. Facade này đóng vai trò như là một proxy để truy cập vào implementation của interface `Illuminate\Contracts\Cache\Factory`. Bất kỳ hàm gọi nào mà chúng ta thực hiện bằng cách sử dụng facade đều sẽ được chuyển đến instance cache service của Laravel.

Nếu chúng ta nhìn vào class `Illuminate\Support\Facades\Cache`, bạn sẽ thấy rằng không có một phương thức static `get` nào cả:

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

Thay vào đó, facade `Cache` sẽ được extend từ class `Facade` và định nghĩa một phương thức là `getFacadeAccessor()`. Công việc của phương thức này là trả về tên của một liên kết đã có trong service container. Khi người dùng tham chiếu bất kỳ phương thức tĩnh nào trên facade `Cache`, Laravel sẽ resolve một liên kết có tên là `cache` từ trong [service container](/docs/{{version}}/container) và chạy phương thức được gọi (trong trường hợp này là `get`) trên đối tượng đó.

<a name="real-time-facades"></a>
## Real-Time Facades

Sử dụng real-time facade, bạn có thể coi bất kỳ class nào trong ứng dụng của bạn như là một facade. Để minh họa cách sử dụng này, đầu tiên chúng ta hãy xem một số code không sử dụng facade thời gian thực. Ví dụ: giả sử model `Podcast` của chúng ta có một phương thức là `publish`. Tuy nhiên, để publish một podcast, chúng ta cần khai báo một instance `Publisher`:

    <?php

    namespace App\Models;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @param  Publisher  $publisher
         * @return void
         */
        public function publish(Publisher $publisher)
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

Việc khai báo một implementation của publisher vào trong phương thức này cho phép chúng ta dễ dàng kiểm tra phương thức này một cách độc lập vì chúng ta có thể giả định publisher đã được khai báo. Tuy nhiên, nó đòi hỏi chúng ta phải luôn truyền vào một instance publisher cho mỗi lần chúng ta gọi phương thức `publish`. Sử dụng các real-time facade, chúng ta có thể duy trì khả năng kiểm tra tương ứng trong khi không bắt buộc phải truyền vào instance của `Publisher`. Để tạo real-time facade, thêm tiền tố namespace của class được import với `Facades`:

    <?php

    namespace App\Models;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @return void
         */
        public function publish()
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

Khi real-time facade được sử dụng, việc implementation của publisher sẽ được resolve từ service container bằng cách sử dụng phần interface hoặc tên class xuất hiện phía sau tiền tố `Facades`. Khi testing, chúng ta có thể sử dụng helper built-in facade testing của Laravel để giả lập phương thức này được gọi:

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
         *
         * @return void
         */
        public function test_podcast_can_be_published()
        {
            $podcast = Podcast::factory()->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>
## Tham khảo Class Facade

Dưới đây bạn có thể tìm thấy mọi facade và class cơ sở nó. Đây là một công cụ hữu ích để nhanh chóng để đào sâu vào tài liệu API cho một facade gốc. [Service container binding](/docs/{{version}}/container) key cũng được kèm theo nếu trong trường hợp bạn cần dùng.

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  |  `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html)  |  `auth.driver`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Broadcast  |  [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html)  |  &nbsp;
Broadcast (Instance)  |  [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html)  |  &nbsp;
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html)  |  `cache`
Cache (Instance)  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache.store`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
Date  |  [Illuminate\Support\DateFactory](https://laravel.com/api/{{version}}/Illuminate/Support/DateFactory.html)  |  `date`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Http  |  [Illuminate\Http\Client\Factory](https://laravel.com/api/{{version}}/Illuminate/Http/Client/Factory.html)  |  &nbsp;
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html)  |  `redis`
Redis (Instance)  |  [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html)  |  `redis.connection`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Response (Instance)  |  [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  `session.store`
Storage  |  [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html)  |  `filesystem`
Storage (Instance)  |  [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html)  |  `filesystem.disk`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html)  |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  &nbsp;
