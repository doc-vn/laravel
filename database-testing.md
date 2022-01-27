# Database Testing

- [Giới thiệu](#introduction)
- [Tạo Factory](#generating-factories)
- [Reset database sau mỗi lần test](#resetting-the-database-after-each-test)
- [Viết Factory](#writing-factories)
    - [Factory States](#factory-states)
    - [Factory Callbacks](#factory-callbacks)
- [Dùng Factory](#using-factories)
    - [Tạo Model](#creating-models)
    - [Lưu trữ Model](#persisting-models)
    - [Quan hệ](#relationships)
- [Assertion có sẵn](#available-assertions)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp nhiều công cụ hữu ích để giúp bạn dễ dàng test các cơ sở dữ liệu của bạn. Trước tiên, bạn có thể sử dụng các helper `assertDatabaseHas` để yêu cầu những dữ liệu có trong cơ sở dữ liệu phải giống với một bộ tiêu chí nhất định. Ví dụ: nếu bạn muốn yêu cầu phải có một bản ghi trong bảng `users` với giá trị `email` là `sally@example.com`, bạn có thể làm như sau:

    public function testDatabase()
    {
        // Make call to application...

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com'
        ]);
    }

Bạn cũng có thể sử dụng helper `assertDatabaseMissing` để yêu cầu dữ liệu phải không được có trong cơ sở dữ liệu.

Tất nhiên, phương thức `assertDatabaseHas` và những phương thức helper khác giống như nó là để cho thuận tiện hơn. Bạn có thể tự do sử dụng bất kỳ phương thức kiểm tra nào của PHPUnit để bổ sung cho các bài test của bạn.

<a name="generating-factories"></a>
## Tạo Factory

Để tạo một factory, hãy sử dụng [Lệnh Artisan](/docs/{{version}}/artisan) `make:factory`:

    php artisan make:factory PostFactory

Factory mới sẽ được tạo trong thư mục `database/factories` của bạn.

Tùy chọn `--model` có thể được sử dụng để khai báo tên của model sẽ được tạo ra bởi factory. Dựa theo tùy chọn này mà tên model sẽ được khai báo vào trong file factory được tạo:

    php artisan make:factory PostFactory --model=Post

<a name="resetting-the-database-after-each-test"></a>
## Reset database sau mỗi lần test

Việc reset lại cơ sở dữ liệu của bạn sau mỗi lần kiểm tra thường rất hữu ích để dữ liệu từ những lần kiểm tra trước sẽ không còn can thiệp được vào các lần kiểm tra sau. Trait `RefreshDatabase` là cách tiếp cận tối ưu nhất để migration cơ sở dữ liệu test cho bạn, nó không phụ thuộc vào việc bạn đang sử dụng cơ sở dữ liệu trong bộ nhớ hay cơ sở dữ liệu truyền thống. Hãy dùng trait trong class test của bạn và mọi thứ sẽ được xử lý cho bạn:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

<a name="writing-factories"></a>
## Viết Factory

Trước khi test, bạn có thể cần thêm một vài bản ghi vào trong cơ sở dữ liệu của bạn trước khi thực hiện test. Thay vì khai báo thủ công các giá trị cho từng cột khi bạn tạo dữ liệu test này, Laravel cho phép bạn định nghĩa một loạt các thuộc tính mặc định cho từng [Eloquent models](/docs/{{version}}/eloquent) bằng cách sử dụng các model factory. Để bắt đầu, hãy xem file `database/factories/UserFactory.php` trong ứng dụng của bạn. Mặc định, file này chứa sẵn một định nghĩa của factory:

    use Faker\Generator as Faker;

    $factory->define(App\User::class, function (Faker $faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => '$2y$10$TKh8H1.PfQx37YgCzwiKb.KjNyWgaHb9cbcoQgdIVFlYg7B77UdFm', // secret
            'remember_token' => str_random(10),
        ];
    });

Closure đóng vai trò là định nghĩa của factory, bạn có thể trả về các giá trị test mặc định cho tất cả các thuộc tính trong model. Closure sẽ nhận vào một instance của thư viện [Faker](https://github.com/fzaninotto/Faker) PHP, cho phép bạn dễ dàng tạo các loại dữ liệu giả ngẫu nhiên khác nhau để test.

Bạn cũng có thể tạo thêm các file factory cho từng model để tổ chức tốt hơn. Ví dụ: bạn có thể tạo các file `UserFactory.php` và `CommentFactory.php` trong thư mục `database/factories` của bạn. Tất cả các file trong thư mục `factories` sẽ tự động được load bởi Laravel.

> {tip} Bạn có thể cài đặt ngôn ngữ của Faker bằng cách thêm tùy chọn `faker_locale` vào file cấu hình `config/app.php` của bạn.

<a name="factory-states"></a>
### Factory States

Các state cho phép bạn định nghĩa các thay đổi riêng biệt để áp dụng cho từng model factory. Ví dụ, model `User` của bạn có thể có state là `delinquent` và nó cần thay đổi một giá trị mặc định trong các thuộc tính của bạn. Bạn có thể định nghĩa các thay đổi state này bằng phương thức `state`. Đối với các state đơn giản, bạn có thể truyền một mảng các thay đổi của thuộc tính:

    $factory->state(App\User::class, 'delinquent', [
        'account_status' => 'delinquent',
    ]);

Nếu state của bạn mà yêu cầu tính toán hoặc một instance `$faker`, bạn có thể sử dụng một Closure để tính toán các thay đổi thuộc tính của state:

    $factory->state(App\User::class, 'address', function ($faker) {
        return [
            'address' => $faker->address,
        ];
    });

<a name="factory-callbacks"></a>
### Factory Callbacks

Các lệnh Factory callback sẽ được đăng ký bằng cách sử dụng các phương thức `afterMaking` và `afterCreating`, đồng thời cho phép bạn thực hiện thêm các tác vụ sau khi making hoặc creating một model. Ví dụ: bạn có thể sử dụng callback để liên kết các model mới với model đã được tạo:

    $factory->afterMaking(App\User::class, function ($user, $faker) {
        // ...
    });

    $factory->afterCreating(App\User::class, function ($user, $faker) {
        $user->accounts()->save(factory(App\Account::class)->make());
    });

Bạn cũng có thể định nghĩa callback cho [factory states](#factory-states):

    $factory->afterMakingState(App\User::class, 'delinquent', function ($user, $faker) {
        // ...
    });

    $factory->afterCreatingState(App\User::class, 'delinquent', function ($user, $faker) {
        // ...
    });

<a name="using-factories"></a>
## Dùng Factory

<a name="creating-models"></a>
### Tạo Model

Khi bạn đã định nghĩa các factory của bạn, bạn có thể sử dụng hàm global `factory` trong các bài test hoặc trong các file seed của bạn để tạo ra các instance model. Bây giờ, chúng ta hãy xem qua một vài ví dụ về việc tạo model. Đầu tiên, chúng ta sẽ sử dụng phương thức `make` để tạo các model nhưng không lưu chúng vào cơ sở dữ liệu:

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // Use model in tests...
    }

Bạn cũng có thể tạo một Collection chứa nhiều model hoặc tạo nhiều model với một loại nhất định:

    // Create three App\User instances...
    $users = factory(App\User::class, 3)->make();

#### Applying States

Bạn cũng có thể dùng bất kỳ [states](#factory-states) nào của bạn cho các model. Nếu bạn muốn dùng nhiều loại state cho các model, bạn nên khai báo tên của từng state mà bạn muốn áp dụng:

    $users = factory(App\User::class, 5)->states('delinquent')->make();

    $users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();

#### Overriding Attributes

Nếu bạn muốn ghi đè một số giá trị mặc định của model của bạn, bạn có thể truyền vào một mảng các giá trị cho phương thức `make`. Chỉ các giá trị được khai báo sẽ được thay đổi trong khi các phần còn lại của các giá trị khác vẫn sẽ được set theo giá trị mặc định của chúng như được khai báo ban đầu bởi factory:

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);

<a name="persisting-models"></a>
### Lưu trữ Model

Phương thức `create` không chỉ tạo ra các instance model mà còn lưu chúng vào cơ sở dữ liệu bằng phương thức `save` của Eloquent:

    public function testDatabase()
    {
        // Create a single App\User instance...
        $user = factory(App\User::class)->create();

        // Create three App\User instances...
        $users = factory(App\User::class, 3)->create();

        // Use model in tests...
    }

Bạn có thể ghi đè các thuộc tính trong model bằng cách truyền thêm một mảng cho phương thức `create`:

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
    ]);

<a name="relationships"></a>
### Quan hệ

Trong ví dụ này, chúng ta sẽ gắn thêm một quan hệ cho một số model đã được tạo. Khi sử dụng phương thức `create` để tạo nhiều model, một [instance collection](/docs/{{version}}/eloquent-collections) Eloquent sẽ được trả về, cho phép bạn sử dụng bất kỳ phương thức nào do collection cung cấp, chẳng hạn như `each`:

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function ($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

#### Relations & Attribute Closures

Bạn cũng có thể gắn thêm các quan hệ cho các model bằng các thuộc tính Closure trong định nghĩa factory của bạn. Ví dụ: nếu bạn muốn tạo một instance `User` mới khi đang tạo một `Post`, bạn có thể làm như sau:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });

Các Closure này cũng nhận vào một mảng các thuộc tính của factory mà đã được định nghĩa cho chúng:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });

<a name="available-assertions"></a>
## Assertion có sẵn

Laravel cung cấp sẵn một số phương thức kiểm tra cơ sở dữ liệu cho các bài test [PHPUnit] (https://phastait.de/) của bạn:

Method  | Description
------------- | -------------
`$this->assertDatabaseHas($table, array $data);`  |  Yêu cầu một bảng trong cơ sở dữ liệu phải chứa dữ liệu đã cho.
`$this->assertDatabaseMissing($table, array $data);`  |  Yêu cầu một bảng trong cơ sở dữ liệu không được chứa dữ liệu đã cho.
`$this->assertSoftDeleted($table, array $data);`  |  Yêu cầu bản ghi đã cho đã bị soft delete.
