# Starter Kits

- [Giới thiệu](#introduction)
- [Tạo một ứng dụng bằng Starter Kit](#creating-an-application)
- [Các Starter Kit có sẵn](#available-starter-kits)
    - [React](#react)
    - [Svelte](#svelte)
    - [Vue](#vue)
    - [Livewire](#livewire)
- [Tùy chỉnh Starter Kit](#starter-kit-customization)
    - [React](#react-customization)
    - [Svelte](#svelte-customization)
    - [Vue](#vue-customization)
    - [Livewire](#livewire-customization)
- [Authentication](#authentication)
    - [Enable và disable features](#enabling-and-disabling-features)
    - [Tùy chỉnh User Creation và Password Reset](#customizing-actions)
    - [Two-Factor Authentication](#two-factor-authentication)
    - [Rate Limiting](#rate-limiting)
- [Teams](#teams)
- [WorkOS AuthKit Authentication](#workos)
- [Inertia SSR](#inertia-ssr)
- [Các bộ Starter Kit của cộng đồng](#community-maintained-starter-kits)
- [Các câu hỏi thường gặp](#faqs)

<a name="introduction"></a>
## Giới thiệu

Để giúp bạn nhanh chóng bắt đầu xây dựng ứng dụng Laravel mới, chúng tôi rất vui khi cung cấp các [bộ công cụ khởi tạo](https://laravel.com/starter-kits). Những bộ công cụ này mang lại cho bạn một khởi tạo thuận lợi để xây dựng ứng dụng Laravel tiếp theo của bạn, nó bao gồm các route, controller và view mà bạn cần để đăng ký và xác thực người dùng. Các bộ công cụ khởi tạo sử dụng [Laravel Fortify](/docs/{{version}}/fortify) để cung cấp tính năng xác thực.

Mặc dù bạn có thể thoải mái sử dụng những bộ công cụ khởi tạo này nhưng chúng không bắt buộc. Bạn có thể tự do xây dựng ứng dụng của riêng bạn ngay từ đầu bằng cách cài đặt một bản mới của Laravel. Dù bằng cách nào, chúng tôi biết bạn sẽ xây dựng được điều gì đó rất tuyệt vời!

<a name="creating-an-application"></a>
## Tạo một ứng dụng bằng Starter Kit

Để tạo một ứng dụng Laravel mới bằng một trong những bộ công cụ khởi tạo của chúng tôi, trước tiên bạn nên [cài đặt PHP và công cụ CLI của Laravel](/docs/{{version}}/installation#installing-php). Nếu bạn đã cài sẵn PHP và Composer, bạn có thể cài đặt công cụ Laravel installer CLI thông qua Composer:

```shell
composer global require laravel/installer
```

Sau đó, hãy tạo một ứng dụng Laravel mới bằng công cụ Laravel installer CLI. Laravel installer sẽ hỏi bạn chọn bộ công cụ khởi tạo mà bạn thích:

```shell
laravel new my-app
```

Sau khi tạo ứng dụng Laravel của bạn, bạn chỉ cần cài đặt các library frontend thông qua NPM và khởi chạy Laravel development server:

```shell
cd my-app
npm install && npm run build
composer run dev
```

Sau khi đã khởi chạy Laravel development server, bạn có thể truy cập ứng dụng của bạn thông qua trình duyệt web tại địa chỉ [http://localhost:8000](http://localhost:8000).

<a name="available-starter-kits"></a>
## Các Starter Kit có sẵn

<a name="react"></a>
### React

Bộ công cụ khởi tạo React của chúng tôi mang tới một khởi đầu mạnh mẽ và hiện đại để xây dựng các ứng dụng Laravel với frontend React thông qua [Inertia](https://inertiajs.com).

Inertia cho phép bạn xây dựng các ứng dụng React single page hiện đại bằng cách sử dụng routing và controller phía server truyền thống. Điều này giúp bạn tận hưởng sức mạnh frontend của React kết hợp với hiệu năng backend tuyệt vời của Laravel và tốc độ biên dịch cực nhanh của Vite.

Bộ công cụ khởi tạo React sử dụng React 19, TypeScript, Tailwind và thư viện [shadcn/ui](https://ui.shadcn.com).

<a name="svelte"></a>
### Svelte

Bộ công cụ khởi tạo Svelte của chúng tôi cung cấp một khởi đầu hiện đại và mạnh mẽ để xây dựng các ứng dụng Laravel với frontend Svelte thông qua [Inertia](https://inertiajs.com).

Inertia cho phép bạn xây dựng các ứng dụng Svelte single page hiện đại bằng cách sử dụng routing và controller phía server truyền thống. Điều này giúp bạn tận hưởng sức mạnh frontend của Svelte kết hợp với hiệu năng backend tuyệt vời của Laravel và tốc độ biên dịch cực nhanh của Vite.

Bộ công cụ khởi tạo Svelte sử dụng Svelte 5, TypeScript, Tailwind và thư viện component [shadcn-svelte](https://www.shadcn-svelte.com/).

<a name="vue"></a>
### Vue

Bộ công cụ khởi tạo Vue của chúng tôi mang tới một khởi đầu tuyệt vời để xây dựng các ứng dụng Laravel với frontend Vue thông qua [Inertia](https://inertiajs.com).

Inertia cho phép bạn xây dựng các ứng dụng Vue single page hiện đại bằng cách sử dụng routing và controller phía server truyền thống. Điều này giúp bạn tận hưởng sức mạnh frontend của Vue kết hợp với hiệu năng backend tuyệt vời của Laravel và tốc độ biên dịch cực nhanh của Vite.

Bộ công cụ khởi tạo Vue sử dụng Vue Composition API, TypeScript, Tailwind và thư viện [shadcn-vue](https://www.shadcn-vue.com/).

<a name="livewire"></a>
### Livewire

Bộ công cụ khởi tạo Livewire của chúng tôi cung cấp một khởi đầu hoàn hảo để xây dựng các ứng dụng Laravel với frontend [Laravel Livewire](https://livewire.laravel.com).

Livewire là một cách mạnh mẽ để xây dựng các giao diện frontend động và phản ứng linh hoạt chỉ bằng PHP. Nó cực kỳ phù hợp cho các đội ngũ chủ yếu sử dụng Blade template và đang tìm kiếm một giải pháp thay thế đơn giản hơn cho các framework SPA chạy bằng JavaScript như React, Svelte, và Vue.

Bộ công cụ khởi tạo Livewire sử dụng Livewire, Tailwind và thư viện [Flux UI](https://fluxui.dev).

<a name="starter-kit-customization"></a>
## Tùy chỉnh Starter Kit

<a name="react-customization"></a>
### React

Bộ công cụ khởi tạo React của chúng tôi được xây dựng bằng Inertia 3, React 19, Tailwind 4 và [shadcn/ui](https://ui.shadcn.com). Giống như tất cả các bộ công cụ khởi tạo khác của chúng tôi, toàn bộ code backend và frontend đều nằm trong ứng dụng của bạn để bạn có thể tùy chỉnh bất kỳ thứ gì bạn muốn.

Phần lớn code frontend nằm trong thư mục `resources/js`. Bạn có thể tự do chỉnh sửa bất kỳ đoạn code nào để tùy chỉnh giao diện và hành vi của ứng dụng:

```text
resources/js/
├── components/    # Reusable React components
├── hooks/         # React hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

Để export thêm các component shadcn, trước tiên hãy [tìm component mà bạn muốn sử dụng](https://ui.shadcn.com). Sau đó, hãy tiến hành bằng cách sử dụng lệnh `npx`:

```shell
npx shadcn@latest add switch
```

Trong ví dụ này, câu lệnh sẽ export component Switch vào `resources/js/components/ui/switch.tsx`. Sau khi component đã được export, bạn có thể sử dụng nó trong bất kỳ trang nào mà bạn muốn:

```jsx
import { Switch } from "@/components/ui/switch"

const MyPage = () => {
  return (
    <div>
      <Switch />
    </div>
  );
};

export default MyPage;
```

<a name="react-available-layouts"></a>
#### Available Layouts

Bộ công cụ khởi tạo React có chứa hai layout chính khác nhau để bạn có thể lựa chọn: layout "sidebar" và layout "header". Layout sidebar là layout mặc định, nhưng bạn có thể chuyển sang layout header bằng cách thay đổi layout được import ở phía trên cùng trong file `resources/js/layouts/app-layout.tsx` của ứng dụng:

```js
import AppLayoutTemplate from '@/layouts/app/app-sidebar-layout'; // [tl! remove]
import AppLayoutTemplate from '@/layouts/app/app-header-layout'; // [tl! add]
```

<a name="react-sidebar-variants"></a>
#### Sidebar Variants

Layout sidebar có chứa ba biến thể khác nhau: biến thể sidebar mặc định, biến thể "inset" và biến thể "floating". Bạn có thể lựa chọn biến thể mà bạn thích bằng cách thay đổi component `resources/js/components/app-sidebar.tsx`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="react-authentication-page-layout-variants"></a>
#### Authentication Page Layout Variants

Các trang xác thực đi kèm với bộ công cụ khởi tạo React, chẳng hạn như trang đăng nhập và trang đăng ký, cũng có chứa ba biến thể layout khác nhau: "simple", "card" và "split".

Để thay đổi layout xác thực của bạn, hãy thay đổi layout được import ở phía trên cùng trong file `resources/js/layouts/auth-layout.tsx` của ứng dụng:

```js
import AuthLayoutTemplate from '@/layouts/auth/auth-simple-layout'; // [tl! remove]
import AuthLayoutTemplate from '@/layouts/auth/auth-split-layout'; // [tl! add]
```

<a name="svelte-customization"></a>
### Svelte

Bộ công cụ khởi tạo Svelte của chúng tôi được xây dựng bằng Inertia 3, Svelte 5, Tailwind và [shadcn-svelte](https://www.shadcn-svelte.com/). Giống như tất cả các bộ công cụ khởi tạo khác của chúng tôi, toàn bộ code backend và frontend đều nằm trong ứng dụng của bạn để bạn có thể tùy chỉnh bất kỳ thứ gì bạn muốn.

Phần lớn code frontend nằm trong thư mục `resources/js`. Bạn có thể tự do chỉnh sửa bất kỳ đoạn code nào để tùy chỉnh giao diện và hành vi của ứng dụng:

```text
resources/js/
├── components/    # Reusable Svelte components
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration and Svelte rune modules
├── pages/         # Page components
└── types/         # TypeScript definitions
```

Để export thêm các component shadcn-svelte, trước tiên hãy [tìm component mà bạn muốn sử dụng](https://www.shadcn-svelte.com). Sau đó, hãy tiến hành bằng cách sử dụng lệnh `npx`:

```shell
npx shadcn-svelte@latest add switch
```

Trong ví dụ này, câu lệnh sẽ export component Switch vào `resources/js/components/ui/switch/switch.svelte`. Sau khi component đã được export, bạn có thể sử dụng nó trong bất kỳ trang nào của bạn:

```svelte
<script lang="ts">
    import { Switch } from '@/components/ui/switch'
</script>

<div>
    <Switch />
</div>
```

<a name="svelte-available-layouts"></a>
#### Available Layouts

Bộ công cụ khởi tạo Svelte có chứa hai layout chính khác nhau để bạn lựa chọn: layout "sidebar" và layout "header". Layout sidebar là mặc định, nhưng bạn có thể chuyển sang layout header bằng cách thay đổi layout được import ở phía trên cùng trong file `resources/js/layouts/AppLayout.svelte` của ứng dụng:

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.svelte'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.svelte'; // [tl! add]
```

<a name="svelte-sidebar-variants"></a>
#### Sidebar Variants

Layout sidebar có chứa ba biến thể khác nhau: biến thể sidebar mặc định, biến thể "inset" và biến thể "floating". Bạn có thể lựa chọn biến thể mà bạn thích bằng cách thay đổi component `resources/js/components/AppSidebar.svelte`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="svelte-authentication-page-layout-variants"></a>
#### Authentication Page Layout Variants

Các trang xác thực đi kèm với bộ công cụ khởi tạo Svelte, chẳng hạn như trang đăng nhập và trang đăng ký, cũng có chứa ba biến thể layout khác nhau: "simple", "card" và "split".

Để thay đổi layout xác thực của bạn, hãy thay đổi layout được import ở phía trên cùng trong file `resources/js/layouts/AuthLayout.svelte` của ứng dụng:

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.svelte'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.svelte'; // [tl! add]
```

<a name="vue-customization"></a>
### Vue

Bộ công cụ khởi tạo Vue của chúng tôi được xây dựng bằng Inertia 3, Vue 3 Composition API, Tailwind và [shadcn-vue](https://www.shadcn-vue.com/). Giống như tất cả các bộ công cụ khởi tạo khác của chúng tôi, toàn bộ code backend và frontend đều nằm trong ứng dụng của bạn để bạn có thể tùy chỉnh bất kỳ thứ gì bạn muốn.

Phần lớn code frontend nằm trong thư mục `resources/js`. Bạn có thể tự do chỉnh sửa bất kỳ đoạn code nào để tùy chỉnh giao diện và hành vi của ứng dụng:

```text
resources/js/
├── components/    # Reusable Vue components
├── composables/   # Vue composables / hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

Để export thêm các component shadcn-vue, trước tiên hãy [tìm component mà bạn muốn sử dụng](https://www.shadcn-vue.com). Sau đó, hãy tiến hành bằng cách sử dụng lệnh `npx`:

```shell
npx shadcn-vue@latest add switch
```

Trong ví dụ này, câu lệnh sẽ export component Switch vào `resources/js/components/ui/Switch.vue`. Sau khi component đã được export, bạn có thể sử dụng nó trong bất kỳ trang nào mà bạn muốn:

```vue
<script setup lang="ts">
import { Switch } from '@/components/ui/switch'
</script>

<template>
    <div>
        <Switch />
    </div>
</template>
```

<a name="vue-available-layouts"></a>
#### Available Layouts

Bộ công cụ khởi tạo Vue có chứa hai layout chính khác nhau để bạn có thể lựa chọn: layout "sidebar" và layout "header". Layout sidebar là layout mặc định, nhưng bạn có thể chuyển sang layout header bằng cách thay đổi layout được import ở phía trên cùng trong file `resources/js/layouts/AppLayout.vue` của ứng dụng:

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.vue'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.vue'; // [tl! add]
```

<a name="vue-sidebar-variants"></a>
#### Sidebar Variants

Layout sidebar có chứa ba biến thể khác nhau: biến thể sidebar mặc định, biến thể "inset" và biến thể "floating". Bạn có thể lựa chọn biến thể mà bạn thích bằng cách thay đổi component `resources/js/components/AppSidebar.vue`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="vue-authentication-page-layout-variants"></a>
#### Authentication Page Layout Variants

Các trang xác thực đi kèm với bộ công cụ khởi tạo Vue, chẳng hạn như trang đăng nhập và trang đăng ký, cũng có chứa ba biến thể layout khác nhau: "simple", "card" và "split".

Để thay đổi layout xác thực của bạn, hãy thay đổi layout được import ở phía trên cùng trong file `resources/js/layouts/AuthLayout.vue` của ứng dụng:

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.vue'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.vue'; // [tl! add]
```

<a name="livewire-customization"></a>
### Livewire

Bộ công cụ khởi tạo Livewire của chúng tôi được xây dựng bằng Livewire 4, Tailwind và [Flux UI](https://fluxui.dev/). Giống như tất cả các bộ công cụ khởi tạo khác của chúng tôi, toàn bộ code backend và frontend đều nằm trong ứng dụng của bạn để bạn có thể tùy chỉnh bất kỳ thứ gì bạn muốn.

Phần lớn code frontend nằm trong thư mục `resources/views`. Bạn có thể tự do chỉnh sửa bất kỳ đoạn code nào để tùy chỉnh giao diện và hành vi của ứng dụng:

```text
resources/views
├── components            # Reusable components
├── flux                  # Customized Flux components
├── layouts               # Application layouts
├── pages                 # Livewire pages
├── partials              # Reusable Blade partials
├── dashboard.blade.php   # Authenticated user dashboard
├── welcome.blade.php     # Guest user welcome page
```

<a name="livewire-available-layouts"></a>
#### Available Layouts

Bộ công cụ khởi tạo Livewire có chứa hai layout chính khác nhau để bạn có thể lựa chọn: layout "sidebar" và layout "header". Layout sidebar là layout mặc định, nhưng bạn có thể chuyển sang layout header bằng cách thay đổi layout được sử dụng trong file `resources/views/layouts/app.blade.php` của ứng dụng. Ngoài ra, bạn nên thêm thuộc tính `container` vào component Flux main:

```blade
<x-layouts::app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts::app.header>
```

<a name="livewire-authentication-page-layout-variants"></a>
#### Authentication Page Layout Variants

Các trang xác thực đi kèm với bộ công cụ khởi tạo Livewire, chẳng hạn như trang đăng nhập và trang đăng ký, cũng có chứa ba biến thể layout khác nhau: "simple", "card" và "split".

Để thay đổi layout xác thực của bạn, hãy thay đổi layout được sử dụng trong file `resources/views/layouts/auth.blade.php` của ứng dụng:

```blade
<x-layouts::auth.split>
    {{ $slot }}
</x-layouts::auth.split>
```

<a name="authentication"></a>
## Authentication

Tất cả các bộ công cụ khởi tạo đều sử dụng [Laravel Fortify](/docs/{{version}}/fortify) để xử lý xác thực. Fortify cung cấp các route, controller và logic cho việc đăng nhập, đăng ký, reset mật khẩu, xác minh email và nhiều hơn nữa.

Fortify tự động đăng ký các route xác thực sau đây dựa trên các chức năng được kích hoạt trong file cấu hình `config/fortify.php` của ứng dụng:

<div class="overflow-auto">

| Route                              | Method   | Description                             |
| ---------------------------------- | -------- | --------------------------------------- |
| `/login`                           | `GET`    | Hiển thị form đăng nhập               |
| `/login`                           | `POST`   | Xác thực người dùng                   |
| `/logout`                          | `POST`   | Đăng xuất người dùng                  |
| `/register`                        | `GET`    | Hiển thị form đăng ký                 |
| `/register`                        | `POST`   | Tạo người dùng mới                    |
| `/forgot-password`                 | `GET`    | Hiển thị form yêu cầu reset mật khẩu  |
| `/forgot-password`                 | `POST`   | Gửi liên kết reset mật khẩu           |
| `/reset-password/{token}`          | `GET`    | Hiển thị form reset mật khẩu          |
| `/reset-password`                  | `POST`   | Cập nhật mật khẩu                     |
| `/email/verify`                    | `GET`    | Hiển thị thông báo xác minh email     |
| `/email/verify/{id}/{hash}`        | `GET`    | Xác minh địa chỉ email                |
| `/email/verification-notification` | `POST`   | Gửi lại email xác minh                |
| `/user/confirm-password`           | `GET`    | Hiển thị form xác nhận mật khẩu       |
| `/user/confirm-password`           | `POST`   | Xác nhận mật khẩu                     |
| `/two-factor-challenge`            | `GET`    | Hiển thị form xác thực 2FA            |
| `/two-factor-challenge`            | `POST`   | Xác thực mã 2FA                       |

</div>

Lệnh Artisan `php artisan route:list` có thể được sử dụng để hiển thị tất cả các route có trong ứng dụng của bạn.

<a name="enabling-and-disabling-features"></a>
### Enable và disable features

Bạn có thể kiểm soát các chức năng nào trong Fortify sẽ được kích hoạt trong file cấu hình `config/fortify.php` của ứng dụng:

```php
use Laravel\Fortify\Features;

'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

Để disable một chức năng, hãy comment hoặc xóa chức năng đó ra khỏi mảng `features`. Ví dụ: hãy xóa `Features::registration()` để tắt chức năng đăng ký.

Khi sử dụng bộ công cụ khởi tạo [React](#react), [Svelte](#svelte) hoặc [Vue](#vue), bạn cũng sẽ cần xóa mọi tham chiếu đến các route của chức năng đã bị disable trong code frontend của bạn. Ví dụ: nếu bạn tắt chức năng xác minh email, bạn nên xóa các phần import và tham chiếu đến các route `verification` trong các component React, Svelte, hoặc Vue của bạn. Điều này là cần thiết vì các bộ công cụ khởi tạo này sử dụng Wayfinder để điều hướng an toàn, nó tự động tạo ra các định nghĩa route lúc build. Nhưng nếu bạn tham chiếu đến các route không còn tồn tại, ứng dụng của bạn sẽ không thể build thành công.

<a name="customizing-actions"></a>
### Tùy chỉnh User Creation và Password Reset

Khi người dùng đăng ký hoặc reset mật khẩu, Fortify sẽ gọi các class action nằm trong thư mục `app/Actions/Fortify` của ứng dụng:

<div class="overflow-auto">

| File                          | Description                               |
| ----------------------------- | ----------------------------------------- |
| `CreateNewUser.php`           | Xác thực và tạo người dùng mới            |
| `ResetUserPassword.php`       | Xác thực và cập nhật mật khẩu người dùng  |
| `PasswordValidationRules.php` | Định nghĩa các quy tắc xác thực mật khẩu  |

</div>

Ví dụ, để tùy chỉnh logic đăng ký của ứng dụng, bạn nên tùy chỉnh action `CreateNewUser`:

```php
public function create(array $input): User
{
    Validator::make($input, [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'email', 'max:255', 'unique:users'],
        'phone' => ['required', 'string', 'max:20'], // [tl! add]
        'password' => $this->passwordRules(),
    ])->validate();

    return User::create([
        'name' => $input['name'],
        'email' => $input['email'],
        'phone' => $input['phone'], // [tl! add]
        'password' => Hash::make($input['password']),
    ]);
}
```

<a name="two-factor-authentication"></a>
### Two-Factor Authentication

Các bộ công cụ khởi tạo có sẵn chức năng xác thực hai yếu tố (2FA), cho phép người dùng bảo mật tài khoản của họ bằng bất kỳ ứng dụng xác thực nào tương thích với TOTP. 2FA được kích hoạt mặc định thông qua `Features::twoFactorAuthentication()` trong file cấu hình `config/fortify.php` của ứng dụng.

Tùy chọn `confirm` sẽ yêu cầu người dùng xác minh mã trước khi 2FA được kích hoạt, trong khi `confirmPassword` sẽ yêu cầu xác nhận mật khẩu trước khi kích hoạt hoặc vô hiệu hóa 2FA. Để biết thêm chi tiết, hãy xem [tài liệu xác thực hai yếu tố của Fortify](/docs/{{version}}/fortify#two-factor-authentication).

<a name="rate-limiting"></a>
### Rate Limiting

Giới hạn tỷ lệ giúp ngăn chặn các cuộc tấn công brute-force và các lần thử đăng nhập lặp đi lặp lại gây quá tải cho các url xác thực. Bạn có thể tùy chỉnh hành vi giới hạn tỷ lệ của Fortify trong `FortifyServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;

RateLimiter::for('login', function ($request) {
    return Limit::perMinute(5)->by($request->email.$request->ip());
});
```

<a name="teams"></a>
## Teams

Các bộ khởi tạo starter kit React, Svelte, Vue và Livewire cũng hỗ trợ chức năng team. Khi chức năng này được kích hoạt, mỗi user sẽ thuộc về một hoặc nhiều team và có một team hiện tại làm mặc định. Trong quá trình đăng ký, user mới sẽ tự động được đưa vào một team của cá nhân. Các starter kit cũng chứa các màn hình quản lý team để tạo team, chuyển đổi giữa các team, mời thành viên và cập nhật thông tin team.

Khi một route bị giới hạn trong team hiện tại, thì tên team hiện tại sẽ được nằm trong URL. Ví dụ: route dashboard sẽ trở thành `/{current_team}/dashboard`, trong khi các trang quản lý team sử dụng các route như `settings/teams/{team}`. Khi sử dụng các route parameter là `{current_team}` và `{team}`, các starter kit sẽ tự động đảm bảo các user đã xác thực phải thuộc về team được yêu cầu thì mới cho phép truy cập vào route.

Để việc tạo URL có chứa tên team được thuận tiện hơn, các starter kit sẽ đăng ký cấu hình URL mặc định cho team hiện tại của người dùng đã xác thực. Điều này cho phép các lệnh gọi tới helper như `route('dashboard')` sẽ tự động chứa tên team hiện tại. Khi user đăng nhập, đăng ký hoặc chuyển đổi qua lại giữa các team, các starter kit sẽ cập nhật team hiện tại và refresh cấu hình URL mặc định này để các link được tạo ra sẽ tiếp tục sử dụng đúng nội dung của team.

Khi tạo hoặc đổi tên một team, các starter kit cũng ngăn user tạo ra các tên đặc biệt đã được khai báo cho hệ thống vì chúng có thể tạo ra các thành phần đường dẫn không an toàn hoặc xung đột. Ví dụ, tên có thể xung đột với prefix của route như `settings`, `login` hoặc `dashboard` sẽ không được phép sử dụng.

<a name="workos"></a>
## WorkOS AuthKit Authentication

Mặc định, các bộ công cụ khởi tạo React, Svelte, Vue và Livewire đều sử dụng hệ thống xác thực có sẵn của Laravel để cung cấp các chức năng đăng nhập, đăng ký, reset mật khẩu, xác minh email và nhiều hơn nữa. Ngoài ra, chúng tôi cũng cung cấp một biến thể sử dụng [WorkOS AuthKit](https://authkit.com) cho mỗi bộ công cụ khởi tạo, cung cấp các chức năng:

<div class="content-list" markdown="1">

- Xác thực qua mạng xã hội (Google, Microsoft, GitHub và Apple)
- Xác thực bằng Passkey
- Xác thực qua email "Magic Auth"
- SSO

</div>

Sử dụng WorkOS làm chức năng xác thực [yêu cầu bạn phải có tài khoản WorkOS](https://workos.com). WorkOS cung cấp xác thực miễn phí cho các ứng dụng có tới 1 triệu người dùng hoạt động hàng tháng.

Để sử dụng WorkOS AuthKit làm chức năng xác thực cho ứng dụng của bạn, hãy chọn tùy chọn WorkOS khi tạo ứng dụng mới bằng bộ công cụ khởi tạo thông qua `laravel new`.

### Configuring Your WorkOS Starter Kit

Sau khi tạo một ứng dụng mới bằng bộ công cụ khởi tạo sử dụng WorkOS, bạn nên set các biến môi trường `WORKOS_CLIENT_ID`, `WORKOS_API_KEY` và `WORKOS_REDIRECT_URL` trong file `.env` của ứng dụng. Các biến này phải giống với các giá trị được cung cấp trong dashboard WorkOS cho ứng dụng của bạn:

```ini
WORKOS_CLIENT_ID=your-client-id
WORKOS_API_KEY=your-api-key
WORKOS_REDIRECT_URL="${APP_URL}/authenticate"
```

Ngoài ra, bạn nên cấu hình URL homepage của ứng dụng trong dashboard WorkOS. Đây là URL mà người dùng sẽ được chuyển đến sau khi họ đăng xuất khỏi ứng dụng của bạn.

<a name="configuring-authkit-authentication-methods"></a>
#### Configuring AuthKit Authentication Methods

Khi sử dụng bộ công cụ khởi tạo sử dụng WorkOS, chúng tôi khuyên bạn nên disable chức năng xác thực "Email + Password" trong cài đặt cấu hình WorkOS AuthKit của ứng dụng, đồng thời chỉ cho phép người dùng xác thực thông qua các phương thức xác thực mạng xã hội, passkey, "Magic Auth" và SSO. Điều này giúp ứng dụng của bạn hoàn toàn tránh được việc xử lý mật khẩu của người dùng.

<a name="configuring-authkit-session-timeouts"></a>
#### Configuring AuthKit Session Timeouts

Ngoài ra, chúng tôi khuyên bạn nên cấu hình thời gian kết thúc session của WorkOS AuthKit giống với ngưỡng hết hạn session được cấu hình trong ứng dụng Laravel của bạn, thông thường là hai giờ.

<a name="inertia-ssr"></a>
### Inertia SSR

Các bộ công cụ khởi tạo React, Svelte, và Vue tương thích với khả năng [server-side rendering](https://inertiajs.com/server-side-rendering) của Inertia. Để build một bundle tương thích với Inertia SSR cho ứng dụng của bạn, hãy chạy lệnh `build:ssr`:

```shell
npm run build:ssr
```

Để thuận tiện, một lệnh `composer dev:ssr` cũng có sẵn. Lệnh này sẽ khởi chạy cả Laravel development server và Inertia SSR server sau khi hoàn tất việc build bundle tương thích SSR cho ứng dụng của bạn, giúp bạn có thể kiểm tra ứng dụng ngay tại môi trường local bằng engine server-side rendering của Inertia:

```shell
composer dev:ssr
```

<a name="community-maintained-starter-kits"></a>
### Các bộ Starter Kit của cộng đồng

Khi tạo một ứng dụng Laravel mới bằng Laravel installer, bạn cũng có thể cung cấp bất kỳ bộ công cụ khởi tạo nào có sẵn do cộng đồng phát triển trên Packagist vào flag `--using`:

```shell
laravel new my-app --using=example/starter-kit
```

<a name="creating-starter-kits"></a>
#### Creating Starter Kits

Để đảm bảo bộ công cụ khởi tạo của bạn có sẵn cho người khác sử dụng, bạn sẽ cần đưa nó lên [Packagist](https://packagist.org). Bộ công cụ khởi tạo của bạn nên định nghĩa các biến môi trường cần thiết trong file `.env.example`, và bất kỳ lệnh nào cần thiết sau khi cài đặt nên được liệt kê trong mảng `post-create-project-cmd` của file `composer.json` trong bộ công cụ khởi tạo của bạn.

<a name="faqs"></a>
### Các câu hỏi thường gặp

<a name="faq-upgrade"></a>
#### How do I upgrade?

Mỗi bộ công cụ khởi tạo mang đến cho bạn một điểm khởi đầu vững chắc cho ứng dụng tiếp theo của bạn. Với toàn quyền sở hữu code, bạn có thể tinh chỉnh, tùy chỉnh và xây dựng ứng dụng của mình chính xác theo cách bạn mong muốn. Tuy nhiên, bạn không cần phải cập nhật chính bộ công cụ khởi tạo đó.

<a name="faq-enable-email-verification"></a>
#### How do I enable email verification?

Chức năng xác minh email có thể được thêm vào bằng cách bỏ comment dòng import `MustVerifyEmail` trong model `App/Models/User.php` của bạn và đảm bảo model này implement interface `MustVerifyEmail`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
// ...

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

Sau khi đăng ký, người dùng sẽ nhận được một email xác minh. Để giới hạn quyền truy cập vào một số route cho đến khi địa chỉ email của người dùng được xác minh, hãy thêm middleware `verified` vào các route đó:

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('dashboard', function () {
        return Inertia::render('dashboard');
    })->name('dashboard');
});
```

> [!NOTE]
> Việc xác minh email sẽ là không bắt buộc khi sử dụng biến thể [WorkOS](#workos) của các bộ công cụ khởi tạo.

<a name="faq-modify-email-template"></a>
#### How do I modify the default email template?

Bạn có thể muốn tùy chỉnh email template mặc định để phù hợp hơn với thương hiệu ứng dụng của bạn. Để chỉnh sửa template này, bạn nên export các view email vào ứng dụng của bạn bằng lệnh sau:

```
php artisan vendor:publish --tag=laravel-mail
```

Lệnh này sẽ tạo ra một vài file trong thư mục `resources/views/vendor/mail`. Bạn có thể chỉnh sửa bất kỳ file nào trong số này cũng như file `resources/views/vendor/mail/themes/default.css` để thay đổi giao diện và diện mạo của email template mặc định.
