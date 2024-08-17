# Mocking

- [Giới thiệu](#introduction)
- [Giả Object](#mocking-objects)
- [Giả Facades](#mocking-facades)
    - [Facade Spies](#facade-spies)
- [Bus Fake](#bus-fake)
    - [Job Chains](#bus-job-chains)
    - [Job Batches](#job-batches)
- [Event Fake](#event-fake)
    - [Scoped Event Fakes](#scoped-event-fakes)
- [HTTP Fake](#http-fake)
- [Mail Fake](#mail-fake)
- [Notification Fake](#notification-fake)
- [Queue Fake](#queue-fake)
    - [Job Chains](#job-chains)
- [Storage Fake](#storage-fake)
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

    public function test_something_can_be_mocked()
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
         *
         * @return \Illuminate\Http\Response
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

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Cache;
    use Tests\TestCase;

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

> **Warning**
> Bạn không nên làm giả facade `Request`. Thay vào đó, hãy truyền input mà bạn mong muốn vào [phương thức của HTTP helper](/docs/{{version}}/http-tests), chẳng hạn như `get` và `post` khi chạy test của bạn. Tương tự như vậy, thay vì làm giả facade `Config`, hãy gọi phương thức `Config::set` trong các test của bạn.

<a name="facade-spies"></a>
### Facade Spies

Nếu bạn muốn [spy](http://docs.mockery.io/en/latest/reference/spies.html) trên một facade, bạn có thể gọi phương thức `spy` trên facade tương ứng. Spy cũng giống như những mock; tuy nhiên, spy sẽ ghi lại mọi tương tác giữa spy và code đang được kiểm tra, cho phép bạn đưa ra yêu cầu sau khi code được chạy xong:

    use Illuminate\Support\Facades\Cache;

    public function test_values_are_be_stored_in_cache()
    {
        Cache::spy();

        $response = $this->get('/');

        $response->assertStatus(200);

        Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
    }

<a name="bus-fake"></a>
## Bus Fake

Khi kiểm tra code gửi job, bạn thường muốn yêu cầu rằng một job nhất định đã được gửi nhưng không thực sự được queue hoặc thực thi job đó. Điều này là do việc thực thi job thường có thể được kiểm tra trong một class kiểm tra riêng biệt.

Bạn có thể sử dụng phương thức `fake` của facade `Bus` để ngăn job đó được gửi đi. Sau đó, sau khi chạy xong code đang được kiểm tra, bạn có thể kiểm tra những job mà ứng dụng đã cố gửi đi bằng cách sử dụng các phương thức `assertDispatched` và `assertNotDispatched`:

    <?php

    namespace Tests\Feature;

    use App\Jobs\ShipOrder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Bus;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped()
        {
            Bus::fake();

            // Perform order shipping...

            // Assert that a job was dispatched...
            Bus::assertDispatched(ShipOrder::class);

            // Assert a job was not dispatched...
            Bus::assertNotDispatched(AnotherJob::class);

            // Assert that a job was dispatched synchronously...
            Bus::assertDispatchedSync(AnotherJob::class);

            // Assert that a job was not dispatched synchronously...
            Bus::assertNotDispatchedSync(AnotherJob::class);

            // Assert that a job was dispatched after the response was sent...
            Bus::assertDispatchedAfterResponse(AnotherJob::class);

            // Assert a job was not dispatched after response was sent...
            Bus::assertNotDispatchedAfterResponse(AnotherJob::class);

            // Assert no jobs were dispatched...
            Bus::assertNothingDispatched();
        }
    }

Bạn có thể truyền một closure cho các phương thức có sẵn để yêu cầu một job đã được gửi đi phải vượt qua một "truth test" nhất định. Nếu ít nhất một job được gửi đi vượt qua bài kiểm tra truth test thì kiểm tra sẽ thành công. Ví dụ: bạn có thể muốn yêu cầu một job đã được gửi đi cho một đơn hàng cụ thể:

    Bus::assertDispatched(function (ShipOrder $job) use ($order) {
        return $job->order->id === $order->id;
    });

<a name="faking-a-subset-of-jobs"></a>
#### Faking A Subset Of Jobs

Nếu bạn muốn ngăn gửi đi một số job nhất định, bạn có thể truyền các job giả cho phương thức `fake`:

    /**
     * Test order process.
     */
    public function test_orders_can_be_shipped()
    {
        Bus::fake([
            ShipOrder::class,
        ]);

        // ...
    }

Bạn có thể làm giả tất cả các job ngoại trừ một số job được chỉ định trong phương thức `except`:

    Bus::fake()->except([
        ShipOrder::class,
    ]);

<a name="bus-job-chains"></a>
### Job Chains

Phương thức `assertChained` của facade `Bus` có thể được sử dụng để yêu cầu một [chuỗi job](/docs/{{version}}/queues#job-chaining) đã được gửi đi. Phương thức `assertChained` sẽ chấp nhận vào một loạt các job được nối với nhau làm tham số đầu tiên:

    use App\Jobs\RecordShipment;
    use App\Jobs\ShipOrder;
    use App\Jobs\UpdateInventory;
    use Illuminate\Support\Facades\Bus;

    Bus::assertChained([
        ShipOrder::class,
        RecordShipment::class,
        UpdateInventory::class
    ]);

Như bạn có thể thấy trong ví dụ trên, một mảng chained job có thể là một mảng gồm tên class của các job. Tuy nhiên, bạn cũng có thể cung cấp một loạt các instance job thực tế. Khi làm như vậy, Laravel sẽ đảm bảo rằng các instance job thuộc cùng class và có cùng giá trị thuộc tính của các chained job được ứng dụng của bạn gửi đi:

    Bus::assertChained([
        new ShipOrder,
        new RecordShipment,
        new UpdateInventory,
    ]);

<a name="job-batches"></a>
### Job Batches

Phương thức `assertBatched` của facade `Bus` có thể được sử dụng để yêu cầu một [loạt job](/docs/{{version}}/queues#job-batching) đã được gửi đi. Closure được cung cấp cho phương thức `assertBatched` sẽ nhận được một instance của `Illuminate\Bus\PendingBatch`, có thể được sử dụng để kiểm tra các job có trong batch:

    use Illuminate\Bus\PendingBatch;
    use Illuminate\Support\Facades\Bus;

    Bus::assertBatched(function (PendingBatch $batch) {
        return $batch->name == 'import-csv' &&
               $batch->jobs->count() === 10;
    });

<a name="testing-job-batch-interaction"></a>
#### Testing Job / Batch Interaction

Ngoài ra, đôi khi bạn có thể cần kiểm tra tương tác của một job với batch của nó. Ví dụ, bạn có thể cần kiểm tra xem một job có thể huỷ xử lý tiếp theo của batch của nó hay không. Để thực hiện việc này, bạn cần chỉ định một batch giả cho job thông qua phương thức `withFakeBatch`. Phương thức `withFakeBatch` trả về một bộ chứa instance job và batch giả:

    [$job, $batch] = (new ShipOrder)->withFakeBatch();

    $job->handle();

    $this->assertTrue($batch->cancelled());
    $this->assertEmpty($batch->added);

<a name="event-fake"></a>
## Event Fake

Khi test tra code gửi event, bạn có thể muốn Laravel không thực sự chạy listener của event. Bằng cách sử dụng phương thức `fake` của facade `Event`, bạn có thể ngăn listener thực thi, chạy code đang được test và sau đó xác nhận những event nào đã được ứng dụng của bạn gửi đi bằng cách sử dụng các phương thức `assertDispatched`, `assertNotDispatched` và `assertNothingDispatched`:

    <?php

    namespace Tests\Feature;

    use App\Events\OrderFailedToShip;
    use App\Events\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Event;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function test_orders_can_be_shipped()
        {
            Event::fake();

            // Perform order shipping...

            // Assert that an event was dispatched...
            Event::assertDispatched(OrderShipped::class);

            // Assert an event was dispatched twice...
            Event::assertDispatched(OrderShipped::class, 2);

            // Assert an event was not dispatched...
            Event::assertNotDispatched(OrderFailedToShip::class);

            // Assert that no events were dispatched...
            Event::assertNothingDispatched();
        }
    }

Bạn có thể truyền một closure cho các phương thức `assertDispatched` hoặc `assertNotDispatched` để yêu cầu một event đã được gửi đi đã vượt qua một bài kiểm tra "truth test". Nếu ít nhất một event được gửi đi vượt qua bài kiểm tra "truth test" thì bài kiểm tra đó sẽ thành công:

    Event::assertDispatched(function (OrderShipped $event) use ($order) {
        return $event->order->id === $order->id;
    });

Nếu bạn chỉ muốn yêu cầu event listener đang lắng nghe một event nhất định, bạn có thể sử dụng phương thức `assertListening`:

    Event::assertListening(
        OrderShipped::class,
        SendShipmentNotification::class
    );

> **Warning**
> Sau khi bạn gọi `Event::fake()`, thì sẽ không có event listener nào được thực thi. Vì vậy, nếu các bài test của bạn đang sử dụng các model factory mà có dựa vào các event, chẳng hạn như tạo UUID trong event `creating` của một model, thì bạn nên gọi `Event::fake()` **sau khi** sử dụng các factory đó của bạn.

<a name="faking-a-subset-of-events"></a>
#### Giả một tập hợp các event

Nếu bạn chỉ muốn làm giả event listener cho một nhóm event cụ thể, thì bạn có thể truyền chúng sang phương thức `fake` hoặc `fakeFor`:

    /**
     * Test order process.
     */
    public function test_orders_can_be_processed()
    {
        Event::fake([
            OrderCreated::class,
        ]);

        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        // Other events are dispatched as normal...
        $order->update([...]);
    }

Bạn có thể làm giả tất cả các event ngoại trừ một số event được chỉ định trong phương thức `except`:

    Event::fake()->except([
        OrderCreated::class,
    ]);

<a name="scoped-event-fakes"></a>
### Scoped Event Fakes

Nếu bạn chỉ muốn làm giả một event listener trong một phần bài test của bạn, bạn có thể sử dụng phương thức `fakeFor`:

    <?php

    namespace Tests\Feature;

    use App\Events\OrderCreated;
    use App\Models\Order;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Test order process.
         */
        public function test_orders_can_be_processed()
        {
            $order = Event::fakeFor(function () {
                $order = Order::factory()->create();

                Event::assertDispatched(OrderCreated::class);

                return $order;
            });

            // Events are dispatched as normal and observers will run ...
            $order->update([...]);
        }
    }

<a name="http-fake"></a>
## HTTP Fake

Phương thức `fake` của facade `Http` cho phép bạn hướng dẫn HTTP client trả về các stubbed hoặc dummy response khi một request được tạo. Để biết thêm thông tin chi tiết về việc faking các request HTTP được gửi đi, vui lòng tham khảo [tài liệu testing HTTP Client](/docs/{{version}}/http-client#testing).

<a name="mail-fake"></a>
## Mail Fake

Bạn có thể sử dụng phương thức `fake` của facade `Mail` để ngăn mail được gửi. Thông thường, việc gửi mail sẽ không liên quan đến đoạn code mà bạn đang thực sự kiểm tra. Rất có thể, chỉ cần yêu cầu Laravel đã gửi một mail nhất định là đủ.

Sau khi, bạn gọi phương thức `fake` của facade `Mail`, bạn có thể xác nhận [mailables](/docs/{{version}}/mail) đã được gửi đi cho người dùng hay chưa và thậm chí kiểm tra dữ liệu nó nhận được.

    <?php

    namespace Tests\Feature;

    use App\Mail\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Mail;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped()
        {
            Mail::fake();

            // Perform order shipping...

            // Assert that no mailables were sent...
            Mail::assertNothingSent();

            // Assert that a mailable was sent...
            Mail::assertSent(OrderShipped::class);

            // Assert a mailable was sent twice...
            Mail::assertSent(OrderShipped::class, 2);

            // Assert a mailable was not sent...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

Nếu bạn đang sử dụng queue mailable để gửi ở dưới background, bạn nên sử dụng phương thức `assertQueued` thay vì `assertSent`:

    Mail::assertQueued(OrderShipped::class);

    Mail::assertNotQueued(OrderShipped::class);

    Mail::assertNothingQueued();

Bạn có thể truyền một closure cho các phương thức `assertSent`, `assertNotSent`, `assertQueued` hoặc `assertNotQueued` để yêu cầu một mail đã vượt qua bài kiểm tra "truth test" nhất định. Nếu ít nhất một mail được gửi đi vượt qua bài kiểm tra "truth test" thì bài kiểm tra đó sẽ thành công:

    Mail::assertSent(function (OrderShipped $mail) use ($order) {
        return $mail->order->id === $order->id;
    });

Khi gọi các phương thức xác nhận của facade `Mail`, instance mailable được chấp nhận bởi closure sẽ cung cấp các phương thức hữu ích để kiểm tra mail:

    Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
        return $mail->hasTo($user->email) &&
               $mail->hasCc('...') &&
               $mail->hasBcc('...') &&
               $mail->hasReplyTo('...') &&
               $mail->hasFrom('...') &&
               $mail->hasSubject('...');
    });

Instance mailable cũng chứa một số phương thức hữu ích để bạn có thể kiểm tra file đính kèm trong mail:

    use Illuminate\Mail\Mailables\Attachment;

    Mail::assertSent(OrderShipped::class, function ($mail) {
        return $mail->hasAttachment(
            Attachment::fromPath('/path/to/file')
                    ->as('name.pdf')
                    ->withMime('application/pdf')
        );
    });

    Mail::assertSent(OrderShipped::class, function ($mail) {
        return $mail->hasAttachment(
            Attachment::fromStorageDisk('s3', '/path/to/file')
        );
    });

    Mail::assertSent(OrderShipped::class, function ($mail) use ($pdfData) {
        return $mail->hasAttachment(
            Attachment::fromData(fn () => $pdfData, 'name.pdf')
        );
    });

Bạn có thể nhận thấy có hai phương thức để xác nhận mail chưa được gửi: `assertNotSent` và `assertNotQueued`. Đôi khi bạn có thể muốn yêu cầu không có mail nào được gửi **hoặc** được queued. Để thực hiện điều này, bạn có thể sử dụng các phương thức `assertNothingOutgoing` và `assertNotOutgoing`:

    Mail::assertNothingOutgoing();

    Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
        return $mail->order->id === $order->id;
    });

<a name="testing-mailable-content"></a>
#### Testing Mailable Content

Chúng tôi đề xuất việc kiểm tra nội dung của mailable sẽ riêng biệt với các bài kiểm tra một mailable đã được "gửi" đến một người dùng cụ thể hay chưa. Để tìm hiểu thêm về cách kiểm tra nội dung của mailable, hãy xem tài liệu của chúng tôi về [testing mailables](/docs/{{version}}/mail#testing-mailables).

<a name="notification-fake"></a>
## Notification Fake

Bạn có thể sử dụng phương thức `fake` của facade `Notification` để ngăn notification được gửi đi. Thông thường, việc gửi thông báo sẽ không liên quan đến đoạn code mà bạn đang thực sự kiểm tra. Rất có thể, chỉ cần yêu cầu Laravel đã gửi một thông báo nhất định là đủ.

Sau khi, bạn gọi phương thức `fake` của facade `Notification`, bạn có thể xác nhận [notifications](/docs/{{version}}/notifications) đã được gửi đi cho người dùng hay chưa và thậm chí kiểm tra dữ liệu nó nhận được.

    <?php

    namespace Tests\Feature;

    use App\Notifications\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Notification;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped()
        {
            Notification::fake();

            // Perform order shipping...

            // Assert that no notifications were sent...
            Notification::assertNothingSent();

            // Assert a notification was sent to the given users...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // Assert a notification was not sent...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );

            // Assert that a given number of notifications were sent...
            Notification::assertCount(3);
        }
    }

Bạn có thể truyền một closure cho các phương thức `assertSentTo` hoặc `assertNotSentTo` để yêu cầu một thông báo đã vượt qua bài kiểm tra "truth test" nhất định. Nếu ít nhất một thông báo được gửi đi vượt qua bài kiểm tra "truth test" thì bài kiểm tra đó sẽ thành công:

    Notification::assertSentTo(
        $user,
        function (OrderShipped $notification, $channels) use ($order) {
            return $notification->order->id === $order->id;
        }
    );

<a name="on-demand-notifications"></a>
#### On-Demand Notifications

Nếu code bạn đang kiểm tra việc gửi một [thông báo theo yêu cầu](/docs/{{version}}/notifications#on-demand-notifications), bạn có thể kiểm tra xem thông báo theo yêu cầu đã được gửi hay chưa qua phương thức `assertSentOnDemand`:

    Notification::assertSentOnDemand(OrderShipped::class);

Bằng cách truyền vào một closure làm tham số thứ hai cho phương thức `assertSentOnDemand`, bạn có thể xác định xem thông báo theo yêu cầu đó có được gửi đến đúng địa chỉ "route" hay không:

    Notification::assertSentOnDemand(
        OrderShipped::class,
        function ($notification, $channels, $notifiable) use ($user) {
            return $notifiable->routes['mail'] === $user->email;
        }
    );

<a name="queue-fake"></a>
## Queue Fake

Bạn có thể sử dụng phương thức `fake` của facade `Queue` để ngăn chặn các queued job bị đẩy vào queue. Rất có thể, bạn chỉ cần kiểm tra Laravel đã đẩy một job nhất định vào một queue cụ thể là đủ vì bản thân các queued job đó có thể được kiểm tra trong một class kiểm thử khác.

Sau khi gọi phương thức `fake` của facade `Queue`, bạn có thể kiểm tra ứng dụng đã đẩy các job vào queue hay chưa:

    <?php

    namespace Tests\Feature;

    use App\Jobs\AnotherJob;
    use App\Jobs\FinalJob;
    use App\Jobs\ShipOrder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Queue;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped()
        {
            Queue::fake();

            // Perform order shipping...

            // Assert that no jobs were pushed...
            Queue::assertNothingPushed();

            // Assert a job was pushed to a given queue...
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // Assert a job was pushed twice...
            Queue::assertPushed(ShipOrder::class, 2);

            // Assert a job was not pushed...
            Queue::assertNotPushed(AnotherJob::class);
        }
    }

Bạn có thể truyền một closure cho các phương thức `assertPushed` hoặc `assertNotPushed` để yêu cầu một job đã vượt qua bài kiểm tra "truth test" nhất định. Nếu ít nhất một job được đẩy đi vượt qua bài kiểm tra "truth test" thì bài kiểm tra đó sẽ thành công:

    Queue::assertPushed(function (ShipOrder $job) use ($order) {
        return $job->order->id === $order->id;
    });

Nếu bạn chỉ cần làm giả các job cụ thể trong khi cho phép các job khác được thực hiện bình thường, bạn có thể truyền tên của các class job cần làm giả vào phương thức `fake`:

    public function test_orders_can_be_shipped()
    {
        Queue::fake([
            ShipOrder::class,
        ]);

        // Perform order shipping...

        // Assert a job was pushed twice...
        Queue::assertPushed(ShipOrder::class, 2);
    }

<a name="job-chains"></a>
### Job Chains

Các phương thức `assertPushedWithChain` và `assertPushedWithoutChain` của facade `Queue` có thể được sử dụng để kiểm tra một chuỗi job của một job được đẩy. Phương thức `assertPushedWithChain` sẽ chấp nhận job chính làm tham số đầu tiên và một loạt các job đi theo làm tham số thứ hai:

    use App\Jobs\RecordShipment;
    use App\Jobs\ShipOrder;
    use App\Jobs\UpdateInventory;
    use Illuminate\Support\Facades\Queue;

    Queue::assertPushedWithChain(ShipOrder::class, [
        RecordShipment::class,
        UpdateInventory::class
    ]);

Như bạn có thể thấy trong ví dụ trên, mảng job đi theo có thể là một mảng tên class của job. Tuy nhiên, bạn cũng có thể cung cấp một mảng các instance job thực tế. Khi làm như vậy, Laravel sẽ đảm bảo các instance job thuộc cùng class và có cùng giá trị thuộc tính của các job đi theo được ứng dụng của bạn gửi đi:

    Queue::assertPushedWithChain(ShipOrder::class, [
        new RecordShipment,
        new UpdateInventory,
    ]);

Bạn có thể sử dụng phương thức `assertPushedWithoutChain` để kiểm tra một job được đẩy đi mà bỏ qua chuỗi job của nó:

    Queue::assertPushedWithoutChain(ShipOrder::class);

<a name="storage-fake"></a>
## Storage Fake

Phương thức `fake` của facade `Storage` cho phép bạn dễ dàng tạo một disk giả, kết hợp với các tiện ích tạo file của class `Illuminate\Http\UploadedFile`, sẽ giúp đơn giản hóa rất nhiều việc kiểm tra file upload. Ví dụ:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_albums_can_be_uploaded()
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

Mặc định, phương thức `fake` sẽ xóa tất cả các file có trong thư mục tạm thời của nó. Nếu bạn muốn giữ lại các file này, bạn có thể sử dụng phương thức "persistentFake" thay cho phương thức `fake`. Để biết thêm thông tin về việc kiểm tra file upload, bạn có thể tham khảo [thông tin về file upload của tài liệu kiểm tra HTTP](/docs/{{version}}/http-tests#testing-file-uploads).

> **Warning**
> Phương thức `image` sẽ yêu cầu [extension GD](https://www.php.net/manual/en/book.image.php).

<a name="interacting-with-time"></a>
## Tương tác với Time

Khi kiểm tra, đôi khi bạn có thể cần sửa thời gian được trả về bởi helper, chẳng hạn như `now` hoặc `Illuminate\Support\Carbon::now()`. Rất may, class kiểm tra cơ bản của Laravel đã chứa các helper cho phép bạn thao tác với thời gian hiện tại:

    use Illuminate\Support\Carbon;

    public function testTimeCanBeManipulated()
    {
        // Travel into the future...
        $this->travel(5)->milliseconds();
        $this->travel(5)->seconds();
        $this->travel(5)->minutes();
        $this->travel(5)->hours();
        $this->travel(5)->days();
        $this->travel(5)->weeks();
        $this->travel(5)->years();

        // Freeze time and resume normal time after executing closure...
        $this->freezeTime(function (Carbon $time) {
            // ...
        });

        // Travel into the past...
        $this->travel(-5)->hours();

        // Travel to an explicit time...
        $this->travelTo(now()->subHours(6));

        // Return back to the present time...
        $this->travelBack();
    }
