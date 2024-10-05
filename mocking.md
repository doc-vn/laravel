# Mocking

- [Giới thiệu](#introduction)
- [Giả Object](#mocking-objects)
- [Giả Facades](#mocking-facades)
    - [Facade Spies](#facade-spies)
- [Tương tác với Time](#interacting-with-time)

<a name="introduction"></a>
## Giới thiệu

Khi test các application của Laravel, bạn có thể muốn "làm giả" các khía cạnh nhất định của application để chúng không thực sự được thực thi trong khi test. Ví dụ: khi test một controller gửi một event, bạn có thể muốn làm giả một event listener để chúng không thực sự được thực thi trong quá trình test. Điều này cho phép bạn chỉ kiểm tra HTTP response của controller mà không phải lo lắng về việc thực thi của event listener vì các event listener có thể được kiểm tra trong một test case của riêng nó.

Mặc định, Laravel cung cấp các phương thức helper để làm giả các event, job và các facade khác. Những helper này chủ yếu cung cấp một layer dựa trên Mockery để bạn không phải tự thực hiện các việc gọi phương thức Mockery phức tạp.

<a name="mocking-objects"></a>
## Giả Object

Khi giả một đối tượng sẽ được tích hợp vào ứng dụng của bạn thông qua [service container](/docs/{{version}}/container) của Laravel, bạn sẽ cần phải liên kết instance giả của bạn vào container dưới dạng liên kết `instance`. Điều này sẽ hướng dẫn container sử dụng instance đối tượng được làm giả của bạn thay vì khởi tạo chính đối tượng đó:

    use App\Service;
    use Mockery;
    use Mockery\MockInterface;

    public function test_something_can_be_mocked(): void
    {
        $this->instance(
            Service::class,
            Mockery::mock(Service::class, function (MockInterface $mock) {
                $mock->shouldReceive('process')->once();
            })
        );
    }

Để làm cho việc này thuận tiện hơn, bạn có thể sử dụng phương thức `mock` được cung cấp bởi class test case của Laravel. Trong ví dụ ở dưới đây sẽ tương đương với ví dụ ở trên:

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->mock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

Bạn có thể sử dụng phương thức `partialMock` khi bạn chỉ cần làm giả một vài phương thức của một đối tượng. Các phương thức không bị làm giả sẽ được thực thi bình thường khi được gọi:

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->partialMock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

Tương tự, nếu bạn muốn [spy](http://docs.mockery.io/en/latest/reference/spies.html) một đối tượng, class test case của Laravel cũng cung cấp phương thức `spy` là một phương thức wrapper cho phương thức `Mockery::spy`. Spy cũng giống như những mock; tuy nhiên, spy sẽ ghi lại mọi tương tác giữa spy và code đang được kiểm tra, cho phép bạn đưa ra yêu cầu sau khi code được chạy xong:

    use App\Service;

    $spy = $this->spy(Service::class);

    // ...

    $spy->shouldHaveReceived('process');

<a name="mocking-facades"></a>
## Giả Facades

Không giống như các phương thức static call truyền thống, [facades](/docs/{{version}}/facades) (bao gồm cả [real-time facades](/docs/{{version}}/facades#real-time-facades)) cũng có thể bị làm giả. Điều này cung cấp một lợi thế lớn so với các phương thức static truyền thống và cho phép bạn khả năng test nếu bạn đang sử dụng khai báo phụ thuộc. Khi test, bạn có thể muốn làm giả việc gọi đến facade của Laravel trong controller của bạn. Ví dụ, hãy xem hành động của controller sau:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Retrieve a list of all users of the application.
         */
        public function index(): array
        {
            $value = Cache::get('key');

            return [
                // ...
            ];
        }
    }

Chúng ta có thể làm giả việc gọi đến facade `Cache` bằng cách sử dụng phương thức `shouldReceive`, nó sẽ trả về một instance giả của [Mockery](https://github.com/padraic/mockery). Vì các facade được resolve và quản lý bởi [service container](/docs/{{version}}/container), nên chúng có khả năng test cao hơn nhiều so với một class static thông thường. Ví dụ: chúng ta hãy làm giả việc gọi đến phương thức `get` của facade `Cache`:

    <?php

    namespace Tests\Feature;

    use Illuminate\Support\Facades\Cache;
    use Tests\TestCase;

    class UserControllerTest extends TestCase
    {
        public function test_get_index(): void
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> [!WARNING]
> Bạn không nên làm giả facade `Request`. Thay vào đó, hãy truyền input mà bạn mong muốn vào [phương thức của HTTP helper](/docs/{{version}}/http-tests), chẳng hạn như `get` và `post` khi chạy test của bạn. Tương tự như vậy, thay vì làm giả facade `Config`, hãy gọi phương thức `Config::set` trong các test của bạn.

<a name="facade-spies"></a>
### Facade Spies

Nếu bạn muốn [spy](http://docs.mockery.io/en/latest/reference/spies.html) trên một facade, bạn có thể gọi phương thức `spy` trên facade tương ứng. Spy cũng giống như những mock; tuy nhiên, spy sẽ ghi lại mọi tương tác giữa spy và code đang được kiểm tra, cho phép bạn đưa ra yêu cầu sau khi code được chạy xong:

    use Illuminate\Support\Facades\Cache;

    public function test_values_are_be_stored_in_cache(): void
    {
        Cache::spy();

        $response = $this->get('/');

        $response->assertStatus(200);

        Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
    }

<a name="interacting-with-time"></a>
## Tương tác với Time

Khi kiểm tra, đôi khi bạn có thể cần sửa thời gian được trả về bởi helper, chẳng hạn như `now` hoặc `Illuminate\Support\Carbon::now()`. Rất may, class kiểm tra cơ bản của Laravel đã chứa các helper cho phép bạn thao tác với thời gian hiện tại:

    use Illuminate\Support\Carbon;

    public function test_time_can_be_manipulated(): void
    {
        // Travel into the future...
        $this->travel(5)->milliseconds();
        $this->travel(5)->seconds();
        $this->travel(5)->minutes();
        $this->travel(5)->hours();
        $this->travel(5)->days();
        $this->travel(5)->weeks();
        $this->travel(5)->years();

        // Travel into the past...
        $this->travel(-5)->hours();

        // Travel to an explicit time...
        $this->travelTo(now()->subHours(6));

        // Return back to the present time...
        $this->travelBack();
    }

Bạn cũng có thể cung cấp một closure cho các phương thức di chuyển thời gian. Closure sẽ được gọi với thời gian đã được chỉ định. Sau khi closure được chạy, thời gian sẽ tiếp tục trở về bình thường:

    $this->travel(5)->days(function () {
        // Test something five days into the future...
    });

    $this->travelTo(now()->subDays(10), function () {
        // Test something during a given moment...
    });

Phương thức `freezeTime` có thể được sử dụng để giữ thời gian hiện tại. Tương tự, phương thức `freezeSecond` sẽ giữ thời gian hiện tại nhưng tại thời điểm bắt đầu của giây:

    use Illuminate\Support\Carbon;

    // Freeze time and resume normal time after executing closure...
    $this->freezeTime(function (Carbon $time) {
        // ...
    });

    // Freeze time at the current second and resume normal time after executing closure...
    $this->freezeSecond(function (Carbon $time) {
        // ...
    })

Như bạn có thể thấy, tất cả các phương thức được thảo luận ở trên chủ yếu hữu ích để kiểm tra hành động của ứng dụng mà nhạy cảm với thời gian, chẳng hạn như khóa các bài đăng không hoạt động trên một diễn đàn:

    use App\Models\Thread;

    public function test_forum_threads_lock_after_one_week_of_inactivity()
    {
        $thread = Thread::factory()->create();

        $this->travel(1)->week();

        $this->assertTrue($thread->isLockedByInactivity());
    }
