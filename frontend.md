# JavaScript & CSS Scaffolding

- [Giới thiệu](#introduction)
- [Viết CSS](#writing-css)
- [Viết JavaScript](#writing-javascript)
    - [Viết Vue component](#writing-vue-components)
    - [Dùng React](#using-react)

<a name="introduction"></a>
## Giới thiệu

Mặc dù Laravel không ra lệnh bạn phải sử dụng JavaScript hoặc CSS pre-processors nào, nhưng nó cung cấp một điểm khởi đầu cơ bản bằng cách sử dụng [Bootstrap](https://getbootstrap.com/) và [Vue](https://vuejs.org) sẽ hữu ích cho nhiều application. Mặc định, Laravel sử dụng [NPM](https://www.npmjs.org) để cài đặt cả hai package frontend này.

#### CSS

[Laravel Mix](/docs/{{version}}/mix) cung cấp một API rõ ràng, dễ hiểu khi biên dịch SASS hoặc Less, là các extension của CSS có thêm biến, mixins và các chức năng mạnh mẽ khác giúp làm việc với CSS thêm thú vị hơn. Trong tài liệu này, chúng ta sẽ thảo luận ngắn gọn về việc biên dịch CSS nói chung; tuy nhiên, bạn nên tham khảo [Tài liệu Laravel Mix](/docs/{{version}}/mix) đầy đủ để biết thêm thông tin về việc biên dịch SASS hoặc Less.

#### JavaScript

Laravel không yêu cầu bạn sử dụng một framework hoặc một thư viện JavaScript cụ thể nào để xây dựng các application của bạn. Thực tế, bạn không cần phải sử dụng JavaScript ở tất cả. Tuy nhiên, Laravel có chứa một số trợ giúp cơ bản để làm nó dễ dàng hơn khi bắt đầu viết JavaScript bằng thư viện [Vue](https://vuejs.org). Vue cung cấp một API dễ hiểu để xây dựng các application JavaScript mạnh mẽ bằng cách sử dụng các component. Cũng như CSS, chúng ta có thể sử dụng Laravel Mix để dễ dàng biên dịch các component JavaScript thành một file JavaScript sẵn sàng cho trình duyệt.

#### Removing The Frontend Scaffolding

Nếu bạn muốn loại bỏ trợ giúp frontend ra khỏi application của bạn, bạn có thể sử dụng lệnh Artisan `preset`. Lệnh này, khi được kết hợp với tùy chọn `none`, sẽ xóa trợ giúp Bootstrap và Vue khỏi application của bạn, chỉ để lại một file SASS trống và một vài thư viện tiện ích JavaScript phổ biến:

    php artisan preset none

<a name="writing-css"></a>
## Viết CSS

File `package.json` của Laravel có chứa package `bootstrap-sass` để giúp bạn bắt đầu tạo một trang cho application của bạn bằng Bootstrap. Tuy nhiên, bạn có thêm hoặc xóa thoải mái các package này ra khỏi file `package.json` cho application của bạn. Bạn không bắt buộc phải sử dụng framework Bootstrap để xây dựng application Laravel của bạn - nó được cung cấp như là một điểm khởi đầu tốt cho những người chọn sử dụng nó.

Trước khi biên dịch CSS của bạn, hãy cài đặt các library fontent vào project của bạn bằng cách sử dụng [Node package manager (NPM)](https://www.npmjs.org):

    npm install

Khi các library đã được cài đặt xong bằng cách sử dụng `npm install`, bạn có thể biên dịch các file SASS của mình thành code CSS bằng cách sử dụng [Laravel Mix](/docs/{{version}}/mix#working-with-stylesheets). Lệnh `npm run dev` sẽ xử lý theo các hướng dẫn đã được viết trong file `webpack.mix.js` của bạn. Thông thường, CSS đã biên dịch của bạn sẽ được lưu trong thư mục `public/css`:

    npm run dev

`webpack.mix.js` mặc định đi kèm với Laravel sẽ biên dịch file SASS `resources/assets/sass/app.scss`. File `app.scss` này sẽ import một file của các biến SASS và load Bootstrap, cung cấp một điểm khởi đầu tốt cho hầu hết các application. Hãy thoải mái tùy chỉnh file `app.scss` theo cách bạn muốn hoặc thậm chí sử dụng một pre-processor hoàn toàn khác bằng cách [cấu hình Laravel Mix](/docs/{{version}}/mix).

<a name="writing-javascript"></a>
## Viết JavaScript

Tất cả các library JavaScript được yêu cầu bởi application của bạn có thể được tìm thấy trong file `package.json` ở thư mục gốc của project. File này tương tự như file `composer.json` ngoại trừ việc nó định nghĩa các library của JavaScript thay vì library của PHP. Bạn có thể cài đặt các library này bằng [Node package manager (NPM)](https://www.npmjs.org):

    npm install

> {tip} Theo mặc định, file `package.json` của Laravel sẽ bao gồm một package gói như `vue` và `axios` để giúp bạn bắt đầu xây dựng application JavaScript của bạn. Bạn có thêm hoặc xóa thoải mái các package này ra khỏi file `package.json` cho application của bạn.

Khi các package đã được cài đặt xong, bạn có thể sử dụng lệnh `npm run dev` để [biên dịch your assets](/docs/{{version}}/mix). Webpack là một gói mô-đun cho các application JavaScript. Khi bạn chạy lệnh `npm run dev`, Webpack sẽ thực thi các hướng dẫn đã được ghi trong file  `webpack.mix.js` của bạn:

    npm run dev

Theo mặc định, file `webpack.mix.js` của Laravel biên dịch SASS của bạn và file `resources/assets/js/app.js`. Trong file `app.js`, bạn có thể đăng ký các Vue component của bạn hoặc, nếu bạn thích một framework khác, hãy cấu hình application JavaScript của riêng bạn. Các file JavaScript đã được biên dịch thường sẽ được lưu trong thư mục `public/js`.

> {tip} File `app.js` sẽ load file `resources/assets/js/bootstrap.js` để khởi động và cấu hình Vue, Axios, jQuery và tất cả các library JavaScript khác. Nếu bạn muốn cấu hình thêm các library JavaScript khác, bạn có thể làm như vậy trong file này.

<a name="writing-vue-components"></a>
### Viết Vue component

Theo mặc định, các application Laravel mới có chứa một component Vue `ExampleComponent.vue` mẫu nằm trong thư mục `resources/assets/js/components`. File `ExampleComponent.vue` là một ví dụ về [single file Vue component](https://vuejs.org/guide/single-file-components) định nghĩa template JavaScript và HTML của nó trong cùng một file. Các single file component cung cấp một cách tiếp cận rất thuận tiện để xây dựng các application điều khiển JavaScript. Example component được đăng ký trong file `app.js` của bạn:

    Vue.component(
        'example-component',
        require('./components/ExampleComponent.vue')
    );

Để sử dụng component trong application của bạn, bạn có thể đặt nó vào một trong các template HTML của bạn. Ví dụ, sau khi chạy lệnh Artisan `make:auth` để tạo màn hình authentication và màn hình đăng ký cho application của bạn, bạn có thể đặt component vào template Blade `home.blade.php`:

    @extends('layouts.app')

    @section('content')
        <example-component></example-component>
    @endsection

> {tip} Hãy nhớ rằng, bạn nên chạy lệnh `npm run dev` mỗi khi bạn thay đổi component Vue. Hoặc, bạn có thể chạy lệnh `npm run watch` để theo dõi và tự động biên dịch lại các component của bạn mỗi khi chúng được sửa đổi.

Tất nhiên, nếu bạn muốn tìm hiểu thêm về cách viết các Vue component, bạn nên đọc [Vue documentation](https://vuejs.org/guide/), cung cấp tổng quan kỹ lưỡng, dễ đọc về toàn bộ Vue framework.

<a name="using-react"></a>
### Dùng React

Nếu bạn thích sử dụng React để xây dựng application JavaScript của mình, thì Laravel làm nó dễ dàng để chuyển đổi trợ giúp Vue với trợ giúp React. Trên bất kỳ application Laravel mới nào, bạn có thể sử dụng lệnh `preset` với tùy chọn `react`:

    php artisan preset react

Lệnh này sẽ loại bỏ trợ giúp Vue và thay thế nó bằng trợ giúp React, bao gồm một example component.
