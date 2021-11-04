# Request Lifecycle

- [Giới thiệu](#introduction)
- [Tổng quan vòng đời](#lifecycle-overview)
- [Tập trung vào service providers](#focus-on-service-providers)

<a name="introduction"></a>
## Giới thiệu

Khi sử dụng bất kỳ công cụ nào trong thế giới thực, bạn sẽ cảm thấy tự tin hơn nếu bạn hiểu cách thức hoạt động của công cụ đó. Phát triển ứng dụng cũng không có gì khác. Khi bạn hiểu cách hoạt động của các công cụ phát triển, bạn sẽ cảm thấy thoải mái và tự tin hơn khi sử dụng các công cụ đó.

Mục tiêu của tài liệu này là cung cấp cho bạn một cái nhìn tổng quan về cách thức hoạt động của Laravel framework. Bằng cách hiểu rõ về tổng quan của framework, mọi thứ sẽ bớt "magic" hơn và bạn sẽ tự tin hơn khi xây dựng ứng dụng của bạn. Nếu bạn không hiểu tất cả, thì cũng đừng nản lòng! Chỉ cần cố gắng nắm bắt cơ bản những gì đang diễn ra, và kiến thức của bạn sẽ phát triển khi bạn khám phá các phần khác của tài liệu.

<a name="lifecycle-overview"></a>
## Tổng quan vòng đời

### Điều đầu tiên

Điểm vào đầu tiên cho tất cả các request tới application Laravel là file `public/index.php`. Tất cả các request được điều hướng đến file này theo cấu hình web server (Apache / Nginx) của bạn. File `index.php` không chứa nhiều code. Thay vào đó, nó là nơi khởi đầu để load các phần còn lại của framework.

File `index.php` sẽ load các định nghĩa từ Composer generated autoloader, và sau đó lấy instance application Laravel từ file `bootstrap/app.php`. Sau đó, là thực hiện tạo một instance của application / [service container](/docs/{{version}}/container).

### HTTP và Console Kernels

Tiếp theo, request sẽ được gửi đến HTTP kernel hoặc console kernel, tuỳ thuộc vào loại của request đến. Hai kernel này đóng vai trò là trung tâm của tất cả các request sẽ phải đi qua. Hiện tại, bạn hãy chỉ tập trung vào HTTP kernel đang được lưu trong file `app/Http/Kernel.php`.

HTTP kernel được extend từ class `Illuminate\Foundation\Http\Kernel`, class này định nghĩa môt danh sách `bootstrappers` sẽ được chạy trước khi request được xử lý. Các bootstrappers này sẽ cấu hình xử lý lỗi, cấu hình logging, [xác định môi trường của application](/docs/{{version}}/configuration#environment-configuration), và thực hiện các tác vụ khác cần được thực hiện trước khi request thực sự được xử lý.

HTTP kernel cũng định nghĩa một danh sách các HTTP [middleware](/docs/{{version}}/middleware) mà tất cả các request phải chạy qua trước khi được xử lý bởi application. Các middleware này sẽ xử lý việc đọc ghi [HTTP session](/docs/{{version}}/session), kiểm tra nếu application đang ở trong chế độ maintenance, [kiểm tra CSRF token](/docs/{{version}}/csrf), và hơn thế nữa.

Cấu trúc của phương thức `handle` trong HTTP kernel khá đơn giản: nó nhận vào một `Request` và trả về một `Response`. Hãy nghĩ đơn giản Kernel như là một hộp đen chứa toàn bộ code xử lý của application của bạn. Cung cấp cho nó một HTTP request và nó sẽ trả về một HTTP response.

#### Service Providers

Một trong những hành động khởi động Kernel quan trọng nhất là load các [service providers](/docs/{{version}}/providers) cho application của bạn. Tất cả các service providers cho application được cấu hình ở mảng `providers` trong file `config/app.php`. Đầu tiên, phương thức `register` của tất cả các providers đó sẽ được gọi, sau đó, khi tất cả các providers đã được đăng ký, thì phương thức` boot` sẽ được gọi.

Service providers sẽ chịu trách nhiệm khởi tạo tất cả các thành phần khác nhau của framework, chẳng hạn như database, queue, validation và route. Vì các thành phần này cấu hình mọi tính năng được cung cấp bởi framework, nên service provider là khía cạnh quan trọng nhất của toàn bộ quá trình khởi tạo của Laravel.

#### Gửi request

Khi application đã được khởi động và tất cả các service providers đã được đăng ký, `Request` sẽ được gửi đến router để được gửi đi. Route sẽ gửi request đến một route hoặc controller khác, cũng như chạy bất kỳ middleware nào nếu cần thiết.

<a name="focus-on-service-providers"></a>
## Tập trung vào service providers

Service providers là chìa khóa để khởi động một apllication Laravel. Đầu tiên, Instance application sẽ được khởi tạo, sau đó các service provider sẽ được đăng ký và request sẽ được xử lý bởi application đã được khởi tạo. Nó thực sự đơn giản!

Nắm vững cách thức một ứng dụng Laravel được xây dựng và được khởi động thông qua các service providers là rất có giá trị. Dĩ nhiên, các service providers mặc định của application của bạn sẽ được lưu trữ trong thư mục `app/Providers`.

Mặc định, `AppServiceProvider` là trống. Provider này là một nơi tuyệt vời để thêm phần khởi động dành riêng cho application của bạn và các service container bindings. Tất nhiên, đối với các ứng dụng lớn, bạn có thể muốn tạo nhiều service providers, mỗi loại lại có cách khởi động khác nhau.
