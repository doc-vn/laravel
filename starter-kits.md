# Starter Kits

- [Giới thiệu](#introduction)
- [Laravel Breeze](#laravel-breeze)
    - [Cài đặt](#laravel-breeze-installation)
    - [Breeze và Inertia](#breeze-and-inertia)
    - [Breeze và Next.js / API](#breeze-and-next)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## Giới thiệu

Để giúp bạn bắt đầu nhanh xây dựng ứng dụng Laravel mới của bạn, chúng tôi rất vui được cung cấp bộ công cụ khởi tạo ứng dụng và xác thực. Bộ công cụ này tự động tạo khung sẵn cho ứng dụng của bạn với các route, controller và view mà bạn cần để đăng ký và xác thực người dùng ứng dụng của bạn.

Mặc dù bạn có thể thoải mái sử dụng những bộ công cụ khởi tạo này nhưng chúng không bắt buộc. Bạn có thể tự do xây dựng ứng dụng của riêng bạn ngay từ đầu bằng cách cài đặt một bản mới của Laravel. Dù bằng cách nào, chúng tôi biết bạn sẽ xây dựng được điều gì đó rất tuyệt vời!

<a name="laravel-breeze"></a>
## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze) là cách triển khai đơn giản, tối thiểu tất cả [tính năng xác thực](/docs/{{version}}/authentication), bao gồm đăng nhập, đăng ký, reset mật khẩu, xác minh email và xác nhận mật khẩu. Layer view mặc định của Laravel Breeze được tạo thành từ [Blade](/docs/{{version}}/blade) đơn giản với [Tailwind CSS](https://tailwindcss.com).

Breeze cung cấp một điểm khởi đầu tuyệt vời để bắt đầu một ứng dụng Laravel mới và cũng là sự lựa chọn tuyệt vời cho các dự án có kế hoạch đưa Blade của họ lên một tầm cao mới với [Laravel Livewire](https://laravel-livewire.com).

<a name="laravel-breeze-installation"></a>
### Cài đặt

Trước tiên, bạn nên [tạo một ứng dụng Laravel mới](/docs/{{version}}/installation), cấu hình cơ sở dữ liệu và chạy [migration cơ sở dữ liệu](/docs/{{version}}/migrations):

```bash
curl -s https://laravel.build/example-app | bash

cd example-app

php artisan migrate
```

Khi bạn đã tạo xong một ứng dụng Laravel mới, bạn có thể cài đặt Laravel Breeze bằng Composer:

```bash
composer require laravel/breeze:1.9.2
```

Sau khi Composer đã cài đặt xong package Laravel Breeze, bạn có thể chạy lệnh Artisan `breeze:install`. Lệnh này sẽ export ra các view xác thực, route, controller và các resource khác cho ứng dụng của bạn. Laravel Breeze sẽ export tất cả các code của nó vào ứng dụng của bạn để bạn có toàn quyền kiểm soát và hiển thị các tính năng cũng như cách triển khai của nó. Sau khi Breeze đã được cài đặt, bạn cũng nên compile lại asset của bạn để tạo file CSS cho ứng dụng:

```nothing
php artisan breeze:install

npm install
npm run dev
php artisan migrate
```

Tiếp theo, bạn có thể điều hướng đến URL `/login` hoặc URL `/register` của ứng dụng trong trình duyệt web của bạn. Tất cả các route của Breeze đều được định nghĩa trong file `routes/auth.php`.

> {tip} Để tìm hiểu thêm về cách compile CSS và JavaScript cho ứng dụng của bạn, hãy xem [tài liệu về Laravel Mix](/docs/{{version}}/mix#running-mix).

<a name="breeze-and-inertia"></a>
### Breeze và Inertia

Laravel Breeze cũng cung cấp một triển khai frontend [Inertia.js](https://inertiajs.com) do Vue hoặc React hỗ trợ. Để sử dụng Inertia stack, hãy chỉ định `vue` hoặc `react` làm stack mong muốn của bạn khi thực hiện lệnh Artisan `breeze:install`:

```nothing
php artisan breeze:install vue

// Or...

php artisan breeze:install react

npm install
npm run dev
php artisan migrate
```

<a name="breeze-and-next"></a>
### Breeze và Next.js / API

Laravel Breeze cũng có thể xây dựng một API xác thực sẵn sàng cho xác thực các ứng dụng JavaScript hiện đại, chẳng hạn như các ứng dụng được tạo bởi [Next](https://nextjs.org), [Nuxt](https://nuxt.com) và các ứng dụng khác. Để bắt đầu, hãy chỉ định stack `api` làm stack mong muốn của bạn khi thực hiện lệnh Artisan `breeze:install`:

```nothing
php artisan breeze:install api

php artisan migrate
```

Trong khi cài đặt, Breeze sẽ thêm biến môi trường `FRONTEND_URL` vào file `.env` của ứng dụng của bạn. URL này phải là URL của ứng dụng JavaScript của bạn. Thông thường, đây sẽ là `http://localhost:3000` khi trong quá trình phát triển local.

<a name="next-reference-implementation"></a>
#### Next.js Reference Implementation

Cuối cùng, bạn đã sẵn sàng để ghép phần backend với phần frontend mà bạn chọn. Việc triển khai reference của Next.js của frontend Breeze [đã có trên GitHub](https://github.com/laravel/breeze-next). Phần frontend này được Laravel duy trì và chứa giao diện người dùng giống như Blade truyền thống và Inertia stack do Breeze cung cấp.

<a name="laravel-jetstream"></a>
## Laravel Jetstream

Trong khi Laravel Breeze cung cấp một điểm khởi đầu đơn giản và tối thiểu để xây dựng một ứng dụng Laravel mới, thì Jetstream lại tăng cường chức năng đó bằng các tính năng mạnh mẽ hơn và các stack công nghệ về frontend. **Đối với những người mới làm quen với Laravel, chúng tôi khuyên bạn chỉ nên tìm hiểu các kiến thức cơ bản về Laravel Breeze trước khi chuyển sang Laravel Jetstream.**

Jetstream cung cấp một scaffolding ứng dụng được thiết kế đẹp mắt cho Laravel và chứa các form đăng nhập, đăng ký, xác minh email, xác thực hai yếu tố, quản lý session, hỗ trợ API thông qua Laravel Sanctum và tùy chọn quản lý team. Jetstream được thiết kế bằng [Tailwind CSS](https://tailwindcss.com) và cho bạn lựa chọn [Livewire](https://laravel-livewire.com) hoặc [Inertia.js](https://inertiajs.com) để chạy frontend scaffolding.

Bạn có thể tìm thấy tài liệu đầy đủ về cách cài đặt Laravel Jetstream trong [tài liệu Jetstream chính thức](https://jetstream.laravel.com/introduction.html).
