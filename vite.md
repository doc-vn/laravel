# Asset Bundling (Vite)

- [Giới thiệu](#introduction)
- [Installation và Setup](#installation)
  - [Cài đặt Node](#installing-node)
  - [Cài đặt Vite và Laravel Plugin](#installing-vite-and-laravel-plugin)
  - [Cấu hình Vite](#configuring-vite)
  - [Loading script và style của bạn](#loading-your-scripts-and-styles)
- [Chạy Vite](#running-vite)
- [Working với JavaScript](#working-with-scripts)
  - [Aliases](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Svelte](#svelte)
  - [Inertia](#inertia)
  - [URL Processing](#url-processing)
- [Working với Stylesheets](#working-with-stylesheets)
- [Working với Fonts](#working-with-fonts)
  - [Font Providers](#font-providers)
  - [Local Fonts](#local-fonts)
  - [Font Options](#font-options)
- [Working với Blade và Routes](#working-with-blade-and-routes)
  - [Processing Static Assets với Vite](#blade-processing-static-assets)
  - [Refreshing On Save](#blade-refreshing-on-save)
  - [Aliases](#blade-aliases)
- [Tải trước Asset](#asset-prefetching)
- [Tùy biến Base URLs](#custom-base-urls)
- [Environment Variables](#environment-variables)
- [Disabling Vite In Tests](#disabling-vite-in-tests)
- [Server-Side Rendering (SSR)](#ssr)
- [Script và Style Tag Attributes](#script-and-style-attributes)
  - [Content Security Policy (CSP) Nonce](#content-security-policy-csp-nonce)
  - [Subresource Integrity (SRI)](#subresource-integrity-sri)
  - [Arbitrary Attributes](#arbitrary-attributes)
- [Tùy biến nâng cao](#advanced-customization)
  - [Dev Server Cross-Origin Resource Sharing (CORS)](#cors)
  - [Correcting Dev Server URLs](#correcting-dev-server-urls)

<a name="introduction"></a>
## Giới thiệu

[Vite](https://vitejs.dev) là một công cụ xây dựng frontend hiện đại cung cấp một môi trường phát triển cực nhanh và đóng gói code của bạn để deploy lên môi trường production. Khi xây dựng ứng dụng bằng Laravel, bạn thường sẽ sử dụng Vite để đóng gói các file CSS và JavaScript của ứng dụng thành các asset sẵn sàng trên môi trường production.

Laravel tích hợp liền mạch với Vite bằng cách cung cấp plugin chính thức và lệnh Blade để load các asset của bạn cho mục đích development và production.

<a name="installation"></a>
## Installation và Setup

> [!NOTE]
> Tài liệu sau đây sẽ thảo luận về cách cài đặt và cấu hình plugin Laravel Vite. Tuy nhiên, [bộ khởi tạo](/docs/{{version}}/starter-kits) của Laravel đã chứa tất cả các scaffolding này và là cách nhanh nhất để bắt đầu với Laravel và Vite.

<a name="installing-node"></a>
### Cài đặt Node

Bạn phải đảm bảo là Node.js (16+) và NPM đã được cài đặt trước khi chạy Vite và plugin Laravel:

```shell
node -v
npm -v
```

Bạn có thể dễ dàng cài đặt phiên bản mới nhất của Node và NPM bằng phần mềm cài đặt đồ họa từ [trang web chính thức của Node](https://nodejs.org/en/download/). Hoặc, nếu bạn đang sử dụng [Laravel Sail](https://laravel.com/docs/{{version}}/sail), bạn có thể gọi Node và NPM thông qua Sail:

```shell
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

<a name="installing-vite-and-laravel-plugin"></a>
### Cài đặt Vite và Laravel Plugin

Trong bản cài đặt Laravel mới, bạn sẽ tìm thấy file `package.json` trong thư mục root của ứng dụng. Mặc định, file `package.json` đã chứa mọi thứ mà bạn cần để bắt đầu sử dụng Vite và plugin Laravel. Bạn có thể cài đặt các library giao diện người dùng của ứng dụng thông qua NPM:

```shell
npm install
```

<a name="configuring-vite"></a>
### Cấu hình Vite

Vite được cấu hình thông qua file `vite.config.js` trong thư mục root của dự án. Bạn có thể tùy chỉnh file này tùy theo nhu cầu của bạn và cũng có thể cài đặt bất kỳ plugin nào khác mà ứng dụng của bạn yêu cầu, chẳng hạn như `@vitejs/plugin-react`, `@sveltejs/vite-plugin-svelte` hoặc `@vitejs/plugin-vue`.

Plugin Laravel Vite yêu cầu bạn chỉ định đầu vào cho ứng dụng của bạn. Đây có thể là các file JavaScript hoặc CSS và cả các ngôn ngữ tiền xử lý như TypeScript, JSX, TSX và Sass.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

Nếu bạn đang xây dựng một SPA, chứa các ứng dụng được xây dựng bằng Inertia, Vite sẽ hoạt động tốt nhất mà không cần đầu vào CSS:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

Thay vào đó, bạn nên import CSS của bạn qua JavaScript. Thông thường, điều này sẽ được thực hiện trong file `resources/js/app.js` của ứng dụng:

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Plugin Laravel cũng hỗ trợ nhiều đầu vào và các tùy chọn cấu hình nâng cao như [đầu vào SSR](#ssr).

<a name="working-with-a-secure-development-server"></a>
#### Working With A Secure Development Server

Nếu máy chủ web phát triển local của bạn đang chạy ứng dụng của bạn dưới giao thức HTTPS, thì bạn có thể gặp lỗi khi kết nối với máy chủ phát triển Vite.

Nếu bạn đang sử dụng [Laravel Herd](https://herd.laravel.com) và cần bảo vệ một trang web hoặc bạn đang sử dụng [Laravel Valet](/docs/{{version}}/valet) và đã chạy [lệnh secure](/docs/{{version}}/valet#securing-sites) trên ứng dụng của bạn, plugin Laravel Vite sẽ tự động phát hiện và sử dụng chứng chỉ TLS đã tạo cho bạn.

Nếu bạn đang muốn bảo vệ một trang web mà tên host của trang web đó không khớp với tên thư mục của ứng dụng, bạn có thể chỉ định tên host đó vào trong file `vite.config.js` của ứng dụng:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            detectTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```

Khi sử dụng máy chủ web khác, bạn nên tạo chứng chỉ và cấu hình Vite theo cách thủ công để sử dụng các chứng chỉ đã tạo đó:

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

Nếu bạn không thể tạo chứng chỉ cho hệ thống của bạn, bạn có thể cài đặt và cấu hình [plugin @vitejs/plugin-basic-ssl](https://github.com/vitejs/vite-plugin-basic-ssl). Khi sử dụng chứng chỉ này, bạn sẽ cần chấp nhận cảnh báo chứng chỉ cho máy chủ phát triển của Vite trong trình duyệt của bạn và link "Local" trong console khi chạy lệnh `npm run dev`.

<a name="configuring-hmr-in-sail-on-wsl2"></a>
#### Running the Development Server in Sail on WSL2

Khi chạy máy chủ phát triển Vite trong [Laravel Sail](/docs/{{version}}/sail) trên Windows Subsystem cho Linux 2 (WSL2), bạn nên thêm cấu hình sau vào file `vite.config.js` để đảm bảo rằng trình duyệt có thể giao tiếp với máy chủ phát triển:

```js
// ...

export default defineConfig({
    // ...
    server: { // [tl! add:start]
        hmr: {
            host: 'localhost',
        },
    }, // [tl! add:end]
});
```

Nếu những thay đổi trong file của bạn không được phản ánh trong trình duyệt khi máy chủ phát triển đang chạy, bạn cũng có thể cần cấu hình tùy chọn [server.watch.usePolling](https://vitejs.dev/config/server-options.html#server-watch) của Vite.

<a name="loading-your-scripts-and-styles"></a>
### Loading script và style của bạn

Khi đã cấu hình các đầu vào Vite, bạn có thể tham chiếu chúng trong lệnh Blade `@vite()` mà bạn đã thêm vào `<head>` của template gốc của ứng dụng:

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

Nếu bạn import CSS thông qua JavaScript, bạn chỉ cần đưa đầu nhập JavaScript vào:

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

Lệnh `@vite` sẽ tự động phát hiện máy chủ phát triển Vite và tích hợp Vite client để kích hoạt Hot Module Replacement. Trong chế độ build, lệnh sẽ load các asset đã biên dịch và đã version của bạn, và bất kỳ file CSS nào mà bạn đã import.

Nếu cần, bạn cũng có thể chỉ định đường dẫn build các asset đã biên dịch của bạn khi gọi lệnh `@vite`:

```blade
<!doctype html>
<head>
    {{-- Given build path is relative to public path. --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="inline-assets"></a>
#### Inline Assets

Thỉnh thoảng bạn có thể cần phải thêm một nội dung raw của asset thay vì link đến một URL version của asset. Ví dụ: bạn có thể cần thêm nội dung asset trực tiếp vào trong trang HTML của bạn khi truyền nội dung HTML đến PDF generator. Bạn có thể xuất nội dung của asset Vite bằng phương thức `content` do facade `Vite` cung cấp:

```blade
@use('Illuminate\Support\Facades\Vite')

<!doctype html>
<head>
    {{-- ... --}}

    <style>
        {!! Vite::content('resources/css/app.css') !!}
    </style>
    <script>
        {!! Vite::content('resources/js/app.js') !!}
    </script>
</head>
```

<a name="running-vite"></a>
## Chạy Vite

Có hai cách để bạn có thể chạy Vite. Bạn có thể chạy máy chủ phát triển thông qua lệnh `dev`, lệnh này hữu ích khi phát triển local. Máy chủ phát triển sẽ tự động phát hiện các thay đổi đối với các file của bạn và ngay lập tức phản ánh chúng vào trong bất kỳ cửa sổ trình duyệt nào đang được mở.

Hoặc, chạy lệnh `build` sẽ tạo version và đóng gói các asset của ứng dụng và chuẩn bị chúng để bạn triển khai lên môi trường production:

```shell
# Run the Vite development server...
npm run dev

# Build and version the assets for production...
npm run build
```

Nếu bạn đang chạy server phát triển ở [Sail](/docs/{{version}}/sail) trên WSL2, bạn có thể cần một số tùy chọn [cấu hình bổ sung](#configuring-hmr-in-sail-on-wsl2).

<a name="working-with-scripts"></a>
## Working với JavaScript

<a name="aliases"></a>
### Aliases

Mặc định, plugin Laravel sẽ cung cấp một bí danh chung để giúp bạn bắt đầu ngay và import các asset của ứng dụng một cách thuận tiện:

```js
{
    '@' => '/resources/js'
}
```

Bạn có thể ghi đè bí danh `'@'` bằng cách thêm bí danh của riêng bạn vào file cấu hình `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### Vue

Nếu bạn muốn xây dựng giao diện người dùng của bạn bằng cách sử dụng framework [Vue](https://vuejs.org/), thì bạn cũng sẽ cần phải cài đặt plugin `@vitejs/plugin-vue`:

```shell
npm install --save-dev @vitejs/plugin-vue
```

Sau đó, bạn có thể thêm plugin trong file cấu hình `vite.config.js` của bạn. Có một số tùy chọn thêm mà bạn sẽ cần khi sử dụng plugin Vue với Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // The Vue plugin will re-write asset URLs, when referenced
                    // in Single File Components, to point to the Laravel web
                    // server. Setting this to `null` allows the Laravel plugin
                    // to instead re-write asset URLs to point to the Vite
                    // server instead.
                    base: null,

                    // The Vue plugin will parse absolute URLs and treat them
                    // as absolute paths to files on disk. Setting this to
                    // `false` will leave absolute URLs un-touched so they can
                    // reference assets in the public directory as expected.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> [!NOTE]
> [Bộ công cụ khởi tạo](/docs/{{version}}/starter-kits) của Laravel đã chứa cấu hình Laravel, Vue và Vite phù hợp. Những bộ khởi tạo này là cách nhanh nhất để bắt đầu với Laravel, Vue và Vite.

<a name="react"></a>
### React

Nếu bạn muốn xây dựng giao diện người dùng của bạn bằng framework [React](https://reactjs.org/), thì bạn cũng sẽ cần cài đặt plugin `@vitejs/plugin-react`:

```shell
npm install --save-dev @vitejs/plugin-react
```

Sau đó, bạn có thể thêm plugin vào file cấu hình `vite.config.js` của bạn:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

Bạn sẽ cần đảm bảo rằng bất kỳ file nào chứa JSX đều có phần extension là `.jsx` hoặc `.tsx`, hãy nhớ cập nhật đầu nhập của bạn, nếu cần, như [hiển thị ở trên](#configuring-vite).

Bạn cũng sẽ cần phải thêm lệnh Blade `@viteReactRefresh` cùng với lệnh `@vite` đã tồn tại trước đó của bạn.

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

Lệnh `@viteReactRefresh` phải được gọi trước lệnh `@vite`.

> [!NOTE]
> [Bộ công cụ khởi tạo](/docs/{{version}}/starter-kits) của Laravel đã chứa cấu hình Laravel, React và Vite phù hợp. Những bộ công cụ khởi tạo này là cách nhanh nhất để bắt đầu với Laravel, React và Vite.

<a name="svelte"></a>
### Svelte

Nếu bạn muốn xây dựng frontend của bạn bằng framework [Svelte](https://svelte.dev/), thì bạn cũng sẽ cần cài đặt plugin `@sveltejs/vite-plugin-svelte`:

```shell
npm install --save-dev @sveltejs/vite-plugin-svelte
```

Sau đó, bạn có thể đưa plugin này vào trong file cấu hình `vite.config.js` của bạn.

```js
import { svelte } from '@sveltejs/vite-plugin-svelte';
import laravel from 'laravel-vite-plugin';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    laravel({
      input: ['resources/js/app.ts'],
      ssr: 'resources/js/ssr.ts',
      refresh: true,
    }),
    svelte(),
  ],
});
```

> [!NOTE]
> [Các bộ khởi tạo](/docs/{{version}}/starter-kits) của Laravel đã chứa sẵn cấu hình cho Laravel, Svelte và Vite phù hợp. Những bộ khởi tạo này là cách nhanh nhất để bắt đầu ứng dụng Laravel với Svelte và Vite.

<a name="inertia"></a>
### Inertia

Plugin Laravel Vite cung cấp hàm `resolvePageComponent` tiện lợi để giúp bạn resolve các component page Inertia của bạn. Dưới đây là ví dụ về helper được sử dụng với Vue 3; tuy nhiên, bạn cũng có thể sử dụng hàm này trong các framework khác như React hoặc Svelte:

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

Nếu bạn đang sử dụng tính năng code splitting của Vite với Inertia, chúng tôi khuyên bạn nên cấu hình [asset prefetching](#asset-prefetching).

> [!NOTE]
> [Bộ công cụ khởi tạo](/docs/{{version}}/starter-kits) của Laravel đã chứa cấu hình Laravel, Inertia và Vite phù hợp. Những bộ khởi tạo này là cách nhanh nhất để bắt đầu với Laravel, Inertia và Vite.

<a name="url-processing"></a>
### URL Processing

Khi sử dụng Vite và tham chiếu asset trong HTML, CSS hoặc JS của ứng dụng, có một số lưu ý cần cân nhắc. Đầu tiên, nếu bạn tham chiếu asset bằng đường dẫn tuyệt đối, Vite sẽ không chứa asset vào trong bản build; do đó, bạn nên đảm bảo rằng asset có sẵn trong thư mục public của bạn. Bạn nên tránh sử dụng đường dẫn tuyệt đối khi sử dụng một [CSS entrypoint chuyên dụng](#configuring-vite) bởi vì, trong quá trình phát triển, các trình duyệt sẽ cố gắng load các đường dẫn này từ server develop của Vite, nơi CSS được lưu trữ, thay vì từ thư mục public của bạn.

Khi tham chiếu đường dẫn asset tương đối, bạn nên nhớ rằng đường dẫn là tương đối tới file mà chúng được tham chiếu. Bất kỳ asset nào được tham chiếu thông qua đường dẫn tương đối sẽ được Vite viết lại, version hóa và đóng gói.

Hãy xem cấu trúc project sau:

```text
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

Ví dụ sau đây minh họa cách Vite xử lý URL tương đối và tuyệt đối:

```html
<!-- This asset is not handled by Vite and will not be included in the build -->
<img src="/taylor.png">

<!-- This asset will be re-written, versioned, and bundled by Vite -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## Working với Stylesheets

> [!NOTE]
> [Bộ khởi tạo](/docs/{{version}}/starter-kits) của Laravel đã chứa sẵn cấu hình Tailwind và Vite phù hợp. Hoặc, nếu bạn muốn sử dụng Tailwind và Laravel mà không cần sử dụng một trong các bộ khởi tạo của chúng tôi, hãy xem [hướng dẫn cài đặt Tailwind cho Laravel](https://tailwindcss.com/docs/guides/laravel).

Tất cả các ứng dụng Laravel đều có chứa Tailwind và một file `vite.config.js` đã được cấu hình. Vì vậy, bạn chỉ cần khởi động server phát triển Vite hoặc chạy lệnh Composer `dev`, lệnh này sẽ khởi động server phát triển Laravel và Vite:

```shell
composer run dev
```

CSS của ứng dụng của bạn có thể được đặt trong file `resources/css/app.css`.

<a name="working-with-fonts"></a>
## Working với Fonts

Plugin Laravel Vite có thể chạy các font tự host được tối ưu cho ứng dụng của bạn. Khi font được cấu hình, plugin sẽ resolve ra các file font được yêu cầu, xuất chúng thành các asset Vite, tạo CSS font và ghi một font manifest mà lệnh [`@fonts`](/docs/{{version}}/blade#fonts) của Blade có thể sử dụng được.

Để cấu hình font, hãy import một hoặc nhiều provider helper từ `laravel-vite-plugin/fonts` và thêm chúng vào tùy chọn `fonts` của plugin Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { google } from 'laravel-vite-plugin/fonts';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            fonts: [
                google('Inter', {
                    alias: 'sans',
                    weights: [400, 500, 600, 700],
                    styles: ['normal', 'italic'],
                    subsets: ['latin'],
                    display: 'swap',
                    preload: [
                        { weight: 400 },
                        { weight: 700 },
                    ],
                    fallbacks: ['system-ui', 'sans-serif'],
                }),
            ],
        }),
    ],
});
```

Trong ví dụ này, font `Inter` sẽ có sẵn thông qua alias `sans`. Plugin sẽ tạo biến CSS `--font-sans` và một utility class `.font-sans` áp dụng cho font đã được tạo.

<a name="font-providers"></a>
### Font Providers

Plugin Laravel Vite có sẵn các provider helper cho Google Fonts, Bunny Fonts, Fontsource và local font:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { bunny, fontsource, google, local } from 'laravel-vite-plugin/fonts';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            fonts: [
                google('Inter', { alias: 'sans' }),
                bunny('Figtree', { alias: 'body' }),
                fontsource('JetBrains Mono', { alias: 'mono' }),
                local('Brand Sans', {
                    alias: 'brand',
                    src: 'resources/fonts/brand-sans',
                }),
            ],
        }),
    ],
});
```

Provider `fontsource` sẽ đọc font từ một package Fontsource đã được cài đặt. Mặc định, tên package sẽ được lấy ra từ tên font family, ví dụ như `@fontsource/jetbrains-mono`. Nếu ứng dụng của bạn sử dụng tên package khác, bạn có thể chỉ định nó bằng tùy chọn `package`.

<a name="local-fonts"></a>
### Local Fonts

Khi sử dụng local font, tùy chọn `src` sẽ trỏ đến một file font, một thư mục hoặc một glob pattern. Plugin sẽ tự động tìm các file font được hỗ trợ và suy luận các giá trị weight cũng như style từ tên file:

```js
local('Brand Sans', {
    alias: 'brand',
    src: 'resources/fonts/brand-sans/*.woff2',
})
```

Nếu bạn cần kiểm soát hoàn toàn các loại font, bạn có thể định nghĩa chúng một cách rõ ràng bằng tùy chọn `variants`:

```js
local('Brand Sans', {
    alias: 'brand',
    variants: [
        { src: 'resources/fonts/BrandSans-Regular.woff2', weight: 400 },
        { src: 'resources/fonts/BrandSans-Italic.woff2', weight: 400, style: 'italic' },
        { src: ['resources/fonts/BrandSans-Bold.woff2', 'resources/fonts/BrandSans-Bold.ttf'], weight: 700 },
    ],
})
```

<a name="font-options"></a>
### Font Options

Tùy thuộc vào provider, mà định nghĩa font có thể chấp nhận một số tùy chọn cho phép bạn tùy chỉnh CSS font được tạo:

<div class="content-list" markdown="1">

- `alias` định nghĩa tên được lệnh `@fonts` của Blade sử dụng và mặc định là tên của font family.
- `variable` định nghĩa biến CSS được tạo và mặc định là `--font-{alias}`.
- `weights` định nghĩa các font weight cho remote hoặc Fontsource cần được lấy ra và mặc định là `[400]`.
- `styles` định nghĩa các font style cho remote hoặc Fontsource cần được lấy ra và mặc định là `['normal']`.
- `subsets` định nghĩa các font subset cho remote hoặc Fontsource cần được lấy ra và mặc định là `['latin']`.
- `display` định nghĩa giá trị `font-display` và mặc định là `swap`.
- `preload` điều khiển các biến thể WOFF2 nào sẽ được load trước. Tùy chọn này có thể là `true`, `false` hoặc một mảng các selector `{ weight, style }`.
- `fallbacks` định nghĩa các font dự phòng sẽ được thêm vào font stack đã được tạo.
- `optimizedFallbacks` cố gắng tạo các font face fallback được điều chỉnh theo số liệu metric bằng cách sử dụng package tùy chọn `fontaine` và mặc định là `true`.

</div>

Optimized fallback sẽ yêu cầu package `fontaine`, vốn không được cài đặt mặc định. Nếu bạn muốn Laravel tạo các font face fallback được điều chỉnh theo metric, bạn hãy cài đặt `fontaine` như một dev dependency:

```shell
npm install --save-dev fontaine
```

Nếu `fontaine` chưa được cài đặt hoặc không thể đọc file font, Laravel sẽ bỏ qua optimized fallback cho font đó và tiếp tục sử dụng các font được cấu hình thông qua tùy chọn `fallbacks`.

Các local font được lấy từ các tùy chọn `src` hoặc `variants` được mô tả ở trên thay vì sử dụng `weights`, `styles` và `subsets`.

<a name="working-with-blade-and-routes"></a>
## Working với Blade và Routes

<a name="blade-processing-static-assets"></a>
### Processing Static Assets với Vite

Khi tham chiếu đến asset trong JavaScript hoặc CSS của bạn, Vite sẽ tự động xử lý và tạo version cho chúng. Ngoài ra, khi xây dựng các ứng dụng dựa trên Blade, Vite cũng có thể xử lý và tạo version cho các asset tĩnh mà bạn chỉ tham chiếu trong các template Blade.

Tuy nhiên, để thực hiện điều này, bạn cần phải cho Vite biết về các asset của bạn bằng cách khai báo chúng trong tùy chọn `assets` của plugin. Tùy chọn này chỉ dành cho các file tĩnh mà bạn muốn tham chiếu trực tiếp bằng `Vite::asset`. Nếu bạn muốn Laravel tạo ra CSS font và các link preload, hãy sử dụng tùy chọn [`fonts`](#working-with-fonts) thay thế.

Ví dụ: nếu bạn muốn xử lý và tạo version cho tất cả hình ảnh được lưu trong `resources/images` và tất cả các font chữ được lưu trong `resources/fonts`, bạn nên thêm nội dung sau vào cấu hình Vite của bạn:

```js
laravel({
    input: 'resources/js/app.js',
    assets: ['resources/images/**', 'resources/fonts/**'],
})
```

Các asset này hiện sẽ được Vite xử lý khi chạy `npm run build`. Sau đó, bạn có thể tham chiếu các asset này vào trong các template Blade bằng phương thức `Vite::asset`, phương thức này sẽ trả về URL đã version hoá cho một asset nhất định:

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

> [!NOTE]
> Các phiên bản trước phiên bản 3 của plugin Laravel Vite, các asset static phải được import vào trong ứng dụng của bạn bằng `import.meta.glob`. Tuy nhiên, tùy chọn `assets` đã được giới thiệu trong Vite 8.

<a name="blade-refreshing-on-save"></a>
### Refreshing On Save

Khi ứng dụng của bạn được built bằng cách sử dụng render từ server-side truyền thống với Blade, Vite có thể cải thiện quy trình phát triển của bạn bằng cách tự động refresh trình duyệt khi bạn thực hiện một thay đổi trong ứng dụng của bạn. Để bắt đầu, bạn chỉ cần chỉ định tùy chọn `refresh` là `true`.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

Khi tùy chọn `refresh` là `true`, thì việc save file trong các thư mục sau sẽ kích hoạt trình duyệt sẽ thực hiện việc refresh toàn bộ trang trong khi bạn đang chạy `npm run dev`:

- `app/Livewire/**`
- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

Việc theo dõi thư mục `routes/**` sẽ hữu ích nếu bạn đang sử dụng [Ziggy](https://github.com/tighten/ziggy) để tạo các link route trong frontend ứng dụng của bạn.

Nếu các đường dẫn mặc định này không phù hợp với nhu cầu của bạn, bạn có thể chỉ định danh sách đường dẫn riêng để theo dõi:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
```

Về cơ bản, plugin Laravel Vite sẽ sử dụng package [vite-plugin-full-reload](https://github.com/ElMassimo/vite-plugin-full-reload), cung cấp một số tùy chọn cấu hình nâng cao để tinh chỉnh hành vi của tính năng này. Nếu bạn cần mức tùy chỉnh này, bạn có thể cung cấp định nghĩa `config`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
```

<a name="blade-aliases"></a>
### Aliases

Trong các ứng dụng JavaScript, việc [tạo bí danh](#aliases) cho các thư mục hay được sử dụng là điều rất phổ biến. Tuy nhiên, bạn cũng có thể tạo bí danh để sử dụng trong Blade bằng cách sử dụng phương thức `macro` trên class `Illuminate\Support\Facades\Vite`. Thông thường, "macro" phải được định nghĩa trong phương thức `boot` của [service provider](/docs/{{version}}/providers):

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
}
```

Sau khi macro đã được định nghĩa xong, nó có thể được gọi nó trong các template của bạn. Ví dụ, chúng ta có thể sử dụng macro `image` được định nghĩa ở trên để tham chiếu đến một asset nằm tại `resources/images/logo.png`:

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="asset-prefetching"></a>
## Tải trước Asset

Khi xây dựng một SPA bằng tính năng code splitting của Vite, các asset cần thiết sẽ được lấy ra trên mỗi lần chuyển trang. Hành vi này có thể dẫn đến việc render UI bị chậm trễ. Nếu đây là vấn đề đối với frontend framework mà bạn chọn, Laravel cung cấp khả năng load trước các asset JavaScript và CSS của ứng dụng ngay trong lần load trang đầu tiên.

Bạn có thể hướng dẫn Laravel chủ động load trước các asset của bạn bằng cách gọi phương thức `Vite::prefetch` trong phương thức `boot` của một [service provider](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Vite;
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
        Vite::prefetch(concurrency: 3);
    }
}
```

Trong ví dụ trên, các asset sẽ được load trước với tối đa `3` lượt chạy đồng thời trên mỗi lần load trang. Bạn có thể thay đổi số lượng đồng thời này để phù hợp với nhu cầu của ứng dụng hoặc không chỉ định giới hạn đồng thời nếu ứng dụng cần tải xuống tất cả các asset cùng một lúc:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::prefetch();
}
```

Mặc định, việc load trước sẽ bắt đầu khi event [page _load_](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event) được kích hoạt. Nếu bạn muốn tùy chỉnh thời điểm bắt đầu load trước, bạn có thể chỉ định một event mà Vite sẽ lắng nghe:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::prefetch(event: 'vite:prefetch');
}
```

Với đoạn code ở trên, việc load trước sẽ bắt đầu khi bạn kích hoạt event `vite:prefetch` trên object `window`. Ví dụ, bạn có thể bắt đầu việc load trước sau ba giây kể từ khi trang được load:

```html
<script>
    addEventListener('load', () => setTimeout(() => {
        dispatchEvent(new Event('vite:prefetch'))
    }, 3000))
</script>
```

<a name="custom-base-urls"></a>
## Tùy biến Base URLs

Nếu asset đã biên dịch Vite của bạn được triển khai tới một domain khác, khác với ứng dụng của bạn, chẳng hạn như thông qua một CDN, thì bạn phải chỉ định biến môi trường `ASSET_URL` trong file `.env` của ứng dụng:

```env
ASSET_URL=https://cdn.example.com
```

Sau khi cấu hình URL asset, tất cả các URL được viết lại cho asset của bạn sẽ được thêm tiền tố bằng giá trị đã cấu hình:

```text
https://cdn.example.com/build/assets/app.9dce8d17.js
```

Hãy nhớ rằng [URL tuyệt đối sẽ không được Vite viết lại](#url-processing), vì vậy chúng sẽ không được thêm tiền tố.

<a name="environment-variables"></a>
## Environment Variables

Bạn có thể tích hợp các biến môi trường vào JavaScript bằng cách thêm tiền tố `VITE_` vào file `.env` của ứng dụng:

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

Bạn có thể truy cập vào các biến môi trường đã được tích hợp thông qua đối tượng `import.meta.env`:

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## Disabling Vite In Tests

Vite của Laravel sẽ cố gắng resolve các asset của bạn khi chạy test, nhưng nó sẽ yêu cầu bạn phải chạy máy chủ phát triển Vite hoặc build các asset của bạn.

Nếu bạn muốn mock Vite trong quá trình test, bạn có thể gọi phương thức `withoutVite`, phương thức này có sẵn cho bất kỳ class test nào được extend từ class `TestCase` của Laravel:

```php tab=Pest
test('without vite example', function () {
    $this->withoutVite();

    // ...
});
```

```php tab=PHPUnit
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_vite_example(): void
    {
        $this->withoutVite();

        // ...
    }
}
```

Nếu bạn muốn disable Vite cho tất cả các bài test, bạn có thể gọi phương thức `withoutVite` từ phương thức `setUp` trên class `TestCase` base của bạn:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## Server-Side Rendering (SSR)

Plugin Laravel Vite sẽ giúp bạn dễ dàng thiết lập render từ server-side với Vite. Để bắt đầu, hãy tạo một đầu vào SSR tại `resources/js/ssr.js` và chỉ định đầu vào bằng cách truyền tùy chọn cấu hình cho plugin Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

Để đảm bảo bạn không quên build lại đầu vào SSR, chúng tôi khuyên bạn nên thêm tập lệnh "build" vào trong `package.json` của ứng dụng để tạo bản build SSR:

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

Sau đó, để build và khởi động máy chủ SSR, bạn có thể chạy các lệnh sau:

```shell
npm run build
node bootstrap/ssr/ssr.js
```

Nếu bạn đang sử dụng [SSR với Inertia](https://inertiajs.com/server-side-rendering), bạn có thể sử dụng lệnh Artisan `inertia:start-ssr` để khởi động máy chủ SSR:

```shell
php artisan inertia:start-ssr
```

> [!NOTE]
> [Bộ công cụ khởi tạo](/docs/{{version}}/starter-kits) của Laravel đã chứa cấu hình Laravel, Inertia SSR và Vite phù hợp. Những bộ công cụ khởi tạo này là cách nhanh nhất để bắt đầu với Laravel, Inertia SSR và Vite.

<a name="script-and-style-attributes"></a>
## Script và Style Tag Attributes

<a name="content-security-policy-csp-nonce"></a>
### Content Security Policy (CSP) Nonce

Nếu bạn muốn đưa một [thuộc tính nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) vào trong tag script và style của bạn như một phần của [chính sách bảo mật nội dung](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP), bạn có thể tạo hoặc chỉ định một nonce bằng phương thức `useCspNonce` trong [middleware](/docs/{{version}}/middleware) tùy chỉnh:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Vite;
use Symfony\Component\HttpFoundation\Response;

class AddContentSecurityPolicyHeaders
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
```

Sau khi gọi phương thức `useCspNonce`, Laravel sẽ tự động thêm các thuộc tính `nonce` vào tất cả các tag script và style được tạo.

Nếu bạn cần chỉ định nonce ở một nơi khác, bao gồm cả [lệnh Ziggy `@route`](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy) ở trong [bộ công cụ khởi tạo](/docs/{{version}}/starter-kits) của Laravel, bạn có thể lấy ra nó bằng phương thức `cspNonce`:

```blade
@routes(nonce: Vite::cspNonce())
```

Nếu bạn đã có nonce và bạn muốn hướng dẫn Laravel sử dụng nó, bạn có thể truyền nonce đó cho phương thức `useCspNonce`:

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### Subresource Integrity (SRI)

Nếu file manifest Vite của bạn có chứa các hàm hash `integrity` cho các asset của bạn, Laravel sẽ tự động thêm thuộc tính `integrity` vào bất kỳ thẻ script và style nào mà nó tạo ra để thực thi [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity). Mặc định, Vite sẽ không chứa hàm hash `integrity` trong file manifest của nó, nhưng bạn có thể bật nó bằng cách cài đặt plugin [vite-plugin-manifest-sri](https://www.npmjs.com/package/vite-plugin-manifest-sri) NPM:

```shell
npm install --save-dev vite-plugin-manifest-sri
```

Sau đó, bạn có thể enable plugin này trong file `vite.config.js` của bạn:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```

Nếu cần, bạn cũng có thể tùy chỉnh key manifest chỗ mà hàm hash integrity có thể tìm thấy:

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

Nếu bạn muốn disable hoàn toàn tính năng tự động phát hiện này, bạn có thể truyền `false` vào phương thức `useIntegrityKey`:

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### Arbitrary Attributes

Nếu bạn cần chứa các thuộc tính bổ sung vào tag script và style của bạn, chẳng hạn như thuộc tính [data-turbo-track](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change), bạn có thể chỉ định chúng thông qua các phương thức `useScriptTagAttributes` và `useStyleTagAttributes`. Thông thường, các phương thức này sẽ được gọi từ một [service provider](/docs/{{version}}/providers):

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // Specify a value for the attribute...
    'async' => true, // Specify an attribute without a value...
    'integrity' => false, // Exclude an attribute that would otherwise be included...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

Nếu bạn cần thêm thuộc tính theo một điều kiện nhất định, bạn có thể truyền một callback để nhận đường dẫn source asset, URL của asset, một phần manifest và toàn bộ manifest:

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> [!WARNING]
> Các tham số `$chunk` và `$manifest` sẽ là `null` khi máy chủ phát triển Vite đang chạy.

<a name="advanced-customization"></a>
## Tùy biến nâng cao

Mặc định, plugin Vite của Laravel sẽ sử dụng các quy ước hợp lý có thể áp dụng được cho hầu hết các ứng dụng; tuy nhiên, thỉnh thoảng bạn có thể cần tùy chỉnh hành vi của Vite. Để enable thêm các tùy chọn tùy chỉnh bổ sung, chúng tôi cung cấp các phương thức và các tùy chọn sau có thể được sử dụng thay cho lệnh Blade `@vite`:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // Customize the "hot" file...
            ->useBuildDirectory('bundle') // Customize the build directory...
            ->useManifestFilename('assets.json') // Customize the manifest filename...
            ->withEntryPoints(['resources/js/app.js']) // Specify the entry points...
            ->createAssetPathsUsing(function (string $path, ?bool $secure) { // Customize the backend path generation for built assets...
                return "https://cdn.example.com/{$path}";
            })
    }}
</head>
```

Trong file `vite.config.js`, bạn cũng nên chỉ định cùng một cấu hình:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // Customize the "hot" file...
            buildDirectory: 'bundle', // Customize the build directory...
            input: ['resources/js/app.js'], // Specify the entry points...
        }),
    ],
    build: {
      manifest: 'assets.json', // Customize the manifest filename...
    },
});
```

<a name="cors"></a>
### Dev Server Cross-Origin Resource Sharing (CORS)

Nếu bạn đang gặp phải các vấn đề về Cross-Origin Resource Sharing (CORS) trong trình duyệt khi load các asset từ server dev Vite, bạn có thể cần phải cấp quyền truy cập cho origin tùy chỉnh của bạn vào server dev. Vite kết hợp với plugin Laravel cho phép các origin sau mà không cần cấu hình gì thêm:

- `::1`
- `127.0.0.1`
- `localhost`
- `*.test`
- `*.localhost`
- `APP_URL` trong file `.env` của project

Cách dễ nhất để cho phép một origin tùy chỉnh cho project của bạn là đảm bảo rằng biến môi trường `APP_URL` của ứng dụng khớp với origin mà bạn đang truy cập trong trình duyệt. Ví dụ: nếu bạn đang truy cập `https://my-app.laravel`, thì bạn nên cập nhật file `.env` của bạn là địa chỉ trên:

```env
APP_URL=https://my-app.laravel
```

Nếu bạn cần quyền kiểm soát chi tiết hơn đối với các origin, chẳng hạn như hỗ trợ nhiều origin, bạn nên sử dụng [cấu hình server CORS mặc định linh hoạt và toàn diện của Vite](https://vite.dev/config/server-options.html#server-cors). Ví dụ: bạn có thể chỉ định nhiều origin trong tùy chọn cấu hình `server.cors.origin` ở file `vite.config.js` của project:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
    ],
    server: {  // [tl! add]
        cors: {  // [tl! add]
            origin: [  // [tl! add]
                'https://backend.laravel',  // [tl! add]
                'http://admin.laravel:8566',  // [tl! add]
            ],  // [tl! add]
        },  // [tl! add]
    },  // [tl! add]
});
```

Bạn cũng có thể thêm các pattern regex, điều này có thể hữu ích nếu bạn muốn cho phép tất cả các origin cho một top-level domain nhất định, chẳng hạn như `*.laravel`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
    ],
    server: {  // [tl! add]
        cors: {  // [tl! add]
            origin: [ // [tl! add]
                // Supports: SCHEME://DOMAIN.laravel[:PORT] [tl! add]
                /^https?:\/\/.*\.laravel(:\d+)?$/, //[tl! add]
            ], // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

<a name="correcting-dev-server-urls"></a>
### Correcting Dev Server URLs

Một số plugin trong hệ sinh thái Vite sẽ cho là các URL bắt đầu bằng dấu gạch chéo sẽ luôn trỏ đến máy chủ phát triển Vite. Tuy nhiên, do bản chất Vite được tích hợp vào trong Laravel, thì điều này không đúng.

Ví dụ, plugin `vite-imagetools` sẽ xuất ra các URL như sau khi Vite đang chạy asset cho bạn:

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

Plugin `vite-imagetools` đang cho là các URL output sẽ bị Vite chặn lại và sau đó plugin có thể xử lý tất cả các URL bắt đầu bằng `/@imagetools`. Nếu bạn đang sử dụng các plugin mà theo hành vi này, bạn sẽ cần phải sửa thủ công các URL. Bạn có thể thực hiện việc này trong file `vite.config.js` của bạn bằng cách sử dụng tùy chọn `transformOnServe`.

Trong ví dụ cụ thể này, chúng ta sẽ thêm URL máy chủ dev vào trước tất cả các lần xuất hiện của `/@imagetools` có trong code được tạo:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll('/@imagetools', devServerUrl+'/@imagetools'),
        }),
        imagetools(),
    ],
});
```

Bây giờ, khi Vite đang chạy Assets, nó sẽ xuất ra các URL trỏ đến máy chủ phát triển Vite:

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```
