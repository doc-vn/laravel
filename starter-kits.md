# Starter Kits

- [Giới thiệu](#introduction)
- [Laravel Breeze](#laravel-breeze)
    - [Cài đặt](#laravel-breeze-installation)
    - [Breeze và Blade](#breeze-and-blade)
    - [Breeze và React / Vue](#breeze-and-inertia)
    - [Breeze và Next.js / API](#breeze-and-next)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## Giới thiệu

Để giúp bạn bắt đầu nhanh xây dựng ứng dụng Laravel mới của bạn, chúng tôi rất vui được cung cấp bộ công cụ khởi tạo ứng dụng và xác thực. Bộ công cụ này tự động tạo khung sẵn cho ứng dụng của bạn với các route, controller và view mà bạn cần để đăng ký và xác thực người dùng ứng dụng của bạn.

Mặc dù bạn có thể thoải mái sử dụng những bộ công cụ khởi tạo này nhưng chúng không bắt buộc. Bạn có thể tự do xây dựng ứng dụng của riêng bạn ngay từ đầu bằng cách cài đặt một bản mới của Laravel. Dù bằng cách nào, chúng tôi biết bạn sẽ xây dựng được điều gì đó rất tuyệt vời!

<a name="laravel-breeze"></a>
## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze) là cách triển khai đơn giản, tối thiểu tất cả các [tính năng xác thực](/docs/{{version}}/authentication), bao gồm đăng nhập, đăng ký, reset mật khẩu, xác minh email và xác nhận mật khẩu. Ngoài ra, Breeze còn có một trang "profile" đơn giản nơi mà người dùng có thể cập nhật tên, địa chỉ email và mật khẩu của họ.

Layer view mặc định của Laravel Breeze được tạo thành từ [template Blade](/docs/{{version}}/blade) đơn giản với [Tailwind CSS](https://tailwindcss.com). Hoặc, Breeze cũng có thể xây dựng ứng dụng của bạn bằng Vue hoặc React và [Inertia](https://inertiajs.com).

Breeze cung cấp một điểm khởi đầu tuyệt vời để bắt đầu một ứng dụng Laravel mới và cũng là một sự lựa chọn tuyệt vời cho các dự án có kế hoạch đưa Blade của họ lên một tầm cao mới với [Laravel Livewire](https://laravel-livewire.com).

<img src="https://laravel.com/img/docs/breeze-register.png">

#### Laravel Bootcamp

Nếu bạn mới làm quen với Laravel, vui lòng tham gia [Laravel Bootcamp](https://bootcamp.laravel.com). Laravel Bootcamp sẽ hướng dẫn bạn xây dựng ứng dụng Laravel bằng Breeze từ những bước đầu tiên. Đó là một cách tuyệt vời để tìm hiểu mọi thứ mà Laravel và Breeze cung cấp.

<a name="laravel-breeze-installation"></a>
### Cài đặt

Trước tiên, bạn nên [tạo một ứng dụng Laravel mới](/docs/{{version}}/installation), cấu hình cơ sở dữ liệu và chạy [migration cơ sở dữ liệu](/docs/{{version}}/migrations). Khi bạn đã tạo xong một ứng dụng Laravel mới, bạn có thể cài đặt Laravel Breeze bằng Composer:

```shell
composer require laravel/breeze --dev
```

Sau khi Breeze đã được cài đặt, bạn có thể xây dựng ứng dụng của bạn bằng một trong các "stack" Breeze được thảo luận trong tài liệu bên dưới.

<a name="breeze-and-blade"></a>
### Breeze và Blade

Sau khi Composer đã cài đặt xong package Laravel Breeze, bạn có thể chạy lệnh Artisan `breeze:install`. Lệnh này sẽ export ra các view xác thực, route, controller và các resource khác cho ứng dụng của bạn. Laravel Breeze sẽ export tất cả các code của nó vào ứng dụng của bạn để bạn có toàn quyền kiểm soát và hiển thị các tính năng cũng như cách triển khai của nó.

"Stack" Breeze mặc định là stack Blade, sử dụng [templates Blade](/docs/{{version}}/blade) đơn giản để render frontend cho ứng dụng. Stack Blade có thể được cài đặt bằng cách gõ lệnh `breeze:install` mà không cần thêm bất kỳ tham số nào khác. Sau khi Breeze đã được cài đặt xong, bạn cũng nên biên dịch các asset frontend của ứng dụng:

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

Tiếp theo, bạn có thể điều hướng đến URL `/login` hoặc URL `/register` của ứng dụng trong trình duyệt web của bạn. Tất cả các route của Breeze đều được định nghĩa trong file `routes/auth.php`.

<a name="dark-mode"></a>
#### Dark Mode

Nếu bạn muốn Breeze cũng hỗ trợ "chế độ dark mode" khi tạo giao diện người dùng cho ứng dụng, bạn chỉ cần cung cấp thêm lệnh `--dark` khi thực hiện lệnh `breeze:install`:

```shell
php artisan breeze:install --dark
```

> **Note**
> Để tìm hiểu thêm về cách compile CSS và JavaScript cho ứng dụng của bạn, hãy xem [tài liệu về Vite](/docs/{{version}}/vite#running-vite) của Laravel.

<a name="breeze-and-inertia"></a>
### Breeze và React / Vue

Laravel Breeze cũng cung cấp scaffolding React và Vue thông qua một implementation frontend [Inertia](https://inertiajs.com). Inertia cho phép bạn xây dựng các ứng dụng single-page React và Vue hiện đại bằng cách sử dụng routing và controller giống như ở một server-side cổ điển.

Inertia cho phép bạn tận hưởng sức mạnh frontend của React và Vue kết hợp với năng suất backend đáng kinh ngạc của Laravel và trình biên dịch [Vite](https://vitejs.dev) cực nhanh. Để sử dụng Inertia stack, hãy chỉ định `vue` hoặc `react` làm stack mong muốn của bạn khi thực hiện lệnh Artisan `breeze:install`. Sau khi scaffolding của Breeze đã được cài đặt xong, bạn cũng nên biên dịch các asset frontend của ứng dụng:

```shell
php artisan breeze:install vue

# Or...

php artisan breeze:install react

php artisan migrate
npm install
npm run dev
```

Tiếp theo, bạn có thể điều hướng đến URL `/login` hoặc `/register` của ứng dụng trên trình duyệt web của bạn. Tất cả các route của Breeze đều đã được định nghĩa trong file `routes/auth.php`.

<a name="server-side-rendering"></a>
#### Server-Side Rendering

Nếu bạn muốn Breeze hỗ trợ [Inertia SSR](https://inertiajs.com/server-side-rendering), bạn có thể cung cấp tùy chọn `ssr` khi gọi lệnh `breeze:install`:

```shell
php artisan breeze:install vue --ssr
php artisan breeze:install react --ssr
```

<a name="breeze-and-next"></a>
### Breeze và Next.js / API

Laravel Breeze cũng có thể xây dựng một API xác thực sẵn sàng cho xác thực các ứng dụng JavaScript hiện đại, chẳng hạn như các ứng dụng được tạo bởi [Next](https://nextjs.org), [Nuxt](https://nuxt.com) và các ứng dụng khác. Để bắt đầu, hãy chỉ định stack `api` làm stack mong muốn của bạn khi thực hiện lệnh Artisan `breeze:install`:

```shell
php artisan breeze:install api

php artisan migrate
```

Trong khi cài đặt, Breeze sẽ thêm biến môi trường `FRONTEND_URL` vào file `.env` của ứng dụng của bạn. URL này phải là URL của ứng dụng JavaScript của bạn. Thông thường, đây sẽ là `http://localhost:3000` khi trong quá trình phát triển local. Ngoài ra, bạn nên đảm bảo là `APP_URL` đã được set thành `http://localhost:8000`, đây là URL mặc định được lệnh Artisan `serve` sử dụng.

<a name="next-reference-implementation"></a>
#### Next.js Reference Implementation

Cuối cùng, bạn đã sẵn sàng để ghép phần backend với phần frontend mà bạn chọn. Việc triển khai reference của Next.js của frontend Breeze [đã có trên GitHub](https://github.com/laravel/breeze-next). Phần frontend này được Laravel duy trì và chứa giao diện người dùng giống như Blade truyền thống và Inertia stack do Breeze cung cấp.

<a name="laravel-jetstream"></a>
## Laravel Jetstream

Trong khi Laravel Breeze cung cấp một điểm khởi đầu đơn giản và tối thiểu để xây dựng một ứng dụng Laravel mới, thì Jetstream lại tăng cường chức năng đó bằng các tính năng mạnh mẽ hơn và các stack công nghệ về frontend. **Đối với những người mới làm quen với Laravel, chúng tôi khuyên bạn chỉ nên tìm hiểu các kiến thức cơ bản về Laravel Breeze trước khi chuyển sang Laravel Jetstream.**

Jetstream cung cấp một scaffolding ứng dụng được thiết kế đẹp mắt cho Laravel và chứa các form đăng nhập, đăng ký, xác minh email, xác thực hai yếu tố, quản lý session, hỗ trợ API thông qua Laravel Sanctum và tùy chọn quản lý team. Jetstream được thiết kế bằng [Tailwind CSS](https://tailwindcss.com) và cho bạn lựa chọn [Livewire](https://laravel-livewire.com) hoặc [Inertia](https://inertiajs.com) để chạy frontend scaffolding.

Bạn có thể tìm thấy tài liệu đầy đủ về cách cài đặt Laravel Jetstream trong [tài liệu Jetstream chính thức](https://jetstream.laravel.com/introduction.html).
