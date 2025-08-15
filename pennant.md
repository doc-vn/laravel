# Laravel Pennant

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
- [Định nghĩa chức năng](#defining-features)
    - [Chức năng dựa trên class](#class-based-features)
- [Kiểm tra chức năng](#checking-features)
    - [Điều kiện chạy](#conditional-execution)
    - [Trait `HasFeatures`](#the-has-features-trait)
    - [Blade Directive](#blade-directive)
    - [Middleware](#middleware)
    - [In-Memory Cache](#in-memory-cache)
- [Scope](#scope)
    - [Chỉ định Scope](#specifying-the-scope)
    - [Scope mặc định](#default-scope)
    - [Nullable Scope](#nullable-scope)
    - [Identifying Scope](#identifying-scope)
    - [Serializing Scope](#serializing-scope)
- [Rich Feature Values](#rich-feature-values)
- [Lấy Multiple Features](#retrieving-multiple-features)
- [Eager Loading](#eager-loading)
- [Updating Values](#updating-values)
    - [Bulk Updates](#bulk-updates)
    - [Purging Features](#purging-features)
- [Testing](#testing)
- [Thêm Custom Pennant Drivers](#adding-custom-pennant-drivers)
    - [Implementing the Driver](#implementing-the-driver)
    - [Đăng ký Driver](#registering-the-driver)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

[Laravel Pennant](https://github.com/laravel/pennant) là một package feature flag đơn giản và nhẹ - không có phần rườm rà. Feature flag cho phép bạn triển khai các chức năng mới của ứng dụng một cách tự tin, test A/B các thiết kế giao diện mới, bổ sung cho cách phát triển dựa trên trunk và nhiều chức năng khác.

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt Pennant vào dự án của bạn bằng trình quản lý package Composer:

```shell
composer require laravel/pennant
```

Tiếp theo, bạn nên export các file cấu hình và migration của Pennant bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Pennant\PennantServiceProvider"
```

Cuối cùng, bạn nên chạy migration cơ sở dữ liệu của ứng dụng. Thao tác này sẽ tạo bảng `features` mà Pennant sử dụng để cung cấp năng lực cho driver `database` của nó:

```shell
php artisan migrate
```

<a name="configuration"></a>
## Cấu hình

Sau khi export các asset của Pennant, file cấu hình của nó sẽ nằm tại `config/pennant.php`. File cấu hình này cho phép bạn chỉ định cơ chế lưu trữ mặc định mà sẽ được Pennant sử dụng để lưu các giá trị flag feature.

Pennant hiện tại hỗ trợ lưu các giá trị flag feature trong một mảng trong bộ nhớ memory thông qua driver `array`. Hoặc, Pennant cũng có thể lưu các giá trị flag feature này một trong cơ sở dữ liệu thông qua driver `database`, đây là cơ chế lưu trữ mặc định được Pennant sử dụng.

<a name="defining-features"></a>
## Định nghĩa chức năng

Để định nghĩa một chức năng mới, bạn có thể sử dụng phương thức `define` do facade `Feature` cung cấp. Bạn sẽ cần cung cấp tên chức năng, và một closure sẽ được gọi để resolve giá trị khởi tạo đầu tiên của chức năng.

Thông thường, các chức năng được định nghĩa trong một service provider bằng cách sử dụng facade `Feature`. Closure sẽ nhận vào "scope" để kiểm tra chức năng. Phổ biến nhất, scope là người dùng hiện tại đang được xác thực. Trong ví dụ này, chúng ta sẽ định nghĩa một chức năng để triển khai một API mới cho người dùng ứng dụng của chúng ta theo từng bước:

```php
<?php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Lottery;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::define('new-api', fn (User $user) => match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        });
    }
}
```

Như bạn có thể thấy, chúng ta có các quy tắc sau cho chức năng của chúng ta như sau:

- Tất cả các thành viên trong team nội bộ có thể sử dụng API mới.
- Bất kỳ khách hàng nào có lưu lượng truy cập cao đều không được sử dụng API mới.
- Nếu không, chức năng này sẽ được chỉ định ngẫu nhiên cho những người dùng có cơ hội kích hoạt là 1/100.

Lần đầu tiên chức năng `new-api` sẽ được kiểm tra cho một người dùng nhất định, kết quả của closure sẽ được lưu bởi driver storage. Lần tiếp theo chức năng được kiểm tra đối với cùng một người dùng, thì giá trị đó sẽ được lấy từ bộ nhớ và closure sẽ không được gọi nữa.

Để thuận tiện, nếu định nghĩa một chức năng mới của bạn chỉ trả về giá trị ngẫu nhiên, bạn có thể bỏ qua hoàn toàn phần closure:

    Feature::define('site-redesign', Lottery::odds(1, 1000));

<a name="class-based-features"></a>
### Chức năng dựa trên class

Pennant cũng cho phép bạn định nghĩa các chức năng dựa trên class. Không giống như các định nghĩa chức năng dựa trên closure, bạn không cần phải đăng ký một chức năng dựa trên class trong một service provider. Để tạo một chức năng dựa trên class, bạn có thể gọi lệnh Artisan `pennant:feature`. Mặc định, class chức năng mới sẽ được lưu trong thư mục `app/Features` của ứng dụng của bạn:

```shell
php artisan pennant:feature NewApi
```

Khi viết một class chức năng, bạn chỉ cần định nghĩa một phương thức `resolve`, phương thức này sẽ được gọi để resolve giá trị khởi tạo đầu tiên của chức năng cho một scope nhất định. Một lần nữa, scope thường sẽ là người dùng hiện đang được xác thực:

```php
<?php

namespace App\Features;

use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * Resolve the feature's initial value.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

> [!NOTE] Các class chức năng được resolve thông qua [container](/docs/{{version}}/container), do đó bạn có thể tích hợp thêm các phụ thuộc vào hàm constructor của class chức năng khi cần.

#### Customizing the Stored Feature Name

Mặc định, Pennant sẽ lưu trữ tên class của class chức năng. Nếu bạn muốn tách tên chức năng đã lưu ra khỏi cấu trúc bên trong của ứng dụng, bạn có thể chỉ định thuộc tính `$name` trên class chức năng. Giá trị của thuộc tính này sẽ được lưu thay cho tên class:

```php
<?php

namespace App\Features;

class NewApi
{
    /**
     * The stored name of the feature.
     *
     * @var string
     */
    public $name = 'new-api';

    // ...
}
```

<a name="checking-features"></a>
## Kiểm tra chức năng

Để xác định xem một chức năng có đang hoạt động hay không, bạn có thể sử dụng phương thức `active` trên facade `Feature`. Mặc định, các chức năng sẽ được kiểm tra trên các người dùng hiện tại đang được xác thực:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::active('new-api')
                ? $this->resolveNewApiResponse($request)
                : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

Mặc dù các chức năng sẽ được kiểm tra mặc định cho người dùng hiện tại đang được xác thực, bạn cũng có thể dễ dàng kiểm tra chức năng này cho những người dùng khác hoặc [scope](#scope). Để thực hiện việc này, hãy sử dụng phương thức `for` do facade `Feature` cung cấp:

```php
return Feature::for($user)->active('new-api')
        ? $this->resolveNewApiResponse($request)
        : $this->resolveLegacyApiResponse($request);
```

Pennant cũng cung cấp thêm một số phương thức tiện lợi có thể hiệu quả khi xác định một chức năng nào đó có đang hoạt động hay không:

```php
// Determine if all of the given features are active...
Feature::allAreActive(['new-api', 'site-redesign']);

// Determine if any of the given features are active...
Feature::someAreActive(['new-api', 'site-redesign']);

// Determine if a feature is inactive...
Feature::inactive('new-api');

// Determine if all of the given features are inactive...
Feature::allAreInactive(['new-api', 'site-redesign']);

// Determine if any of the given features are inactive...
Feature::someAreInactive(['new-api', 'site-redesign']);
```

> [!NOTE]
> Khi sử dụng Pennant bên ngoài HTTP, chẳng hạn như trong một lệnh Artisan hoặc một queued job, bạn nên [chỉ định phạm vi của chức năng](#specifying-the-scope). Ngoài ra, bạn nên định nghĩa [phạm vi mặc định](#default-scope) có cả HTTP đã xác thực và chưa xác thực.

<a name="checking-class-based-features"></a>
#### Checking Class Based Features

Đối với các chức năng dựa trên class, bạn nên cung cấp tên class khi kiểm tra chức năng:

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::active(NewApi::class)
                ? $this->resolveNewApiResponse($request)
                : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

<a name="conditional-execution"></a>
### Điều kiện chạy

Phương thức `when` có thể được sử dụng để thực hiện một closure nhất định nếu một chức năng đang hoạt động. Ngoài ra, một closure thứ hai cũng có thể được cung cấp và sẽ được thực hiện nếu chức năng này chưa hoạt động:

    <?php

    namespace App\Http\Controllers;

    use App\Features\NewApi;
    use Illuminate\Http\Request;
    use Illuminate\Http\Response;
    use Laravel\Pennant\Feature;

    class PodcastController
    {
        /**
         * Display a listing of the resource.
         */
        public function index(Request $request): Response
        {
            return Feature::when(NewApi::class,
                fn () => $this->resolveNewApiResponse($request),
                fn () => $this->resolveLegacyApiResponse($request),
            );
        }

        // ...
    }

Phương thức `unless` đóng vai trò là phương thức ngược lại của phương thức `when`, thực thi lệnh closure đầu tiên nếu chức năng không hoạt động:

    return Feature::unless(NewApi::class,
        fn () => $this->resolveLegacyApiResponse($request),
        fn () => $this->resolveNewApiResponse($request),
    );

<a name="the-has-features-trait"></a>
### Trait `HasFeatures`

Trait `HasFeatures` của Pennant có thể được thêm vào model `User` của ứng dụng (hoặc bất kỳ model nào khác mà có chức năng) để cung cấp một cách thuận tiện để kiểm tra các chức năng trực tiếp từ model:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Pennant\Concerns\HasFeatures;

class User extends Authenticatable
{
    use HasFeatures;

    // ...
}
```

Sau khi trait đã được thêm vào model của bạn, bạn có thể dễ dàng kiểm tra các chức năng bằng cách gọi phương thức `features`:

```php
if ($user->features()->active('new-api')) {
    // ...
}
```

Tất nhiên, phương thức `features` cũng cung cấp quyền truy cập vào nhiều phương thức khác để tương tác với các chức năng:

```php
// Values...
$value = $user->features()->value('purchase-button')
$values = $user->features()->values(['new-api', 'purchase-button']);

// State...
$user->features()->active('new-api');
$user->features()->allAreActive(['new-api', 'server-api']);
$user->features()->someAreActive(['new-api', 'server-api']);

$user->features()->inactive('new-api');
$user->features()->allAreInactive(['new-api', 'server-api']);
$user->features()->someAreInactive(['new-api', 'server-api']);

// Conditional execution...
$user->features()->when('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);

$user->features()->unless('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);
```

<a name="blade-directive"></a>
### Blade Directive

Để việc kiểm tra các chức năng trong Blade trở nên liền mạch, Pennant cung cấp lệnh `@feature`:

```blade
@feature('site-redesign')
    <!-- 'site-redesign' is active -->
@else
    <!-- 'site-redesign' is inactive -->
@endfeature
```

<a name="middleware"></a>
### Middleware

Pennant cũng chứa một [middleware](/docs/{{version}}/middleware) có thể được sử dụng để xác minh người dùng hiện đang được xác thực có quyền truy cập vào một chức năng trước khi route được gọi hay không. Bạn có thể chỉ định middleware này cho một route và chỉ định các chức năng cần thiết để truy cập route đó. Nếu bất kỳ chức năng nào được chỉ định không hoạt động đối với người dùng hiện đang được xác thực, thì một phản hồi HTTP `400 Bad Request` sẽ được route trả về. Nhiều chức năng có thể được truyền vào phương thức static `using`.

```php
use Illuminate\Support\Facades\Route;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

Route::get('/api/servers', function () {
    // ...
})->middleware(EnsureFeaturesAreActive::using('new-api', 'servers-api'));
```

<a name="customizing-the-response"></a>
#### Customizing the Response

Nếu bạn muốn tùy chỉnh phản hồi được trả về bởi middleware khi một trong các chức năng được liệt kê không hoạt động, bạn có thể sử dụng phương thức `whenInactive` do middleware `EnsureFeaturesAreActive` cung cấp. Thông thường, phương thức này phải được gọi trong phương thức `boot` của một trong những service provider của ứng dụng:

```php
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    EnsureFeaturesAreActive::whenInactive(
        function (Request $request, array $features) {
            return new Response(status: 403);
        }
    );

    // ...
}
```

<a name="in-memory-cache"></a>
### In-Memory Cache

Khi kiểm tra một chức năng, Pennant sẽ tạo một bộ nhớ cache cho kết quả. Nếu bạn đang sử dụng driver `database`, điều này có nghĩa là việc kiểm tra lại cùng một feature flag trong một request đơn sẽ không kích hoạt thêm các truy vấn vào cơ sở dữ liệu. Điều này cũng để đảm bảo các chức năng có kết quả nhất quán trong suốt thời gian của request.

Nếu bạn cần xóa cache trong bộ nhớ, bạn có thể sử dụng phương thức `flushCache` do facade `Feature` cung cấp:

    Feature::flushCache();

<a name="scope"></a>
## Scope

<a name="specifying-the-scope"></a>
### Chỉ định Scope

Như đã thảo luận, các chức năng thường được kiểm tra đối với người dùng hiện đang được xác thực. Tuy nhiên, điều này không phải lúc nào cũng phù hợp với nhu cầu của bạn. Do đó, bạn có thể chỉ định phạm vi mà bạn muốn kiểm tra cho một chức năng nhất định thông qua phương thức `for` của facade `Feature`:

```php
return Feature::for($user)->active('new-api')
        ? $this->resolveNewApiResponse($request)
        : $this->resolveLegacyApiResponse($request);
```

Tất nhiên, phạm vi của chức năng không bị giới hạn ở mỗi "người dùng". Hãy tưởng tượng bạn đang xây dựng một trải nghiệm thanh toán mới mà bạn đang dự định triển khai cho toàn bộ các team thay vì người dùng cá nhân. Có thể bạn muốn các team dùng lâu hơn sẽ được giới thiệu chậm hơn so các team mới. Closure chức năng của bạn có thể trông giống như sau:

```php
use App\Models\Team;
use Carbon\Carbon;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('billing-v2', function (Team $team) {
    if ($team->created_at->isAfter(new Carbon('1st Jan, 2023'))) {
        return true;
    }

    if ($team->created_at->isAfter(new Carbon('1st Jan, 2019'))) {
        return Lottery::odds(1 / 100);
    }

    return Lottery::odds(1 / 1000);
});
```

Bạn sẽ nhận thấy rằng closure mà chúng ta đã định nghĩa sẽ không chấp nhận đầu vào một `User`, mà thay vào đó là một model `Team`. Để xác định xem chức năng có hoạt động với một team hay không, bạn nên truyền team đến phương thức `for` được cung cấp bởi facade `Feature`:

```php
if (Feature::for($user->team)->active('billing-v2')) {
    return redirect()->to('/billing/v2');
}

// ...
```

<a name="default-scope"></a>
### Scope mặc định

Bạn cũng có thể tùy chỉnh phạm vi mặc định sẽ sử dụng để kiểm tra chức năng. Ví dụ: tất cả các chức năng của bạn đang được kiểm tra đối với một team người dùng hiện đang được xác thực thay vì người dùng. Thay vì phải gọi `Feature::for($user->team)` mỗi khi bạn kiểm tra một chức năng, thay vào đó, bạn có thể chỉ định team là phạm vi mặc định. Thông thường, điều này nên được thực hiện ở một trong các service provider của ứng dụng của bạn:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::resolveScopeUsing(fn ($driver) => Auth::user()?->team);

        // ...
    }
}
```

Nếu không có phạm vi nào được cung cấp rõ ràng thông qua phương thức `for`, kiểm tra chức năng hiện tại sẽ sử dụng team người dùng đang được xác thực làm phạm vi mặc định:

```php
Feature::active('billing-v2');

// Nó tương đương với ...

Feature::for($user->team)->active('billing-v2');
```

<a name="nullable-scope"></a>
### Nullable Scope

Nếu phạm vi bạn cung cấp khi kiểm tra chức năng là `null` và định nghĩa của chức năng không hỗ trợ `null` thông qua một dạng nullable hoặc thêm giá trị `null` trong một kiểu union, Pennant sẽ tự động trả về `false` làm giá trị kết quả của chức năng.

Vì vậy, nếu phạm vi mà bạn chuyển cho một chức năng có khả năng là `null` và bạn muốn gọi giá trị resolver của chức năng, bạn nên xem xét đến định nghĩa của chức năng. Phạm vi `null` có thể xảy ra nếu bạn đang kiểm tra một chức năng trong một lệnh Artisan, queued job hoặc route chưa có xác thực. Vì thông thường không có người dùng được xác thực trong các bối cảnh này, phạm vi mặc định sẽ là `null`.

Nếu bạn không thể [chỉ định rõ ràng  phạm vi chức năng của bạn](#specifying-the-scope) thì bạn nên đảm bảo là phạm vi của bạn chấp nhận "nullable" và xử lý giá trị `null` đó trong logic định nghĩa chức năng của bạn:

```php
use App\Models\User;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('new-api', fn (User $user) => match (true) {// [tl! remove]
Feature::define('new-api', fn (User|null $user) => match (true) {// [tl! add]
    $user === null => true,// [tl! add]
    $user->isInternalTeamMember() => true,
    $user->isHighTrafficCustomer() => false,
    default => Lottery::odds(1 / 100),
});
```

<a name="identifying-scope"></a>
### Identifying Scope

Driver lưu trữ `array` và `database` được tích hợp sẵn bên trong của Pennant biết cách lưu trữ chính xác định danh phạm vi cho tất cả các kiểu dữ liệu PHP cũng như các model Eloquent. Tuy nhiên, nếu ứng dụng của bạn sử dụng driver Pennant của bên thứ ba, thì driver đó có thể không biết cách lưu trữ định danh của model Eloquent hoặc các kiểu tùy chỉnh khác trong ứng dụng của bạn.

Trong trường hợp như vậy, Pennant cho phép bạn định dạng các giá trị phạm vi để lưu trữ bằng cách implement contract `FeatureScopeable` trên các đối tượng có trong ứng dụng của bạn được sử dụng làm phạm vi Pennant.

Ví dụ, hãy tưởng tượng bạn đang sử dụng hai driver chức năng khác nhau trong một ứng dụng duy nhất: driver `database` có sẵn và driver "Flag Rocket" của bên thứ ba. Driver "Flag Rocket" không biết cách lưu trữ model Eloquent đúng cách. Thay vào đó, nó yêu cầu một instance `FlagRocketUser`. Bằng cách implement `toFeatureIdentifier` được định nghĩa bởi contract `FeatureScopeable`, chúng ta có thể tùy chỉnh giá trị phạm vi lưu trữ được cung cấp cho mỗi driver được ứng dụng sử dụng:

```php
<?php

namespace App\Models;

use FlagRocket\FlagRocketUser;
use Illuminate\Database\Eloquent\Model;
use Laravel\Pennant\Contracts\FeatureScopeable;

class User extends Model implements FeatureScopeable
{
    /**
     * Cast the object to a feature scope identifier for the given driver.
     */
    public function toFeatureIdentifier(string $driver): mixed
    {
        return match($driver) {
            'database' => $this,
            'flag-rocket' => FlagRocketUser::fromId($this->flag_rocket_id),
        };
    }
}
```

<a name="serializing-scope"></a>
### Serializing Scope

Mặc định, Pennant sẽ sử dụng tên class khi lưu một chức năng được liên kết với một model Eloquent. Nếu bạn đã sử dụng [Eloquent morph map](/docs/{{version}}/eloquent-relationships#custom-polymorphic-types), bạn có thể hướng dẫn để Pennant cũng sử dụng morph map đó để tách chức năng đã lưu trữ ra khỏi cấu trúc ứng dụng của bạn.

Để đạt được điều này, sau khi định nghĩa Eloquent morph map của bạn trong một service provider, bạn có thể gọi phương thức `useMorphMap` của facade `Feature`:

```php
use Illuminate\Database\Eloquent\Relations\Relation;
use Laravel\Pennant\Feature;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);

Feature::useMorphMap();
```

<a name="rich-feature-values"></a>
## Rich Feature Values

Hiện tại, chúng ta mới chủ yếu thảo luận về cách hiển thị các chức năng ở trạng thái nhị phân, nghĩa là chúng "hoạt động" hoặc "không hoạt động", nhưng Pennant cũng cho phép bạn lưu trữ thêm các giá trị khác.

Ví dụ, hãy tưởng tượng bạn đang thử nghiệm ba màu mới cho nút "mua ngay" của ứng dụng. Thay vì trả về `true` hoặc `false` từ định nghĩa chức năng, bạn có thể trả về một chuỗi:

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn (User $user) => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

Bạn có thể lấy giá trị của chức năng `purchase-button` bằng phương thức `value`:

```php
$color = Feature::value('purchase-button');
```

Lệnh Blade có sẵn của Pennant cũng giúp bạn dễ dàng hiển thị nội dung dựa trên giá trị hiện tại của chức năng:

```blade
@feature('purchase-button', 'blue-sapphire')
    <!-- 'blue-sapphire' is active -->
@elsefeature('purchase-button', 'seafoam-green')
    <!-- 'seafoam-green' is active -->
@elsefeature('purchase-button', 'tart-orange')
    <!-- 'tart-orange' is active -->
@endfeature
```

> [!NOTE] Khi sử dụng các giá trị khác, điều quan trọng bạn phải biết là một chức năng sẽ được coi là "hoạt động" khi nó có giá trị nào khác, khác với giá trị `false`.

Khi gọi phương thức [điều kiện `when`](#conditional-execution), giá trị khác của chức năng sẽ được cung cấp cho hàm closure đầu tiên:

    Feature::when('purchase-button',
        fn ($color) => /* ... */,
        fn () => /* ... */,
    );

Tương tự như vậy, khi gọi phương thức điều kiện `unless`, giá trị khác của chức năng sẽ được cung cấp cho hàm closure tùy chọn thứ hai:

    Feature::unless('purchase-button',
        fn () => /* ... */,
        fn ($color) => /* ... */,
    );

<a name="retrieving-multiple-features"></a>
## Lấy Multiple Features

Phương thức `values` cho phép lấy ra nhiều chức năng cho một phạm vi nhất định:

```php
Feature::values(['billing-v2', 'purchase-button']);

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
// ]
```

Hoặc, bạn có thể sử dụng phương thức `all` để lấy ra giá trị của tất cả các chức năng đã được định nghĩa cho một phạm vi nhất định:

```php
Feature::all();

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

Tuy nhiên, các chức năng dựa trên class mà được đăng ký động và Pennant sẽ không biết cho đến khi chúng được kiểm tra. Điều này có nghĩa là các chức năng dựa trên class của ứng dụng có thể không có trong kết quả trả về bởi phương thức `all` nếu chúng chưa được kiểm tra trong request hiện tại.

Nếu bạn muốn đảm bảo các class chức năng luôn có khi sử dụng phương thức `all`, bạn có thể sử dụng hàm discovery của Pennant. Để bắt đầu, hãy gọi phương thức `discover` trong một service provider của ứng dụng:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Laravel\Pennant\Feature;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Feature::discover();

            // ...
        }
    }

Phương thức `discover` sẽ đăng ký tất cả các class chức năng có trong thư mục `app/Features` của ứng dụng. Phương thức `all` giờ đây sẽ thêm các class này vào trong kết quả, bất kể chúng đã được kiểm tra trong request hiện tại hay chưa:

```php
Feature::all();

// [
//     'App\Features\NewApi' => true,
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

<a name="eager-loading"></a>
## Eager Loading

Mặc dù Pennant sẽ lưu cache tất cả các chức năng đã được resolve cho một request duy nhất, nhưng bạn vẫn có thể gặp phải các vấn đề về hiệu suất. Để khắc phục điều này, Pennant có cung cấp một eager load cho các giá trị chức năng.

Để minh họa điều này, hãy tưởng tượng rằng chúng ta đang kiểm tra xem một chức năng có hoạt động trong một vòng lặp hay không:

```php
use Laravel\Pennant\Feature;

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

Giả sử chúng ta đang sử dụng driver cơ sở dữ liệu, đoạn code sau sẽ thực hiện một truy vấn vào cơ sở dữ liệu cho mỗi người dùng trong vòng lặp - và điều đó có thể thực hiện hàng trăm truy vấn. Tuy nhiên, bằng cách sử dụng phương thức `load` của Pennant, chúng ta có thể loại bỏ lỗi hiệu suất tiềm ẩn này bằng cách eager load các giá trị chức năng cho một tập hợp người dùng hoặc một tập hợp các phạm vi:

```php
Feature::for($users)->load(['notifications-beta']);

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

Để load các giá trị chức năng chỉ khi chúng chưa được load, bạn có thể sử dụng phương thức `loadMissing`:

```php
Feature::for($users)->loadMissing([
    'new-api',
    'purchase-button',
    'notifications-beta',
]);
```

<a name="updating-values"></a>
## Updating Values

Khi giá trị của một chức năng được resolve lần đầu tiên, driver sẽ lưu trữ kết quả vào cơ sở dữ liệu. Điều này thường cần thiết để đảm bảo trải nghiệm nhất quán của người dùng qua các request. Tuy nhiên, đôi khi, bạn có thể muốn cập nhật lại giá trị được lưu của chức năng.

Để thực hiện điều này, bạn có thể sử dụng phương thức `activate` và `deactivate` để bật hoặc tắt một chức năng:

```php
use Laravel\Pennant\Feature;

// Activate the feature for the default scope...
Feature::activate('new-api');

// Deactivate the feature for the given scope...
Feature::for($user->team)->deactivate('billing-v2');
```

Bạn cũng có thể thiết lập giá trị khác cho một chức năng bằng cách cung cấp tham số thứ hai cho phương thức `activate`:

```php
Feature::activate('purchase-button', 'seafoam-green');
```

Để hướng dẫn Pennant xoá giá trị đã lưu của một chức năng, bạn có thể sử dụng phương thức `forget`. Khi chức năng được kiểm tra lại, Pennant sẽ resolve giá trị của chức năng từ định nghĩa chức năng của nó:

```php
Feature::forget('purchase-button');
```

<a name="bulk-updates"></a>
### Bulk Updates

Để cập nhật hàng loạt các giá trị chức năng đã lưu trữ, bạn có thể sử dụng phương thức `activateForEveryone` và `deactivateForEveryone`.

Ví dụ, hãy tưởng tượng bạn đang tự tin vào tính ổn định của chức năng `new-api` và đã tìm ra màu `'purchase-button'` tốt nhất cho quá trình thanh toán của bạn - bạn có thể cập nhật giá trị chức năng cho tất cả người dùng:

```php
use Laravel\Pennant\Feature;

Feature::activateForEveryone('new-api');

Feature::activateForEveryone('purchase-button', 'seafoam-green');
```

Ngoài ra, bạn cũng có thể tắt chức năng này cho tất cả người dùng:

```php
Feature::deactivateForEveryone('new-api');
```

> [!NOTE] Thao tác này sẽ chỉ cập nhật các giá trị chức năng đã được resolve và được lưu trữ bởi driver lưu trữ của Pennant. Bạn cũng sẽ cần cập nhật định nghĩa chức năng trong ứng dụng của bạn.

<a name="purging-features"></a>
### Purging Features

Thỉnh thoảng, việc xóa hết một chức năng ra khỏi bộ lưu trữ có thể hữu ích. Điều này thường xảy ra nếu bạn xóa một chức năng nào đó ra khỏi ứng dụng hoặc điều chỉnh lại định nghĩa của chức năng và bạn muốn chạy lại cho tất cả người dùng.

Bạn có thể xóa tất cả các giá trị đã được lưu cho một chức năng bằng phương thức `purge`:

```php
// Purging a single feature...
Feature::purge('new-api');

// Purging multiple features...
Feature::purge(['new-api', 'purchase-button']);
```

Nếu bạn muốn xóa _tất cả_ các chức năng ra khỏi bộ lưu trữ, bạn có thể gọi phương thức `purge` và không cần bất kỳ thêm số nào:

```php
Feature::purge();
```

Vì việc xóa các chức năng có thể hữu ích như một phần của quy trình deploy ứng dụng, Pennant có chứa một lệnh Artisan `pennant:purge` sẽ xóa các chức năng được cung cấp ra khỏi bộ lưu trữ:

```sh
php artisan pennant:purge new-api

php artisan pennant:purge new-api purchase-button
```

Bạn cũng có thể xóa tất cả các chức năng _trừ_ những chức năng có trong danh sách chức năng nhất định. Ví dụ, hãy tưởng tượng bạn muốn xóa tất cả các chức năng nhưng vẫn giữ lại các chức năng "new-api" và "purchase-button" trong bộ lưu trữ. Để thực hiện điều này, bạn có thể truyền tên các chức năng đó vào tùy chọn `--except`:

```sh
php artisan pennant:purge --except=new-api --except=purchase-button
```

Để thuận tiện, lệnh `pennant:purge` cũng hỗ trợ flag `--except-registered`. Flag này cho biết tất cả các chức năng, ngoại trừ những chức năng đã được đăng ký trong một service provider, còn lại tất cả đều được xóa:

```sh
php artisan pennant:purge --except-registered
```

<a name="testing"></a>
## Testing

Khi test code tương tác với feature flag, cách dễ nhất để kiểm soát giá trị trả về của feature flag trong các bài test của bạn là chỉ cần định nghĩa lại chức năng. Ví dụ: hãy tưởng tượng bạn có chức năng sau được định nghĩa trong một service provider của ứng dụng:

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn () => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

Để sửa giá trị trả về của chức năng trong các bài test, bạn có thể định nghĩa lại chức năng ở đầu bài test. Bài test sau sẽ luôn được pass, ngay cả khi hàm `Arr::random()` vẫn còn trong service provider:

```php
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define('purchase-button', 'seafoam-green');

    $this->assertSame('seafoam-green', Feature::value('purchase-button'));
}
```

Có thể sử dụng cách tiếp cận tương tự cho các chức năng dựa trên class:

```php
use App\Features\NewApi;
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define(NewApi::class, true);

    $this->assertTrue(Feature::value(NewApi::class));
}
```

Nếu chức năng của bạn trả về một instance `Lottery`, thì sẽ có một số [helper testing hữu ích](/docs/{{version}}/helpers#testing-lotteries).

<a name="store-configuration"></a>
#### Store Configuration

Bạn có thể cấu hình bộ lưu trữ mà Pennant sẽ sử dụng trong quá trình testing bằng cách định nghĩa biến môi trường `PENNANT_STORE` trong file `phpunit.xml` của ứng dụng:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit colors="true">
    <!-- ... -->
    <php>
        <env name="PENNANT_STORE" value="array"/>
        <!-- ... -->
    </php>
</phpunit>
```

<a name="adding-custom-pennant-drivers"></a>
## Thêm Custom Pennant Drivers

<a name="implementing-the-driver"></a>
#### Implementing the Driver

Nếu không có driver lưu trữ nào hiện có của Pennant phù hợp với nhu cầu sử dụng ứng dụng của bạn, bạn có thể sẽ phải tự viết driver lưu trữ riêng cho bạn. Driver tùy chỉnh của bạn nên implement interface `Laravel\Pennant\Contracts\Driver`:

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;

class RedisFeatureDriver implements Driver
{
    public function define(string $feature, callable $resolver): void {}
    public function defined(): array {}
    public function getAll(array $features): array {}
    public function get(string $feature, mixed $scope): mixed {}
    public function set(string $feature, mixed $scope, mixed $value): void {}
    public function setForAllScopes(string $feature, mixed $value): void {}
    public function delete(string $feature, mixed $scope): void {}
    public function purge(array|null $features): void {}
}
```

Bây giờ, chúng ta chỉ cần implement từng phương thức này bằng kết nối Redis. Để biết thêm ví dụ về cách implement của từng phương thức này, hãy xem `Laravel\Pennant\Drivers\DatabaseDriver` trong [source code của Pennant](https://github.com/laravel/pennant/blob/1.x/src/Drivers/DatabaseDriver.php)

> [!NOTE]
> Laravel không cung cấp sẵn các thư mục chứa các extension. Bạn có thể tự do đặt chúng ở bất kỳ đâu mà bạn muốn. Trong ví dụ này, chúng ta đã tạo một thư mục `Extensions` để chứa driver `RedisFeatureDriver`.

<a name="registering-the-driver"></a>
#### Đăng ký Driver

Sau khi driver của bạn đã được implement, bạn đã sẵn sàng để đăng ký nó với Laravel. Để thêm driver vào Pennant, bạn có thể sử dụng phương thức `extend` được cung cấp bởi facade `Feature`. Bạn nên gọi phương thức `extend` từ phương thức `boot` của một trong các [service provider](/docs/{{version}}/providers) của ứng dụng:

```php
<?php

namespace App\Providers;

use App\Extensions\RedisFeatureDriver;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

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
        Feature::extend('redis', function (Application $app) {
            return new RedisFeatureDriver($app->make('redis'), $app->make('events'), []);
        });
    }
}
```

Sau khi driver đã được đăng ký, bạn có thể sử dụng driver `redis` trong file cấu hình `config/pennant.php` của ứng dụng:

    'stores' => [

        'redis' => [
            'driver' => 'redis',
            'connection' => null,
        ],

        // ...

    ],

<a name="events"></a>
## Events

Pennant có gửi nhiều event khác nhau mà có thể hữu ích cho bạn khi bạn theo dõi feature flag trong toàn bộ ứng dụng.

### `Laravel\Pennant\Events\RetrievingKnownFeature`

Event này được gửi đi lần đầu tiên khi một chức năng được kiểm tra trong một request cho một phạm vi cụ thể. Event này có thể hữu ích để tạo và theo dõi số liệu của feature flag khi đang được sử dụng trong toàn bộ ứng dụng của bạn.

### `Laravel\Pennant\Events\RetrievingUnknownFeature`

Event này được gửi đi lần đầu tiên khi một chức năng không xác định được kiểm tra trong một request cho một phạm vi cụ thể. Event này có thể hữu ích nếu bạn định xóa feature flag, nhưng có thể đã vô tình để lại một số tham chiếu khác liên quan đến nó trong ứng dụng.

Ví dụ, bạn có thể thấy hữu ích khi listen các event này và `report` hoặc đưa ra một exception khi nó xảy ra:

```php
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Event;
use Laravel\Pennant\Events\RetrievingUnknownFeature;

class EventServiceProvider extends ServiceProvider
{
    /**
     * Register any other events for your application.
     */
    public function boot(): void
    {
        Event::listen(function (RetrievingUnknownFeature $event) {
            report("Resolving unknown feature [{$event->feature}].");
        });
    }
}
```

### `Laravel\Pennant\Events\DynamicallyDefiningFeature`

Event này được gửi đi khi một chức năng dựa trên class được kiểm tra lần đầu tiên trong một request.
