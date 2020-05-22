# Compiling Assets (Laravel Mix)

- [Giới thiệu](#introduction)
- [Installation và setup](#installation)
- [Chạy mix](#running-mix)
- [Làm việc cùng stylesheets](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [Code CSS](#plain-css)
    - [Xử lý URL](#url-processing)
    - [Source Maps](#css-source-maps)
- [Làm việc cùng javaScript](#working-with-scripts)
    - [Vendor Extraction](#vendor-extraction)
    - [React](#react)
    - [Vanilla JS](#vanilla-js)
    - [Tuỳ biến cấu hình webpack](#custom-webpack-configuration)
- [Copying Files và thư mục](#copying-files-and-directories)
- [Versioning / Cache Busting](#versioning-and-cache-busting)
- [Browsersync Reloading](#browsersync-reloading)
- [Environment Variables](#environment-variables)
- [Thông báo](#notifications)

<a name="introduction"></a>
## Giới thiệu

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix) cung cấp API dễ hiểu để định nghĩa các bước xây dựng Webpack cho application Laravel của bạn bằng cách sử dụng một số CSS phổ biến và JavaScript pre-processors. Thông qua phương thức móc nối đơn giản, bạn có thể dễ dàng định nghĩa asset pipeline của bạn. Ví dụ:

    mix.js('resources/assets/js/app.js', 'public/js')
       .sass('resources/assets/sass/app.scss', 'public/css');

Nếu bạn đã từng bối rối và choáng ngợp khi bắt đầu với Webpack và biên dịch asset, bạn sẽ thích Laravel Mix. Tuy nhiên, bạn không bắt buộc phải sử dụng nó trong khi phát triển application của mình. Tất nhiên, bạn có thể thoải mái sử dụng bất kỳ công cụ asset pipeline nào bạn muốn, hoặc thậm chí không dùng gì cả.

<a name="installation"></a>
## Installation và setup

#### Installing Node

Trước khi kích hoạt Mix, trước tiên bạn phải đảm bảo rằng Node.js và NPM đã được cài đặt trên máy của bạn.

    node -v
    npm -v

Mặc định, Laravel Homestead đã chứa mọi thứ bạn cần; tuy nhiên, nếu bạn không sử dụng Vagrant, thì bạn có thể dễ dàng cài đặt phiên bản Node và NPM mới nhất bằng các cài đặt đồ họa từ [trang tải xuống của họ](https://nodejs.org/en/download/).

#### Laravel Mix

Bước duy nhất còn lại là cài đặt Laravel Mix. Trong bản cài đặt đầu tiên của Laravel, bạn sẽ tìm thấy file `package.json`  trong thư mục gốc của bạn. File `package.json`  mặc định sẽ có mọi thứ bạn cần để bắt đầu. Hãy nghĩ điều này giống như file `composer.json` của bạn, ngoại trừ việc nó định nghĩa các library Node thay vì PHP. Bạn có thể cài đặt các library mà nó khai báo bằng cách chạy:

    npm install

<a name="running-mix"></a>
## Chạy mix

Mix là một lớp cấu hình ở trên [Webpack](https://webpack.js.org), vì vậy, để chạy các tác vụ Mix của bạn, bạn chỉ cần thực thi một trong các đoạn NPM script đã được cài đặt cùng file mặc định `package.json` của Laravel:

    // Run all Mix tasks...
    npm run dev

    // Run all Mix tasks and minify output...
    npm run production

#### Theo dõi Assets để thay đổi

Lệnh `npm run watch` sẽ liên tục chạy trong terminal của bạn và theo dõi tất cả các file có liên quan để thay đổi. Webpack sau đó sẽ tự động biên dịch lại assets của bạn khi phát hiện thay đổi:

    npm run watch

Bạn có thể thấy rằng trong một số môi trường nhất định, Webpack không cập nhật lại khi file của bạn thay đổi. Nếu đó là trường hợp xảy ra trên system của bạn, hãy dùng thử lệnh `watch-poll`:

    npm run watch-poll

<a name="working-with-stylesheets"></a>
## Làm việc cùng stylesheets

File `webpack.mix.js` là điểm khởi đầu của bạn cho tất cả quá trình biên dịch asset. Hãy nghĩ nó như một wrapper cấu hình nhẹ bao bọc Webpack. Các tác vụ Mix có thể được kết nối với nhau để định nghĩa chính xác cách asset của bạn sẽ được biên dịch.

<a name="less"></a>
### Less

Phương thức `less` có thể được sử dụng để biên dịch [Less](http://lesscss.org/) thành CSS. Hãy biên dịch file chính `app.less` của chúng ta thành `public/css/app.css`.

    mix.less('resources/assets/less/app.less', 'public/css');

Có thể gọi nhiều lần phương thức `less` để biên dịch nhiều file:

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');

Nếu bạn muốn tùy chỉnh tên file của CSS đã biên dịch, bạn có thể pass một đường dẫn file đầy đủ làm tham số thứ hai cho phương thức `less`:

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');

Nếu bạn cần ghi đè [tùy chọn Less plug-in](https://github.com/webpack-contrib/less-loader#options), bạn có thể pass một đối tượng làm tham số thứ ba cho `mix.less()`:

    mix.less('resources/assets/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>
### Sass

Phương thức `sass` cho phép bạn biên dịch [Sass](http://sass-lang.com/) thành CSS. Bạn có thể sử dụng phương thức như sau:

    mix.sass('resources/assets/sass/app.scss', 'public/css');

Một lần nữa, giống như phương thức `less`, bạn có thể biên dịch nhiều file Sass thành các file CSS tương ứng và thậm chí tùy chỉnh cả thư mục đầu ra của file CSS:

    mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');

[Tùy chọn Node-Sass plug-in](https://github.com/sass/node-sass#options) có thể được cung cấp làm tham số thứ ba:

    mix.sass('resources/assets/sass/app.sass', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>
### Stylus

Tương tự như Less và Sass, phương thức `stylus` cho phép bạn biên dịch [Stylus](http://stylus-lang.com/) thành CSS:
    mix.stylus('resources/assets/stylus/app.styl', 'public/css');

Bạn cũng có thể cài đặt thêm các Stylus plug-in, chẳng hạn như [Rupture](https://github.com/jescalan/rupture). Đầu tiên, cài đặt plug-in thông qua NPM (`npm install rupture`) và sau đó require nó trong lệnh gọi của bạn  `mix.stylus()`:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>
### PostCSS

[PostCSS] (http://postcss.org/), một công cụ mạnh mẽ để chuyển đổi CSS của bạn, được thêm vào trong Laravel Mix. Mặc định, Mix sử dụng plug-in phổ biến [Autoprefixer](https://github.com/postcss/autoprefixer) để tự động áp dụng tất cả các tiền tố CSS3 vendor cần thiết. Tuy nhiên, bạn có thể thoải mái thêm bất kỳ plug-in nào phù hợp với ứng dụng của mình. Đầu tiên, cài đặt plug-in mong muốn thông qua NPM và sau đó tham chiếu nó trong file `webpack.mix.js` của bạn:

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });

<a name="plain-css"></a>
### Code CSS

Nếu bạn chỉ muốn ghép một số code CSS stylesheet thành một file duy nhất, bạn có thể sử dụng phương thức `styles`.

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="url-processing"></a>
### Xử lý URL

Bởi vì Laravel Mix được xây dựng dựa trên Webpack, điều quan trọng là phải hiểu một vài khái niệm về Webpack. Để biên dịch CSS, Webpack sẽ viết lại và tối ưu hóa mọi lệnh gọi `url()` trong code css của bạn. Mặc dù điều này ban đầu nghe có vẻ lạ, nhưng đây là một chức năng cực kỳ mạnh mẽ. Hãy tưởng tượng rằng chúng ta muốn biên dịch Sass có chứa một URL liên kết với một hình ảnh:

    .example {
        background: url('../images/example.png');
    }

> {note} Đường dẫn tuyệt đối cho mọi `url()` sẽ bị bỏ qua khỏi việc viết lại URL. Ví dụ: `url('/images/thing.png')` hoặc `url('http://example.com/images/thing.png')` sẽ không bị sửa.

Mặc định, Laravel Mix và Webpack sẽ tìm `example.png`, sao chép nó vào thư mục `public/images` của bạn, sau đó viết lại `url()` trong file css được tạo. Như vậy, CSS đã biên dịch của bạn sẽ là:

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

Tính năng này rất hữu ích, nhưng có thể cấu trúc thư mục hiện tại của bạn đã được cấu hình theo cách mà bạn muốn. Nếu trường hợp đó, bạn có thể vô hiệu hóa việc viết lại `url()` như sau:

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });

Với phần thêm này vào file `webpack.mix.js` của bạn, Mix sẽ không còn match với bất kỳ `url()` hoặc sao chép assets nào vào thư mục public của bạn. Nói cách khác, CSS được biên dịch ra sẽ trông giống như cách bạn gõ ban đầu:

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>
### Source Maps

Mặc dù mặc định bị disabled, nhưng source map có thể được kích hoạt bằng cách gọi phương thức `mix.sourceMaps()` trong file `webpack.mix.js` của bạn. Mặc dù nó có thể làm tăng thêm thời gian biên dịch và hiệu năng, nhưng điều này sẽ cung cấp thêm thông tin gỡ lỗi cho các tool develope trên trình duyệt của bạn khi sử dụng các assets được biên dịch.

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();

<a name="working-with-scripts"></a>
## Làm việc cùng javaScript

Mix cung cấp một số tính năng để giúp bạn làm việc với các file JavaScript của mình, chẳng hạn như biên dịch ECMAScript 2015, module bundling, thu nhỏ và nối các file JavaScript. Thậm chí, tất cả đều hoạt động trơn tru, không đòi hỏi bất kỳ một cấu hình tùy chỉnh nào:

    mix.js('resources/assets/js/app.js', 'public/js');

Với chỉ một dòng code duy nhất, giờ đây bạn có thể làm:

<div class="content-list" markdown="1">
- Cú pháp ES2015.
- Modules
- Biên dịch các file `.vue`.
- Thu nhỏ cho môi trương product.
</div>

<a name="vendor-extraction"></a>
### Vendor Extraction

Một nhược điểm tiềm ẩn trong việc kết hợp tất cả JavaScript dành riêng cho application với các vendor library của bạn là nó khiến cho việc lưu trữ lâu dài trở nên khó khăn hơn. Ví dụ: một bản cập nhật nhỏ trong code application của bạn cũng sẽ buộc trình duyệt tải xuống lại tất cả các vendor library của bạn ngay cả khi chúng không thay đổi.

Nếu bạn thương xuyên cập nhật JavaScript của application, bạn nên xem xét lấy tất cả các vendor library vào một file của riêng. Theo cách này, một thay đổi đối với code application của bạn sẽ không ảnh hưởng đến cache của file `vendor.js` rất lớn của bạn. Phương thức `extract` của Mix làm cho điều này trở nên dễ dàng:

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])

Phương thức `extract` chấp nhận một mảng của tất cả các thư viện hoặc modules mà bạn muốn lấy thành một file `vendor.js. Sử dụng đoạn code trên làm ví dụ, Mix sẽ tạo các file sau:

<div class="content-list" markdown="1">
- `public/js/manifest.js`: *The Webpack manifest runtime*
- `public/js/vendor.js`: *Các vendor library của bạn*
- `public/js/app.js`: * code application của bạn*
</div>

Để tránh lỗi JavaScript, hãy đảm bảo load các file này theo đúng thứ tự:

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="react"></a>
### React

Mix có thể tự động cài đặt các plug-in cần thiết Babel cho hỗ trợ React. Để bắt đầu, hãy thay thế gọi `mix.js()` của bạn bằng `mix.react()`:

    mix.react('resources/assets/js/app.jsx', 'public/js');

Sau đó, Mix sẽ download và thêm plug-in Babel `babel-preset-react` thích hợp.

<a name="vanilla-js"></a>
### Vanilla JS

Tương tự như việc kết hợp các code css với `mix.styles()`, bạn cũng có thể kết hợp và thu nhỏ bất kỳ file JavaScript nào với phương thức`scripts()`:

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');

Tùy chọn này đặc biệt hữu ích cho các dự án cũ, nơi mà bạn không dùng biên dịch Webpack cho JavaScript của mình.

> {tip} Một biến thể nhỏ của `mix.scripts()` là `mix.babel()`. Chức năng của nó giống hệt với `scripts`; tuy nhiên, file được nối lại sẽ nhận được trình biên dịch Babel, trình biên dịch này sẽ dịch bất kỳ code ES2015 nào sang JavaScript vanilla mà tất cả các trình duyệt sẽ đọc được.

<a name="custom-webpack-configuration"></a>
### Tuỳ biến cấu hình webpack

Mặc định, Laravel Mix sẽ tham chiếu file `webpack.config.js` được cấu hình sẵn để giúp bạn khởi động và chạy nhanh nhất có thể. Đôi khi, bạn có thể cần phải sửa đổi thủ công file này. Bạn có thể có một loader hoặc một plug-in đặc biệt cần được tham chiếu hoặc có thể bạn thích sử dụng Stylus thay vì Sass. Trong những trường hợp như vậy, bạn có hai lựa chọn:

#### Merging cấu hình tùy chỉnh

Mix cung cấp một phương thức `webpackConfig` hữu ích cho phép bạn merge bất kỳ phần ghi đè nhỏ nào vào cấu hình Webpack. Đây là một lựa chọn đặc biệt hấp dẫn, vì nó không yêu cầu bạn sao chép và giữ bản sao của file `webpack.config.js`. Phương thức `webpackConfig` chấp nhận một đối tượng, trong đó có chứa bất kỳ [cấu hình dành riêng cho Webpack](https://webpack.js.org/configuration/) mà bạn muốn áp dụng.

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

#### Tuỳ chỉnh file cấu hình

Nếu bạn muốn tùy chỉnh hoàn toàn cấu hình Webpack của mình, hãy sao chép file `node_modules/laravel-mix/setup/webpack.config.js` vào thư mục gốc của dự án. Tiếp theo, trỏ tất cả các tham chiếu `--config` trong file `package.json` của bạn vào file cấu hình mới được sao chép. Nếu bạn chọn áp dụng phương pháp này để tùy chỉnh, mọi cập nhật trong tương lai cho `webpack.config.js` của Mix phải được merge thủ công vào file tùy chỉnh của bạn.

<a name="copying-files-and-directories"></a>
## Copying Files và thư mục

Phương thức `copy` có thể được sử dụng để sao chép các file và thư mục vào các vị trí mới. Điều này có thể hữu ích khi một asset cụ thể trong thư mục `node_modules` của bạn cần được chuyển đến thư mục `public` của bạn.

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

Khi sao chép một thư mục, phương thức `copy` sẽ xoá hết thư mục con trong cấu trúc của thư mục. Để duy trì cấu trúc ban đầu của thư mục, bạn nên sử dụng phương thức `copyDirectory` thay thế:

    mix.copyDirectory('assets/img', 'public/img');

<a name="versioning-and-cache-busting"></a>
## Versioning / Cache Busting

Nhiều developer muốn thêm hậu tố vào sau assets đã được biên dịch của họ bằng một timestamp hoặc unique token để buộc các trình duyệt tải assets mới thay vì lấy các bản cũ của assets. Mix có thể xử lý việc này cho bạn bằng cách sử dụng phương thức `version`.

Phương thức `version` sẽ tự động nối một hàm hash duy nhất vào tên file của tất cả các file được biên dịch, cho phép tạo bộ đệm cache thuận tiện hơn:

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();

Sau khi tạo file đã được version, bạn sẽ không biết chính xác tên file là gì. Vì vậy, bạn có thể sử dụng hàm global helper `mix` của Laravel trong [views](/docs/{{version}}/views) để load asset được hash. Hàm `mix` sẽ tự động xác định tên hiện tại của file đã được hash:

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">

Vì các file đã được version thường không cần thiết trong quá trình phát triển, bạn có thể thêm điều kiện để tạo version là chỉ chạy trong khi `npm run production`:

    mix.js('resources/assets/js/app.js', 'public/js');

    if (mix.inProduction()) {
        mix.version();
    }

<a name="browsersync-reloading"></a>
## Browsersync Reloading

[BrowserSync](https://browsersync.io/) có thể tự động theo dõi các file của bạn để thay đổi và đưa các thay đổi của bạn vào trình duyệt mà không yêu cầu refresh thủ công. Bạn có thể kích hoạt hỗ trợ bằng cách gọi phương thức `mix.browserSync()`:

    mix.browserSync('my-domain.test');

    // Or...

    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.test'
    });

Bạn có thể pass một chuỗi (proxy) hoặc đối tượng (BrowserSync settings) cho phương thức này. Tiếp theo, khởi động server dev của Webpack bằng lệnh `npm run watch`. Bây giờ, khi bạn sửa đổi một script hoặc file PHP, trình duyệt sẽ tự động refresh trang ngay lập tức để phản ánh các thay đổi của bạn.

<a name="environment-variables"></a>
## Environment Variables

Bạn có thể đưa các biến môi trường vào Mix bằng cách thêm tiền tố vào file `.env` của bạn với `MIX_`:

    MIX_SENTRY_DSN_PUBLIC=http://example.com

Sau khi biến đã được định nghĩa trong file `.env` của bạn, bạn có thể truy cập thông qua đối tượng `process.env`. Nếu giá trị thay đổi trong khi bạn đang chạy một tác vụ `watch`, bạn sẽ cần khởi động lại tác vụ:

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>
## Thông báo

Khi sẵn sàng, Mix sẽ tự động hiển thị thông báo của hệ điều hành cho mỗi lần biên dịch. Điều này sẽ cung cấp cho bạn thông tin rằng liệu việc biên dịch có thành công hay không. Tuy nhiên, có thể có trường hợp bạn muốn vô hiệu hóa các thông báo này. Một ví dụ như vậy có thể kích hoạt Mix trên server production của bạn. Thông báo có thể bị vô hiệu hóa, thông qua phương thức `disableNotifications`.

    mix.disableNotifications();
