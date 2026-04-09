# Database Testing

- [Giới thiệu](#introduction)
    - [Reset database sau mỗi lần test](#resetting-the-database-after-each-test)
- [Model Factories](#model-factories)
- [Chạy Seeders](#running-seeders)
- [Assertion có sẵn](#available-assertions)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp nhiều công cụ và assertion hữu ích để giúp bạn kiểm tra các driven database của ứng dụng của bạn dễ dàng hơn. Ngoài ra, các model factory và các seeder của Laravel giúp bạn dễ dàng tạo các record cơ sở dữ liệu thử nghiệm bằng cách sử dụng các model Eloquent và các quan hệ trong ứng dụng của bạn. Chúng ta sẽ thảo luận về tất cả các tính năng mạnh mẽ này trong tài liệu dưới đây.

<a name="resetting-the-database-after-each-test"></a>
### Reset database sau mỗi lần test

Trước khi tiếp tục, hãy thảo luận về cách reset lại cơ sở dữ liệu của bạn sau mỗi lần kiểm tra của bạn thường rất hữu ích để dữ liệu từ những lần kiểm tra trước sẽ không còn can thiệp được vào các lần kiểm tra sau. Trait `Illuminate\Foundation\Testing\RefreshDatabase` có sẵn của Laravel sẽ giải quyết vấn đề này cho bạn. Bạn chỉ cần sử dụng trait này trong class test của bạn:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('basic example', function () {
    $response = $this->get('/');

    // ...
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        $response = $this->get('/');

        // ...
    }
}
```

Trait `Illuminate\Foundation\Testing\RefreshDatabase` sẽ không migrate vào cơ sở dữ liệu của bạn nếu schema của bạn được cập nhật. Thay vào đó, nó sẽ chỉ thực hiện test trong một transaction cơ sở dữ liệu. Do đó, mọi record được thêm vào cơ sở dữ liệu bằng các test case không dùng trait này vẫn có thể tồn tại trong cơ sở dữ liệu.

Nếu muốn reset lại hoàn toàn cơ sở dữ liệu, bạn có thể sử dụng các trait `Illuminate\Foundation\Testing\DatabaseMigrations` hoặc `Illuminate\Foundation\Testing\DatabaseTruncation` thay thế. Tuy nhiên, cả hai tùy chọn này đều chậm hơn đáng kể so với trait `RefreshDatabase`.

<a name="model-factories"></a>
## Model Factories

Khi test, bạn có thể cần thêm một vài bản ghi vào cơ sở dữ liệu trước khi thực hiện test. Thay vì chỉ định giá trị của từng cột theo cách thủ công khi bạn thực hiện tạo dữ liệu test này, Laravel cho phép bạn định nghĩa một tập hợp các thuộc tính mặc định cho mỗi [Eloquent models](/docs/{{version}}/eloquent) của bạn bằng cách sử dụng [model factory](/docs/{{version}}/eloquent-factories).

Để hiểu thêm về cách tạo và sử dụng các model factory để tạo các model, vui lòng tham khảo [tài liệu đầy đủ về model factory](/docs/{{version}}/eloquent-factories). Khi bạn đã định nghĩa xong model factory, bạn có thể sử dụng model factory này trong bài test của bạn để tạo model:

```php tab=Pest
use App\Models\User;

test('models can be instantiated', function () {
    $user = User::factory()->create();

    // ...
});
```

```php tab=PHPUnit
use App\Models\User;

public function test_models_can_be_instantiated(): void
{
    $user = User::factory()->create();

    // ...
}
```

<a name="running-seeders"></a>
## Chạy Seeders

Nếu bạn muốn sử dụng [database seeders](/docs/{{version}}/seeding) để tạo cơ sở dữ liệu trong quá trình test chức năng của bạn, bạn có thể gọi phương thức `seed`. Mặc định, phương thức `seed` sẽ chạy `DatabaseSeeder`, phương thức này sẽ chạy tất cả các seeder khác của bạn. Ngoài ra, bạn cũng có thể truyền vào một tên của class seeder cụ thể cho phương thức `seed`:

```php tab=Pest
<?php

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('orders can be created', function () {
    // Run the DatabaseSeeder...
    $this->seed();

    // Run a specific seeder...
    $this->seed(OrderStatusSeeder::class);

    // ...

    // Run an array of specific seeders...
    $this->seed([
        OrderStatusSeeder::class,
        TransactionStatusSeeder::class,
        // ...
    ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Test creating a new order.
     */
    public function test_orders_can_be_created(): void
    {
        // Run the DatabaseSeeder...
        $this->seed();

        // Run a specific seeder...
        $this->seed(OrderStatusSeeder::class);

        // ...

        // Run an array of specific seeders...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    }
}
```

Ngoài ra, bạn có thể hướng dẫn Laravel tự động khởi tạo cơ sở dữ liệu trước mỗi lần kiểm tra bằng cách sử dụng trait `RefreshDatabase`. Bạn có thể thực hiện việc này bằng cách thêm thuộc tính `Seed` vào class test cơ sở của bạn:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\Attributes\Seed;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

#[Seed]
abstract class TestCase extends BaseTestCase
{
}
```

Khi thuộc tính `Seed` đã được thêm vào, bài test sẽ chạy class `Database\Seeders\DatabaseSeeder` trước mỗi bài test và sử dụng trait `RefreshDatabase`. Tuy nhiên, bạn có thể chỉ định một seeder cụ thể sẽ được thực thi bằng cách dùng thuộc tính `Seeder` trên class test của bạn:

```php
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Illuminate\Foundation\Testing\Attributes\Seeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

#[Seeder(OrderStatusSeeder::class)]
class OrderTest extends TestCase
{
    use RefreshDatabase;

    // ...
}
```

<a name="available-assertions"></a>
## Assertion có sẵn

Laravel cung cấp một số assertion cơ sở dữ liệu cho các test chức năng [Pest](https://pestphp.com) hoặc [PHPUnit](https://phpunit.de) của bạn. Chúng ta sẽ thảo luận về từng assertion dưới đây.

<a name="assert-database-count"></a>
#### assertDatabaseCount

Yêu cầu một bảng trong cơ sở dữ liệu phải chứa một số record đã cho:

```php
$this->assertDatabaseCount('users', 5);
```

<a name="assert-database-empty"></a>
#### assertDatabaseEmpty

Yêu cầu một bảng trong cơ sở dữ liệu không chứa record nào:

```php
$this->assertDatabaseEmpty('users');
```

<a name="assert-database-has"></a>
#### assertDatabaseHas

Yêu cầu một bảng trong cơ sở dữ liệu sẽ chứa các record khớp với các ràng buộc truy vấn khóa và giá trị đã cho:

```php
$this->assertDatabaseHas('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-database-missing"></a>
#### assertDatabaseMissing

Yêu cầu một bảng trong cơ sở dữ liệu không chứa các record khớp với các ràng buộc truy vấn khóa và giá trị đã cho:

```php
$this->assertDatabaseMissing('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-deleted"></a>
#### assertSoftDeleted

Phương thức `assertSoftDeleted` có thể được sử dụng để yêu cầu một model Eloquent nhất định đã bị "soft deleted":

```php
$this->assertSoftDeleted($user);
```

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

Phương thức `assertNotSoftDeleted` có thể được sử dụng để yêu cầu một model Eloquent nhất định chưa bị "soft deleted":

```php
$this->assertNotSoftDeleted($user);
```

<a name="assert-model-exists"></a>
#### assertModelExists

Yêu cầu một model hoặc một collection model sẽ tồn tại trong cơ sở dữ liệu:

```php
use App\Models\User;

$user = User::factory()->create();

$this->assertModelExists($user);
```

<a name="assert-model-missing"></a>
#### assertModelMissing

Yêu cầu một model hoặc một collection model sẽ không tồn tại trong cơ sở dữ liệu:

```php
use App\Models\User;

$user = User::factory()->create();

$user->delete();

$this->assertModelMissing($user);
```

<a name="expects-database-query-count"></a>
#### expectsDatabaseQueryCount

Phương thức `expectsDatabaseQueryCount` có thể được gọi khi bắt đầu bài test để chỉ định tổng số truy vấn vào cơ sở dữ liệu mà bạn dự kiến sẽ chạy trong quá trình test. Nếu số lượng truy vấn được thực hiện thực tế không khớp với kỳ vọng này thì bài test sẽ thất bại:

```php
$this->expectsDatabaseQueryCount(5);

// Test...
```
