# Search

- [Giới thiệu](#introduction)
    - [Full-Text Search](#introduction-full-text-search)
    - [Semantic / Vector Search](#introduction-semantic-vector-search)
    - [Reranking](#introduction-reranking)
    - [Scout Search Engines](#introduction-scout-search-engines)
- [Full-Text Search](#full-text-search)
    - [Adding Full-Text Indexes](#adding-full-text-indexes)
    - [Running Full-Text Queries](#running-full-text-queries)
- [Semantic / Vector Search](#semantic-vector-search)
    - [Generating Embeddings](#generating-embeddings)
    - [Storing and Indexing Vectors](#storing-and-indexing-vectors)
    - [Querying by Similarity](#querying-by-similarity)
- [Reranking Results](#reranking-results)
- [Laravel Scout](#laravel-scout)
    - [Database Engine](#database-engine)
    - [Third-Party Engines](#third-party-engines)
- [Combining Techniques](#combining-techniques)

<a name="introduction"></a>
## Giới thiệu

Hầu hết mọi ứng dụng đều cần chức năng tìm kiếm. Dù người dùng của bạn đang tìm kiếm các bài viết liên quan trong cơ sở dữ liệu, hoặc danh mục sản phẩm, hoặc đặt câu hỏi bằng ngôn ngữ tự nhiên trên một tập hợp tài liệu, thì Laravel đều cung cấp các công cụ được tích hợp sẵn để xử lý các tình huống này — và thường thì bạn không cần sử dụng thêm bất kỳ dịch vụ bên ngoài nào.

Phần lớn ứng dụng sẽ thấy rằng các tùy chọn cơ sở dữ liệu được tích hợp sẵn của Laravel là hoàn toàn đủ dùng — các dịch vụ tìm kiếm bên ngoài chỉ cần thiết khi bạn cần các tính năng như xử lý lỗi chính tả, lọc đa chiều, hoặc tìm kiếm theo vị trí địa lý ở quy mô rất lớn.

<a name="introduction-full-text-search"></a>
#### Full-Text Search

Khi bạn cần sắp xếp theo độ liên quan của từ khóa — tức là cơ sở dữ liệu chấm điểm và sắp xếp kết quả dựa trên mức độ giống với từ khóa đang tìm kiếm — thì phương thức `whereFullText` của query builder trong Laravel sẽ tận dụng các full-text index tương ứng của MariaDB, MySQL và PostgreSQL. Full-text search có thể hiểu được ranh giới từ và các từ gốc, vì vậy một tìm kiếm có từ "running" có thể khớp với các record có từ "run". Mà không cần sử dụng dịch vụ bên ngoài.

<a name="introduction-semantic-vector-search"></a>
#### Semantic / Vector Search

Đối với tìm kiếm ngữ nghĩa dựa trên trí tuệ nhân tạo, kết quả sẽ khớp với *ý nghĩa* chứ không phải từ khóa chính xác, phương thức `whereVectorSimilarTo` của query builder sẽ sử dụng vector embeddings được lưu trong PostgreSQL với extension `pgvector`. Ví dụ, tìm kiếm từ khóa "best wineries in Napa Valley" có thể trả về bài viết có tiêu đề là "Top Vineyards to Visit" — dù các từ khóa không trùng nhau. Tìm kiếm vector sẽ yêu cầu PostgreSQL với extension `pgvector` và [Laravel AI SDK](/docs/{{version}}/ai-sdk).

<a name="introduction-reranking"></a>
#### Reranking

[AI SDK](/docs/{{version}}/ai-sdk) của Laravel cung cấp khả năng reranking, sử dụng các mô hình AI để sắp xếp lại bất kỳ tập hợp kết quả nào theo mức độ liên quan ngữ nghĩa với câu truy vấn. Reranking đặc biệt hiệu quả khi là bước thứ hai sau một bước truy xuất đầu tiên như tìm kiếm full-text search — nó cho bạn cả tốc độ lẫn độ chính xác ngữ nghĩa.

<a name="introduction-scout-search-engines"></a>
#### Laravel Scout Search

Với các ứng dụng muốn dùng trait `Searchable` để tự động đồng bộ search index với các Eloquent model, [Laravel Scout](/docs/{{version}}/scout) cung cấp cả database engine lẫn các driver cho dịch vụ bên thứ ba như Algolia, Meilisearch và Typesense.

<a name="full-text-search"></a>
## Full-Text Search

Mặc dù các truy vấn `LIKE` hoạt động tốt cho việc tìm các chuỗi con đơn giản, nhưng chúng không hiểu ngữ nghĩa. Tìm kiếm `LIKE` cho "running" sẽ không tìm thấy các record chứa từ "run", và kết quả không được sắp xếp theo độ liên quan — chúng chỉ trả về theo thứ tự cơ sở dữ liệu được tìm thấy. Full-text search sẽ giải quyết cả hai vấn đề này bằng cách sử dụng các index chuyên dụng hiểu ranh giới từ, từ gốc và đánh giá độ liên quan, cho phép cơ sở dữ liệu trả về kết quả liên quan nhất ở vị trí đầu tiên.

Full-text search đã được tích hợp sẵn trong MariaDB, MySQL và PostgreSQL — không cần sử dụng dịch vụ bên ngoài. Bạn chỉ cần thêm full-text index vào các cột muốn tìm kiếm, rồi sau đó dùng phương thức `whereFullText` của query builder.

> [!WARNING]
> Full-text search hiện được hỗ trợ bởi MariaDB, MySQL và PostgreSQL.

<a name="adding-full-text-indexes"></a>
### Adding Full-Text Indexes

Để sử dụng full-text search, trước tiên hãy thêm full-text index vào các cột mà bạn muốn tìm kiếm. Bạn có thể thêm index vào một cột hoặc truyền một mảng các cột để tạo index kết hợp tìm kiếm trên nhiều field cùng một lúc:

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('body');
    $table->timestamps();

    $table->fullText(['title', 'body']);
});
```

Trên PostgreSQL, bạn có thể chỉ định cấu hình ngôn ngữ cho index, kiểm soát cách các từ được xử lý:

```php
$table->fullText('body')->language('english');
```

Để xem thêm thông tin về tạo index, tham khảo [tài liệu migration](/docs/{{version}}/migrations#available-index-types).

<a name="running-full-text-queries"></a>
### Running Full-Text Queries

Một khi index đã được thêm, hãy dùng phương thức `whereFullText` của query builder để tìm kiếm. Laravel sẽ tạo câu lệnh SQL phù hợp cho driver cơ sở dữ liệu của bạn — ví dụ, `MATCH(...) AGAINST(...)` trên MariaDB và MySQL, và `to_tsvector(...) @@ plainto_tsquery(...)` trên PostgreSQL:

```php
$articles = Article::whereFullText('body', 'web developer')->get();
```

Khi dùng MariaDB và MySQL, kết quả tự động được sắp xếp theo điểm liên quan. Trên PostgreSQL, câu lệnh `whereFullText` sẽ lấy ra các bản ghi khớp nhưng không sắp xếp theo mức độ liên quan — nên nếu cần sắp xếp tự động trên PostgreSQL, hãy cân nhắc sử dụng [database engine của Scout](#database-engine), cái này sẽ xử lý giúp bạn.

Nếu bạn đã tạo full-text index kết hợp trên nhiều cột, bạn có thể tìm kiếm trên tất cả các cột bằng cách truyền một mảng các cột vào `whereFullText`:

```php
$articles = Article::whereFullText(
    ['title', 'body'], 'web developer'
)->get();
```

Phương thức `orWhereFullText` có thể được dùng để thêm một mệnh đề full-text search khác dưới dạng điều kiện "or". Xem chi tiết trong [tài liệu query builder](/docs/{{version}}/queries#full-text-where-clauses).

<a name="semantic-vector-search"></a>
## Semantic / Vector Search

Full-text search sẽ dựa vào việc khớp với từ khóa — các từ trong truy vấn phải xuất hiện (dưới dạng nào đó) trong dữ liệu. Tìm kiếm ngữ nghĩa có cách tiếp cận hoàn toàn khác: nó sử dụng vector embeddings do AI tạo ra để biểu diễn *ý nghĩa* của văn bản dưới dạng mảng số, rồi tìm kết quả có ý nghĩa gần giống nhất với truy vấn. Ví dụ, tìm kiếm "best wineries in Napa Valley" có thể lấy ra các bài viết có tiêu đề "Top Vineyards to Visit" — dù các từ không giống nhau một chút nào.

Quy trình cơ bản của vector search là: tạo embedding (mảng số) cho từng đoạn nội dung và lưu cùng với dữ liệu, rồi sau đó khi tìm kiếm, tạo embedding cho truy vấn của người dùng và tìm các embedding đã lưu gần nhất trong không gian vector.

> [!NOTE]
> Tìm kiếm vector yêu cầu [Laravel AI SDK](/docs/{{version}}/ai-sdk) và được hỗ trợ bởi PostgreSQL (yêu cầu extension `pgvector`) và MongoDB (yêu cầu [package Laravel MongoDB](https://laravel.com/docs/13.x/mongodb)). Tất cả cơ sở dữ liệu Postgres trên [Laravel Cloud](https://laravel.com/cloud) đều đã được cài đặt sẵn extension `pgvector`.

<a name="generating-embeddings"></a>
### Generating Embeddings

Embedding là một mảng số nhiều chiều (thường có hàng trăm hoặc hàng nghìn số) biểu diễn ý nghĩa của một đoạn văn bản. Bạn có thể tạo embeddings cho một chuỗi bằng phương thức `toEmbeddings` có sẵn trên class `Stringable` của Laravel:

```php
use Illuminate\Support\Str;

$embedding = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

Để tạo embeddings cho nhiều input cùng lúc — nó sẽ hiệu quả hơn việc tạo từng cái một vì chỉ cần một lần gọi API đến provider embedding — sử dụng class `Embeddings`:

```php
use Laravel\Ai\Embeddings;

$response = Embeddings::for([
    'Napa Valley has great wine.',
    'Laravel is a PHP framework.',
])->generate();

$response->embeddings; // [[0.123, 0.456, ...], [0.789, 0.012, ...]]
```

Xem thêm chi tiết về cấu hình embedding provider, tùy chỉnh số chiều và caching, tham khảo [tài liệu AI SDK](/docs/{{version}}/ai-sdk#embeddings).

<a name="storing-and-indexing-vectors"></a>
### Storing and Indexing Vectors

Để lưu vector embeddings, hãy định nghĩa một cột `vector` trong migration, chỉ định số chiều khớp với output của embedding provider (ví dụ, 1536 cho model `text-embedding-3-small` của OpenAI). Bạn cũng nên gọi `index` trên cột để tạo một index HNSW (Hierarchical Navigable Small World), giúp tăng đáng kể tốc độ tìm kiếm trên tập dữ liệu lớn:

```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->vector('embedding', dimensions: 1536)->index();
    $table->timestamps();
});
```

Phương thức `Schema::ensureVectorExtensionExists` sẽ đảm bảo extension `pgvector` sẽ được kích hoạt trên PostgreSQL trước khi tạo bảng.

Trên Eloquent model, hãy cast cột vector sang `array` để Laravel tự động xử lý việc chuyển giữa mảng PHP và định dạng vector của cơ sở dữ liệu:

```php
protected function casts(): array
{
    return [
        'embedding' => 'array',
    ];
}
```

Để xem thêm chi tiết về các kiểu cột và index vector, tham khảo [tài liệu migration](/docs/{{version}}/migrations#available-column-types).

<a name="querying-by-similarity"></a>
### Querying by Similarity

Một khi đã lưu embeddings cho nội dung của bạn, bạn có thể tìm kiếm các record tương tự bằng phương thức `whereVectorSimilarTo`. Phương thức này sẽ so sánh embedding được cho với các vector đã lưu trước đó bằng cosine similarity, lọc bỏ kết quả dưới ngưỡng `minSimilarity`, và tự động sắp xếp kết quả theo độ liên quan — record giống với câu truy vấn nhất sẽ hiện đầu tiên. Ngưỡng nên là giá trị từ `0.0` đến `1.0`, trong đó `1.0` có nghĩa là hai vector giống nhau hoàn toàn:

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

Để thuận lợi hơn, khi truyền một chuỗi string thay vì mảng embedding, Laravel sẽ tự động tạo embedding cho bạn dùng embedding provider đã được cấu hình sẵn. Nghĩa là bạn có thể truyền thẳng truy vấn của người dùng mà không cần chuyển thủ công sang embedding:

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', 'best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

Để kiểm soát sâu hơn đối với các truy vấn vector, các phương thức `whereVectorDistanceLessThan`, `selectVectorDistance` và `orderByVectorDistance` cũng có sẵn. Các phương thức này cho phép bạn làm việc trực tiếp với các giá trị khoảng cách thay vì điểm tương đồng, chọn khoảng cách được tính toán làm một cột trong kết quả của bạn hoặc kiểm soát thứ tự sắp xếp. Để biết chi tiết, hãy tham khảo [tài liệu query builder](/docs/{{version}}/queries#vector-similarity-clauses) và [tài liệu AI SDK](/docs/{{version}}/ai-sdk#querying-embeddings).

<a name="reranking-results"></a>
## Reranking Results

Reranking là một kỹ thuật trong đó một mô hình AI sắp xếp lại một tập hợp các kết quả dựa trên mức độ liên quan ngữ nghĩa của từng kết quả đối với một truy vấn nhất định. Không giống như tìm kiếm vector, yêu cầu bạn phải tính toán trước và lưu trữ embeddings, reranking hoạt động trên bất kỳ tập hợp văn bản nào — nó lấy nội dung raw và truy vấn làm input và trả về các mục được sắp xếp theo mức độ liên quan.

Reranking đặc biệt mạnh mẽ như một giai đoạn thứ hai sau một bước truy xuất ban đầu nhanh chóng. Ví dụ, bạn có thể sử dụng tìm kiếm full-text để nhanh chóng thu hẹp hàng nghìn bản ghi xuống còn 50 bản ghi, và sau đó sử dụng reranking để sắp xếp các kết quả liên quan nhất lên đầu. Cách tiếp cận "truy xuất rồi rerank" này mang lại cho bạn cả tốc độ lẫn độ chính xác ngữ nghĩa.

You may rerank an array of strings using the `Reranking` class:

```php
use Laravel\Ai\Reranking;

$response = Reranking::of([
    'Django is a Python web framework.',
    'Laravel is a PHP web application framework.',
    'React is a JavaScript library for building user interfaces.',
])->rerank('PHP frameworks');

$response->first()->document; // "Laravel is a PHP web application framework."
```

Các Laravel collection cũng có các macro `rerank` nhận tên field (hoặc closure) và một truy vấn, giúp dễ dàng rerank các kết quả trong Eloquent:

```php
$articles = Article::all()
    ->rerank('body', 'Laravel tutorials');
```

Để xem thêm chi tiết về cấu hình provider reranking và các tùy chọn, tham khảo [tài liệu AI SDK](/docs/{{version}}/ai-sdk#reranking).

<a name="laravel-scout"></a>
## Laravel Scout

Các kỹ thuật tìm kiếm được mô tả ở trên đều là các phương thức query builder mà bạn gọi trực tiếp trong code. [Laravel Scout](/docs/{{version}}/scout) có cách tiếp cận khác: nó cung cấp trait `Searchable` để thêm vào các Eloquent model, và Scout tự động giữ search index đồng bộ khi các record được tạo, cập nhật và xóa. Điều này đặc biệt thuận tiện khi bạn muốn các model luôn có thể tìm kiếm mà không cần quản lý việc cập nhật các index.

<a name="database-engine"></a>
### Database Engine

Database engine tích hợp của Scout thực hiện các tìm kiếm full-text và `LIKE` trên cơ sở dữ liệu hiện tại của bạn — không cần dịch vụ bên ngoài hay cơ sở hạ tầng thêm. Chỉ cần thêm trait `Searchable` vào model và định nghĩa phương thức `toSearchableArray` trả về các cột mà bạn muốn tìm kiếm.

Bạn có thể dùng PHP attributes để kiểm soát cách tìm kiếm cho từng cột. `SearchUsingFullText` sẽ dùng full-text index của cơ sở dữ liệu, `SearchUsingPrefix` chỉ lấy kết quả giống với từ đầu chuỗi (`example%`), và các cột không có attribute sẽ sử dụng cách `LIKE` mặc định với wildcard ở cả hai đầu (`%example%`):

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;
use Laravel\Scout\Searchable;

class Article extends Model
{
    use Searchable;

    #[SearchUsingPrefix(['id'])]
    #[SearchUsingFullText(['title', 'body'])]
    public function toSearchableArray(): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
        ];
    }
}
```

> [!WARNING]
> Trước khi chỉ định một cột nên dùng full-text query, hãy đảm bảo rằng cột đó đã có [full-text index](/docs/{{version}}/migrations#available-index-types).

Một khi đã thêm trait, bạn có thể tìm kiếm model bằng phương thức `search` của Scout. Database engine của Scout sẽ tự động sắp xếp kết quả theo độ liên quan, kể cả trên PostgreSQL:

```php
$articles = Article::search('Laravel')->get();
```

Database engine là lựa chọn tuyệt vời khi nhu cầu tìm kiếm của bạn ở mức vừa phải và bạn muốn Scout tự động đồng bộ index mà không cần triển khai dịch vụ bên ngoài. Nó xử lý tốt các trường hợp tìm kiếm phổ biến nhất, bao gồm lọc, phân trang và xử lý các bản ghi soft-deleted. Xem chi tiết trong [tài liệu Scout](/docs/{{version}}/scout#database-engine).

<a name="third-party-engines"></a>
### Third-Party Engines

Scout cũng hỗ trợ các search engine bên thứ ba như [Algolia](https://www.algolia.com/), [Meilisearch](https://www.meilisearch.com) và [Typesense](https://typesense.org). Các dịch vụ tìm kiếm chuyên dụng này cung cấp các tính năng nâng cao như xử lý lỗi chính tả, lọc đa chiều, tìm kiếm theo vị trí địa lý và các quy tắc sắp xếp tùy chỉnh — các tính năng quan trọng ở quy mô rất lớn hoặc khi bạn cần trải nghiệm tìm kiếm-theo-từng-chữ cực kì mượt mà.

Vì Scout cung cấp API thống nhất trên tất cả các driver, việc chuyển từ database engine sang một engine bên thứ ba sau này chỉ cần thay đổi code rất ít. Bạn có thể bắt đầu với database engine và chỉ cần chuyển sang dịch vụ bên thứ ba nếu như nhu cầu ứng dụng của bạn vượt quá khả năng của cơ sở dữ liệu.

Xem thêm chi tiết về cấu hình các engine bên thứ ba trong [tài liệu Scout](/docs/{{version}}/scout).

> [!NOTE]
> Nhiều ứng dụng không bao giờ cần đến search engine bên ngoài. Các kỹ thuật tích hợp sẵn được mô tả trong trang này đã đủ cho đa số trường hợp sử dụng.

<a name="combining-techniques"></a>
## Combining Techniques

Các kỹ thuật tìm kiếm được mô tả trong trang này không loại trừ lẫn nhau — kết hợp chúng thường mang lại kết quả tốt nhất. Dưới đây là hai trường hợp phổ biến cho thấy cách các công cụ này hoạt động với nhau.

**Full-Text Retrieval + Reranking**

Dùng tìm kiếm full-text search để nhanh chóng thu hẹp tập dữ liệu lớn xuống một tập dữ liệu nhỏ, rồi áp dụng reranking để sắp xếp các kết quả theo độ liên quan ngữ nghĩa. Cách này cho bạn tốc độ của tìm kiếm full-text search với độ chính xác do AI hỗ trợ:

```php
$articles = Article::query()
    ->whereFullText('body', $request->input('query'))
    ->limit(50)
    ->get()
    ->rerank('body', $request->input('query'), limit: 10);
```

**Vector Search + Traditional Filters**

Kết hợp giữa vector search với các mệnh đề `where` thông thường để giới hạn tìm kiếm ngữ nghĩa trên một tập hợp các bản ghi. Điều này hữu ích khi bạn muốn tìm kiếm theo ngữ nghĩa nhưng cần lọc kết quả theo quyền sở hữu, danh mục, hoặc bất kỳ thuộc tính nào khác:

```php
$documents = Document::query()
    ->where('team_id', $user->team_id)
    ->whereVectorSimilarTo('embedding', $request->input('query'))
    ->limit(10)
    ->get();
```
