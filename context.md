# Context

- [Giới thiệu](#introduction)
    - [Cách thức hoạt động](#how-it-works)
- [Ghi lại Context](#capturing-context)
    - [Stacks](#stacks)
- [Lấy Context](#retrieving-context)
    - [Xác định sự tồn tại của một item](#determining-item-existence)
- [Xoá Context](#removing-context)
- [Context ẩn](#hidden-context)
- [Events](#events)
    - [Dehydrating](#dehydrating)
    - [Hydrated](#hydrated)

<a name="introduction"></a>
## Giới thiệu

Chức năng "context" của Laravel cho phép bạn thu thập, truy xuất và chia sẻ thông tin xuyên suốt thông qua các request, job và các command đang được chạy trong ứng dụng. Thông tin thu thập này cũng có trong log do ứng dụng ghi lại, giúp bạn hiểu sâu hơn về lịch sử chạy ứng dụng xảy ra trước khi một mục log được ghi lại và cho phép bạn theo dõi luồng chạy trong toàn bộ hệ thống phân tán.

<a name="how-it-works"></a>
### Cách thức hoạt động

Cách tốt nhất để hiểu chức năng context của Laravel là bạn xem nó hoạt động như thế nào bằng cách sử dụng các chức năng ghi log có sẵn. Để bắt đầu, bạn có thể [thêm thông tin vào context](#capturing-context) bằng cách sử dụng facade `Context`. Trong ví dụ này, chúng ta sẽ sử dụng một [middleware](/docs/{{version}}/middleware) để thêm URL request và một ID theo dõi duy nhất vào trong context trên mỗi request đến:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AddContext
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        Context::add('url', $request->url());
        Context::add('trace_id', Str::uuid()->toString());

        return $next($request);
    }
}
```

Thông tin được thêm vào context cũng sẽ tự động được thêm vào dưới dạng metadata cho bất kỳ [log](/docs/{{version}}/logging) nào được ghi trong suốt quá trình request. Việc thêm context dưới dạng metadata cho phép phân biệt thông tin được truyền đến log với thông tin được chia sẻ qua `Context`. Ví dụ, hãy tưởng tượng chúng ta đang viết log sau:

```php
Log::info('User authenticated.', ['auth_id' => Auth::id()]);
```

Log được viết ra sẽ chứa `auth_id`, nhưng nó cũng sẽ chứa `url` và `trace_id` của context dưới dạng metadata:

```
User authenticated. {"auth_id":27} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

Thông tin được thêm vào context cũng sẵn sàng cho các job được gửi đến queue. Ví dụ, hãy tưởng tượng chúng ta đang gửi một job `ProcessPodcast` đến queue sau khi thêm một số thông tin vào context:

```php
// In our middleware...
Context::add('url', $request->url());
Context::add('trace_id', Str::uuid()->toString());

// In our controller...
ProcessPodcast::dispatch($podcast);
```

Khi job được gửi, bất kỳ thông tin nào hiện tại nào đang được lưu trữ trong context cũng sẽ được thu thập và chia sẻ với job. Thông tin thu thập được sau đó sẽ được convert trở lại vào context hiện tại trong khi job đang được chạy. Vì vậy, nếu phương thức xử lý job của chúng ta có ghi log thì thông tin context cũng sẽ được ghi:

```php
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Log::info('Processing podcast.', [
            'podcast_id' => $this->podcast->id,
        ]);

        // ...
    }
}
```

Kết quả log sẽ chứa các thông tin được thêm vào context trong request ban đầu gửi job:

```
Processing podcast. {"podcast_id":95} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

Mặc dù chúng ta sẽ tập trung vào các tính năng ghi log có sẵn của context Laravel, nhưng tài liệu sau đây sẽ minh họa cách context cho phép bạn chia sẻ thông tin qua HTTP request và queued job và thậm chí cách thêm [dữ liệu ẩn context](#hidden-context) sẽ không được ghi vào log.

<a name="capturing-context"></a>
## Ghi lại Context

Bạn có thể lưu thông tin trong context hiện tại bằng cách sử dụng phương thức `add` của facade `Context`:

```php
use Illuminate\Support\Facades\Context;

Context::add('key', 'value');
```

Để thêm nhiều item cùng lúc, bạn có thể truyền một mảng vào phương thức `add`:

```php
Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);
```

Phương thức `add` sẽ ghi đè bất kỳ giá trị hiện đang có nếu trùng khóa. Nếu bạn chỉ muốn thêm thông tin vào context nếu khóa chưa tồn tại, thì bạn có thể sử dụng phương thức `addIf`:

```php
Context::add('key', 'first');

Context::get('key');
// "first"

Context::addIf('key', 'second');

Context::get('key');
// "first"
```

<a name="conditional-context"></a>
#### Conditional Context

Phương thức `when` có thể được sử dụng để thêm dữ liệu vào context dựa trên một điều kiện nhất định. Closure đầu tiên được cung cấp cho phương thức `when` sẽ được gọi nếu kết quả của điều kiện trả về là `true`, thì khi đó closure thứ hai sẽ được gọi nếu điều kiện trả về là `false`:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Context;

Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

<a name="stacks"></a>
### Stacks

Context cũng cung cấp khả năng tạo "ngăn xếp", là danh sách dữ liệu được lưu trữ theo thứ tự chúng được thêm vào. Bạn có thể thêm thông tin vào ngăn xếp bằng cách gọi phương thức `push`:

```php
use Illuminate\Support\Facades\Context;

Context::push('breadcrumbs', 'first_value');

Context::push('breadcrumbs', 'second_value', 'third_value');

Context::get('breadcrumbs');
// [
//     'first_value',
//     'second_value',
//     'third_value',
// ]
```

Ngăn xếp có thể hữu ích khi ghi lại các thông tin lịch sử về một request, chẳng hạn như các event đang diễn ra trong toàn bộ ứng dụng của bạn. Ví dụ: bạn có thể tạo ra một event listener để đưa vào ngăn xếp mỗi khi một câu truy vấn được thực thi và ghi lại câu lệnh SQL đó và thời gian chạy:

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\DB;

DB::listen(function ($event) {
    Context::push('queries', [$event->time, $event->sql]);
});
```

Bạn có thể xác định xem một giá trị có nằm trong ngăn xếp hay không bằng cách sử dụng các phương thức `stackContains` và `hiddenStackContains`:

```php
if (Context::stackContains('breadcrumbs', 'first_value')) {
    //
}

if (Context::hiddenStackContains('secrets', 'first_value')) {
    //
}
```

Các phương thức `stackContains` và `hiddenStackContains` cũng chấp nhận một closure làm tham số thứ hai, cho phép kiểm soát nhiều hơn đối với hoạt động so sánh giá trị:

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;

return Context::stackContains('breadcrumbs', function ($value) {
    return Str::startsWith($value, 'query_');
});
```

<a name="retrieving-context"></a>
## Lấy Context

Bạn có thể lấy thông tin từ context bằng cách sử dụng phương thức `get` của facade `Context`:

```php
use Illuminate\Support\Facades\Context;

$value = Context::get('key');
```

Phương thức `only` có thể được sử dụng để lấy ra một tập hợp con của thông tin có trong context:

```php
$data = Context::only(['first_key', 'second_key']);
```

Phương thức `pull` có thể được sử dụng để lấy ra thông tin từ context và xóa nó ra khỏi context:

```php
$value = Context::pull('key');
```

Nếu dữ liệu context được lưu trong một [ngăn xếp](#stacks), thì bạn có thể lấy các mục ra khỏi ngăn xếp luôn bằng phương thức `pop`:

```php
Context::push('breadcrumbs', 'first_value', 'second_value');

Context::pop('breadcrumbs')
// second_value

Context::get('breadcrumbs');
// ['first_value']
```

Nếu bạn muốn lấy ra toàn bộ thông tin được lưu trong context, bạn có thể gọi phương thức `all`:

```php
$data = Context::all();
```

<a name="determining-item-existence"></a>
### Xác định sự tồn tại của một item

Bạn có thể sử dụng phương thức `has` và `missing` để xác định xem context có lưu giá trị nào cho khóa đã cho hay không:

```php
use Illuminate\Support\Facades\Context;

if (Context::has('key')) {
    // ...
}

if (Context::missing('key')) {
    // ...
}
```

Phương thức `has` sẽ trả về `true` bất kể giá trị được lưu là bao nhiêu. Vì vậy, ví dụ, một khóa mà có giá trị là `null` thì cũng sẽ được coi là có:

```php
Context::add('key', null);

Context::has('key');
// true
```

<a name="removing-context"></a>
## Xoá Context

Phương thức `forget` có thể được sử dụng để xóa một khóa và giá trị của nó ra khỏi context hiện tại:

```php
use Illuminate\Support\Facades\Context;

Context::add(['first_key' => 1, 'second_key' => 2]);

Context::forget('first_key');

Context::all();

// ['second_key' => 2]
```

Bạn có thể xoá nhiều khóa cùng một lúc bằng cách cung cấp một mảng cho phương thức `forget`:

```php
Context::forget(['first_key', 'second_key']);
```

<a name="hidden-context"></a>
## Context ẩn

Context cũng cung cấp khả năng lưu trữ dữ liệu "bị ẩn". Thông tin ẩn này không được thêm vào log và không thể truy cập thông qua các phương thức truy xuất dữ liệu được nêu ở trên. Context cung cấp một tập hợp các phương thức khác nhau để tương tác với những thông tin context ẩn này:

```php
use Illuminate\Support\Facades\Context;

Context::addHidden('key', 'value');

Context::getHidden('key');
// 'value'

Context::get('key');
// null
```

Các phương thức "ẩn" tương ứng với chức năng của các phương thức không ẩn đã được ghi lại ở trên:

```php
Context::addHidden(/* ... */);
Context::addHiddenIf(/* ... */);
Context::pushHidden(/* ... */);
Context::getHidden(/* ... */);
Context::pullHidden(/* ... */);
Context::popHidden(/* ... */);
Context::onlyHidden(/* ... */);
Context::allHidden(/* ... */);
Context::hasHidden(/* ... */);
Context::forgetHidden(/* ... */);
```

<a name="events"></a>
## Events

Context sẽ gửi hai event cho phép bạn tham gia vào quá trình convert và convert ngược lại của context.

Để minh họa cách sử dụng các event này, hãy tưởng tượng là trong một middleware của ứng dụng, bạn set giá trị cấu hình `app.locale` dựa trên header `Accept-Language` của request HTTP đến. Các event của Context cho phép bạn nắm bắt giá trị này trong quá trình request và khôi phục nó trong queue, để đảm bảo các thông báo được gửi vào trong queue có giá trị `app.locale` chính xác. Chúng ta có thể sử dụng các event của Context và dữ liệu [ẩn](#hidden-context) của nó để làm điều này, tài liệu sau đây sẽ minh họa.

<a name="dehydrating"></a>
### Dehydrating

Bất cứ khi nào một job được gửi đến queue, dữ liệu trong context cũng sẽ được "chuyển đổi" và được ghi lại cùng với payload của job đó. Phương thức `Context::dehydrating` cho phép bạn đăng ký một closure sẽ được gọi trong khi quá trình chuyển đổi đang được diễn ra. Trong closure này, bạn có thể thực hiện các thay đổi đối với dữ liệu sẽ được chia sẻ với job trong queue.

Thông thường, bạn nên đăng ký lệnh callback `dehydrating` trong phương thức `boot` của class `AppServiceProvider` trong ứng dụng của bạn:

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::dehydrating(function (Repository $context) {
        $context->addHidden('locale', Config::get('app.locale'));
    });
}
```

> [!NOTE]
> Bạn không nên sử dụng facade `Context` trong callback `dehydrating`, vì điều đó sẽ thay đổi context của process hiện tại. Hãy đảm bảo bạn chỉ thực hiện các thay đổi đối với repository được truyền đến trong callback.

<a name="hydrated"></a>
### Hydrated

Bất cứ khi nào một queued job bắt đầu thực thi trên queue, thì bất kỳ context nào mà được chia sẻ với job đó cũng sẽ được "chuyển ngược lại" context hiện tại. Phương thức `Context::hydrated` cho phép bạn đăng ký một closure sẽ được gọi trong quá trình chuyển ngược lại đó.

Thông thường, bạn nên đăng ký lệnh callback `hydrated` trong phương thức `boot` của class `AppServiceProvider` trong ứng dụng của bạn:

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::hydrated(function (Repository $context) {
        if ($context->hasHidden('locale')) {
            Config::set('app.locale', $context->getHidden('locale'));
        }
    });
}
```

> [!NOTE]
> Bạn không nên sử dụng facade `Context` trong callback `hydrated` và thay vào đó hãy đảm bảo bạn chỉ thực hiện thay đổi đối với repository được truyền đến trong callback.
