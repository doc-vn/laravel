# Compiling Assets (Mix)

- [Giới thiệu](#introduction)
- [Installation và setup](#installation)
- [Chạy mix](#running-mix)
- [Làm việc cùng stylesheets](#working-with-stylesheets)
    - [Tailwind CSS](#tailwindcss)
    - [PostCSS](#postcss)
    - [Sass](#sass)
    - [Xử lý URL](#url-processing)
    - [Source Maps](#css-source-maps)
- [Làm việc cùng javaScript](#working-with-scripts)
    - [Vue](#vue)
    - [React](#react)
    - [Vendor Extraction](#vendor-extraction)
    - [Tuỳ biến cấu hình webpack](#custom-webpack-configuration)
- [Versioning / Cache Busting](#versioning-and-cache-busting)
- [Browsersync Reloading](#browsersync-reloading)
- [Environment Variables](#environment-variables)
- [Thông báo](#notifications)

<a name="introduction"></a>
## Giới thiệu

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix) là một package được phát triển bởi Jeffrey Way [Laracasts](https://laracasts.com), cung cấp một API dễ hiểu để định nghĩa các bước xây dựng [webpack](https://webpack.js.org) cho application Laravel của bạn bằng cách sử dụng một số CSS phổ biến và JavaScript pre-processors.

Nói cách khác, Mix giúp bạn dễ dàng biên dịch và thu nhỏ các file CSS và JavaScript trong ứng dụng của bạn. Thông qua cách kết hợp nhiều phương thức đơn giản, bạn có thể dễ dàng định nghĩa asset pipeline của bạn. Ví dụ:

    mix.js('resources/js/app.js', 'public/js')
        .postCss('resources/css/app.css', 'public/css');

Nếu bạn đã từng bối rối và choáng ngợp khi bắt đầu với Webpack và biên dịch asset, bạn sẽ thích Laravel Mix. Tuy nhiên, bạn không bắt buộc phải sử dụng nó trong khi phát triển application của bạn; Bạn có thể thoải mái sử dụng bất kỳ công cụ asset pipeline nào bạn muốn, hoặc thậm chí không dùng gì cả.

> {tip} Nếu bạn cần bắt đầu xây dựng ứng dụng của bạn bằng Laravel và [Tailwind CSS](https://tailwindcss.com), bạn có thể xem xét một [bộ công cụ starter kits](/docs/{{version}}/starter-kits).

<a name="installation"></a>
## Installation và setup

<a name="installing-node"></a>
#### Installing Node

Để chạy Mix, trước tiên bạn phải đảm bảo rằng Node.js và NPM đã được cài đặt trên máy của bạn:

    node -v
    npm -v

Bạn có thể dễ dàng cài đặt phiên bản Node và NPM mới nhất bằng cách cài đặt từ [website chính thức của Node ](https://nodejs.org/en/download/). Hoặc, nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail), bạn có thể sử dụng Node và NPM thông qua Sail:

    ./sail node -v
    ./sail npm -v

<a name="installing-laravel-mix"></a>
#### Installing Laravel Mix

Bước duy nhất còn lại là cài đặt Laravel Mix. Trong bản cài đặt đầu tiên của Laravel, bạn sẽ tìm thấy file `package.json` trong thư mục gốc của bạn. File `package.json` mặc định sẽ có mọi thứ bạn cần để bắt đầu sử dụng Laravel Mix. Hãy nghĩ file này giống như file `composer.json` của bạn, ngoại trừ việc nó định nghĩa các library của NodeJS thay vì các library PHP. Bạn có thể cài đặt các library này bằng cách chạy:

    npm install

<a name="running-mix"></a>
## Chạy mix

Mix là một lớp cấu hình nằm phía trên của [Webpack](https://webpack.js.org), vì vậy, để chạy các tác vụ Mix của bạn, bạn chỉ cần thực thi một trong các lệnh NPM script đã được cài đặt sẵn trong file `package.json` của Laravel. Khi bạn chạy lệnh `dev` hoặc `production`, tất cả nội dung CSS và JavaScript có trong ứng dụng của bạn sẽ được biên dịch và lưu vào trong thư mục `public` của ứng dụng:

    // Run all Mix tasks...
    npm run dev

    // Run all Mix tasks and minify output...
    npm run prod

<a name="watching-assets-for-changes"></a>
#### Theo dõi Assets để thay đổi

Lệnh `npm run watch` sẽ liên tục được chạy trong terminal của bạn và theo dõi tất cả các file CSS và JavaScript có liên quan để thay đổi. Webpack sẽ tự động biên dịch lại assets của bạn khi phát hiện thấy có các thay đổi trong những file này:

    npm run watch

Webpack có thể không phát hiện ra được các thay đổi có trong file của bạn trong một số môi trường local bộ nhất định. Nếu đó là trường hợp xảy ra trên máy của bạn, hãy dùng thử lệnh `watch-poll`:

    npm run watch-poll

<a name="working-with-stylesheets"></a>
## Làm việc cùng stylesheets

File `webpack.mix.js` trong application của bạn là một điểm khởi đầu của bạn cho tất cả các quá trình biên dịch asset. Hãy nghĩ nó như một cấu hình bao bọc [webpack](https://webpack.js.org). Các tác vụ Mix có thể được kết nối với nhau để định nghĩa chính xác cách asset của bạn sẽ được biên dịch.

<a name="tailwindcss"></a>
### Tailwind CSS

[Tailwind CSS](https://tailwindcss.com) là một framework hiện đại, tiện lợi để giúp bạn xây dựng các trang web mà không cần phải rời khỏi trang HTML của bạn. Hãy cùng tìm hiểu cách bắt đầu sử dụng framework này trong project Laravel với Laravel Mix. Trước tiên, chúng ta nên cài đặt Tailwind bằng NPM và tạo file cấu hình Tailwind:

    npm install

    npm install -D tailwindcss

    npx tailwindcss init

Lệnh `init` sẽ tạo file `tailwind.config.js`. Phần `content` của file này cho phép bạn cấu hình đường dẫn đến tất cả các template HTML, JavaScript component và bất kỳ file source nào khác mà chứa tên class Tailwind để bất kỳ class CSS nào mà không được sử dụng trong các file này sẽ bị xóa khỏi bản build CSS production của bạn:

```js
content: [
    './storage/framework/views/*.php',
    './resources/**/*.blade.php',
    './resources/**/*.js',
    './resources/**/*.vue',
],
```

Tiếp theo, bạn nên thêm từng "layers" của Tailwind vào file `resources/css/app.css` của ứng dụng:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Sau khi đã cấu hình các layers của Tailwind, bạn đã sẵn sàng cập nhật file `webpack.mix.js` của ứng dụng để biên dịch CSS hỗ trợ Tailwind:

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css', [
        require('tailwindcss'),
    ]);
```

Cuối cùng, bạn nên khai báo stylesheet của bạn trong layout template chính của ứng dụng. Nhiều ứng dụng chọn lưu các template này tại `resources/views/layouts/app.blade.php`. Ngoài ra, hãy đảm bảo là bạn đã thêm tag `meta` của responsive viewport nếu nó chưa có:

```html
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link href="/css/app.css" rel="stylesheet">
</head>
```

<a name="postcss"></a>
### PostCSS

[PostCSS](https://postcss.org/) là một công cụ mạnh mẽ để chuyển đổi CSS của bạn và đã có sẵn trong Laravel Mix. Mặc định, Mix sử dụng plugin phổ biến [Autoprefixer](https://github.com/postcss/autoprefixer) để tự động áp dụng tất cả các tiền tố CSS3 vendor cần thiết. Tuy nhiên, bạn có thể thoải mái thêm bất kỳ các plugin nào mà phù hợp với ứng dụng của bạn.

Trước tiên, hãy cài đặt plugin mong muốn thông qua NPM và thêm nó vào array plugin của bạn khi gọi phương thức `postCss` của Mix. Phương thức `postCss` sẽ chấp nhận một đường dẫn đến file CSS của bạn làm tham số đầu tiên và thư mục nơi file đã biên dịch sẽ được lưu vào làm tham số thứ hai:

    mix.postCss('resources/css/app.css', 'public/css', [
        require('postcss-custom-properties')
    ]);

Hoặc, bạn có thể thực thi `postCss` mà không cần thêm plugin để đơn giản hoá quá trình biên dịch và rút gọn CSS:

    mix.postCss('resources/css/app.css', 'public/css');

<a name="sass"></a>
### Sass

Phương thức `sass` cho phép bạn biên dịch các file [Sass](https://sass-lang.com/) thành CSS mà trình duyệt web có thể đọc hiểu được. Phương thức `sass` sẽ chấp nhận một đường dẫn đến file Sass của bạn làm tham số đầu tiên và thư mục nơi mà file đã biên dịch sẽ được lưu vào làm tham số thứ hai:

    mix.sass('resources/sass/app.scss', 'public/css');

Bạn có thể biên dịch nhiều file Sass thành các file CSS tương ứng của riêng chúng và thậm chí tùy chỉnh thư mục lưu các file CSS này bằng cách gọi nhiều lần phương thức `sass`:

    mix.sass('resources/sass/app.sass', 'public/css')
        .sass('resources/sass/admin.sass', 'public/css/admin');

<a name="url-processing"></a>
### Xử lý URL

Bởi vì Laravel Mix được xây dựng dựa trên webpack, nên điều quan trọng là phải hiểu một vài khái niệm về webpack. Để biên dịch CSS, webpack sẽ viết lại và tối ưu hóa mọi lệnh gọi `url()` trong code css của bạn. Mặc dù điều này ban đầu nghe có vẻ lạ, nhưng đây là một chức năng cực kỳ mạnh mẽ. Hãy tưởng tượng rằng chúng ta muốn biên dịch code Sass có chứa một URL liên kết với một hình ảnh:

    .example {
        background: url('../images/example.png');
    }

> {note} Đường dẫn tuyệt đối cho mọi `url()` sẽ được bỏ qua khỏi việc viết lại. Ví dụ như `url('/images/thing.png')` hoặc `url('http://example.com/images/thing.png')` sẽ không bị viết lại.

Mặc định, Laravel Mix và webpack sẽ tìm file `example.png`, và sao chép nó vào thư mục `public/images` của bạn, sau đó viết lại `url()` trong file css được tạo ra. Như vậy, CSS mà được biên dịch ra của bạn sẽ là:

    .example {
        background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

Tính năng này rất hữu ích, nhưng có thể cấu trúc thư mục hiện tại của bạn đã bị thay đổi theo cách mà bạn muốn. Nếu trong trường hợp đó, bạn có thể vô hiệu hóa việc viết lại `url()` như sau:

    mix.sass('resources/sass/app.scss', 'public/css').options({
        processCssUrls: false
    });

Với đoạn code này vào trong file `webpack.mix.js` của bạn, Mix sẽ không còn match với bất kỳ `url()` hoặc sao chép assets nào vào thư mục public của bạn. Nói cách khác, CSS được biên dịch ra sẽ trông giống như cách mà bạn đã khai báo ban đầu:

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>
### Source Maps

Mặc định bị disabled, nhưng source map có thể được kích hoạt bằng cách gọi phương thức `mix.sourceMaps()` trong file `webpack.mix.js` của bạn. Mặc dù nó có thể làm tăng thêm thời gian biên dịch và hiệu năng, nhưng điều này sẽ cung cấp thêm thông tin gỡ lỗi cho các tool develop trên trình duyệt của bạn khi sử dụng các assets đã được biên dịch:

    mix.js('resources/js/app.js', 'public/js')
        .sourceMaps();

<a name="style-of-source-mapping"></a>
#### Style Of Source Mapping

Webpack cung cấp nhiều [kiểu source mapping](https://webpack.js.org/configuration/devtool/#devtool). Mặc định, kiểu source mapping của Mix sẽ được set thành `eval-source-map`, cung cấp thời gian rebuild nhanh chóng. Nếu bạn muốn thay đổi kiểu mapping, bạn có thể làm như vậy bằng cách sử dụng phương thức `sourceMaps`:

    let productionSourceMaps = false;

    mix.js('resources/js/app.js', 'public/js')
        .sourceMaps(productionSourceMaps, 'source-map');

<a name="working-with-scripts"></a>
## Làm việc cùng javaScript

Mix cung cấp một số tính năng để giúp bạn làm việc với các file JavaScript của bạn, chẳng hạn như biên dịch modern ECMAScript, module bundling, thu nhỏ file và nối các file JavaScript. Thậm chí, tất cả đều hoạt động trơn tru, không đòi hỏi bất kỳ một cấu hình tùy chỉnh nào:

    mix.js('resources/js/app.js', 'public/js');

Với chỉ một dòng code duy nhất, giờ đây bạn có thể làm:

<div class="content-list" markdown="1">

- Cú pháp EcmaScript mới nhất.
- Modules
- Thu nhỏ file cho môi trương production.

</div>

<a name="vue"></a>
### Vue

Mix sẽ tự động cài đặt các plugin Babel cần thiết để hỗ trợ biên dịch các file component Vue khi sử dụng phương thức `vue`. Bạn sẽ không cần cấu hình thêm bất kỳ điều gì:

    mix.js('resources/js/app.js', 'public/js')
       .vue();

Khi JavaScript của bạn đã được biên dịch, bạn có thể khai báo nó trong ứng dụng của bạn:

```html
<head>
    <!-- ... -->

    <script src="/js/app.js"></script>
</head>
```

<a name="react"></a>
### React

Mix có thể tự động cài đặt các plugin Babel cần thiết để hỗ trợ React. Để bắt đầu, hãy gọi phương thức `react`:

    mix.js('resources/js/app.jsx', 'public/js')
       .react();

Ở phía hậu trường, Mix sẽ tự động tải xuống và khai báo các plugin Babel `babel-preset-react` thích hợp. Khi JavaScript của bạn đã được biên dịch xong, bạn có thể khai báo chúng trong ứng dụng của bạn:

```html
<head>
    <!-- ... -->

    <script src="/js/app.js"></script>
</head>
```

<a name="vendor-extraction"></a>
### Vendor Extraction

Một nhược điểm trong việc kết hợp các JavaScript dành riêng cho application của bạn với các vendor library của bạn như là React và Vue là nó khiến cho việc lưu trữ lâu dài trở nên khó khăn hơn. Ví dụ: một bản cập nhật nhỏ trong code application của bạn cũng sẽ buộc trình duyệt phải tải lại tất cả các vendor library của bạn ngay cả khi chúng không thay đổi.

Nếu bạn thường xuyên cập nhật JavaScript trong application của bạn, thì bạn nên xem xét đưa tất cả các vendor library vào một file của riêng. Theo cách này, một thay đổi đối với code application của bạn sẽ không ảnh hưởng đến cache của file `vendor.js` rất lớn của bạn. Phương thức `extract` của Mix làm cho điều này trở nên dễ dàng:

    mix.js('resources/js/app.js', 'public/js')
        .extract(['vue'])

Phương thức `extract` chấp nhận một mảng của tất cả các thư viện hoặc modules mà bạn muốn thành một file `vendor.js riêng. Sử dụng đoạn code trên làm ví dụ, Mix sẽ tạo ra các file như sau:

<div class="content-list" markdown="1">

- `public/js/manifest.js`: *The Webpack manifest runtime*
- `public/js/vendor.js`: *Các vendor library của bạn*
- `public/js/app.js`: *code application của bạn*

</div>

Để tránh lỗi JavaScript, hãy đảm bảo load các file này theo đúng thứ tự như ở dưới đây:

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="custom-webpack-configuration"></a>
### Tuỳ biến cấu hình webpack

Nhưng đôi khi, bạn có thể cần phải tự sửa các cấu hình Webpack cơ bản. Ví dụ, bạn có thể có một loader hoặc một plugin đặc biệt cần được khai báo.

Mix cung cấp một phương thức `webpackConfig` hữu ích cho phép bạn merge bất kỳ phần ghi đè nào vào cấu hình Webpack. Điều này đặc biệt hấp dẫn, vì nó không yêu cầu bạn phải sao chép và giữ bản sao của file `webpack.config.js`. Phương thức `webpackConfig` chấp nhận một đối tượng, trong đó có chứa bất kỳ [cấu hình dành riêng cho Webpack](https://webpack.js.org/configuration/) nào mà bạn muốn áp dụng.

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/js')
            ]
        }
    });

<a name="versioning-and-cache-busting"></a>
## Versioning / Cache Busting

Nhiều developer muốn thêm một hậu tố timestamp hoặc unique token vào sau các assets đã được biên dịch của họ để buộc các trình duyệt tải lại các assets mới thay vì dùng lại các bản cũ của assets. Mix có thể tự động xử lý việc này cho bạn bằng cách sử dụng phương thức `version`.

Phương thức `version` sẽ thêm một chuỗi hash duy nhất vào sau tên file của tất cả các file đã biên dịch, cho phép một cơ chế tạo bộ nhớ cache một cách thuận tiện hơn:

    mix.js('resources/js/app.js', 'public/js')
        .version();

Sau khi tạo file đã được version, bạn sẽ không biết chính xác tên file. Vì vậy, bạn nên sử dụng hàm global helper `mix` của Laravel trong [views](/docs/{{version}}/views) của bạn để load ra các URL version asset đã được hash. The `mix` function will automatically determine the current name of the hashed file:

    <script src="{{ mix('/js/app.js') }}"></script>

Vì các file version thường không cần thiết trong quá trình phát triển, nên bạn có thể thêm điều kiện để tạo file version chỉ chạy trong khi `npm run prod`:

    mix.js('resources/js/app.js', 'public/js');

    if (mix.inProduction()) {
        mix.version();
    }

<a name="custom-mix-base-urls"></a>
#### Custom Mix Base URLs

Nếu các asset của bạn đã được Mix biên dịch và được deploy cho một CDN tách biệt với ứng dụng của bạn, bạn sẽ cần thay đổi URL được tạo bởi hàm `mix`. Bạn có thể làm như vậy bằng cách thêm tùy chọn cấu hình `mix_url` vào file cấu hình `config/app.php` trong application của bạn:

    'mix_url' => env('MIX_ASSET_URL', null)

Sau khi cấu hình Mix URL, Hàm `mix` sẽ tạo tiền tố cho URL đã cấu hình khi tạo URL cho asset:

```bash
https://cdn.example.com/js/app.js?id=1964becbdd96414518cd
```

<a name="browsersync-reloading"></a>
## Browsersync Reloading

[BrowserSync](https://browsersync.io/) có thể tự động theo dõi các file thay đổi của bạn và đưa các thay đổi đó vào trình duyệt mà không yêu cầu bạn phải refresh trình duyệt. Bạn có thể kích hoạt hỗ trợ này bằng cách gọi phương thức `mix.browserSync()`:

```js
mix.browserSync('laravel.test');
```

[Tùy chọn BrowserSync](https://browsersync.io/docs/options) có thể được chỉ định bằng cách truyền một đối tượng JavaScript cho phương thức `browserSync`:

```js
mix.browserSync({
    proxy: 'laravel.test'
});
```

Tiếp theo, khởi động server phát triển của webpack bằng lệnh `npm run watch`. Bây giờ, khi bạn thay đổi một script hoặc một file PHP bạn có thể thấy trình duyệt refresh ngay lập tức để phản ánh các thay đổi của bạn.

<a name="environment-variables"></a>
## Environment Variables

Bạn có thể đưa các biến môi trường vào file script `webpack.mix.js` của bạn bằng cách thêm tiền tố `MIX_` vào một trong các biến môi trường trong file `.env` của bạn:

    MIX_SENTRY_DSN_PUBLIC=http://example.com

Sau khi các biến đã được định nghĩa trong file `.env` của bạn, bạn có thể truy cập vào nó thông qua đối tượng `process.env`. Tuy nhiên, bạn sẽ cần phải khởi động lại các task nếu các giá trị của biến môi trường này bị thay đổi trong khi task vẫn đang chạy:

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>
## Thông báo

Khi sẵn sàng, Mix sẽ tự động hiển thị một thông báo của hệ điều hành khi biên dịch, sẽ cung cấp cho bạn những thông tin rằng việc biên dịch có thành công hay không. Tuy nhiên, có thể có những trường hợp bạn muốn tắt các thông báo này. Một trong những ví dụ như vậy là trên môi trường server production của bạn. Thông báo có thể bị tắt bởi phương thức `disableNotifications`:

    mix.disableNotifications();
