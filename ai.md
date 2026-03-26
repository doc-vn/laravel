# AI Assisted Development

 - [Giới thiệu](#introduction)
     - [Tại sao Laravel lại phù hợp cho AI](#why-laravel-for-ai-development)
 - [Laravel Boost](#laravel-boost)
     - [Cài đặt](#installation)
     - [Các tool có sẵn](#available-tools)
     - [AI Guidelines](#ai-guidelines)
     - [Agent Skills](#agent-skills)
     - [Tìm kiếm tài liệu](#documentation-search)
     - [Tích hợp Agents](#agents-integration)

<a name="introduction"></a>
## Giới thiệu

Laravel có vị trí độc đáo để trở thành framework tốt nhất cho việc phát triển có sự hỗ trợ của AI và các agent thông minh. Sự nổi lên của các agent lập trình AI như [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [OpenCode](https://opencode.ai), [Cursor](https://cursor.com) và [GitHub Copilot](https://github.com/features/copilot) đã thay đổi cách các lập trình viên viết code. Những công cụ này có thể tạo ra toàn bộ tính năng, gỡ lỗi các vấn đề phức tạp và tái cấu trúc code với tốc độ chưa từng có — nhưng hiệu quả của chúng phụ thuộc rất nhiều vào mức độ hiểu biết của chúng về codebase của bạn.

<a name="why-laravel-for-ai-development"></a>
### Tại sao Laravel lại phù hợp cho AI

Các quy ước có định hướng và cấu trúc được xác định rõ ràng của Laravel khiến nó trở thành một framework lý tưởng cho việc phát triển có sự hỗ trợ của AI. Khi bạn yêu cầu agent AI thêm một controller, nó biết chính xác lưu file đó ở đâu. Khi bạn cần một migration mới, các quy ước đặt tên và vị trí file đều có thể đoán trước được. Sự nhất quán này loại bỏ việc đoán mò thường cản trở các công cụ AI trong các framework kém linh hoạt hơn.

Ngoài việc tổ chức file, cú pháp dễ hiểu và tài liệu toàn diện của Laravel cung cấp cho các agent AI ngữ cảnh cần thiết để tạo ra code chính xác, đúng chuẩn. Các tính năng như Eloquent relationships, form requests, và middleware tuân theo các pattern mà agent có thể hiểu và sao chép một cách đáng tin cậy. Kết quả là code do AI tạo ra trông giống như được viết bởi một lập trình viên Laravel giàu kinh nghiệm, chứ không phải vá ví từ những đoạn PHP thông thường.

<a name="laravel-boost"></a>
## Laravel Boost

[Laravel Boost](https://github.com/laravel/boost) lấp đầy khoảng cách giữa các agent lập trình AI và ứng dụng Laravel của bạn. Boost là một server MCP (Model Context Protocol) được trang bị hơn 15 công cụ chuyên dụng, cung cấp cho các agent AI cái nhìn sâu sắc về cấu trúc ứng dụng, cơ sở dữ liệu, route và nhiều thứ khác. Khi bạn cài Boost, agent AI của bạn sẽ biến từ một trợ lý lập trình đa năng thành một chuyên gia Laravel hiểu rõ ứng dụng của bạn.

Boost cung cấp ba khả năng chính: một bộ công cụ MCP để kiểm tra và tương tác với ứng dụng, các hướng dẫn AI có thể kết hợp được tạo riêng cho hệ sinh thái Laravel, và một API tài liệu mạnh mẽ chứa hơn 17.000 mảnh kiến thức dành riêng cho Laravel.

<a name="installation"></a>
### Cài đặt

Boost có thể được cài đặt trong các ứng dụng Laravel 10, 11, 12 và 13 chạy PHP 8.1 trở lên. Để bắt đầu, hãy cài Boost như một development library:

```shell
composer require laravel/boost --dev
```

Sau khi cài xong, hãy chạy installer:

```shell
php artisan boost:install
```

Bộ cài đặt sẽ tự động phát hiện IDE và các agent AI của bạn, cho phép bạn chọn các tích hợp phù hợp với project. Boost sẽ tạo ra các file cấu hình cần thiết, chẳng hạn như `.mcp.json` cho các editor tương thích MCP và các file hướng dẫn ngữ cảnh cho AI.

> [!NOTE]
> Các file cấu hình được tạo ra như `.mcp.json`, `CLAUDE.md` và `boost.json` có thể được thêm vào `.gitignore` một cách an toàn nếu bạn muốn mỗi lập trình viên tự cấu hình môi trường phát triển của riêng mình.

<a name="available-tools"></a>
### Các tool có sẵn

Boost cung cấp một bộ công cụ toàn diện cho các agent AI thông qua Model Context Protocol. Những công cụ này cho phép các agent hiểu sâu và tương tác với ứng dụng Laravel của bạn:

<div class="content-list" markdown="1">

- **Kiểm tra ứng dụng** - Truy vấn phiên bản PHP và Laravel, liệt kê các package đã cài, và kiểm tra cấu hình và biến môi trường của ứng dụng.
- **Công cụ Database** - Kiểm tra schema cơ sở dữ liệu, chạy các truy vấn read-only, và nắm rõ cấu trúc dữ liệu mà không cần rời khỏi cuộc hội thoại.
- **Kiểm tra Route** - Liệt kê tất cả các route đã đăng ký cùng với middleware, controller và các tham số của chúng.
- **Lệnh Artisan** - Khám phá các lệnh Artisan có sẵn và các tham số của chúng, giúp agent gợi ý và thực hiện đúng lệnh cho các tác vụ của bạn.
- **Phân tích Log** - Đọc và phân tích các file log của ứng dụng để hỗ trợ gỡ lỗi.
- **Browser Logs** - Truy cập console log và lỗi của trình duyệt khi phát triển với các công cụ frontend của Laravel.
- **Tích hợp Tinker** - Chạy các code PHP trong ngữ cảnh ứng dụng qua Laravel Tinker, cho phép agent kiểm tra và xác nhận hành vi.
- **Tìm kiếm tài liệu** - Tìm kiếm tài liệu hệ sinh thái Laravel với kết quả được tùy chỉnh theo các phiên bản package bạn đã cài.

</div>

<a name="ai-guidelines"></a>
### AI Guidelines

Boost bao gồm một bộ hướng dẫn AI toàn diện được tạo riêng cho hệ sinh thái Laravel. Các hướng dẫn này sẽ dạy cho agent AI cách viết code Laravel chuẩn mực, tuân theo các quy ước của framework và tránh các lỗi phổ biến. Các hướng dẫn này có thể kết hợp linh hoạt và nhận biết các phiên bản, nghĩa là agent sẽ nhận được hướng dẫn phù hợp với đúng phiên bản package bạn đang dùng.

Các hướng dẫn này có sẵn cho Laravel và hơn 16 package trong hệ sinh thái Laravel, bao gồm:

<div class="content-list" markdown="1">

- Livewire (2.x, 3.x, and 4.x)
- Inertia.js (React, Svelte, and Vue variants)
- Tailwind CSS (3.x and 4.x)
- Filament (3.x and 4.x)
- PHPUnit
- Pest PHP
- Laravel Pint
- Và nhiều hơn thế nữa

</div>

Khi bạn chạy `boost:install`, Boost sẽ tự động phát hiện các package mà ứng dụng sử dụng và tổng hợp các hướng dẫn liên quan vào các file ngữ cảnh AI của project.

<a name="agent-skills"></a>
### Agent Skills

[Agent Skills](https://agentskills.io/home) là các module kiến thức nhẹ, có mục tiêu cụ thể mà agent có thể gọi đến khi làm việc trên các lĩnh vực cụ thể. Khác với guidelines được load sẵn từ đầu, skills cho phép gọi đến các pattern và best practice chỉ khi cần thiết, giảm load ngữ cảnh và cải thiện độ liên quan của code do AI tạo ra.

Skills có sẵn cho các package Laravel phổ biến như Livewire, Inertia, Tailwind CSS, Pest và nhiều package khác. Khi bạn chạy `boost:install` và chọn skills là một chức năng, skills sẽ tự động được cài đặt dựa trên các package được phát hiện trong `composer.json`.

<a name="documentation-search"></a>
### Tìm kiếm tài liệu

Boost chứa một API tài liệu mạnh mẽ, cung cấp cho các agent AI quyền truy cập vào hơn 17.000 mảnh tài liệu hệ sinh thái Laravel. Khác với các tìm kiếm web thông thường, tài liệu này được đánh index, vector hóa và lọc để khớp với đúng phiên bản package của bạn.

Khi agent cần hiểu cách hoạt động của một tính năng, chúng có thể tìm kiếm trong tài liệu API của Boost và nhận được thông tin chính xác, theo phiên bản cụ thể. Điều này loại bỏ một vấn đề phổ biến là các agent AI gợi ý các phương thức đã lỗi thời hoặc cú pháp từ các phiên bản framework trước đó.

<a name="agent-integration"></a>
### Tích hợp Agents

Boost tích hợp được với các IDE và công cụ AI phổ biến có hỗ trợ Model Context Protocol. Để xem hướng dẫn cài đặt chi tiết cho Cursor, Claude Code, Codex, Gemini CLI, GitHub Copilot và Junie, xem mục [cấu hình Agent của bạn](/docs/{{version}}/boost#set-up-your-agents) trong tài liệu Boost.
