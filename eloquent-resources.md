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

Khi xây dựng API, bạn có thể cần một class chuyển đổi nằm giữa các model Eloquent và các response JSON được trả về cho người dùng application của bạn. Ví dụ: bạn có thể muốn hiển thị các thuộc tính nhất định cho một nhóm người dùng chứ không phải tất cả những người khác hoặc bạn có thể muốn luôn chứa các quan hệ nhất định trong định dạng JSON của các model của bạn. Các class resource của Eloquent cho phép bạn chuyển đổi một cách rõ ràng và dễ hiểu các model cũng như các collection model của bạn thành JSON.

Tất nhiên, bạn cũng có thể chuyển đổi các model hoặc collection Eloquent thành JSON bằng các phương thức `toJson` của chúng; tuy nhiên, các resource của Eloquent sẽ cung cấp nhiều khả năng kiểm soát mạnh mẽ và chi tiết hơn đối với quá trình chuyển hóa JSON của các model của bạn và các quan hệ của chúng.

<a name="generating-resources"></a>
## Tạo Resources

Để tạo một class resource, bạn có thể sử dụng lệnh Artisan `make:resource`. Mặc định, resource sẽ được lưu vào trong thư mục `app/Http/Resources` của application của bạn. Các resource sẽ được extend từ class `Illuminate\Http\Resources\Json\JsonResource`:

```shell
php artisan make:resource UserResource
```

<a name="generating-resource-collections"></a>
#### Resource Collections

Ngoài việc tạo các resource dùng để chuyển đổi cho các model riêng biệt, bạn cũng có thể tạo các resource để chuyển đổi một collection của model. Điều này cho phép JSON response của bạn có thể thêm các liên kết hoặc các thông tin khác có liên quan đến toàn bộ collection của một resource.

Để tạo một resource collection, bạn hãy sử dụng cờ `--collection` khi tạo resource. Hoặc có từ `Collection` trong tên của resource cũng cho Laravel biết rằng nó cần tạo ra một resource collection. Resource collection được extend từ class `Illuminate\Http\Resources\Json\ResourceCollection`:

```shell
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

<a name="concept-overview"></a>
## Khái niệm tổng quan

> [!NOTE]
> Đây là tổng quan về resource và resource collection. Bạn được khuyến khích đọc các phần khác của tài liệu này để hiểu sâu hơn về khả năng tùy biến và sức mạnh của các resource có thể cung cấp cho bạn.

Trước khi đi sâu vào tất cả các tùy chọn có sẵn cho bạn khi bạn viết resource, trước tiên chúng ta hãy xem về cách sử dụng resource trong Laravel. Một class resource sẽ đại diện cho một model cần chuyển đổi thành dạng JSON. Ví dụ, đây là một resource class `UserResource` đơn giản:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
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

Mọi class resource đều định nghĩa một phương thức `toArray` trả về mảng các thuộc tính sẽ được chuyển đổi thành JSON trước khi resource đó được trả về dưới dạng response từ một route hoặc một phương thức trong controller.

Lưu ý rằng chúng ta có thể truy cập vào các thuộc tính của model trực tiếp từ biến `$this`. Điều này là do một resource class sẽ tự động chuyển các thuộc tính và các phương thức xuống model để dễ dàng truy cập thuận tiện hơn. Sau khi resource đã được định nghĩa xong, nó có thể được trả về từ một route hoặc controller. Resource chấp nhận instance model thông qua hàm khởi tạo của nó:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function (string $id) {
        return new UserResource(User::findOrFail($id));
    });

<a name="resource-collections"></a>
### Resource Collections

Nếu bạn đang trả về một resource collection hoặc một response đang được phân trang, bạn nên sử dụng phương thức `collection` được cung cấp bởi resource class khi tạo instance resource trong route hoặc controller của bạn:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all());
    });

Chú ý rằng điều này sẽ không cho phép bạn thêm bất kỳ dữ liệu meta tuỳ chỉnh nào để có thể được trả về cùng với collection của bạn. Nếu bạn muốn tùy chỉnh response của resource collection, bạn có thể tạo một resource chuyên dụng để tạo collection:

```shell
php artisan make:resource UserCollection
```

Khi class resource collection đã được tạo, bạn có thể dễ dàng định nghĩa bất kỳ dữ liệu meta nào cần có trong response:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @return array<int|string, mixed>
         */
        public function toArray(Request $request): array
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

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="preserving-collection-keys"></a>
#### Preserving Collection Keys

Khi trả về một resource collection từ một route, Laravel sẽ reset lại các khóa của collection để chúng có thứ tự sắp xếp từ 0. Tuy nhiên, bạn có thể thêm thuộc tính `preserveKeys` vào class resource của bạn để cho biết liệu khóa collection có được giữ nguyên hay không:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Indicates if the resource's collection keys should be preserved.
         *
         * @var bool
         */
        public $preserveKeys = true;
    }

Khi thuộc tính `secureKeys` được set thành `true`, các khóa của collection sẽ được giữ nguyên khi collection được trả về từ mmột route hoặc một controller:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all()->keyBy->id);
    });

<a name="customizing-the-underlying-resource-class"></a>
#### Tùy biến Resource Class cơ bản

Thông thường, thuộc tính `$this->collection` của một resource collection sẽ được tự động nối với kết quả của việc ánh xạ của từng item của collection với class resource của nó. Class resource được giả định là tên class của collection mà không có chuỗi `Collection` ở cuối tên class. Ngoài ra, tùy thuộc vào sở thích cá nhân của bạn, resource class có thể có hoặc không có hậu tố `Resource`.

Ví dụ: `UserCollection` sẽ thử ánh xạ các instance user vào một resource có thể `UserResource`. Để tùy biến hành động này, bạn có thể ghi đè thuộc tính `$collects` của resource collection của bạn:

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
        public $collects = Member::class;
    }

<a name="writing-resources"></a>
## Viết Resources

> [!NOTE]
> Nếu bạn chưa đọc phần [khái niệm tổng quan](#concept-overview), bạn được khuyến khích đọc nó trước khi tiếp tục với phần này.

Resource chỉ cần chuyển đổi một model thành một mảng. Vì vậy, mỗi resource chứa một phương thức `toArray` để giúp chuyển các thuộc tính của model của bạn thành một mảng thân thiện với API để có thể được trả về từ các route hoặc controller của ứng dụng của bạn:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
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

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function (string $id) {
        return new UserResource(User::findOrFail($id));
    });

<a name="relationships"></a>
#### Relationships

Nếu bạn muốn thêm các quan hệ vào trong một response của bạn, bạn có thể thêm chúng vào mảng được trả về trong phương thức `toArray` của resource của bạn. Trong ví dụ này, chúng ra sẽ sử dụng phương thức `collection` của resource `PostResource` để thêm các post trên blog của người dùng vào response của resource:

    use App\Http\Resources\PostResource;
    use Illuminate\Http\Request;

    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
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

> [!NOTE]
> Nếu bạn chỉ thêm các quan hệ chỉ khi chúng đã được load, hãy xem tài liệu về [điều kiện cho quan hệ](#conditional-relationships).

<a name="writing-resource-collections"></a>
#### Resource Collections

Trong khi các resource sẽ chuyển một model thành một mảng, thì các resource collection sẽ chuyển một collection của model thành một mảng. Tuy nhiên, không nhất thiết phải định nghĩa một class resource collection cho từng loại model của bạn vì tất cả các resource đều được cung cấp một phương thức `collection` để tạo các resource collection "ad-hoc" một cách nhanh chóng:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all());
    });

Tuy nhiên, nếu bạn cần tùy chỉnh dữ liệu meta được trả về cùng với collection, bạn sẽ cần phải định nghĩa riêng một resource collection của chính bạn:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
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

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### Data Wrapping

Mặc định, resource ngoài cùng của bạn sẽ được bao bọc bởi một key `data` khi chúng được chuyển đổi thành JSON. Vì thế, một response resource collection có thể trông như sau:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ]
}
```

Nếu bạn muốn vô hiệu hóa việc bao bọc resource này, bạn nên gọi phương thức `withoutWrapping` trên class `Illuminate\Http\Resources\Json\JsonResource`. Thông thường, bạn nên gọi phương thức này từ `AppServiceProvider` hoặc từ một [service provider](/docs/{{version}}/providers) khác để được load cho mọi request trong application của bạn:

    <?php

    namespace App\Providers;

    use Illuminate\Http\Resources\Json\JsonResource;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            JsonResource::withoutWrapping();
        }
    }

> [!WARNING]
> Phương thức `withoutWrapping` chỉ ảnh hưởng đến response ở ngoài cùng và sẽ không xóa các key `data` mà bạn đã thêm vào bên trong resource collection.

<a name="wrapping-nested-resources"></a>
#### Wrapping Nested Resources

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
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return ['data' => $this->collection];
        }
    }

<a name="data-wrapping-and-pagination"></a>
#### Data Wrapping And Phân trang

Khi trả về một collection được phân trang thông qua một response resource, Laravel sẽ bao bọc dữ liệu resource của bạn trong một key `data` ngay cả khi phương thức `withoutWrapping` đã được gọi. Điều này là do trong response được phân trang luôn chứa các key `meta` và `links` cùng với các thông tin về trạng thái của phân trang:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="pagination"></a>
### Phân trang

Bạn có thể truyền một instance phân trang của Laravel cho phương thức `collection` của một resource hoặc một resource collection tùy biến:

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

Các response được phân trang luôn chứa các key `meta` và `links` cùng với các thông tin về trạng thái của phân trang:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="customizing-the-pagination-information"></a>
#### Customizing the Pagination Information

Nếu bạn muốn tùy chỉnh thông tin được chứa trong các key `links` hoặc `meta` của response phân trang, bạn có thể định nghĩa phương thức `paginationInformation` trên các resource. Phương thức này sẽ nhận vào dữ liệu `$paginated` và một mảng thông tin `$default` có chứa các key `links` và `meta`:

    /**
     * Customize the pagination information for the resource.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  array $paginated
     * @param  array $default
     * @return array
     */
    public function paginationInformation($request, $paginated, $default)
    {
        $default['links']['custom'] = 'https://example.com';

        return $default;
    }

<a name="conditional-attributes"></a>
### Điều kiện cho thuộc tính

Đôi khi bạn có thể chỉ muốn thêm một số thuộc tính vào trong một response resource nếu một điều kiện được đáp ứng. Ví dụ: bạn có thể chỉ muốn thêm một giá trị nếu người dùng hiện tại đang là "quản trị viên". Laravel cung cấp nhiều phương thức helper để hỗ trợ cho bạn trong những tình huống này. Phương thức `when` có thể được sử dụng để thêm một điều kiện cho một thuộc tính vào response resource:

    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Trong ví dụ này, khóa `secret` sẽ chỉ được trả về trong response resource nếu phương thức `$this->isAdmin()` của người dùng hiện tại trả về giá trị `true`. Nếu phương thức trả về giá trị `false`, thì khóa `secret` sẽ bị xóa khỏi response resource trước khi nó được gửi về cho client. Phương thức `when` cho phép bạn định nghĩa một resource mà không cần dùng đến các câu lệnh có điều kiện khi xây dựng một mảng.

Phương thức `when` cũng chấp nhận một closure là tham số thứ hai của nó, cho phép bạn tính toán giá trị trả về nếu điều kiện đã cho là `true`:

    'secret' => $this->when($request->user()->isAdmin(), function () {
        return 'secret-value';
    }),

Phương thức `whenHas` có thể được sử dụng để chứa một thuộc tính nếu nó thực sự có trên model:

    'name' => $this->whenHas('name'),

Ngoài ra, phương thức `whenNotNull` có thể được sử dụng để đưa một thuộc tính vào resource response nếu thuộc tính đó không rỗng:

    'name' => $this->whenNotNull($this->name),

<a name="merging-conditional-attributes"></a>
#### Merging Điều kiện cho thuộc tính

Thỉnh thoảng bạn có thể có một số thuộc tính chỉ được đưa vào trong một response resource dựa trên cùng một điều kiện nào đó. Trong trường hợp này, bạn có thể sử dụng phương thức `mergeWhen` để thêm các thuộc tính vào trong response chỉ khi một điều kiện là `true`:

    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($request->user()->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Một lần nữa, nếu điều kiện trả về giá trị là `false`, các thuộc tính này sẽ bị xóa ra khỏi response resource trước khi nó được gửi về client.

> [!WARNING]
> Không nên sử dụng phương thức `mergeWhen` trong các mảng mà có sử dụng cả khoá string và khóa numeric. Hơn nữa, nó cũng không nên được sử dụng trong các mảng với các khóa numeric không được sắp xếp theo tuần tự.

<a name="conditional-relationships"></a>
### Điều kiện cho quan hệ

Ngoài các thuộc tính load có điều kiện, bạn cũng có thể thêm các điều kiện cho các quan hệ trong các response resource dựa trên việc quan hệ đó đã được load trên model hay chưa. Điều này cho phép controller của bạn quyết định xem những quan hệ nào sẽ được load trong model và resource của bạn có thể dễ dàng chứa nó chỉ khi nó đã được load. Cuối cùng, điều này giúp dễ dàng tránh được các sự cố truy vấn "N+1" trong resource của bạn.

Phương thức `whenLoaded` có thể được sử dụng để load một quan hệ theo điều kiện. Để tránh load các quan hệ không cần thiết, phương thức này chấp nhận tên của quan hệ thay vì chính quan hệ đó:

    use App\Http\Resources\PostResource;

    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
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

<a name="conditional-relationship-counts"></a>
#### Conditional Relationship Counts

Ngoài điều kiện cho quan hệ, bạn có thể thêm "counts" quan hệ trên các resource response của bạn dựa trên việc count của quan hệ đó đã được load trên model hay chưa:

    new UserResource($user->loadCount('posts'));

Phương thức `whenCounted` có thể được sử dụng để đưa count quan hệ vào resource response của bạn một cách có điều kiện. Phương thức này tránh việc chứa thuộc tính một cách không cần thiết nếu không có count quan hệ đó:

    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts_count' => $this->whenCounted('posts'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Trong ví dụ này, nếu count quan hệ `posts` chưa được load, khóa `posts_count` sẽ bị xóa khỏi resource response trước khi nó được gửi đến client.

Các loại tính toán khác, chẳng hạn như `avg`, `sum`, `min` và `max` cũng có thể được load có điều kiện bằng phương thức `whenAggregated`:

```php
'words_avg' => $this->whenAggregated('posts', 'words', 'avg'),
'words_sum' => $this->whenAggregated('posts', 'words', 'sum'),
'words_min' => $this->whenAggregated('posts', 'words', 'min'),
'words_max' => $this->whenAggregated('posts', 'words', 'max'),
```

<a name="conditional-pivot-information"></a>
#### Conditional Pivot Information

Ngoài việc thêm các thông tin quan hệ có điều kiện vào trong các response resource của bạn, bạn cũng có thể thêm các điều kiện cho dữ liệu từ các bảng trung gian của quan hệ nhiều-nhiều bằng cách sử dụng phương thức `whenPivotLoaded`. Phương thức `whenPivotLoaded` chấp nhận tên của bảng pivot làm tham số đầu tiên. Tham số thứ hai phải là một closure sẽ trả về giá trị được trả về nếu thông tin pivot đó tồn tại trên model:

    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_user', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

Nếu quan hệ của bạn đang sử dụng một [model bảng trung gian tùy chỉnh](/docs/{{version}}/eloquent-relationships#defining-custom-intermediate-table-models), bạn có thể truyền một instance của model bảng trung gian làm tham số đầu tiên cho phương thức `whenPivotLoaded`:

    'expires_at' => $this->whenPivotLoaded(new Membership, function () {
        return $this->pivot->expires_at;
    }),

Nếu bảng trung gian của bạn đang sử dụng một tên accessor khác không phải là `pivot`, bạn có thể sử dụng phương thức `whenPivotLoadedAs`:

    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
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
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

Khi trả về thêm một dữ liệu meta từ resource của bạn, bạn sẽ không phải lo lắng về việc vô tình ghi đè các key `links` hoặc `meta` được Laravel tự động thêm khi trả về các response để phân trang. Bất kỳ `links` nào mà bạn đã định nghĩa sẽ được merge với các link đã được cung cấp bởi paginator.

<a name="top-level-meta-data"></a>
#### Top Level Meta Data

Thỉnh thoảng bạn có thể chỉ muốn thêm một số dữ liệu meta nhất định vào một response resource nếu resource đó là resource ngoài cùng được trả về. Thông thường, điều này sẽ chứa những thông tin meta về toàn bộ response. Để định nghĩa những dữ liệu meta như thế này, hãy thêm một phương thức `with` vào trong class resource của bạn. Phương thức này sẽ trả về một mảng dữ liệu meta sẽ được chứa trong response resource chỉ khi resource đó là resource ngoài cùng được chuyển đổi:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return parent::toArray($request);
        }

        /**
         * Get additional data that should be returned with the resource array.
         *
         * @return array<string, mixed>
         */
        public function with(Request $request): array
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

<a name="adding-meta-data-when-constructing-resources"></a>
#### Thêm Meta Data When Constructing Resources

Bạn cũng có thể thêm dữ liệu khi khởi tạo một instance resource trong route hoặc controller của bạn. Phương thức `additional`, có sẵn trên tất cả các resource, chấp nhận một mảng dữ liệu cần được thêm vào response resource:

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## Resource Responses

Như bạn đã đọc, resources có thể được trả về trực tiếp từ một route hoặc một controller:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function (string $id) {
        return new UserResource(User::findOrFail($id));
    });

Tuy nhiên, thỉnh thoảng bạn có thể cần tùy biến HTTP response trước khi nó được gửi về client. Có hai cách để thực hiện điều này. Đầu tiên, bạn có thể gắn thêm phương thức `response` vào trong resource. Phương thức này sẽ trả về một instance `Illuminate\Http\JsonResponse`, cho phép bạn toàn quyền kiểm soát các header của response:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

Ngoài ra, bạn cũng có thể định nghĩa một phương thức `withResponse` vào trong chính resource của bạn. Phương thức này sẽ được gọi khi resource được trả về là resource ngoài cùng nhất trong một response:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\JsonResponse;
    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * Customize the outgoing response for the resource.
         */
        public function withResponse(Request $request, JsonResponse $response): void
        {
            $response->header('X-Value', 'True');
        }
    }
