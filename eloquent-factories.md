# Eloquent: Factories

- [Giới thiệu](#introduction)
- [Định nghĩa model factoy](#defining-model-factories)
    - [Tạo factory](#generating-factories)
    - [Factory States](#factory-states)
    - [Factory Callbacks](#factory-callbacks)
- [Tạo model dùng Factory](#creating-models-using-factories)
    - [Khởi tạo model](#instantiating-models)
    - [Persisting Models](#persisting-models)
    - [Sequences](#sequences)
- [Quan hệ trong factory](#factory-relationships)
    - [Quan hệ số nhiều](#has-many-relationships)
    - [Quan hệ thuộc về](#belongs-to-relationships)
    - [Quan hệ nhiều - nhiều](#many-to-many-relationships)
    - [Quan hệ đa hình](#polymorphic-relationships)
    - [Định nghĩa quan hệ trong factory](#defining-relationships-within-factories)
    - [Tái sử dụng một model cho quan hệ](#recycling-an-existing-model-for-relationships)

<a name="introduction"></a>
## Giới thiệu

Khi kiểm tra ứng dụng hoặc seeding cơ sở dữ liệu, bạn có thể cần thêm một vài record vào cơ sở dữ liệu. Thay vì tự làm thủ công từng cột, Laravel cho phép bạn định nghĩa một tập hợp các thuộc tính mặc định cho từng [model Eloquent](/docs/{{version}}/eloquent) bằng cách sử dụng các model factory.

Để xem ví dụ về cách viết một factory, hãy xem file `database/factories/UserFactory.php` trong ứng dụng của bạn. File này đi kèm trong tất cả các ứng dụng Laravel mới và chứa định nghĩa factory sau:

    namespace Database\Factories;

    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * Define the model's default state.
         *
         * @return array
         */
        public function definition()
        {
            return [
                'name' => fake()->name(),
                'email' => fake()->unique()->safeEmail(),
                'email_verified_at' => now(),
                'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
                'remember_token' => Str::random(10),
            ];
        }
    }

Như bạn có thể thấy, ở dạng cơ bản nhất, các factory là các class được extend từ các class factory base của Laravel và định nghĩa một phương thức `definition`. Phương thức `definition` này sẽ trả về một tập hợp các giá trị thuộc tính mặc định sẽ được áp dụng khi tạo model bằng cách sử dụng factory.

Thông qua helper `fake`, các factory có quyền truy cập vào các thư viện PHP của [Faker](https://github.com/FakerPHP/Faker), cho phép bạn tạo các loại dữ liệu ngẫu nhiên khác nhau để thử nghiệm và seeding một cách thuận tiện.

> **Note**
> Bạn có thể cài đặt ngôn ngữ Faker trong application của bạn bằng cách thêm tùy chọn `faker_locale` vào file cấu hình `config/app.php` của bạn.

<a name="defining-model-factories"></a>
## Định nghĩa model factoy

<a name="generating-factories"></a>
### Tạo factory

Để tạo một factory, hãy chạy [lệnh Artisan](/docs/{{version}}/artisan) `make:factory`:

```shell
php artisan make:factory PostFactory
```

Một class factory mới sẽ được lưu trong thư mục `database/factories` của bạn.

<a name="factory-and-model-discovery-conventions"></a>
#### Model & Factory Discovery Conventions

Khi mà bạn đã định nghĩa xong các factory của bạn, bạn có thể sử dụng phương thức tĩnh `factory` được cung cấp cho các model của bạn thông qua trait `Illuminate\Database\Eloquent\Factories\HasFactory` để khởi tạo một instance factory cho model đó.

Phương thức `factory` của trait `HasFactory` sẽ sử dụng các quy ước đặt tên để xác định các factory thích hợp cho model mà trait được chỉ định. Cụ thể, phương thức sẽ tìm kiếm một factory trong namespace `Database\Factories` và có tên class khớp với tên có hậu tố là `Factory`. Nếu các quy ước này không được áp dụng cho ứng dụng hoặc factory của bạn, bạn có thể ghi đè lên phương thức `newFactory` này trên model của bạn để trả về trực tiếp một instance của factory tương ứng của model:

    use Database\Factories\Administration\FlightFactory;

    /**
     * Create a new factory instance for the model.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    protected static function newFactory()
    {
        return FlightFactory::new();
    }

Tiếp theo, định nghĩa thuộc tính `model` trên factory tương ứng:

    use App\Administration\Flight;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class FlightFactory extends Factory
    {
        /**
         * The name of the factory's corresponding model.
         *
         * @var string
         */
        protected $model = Flight::class;
    }

<a name="factory-states"></a>
### Factory States

Các phương thức state cho phép bạn định nghĩa các thay đổi riêng biệt để áp dụng cho từng model factory. Ví dụ: factory `Database\Factories\UserFactory` của bạn có thể chứa một phương thức state `suspended` để sửa một trong các giá trị thuộc tính mặc định của nó.

Các phương thức chuyển đổi trạng thái thường gọi trong phương thức `state` do class base của Laravel cung cấp. Phương thức `state` sẽ chấp nhận một closure và nhận vào một mảng thuộc tính được định nghĩa cho factory và sẽ trả về một mảng thuộc tính để sửa:

    /**
     * Indicate that the user is suspended.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    public function suspended()
    {
        return $this->state(function (array $attributes) {
            return [
                'account_status' => 'suspended',
            ];
        });
    }

#### "Trashed" State

Nếu model Eloquent của bạn có chức năng [soft deleted](/docs/{{version}}/eloquent#soft-deleting), thì bạn có thể gọi phương thức state `trashed` có sẵn để chỉ ra rằng model được tạo sẽ bị "soft deleted". Bạn không cần phải định nghĩa state `trashed` vì nó tự động có sẵn trong tất cả các factory:

    use App\Models\User;

    $user = User::factory()->trashed()->create();

<a name="factory-callbacks"></a>
### Factory Callbacks

Các lệnh Factory callback sẽ được đăng ký bằng cách sử dụng bởi các phương thức `afterMaking` và `afterCreating` đồng thời cho phép bạn thực hiện thêm các tác vụ sau khi tạo một model. Bạn nên đăng ký các callback này bằng cách định nghĩa một phương thức `configure` trong class factory của bạn. Phương thức này sẽ được Laravel gọi tự động khi khởi tạo factory:

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * Configure the model factory.
         *
         * @return $this
         */
        public function configure()
        {
            return $this->afterMaking(function (User $user) {
                //
            })->afterCreating(function (User $user) {
                //
            });
        }

        // ...
    }

<a name="creating-models-using-factories"></a>
## Tạo model dùng Factory

<a name="instantiating-models"></a>
### Khởi tạo model

Khi bạn đã định nghĩa xong các factory của bạn, bạn có thể sử dụng phương thức static `factory` được cung cấp cho các model của bạn bởi trait `Illuminate\Database\Eloquent\Factories\HasFactory` để khởi tạo một instance factory cho model đó. Chúng ta hãy xem một vài ví dụ về việc tạo model này. Đầu tiên, chúng ta sẽ sử dụng phương thức `make` để tạo ra các model mà chưa lưu chúng vào cơ sở dữ liệu:

    use App\Models\User;

    $user = User::factory()->make();

Bạn có thể tạo một collection nhiều model bằng cách sử dụng phương thức `count`:

    $users = User::factory()->count(3)->make();

<a name="applying-states"></a>
#### Applying States

Bạn cũng có thể dùng bất kỳ [states](#factory-states) nào của bạn cho các model. Nếu bạn muốn dùng nhiều loại state cho các model, bạn đơn giản chỉ cần gọi trực tiếp các phương thức state đó:

    $users = User::factory()->count(5)->suspended()->make();

<a name="overriding-attributes"></a>
#### Overriding Attributes

Nếu bạn muốn ghi đè một số giá trị mặc định của model của bạn, bạn có thể truyền vào một mảng các giá trị cho phương thức `make`. Chỉ các thuộc tính được khai báo sẽ được thay đổi trong khi các phần còn lại của các thuộc tính khác vẫn sẽ được set theo giá trị mặc định của chúng như được khai báo ban đầu bởi factory:

    $user = User::factory()->make([
        'name' => 'Abigail Otwell',
    ]);

Ngoài ra, phương thức `state` có thể được gọi trực tiếp trên instance factory để thực hiện state nay bên trong phương thức:

    $user = User::factory()->state([
        'name' => 'Abigail Otwell',
    ])->make();

> **Note**
[Các bảo vệ mass assignment](/docs/{{version}}/eloquent#mass-assignment) sẽ tự động bị tắt khi tạo model bằng factory.

<a name="persisting-models"></a>
### Persisting Models

Phương thức `create` sẽ khởi tạo các instance của model và lưu chúng vào cơ sở dữ liệu bằng cách sử dụng phương thức `save` của Eloquent:

    use App\Models\User;

    // Create a single App\Models\User instance...
    $user = User::factory()->create();

    // Create three App\Models\User instances...
    $users = User::factory()->count(3)->create();

Bạn có thể ghi đè các thuộc tính model mặc định của factory cách truyền một mảng các thuộc tính cho phương thức `create`:

    $user = User::factory()->create([
        'name' => 'Abigail',
    ]);

<a name="sequences"></a>
### Sequences

Đôi khi bạn có thể muốn thay thế giá trị của một thuộc tính cho những model đã được tạo. Bạn có thể thực hiện điều này bằng cách định nghĩa một state dưới dạng một chuỗi. Ví dụ: bạn có thể muốn thay thế giá trị của cột `admin` giữa `Y` và `N` cho mỗi người dùng đã tạo:

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Sequence;

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        ['admin' => 'Y'],
                        ['admin' => 'N'],
                    ))
                    ->create();

Trong ví dụ này, năm người dùng sẽ được tạo với giá trị `admin` là `Y` và năm người dùng sẽ được tạo với giá trị `admin` là `N`.

Nếu cần, bạn có thể thêm một closure vào như một giá trị chuỗi. Closure sẽ được gọi mỗi khi chuỗi cần một giá trị mới:

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        fn ($sequence) => ['role' => UserRoles::all()->random()],
                    ))
                    ->create();

Trong một sequence closure, bạn có thể truy cập vào các thuộc tính `$index` hoặc `$count` trên instance sequence được khai báo trong closure. Thuộc tính `$index` sẽ chứa số lần lặp chuỗi, trong khi thuộc tính `$count` chứa tổng số lần chuỗi sẽ được gọi:

    $users = User::factory()
                    ->count(10)
                    ->sequence(fn ($sequence) => ['name' => 'Name '.$sequence->index])
                    ->create();

Để thuận tiện, các sequence cũng có thể được áp dụng bằng phương thức `sequence`, phương thức này chỉ đơn giản là gọi phương thức `state` ở bên trong. Phương thức `sequence` sẽ chấp nhận một closure hoặc một mảng các thuộc tính đã được sắp xếp theo trình tự:

    $users = User::factory()
                    ->count(2)
                    ->sequence(
                        ['name' => 'First User'],
                        ['name' => 'Second User'],
                    )
                    ->create();

<a name="factory-relationships"></a>
## Quan hệ trong factory

<a name="has-many-relationships"></a>
### Quan hệ số nhiều

Tiếp theo, hãy khám phá việc xây dựng các quan hệ model Eloquent bằng cách sử dụng các phương thức linh hoạt factory của Laravel. Đầu tiên, giả sử rằng ứng dụng của chúng ta có model `App\Models\User` và model `App\Models\Post`. Ngoài ra, giả sử rằng model `User` định nghĩa một quan hệ `hasMany` với `Post`. Chúng ta có thể tạo một user có ba bài post bằng cách sử dụng phương thức `has` được cung cấp bởi các factory của Laravel. Phương thức `has` chấp nhận một instance factory:

    use App\Models\Post;
    use App\Models\User;

    $user = User::factory()
                ->has(Post::factory()->count(3))
                ->create();

Theo quy ước, khi truyền một model `Post` cho phương thức `has`, Laravel sẽ giả định rằng model `User` sẽ có một phương thức `posts` để định nghĩa quan hệ đó. Nếu cần, bạn có thể chỉ định rõ tên của quan hệ mà bạn muốn thao tác:

    $user = User::factory()
                ->has(Post::factory()->count(3), 'posts')
                ->create();

Tất nhiên, bạn có thể thực hiện các thao tác state trên các model quan hệ. Ngoài ra, bạn có thể truyền một closure state nếu state của bạn yêu cầu quyền truy cập vào model gốc:

    $user = User::factory()
                ->has(
                    Post::factory()
                            ->count(3)
                            ->state(function (array $attributes, User $user) {
                                return ['user_type' => $user->type];
                            })
                )
                ->create();

<a name="has-many-relationships-using-magic-methods"></a>
#### Using Magic Methods

Để thuận tiện, bạn có thể sử dụng các phương thức magic factory relationship của Laravel để xây dựng quan hệ. Ví dụ: trong ví dụ sau sẽ sử dụng quy ước để xác định các model có quan hệ sẽ được tạo thông qua phương thức quan hệ `posts` trên model `User`:

    $user = User::factory()
                ->hasPosts(3)
                ->create();

Khi sử dụng các phương thức magic method để tạo các quan hệ của factory, bạn có thể truyền vào một mảng các thuộc tính để ghi đè lên các model quan hệ:

    $user = User::factory()
                ->hasPosts(3, [
                    'published' => false,
                ])
                ->create();

Bạn có thể cung cấp một closure dựa trên state transformation nếu thay đổi state của bạn yêu cầu quyền truy cập vào model gốc:

    $user = User::factory()
                ->hasPosts(3, function (array $attributes, User $user) {
                    return ['user_type' => $user->type];
                })
                ->create();

<a name="belongs-to-relationships"></a>
### Quan hệ thuộc về

Bây giờ chúng ta sẽ khám phá cách xây dựng quan hệ "nhiều" bằng cách sử dụng các factory, đầu tiên, hãy khám phá đầu ngược lại của quan hệ "nhiều". Phương thức `for` có thể được sử dụng để định nghĩa model gốc chứa các model được tạo tại factory. Ví dụ: chúng ta có thể tạo ba instance model `App\Models\Post` thuộc về một người dùng:

    use App\Models\Post;
    use App\Models\User;

    $posts = Post::factory()
                ->count(3)
                ->for(User::factory()->state([
                    'name' => 'Jessica Archer',
                ]))
                ->create();

Nếu bạn đã có một instance model gốc sẽ được liên kết với các model bạn đang tạo, bạn có thể truyền instance model đó vào phương thức `for`:

    $user = User::factory()->create();

    $posts = Post::factory()
                ->count(3)
                ->for($user)
                ->create();

<a name="belongs-to-relationships-using-magic-methods"></a>
#### Using Magic Methods

Để thuận tiện, bạn có thể sử dụng các phương thức magic factory relationship của Laravel để định nghĩa các quan hệ "thuộc về". Ví dụ: trong ví dụ sau sẽ sử dụng quy ước để xác định ba bài đăng sẽ phải thuộc về quan hệ `user` trên model `Post`:

    $posts = Post::factory()
                ->count(3)
                ->forUser([
                    'name' => 'Jessica Archer',
                ])
                ->create();

<a name="many-to-many-relationships"></a>
### Quan hệ nhiều - nhiều

Giống như [quan hệ số nhiều](#has-many-relationships), quan hệ "nhiều với nhiều" có thể được tạo ra bằng cách sử dụng phương thức `has`:

    use App\Models\Role;
    use App\Models\User;

    $user = User::factory()
                ->has(Role::factory()->count(3))
                ->create();

<a name="pivot-table-attributes"></a>
#### Pivot Table Attributes

Nếu bạn cần định nghĩa các thuộc tính sẽ được set trên bảng pivot hoặc trung gian được liên kết với các model, bạn có thể sử dụng phương thức `hasAttached`. Phương thức này chấp nhận một mảng các tên và giá trị thuộc tính của bảng pivot làm tham số thứ hai của nó:

    use App\Models\Role;
    use App\Models\User;

    $user = User::factory()
                ->hasAttached(
                    Role::factory()->count(3),
                    ['active' => true]
                )
                ->create();

Bạn có thể cung cấp một closure dựa trên state transformation nếu thay đổi state của bạn yêu cầu quyền truy cập vào model quan hệ:

    $user = User::factory()
                ->hasAttached(
                    Role::factory()
                        ->count(3)
                        ->state(function (array $attributes, User $user) {
                            return ['name' => $user->name.' Role'];
                        }),
                    ['active' => true]
                )
                ->create();

Nếu bạn đã có các instance model mà bạn muốn attache vào các model mà bạn đang tạo, bạn có thể truyền các instance model vào phương thức `hasAttached`. Trong ví dụ này, ba quyền giống nhau sẽ được gán cho cả ba người dùng:

    $roles = Role::factory()->count(3)->create();

    $user = User::factory()
                ->count(3)
                ->hasAttached($roles, ['active' => true])
                ->create();

<a name="many-to-many-relationships-using-magic-methods"></a>
#### Using Magic Methods

Để thuận tiện, bạn có thể sử dụng các phương thức magic factory relationship của Laravel để định nghĩa quan hệ nhiều-nhiều. Ví dụ: trong ví dụ sau sẽ sử dụng các quy ước để xác định các model quan hệ sẽ được tạo thông qua phương thức quan hệ `roles` trên model `User`:

    $user = User::factory()
                ->hasRoles(1, [
                    'name' => 'Editor'
                ])
                ->create();

<a name="polymorphic-relationships"></a>
### Quan hệ đa hình

[Quan hệ đa hình](/docs/{{version}}/eloquent-relationships#polymorphic-relationships) cũng có thể được tạo ra bằng cách sử dụng các factory. Các quan hệ "morph many" đa hình được tạo ra theo cách tương tự như các quan hệ "has many" điển hình. Ví dụ: nếu model `App\Models\Post` có quan hệ `morphMany` với model `App\Models\Comment`:

    use App\Models\Post;

    $post = Post::factory()->hasComments(3)->create();

<a name="morph-to-relationships"></a>
#### Morph To Relationships

Các phương thức magic có thể không được sử dụng để tạo các quan hệ `morphTo`. Thay vào đó, phương thức `for` sẽ phải được sử dụng trực tiếp và tên của quan hệ phải được cung cấp cho phương thức. Ví dụ: hãy tưởng tượng rằng model `Comment` có phương thức `commentable` định nghĩa quan hệ `morphTo`. Trong tình huống này, chúng ta có thể tạo ra ba nhận xét thuộc vào một bài đăng bằng cách sử dụng trực tiếp phương thức `for`:

    $comments = Comment::factory()->count(3)->for(
        Post::factory(), 'commentable'
    )->create();

<a name="polymorphic-many-to-many-relationships"></a>
#### Polymorphic Many To Many Relationships

Các quan hệ "nhiều-nhiều" (`morphToMany` / `morphedByMany`) đa hình có thể được tạo ra giống như các quan hệ "nhiều-nhiều" không đa hình:

    use App\Models\Tag;
    use App\Models\Video;

    $videos = Video::factory()
                ->hasAttached(
                    Tag::factory()->count(3),
                    ['public' => true]
                )
                ->create();

Tất nhiên, phương thức magic `has` cũng có thể được sử dụng để tạo ra các quan hệ "nhiều-nhiều" đa hình:

    $videos = Video::factory()
                ->hasTags(3, ['public' => true])
                ->create();

<a name="defining-relationships-within-factories"></a>
### Định nghĩa quan hệ trong factory

Để định nghĩa một quan hệ trong factory model của bạn, thông thường bạn sẽ cần chỉ định một instance factory mới cho khóa ngoại của quan hệ. Điều này thường được thực hiện cho các quan hệ "phía ngược lại" chẳng hạn như quan hệ `belongsTo` và `morphTo`. Ví dụ: nếu bạn muốn tạo người dùng mới khi tạo bài đăng, bạn có thể làm như sau:

    use App\Models\User;

    /**
     * Define the model's default state.
     *
     * @return array
     */
    public function definition()
    {
        return [
            'user_id' => User::factory(),
            'title' => fake()->title(),
            'content' => fake()->paragraph(),
        ];
    }

Nếu các cột của quan hệ phụ thuộc vào factory định nghĩa nó, thì bạn có thể gán một closure cho một thuộc tính. Closure sẽ nhận vào một mảng thuộc tính được so sánh trong factory:

    /**
     * Define the model's default state.
     *
     * @return array
     */
    public function definition()
    {
        return [
            'user_id' => User::factory(),
            'user_type' => function (array $attributes) {
                return User::find($attributes['user_id'])->type;
            },
            'title' => fake()->title(),
            'content' => fake()->paragraph(),
        ];
    }

<a name="recycling-an-existing-model-for-relationships"></a>
### Tái sử dụng một model cho quan hệ

Nếu bạn có các model có quan hệ chung với một model khác, bạn có thể sử dụng phương thức `recycle` để đảm bảo một instance duy nhất của model sẽ được tái sử dụng cho tất cả các quan hệ do factory tạo ra.

Ví dụ, hãy tưởng tượng bạn có các model `Airline`, `Flight` và `Ticket`, trong đó ticket sẽ thuộc về một hãng hàng không và một chuyến bay, và chuyến bay cũng thuộc về một hãng hàng không. Khi tạo ticket, có thể bạn sẽ muốn cùng một hãng hàng không cho cả ticket và cả chuyến bay, vì thế bạn có thể truyền một instance của hãng hàng không cho phương thức `recycle`:

    Ticket::factory()
        ->recycle(Airline::factory()->create())
        ->create();

Bạn có thể thấy phương thức `recycle` đặc biệt hữu ích nếu bạn có các model thuộc về một người dùng hoặc một nhóm chung.

Phương thức `recycle` cũng chấp nhận một collection các model hiện có. Khi một collection được cung cấp cho phương thức `recycle`, một model ngẫu nhiên từ trong collection sẽ được chọn khi factory cần một model loại đó:

    Ticket::factory()
        ->recycle($airlines)
        ->create();
