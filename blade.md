# Blade Templates

- [Giới thiệu](#introduction)
- [Kế thừa template](#template-inheritance)
    - [Định nghĩa một layout](#defining-a-layout)
    - [Kế thừa một Layout](#extending-a-layout)
- [Components và Slots](#components-and-slots)
- [Hiển thị dữ liệu](#displaying-data)
    - [Blade và JavaScript Frameworks](#blade-and-javascript-frameworks)
- [Control Structures](#control-structures)
    - [Lệnh if](#if-statements)
    - [Lệnh switch](#switch-statements)
    - [Loops](#loops)
    - [Biến loop](#the-loop-variable)
    - [Comments](#comments)
    - [PHP](#php)
- [Thêm Sub-Views](#including-sub-views)
    - [Tạo Views cho Collections](#rendering-views-for-collections)
- [Stacks](#stacks)
- [Service Injection](#service-injection)
- [Mở rộng blade](#extending-blade)
    - [Tuỳ biến lệnh if](#custom-if-statements)

<a name="introduction"></a>
## Giới thiệu

Blade là công cụ tạo template đơn giản nhưng mạnh mẽ được cung cấp cùng với Laravel. Không giống như các công cụ tạo template PHP phổ biến khác, Blade không hạn chế bạn sử dụng code PHP trong view của bạn. Trên thực tế, tất cả các view Blade được biên dịch thành code PHP và được lưu vào cache cho đến khi chúng được sửa đổi, có nghĩa là về cơ bản Blade không làm tăng chi phí chung cho application của bạn. Các file Blade sử dụng phần đuôi mở rộng file là `.blade.php` và thường được lưu trữ trong thư mục `resources/views`.

<a name="template-inheritance"></a>
## Kế thừa template

<a name="defining-a-layout"></a>
### Định nghĩa một layout

Hai trong số những lợi ích chính của việc sử dụng Blade là _kế thừa template_ và _sections_. Để bắt đầu, chúng ta hãy xem một ví dụ đơn giản. Đầu tiên, chúng ta hãy xem thử một layout trang "master". Vì hầu hết các trang web đều cố gắng duy trì cùng một layout chung trên các trang khác nhau, nên Blade rất tiện để định nghĩa cho layout này chỉ trong một file view Blade duy nhất:

    <!-- Stored in resources/views/layouts/app.blade.php -->

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

Như bạn có thể thấy, file này chứa code HTML. Tuy nhiên, hãy lưu ý các lệnh `@section` và `@yield`. Lệnh `@section`, cũng như cái tên của nó, dùng để định nghĩa một phần của nội dung, trong khi lệnh `@yield` được sử dụng để hiển thị nội dung của một phần nhất định.

Bây giờ chúng ta đã định nghĩa xong một layout cho application của bạn, hãy bắt đầu định nghĩa một trang con kế thừa từ layout đó.

<a name="extending-a-layout"></a>
### Kế thừa một Layout

Khi định nghĩa một view con, sử dụng lệnh `@extends` của Blade để chỉ định layout mà view con đó được "kế thừa". Các view mà được mở rộng từ một layout Blade có thể đưa thêm nội dung vào các phần của layout bằng cách sử dụng các lệnh `@section`. Nhớ rằng, có thể thấy trong ví dụ trên, nội dung của các phần này sẽ được hiển thị trong layout bằng cách sử dụng `@yield`:

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

Trong ví dụ này, phần `sidebar` đang sử dụng lệnh `@@parent` để nối thêm (chứ không phải ghi đè) vào sidebar của layout. Lệnh `@@parent` sẽ được thay thế bằng nội dung của layout khi view được hiển thị.

> {tip} Trái với ví dụ trước, phần `sidebar` này kết thúc bằng `@endsection` thay vì `@show`. Lệnh `@endsection` sẽ chỉ xác định một section trong khi `@show` sẽ xác định và **render** section đó luôn.

Blade view có thể được trả về từ route dùng global helper `view`:

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## Components và Slots

Các component và slot cung cấp lợi ích tương tự cho các section và layout; tuy nhiên, một số có thể tìm thấy mô hình tư duy của các component và slot là dễ hiểu hơn. Trước tiên, hãy tưởng tượng một component "cảnh báo" có thể tái sử dụng và chúng ta muốn sử dụng lại nó trong suốt ứng dụng của mình:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

Biến `{{ $slot }}` sẽ chứa nội dung mà chúng ta muốn đưa vào component. Bây giờ, để xây dựng component này, chúng ta có thể sử dụng lệnh `@component` Blade:

    @component('alert')
        <strong>Whoops!</strong> Something went wrong!
    @endcomponent

Thỉnh thoảng nó rất hữu ích để định nghĩa nhiều slot cho một component. Hãy chỉnh sửa component cảnh báo của chúng ta để cho phép injection một "title". Các slot đã được đặt tên có thể được hiển thị bằng cách "echoing" biến khớp với tên của chúng:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

Bây giờ, chúng ta có thể inject nội dung vào slot được đặt tên bằng cách sử dụng lệnh `@slot`. Bất kỳ nội dung nào không nằm trong lệnh `@slot` sẽ được pass đến component trong biến `$slot`:

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        You are not allowed to access this resource!
    @endcomponent

#### Pass dữ liệu bổ sung đến component

Thỉnh thoảng bạn có thể cần pass dữ liệu bổ sung cho một component. Vì lý do này, bạn có thể pass một mảng dữ liệu làm tham số thứ hai cho lệnh  `@component`. Tất cả dữ liệu sẽ được cung cấp cho component template dưới dạng các biến:

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

<a name="displaying-data"></a>
## Hiển thị dữ liệu

Bạn có thể hiển thị dữ liệu đã được pass đến Blade view của bạn bằng cách wrap biến trong hai lần dấu ngoặc nhọn. Ví dụ: một route như sau:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

Bạn có thể hiển thị nội dung của biến `name` như thế này:

    Hello, {{ $name }}.

Tất nhiên, bạn không bị giới hạn trong việc hiển thị nội dung của các biến đã được pass đến view. Bạn cũng có thể echo ra kết quả của bất kỳ hàm PHP nào. Thực tế, bạn có thể đặt bất kỳ code PHP nào bạn muốn vào trong câu lệnh echo Blade:

    The current UNIX timestamp is {{ time() }}.

> {tip} Các câu lệnh Blade `{{ }}` sẽ được tự động gửi qua hàm `htmlspecialchars` của PHP để ngăn chặn các cuộc tấn công XSS.

#### Hiển thị dữ liệu unescaped

Mặc định, các câu lệnh Blade `{{ }}` được tự động gửi qua hàm `htmlspecialchars` của PHP để ngăn chặn các cuộc tấn công XSS. Nếu bạn không muốn dữ liệu của mình được escaped, bạn có thể sử dụng cú pháp sau:

    Hello, {!! $name !!}.

> {note} Hãy cẩn thận khi echo nội dung được cung cấp bởi người dùng application của bạn. Luôn sử dụng escaped, với cú pháp hai lần dấu ngoặc nhọn để ngăn chặn các cuộc tấn công XSS khi hiển thị dữ liệu do người dùng cung cấp.

#### Tạo JSON

Thỉnh thoảng bạn có thể pass một mảng vào view của bạn với ý định hiển thị nó dưới dạng JSON để khởi tạo một biến JavaScript. Ví dụ:

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

Tuy nhiên, thay vì gọi thủ công `json_encode`, bạn có thể sử dụng lệnh Blade `@json`:

    <script>
        var app = @json($array);
    </script>

<a name="blade-and-javascript-frameworks"></a>
### Blade và JavaScript Frameworks

Do nhiều framework JavaScript cũng sử dụng hai lần dấu ngoặc nhọn để biểu thị một biểu thức đã cho sẽ được hiển thị trong trình duyệt, nên bạn có thể sử dụng ký hiệu `@` để thông báo cho Blade rendering engine biểu thức này sẽ không bị chạm vào. Ví dụ:

    <h1>Laravel</h1>

    Hello, @{{ name }}.

Trong ví dụ này, ký hiệu `@` sẽ bị xóa bởi Blade; tuy nhiên, biểu thức `{{ name }}` sẽ vẫn còn và chưa được xử lý bởi Blade engine, cho phép thay vào đó sẽ được render bởi framework JavaScript của bạn.

#### Lệnh `@verbatim`

Nếu trong template của bạn đang hiển thị nhiều biến JavaScript, bạn có thể wrap HTML trong lệnh `@verbatim` để bạn không phải đặt tiền tố cho mỗi câu lệnh echo Blade bằng ký hiệu `@`:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## Control Structures

Ngoài kế thừa khiển và hiển thị dữ liệu, Blade cũng cung cấp các shortcut thuận tiện cho các cấu trúc điều khiển PHP phổ biến, chẳng hạn như các câu lệnh và vòng lặp có điều kiện. Các shortcut này cung cấp một cách làm việc rất gọn gàng, gọn gàng với các cấu trúc điều khiển PHP, trong khi vẫn quen thuộc với các hàm tương tư trong PHP.

<a name="if-statements"></a>
### Lệnh if

Bạn có thể xây dựng các câu lệnh `if` bằng cách sử dụng các lệnh `@if`, `@elseif`, `@else`, và `@endif`. Các lệnh này hoạt động giống hệt với các hàm tương tư trong PHP:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

Để thuận tiện, Blade cũng cung cấp một lệnh `@unless`:

    @unless (Auth::check())
        You are not signed in.
    @endunless

Ngoài các lệnh có điều kiện đã được thảo luận ở trên, các lệnh `@isset` và `@empty` có thể được sử dụng làm các shortcut thuận tiện cho các hàm PHP tương ứng của chúng:

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### Lệnh authentication

Các lệnh `@auth` và `@guest` có thể được sử dụng để nhanh chóng xác định xem người dùng hiện tại có được xác thực chưa hay là khách:

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

Nếu cần, bạn có thể chỉ định [authentication guard](/docs/{{version}}/authentication) cần được kiểm tra khi sử dụng các lệnh `@auth` và `@guest`:

    @auth('admin')
        // The user is authenticated...
    @endauth

    @guest('admin')
        // The user is not authenticated...
    @endguest

#### Lệnh section

Bạn có thể kiểm tra xem một section có nội dung hay không bằng cách sử dụng lệnh `@hasSection`:

    @hasSection('navigation')
        <div class="pull-right">
            @yield('navigation')
        </div>

        <div class="clearfix"></div>
    @endif

<a name="switch-statements"></a>
### Lệnh switch

Các câu lệnh switch có thể được xây dựng bằng cách sử dụng các lệnh `@switch`, `@case`, `@break`, `@default` và `@endswitch`:

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

<a name="loops"></a>
### Loops

Ngoài các câu lệnh có điều kiện, Blade cung cấp các lệnh đơn giản để làm việc với các cấu trúc vòng lặp của PHP. Một lần nữa, các lệnh này hoạt động giống hệt với các hàm PHP tương ứng của chúng:

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

> {tip} Khi lặp, bạn có thể sử dụng [loop variable](#the-loop-variable) để thu được thông tin có giá trị về vòng lặp, chẳng hạn như bạn đang ở vòng lặp đầu tiên hoặc vòng lặp cuối cùng.

Khi sử dụng các vòng lặp, bạn cũng có thể kết thúc vòng lặp hoặc bỏ qua vòng lặp hiện tại:

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

Bạn cũng có thể thêm điều kiện với khai báo lệnh trong một dòng:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### Biến loop

Khi lặp, một biến `$loop` sẽ có sẵn bên trong vòng lặp của bạn. Biến này cung cấp quyền truy cập vào một số thông tin hữu ích như chỉ số vòng lặp hiện tại và liệu đây là lần lặp đầu tiên hay cuối cùng của vòng lặp:

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

Nếu bạn đang ở trong một vòng lặp lồng nhau, bạn có thể truy cập vào biến `$loop` của vòng lặp cha thông qua thuộc tính `parent`:

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach

Biến `$loop` cũng chứa nhiều thuộc tính hữu ích khác:

Property  | Description
------------- | -------------
`$loop->index`  |  The index of the current loop iteration (starts at 0).
`$loop->iteration`  |  The current loop iteration (starts at 1).
`$loop->remaining`  |  The iteration remaining in the loop.
`$loop->count`  |  The total number of items in the array being iterated.
`$loop->first`  |  Whether this is the first iteration through the loop.
`$loop->last`  |  Whether this is the last iteration through the loop.
`$loop->depth`  |  The nesting level of the current loop.
`$loop->parent`  |  When in a nested loop, the parent's loop variable.

<a name="comments"></a>
### Comments

Blade cũng cho phép bạn định nghĩa comment trong view của bạn. Tuy nhiên, không giống như comment HTML, comment Blade không được thêm vào trong HTML do application của bạn trả về:

    {{-- This comment will not be present in the rendered HTML --}}

<a name="php"></a>
### PHP

Trong một số trường hợp, thật hữu ích khi nhúng code PHP vào view của bạn. Bạn có thể sử dụng lệnh Blade `@php` để thực thi một khối lệnh PHP đơn giản trong template của bạn:

    @php
        //
    @endphp

> {tip} Mặc dù Blade cung cấp tính năng này, nhưng việc sử dụng nó thường xuyên có thể là một tín hiệu cho thấy bạn có quá nhiều logic được nhúng trong template của bạn.

<a name="including-sub-views"></a>
## Thêm Sub-Views

Lệnh `@include` của Blade cho phép bạn thêm một view Blade từ bên trong view khác. Tất cả các biến có sẵn cho view chính sẽ được cung cấp cho view được thêm:

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

Mặc dù view được thêm sẽ kế thừa tất cả dữ liệu có sẵn trong view chính, bạn cũng có thể chuyển một mảng dữ liệu bổ sung cho view được thêm:

    @include('view.name', ['some' => 'data'])

Tất nhiên, nếu bạn cố gắng `@include` một view không tồn tại, thì Laravel sẽ đưa ra lỗi. Nếu bạn muốn thêm một view có thể có hoặc không tồn tại, thì bạn nên sử dụng lệnh `@includeIf`:

    @includeIf('view.name', ['some' => 'data'])

Nếu bạn muốn `@include` một view tùy thuộc vào một điều kiện boolean nhất định, bạn có thể sử dụng lệnh `@includeWhen`:

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

Để thêm view đầu tiên tồn tại từ list view nhất định, bạn có thể sử dụng lệnh `includeFirst`:

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])

> {note} Bạn nên tránh sử dụng các hằng số `__DIR__` và `__FILE__` trong view Blade của bạn, vì chúng sẽ dẫn đến vị trí của view được cache và compile.

<a name="rendering-views-for-collections"></a>
### Tạo Views cho Collections

Bạn có thể kết hợp các vòng lặp và các include thành một dòng với lệnh `@each` của Blade:

    @each('view.name', $jobs, 'job')

Tham số đầu tiên là view con để hiển thị cho từng thành phần trong mảng hoặc collection. Tham số thứ hai là mảng hoặc collection bạn muốn lặp, trong khi tham số thứ ba là tên biến sẽ được gán cho lần lặp hiện tại trong view. Vậy, ví dụ, nếu như bạn đang lặp một mảng `jobs`, thông thường bạn sẽ muốn truy cập từng job dưới dạng một biến `job` trong view con. Key cho vòng lặp hiện tại sẽ có sẵn dưới dạng biến `key` trong view con của bạn.

Bạn cũng có thể pass một tham số thứ tư cho lệnh `@each`. Tham số này xác định view sẽ được hiển thị nếu mảng đó là trống.

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} View được hiển thị qua `@each` không kế thừa các biến từ view parent. Nếu view con muốn các biến này, bạn nên sử dụng `@foreach` và `@include` thay thế.

<a name="stacks"></a>
## Stacks

Blade cho phép bạn push đến các stack có tên có thể được hiển thị ở một nơi khác trong view hoặc layout khác. Điều này có thể đặc biệt hữu ích để định nghĩa bất kỳ thư viện JavaScript nào theo yêu cầu của view con của bạn:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

Bạn có thể push đến một stack nhiều lần nếu cần. Để hiển thị nội dung stack hoàn chỉnh, pass tên của stack vào lệnh `@stack`:

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## Service Injection

Lệnh `@inject` có thể được sử dụng để lấy một service từ [service container](/docs/{{version}}/container) của Laravel. Tham số đầu tiên được pass cho `@inject` là tên của biến mà service sẽ được đặt vào, trong khi tham số thứ hai là tên class hoặc tên interface của service bạn muốn resolve:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## Mở rộng blade

Blade cho phép bạn định nghĩa các lệnh tùy biến của riêng bạn bằng phương thức `directive`. Khi trình biên dịch Blade gặp lệnh tùy biến, nó sẽ gọi hàm callback được khai báo với biểu thức mà lệnh này chứa.

Ví dụ sau đây sẽ tạo ra một lệnh `@datetime($var)` sẽ format lại một biến `$var` đã cho, và biến này phải là một instance của `DateTime`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
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

Như bạn có thể thấy, chúng ta sẽ nối phương thức `format` vào bất kỳ biểu thức nào, được pass vào lệnh. Vì vậy, trong ví dụ này, code PHP cuối cùng được tạo bởi lệnh này sẽ là:

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} Sau khi cập nhật logic của lệnh Blade, bạn sẽ cần xóa tất cả các view Blade đã được lưu trong bộ nhớ cache. Các view Blade được lưu trong bộ nhớ cache có thể được loại xoá bằng lệnh Artisan `view:clear`.

<a name="custom-if-statements"></a>
### Tuỳ biến lệnh if

Lập trình một lệnh tùy biến đôi khi lại phức tạp hơn mức cần thiết khi định nghĩa các câu lệnh điều kiện tùy biến đơn giản. Vì lý do đó, Blade cung cấp phương thức `Blade::if` cho phép bạn nhanh chóng định nghĩa các lệnh có điều kiện tùy biến bằng cách sử dụng Closures. Ví dụ: hãy định nghĩa một điều kiện tùy biến kiểm tra môi trường application hiện tại. Chúng ta có thể làm điều này trong phương thức `boot` của `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

Khi điều kiện tùy biến đã được định nghĩa xong, chúng ta có thể dễ dàng sử dụng nó trên các template của bạn:

    @env('local')
        // The application is in the local environment...
    @elseenv('testing')
        // The application is in the testing environment...
    @else
        // The application is not in the local or testing environment...
    @endenv
