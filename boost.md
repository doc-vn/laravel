# Laravel Boost

- [Introduction](#introduction)
- [Installation](#installation)
    - [Set Up Your Agents](#set-up-your-agents)
    - [Keeping Boost Resources Updated](#keeping-boost-resources-updated)
- [MCP Server](#mcp-server)
    - [Available MCP Tools](#available-mcp-tools)
    - [Manually Registering the MCP Server](#manually-registering-the-mcp-server)
- [AI Guidelines](#ai-guidelines)
    - [Available AI Guidelines](#available-ai-guidelines)
    - [Adding Custom AI Guidelines](#adding-custom-ai-guidelines)
    - [Overriding Boost AI Guidelines](#overriding-boost-ai-guidelines)
    - [Third-Party Package AI Guidelines](#third-party-package-ai-guidelines)
- [Agent Skills](#agent-skills)
    - [Available Skills](#available-skills)
    - [Custom Skills](#custom-skills)
    - [Overriding Skills](#overriding-skills)
    - [Third-Party Package Skills](#third-party-package-skills)
- [Guidelines vs. Skills](#guidelines-vs-skills)
- [Documentation API](#documentation-api)
- [Extending Boost](#extending-boost)
    - [Adding Support for Other IDEs / AI Agents](#adding-support-for-other-ides-ai-agents)

<a name="introduction"></a>
## Introduction

Laravel Boost tăng tốc quá trình phát triển bằng cách cung cấp các hướng dẫn và agent skills cần thiết, giúp các agent AI có thể viết các ứng dụng Laravel chất lượng cao, tuân theo các best practice của Laravel.

Boost cũng cung cấp một API tài liệu hệ sinh thái Laravel mạnh mẽ, kết hợp một công cụ MCP được tích hợp với một cơ sở kiến thức rộng lớn chứa hơn 17.000 mảnh thông tin đặc thù về Laravel, tất cả được tăng cường bởi tìm kiếm ngữ nghĩa sử dụng embeddings để cho ra kết quả chính xác, nhận biết ngữ cảnh. Boost hướng dẫn các agent AI như Claude Code và Cursor sử dụng API này để tìm hiểu các tính năng và best practice mới nhất của Laravel.

<a name="installation"></a>
## Installation

Laravel Boost có thể được cài đặt qua Composer:

```shell
composer require laravel/boost --dev
```

Tiếp theo, cài đặt MCP server và coding guidelines:

```shell
php artisan boost:install
```

Lệnh `boost:install` sẽ tạo ra các file guideline và skill phù hợp cho các coding agent bạn đã chọn trong quá trình cài đặt.

Khi Laravel Boost đã được cài, bạn đã sẵn sàng bắt đầu code với Cursor, Claude Code, hoặc agent AI của bạn.

> [!NOTE]
> Bạn có thể thêm file cấu hình MCP được tạo ra (`.mcp.json`), các file guideline (`CLAUDE.md`, `AGENTS.md`, `junie/`, vv...), và các file cấu hình `boost.json` vào `.gitignore` của ứng dụng, vì các file này sẽ được tạo lại khi chạy `boost:install` và `boost:update`.

<a name="set-up-your-agents"></a>
### Set Up Your Agents

```text tab=Cursor
1. Open the command palette (`Cmd+Shift+P` or `Ctrl+Shift+P`)
2. Press `enter` on "/open MCP Settings"
3. Turn the toggle on for `laravel-boost`
```

```text tab=Claude Code
Claude Code support is typically enabled automatically. If you find it isn't, open a shell in the project's directory and run the following command:

claude mcp add -s local -t stdio laravel-boost php artisan boost:mcp
```

```text tab=Codex
Codex support is typically enabled automatically. If you find it isn't, open a shell in the project's directory and run the following command:

codex mcp add laravel-boost -- php "artisan" "boost:mcp"
```

```text tab=Gemini CLI
Gemini CLI support is typically enabled automatically. If you find it isn't, open a shell in the project's directory and run the following command:

gemini mcp add -s project -t stdio laravel-boost php artisan boost:mcp
```

```text tab=GitHub Copilot (VS Code)
1. Open the command palette (`Cmd+Shift+P` or `Ctrl+Shift+P`)
2. Press `enter` on "MCP: List Servers"
3. Arrow to `laravel-boost` and press `enter`
4. Choose "Start server"
```

```text tab=Junie
1. Press `shift` twice to open the command palette
2. Search "MCP Settings" and press `enter`
3. Check the box next to `laravel-boost`
4. Click "Apply" at the bottom right
```

<a name="keeping-boost-resources-updated"></a>
### Keeping Boost Resources Updated

Bạn có thể muốn cập nhật định kỳ các tài nguyên Boost của bạn (AI guidelines và skills) để đảm bảo chúng có các phiên bản mới nhất của package hệ sinh thái Laravel mà bạn đã cài. Bạn có thể dùng lệnh Artisan `boost:update` để thực hiện điều này.

```shell
php artisan boost:update
```

Bạn cũng có thể tự động quá trình này bằng cách thêm vào Composer script "post-update-cmd" của bạn:

```json
{
  "scripts": {
    "post-update-cmd": [
      "@php artisan boost:update --ansi"
    ]
  }
}
```

Mặc định, lệnh `boost:update` sẽ chỉ cập nhật các resource Boost hiện có đã được export trong ứng dụng của bạn. Nếu bạn muốn Boost quét ứng dụng của bạn để tìm ra bất kỳ package mới được cài đặt và cung cấp tùy chọn export các guideline và skill tương ứng của chúng, bạn có thể sử dụng tùy chọn `--discover`:

```shell
php artisan boost:update --discover
```

<a name="mcp-server"></a>
## MCP Server

Laravel Boost cung cấp một server MCP (Model Context Protocol) cho phép các agent AI tương tác với ứng dụng Laravel của bạn qua các tool. Những tool này cho phép agent kiểm tra cấu trúc ứng dụng, truy vấn cơ sở dữ liệu, chạy code và nhiều hơn nữa.

<a name="available-mcp-tools"></a>
### Available MCP Tools

| Name                 | Notes                                                                                                                                  |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Application Info     | Xem phiên bản PHP và Laravel, database engine, danh sách các package hệ sinh thái Laravel cùng phiên bản của chúng, các Eloquent model |
| Browser Logs         | Đọc log và lỗi từ trình duyệt                                                                                                          |
| Database Connections | Kiểm tra các kết nối cơ sở dữ liệu hiện có, bao gồm cả kết nối mặc định                                                                |
| Database Query       | Thực hiện một query vào cơ sở dữ liệu                                                                                                  |
| Database Schema      | Đọc schema của cơ sở dữ liệu                                                                                                           |
| Get Absolute URL     | Chuyển các đường dẫn tương đối thành tuyệt đối để agent tạo ra các URL hợp lệ                                                          |
| Last Error           | Đọc lỗi cuối cùng từ các file log của ứng dụng                                                                                         |
| Read Log Entries     | Đọc N dòng log cuối cùng                                                                                                               |
| Search Docs          | Truy vấn tài liệu API của Laravel để lấy ra tài liệu dựa trên các package đã cài đặt                                                   |

<a name="manually-registering-the-mcp-server"></a>
### Manually Registering the MCP Server

Thỉnh thoảng, bạn có thể cần tự đăng ký Laravel Boost MCP server cho editor của bạn. Bạn có thể đăng ký MCP server với thông tin sau:

<table>
<tr><td><strong>Command</strong></td><td><code>php</code></td></tr>
<tr><td><strong>Args</strong></td><td><code>artisan boost:mcp</code></td></tr>
</table>

JSON example:

```json
{
    "mcpServers": {
        "laravel-boost": {
            "command": "php",
            "args": ["artisan", "boost:mcp"]
        }
    }
}
```

<a name="ai-guidelines"></a>
## AI Guidelines

AI guidelines là các file hướng dẫn có thể kết hợp linh hoạt, được load sẵn khi AI agent khởi động, cung cấp ngữ cảnh cần thiết về các package của hệ sinh thái Laravel cho agent. Những guidelines này chứa các quy ước cốt lõi, best practice và các pattern đặc trưng của framework, giúp agent tạo ra code nhất quán và chất lượng cao.

<a name="available-ai-guidelines"></a>
### Available AI Guidelines

Laravel Boost bao gồm AI guidelines cho các package và framework sau. Guidelines `core` cung cấp lời khuyên tổng quát cho AI về package đó, áp dụng được cho mọi phiên bản.

| Package           | Versions Supported     |
| ----------------- | ---------------------- |
| Core & Boost      | core                   |
| Laravel Framework | core, 10.x, 11.x, 12.x, 13.x |
| Livewire          | core, 2.x, 3.x, 4.x    |
| Flux UI           | core, free, pro        |
| Folio             | core                   |
| Herd              | core                   |
| Inertia Laravel   | core, 1.x, 2.x, 3.x    |
| Inertia React     | core, 1.x, 2.x, 3.x    |
| Inertia Vue       | core, 1.x, 2.x, 3.x    |
| Inertia Svelte    | core, 1.x, 2.x, 3.x    |
| MCP               | core                   |
| Pennant           | core                   |
| Pest              | core, 3.x, 4.x         |
| PHPUnit           | core                   |
| Pint              | core                   |
| Sail              | core                   |
| Tailwind CSS      | core, 3.x, 4.x         |
| Livewire Volt     | core                   |
| Wayfinder         | core                   |
| Enforce Tests     | conditional            |

> **Note:** Để giữ các AI guidelines luôn được cập nhật, bạn hãy xem phần [cập nhật resources Boost](#keeping-boost-resources-updated).

<a name="adding-custom-ai-guidelines"></a>
### Adding Custom AI Guidelines

Để bổ sung các AI guidelines của riêng bạn vào Laravel Boost, hãy thêm các file `.blade.php` hoặc `.md` vào thư mục `.ai/guidelines/*` của ứng dụng. Các file này sẽ tự động được đưa vào cùng với các guidelines của Boost khi bạn chạy `boost:install`.

<a name="overriding-boost-ai-guidelines"></a>
### Overriding Boost AI Guidelines

Bạn có thể ghi đè các AI guidelines có sẵn của Boost bằng cách tạo các guidelines mới trùng với đường dẫn của file cũ. Khi bạn tạo một guideline mới trùng với đường dẫn của guideline Boost cũ, Boost sẽ dùng phiên bản mới thay vì phiên bản cũ.

Ví dụ, để ghi đè guidelines "Inertia React v2 Form Guidance" của Boost, hãy tạo file tại `.ai/guidelines/inertia-react/2/forms.blade.php`. Khi bạn chạy `boost:install`, Boost sẽ sử dụng guideline mới thay vì cái mặc định.

<a name="third-party-package-ai-guidelines"></a>
### Third-Party Package AI Guidelines

Nếu bạn là chủ của một package bên thứ ba và muốn Boost đưa guidelines AI của package của bạn vào, thì hãy thêm file `resources/boost/guidelines/core.blade.php` vào package của bạn. Khi người dùng package của bạn chạy `php artisan boost:install`, Boost sẽ tự động load các guidelines của bạn.

AI guidelines nên cung cấp một cách nhìn tổng quan ngắn về chức năng của package, nêu rõ cấu trúc file hoặc quy ước cần thiết, và giải thích cách tạo hoặc sử dụng các chức năng chính (kèm theo lệnh hoặc một vài đoạn code mẫu). Hãy giữ chúng ngắn gọn, có thể thực hành và tập trung vào thực hành là tốt nhất để AI có thể tạo code đúng cho người dùng của bạn. Dưới đây là một ví dụ mẫu:

```php
## Package Name

This package provides [brief description of functionality].

### Features

- Feature 1: [clear & short description].
- Feature 2: [clear & short description]. Example usage:

@verbatim
<code-snippet name="How to use Feature 2" lang="php">
$result = PackageName::featureTwo($param1, $param2);
</code-snippet>
@endverbatim
```

<a name="agent-skills"></a>
## Agent Skills

[Agent Skills](https://agentskills.io/home) là các module kiến thức nhẹ, có mục tiêu cụ thể và các agent có thể kích hoạt theo yêu cầu khi làm việc trên các lĩnh vực cụ thể. Khác với guidelines được load sẵn từ đầu, skills cho phép load các pattern và best practice chỉ khi cần thiết, giảm tải cho ngữ cảnh và cải thiện độ liên quan của code do AI tạo ra.

Khi bạn chạy `boost:install` và chọn skills là một chức năng, skills sẽ tự động được cài đặt dựa trên các package được phát hiện trong `composer.json`. Ví dụ, nếu dự án của bạn có package `livewire/livewire`, skill `livewire-development` sẽ được cài đặt tự động.

<a name="available-skills"></a>
### Available Skills

| Skill                      | Package        |
| -------------------------- | -------------- |
| fluxui-development         | Flux UI        |
| folio-routing              | Folio          |
| inertia-react-development  | Inertia React  |
| inertia-svelte-development | Inertia Svelte |
| inertia-vue-development    | Inertia Vue    |
| livewire-development       | Livewire       |
| mcp-development            | MCP            |
| pennant-development        | Pennant        |
| pest-testing               | Pest           |
| tailwindcss-development    | Tailwind CSS   |
| volt-development           | Volt           |
| wayfinder-development      | Wayfinder      |

> **Note:** Để giữ các skills luôn được cập nhật, xem phần [Cập nhật resource Boost](#keeping-boost-resources-updated).

<a name="custom-skills"></a>
### Custom Skills

Để tạo các skills tùy biến của riêng bạn, bạn hãy thêm file `SKILL.md` vào thư mục `.ai/skills/{skill-name}/` của ứng dụng. Khi bạn chạy `boost:update`, các skills tùy biến đó sẽ được cài đặt cùng với các skills có sẵn của Boost.

Ví dụ, để tạo một skill tùy biến cho logic nghiệp vụ của ứng dụng:

```
.ai/skills/creating-invoices/SKILL.md
```

<a name="overriding-skills"></a>
### Overriding Skills

Bạn có thể ghi đè các skills có sẵn của Boost bằng cách tạo ra các skill khác có cùng tên. Khi bạn tạo ra một skill khác có cùng tên với skill Boost, thì Boost sẽ dùng phiên bản tùy biến thay vì phiên bản có sẵn.

Ví dụ, để ghi đè skill `livewire-development` của Boost, hãy tạo file tại `.ai/skills/livewire-development/SKILL.md`. Khi bạn chạy `boost:update`, Boost sẽ sử dụng skill tùy biến thay vì dùng cái mặc định.

<a name="third-party-package-skills"></a>
### Third-Party Package Skills

Nếu bạn là chủ của một package bên thứ ba và muốn Boost đưa skills của package đó vào, hãy thêm file `resources/boost/skills/{skill-name}/SKILL.md` vào package của bạn. Khi người dùng chạy `php artisan boost:install`, Boost sẽ tự động cài skills của bạn.

Boost Skills hỗ trợ [định dạng Agent Skills](https://agentskills.io/what-are-skills) và nên được cấu trúc dưới dạng một thư mục chứa file `SKILL.md` với YAML frontmatter và hướng dẫn Markdown. File `SKILL.md` phải có frontmatter bắt buộc là (`name` và `description`) và có thể có thêm các tùy chọn là scripts, templates và tài liệu tham khảo.

Skills nên có một cấu trúc file hoặc quy ước rõ ràng, và giải thích cách tạo hoặc sử dụng các chức năng chính (kèm lệnh hoặc đoạn code mẫu). Hãy giữ chúng ngắn gọn, có thể thực hành và tập trung vào thực hành để AI tạo ra được code đúng cho người dùng của bạn:

```markdown
---
name: package-name-development
description: Build and work with PackageName features, including components and workflows.
---

# Package Name Development

## When to use this skill
Use this skill when working with PackageName features...

## Features

- Feature 1: [clear & short description].
- Feature 2: [clear & short description]. Example usage:

$result = PackageName::featureTwo($param1, $param2);
```

<a name="guidelines-vs-skills"></a>
## Guidelines vs. Skills

Laravel Boost cung cấp hai cách để cung cấp cho agent AI về ứng dụng của bạn: **guidelines** và **skills**.

**Guidelines** được load sẵn khi AI agent khởi động, cung cấp ngữ cảnh cần thiết về các quy ước và best practice của Laravel, áp dụng rộng khắp code base của bạn.

**Skills** được kích hoạt theo yêu cầu khi làm việc trên các tác vụ cụ thể, chứa các pattern chi tiết cho các lĩnh vực cụ thể (như Livewire components hoặc Pest tests). Việc chỉ load skills khi cần, sẽ giúp giảm load ngữ cảnh và cải thiện chất lượng code.

| Aspect      | Guidelines                                  | Skills                            |
| ----------- | ------------------------------------------- | --------------------------------- |
| **Loaded**  | Load trước, luôn hiện hữu                   | Theo yêu cầu, khi cần thiết       |
| **Scope**   | Rộng, nền tảng                              | Tập trung, theo task cụ thể       |
| **Purpose** | Các quy ước cốt lõi và best practice        | Các pattern triển khai chi tiết   |

<a name="documentation-api"></a>
## Documentation API

Laravel Boost chứa một API tài liệu cung cấp cho các agent AI quyền truy cập vào một cơ sở kiến thức phong phú chứa hơn 17.000 mảnh thông tin đặc thú về Laravel. API sử dụng tìm kiếm ngữ nghĩa với embeddings để mang lại kết quả chính xác, nhận biết ngữ cảnh.

Công cụ MCP `Search Docs` cho phép agent truy vấn tài liệu API do Laravel lưu trữ để lấy tài liệu dựa trên các package bạn đã cài. Các AI guidelines và skills của Boost sẽ tự động hướng dẫn coding agent của bạn sử dụng API này.

| Package           | Versions Supported |
| ----------------- | ------------------ |
| Laravel Framework | 10.x, 11.x, 12.x, 13.x |
| Filament          | 2.x, 3.x, 4.x, 5.x |
| Flux UI           | 2.x Free, 2.x Pro  |
| Inertia           | 1.x, 2.x           |
| Livewire          | 1.x, 2.x, 3.x, 4.x |
| Nova              | 4.x, 5.x           |
| Pest              | 3.x, 4.x           |
| Tailwind CSS      | 3.x, 4.x           |

<a name="extending-boost"></a>
## Extending Boost

Boost hoạt động được với nhiều IDE và agent AI phổ biến ngay khi cài xong. Nếu công cụ của bạn chưa được hỗ trợ, bạn có thể tự tạo agent và tích hợp với Boost.

<a name="adding-support-for-other-ides-ai-agents"></a>
### Adding Support for Other IDEs / AI Agents

Để thêm hỗ trợ cho một IDE hoặc agent AI mới, hãy tạo một class kế thừa `Laravel\Boost\Install\Agents\Agent` và implement một hoặc nhiều contract sau tùy theo nhu cầu của bạn:

- `Laravel\Boost\Contracts\SupportsGuidelines` - Thêm hỗ trợ cho AI guidelines.
- `Laravel\Boost\Contracts\SupportsMcp` - Thêm hỗ trợ cho MCP.
- `Laravel\Boost\Contracts\SupportsSkills` - Thêm hỗ trợ cho Agent Skills.

<a name="writing-the-agent"></a>
#### Writing the Agent

```php
<?php

declare(strict_types=1);

namespace App;

use Laravel\Boost\Contracts\SupportsGuidelines;
use Laravel\Boost\Contracts\SupportsMcp;
use Laravel\Boost\Contracts\SupportsSkills;
use Laravel\Boost\Install\Agents\Agent;

class CustomAgent extends Agent implements SupportsGuidelines, SupportsMcp, SupportsSkills
{
    // Your implementation...
}
```

Xem ví dụ implement tại [ClaudeCode.php](https://github.com/laravel/boost/blob/main/src/Install/Agents/ClaudeCode.php).

<a name="registering-the-agent"></a>
#### Registering the Agent

Đăng ký agent tùy chỉnh của bạn trong phương thức `boot` của `App\Providers\AppServiceProvider` trong ứng dụng:

```php
use Laravel\Boost\Boost;

public function boot(): void
{
    Boost::registerAgent('customagent', CustomAgent::class);
}
```

Một khi đã đăng ký xong, agent của bạn sẽ có trong danh sách chọn khi chạy `php artisan boost:install`.
