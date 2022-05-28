# Validation

- [Giới thiệu](#introduction)
- [Validation Quickstart](#validation-quickstart)
    - [Định nghĩa Routes](#quick-defining-the-routes)
    - [Tạo Controller](#quick-creating-the-controller)
    - [Viết Validation Logic](#quick-writing-the-validation-logic)
    - [Hiển thị Validation Errors](#quick-displaying-the-validation-errors)
    - [A Note On Optional Fields](#a-note-on-optional-fields)
- [Form Request Validation](#form-request-validation)
    - [Tạo Form Requests](#creating-form-requests)
    - [Authorizing Form Requests](#authorizing-form-requests)
    - [Tuỳ biến Error Messages](#customizing-the-error-messages)
    - [Tuỳ biến thuộc tính Validation](#customizing-the-validation-attributes)
    - [Chuẩn bị dữ liệu cho Validation](#prepare-input-for-validation)
- [Tạo Validator thủ công](#manually-creating-validators)
    - [Tự dộng chuyển hướng](#automatic-redirection)
    - [Tên của Error Bags](#named-error-bags)
    - [After Validation Hook](#after-validation-hook)
- [Làm việc với Error Messages](#working-with-error-messages)
    - [Tuỳ chỉnh Error Messages](#custom-error-messages)
- [Các Validation Rule có sẵn](#available-validation-rules)
- [Thêm điều kiện cho Rule](#conditionally-adding-rules)
- [Validating mảng](#validating-arrays)
- [Tuỳ biến Validation Rules](#custom-validation-rules)
    - [Dùng đối tượng Rule](#using-rule-objects)
    - [Using Closures](#using-closures)
    - [Dùng Extensions](#using-extensions)
    - [Extension ẩn](#implicit-extensions)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một số cách tiếp cận khác nhau để validate dữ liệu trong application của bạn. Mặc định, class controller base của Laravel đã sử dụng một trait `ValidatesRequests` để cung các một phương thức cho validate request HTTP với nhiều rule validate mạnh mẽ.

<a name="validation-quickstart"></a>
## Validation Quickstart

Để tìm hiểu về các tính năng validation của Laravel, chúng ta hãy xem một ví dụ về validation cho một form và cách hiển thị thông báo lỗi cho người dùng.

<a name="quick-defining-the-routes"></a>
### Định nghĩa Routes

Đầu tiên, giả sử chúng ta có các route sau đã được định nghĩa trong file `routes/web.php`:

    Route::get('post/create', 'PostController@create');

    Route::post('post', 'PostController@store');

Route `GET` sẽ hiển thị một form cho người dùng để tạo một bài đăng mới trong blog, trong khi route `POST` sẽ lưu trữ bài đăng đó vào trong blog trong cơ sở dữ liệu.

<a name="quick-creating-the-controller"></a>
### Tạo Controller

Tiếp theo, chúng ta hãy xem một controller đơn giản xử lý cho các route. Bây giờ chúng ta sẽ bỏ trống phương thức `store`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Show the form to create a new blog post.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate and store the blog post...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### Viết Validation Logic

Bây giờ chúng ta đã sẵn sàng để viết vào phương thức `store` của chúng ta với các logic validate bài đăng trong blog. Để làm điều này, chúng ta sẽ sử dụng phương thức `validate` được cung cấp trong đối tượng `Illuminate\Http\Request`. Nếu pass qua validate rule, code của bạn sẽ được tiếp tục thực thi bình thường; tuy nhiên, nếu validate không thành công, một exception sẽ được đưa ra và một error response thích hợp sẽ tự động được gửi về cho người dùng. Trong trường hợp request HTTP bình thường, thì response sẽ là một chuyển hướng, còn nếu request là kiểu AJAX thì một response JSON sẽ được trả về.

Để hiểu rõ hơn về phương thức `validate`, chúng ta hãy quay lại phương thức` store`:

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid...
    }

Như bạn có thể thấy, chúng ta đã truyền các quy tắc validation mà chúng ta mong muốn vào phương thức `validate`. Một lần nữa, nếu validation thất bại, một response thích hợp sẽ được tự động trả về. Còn nếu validation thành công, controller của chúng ta sẽ tiếp tục được thực thi bình thường.

Ngoài ra, các quy tắc validation có thể được chỉ định dưới dạng các mảng quy tắc thay vì một chuỗi phân cách bằng dấu `|`:

    $validatedData = $request->validate([
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

Bạn có thể sử dụng phương thức `validateWithBag` để kiểm tra một request và lưu bất kỳ thông báo lỗi nào vào trong một [named error bag](#named-error-bags):

    $validatedData = $request->validateWithBag('post', [
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

#### Dừng luôn nếu Validation đầu tiên thất bại

Đôi khi bạn có thể muốn dừng chạy quy tắc validation trên một thuộc tính sau lần thất bại đầu tiên. Để làm như vậy, hãy gán quy tắc `bail` cho thuộc tính:

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

Trong ví dụ này, nếu quy tắc `unique` trong thuộc tính `title` thất bại, quy tắc `max` sẽ không được kiểm tra. Các quy tắc này sẽ được validate theo thứ tự mà chúng được định nghĩa.

#### Lưu ý về các thuộc tính lồng nhau

Nếu request HTTP của bạn chứa các tham số "lồng nhau", bạn có thể định nghĩa chúng trong quy tắc validation bằng cách sử dụng cú pháp "chấm":

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### Hiển thị Validation Errors

Vậy, điều gì sẽ xảy ra nếu các tham số request không pass qua các quy tắc validation đã cho? Như đã đề cập trước đó, Laravel sẽ tự động chuyển hướng người dùng trở lại vị trí trước đó của họ. Ngoài ra, tất cả các lỗi validation sẽ được tự động [flash vào trong session](/docs/{{version}}/session#flash-data).

Một lần nữa, hãy lưu ý rằng chúng ta không phải liên kết bất kỳ thông báo lỗi nào với view trong route `GET` của chúng ta. Điều này là do Laravel sẽ kiểm tra có lỗi trong session có hay không và tự động tạo liên kết chúng với view nếu chúng tồn tại. Biến `$errors` sẽ là một instance của `Illuminate\Support\MessageBag`. Để biết thêm thông tin về cách làm việc với đối tượng này, [xem tài liệu của nó](#working-with-error-messages).

> {tip} Biến `$errors` bị ràng buộc với view thông qua middleware `Illuminate\View\Middleware\ShareErrorsFromSession`, được cung cấp bởi group middleware `web`. **Khi middleware này được áp dụng, thì biến `$errors` này sẽ luôn có tồn tại trong view của bạn**, cho phép bạn giả sử biến `$errors` luôn được khai báo và có thể được sử dụng một cách an toàn hơn.

Vì vậy, trong ví dụ của chúng ta, người dùng sẽ được chuyển hướng về phương thức `create` của controller khi validation thất bại, cho phép chúng ta hiển thị các thông báo lỗi trong view như sau:

    <!-- /resources/views/post/create.blade.php -->

    <h1>Create Post</h1>

    @if ($errors->any())
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- Create Post Form -->

#### The `@error` Directive

Bạn cũng có thể sử dụng lệnh `@error` [Blade](/docs/{{version}}/blade) để kiểm tra xem trong thông báo lỗi validation có tồn tại cho một thuộc tính hay không. Trong lệnh `@error`, bạn có thể echo ra biến `$message` để hiển thị thông báo lỗi:

    <!-- /resources/views/post/create.blade.php -->

    <label for="title">Post Title</label>

    <input id="title" type="text" class="@error('title') is-invalid @enderror">

    @error('title')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

<a name="a-note-on-optional-fields"></a>
### Lưu ý về các field tùy chọn

Mặc định, Laravel sẽ chứa hai middleware là: `TrimStrings` và `ConvertEmptyStringsToNull` trong stack middleware global application. Các middleware này sẽ được liệt kê trong stack bởi class `App\Http\Kernel`. Vì thế, bạn sẽ cần phải đánh dấu các trường request "optional" của bạn là `nullable` nếu bạn không muốn validator coi các giá trị `null` của các trường này là không hợp lệ. Ví dụ:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

Trong ví dụ trên, chúng ta đang định nghĩa là trường `publish_at` có thể là `null` hoặc nếu có giá trị thì phải theo format của date. Nếu chúng ta không thêm `nullable` vào trong định nghĩa quy tắc này, thì validator sẽ coi `null` là một date không hợp lệ.

<a name="quick-ajax-requests-and-validation"></a>
#### AJAX Requests và Validation

Trong ví dụ trên, chúng ta đã sử dụng một form bình thường để gửi dữ liệu đến application. Tuy nhiên, nhiều application sẽ sử dụng các request là AJAX. Nên nếu sử dụng phương thức `validate` trong request là AJAX, thì Laravel sẽ không tạo ra response chuyển hướng. Thay vào đó, Laravel tạo ra một response JSON chứa tất cả các lỗi validation. Response JSON này sẽ được gửi về với HTTP status code là 422.

<a name="form-request-validation"></a>
## Form Request Validation

<a name="creating-form-requests"></a>
### Tạo Form Requests

Đối với các kịch bản validation phức tạp hơn, bạn có thể tạo một "form request". Form requests là các class request tùy biến có chứa logic validation. Để tạo một class form request, hãy sử dụng lệnh Artisan CLI `make:request`:

    php artisan make:request StoreBlogPost

Class được tạo ra sẽ được lưu trong thư mục `app/Http/Requests`. Nếu thư mục này không tồn tại, nó sẽ được tạo khi bạn chạy lệnh `make:request`. Chúng ta hãy thêm một vài quy tắc validation vào phương thức `rules`:

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

> {tip} Bạn có thể khai báo bất kỳ phụ thuộc nào mà bạn cần trong phương thức `rule`. Những phụ thuộc đó sẽ được tự động resolve thông qua Laravel [service container](/docs/{{version}}/container).

Vậy, các quy tắc validation sẽ được so sánh như thế nào? Tất cả những gì bạn cần làm là khai báo nó cho request trong phương thức controller của bạn. Form request đến sẽ được validate trước khi phương thức controller được gọi, nghĩa là bạn không cần làm lộn xộn controller của bạn với bất kỳ logic validate nào:

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...

        // Retrieve the validated input data...
        $validated = $request->validated();
    }

Nếu validation thất bại, một response chuyển hướng sẽ được tạo và đưa người dùng trở về vị trí trước đó của họ. Các lỗi cũng sẽ được flash vào session để chúng có thể được hiển thị. Nếu request là loại request AJAX, response HTTP có status code 422 sẽ được trả về cho người dùng chứa một data JSON gồm các lỗi validation.

#### Thêm After Hooks vào Form Requests

Nếu bạn muốn thêm một "after" hook vào một form request, bạn có thể sử dụng phương thức `withValidator`. Phương thức này nhận vào một validator đã được khởi tạo, cho phép bạn gọi bất kỳ phương thức nào trước khi các quy tắc validation thực sự được so sánh:

    /**
     * Configure the validator instance.
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }

<a name="authorizing-form-requests"></a>
### Authorizing Form Requests

Class form request cũng chứa một phương thức `authorize`. Trong phương thức này, bạn có thể kiểm tra xem người dùng hiện tại thực sự có quyền truy cập vào resource này hay không. Ví dụ: bạn có thể xác định xem người dùng có thực sự là chủ sở hữu của một bình luận trong blog mà họ đang cố cập nhật hay không:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

Vì tất cả các form request đều được mở rộng từ class request của Laravel, nên chúng ta có thể sử dụng phương thức `user` để truy cập vào người dùng hiện tại đang được authenticate. Hãy lưu ý cách gọi đến phương thức `route` trong ví dụ ở trên. Phương thức này cung cấp cho bạn quyền truy cập vào các tham số URI đã được định nghĩa trên route hiện tại, chẳng hạn như tham số `{comment}` trong ví dụ bên dưới:

    Route::post('comment/{comment}');

Nếu phương thức `authorize` trả về `false`,  HTTP response có status code là 403 sẽ được tự động trả về và phương thức trong controller của bạn sẽ không được thực thi.

Nếu bạn muốn logic authorization nằm ở trong một phần khác của application, bạn hãy trả về `true` từ phương thức `authorize`:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

> {tip} Bạn có thể khai báo bất kỳ phụ thuộc nào mà bạn cần trong phương thức `authorize`. Những phụ thuộc đó sẽ được tự động resolve thông qua Laravel [service container](/docs/{{version}}/container).

<a name="customizing-the-error-messages"></a>
### Tuỳ biến Error Messages

Bạn có thể tùy biến các thông báo lỗi được sử dụng bởi form request bằng cách ghi đè phương thức `messages`. Phương thức này sẽ trả về một mảng gồm các cặp thuộc tính / quy tắc và các thông báo lỗi tương ứng của chúng:

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required' => 'A message is required',
        ];
    }

<a name="customizing-the-validation-attributes"></a>
### Tuỳ biến thuộc tính Validation

Nếu bạn muốn phần `:attribute` của message validation được thay thế bằng tên một thuộc tính tùy chỉnh, bạn có thể chỉ định các tên tùy chỉnh đó bằng cách ghi đè phương thức `attributes`. Phương thức này sẽ trả về một mảng gồm thuộc tính và tên:

    /**
     * Get custom attributes for validator errors.
     *
     * @return array
     */
    public function attributes()
    {
        return [
            'email' => 'email address',
        ];
    }

<a name="prepare-input-for-validation"></a>
### Chuẩn bị dữ liệu cho Validation

Nếu bạn cần làm sạch dữ liệu trong request trước khi áp dụng các quy tắc validation của bạn, bạn có thể sử dụng phương thức `prepareForValidation`:

    use Illuminate\Support\Str;

    /**
     * Prepare the data for validation.
     *
     * @return void
     */
    protected function prepareForValidation()
    {
        $this->merge([
            'slug' => Str::slug($this->slug),
        ]);
    }

<a name="manually-creating-validators"></a>
## Tạo Validator thủ công

Nếu bạn không muốn sử dụng phương thức `validate` theo request, bạn có thể tự tạo một instance validator bằng cách sử dụng [facade](/docs/{{version}}/facades) `Validator`. Phương thức `make` trên facade sẽ tạo ra một instance validator mới:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Validator;

    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Store the blog post...
        }
    }

Tham số đầu tiên được truyền cho phương thức `make` là dữ liệu cần được validation. Tham số thứ hai là các quy tắc validation sẽ được áp dụng cho dữ liệu đó.

Sau khi kiểm tra nếu request validation thất bại, bạn có thể sử dụng phương thức `withErrors` để flash các thông báo lỗi vào session. Khi sử dụng phương thức này, biến `$errors` sẽ được tự động chia sẻ với các view của bạn sau khi được chuyển hướng tới, cho phép bạn dễ dàng hiển thị thông báo lỗi cho người dùng. Phương thức `withErrors` chấp nhận một validator và một `MessageBag` hoặc một PHP `array`.

<a name="automatic-redirection"></a>
### Tự dộng chuyển hướng

Nếu bạn muốn tự tạo một validator instance nhưng vẫn muốn tận dụng tính năng chuyển hướng tự động được cung cấp bởi phương thức `validate` của request, bạn có thể gọi phương thức `validate` trên một validator instance đã tồn tại. Nếu validation thất bại, người dùng sẽ tự động được chuyển hướng hoặc trong trường hợp request là AJAX, thì response JSON sẽ được trả về:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

Bạn có thể sử dụng phương thức `validateWithBag` để lưu thông báo lỗi vào trong một [named error bag](#named-error-bags) nếu quá trình kiểm tra không thành công:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validateWithBag('post');

<a name="named-error-bags"></a>
### Tên của Error Bags

Nếu bạn có nhiều form trong một trang, bạn có thể muốn đặt tên cho `MessageBag`, để bạn có thể truy xuất vào các thông báo lỗi cho một form cụ thể. Hãy truyền tên đó làm tham số thứ hai cho phương thức `withErrors`:

    return redirect('register')
                ->withErrors($validator, 'login');

Sau đó, bạn có thể truy cập vào instance `MessageBag` đã được đặt tên từ biến `$errors`:

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>
### After Validation Hook

Validator cũng cho phép bạn gắn các callback sẽ được chạy sau khi validation hoàn tất. Điều này cho phép bạn dễ dàng thực hiện validation thêm hoặc thậm chí là thêm nhiều thông báo lỗi vào message collection. Để bắt đầu, hãy sử dụng phương thức `after` trên một instance validator:

    $validator = Validator::make(...);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-error-messages"></a>
## Làm việc với Error Messages

Sau khi gọi phương thức `errors` trong một instance `Validator`, bạn sẽ nhận về một instance `Illuminate\Support\MessageBag`, có nhiều phương thức để làm việc với các thông báo lỗi. Biến `$errors` mà được tự động cung cấp cho các view cũng là một instance của class `MessageBag`.

#### Lấy lỗi đầu tiên của một field

Để lấy thông báo lỗi đầu tiên cho một field, hãy sử dụng phương thức `first`:

    $errors = $validator->errors();

    echo $errors->first('email');

#### Lấy tất cả các lỗi của một field

Nếu bạn cần lấy tất cả các thông báo lỗi cho một field, hãy sử dụng phương thức `get`:

    foreach ($errors->get('email') as $message) {
        //
    }

Nếu bạn đang validate một mảng field, bạn có thể lấy tất cả các thông báo lỗi cho từng field trong mảng bằng ký tự `*`:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

#### Lấy tất cả các lỗi của tất cả các field

Để lấy một mảng tất cả các thông báo lỗi cho tất cả các field, hãy sử dụng phương thức `all`:

    foreach ($errors->all() as $message) {
        //
    }

#### Xác định một thông báo có tồn tại của một field

Phương thức `has` có thể được sử dụng để xác định xem có tồn tại thông báo lỗi nào cho field đã cho không:

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>
### Tuỳ chỉnh Error Messages

Nếu cần, bạn có thể tùy biến thông báo lỗi cho validation thay vì mặc định. Có một số cách để định nghĩa tùy biến một thông báo lỗi. Đầu tiên, bạn có thể truyền các thông báo lỗi đã được tùy biến làm tham số thứ ba cho phương thức `Validator::make`:

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

Trong ví dụ này, `:attribute` sẽ được thay thế bằng tên thực sự của field mà được validation. Bạn cũng có thể sử dụng các attribute khác trong validation messages. Ví dụ:

    $messages = [
        'same' => 'The :attribute and :other must match.',
        'size' => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in' => 'The :attribute must be one of the following types: :values',
    ];

#### Chỉ định một Custom Message cho một attribute nhất định

Thỉnh thoảng bạn có thể chỉ định một thông báo lỗi tùy biến chỉ cho một field cụ thể. Bạn có thể làm như vậy bằng cách dùng ký hiệu "chấm". Chỉ định tên của attribute trước và sau đó là đến tên của quy tắc:

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### Chỉ định Custom Messages trong file Language

Trong hầu hết các trường hợp, bạn có thể sẽ cần chỉ định các thông báo lỗi tùy biến của bạn vào trong một file language thay vì truyền chúng trực tiếp vào `Validator`. Để làm như vậy, hãy thêm các thông báo lỗi của bạn vào mảng `custom` trong file language `resources/lang/xx/validation.php`.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

#### Chỉ định Custom giá trị Attributes

Nếu bạn muốn phần `:attribute` trong thông báo validation của bạn được thay thế bằng một tên attribute tùy biến, bạn có thể chỉ định tên tùy biến này trong mảng `attributes` của file language `resources/lang/xx/validation.php`:

    'attributes' => [
        'email' => 'email address',
    ],

Bạn cũng có thể truyền các thuộc tính tùy chỉnh làm tham số thứ tư cho phương thức `Validator::make`:

    $customAttributes = [
        'email' => 'email address',
    ];

    $validator = Validator::make($input, $rules, $messages, $customAttributes);

#### Chỉ định Custom Values trong file Language

Thỉnh thoảng bạn có thể cần thay thế phần `:value` trong message validation của bạn bằng một giá trị tuỳ biến. Ví dụ: hãy xem xét rule sau, nó sẽ quy đinh rằng số thẻ tín dụng là bắt buộc nếu `payment_type` có giá trị là `cc`:

    $request->validate([
        'credit_card_number' => 'required_if:payment_type,cc'
    ]);

Nếu rule validation này không thành công, thì nó sẽ tạo ra một thông báo lỗi như sau:

    The credit card number field is required when payment type is cc.

Thay vì hiển thị `cc` làm giá trị của payment type, bạn có thể chỉ định giá trị tùy biến trong file ngôn ngữ `validation` của bạn bằng cách định nghĩa mảng `values`:

    'values' => [
        'payment_type' => [
            'cc' => 'credit card'
        ],
    ],

Bây giờ, nếu rule validation không thành công, thì nó sẽ tạo ra thông báo lỗi như sau:

    The credit card number field is required when payment type is credit card.

<a name="available-validation-rules"></a>
## Các Validation Rule có sẵn

Dưới đây là danh sách tất cả các quy tắc validation có sẵn và chức năng của chúng:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[After Or Equal (Date)](#rule-after-or-equal)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Bail](#rule-bail)
[Before (Date)](#rule-before)
[Before Or Equal (Date)](#rule-before-or-equal)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[Email](#rule-email)
[Ends With](#rule-ends-with)
[Exclude If](#rule-exclude-if)
[Exclude Unless](#rule-exclude-unless)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Greater Than](#rule-gt)
[Greater Than Or Equal](#rule-gte)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Less Than](#rule-lt)
[Less Than Or Equal](#rule-lte)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Not In](#rule-not-in)
[Not Regex](#rule-not-regex)
[Nullable](#rule-nullable)
[Numeric](#rule-numeric)
[Password](#rule-password)
[Present](#rule-present)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[Sometimes](#conditionally-adding-rules)
[Starts With](#rule-starts-with)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)
[UUID](#rule-uuid)

</div>

<a name="rule-accepted"></a>
#### accepted

Field được validation phải là _yes_, _on_, _1_ hoặc _true_. Điều này hữu ích để validation chấp nhận "Điều khoản dịch vụ".

<a name="rule-active-url"></a>
#### active_url

Field được validation phải có bản ghi A hoặc AAAA hợp lệ theo hàm PHP `dns_get_record`. Hostname của URL đã cung cấp sẽ được lấy bằng cách sử dụng hàm PHP `parse_url` trước khi được truyền đến `dns_get_record`.

<a name="rule-after"></a>
#### after:_date_

Field được validation phải là một giá trị sau một ngày nhất định. Tham số date sẽ được truyền vào hàm PHP `strtotime`:

    'start_date' => 'required|date|after:tomorrow'

Thay vì truyền một chuỗi date được chạy bởi hàm `strtotime`, bạn có thể chỉ định một field khác để so sánh với ngày:

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

Field được validation phải là một giá trị sau hoặc bằng ngày đã cho. Để biết thêm thông tin, hãy xem quy tắc [after](#rule-after).

<a name="rule-alpha"></a>
#### alpha

Field được validation phải hoàn toàn là các ký tự chữ cái.

<a name="rule-alpha-dash"></a>
#### alpha_dash

Field được validation có thể có các ký tự chữ cái và số, cũng như dấu gạch ngang và dấu gạch dưới.

<a name="rule-alpha-num"></a>
#### alpha_num

Field được validation phải hoàn toàn là các ký tự chữ cái và số.

<a name="rule-array"></a>
#### array

Field được validation phải là một PHP `array`.

<a name="rule-bail"></a>
#### bail

Dừng chạy các validation rule nếu lần validation đầu tiên không thành công.

<a name="rule-before"></a>
#### before:_date_

Field được validation là một giá trị trước ngày đã cho. Tham số date sẽ được truyền vào hàm `strtotime` của PHP. Ngoài ra, giống như quy tắc [`after`](#rule-after), tên của một field khác cũng có thể được cung cấp dưới dạng như một giá trị kiểu `date`.

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

Field được validation là một giá trị trước hoặc bằng với ngày đã cho. Tham số date sẽ được truyền vào hàm `strtotime` của PHP. Ngoài ra, giống như quy tắc [`after`](#rule-after), tên của một field khác cũng có thể được cung cấp dưới dạng như một giá trị kiểu `date`.

<a name="rule-between"></a>
#### between:_min_,_max_

Field được validation phải có kích thước ở giữa _min_ và _max_ đã cho. Chuỗi, số, mảng và file sẽ được so sánh theo cùng một quy tắc với quy tắc [`size`](#rule-size).

<a name="rule-boolean"></a>
#### boolean

Field được validation phải có thể được cast là boolean. Input được chấp nhận là `true`, `false`, `1`, `0`, `"1"`, và `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

Field được validation phải có field là `foo_confirmation`. Ví dụ: nếu field được validation là `password`, thì field `password_confirmation` phải có tồn tại trong input.

<a name="rule-date"></a>
#### date

Field được validation phải là một ngày hợp lệ và non-relative theo hàm PHP `strtotime`.

<a name="rule-date-equals"></a>
#### date_equals:_date_

Field được validation phải bằng ngày đã cho. Tham số date sẽ được truyền vào hàm `strtotime` của PHP.

<a name="rule-date-format"></a>
#### date_format:_format_

Field được validation phải khớp với _format_ đã cho. Bạn nên sử dụng **một trong hai** `date` hoặc `date_format` khi validate một field, không dùng cả hai. Quy tắc validation này hỗ trợ tất cả các định dạng mà được hỗ trợ bởi class [DateTime](https://www.php.net/manual/en/class.datetime.php) của PHP.

<a name="rule-different"></a>
#### different:_field_

Field được validation phải có giá trị khác với _field_.

<a name="rule-digits"></a>
#### digits:_value_

Field được validation phải là _numeric_ và phải có độ dài chính xác là _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

Field được validation phải là _numeric_ và phải có độ dài ở giữa _min_ và _max_ đã cho.

<a name="rule-dimensions"></a>
#### dimensions

File được validation là một image đáp ứng các điều kiện về kích thước hoặc các quy định được tạo bởi các tham số của quy tắc:

    'avatar' => 'dimensions:min_width=100,min_height=200'

Các điều kiện có thể được dùng là: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

Một điều kiện _ratio_ phải được biểu diễn dưới dạng chiều rộng chia cho chiều cao. Điều này có thể được quy định bằng một câu lệnh như `3/2` hoặc nếu float là `1.5`:

    'avatar' => 'dimensions:ratio=3/2'

Vì quy tắc này yêu cầu một số tham số, nên bạn có thể sử dụng phương thức `Rule::dimensions` để dễ dàng xây dựng các quy tắc:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

Khi làm việc với mảng, field được validation phải không được có bất kỳ giá trị trùng lặp nào.

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

Trường được validation phải ở định dạng một địa chỉ e-mail. Về cơ bản, quy tắc validation này sử dụng package [`egulias/email-validator`](https://github.com/egulias/EmailValidator) để validation. Mặc định, validation `RFCValidation` sẽ được áp dụng, nhưng bạn cũng có thể áp dụng các kiểu validation khác:

    'email' => 'email:rfc,dns'

Ví dụ trên sẽ áp dụng validation `RFCValidation` và `DNSCheckValidation`. Dưới đây là một danh sách đầy đủ gồm các kiểu validation mà bạn có thể áp dụng:

<div class="content-list" markdown="1">

- `rfc`: `RFCValidation`
- `strict`: `NoRFCWarningsValidation`
- `dns`: `DNSCheckValidation`
- `spoof`: `SpoofCheckValidation`
- `filter`: `FilterEmailValidation`

</div>

Mặc định validator `filter` sẽ sử dụng hàm `filter_var` của PHP, đi kèm với Laravel và là hành vi của phiên bản Laravel trước phiên bản 5.8. Validator `dns` và `spoof` sẽ yêu cầu extension `intl` của PHP.

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

Field được validation phải kết thúc bằng một trong các giá trị đã cho.

<a name="rule-exclude-if"></a>
#### exclude_if:_anotherfield_,_value_

Field được validation sẽ bị loại trừ khỏi dữ liệu request được trả về từ phương thức `validate` và `validated` nếu _một field khác_ có giá trị bằng giá trị _value_.

<a name="rule-exclude-unless"></a>
#### exclude_unless:_anotherfield_,_value_

Field được validation sẽ bị loại trừ khỏi dữ liệu request được trả về từ phương thức `validate` và `validated` trừ khi _một field khác_ có giá trị bằng giá trị _value_.

<a name="rule-exists"></a>
#### exists:_table_,_column_

Field được validation phải tồn tại trong một bảng cơ sở dữ liệu nhất định.

#### Cách sử dụng cơ bản của Exists Rule

    'state' => 'exists:states'

Nếu tùy chọn `column` không được chỉ định, thì tên field đó sẽ được sử dụng.

#### Tùy chỉnh tên cột

    'state' => 'exists:states,abbreviation'

Đôi khi, bạn có thể cần chỉ định một kết nối cơ sở dữ liệu cụ thể sẽ được sử dụng cho truy vấn `exists`. Bạn có thể thực hiện điều này bằng cách thêm tên kết nối vào tên bảng thông qua cú pháp "chấm":

    'email' => 'exists:connection.staff,email'

Thay vì chỉ định trực tiếp tên bảng, bạn có thể chỉ định tên model Eloquent sẽ được sử dụng để xác định tên bảng:

    'user_id' => 'exists:App\User,id'

Nếu bạn muốn tùy chỉnh truy vấn được thực thi theo quy tắc validation, bạn có thể sử dụng class `Rule` để dễ dàng định nghĩa các quy tắc. Trong ví dụ này, chúng ta cũng sẽ định nghĩa các quy tắc validation là một mảng thay vì sử dụng ký tự `|` để phân định chúng:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);

<a name="rule-file"></a>
#### file

Field được validation phải là một tệp được tải lên thành công.

<a name="rule-filled"></a>
#### filled

Field được validation phải không được trống khi nó có tồn tại.

<a name="rule-gt"></a>
#### gt:_field_

Field được validation phải lớn hơn _field_ đã cho. Hai field phải cùng loại. Các loại chuỗi, số, mảng và file sẽ được đánh giá bằng cách sử dụng các quy ước giống như quy ước của [`size`](#rule-size).

<a name="rule-gte"></a>
#### gte:_field_

Field được validation phải lớn hơn hoặc bằng _field_ đã cho. Hai field phải cùng loại. Các loại chuỗi, số, mảng và file sẽ được đánh giá bằng cách sử dụng các quy ước giống như quy ước của [`size`](#rule-size).

<a name="rule-image"></a>
#### image

Field được validation phải là một image (jpeg, png, bmp, gif, svg, hoặc webp)

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

Field được validation phải có trong danh sách các giá trị đã cho. Vì quy tắc này thường yêu cầu bạn phải `implode` một mảng, nên phương thức `Rule::in` có thể được sử dụng để dễ dàng xây dựng quy tắc:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_.*

Field được validation phải tồn tại trong các giá trị của _anotherfield_.

<a name="rule-integer"></a>
#### integer

Field được validation phải là một integer.

> {note} Quy tắc validation này không xác minh được input thuộc loại biến kiểu "số nguyên", mà chỉ xác minh được rằng input là một chuỗi hoặc là một giá trị số có chứa một số nguyên.

<a name="rule-ip"></a>
#### ip

Field được validation phải là một địa chỉ IP.

#### ipv4

Field được validation phải là một địa chỉ IPv4.

#### ipv6

Field được validation phải là một địa chỉ IPv6.

<a name="rule-json"></a>
#### json

Field được validation phải là một chuỗi JSON.

<a name="rule-lt"></a>
#### lt:_field_

Field được validation phải nhỏ hơn _field_ đã cho. Hai field phải cùng loại. Các loại chuỗi, số, mảng và file sẽ được đánh giá bằng cách sử dụng các quy ước giống như quy ước của [`size`](#rule-size).

<a name="rule-lte"></a>
#### lte:_field_

Field được validation phải nhỏ hơn hoặc bằng _field_ đã cho. Hai field phải cùng loại. Các loại chuỗi, số, mảng và file sẽ được đánh giá bằng cách sử dụng các quy ước giống như quy ước của [`size`](#rule-size).

<a name="rule-max"></a>
#### max:_value_

Field được validation phải nhỏ hơn hoặc bằng maximum của _value_. Chuỗi, số, mảng và file sẽ được so sánh theo cùng một quy tắc với quy tắc [`size`](#rule-size).

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

File được validation phải khớp với một trong các loại MIME đã cho:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

Để xác định loại MIME của file được tải lên, nội dung của file sẽ được đọc và framework sẽ cố gắng đoán loại MIME, nó có thể khác với loại MIME do khách hàng cung cấp.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

File được validation phải có loại MIME tương ứng với một trong các extension đã được liệt kê.

#### Cách dùng của MIME Rule

    'photo' => 'mimes:jpeg,bmp,png'

Mặc dù bạn chỉ cần định nghĩa extension của file, nhưng thực ra quy tắc này sẽ validate loại MIME của file bằng cách đọc nội dung của file đó và đoán loại MIME của nó.

Một danh sách đầy đủ các loại MIME và các extension tương ứng của chúng có thể được tìm thấy tại vị trí sau: [https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

Field được validation phải có _value_ tối thiểu. Chuỗi, số, mảng và file sẽ được so sánh theo cùng một quy tắc với quy tắc [`size`](#rule-size).

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

Field được validation không được chứa trong một danh sách giá trị đã cho. Phương thức `Rule::notIn` có thể được sử dụng để dễ dàng xây dựng quy tắc:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

Field được validation phải không được khớp với biểu thức chính quy đã cho.

Quy tắc này sử dụng hàm `preg_match` trong PHP. Biểu thức được chỉ định phải tuân theo một định dạng được yêu cầu bởi `preg_match` và do đó, nó cũng chứa các dấu phân cách. Ví dụ: `'email' => 'not_regex:/^.+$/i'`.

**Note:** Khi sử dụng mẫu `regex` hoặc `not_regex`, có thể cần phải khai báo các quy tắc trong một mảng thay vì sử dụng các dấu "|" để phân cách, đặc biệt nếu biểu thức chính quy của bạn chứa ký tự đó.

<a name="rule-nullable"></a>
#### nullable

Field được validation có thể là `null`. Điều này đặc biệt hữu ích khi validation loại dữ liệu nguyên thủy chẳng hạn như chuỗi hoặc số nguyên có thể chứa giá trị `null`.

<a name="rule-numeric"></a>
#### numeric

Field được validation phải là numeric.

<a name="rule-password"></a>
#### password

Field được validation phải khớp với mật khẩu của người dùng đã xác thực. Bạn có thể chỉ định một guard authentication bằng cách sử dụng tham số đầu tiên của quy tắc:

    'password' => 'password:api'

<a name="rule-present"></a>
#### present

Field được validation phải có tồn tại trong dữ liệu input nhưng có thể trống.

<a name="rule-regex"></a>
#### regex:_pattern_

Field được validation phải phù hợp với biểu thức chính quy định.

Quy tắc này sử dụng hàm `preg_match` trong PHP. Biểu thức được chỉ định phải tuân theo một định dạng được yêu cầu bởi `preg_match` và do đó, nó cũng chứa các dấu phân cách. Ví dụ: `'email' => 'regex:/^.+@.+$/i'`.

**Note:** Khi sử dụng quy tắc `regex` hoặc `not_regex`, có thể bạn cần phải khai báo các quy tắc đó vào trong một mảng thay vì sử dụng các dấu "|" để phân cách, đặc biệt nếu biểu thức chính quy của bạn có chứa ký tự đó.

<a name="rule-required"></a>
#### required

Field được validation phải có tồn tại trong dữ liệu input và không được trống. Một field được coi là "trống" nếu một trong các điều kiện sau là đúng:

<div class="content-list" markdown="1">

- Giá trị là `null`.
- Giá trị là chuỗi trống.
- Giá trị là một mảng rỗng hoặc có `Countable` của object là trống.
- Giá trị là một file được upload nhưng không có path.

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

Field được validation phải có tồn tại và không được trống nếu trường _anotherfield_ bằng với giá trị _value_.

Nếu bạn muốn tạo một điều kiện phức tạp hơn cho quy tắc `required_if`, thì bạn có thể sử dụng phương thức `Rule::requiredIf`. Phương thức này chấp nhận một boolean hoặc một Closure. Khi bạn truyền vào một Closure, thì Closure này sẽ trả về một giá trị `true` hoặc `false` để xem field đang được validation có bắt buộc hay không:

    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf(function () use ($request) {
            return $request->user()->is_admin;
        }),
    ]);

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

Field được validation phải có tồn tại và không được trống khi trường _anotherfield_ không bằng với giá trị _value_.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

Field được validation phải có tồn tại và không được trống _chỉ khi_ một trong các field khác được khai báo có tồn tại.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Field được validation phải có tồn tại và không được trống _chỉ khi_ tất cả các field khác được khai báo đều có tồn tại.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

Field được validation phải có tồn tại và không được trống _chỉ khi_ một trong các field khác được khai báo không tồn tại.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Field được validation phải có tồn tại và không được trống _chỉ khi_ tất cả các field khác được khai báo đều không tồn tại.

<a name="rule-same"></a>
#### same:_field_

_field_ đã cho phải match với field được validation.

<a name="rule-size"></a>
#### size:_value_

Field được validation phải có kích thước khớp với _value_ đã cho. Đối với dữ liệu chuỗi, _value_ tương ứng với số lượng ký tự. Đối với dữ liệu số, _value_ tương ứng với một giá trị số nguyên đã cho (thuộc tính cũng phải có quy tắc `numeric` hoặc `integer`). Đối với một mảng, _size_ tương ứng với `count` của mảng. Đối với file, _size_ tương ứng với kích thước file tính bằng kilobyte. Hãy xem một số ví dụ:

    // Validate that a string is exactly 12 characters long...
    'title' => 'size:12';

    // Validate that a provided integer equals 10...
    'seats' => 'integer|size:10';

    // Validate that an array has exactly 5 elements...
    'tags' => 'array|size:5';

    // Validate that an uploaded file is exactly 512 kilobytes...
    'image' => 'file|size:512';

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

Field được validation phải bắt đầu bằng một trong các giá trị đã cho.

<a name="rule-string"></a>
#### string

Field được validation phải là một chuỗi. Nếu bạn muốn cho phép field được `null`, thì bạn nên gán quy tắc `nullable` cho field.

<a name="rule-timezone"></a>
#### timezone

Field được validation phải là một định danh múi giờ hợp lệ theo hàm PHP `timezone_identifiers_list`.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

Field được validation phải không tồn tại trong một bảng cơ sở dữ liệu.

**Khai báo tên table và tên cột**

Thay vì chỉ định trực tiếp tên bảng, bạn có thể chỉ định tên model Eloquent sẽ được sử dụng để xác định tên bảng:

    'email' => 'unique:App\User,email_address'

Tùy chọn `column` có thể được sử dụng để chỉ định tên cột sẽ được sử dụng trong cơ sở dữ liệu. Nếu tùy chọn `column` không được chỉ định, thì tên field sẽ được sử dụng.

    'email' => 'unique:users,email_address'

**Khai báo database connection**

Đôi khi, bạn có thể cần cài đặt một custom connection cho các truy vấn cơ sở dữ liệu được tạo bởi Validator. Như đã thấy ở trên, việc đặt `unique:users` làm quy tắc validate sẽ sử dụng kết nối cơ sở dữ liệu mặc định để truy vấn cơ sở dữ liệu. Để ghi đè lên điều này, bạn cần khai báo thêm thông tin kết nối và tên bảng bằng cú pháp "chấm":

    'email' => 'unique:connection.users,email_address'

**Bỏ qua một ID nhất định:**

Đôi khi, bạn có thể muốn bỏ qua một ID nhất định trong khi kiểm tra unique. Ví dụ: hãy xem thử màn hình "update profile" bao gồm tên người dùng, địa chỉ email và vị trí. Bạn có thể sẽ muốn kiểm tra rằng địa chỉ email có là unique hay không. Tuy nhiên, nếu người dùng chỉ thay đổi field tên chứ không phải field e-mail, bạn không thể tạo ra lỗi validate vì người dùng đã là chủ sở hữu của địa chỉ email đó.

Để hướng dẫn validator bỏ qua ID của người dùng, chúng ta sẽ sử dụng class `Rule` để dễ dàng khai báo quy tắc. Trong ví dụ này, chúng ta cũng sẽ khai báo các quy tắc validation là một mảng thay vì sử dụng ký tự `|` để phân chia các quy tắc:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

> {note} Bạn đừng bao giờ truyền bất kỳ input nào do người dùng kiểm soát vào trong phương thức `ignore`. Thay vào đó, bạn chỉ nên truyền một ID duy nhất do hệ thống tạo ra, chẳng hạn như ID hoặc UUID tăng tự động từ một instance model Eloquent. Nếu không, ứng dụng của bạn sẽ dễ bị tấn công bởi SQL injection.

Thay vì truyền giá trị khóa của model cho phương thức `ignore`, bạn có thể truyền toàn bộ instance của model đó cho phương thức. Và Laravel sẽ tự động trích xuất khóa của model đó:

    Rule::unique('users')->ignore($user)

Nếu bảng của bạn sử dụng tên cột khóa chính khác với `id`, bạn có thể chỉ định tên của cột khi gọi phương thức `ignore`:

    Rule::unique('users')->ignore($user->id, 'user_id')

Mặc định, quy tắc `unique` sẽ kiểm tra tính duy nhất của cột mà khớp với tên của thuộc tính đang được validation. Tuy nhiên, bạn có thể truyền tên một cột khác làm tham số thứ hai cho phương thức `unique`:

    Rule::unique('users', 'email_address')->ignore($user->id),

**Thêm điều kiện where:**

Bạn cũng có thể khai báo thêm các điều kiện truy vấn bằng cách sử dụng câu lệnh truy vấn dùng phương thức `where`. Ví dụ: hãy thêm một điều kiện là `account_id` phải là `1`:

    'email' => Rule::unique('users')->where(function ($query) {
        return $query->where('account_id', 1);
    })

<a name="rule-url"></a>
#### url

Field được validation phải là một URL hợp lệ.

<a name="rule-uuid"></a>
#### uuid

Field được validation phải là một mã định danh (UUID) RFC 4122 (phiên bản 1, 3, 4 hoặc 5).

<a name="conditionally-adding-rules"></a>
## Thêm điều kiện cho Rule

#### Skipping Validation When Fields Have Certain Values

Đôi khi bạn có thể muốn không kiểm tra một trường nhất định nếu một trường khác có giá trị đã cho. Bạn có thể thực hiện điều này bằng cách sử dụng quy tắc validation `exclude_if`. Trong ví dụ này, các trường `appointment_date` và `doctor_name` sẽ không bị kiểm tra nếu trường `has_appointment` có giá trị là `false`:

    $v = Validator::make($data, [
        'has_appointment' => 'required|bool',
        'appointment_date' => 'exclude_if:has_appointment,false|required|date',
        'doctor_name' => 'exclude_if:has_appointment,false|required|string',
    ]);

Ngoài ra, bạn có thể sử dụng quy tắc `exclude_unless` để không kiểm tra một trường nhất định trừ khi một trường khác có một giá trị đã cho:

    $v = Validator::make($data, [
        'has_appointment' => 'required|bool',
        'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
        'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
    ]);

#### Validate khi tồn tại

Trong một số trường hợp, bạn có thể muốn chạy kiểm tra validation đối với một field **chỉ** khi field đó có trong mảng input. Để nhanh chóng thực hiện điều này, hãy thêm quy tắc `sometimes` vào danh sách quy tắc của bạn:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

Trong ví dụ trên, field `email` sẽ chỉ được validate nếu nó có trong mảng `$data`.

> {tip} Nếu bạn đang validate một field luôn luôn tồn tại nhưng có thể trống, hãy xem [ghi chú này trên các field tùy chọn](#a-note-on-optional-fields)

#### Validation các điều kiện phức tạp

Thỉnh thoảng bạn có thể muốn thêm các quy tắc validation dựa trên logic điều kiện phức tạp hơn. Ví dụ: bạn muốn bắt buộc nhập một field nếu một field khác có giá trị lớn hơn 100. Hoặc, bạn có thể cần hai field có giá trị mặc định, khi một field khác có tồn tại. Thêm các quy tắc validation này không phải là một điều khó. Đầu tiên, tạo một instance `Validator` với _static rules_ không bao giờ thay đổi:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Giả sử application web của chúng ta là dành cho người sưu tầm trò chơi. Nếu một nhà sưu tập trò chơi đăng ký với application của chúng ta và họ sở hữu hơn 100 trò chơi, chúng ta muốn họ giải thích lý do tại sao họ sở hữu nhiều trò chơi như vậy. Ví dụ, có thể họ điều hành một cửa hàng bán lại trò chơi, hoặc có thể họ chỉ thích thu thập. Để có thêm điều kiện cho các yêu cầu này, chúng ta có thể sử dụng phương thức `sometimes` trên instance `Validator`.

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

Tham số đầu tiên được truyền cho phương thức `sometimes` là tên của field mà chúng ta đang validate. Tham số thứ hai là các quy tắc mà chúng ta muốn thêm. Và nếu `Closure` được truyền làm tham số thứ ba trả về `true`, thì quy tắc mới được valdiate. Phương thức này làm cho nó dễ dàng để xây dựng các validate có điều kiện phức tạp. Bạn thậm chí có thể thêm các validate có điều kiện cho một số field cùng một lúc:

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} Tham số `$input` được truyền cho `Closure` của bạn sẽ là một instance của `Illuminate\Support\Fluent` và có thể được sử dụng để truy cập vào field hoặc input đầu vào của bạn.

<a name="validating-arrays"></a>
## Validating mảng

Validate một mảng được dựa trên các field từ một form input không phải là một vấn đề khó khăn. Bạn có thể sử dụng "ký hiệu chấm" để validate các thuộc tính có trong một mảng. Ví dụ: nếu request HTTP chứa một field `photos[profile]`, bạn có thể validate nó như sau:

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

Bạn cũng có thể validate từng phần tử trong một mảng. Ví dụ: để validate rằng mỗi e-mail có trong mảng input field đã cho là duy nhất, bạn có thể làm như sau:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Tương tự, bạn có thể sử dụng ký tự `*` khi định nghĩa các thông báo validation trong các file language của bạn, giúp dễ dàng sử dụng một thông báo validation duy nhất cho mảng dựa trên các field:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## Tuỳ biến Validation Rules

<a name="using-rule-objects"></a>
### Dùng đối tượng Rule

Laravel cung cấp một loạt các quy tắc validation hữu ích; tuy nhiên, bạn có thể muốn khai báo thêm một số quy tắc của riêng bạn. Một phương thức đăng ký custom validation rule là sử dụng các đối tượng rule. Để tạo một đối tượng rule mới, bạn có thể sử dụng lệnh Artisan `make:rule`. Hãy sử dụng lệnh này để tạo rule xác minh chuỗi là chữ hoa. Laravel sẽ tạo rule mới trong thư mục `app/Rules`:

    php artisan make:rule Uppercase

Khi rule đã được tạo, chúng ta đã sẵn sàng xác định hành vi của nó. Một đối tượng rule sẽ chứa hai phương thức: `passes` và `message`. Phương thức `passes` nhận vào giá trị và tên thuộc tính và sẽ trả về `true` hoặc `false` tùy thuộc vào giá trị thuộc tính có hợp lệ hay không. Còn phương thức `message` sẽ trả về một thông báo lỗi validation, nó sẽ được sử dụng khi validation thất bại:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class Uppercase implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }

        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }

Bạn có thể gọi helper `trans` từ phương thức `message` nếu bạn muốn trả về một thông báo lỗi từ các file language của bạn:

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

Khi rule đã được định nghĩa xong, bạn có thể gán nó vào một validator bằng cách truyền một instance của đối tượng rule này cùng với các quy tắc validation khác của bạn:

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', 'string', new Uppercase],
    ]);

<a name="using-closures"></a>
### Using Closures

Nếu bạn cần chức năng của một rule tùy chỉnh trong đúng một lần trong toàn bộ ứng dụng của bạn, bạn có thể sử dụng Closure thay vì một đối tượng rule. Closure nhận vào tên của thuộc tính, giá trị của thuộc tính đó và một callback `$fail` sẽ được gọi nếu quá trình validation không thành công:

    $validator = Validator::make($request->all(), [
        'title' => [
            'required',
            'max:255',
            function ($attribute, $value, $fail) {
                if ($value === 'foo') {
                    $fail($attribute.' is invalid.');
                }
            },
        ],
    ]);

<a name="using-extensions"></a>
### Dùng Extensions

Một cách khác để đăng ký một tùy biến rule validation là sử dụng phương thức `extend` trên [facade](/docs/{{version}}/facades) `Validator`. Hãy sử dụng phương thức này trong một [service provider](/docs/{{version}}/providers) để đăng ký một tùy biến rule validation:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }
    }

Một tuỳ biến validator Closure sẽ nhận vào bốn tham số: tên của `$attribute` đang được validate, `$value` của thuộc tính đó, một mảng `$parameters` được truyền đến rule và instance `Validator`.

Bạn cũng có thể truyền một class và phương thức của nó cho phương thức `extend` thay vì một Closure:

    Validator::extend('foo', 'FooValidator@validate');

#### Định nghĩa Error Message

Bạn cũng sẽ cần định nghĩa một thông báo lỗi cho quy tắc tùy biến của bạn. Bạn có thể làm như vậy bằng cách sử dụng một mảng thông báo tùy biến hoặc bằng cách thêm một mục vào trong file language validation. Thông báo này phải được đặt ở vị trí đầu tiên của mảng, không nằm trong mảng `custom`, mảng `custom` chỉ dành cho các thông báo lỗi của một thuộc tính cụ thể:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

Khi tạo một tùy biến quy tắc validation, đôi khi bạn có thể cần định nghĩa các thông báo lỗi một cách trực tiếp. Bạn có thể làm như vậy bằng cách tạo tùy biến Validator bằng `extend` như được mô tả ở trên, và sau đó thực hiện gọi đến phương thức `replacer` trên facade `Validator`. Bạn có thể làm điều này trong phương thức `boot` của một [service provider](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

<a name="implicit-extensions"></a>
### Extension ẩn

Mặc định, khi một thuộc tính được validate không tồn tại hoặc là một chuỗi trống, thì các quy tắc validation thông thường, và cả các extension tùy biến đều sẽ được không chạy. Ví dụ: quy tắc [`unique`](#rule-unique) sẽ không được chạy cho một chuỗi trống:

    $rules = ['name' => 'unique:users,name'];

    $input = ['name' => ''];

    Validator::make($input, $rules)->passes(); // true

Để một quy tắc chạy ngay cả khi một thuộc tính trống, quy tắc đó phải tưởng tượng rằng thuộc tính là bắt buộc. Để tạo một phần extension "ẩn" như vậy, hãy sử dụng phương thức `Validator::extendImplicit()`:

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} Một extension "ẩn" chỉ _tưởng tượng_ rằng thuộc tính đó là bắt buộc. Việc nó thực sự validate một thuộc tính bị thiếu hoặc bị trống hay không là tùy thuộc vào bạn.

#### Đối tượng quy tắc ẩn

Nếu bạn muốn một đối tượng quy tắc chạy khi một thuộc tính trống, bạn nên implement interface `Illuminate\Contracts\Validation\ImplicitRule`. Interface này phục vụ như là một "marker interface" cho validator; do đó, nó không chứa bất kỳ phương thức nào mà bạn cần phải implement.
