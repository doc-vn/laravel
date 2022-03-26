# Eloquent: API Resources

- [Giới thiệu](#introduction)
- [Tạo Resources](#generating-resources)
- [Khái niệm tổng quan](#concept-overview)
    - [Resource Collections](#resource-collections)
- [Viết Resources](#writing-resources)
    - [Data Wrapping](#data-wrapping)
    - [Phân trang](#pagination)
    - [Điều kiện cho thuộc tính](#conditional-attributes)
    - [Điều kiện cho quan hệ](#conditional-relationships)
    - [Thêm Meta Data](#adding-meta-data)
- [Resource Responses](#resource-responses)

<a name="introduction"></a>
## Giới thiệu

Khi xây dựng API, bạn có thể cần một class chuyển đổi nằm giữa các model Eloquent và các response JSON được trả về cho người dùng application của bạn. Các class resource của Laravel cho phép bạn chuyển đổi một cách rõ ràng và dễ hiểu các model cũng như các collection model của bạn thành JSON.

<a name="generating-resources"></a>
## Tạo Resources

Để tạo một class resource, bạn có thể sử dụng lệnh Artisan `make:resource`. Mặc định, resource sẽ được lưu vào trong thư mục `app/Http/Resources` của application của bạn. Các resource sẽ được extend từ class `Illuminate\Http\Resources\Json\JsonResource`:

    php artisan make:resource User

#### Resource Collections

Ngoài việc tạo các resource dùng để chuyển đổi cho các model riêng biệt, bạn cũng có thể tạo các resource để chuyển đổi một collection của model. Điều này cho phép response của bạn có thể thêm các liên kết hoặc các thông tin khác có liên quan đến toàn bộ collection của một resource.

Để tạo một resource collection, bạn hãy sử dụng cờ `--collection` khi tạo resource. Hoặc có từ `Collection` trong tên của resource cũng cho Laravel biết rằng nó cần tạo ra một resource collection. Resource collection được extend từ class `Illuminate\Http\Resources\Json\ResourceCollection`:

    php artisan make:resource Users --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>
## Khái niệm tổng quan

> {tip} Đây là tổng quan về resource và resource collection. Bạn được khuyến khích đọc các phần khác của tài liệu này để hiểu sâu hơn về khả năng tùy biến và sức mạnh của các resource có thể cung cấp cho bạn.

Trước khi đi sâu vào tất cả các tùy chọn có sẵn cho bạn khi bạn viết resource, trước tiên chúng ta hãy xem về cách sử dụng resource trong Laravel. Một class resource sẽ đại diện cho một model cần chuyển đổi thành dạng JSON. Ví dụ, đây là một resource class `User` đơn giản:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Mọi class resource đều định nghĩa một phương thức `toArray` trả về mảng các thuộc tính sẽ được chuyển đổi thành JSON trước khi gửi về response. Lưu ý rằng chúng ta có thể truy cập vào các thuộc tính của model trực tiếp từ biến `$this`. Điều này là do class resource sẽ tự động chuyển hướng các thuộc tính và phương thức truy cập vào model để dễ dàng hơn khi truy cập. Khi resource đã được định nghĩa xong, nó có thể được trả về từ một route hoặc một controller:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

<a name="resource-collections"></a>
### Resource Collections

Nếu bạn đang trả về một resource collection hoặc một response đang được phân trang, bạn có thể sử dụng phương thức `collection` khi tạo instance resource trong route hoặc controller của bạn:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Chú ý rằng điều này sẽ không cho phép bạn thêm bất kỳ dữ liệu meta nào để có thể được trả về cùng với collection. Nếu bạn muốn tùy chỉnh response của resource collection, bạn có thể tạo một resource chuyên dụng để tạo collection:

    php artisan make:resource UserCollection

Khi class resource collection đã được tạo, bạn có thể dễ dàng định nghĩa bất kỳ dữ liệu meta nào cần có trong response:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Sau khi định nghĩa xong resource collection của bạn, nó có thể được trả về từ một route hoặc một controller:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

#### Preserving Collection Keys

Khi trả về một resource collection từ một route, Laravel sẽ reset lại các khóa của collection để chúng có thứ tự sắp xếp từ 0. Tuy nhiên, bạn có thể thêm thuộc tính `preserveKeys` vào class resource của bạn để cho biết liệu khóa collection có được giữ nguyên hay không:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Indicates if the resource's collection keys should be preserved.
         *
         * @var bool
         */
        public $preserveKeys = true;
    }

Khi thuộc tính `secureKeys` được set thành `true`, các khóa của collection sẽ được giữ nguyên:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all()->keyBy->id);
    });

#### Tùy biến Resource Class cơ bản

Thông thường, thuộc tính `$this->collection` của một resource collection sẽ được tự động nối với kết quả của việc ánh xạ của từng item của collection với class resource của nó. Class resource được giả định là tên class của collection mà không có chuỗi `Collection` ở đằng sau.

Ví dụ: `UserCollection` sẽ thử ánh xạ các instance user vào một resource có thể `User`. Để tùy biến hành động này, bạn có thể ghi đè thuộc tính `$collects` của resource collection của bạn:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * The resource that this resource collects.
         *
         * @var string
         */
        public $collects = 'App\Http\Resources\Member';
    }

<a name="writing-resources"></a>
## Viết Resources

> {tip} Nếu bạn chưa đọc phần [khái niệm tổng quan](#concept-overview), bạn được khuyến khích đọc nó trước khi tiếp tục với phần này.

Về bản chất, resource rất đơn giản. Nó chỉ cần chuyển đổi một model thành một mảng. Vì vậy, mỗi resource chứa một phương thức `toArray` để giúp chuyển các thuộc tính của model của bạn thành một mảng thân thiện với API để có thể được trả về cho mọi người dùng của bạn:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Khi một resource đã được định nghĩa xong, nó có thể được trả về trực tiếp từ một route hoặc một controller:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

#### Relationships

Nếu bạn muốn thêm các quan hệ vào trong một response của bạn, bạn có thể thêm chúng vào mảng được trả về trong phương thức `toArray` của bạn. Trong ví dụ này, chúng ra sẽ sử dụng phương thức `collection` của resource `Post` để thêm các post trên blog của người dùng vào response của resource:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} Nếu bạn chỉ thêm các quan hệ chỉ khi chúng đã được load, hãy xem tài liệu về [điều kiện cho quan hệ](#conditional-relationships).

#### Resource Collections

Trong khi các resource sẽ chuyển một model thành một mảng, thì các resource collection sẽ chuyển một collection của model thành một mảng. Không nhất thiết phải định nghĩa một class resource collection cho từng loại model của bạn vì tất cả các resource đều được cung cấp một phương thức `collection` để tạo các resource collection "ad-hoc" một cách nhanh chóng:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Tuy nhiên, nếu bạn cần tùy chỉnh dữ liệu meta được trả về cùng với collection, bạn sẽ cần phải định nghĩa riêng một resource collection:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Giống như resource, resource collection có thể được trả về trực tiếp từ các route hoặc controller:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### Data Wrapping

Mặc định, resource ngoài cùng của bạn sẽ được bao bọc bởi một key `data` khi chúng được chuyển đổi thành JSON. Vì thế, một response resource collection có thể trông như sau:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ]
    }

Nếu bạn muốn vô hiệu hóa việc bao bọc resource này, bạn có thể sử dụng phương thức `withoutWrapping` trên class resource. Thông thường, bạn nên gọi phương thức này từ `AppServiceProvider` hoặc từ một [service provider](/docs/{{version}}/providers) khác để được load cho mọi request trong application của bạn:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Resources\Json\Resource;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Resource::withoutWrapping();
        }
    }

> {note} Phương thức `withoutWrapping` chỉ ảnh hưởng đến response ở ngoài cùng và sẽ không xóa các key `data` mà bạn đã thêm vào bên trong resource collection.

### Wrapping Nested Resources

Bạn có toàn quyền tự do định nghĩa các quan hệ của resource của bạn được bao bọc. Nếu bạn muốn tất cả các resource collection được bao bọc bởi một key `data`, kể cả việc chúng lồng nhau, bạn nên định nghĩa một class resource collection cho mỗi resource và trả về collection đó trong một key `data`.

Bạn có thể tự hỏi liệu rằng điều này có khiến resource ngoài cùng của bạn có bị bao bọc trong hai key `data`. Đừng lo lắng, Laravel sẽ không bao giờ để resource của bạn vô tình bị bao bọc lặp lại như vậy, vì vậy bạn không phải lo lắng về mức độ lồng nhau của resource collection mà bạn đang chuyển đổi:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

### Data Wrapping And Phân trang

Khi trả về một collection được phân trang trong một response resource, Laravel sẽ bao bọc dữ liệu resource của bạn trong một key `data` ngay cả khi phương thức `withoutWrapping` đã được gọi. Điều này là do trong response được phân trang luôn chứa các key `meta` và `links` cùng với các thông tin về trạng thái của phân trang:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="pagination"></a>
### Phân trang

Bạn luôn có thể truyền một instance phân trang cho phương thức `collection` của một resource hoặc một resource collection tùy biến:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

Các response được phân trang luôn chứa các key `meta` và `links` cùng với các thông tin về trạng thái của phân trang:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="conditional-attributes"></a>
### Điều kiện cho thuộc tính

Đôi khi bạn có thể chỉ muốn thêm một số thuộc tính vào trong một response resource nếu một điều kiện được đáp ứng. Ví dụ: bạn có thể chỉ muốn thêm một giá trị nếu người dùng hiện tại đang là "quản trị viên". Laravel cung cấp nhiều phương thức helper để hỗ trợ cho bạn trong những tình huống này. Phương thức `when` có thể được sử dụng để thêm một điều kiện cho một thuộc tính vào response resource:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when(Auth::user()->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Trong ví dụ này, khóa `secret` sẽ chỉ được trả về trong response resource nếu phương thức `$this->isAdmin()` của người dùng hiện tại trả về giá trị `true`. Nếu phương thức trả về giá trị `false`, thì khóa `secret` sẽ bị xóa khỏi response resource trước khi nó được gửi về cho client. Phương thức `when` cho phép bạn định nghĩa một resource mà không cần dùng đến các câu lệnh có điều kiện khi xây dựng một mảng.

Phương thức `when` cũng chấp nhận một Closure là tham số thứ hai của nó, cho phép bạn tính toán giá trị trả về nếu điều kiện đã cho là `true`:

    'secret' => $this->when(Auth::user()->isAdmin(), function () {
        return 'secret-value';
    }),

#### Merging Điều kiện cho thuộc tính

Thỉnh thoảng bạn có thể có một số thuộc tính chỉ được đưa vào trong một response resource dựa trên cùng một điều kiện nào đó. Trong trường hợp này, bạn có thể sử dụng phương thức `mergeWhen` để thêm các thuộc tính vào trong response chỉ khi một điều kiện là `true`:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen(Auth::user()->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Một lần nữa, nếu điều kiện trả về giá trị là `false`, các thuộc tính này sẽ bị xóa ra khỏi response resource trước khi nó được gửi về client.

> {note} Không nên sử dụng phương thức `mergeWhen` trong các mảng mà có sử dụng cả khoá string và khóa numeric. Hơn nữa, nó cũng không nên được sử dụng trong các mảng với các khóa numeric không được sắp xếp theo tuần tự.

<a name="conditional-relationships"></a>
### Điều kiện cho quan hệ

Ngoài các thuộc tính load có điều kiện, bạn cũng có thể thêm các điều kiện cho các quan hệ trong các response resource dựa trên việc quan hệ đó đã được load trên model hay chưa. Điều này cho phép controller của bạn quyết định xem những quan hệ nào sẽ được load trong model và resource của bạn có thể dễ dàng chứa nó chỉ khi nó đã được load.

Cuối cùng, điều này cũng sẽ giúp bạn dễ dàng tránh được các vấn đề về truy vấn "N+1" trong resource của bạn. Phương thức `whenLoaded` có thể được sử dụng để thêm một điều kiện để load một quan hệ. Để tránh load các quan hệ không cần thiết, phương thức này chấp nhận tên một quan hệ thay vì chính quan hệ đó:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Trong ví dụ này, nếu quan hệ chưa được load, thì khóa `posts` sẽ bị xóa bỏ ra khỏi response resource trước khi nó được gửi về client.

#### Conditional Pivot Information

Ngoài việc thêm các thông tin quan hệ có điều kiện vào trong các response resource của bạn, bạn cũng có thể thêm các điều kiện cho dữ liệu từ các bảng trung gian của quan hệ nhiều-nhiều bằng cách sử dụng phương thức `whenPivotLoaded`. Phương thức `whenPivotLoaded` chấp nhận tên của bảng pivot làm tham số đầu tiên. Tham số thứ hai phải là một Closure định nghĩa giá trị được trả về nếu thông tin pivot đó tồn tại trên model:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_user', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

Nếu bảng trung gian của bạn đang sử dụng một tên accessor khác không phải là `pivot`, bạn có thể sử dụng phương thức `whenPivotLoadedAs`:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
                return $this->subscription->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>
### Thêm Meta Data

Một số tiêu chuẩn API JSON sẽ yêu cầu thêm dữ liệu meta vào các response của resource và resource collection của bạn. Điều này thường chứa những thông tin như `links` đến resource hoặc resource quan hệ hoặc dữ liệu meta về chính resource đó. Nếu bạn cần trả về thêm dữ liệu meta cho một resource, hãy cho nó vào phương thức `toArray` của bạn. Ví dụ: bạn có thể chứa thông tin `link` khi chuyển đổi một resource collection:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

Khi trả về thêm một dữ liệu meta từ resource của bạn, bạn sẽ không phải lo lắng về việc vô tình ghi đè các key `links` hoặc `meta` được Laravel tự động thêm khi trả về các response để phân trang. Bất kỳ `links` nào mà bạn đã định nghĩa sẽ được merge với các link đã được cung cấp bởi paginator.

#### Top Level Meta Data

Thỉnh thoảng bạn có thể chỉ muốn thêm một số dữ liệu meta nhất định vào một response resource nếu resource đó là resource ngoài cùng được trả về. Thông thường, điều này sẽ chứa những thông tin meta về toàn bộ response. Để định nghĩa những dữ liệu meta như thế này, hãy thêm một phương thức `with` vào trong class resource của bạn. Phương thức này sẽ trả về một mảng dữ liệu meta sẽ được chứa trong response resource chỉ khi resource đó là resource ngoài cùng được tạo:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * Get additional data that should be returned with the resource array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

#### Thêm Meta Data When Constructing Resources

Bạn cũng có thể thêm dữ liệu khi khởi tạo một instance resource trong route hoặc controller của bạn. Phương thức `additional`, có sẵn trên tất cả các resource, chấp nhận một mảng dữ liệu cần được thêm vào response resource:

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## Resource Responses

Như bạn đã đọc, resources có thể được trả về trực tiếp từ một route hoặc một controller:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

Tuy nhiên, thỉnh thoảng bạn có thể cần tùy biến HTTP response trước khi nó được gửi về client. Có hai cách để thực hiện điều này. Đầu tiên, bạn có thể gắn thêm phương thức `response` vào trong resource. Phương thức này sẽ trả về một instance `Illuminate\Http\Response`, cho phép bạn toàn quyền kiểm soát các header của response:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

Ngoài ra, bạn cũng có thể định nghĩa một phương thức `withResponse` vào trong chính resource của bạn. Phương thức này sẽ được gọi khi resource được trả về là resource ngoài cùng nhất trong một response:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * Customize the outgoing response for the resource.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Illuminate\Http\Response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }
