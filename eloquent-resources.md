# Eloquent: API Resources

- [Giới thiệu](#introduction)
- [Tạo Resources](#generating-resources)
- [Khái niệm tổng quan](#concept-overview)
- [Viết Resources](#writing-resources)
    - [Data Wrapping](#data-wrapping)
    - [Phân trang](#pagination)
    - [Điều kiện cho thuộc tính](#conditional-attributes)
    - [Điều kiện cho quan hệ](#conditional-relationships)
    - [Thêm Meta Data](#adding-meta-data)
- [Resource Responses](#resource-responses)

<a name="introduction"></a>
## Giới thiệu

Khi xây dựng API, bạn có thể cần một lớp chuyển đổi nằm giữa các model Eloquent của bạn và các response JSON được trả về cho người dùng application của bạn. Các class resource của Laravel cho phép bạn chuyển đổi một cách rõ ràng và dễ hiểu các model và collection model của bạn thành JSON.

<a name="generating-resources"></a>
## Tạo Resources

Để tạo một class resource, bạn có thể sử dụng lệnh Artisan `make:resource`. Mặc định, resource sẽ được đặt trong thư mục `app/Http/Resources` của application của bạn. Các resource sẽ được mở rộng từ class `Illuminate\Http\Resources\Json\Resource`:

    php artisan make:resource UserResource

#### Resource Collections

Ngoài việc tạo các resource biến đổi các model riêng biệt, bạn có thể tạo các resource chịu trách nhiệm chuyển đổi các collection của model. Điều này cho phép response của bạn chứa các liên kết và thông tin meta khác có liên quan đến toàn bộ collection của một resource.

Để tạo một resource collection, bạn nên sử dụng cờ `--collection` khi tạo resource. Hoặc, chứa từ `Collection` trong tên resource sẽ cho Laravel biết rằng nó sẽ tạo ra một resource collection. Resource collection được mở rộng từ class `Illuminate\Http\Resources\Json\ResourceCollection`:

    php artisan make:resource Users --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>
## Khái niệm tổng quan

> {tip} Đây là tổng quan ở mức cao về resource và resource collection. Bạn được khuyến khích đọc các phần khác của tài liệu này để hiểu sâu hơn về khả năng tùy biến và sức mạnh được cung cấp cho bạn bởi các resource.

Trước khi đi sâu vào tất cả các tùy chọn có sẵn cho bạn khi viết resource, trước tiên chúng ta hãy xem ở mức độ cao về cách sử dụng resource trong Laravel. Một class resource đại diện cho một model cần được chuyển đổi thành cấu trúc JSON. Ví dụ, đây là một class `UserResource` đơn giản:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
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

Mọi class resource định nghĩa một phương thức `toArray` trả về mảng các thuộc tính sẽ được chuyển đổi thành JSON khi gửi response. Lưu ý rằng chúng ta có thể truy cập vào các thuộc tính model trực tiếp từ biến `$this`. Điều này là do một class resource sẽ tự động proxy thuộc tính và phương thức truy cập vào model bên dưới để thuận tiện truy cập. Khi resource đã được định nghĩa, thì nó có thể được trả về từ một route hoặc một controller:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

### Resource Collections

Nếu bạn đang trả về một collection các resource hoặc một response được phân trang, bạn có thể sử dụng phương thức `collection` khi tạo instance resource trong route hoặc controller của bạn:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Tất nhiên, điều này không cho phép thêm bất kỳ dữ liệu meta nào có thể được trả về cùng với collection. Nếu bạn muốn tùy chỉnh response của resource collection, bạn có thể tạo một resource chuyên dụng để tạo collection:

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
         * @param  \Illuminate\Http\Request
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

Sau khi định nghĩa resource collection của bạn, nó có thể được trả về từ một route hoặc controller:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="writing-resources"></a>
## Viết Resources

> {tip} Nếu bạn chưa đọc phần [khái niệm tổng quan](#concept-overview), bạn được khuyến khích đọc nó trước khi tiếp tục với phần này.

Về bản chất, resource rất đơn giản. Nó chỉ cần chuyển đổi một model nhất định thành một mảng. Vì vậy, mỗi resource chứa một phương thức `toArray` giúp chuyển các thuộc tính của model của bạn thành một mảng thân thiện với API đề có thể được trả về cho người dùng của bạn:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
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

Khi một resource đã được định nghĩa, nó có thể được trả về trực tiếp từ một route hoặc controller:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

#### Relationships

Nếu bạn muốn chứa các resource quan hệ trong response của bạn, bạn có thể thêm chúng vào mảng được trả về bởi phương thức `toArray` của bạn. Trong ví dụ này, chúng ra sẽ sử dụng phương thức `collection` của resource `Post` để thêm các post trên blog của người dùng vào response của resource:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} Nếu bạn chỉ muốn chứa các quan hệ khi chúng đã được load, hãy xem tài liệu về [điều kiện cho quan hệ](#conditional-relationships).

#### Resource Collections

Trong khi resource chuyển một model thành một mảng, thì các resource collection sẽ chuyển một collection của model thành một mảng. Không nhất thiết phải định nghĩa một class resource collection cho từng loại model của bạn vì tất cả các resource đều được cung cấp một phương thức `collection` để tạo resource collection "ad-hoc" một cách nhanh chóng:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Tuy nhiên, nếu bạn cần tùy chỉnh dữ liệu meta được trả về cùng với collection, nó sẽ cần phải định nghĩa một resource collection:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
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

Giống như singular resource, resource collection có thể được trả về trực tiếp từ các route hoặc controller:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### Data Wrapping

Mặc định, resource ngoài cùng của bạn sẽ được wrap trong một key `data`, khi response resource, nó sẽ được chuyển đổi thành JSON. Vì thế, ví dụ, một response resource collection sẽ trông như sau:

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

Nếu bạn muốn vô hiệu hóa việc wrap resource, bạn có thể sử dụng phương thức `withoutWrapping` trên class resource. Thông thường, bạn nên gọi phương thức này từ `AppServiceProvider` hoặc [service provider](/docs/{{version}}/providers) khác sẽ được load cho mọi request cho application của bạn:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Resources\Json\Resource;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Resource::withoutWrapping();
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} Phương thức `withoutWrapping` chỉ ảnh hưởng đến response ở ngoài cùng và sẽ không xóa các key `data` mà bạn tự thêm vào resource collection của riêng bạn.

### Wrapping Nested Resources

Bạn có toàn quyền tự do xác định cách mà các quan hệ của resource của bạn được wrap. Nếu bạn muốn tất cả các resource collection được wrap trong một key `data`, kể cả việc lồng chúng, bạn nên định nghĩa một class resource collection cho mỗi resource và trả về collection trong một key `data`.

Tất nhiên, bạn có thể tự hỏi liệu rằng điều này có khiến resource ngoài cùng của bạn được wrap trong hai key `data`. Đừng lo lắng, Laravel sẽ không bao giờ để resource của bạn vô tình bị wrap đôi, vì vậy bạn không phải lo lắng về mức độ lồng nhau của resource collection mà bạn đang chuyển đổi:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

### Data Wrapping And Phân trang

Khi trả về các collection được phân trang trong một response resource, Laravel sẽ wrap dữ liệu resource của bạn trong một key `data` ngay cả khi phương thức `withoutWrapping` đã được gọi. Điều này là do các response được phân trang luôn chứa các key `meta` và `links` với thông tin về trạng thái của phân trang:

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

Bạn luôn có thể pass một instance phân trang cho phương thức `collection` của một resource hoặc cho một resource collectio tùy biến:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

Các response được phân trang luôn chứa các key `meta` và `links` với thông tin về trạng thái của phân trang:

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

Đôi khi bạn có thể chỉ muốn chứa một thuộc tính trong một response resource nếu một điều kiện nhất định được đáp ứng. Ví dụ: bạn có thể chỉ muốn chứa một giá trị nếu người dùng hiện tại là "quản trị viên". Laravel cung cấp nhiều phương thức helper để hỗ trợ bạn trong những tình huống này. Phương thức `when` có thể được sử dụng để thêm một điều kiện của một thuộc tính vào response resource:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($this->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Trong ví dụ này, khóa `secret` sẽ chỉ được trả về trong response resource nếu phương thức `$this->isAdmin()` trả về giá trị `true`. Nếu phương thức trả về `false`, thì khóa `secret` sẽ bị xóa khỏi response resource trước khi nó được gửi cho client. Phương thức `when` cho phép bạn định nghĩa một resource của bạn mà không cần dùng đến các câu lệnh có điều kiện khi xây dựng mảng.

Phương thức `when` cũng chấp nhận một Closure là tham số thứ hai của nó, cho phép bạn tính giá trị kết quả chỉ khi điều kiện đã cho là `true`:

    'secret' => $this->when($this->isAdmin(), function () {
        return 'secret-value';
    }),

> {tip} Hãy nhớ rằng, phương thức gọi trên proxy resource xuống instance model. Vì vậy, trong trường hợp này, phương thức `isAdmin` đang proxy cho model Eloquent ban đầu mà đã được chuyển cho resource.

#### Merging Điều kiện cho thuộc tính

Thỉnh thoảng bạn có thể có một số thuộc tính chỉ nên được đưa vào một response resource dựa trên cùng một điều kiện nào đó. Trong trường hợp này, bạn có thể sử dụng phương thức `mergeWhen` để chứa các thuộc tính trong response chỉ khi điều kiện đã cho là `true`:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($this->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Một lần nữa, nếu điều kiện đã cho là `false`, các thuộc tính này sẽ bị xóa ra khỏi response resource trước khi nó được gửi đến client.

> {note} Không nên sử dụng phương thức `mergeWhen` trong các mảng mà trộn giữa các khoá string và khóa numeric. Hơn nữa, nó không nên được sử dụng trong các mảng với các khóa numeric mà không được sắp xếp theo tuần tự.

<a name="conditional-relationships"></a>
### Điều kiện cho quan hệ

Ngoài các thuộc tính load có điều kiện, bạn cũng có thể thêm điều kiện cho các quan hệ trên các response resource của bạn dựa trên việc quan hệ đã được load trên model hay chưa. Điều này cho phép controller của bạn quyết định những quan hệ nào sẽ được load trên model và resource của bạn có thể dễ dàng chứa chúng chỉ khi chúng đã được load.

Cuối cùng, điều này giúp bạn dễ dàng tránh các vấn đề truy vấn "N+1" trong resource của bạn. Phương thức `whenLoaded` có thể được sử dụng để thêm một điều kiện để load một quan hệ. Để tránh load các quan hệ không cần thiết, phương thức này chấp nhận tên của quan hệ thay vì quan hệ chính nó:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Trong ví dụ này, nếu quan hệ chưa được load, khóa `posts` sẽ bị xóa ra khỏi response resource trước khi nó được gửi đến client.

#### Conditional Pivot Information

Ngoài điều kiện chứa dữ liệu về quan hệ trong các response resource của bạn, bạn có thể chứa một điều kiện cho dữ liệu từ các bảng trung gian của quan hệ nhiều-nhiều bằng cách sử dụng phương thức `whenPivotLoaded`. Phương thức `whenPivotLoaded` chấp nhận tên của bảng pivot làm tham số đầu tiên. Tham số thứ hai phải là một Closure định nghĩa giá trị được trả về nếu thông tin pivot có sẵn trên model:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_users', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>
### Thêm Meta Data

Một số tiêu chuẩn API JSON yêu cầu thêm dữ liệu meta vào các response của resource và resource collection của bạn. Điều này thường chứa những thông tin như `links` đến resource hoặc resource quan hệ hoặc dữ liệu meta về chính resource đó. Nếu bạn cần trả về thêm dữ liệu meta cho một resource, hãy đưa nó vào phương thức `toArray` của bạn. Ví dụ: bạn có thể chứa thông tin `link` khi chuyển đổi một resource collection:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
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

Khi trả về thêm dữ liệu meta từ resource của bạn, bạn sẽ không phải lo lắng về việc vô tình ghi đè các key `links` hoặc `meta` được Laravel tự động thêm khi trả về các response để phân trang. Bất kỳ `links` nào mà bạn đã định nghĩa sẽ được merge với các link được cung cấp bởi paginator.

#### Top Level Meta Data

Thỉnh thoảng bạn có thể chỉ muốn thêm một số dữ liệu meta nhất định vào một response resource nếu resource là resource ngoài cùng được trả về. Thông thường, điều này chứa thông tin meta về toàn bộ response. Để xác định dữ liệu meta này, hãy thêm một phương thức `with` vào class resource của bạn. Phương thức này sẽ trả về một mảng dữ liệu meta sẽ được chứa trong response resource chỉ khi resource là resource ngoài cùng được tạo:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * Get additional data that should be returned with the resource array.
         *
         * @param \Illuminate\Http\Request  $request
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

Bạn cũng có thể thêm dữ liệu cấp cao khi khởi tạo các instance resource trong route hoặc controller của bạn. Phương thức `additional`, có sẵn trên tất cả các resource, chấp nhận một mảng dữ liệu cần được thêm vào response resource:

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## Resource Responses

Như bạn đã đọc, resources có thể được trả về trực tiếp từ các route và controller:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

Tuy nhiên, thỉnh thoảng bạn có thể cần tùy biến HTTP response trả về trước khi nó được gửi đến client. Có hai cách để thực hiện điều này. Đầu tiên, bạn có thể kết hợp thêm phương thức `response` lên resource. Phương thức này sẽ trả về một instance `Illuminate\Http\Response`, cho phép bạn toàn quyền kiểm soát các header của response:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

Ngoài ra, bạn có thể định nghĩa một phương thức `withResponse` trong chính resource. Phương thức này sẽ được gọi khi resource được trả về là resource ngoài cùng nhất trong một response:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
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
         * @param  \Illuminate\Http\Request
         * @param  \Illuminate\Http\Response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }
