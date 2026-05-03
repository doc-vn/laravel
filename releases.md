# Release Notes

- [Cấu trúc phiên bản](#versioning-scheme)
- [Chính sách hỗ trợ](#support-policy)
- [Laravel 12](#laravel-12)

<a name="versioning-scheme"></a>
## Cấu trúc phiên bản

Laravel và các package khác của nó tuân theo [Phiên bản Semantic](https://semver.org). Các phiên bản được phát hành chính thức của framework được phát hành một năm một lần (~Q1), trong khi các bản phát hành nhỏ hơn và các bản sửa lỗi có thể được phát hành thường xuyên hơn, có thể là mỗi tuần. Các bản phát hành nhỏ và các bản sửa lỗi sẽ **không bao giờ** chứa các thay đổi mà có thể dẫn đến hệ thống của bạn bị lỗi.

Khi sủ dụng framework Laravel hoặc các component của nó từ application của bạn hoặc từ package, bạn phải luôn luôn sử dụng một ràng buộc phiên bản, chẳng hạn như là `^12.0`, Vì các bản phát hành chính thức của Laravel có thể chứa các thay đổi mà có thể làm hệ thống của bạn bị lỗi. Tuy nhiên, chúng tôi sẽ cố gắng đảm bảo rằng: bạn có thể cập nhật lên bản phát hành chính thức trong một ngày hoặc ít hơn.

<a name="named-arguments"></a>
#### Named Arguments

[Đặt tên cho tham số](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) không nằm trong nguyên tắc tương thích ngược của Laravel. Chúng tôi có thể đổi tên các tham số bất cứ khi nào để cải thiện codebase của Laravel. Do đó, việc sử dụng các kiểu đặt tên cho tham số khi gọi các phương thức của Laravel nên được thực hiện một cách cẩn trọng và nên hiểu rằng tên tham số có thể thay đổi trong tương lai.

<a name="support-policy"></a>
## Chính sách hỗ trợ

Đối với tất cả các bản phát hành chính thức, các bản sửa lỗi sẽ được cung cấp trong 18 tháng và các bản sửa lỗi bảo mật được cung cấp trong 2 năm. Đối với tất cả các thư viện, chỉ bản phát hành chính thức mới nhất mới nhận được các bản sửa lỗi. Ngoài ra, hãy xem các phiên bản cơ sở dữ liệu [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

<div class="overflow-auto">

| Version | PHP (*)   | Release                  | Bug Fixes Until          | Security Fixes Until       |
| ------- |-----------| -----------------------  | ------------------------ | -------------------------- |
| 10      | 8.1 - 8.3 | ngày 14 tháng 2 năm 2023 | ngày 6 tháng 8 năm 2024  | ngày 4 tháng 2 năm 2025    |
| 11      | 8.2 - 8.4 | ngày 12 tháng 3 năm 2024 | ngày 3 tháng 9 năm 2025  | ngày 12 tháng 3 năm 2026   |
| 12      | 8.2 - 8.5 | ngày 24 tháng 2 năm 2025 | ngày 13 tháng 8 năm 2026 | ngày 24 tháng 2 năm 2027   |
| 13      | 8.3 - 8.5 | Q1 2026                  | Q3 2027                  | Q1 2028                    |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>End of life</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>Security fixes only</div>
    </div>
</div>

(*) Supported PHP versions

<a name="laravel-11"></a>
## Laravel 12

Laravel 12 tiếp tục những cải tiến đã có trong Laravel 11.x bằng cách cập nhật các thư viện và giới thiệu các starter kit mới cho React, Svelte, Vue và Livewire, bao gồm các tùy chọn sử dụng [WorkOS AuthKit](https://authkit.com) để xác thực người dùng. Phiên bản WorkOS của các starter kit của chúng tôi sẽ cung cấp các tính năng xác thực qua mạng xã hội, passkey và hỗ trợ SSO.

<a name="minimal-breaking-changes"></a>
### Minimal Breaking Changes

Trọng tâm của chúng tôi trong lần phát hành này là giảm thiểu các breaking change. Thay vào đó, chúng tôi sẽ cố gắng mang lại những cải tiến liên tục về chất lượng trong suốt cả năm mà không làm hỏng các ứng dụng hiện có.

Do đó, bản phát hành Laravel 12 là một "bản phát hành bảo trì" tương đối nhỏ để nâng cấp các dependency hiện có. Xét theo khía cạnh này, hầu hết các ứng dụng Laravel có thể nâng cấp lên Laravel 12 mà không cần bất kỳ thay đổi code nào của ứng dụng.

<a name="new-application-starter-kits"></a>
### New Application Starter Kits

Laravel 12 giới thiệu các [application starter kits](/docs/{{version}}/starter-kits) mới cho React, Svelte, Vue và Livewire. Starter kit React, Svelte và Vue sẽ sử dụng Inertia 2, TypeScript, [shadcn/ui](https://ui.shadcn.com) và Tailwind, trong khi starter kit Livewire sẽ sử dụng thư viện component [Flux UI](https://fluxui.dev) dựa trên Tailwind và Laravel Volt.

Các starter kit React, Svelte, Vue và Livewire đều sử dụng hệ thống xác thực có sẵn của Laravel để cung cấp các tính năng đăng nhập, đăng ký, reset mật khẩu, xác minh email và hơn thế nữa. Ngoài ra, chúng tôi cũng giới thiệu một biến thể [được hỗ trợ bởi WorkOS AuthKit](https://authkit.com) cho mỗi starter kit, cung cấp các tính năng xác thực qua mạng xã hội, passkey và hỗ trợ SSO. WorkOS cung cấp tính năng xác thực miễn phí cho các ứng dụng có tối đa 1 triệu người dùng hoạt động hàng tháng.

Với việc giới thiệu các application starter kit mới này, Laravel Breeze và Laravel Jetstream sẽ không còn nhận được các bản cập nhật nữa.

Để bắt đầu với các starter kit mới của chúng tôi, hãy xem [tài liệu về starter kit](/docs/{{version}}/starter-kits).
