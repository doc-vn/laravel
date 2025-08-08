# Laravel Mix

- [Giới thiệu](#introduction)

<a name="introduction"></a>
## Giới thiệu

[Laravel Mix](https://github.com/laravel-mix/laravel-mix) là một package được phát triển bởi Jeffrey Way [Laracasts](https://laracasts.com), cung cấp một API dễ hiểu để định nghĩa các bước xây dựng [webpack](https://webpack.js.org) cho application Laravel của bạn bằng cách sử dụng một số CSS phổ biến và JavaScript pre-processors.

Nói cách khác, Mix giúp bạn dễ dàng biên dịch và thu nhỏ các file CSS và JavaScript trong ứng dụng của bạn. Thông qua cách kết hợp nhiều phương thức đơn giản, bạn có thể dễ dàng định nghĩa asset pipeline của bạn. Ví dụ:

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

Nếu bạn đã từng bối rối và choáng ngợp khi bắt đầu với Webpack và biên dịch asset, bạn sẽ thích Laravel Mix. Tuy nhiên, bạn không bắt buộc phải sử dụng nó trong khi phát triển application của bạn; Bạn có thể thoải mái sử dụng bất kỳ công cụ asset pipeline nào bạn muốn, hoặc thậm chí không dùng gì cả.

> [!NOTE]
> Vite đã thay thế Laravel Mix trong các cài đặt mới của Laravel. Để biết thêm về tài liệu Mix, vui lòng truy cập trang web [Laravel Mix chính thức](https://laravel-mix.com/). Nếu bạn muốn chuyển sang Vite, vui lòng xem [hướng dẫn chuyển sang Vite](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite) của chúng tôi.
