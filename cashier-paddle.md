# Laravel Cashier Paddle

- [Giới thiệu](#introduction)
- [Cập nhật Cashier](#upgrading-cashier)
- [Cài đặt](#installation)
    - [Paddle Sandbox](#paddle-sandbox)
    - [Database Migrations](#database-migrations)
- [Cấu hình](#configuration)
    - [Billable Model](#billable-model)
    - [API Keys](#api-keys)
    - [Paddle JS](#paddle-js)
    - [Cấu hình đơn vị tiền tệ](#currency-configuration)
    - [Ghi đè Model mặc định](#overriding-default-models)
- [Khái niệm cốt lõi](#core-concepts)
    - [Pay Links](#pay-links)
    - [Inline Checkout](#inline-checkout)
    - [User Identification](#user-identification)
- [Giá](#prices)
- [Customers](#customers)
    - [Customer mặc định](#customer-defaults)
- [Subscriptions](#subscriptions)
    - [Tạo Subscription](#creating-subscriptions)
    - [Kiểm tra trạng thái Subscription](#checking-subscription-status)
    - [Subscription tính phí một lần](#subscription-single-charges)
    - [Cập nhật thông tin thanh toán](#updating-payment-information)
    - [Thay đổi gói](#changing-plans)
    - [Subscription số lượng lớn](#subscription-quantity)
    - [Subscription modifier](#subscription-modifiers)
    - [Nhiều Subscriptions](#multiple-subscriptions)
    - [Tạm dừng Subscriptions](#pausing-subscriptions)
    - [Huỷ Subscriptions](#cancelling-subscriptions)
- [Subscription dành cho dùng thử](#subscription-trials)
    - [Khai báo trước phương thức thanh toán](#with-payment-method-up-front)
    - [Khai báo phương thức thanh toán sau](#without-payment-method-up-front)
- [Xử lý Paddle Webhooks](#handling-paddle-webhooks)
    - [Định nghĩa xử lý Webhook Event](#defining-webhook-event-handlers)
    - [Kiểm tra định dạng Webhook](#verifying-webhook-signatures)
- [Phí](#single-charges)
    - [Tính phí một lần](#simple-charge)
    - [Tính phí sản phẩm](#charging-products)
    - [Hoàn trả](#refunding-orders)
- [Biên lai](#receipts)
    - [Các khoản thanh toán quá khứ và tương lai](#past-and-upcoming-payments)
- [Xử lý các khoản thanh toán không thành công](#handling-failed-payments)
- [Testing](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) cung cấp một interface dễ hiểu, rõ ràng cho các dịch vụ thanh toán subscription trực tuyến như [Paddle's](https://paddle.com). Nó gần như đã xử lý tất cả các đoạn code mà bạn đang sợ viết mà có liên quan đến các phần thanh toán subscription. Ngoài quản lý subscription cơ bản, Cashier cũng có thể xử lý cả các phiếu giảm giá, chuyển đổi subscription, đăng ký "nhiều" subscription, thời hạn hủy bỏ và nhiều hơn thế.

Trong khi làm việc với Cashier, chúng tôi khuyên bạn cũng nên xem qua [hướng dẫn sử dụng](https://developer.paddle.com/guides) của Paddle và [tài liệu API](https://developer.paddle.com/api-reference).

<a name="upgrading-cashier"></a>
## Cập nhật Cashier

Khi nâng cấp lên phiên bản mới của Cashier, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md).

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt package Cashier cho Paddle bằng trình quản lý package Composer:

```shell
composer require laravel/cashier-paddle
```

> **Warning**
> Để đảm bảo Cashier xử lý đúng tất cả các event của Paddle, hãy nhớ [thiết lập xử lý webhook của Cashier](#handling-stripe-webhooks).

<a name="paddle-sandbox"></a>
### Paddle Sandbox

Trong quá trình phát triển local và staging, bạn nên [đăng ký một tài khoản Paddle Sandbox](https://developer.paddle.com/getting-started/sandbox). Tài khoản này sẽ cung cấp cho bạn một môi trường sandbox để thử nghiệm và phát triển các ứng dụng của bạn mà không cần thực hiện thanh toán. Bạn có thể sử dụng [test card numbers](https://developer.paddle.com/getting-started/sandbox#test-cards) của Paddle để mô phỏng các tình huống thanh toán khác nhau.

Khi sử dụng môi trường Paddle Sandbox, bạn nên set biến môi trường `PADDLE_SANDBOX` thành `true` trong file `.env` của ứng dụng:

```ini
PADDLE_SANDBOX=true
```

Sau khi hoàn thành việc phát triển ứng dụng của bạn, bạn có thể [đăng ký tài khoản nhà cung cấp Paddle](https://paddle.com). Trước khi ứng dụng của bạn được deploy lên production, Paddle sẽ cần phê duyệt domain ứng dụng của bạn.

<a name="database-migrations"></a>
### Database Migrations

Service provider Cashier sẽ đăng ký một thư mục database migration của nó, vì vậy hãy nhớ migrate cơ sở dữ liệu của bạn sau khi cài đặt xong package. Việc migrate Cashier sẽ tạo ra một bảng `customers` mới. Ngoài ra, một bảng `subscriptions` mới cũng sẽ được tạo để lưu trữ tất cả các subscription của khách hàng của bạn. Cuối cùng, một bảng `receipts` mới cũng sẽ được tạo để lưu tất cả thông tin biên lai của ứng dụng của bạn:

```shell
php artisan migrate
```

Nếu bạn cần ghi đè các migration đi kèm với Cashier, bạn có thể export chúng bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

Nếu bạn muốn ngăn việc migration của Cashier chạy, bạn có thể sử dụng phương thức `ignoreMigrations` được Cashier cung cấp. Thông thường, phương thức này nên được gọi trong phương thức `register` trong `AppServiceProvider` của bạn:

    use Laravel\Paddle\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::ignoreMigrations();
    }

<a name="configuration"></a>
## Cấu hình

<a name="billable-model"></a>
### Billable Model

Trước khi sử dụng Cashier, hãy thêm trait `Billable` vào định nghĩa model của bạn. Trait này sẽ cung cấp các phương thức khác nhau cho phép bạn thực hiện các tác vụ thanh toán phổ biến, chẳng hạn như tạo subscription, áp dụng phiếu giảm giá hoặc cập nhật thông tin phương thức thanh toán:

    use Laravel\Paddle\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Nếu bạn có các billable model không phải là các user, bạn cũng có thể thêm trait này vào các class đó:

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Paddle\Billable;

    class Team extends Model
    {
        use Billable;
    }

<a name="api-keys"></a>
### API Keys

Tiếp theo, bạn nên cấu hình key của Paddle trong file `.env` của application. Bạn có thể lấy key API Paddle của bạn từ bảng điều khiển của Paddle.

```ini
PADDLE_VENDOR_ID=your-paddle-vendor-id
PADDLE_VENDOR_AUTH_CODE=your-paddle-vendor-auth-code
PADDLE_PUBLIC_KEY="your-paddle-public-key"
PADDLE_SANDBOX=true
```

Biến môi trường `PADDLE_SANDBOX` phải được set thành `true` khi bạn đang sử dụng trong [môi trường Sandbox của Paddle](#paddle-sandbox). Biến `PADDLE_SANDBOX` phải được set thành `false` nếu bạn đang triển khai ứng dụng của bạn sang production và đang sử dụng môi trường nhà cung cấp trực tiếp của Paddle.

<a name="paddle-js"></a>
### Paddle JS

Paddle dựa vào thư viện JavaScript của riêng nó để khởi chạy giao diện thanh toán Paddle. Bạn có thể load thư viện JavaScript này bằng cách viết lệnh Blade `@paddleJS` ngay trước thẻ `</head>` của layout ứng dụng của bạn:

```blade
<head>
    ...

    @paddleJS
</head>
```

<a name="currency-configuration"></a>
### Cấu hình đơn vị tiền tệ

Đơn vị tiền mặc định của Cashier là Đô la Mỹ (USD). Bạn có thể thay đổi loại tiền mặc định này bằng cách định nghĩa biến môi trường `CASHIER_CURRENCY` trong file `.env` của ứng dụng của bạn:

```ini
CASHIER_CURRENCY=EUR
```

Ngoài việc cấu hình đơn vị tiền tệ của Cashier, bạn cũng có thể chỉ định ngôn ngữ được sử dụng khi định dạng tiền tệ để hiển thị trong hóa đơn. Cashier sử dụng [class `NumberFormatter` của PHP](https://www.php.net/manual/en/class.numberformatter.php) để set ngôn ngữ tiền tệ:

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> **Warning**
> Để sử dụng các ngôn ngữ khác, khác với ngôn ngữ `en`, hãy đảm bảo là extension của PHP `ext-intl` đã được cài đặt và cấu hình trên server của bạn.

<a name="overriding-default-models"></a>
### Ghi đè Model mặc định

Bạn có thể tự do extend các model mà Cashier sử dụng bên trong bằng cách định nghĩa model của riêng bạn và extend model Cashier tương ứng:

    use Laravel\Paddle\Subscription as CashierSubscription;

    class Subscription extends CashierSubscription
    {
        // ...
    }

Sau khi định nghĩa model của bạn, bạn có thể hướng dẫn Cashier sử dụng model tùy chỉnh của bạn thông qua class `Laravel\Paddle\Cashier`. Thông thường, bạn nên thông báo cho Cashier về các model tùy chỉnh của bạn trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

    use App\Models\Cashier\Receipt;
    use App\Models\Cashier\Subscription;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Cashier::useReceiptModel(Receipt::class);
        Cashier::useSubscriptionModel(Subscription::class);
    }

<a name="core-concepts"></a>
## Khái niệm cốt lõi

<a name="pay-links"></a>
### Pay Links

Paddle bị thiếu các API CRUD mở rộng để thực hiện các thay đổi trạng thái của subscription. Do đó, hầu hết các tương tác với Paddle đều được thực hiện thông qua [giao diện thanh toán](https://developer.paddle.com/guides/how-tos/checkout/paddle-checkout) của nó. Trước khi chúng ta có thể hiển thị giao diện thanh toán đó, chúng ta sẽ cần phải tạo một "link thanh toán" bằng cách sử dụng Cashier. Một "link thanh toán" sẽ thông báo cho checkout widget biết về thao tác thanh toán mà chúng ta muốn thực hiện:

    use App\Models\User;
    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $request->user()->newSubscription('default', $premium = 34567)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Cashier có chứa một [Blade component](/docs/{{version}}/blade#components) `paddle-button`. Chúng ta có thể truyền link URL thanh toán vào component này dưới dạng một "prop". Khi nhấp vào button này, giao diện thanh toán của Paddle sẽ được hiển thị:

```html
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Mặc định, điều này sẽ hiển thị một button có style Paddle cơ bản. Bạn có thể xóa tất cả style Paddle bằng cách thêm thuộc tính `data-theme="none"` vào component:

```html
<x-paddle-button :url="$payLink" class="px-8 py-4" data-theme="none">
    Subscribe
</x-paddle-button>
```

Giao diện thanh toán bằng Paddle là không đồng bộ. Sau khi người dùng tạo hoặc cập nhật một subscription trong giao diện, Paddle sẽ gửi webhook đến ứng dụng của bạn để bạn có thể cập nhật đúng trạng thái của subscription trong cơ sở dữ liệu. Do đó, điều quan trọng nhất là bạn phải [thiết lập webhook](#handling-paddle-webhooks) để tương thích với các thay đổi trạng thái từ Paddle.

Để biết thêm thông tin về link thanh toán, bạn có thể xem lại [tài liệu Paddle API về tạo link thanh toán](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink).

> **Warning**
> Sau khi thay đổi trạng thái đăng ký, độ trễ để nhận được webhook tương ứng thường rất nhỏ nhưng bạn nên tính đến điều này trong ứng dụng của bạn bằng cách cân nhắc rằng subscription của người dùng của bạn có thể không khả dụng ngay sau khi hoàn tất quy trình thanh toán.

<a name="manually-rendering-pay-links"></a>
#### Manually Rendering Pay Links

Bạn cũng có thể hiển thị một link thanh toán theo cách thủ công mà không cần phải sử dụng các component Blade có sẵn của Laravel. Để bắt đầu, hãy tạo URL một link thanh toán như minh họa trong các ví dụ trước:

    $payLink = $request->user()->newSubscription('default', $premium = 34567)
        ->returnTo(route('home'))
        ->create();

Tiếp theo, chỉ đơn giản là gán URL link thanh toán vào element `a` trong HTML của bạn:

    <a href="#!" class="ml-4 paddle_button" data-override="{{ $payLink }}">
        Paddle Checkout
    </a>

<a name="payments-requiring-additional-confirmation"></a>
#### Payments Requiring Additional Confirmation

Thỉnh thoảng cần phải xác minh thêm để xác nhận và xử lý thanh toán. Khi điều này xảy ra, Paddle sẽ hiển thị thêm màn hình xác nhận thanh toán. Màn hình xác nhận thanh toán do Paddle hoặc Cashier hiển thị có thể được điều chỉnh cho phù hợp với quy trình thanh toán của ngân hàng hoặc nhà phát hành thẻ cụ thể và có thể chứa thêm xác nhận thẻ, như một khoản phí nhỏ tạm thời, hoặc xác thực thiết bị riêng biệt hoặc các hình thức xác minh khác.

<a name="inline-checkout"></a>
### Inline Checkout

Nếu bạn không muốn sử dụng giao diện thanh toán theo kiểu "overlay" của Paddle, Paddle cũng cung cấp thêm một tùy chọn để hiển thị giao diện inline. Mặc dù cách tiếp cận này không cho phép bạn điều chỉnh bất kỳ trường HTML nào của thanh toán, nhưng nó cho phép bạn nhúng giao diện đó vào ứng dụng của bạn.

Để giúp bạn dễ dàng bắt đầu với tính năng thanh toán inline, Cashier đã chứa một Blade component `paddle-checkout`. Để bắt đầu, bạn nên [tạo link thanh toán](#pay-links) và truyền link thanh toán đó vào thuộc tính `override` của component:

```blade
<x-paddle-checkout :override="$payLink" class="w-full" />
```

Để điều chỉnh chiều cao của component inline checkout, bạn có thể truyền thuộc tính `height` cho Blade component:

```blade
<x-paddle-checkout :override="$payLink" class="w-full" height="500" />
```

<a name="inline-checkout-without-pay-links"></a>
#### Inline Checkout Without Pay Links

Ngoài ra, bạn có thể tùy chỉnh giao diện con bằng các tùy chọn thay vì sử dụng một link thanh toán:

```blade
@php
$options = [
    'product' => $productId,
    'title' => 'Product Title',
];
@endphp

<x-paddle-checkout :options="$options" class="w-full" />
```

Vui lòng tham khảo [hướng dẫn về thanh toán inline](https://developer.paddle.com/guides/how-tos/checkout/inline-checkout) của Paddle cũng như [tham khảo tham số](https://developer.paddle.com/reference/paddle-js/parameters) của nó để biết thêm thông tin chi tiết về các tùy chọn có sẵn.

> **Warning**
> Nếu bạn cũng muốn sử dụng tùy chọn `passthrough` khi chỉ định các tùy chọn tùy chỉnh, thì bạn nên cung cấp một mảng khóa và giá trị làm giá trị của nó. Cashier sẽ tự động xử lý việc chuyển đổi mảng đó thành chuỗi JSON. Ngoài ra, tùy chọn `customer_id` sẽ được dành riêng cho việc sử dụng bên trong Cashier .

<a name="manually-rendering-an-inline-checkout"></a>
#### Manually Rendering An Inline Checkout

Bạn cũng có thể hiển thị thanh toán inline theo cách thủ công mà không cần sử dụng các component Blade có sẵn của Laravel. Để bắt đầu, hãy tạo URL link thanh toán [như minh họa trong các ví dụ trước](#pay-links).

Tiếp theo, bạn có thể sử dụng Paddle.js để khởi tạo thanh toán. Để đơn giản hóa ví dụ này, chúng ta sẽ minh họa điều này bằng cách sử dụng [Alpine.js](https://github.com/alpinejs/alpine); tuy nhiên, bạn có thể tự do chuyển ví dụ này sang giao diện người dùng của riêng bạn:

```alpine
<div class="paddle-checkout" x-data="{}" x-init="
    Paddle.Checkout.open({
        override: {{ $payLink }},
        method: 'inline',
        frameTarget: 'paddle-checkout',
        frameInitialHeight: 366,
        frameStyle: 'width: 100%; background-color: transparent; border: none;'
    });
">
</div>
```

<a name="user-identification"></a>
### User Identification

Trái ngược với Stripe, người dùng Paddle là duy nhất trên tất cả Paddle, không phải duy nhất trên mỗi tài khoản Paddle. Do đó, API của Paddle hiện không cung cấp các phương thức cập nhật thông tin chi tiết của người dùng, chẳng hạn như địa chỉ email. Khi tạo link thanh toán, Paddle sẽ xác định người dùng bằng tham số `customer_email`. Khi tạo một subscription, Paddle sẽ cố gắng tìm email người dùng đã cung cấp với người dùng Paddle hiện có.

Đối với hành vi này, có một số điều quan trọng cần lưu ý khi sử dụng Cashier và Paddle. Trước tiên, bạn nên biết rằng mặc dù các subscription trong Cashier được gắn với cùng một người dùng trong ứng dụng của bạn, **nhưng những subscription đó cũng có thể được liên kết với những người dùng khác nhau trong hệ thống Paddle**. Thứ hai, mỗi subscription có một thông tin về phương thức thanh toán được kết nối riêng và cũng có thể có các địa chỉ email khác nhau trong hệ thống Paddle (tùy thuộc vào email nào đã được chỉ định cho người dùng khi subscription được tạo).

Do đó, khi hiển thị các subscription, bạn phải luôn thông báo cho người dùng biết rằng địa chỉ email hoặc thông tin phương thức thanh toán nào đang được kết nối tới subscription và điều này xảy ra trên từng subscription. Việc lấy thông tin này có thể được thực hiện bằng các phương thức đã được cung cấp trên model `Laravel\Paddle\Subscription`:

    $subscription = $user->subscription('default');

    $subscription->paddleEmail();
    $subscription->paymentMethod();
    $subscription->cardBrand();
    $subscription->cardLastFour();
    $subscription->cardExpirationDate();

Hiện không có cách nào để sửa địa chỉ email của người dùng thông qua Paddle API. Khi người dùng muốn cập nhật địa chỉ email của họ trong Paddle, cách duy nhất để họ thực hiện là liên hệ với bộ phận chăm sóc khách hàng của Paddle. Khi giao tiếp với Paddle, họ cần cung cấp giá trị `paddleEmail` của subscription để hỗ trợ Paddle cập nhật thông tin vào đúng người dùng.

<a name="prices"></a>
## Giá

Paddle cho phép bạn tùy chỉnh giá dựa trên mỗi đơn vị tiền tệ, về cơ bản cho phép bạn cấu hình các mức giá khác nhau cho từng quốc gia khác nhau. Cashier Paddle cũng cho phép bạn lấy ra tất cả các mức giá cho một sản phẩm nhất định bằng phương thức `productPrices`. Phương thức này chấp nhận một ID của sản phẩm mà bạn muốn lấy giá ra:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456]);

Đơn vị tiền tệ sẽ được xác định dựa trên địa chỉ IP của request; tuy nhiên, bạn có thể tùy chọn cung cấp một quốc gia cụ thể để lấy ra giá cho sản phẩm đó:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456], ['customer_country' => 'BE']);

Sau khi lấy ra giá, bạn có thể hiển thị chúng theo cách bạn muốn:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->gross() }}</li>
    @endforeach
</ul>
```

Bạn cũng có thể hiển thị giá thực (không bao gồm thuế) và hiển thị số tiền thuế một cách riêng biệt:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->net() }} (+ {{ $price->price()->tax() }} tax)</li>
    @endforeach
</ul>
```

Nếu bạn muốn lấy ra giá cho các gói subscription, bạn có thể hiển thị giá ban đầu và giá định kỳ của subscription một cách riêng biệt:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - Initial: {{ $price->initialPrice()->gross() }} - Recurring: {{ $price->recurringPrice()->gross() }}</li>
    @endforeach
</ul>
```

Để biết thêm thông tin, hãy [xem tài liệu API của Paddle về giá](https://developer.paddle.com/api-reference/checkout-api/prices/getprices).

<a name="prices-customers"></a>
#### Customers

Nếu người dùng đã là một customer và bạn muốn hiển thị giá áp dụng cho customer đó, bạn có thể làm như sau bằng cách lấy ra giá trực tiếp từ instance customer:

    use App\Models\User;

    $prices = User::find(1)->productPrices([123, 456]);

Trong nội bộ, Cashier sẽ sử dụng [phương thức paddleCountry`](#customer-defaults) của người dùng để lấy ra giá theo đơn vị tiền tệ của họ. Vì vậy, ví dụ: một người dùng sống ở Hoa Kỳ sẽ thấy giá bằng USD trong khi người dùng ở Bỉ sẽ thấy giá bằng EUR. Nếu không tìm thấy đơn vị tiền tệ phù hợp, đơn vị tiền tệ mặc định của sản phẩm sẽ được sử dụng. Bạn có thể tùy chỉnh tất cả các mức giá của sản phẩm hoặc gói subscription trong bảng điều khiển Paddle.

<a name="prices-coupons"></a>
#### Coupons

Bạn cũng có thể chọn hiển thị giá sau khi đã dùng phiếu giảm giá. Khi gọi phương thức `productPrices`, phiếu giảm giá có thể được truyền vào dưới dạng là một chuỗi string được phân tách bằng dấu phẩy:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456], [
        'coupons' => 'SUMMERSALE,20PERCENTOFF'
    ]);

Sau đó, hiển thị giá đã tính bằng phương thức `price`:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->gross() }}</li>
    @endforeach
</ul>
```

Bạn có thể hiển thị giá niêm yết ban đầu (không có phiếu giảm giá) bằng phương thức `listPrice`:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->listPrice()->gross() }}</li>
    @endforeach
</ul>
```

> **Warning**
> Khi sử dụng API price, Paddle chỉ cho phép áp dụng phiếu giảm giá cho các sản phẩm mua một lần và không cho phép sử dụng cho các gói subscription.

<a name="customers"></a>
## Customers

<a name="customer-defaults"></a>
### Customer mặc định

Cashier cho phép bạn định nghĩa một số trường mặc định hữu ích cho khách hàng của bạn khi tạo link thanh toán. Việc set các giá trị mặc định này cho phép bạn điền trước địa chỉ email, quốc gia và mã bưu điện của khách hàng để họ có thể chuyển ngay sang phần thanh toán của giao diện thanh toán. Bạn có thể set các giá trị mặc định này bằng cách ghi đè các phương thức sau trong billable model của bạn:

    /**
     * Get the customer's email address to associate with Paddle.
     *
     * @return string|null
     */
    public function paddleEmail()
    {
        return $this->email;
    }

    /**
     * Get the customer's country to associate with Paddle.
     *
     * This needs to be a 2 letter code. See the link below for supported countries.
     *
     * @return string|null
     * @link https://developer.paddle.com/reference/platform-parameters/supported-countries
     */
    public function paddleCountry()
    {
        //
    }

    /**
     * Get the customer's postal code to associate with Paddle.
     *
     * See the link below for countries which require this.
     *
     * @return string|null
     * @link https://developer.paddle.com/reference/platform-parameters/supported-countries#countries-requiring-postcode
     */
    public function paddlePostcode()
    {
        //
    }

Các giá trị mặc định này sẽ được sử dụng cho mọi hành động của Cashier tạo ra [link thanh toán](#pay-links).

<a name="subscriptions"></a>
## Subscriptions

<a name="creating-subscriptions"></a>
### Tạo Subscription

Để tạo một subscription, trước tiên hãy lấy ra một instance billable model trong database của bạn, thường là một instance của `App\Models\User`. Khi bạn đã lấy được instance của model, bạn có thể sử dụng phương thức `newSubscription` để tạo ra một link thanh toán subscription cho model của bạn:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $request->user()->newSubscription('default', $premium = 12345)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Tham số đầu tiên được truyền cho phương thức `newSubscription` phải là tên internal của subscription. Nếu ứng dụng của bạn chỉ cung cấp một loại subscription duy nhất, bạn có thể gọi nó là `default` hoặc `primary`. Tên subscription này chỉ dành cho việc sử dụng ứng dụng nội bộ và không nhằm mục đích hiển thị cho người dùng. Ngoài ra, nó cũng không được chứa các khoảng trắng và nó không được thay đổi sau khi tạo subscription. Tham số thứ hai được đưa vào trong phương thức `newSubscription` là gói cụ thể mà người dùng đang subscription. Giá trị này phải tương ứng với dentifier của gói trong Paddle. Phương thức `returnTo` chấp nhận một URL mà người dùng của bạn sẽ được chuyển hướng về sau khi họ đã hoàn tất thanh toán.

Phương thức `create` sẽ tạo ra một link thanh toán mà bạn có thể sử dụng để tạo ra nút thanh toán. Nút thanh toán có thể được tạo bằng cách sử dụng [Blade component](/docs/{{version}}/blade#components) `paddle-button` đã được đi kèm với Cashier Paddle:

```blade
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Sau khi người dùng hoàn tất quá trình thanh toán của họ, webhook `subscription_created` sẽ được gửi từ Paddle. Cashier sẽ nhận webhook này và thiết lập đăng ký cho khách hàng của bạn. Để đảm bảo ứng dụng của bạn nhận và xử lý được tất cả các webhook một đúng cách, hãy đảm bảo rằng bạn đã [thiết lập xử lý webhook](#handling-paddle-webhooks).

<a name="additional-details"></a>
#### Additional Details

Nếu bạn muốn chỉ định thêm về khách hàng hoặc chi tiết subscription, bạn có thể làm như sau bằng cách truyền chúng dưới dạng một mảng khóa và giá trị vào phương thức `create`. Để tìm hiểu thêm về các trường được Paddle hỗ trợ, hãy xem tài liệu của Paddle về [tạo link thanh toán](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink):

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->create([
            'vat_number' => $vatNumber,
        ]);

<a name="subscriptions-coupons"></a>
#### Coupons

Nếu bạn muốn áp dụng một phiếu giảm giá khi tạo subscription, bạn có thể sử dụng phương thức `withCoupon`:

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->withCoupon('code')
        ->create();

<a name="metadata"></a>
#### Metadata

Bạn cũng có thể truyền một mảng dữ liệu bằng phương thức `withMetadata`:

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->withMetadata(['key' => 'value'])
        ->create();

> **Warning**
> Khi cung cấp mảng dữ liệu này, vui lòng tránh sử dụng `subscription_name` làm khóa của mảng dữ liệu đó. Khóa này sẽ được sử dụng bên trong Cashier.

<a name="checking-subscription-status"></a>
### Kiểm tra trạng thái Subscription

Sau khi người dùng subscription vào ứng dụng của bạn, bạn có thể kiểm tra trạng thái subscription của họ bằng nhiều phương thức tiện lợi khác nhau. Đầu tiên, phương thức `subscribed` sẽ trả về `true` nếu người dùng có subscription đang hoạt động, ngay cả khi subscription hiện tại đang trong thời gian dùng thử:

    if ($user->subscribed('default')) {
        //
    }

Phương thức `subscribed` cũng là một ví dụ tốt cho một [route middleware](/docs/{{version}}/middleware), cho phép bạn chặn các quyền truy cập vào các route và controller mà dựa trên trạng thái subscription của người dùng:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class EnsureUserIsSubscribed
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->user() && ! $request->user()->subscribed('default')) {
                // This user is not a paying customer...
                return redirect('billing');
            }

            return $next($request);
        }
    }

Nếu bạn muốn xác định xem người dùng đó có còn trong thời gian dùng thử hay không, bạn có thể sử dụng phương thức `onTrial`. Phương thức này có thể hữu ích để xác định xem bạn có nên hiển thị cảnh báo cho người dùng biết rằng họ vẫn đang trong thời gian dùng thử:

    if ($user->subscription('default')->onTrial()) {
        //
    }

Phương thức `subscribedToPlan` có thể được sử dụng để xác định xem người dùng có đăng ký gói dịch vụ đã cho hay không dựa vào ID của gói trong Paddle. Trong ví dụ này, chúng ta sẽ xác định xem subscription `default` của người dùng có đăng ký gói `monthly` hay không:

    if ($user->subscribedToPlan($monthly = 12345, 'default')) {
        //
    }

Bằng cách truyền một mảng cho phương thức `subscribedToPlan`, bạn có thể xác định xem subscription `default` của người dùng có được đăng ký với gói `monthly` hay `yearly` hay không:

    if ($user->subscribedToPlan([$monthly = 12345, $yearly = 54321], 'default')) {
        //
    }

Phương thức `recurring` có thể được sử dụng để xác định xem người dùng hiện tại có đang đăng ký và không còn trong thời gian dùng thử hay không:

    if ($user->subscription('default')->recurring()) {
        //
    }

<a name="cancelled-subscription-status"></a>
#### Cancelled Subscription Status

Để xác định xem người dùng đã từng subscription nhưng sau đó đã hủy, bạn có thể sử dụng phương thức `cancelled`:

    if ($user->subscription('default')->cancelled()) {
        //
    }

Bạn cũng có thể xác định xem người dùng đã hủy subscription của họ hay chưa, hay vẫn còn trong "thời gian có hiệu lực" cho đến khi subscription hết hạn. Ví dụ: nếu người dùng hủy subscription vào ngày 5 tháng 3 mà dự kiến ban đầu là sẽ hết hạn vào ngày 10 tháng 3, thì người dùng sẽ ở trong "thời gian có hiệu lực" của họ cho đến ngày 10 tháng 3. Lưu ý rằng phương thức `subscribed` vẫn trả về `true` trong thời gian này:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Để xác định xem người dùng đã hủy subscription và không còn trong "thời gian subscription" của họ, bạn có thể sử dụng phương thức `ended`:

    if ($user->subscription('default')->ended()) {
        //
    }

<a name="past-due-status"></a>
#### Past Due Status

Nếu một khoản thanh toán không thành công cho một subscription, nó sẽ được đánh dấu là `past_due`. Khi subscription của bạn ở trạng thái này, nó sẽ không hoạt động cho đến khi nào khách hàng cập nhật thông tin thanh toán của họ. Bạn có thể xác định xem một subscription có quá hạn hay không bằng cách sử dụng phương thức `pastDue` trên instance subscription:

    if ($user->subscription('default')->pastDue()) {
        //
    }

Khi một subscription quá hạn, bạn nên hướng dẫn người dùng [cập nhật lại thông tin thanh toán của họ](#updating-payment-information). Bạn có thể cấu hình cách xử lý các subscription quá hạn trong [cài đặt subscription của Paddle](https://vendors.paddle.com/subscription-settings).

Nếu bạn muốn các subscription vẫn được coi là hoạt động khi chúng ở trạng thái `past_due`, thì bạn có thể sử dụng phương thức `keepPastDueSubscriptionsActive` do Cashier cung cấp. Thông thường, phương thức này nên được gọi trong phương thức `register` của `AppServiceProvider` của bạn:

    use Laravel\Paddle\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> **Warning**
> Khi một subscription ở trạng thái `past_due`, thì bạn không thể thay đổi subscription cho đến khi thông tin thanh toán được cập nhật. Do đó, các phương thức `swap` và `updateQuantity` sẽ tạo ra một exception khi subscription ở trạng thái `past_due`.

<a name="subscription-scopes"></a>
#### Subscription Scopes

Hầu hết các trạng thái subscription cũng có sẵn dưới dạng query scope giúp cho bạn có thể dễ dàng truy vấn dữ liệu subscription của bạn ở một trạng thái nhất định:

    // Get all active subscriptions...
    $subscriptions = Subscription::query()->active()->get();

    // Get all of the cancelled subscriptions for a user...
    $subscriptions = $user->subscriptions()->cancelled()->get();

Dưới đây là một danh sách đầy đủ các scope có sẵn:

    Subscription::query()->active();
    Subscription::query()->onTrial();
    Subscription::query()->notOnTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();
    Subscription::query()->ended();
    Subscription::query()->paused();
    Subscription::query()->notPaused();
    Subscription::query()->onPausedGracePeriod();
    Subscription::query()->notOnPausedGracePeriod();
    Subscription::query()->cancelled();
    Subscription::query()->notCancelled();
    Subscription::query()->onGracePeriod();
    Subscription::query()->notOnGracePeriod();

<a name="subscription-single-charges"></a>
### Subscription tính phí một lần

Các subscription tính phí một lần cho phép bạn tính phí người dùng với khoản phí một lần trên các gói subscription của họ:

    $response = $user->subscription('default')->charge(12.99, 'Support Add-on');

Ngược lại với [phí](#single-charges), phương thức này sẽ tính phí trực tiếp trên phương thức thanh toán được lưu của khách hàng cho subscription. Số tiền tính phí phải luôn được định nghĩa bằng với đơn vị tiền tệ của subscription.

<a name="updating-payment-information"></a>
### Cập nhật thông tin thanh toán

Paddle luôn lưu một phương thức thanh toán cho mỗi subscription. Nếu bạn muốn cập nhật phương thức thanh toán mặc định cho một subscription, trước tiên bạn nên tạo một "update URL" cho subscription bằng phương thức `updateUrl` trên model subscription:

    use App\Models\User;

    $user = User::find(1);

    $updateUrl = $user->subscription('default')->updateUrl();

Sau đó, bạn có thể sử dụng URL được tạo ra kết hợp với Blade component `paddle-button` do Cashier cung cấp để cho phép người dùng khởi chạy giao diện Paddle và cập nhật thông tin thanh toán của họ:

```html
<x-paddle-button :url="$updateUrl" class="px-8 py-4">
    Update Card
</x-paddle-button>
```

Khi người dùng cập nhật xong thông tin của họ, một webhook `subscription_updated` sẽ được gửi bởi Paddle và chi tiết subscription sẽ được cập nhật trong cơ sở dữ liệu của ứng dụng của bạn.

<a name="changing-plans"></a>
### Thay đổi gói

Sau khi một người dùng đã subscription vào application của bạn, đôi khi họ có thể muốn thay đổi sang gói subscription mới. Để cập nhật gói subscription cho một người dùng, hãy truyền vào một identifier của gói mới của bên Paddle vào phương thức `swap`:

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->swap($premium = 34567);

Nếu bạn muốn thay đổi gói và lập hóa đơn ngay cho người dùng thay vì đợi đến chu kỳ thanh toán tiếp theo của họ, bạn có thể sử dụng phương pháp `swapAndInvoice`:

    $user = User::find(1);

    $user->subscription('default')->swapAndInvoice($premium = 34567);

> **Warning**
> Các gói có thể không được hoán đổi khi bản dùng thử đang được active. Để biết thêm thông tin về hạn chế này, vui lòng xem [tài liệu Paddle](https://developer.paddle.com/api-reference/subscription-api/users/updateuser#usage-notes).

<a name="prorations"></a>
#### Prorations

Mặc định, Paddle sẽ tính phí khi hoán đổi giữa các gói. Phương thức `noProrate` có thể được sử dụng để cập nhật nhiều subscription mà không bị tính phí:

    $user->subscription('default')->noProrate()->swap($premium = 34567);

<a name="subscription-quantity"></a>
### Subscription số lượng lớn

Thỉnh thoảng subscription có thể bị ảnh hưởng bởi "số lượng". Ví dụ: một application quản lý project có thể tính phí $10 mỗi tháng **cho mỗi người dùng** trên mỗi project. Để dễ dàng tăng hoặc giảm số lượng subscription của bạn, hãy sử dụng các phương thức `incrementQuantity` hoặc `decrementQuantity`:

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription('default')->incrementQuantity(5);

    $user->subscription('default')->decrementQuantity();

    // Subtract five from the subscription's current quantity...
    $user->subscription('default')->decrementQuantity(5);

Ngoài ra, bạn cũng có thể set một số lượng cụ thể bằng phương thức `updateQuantity`:

    $user->subscription('default')->updateQuantity(10);

Phương thức `noProrate` có thể được sử dụng để cập nhật số lượng của subscription mà không cần chia tỷ lệ phí:

    $user->subscription('default')->noProrate()->updateQuantity(10);

<a name="subscription-modifiers"></a>
### Subscription modifier

Subscription modifier cho phép bạn triển khai [thanh toán theo số liệu](https://developer.paddle.com/guides/how-tos/subscriptions/metered-billing#using-subscription-price-modifiers) hoặc mở rộng subscription cùng với thêm các add-ons.

Ví dụ: bạn có thể muốn cung cấp thêm một add-on "Premium Support" với subscription tiêu chuẩn của bạn. Bạn có thể tạo ra modifier này như sau:

    $modifier = $user->subscription('default')->newModifier(12.99)->create();

Ví dụ trên sẽ thêm add-on $12,99 vào subscription. Mặc định, khoản phí này sẽ lặp lại trên mỗi khoảng thời gian bạn đã cấu hình cho subscription. Nếu muốn, bạn có thể thêm mô tả vào modifier bằng phương thức `description` của modifier:

    $modifier = $user->subscription('default')->newModifier(12.99)
        ->description('Premium Support')
        ->create();

Để minh họa cách triển khai thanh toán theo số liệu bằng modifiers, hãy tưởng tượng ứng dụng của bạn tính phí cho mỗi tin nhắn SMS do người dùng gửi. Trước tiên, bạn nên tạo một gói $0 trong bảng điều khiển Paddle của bạn. Sau khi người dùng đã đăng ký gói này, bạn có thể thêm các modifiers đại diện cho từng khoản phí riêng lẻ vào gói đăng ký này:

    $modifier = $user->subscription('default')->newModifier(0.99)
        ->description('New text message')
        ->oneTime()
        ->create();

Như bạn có thể thấy, chúng ta đã gọi phương thức `oneTime` khi tạo modifier này. Phương thức này sẽ đảm bảo modifier chỉ bị tính phí một lần và không lặp lại sau mỗi khoảng thời gian thanh toán.

<a name="retrieving-modifiers"></a>
#### Retrieving Modifiers

Bạn có thể lấy ra một danh sách tất cả các modifier cho một subscription thông qua phương thức `modifiers`:

    $modifiers = $user->subscription('default')->modifiers();

    foreach ($modifiers as $modifier) {
        $modifier->amount(); // $0.99
        $modifier->description; // New text message.
    }

<a name="deleting-modifiers"></a>
#### Deleting Modifiers

Modifier có thể bị xóa bằng cách gọi phương thức `delete` trên instance `Laravel\Paddle\Modifier`:

    $modifier->delete();

<a name="multiple-subscriptions"></a>
### Nhiều Subscriptions

Paddle cho phép khách hàng của bạn có thể subscription nhiều loại cùng một lúc. Ví dụ: bạn có thể đang điều hành một phòng gym cung cấp các gói đăng ký bơi và các gói đăng ký tập thể dục và mỗi gói đăng ký lại có thể có các mức giá khác nhau. Tất nhiên, khách hàng có thể đăng ký một hoặc cả hai gói.

Khi ứng dụng của bạn tạo các đăng ký, bạn có thể cung cấp tên của đăng ký cho phương thức `newSubscription`. Tên có thể là bất kỳ chuỗi nào mà đại diện cho loại đăng ký mà người dùng đang muốn sử dụng:

    use Illuminate\Http\Request;

    Route::post('/swimming/subscribe', function (Request $request) {
        $request->user()
            ->newSubscription('swimming', $swimmingMonthly = 12345)
            ->create($request->paymentMethodId);

        // ...
    });

Trong ví dụ trên, chúng ta đã đăng ký bơi hàng tháng cho khách hàng. Nhưng, sau này có thể họ muốn chuyển sang đăng ký theo dạng hàng năm. Khi điều chỉnh đăng ký của khách hàng, chúng ta có thể chỉ cần hoán đổi giá của đăng ký `swimming`:

    $user->subscription('swimming')->swap($swimmingYearly = 34567);

Tất nhiên, bạn cũng có thể hủy đăng ký:

    $user->subscription('swimming')->cancel();

<a name="pausing-subscriptions"></a>
### Tạm dừng Subscriptions

Để tạm dừng một subscription, hãy gọi phương thức `pause` trên subscription của người dùng:

    $user->subscription('default')->pause();

Khi một subscription bị tạm dừng, Cashier sẽ tự động set cột `paused_from` trong cơ sở dữ liệu của bạn. Cột này được sử dụng để biết khi nào thì phương thức `paused` sẽ bắt đầu trả về `true`. Ví dụ: nếu một khách hàng tạm dừng subscription vào ngày 1 tháng 3, nhưng subscription đó có thời hạn đến ngày 5 tháng 3, thì phương thức `paused` sẽ tiếp tục trả về giá trị `false` cho đến hết ngày 5 tháng 3. Điều này được thực hiện là vì người dùng thường được phép tiếp tục sử dụng ứng dụng cho đến khi kết thúc chu kỳ thanh toán của họ.

Bạn có thể xác định xem người dùng đã tạm dừng subscription của họ nhưng vẫn đang trong "thời gian có hạn subscription" bằng cách sử dụng phương thức `onPausedGracePeriod`:

    if ($user->subscription('default')->onPausedGracePeriod()) {
        //
    }

Để resume lại một subscription đã tạm dừng, bạn có thể gọi phương thức `unpause` trên subscription của người dùng:

    $user->subscription('default')->unpause();

> **Warning**
> Không thể sửa subscription khi nó đang bị tạm dừng. Nếu bạn muốn chuyển sang một gói khác hoặc cập nhật số lượng subscription, trước tiên bạn phải resume lại subscription.

<a name="cancelling-subscriptions"></a>
### Huỷ Subscriptions

Để hủy một subscription, hãy gọi phương thức `cancel` trên subscription của người dùng:

    $user->subscription('default')->cancel();

Khi một subscription bị hủy, Cashier sẽ tự động set cột `ends_at` trong cơ sở dữ liệu của bạn. Cột này được sử dụng để biết xem khi nào phương thức `subscribed` sẽ bắt đầu trả về `false`. Ví dụ: nếu khách hàng hủy subscription vào ngày 1 tháng 3, nhưng subscription không thể kết thúc cho đến khi hết ngày 5 tháng 3, thì phương thức `subscribed` vẫn sẽ tiếp tục trả về `true` cho đến ngày 5 tháng 3. Điều này được thực hiện là vì người dùng thường được phép tiếp tục sử dụng ứng dụng cho đến khi kết thúc chu kỳ thanh toán của họ.

Bạn có thể biết những người dùng đã hủy subscription của họ nhưng vẫn đang trong "thời gian subscription có hiệu lực" bằng cách sử dụng phương thức `onGracePeriod`:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Nếu bạn muốn hủy subscription ngay lập tức, hãy gọi phương thức `cancelNow` trên subscription của người dùng:

    $user->subscription('default')->cancelNow();

> **Warning**
> Subscription của Paddle sẽ không thể resume sau khi nó bị hủy. Nếu khách hàng của bạn muốn resume lại subscription của họ, họ sẽ phải đăng ký lại một subscription mới.

<a name="subscription-trials"></a>
## Subscription dành cho dùng thử

<a name="with-payment-method-up-front"></a>
### Khai báo trước phương thức thanh toán

> **Warning**
> Trong khi dùng thử và thu thập thông tin về phương thức thanh toán, Paddle sẽ chặn bất kỳ thay đổi nào liên quan đến subscription, chẳng hạn như chuyển đổi gói hoặc cập nhật số lượng subscription. Nếu bạn muốn cho phép khách hàng chuyển đổi gói trong thời gian dùng thử, subscription phải được hủy và tạo lại từ đầu.

Nếu bạn muốn cung cấp thời gian dùng thử cho khách hàng của bạn trong khi vẫn muốn thu thập thông tin thanh toán của khách hàng, bạn nên sử dụng phương thức `trialDays` khi tạo link thanh toán cho subscription của bạn:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $request->user()->newSubscription('default', $monthly = 12345)
                    ->returnTo(route('home'))
                    ->trialDays(10)
                    ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Phương thức này sẽ set ngày kết thúc của thời gian dùng thử vào trong bản ghi subscription trong cơ sở dữ liệu của application của bạn, và sẽ bảo với Paddle là sẽ không tính phí khách hàng cho đến khi hết ngày dùng thử.

> **Warning**
> Nếu subscription của khách hàng không bị hủy trước ngày kết thúc dùng thử, họ sẽ bị tính phí ngay khi hết hạn dùng thử, vì vậy bạn nên chắc chắn là đã thông báo cho khách hàng biết về ngày kết thúc dùng thử của họ.

Bạn có thể xác định xem người dùng hiện tại có đang trong thời gian dùng thử hay không bằng cách sử dụng phương thức `onTrial` trên instance người dùng hoặc phương thức `onTrial` trên instance subscription. Hai ví dụ dưới đây là giống nhau:

    if ($user->onTrial('default')) {
        //
    }

    if ($user->subscription('default')->onTrial()) {
        //
    }

Để xác định xem bản dùng thử hiện tại đã hết hạn hay chưa, bạn có thể sử dụng phương thức `hasExpiredTrial`:

    if ($user->hasExpiredTrial('default')) {
        //
    }

    if ($user->subscription('default')->hasExpiredTrial()) {
        //
    }

<a name="defining-trial-days-in-paddle-cashier"></a>
#### Defining Trial Days In Paddle / Cashier

Bạn có thể chọn định nghĩa số ngày dùng thử nhận được khi đăng ký gói của bạn trong bảng điều khiển Paddle hoặc luôn truyền chúng vào thông qua Cashier. Nếu bạn chọn định nghĩa ngày dùng thử cho gói của bạn trong Paddle, thì bạn nên biết rằng các subscription mới, bao gồm cả subscription mới cho những khách hàng đã có subscription trước đó, sẽ luôn nhận được thời gian dùng thử trừ khi bạn gọi phương thức `trialDays(0)`.

<a name="without-payment-method-up-front"></a>
### Khai báo phương thức thanh toán sau

Nếu bạn muốn cung cấp thời gian dùng thử mà không muốn thu thập thông tin thanh toán của người dùng, bạn có thể set cột `trial_ends_at` trong bản ghi của người dùng thành ngày kết thúc dùng thử mà bạn mong muốn. Điều này thường được thực hiện trong quá trình đăng ký người dùng:

    use App\Models\User;

    $user = User::create([
        // ...
    ]);

    $user->createAsCustomer([
        'trial_ends_at' => now()->addDays(10)
    ]);

Cashier sẽ coi các loại dùng thử như thế này là "dùng thử đại trà", vì nó sẽ không được gắn với bất kỳ thông tin subscription nào. Phương thức `onTrial` trên instance `User` sẽ trả về `true` nếu ngày hiện tại không vượt quá giá trị của ngày `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

Khi bạn đã sẵn sàng tạo một subscription thực sự cho người dùng, bạn có thể sử dụng phương thức `newSubscription` như bình thường:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $user->newSubscription('default', $monthly = 12345)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Để lấy ra ngày kết thúc dùng thử của một người dùng, bạn có thể sử dụng phương thức `trialEndsAt`. Phương thức này sẽ trả về một instance date Carbon nếu người dùng đang dùng thử hoặc `null` nếu họ không đang dùng thử. Bạn cũng có thể truyền một tham số tên subscription tùy chọn nếu bạn muốn biết ngày kết thúc dùng thử cho một subscription cụ thể khác với subscription mặc định:

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }

Bạn có thể sử dụng phương thức `onGenericTrial` nếu bạn muốn biết cụ thể rằng người dùng đang trong thời gian dùng thử "generic" và chưa tạo subscription thực tế:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

> **Warning**
> Không có cách nào để gia hạn hoặc sửa thời gian dùng thử trên một subscription Paddle sau khi đã được tạo.

<a name="handling-paddle-webhooks"></a>
## Xử lý Paddle Webhooks

Paddle có thể thông báo cho ứng dụng của bạn về nhiều event thông qua webhook. Mặc định, một route sẽ trỏ đến một controller webhook của Cashier được đăng ký bởi service provider của Cashier. Controller này sẽ xử lý tất cả các request webhook gửi đến.

Mặc định, controller này sẽ tự động xử lý việc hủy đăng ký khi có quá nhiều khoản phí không thành công ([như được định nghĩa bởi cài đặt dunning Paddle của bạn](https://vendors.paddle.com/recover-settings#dunning-form-id)), cập nhật đăng ký và thay đổi phương thức thanh toán ; tuy nhiên, như bạn sẽ sớm khám phá ra rằng bạn có thể mở rộng controller này để xử lý bất kỳ sự kiện webhook nào của Paddle mà bạn muốn.

Để đảm bảo ứng dụng của bạn có thể xử lý Paddle webhook, hãy nhớ [cấu hình URL webhook trong bảng điều khiển Paddle](https://vendors.paddle.com/alerts-webhooks). Mặc định, webhook controller của Cashier sẽ response trên đường dẫn URL là `/paddle/webhook`. Danh sách đầy đủ của tất cả các webhook mà bạn nên bật trong bảng điều khiển Paddle sẽ là:

- Subscription Created
- Subscription Updated
- Subscription Cancelled
- Payment Succeeded
- Subscription Payment Succeeded

> **Warning**
> Hãy đảm bảo là bạn đã bảo vệ các request bằng các middleware [kiểm tra định dạng webhook](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures) của Cashier.

<a name="webhooks-csrf-protection"></a>
#### Webhooks & CSRF Protection

Vì các webhook của Paddle cần bỏ qua bước [bảo vệ CSRF](/docs/{{version}}/csrf) của Laravel, nên bạn hãy chắc chắn là đã khai báo URI của Paddle là một ngoại lệ trong middleware `App\Http\Middleware\VerifyCsrfToken` của bạn hoặc bạn có thể khai báo route này ra khỏi group middleware `web`:

    protected $except = [
        'paddle/*',
    ];

<a name="webhooks-local-development"></a>
#### Webhooks & Local Development

Để Paddle có thể gửi webhook cho ứng dụng của bạn trong quá trình phát triển ở local, bạn cần chia sẻ ứng dụng của bạn lên các dịch vụ chia sẻ trang web, chẳng hạn như [Ngrok](https://ngrok.com/) hoặc [Expose](https://expose.dev/docs/introduction). Nếu bạn đang phát triển ứng dụng của bạn bằng cách sử dụng [Laravel Sail](/docs/{{version}}/sail), thì bạn có thể sử dụng [lệnh chia sẻ trang web](/docs/{{version}}/sail#sharing-your-site) của Sail.

<a name="defining-webhook-event-handlers"></a>
### Định nghĩa xử lý Webhook Event

Cashier sẽ tự động xử lý hủy subscription nếu như các lần trả phí không thành công và các webhook Paddle phổ biến khác. Tuy nhiên, nếu bạn  muốn xử lý thêm các sự kiện webhook khác, thì bạn có thể làm như vậy bằng cách listening các event sau do Cashier gửi đi:

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

Cả hai event đều chứa toàn bộ payload của webhook Paddle. Ví dụ: nếu bạn muốn xử lý webhook `invoice.payment_succeeded`, thì bạn có thể đăng ký [listener](/docs/{{version}}/events#defining-listeners) sẽ xử lý event đó:

    <?php

    namespace App\Listeners;

    use Laravel\Paddle\Events\WebhookReceived;

    class PaddleEventListener
    {
        /**
         * Handle received Paddle webhooks.
         *
         * @param  \Laravel\Paddle\Events\WebhookReceived  $event
         * @return void
         */
        public function handle(WebhookReceived $event)
        {
            if ($event->payload['alert_name'] === 'payment_succeeded') {
                // Handle the incoming event...
            }
        }
    }

Khi listener của bạn đã được định nghĩa xong, bạn có thể đăng ký nó trong `EventServiceProvider` của ứng dụng:

    <?php

    namespace App\Providers;

    use App\Listeners\PaddleEventListener;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    use Laravel\Paddle\Events\WebhookReceived;

    class EventServiceProvider extends ServiceProvider
    {
        protected $listen = [
            WebhookReceived::class => [
                PaddleEventListener::class,
            ],
        ];
    }

Cashier cũng phát ra các event dành riêng cho các loại webhook đã nhận. Ngoài toàn bộ payload từ Paddle, chúng cũng chứa các model đã được sử dụng để xử lý webhook, chẳng hạn như billable model, subscription hoặc receipt:

<div class="content-list" markdown="1">

- `Laravel\Paddle\Events\PaymentSucceeded`
- `Laravel\Paddle\Events\SubscriptionPaymentSucceeded`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionCancelled`

</div>

Bạn cũng có thể ghi đè route webhook mặc định bằng cách định nghĩa biến môi trường `CASHIER_WEBHOOK` trong file `.env` của application của bạn. Giá trị này phải là một URL đầy đủ cho route webhook của bạn và cần giống với URL mà được set trong bảng điều khiển Paddle của bạn:

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### Kiểm tra định dạng Webhook

Để bảo mật webhook của bạn, bạn có thể sử dụng [định dạng webhook của Paddle](https://developer.paddle.com/webhook-reference/verifying-webhooks). Để thuận tiện, Cashier đã tự động thêm một middleware để kiểm tra các request webhook Paddle đến application là hợp lệ.

Để bật kiểm tra webhook, hãy đảm bảo rằng biến môi trường `PADDLE_PUBLIC_KEY` được định nghĩa trong file `.env` của application của bạn. Public key có thể được lấy ra từ trang tổng quan trong tài khoản Paddle của bạn.

<a name="single-charges"></a>
## Phí

<a name="simple-charge"></a>
### Tính phí một lần

Nếu bạn muốn thực hiện một khoản tính phí một lần đối với một khách hàng, bạn có thể sử dụng phương pháp `charge` trên instance billable model để tạo link thanh toán cho khoản phí đó. Phương thức `charge` chấp nhận số tiền phí (float) làm tham số đầu tiên và mô tả về khoản phí đó làm tham số thứ hai:

    use Illuminate\Http\Request;

    Route::get('/store', function (Request $request) {
        return view('store', [
            'payLink' => $user->charge(12.99, 'Action Figure')
        ]);
    });

Sau khi đã tạo link thanh toán, bạn có thể sử dụng Blade component `paddle-button` do Cashier cung cấp để cho phép người dùng khởi chạy giao diện Paddle và hoàn thành khoản phí đó:

```blade
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Buy
</x-paddle-button>
```

Phương thức `charge` sẽ chấp nhận thêm một mảng làm tham số thứ ba của nó, cho phép bạn truyền bất kỳ tùy chọn nào mà bạn muốn để tạo link thanh toán Paddle. Vui lòng tham khảo [tài liệu của Paddle](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink) để tìm hiểu thêm về các tùy chọn có sẵn cho bạn khi tạo phí:

    $payLink = $user->charge(12.99, 'Action Figure', [
        'custom_option' => $value,
    ]);

Các khoản phí sẽ được tính theo đơn vị tiền tệ được chỉ định trong tùy chọn cấu hình `cashier.currency`. Mặc định, giá trị này được set thành USD. Bạn có thể ghi đè đơn vị tiền tệ mặc định này bằng cách định nghĩa biến môi trường `CASHIER_CURRENCY` trong file `.env` application của của bạn:

```ini
CASHIER_CURRENCY=EUR
```

Bạn cũng có thể [ghi đè giá tiền theo đơn vị tiền tệ](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink#price-overrides) bằng cách sử dụng hệ thống dynamic pricing matching của Paddle. Để làm như thế, hãy truyền một mảng giá tiền thay vì một số tiền cố định:

    $payLink = $user->charge([
        'USD:19.99',
        'EUR:15.99',
    ], 'Action Figure');

<a name="charging-products"></a>
### Tính phí sản phẩm

Nếu bạn muốn tính phí một lần cho một sản phẩm cụ thể được cấu hình trong Paddle, bạn có thể sử dụng phương thức `chargeProduct` trên một instance billable model để tạo ra một link thanh toán:

    use Illuminate\Http\Request;

    Route::get('/store', function (Request $request) {
        return view('store', [
            'payLink' => $request->user()->chargeProduct($productId = 123)
        ]);
    });

Sau đó, bạn có thể cung cấp link thanh toán đó đến component `paddle-button` để cho phép người dùng khởi chạy giao diện Paddle:

```blade
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Buy
</x-paddle-button>
```

Phương thức `chargeProduct` sẽ chấp nhận thêm một mảng làm tham số thứ hai của nó, cho phép bạn truyền bất kỳ tùy chọn nào mà bạn muốn để tạo link thanh toán Paddle. Vui lòng tham khảo [tài liệu của Paddle](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink) để tìm hiểu thêm về các tùy chọn có sẵn cho bạn khi tạo phí:

    $payLink = $user->chargeProduct($productId, [
        'custom_option' => $value,
    ]);

<a name="refunding-orders"></a>
### Hoàn trả

Nếu bạn cần hoàn trả một phí đã được thanh toán trong Paddle, bạn có thể sử dụng phương thức `refund`. Phương thức này chấp nhận một Paddle order ID làm tham số đầu tiên của nó. Bạn có thể lấy ra biên lai cho một billable model nhất định bằng phương thức `receipts`:

    use App\Models\User;

    $user = User::find(1);

    $receipt = $user->receipts()->first();

    $refundRequestId = $user->refund($receipt->order_id);

Bạn có thể tùy ý chỉ định một số tiền cụ thể để hoàn trả cũng như lý do hoàn trả:

    $receipt = $user->receipts()->first();

    $refundRequestId = $user->refund(
        $receipt->order_id, 5.00, 'Unused product time'
    );

> **Note**
> Bạn có thể sử dụng `$refundRequestId` làm tham chiếu cho khoản tiền đã hoàn lại khi liên hệ với bộ phận hỗ trợ của Paddle.

<a name="receipts"></a>
## Biên lai

Bạn có thể dễ dàng lấy ra một mảng các biên lai của một model billable bằng cách sử dụng thuộc tính `receipts`:

    use App\Models\User;

    $user = User::find(1);

    $receipts = $user->receipts;

Khi liệt kê biên lai cho khách hàng, bạn có thể sử dụng các phương thức của instance biên lai để hiển thị các thông tin liên quan về biên lại. Ví dụ: bạn có thể muốn liệt kê mọi biên lai vào trong một bảng, và cho phép người dùng dễ dàng tải xuống bất kỳ biên lai nào:

```html
<table>
    @foreach ($receipts as $receipt)
        <tr>
            <td>{{ $receipt->paid_at->toFormattedDateString() }}</td>
            <td>{{ $receipt->amount() }}</td>
            <td><a href="{{ $receipt->receipt_url }}" target="_blank">Download</a></td>
        </tr>
    @endforeach
</table>
```

<a name="past-and-upcoming-payments"></a>
### Các khoản thanh toán quá khứ và tương lai

Bạn có thể sử dụng phương thức `lastPayment` và `nextPayment` để nhận về và hiển thị các khoản thanh toán quá khứ hoặc tương lai của khách hàng cho các subscription định kỳ:

    use App\Models\User;

    $user = User::find(1);

    $subscription = $user->subscription('default');

    $lastPayment = $subscription->lastPayment();
    $nextPayment = $subscription->nextPayment();

Cả hai phương thức này sẽ trả về một instance của `Laravel\Paddle\Payment`; tuy nhiên, `nextPayment` sẽ trả về `null` nếu chu kỳ thanh toán đã kết thúc (chẳng hạn như khi subscription bị hủy):

```blade
Next payment: {{ $nextPayment->amount() }} due on {{ $nextPayment->date()->format('d/m/Y') }}
```

<a name="handling-failed-payments"></a>
## Xử lý các khoản thanh toán không thành công

Thỉnh thoảng nhiều khi thanh toán subscription có thể không thành công vì nhiều lý do khác nhau, chẳng hạn như thẻ hết hạn hoặc thẻ không có đủ tiền. Khi điều này xảy ra, chúng tôi khuyên bạn nên để Paddle xử lý các lỗi thanh toán cho bạn. Cụ thể, bạn có thể [thiết lập email thanh toán tự động của Paddle](https://vendors.paddle.com/subscription-settings) trong trang tổng quan của Paddle.

Ngoài ra, bạn cũng có thể thực hiện những tùy chỉnh chính xác hơn bằng cách [lắng nghe](/docs/{{version}}/events) event [`subscription_payment_failed`] của Paddle thông qua event `WebhookReceived` được Cashier gửi đi. Bạn cũng nên đảm bảo "thanh toán subscription không thành công" đã được bật trong cài đặt Webhook trong trang tổng quan Paddle của bạn:

    <?php

    namespace App\Listeners;

    use Laravel\Paddle\Events\WebhookReceived;

    class PaddleEventListener
    {
        /**
         * Handle received Paddle webhooks.
         *
         * @param  \Laravel\Paddle\Events\WebhookReceived  $event
         * @return void
         */
        public function handle(WebhookReceived $event)
        {
            if ($event->payload['alert_name'] === 'subscription_payment_failed') {
                // Handle the failed subscription payment...
            }
        }
    }

Khi listener của bạn đã được định nghĩa, bạn nên đăng ký nó trong `EventServiceProvider` của ứng dụng:

    <?php

    namespace App\Providers;

    use App\Listeners\PaddleEventListener;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    use Laravel\Paddle\Events\WebhookReceived;

    class EventServiceProvider extends ServiceProvider
    {
        protected $listen = [
            WebhookReceived::class => [
                PaddleEventListener::class,
            ],
        ];
    }

<a name="testing"></a>
## Testing

Trong khi testing, bạn nên test luồng thanh toán của bạn bằng cách thủ công để đảm bảo các tích hợp của bạn hoạt động như mong đợi.

Đối với các automated test, chứa cả những bài test được thực hiện trong môi trường CI, bạn có thể sử dụng [HTTP Client của Laravel](/docs/{{version}}/http-client#testing) để fake các request HTTP được thực hiện tới Paddle. Mặc dù điều này không kiểm tra các phản hồi thực tế từ Paddle, nhưng nó cũng cung cấp một cách để kiểm tra ứng dụng của bạn mà không thực sự gọi đến API của Paddle.
