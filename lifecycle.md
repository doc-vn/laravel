# Request Lifecycle

- [Giới thiệu](#introduction)
- [Tổng quan vòng đời](#lifecycle-overview)
    - [Bước đầu tiên](#first-steps)
    - [HTTP / Console Kernels](#http-console-kernels)
    - [Service Providers](#service-providers)
    - [Routing](#routing)
    - [Finishing Up](#finishing-up)
- [Tập trung vào service providers](#focus-on-service-providers)

<a name="introduction"></a>
## Giới thiệu

Khi sử dụng bất kỳ công cụ nào trong thế giới thực, bạn sẽ cảm thấy tự tin hơn nếu bạn hiểu cách thức hoạt động của công cụ đó. Phát triển ứng dụng cũng không có gì khác. Khi bạn hiểu cách hoạt động của các công cụ phát triển, bạn sẽ cảm thấy thoải mái và tự tin hơn khi sử dụng các công cụ đó.

Mục tiêu của tài liệu này là cung cấp cho bạn một cái nhìn tổng quan về cách thức hoạt động của Laravel framework. Bằng cách hiểu rõ về tổng quan của framework, mọi thứ sẽ bớt "magic" hơn và bạn sẽ tự tin hơn khi xây dựng ứng dụng của bạn. Nếu bạn không hiểu tất cả, thì cũng đừng nản lòng! Chỉ cần cố gắng nắm bắt cơ bản những gì đang diễn ra, và kiến thức của bạn sẽ phát triển khi bạn khám phá các phần khác của tài liệu.

<a name="lifecycle-overview"></a>
## Tổng quan vòng đời

<a name="first-steps"></a>
### Bước đầu tiên

Điểm vào đầu tiên cho tất cả các request tới application Laravel là file `public/index.php`. Tất cả các request được điều hướng đến file này theo cấu hình web server (Apache / Nginx) của bạn. File `index.php` không chứa nhiều code. Thay vào đó, nó là nơi khởi đầu để load các phần còn lại của framework.

File `index.php` sẽ load các định nghĩa từ Composer generated autoloader, và sau đó lấy instance application Laravel từ `bootstrap/app.php`. Sau đó, là thực hiện tạo một instance của application / [service container](/docs/{{version}}/container).

<a name="http-console-kernels"></a>
### HTTP và Console Kernels

Tiếp theo, request sẽ được gửi đến HTTP kernel hoặc console kernel, tuỳ thuộc vào loại của request đến. Hai kernel này đóng vai trò là trung tâm của tất cả các request sẽ phải đi qua. Hiện tại, bạn hãy chỉ tập trung vào HTTP kernel đang được lưu trong file `app/Http/Kernel.php`.

HTTP kernel được extend từ class `Illuminate\Foundation\Http\Kernel`, class này định nghĩa môt danh sách `bootstrappers` sẽ được chạy trước khi request được xử lý. Các bootstrappers này sẽ cấu hình xử lý lỗi, cấu hình logging, [xác định môi trường của application](/docs/{{version}}/configuration#environment-configuration), và thực hiện các tác vụ khác cần được thực hiện trước khi request thực sự được xử lý. Thông thường, các class này xử lý cấu hình nội bộ tring Laravel mà bạn không cần phải lo lắng.

HTTP kernel cũng định nghĩa một danh sách các HTTP [middleware](/docs/{{version}}/middleware) mà tất cả các request phải chạy qua trước khi được xử lý bởi application. Các middleware này sẽ xử lý việc đọc ghi [HTTP session](/docs/{{version}}/session), kiểm tra nếu application đang ở trong chế độ maintenance, [kiểm tra CSRF token](/docs/{{version}}/csrf), và hơn thế nữa. Chúng ta sẽ nói thêm về những điều này sớm.

Cấu trúc của phương thức `handle` trong HTTP kernel khá đơn giản: nó nhận vào một `Request` và trả về một `Response`. Hãy nghĩ đơn giản Kernel như là một hộp đen chứa toàn bộ code xử lý của application của bạn. Cung cấp cho nó một HTTP request và nó sẽ trả về một HTTP response.

<a name="service-providers"></a>
#### Service Providers

Một trong những hành động khởi động Kernel quan trọng nhất là load các [service providers](/docs/{{version}}/providers) cho application của bạn. Các service provider chịu trách nhiệm khởi động tất cả các thành phần khác nhau của framework, chẳng hạn như cơ sở dữ liệu, hàng đợi, xác thực và các thành phần routing. Tất cả các service providers cho application được cấu hình ở mảng `providers` trong file `config/app.php`.

Laravel sẽ lặp danh sách của các provider này và khởi tạo từng provider một trong số họ. Sau khi khởi tạo các provider, phương thức `register` sẽ được gọi trên tất cả các provider đó. Và sau đó, khi tất cả các provider đã được đăng ký, phương thức `boot` sẽ được gọi trên mỗi provider. Điều này là do các service provider có thể phụ thuộc vào các liên kết container đang được đăng ký và khả dụng vào thời điểm phương thức `boot` của nó được thực thi.

Về cơ bản, mọi tính năng chính do Laravel cung cấp đều được khởi động và cấu hình bởi service provider. Vì nó khởi động và cấu hình rất nhiều tính năng được cung cấp bởi framework, nên các service provider là khía cạnh quan trọng nhất của toàn bộ quy trình khởi động Laravel.

<a name="routing"></a>
### Routing

Một trong những service provider quan trọng nhất trong ứng dụng của bạn là `App\Providers\RouteServiceProvider`. Service provider này load các file route có trong thư mục `routes` của ứng dụng của bạn. Hãy tiếp tục, mở code `RouteServiceProvider` và xem nó hoạt động như thế nào!

Khi application đã được khởi động và tất cả các service providers đã được đăng ký, `Request` sẽ được gửi đến router để được gửi đi. Route sẽ gửi request đến một route hoặc controller khác, cũng như chạy bất kỳ middleware nào nếu cần thiết.

Middleware cung cấp một cơ chế thuận tiện để lọc hoặc kiểm tra các request HTTP đi vào ứng dụng của bạn. Ví dụ: Laravel có chứa một middleware dùng để xác minh xem người dùng ứng dụng của bạn có được xác thực hay chưa. Nếu người dùng chưa được xác thực, middleware sẽ chuyển hướng người dùng đến màn hình đăng nhập. Tuy nhiên, nếu người dùng đã được xác thực, middleware sẽ cho phép request tiến sâu hơn vào ứng dụng của bạn. Một số middleware được gán cho tất cả các route trong ứng dụng, giống như middleware được định nghĩa trong thuộc tính `$middleware` của HTTP kernel của bạn, trong khi một số middleware khác chỉ được gán cho một số route hoặc nhóm route cụ thể. Bạn có thể tìm hiểu thêm về các middleware này bằng cách đọc [tài liệu middleware](/docs/{{version}}/middleware) đầy đủ.

Nếu request được pass qua tất cả middleware được chỉ định cho route đó, phương thức route hoặc controller sẽ được thực thi và response do route hoặc phương thức controller đó trả về sẽ được gửi lại thông qua chuỗi middleware của route.

<a name="finishing-up"></a>
### Finishing Up

Sau khi phương thức route hoặc controller trả về một response, thì response đó sẽ được truyền ngược ra ngoài thông qua middleware của route, tạo ra cơ hội cho phép ứng dụng của bạn sửa hoặc kiểm tra trước khi response được gửi đi.

Cuối cùng, sau khi response quay trở lại thông qua middleware, phương thức `handle` của HTTP kernel sẽ trả về đối tượng response đó và file `index.php` gọi phương thức `send` trên response được trả về. Phương thức `send` sẽ gửi nội dung của response đến trình duyệt web của người dùng. Chúng ta đã kết thúc hành trình của mình trong toàn bộ vòng đời request của Laravel!

<a name="focus-on-service-providers"></a>
## Tập trung vào service providers

Service provider là chìa khóa để khởi động một apllication Laravel. Đầu tiên, instance application sẽ được khởi tạo, sau đó các service provider sẽ được đăng ký và request sẽ được xử lý bởi application đã được khởi tạo. Nó thực sự đơn giản!

Nắm vững cách thức một ứng dụng Laravel được xây dựng và được khởi động thông qua các service providers là rất có giá trị. Các service providers mặc định của application của bạn sẽ được lưu trữ trong thư mục `app/Providers`.

Mặc định, `AppServiceProvider` là trống. Provider này là một nơi tuyệt vời để thêm phần khởi động dành riêng cho application của bạn và các service container bindings. Đối với các ứng dụng lớn, bạn có thể muốn tạo nhiều service providers, mỗi loại lại có cách khởi động cho các service cụ thể được ứng dụng của bạn sử dụng.
