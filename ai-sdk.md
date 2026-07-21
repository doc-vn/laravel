# Laravel AI SDK

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cấu hình](#configuration)
    - [Tùy biến Base URLs](#custom-base-urls)
    - [Các provider tương thích với OpenAI](#openai-compatible-providers)
    - [Provider Support](#provider-support)
- [Agents](#agents)
    - [Prompting](#prompting)
    - [Context hội thoại](#conversation-context)
    - [Cấu trúc Output](#structured-output)
    - [Attachments](#attachments)
    - [Streaming](#streaming)
    - [Broadcasting](#broadcasting)
    - [Queueing](#queueing)
    - [Tools](#tools)
    - [File Storage Tools](#file-storage-tools)
    - [MCP Tools](#mcp-tools)
    - [Provider Tools](#provider-tools)
    - [Sub-Agents](#sub-agents)
    - [Middleware](#middleware)
    - [Anonymous Agents](#anonymous-agents)
    - [Cấu hình Agent](#agent-configuration)
    - [Provider Options](#provider-options)
- [Images](#images)
- [Audio (TTS)](#audio)
- [Transcription (STT)](#transcription)
- [Embeddings](#embeddings)
    - [Querying Embeddings](#querying-embeddings)
    - [Caching Embeddings](#caching-embeddings)
- [Reranking](#reranking)
- [Files](#files)
- [Vector Stores](#vector-stores)
    - [Thêm File vào Stores](#adding-files-to-stores)
- [Failover](#failover)
- [Testing](#testing)
    - [Agents](#testing-agents)
    - [Images](#testing-images)
    - [Audio](#testing-audio)
    - [Transcriptions](#testing-transcriptions)
    - [Embeddings](#testing-embeddings)
    - [Reranking](#testing-reranking)
    - [Files](#testing-files)
    - [Vector Stores](#testing-vector-stores)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

[Laravel AI SDK](https://github.com/laravel/ai) cung cấp một API thống nhất, rõ ràng để tương tác với các AI provider như OpenAI, Anthropic, Gemini và nhiều provider khác. Với AI SDK, bạn có thể xây dựng các agent thông minh với các công cụ và kết quả có cấu trúc, tạo hình ảnh, tổng hợp và dịch âm thanh, tạo vector embeddings và nhiều tính năng khác — tất cả thông qua một giao diện nhất quán, thân thiện với Laravel.

<a name="installation"></a>
## Cài đặt

Bạn có thể cài đặt Laravel AI SDK qua Composer:

```shell
composer require laravel/ai
```

Tiếp theo, bạn cần export ra các file cấu hình và migration của AI SDK bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
```

Cuối cùng, hãy chạy các database migration của ứng dụng. Lệnh này sẽ tạo các bảng `agent_conversations` và `agent_conversation_messages` mà AI SDK sẽ sử dụng để lưu trữ đoạn hội thoại:

```shell
php artisan migrate
```

<a name="configuration"></a>
### Cấu hình

Bạn có thể định nghĩa thông tin xác thực của các AI provider trong file `config/ai.php` hoặc dưới dạng biến môi trường trong file `.env`:

```ini
ANTHROPIC_API_KEY=
AZURE_OPENAI_API_KEY=
COHERE_API_KEY=
DEEPSEEK_API_KEY=
ELEVENLABS_API_KEY=
GEMINI_API_KEY=
GROQ_API_KEY=
MISTRAL_API_KEY=
OLLAMA_API_KEY=
OPENAI_API_KEY=
OPENAI_COMPATIBLE_API_KEY=
OPENAI_COMPATIBLE_URL=
OPENROUTER_API_KEY=
JINA_API_KEY=
VOYAGEAI_API_KEY=
XAI_API_KEY=
```

Các model mặc định được sử dụng cho text, hình ảnh, âm thanh, dịch và embeddings cũng có thể được cấu hình trong file `config/ai.php` của ứng dụng.

<a name="custom-base-urls"></a>
### Tùy biến Base URLs

Mặc định, Laravel AI SDK sẽ kết nối trực tiếp tới URL API public của từng provider. Tuy nhiên, bạn có thể cần route các request qua một URL khác — chẳng hạn khi sử dụng dịch vụ proxy để quản lý API key tập trung, áp dụng giới hạn chạy, hoặc route lưu lượng qua một gateway của tổ chức.

Bạn có thể cấu hình tùy chỉnh URL bằng cách thêm tham số `url` vào cấu hình provider của bạn:

```php
'providers' => [
    'openai' => [
        'driver' => 'openai',
        'key' => env('OPENAI_API_KEY'),
        'url' => env('OPENAI_URL'),
    ],

    'anthropic' => [
        'driver' => 'anthropic',
        'key' => env('ANTHROPIC_API_KEY'),
        'url' => env('ANTHROPIC_BASE_URL'),
    ],
],
```

Tính năng này hữu ích khi chuyển hướng các request qua dịch vụ proxy (như LiteLLM hoặc Azure OpenAI Gateway) hoặc sử dụng các URL thay thế khác.

Việc tùy chỉnh URL được hỗ trợ cho các provider sau: OpenAI, Anthropic, Gemini, Groq, Cohere, DeepSeek, xAI, và OpenRouter.

<a name="openai-compatible-providers"></a>
### Các provider tương thích với OpenAI

Nếu bạn đang sử dụng một API tương thích với OpenAI, chẳng hạn như LM Studio, vLLM, Together, Fireworks hoặc một local gateway, bạn có thể cấu hình một provider `openai-compatible`. Tùy chọn `url` là bắt buộc, trong khi tùy chọn `key` thì không bắt buộc và sẽ được gửi dưới dạng token bearer nếu có:

```php
'providers' => [
    'local' => [
        'driver' => 'openai-compatible',
        'url' => env('LOCAL_AI_URL'),
        'key' => env('LOCAL_AI_API_KEY'),
    ],
],
```

Sau khi cấu hình xong, bạn có thể sử dụng thông qua tên provider giống như bất kỳ provider nào khác:

```php
agent()->prompt('What is Laravel?', provider: 'local', model: 'local-model');
```

Bạn cũng có thể cấu hình một model text mặc định cho provider đó để bạn không cần phải truyền một model một cách rõ ràng:

```php
'local' => [
    'driver' => 'openai-compatible',
    'url' => env('LOCAL_AI_URL'),
    'key' => env('LOCAL_AI_API_KEY'),
    'models' => [
        'text' => [
            'default' => env('LOCAL_AI_MODEL'),
        ],
    ],
],
```

Các provider tương thích với OpenAI sẽ hỗ trợ tạo văn bản, streaming, các tool, structured output và đính kèm hình ảnh. Nếu endpoint của bạn yêu cầu thêm các trường trong body của request, hãy cung cấp chúng bằng cách sử dụng [tùy chọn provider](#provider-options).

<a name="provider-support"></a>
### Provider Support

AI SDK hỗ trợ nhiều provider khác nhau cho các tính năng của nó. Bảng dưới đây sẽ tóm tắt các tính năng có sẵn của từng provider:

<div class="overflow-auto">

| Feature | Providers |
|---|---|
| Text | OpenAI, OpenAI Compatible, Anthropic, Gemini, Azure, Bedrock, Groq, xAI, DeepSeek, Mistral, Ollama, OpenRouter |
| Images | OpenAI, Gemini, xAI, Azure, Bedrock, OpenRouter |
| TTS | OpenAI, ElevenLabs, Gemini |
| STT | OpenAI, ElevenLabs, Mistral, Gemini |
| Embeddings | OpenAI, Gemini, Azure, Bedrock, Cohere, Mistral, Jina, VoyageAI, Ollama, OpenRouter |
| Reranking | Cohere, Jina, VoyageAI |
| Files | OpenAI, Anthropic, Gemini, Azure |

</div>

Enum `Laravel\Ai\Enums\Lab` có thể được sử dụng để reference đến các provider trong code thay vì dùng chuỗi:

```php
use Laravel\Ai\Enums\Lab;

Lab::Anthropic;
Lab::OpenAI;
Lab::OpenAiCompatible;
Lab::Gemini;
// ...
```

<a name="agents"></a>
## Agents

Agents là block cơ bản để tương tác với các provider AI trong Laravel AI SDK. Mỗi agent là một class PHP chuyên biệt, bao gồm các hướng dẫn, context hội thoại, công cụ và cấu trúc output cần thiết để tương tác với một mô hình ngôn ngữ lớn. Hãy coi một agent như một trợ lý chuyên biệt — một huấn luyện viên bán hàng, một công cụ phân tích tài liệu, một bot hỗ trợ — mà bạn cấu hình một lần và gửi prompt khi cần trong suốt ứng dụng của bạn.

Bạn có thể tạo một agent bằng lệnh Artisan `make:agent`:

```shell
php artisan make:agent SalesCoach

php artisan make:agent SalesCoach --structured
```

Trong class agent được tạo ra, bạn có thể định nghĩa system prompt, hướng dẫn, context tin nhắn, các công cụ có sẵn và cấu trúc output (đối với kết quả có cấu trúc):

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\RetrievePreviousTranscripts;
use App\Models\History;
use App\Models\User;
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Messages\Message;
use Laravel\Ai\Promptable;
use Stringable;

class SalesCoach implements Agent, Conversational, HasTools, HasStructuredOutput
{
    use Promptable;

    public function __construct(public User $user) {}

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): Stringable|string
    {
        return 'You are a sales coach, analyzing transcripts and providing feedback and an overall sales strength score.';
    }

    /**
     * Get the list of messages comprising the conversation so far.
     */
    public function messages(): iterable
    {
        return History::where('user_id', $this->user->id)
            ->latest()
            ->limit(50)
            ->get()
            ->reverse()
            ->map(function ($message) {
                return new Message($message->role, $message->content);
            })->all();
    }

    /**
     * Get the tools available to the agent.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new RetrievePreviousTranscripts,
        ];
    }

    /**
     * Get the agent's structured output schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'feedback' => $schema->string()->required(),
            'score' => $schema->integer()->min(1)->max(10)->required(),
        ];
    }
}
```

<a name="prompting"></a>
### Prompting

Để gửi prompt tới một agent, trước tiên hãy tạo một instance bằng phương thức `make` hoặc khởi tạo thông thường, sau đó gọi `prompt`:

```php
$response = (new SalesCoach)
    ->prompt('Analyze this sales transcript...');

return (string) $response;
```

Phương thức `make` sẽ resolve agent của bạn từ container ra, và cho phép dependency injection tự động. Bạn cũng có thể truyền thêm tham số vào constructor của agent:

```php
$agent = SalesCoach::make(user: $user);
```

Bằng cách truyền thêm tham số vào phương thức `prompt`, bạn có thể ghi đè provider, model hoặc thời gian timeout mặc định khi gửi prompt:

```php
$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: Lab::Anthropic,
    model: 'claude-haiku-4-5-20251001',
    timeout: 120,
);
```

<a name="conversation-context"></a>
### Context hội thoại

Nếu agent của bạn implement interface `Conversational`, bạn có thể sử dụng phương thức `messages` để trả về context hội thoại trước đó (nếu có):

```php
use App\Models\History;
use Laravel\Ai\Messages\Message;

/**
 * Get the list of messages comprising the conversation so far.
 */
public function messages(): iterable
{
    return History::where('user_id', $this->user->id)
        ->latest()
        ->limit(50)
        ->get()
        ->reverse()
        ->map(function ($message) {
            return new Message($message->role, $message->content);
        })->all();
}
```

<a name="remembering-conversations"></a>
#### Remembering Conversations

> **Note:** Trước khi sử dụng trait `RemembersConversations`, bạn cần export và chạy các migration của AI SDK bằng lệnh Artisan `vendor:publish`. Các migration này sẽ tạo ra các bảng cơ sở dữ liệu cần thiết để lưu trữ lịch sử hội thoại.

Nếu bạn muốn Laravel tự động lưu và lấy ra lịch sử hội thoại cho các agent của bạn, bạn có thể sử dụng trait `RemembersConversations`. Trait này cung cấp một cách đơn giản để lưu các tin nhắn hội thoại vào cơ sở dữ liệu mà không cần bạn phải tự tay implement interface `Conversational`:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Concerns\RemembersConversations;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, Conversational
{
    use Promptable, RemembersConversations;

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): string
    {
        return 'You are a sales coach...';
    }
}
```

Khi sử dụng trait `RemembersConversations`, bạn không cần định nghĩa thêm phương thức `messages` trong class agent của bạn. Nếu phương thức `messages` tồn tại, nó sẽ được ưu tiên hơn so với implementation của trait và lịch sử hội thoại sẽ không được load ra từ cơ sở dữ liệu.

Để bắt đầu một cuộc hội thoại mới cho một người dùng, hãy gọi phương thức `forUser` trước khi gửi prompt:

```php
$response = (new SalesCoach)->forUser($user)->prompt('Hello!');

$conversationId = $response->conversationId;
```

ID của cuộc hội thoại sẽ được trả về trong response và có thể lưu lại để tham chiếu sau. Nếu bạn muốn lấy tất cả các cuộc hội thoại của một người dùng bằng Eloquent, bạn có thể thêm trait `HasConversations` vào model user của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Ai\Concerns\HasConversations;

class User extends Authenticatable
{
    use HasConversations;
}
```

Sau khi trait đã được thêm vào model của bạn, bạn có thể lấy và truy vấn các cuộc hội thoại của người dùng thông qua quan hệ `conversations`:

```php
$conversations = $user->conversations()
    ->latest('updated_at')
    ->paginate(20);
```

Để tiếp tục một cuộc hội thoại hiện có, hãy sử dụng phương thức `continue`:

```php
$response = (new SalesCoach)
    ->continue($conversationId, as: $user)
    ->prompt('Tell me more about that.');
```

Khi sử dụng trait `RemembersConversations`, các tin nhắn trước đó sẽ được tự động load lại và tự động đưa vào context hội thoại khi gửi prompt. Các tin nhắn mới (của cả người dùng lẫn assistant) sẽ được tự động lưu lại sau mỗi tương tác.

<a name="structured-output"></a>
### Cấu trúc Output

Nếu bạn muốn agent trả về kết quả output có cấu trúc, hãy implement interface `HasStructuredOutput`, nó sẽ yêu cầu agent của bạn định nghĩa một phương thức `schema`:

```php
<?php

namespace App\Ai\Agents;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasStructuredOutput
{
    use Promptable;

    // ...

    /**
     * Get the agent's structured output schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'score' => $schema->integer()->required(),
        ];
    }
}
```

Khi gửi prompt tới một agent trả về kết quả có cấu trúc, bạn có thể truy cập vào `StructuredAgentResponse` được trả về giống như một mảng:

```php
$response = (new SalesCoach)->prompt('Analyze this sales transcript...');

return $response['score'];
```

<a name="structured-output-nested-objects"></a>
#### Nested Objects

Để định nghĩa kết quả output có cấu trúc lồng nhau, hãy sử dụng phương thức `object` với một closure:

```php
<?php

namespace App\Ai\Agents;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasStructuredOutput
{
    use Promptable;

    // ...

    /**
     * Get the agent's structured output schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'score' => $schema->integer()->required(),
            'metadata' => $schema->object(fn ($schema) => [
                'confidence' => $schema->string()->enum(['low', 'medium', 'high'])->required(),
                'language' => $schema->string()->required(),
            ])->required(),
        ];
    }
}
```

<a name="structured-output-arrays-of-objects"></a>
#### Arrays of Objects

Nếu agent của bạn trả về một danh sách các item có cấu trúc, hãy kết hợp các phương thức `array` và `object`:

```php
public function schema(JsonSchema $schema): array
{
    return [
        'feedback' => $schema->array()
            ->items(
                $schema->object(fn ($schema) => [
                    'comment' => $schema->string()->required(),
                    'score' => $schema->integer()->required(),
                ])
            )
            ->required(),
    ];
}
```

Nếu một giá trị có thể giống với một trong nhiều schema, hãy sử dụng phương thức `anyOf`:

```php
public function schema(JsonSchema $schema): array
{
    return [
        'content' => $schema->anyOf([
            $schema->object(fn ($schema) => [
                'type' => $schema->string()->enum(['article'])->required(),
                'title' => $schema->string()->required(),
            ]),
            $schema->object(fn ($schema) => [
                'type' => $schema->string()->enum(['image'])->required(),
                'url' => $schema->string()->required(),
            ]),
        ])->required(),
    ];
}
```

<a name="attachments"></a>
### Attachments

Khi gửi prompt, bạn cũng có thể đính kèm các tệp file ảnh hoặc tài liệu để cho phép mô hình kiểm tra chúng:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached sales transcript...',
    attachments: [
        Files\Document::fromStorage('transcript.pdf') // Attach a document from a filesystem disk...
        Files\Document::fromPath('/home/laravel/transcript.md') // Attach a document from a local path...
        $request->file('transcript'), // Attach an uploaded file...
    ]
);
```

Tương tự, class `Laravel\Ai\Files\Image` có thể được dùng để đính kèm file ảnh vào prompt:

```php
use App\Ai\Agents\ImageAnalyzer;
use Laravel\Ai\Files;

$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Files\Image::fromStorage('photo.jpg') // Attach an image from a filesystem disk...
        Files\Image::fromPath('/home/laravel/photo.jpg') // Attach an image from a local path...
        $request->file('photo'), // Attach an uploaded file...
    ]
);
```

<a name="streaming"></a>
### Streaming

Bạn có thể stream một response của một agent bằng cách gọi phương thức `stream`. `StreamableAgentResponse` được trả về có thể được trả về từ một route để tự động gửi response streaming (SSE) cho client:

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)->stream('Analyze this sales transcript...');
});
```

Phương thức `then` có thể được dùng để cung cấp một closure sẽ được gọi khi toàn bộ response đã được stream đến client xong:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Responses\StreamedAgentResponse;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->then(function (StreamedAgentResponse $response) {
            // $response->text, $response->events, $response->usage...
        });
});
```

Ngoài ra, bạn có thể lặp các event đã được stream:

```php
$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    // ...
}
```

<a name="streaming-using-the-vercel-ai-sdk-protocol"></a>
#### Streaming Using the Vercel AI SDK Protocol

Bạn có thể stream các event thông qua [giao thức stream của Vercel AI SDK](https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol) bằng cách gọi phương thức `usingVercelDataProtocol` trên streamable response:

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->usingVercelDataProtocol();
});
```

<a name="broadcasting"></a>
### Broadcasting

Bạn có thể broadcast các event streaming theo một vài cách khác nhau. Đầu tiên, bạn có thể gọi phương thức `broadcast` hoặc `broadcastNow` trên một streamed event:

```php
use App\Ai\Agents\SalesCoach;
use Illuminate\Broadcasting\Channel;

$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    $event->broadcast(new Channel('channel-name'));
}
```

Hoặc bạn cũng có thể gọi phương thức `broadcastOnQueue` của agent để đưa thao tác agent vào queue và broadcast các event streaming khi chúng sẵn sàng:

```php
(new SalesCoach)->broadcastOnQueue(
    'Analyze this sales transcript...'
    new Channel('channel-name'),
);
```

<a name="skipping-oversized-events"></a>
#### Skipping Oversized Events

Một số nền tảng broadcasting sẽ giới hạn tin nhắn WebSocket ở khoảng 10KB. Các event stream chứa nhiều dữ liệu, như kết quả của tool, có thể vượt quá giới hạn này và khiến việc broadcasting thất bại. Bạn có thể loại các loại event cụ thể ra khỏi việc broadcasting này bằng thuộc tính `WithoutBroadcasting`:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Attributes\WithoutBroadcasting;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Promptable;
use Laravel\Ai\Streaming\Events\ToolCall;
use Laravel\Ai\Streaming\Events\ToolResult;

#[WithoutBroadcasting(ToolCall::class, ToolResult::class)]
class SearchAgent implements Agent, HasTools
{
    use Promptable;

    // ...
}
```

Các event bị loại sẽ không bao giờ được broadcast, nhưng chúng vẫn sẽ được lưu vào bảng `agent_conversation_messages`, để frontend của bạn có thể load toàn bộ dữ liệu tool sau khi stream kết thúc. Điều này hoạt động cho cả broadcasting trong queue (`broadcastOnQueue`) và broadcasting đồng bộ (`broadcast` / `broadcastNow`).

<a name="queueing"></a>
### Queueing

Sử dụng phương thức `queue` của agent, bạn có thể gửi prompt cho agent nhưng để nó xử lý response ở trong background, giữ cho ứng dụng luôn nhanh nhạy và phản hồi tốt. Các phương thức `then` và `catch` có thể được dùng để đăng ký các closure sẽ được gọi khi có response hoặc khi xảy ra ngoại lệ:

```php
use Illuminate\Http\Request;
use Laravel\Ai\Responses\AgentResponse;
use Throwable;

Route::post('/coach', function (Request $request) {
    (new SalesCoach)
        ->queue($request->input('transcript'))
        ->then(function (AgentResponse $response) {
            // ...
        })
        ->catch(function (Throwable $e) {
            // ...
        });

    return back();
});
```

<a name="tools"></a>
### Tools

Các tool có thể được sử dụng để cung cấp cho agents thêm những khả năng mà chúng có thể sử dụng khi phản hồi các prompt. Các tool có thể được tạo ra bằng lệnh Artisan `make:tool`:

```shell
php artisan make:tool RandomNumberGenerator
```

Tool được tạo ra sẽ được lưu trong thư mục `app/Ai/Tools` của ứng dụng. Mỗi tool chứa một phương thức `handle` sẽ được agent gọi khi nó cần sử dụng tool đó:

```php
<?php

namespace App\Ai\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Tool;
use Laravel\Ai\Tools\Request;
use Stringable;

class RandomNumberGenerator implements Tool
{
    /**
     * Get the description of the tool's purpose.
     */
    public function description(): Stringable|string
    {
        return 'This tool may be used to generate cryptographically secure random numbers.';
    }

    /**
     * Execute the tool.
     */
    public function handle(Request $request): Stringable|string
    {
        return (string) random_int($request['min'], $request['max']);
    }

    /**
     * Get the tool's schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'min' => $schema->integer()->min(0)->required(),
            'max' => $schema->integer()->required(),
        ];
    }
}
```

Một khi đã định nghĩa xong tool, bạn có thể trả nó về từ phương thức `tools` của bất kỳ agent nào:

```php
use App\Ai\Tools\RandomNumberGenerator;

/**
 * Get the tools available to the agent.
 *
 * @return Tool[]
 */
public function tools(): iterable
{
    return [
        new RandomNumberGenerator,
    ];
}
```

<a name="similarity-search"></a>
#### Similarity Search

Công cụ `SimilaritySearch` cho phép các agent tìm kiếm các tài liệu tương tự với một truy vấn bằng cách sử dụng vector embeddings được lưu trong cơ sở dữ liệu. Điều này hữu ích cho retrieval-augmented generation (RAG) khi bạn muốn agent có khả năng tìm kiếm dữ liệu của ứng dụng.

Cách đơn giản nhất để tạo tool tìm kiếm là dùng phương thức `usingModel` với một Eloquent model có vector embeddings:

```php
use App\Models\Document;
use Laravel\Ai\Tools\SimilaritySearch;

public function tools(): iterable
{
    return [
        SimilaritySearch::usingModel(Document::class, 'embedding'),
    ];
}
```

Tham số đầu tiên là class Eloquent model, tham số thứ hai là cột chứa vector embeddings.

Bạn cũng có thể đặt ngưỡng tương đồng tối thiểu trong khoảng từ `0.0` đến `1.0` và một closure để tùy chỉnh query:

```php
SimilaritySearch::usingModel(
    model: Document::class,
    column: 'embedding',
    minSimilarity: 0.7,
    limit: 10,
    query: fn ($query) => $query->where('published', true),
),
```

Để có nhiều quyền hạn hơn, bạn có thể tạo tool tìm kiếm với một closure tùy chỉnh trả về kết quả tìm kiếm:

```php
use App\Models\Document;
use Laravel\Ai\Tools\SimilaritySearch;

public function tools(): iterable
{
    return [
        new SimilaritySearch(using: function (string $query) {
            return Document::query()
                ->where('user_id', $this->user->id)
                ->whereVectorSimilarTo('embedding', $query)
                ->limit(10)
                ->get();
        }),
    ];
}
```

Bạn có thể tùy chỉnh mô tả của tool bằng phương thức `withDescription`:

```php
SimilaritySearch::usingModel(Document::class, 'embedding')
    ->withDescription('Search the knowledge base for relevant articles.'),
```

<a name="file-storage-tools"></a>
### File Storage Tools

Factory tool `FileStorage` cho phép bạn cấp quyền cho agent truy cập vào một [filesystem disk](/docs/{{version}}/filesystem) của Laravel. Phương thức `all` sẽ trả về các tool cho phép agent list, đọc, kiểm tra, tạo URL, ghi, xóa và sao chép các file trên disk đã cho:

```php
use Laravel\Ai\Tools\FileStorage;

public function tools(): iterable
{
    return FileStorage::all('local');
}
```

Nếu agent của bạn chỉ có quyền kiểm tra file, hãy sử dụng phương thức `readOnly`:

```php
return FileStorage::readOnly('local');
```

Các phương thức này trả về một `Illuminate\Support\Collection`, cho phép bạn lọc thêm các tool sẽ được cung cấp cho agent:

```php
use Laravel\Ai\Tools\Filesystem\DeleteFile;

return FileStorage::all('s3')
    ->reject(fn ($tool) => $tool instanceof DeleteFile);
```

<a name="mcp-tools"></a>
### MCP Tools

Nếu ứng dụng của bạn sử dụng [Laravel MCP](/docs/{{version}}/mcp), bạn có thể cung cấp cho các agent của bạn các tool được hiển thị bởi server [Model Context Protocol](https://modelcontextprotocol.io). Bằng cách sử dụng [Laravel MCP client](/docs/{{version}}/mcp#client), bạn có thể kết nối với một server MCP remote hoặc local và truyền trực tiếp các tool của nó cho agent của bạn.

> [!NOTE]
> Các tool MCP yêu cầu package [Laravel MCP](/docs/{{version}}/mcp) phải được cài đặt trong ứng dụng của bạn.

Vì phương thức `tools` của MCP client sẽ trả về một collection, hãy chuyển nó về mảng `tools` của agent bằng toán tử `...`:

```php
use App\Ai\Tools\RandomNumberGenerator;
use Laravel\Mcp\Client;

/**
 * Get the tools available to the agent.
 *
 * @return Tool[]
 */
public function tools(): iterable
{
    return [
        ...Client::web('https://mcp.example.com')
            ->withToken($token)
            ->tools(),

        new RandomNumberGenerator,
    ];
}
```

AI SDK sẽ tự động bọc từng tool MCP để các agent có thể gọi nó giống như bất kỳ tool nào khác. Bạn cũng có thể sử dụng [tên của một MCP client đã được đặt tên](/docs/{{version}}/mcp#named-clients):

```php
use Laravel\Mcp\Facades\Mcp;

public function tools(): iterable
{
    return [
        ...Mcp::client('github')->tools(),
    ];
}
```

Hoặc kết nối với một [local MCP server](/docs/{{version}}/mcp#client-connecting):

```php
use Laravel\Mcp\Client;

public function tools(): iterable
{
    return [
        ...Client::local('php', ['artisan', 'mcp:start'])->tools(),
    ];
}
```

Để biết thêm thông tin về việc tạo và xác thực các MCP client, bao gồm cả token bearer và OAuth, hãy tham khảo [tài liệu MCP client](/docs/{{version}}/mcp#client).

<a name="provider-tools"></a>
### Provider Tools

Provider tools là những công cụ đặc biệt được các provider AI triển khai native, cung cấp các khả năng như tìm kiếm web, lấy nội dung từ URL và tìm kiếm file. Khác với các tool thông thường, provider tools được chạy bởi chính provider chứ không phải ứng dụng của bạn.

Provider tools có thể được trả về từ phương thức `tools` của agent.

<a name="web-search"></a>
#### Web Search

Provider tool `WebSearch` cho phép các agent tìm kiếm web để lấy thông tin thời gian thực. Điều này hữu ích khi trả lời các câu hỏi về các sự kiện hiện tại, dữ liệu mới, hoặc các chủ đề có thể đã bị thay đổi từ thời điểm mô hình được traning dữ liệu.

**Supported providers:** Anthropic, OpenAI, Gemini, OpenRouter

```php
use Laravel\Ai\Providers\Tools\WebSearch;

public function tools(): iterable
{
    return [
        new WebSearch,
    ];
}
```

Bạn có thể cấu hình công cụ tìm kiếm web để giới hạn số lần tìm kiếm hoặc giới hạn kết quả vào trong các tên miền cụ thể:

```php
(new WebSearch)->max(5)->allow(['laravel.com', 'php.net']),
```

Để lọc kết quả tìm kiếm theo vị trí của người dùng, hãy sử dụng phương thức `location`:

```php
(new WebSearch)->location(
    city: 'New York',
    region: 'NY',
    country: 'US'
);
```

<a name="web-fetch"></a>
#### Web Fetch

Công cụ provider `WebFetch` cho phép các agent lấy và đọc nội dung của các trang web. Điều này hữu ích khi bạn cần agent phân tích các URL cụ thể hoặc lấy thông tin chi tiết từ các trang web đã biết.

**Supported providers:** Anthropic, Gemini

```php
use Laravel\Ai\Providers\Tools\WebFetch;

public function tools(): iterable
{
    return [
        new WebFetch,
    ];
}
```

Bạn có thể cấu hình công cụ web fetch để giới hạn số lần fetch hoặc giới hạn trong các tên miền cụ thể:

```php
(new WebFetch)->max(3)->allow(['docs.laravel.com']),
```

<a name="file-search"></a>
#### File Search

Công cụ provider `FileSearch` cho phép các agent tìm kiếm qua các [file](#files) được lưu trong [vector stores](#vector-stores). Tính năng này cho phép retrieval-augmented generation (RAG) bằng cách cho phép agent tìm kiếm trong các tài liệu mà bạn đã upload để lấy thông tin.

**Supported providers:** OpenAI, Gemini

```php
use Laravel\Ai\Providers\Tools\FileSearch;

public function tools(): iterable
{
    return [
        new FileSearch(stores: ['store_id']),
    ];
}
```

Bạn có thể cung cấp nhiều vector store ID để tìm kiếm trên nhiều store:

```php
new FileSearch(stores: ['store_1', 'store_2']);
```

Nếu các file của bạn có [metadata](#adding-files-to-stores), bạn có thể lọc kết quả tìm kiếm bằng cách truyền vào tham số `where`. Đối với các bộ lọc so sánh đơn giản hãy truyền vào một mảng:

```php
new FileSearch(stores: ['store_id'], where: [
    'author' => 'Taylor Otwell',
    'year' => 2026,
]);
```

Đối với các bộ lọc phức tạp hơn, bạn có thể truyền vào một closure nhận vào một instance của `FileSearchQuery`:

```php
use Laravel\Ai\Providers\Tools\FileSearchQuery;

new FileSearch(stores: ['store_id'], where: fn (FileSearchQuery $query) =>
    $query->where('author', 'Taylor Otwell')
        ->whereNot('status', 'draft')
        ->whereIn('category', ['news', 'updates'])
);
```

<a name="sub-agents"></a>
### Sub-Agents

Các Agent cũng có thể được trả về từ phương thức `tools` của một agent khác. Khi một agent được trả về như một tool, thì agent cha có thể yêu cầu một tác vụ cho sub-agent đó và sử dụng response của sub-agent trong khi trả lời prompt ban đầu. Điều này sẽ hữu ích khi một agent cha cần truy cập vào một agent chuyên biệt có hướng dẫn, có công cụ, có cấu hình model hoặc nhà cung cấp riêng cho chúng.

Ví dụ: một agent hỗ trợ khách hàng có thể yêu cầu các câu hỏi về điều kiện hoàn tiền cho một agent chuyên phụ trách về hoàn tiền:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Promptable;

class CustomerSupportAgent implements Agent, HasTools
{
    use Promptable;

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): string
    {
        return 'You help customers with account, order, and billing questions. Delegate refund policy questions to the refunds specialist.';
    }

    /**
     * Get the tools available to the agent.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new RefundsAgent,
        ];
    }
}
```

Để tùy chỉnh cách sub-agent được hiển thị cho agent cha, hãy implement interface `CanActAsTool` trên sub-agent và định nghĩa tên cũng như mô tả mà công cụ sẽ hiển thị:

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\LookupOrder;
use Laravel\Ai\Attributes\Provider;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\CanActAsTool;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

#[Provider(Lab::Anthropic)]
class RefundsAgent implements Agent, CanActAsTool, HasTools
{
    use Promptable;

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): string
    {
        return 'You are a refunds specialist. Use order details and the refund policy to give concise eligibility guidance.';
    }

    /**
     * Get the agent's tool name.
     */
    public function name(): string
    {
        return 'refunds_specialist';
    }

    /**
     * Get the agent's tool description.
     */
    public function description(): string
    {
        return 'Determine whether an order is eligible for a refund and explain the next step.';
    }

    /**
     * Get the tools available to the agent.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new LookupOrder,
        ];
    }
}
```

Nếu một sub-agent không implement `CanActAsTool`, Laravel sẽ sử dụng tên class base của agent làm tên công cụ và một mô tả chung là sẽ yêu cầu agent cha phải truyền vào một mô tả tác vụ rõ ràng, độc lập. Mỗi lần gọi sub-agent sẽ chạy trong một môi trường độc lập và không thể lấy lịch sử cuộc hội thoại của agent cha.

<a name="middleware"></a>
### Middleware

Agent hỗ trợ middleware, cho phép bạn can thiệp và chỉnh sửa các prompt trước khi chúng được gửi đến provider. Middleware có thể được tạo ra bằng lệnh Artisan `make:agent-middleware`:

```shell
php artisan make:agent-middleware LogPrompts
```

Middleware được tạo ra sẽ được lưu trong thư mục `app/Ai/Middleware` của ứng dụng. Để thêm middleware vào một agent, hãy implement interface `HasMiddleware` và định nghĩa phương thức `middleware` trả về một mảng chứa các class middleware:

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Middleware\LogPrompts;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasMiddleware;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasMiddleware
{
    use Promptable;

    // ...

    /**
     * Get the agent's middleware.
     */
    public function middleware(): array
    {
        return [
            new LogPrompts,
        ];
    }
}
```

Mỗi class middleware cần định nghĩa một phương thức `handle`, nhận vào một `AgentPrompt` và một `Closure` để chuyển prompt đến middleware tiếp theo:

```php
<?php

namespace App\Ai\Middleware;

use Closure;
use Laravel\Ai\Prompts\AgentPrompt;

class LogPrompts
{
    /**
     * Handle the incoming prompt.
     */
    public function handle(AgentPrompt $prompt, Closure $next)
    {
        Log::info('Prompting agent', ['prompt' => $prompt->prompt]);

        return $next($prompt);
    }
}
```

Bạn có thể sử dụng phương thức `then` trên response để chạy code sau khi agent hoàn tất xử lý. Tính năng này hoạt động cho cả response đồng bộ lẫn streaming:

```php
public function handle(AgentPrompt $prompt, Closure $next)
{
    return $next($prompt)->then(function (AgentResponse $response) {
        Log::info('Agent responded', ['text' => $response->text]);
    });
}
```

<a name="anonymous-agents"></a>
### Anonymous Agents

Thỉnh thoảng bạn muốn tương tác nhanh với một model mà không cần tạo một class agent riêng. Bạn có thể tạo một agent ẩn, tạm thời bằng hàm `agent`:

```php
use function Laravel\Ai\{agent};

$response = agent(
    instructions: 'You are an expert at software development.',
    messages: [],
    tools: [],
)->prompt('Tell me about Laravel')
```

Các agent ẩn cũng có thể tạo ra kết quả output có cấu trúc:

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;

use function Laravel\Ai\{agent};

$response = agent(
    schema: fn (JsonSchema $schema) => [
        'number' => $schema->integer()->required(),
    ],
)->prompt('Generate a random number less than 100')
```

<a name="agent-configuration"></a>
### Cấu hình Agent

Bạn có thể cấu hình các tùy chọn text generation cho một agent bằng cách sử dụng PHP attributes. Có sẵn các attribute sau:

- `MaxSteps`: Số bước tối đa mà agent có thể thực hiện khi sử dụng các tool.
- `MaxTokens`: Số lượng token tối đa mà model có thể sinh ra.
- `Model`: Model mà agent nên sử dụng.
- `Provider`: Provider AI (hoặc các provider cho cơ chế failover) sẽ được agent sử dụng.
- `Temperature`: Tính sáng tạo để sử dụng cho việc sinh văn bản (từ 0.0 đến 1.0).
- `Timeout`: Thời gian chờ tính bằng giây cho các request của agent (mặc định: 60).
- `TopP`: Xác suất giới hạn được sử dụng để tạo văn bản (0.0 đến 1.0).
- `UseCheapestModel`: Sử dụng model văn bản rẻ nhất của provider để tối ưu chi phí.
- `UseSmartestModel`: Sử dụng mô hình văn bản thông minh nhất của provider cho các tác vụ phức tạp.

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Attributes\MaxSteps;
use Laravel\Ai\Attributes\MaxTokens;
use Laravel\Ai\Attributes\Model;
use Laravel\Ai\Attributes\Provider;
use Laravel\Ai\Attributes\Temperature;
use Laravel\Ai\Attributes\Timeout;
use Laravel\Ai\Attributes\TopP;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

#[Provider(Lab::Anthropic)]
#[Model('claude-haiku-4-5-20251001')]
#[MaxSteps(10)]
#[MaxTokens(4096)]
#[Temperature(0.7)]
#[Timeout(120)]
#[TopP(0.9)]
class SalesCoach implements Agent
{
    use Promptable;

    // ...
}
```

Các attribute `UseCheapestModel` và `UseSmartestModel` cho phép bạn tự động chọn mô hình tối ưu về chi phí hoặc thông minh cho một provider cụ thể mà không cần chỉ định tên mô hình. Điều này rất hữu ích khi bạn muốn tối ưu chi phí hoặc khả năng trên các provider khác nhau:

```php
use Laravel\Ai\Attributes\UseCheapestModel;
use Laravel\Ai\Attributes\UseSmartestModel;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Promptable;

#[UseCheapestModel]
class SimpleSummarizer implements Agent
{
    use Promptable;

    // Will use the cheapest model (e.g., Haiku)...
}

#[UseSmartestModel]
class ComplexReasoner implements Agent
{
    use Promptable;

    // Will use the most capable model (e.g., Opus)...
}
```

> [!NOTE]
> Model thực tế được chọn bởi `UseCheapestModel` và `UseSmartestModel` có thể thay đổi giữa các phiên bản của Laravel AI SDK khi các nhà cung cấp phát hành các model mới. Việc đổi các model này có thể dẫn đến những thay đổi về hành vi, các tham số bị xoá hoặc chênh lệch chi phí. Nếu bạn cần một model và mức giá ổn định, dễ dự đoán, hãy chỉ định rõ model mà bạn muốn dùng bằng cách sử dụng thuộc tính `Model`.

<a name="provider-options"></a>
### Provider Options

Nếu agent của bạn cần truyền các tùy chọn đặc thù của provider (chẳng hạn như effort suy luận của OpenAI hoặc thiết lập penalty), hãy implement interface `HasProviderOptions` và định nghĩa phương thức `providerOptions`:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasProviderOptions;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasProviderOptions
{
    use Promptable;

    // ...

    /**
     * Get provider-specific generation options.
     */
    public function providerOptions(Lab|string $provider): array
    {
        return match ($provider) {
            Lab::OpenAI => [
                'reasoning' => ['effort' => 'low'],
                'frequency_penalty' => 0.5,
                'presence_penalty' => 0.3,
            ],
            Lab::Anthropic => [
                'thinking' => ['budget_tokens' => 1024],
                'cache_control' => ['type' => 'ephemeral'],
            ],
            default => [],
        };
    }
}
```

Phương thức `providerOptions` nhận vào provider đang được sử dụng (enum `Lab` hoặc string), cho phép bạn trả về các tùy chọn khác nhau cho mỗi provider. Điều này đặc biệt hữu ích khi sử dụng [failover](#failover), vì mỗi provider dự phòng có thể nhận cấu hình riêng của chúng.

Ví dụ Anthropic ở trên cũng kích hoạt [lưu bộ nhớ cache prompt](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) thông qua `cache_control`.

<a name="images"></a>
## Images

Class `Laravel\Ai\Image` có thể được sử dụng để tạo hình ảnh thông qua các provider `openai`, `gemini` hoặc `xai`:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

Các phương thức `square`, `portrait`, và `landscape` có thể được dùng để kiểm soát tỷ lệ khung hình của ảnh, trong khi phương thức `quality` có thể hướng dẫn mô hình về chất lượng ảnh cuối cùng (`high`, `medium`, `low`). Phương thức `timeout` được sử dụng để chỉ định thời gian chờ tính bằng giây:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')
    ->quality('high')
    ->landscape()
    ->timeout(120)
    ->generate();
```

Bạn cũng có thể đính kèm các hình ảnh tham chiếu bằng phương thức `attachments`:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Image;

$image = Image::of('Update this photo of me to be in the style of an impressionist painting.')
    ->attachments([
        Files\Image::fromStorage('photo.jpg'),
        // Files\Image::fromPath('/home/laravel/photo.jpg'),
        // Files\Image::fromUrl('https://example.com/photo.jpg'),
        // $request->file('photo'),
    ])
    ->landscape()
    ->generate();
```

Các ảnh được tạo ra có thể dễ dàng lưu trữ trên các disk mặc định đã được cấu hình trong file `config/filesystems.php` của ứng dụng:

```php
$image = Image::of('A donut sitting on the kitchen counter');

$path = $image->store();
$path = $image->storeAs('image.jpg');
$path = $image->storePublicly();
$path = $image->storePubliclyAs('image.jpg');
```

Quá trình tạo ảnh cũng có thể được đưa vào queue:

```php
use Laravel\Ai\Image;
use Laravel\Ai\Responses\ImageResponse;

Image::of('A donut sitting on the kitchen counter')
    ->portrait()
    ->queue()
    ->then(function (ImageResponse $image) {
        $path = $image->store();

        // ...
    });
```

<a name="audio"></a>
## Audio

Class `Laravel\Ai\Audio` có thể dùng để tạo file âm thanh từ một văn bản cho trước:

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();

$rawContent = (string) $audio;
```

Bạn cũng có thể tạo file âm thanh từ một chuỗi bằng phương thức `toAudio` có sẵn thông qua class `Stringable` của Laravel:

```php
use Illuminate\Support\Str;

$audio = Str::of('I love coding with Laravel.')->toAudio();
```

Các phương thức `male`, `female` và `voice` có thể dùng để xác định giọng đọc cho âm thanh được tạo ra:

```php
$audio = Audio::of('I love coding with Laravel.')
    ->female()
    ->generate();

$audio = Audio::of('I love coding with Laravel.')
    ->voice('voice-id-or-name')
    ->generate();
```

Tương tự, phương thức `instructions` có thể được sử dụng để hướng dẫn mô hình cách tạo ra âm thanh dựa theo giọng văn sẽ như thế nào (ở bên dưới giọng sẽ giống như một tên cướp biển):

```php
$audio = Audio::of('I love coding with Laravel.')
    ->female()
    ->instructions('Said like a pirate')
    ->generate();
```

Âm thanh được tạo ra có thể dễ dàng lưu trong disk mặc định được cấu hình trong file `config/filesystems.php` của ứng dụng:

```php
$audio = Audio::of('I love coding with Laravel.')->generate();

$path = $audio->store();
$path = $audio->storeAs('audio.mp3');
$path = $audio->storePublicly();
$path = $audio->storePubliclyAs('audio.mp3');
```

Quá trình tạo âm thanh cũng có thể đưa vào queue:

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Responses\AudioResponse;

Audio::of('I love coding with Laravel.')
    ->queue()
    ->then(function (AudioResponse $audio) {
        $path = $audio->store();

        // ...
    });
```

<a name="transcription"></a>
## Transcriptions

Class `Laravel\Ai\Transcription` có thể được sử dụng để tạo văn bản từ một file âm thanh:

```php
use Laravel\Ai\Transcription;

$transcript = Transcription::fromPath('/home/laravel/audio.mp3')->generate();
$transcript = Transcription::fromStorage('audio.mp3')->generate();
$transcript = Transcription::fromUpload($request->file('audio'))->generate();

return (string) $transcript;
```

Phương thức `diarize` có thể được dùng để chỉ định bạn muốn response chứa cả bản dịch phân loại theo người nói bên cạnh văn bản raw, cho phép bạn truy cập bản dịch được chia theo từng người nói:

```php
$transcript = Transcription::fromStorage('audio.mp3')
    ->diarize()
    ->generate();
```

Quá trình tạo bản dịch cũng có thể được đưa vào queue:

```php
use Laravel\Ai\Transcription;
use Laravel\Ai\Responses\TranscriptionResponse;

Transcription::fromStorage('audio.mp3')
    ->queue()
    ->then(function (TranscriptionResponse $transcript) {
        // ...
    });
```

<a name="embeddings"></a>
## Embeddings

Bạn có thể dễ dàng tạo vector embeddings cho bất kỳ chuỗi nào bằng phương thức mới `toEmbeddings` có sẵn trong class `Stringable` của Laravel:

```php
use Illuminate\Support\Str;

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

Ngoài ra, bạn có thể sử dụng class `Embeddings` để tạo embeddings cho nhiều input cùng một lúc:

```php
use Laravel\Ai\Embeddings;

$response = Embeddings::for([
    'Napa Valley has great wine.',
    'Laravel is a PHP framework.',
])->generate();

$response->embeddings; // [[0.123, 0.456, ...], [0.789, 0.012, ...]]
```

Bạn có thể chỉ định số chiều (dimensions) và provider cho các embeddings:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->dimensions(1536)
    ->generate(Lab::OpenAI, 'text-embedding-3-small');
```

<a name="querying-embeddings"></a>
### Querying Embeddings

Sau khi đã tạo xong embeddings, bạn thường sẽ lưu chúng vào một cột `vector` trong cơ sở dữ liệu để truy vấn sau này. Laravel hỗ trợ native các cột vector trên PostgreSQL thông qua extension `pgvector`. Để bắt đầu, hãy định nghĩa một cột `vector` trong migration của bạn, xác định số chiều:

```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->vector('embedding', dimensions: 1536);
    $table->timestamps();
});
```

Bạn cũng có thể thêm một vector index để tăng tốc độ tìm kiếm. Khi gọi `index` trên một cột vector, Laravel sẽ tự động tạo một HNSW index với cosine distance:

```php
$table->vector('embedding', dimensions: 1536)->index();
```

Trên Eloquent model của bạn, bạn nên cast cột vector sang `array`:

```php
protected function casts(): array
{
    return [
        'embedding' => 'array',
    ];
}
```

Để truy vấn các bản ghi giống nhau, hãy sử dụng phương thức `whereVectorSimilarTo`. Phương thức này lọc kết quả theo độ tương đồng cosine tối thiểu (từ `0.0` đến `1.0`, trong đó `1.0` là giống nhau hoàn toàn) và sắp xếp kết quả theo độ tương đồng:

```php
use App\Models\Document;

$documents = Document::query()
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

Biến `$queryEmbedding` có thể là một mảng float hoặc một chuỗi văn bản. Khi truyền vào một chuỗi, Laravel sẽ tự động tạo embeddings cho chuỗi đó:

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', 'best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

Nếu bạn cần kiểm soát nhiều hơn, bạn có thể sử dụng các phương thức cấp thấp hơn `whereVectorDistanceLessThan`, `selectVectorDistance`, và `orderByVectorDistance` một cách độc lập:

```php
$documents = Document::query()
    ->select('*')
    ->selectVectorDistance('embedding', $queryEmbedding, as: 'distance')
    ->whereVectorDistanceLessThan('embedding', $queryEmbedding, maxDistance: 0.3)
    ->orderByVectorDistance('embedding', $queryEmbedding)
    ->limit(10)
    ->get();
```

Nếu bạn muốn cung cấp cho agent khả năng thực hiện tìm kiếm như một tool, hãy xem tài liệu về [Similarity Search](#similarity-search).

> [!NOTE]
> Các truy vấn vector hiện chỉ được hỗ trợ trên kết nối PostgreSQL sử dụng extension `pgvector`.

<a name="caching-embeddings"></a>
### Caching Embeddings

Quá trình tạo embedding có thể được lưu cache để tránh các lần gọi API thừa. Để kích hoạt cache, hãy thiết lập tùy chọn `ai.caching.embeddings.cache` thành `true`:

```php
'caching' => [
    'embeddings' => [
        'cache' => true,
        'store' => env('CACHE_STORE', 'database'),
        // ...
    ],
],
```

Khi cache được bật, embeddings sẽ được lưu trong 30 ngày. Khóa cache dựa trên provider, mô hình, số chiều và nội dung input, đảm bảo rằng các request giống nhau trả về kết quả đã cache trong khi các cấu hình khác sẽ tạo ra các embedding mới.

Bạn cũng có thể bật cache cho một request cụ thể bằng phương thức `cache`, ngay cả khi global cache đang tắt:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->cache()
    ->generate();
```

Bạn có thể chỉ định thời gian lưu trữ tính bằng giây:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->cache(seconds: 3600) // Cache for 1 hour
    ->generate();
```

Phương thức `toEmbeddings` của Stringable cũng chấp nhận tham số `cache`:

```php
// Cache with default duration...
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings(cache: true);

// Cache for a specific duration...
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings(cache: 3600);
```

<a name="reranking"></a>
## Reranking

Reranking cho phép bạn sắp xếp lại danh sách các tài liệu dựa trên mức độ liên quan của chúng đối với một truy vấn. Điều này hữu ích để cải thiện kết quả tìm kiếm bằng cách sử dụng khả năng hiểu ngôn ngữ.

Class `Laravel\Ai\Reranking` có thể được sử dụng để rerank các tài liệu:

```php
use Laravel\Ai\Reranking;

$response = Reranking::of([
    'Django is a Python web framework.',
    'Laravel is a PHP web application framework.',
    'React is a JavaScript library for building user interfaces.',
])->rerank('PHP frameworks');

// Access the top result...
$response->first()->document; // "Laravel is a PHP web application framework."
$response->first()->score;    // 0.95
$response->first()->index;    // 1 (original position)
```

Phương thức `limit` có thể được dùng để giới hạn số lượng kết quả được trả về:

```php
$response = Reranking::of($documents)
    ->limit(5)
    ->rerank('search query');
```

<a name="reranking-collections"></a>
### Reranking Collections

Để thuận tiện, các Laravel collection có thể được rerank bằng macro `rerank`. Tham số đầu tiên sẽ chỉ định các field để rerank và tham số thứ hai là truy vấn:

```php
// Rerank by a single field...
$posts = Post::all()
    ->rerank('body', 'Laravel tutorials');

// Rerank by multiple fields (sent as JSON)...
$reranked = $posts->rerank(['title', 'body'], 'Laravel tutorials');

// Rerank using a closure to build the document...
$reranked = $posts->rerank(
    fn ($post) => $post->title.': '.$post->body,
    'Laravel tutorials'
);
```

Bạn cũng có thể giới hạn số lượng kết quả và chỉ định provider:

```php
$reranked = $posts->rerank(
    by: 'content',
    query: 'Laravel tutorials',
    limit: 10,
    provider: Lab::Cohere
);
```

<a name="files"></a>
## Files

Class `Laravel\Ai\Files` hoặc các class file khác có thể được sử dụng để lưu các file cùng với AI provider của bạn để dùng cho các cuộc hội thoại sau này. Điều này hữu ích đối với các tài liệu lớn hoặc các file bạn mà muốn tham chiếu nhiều lần mà không cần upload lại:

```php
use Laravel\Ai\Files\Document;
use Laravel\Ai\Files\Image;

// Store a file from a local path...
$response = Document::fromPath('/home/laravel/document.pdf')->put();
$response = Image::fromPath('/home/laravel/photo.jpg')->put();

// Store a file that is stored on a filesystem disk...
$response = Document::fromStorage('document.pdf', disk: 'local')->put();
$response = Image::fromStorage('photo.jpg', disk: 'local')->put();

// Store a file that is stored on a remote URL...
$response = Document::fromUrl('https://example.com/document.pdf')->put();
$response = Image::fromUrl('https://example.com/photo.jpg')->put();

return $response->id;
```

Bạn cũng có thể lưu nội dung raw hoặc các file được upload:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Files\Document;

// Store raw content...
$stored = Document::fromString('Hello, World!', 'text/plain')->put();

// Store an uploaded file...
$stored = Document::fromUpload($request->file('document'))->put();
```

Khi file đã được lưu, bạn có thể tham chiếu lại file đó khi tạo text thông qua các agent thay vì upload lại:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached sales transcript...'
    attachments: [
        Files\Document::fromId('file-id') // Attach a stored document...
    ]
);
```

Để lấy ra một file đã lưu trước đó, hãy dùng phương thức `get` trên một instance của file:

```php
use Laravel\Ai\Files\Document;

$file = Document::fromId('file-id')->get();

$file->id;
$file->mimeType();
```

Để xóa một file khỏi provider, hãy dùng phương thức `delete`:

```php
Document::fromId('file-id')->delete();
```

Mặc định, class `Files` sử dụng provider AI mặc định trong file `config/ai.php`. Với phần lớn các thao tác, bạn có thể chỉ định một provider khác bằng tham số `provider`:

```php
$response = Document::fromPath(
    '/home/laravel/document.pdf'
)->put(provider: Lab::Anthropic);
```

Bạn có thể truyền thêm các tùy chọn upload cụ thể cho từng provider bằng cách sử dụng phương thức `withProviderOptions`. Ví dụ: bạn có thể thiết lập thuộc tính `purpose` của file cho OpenAI:

```php
use Laravel\Ai\Files\Document;

$response = Document::fromPath('/home/laravel/knowledge.txt')
    ->withProviderOptions(['purpose' => 'assistants'])
    ->put();
```

Để giới hạn phạm vi các tùy chọn cho từng provider, hãy truyền một closure nhận provider hiện tại làm tham số:

```php
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Files\Document;

$response = Document::fromPath('/home/laravel/training.jsonl')
    ->withProviderOptions(fn (Lab|string $provider) => match ($provider) {
        Lab::OpenAI => ['purpose' => 'fine-tune'],
        default => [],
    })
    ->put();
```

<a name="using-stored-files-in-conversations"></a>
### Using Stored Files in Conversations

Một khi file đã được lưu vào provider, bạn có thể tham chiếu lại nó trong các agent bằng phương thức `fromId` của class `Document` hoặc `Image`:

```php
use App\Ai\Agents\DocumentAnalyzer;
use Laravel\Ai\Files;
use Laravel\Ai\Files\Document;

$stored = Document::fromPath('/path/to/report.pdf')->put();

$response = (new DocumentAnalyzer)->prompt(
    'Summarize this document.',
    attachments: [
        Document::fromId($stored->id),
    ],
);
```

Tương tự, các hình ảnh mà đã được lưu cũng có thể được tham chiếu bằng class `Image`:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Files\Image;

$stored = Image::fromPath('/path/to/photo.jpg')->put();

$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Image::fromId($stored->id),
    ],
);
```

<a name="vector-stores"></a>
## Vector Stores

Các vector store cho phép bạn tạo ra các bộ sưu tập file có thể tìm kiếm được, có thể được dùng cho retrieval-augmented generation (RAG). Class `Laravel\Ai\Stores` cung cấp các phương thức để tạo, truy xuất và xóa các vector store:

```php
use Laravel\Ai\Stores;

// Create a new vector store...
$store = Stores::create('Knowledge Base');

// Create a store with additional options...
$store = Stores::create(
    name: 'Knowledge Base',
    description: 'Documentation and reference materials.',
    expiresWhenIdleFor: days(30),
);

return $store->id;
```

Để lấy ra một vector store hiện có bằng ID của nó, hãy sử dụng phương thức `get`:

```php
use Laravel\Ai\Stores;

$store = Stores::get('store_id');

$store->id;
$store->name;
$store->fileCounts;
$store->ready;
```

Để xóa một vector store, sử dụng phương thức `delete` trên class `Stores` hoặc trên một store instance:

```php
use Laravel\Ai\Stores;

// Delete by ID...
Stores::delete('store_id');

// Or delete via a store instance...
$store = Stores::get('store_id');

$store->delete();
```

<a name="adding-files-to-stores"></a>
### Thêm File vào Stores

Một khi có một vector store, bạn có thể thêm [files](#files) vào nó bằng phương thức `add`. Các file được thêm vào store sẽ tự động được đánh index cho tìm kiếm bằng cách sử dụng [file search provider tool](#file-search):

```php
use Laravel\Ai\Files\Document;
use Laravel\Ai\Stores;

$store = Stores::get('store_id');

// Add a file that has already been stored with the provider...
$document = $store->add('file_id');
$document = $store->add(Document::fromId('file_id'));

// Or, store and add a file in one step...
$document = $store->add(Document::fromPath('/path/to/document.pdf'));
$document = $store->add(Document::fromStorage('manual.pdf'));
$document = $store->add($request->file('document'));

$document->id;
$document->fileId;
```

> **Note:** Thông thường, khi thêm các file đã lưu trước đó vào vector store, ID trả về sẽ trùng với ID đã gán trước đó của file; tuy nhiên, một số provider có thể trả về một "document ID" mới khác biệt với ID cũ. Do đó, bạn nên lưu cả hai ID vào trong cơ sở dữ liệu để tham chiếu sau này.

Bạn có thể đính kèm thêm metadata vào các file khi thêm vào store. Metadata này có thể được dùng để lọc kết quả tìm kiếm khi dùng [file search provider tool](#file-search):

```php
$store->add(Document::fromPath('/path/to/document.pdf'), metadata: [
    'author' => 'Taylor Otwell',
    'department' => 'Engineering',
    'year' => 2026,
]);
```

Để xóa một file ra khỏi store, hãy dùng phương thức `remove`:

```php
$store->remove('file_id');
```

Xóa một file ra khỏi vector store không có nghĩa là xóa nó ra khỏi [file storage](#files) của provider. Để xóa file ra khỏi cả hai store, hãy dùng tham số `deleteFile`:

```php
$store->remove('file_abc123', deleteFile: true);
```

<a name="failover"></a>
## Failover

Khi gửi prompt hoặc tạo các nội dung khác, bạn có thể cung cấp một mảng các provider hoặc model để tự động dự phòng sang provider hoặc model khác nếu gặp sự cố hoặc bị giới hạn tỷ lệ ở provider chính:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Image;

$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: [Lab::OpenAI, Lab::Anthropic],
);

$image = Image::of('A donut sitting on the kitchen counter')
    ->generate(provider: [Lab::Gemini, Lab::xAI]);
```

Cơ chế dự phòng chỉ xảy ra khi một ngoại lệ `FailoverableException` được đưa ra — ví dụ như giới hạn lượt yêu cầu (`RateLimitedException`), nhà cung cấp bị quá tải hoặc không khả dụng (`ProviderOverloadedException`), hoặc không đủ số dư tài khoản (`InsufficientCreditsException`). Các lỗi thông thường, chẳng hạn như lỗi xác thực hoặc yêu cầu không hợp lệ, sẽ không kích hoạt dự phòng.

Khi bạn truyền vào một danh sách các nhà cung cấp, chẳng hạn như `[Lab::OpenAI, Lab::Anthropic]`, mỗi nhà cung cấp sẽ sử dụng model mặc định của họ. Để chỉ định một model cụ thể cho từng nhà cung cấp trong danh sách dự phòng, hãy truyền vào một mảng gồm key là nhà cung cấp có sử dụng `value` của enum `Lab` (bởi vì các enum không thể được sử dụng trực tiếp làm key cho mảng PHP):

```php
use Laravel\Ai\Enums\Lab;

$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: [
        Lab::Gemini->value => 'gemini-3-flash-preview',
        Lab::DeepSeek->value => 'deepseek-v4-pro',
    ],
);
```

<a name="testing"></a>
## Testing

<a name="testing-agents"></a>
### Agents

Để giả lập các response của agent trong các bài test, gọi phương thức `fake` trên class của agent. Bạn có thể cấp một mảng các response hoặc một closure:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Prompts\AgentPrompt;

// Automatically generate a fixed response for every prompt...
SalesCoach::fake();

// Provide a list of prompt responses...
SalesCoach::fake([
    'First response',
    'Second response',
]);

// Dynamically handle prompt responses based on the incoming prompt...
SalesCoach::fake(function (AgentPrompt $prompt) {
    return 'Response for: '.$prompt->prompt;
});
```

Khi viết fake cho một agent trả về output có cấu trúc, bạn có thể cung cấp các mảng làm response. Agent sẽ trả về một response có cấu trúc chứa dữ liệu đã cho:

```php
SalesCoach::fake([
    ['score' => 87],
]);
```

> **Note:** Khi `Agent::fake()` được gọi trên một agent trả về output có cấu trúc và dữ liệu fake không được cung cấp một cách rõ ràng, Laravel sẽ tự động tạo dữ liệu fake giống với schema mà agent đã định nghĩa.

Sau khi gửi prompt, bạn có thể thực hiện các kiểm tra về các prompt đã nhận:

```php
use Laravel\Ai\Prompts\AgentPrompt;

SalesCoach::assertPrompted('Analyze this...');

SalesCoach::assertPrompted(function (AgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotPrompted('Missing prompt');

SalesCoach::assertNeverPrompted();
```

Đối với các lời gọi agent trong queue, hãy sử dụng các phương thức kiểm tra dành cho queue:

```php
use Laravel\Ai\QueuedAgentPrompt;

SalesCoach::assertQueued('Analyze this...');

SalesCoach::assertQueued(function (QueuedAgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotQueued('Missing prompt');

SalesCoach::assertNeverQueued();
```

Để đảm bảo mọi lượt gọi agent đều có fake response tương ứng, hãy dùng `preventStrayPrompts`. Nếu agent được gọi mà không có fake response, một exception sẽ được đưa ra:

```php
SalesCoach::fake()->preventStrayPrompts();
```

<a name="testing-images"></a>
### Images

Quá trình tạo hình ảnh có thể được fake bằng cách gọi phương thức `fake` trên class `Image`. Sau khi được fake, các kiểm tra có thể được thực hiện với các prompt tạo ảnh:

```php
use Laravel\Ai\Image;
use Laravel\Ai\Prompts\ImagePrompt;
use Laravel\Ai\Prompts\QueuedImagePrompt;

// Automatically generate a fixed response for every prompt...
Image::fake();

// Provide a list of prompt responses...
Image::fake([
    base64_encode($firstImage),
    base64_encode($secondImage),
]);

// Dynamically handle prompt responses based on the incoming prompt...
Image::fake(function (ImagePrompt $prompt) {
    return base64_encode('...');
});
```

Sau khi tạo ảnh, bạn có thể thực hiện các kiểm tra về các prompt đã nhận:

```php
Image::assertGenerated(function (ImagePrompt $prompt) {
    return $prompt->contains('sunset') && $prompt->isLandscape();
});

Image::assertNotGenerated('Missing prompt');

Image::assertNothingGenerated();
```

Đối với các lần tạo ảnh trong queue, hãy sử dụng các phương thức kiểm tra dành cho queue:

```php
Image::assertQueued(
    fn (QueuedImagePrompt $prompt) => $prompt->contains('sunset')
);

Image::assertNotQueued('Missing prompt');

Image::assertNothingQueued();
```

Để đảm bảo mọi lần tạo ảnh đều có fake response, hãy dùng `preventStrayImages`. Nếu ảnh được tạo mà không có fake response, một exception sẽ được đưa ra:

```php
Image::fake()->preventStrayImages();
```

<a name="testing-audio"></a>
### Audio

Quá trình tạo âm thanh có thể được fake bằng cách gọi phương thức `fake` trên class `Audio`. Sau đó các kiểm tra có thể được thực hiện với các prompt âm thanh:

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Prompts\AudioPrompt;
use Laravel\Ai\Prompts\QueuedAudioPrompt;

// Automatically generate a fixed response for every prompt...
Audio::fake();

// Provide a list of prompt responses...
Audio::fake([
    base64_encode($firstAudio),
    base64_encode($secondAudio),
]);

// Dynamically handle prompt responses based on the incoming prompt...
Audio::fake(function (AudioPrompt $prompt) {
    return base64_encode('...');
});
```

Sau khi tạo âm thanh, bạn có thể thực hiện các kiểm tra về các prompt đã nhận:

```php
Audio::assertGenerated(function (AudioPrompt $prompt) {
    return $prompt->contains('Hello') && $prompt->isFemale();
});

Audio::assertNotGenerated('Missing prompt');

Audio::assertNothingGenerated();
```

Đối với các lần tạo âm thanh trong queue, hãy sử dụng các phương thức kiểm tra dành cho queue:

```php
Audio::assertQueued(
    fn (QueuedAudioPrompt $prompt) => $prompt->contains('Hello')
);

Audio::assertNotQueued('Missing prompt');

Audio::assertNothingQueued();
```

Để đảm bảo mọi lần tạo âm thanh đều có fake response, hãy dùng `preventStrayAudio`. Nếu âm thanh được tạo mà không có fake response, một exception sẽ được đưa ra:

```php
Audio::fake()->preventStrayAudio();
```

<a name="testing-transcriptions"></a>
### Transcriptions

Quá trình tạo văn bản dịch có thể được fake bằng cách gọi `fake` trên class `Transcription`. Sau đó các kiểm tra có thể được thực hiện với các prompt văn bản dịch:

```php
use Laravel\Ai\Transcription;
use Laravel\Ai\Prompts\TranscriptionPrompt;
use Laravel\Ai\Prompts\QueuedTranscriptionPrompt;

// Automatically generate a fixed response for every prompt...
Transcription::fake();

// Provide a list of prompt responses...
Transcription::fake([
    'First transcription text.',
    'Second transcription text.',
]);

// Dynamically handle prompt responses based on the incoming prompt...
Transcription::fake(function (TranscriptionPrompt $prompt) {
    return 'Transcribed text...';
});
```

Sau khi tạo văn bản dịch, bạn có thể kiểm tra các prompt đã nhận:

```php
Transcription::assertGenerated(function (TranscriptionPrompt $prompt) {
    return $prompt->language === 'en' && $prompt->isDiarized();
});

Transcription::assertNotGenerated(
    fn (TranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingGenerated();
```

Đối với các lần tạo văn bản dịch trong queue, hãy sử dụng các phương thức kiểm tra dành cho queue:

```php
Transcription::assertQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->isDiarized()
);

Transcription::assertNotQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingQueued();
```

Để đảm bảo mọi văn bản dịch đều có fake response, hãy dùng `preventStrayTranscriptions`. Nếu văn bản dịch được tạo mà không có fake response, một exception sẽ được đưa ra:

```php
Transcription::fake()->preventStrayTranscriptions();
```

<a name="testing-embeddings"></a>
### Embeddings

Quá trình tạo embeddings có thể được fake bằng cách gọi `fake` trên class `Embeddings`. Sau đó các kiểm tra có thể được thực hiện với các prompt embeddings:

```php
use Laravel\Ai\Embeddings;
use Laravel\Ai\Prompts\EmbeddingsPrompt;
use Laravel\Ai\Prompts\QueuedEmbeddingsPrompt;

// Automatically generate fake embeddings of the proper dimensions for every prompt...
Embeddings::fake();

// Provide a list of prompt responses...
Embeddings::fake([
    [$firstEmbeddingVector],
    [$secondEmbeddingVector],
]);

// Dynamically handle prompt responses based on the incoming prompt...
Embeddings::fake(function (EmbeddingsPrompt $prompt) {
    return array_map(
        fn () => Embeddings::fakeEmbedding($prompt->dimensions),
        $prompt->inputs
    );
});
```

Sau khi tạo embeddings, bạn có thể kiểm tra các prompt đã nhận:

```php
Embeddings::assertGenerated(function (EmbeddingsPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->dimensions === 1536;
});

Embeddings::assertNotGenerated(
    fn (EmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingGenerated();
```

Đối với embeddings trong queue, hãy sử dụng các phương thức kiểm tra dành cho queue:

```php
Embeddings::assertQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Laravel')
);

Embeddings::assertNotQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingQueued();
```

Để đảm bảo mọi embeddings đều có fake response, hãy dùng `preventStrayEmbeddings`. Nếu embeddings được tạo mà không có fake response, một exception sẽ được đưa ra:

```php
Embeddings::fake()->preventStrayEmbeddings();
```

<a name="testing-reranking"></a>
### Reranking

Các hoạt động reranking có thể được fake bằng cách gọi `fake` trên class `Reranking`:

```php
use Laravel\Ai\Reranking;
use Laravel\Ai\Prompts\RerankingPrompt;
use Laravel\Ai\Responses\Data\RankedDocument;

// Automatically generate a fake reranked responses...
Reranking::fake();

// Provide custom responses...
Reranking::fake([
    [
        new RankedDocument(index: 0, document: 'First', score: 0.95),
        new RankedDocument(index: 1, document: 'Second', score: 0.80),
    ],
]);
```

Sau khi reranking, bạn có thể kiểm tra các hoạt động đã thực hiện:

```php
Reranking::assertReranked(function (RerankingPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->limit === 5;
});

Reranking::assertNotReranked(
    fn (RerankingPrompt $prompt) => $prompt->contains('Django')
);

Reranking::assertNothingReranked();
```

<a name="testing-files"></a>
### Files

Các thao tác file có thể được fake bằng cách gọi `fake` trên class `Files`:

```php
use Laravel\Ai\Files;

Files::fake();
```

Một khi các thao tác file đã được fake, bạn có thể kiểm tra các lần upload và xóa đã xảy ra:

```php
use Laravel\Ai\Contracts\Files\StorableFile;
use Laravel\Ai\Files\Document;

// Store files...
Document::fromString('Hello, Laravel!', mimeType: 'text/plain')
    ->as('hello.txt')
    ->put();

// Make assertions...
Files::assertStored(fn (StorableFile $file) =>
    (string) $file === 'Hello, Laravel!' &&
        $file->mimeType() === 'text/plain';
);

Files::assertNotStored(fn (StorableFile $file) =>
    (string) $file === 'Hello, World!'
);

Files::assertNothingStored();
```

Để kiểm tra việc xóa file, bạn có thể truyền vào một file ID:

```php
Files::assertDeleted('file-id');
Files::assertNotDeleted('file-id');
Files::assertNothingDeleted();
```

<a name="testing-vector-stores"></a>
### Vector Stores

Các thao tác vector store có thể được fake bằng cách gọi `fake` trên class `Stores`. Việc fake stores cũng sẽ tự động fake [các thao tác file](#files):

```php
use Laravel\Ai\Stores;

Stores::fake();
```

Sau khi các thao tác store được fake, bạn có thể kiểm tra các store đã được tạo hoặc xóa:

```php
use Laravel\Ai\Stores;

// Create store...
$store = Stores::create('Knowledge Base');

// Make assertions...
Stores::assertCreated('Knowledge Base');

Stores::assertCreated(fn (string $name, ?string $description) =>
    $name === 'Knowledge Base'
);

Stores::assertNotCreated('Other Store');

Stores::assertNothingCreated();
```

Để kiểm tra việc xóa store, bạn có thể cung cấp store ID:

```php
Stores::assertDeleted('store_id');
Stores::assertNotDeleted('other_store_id');
Stores::assertNothingDeleted();
```

Để kiểm tra xem file đã được thêm vào hoặc xóa ra khỏi một store, hãy sử dụng các phương thức kiểm tra trên một `Store` instance cụ thể:

```php
Stores::fake();

$store = Stores::get('store_id');

// Add / remove files...
$store->add('added_id');
$store->remove('removed_id');

// Make assertions...
$store->assertAdded('added_id');
$store->assertRemoved('removed_id');

$store->assertNotAdded('other_file_id');
$store->assertNotRemoved('other_file_id');
```

Nếu một file được lưu trong [file storage](#files) của provider và được thêm vào vector store trong cùng một request, bạn có thể không biết được provider ID của file. Trong trường hợp này, bạn có thể truyền một closure vào phương thức `assertAdded` để kiểm tra nội dung của file đã thêm:

```php
use Laravel\Ai\Contracts\Files\StorableFile;
use Laravel\Ai\Files\Document;

$store->add(Document::fromString('Hello, World!', 'text/plain')->as('hello.txt'));

$store->assertAdded(fn (StorableFile $file) => $file->name() === 'hello.txt');
$store->assertAdded(fn (StorableFile $file) => $file->content() === 'Hello, World!');
```

<a name="events"></a>
## Events

Laravel AI SDK sẽ gửi nhiều loại [event](/docs/{{version}}/events), bao gồm:

- `AddingFileToStore`
- `AgentPrompted`
- `AgentStreamed`
- `AudioGenerated`
- `CreatingStore`
- `EmbeddingsGenerated`
- `FileAddedToStore`
- `FileDeleted`
- `FileRemovedFromStore`
- `FileStored`
- `GeneratingAudio`
- `GeneratingEmbeddings`
- `GeneratingImage`
- `GeneratingTranscription`
- `ImageGenerated`
- `InvokingTool`
- `PromptingAgent`
- `RemovingFileFromStore`
- `Reranked`
- `Reranking`
- `StoreCreated`
- `StoringFile`
- `StreamingAgent`
- `ToolInvoked`
- `TranscriptionGenerated`

Bạn có thể lắng nghe bất kỳ event nào có trong danh sách ở trên để ghi log hoặc lưu trữ thông tin sử dụng AI SDK.
