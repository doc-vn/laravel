# Frontend

- [Giới thiệu](#introduction)
- [Dùng PHP](#using-php)
    - [PHP và Blade](#php-and-blade)
    - [Livewire](#livewire)
    - [Starter Kits](#php-starter-kits)
- [Dùng Vue hoặc React](#using-vue-react)
    - [Inertia](#inertia)
    - [Starter Kits](#inertia-starter-kits)
- [Đóng gói assets](#bundling-assets)

<a name="introduction"></a>
## Giới thiệu

Laravel là một framework backend cung cấp tất cả các tính năng mà bạn cần để xây dựng các ứng dụng web hiện đại, chẳng hạn như [routing](/docs/{{version}}/routing), [validation](/docs/{{version}}/validation), [caching](/docs/{{version}}/cache), [queues](/docs/{{version}}/queues), [file storage](/docs/{{version}}/filesystem), vv. Tuy nhiên, chúng tôi tin rằng điều quan trọng là cung cấp cho các nhà phát triển trải nghiệm full-stack tuyệt vời, bao gồm cả các phương pháp mạnh mẽ để xây dựng front-end cho ứng dụng.

Có hai cách chính để giải quyết phát triển front-end khi xây dựng ứng dụng bằng Laravel và cách mà bạn chọn sẽ được xác định bằng việc bạn muốn xây dựng front-end của bạn bằng cách dùng PHP hay bằng cách sử dụng các framework JavaScript như Vue và React. Chúng tôi sẽ thảo luận về cả hai cách này ở bên dưới để bạn có thể đưa ra quyết định sáng suốt về cách tiếp cận tốt nhất để phát triển front-end cho ứng dụng của bạn.

<a name="using-php"></a>
## Dùng PHP

<a name="php-and-blade"></a>
### PHP và Blade

Trước đây, hầu hết các ứng dụng PHP đều render ra HTML cho trình duyệt chạy là qua các HTML template đơn giản xen kẽ với các câu lệnh `echo` của PHP để render ra dữ liệu được lấy từ cơ sở dữ liệu trong quá trình request:

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

Trong Laravel, cách hiển thị HTML này vẫn có thể đạt được bằng cách sử dụng [views](/docs/{{version}}/views) và [Blade](/docs/{{version}}/blade). Blade là một ngôn ngữ template cực kỳ nhẹ cung cấp cú pháp ngắn gọn, tiện lợi để hiển thị dữ liệu, lặp lại dữ liệu và nhiều hơn nữa:

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

Khi xây dựng ứng dụng theo cách này, các submit form và các tương tác khác ở trên trang HTML thường nhận về một trang HTML document mới từ server và toàn bộ trang HTML sẽ được load lại từ đầu. Ngay cả ngày nay, nhiều ứng dụng vẫn có thể hoàn toàn phù hợp cách xây dựng giao diện người dùng này thông qua các template Blade đơn giản.

<a name="growing-expectations"></a>
#### Growing Expectations

Tuy nhiên, khi kỳ vọng của người dùng về các ứng dụng web ngày một lớn hơn, nhiều nhà phát triển cảm thấy cần phải xây dựng một giao diện người dùng năng động hơn với các tương tác, để có cảm giác được trau chuốt hơn. Theo quan điểm này, một số nhà phát triển chọn bắt đầu xây dựng giao diện người dùng của ứng dụng bằng các framework JavaScript như Vue và React.

Những người khác, thích gắn bó với những ngôn ngữ backend mà họ cảm thấy quen thuộc, đã phát triển các giải pháp cho phép xây dựng giao diện người dùng ứng dụng web hiện đại trong khi vẫn chủ yếu sử dụng ngôn ngữ backend mà họ lựa chọn. Ví dụ, trong hệ sinh thái [Rails](https://rubyonrails.org/), điều này đã thúc đẩy việc tạo ra các thư viện như [Turbo](https://turbo.hotwired.dev/) [Hotwire](https://hotwired.dev/) và [Stimulus](https://stimulus.hotwired.dev/).

Trong hệ sinh thái Laravel, nhu cầu tạo ra một giao diện người dùng hiện đại, năng động bằng cách sử dụng PHP chủ yếu đã dẫn đến việc tạo ra [Laravel Livewire](https://livewire.laravel.com) và [Alpine.js](https://alpinejs.dev/).

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://livewire.laravel.com) là một framework xây dựng giao diện người dùng được hỗ trợ bởi Laravel mang lại cảm giác năng động, hiện đại và sống động giống như các giao diện người dùng được xây dựng bằng các framework JavaScript hiện đại như Vue và React.

Khi sử dụng Livewire, bạn sẽ tạo ra các "components" Livewire để hiển thị một phần riêng biệt của UI và hiển thị các phương thức và dữ liệu có thể được gọi và tương tác từ giao diện người dùng của ứng dụng. Ví dụ, một component "Counter" đơn giản có thể trông như sau:

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

Và template tương ứng cho counter sẽ được viết như sau:

```blade
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

Như bạn có thể thấy, Livewire cho phép bạn viết các thuộc tính HTML mới như `wire:click` để kết nối frontend và backend của ứng dụng Laravel. Ngoài ra, bạn có thể hiển thị trạng thái hiện tại của component bằng các biểu thức Blade đơn giản.

Đối với nhiều người, Livewire đã cách mạng hóa quá trình phát triển front-end với Laravel, cho phép họ duy trì sự thoải mái của Laravel trong khi vẫn xây dựng được các ứng dụng web hiện đại, năng động. Thông thường, các nhà phát triển sử dụng Livewire cũng sẽ sử dụng [Alpine.js](https://alpinejs.dev/) để "rải" code JavaScript lên front-end của họ chỉ khi cần thiết, chẳng hạn như để hiển thị một dialog.

Nếu bạn mới làm quen với Laravel, chúng tôi khuyên bạn nên làm quen với cách sử dụng cơ bản của [views](/docs/{{version}}/views) và [Blade](/docs/{{version}}/blade). Sau đó, hãy tham khảo [tài liệu chính thức của Laravel Livewire](https://livewire.laravel.com) để tìm hiểu cách đưa ứng dụng của bạn lên một tầm cao mới với các tương tác component Livewire.

<a name="php-starter-kits"></a>
### Starter Kits

Nếu bạn muốn xây dựng frontend của bạn bằng PHP và Livewire, bạn có thể tận dụng [starter kits](/docs/{{version}}/starter-kits) Breeze hoặc Jetstream của chúng tôi để khởi tạo quá trình phát triển ứng dụng của bạn. Cả hai bộ khởi tạo này đều hỗ trợ các flow xác thực backend và frontend cho ứng dụng của bạn bằng [Blade](/docs/{{version}}/blade) và [Tailwind](https://tailwindcss.com) để bạn có thể dễ dàng bắt đầu xây dựng bước tiếp theo của bạn.

<a name="using-vue-react"></a>
## Dùng Vue hoặc React

Mặc dù có thể xây dựng frontend hiện đại bằng Laravel và Livewire, nhưng nhiều nhà phát triển vẫn thích tận dụng sức mạnh của một số framework JavaScript như Vue hoặc React. Điều này cho phép các nhà phát triển tận dụng hệ sinh thái phong phú của các package và công cụ JavaScript có sẵn thông qua NPM.

Tuy nhiên, nếu không có công cụ bổ sung, việc ghép Laravel với Vue hoặc React sẽ khiến chúng ta phải giải quyết nhiều vấn đề phức tạp như điều hướng bên client, tái tạo lại dữ liệu và xác thực. Điều hướng bên client thường được đơn giản hóa bằng cách sử dụng các framework Vue hoặc React như [Nuxt](https://nuxt.com/) và [Next](https://nextjs.org/); tuy nhiên, tái tạo lại dữ liệu và xác thực vẫn là những vấn đề phức tạp và cồng kềnh cần được giải quyết khi ghép một framework backend như Laravel với các framework giao diện người dùng.

Ngoài ra, các nhà phát triển phải duy trì hai repository code riêng, thường cần phải phối hợp bảo trì, phát hành và triển khai trên cả hai repository. Mặc dù những vấn đề này không phải là không thể vượt qua, nhưng chúng tôi không tin rằng đây là cách hiệu quả hoặc thú vị để phát triển ứng dụng.

<a name="inertia"></a>
### Inertia

Rất may, Laravel cũng cung cấp những điều tốt nhất cho cả hai thế giới. [Inertia](https://inertiajs.com) sẽ thu hẹp khoảng cách giữa ứng dụng Laravel của bạn và giao diện người dùng Vue hoặc React hiện đại của bạn, cho phép bạn xây dựng giao diện người dùng hiện đại, hoàn chỉnh bằng Vue hoặc React trong khi vẫn tận dụng các route và  các controller Laravel để điều hướng, cung cấp dữ liệu và xác thực — tất cả trong một repository code duy nhất. Với cách tiếp cận này, bạn có thể tận hưởng toàn bộ sức mạnh của cả Laravel và Vue / React mà không làm giảm khả năng của bất kỳ công cụ nào.

Sau khi cài đặt Inertia vào ứng dụng Laravel của bạn, bạn sẽ viết các route và controller như bình thường. Tuy nhiên, thay vì trả về một template Blade từ controller của bạn, bạn sẽ trả về một trang Inertia:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): Response
    {
        return Inertia::render('Users/Profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Trang Inertia tương ứng với một component Vue hoặc React, thường được lưu trữ trong thư mục `resources/js/Pages` của ứng dụng của bạn. Dữ liệu được cung cấp cho trang thông qua phương thức `Inertia::render` sẽ được sử dụng để tái tạo lại thành thuộc tính "props" trong component của trang:

```vue
<script setup>
import Layout from '@/Layouts/Authenticated.vue';
import { Head } from '@inertiajs/vue3';

const props = defineProps(['user']);
</script>

<template>
    <Head title="User Profile" />

    <Layout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Profile
            </h2>
        </template>

        <div class="py-12">
            Hello, {{ user.name }}
        </div>
    </Layout>
</template>
```

Như bạn có thể thấy, Inertia cho phép bạn tận dụng toàn bộ sức mạnh của Vue hoặc React khi xây dựng front-end, đồng thời cung cấp cầu nối nhẹ nhàng giữa back-end chạy bằng Laravel và front-end chạy bằng JavaScript.

#### Server-Side Rendering

Nếu bạn lo ngại về việc chuyển sang Inertia vì ứng dụng của bạn yêu cầu tạo trang html bên phía máy chủ, đừng lo lắng. Inertia cũng [hỗ trợ tạo trang từ phía máy chủ](https://inertiajs.com/server-side-rendering). Và khi deploy ứng dụng của bạn thông qua [Laravel Forge](https://forge.laravel.com), bạn có thể dễ dàng đảm bảo rằng quy trình tạo trang từ phía máy chủ của Inertia luôn được chạy.

<a name="inertia-starter-kits"></a>
### Starter Kits

Nếu bạn muốn xây dựng frontend của bạn bằng Inertia và Vue / React, bạn có thể tận dụng [bộ khởi tạo](/docs/{{version}}/starter-kits#breeze-and-inertia) Breeze hoặc Jetstream của chúng tôi để bắt đầu quá trình phát triển ứng dụng của bạn. Cả hai bộ khởi tạo này đều hỗ trợ flow xác thực cả về backend lẫn frontend cho ứng dụng bằng Inertia, Vue / React, [Tailwind](https://tailwindcss.com) và [Vite](https://vitejs.dev) để bạn có thể bắt đầu xây dựng bước tiếp theo của bạn.

<a name="bundling-assets"></a>
## Đóng gói assets

Bất kể bạn chọn phát triển frontend của bạn bằng Blade và Livewire hay Inertia và Vue hoặc React, bạn có thể sẽ cần phải đóng gói CSS của ứng dụng vào các asset sẵn sàng cho production. Tất nhiên, nếu bạn chọn xây dựng frontend của ứng dụng bằng Vue hoặc React, bạn cũng sẽ cần phải đóng gói các component của bạn vào các asset JavaScript sẵn sàng cho trình duyệt.

Mặc định, Laravel sử dụng [Vite](https://vitejs.dev) để đóng gói asset của bạn. Vite cung cấp thời gian build cực nhanh và Hot Module Replacement (HMR) gần như tức thời trong quá trình phát triển ở local. Trong tất cả các ứng dụng Laravel mới, và cả những ứng dụng sử dụng [bộ khởi tạo](/docs/{{version}}/starter-kits) của chúng tôi, bạn sẽ tìm thấy file `vite.config.js`, file này sẽ load plugin Laravel Vite nhẹ của chúng tôi giúp Vite trở nên thú vị khi sử dụng với các ứng dụng Laravel.

Cách nhanh nhất để bắt đầu với Laravel và Vite là bắt đầu phát triển ứng dụng của bạn bằng [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze), bộ công cụ khởi tạo đơn giản nhất của chúng tôi giúp bạn khởi tạo ứng dụng của mình bằng cách cung cấp nền tảng xác thực bằng cả frontend lẫn cả backend.

> [!NOTE]
> Để biết thêm tài liệu chi tiết về việc sử dụng Vite cùng Laravel, vui lòng xem thêm [tài liệu chuyên dụng về cách đóng gói và cách biên dịch asset của bạn](/docs/{{version}}/vite).
