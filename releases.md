# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 9](#laravel-9)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Laravel và các package khác của nó tuân theo [Phiên bản Semantic](https://semver.org). Các phiên bản được phát hành chính thức của framework được phát hành một năm một lần (khoảng tháng 2), trong khi các bản phát hành nhỏ hơn và các bản sửa lỗi có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ và các bản sửa lỗi sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `^9.0`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

<a name="named-arguments"></a>
#### Named Arguments

[Đặt tên cho tham số](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) không nằm trong nguyên tắc tương thích ngược của Laravel. Chúng tôi có thể đổi tên các tham số bất cứ khi nào để cải thiện codebase của Laravel. Do đó, việc sử dụng các kiểu đặt tên cho tham số khi gọi các phương thức của Laravel nên được thực hiện một cách cẩn trọng và nên hiểu rằng tên tham số có thể thay đổi trong tương lai.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với tất cả các bản phát hành chính thức, các bản sửa lỗi sẽ được cung cấp trong 18 tháng và các bản sửa lỗi bảo mật được cung cấp trong 2 năm. Đối với tất cả các thư viện, bao gồm cả Lumen, chỉ bản phát hành chính thức mới nhất mới nhận được các bản sửa lỗi. Ngoài ra, hãy xem các phiên bản cơ sở dữ liệu [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

| Version | PHP (*) | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- | --- |
| 6 (LTS) | 7.2 - 8.0 | ngày 3 tháng 9 năm 2019 | ngày 25 tháng 1 năm 2022 | ngày 6 tháng 9 năm 2022 |
| 7 | 7.2 - 8.0 | ngày 3 tháng 3 năm 2020 | ngày 6 tháng 10 năm 2020 | ngày 3 tháng 3 năm 2021 |
| 8 | 7.3 - 8.1 | ngày 8 tháng 9 năm 2020 | ngày 26 tháng 7 năm 2022 | ngày 24 tháng 1 năm 2023 |
| 9 | 8.0 - 8.2 | ngày 8 tháng 2 năm 2022 | ngày 8 tháng 8 năm 2023 | ngày 6 tháng 2 năm 2024 |
| 10 | 8.1 - 8.2 | Q1 2023 | ngày 6 tháng 8 năm 2024 | ngày 4 tháng 2 năm 2025 |

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>End of life</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>Security fixes only</div>
    </div>
</div>

(*) Supported PHP versions

<a name="laravel-9"></a>
## Laravel 9

Như bạn có thể biết, Laravel đã chuyển sang phát hành theo năm từ bản phát hành Laravel 8. Trước đây, các phiên bản chính được phát hành sau mỗi 6 tháng. Sự chuyển đổi này nhằm mục đích giảm bớt gánh nặng bảo trì cho cộng đồng và thách thức nhóm phát triển của chúng tôi cung cấp các tính năng mới tuyệt vời, mạnh mẽ mà không giới thiệu các thay đổi nghiêm trọng. Do đó, chúng tôi đã cung cấp nhiều tính năng mạnh mẽ cho Laravel 8 mà không phá vỡ khả năng tương thích ngược, chẳng hạn như hỗ trợ testing song song, bộ công cụ khởi tạo Breeze được cải tiến, cải thiện HTTP client và thậm chí cả các loại quan hệ Eloquent mới như là "có một trong nhiều".

Do đó, cam kết cung cấp những tính năng mới tuyệt vời trong bản phát hành hiện tại nên có thể sẽ khiến các bản phát hành "chính" trong tương lai chủ yếu được sử dụng cho các công việc "bảo trì" như nâng cấp các library, có thể được thấy trong các release note này.

Laravel 9 vẫn sẽ tiếp tục những cải tiến được thực hiện trong Laravel 8.x bằng cách hỗ trợ các component Symfony 6.0, Symfony Mailer, Flysystem 3.0, cải thiện output `route:list`, driver cơ sở dữ liệu cho Laravel Scout, cú pháp accessor/mutator Eloquent mới, liên kết route ngầm thông qua Enum và một loạt các bản sửa lỗi và cải tiến khả năng sử dụng khác.

<a name="php-8"></a>
### PHP 8.0

Laravel 9.x requires a minimum PHP version of 8.0.

<a name="symfony-mailer"></a>
### Symfony Mailer

_Hỗ trợ Symfony Mailer được đóng góp bởi [Dries Vints](https://github.com/driesvints)_, [James Brooks](https://github.com/jbrooksuk), và [Julius Kiekbusch](https://github.com/Jubeki).

Các phiên bản trước của Laravel sử dụng thư viện [Swift Mailer](https://swiftmailer.symfony.com/docs/introduction.html) để gửi email. Tuy nhiên, thư viện này không còn được bảo trì nữa và đã được thay đổi bằng Symfony Mailer.

Vui lòng xem [hướng dẫn nâng cấp](/docs/{{version}}/upgrade#symfony-mailer) để tìm hiểu thêm về cách đảm bảo ứng dụng của bạn tương thích với Symfony Mailer.

<a name="flysystem-3"></a>
### Flysystem 3.x

_Hỗ trợ Flysystem 3.x được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel 9.x nâng cấp library Flysystem của chúng tôi lên Flysystem 3.x. Flysystem sẽ hỗ trợ tất cả các tương tác filesystem do facade `Storage` cung cấp.

Vui lòng xem [hướng dẫn nâng cấp](/docs/{{version}}/upgrade#flysystem-3) để tìm hiểu thêm về cách đảm bảo ứng dụng của bạn tương thích với Flysystem 3.x.

<a name="eloquent-accessors-and-mutators"></a>
### Improved Eloquent Accessors / Mutators

_Cải tiến accessors / mutators của Eloquent được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel 9.x cung cấp một cách mới để định nghĩa các Eloquent [accessors và mutators](/docs/{{version}}/eloquent-mutators#accessors-and-mutators). Trong các phiên bản Laravel trước, cách duy nhất để định nghĩa các accessors và mutators này là định nghĩa các phương thức có tiền tố trên model của bạn như sau:

```php
public function getNameAttribute($value)
{
    return strtoupper($value);
}

public function setNameAttribute($value)
{
    $this->attributes['name'] = $value;
}
```

Tuy nhiên, trong Laravel 9.x, bạn có thể định nghĩa một accessor và mutator trong một phương thức duy nhất, không có tiền tố bằng cách khai báo kiểu trả về là `Illuminate\Database\Eloquent\Casts\Attribute`:

```php
use Illuminate\Database\Eloquent\Casts\Attribute;

public function name(): Attribute
{
    return new Attribute(
        get: fn ($value) => strtoupper($value),
        set: fn ($value) => $value,
    );
}
```

Ngoài ra, cách tiếp cận mới này là để định nghĩa các accessor sẽ lưu cache các giá trị đối tượng được trả về bởi thuộc tính, giống như [các class cast kiểu tùy biến](/docs/{{version}}/eloquent-mutators#custom-casts):

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

public function address(): Attribute
{
    return new Attribute(
        get: fn ($value, $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```

<a name="enum-casting"></a>
### Enum Eloquent Attribute Casting

> **Warning**
> Enums casting chỉ khả dụng trên PHP 8.1+.

_Enum casting được đóng góp bởi [Mohamed Said](https://github.com/themsaid)_.

Eloquent hiện cho phép bạn cast kiểu các giá trị thuộc tính của bạn sang PHP ["backed" Enums](https://www.php.net/manual/en/language.enumerations.backed.php). Để thực hiện điều này, bạn có thể chỉ định thuộc tính và enum nào mà bạn muốn cast kiểu trong mảng thuộc tính `$casts` của model:

    use App\Enums\ServerStatus;

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'status' => ServerStatus::class,
    ];

Sau khi bạn đã định nghĩa kiểu cast trên model của bạn, thuộc tính được chỉ định sẽ tự động được cast kiểu sang hoặc từ một enum khi bạn tương tác với thuộc tính đó:

    if ($server->status == ServerStatus::Provisioned) {
        $server->status = ServerStatus::Ready;

        $server->save();
    }

<a name="implicit-route-bindings-with-enums"></a>
### Implicit Route Bindings With Enums

_Liên kết route ẩn được đóng góp bởi [Nuno Maduro](https://github.com/nunomaduro)_.

PHP 8.1 dã giới thiệu hỗ trợ [Enum](https://www.php.net/manual/en/language.enumerations.backed.php). Laravel 9.x cũng giới thiệu khả năng khai báo kiểu cho Enum trên định nghĩa route của bạn và Laravel sẽ chỉ gọi route đó nếu tham số của route đó là một giá trị Enum hợp lệ trong URI. Nếu không, response HTTP 404 sẽ được tự động trả về. Ví dụ, với Enum sau:

```php
enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

Bạn có thể định nghĩa một route chỉ được gọi nếu tham số route `{category}` là `fruits` hoặc `people`. Nếu không, response HTTP 404 sẽ được trả về:

```php
Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="forced-scoping-of-route-bindings"></a>
### Forced Scoping Of Route Bindings

_Liên kết phạm vi bắt buộc được đóng góp bởi [Claudio Dekker](https://github.com/claudiodekker)_.

Trong các phiên bản trước của Laravel, bạn có thể muốn giới hạn phạm vi model Eloquent thứ hai trong định nghĩa route sao cho nó phải là con của model Eloquent trước đó. Ví dụ: hãy xem xét định nghĩa route sau để lấy ra một bài đăng blog bằng slug của một người dùng cụ thể:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

Khi sử dụng liên kết ngầm có khóa tùy chỉnh làm tham số route lồng nhau, Laravel sẽ tự động xác định phạm vi truy vấn để lấy ra model lồng nhau của cha mẹ bằng cách sử dụng các quy ước để đoán tên quan hệ trên cha mẹ. Tuy nhiên, hành vi này trước đây chỉ được Laravel hỗ trợ khi sử dụng khóa tùy chỉnh cho liên kết route con.

Tuy nhiên, trong Laravel 9.x, bây giờ bạn có thể hướng dẫn Laravel xác định phạm vi liên kết "con" ngay cả khi khóa tùy chỉnh không được cung cấp. Để làm như vậy, bạn có thể gọi phương thức `scopeBindings` khi định nghĩa route của bạn:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    })->scopeBindings();

Hoặc, bạn có thể hướng dẫn nhóm route sử dụng các liên kết có phạm vi:

    Route::scopeBindings()->group(function () {
        Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
            return $post;
        });
    });

<a name="controller-route-groups"></a>
### Controller Route Groups

_Group route được đóng góp bởi [Luke Downing](https://github.com/lukeraymonddowning)_.

Bây giờ bạn có thể sử dụng phương thức `controller` để định nghĩa controller chung cho tất cả các route có trong nhóm. Sau đó, khi định nghĩa các route, bạn chỉ cần cung cấp phương thức controller mà chúng gọi:

    use App\Http\Controllers\OrderController;

    Route::controller(OrderController::class)->group(function () {
        Route::get('/orders/{id}', 'show');
        Route::post('/orders', 'store');
    });

<a name="full-text"></a>
### Full Text Indexes / Where Clauses

_Full text index và mệnh đề "where" được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell) và [Dries Vints](https://github.com/driesvints)_.

Khi sử dụng MySQL hoặc PostgreSQL, phương thức `fullText` hiện có thể được thêm vào định nghĩa cột để tạo index full text:

    $table->text('bio')->fullText();

Ngoài ra, các phương thức `whereFullText` và `orWhereFullText` có thể được sử dụng để thêm các mệnh đề "where" full text vào truy vấn cho các cột có [index full text](/docs/{{version}}/migrations#available-index-types). Các phương thức này sẽ được Laravel chuyển thành câu lệnh SQL phù hợp cho hệ thống cơ sở dữ liệu. Ví dụ, một mệnh đề `MATCH AGAINST` sẽ được tạo cho các ứng dụng sử dụng MySQL:

    $users = DB::table('users')
               ->whereFullText('bio', 'web developer')
               ->get();

<a name="laravel-scout-database-engine"></a>
### Laravel Scout Database Engine

_Laravel Scout database engine được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell) và [Dries Vints](https://github.com/driesvints)_.

Nếu ứng dụng của bạn tương tác với các cơ sở dữ liệu có quy mô vừa và nhỏ hoặc có khối lượng công việc nhẹ, giờ đây bạn có thể sử dụng "database" engine của Scout thay vì một dịch vụ tìm kiếm chuyên biệt như Algolia hoặc MeiliSearch. Database engine sẽ sử dụng các mệnh đề "where like" và index full text khi lọc kết quả từ cơ sở dữ liệu của bạn để xác định kết quả tìm kiếm phù hợp cho truy vấn của bạn.

Để tìm hiểu thêm về database engine của Scout, hãy tham khảo [tài liệu Scout](/docs/{{version}}/scout).

<a name="rendering-inline-blade-templates"></a>
### Rendering Inline Blade Templates

_Render inline Blade template được đóng góp bởi [Jason Beggs](https://github.com/jasonlbeggs). Render inline Blade component được đóng góp bởi [Toby Zerner](https://github.com/tobyzerner)_.

Thỉnh thoảng bạn có thể cần chuyển chuỗi raw Blade template string thành một chuỗi HTML hợp lệ. Bạn có thể thực hiện điều này bằng phương thức `render` được cung cấp bởi facade `Blade`. Phương thức `render` sẽ chấp nhận chuỗi Blade template string và một mảng dữ liệu tùy chọn để cung cấp cho template:

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Tương tự như vậy, phương thức `renderComponent` cũng có thể được sử dụng để hiển thị một class component nhất định bằng cách truyền instance component đó cho phương thức:

```php
use App\View\Components\HelloComponent;

return Blade::renderComponent(new HelloComponent('Julian Bashir'));
```

<a name="slot-name-shortcut"></a>
### Slot Name Shortcut

_Các shortcut slot name được đóng góp bởi [Caleb Porzio](https://github.com/calebporzio)._

Trong các phiên bản trước của Laravel, tên của slot được cung cấp bằng cách sử dụng thuộc tính `name` trên tag `x-slot`:

```blade
<x-alert>
    <x-slot name="title">
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

Tuy nhiên, bắt đầu từ Laravel 9.x, bạn có thể chỉ định tên của slot bằng cú pháp ngắn gọn hơn và tiện lợi hơn:

```xml
<x-slot:title>
    Server Error
</x-slot>
```

<a name="checked-selected-blade-directives"></a>
### Checked / Selected Blade Directives

_Các lệnh Blade checked và selected được đóng góp bởi [Ash Allen](https://github.com/ash-jc-allen) và [Taylor Otwell](https://github.com/taylorotwell)_.

Để thuận tiện, bây giờ bạn có thể sử dụng lệnh `@checked` để dễ dàng chỉ ra liệu một checkbox HTML nhất định có được "checked" hay không. Lệnh này sẽ `checked` nếu điều kiện được cung cấp là `true`:

```blade
<input type="checkbox"
        name="active"
        value="active"
        @checked(old('active', $user->active)) />
```

Tương tự như vậy, lệnh `@selected` có thể được sử dụng để chỉ ra liệu một tùy chọn chọn nhất định có nên được "selected" hay không:

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

<a name="bootstrap-5-pagination-views"></a>
### Bootstrap 5 Pagination Views

_Bootstrap 5 pagination view được đóng góp bởi [Jared Lewis](https://github.com/jrd-lewis)_.

Laravel hiện có chứa các pagination view được xây dựng bằng [Bootstrap 5](https://getbootstrap.com/). Để sử dụng các view này thay vì view Tailwind mặc định, bạn có thể gọi phương thức `useBootstrapFive` ​​của paginator trong phương thức `boot` của class `App\Providers\AppServiceProvider` của bạn:

    use Illuminate\Pagination\Paginator;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Paginator::useBootstrapFive();
    }

<a name="improved-validation-of-nested-array-data"></a>
### Improved Validation Of Nested Array Data

_Việc validation các array input lồng nhau được đóng góp bởi [Steve Bauman](https://github.com/stevebauman)_.

Thỉnh thoảng bạn có thể cần truy cập vào giá trị cho một phần tử mảng lồng nhau trong khi gán quy tắc validation cho một thuộc tính. Bây giờ bạn có thể thực hiện việc này bằng phương thức `Rule::forEach`. Phương thức `forEach` sẽ chấp nhận một closure sẽ được gọi cho mỗi lần lặp của thuộc tính mảng đang được validation và sẽ nhận vào giá trị của thuộc tính và tên thuộc tính được fully-expanded, rõ ràng. Closure sẽ trả về một mảng các quy tắc để gán cho phần tử mảng:

    use App\Rules\HasPermission;
    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    $validator = Validator::make($request->all(), [
        'companies.*.id' => Rule::forEach(function ($value, $attribute) {
            return [
                Rule::exists(Company::class, 'id'),
                new HasPermission('manage-company', $value),
            ];
        }),
    ]);

<a name="laravel-breeze-api"></a>
### Laravel Breeze API & Next.js

_Scaffolding API Laravel Breeze và bộ công cụ khởi tạo Next.js được đóng góp bởi [Taylor Otwell](https://github.com/taylorotwell) và [Miguel Piedrafita](https://twitter.com/m1guelpf)_.

Bộ công cụ khởi tạo [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-next) đã nhận thêm chế độ scaffolding "API" và [frontend implementation](https://github.com/laravel/breeze-next) từ [Next.js](https://nextjs.org). Chế độ scaffolding của bộ công cụ khởi tạo này có thể được sử dụng để khởi tạo các ứng dụng Laravel của bạn mà đóng vai trò là API xác thực Laravel Sanctum cho frontend JavaScript.

<a name="exception-page"></a>
### Improved Ignition Exception Page

_Ignition được phát triển bởi [Spatie](https://spatie.be/)._

Ignition, trang gỡ lỗi exception open source do Spatie tạo, đã được thiết kế lại hoàn toàn. Phiên bản cải tiến Ignition mới đã đi kèm với Laravel 9.x và chứa các theme light / dark, chức năng "mở trong editor" có thể tùy chỉnh và nhiều hơn nữa.

<p align="center">
<img width="100%" src="https://user-images.githubusercontent.com/483853/149235404-f7caba56-ebdf-499e-9883-cac5d5610369.png"/>
</p>

<a name="improved-route-list"></a>
### Improved `route:list` CLI Output

_Output CLI `route:list` đã được cải tiến và được đóng góp bởi [Nuno Maduro](https://github.com/nunomaduro)_.

Output CLI `route:list` đã được cải tiến đáng kể cho phiên bản phát hành Laravel 9.x, mang đến trải nghiệm mới tuyệt vời khi khám phá định nghĩa route của bạn.

<p align="center">
<img src="https://user-images.githubusercontent.com/5457236/148321982-38c8b869-f188-4f42-a3cc-a03451d5216c.png"/>
</p>

<a name="test-coverage-support-on-artisan-test-Command"></a>
### Test Coverage Using Artisan `test` Command

_Test coverage khi sử dụng lệnh `test` của Artisan được đóng góp bởi [Nuno Maduro](https://github.com/nunomaduro)_.

Lệnh `test` của Artisan đã nhận được tùy chọn `--coverage` mới mà bạn có thể sử dụng để khám phá lượng bao phủ code mà các bài kiểm tra của bạn cung cấp cho ứng dụng của bạn:

```shell
php artisan test --coverage
```

Kết quả kiểm tra sẽ được hiển thị trực tiếp trong output CLI.

<p align="center">
<img width="100%" src="https://user-images.githubusercontent.com/5457236/150133237-440290c2-3538-4d8e-8eac-4fdd5ec7bd9e.png"/>
</p>

Ngoài ra, nếu bạn muốn chỉ định ngưỡng tối thiểu mà tỷ lệ bao phủ code của bài test của bạn phải đạt được, bạn có thể sử dụng tùy chọn `--min`. Bài test sẽ thất bại nếu ngưỡng tối thiểu đã cho không đạt được:

```shell
php artisan test --coverage --min=80.3
```

<p align="center">
<img width="100%" src="https://user-images.githubusercontent.com/5457236/149989853-a29a7629-2bfa-4bf3-bbf7-cdba339ec157.png"/>
</p>

<a name="soketi-echo-server"></a>
### Soketi Echo Server

_Soketi Echo server được phát triển bởi [Alex Renoki](https://github.com/rennokki)_.

Mặc dù không dành riêng cho Laravel 9.x, Laravel gần đây đã hỗ trợ tài liệu của Soketi, một máy chủ Web Socket tương thích với [Laravel Echo](/docs/{{version}}/broadcasting) được viết cho Node.js. Soketi cung cấp một giải pháp open source thay thế tuyệt vời cho Pusher và Ably cho những ứng dụng muốn quản lý Web Socket server của riêng họ.

Để biết thêm thông tin về cách sử dụng Soketi, vui lòng tham khảo [tài liệu broadcasting](/docs/{{version}}/broadcasting) và [tài liệu Soketi](https://docs.soketi.app/).

<a name="improved-collections-ide-support"></a>
### Improved Collections IDE Support

_Cải thiện hỗ trợ IDE cho collection được đóng góp bởi [Nuno Maduro](https://github.com/nunomaduro)_.

Laravel 9.x đã thêm các cải tiến về định nghĩa kiểu "chung" vào các component collections, cải thiện IDE và hỗ trợ phân tích tĩnh. Các IDE như [PHPStorm](https://blog.jetbrains.com/phpstorm/2021/12/phpstorm-2021-3-release/#support_for_future_laravel_collections) hoặc các công cụ phân tích tĩnh như [PHPStan](https://phpstan.org) giờ đây sẽ hiểu rõ hơn về các collection Laravel chính xác hớn.

<p align="center">
<img width="100%" src="https://user-images.githubusercontent.com/5457236/151783350-ed301660-1e09-44c1-b549-85c6db3f078d.gif"/>
</p>

<a name="new-helpers"></a>
### New Helpers

Laravel 9.x đã giới thiệu hai helper mới, tiện lợi mà bạn có thể sử dụng trong ứng dụng của bạn.

<a name="new-helpers-str"></a>
#### `str`

Hàm `str` trả về một instance `Illuminate\Support\Stringable` mới cho chuỗi đã cho. Hàm này tương đương với phương thức `Str::of`:

    $string = str('Taylor')->append(' Otwell');

    // 'Taylor Otwell'

Nếu không cung cấp tham số cho hàm `str`, hàm sẽ trả về một instance của `Illuminate\Support\Str`:

    $snake = str()->snake('LaravelFramework');

    // 'laravel_framework'

<a name="new-helpers-to-route"></a>
#### `to_route`

Hàm `to_route` sẽ tạo ra response HTTP chuyển hướng cho một route đã được đặt tên nhất định, cung cấp một cách dễ hiểu để chuyển hướng đến các route đã được đặt tên từ các route và controller của bạn:

    return to_route('users.show', ['user' => 1]);

Nếu cần, bạn có thể truyền HTTP status code cần gán cho lệnh chuyển hướng và bất kỳ response header nào làm tham số thứ ba và thứ tư cho phương thức to_route:

    return to_route('users.show', ['user' => 1], 302, ['X-Framework' => 'Laravel']);
