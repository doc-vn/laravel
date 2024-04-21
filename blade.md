# Blade Templates

- [Giới thiệu](#introduction)
    - [Supercharging Blade cùng với Livewire](#supercharging-blade-with-livewire)
- [Hiển thị dữ liệu](#displaying-data)
    - [HTML Entity Encoding](#html-entity-encoding)
    - [Blade và JavaScript Frameworks](#blade-and-javascript-frameworks)
- [Blade Directives](#blade-directives)
    - [Lệnh if](#if-statements)
    - [Lệnh switch](#switch-statements)
    - [Loops](#loops)
    - [Biến loop](#the-loop-variable)
    - [Class điều kiện](#conditional-classes)
    - [Các thuộc tính bổ sung](#additional-attributes)
    - [Thêm Subviews](#including-subviews)
    - [Lệnh `@once`](#the-once-directive)
    - [Raw PHP](#raw-php)
    - [Comments](#comments)
- [Components](#components)
    - [Hiển thị Component](#rendering-components)
    - [Truyền dữ liệu tới Component](#passing-data-to-components)
    - [Thuộc tính Component](#component-attributes)
    - [Reserved Keywords](#reserved-keywords)
    - [Slots](#slots)
    - [Inline Component Views](#inline-component-views)
- [Component động](#dynamic-components)
- [Quản lý Component](#manually-registering-components)
- [Component ẩn](#anonymous-components)
    - [Anonymous Index Components](#anonymous-index-components)
    - [Data Properties / Attributes](#data-properties-attributes)
    - [Accessing Parent Data](#accessing-parent-data)
    - [Anonymous Component Paths](#anonymous-component-paths)
- [Building Layouts](#building-layouts)
    - [Layouts dùng Components](#layouts-using-components)
    - [Layouts dùng Template kế thừa](#layouts-using-template-inheritance)
- [Forms](#forms)
    - [CSRF Field](#csrf-field)
    - [Method Field](#method-field)
    - [Validation Errors](#validation-errors)
- [Stacks](#stacks)
- [Service Injection](#service-injection)
- [Tạo Inline Blade Templates](#rendering-inline-blade-templates)
- [Tạo Blade Fragments](#rendering-blade-fragments)
- [Mở rộng blade](#extending-blade)
    - [Tuỳ chỉnh xử lý hiển thị](#custom-echo-handlers)
    - [Tuỳ biến lệnh if](#custom-if-statements)

<a name="introduction"></a>
## Giới thiệu

Blade là một công cụ tạo template đơn giản nhưng mạnh mẽ được đi kèm cùng với Laravel. Không giống như một số công cụ tạo template khác của PHP, Blade không hạn chế bạn sử dụng code PHP trong template của bạn. Trên thực tế, tất cả các template Blade được biên dịch thành code PHP và được lưu vào trong cache, cho đến khi chúng được sửa đổi, có nghĩa là về cơ bản Blade không làm tăng chi phí chung cho application của bạn. Các file template Blade sử dụng phần đuôi mở rộng là `.blade.php` và thường được lưu trữ trong thư mục `resources/views`.

Blade view có thể được trả về từ các route hoặc controller bằng cách sử dụng helper `view` global. Tất nhiên, như đã đề cập trong tài liệu về [views](/docs/{{version}}/views), dữ liệu có thể được truyền cho Blade view bằng tham số thứ hai của helper `view`:

    Route::get('/', function () {
        return view('greeting', ['name' => 'Finn']);
    });

<a name="supercharging-blade-with-livewire"></a>
### Supercharging Blade cùng với Livewire

Bạn muốn đưa các template Blade của bạn lên một tầm cao mới và dễ dàng xây dựng các giao diện động? Hãy xem [Laravel Livewire](https://laravel-livewire.com). Livewire cho phép bạn viết các component Blade được tăng cường chức năng động mà thông thường chỉ có thể thực hiện được thông qua các framework front-end như React hoặc Vue, cung cấp một phương án tuyệt vời để xây dựng các giao diện front-end hiện đại, độ tương tác cao mà không cần đến sự phức tạp, render phía client hoặc xây dựng các bước của nhiều framework JavaScript.

<a name="displaying-data"></a>
## Hiển thị dữ liệu

Bạn có thể hiển thị dữ liệu mà đã được truyền đến Blade view của bạn bằng cách đặt tên biến vào trong hai lần dấu ngoặc nhọn. Ví dụ: một route như sau:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

Bạn có thể hiển thị nội dung của biến `name` như thế này:

```blade
Hello, {{ $name }}.
```

> **Note**
> Các câu lệnh echo `{{ }}` này của Blade sẽ được tự động gửi qua hàm `htmlspecialchars` của PHP để chặn các cuộc tấn công XSS.

Bạn không bị giới hạn trong việc hiển thị nội dung của các biến đã được truyền đến view. Bạn cũng có thể echo ra kết quả với bất kỳ hàm PHP nào tương tự. Thực tế, bạn có thể set bất kỳ code PHP nào mà bạn muốn vào trong lệnh echo của Blade:

```blade
The current UNIX timestamp is {{ time() }}.
```

<a name="html-entity-encoding"></a>
### HTML Entity Encoding

Mặc định, Blade (cũng như helper `e` của Laravel) sẽ mã hóa kép các thực thể HTML. Nếu bạn không muốn mã hóa kép này, hãy gọi phương thức `Blade::withoutDoubleEncoding` từ phương thức `boot` của `AppServiceProvider` của bạn:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::withoutDoubleEncoding();
        }
    }

<a name="displaying-unescaped-data"></a>
#### Hiển thị dữ liệu unescaped

Mặc định, các câu lệnh Blade `{{ }}` này sẽ được tự động gửi qua hàm `htmlspecialchars` của PHP để ngăn chặn các cuộc tấn công XSS. Nếu bạn không muốn dữ liệu của bạn được escaped, bạn có thể sử dụng cú pháp sau:

```blade
Hello, {!! $name !!}.
```

> **Warning**
> Bạn hãy cẩn thận khi hiển thị một nội dung mà được cung cấp bởi người dùng. Bạn hãy luôn sử dụng escaped với cú pháp hai lần dấu ngoặc nhọn để ngăn chặn các cuộc tấn công XSS khi hiển thị dữ liệu do người dùng cung cấp.

<a name="blade-and-javascript-frameworks"></a>
### Blade và JavaScript Frameworks

Do nhiều framework JavaScript cũng sử dụng hai lần dấu ngoặc nhọn để hiển thị dữ liệu trong trình duyệt, nên bạn có thể sử dụng ký hiệu `@` để thông báo cho Blade rendering engine là biểu thức này sẽ không được laravel rendering. Ví dụ:

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

Trong ví dụ này, ký hiệu `@` sẽ bị xóa bởi Blade; tuy nhiên, biểu thức `{{ name }}` sẽ vẫn còn và sẽ không được xử lý bởi Blade engine, điều này cho phép các biểu thức đó sẽ được render bởi framework JavaScript của bạn.

Các symbol `@` cũng có thể được sử dụng cho các lệnh Blade:

```blade
{{-- Blade template --}}
@@if()

<!-- HTML output -->
@if()
```

<a name="rendering-json"></a>
#### Rendering JSON

Đôi khi, bạn có thể truyền một mảng vào view của bạn với ý định hiển thị mảng đó dưới dạng JSON để khởi tạo một biến bên JavaScript. Ví dụ:

```blade
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

Tuy nhiên, thay vì gọi `json_encode` theo cách thủ công, bạn có thể sử dụng phương thức `Illuminate\Support\Js::from`. Phương thức `from` chấp nhận các tham số giống như hàm `json_encode` của PHP; tuy nhiên, nó sẽ đảm bảo rằng JSON kết quả được escaped đúng cách khi đưa vào trong các quote HTML. Phương thức `from` sẽ trả về một chuỗi `JSON.parse`, câu lệnh JavaScript sẽ chuyển đổi đối tượng hoặc mảng đã cho thành một đối tượng JavaScript hợp lệ:

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

Các phiên bản mới nhất của Laravel framework có chứa một facade `Js`, cung cấp quyền truy cập thuận tiện vào chức năng này trong các template Blade của bạn:

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

> **Warning**
> Bạn chỉ nên sử dụng phương thức `Js::from` để tạo ra các biến hiện có dưới dạng JSON. Việc tạo thêm các template Blade dựa trên các biểu thức thông thường và việc cố gắng truyền một biểu thức phức tạp tới câu lệnh này có thể gây ra các lỗi không mong muốn.

<a name="the-at-verbatim-directive"></a>
#### Lệnh `@verbatim`

Nếu trong template của bạn đang hiển thị nhiều biến JavaScript, bạn có thể bao bọc các lệnh đó trong lệnh `@verbatim` để bạn không phải đặt tiền tố cho mỗi câu lệnh echo Blade bằng ký hiệu `@`:

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

<a name="blade-directives"></a>
## Blade Directives

Ngoài việc kế thừa template và hiển thị dữ liệu, Blade cũng cung cấp các shortcut cho các cấu trúc điều khiển PHP phổ biến, chẳng hạn như các câu lệnh và các vòng lặp có điều kiện. Các shortcut này cung cấp một cách làm việc rất ngắn gọn, gọn gàng với các cấu trúc điều khiển PHP trong khi vẫn quen thuộc với các hàm tương tự trong PHP.

<a name="if-statements"></a>
### Lệnh if

Bạn có thể xây dựng các câu lệnh `if` bằng cách sử dụng các lệnh `@if`, `@elseif`, `@else`, và `@endif`. Các lệnh này hoạt động giống hệt với các hàm tương tự trong PHP:

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

Để thuận tiện, Blade cũng cung cấp một lệnh `@unless`:

```blade
@unless (Auth::check())
    You are not signed in.
@endunless
```

Ngoài các lệnh có điều kiện đã được thảo luận ở trên, các lệnh `@isset` và `@empty` cũng có thể được sử dụng làm các shortcut cho các hàm PHP tương ứng của chúng:

```blade
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

<a name="authentication-directives"></a>
#### Lệnh authentication

Các lệnh `@auth` và `@guest` có thể được sử dụng để xác định xem người dùng hiện tại đã được [xác thực](/docs/{{version}}/authentication) chưa hay là khách:

```blade
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

Nếu cần, bạn có thể chỉ định thêm authentication guard cần được kiểm tra khi sử dụng các lệnh `@auth` và `@guest`:

```blade
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

<a name="environment-directives"></a>
#### Environment Directives

Bạn có thể kiểm tra xem ứng dụng có đang chạy trong môi trường production hay không bằng cách sử dụng lệnh `@production`:

```blade
@production
    // Production specific content...
@endproduction
```

Hoặc, bạn có thể xác định xem ứng dụng có đang chạy trong một môi trường cụ thể nào hay không bằng cách sử dụng lệnh `@env`:

```blade
@env('staging')
    // The application is running in "staging"...
@endenv

@env(['staging', 'production'])
    // The application is running in "staging" or "production"...
@endenv
```

<a name="section-directives"></a>
#### Lệnh section

Bạn có thể kiểm tra xem một section thừa kế có nội dung hay không bằng cách sử dụng lệnh `@hasSection`:

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

Bạn có thể sử dụng lệnh `sectionMissing` để xác định một section không có nội dung:

```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

<a name="switch-statements"></a>
### Lệnh switch

Các câu lệnh switch có thể được xây dựng bằng cách sử dụng các lệnh `@switch`, `@case`, `@break`, `@default` và `@endswitch`:

```blade
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

<a name="loops"></a>
### Loops

Ngoài các câu lệnh có điều kiện, Blade cung cấp các lệnh đơn giản để làm việc với các cấu trúc vòng lặp của PHP. Một lần nữa, các lệnh này hoạt động giống hệt với các hàm PHP tương ứng của chúng:

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

> **Note**
> Khi lặp vòng lặp `foreach`, bạn có thể sử dụng [biến loop](#the-loop-variable) để nhận về các thông tin có giá trị về vòng lặp, chẳng hạn như bạn đang ở vòng lặp đầu tiên hoặc vòng lặp cuối cùng.

Khi sử dụng các vòng lặp, bạn cũng có thể bỏ qua vòng lặp hiện tại hoặc kết thúc vòng lặp bằng cách sử dụng các lệnh `@continue` và `@break`:

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

Bạn cũng có thể thêm các điều kiện continue hoặc break trong định nghĩa của lệnh đó:

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

<a name="the-loop-variable"></a>
### Biến loop

Khi lặp vòng lặp `foreach`, một biến `$loop` sẽ có sẵn bên trong vòng lặp của bạn. Biến này cung cấp quyền truy cập vào một số thông tin hữu ích như vòng lặp hiện tại và liệu đây có phải là vòng lặp đầu tiên hay là vòng lặp cuối cùng:

```blade
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

Nếu bạn đang ở trong một vòng lặp lồng nhau, bạn có thể truy cập vào biến `$loop` của vòng lặp cha thông qua thuộc tính `parent`:

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

Biến `$loop` cũng chứa nhiều thuộc tính hữu ích khác:

| Property           | Description                                                                  |
|--------------------|------------------------------------------------------------------------------|
`$loop->index`       |  Index của vòng lặp hiện tại (bắt đầu từ 0).                                 |
`$loop->iteration`   |  Vòng lặp hiện tại (bắt đầu từ 1).                                           |
`$loop->remaining`   |  Các lần lặp còn lại trong vòng lặp.                                         |
`$loop->count`       |  Tổng số item trong mảng đang được lặp lại.                                  |
`$loop->first`       |  Đây có phải là lần lặp đầu tiên của vòng lặp hay không.                     |
`$loop->last`        |  Đây có phải là lần lặp cuối cùng của vòng lặp hay không.                    |
`$loop->even`        |  Đây có phải là lần lặp chẵn của vòng lặp hay không.                         |
`$loop->odd`         |  Đây có phải là lần lặp lẻ của vòng lặp hay không.                           |
`$loop->depth`       |  Mức lồng của vòng lặp hiện tại.                                             |
`$loop->parent`      |  Khi ở trong một vòng lặp lồng nhau, biến này là biến của vòng lặp ngoài.    |

<a name="conditional-classes"></a>
### Class điều kiện và styles

Lệnh `@class` sẽ biên dịch có điều kiện một string class CSS. Lệnh chấp nhận một mảng các class trong đó khóa mảng chứa class hoặc các class bạn muốn thêm vào, trong khi giá trị là một biểu thức trả về boolean. Nếu phần tử trong mảng đó có một key là số, thì nó sẽ luôn được đưa vào danh sách class được hiển thị:

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

Tương tự như vậy, lệnh `@style` có thể được sử dụng để thêm có điều kiện các inline CSS style vào một phần tử HTML:

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

<a name="additional-attributes"></a>
### Các thuộc tính bổ sung

Để thuận tiện, bạn có thể sử dụng lệnh `@checked` để dễ dàng chỉ ra nếu một input checkbox HTML nhất định được "checked". Lệnh này sẽ `checked` nếu điều kiện được cung cấp là `true`:

```blade
<input type="checkbox"
        name="active"
        value="active"
        @checked(old('active', $user->active)) />
```

Tương tự như vậy, lệnh `@selected` có thể được sử dụng để chỉ ra liệu một select nhất định có nên được "chọn" hay không:

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

Ngoài ra, lệnh `@disabled` có thể được sử dụng để chỉ ra liệu một phần tử nhất định có nên bị "disabled" hay không:

```blade
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

Hơn nữa, lệnh `@readonly` có thể được sử dụng để chỉ ra liệu một phần tử nhất định có ở trạng thái "readonly" hay không:

```blade
<input type="email"
        name="email"
        value="email@laravel.com"
        @readonly($user->isNotAdmin()) />
```

Ngoài ra, lệnh `@required` có thể được sử dụng để chỉ ra liệu một phần tử nhất định có cần "required" hay không:

```blade
<input type="text"
        name="title"
        value="title"
        @required($user->isAdmin()) />
```

<a name="including-subviews"></a>
### Thêm Subviews

> **Note**
> Mặc dù bạn được tự do sử dụng lệnh `@include`, Blade [components](#components) cung cấp chức năng tương tự và mang lại một số lợi ích hơn so với lệnh `@include`, chẳng hạn như liên kết dữ liệu và thuộc tính.

Lệnh `@include` của Blade cho phép bạn thêm một Blade view vào bên trong một view xem khác. Tất cả các biến có sẵn của view chính sẽ được cung cấp cho view được thêm vào:

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Mặc dù view được thêm vào sẽ kế thừa tất cả dữ liệu có sẵn trong view chính, nhưng bạn cũng có thể truyền thêm một mảng dữ liệu bổ sung sẽ được cung cấp cho view được thêm vào:

```blade
@include('view.name', ['status' => 'complete'])
```

Nếu bạn cố gắng `@include` thêm một view không tồn tại, Laravel sẽ báo lỗi. Nếu bạn muốn thêm một view có thể có hoặc không, bạn nên sử dụng lệnh `@includeIf`:

```blade
@includeIf('view.name', ['status' => 'complete'])
```

Nếu bạn muốn `@include` một view nếu một biểu thức boolean đã cho trả về là `true` hoặc `false`, thì bạn có thể sử dụng các lệnh `@includeWhen` và `@includeUnless`:

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

Để thêm view đầu tiên tồn tại từ một mảng view nhất định, bạn có thể sử dụng lệnh `includeFirst`:

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

> **Warning**
> Bạn nên tránh sử dụng các hằng số `__DIR__` và `__FILE__` trong view Blade của bạn, vì chúng sẽ đề cập đến vị trí của view khi được biên dịch và được lưu trong bộ nhớ cache.

<a name="rendering-views-for-collections"></a>
#### Rendering Views For Collections

Bạn có thể kết hợp các vòng lặp và lệnh thêm view thành một dòng với lệnh `@each` của Blade:

```blade
@each('view.name', $jobs, 'job')
```

Tham số đầu tiên của lệnh `@each` là view để hiển thị cho từng phần tử trong mảng hoặc collection. Tham số thứ hai là mảng hoặc collection mà bạn muốn lặp, trong khi tham số thứ ba là tên biến sẽ được gán cho lần lặp hiện tại trong view. Vì vậy, ví dụ: nếu bạn đang lặp một mảng `jobs`, thông thường bạn sẽ muốn truy cập vào từng công việc dưới dạng một biến `job` trong view. Khóa mảng cho lần lặp hiện tại sẽ có sẵn dưới dạng biến `key` trong view.

Bạn cũng có thể truyền đối số thứ tư cho lệnh `@each`. Đối số này định nghĩa view nào sẽ được hiển thị nếu mảng đã cho là trống.

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

> **Warning**
> View được hiển thị qua `@each` không kế thừa các biến từ view gốc. Nếu view con yêu cầu các biến này, bạn nên sử dụng lệnh `@foreach` và `@include` để thay thế.

<a name="the-once-directive"></a>
### Lệnh `@once`

Lệnh `@once` cho phép bạn định nghĩa một phần của template sẽ chỉ được hiển thị một lần trong mỗi chu kỳ tạo. Điều này có thể hữu ích để đưa một đoạn JavaScript nhất định vào tiêu đề của trang bằng cách sử dụng [stacks](#stacks). Ví dụ: nếu bạn đang hiển thị một [component](#components) đã cho trong một vòng lặp, bạn có thể chỉ muốn đưa JavaScript tới tiêu đề trong lần đầu tiên khi component được hiển thị:

```blade
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce
```

Vì lệnh `@once` thường được sử dụng kết hợp với lệnh `@push` hoặc `@prepend`, nên lệnh `@pushOnce` và `@prependOnce` có thể giúp bạn sử dụng thuận tiện hơn:

```blade
@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

<a name="raw-php"></a>
### Raw PHP

Trong một số trường hợp, sẽ hữu ích khi nhúng mã PHP vào view của bạn. Bạn có thể sử dụng lệnh `@php` của Blade để thực thi một đoạn code PHP đơn giản trong template của bạn:

```blade
@php
    $counter = 1;
@endphp
```

Nếu bạn chỉ cần viết một câu lệnh PHP duy nhất, bạn có thể đưa câu lệnh đó vào trong lệnh `@php`:

```blade
@php($counter = 1)
```

<a name="comments"></a>
### Comments

Blade cũng cho phép bạn định nghĩa một comment trong view của bạn. Tuy nhiên, không giống như các comment trong HTML, các comment của Blade không được thêm vào trong HTML mà ứng dụng của bạn trả về:

```blade
{{-- This comment will not be present in the rendered HTML --}}
```

<a name="components"></a>
## Components

Các component và slot sẽ cung cấp các lợi ích tương tự cho các section, layouts, và includes; tuy nhiên, một số có thể thấy model của các component và slot sẽ dễ hiểu hơn. Có hai cách tiếp cận để viết các component: các component dựa trên class và các component ẩn.

Để tạo một component dựa trên class, bạn có thể sử dụng lệnh Artisan `make:component`. Để minh họa cách sử dụng của các component này, chúng ta sẽ tạo một component `Alert` đơn giản. Lệnh `make:component` sẽ lưu component vào trong thư mục `app/View/Components`:

```shell
php artisan make:component Alert
```

Lệnh `make:component` cũng sẽ tạo một view template cho component. View sẽ được đặt trong thư mục `resources/views/components`. Khi viết các component cho ứng dụng của riêng bạn, các component sẽ tự động được phát hiện trong thư mục `app/View/Components` và thư mục `resources/views/components`, vì vậy thông thường không cần đăng ký component nào nữa.

Bạn cũng có thể tạo các component trong các thư mục con:

```shell
php artisan make:component Forms/Input
```

Lệnh trên sẽ tạo một component `Input` trong thư mục `app/View/Components/Forms` và view sẽ được lưu trong thư mục `resources/views/components/forms`.

Nếu bạn muốn tạo một component ẩn (một component chỉ có một Blade template và không có class), bạn có thể sử dụng flag `--view` khi gọi lệnh `make:component`:

```shell
php artisan make:component forms.input --view
```

Lệnh trên sẽ tạo một file Blade ở `resources/views/components/forms/input.blade.php` có thể được hiển thị dưới dạng một component thông qua `<x-forms.input />`.

<a name="manually-registering-package-components"></a>
#### Manually Registering Package Components

Khi viết các component cho ứng dụng của bạn, các component sẽ tự động được đăng ký trong thư mục `app/View/Components` và thư mục `resources/views/components`.

Tuy nhiên, nếu bạn đang xây dựng một package sử dụng các component Blade, bạn sẽ cần phải đăng ký thủ công các class component của bạn và các bí danh tag HTML của nó. Thông thường, bạn nên đăng ký các component của bạn trong phương thức `boot` của service provider trong package của bạn:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot()
    {
        Blade::component('package-alert', Alert::class);
    }

Khi component của bạn đã được đăng ký, nó có thể được hiển thị bằng bí danh tag của nó:

```blade
<x-package-alert/>
```

Ngoài ra, bạn có thể sử dụng phương thức `componentNamespace` để tự động load các class component theo quy ước. Ví dụ: package `Nightshade` có thể có các component `Calendar` và `ColorPicker` nằm trong namespace là `Package\Views\Components`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

Điều này sẽ cho phép sử dụng các package component theo namespace của họ bằng cách sử dụng cú pháp như sau: `package-name::`:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade sẽ tự động phát hiện class được liên kết với component này bằng quy ước đặt tên pascal-casing theo tên của component. Các thư mục con cũng được hỗ trợ bằng ký hiệu "dấu chấm".

<a name="rendering-components"></a>
### Hiển thị Component

Để hiển thị một component, bạn có thể sử dụng tag component Blade trong một template Blade của bạn. Các tag component Blade bắt đầu bằng chuỗi `x-` theo sau là tên kebab case của class component:

```blade
<x-alert/>

<x-user-profile/>
```

Nếu class component được lồng sâu hơn trong thư mục `app/View/Components`, bạn có thể sử dụng ký tự `.` để biểu thị sự lồng thư mục. Ví dụ: nếu chúng ta giả sử là một component được lưu tại `app/View/Components/Inputs/Button.php`, thì chúng ta có thể hiển thị nó như sau:

```blade
<x-inputs.button/>
```

<a name="passing-data-to-components"></a>
### Truyền dữ liệu tới Components

Bạn có thể truyền dữ liệu đến các component Blade bằng cách sử dụng các thuộc tính HTML. Các giá trị theo kiểu nguyên thủy hoặc hard code có thể được truyền đến component bằng các chuỗi thuộc tính HTML đơn giản. Các biểu thức hoặc biến PHP phải được truyền đến component thông qua các thuộc tính có ký tự `:` làm tiền tố:

```blade
<x-alert type="error" :message="$message"/>
```

Bạn nên định nghĩa tất cả các dữ liệu của thuộc tính của component trong phương thức khởi tạo class của nó. Tất cả các thuộc tính public trong một component sẽ được tự động truyền view của component. Không cần thiết phải truyền dữ liệu vào view từ phương thức `render` của component:

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * The alert type.
         *
         * @var string
         */
        public $type;

        /**
         * The alert message.
         *
         * @var string
         */
        public $message;

        /**
         * Create the component instance.
         *
         * @param  string  $type
         * @param  string  $message
         * @return void
         */
        public function __construct($type, $message)
        {
            $this->type = $type;
            $this->message = $message;
        }

        /**
         * Get the view / contents that represent the component.
         *
         * @return \Illuminate\View\View|\Closure|string
         */
        public function render()
        {
            return view('components.alert');
        }
    }

Khi component của bạn được tạo, bạn có thể hiển thị nội dung của các biến public của component bằng cách echo các biến theo tên của nó:

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

<a name="casing"></a>
#### Casing

Các tham số của hàm khởi tạo component phải được chỉ định bằng cách sử dụng quy tắc đặt tên `camelCase`, trong khi quy tắc đặt tên `kebab-case` nên được sử dụng khi tham chiếu đến tên tham số đó trong các thuộc tính HTML của bạn. Ví dụ: cho hàm khởi tạo component sau:

    /**
     * Create the component instance.
     *
     * @param  string  $alertType
     * @return void
     */
    public function __construct($alertType)
    {
        $this->alertType = $alertType;
    }

Tham số `$alertType` có thể được cung cấp vào component như sau:

```blade
<x-alert alert-type="danger" />
```

<a name="short-attribute-syntax"></a>
#### Short Attribute Syntax

Khi truyền một thuộc tính tới một component, bạn cũng có thể sử dụng cú pháp "short attribute". Điều này thường tiện lợi vì tên thuộc tính thường giống với tên biến mà chúng đảm nhiệm:

```blade
{{-- Short attribute syntax... --}}
<x-profile :$userId :$name />

{{-- Is equivalent to... --}}
<x-profile :user-id="$userId" :name="$name" />
```

<a name="escaping-attribute-rendering"></a>
#### Escaping Attribute Rendering

Vì một số framework JavaScript chẳng hạn như Alpine.js cũng sử dụng thuộc tính có tiền tố dấu hai chấm, nên bạn có thể sử dụng tiền tố dấu hai chấm (`::`) để thông báo cho Blade rằng thuộc tính không phải là biểu thức PHP. Ví dụ: như component sau:

```blade
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

HTML sau đây sẽ được hiển thị bởi Blade:

```blade
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

<a name="component-methods"></a>
#### Component Methods

Ngoài các biến public có sẵn trong component template của bạn, bất kỳ phương thức public nào có trong component cũng có thể được gọi. Ví dụ: hãy tưởng tượng một component có phương thức `isSelected`:

    /**
     * Determine if the given option is the currently selected option.
     *
     * @param  string  $option
     * @return bool
     */
    public function isSelected($option)
    {
        return $option === $this->selected;
    }

Bạn có thể thực thi phương thức này từ trong component template của bạn bằng cách gọi một biến khớp với tên của phương thức đó:

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

<a name="using-attributes-slots-within-component-class"></a>
#### Accessing Attributes & Slots Within Component Classes

Các Blade component cũng cho phép bạn truy cập vào tên component, thuộc tính và slot bên trong phương thức render của class. Tuy nhiên, để truy cập vào các dữ liệu này, bạn nên trả về một Closure từ phương thức `render` của component của bạn. Closure sẽ nhận vào một mảng `$data` làm tham số duy nhất của nó. Mảng này sẽ chứa một số phần tử cung cấp thông tin về component:

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|\Closure|string
     */
    public function render()
    {
        return function (array $data) {
            // $data['componentName'];
            // $data['attributes'];
            // $data['slot'];

            return '<div>Components content</div>';
        };
    }

Tên `componentName` sẽ là tên được sử dụng trong thẻ HTML sau tiền tố `x-`. Vì vậy, `componentName` của `<x-alert />` sẽ là `alert`. Phần tử `attributes` sẽ chứa tất cả các thuộc tính có trong thẻ HTML. Phần tử `slot` là một instance `Illuminate\Support\HtmlString` với nội dung là của slot trong component.

Closure sẽ trả về một chuỗi. Nếu chuỗi được trả về tương ứng với view hiện có, thì view đó sẽ được hiển thị; nếu không, chuỗi được trả về sẽ được xem là view Blade inline.

<a name="additional-dependencies"></a>
#### Additional Dependencies

Nếu component của bạn yêu cầu các component khác từ [service container](/docs/{{version}}/container) của Laravel, bạn có thể liệt kê chúng lên trước các thuộc tính dữ liệu được truyền vào của component và chúng sẽ tự động được container đưa vào:

```php
use App\Services\AlertCreator;

/**
 * Create the component instance.
 *
 * @param  \App\Services\AlertCreator  $creator
 * @param  string  $type
 * @param  string  $message
 * @return void
 */
public function __construct(AlertCreator $creator, $type, $message)
{
    $this->creator = $creator;
    $this->type = $type;
    $this->message = $message;
}
```

<a name="hiding-attributes-and-methods"></a>
#### Hiding Attributes / Methods

Nếu bạn muốn ngăn không cho một số phương thức hoặc thuộc tính công khai hiển thị dưới dạng là một biến cho template component của bạn, bạn có thể thêm chúng vào thuộc tính mảng `$except` trên component của bạn:

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * The alert type.
         *
         * @var string
         */
        public $type;

        /**
         * The properties / methods that should not be exposed to the component template.
         *
         * @var array
         */
        protected $except = ['type'];
    }

<a name="component-attributes"></a>
### Thuộc tính Component

Chúng ta đã xem cách truyền các thuộc tính dữ liệu cho một component; tuy nhiên, đôi khi bạn có thể cần chỉ định thêm các thuộc tính HTML, chẳng hạn như `class`, không phải là một dữ liệu cần thiết để một component hoạt động. Thông thường, bạn sẽ muốn truyền các thuộc tính bổ sung đó vào phần tử gốc của component template. Ví dụ: hãy tưởng tượng chúng ta muốn hiển thị một component `alert` như sau:

```blade
<x-alert type="error" :message="$message" class="mt-4"/>
```

Tất cả các thuộc tính mà không nằm trong phương thức khởi tạo của component sẽ tự động được thêm vào "attribute bag" của component. Attribute bag này được tạo tự động cho component thông qua biến `$attributes`. Tất cả các thuộc tính có thể được hiển thị trong component bằng cách echo biến này:

```blade
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

> **Warning**
> Hiện tại, việc sử dụng các lệnh như `@env` trong các thẻ component không được hỗ trợ. Ví dụ: `<x-alert :live="@env('production')"/>` sẽ không được biên dịch.

<a name="default-merged-attributes"></a>
#### Default / Merged Attributes

Thỉnh thoảng bạn có thể cần chỉ định các giá trị mặc định cho các thuộc tính hoặc merge thêm các giá trị vào một số thuộc tính của component. Để thực hiện điều này, bạn có thể sử dụng phương thức `merge` của attribute bag. Phương thức này đặc biệt hữu ích để định nghĩa một set các class CSS mặc định luôn được áp dụng cho một component:

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Nếu chúng ta giả sử rằng component này được sử dụng như sau:

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

Cuối cùng, HTML mà được tạo ra của component sẽ xuất hiện như thế này:

```blade
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

<a name="conditionally-merge-classes"></a>
#### Conditionally Merge Classes

Thỉnh thoảng bạn có thể muốn hợp nhất các class nếu một điều kiện nhất định là `true`. Bạn có thể thực hiện điều này thông qua phương thức `class`, phương thức này chấp nhận một mảng các class trong đó khóa của mảng sẽ chứa tên class hoặc các class mà bạn muốn thêm, trong khi đó giá trị là một biểu thức boolean. Nếu phần tử mảng có một khóa là dạng số, thì nó sẽ luôn luôn được thêm vào danh sách class được hiển thị:

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

Nếu bạn cần hợp nhất các thuộc tính khác nhau vào component của bạn, bạn có thể kết hợp chuỗi phương thức `merge` vào phương thức `class`:

```blade
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

> **Note**
> Nếu bạn cần biên dịch có điều kiện các class trên các element HTML khác mà không nhận các thuộc tính được hợp nhất, bạn có thể sử dụng lệnh [`@class`](#conditional-classes).

<a name="non-class-attribute-merging"></a>
#### Non-Class Attribute Merging

Khi hợp nhất các thuộc tính không phải là thuộc tính `class`, các giá trị được cung cấp cho phương thức `merge` sẽ được coi là giá trị "mặc định" của thuộc tính. Tuy nhiên, không giống như các thuộc tính `class`, các thuộc tính này sẽ không được hợp nhất với các giá trị thuộc tính được đưa vào. Thay vào đó, chúng sẽ bị ghi đè. Ví dụ: việc implementation component `button` có thể giống như sau:

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

Để hiển thị component button bằng một `type` tùy chỉnh, nó có thể được chỉ định khi sử dụng component. Nếu không có loại nào được chỉ định, thì loại `button` sẽ được sử dụng:

```blade
<x-button type="submit">
    Submit
</x-button>
```

HTML mà được hiển thị cho component `button` trong ví dụ này sẽ như sau:

```blade
<button type="submit">
    Submit
</button>
```

Nếu bạn muốn một thuộc tính không phải là `class` có giá trị mặc định và các giá trị này được đưa vào và kết hợp với nhau, bạn có thể sử dụng phương thức `prepends`. Trong ví dụ này, thuộc tính `data-controller` sẽ luôn bắt đầu bằng `profile-controller` và mọi giá trị `data-controller` được thêm vào sẽ được đặt sau giá trị mặc định:

```blade
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

<a name="filtering-attributes"></a>
#### Retrieving & Filtering Attributes

Bạn có thể lọc các thuộc tính bằng phương thức `filter`. Phương thức này chấp nhận một closure sẽ trả về giá trị `true` nếu bạn muốn giữ lại các thuộc tính trong attribute bag:

```blade
{{ $attributes->filter(fn ($value, $key) => $key == 'foo') }}
```

Để thuận tiện, bạn có thể sử dụng phương thức `whereStartsWith` để lấy ra tất cả các thuộc tính có khóa bắt đầu bằng một chuỗi đã cho:

```blade
{{ $attributes->whereStartsWith('wire:model') }}
```

Ngược lại, phương thức `whereDoesntStartWith` có thể được sử dụng để loại trừ tất cả các thuộc tính có khóa bắt đầu bằng một chuỗi đã cho:

```blade
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

Sử dụng phương thức `first`, bạn có thể hiển thị thuộc tính đầu tiên có trong một attribute bag nhất định:

```blade
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

Nếu bạn muốn kiểm tra xem một thuộc tính có trong component hay không, bạn có thể sử dụng phương thức `has`. Phương thức này chấp nhận tên một thuộc tính làm tham số duy nhất của nó và trả về một giá trị boolean cho biết thuộc tính đó có tồn tại hay không:

```blade
@if ($attributes->has('class'))
    <div>Class attribute is present</div>
@endif
```

Bạn có thể lấy ra giá trị của một thuộc tính cụ thể bằng phương thức `get`:

```blade
{{ $attributes->get('class') }}
```

<a name="reserved-keywords"></a>
### Reserved Keywords

Mặc định, một số từ khóa được dành riêng cho mục đích sử dụng nội bộ của Blade để hiển thị cho các component. Các từ khóa sau không thể định nghĩa là một thuộc tính công khai hoặc tên phương thức trong các component của bạn:

<div class="content-list" markdown="1">

- `data`
- `render`
- `resolveView`
- `shouldRender`
- `view`
- `withAttributes`
- `withName`

</div>

<a name="slots"></a>
### Slots

Bạn thường sẽ cần truyền một nội dung bổ sung vào component của bạn thông qua "slots". Các slot của component sẽ được hiển thị bằng cách lặp lại biến `$slot`. Để khám phá khái niệm này, hãy tưởng tượng rằng một component `alert` có dạng như sau:

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Chúng ta có thể truyền nội dung vào `slot` bằng cách đưa nội dung vào trong component:

```blade
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

Thỉnh thoảng một component có thể cần hiển thị nhiều slot khác nhau ở các vị trí khác nhau trong component. Hãy thử sửa alert component của chúng ta để cho phép chèn một slot "title" vào component:

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Bạn có thể định nghĩa nội dung của một slot cụ thể bằng tag `x-slot`. Bất kỳ nội dung nào mà không nằm trong tag `x-slot` sẽ được truyền đến component thông qua biến `$slot`:

```xml
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

<a name="scoped-slots"></a>
#### Scoped Slots

Nếu bạn đã sử dụng một JavaScript framework như Vue, bạn có thể quen thuộc với các "scoped slot", cho phép bạn truy cập vào dữ liệu hoặc phương thức của component trong slot của bạn. Bạn cũng có thể làm ra hành vi tương tự đó trong Laravel bằng cách định nghĩa các phương thức hoặc thuộc tính public trong component của bạn và truy cập vào component đó trong slot của bạn thông qua biến `$component`. Trong ví dụ này, chúng ta sẽ giả định rằng component `x-alert` có một phương thức `formatAlert` công khai được định nghĩa trong class component của nó:

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

<a name="slot-attributes"></a>
#### Slot Attributes

Giống như các component của Blade, bạn có thể gán [thuộc tính](#component-attributes) bổ sung cho các slot, chẳng hạn như tên class CSS:

```xml
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>

    Content

    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
```

Để tương tác với các thuộc tính của slot, bạn có thể truy cập vào thuộc tính `attributes` của biến slot. Để biết thêm thông tin về cách tương tác với các thuộc tính này, vui lòng tham khảo tài liệu về [thuộc tính component](#component-attributes):

```blade
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

<a name="inline-component-views"></a>
### Inline Component Views

Đối với các component rất nhỏ, bạn có thể cảm thấy cồng kềnh khi quản lý cả một class component và template view của component đó. Vì lý do đó, bạn có thể trả về một component trực tiếp từ phương thức `render`:

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|\Closure|string
     */
    public function render()
    {
        return <<<'blade'
            <div class="alert alert-danger">
                {{ $slot }}
            </div>
        blade;
    }

<a name="generating-inline-view-components"></a>
#### Generating Inline View Components

Để tạo một component theo dạng inline view, bạn có thể sử dụng tùy chọn `inline` khi chạy lệnh `make:component`:

```shell
php artisan make:component Alert --inline
```

<a name="dynamic-components"></a>
### Dynamic Components

Thỉnh thoảng bạn có thể cần render một component nhưng không biết component nào sẽ được render cho đến thực sự được chạy. Trong trường hợp này, bạn có thể sử dụng component `dynamic-component` có sẵn của Laravel để render ra một component dựa trên giá trị hoặc biến khi được chạy:

```blade
<x-dynamic-component :component="$componentName" class="mt-4" />
```

<a name="manually-registering-components"></a>
### Manually Registering Components

> **Warning**
> Tài liệu sau đây sẽ nói về cách đăng ký component thủ công chủ yếu áp dụng cho những người đang viết các package Laravel bao gồm cả các view component. Nếu bạn không viết package, thì phần tài liệu component này có thể không liên quan đến bạn.

Khi viết các component cho ứng dụng của bạn, các component sẽ tự động được phát hiện trong thư mục `app/View/Components` và thư mục `resources/views/components`.

Tuy nhiên, nếu bạn đang xây dựng một package sử dụng các component Blade hoặc đặt các component trong các thư mục không theo quy ước trên, bạn sẽ cần phải đăng ký thủ công các class component của bạn và các bí danh thẻ HTML của nó để Laravel có thể biết nơi để tìm component đó. Bạn nên đăng ký các component của bạn trong phương thức `boot` của service provider trong package của bạn:

    use Illuminate\Support\Facades\Blade;
    use VendorPackage\View\Components\AlertComponent;

    /**
     * Bootstrap your package's services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::component('package-alert', AlertComponent::class);
    }

Sau khi component của bạn đã được đăng ký, nó có thể được hiển thị bằng cách sử dụng bí danh thẻ html của nó:

```blade
<x-package-alert/>
```

#### Autoloading Package Components

Ngoài ra, bạn có thể sử dụng phương thức `componentNamespace` để autoload các class component theo quy ước. Ví dụ, một package `Nightshade` có thể có các component `Calendar` và `ColorPicker` nằm trong namespace `Package\Views\Components`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

Điều này sẽ cho phép bạn sử dụng các package component theo namespace của library bằng cú pháp `package-name::`:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade sẽ tự động phát hiện class được liên kết với component bằng cách phân biệt chữ hoa và chữ thường trong tên component. Các thư mục con cũng được hỗ trợ bằng cách dùng ký hiệu "dot".

<a name="anonymous-components"></a>
## Component ẩn

Tương tự như các component inline, các component ẩn cũng cung cấp cơ chế quản lý một component thông qua một file duy nhất. Tuy nhiên, các component ẩn sẽ sử dụng một file view và không có class nào liên kết đến nó. Để định nghĩa một component ẩn, bạn chỉ cần set một template Blade vào trong thư mục `resources/views/components` của bạn. Ví dụ: giả sử bạn đã định nghĩa một component tại `resources/views/components/alert.blade.php`, và bạn có thể hiển thị nó như sau:

```blade
<x-alert/>
```

Bạn có thể sử dụng ký tự `.` để cho biết một component được nằm sâu bên trong thư mục `component`. Ví dụ: giả sử component được định nghĩa tại `resources/views/components/inputs/button.blade.php`, bạn có thể hiển thị nó như sau:

```blade
<x-inputs.button/>
```

<a name="anonymous-index-components"></a>
### Anonymous Index Components

Thỉnh thoảng, khi một component được tạo thành từ nhiều template Blade, bạn có thể muốn nhóm các template của component đã cho trong một thư mục. Ví dụ: hãy tưởng tượng một component "accordion" với cấu trúc thư mục sau:

```none
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

Cấu trúc thư mục này cho phép bạn hiển thị component accordion và item của nó như sau:

```blade
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

Tuy nhiên, để hiển thị component accordion thông qua `x-accordion`, chúng ta buộc phải đặt template của component "index" vào trong thư mục `resources/views/components` thay vì lồng nó vào trong thư mục `accordion` với các template liên quan khác đến accordion.

Rất may, Blade cho phép bạn đặt file `index.blade.php` vào trong thư mục template của component. Khi có template `index.blade.php` cho component, nó sẽ được hiển thị dưới dạng "gốc" của component. Vì vậy, chúng ta có thể tiếp tục sử dụng cùng một cú pháp Blade được đưa ra trong ví dụ trên; tuy nhiên, chúng ta sẽ điều chỉnh cấu trúc thư mục của mình như sau:

```none
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
```

<a name="data-properties-attributes"></a>
### Data Properties / Attributes

Vì các component ẩn này không có bất kỳ class nào liên kết đến với nó, và bạn có thể tự hỏi là làm cách nào để phân biệt được dữ liệu nào là được truyền vào cho component dưới dạng biến và thuộc tính nào sẽ được set vào trong [attribute bag](#component-attributes) của component.

Bạn có thể chỉ định các thuộc tính sẽ được coi là biến dữ liệu bằng cách sử dụng lệnh `@props` ở đầu file template Blade của component của bạn. Tất cả các thuộc tính khác có trong component sẽ luôn có sẵn trong attribute bag của component. Nếu bạn muốn cung cấp cho một biến dữ liệu với một giá trị mặc định, bạn có thể chỉ định tên của biến đó làm khóa của mảng và giá trị mặc định đó sẽ là giá trị của mảng:

```blade
<!-- /resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Với định nghĩa component như trên, chúng ta có thể tạo ra component như sau:

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

<a name="accessing-parent-data"></a>
### Accessing Parent Data

Thỉnh thoảng bạn có thể muốn truy cập vào dữ liệu từ component cha bên trong component con. Trong những trường hợp này, bạn có thể sử dụng lệnh `@aware`. Ví dụ: hãy tưởng tượng chúng ta đang xây dựng một component menu phức tạp có chứa component cha là `<x-menu>` và component con là `<x-menu.item>`:

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

Component `<x-menu>` có thể có cách triển khai như sau:

```blade
<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

Bởi vì thuộc tính `color` chỉ được truyền vào component cha (`<x-menu>`), nên nó sẽ không được truyền vào component con `<x-menu.item>`. Tuy nhiên, nếu chúng ta sử dụng lệnh `@aware`, chúng ta cũng có thể làm cho nó có khả năng truyền vào bên trong `<x-menu.item>`:

```blade
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

> **Warning**
> Lệnh `@aware` sẽ không thể truy cập được vào dữ liệu cha nếu bạn không truyền rõ ràng dữ liệu vào component cha thông qua các thuộc tính HTML. Mặc định, các giá trị `@props` mà không được truyền rõ ràng đến component cha thì nó cũng không thể truy cập được bởi lệnh `@aware`.

<a name="anonymous-component-paths"></a>
### Anonymous Component Paths

Như đã thảo luận trước đó, các component ẩn thường được định nghĩa bằng cách lưu một template Blade vào trong thư mục `resources/views/components` của bạn. Tuy nhiên, đôi khi bạn có thể muốn đăng ký các đường dẫn component ẩn khác, khác với đường dẫn mặc định của Laravel.

Phương thức `anonymousComponentPath` chấp nhận một "đường dẫn" đến vị trí component ẩn làm tham số đầu tiên và một tùy chọn "namespace" mà các component nên được đặt ở sau namespace đó làm tham số thứ hai. Thông thường, phương thức này nên được gọi từ phương thức `boot` của một trong các [service providers](/docs/{{version}}/providers) của ứng dụng của bạn:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::anonymousComponentPath(__DIR__.'/../components');
    }

Khi đường dẫn component đã được đăng ký mà không có tiền tố được chỉ định như ở trong ví dụ trên, thì chúng có thể được hiển thị trong các component Blade của bạn mà không cần tiền tố tương ứng. Ví dụ, nếu một component `panel.blade.php` tồn tại trong đường dẫn đã đăng ký ở trên, thì nó có thể được hiển thị như sau:

```blade
<x-panel />
```

Tiền tố "namespaces" có thể được cung cấp làm tham số thứ hai cho phương thức `anonymousComponentPath`:

    Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');

Khi cung cấp tiền tố, các component trong "namespace" đó có thể được hiển thị bằng cách thêm tiền tố namespace của component vào tên của component khi component được hiển thị:

```blade
<x-dashboard::panel />
```

<a name="building-layouts"></a>
## Building Layouts

<a name="layouts-using-components"></a>
### Layouts dùng Components

Hầu hết các ứng dụng web duy trì cùng một layout chung trên các trang web khác nhau. Sẽ cực kỳ cồng kềnh và khó duy trì ứng dụng của chúng ta nếu chúng ta phải lặp lại toàn bộ HTML layout trong mọi view của chúng ta tạo. Rất may và sẽ rất thuận tiện nếu chúng ta định nghĩ layout này dưới dạng một [Blade component](#components) duy nhất và sau đó sử dụng nó trong toàn bộ ứng dụng của chúng ta.

<a name="defining-the-layout-component"></a>
#### Defining The Layout Component

Ví dụ, hãy tưởng tượng chúng ta đang xây dựng một ứng dụng danh sách "những việc cần làm". Chúng ta có thể định nghĩa một component `layout` giống như sau:

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

<a name="applying-the-layout-component"></a>
#### Applying The Layout Component

Khi component `layout` đã được định nghĩa, chúng ta có thể tạo view Blade sử dụng component đó. Trong ví dụ này, chúng ta sẽ định nghĩa một view đơn giản hiển thị danh sách các công việc của chúng ta:

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

Hãy nhớ rằng, mặc định, nội dung được thêm vào một component sẽ được cung cấp dưới dạng biến `$slot` trong component `layout` của chúng ta. Như bạn có thể nhận thấy, `layout` của chúng ta cũng ưu tiên slot `$title` nếu nó được cung cấp; nếu không, tiêu đề mặc định sẽ được hiển thị. Chúng ta có thể thêm tiêu đề tùy chỉnh từ view danh sách công việc của bạn bằng cách sử dụng cú pháp slot tiêu chuẩn được thảo luận trong [tài liệu component](#components):

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        Custom Title
    </x-slot>

    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

Bây giờ chúng ta đã định nghĩa xong view danh sách công việc và layout của bạn, bây giờ chúng ta chỉ cần trả lại layout `task` này từ một route:

    use App\Models\Task;

    Route::get('/tasks', function () {
        return view('tasks', ['tasks' => Task::all()]);
    });

<a name="layouts-using-template-inheritance"></a>
### Layouts dùng Template kế thừa

<a name="defining-a-layout"></a>
#### Defining A Layout

Layout cũng có thể được tạo thông qua "kế thừa template". Đây là cách chính để xây dựng ứng dụng trước khi giới thiệu [component](#components).

Để bắt đầu, chúng ta hãy xem một ví dụ đơn giản. Đầu tiên, chúng ta sẽ kiểm tra layout của một trang. Vì hầu hết các ứng dụng web sẽ duy trì cùng một layout chung trên nhiều trang khác nhau, nên sẽ thuận tiện khi định nghĩa layout này dưới dạng một Blade view duy nhất:

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

Như bạn có thể thấy, file này chứa code HTML điển hình. Tuy nhiên, hãy lưu ý các lệnh `@section` và `@yield`. Lệnh `@section`, như tên của nó, định nghĩa một section nội dung, trong khi lệnh `@yield` được sử dụng để hiển thị nội dung của một section nhất định.

Bây giờ chúng ta đã định nghĩa xong một layout cho ứng dụng của bạn, tiếp theo, hãy định nghĩa một trang con kế thừa từ layout đó.

<a name="extending-a-layout"></a>
#### Extending A Layout

Khi định nghĩa view con, hãy sử dụng lệnh Blade `@extends` để chỉ định layout nào mà view con sẽ "kế thừa". Các view sẽ extend layout Blade và có thể đưa nội dung vào các section của layout bằng cách sử dụng lệnh `@section`. Hãy nhớ rằng, như đã thấy trong ví dụ trên, nội dung của các section này sẽ được hiển thị trong layout bằng cách sử dụng `@yield`:

```blade
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @@parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

Trong ví dụ này, section `sidebar` đang sử dụng lệnh `@@parent` để nối thêm (chứ không phải ghi đè) nội dung vào sidebar của layout. Lệnh `@@parent` sẽ được thay thế bằng nội dung của layout khi view được hiển thị.

> **Note**
> Trái ngược với ví dụ trước đó, section `sidebar` này kết thúc bằng `@endsection` thay vì `@show`. Lệnh `@endsection` sẽ định nghĩa kết thúc một section trong khi `@show` cũng sẽ định nghĩa kết thúc một section nhưng nó cũng định nghĩa thêm một lệnh `@yield` để cho layout con để có thể định nghĩa thêm nội dung vào layout chính.

Lệnh `@yield` cũng chấp nhận một giá trị mặc định làm tham số thứ hai của nó. Giá trị này sẽ được hiển thị nếu section đang được tạo là undefined:

```blade
@yield('content', 'Default content')
```

<a name="forms"></a>
## Forms

<a name="csrf-field"></a>
### CSRF Field

Bất cứ khi nào bạn định nghĩa một HTML form trong ứng dụng của bạn, bạn nên thêm một trường hidden CSRF token vào trong form của bạn để middleware [bảo vệ CSRF](/docs/{{version}}/csrf) có thể xác thực request đó. Bạn có thể sử dụng lệnh Blade `@csrf` để tạo trường token đó:

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

<a name="method-field"></a>
### Method Field

Vì các HTML form không thể tạo các request `PUT`, `PATCH`, hoặc `DELETE`, nên bạn sẽ cần thêm trường `_method` hidden vào để làm giả các hành động cho các HTTP này. Lệnh Blade `@method` có thể tạo ra trường như vậy cho bạn:

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

<a name="validation-errors"></a>
### Validation Errors

Lệnh `@error` có thể được sử dụng để nhanh chóng kiểm tra xem trong [thông báo lỗi validation](/docs/{{version}}/validation#quick-displaying-the-validation-errors) có tồn tại lỗi cho một thuộc tính hay không. Trong lệnh `@error` bạn có thể dùng biến `$message` để hiển thị thông báo lỗi:

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Do lệnh `@error` sẽ được biên dịch thành câu lệnh "if", nên bạn có thể sử dụng lệnh `@else` để hiển thị nội dung khi không có lỗi cho một thuộc tính:

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror">
```

Bạn có thể truyền [tên của một error bag cụ thể](/docs/{{version}}/validation#named-error-bags) làm tham số thứ hai cho lệnh `@error` để lấy ra thông báo lỗi validation trên các trang chứa nhiều form:

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror">

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

<a name="stacks"></a>
## Stacks

Blade cho phép bạn khai báo thêm các file hoặc các biến vào trong các stack đã được tên, để có thể được hiển thị ở một nơi khác trong view hoặc layout. Điều này có thể đặc biệt hữu ích để định nghĩa một thư viện JavaScript nào đó theo yêu cầu của view con của bạn:

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

Nếu bạn muốn `@push` một nội dung nếu một biểu thức boolean trả về kết quả là `true`, thì bạn có thể sử dụng lệnh `@pushIf`:

```blade
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

Bạn có thể khai báo cho một stack nhiều lần nếu cần. Để hiển thị nội dung stack hoàn chỉnh, truyền tên của stack vào lệnh `@stack`:

```blade
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

Nếu bạn muốn thêm nội dung vào đầu một stack, bạn có thể sử dụng lệnh `@prepend`:

```blade
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

<a name="service-injection"></a>
## Service Injection

Lệnh `@inject` có thể được sử dụng để lấy một service ra từ [service container](/docs/{{version}}/container). Tham số đầu tiên được truyền vào `@inject` là tên biến mà service sẽ được set, trong khi tham số thứ hai là tên class hoặc là tên một interface của service mà bạn muốn resolve:

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

<a name="rendering-inline-blade-templates"></a>
## Tạo Inline Blade Templates

Thỉnh thoảng bạn có thể cần chuyển đổi một chuỗi Blade template thô thành HTML hợp lệ. Bạn có thể thực hiện việc này bằng phương thức `render` được cung cấp bởi facade `Blade`. Phương thức `render` sẽ chấp nhận chuỗi Blade template và một mảng dữ liệu tùy chọn để cung cấp cho template:

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Laravel sẽ render các inline Blade template bằng cách viết chúng vào thư mục `storage/framework/views`. Nếu bạn muốn Laravel xóa các file tạm thời này sau khi render Blade template xong, bạn có thể cung cấp tham số `deleteCachedView` cho phương thức:

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

<a name="rendering-blade-fragments"></a>
## Tạo Blade Fragments

Khi sử dụng các frontend framework như [Turbo](https://turbo.hotwired.dev/) và [htmx](https://htmx.org/), đôi khi bạn chỉ cần trả về một phần của Blade template trong response HTTP của bạn. Các "fragments" của Blade cho phép bạn thực hiện điều này. Để bắt đầu, hãy tạo một phần Blade template của bạn trong các lệnh `@fragment` và `@endfragment`:

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

Sau đó, khi hiển thị view mà sử dụng template này, bạn có thể gọi phương thức `fragment` để chỉ định rằng chỉ có fragment được chỉ định mới có thể được đưa vào response HTTP gửi về:

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

Phương thức `fragmentIf` cho phép bạn trả về một fragment của view dựa trên một điều kiện nhất định. Nếu không phải, thì toàn bộ view sẽ được trả về:

```php
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```

Các phương thức `fragments` và `fragmentsIf` cho phép bạn trả về nhiều fragment view trong một response. Các fragment sẽ được nối lại với nhau:

```php
view('dashboard', ['users' => $users])
    ->fragments(['user-list', 'comment-list']);

view('dashboard', ['users' => $users])
    ->fragmentsIf(
        $request->hasHeader('HX-Request'),
        ['user-list', 'comment-list']
    );
```

<a name="extending-blade"></a>
## Mở rộng blade

Blade cho phép bạn định nghĩa thêm các lệnh tùy biến của riêng bạn bằng phương thức `directive`. Khi trình biên dịch Blade gặp phải các lệnh tùy biến này, nó sẽ tự động gọi đến hàm callback đã được khai báo cho lệnh này.

Ví dụ sau đây sẽ tạo ra một lệnh `@datetime($var)` để format lại một biến `$var` đã cho, và biến này phải là một instance của `DateTime`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
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
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }
    }

Như bạn có thể thấy, chúng ta sẽ nối phương thức `format` vào bất kỳ biểu thức nào được truyền vào lệnh. Vì vậy, trong ví dụ này, code PHP cuối cùng được tạo ra bởi lệnh này sẽ như sau:

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> **Warning**
> Sau khi cập nhật logic của lệnh Blade, bạn sẽ cần xóa tất cả các view Blade đã được lưu trong bộ nhớ cache. Các view Blade được lưu trong bộ nhớ cache có thể được loại bỏ bằng lệnh Artisan `view:clear`.

<a name="custom-echo-handlers"></a>
### Tuỳ chỉnh xử lý hiển thị

Nếu bạn cố gắng thử "echo" một đối tượng bằng Blade, thì phương thức `__toString` của đối tượng đó sẽ được gọi. Phương thức [`__toString`](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) là một trong những "phương thức magic" được tích hợp sẵn trong PHP. Tuy nhiên, đôi khi bạn có thể không có quyền kiểm soát đối với phương thức `__toString` của một class nhất định, chẳng hạn như khi class mà bạn đang tương tác thuộc về thư viện của third-party.

Trong những trường hợp như vậy, Blade cho phép bạn tuỳ chỉnh xử lý hiển thị cho một loại đối tượng cụ thể đó. Để thực hiện điều này, bạn nên gọi phương thức `stringable` của Blade. Phương thức `stringable` chấp nhận một closure. Closure này sẽ khai báo kiểu đối tượng mà nó chịu trách nhiệm hiển thị. Thông thường, phương thức `stringable` nên được gọi trong phương thức `boot` của class `AppServiceProvider` trong ứng dụng của bạn:

    use Illuminate\Support\Facades\Blade;
    use Money\Money;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::stringable(function (Money $money) {
            return $money->formatTo('en_GB');
        });
    }

Khi tùy chỉnh xử lý hiển thị của bạn đã được định nghĩa xong, bạn chỉ đơn giản là hiển thị đối tượng trong template Blade của bạn:

```blade
Cost: {{ $money }}
```

<a name="custom-if-statements"></a>
### Tuỳ biến lệnh if

Lập trình một lệnh tùy biến đôi khi lại là phức tạp hơn là định nghĩa một câu lệnh điều kiện tùy biến đơn giản. Vì lý do đó, Blade cung cấp phương thức `Blade::if` cho phép bạn nhanh chóng định nghĩa các lệnh tùy biến có điều kiện bằng cách sử dụng closures. Ví dụ: hãy định nghĩa một điều kiện tùy biến để có thể kiểm tra cấu hình "disk" hiện tại của application. Chúng ta có thể làm điều này trong phương thức `boot` của `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('disk', function ($value) {
            return config('filesystems.default') === $value;
        });
    }

Khi điều kiện tùy biến đã được định nghĩa xong, bạn có thể sử dụng nó trong các template của bạn:

```blade
@disk('local')
    <!-- The application is using the local disk... -->
@elsedisk('s3')
    <!-- The application is using the s3 disk... -->
@else
    <!-- The application is using some other disk... -->
@enddisk

@unlessdisk('local')
    <!-- The application is not using the local disk... -->
@enddisk
```
