# Mocking

- [Giới thiệu](#introduction)
- [Bus Fake](#bus-fake)
- [Event Fake](#event-fake)
    - [Scoped Event Fakes](#scoped-event-fakes)
- [Mail Fake](#mail-fake)
- [Notification Fake](#notification-fake)
- [Queue Fake](#queue-fake)
- [Storage Fake](#storage-fake)
- [Facades](#mocking-facades)

<a name="introduction"></a>
## Giới thiệu

Khi test các application của Laravel, bạn có thể muốn "làm giả" các khía cạnh nhất định của application để chúng không thực sự được thực thi trong khi test. Ví dụ: khi test một controller gửi một event, bạn có thể muốn làm giả một event listener để chúng không thực sự được thực thi trong quá trình test. Điều này cho phép bạn chỉ kiểm tra HTTP response của controller mà không phải lo lắng về việc thực thi của event listener, vì các event listener có thể được kiểm tra trong một test case của riêng nó.

Mặc định, Laravel cung cấp helper để làm giả các event, job và facade. Những helper này chủ yếu cung cấp một layer dựa trên Mockery để bạn không phải tự thực hiện các việc gọi phương thức Mockery phức tạp. Tuy nhiên, bạn cũng có thể sử dụng [Mockery](http://docs.mockery.io/en/latest/) hoặc PHPUnit để làm giả hoặc spy của riêng bạn.

<a name="bus-fake"></a>
## Bus Fake

Thay cho việc làm giả, bạn có thể sử dụng phương thức `fake` của facade `Bus` để ngăn job được gửi đi. Khi sử dụng fake, các assertion sẽ được tạo sau khi code test được thực thi:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Bus;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Bus::fake();

            // Perform order shipping...

            Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was not dispatched...
            Bus::assertNotDispatched(AnotherJob::class);
        }
    }

<a name="event-fake"></a>
## Event Fake

Thay cho việc làm giả, bạn có thể sử dụng phương thức `fake` của facade `Event` để ngăn tất cả những event listener sẽ được thực thi. Sau đó, bạn có thể xác nhận các event đã được gửi đi hay chưa và thậm chí kiểm tra dữ liệu mà nó nhận được. Khi sử dụng fake, các assertion sẽ được thực hiện sau khi code test được thực thi:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function testOrderShipping()
        {
            Event::fake();

            // Perform order shipping...

            Event::assertDispatched(OrderShipped::class, function ($e) use ($order) {
                return $e->order->id === $order->id;
            });

            // Assert an event was dispatched twice...
            Event::assertDispatched(OrderShipped::class, 2);

            // Assert an event was not dispatched...
            Event::assertNotDispatched(OrderFailedToShip::class);
        }
    }

> {note} Sau khi bạn gọi `Event::fake()`, thì sẽ không có event listener nào được thực thi. Vì vậy, nếu các bài test của bạn đang sử dụng các model factory mà có dựa vào các event, chẳng hạn như tạo UUID trong event `creating` của một model, thì bạn nên gọi `Event::fake()` **sau khi** sử dụng các factory đó của bạn.

#### Giả một tập hợp các event

Nếu bạn chỉ muốn làm giả event listener cho một nhóm event cụ thể, thì bạn có thể truyền chúng sang phương thức `fake` hoặc `fakeFor`:

    /**
     * Test order process.
     */
    public function testOrderProcess()
    {
        Event::fake([
            OrderCreated::class,
        ]);

        $order = factory(Order::class)->create();

        Event::assertDispatched(OrderCreated::class);

        // Other events are dispatched as normal...
        $order->update([...]);
    }

<a name="scoped-event-fakes"></a>
### Scoped Event Fakes

Nếu bạn chỉ muốn làm giả một event listener trong một phần bài test của bạn, bạn có thể sử dụng phương thức `fakeFor`:

    <?php

    namespace Tests\Feature;

    use App\Order;
    use Tests\TestCase;
    use App\Events\OrderCreated;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        /**
         * Test order process.
         */
        public function testOrderProcess()
        {
            $order = Event::fakeFor(function () {
                $order = factory(Order::class)->create();

                Event::assertDispatched(OrderCreated::class);

                return $order;
            });

            // Events are dispatched as normal and observers will run ...
            $order->update([...]);
        }
    }

<a name="mail-fake"></a>
## Mail Fake

Bạn có thể sử dụng phương thức `fake` của facade `Mail` để ngăn mail được gửi. Sau đó, bạn có thể xác nhận [mailables](/docs/{{version}}/mail) đã được gửi đi cho người dùng hay chưa và thậm chí kiểm tra dữ liệu nó nhận được. Khi sử dụng fake, các assertion được thực hiện sau khi code test được thực thi:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Mail\OrderShipped;
    use Illuminate\Support\Facades\Mail;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Mail::fake();

            // Assert that no mailables were sent...
            Mail::assertNothingSent();

            // Perform order shipping...

            Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
                return $mail->order->id === $order->id;
            });

            // Assert a message was sent to the given users...
            Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
                return $mail->hasTo($user->email) &&
                       $mail->hasCc('...') &&
                       $mail->hasBcc('...');
            });

            // Assert a mailable was sent twice...
            Mail::assertSent(OrderShipped::class, 2);

            // Assert a mailable was not sent...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

Nếu bạn đang sử dụng queue mailable để gửi ở dưới background, bạn nên sử dụng phương thức `assertQueued` thay vì `assertSent`:

    Mail::assertQueued(...);
    Mail::assertNotQueued(...);

<a name="notification-fake"></a>
## Notification Fake

Bạn có thể sử dụng phương thức `fake` của facade `Notification` để ngăn notification được gửi đi. Sau đó, bạn có thể xác nhận [notifications](/docs/{{version}}/notifications) đã được gửi cho người dùng hay chưa và thậm chí kiểm tra dữ liệu nó nhận được. Khi sử dụng fake, các assertion được thực hiện sau khi code test được thực thi:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Notifications\OrderShipped;
    use Illuminate\Support\Facades\Notification;
    use Illuminate\Notifications\AnonymousNotifiable;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Notification::fake();

            // Assert that no notifications were sent...
            Notification::assertNothingSent();

            // Perform order shipping...

            Notification::assertSentTo(
                $user,
                OrderShipped::class,
                function ($notification, $channels) use ($order) {
                    return $notification->order->id === $order->id;
                }
            );

            // Assert a notification was sent to the given users...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // Assert a notification was not sent...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );

            // Assert a notification was sent via Notification::route() method...
            Notification::assertSentTo(
                new AnonymousNotifiable, OrderShipped::class
            );
        }
    }

<a name="queue-fake"></a>
## Queue Fake

Thay cho việc làm giả, bạn có thể sử dụng phương thức `fake` của facade `Queue` để ngăn các job được queue. Sau đó, bạn có thể xác nhận các job đó đã được tạo trong queue hay chưa và thậm chí kiểm tra dữ liệu nó nhận được. Khi sử dụng fake, các assertion sẽ được thực hiện sau khi code test được thực thi:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Queue;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Queue::fake();

            // Assert that no jobs were pushed...
            Queue::assertNothingPushed();

            // Perform order shipping...

            Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was pushed to a given queue...
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // Assert a job was pushed twice...
            Queue::assertPushed(ShipOrder::class, 2);

            // Assert a job was not pushed...
            Queue::assertNotPushed(AnotherJob::class);

            // Assert a job was pushed with a specific chain...
            Queue::assertPushedWithChain(ShipOrder::class, [
                AnotherJob::class,
                FinalJob::class
            ]);
        }
    }

<a name="storage-fake"></a>
## Storage Fake

Phương thức `fake` của facade `Storage` cho phép bạn dễ dàng tạo một disk giả, kết hợp với các tiện ích tạo file của class `UploadedFile`, sẽ giúp đơn giản hóa rất nhiều việc kiểm tra file upload. Ví dụ:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // Assert the file was stored...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

> {tip} Mặc định, phương thức `fake` sẽ xóa tất cả các file trong thư mục temporary của nó. Nếu bạn muốn giữ lại các file này, bạn có thể sử dụng phương thức "persistentFake" thay thế.

<a name="mocking-facades"></a>
## Facades

Không giống như các phương thức static call truyền thống, [facades](/docs/{{version}}/facades) có thể bị làm giả. Điều này cung cấp một lợi thế lớn so với các phương thức static truyền thống và cho phép bạn khả năng test nếu bạn đang sử dụng khai báo phụ thuộc. Khi test, bạn có thể muốn làm giả việc gọi đến facade của Laravel trong controller của bạn. Ví dụ, hãy xem hành động của controller sau:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

Chúng ta có thể làm giả việc gọi đến facade `Cache` bằng cách sử dụng phương thức `shouldReceive`, nó sẽ trả về một instance giả của [Mockery](https://github.com/padraic/mockery). Vì các facade được resolve và quản lý bởi [service container](/docs/{{version}}/container), nên chúng có khả năng test cao hơn nhiều so với một class static thông thường. Ví dụ: chúng ta hãy làm giả việc gọi đến phương thức `get` của facade `Cache`:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class UserControllerTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> {note} Bạn không nên làm giả facade `Request`. Thay vào đó, hãy truyền input mà bạn mong muốn vào phương thức của HTTP helper, chẳng hạn như `get` và `post` khi chạy test của bạn. Tương tự như vậy, thay vì làm giả facade `Config`, hãy gọi phương thức `Config::set` trong các test của bạn.
