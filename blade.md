# Blade Templates

- [Giới thiệu](#introduction)
- [Kế thừa template](#template-inheritance)
    - [Định nghĩa một layout](#defining-a-layout)
    - [Kế thừa một Layout](#extending-a-layout)
- [Hiển thị dữ liệu](#displaying-data)
    - [Blade và JavaScript Frameworks](#blade-and-javascript-frameworks)
- [Control Structures](#control-structures)
    - [Lệnh if](#if-statements)
    - [Lệnh switch](#switch-statements)
    - [Loops](#loops)
    - [Biến loop](#the-loop-variable)
    - [Comments](#comments)
    - [PHP](#php)
    - [Lệnh `@once`](#the-once-directive)
- [Forms](#forms)
    - [CSRF Field](#csrf-field)
    - [Method Field](#method-field)
    - [Validation Errors](#validation-errors)
- [Components](#components)
    - [Hiển thị Components](#displaying-components)
    - [Truyền dữ liệu tới Components](#passing-data-to-components)
    - [Quản lý Attributes](#managing-attributes)
    - [Slots](#slots)
    - [Inline Component Views](#inline-component-views)
    - [Component ẩn](#anonymous-components)
- [Thêm Subviews](#including-subviews)
    - [Tạo Views cho Collections](#rendering-views-for-collections)
- [Stacks](#stacks)
- [Service Injection](#service-injection)
- [Mở rộng blade](#extending-blade)
    - [Tuỳ biến lệnh if](#custom-if-statements)

<a name="introduction"></a>
## Giới thiệu

Blade là một công cụ tạo template đơn giản nhưng mạnh mẽ được đi kèm cùng với Laravel. Không giống như các công cụ tạo template phổ biến khác của PHP, Blade không hạn chế bạn sử dụng code PHP trong view của bạn. Trên thực tế, tất cả các view Blade được biên dịch thành code PHP và được lưu vào trong cache, cho đến khi chúng được sửa đổi, có nghĩa là về cơ bản Blade không làm tăng chi phí chung cho application của bạn. Các file Blade sử dụng phần đuôi mở rộng là `.blade.php` và thường được lưu trữ trong thư mục `resources/views`.

<a name="template-inheritance"></a>
## Kế thừa template

<a name="defining-a-layout"></a>
### Định nghĩa một layout

Hai trong số những lợi ích chính của việc sử dụng Blade là _kế thừa template_ và _sections_. Để bắt đầu, chúng ta hãy xem một ví dụ đơn giản. Đầu tiên, chúng ta hãy xem thử một layout trang "master". Vì hầu hết tất cả các trang web đều cố gắng duy trì một layout chung cho tất cả các trang khác nhau, nên Blade rất tiện cho việc định nghĩa các loại layout này chỉ trong một file view Blade duy nhất:

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

Như bạn có thể thấy, file này chứa code HTML. Tuy nhiên, hãy lưu ý các lệnh `@section` và `@yield`. Lệnh `@section`, như cái tên của nó, nó dùng để định nghĩa một phần của nội dung, trong khi lệnh `@yield` được sử dụng để hiển thị nội dung của một phần nhất định.

Vậy chúng ta đã định nghĩa xong một layout cho application của bạn, bây giờ hãy bắt đầu bằng một định nghĩa của một trang layout con kế thừa từ layout ở trên.

<a name="extending-a-layout"></a>
### Kế thừa một Layout

Khi định nghĩa một view con, bạn hãy sử dụng lệnh `@extends` của Blade để chỉ định layout nào mà view con đó sẽ được "kế thừa". Các view mà được mở rộng từ một layout Blade có thể đưa thêm nội dung vào các section của layout bằng cách sử dụng các lệnh `@section`. Hãy nhớ rằng, như ví dụ ở trên, nội dung của các section này sẽ được hiển thị trong layout bằng cách sử dụng `@yield`:

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

Trong ví dụ này, section `sidebar` đang sử dụng lệnh `@@parent` để nối thêm vào (chứ không phải ghi đè) sidebar của layout. Lệnh `@@parent` sẽ được thay thế bằng nội dung của layout khi view được hiển thị.

> {tip} Trái ngược với ví dụ trước đó, section `sidebar` này kết thúc bằng `@endsection` thay vì `@show`. Lệnh `@endsection` sẽ định nghĩa kết thúc một section trong khi `@show` cũng sẽ định nghĩa kết thúc một section nhưng nó cũng định nghĩa thêm một lệnh `@yield` để cho layout con để có thể định nghĩa thêm nội dung vào layout chính.

Lệnh `@yield` cũng chấp nhận một giá trị mặc định làm tham số thứ hai của nó. Giá trị này sẽ được hiển thị nếu section đang được tạo là undefined:

    @yield('content', View::make('view.name'))

Blade view có thể được trả về từ route khi dùng với global helper `view`:

    Route::get('blade', function () {
        return view('child');
    });

<a name="displaying-data"></a>
## Hiển thị dữ liệu

Bạn có thể hiển thị dữ liệu đã được truyền đến Blade view của bạn bằng cách đặt tên biến vào trong hai lần dấu ngoặc nhọn. Ví dụ: một route như sau:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

Bạn có thể hiển thị nội dung của biến `name` như thế này:

    Hello, {{ $name }}.

> {tip} Các câu lệnh Blade `{{ }}` này sẽ được tự động gửi qua hàm `htmlspecialchars` của PHP để ngăn chặn các cuộc tấn công XSS.

Bạn không bị giới hạn trong việc hiển thị nội dung của các biến đã được truyền đến view. Bạn cũng có thể echo ra kết quả với bất kỳ hàm PHP nào tương tự. Thực tế, bạn có thể set bất kỳ code PHP nào mà bạn muốn vào trong lệnh echo của Blade:

    The current UNIX timestamp is {{ time() }}.

#### Hiển thị dữ liệu unescaped

Mặc định, các câu lệnh Blade `{{ }}` này sẽ được tự động gửi qua hàm `htmlspecialchars` của PHP để ngăn chặn các cuộc tấn công XSS. Nếu bạn không muốn dữ liệu của bạn được escaped, bạn có thể sử dụng cú pháp sau:

    Hello, {!! $name !!}.

> {note} Bạn hãy cẩn thận khi hiển thị một nội dung mà được cung cấp bởi người dùng. Hãy luôn sử dụng escaped với cú pháp hai lần dấu ngoặc nhọn để ngăn chặn các cuộc tấn công XSS khi hiển thị dữ liệu do người dùng cung cấp.

#### Tạo JSON

Thỉnh thoảng bạn có thể muốn truyền một mảng vào view của bạn với ý định là hiển thị nó dưới dạng một chuỗi JSON để khởi tạo một biến JavaScript. Ví dụ:

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

Tuy nhiên, thay vì gọi thủ công `json_encode`, bạn có thể sử dụng lệnh Blade `@json`. Lệnh `@json` chấp nhận các tham số giống như hàm `json_encode` của PHP:

    <script>
        var app = @json($array);

        var app = @json($array, JSON_PRETTY_PRINT);
    </script>

> {note} Bạn chỉ nên sử dụng lệnh `@json` để hiển thị các biến hiện có dưới dạng JSON. Template cho Blade được dựa trên các biểu thức chính quy và việc cố gắng truyền vào một biểu thức phức tạp cho lệnh có thể gây ra lỗi mà bạn không mong muốn.

#### Mã hóa thực thể HTML

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

<a name="blade-and-javascript-frameworks"></a>
### Blade và JavaScript Frameworks

Do nhiều framework JavaScript cũng sử dụng hai lần dấu ngoặc nhọn để hiển thị dữ liệu trong trình duyệt, nên bạn có thể sử dụng ký hiệu `@` để thông báo cho Blade rendering engine là biểu thức này sẽ không được laravel rendering. Ví dụ:

    <h1>Laravel</h1>

    Hello, @{{ name }}.

Trong ví dụ này, ký hiệu `@` sẽ bị xóa bởi Blade; tuy nhiên, biểu thức `{{ name }}` sẽ vẫn còn và sẽ không được xử lý bởi Blade engine, điều này cho phép các biểu thức đó sẽ được render bởi framework JavaScript của bạn.

Các symbol `@` cũng có thể được sử dụng cho các lệnh Blade:

    {{-- Blade --}}
    @@json()

    <!-- HTML output -->
    @json()

#### Lệnh `@verbatim`

Nếu trong template của bạn đang hiển thị nhiều biến JavaScript, bạn có thể bao bọc các lệnh đó trong lệnh `@verbatim` để bạn không phải đặt tiền tố cho mỗi câu lệnh echo Blade bằng ký hiệu `@`:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## Control Structures

Ngoài việc kế thừa template và hiển thị dữ liệu, Blade cũng cung cấp các shortcut cho các cấu trúc điều khiển PHP phổ biến, chẳng hạn như các câu lệnh và các vòng lặp có điều kiện. Các shortcut này cung cấp một cách làm việc rất ngắn gọn, gọn gàng với các cấu trúc điều khiển PHP, trong khi vẫn quen thuộc với các hàm tương tự trong PHP.

<a name="if-statements"></a>
### Lệnh if

Bạn có thể xây dựng các câu lệnh `if` bằng cách sử dụng các lệnh `@if`, `@elseif`, `@else`, và `@endif`. Các lệnh này hoạt động giống hệt với các hàm tương tự trong PHP:

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

Ngoài các lệnh có điều kiện đã được thảo luận ở trên, các lệnh `@isset` và `@empty` cũng có thể được sử dụng làm các shortcut cho các hàm PHP tương ứng của chúng:

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### Lệnh authentication

Các lệnh `@auth` và `@guest` có thể được sử dụng để xác định xem người dùng hiện tại đã được xác thực chưa hay là khách:

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

Nếu cần, bạn có thể chỉ định thêm [authentication guard](/docs/{{version}}/authentication) cần được kiểm tra khi sử dụng các lệnh `@auth` và `@guest`:

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

Bạn có thể sử dụng lệnh `sectionMissing` để xác định một section không có nội dung:

    @sectionMissing('navigation')
        <div class="pull-right">
            @include('default-navigation')
        </div>
    @endif

#### Environment Directives

Bạn có thể kiểm tra xem ứng dụng của bạn có đang chạy trong môi trường production hay không bằng cách sử dụng lệnh `@production`:

    @production
        // Production specific content...
    @endproduction

Hoặc, bạn có thể xác định xem ứng dụng của bạn có đang chạy trong một môi trường cụ thể hay không bằng cách sử dụng lệnh `@env`:

    @env('staging')
        // The application is running in "staging"...
    @endenv

    @env(['staging', 'production'])
        // The application is running in "staging" or "production"...
    @endenv

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

> {tip} Khi lặp, bạn có thể sử dụng [loop variable](#the-loop-variable) để nhận về các thông tin có giá trị về vòng lặp, chẳng hạn như bạn đang ở vòng lặp đầu tiên hoặc vòng lặp cuối cùng.

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

Bạn cũng có thể thêm các điều kiện và khai báo lệnh đó trong một dòng:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### Biến loop

Khi lặp, một biến `$loop` sẽ có sẵn bên trong vòng lặp của bạn. Biến này cung cấp quyền truy cập vào một số thông tin hữu ích như vòng lặp hiện tại và liệu đây có phải là vòng lặp đầu tiên hay là vòng lặp cuối cùng:

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
`$loop->index`  |  Index của vòng lặp hiện tại (bắt đầu từ 0).
`$loop->iteration`  |  Vòng lặp hiện tại (bắt đầu từ 1).
`$loop->remaining`  |  Các lần lặp còn lại trong vòng lặp.
`$loop->count`  |  Tổng số item trong mảng đang được lặp lại.
`$loop->first`  |  Đây có phải là lần lặp đầu tiên của vòng lặp hay không.
`$loop->last`  |  Đây có phải là lần lặp cuối cùng của vòng lặp hay không.
`$loop->even`  |  Đây có phải là lần lặp chẵn của vòng lặp hay không.
`$loop->odd`  |  Đây có phải là lần lặp lẻ của vòng lặp hay không.
`$loop->depth`  |  Mức lồng của vòng lặp hiện tại.
`$loop->parent`  |  Khi ở trong một vòng lặp lồng nhau, biến này là biến của vòng lặp ngoài.

<a name="comments"></a>
### Comments

Blade cũng cho phép bạn định nghĩa các comment trong view của bạn. Tuy nhiên, không giống như comment trong HTML, comment trong Blade sẽ không được hiển thị vào trong HTML do application của bạn trả về:

    {{-- This comment will not be present in the rendered HTML --}}

<a name="php"></a>
### PHP

Trong một số trường hợp, có thể bạn cần nhúng code PHP vào trong view của bạn. Bạn có thể sử dụng lệnh Blade `@php` để thực thi một khối lệnh PHP đơn giản trong template của bạn:

    @php
        //
    @endphp

> {tip} Mặc dù Blade cung cấp tính năng này, nhưng việc sử dụng nó thường xuyên có thể là một tín hiệu cho thấy bạn đang có quá nhiều logic đang được nhúng vào trong template của bạn.

<a name="the-once-directive"></a>
### Lệnh `@once`

Lệnh `@once` cho phép bạn định nghĩa một phần của template sẽ chỉ được kiểm tra một lần duy nhất cho mỗi chu kỳ hiển thị. Điều này có thể hữu ích khi đưa một đoạn code JavaScript nhất định vào tiêu đề của trang web bằng cách sử dụng [stacks](#stacks). Ví dụ: nếu bạn đang hiển thị một [component](#components) trong một vòng lặp, bạn có thể chỉ muốn đưa một đoạn code JavaScript vào tiêu đề trong lần hiển thị đầu tiên của component:

    @once
        @push('scripts')
            <script>
                // Your custom JavaScript...
            </script>
        @endpush
    @endonce

<a name="forms"></a>
## Forms

<a name="csrf-field"></a>
### CSRF Field

Bất cứ khi nào bạn định nghĩa một HTML form trong ứng dụng của bạn, bạn nên thêm một trường hidden CSRF token vào trong form của bạn để middleware [bảo vệ CSRF](https://laravel.com/docs/{{version}}/csrf) có thể xác thực request đó. Bạn có thể sử dụng lệnh Blade `@csrf` để tạo trường token đó:

    <form method="POST" action="/profile">
        @csrf

        ...
    </form>

<a name="method-field"></a>
### Method Field

Vì các HTML form không thể tạo các request `PUT`, `PATCH` hoặc `DELETE`, nên bạn sẽ cần thêm trường `_method` hidden vào để làm giả các hành động cho các HTTP này. Lệnh Blade `@method` có thể tạo ra trường như vậy cho bạn:

    <form action="/foo/bar" method="POST">
        @method('PUT')

        ...
    </form>

<a name="validation-errors"></a>
### Validation Errors

Lệnh `@error` có thể được sử dụng để nhanh chóng kiểm tra xem trong [thông báo lỗi validation](/docs/{{version}}/validation#quick-displaying-the-validation-errors) có tồn tại lỗi cho một thuộc tính hay không. Trong lệnh `@error` bạn có thể lặp lại biến `$message` để hiển thị thông báo lỗi:

    <!-- /resources/views/post/create.blade.php -->

    <label for="title">Post Title</label>

    <input id="title" type="text" class="@error('title') is-invalid @enderror">

    @error('title')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

Bạn có thể truyền [tên của một error bag cụ thể](/docs/{{version}}/validation#named-error-bags) làm tham số thứ hai cho lệnh `@error` để lấy ra thông báo lỗi validation trên các trang chứa nhiều form:

    <!-- /resources/views/auth.blade.php -->

    <label for="email">Email address</label>

    <input id="email" type="email" class="@error('email', 'login') is-invalid @enderror">

    @error('email', 'login')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

<a name="components"></a>
## Components

Các component và slot sẽ cung cấp các lợi ích tương tự cho các section và layout; tuy nhiên, một số có thể thấy model của các component và slot sẽ dễ hiểu hơn. Có hai cách tiếp cận để viết các component: các component dựa trên class và các component ẩn.

Để tạo một component dựa trên class, bạn có thể sử dụng lệnh Artisan `make:component`. Để minh họa cách sử dụng của các component này, chúng ta sẽ tạo một component `Alert` đơn giản. Lệnh `make:component` sẽ lưu component vào trong thư mục `App\View\Components`:

    php artisan make:component Alert

Lệnh `make:component` cũng sẽ tạo một view template cho component. View sẽ được đặt trong thư mục `resources/views/components`.

#### Manually Registering Package Components

Khi viết các component cho ứng dụng của bạn, các component sẽ tự động được đăng ký trong thư mục `app/View/Components` và thư mục `resources/views/components`.

Tuy nhiên, nếu bạn đang xây dựng một package sử dụng các component Blade, bạn sẽ cần phải đăng ký thủ công các class component của bạn và các bí danh tag HTML của nó. Thông thường, bạn nên đăng ký các component của bạn trong phương thức `boot` của service provider trong package của bạn:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot()
    {
        Blade::component('package-alert', AlertComponent::class);
    }

Khi component của bạn đã được đăng ký, nó có thể được hiển thị bằng bí danh tag của nó:

    <x-package-alert/>

<a name="displaying-components"></a>
### Hiển thị Components

Để hiển thị một component, bạn có thể sử dụng tag component Blade trong một template Blade của bạn. Các tag component Blade bắt đầu bằng chuỗi `x-` theo sau là tên kebab case của class component:

    <x-alert/>

    <x-user-profile/>

Nếu class component được lồng sâu hơn trong thư mục `pp\View\Components`, bạn có thể sử dụng ký tự `.` để biểu thị sự lồng thư mục. Ví dụ: nếu chúng ta giả sử là một component được lưu tại `App\View\Components\Inputs\Button.php`, thì chúng ta có thể hiển thị nó như sau:

    <x-inputs.button/>

<a name="passing-data-to-components"></a>
### Truyền dữ liệu tới Components

Bạn có thể truyền dữ liệu đến các component Blade bằng cách sử dụng các thuộc tính HTML. Các giá trị theo kiểu nguyên thủy hoặc hard code có thể được truyền đến component bằng các thuộc tính HTML đơn giản. Các biểu thức hoặc biến PHP phải được truyền đến component thông qua các thuộc tính có tiền tố là `:`:

    <x-alert type="error" :message="$message"/>

Bạn nên định nghĩa các dữ liệu cần thiết của component trong phương thức khởi tạo class của nó. Tất cả các thuộc tính public trong một component sẽ được tự động truyền view của component. Không cần thiết phải truyền dữ liệu vào view từ phương thức `render` của component:

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

    <div class="alert alert-{{ $type }}">
        {{ $message }}
    </div>

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

Tham số `$alertType` có thể được truyền giá trị vào như sau:

    <x-alert alert-type="danger" />

#### Component Methods

Ngoài các biến public có sẵn trong component template của bạn, bất kỳ phương thức public nào có trong component cũng có thể được thực thi. Ví dụ: hãy tưởng tượng một component có phương thức `isSelected`:

    /**
     * Determine if the given option is the current selected option.
     *
     * @param  string  $option
     * @return bool
     */
    public function isSelected($option)
    {
        return $option === $this->selected;
    }

Bạn có thể thực thi phương thức này từ trong component template của bạn bằng cách gọi một biến khớp với tên của phương thức đó:

    <option {{ $isSelected($value) ? 'selected="selected"' : '' }} value="{{ $value }}">
        {{ $label }}
    </option>

#### Using Attributes & Slots Inside The Class

Các Blade component cũng cho phép bạn truy cập vào tên component, thuộc tính và slot bên trong phương thức render của class. Tuy nhiên, để truy cập vào các dữ liệu này, bạn nên trả về một Closure từ phương thức `render` của component của bạn. Closure sẽ nhận vào một mảng `$data` làm tham số duy nhất của nó:

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

            return '<div>Component content</div>';
        };
    }

Tên `componentName` sẽ là tên được sử dụng trong thẻ HTML sau tiền tố `x-`. Vì vậy, `componentName` của `<x-alert />` sẽ là `alert`. Phần tử `attributes` sẽ chứa tất cả các thuộc tính có trong thẻ HTML. Phần tử `slot` là một instance `Illuminate\Support\HtmlString` với nội dung là của slot từ component.

#### Additional Dependencies

Nếu component của bạn yêu cầu các component khác từ [service container](/docs/{{version}}/container) của Laravel, bạn có thể liệt kê chúng lên trước các thuộc tính dữ liệu được truyền vào của component và chúng sẽ tự động được container đưa vào:

    use App\AlertCreator

    /**
     * Create the component instance.
     *
     * @param  \App\AlertCreator  $creator
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

<a name="managing-attributes"></a>
### Quản lý Attributes

Chúng ta đã xem cách truyền các thuộc tính dữ liệu cho một component; tuy nhiên, đôi khi bạn có thể cần chỉ định thêm các thuộc tính HTML, chẳng hạn như `class`, không phải là một dữ liệu cần thiết để một component hoạt động. Thông thường, bạn sẽ muốn truyền các thuộc tính bổ sung đó vào phần tử gốc của component template. Ví dụ: hãy tưởng tượng chúng ta muốn hiển thị một component `alert` như sau:

    <x-alert type="error" :message="$message" class="mt-4"/>

Tất cả các thuộc tính mà không nằm trong phương thức khởi tạo của component sẽ tự động được thêm vào "attribute bag" của component. Attribute bag này được tạo tự động cho component thông qua biến `$attributes`. Tất cả các thuộc tính có thể được hiển thị trong component bằng cách echo biến này:

    <div {{ $attributes }}>
        <!-- Component Content -->
    </div>

> {note} Việc echo các biến (`{{ $attributes }}`) hoặc sử dụng các lệnh trực tiếp như `@env` trên một component hiện không được hỗ trợ.

#### Default / Merged Attributes

Thỉnh thoảng bạn có thể cần chỉ định các giá trị mặc định cho các thuộc tính hoặc merge thêm các giá trị vào một số thuộc tính của component. Để thực hiện điều này, bạn có thể sử dụng phương thức `merge` của attribute bag:

    <div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
        {{ $message }}
    </div>

Nếu chúng ta giả sử rằng component này được sử dụng như sau:

    <x-alert type="error" :message="$message" class="mb-4"/>

Cuối cùng, HTML mà được tạo ra của component sẽ xuất hiện như thế này:

    <div class="alert alert-error mb-4">
        <!-- Contents of the $message variable -->
    </div>

#### Filtering Attributes

Bạn có thể lọc các thuộc tính bằng phương thức `filter`. Phương thức này chấp nhận một Closure sẽ trả về giá trị `true` nếu bạn muốn giữ lại các thuộc tính trong attribute bag:

    {{ $attributes->filter(fn ($value, $key) => $key == 'foo') }}

Để thuận tiện, bạn có thể sử dụng phương thức `whereStartsWith` để lấy ra tất cả các thuộc tính có khóa bắt đầu bằng một chuỗi đã cho:

    {{ $attributes->whereStartsWith('wire:model') }}

Sử dụng phương thức `first`, bạn có thể hiển thị thuộc tính đầu tiên có trong một attribute bag nhất định:

    {{ $attributes->whereStartsWith('wire:model')->first() }}

<a name="slots"></a>
### Slots

Thông thường, bạn sẽ cần truyền thêm nội dung vào component của bạn thông qua "slots". Hãy tưởng tượng rằng một component `alert` mà chúng ta đã tạo có định dạng như sau:

    <!-- /resources/views/components/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

Chúng ta có thể truyền nội dung vào `slot` bằng cách đưa nội dung vào trong component:

    <x-alert>
        <strong>Whoops!</strong> Something went wrong!
    </x-alert>

Thỉnh thoảng một component có thể cần hiển thị nhiều slot khác nhau ở các vị trí khác nhau trong component. Hãy thử sửa alert component của chúng ta để cho phép chèn một "title" vào component:

    <!-- /resources/views/components/alert.blade.php -->

    <span class="alert-title">{{ $title }}</span>

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

Bạn có thể định nghĩa nội dung của một slot cụ thể bằng tag `x-slot`. Mọi nội dung không nằm trong tag `x-slot` sẽ được truyền đến component thông qua biến `$slot`:

    <x-alert>
        <x-slot name="title">
            Server Error
        </x-slot>

        <strong>Whoops!</strong> Something went wrong!
    </x-alert>

#### Scoped Slots

Nếu bạn đã sử dụng một JavaScript framework như Vue, bạn có thể quen thuộc với các "scoped slot", cho phép bạn truy cập vào dữ liệu hoặc phương thức của component trong slot của bạn. Bạn cũng có thể làm ra hành vi tương tự đó trong Laravel bằng cách định nghĩa các phương thức hoặc thuộc tính public trong component của bạn và truy cập vào component đó trong slot của bạn thông qua biến `$component`:

    <x-alert>
        <x-slot name="title">
            {{ $component->formatAlert('Server Error') }}
        </x-slot>

        <strong>Whoops!</strong> Something went wrong!
    </x-alert>

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

#### Generating Inline View Components

Để tạo một component theo dạng inline view, bạn có thể sử dụng tùy chọn `inline` khi chạy lệnh `make:component`:

    php artisan make:component Alert --inline

<a name="anonymous-components"></a>
### Component ẩn

Tương tự như các component inline, các component ẩn cũng cung cấp cơ chế quản lý một component thông qua một file duy nhất. Tuy nhiên, các component ẩn sẽ sử dụng một file view và không có class nào liên kết đến nó. Để định nghĩa một component ẩn, bạn chỉ cần set một template Blade vào trong thư mục `resources/views/components` của bạn. Ví dụ: giả sử bạn đã định nghĩa một component tại `resources/views/components/alert.blade.php`:

    <x-alert/>

Bạn có thể sử dụng ký tự `.` để cho biết một component được nằm sâu bên trong thư mục `component`. Ví dụ: giả sử component được định nghĩa tại `resources/views/components/inputs/button.blade.php`, bạn có thể hiển thị nó như sau:

    <x-inputs.button/>

#### Data Properties / Attributes

Vì các component ẩn này không có bất kỳ class nào liên kết đến với nó, và bạn có thể tự hỏi là làm cách nào để phân biệt được dữ liệu nào là được truyền vào cho component dưới dạng biến và thuộc tính nào sẽ được set vào trong [attribute bag](#managing-attributes) của component.

Bạn có thể chỉ định các thuộc tính sẽ được coi là biến dữ liệu bằng cách sử dụng lệnh `@props` ở đầu file template Blade của component của bạn. Tất cả các thuộc tính khác có trong component sẽ luôn có sẵn trong attribute bag của component. Nếu bạn muốn cung cấp cho một biến dữ liệu với một giá trị mặc định, bạn có thể chỉ định tên của biến đó làm khóa của mảng và giá trị mặc định đó sẽ là giá trị của mảng:

    <!-- /resources/views/components/alert.blade.php -->

    @props(['type' => 'info', 'message'])

    <div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
        {{ $message }}
    </div>

<a name="including-subviews"></a>
## Thêm Subviews

Lệnh `@include` của Blade cho phép bạn thêm một view Blade khác vào trong view hiện tại. Tất cả các biến đã có trong view hiện tại cũng sẽ có trong view được thêm:

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

Mặc dù view được thêm sẽ kế thừa tất cả các dữ liệu có sẵn trong view chính, nhưng bạn cũng có thể chuyển thêm một mảng dữ liệu bổ sung cho view được thêm:

    @include('view.name', ['some' => 'data'])

Nếu bạn thử `@include` một view không tồn tại, thì Laravel sẽ đưa ra một lỗi. Nếu bạn muốn thêm một view có thể có hoặc có thể không tồn tại, thì bạn nên sử dụng lệnh `@includeIf`:

    @includeIf('view.name', ['some' => 'data'])

Nếu bạn muốn `@include` một view nếu một biểu thức boolean trả về giá trị `true`, bạn có thể sử dụng lệnh `@includeWhen`:

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

Nếu bạn muốn `@include` một view nếu một biểu thức boolean trả về giá trị `false`, bạn có thể sử dụng lệnh `@includeUnless`:

    @includeUnless($boolean, 'view.name', ['some' => 'data'])

Để thêm view đầu tiên tồn tại từ một list view nhất định, bạn có thể sử dụng lệnh `includeFirst`:

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])

> {note} Bạn nên tránh sử dụng các hằng số `__DIR__` và `__FILE__` trong view Blade của bạn, vì chúng sẽ dẫn đến vị trí view sẽ được cache và compile.

#### Bí danh

Nếu các Blade con của bạn được lưu trữ trong một thư mục con, bạn có thể muốn đặt bí danh cho chúng để dễ dàng truy cập hơn. Ví dụ, hãy tưởng tượng một Blade con được lưu trữ tại `resources/views/includes/input.blade.php` với nội dung như sau:

    <input type="{{ $type ?? 'text' }}">

Bạn có thể sử dụng phương thức `include` để đặt bí danh cho Blade con từ `includes.input` thành `input`. Thông thường, điều này phải được thực hiện trong phương thức `boot` của `AppServiceProvider` của bạn:

    use Illuminate\Support\Facades\Blade;

    Blade::include('includes.input', 'input');

Sau khi Blade con đã được đặt bí danh, bạn có thể hiển thị nó bằng cách sử dụng tên bí danh như một lệnh Blade:

    @input(['type' => 'email'])

<a name="rendering-views-for-collections"></a>
### Tạo Views cho Collections

Bạn có thể kết hợp các vòng lặp và các include vào một dòng lệnh `@each` của Blade:

    @each('view.name', $jobs, 'job')

Tham số đầu tiên là tên view con để hiển thị cho từng phần tử trong mảng hoặc collection. Tham số thứ hai là mảng hoặc collection mà bạn muốn lặp, trong khi tham số thứ ba là tên biến sẽ được gán cho mỗi lần lặp trong view. Vậy, ví dụ, nếu như bạn đang lặp một mảng `jobs`, thông thường bạn sẽ muốn truy cập từng job dưới dạng tên một biến `job` trong view con. Key cho vòng lặp hiện tại sẽ có sẵn dưới dạng biến `key` trong view con của bạn.

Bạn cũng có thể truyền một tham số thứ tư cho lệnh `@each`. Tham số này định nghĩa view nào sẽ được hiển thị nếu mảng đó trống.

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} View được hiển thị qua `@each` sẽ không kế thừa các biến từ view cha. Nếu bạn muốn các view con của bạn có các biến này, bạn có thể sử dụng `@foreach` và `@include` thay thế.

<a name="stacks"></a>
## Stacks

Blade cho phép bạn khai báo thêm các file hoặc các biến vào trong các stack đã được tên, để có thể được hiển thị ở một nơi khác trong view hoặc layout. Điều này có thể đặc biệt hữu ích để định nghĩa một thư viện JavaScript nào đó theo yêu cầu của view con của bạn:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

Bạn có thể khai báo cho một stack nhiều lần nếu cần. Để hiển thị nội dung stack hoàn chỉnh, truyền tên của stack vào lệnh `@stack`:

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

Nếu bạn muốn thêm nội dung vào đầu một stack, bạn có thể sử dụng lệnh `@prepend`:

    @push('scripts')
        This will be second...
    @endpush

    // Later...

    @prepend('scripts')
        This will be first...
    @endprepend

<a name="service-injection"></a>
## Service Injection

Lệnh `@inject` có thể được sử dụng để lấy một service ra từ [service container](/docs/{{version}}/container). Tham số đầu tiên được truyền vào `@inject` là tên biến mà service sẽ được set, trong khi tham số thứ hai là tên class hoặc là tên một interface của service mà bạn muốn resolve:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

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

> {note} Sau khi cập nhật logic của lệnh Blade, bạn sẽ cần xóa tất cả các view Blade đã được lưu trong bộ nhớ cache. Các view Blade được lưu trong bộ nhớ cache có thể được loại bỏ bằng lệnh Artisan `view:clear`.

<a name="custom-if-statements"></a>
### Tuỳ biến lệnh if

Lập trình một lệnh tùy biến đôi khi lại là phức tạp hơn là định nghĩa một câu lệnh điều kiện tùy biến đơn giản. Vì lý do đó, Blade cung cấp phương thức `Blade::if` cho phép bạn nhanh chóng định nghĩa các lệnh tùy biến có điều kiện bằng cách sử dụng Closures. Ví dụ: hãy định nghĩa một điều kiện tùy biến có thể kiểm tra cloud provider hiện tại của application. Chúng ta có thể làm điều này trong phương thức `boot` của `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('cloud', function ($provider) {
            return config('filesystems.default') === $provider;
        });
    }

Khi điều kiện tùy biến đã được định nghĩa xong, chúng ta có thể dễ dàng sử dụng nó trên các template của bạn:

    @cloud('digitalocean')
        // The application is using the digitalocean cloud provider...
    @elsecloud('aws')
        // The application is using the aws provider...
    @else
        // The application is not using the digitalocean or aws environment...
    @endcloud

    @unlesscloud('aws')
        // The application is not using the aws environment...
    @endcloud
