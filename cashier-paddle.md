# Laravel Cashier Paddle

- [Giới thiệu](#introduction)
- [Cập nhật Cashier](#upgrading-cashier)
- [Cài đặt](#installation)
    - [Paddle Sandbox](#paddle-sandbox)
- [Cấu hình](#configuration)
    - [Billable Model](#billable-model)
    - [API Keys](#api-keys)
    - [Paddle JS](#paddle-js)
    - [Cấu hình đơn vị tiền tệ](#currency-configuration)
    - [Ghi đè Model mặc định](#overriding-default-models)
- [Bắt đầu nhanh](#quickstart)
    - [Bán sản phẩm](#quickstart-selling-products)
    - [Bán subscription](#quickstart-selling-subscriptions)
- [Checkout Sessions](#checkout-sessions)
    - [Overlay Checkout](#overlay-checkout)
    - [Inline Checkout](#inline-checkout)
    - [Guest Checkouts](#guest-checkouts)
- [Price Previews](#price-previews)
    - [Customer Price Previews](#customer-price-previews)
    - [Discounts](#price-discounts)
- [Customers](#customers)
    - [Customer mặc định](#customer-defaults)
    - [Lấy customer](#retrieving-customers)
    - [Tạo customer](#creating-customers)
- [Subscriptions](#subscriptions)
    - [Tạo Subscription](#creating-subscriptions)
    - [Kiểm tra trạng thái Subscription](#checking-subscription-status)
    - [Subscription tính phí một lần](#subscription-single-charges)
    - [Cập nhật thông tin thanh toán](#updating-payment-information)
    - [Thay đổi gói](#changing-plans)
    - [Subscription số lượng lớn](#subscription-quantity)
    - [Subscription với nhiều sản phẩm](#subscriptions-with-multiple-products)
    - [Nhiều Subscriptions](#multiple-subscriptions)
    - [Tạm dừng Subscriptions](#pausing-subscriptions)
    - [Huỷ Subscriptions](#canceling-subscriptions)
- [Subscription dành cho dùng thử](#subscription-trials)
    - [Khai báo trước phương thức thanh toán](#with-payment-method-up-front)
    - [Khai báo phương thức thanh toán sau](#without-payment-method-up-front)
    - [Gia hạn và kích hoạt dùng thử](#extend-or-activate-a-trial)
- [Xử lý Paddle Webhooks](#handling-paddle-webhooks)
    - [Định nghĩa xử lý Webhook Event](#defining-webhook-event-handlers)
    - [Kiểm tra định dạng Webhook](#verifying-webhook-signatures)
- [Tính phí một lần](#single-charges)
    - [Tính phí sản phẩm](#charging-for-products)
    - [Hoàn tiền giao dịch](#refunding-transactions)
    - [Credit transaction](#crediting-transactions)
- [Giao dịch](#transactions)
    - [Các khoản thanh toán quá khứ và tương lai](#past-and-upcoming-payments)
- [Testing](#testing)

<a name="introduction"></a>
## Giới thiệu

> [!WARNING]
> Tài liệu này dành cho việc tích hợp Cashier Paddle 2.x với Paddle Billing. Nếu bạn vẫn đang sử dụng Paddle Classic, bạn nên sử dụng [Cashier Paddle 1.x](https://github.com/laravel/cashier-paddle/tree/1.x).

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) cung cấp một interface dễ hiểu, rõ ràng cho các dịch vụ thanh toán theo subscription của [Paddle](https://paddle.com). Nó xử lý hầu hết các code thanh toán subscription theo mẫu mà bạn đang lo sợ. Ngoài việc quản lý subscription cơ bản. Cashier có thể xử lý: thay đổi subscription, "số lượng" subscription, tạm dừng subscription, gia hạn thời gian hủy subscription...

Trước khi tìm hiểu sâu hơn về Cashier Paddle, chúng tôi khuyên bạn cũng nên xem qua các [khái niệm](https://developer.paddle.com/concepts/overview) và [tài liệu API](https://developer.paddle.com/api-reference/overview) của Paddle.

<a name="upgrading-cashier"></a>
## Cập nhật Cashier

Khi nâng cấp lên phiên bản mới của Cashier, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md).

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt package Cashier cho Paddle bằng trình quản lý package Composer:

```shell
composer require laravel/cashier-paddle
```

Tiếp theo, bạn nên publish các file migration của Cashier bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

Sau đó, bạn nên chạy migration cơ sở dữ liệu của ứng dụng. Migration Cashier sẽ tạo ra một bảng `customers` mới. Ngoài ra, các bảng `subscriptions` và `subscription_items` mới sẽ được tạo để lưu tất cả các subscription của khách hàng. Cuối cùng, một bảng `transactions` mới sẽ được tạo để lưu tất cả các giao dịch Paddle liên quan đến khách hàng của bạn:

```shell
php artisan migrate
```

> [!WARNING]
> Để đảm bảo Cashier xử lý đúng tất cả các event của Paddle, hãy nhớ [thiết lập xử lý webhook của Cashier](#handling-stripe-webhooks).

<a name="paddle-sandbox"></a>
### Paddle Sandbox

Trong quá trình phát triển local và staging, bạn nên [đăng ký một tài khoản Paddle Sandbox](https://sandbox-login.paddle.com/signup). Tài khoản này sẽ cung cấp cho bạn một môi trường sandbox để thử nghiệm và phát triển các ứng dụng của bạn mà không cần thực hiện thanh toán. Bạn có thể sử dụng [test card numbers](https://developer.paddle.com/concepts/payment-methods/credit-debit-card) của Paddle để mô phỏng các tình huống thanh toán khác nhau.

Khi sử dụng môi trường Paddle Sandbox, bạn nên set biến môi trường `PADDLE_SANDBOX` thành `true` trong file `.env` của ứng dụng:

```ini
PADDLE_SANDBOX=true
```

Sau khi hoàn thành việc phát triển ứng dụng của bạn, bạn có thể [đăng ký tài khoản nhà cung cấp Paddle](https://paddle.com). Trước khi ứng dụng của bạn được deploy lên production, Paddle sẽ cần phê duyệt domain ứng dụng của bạn.

<a name="configuration"></a>
## Cấu hình

<a name="billable-model"></a>
### Billable Model

Trước khi sử dụng Cashier, hãy thêm trait `Billable` vào định nghĩa model của bạn. Trait này sẽ cung cấp các phương thức khác nhau cho phép bạn thực hiện các tác vụ thanh toán phổ biến, chẳng hạn như tạo subscription hoặc cập nhật thông tin phương thức thanh toán:

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
PADDLE_CLIENT_SIDE_TOKEN=your-paddle-client-side-token
PADDLE_API_KEY=your-paddle-api-key
PADDLE_RETAIN_KEY=your-paddle-retain-key
PADDLE_WEBHOOK_SECRET="your-paddle-webhook-secret"
PADDLE_SANDBOX=true
```

Biến môi trường `PADDLE_SANDBOX` phải được set thành `true` khi bạn đang sử dụng trong [môi trường Sandbox của Paddle](#paddle-sandbox). Biến `PADDLE_SANDBOX` phải được set thành `false` nếu bạn đang triển khai ứng dụng của bạn sang production và đang sử dụng môi trường nhà cung cấp trực tiếp của Paddle.

`PADDLE_RETAIN_KEY` là tùy chọn và chỉ nên được cài đặt nếu bạn đang sử dụng Paddle với [Retain](https://developer.paddle.com/paddlejs/retain).

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

Bạn có thể chỉ định ngôn ngữ được sử dụng khi định dạng tiền tệ để hiển thị trong hóa đơn. Cashier sử dụng [class `NumberFormatter` của PHP](https://www.php.net/manual/en/class.numberformatter.php) để set ngôn ngữ tiền tệ:

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]
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

    use App\Models\Cashier\Subscription;
    use App\Models\Cashier\Transaction;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Cashier::useSubscriptionModel(Subscription::class);
        Cashier::useTransactionModel(Transaction::class);
    }

<a name="quickstart"></a>
## Bắt đầu nhanh

<a name="quickstart-selling-products"></a>
### Bán sản phẩm

> [!NOTE]
> Trước khi sử dụng Paddle Checkout, bạn nên định nghĩa sản phẩm với giá cố định của nó trong bảng điều khiển Paddle của bạn. Ngoài ra, bạn nên [cấu hình xử lý webhook của Paddle](#handling-paddle-webhooks).

Việc cung cấp thanh toán sản phẩm và subscription thông qua ứng dụng của bạn có thể gây khó khăn. Tuy nhiên, nhờ Cashier và [Checkout Overlay của Paddle](https://www.paddle.com/billing/checkout), bạn có thể dễ dàng tích hợp các thanh toán hiện đại, mạnh mẽ.

Để tính phí khách hàng cho các sản phẩm không định kỳ, tính phí một lần, chúng ta sẽ sử dụng Cashier để hướng dẫn khách hàng đến trang Checkout Overlay của Paddle, nơi họ sẽ cần phải cung cấp thông tin thanh toán và xác nhận giao dịch mua của họ. Sau khi thanh toán qua Checkout Overlay, khách hàng sẽ được chuyển hướng đến URL thành công do bạn đăng ký trong ứng dụng của bạn:

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $request->user()->checkout('pri_deluxe_album')
            ->returnTo(route('dashboard'));

        return view('buy', ['checkout' => $checkout]);
    })->name('checkout');

Như bạn có thể thấy trong ví dụ trên, chúng ta sẽ sử dụng phương thức `checkout` do Cashier cung cấp để chuyển hướng khách hàng đến Paddle Checkout Overlay với một "mã giá" nhất định. Khi sử dụng Paddle, "giá" ám chỉ [giá đã được định nghĩa cho các sản phẩm cụ thể](https://developer.paddle.com/build/products/create-products-prices).

Nếu cần, phương thức `checkout` sẽ tự động tạo một khách hàng trong Paddle và kết nối khách hàng Paddle đó với người dùng tương ứng trong cơ sở dữ liệu của bạn. Sau khi hoàn tất thanh toán, khách hàng sẽ được chuyển hướng đến trang thành công, nơi mà bạn có thể hiển thị thông báo thông tin cho khách hàng.

Trong view `buy`, chúng ta sẽ thêm một nút để hiển thị Checkout Overlay. Blade component `paddle-button` đã được chứa trong Cashier Paddle; tuy nhiên, bạn cũng có thể [tự hiển thị overlay checkout](#manually-rendering-an-overlay-checkout):

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy Product
</x-paddle-button>
```

<a name="providing-meta-data-to-paddle-checkout"></a>
#### Providing Meta Data to Paddle Checkout

Khi bán sản phẩm, thông thường bạn sẽ theo dõi các đơn hàng đã hoàn tất và các sản phẩm đã mua thông qua các model `Cart` và `Order` do chính ứng dụng của bạn định nghĩa. Khi chuyển hướng khách hàng đến Checkout Overlay của Paddle để hoàn tất giao dịch mua, bạn có thể cần cung cấp một mã đơn hàng hiện có để có thể liên kết giao dịch mua đã hoàn tất với đơn hàng tương ứng khi khách hàng được chuyển hướng trở lại ứng dụng của bạn.

Để thực hiện điều này, bạn có thể cung cấp một mảng cho phương thức `checkout`. Hãy tưởng tượng một `Order` đang được chờ xử lý được tạo ra trong ứng dụng khi người dùng bắt đầu quá trình thanh toán. Hãy nhớ rằng, các model `Cart` và `Order` trong ví dụ này chỉ mang tính minh họa và không được Cashier cung cấp. Bạn có thể tự do triển khai các khái niệm này dựa trên nhu cầu của ứng dụng của bạn:

    use App\Models\Cart;
    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/cart/{cart}/checkout', function (Request $request, Cart $cart) {
        $order = Order::create([
            'cart_id' => $cart->id,
            'price_ids' => $cart->price_ids,
            'status' => 'incomplete',
        ]);

        $checkout = $request->user()->checkout($order->price_ids)
            ->customData(['order_id' => $order->id]);

        return view('billing', ['checkout' => $checkout]);
    })->name('checkout');

Như bạn có thể thấy trong ví dụ trên, khi người dùng bắt đầu quy trình thanh toán, chúng ta sẽ cung cấp tất cả các mã giá Paddle liên quan đến cart hoặc order cho phương thức `checkout`. Tất nhiên, ứng dụng của bạn có trách nhiệm liên kết các item này với "shopping cart" hoặc order khi khách hàng thêm chúng. Chúng ta cũng cung cấp ID của order cho Paddle Checkout Overlay thông qua phương thức `customData`.

Tất nhiên, bạn có thể muốn đánh dấu đơn hàng là đã "hoàn tất" sau khi khách hàng đã hoàn tất quy trình thanh toán. Để thực hiện việc này, bạn có thể listen các webhook được Paddle gửi đi và được Cashier đưa ra thông qua các event để lưu trữ thông tin đơn hàng vào trong cơ sở dữ liệu của bạn.

Để bắt đầu, hãy listen event `TransactionCompleted` được Cashier gửi. Thông thường, bạn nên đăng ký listen event này trong phương thức `boot` của một trong các service provider của ứng dụng:

    use App\Listeners\CompleteOrder;
    use Illuminate\Support\Facades\Event;
    use Laravel\Paddle\Events\TransactionCompleted;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::listen(TransactionCompleted::class, CompleteOrder::class);
    }

Trong ví dụ này, listener `CompleteOrder` có thể sẽ trông như sau:

    namespace App\Listeners;

    use App\Models\Order;
    use Laravel\Cashier\Cashier;
    use Laravel\Cashier\Events\TransactionCompleted;

    class CompleteOrder
    {
        /**
         * Handle the incoming Cashier webhook event.
         */
        public function handle(TransactionCompleted $event): void
        {
            $orderId = $event->payload['data']['custom_data']['order_id'] ?? null;

            $order = Order::findOrFail($orderId);

            $order->update(['status' => 'completed']);
        }
    }

Vui lòng tham khảo thêm tài liệu của Paddle để biết thêm thông tin về [dữ liệu chứa trong event `transaction.completed`](https://developer.paddle.com/webhooks/transactions/transaction-completed).

<a name="quickstart-selling-subscriptions"></a>
### Bán subscription

> [!NOTE]
> Trước khi sử dụng Paddle Checkout, bạn nên định nghĩa sản phẩm với giá cố định của nó trong bảng điều khiển Paddle của bạn. Ngoài ra, bạn nên [cấu hình xử lý webhook của Paddle](#handling-paddle-webhooks).

Việc cung cấp thanh toán sản phẩm và subscription thông qua ứng dụng của bạn có thể gây khó khăn. Tuy nhiên, nhờ Cashier và [Checkout Overlay của Paddle](https://www.paddle.com/billing/checkout), bạn có thể dễ dàng tích hợp các thanh toán hiện đại, mạnh mẽ.

Để tìm hiểu cách bán subscription bằng Cashier và Checkout Overlay của Paddle, hãy xem xét một kịch bản đơn giản của subscription service với gói cơ bản hàng tháng (`price_basic_monthly`) và hàng năm (`price_basic_yearly`). Hai mức giá này có thể được nhóm lại thành sản phẩm "cơ bản" (`pro_basic`) trong bảng điều khiển Paddle của chúng ta. Ngoài ra, subscription service của chúng ta có thể cung cấp gói nâng cao là `pro_expert`.

Trước tiên, hãy cùng khám phá cách khách hàng có thể đăng ký dịch vụ của chúng ta. Tất nhiên, bạn có thể tưởng tượng khách hàng có thể ấn vào nút "đăng ký" cho gói cơ bản trên trang giá của ứng dụng. Nút này sẽ gọi đến Paddle Checkout Overlay cho gói mà họ đã chọn. Để bắt đầu, hãy khởi tạo phiên thanh toán thông qua phương thức `checkout`:

    use Illuminate\Http\Request;

    Route::get('/subscribe', function (Request $request) {
        $checkout = $request->user()->checkout('price_basic_monthly')
            ->returnTo(route('dashboard'));

        return view('subscribe', ['checkout' => $checkout]);
    })->name('subscribe');

Trong view `subscribe`, chúng ta sẽ thêm một nút để hiển thị Checkout Overlay. Blade component `paddle-button` đã được chứa trong Cashier Paddle; tuy nhiên, bạn cũng có thể [tự hiển thị overlay checkout](#manually-rendering-an-overlay-checkout):

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Bây giờ, khi ấn vào nút Subscribe, khách hàng sẽ có thể nhập thông tin thanh toán và bắt đầu đăng ký. Để biết khi nào đăng ký của khách hàng thực sự bắt đầu (vì một số phương thức thanh toán cần vài giây để xử lý), chúng ta cũng cần [cấu hình xử lý webhook của Cashier](#handling-paddle-webhooks).

Bây giờ khách hàng có thể bắt đầu đăng ký, chúng ta sẽ cần hạn chế một số phần nhất định của ứng dụng để chỉ có những người dùng nào đăng ký mới có thể truy cập. Tất nhiên, chúng ta có thể luôn xác định được trạng thái đăng ký hiện tại của người dùng thông qua phương thức `subscribed` do trait `Billable` của Cashier cung cấp:

```blade
@if ($user->subscribed())
    <p>You are subscribed.</p>
@endif
```

We can even easily determine if a user is subscribed to specific product or price:

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### Building a Subscribed Middleware

Để thuận tiện, bạn có thể muốn tạo một [middleware](/docs/{{version}}/middleware) để xác định xem request đến có phải từ một người dùng đã đăng ký rồi hay không. Sau khi middleware này được định nghĩa, bạn có thể dễ dàng gán nó cho một route để chặn những người dùng chưa đăng ký truy cập route:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class Subscribed
    {
        /**
         * Handle an incoming request.
         */
        public function handle(Request $request, Closure $next): Response
        {
            if (! $request->user()?->subscribed()) {
                // Redirect user to billing page and ask them to subscribe...
                return redirect('/subscribe');
            }

            return $next($request);
        }
    }

Sau khi middleware đã được định nghĩa, bạn có thể gán nó cho một route:

    use App\Http\Middleware\Subscribed;

    Route::get('/dashboard', function () {
        // ...
    })->middleware([Subscribed::class]);

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### Allowing Customers to Manage Their Billing Plan

Tất nhiên, khách hàng có thể muốn thay đổi gói đăng ký của họ sang một sản phẩm khác hoặc một "cấp" khác. Trong ví dụ trên, chúng ta muốn cho phép khách hàng thay đổi gói đăng ký của họ từ đăng ký hàng tháng sang đăng ký hàng năm. Đối với điều này, bạn sẽ cần phải làm một cái gì đó giống như một button để dẫn đến route bên dưới:

    use Illuminate\Http\Request;

    Route::put('/subscription/{price}/swap', function (Request $request, $price) {
        $user->subscription()->swap($price); // With "$price" being "price_basic_yearly" for this example.

        return redirect()->route('dashboard');
    })->name('subscription.swap');

Bên cạnh việc đổi gói, bạn cũng cần cho phép khách hàng của bạn hủy đăng ký. Giống như việc đổi gói, hãy cung cấp một button dẫn đến route sau:

    use Illuminate\Http\Request;

    Route::put('/subscription/cancel', function (Request $request, $price) {
        $user->subscription()->cancel();

        return redirect()->route('dashboard');
    })->name('subscription.cancel');

Và bây giờ đăng ký của họ sẽ bị hủy vào cuối thời hạn thanh toán.

> [!NOTE]
> Miễn là bạn đã cấu hình xử lý webhook của Cashier, Cashier sẽ tự động giữ các bảng cơ sở dữ liệu liên quan đến Cashier của ứng dụng của bạn đồng bộ thông qua các webhook đến từ Paddle. Vì vậy, ví dụ, khi người dùng hủy đăng ký của họ thông qua bảng điều khiển của Paddle, Cashier sẽ nhận được webhook tương ứng và đánh dấu đăng ký là "đã bị hủy" trong cơ sở dữ liệu ứng dụng của bạn.

<a name="checkout-sessions"></a>
## Checkout Sessions

Hầu hết các hoạt động lập hóa đơn cho khách hàng đều được thực hiện bằng cách "checkout" thông qua [Checkout Overlay widget](https://developer.paddle.com/build/checkout/build-overlay-checkout) của Paddle hoặc bằng cách sử dụng [inline checkout](https://developer.paddle.com/build/checkout/build-branded-inline-checkout).

Trước khi xử lý thanh toán bằng Paddle, bạn nên định nghĩa một [link thanh toán mặc định](https://developer.paddle.com/build/transactions/default-payment-link#set-default-link) của ứng dụng trong bảng điều khiển cài đặt thanh toán Paddle.

<a name="overlay-checkout"></a>
### Overlay Checkout

Trước khi hiển thị Checkout Overlay widget, bạn phải tạo một checkout session bằng Cashier. Checkout session này sẽ thông báo cho checkout widget về thao tác thanh toán cần thực hiện:

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $user->checkout('pri_34567')
            ->returnTo(route('dashboard'));

        return view('billing', ['checkout' => $checkout]);
    });

Cashier đã chứa một [component Blade](/docs/{{version}}/blade#components) `paddle-button`. Bạn có thể truyền checkout session cho component này dưới dạng một "prop". Sau đó, khi nhấp vào nút này, checkout widget của Paddle sẽ được hiển thị:

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Mặc định, điều này sẽ hiển thị một widget mặc định của Paddle. Bạn có thể tùy chỉnh widget này bằng cách thêm [thuộc tính mà được Paddle hỗ trợ](https://developer.paddle.com/paddlejs/html-data-attributes) như thuộc tính `data-theme='light'` vào component:

```html
<x-paddle-button :url="$payLink" class="px-8 py-4" data-theme="light">
    Subscribe
</x-paddle-button>
```

Giao diện thanh toán bằng Paddle là không đồng bộ. Sau khi người dùng tạo hoặc cập nhật một subscription trong giao diện, Paddle sẽ gửi một webhook đến ứng dụng của bạn để bạn có thể cập nhật đúng trạng thái của subscription trong cơ sở dữ liệu của bạn. Do đó, điều quan trọng nhất là bạn phải [thiết lập webhook](#handling-paddle-webhooks) để tương thích với các thay đổi trạng thái từ Paddle.

> [!WARNING]
> Sau khi thay đổi trạng thái đăng ký, độ trễ để nhận được webhook tương ứng thường rất nhỏ nhưng bạn nên tính đến điều này trong ứng dụng của bạn bằng cách cân nhắc rằng subscription của người dùng của bạn có thể không khả dụng ngay sau khi hoàn tất quy trình thanh toán.

<a name="manually-rendering-an-overlay-checkout"></a>
#### Manually Rendering an Overlay Checkout

Bạn cũng có thể hiển thị một overlay checkout theo cách thủ công mà không cần phải sử dụng các component Blade có sẵn của Laravel. Để bắt đầu, hãy tạo một checkout session [như minh họa trong các ví dụ trước](#overlay-checkout):

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $user->checkout('pri_34567')
            ->returnTo(route('dashboard'));

        return view('billing', ['checkout' => $checkout]);
    });

Tiếp theo, bạn có thể sử dụng Paddle.js để khởi tạo checkout. Trong ví dụ này, chúng ta sẽ tạo một link được gán vào class `paddle_button`. Paddle.js sẽ phát hiện class này và hiển thị overlay checkout khi link được nhấn vào:

```blade
<?php
$items = $checkout->getItems();
$customer = $checkout->getCustomer();
$custom = $checkout->getCustomData();
?>

<a
    href='#!'
    class='paddle_button'
    data-items='{!! json_encode($items) !!}'
    @if ($customer) data-customer-id='{{ $customer->paddle_id }}' @endif
    @if ($custom) data-custom-data='{{ json_encode($custom) }}' @endif
    @if ($returnUrl = $checkout->getReturnUrl()) data-success-url='{{ $returnUrl }}' @endif
>
    Buy Product
</a>
```

<a name="inline-checkout"></a>
### Inline Checkout

Nếu bạn không muốn sử dụng giao diện thanh toán theo kiểu "overlay" của Paddle, Paddle cũng cung cấp thêm một tùy chọn để hiển thị giao diện inline. Mặc dù cách tiếp cận này không cho phép bạn điều chỉnh bất kỳ trường HTML nào của thanh toán, nhưng nó cho phép bạn nhúng giao diện đó vào ứng dụng của bạn.

Để giúp bạn dễ dàng bắt đầu với tính năng thanh toán inline, Cashier đã chứa một Blade component `paddle-checkout`. Để bắt đầu, bạn nên [tạo một checkout session](#overlay-checkout):

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $user->checkout('pri_34567')
            ->returnTo(route('dashboard'));

        return view('billing', ['checkout' => $checkout]);
    });

Sau đó, bạn có thể truyền checkout session đến thuộc tính `checkout` của component:

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" />
```

Để điều chỉnh chiều cao của component inline checkout, bạn có thể truyền thuộc tính `height` cho Blade component:

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" height="500" />
```

Vui lòng tham khảo [hướng dẫn về Inline Checkout](https://developer.paddle.com/build/checkout/build-branded-inline-checkout) và [các cài đặt checkout có sẵn](https://developer.paddle.com/build/checkout/set-up-checkout-default-settings) của Paddle để biết thêm chi tiết hơn về các tùy chọn của inline checkout.

<a name="manually-rendering-an-inline-checkout"></a>
#### Manually Rendering An Inline Checkout

Bạn cũng có thể hiển thị thanh toán inline theo cách thủ công mà không cần sử dụng các component Blade có sẵn của Laravel. Để bắt đầu, hãy tạo checkout session [như minh họa trong các ví dụ trước](#inline-checkout):

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $user->checkout('pri_34567')
            ->returnTo(route('dashboard'));

        return view('billing', ['checkout' => $checkout]);
    });

Tiếp theo, bạn có thể sử dụng Paddle.js để khởi tạo thanh toán. Trong ví dụ này, chúng ta sẽ minh họa điều này bằng cách sử dụng [Alpine.js](https://github.com/alpinejs/alpine); tuy nhiên, bạn có thể tự do sửa ví dụ này cho giao diện người dùng của riêng bạn:

```blade
<?php
$options = $checkout->options();

$options['settings']['frameTarget'] = 'paddle-checkout';
$options['settings']['frameInitialHeight'] = 366;
?>

<div class="paddle-checkout" x-data="{}" x-init="
    Paddle.Checkout.open(@json($options));
">
</div>
```

<a name="guest-checkouts"></a>
### Guest Checkouts

Thỉnh thoảng, bạn có thể cần tạo một checkout session cho người dùng mà không cần lập tài khoản trong ứng dụng của bạn. Để thực hiện điều này, bạn có thể sử dụng phương thức `guest`:

    use Illuminate\Http\Request;
    use Laravel\Paddle\Checkout;

    Route::get('/buy', function (Request $request) {
        $checkout = Checkout::guest('pri_34567')
            ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });

Sau đó, bạn có thể cung cấp checkout session cho các Blade component [Paddle button](#overlay-checkout) hoặc component [inline checkout](#inline-checkout).

<a name="price-previews"></a>
## Price Previews

Paddle cho phép bạn tùy chỉnh giá dựa trên mỗi đơn vị tiền tệ, về cơ bản cho phép bạn cấu hình các mức giá khác nhau cho từng quốc gia khác nhau. Cashier Paddle cũng cho phép bạn lấy ra tất cả các những giá bằng phương thức `previewPrices`. Phương thức này chấp nhận các ID giá mà bạn muốn lấy ra:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::previewPrices(['pri_123', 'pri_456']);

Đơn vị tiền tệ sẽ được xác định dựa trên địa chỉ IP của request; tuy nhiên, bạn có thể tùy chọn cung cấp một quốc gia cụ thể để lấy ra giá cho sản phẩm đó:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices(['pri_123', 'pri_456'], ['address' => [
        'country_code' => 'BE',
        'postal_code' => '1234',
    ]]);

Sau khi lấy ra giá, bạn có thể hiển thị chúng theo cách bạn muốn:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

Bạn cũng có thể hiển thị giá thực (không bao gồm thuế) và hiển thị số tiền thuế một cách riêng biệt:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->subtotal() }} (+ {{ $price->tax() }} tax)</li>
    @endforeach
</ul>
```

Để biết thêm thông tin, hãy [xem tài liệu API của Paddle về price preview](https://developer.paddle.com/api-reference/pricing-preview/preview-prices).

<a name="customer-price-previews"></a>
### Customer Price Previews

Nếu người dùng đã là một customer và bạn muốn hiển thị giá áp dụng cho customer đó, bạn có thể làm như sau bằng cách lấy ra giá trực tiếp từ instance customer:

    use App\Models\User;

    $prices = User::find(1)->previewPrices(['pri_123', 'pri_456']);

Trong nội bộ, Cashier sẽ sử dụng customer ID của người dùng để lấy ra giá theo đơn vị tiền tệ của họ. Vì vậy, ví dụ: một người dùng sống ở Hoa Kỳ sẽ thấy giá bằng đô-la US trong khi người dùng ở Bỉ sẽ thấy giá bằng Euros. Nếu không tìm thấy đơn vị tiền tệ phù hợp, đơn vị tiền tệ mặc định của sản phẩm sẽ được sử dụng. Bạn có thể tùy chỉnh tất cả các mức giá của sản phẩm hoặc gói subscription trong bảng điều khiển Paddle.

<a name="price-discounts"></a>
### Discounts

Bạn cũng có thể chọn hiển thị giá sau khi discount. Khi gọi phương thức `previewPrices`, bạn cung cấp ID discount thông qua tùy chọn `discount_id`:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::previewPrices(['pri_123', 'pri_456'], [
        'discount_id' => 'dsc_123'
    ]);

Sau đó, hiển thị giá đã tính:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

<a name="customers"></a>
## Customers

<a name="customer-defaults"></a>
### Customer mặc định

Cashier cho phép bạn định nghĩa một số trường mặc định hữu ích cho khách hàng của bạn khi tạo checkout session. Việc set các giá trị mặc định này cho phép bạn điền trước địa chỉ email và name của khách hàng để họ có thể chuyển ngay sang phần thanh toán của giao diện thanh toán. Bạn có thể set các giá trị mặc định này bằng cách ghi đè các phương thức sau trong billable model của bạn:

    /**
     * Get the customer's name to associate with Paddle.
     */
    public function paddleName(): string|null
    {
        return $this->name;
    }

    /**
     * Get the customer's email address to associate with Paddle.
     */
    public function paddleEmail(): string|null
    {
        return $this->email;
    }

Các giá trị mặc định này sẽ được sử dụng cho mọi hành động trong Cashier khi tạo [checkout session](#checkout-sessions).

<a name="retrieving-customers"></a>
### Lấy customer

Bạn có thể lấy ra một khách hàng dựa theo ID khách hàng Paddle của họ thông qua phương thức `Cashier::findBilable`. Phương thức này sẽ trả về một instance của billable model:

    use Laravel\Cashier\Cashier;

    $user = Cashier::findBillable($customerId);

<a name="creating-customers"></a>
### Tạo customer

Thỉnh thoảng, bạn có thể muốn tạo một khách hàng Paddle mà không cần phải đăng ký subscription. Bạn có thể thực hiện việc này bằng phương thức `createAsCustomer`:

    $customer = $user->createAsCustomer();

Một instance của `Laravel\Paddle\Customer` sẽ được trả về. Sau khi khách hàng đã được tạo trong Paddle, bạn có thể bắt đầu đăng ký subscription sau. Bạn có thể cung cấp một mảng `$options` để truyền vào bất kỳ [tham số tạo khách hàng nào được API Paddle hỗ trợ](https://developer.paddle.com/api-reference/customers/create-customer):

    $customer = $user->createAsCustomer($options);

<a name="subscriptions"></a>
## Subscriptions

<a name="creating-subscriptions"></a>
### Tạo Subscription

Để tạo một subscription, trước tiên hãy lấy ra một instance billable model trong database của bạn, thường là một instance của `App\Models\User`. Khi bạn đã lấy được instance của model, bạn có thể sử dụng phương thức `subscribe` để tạo ra một checkout session cho model của bạn:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $checkout = $request->user()->subscribe($premium = 12345, 'default')
            ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });

Tham số đầu tiên được cung cấp cho phương thức `subscribe` là một giá cụ thể mà người dùng đang đăng ký. Giá này phải tương ứng với mã giá có trong Paddle. Phương thức `returnTo` sẽ chấp nhận một URL mà người dùng của bạn sẽ được chuyển đến sau khi họ hoàn tất thanh toán. Tham số thứ hai được truyền vào cho phương thức `subscribe` phải là tên "type" của subscription. Nếu ứng dụng của bạn chỉ cung cấp một loại subscription duy nhất, bạn có thể gọi nó là `default` hoặc `primary`. Type subscription này chỉ dành cho việc sử dụng ứng dụng nội bộ và không nhằm mục đích hiển thị cho người dùng. Ngoài ra, nó cũng không được chứa các khoảng trắng và nó không được thay đổi sau khi tạo subscription.

Bạn cũng có thể cung cấp một mảng meta data tùy chỉnh cho một subscription bằng phương thức `customData`:

    $checkout = $request->user()->subscribe($premium = 12345, 'default')
        ->customData(['key' => 'value'])
        ->returnTo(route('home'));

Sau khi subscription checkout session được tạo, checkout session có thể được cung cấp cho [Blade component](#overlay-checkout) `paddle-button` có sẵn trong Cashier Paddle:

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Sau khi người dùng hoàn tất quá trình thanh toán của họ, webhook `subscription_created` sẽ được gửi từ Paddle. Cashier sẽ nhận webhook này và thiết lập đăng ký cho khách hàng của bạn. Để đảm bảo ứng dụng của bạn nhận và xử lý được tất cả các webhook một đúng cách, hãy đảm bảo rằng bạn đã [thiết lập xử lý webhook](#handling-paddle-webhooks).

<a name="checking-subscription-status"></a>
### Kiểm tra trạng thái Subscription

Sau khi người dùng subscription vào ứng dụng của bạn, bạn có thể kiểm tra trạng thái subscription của họ bằng nhiều phương thức tiện lợi khác nhau. Đầu tiên, phương thức `subscribed` sẽ trả về `true` nếu người dùng có subscription tồn tại, ngay cả khi subscription hiện tại đang trong thời gian dùng thử:

    if ($user->subscribed()) {
        // ...
    }

Nếu ứng dụng của bạn cung cấp nhiều subscription, bạn có thể chỉ định subscription khi gọi phương thức `subscribed`:

    if ($user->subscribed('default')) {
        // ...
    }

Phương thức `subscribed` cũng là một ví dụ tốt cho một [route middleware](/docs/{{version}}/middleware), cho phép bạn chặn các quyền truy cập vào các route và controller mà dựa trên trạng thái subscription của người dùng:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class EnsureUserIsSubscribed
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if ($request->user() && ! $request->user()->subscribed()) {
                // This user is not a paying customer...
                return redirect('billing');
            }

            return $next($request);
        }
    }

Nếu bạn muốn xác định xem người dùng đó có còn trong thời gian dùng thử hay không, bạn có thể sử dụng phương thức `onTrial`. Phương thức này có thể hữu ích để xác định xem bạn có nên hiển thị cảnh báo cho người dùng biết rằng họ vẫn đang trong thời gian dùng thử:

    if ($user->subscription()->onTrial()) {
        // ...
    }

Phương thức `subscribedToPrice` có thể được sử dụng để xác định xem người dùng có đăng ký gói dịch vụ đã cho hay không dựa vào ID của gói trong Paddle. Trong ví dụ này, chúng ta sẽ xác định xem subscription `default` của người dùng có đăng ký gói `monthly` hay không:

    if ($user->subscribedToPrice($monthly = 'pri_123', 'default')) {
        // ...
    }

Phương thức `recurring` có thể được sử dụng để xác định xem người dùng hiện tại không còn trong thời gian dùng thử hoặc thời gian gia hạn và đang active subscription:

    if ($user->subscription()->recurring()) {
        // ...
    }

<a name="canceled-subscription-status"></a>
#### Canceled Subscription Status

Để xác định xem người dùng đã từng subscription nhưng sau đó đã hủy, bạn có thể sử dụng phương thức `canceled`:

    if ($user->subscription()->canceled()) {
        // ...
    }

Bạn cũng có thể xác định xem người dùng đã hủy subscription của họ hay chưa, hay vẫn còn trong "thời gian có hiệu lực" cho đến khi subscription hết hạn. Ví dụ: nếu người dùng hủy subscription vào ngày 5 tháng 3 mà dự kiến ban đầu là sẽ hết hạn vào ngày 10 tháng 3, thì người dùng sẽ ở trong "thời gian có hiệu lực" của họ cho đến ngày 10 tháng 3. Ngoài ra, phương thức `subscribed` vẫn trả về `true` trong thời gian này:

    if ($user->subscription()->onGracePeriod()) {
        // ...
    }

<a name="past-due-status"></a>
#### Past Due Status

Nếu một khoản thanh toán không thành công cho một subscription, nó sẽ được đánh dấu là `past_due`. Khi subscription của bạn ở trạng thái này, nó sẽ không hoạt động cho đến khi nào khách hàng cập nhật thông tin thanh toán của họ. Bạn có thể xác định xem một subscription có quá hạn hay không bằng cách sử dụng phương thức `pastDue` trên instance subscription:

    if ($user->subscription()->pastDue()) {
        // ...
    }

Khi một subscription quá hạn, bạn nên hướng dẫn người dùng [cập nhật lại thông tin thanh toán của họ](#updating-payment-information).

Nếu bạn muốn các subscription vẫn được coi là tồn tại khi chúng ở trạng thái `past_due`, thì bạn có thể sử dụng phương thức `keepPastDueSubscriptionsActive` do Cashier cung cấp. Thông thường, phương thức này nên được gọi trong phương thức `register` của `AppServiceProvider` của bạn:

    use Laravel\Paddle\Cashier;

    /**
     * Register any application services.
     */
    public function register(): void
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> [!WARNING]
> Khi một subscription ở trạng thái `past_due`, thì bạn không thể thay đổi subscription cho đến khi thông tin thanh toán được cập nhật. Do đó, các phương thức `swap` và `updateQuantity` sẽ tạo ra một exception khi subscription ở trạng thái `past_due`.

<a name="subscription-scopes"></a>
#### Subscription Scopes

Hầu hết các trạng thái subscription cũng có sẵn dưới dạng query scope giúp cho bạn có thể dễ dàng truy vấn dữ liệu subscription của bạn ở một trạng thái nhất định:

    // Get all valid subscriptions...
    $subscriptions = Subscription::query()->valid()->get();

    // Get all of the canceled subscriptions for a user...
    $subscriptions = $user->subscriptions()->canceled()->get();

Dưới đây là một danh sách đầy đủ các scope có sẵn:

    Subscription::query()->valid();
    Subscription::query()->onTrial();
    Subscription::query()->expiredTrial();
    Subscription::query()->notOnTrial();
    Subscription::query()->active();
    Subscription::query()->recurring();
    Subscription::query()->pastDue();
    Subscription::query()->paused();
    Subscription::query()->notPaused();
    Subscription::query()->onPausedGracePeriod();
    Subscription::query()->notOnPausedGracePeriod();
    Subscription::query()->canceled();
    Subscription::query()->notCanceled();
    Subscription::query()->onGracePeriod();
    Subscription::query()->notOnGracePeriod();

<a name="subscription-single-charges"></a>
### Subscription tính phí một lần

Các subscription tính phí một lần cho phép bạn tính phí người dùng với khoản phí một lần trên các gói subscription của họ. Bạn phải cung cấp một hoặc nhiều ID giá khi gọi phương thức `charge`:

    // Charge a single price...
    $response = $user->subscription()->charge('pri_123');

    // Charge multiple prices at once...
    $response = $user->subscription()->charge(['pri_123', 'pri_456']);

Phương thức `charge` sẽ không tính phí khách hàng cho đến thời gian thanh toán tiếp theo của subscription của họ. Nếu bạn muốn tính phí khách hàng ngay lập tức, bạn có thể sử dụng phương thức `chargeAndInvoice` để thay thế:

    $response = $user->subscription()->chargeAndInvoice('pri_123');

<a name="updating-payment-information"></a>
### Cập nhật thông tin thanh toán

Paddle luôn lưu một phương thức thanh toán cho mỗi subscription. Nếu bạn muốn cập nhật phương thức thanh toán mặc định cho một subscription, trước tiên bạn nên chuyển hướng khách hàng của bạn đến trang cập nhật phương thức thanh toán được Paddle cung cấp bằng phương thức `redirectToUpdatePaymentMethod` trên model subscription:

    use Illuminate\Http\Request;

    Route::get('/update-payment-method', function (Request $request) {
        $user = $request->user();

        return $user->subscription()->redirectToUpdatePaymentMethod();
    });

Khi người dùng cập nhật xong thông tin của họ, một webhook `subscription_updated` sẽ được gửi bởi Paddle và chi tiết subscription sẽ được cập nhật trong cơ sở dữ liệu của ứng dụng của bạn.

<a name="changing-plans"></a>
### Thay đổi gói

Sau khi một người dùng đã subscription vào application của bạn, đôi khi họ có thể muốn thay đổi sang gói subscription mới. Để cập nhật gói subscription cho một người dùng, hãy truyền vào một identifier của price mới của bên Paddle vào phương thức `swap`:

    use App\Models\User;

    $user = User::find(1);

    $user->subscription()->swap($premium = 'pri_456');

Nếu bạn muốn thay đổi gói và lập hóa đơn ngay cho người dùng thay vì đợi đến chu kỳ thanh toán tiếp theo của họ, bạn có thể sử dụng phương pháp `swapAndInvoice`:

    $user = User::find(1);

    $user->subscription()->swapAndInvoice($premium = 'pri_456');

<a name="prorations"></a>
#### Prorations

Mặc định, Paddle sẽ tính phí khi hoán đổi giữa các gói. Phương thức `noProrate` có thể được sử dụng để cập nhật nhiều subscription mà không bị tính phí:

    $user->subscription('default')->noProrate()->swap($premium = 'pri_456');

Nếu bạn muốn disable việc tính thêm phí và lập hóa đơn ngay cho khách hàng, bạn có thể sử dụng phương thức `swapAndInvoice` cùng với `noProrate`:

    $user->subscription('default')->noProrate()->swapAndInvoice($premium = 'pri_456');

Hoặc, để không tính phí khách hàng khi thay đổi subscription, bạn có thể sử dụng phương thức `doNotBill`:

    $user->subscription('default')->doNotBill()->swap($premium = 'pri_456');

Để biết thêm thông tin về chính sách của Paddle, vui lòng tham khảo [tài liệu](https://developer.paddle.com/concepts/subscriptions/proration) của Paddle.

<a name="subscription-quantity"></a>
### Subscription số lượng lớn

Thỉnh thoảng subscription có thể bị ảnh hưởng bởi "số lượng". Ví dụ: một application quản lý project có thể tính phí $10 mỗi tháng **cho mỗi người dùng** trên mỗi project. Để dễ dàng tăng hoặc giảm số lượng subscription của bạn, hãy sử dụng các phương thức `incrementQuantity` hoặc `decrementQuantity`:

    $user = User::find(1);

    $user->subscription()->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription()->incrementQuantity(5);

    $user->subscription()->decrementQuantity();

    // Subtract five from the subscription's current quantity...
    $user->subscription()->decrementQuantity(5);

Ngoài ra, bạn cũng có thể set một số lượng cụ thể bằng phương thức `updateQuantity`:

    $user->subscription()->updateQuantity(10);

Phương thức `noProrate` có thể được sử dụng để cập nhật số lượng của subscription mà không cần chia tỷ lệ phí:

    $user->subscription()->noProrate()->updateQuantity(10);

<a name="quantities-for-subscription-with-multiple-products"></a>
#### Quantities for Subscriptions With Multiple Products

Nếu subscription của bạn là một [subscription gồm có nhiều sản phẩm](#subscriptions-with-multiple-products), bạn phải truyền ID giá cho phương thức mà bạn muốn tăng hoặc giảm số lượng làm tham số thứ hai cho các phương thức tăng hoặc giảm:

    $user->subscription()->incrementQuantity(1, 'price_chat');

<a name="subscriptions-with-multiple-products"></a>
### Subscription với nhiều sản phẩm

[Subscription nhiều sản phẩm](https://developer.paddle.com/build/subscriptions/add-remove-products-prices-addons) cho phép bạn chỉ định nhiều sản phẩm cho một subscription duy nhất. Ví dụ, hãy tưởng tượng bạn đang xây dựng một ứng dụng "trợ giúp" khách hàng có giá subscription cơ bản là 10 đô la cho một tháng nhưng cung cấp thêm sản phẩm trò chuyện trực tiếp với giá 15 đô la một tháng.

Khi tạo subscription checkout, bạn có thể chỉ định nhiều sản phẩm cho một subscription nhất định bằng cách truyền một mảng giá làm tham số đầu tiên cho phương thức `subscribe`:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $checkout = $request->user()->subscribe([
            'price_monthly',
            'price_chat',
        ]);

        return view('billing', ['checkout' => $checkout]);
    });

Trong ví dụ trên, khách hàng sẽ có hai mức giá được thêm vào subscription `default` của họ. Cả hai mức giá này sẽ được tính theo các khoảng thời gian thanh toán tương ứng. Nếu cần, bạn có thể truyền một mảng các cặp khóa và giá trị để chỉ ra số lượng cụ thể cho mỗi mức giá:

    $user = User::find(1);

    $checkout = $user->subscribe('default', ['price_monthly', 'price_chat' => 5]);

Nếu bạn muốn thêm một mức giá khác vào một subscription hiện có, bạn phải sử dụng phương thức `swap` của subscription đó. Khi gọi phương thức `swap`, bạn cũng nên thêm cả giá và số lượng hiện tại của subscription đó:

    $user = User::find(1);

    $user->subscription()->swap(['price_chat', 'price_original' => 2]);

Ví dụ trên sẽ thêm giá mới, nhưng khách hàng sẽ không bị tính phí cho đến khi chu kỳ thanh toán tiếp theo của họ. Nếu bạn muốn tính phí cho khách hàng ngay lập tức, bạn có thể sử dụng phương thức `swapAndInvoice`:

    $user->subscription()->swapAndInvoice(['price_chat', 'price_original' => 2]);

Bạn có thể xóa giá ra khỏi subscription bằng phương thức `swap` và không thêm vào mức giá mà bạn muốn xóa bỏ:

    $user->subscription()->swap(['price_original' => 2]);

> [!WARNING]
> Bạn không thể xóa cái giá cuối cùng còn lại trông một subscription. Thay vào đó, bạn chỉ cần hủy subscription.

<a name="multiple-subscriptions"></a>
### Nhiều Subscriptions

Paddle cho phép khách hàng của bạn có thể subscription nhiều loại cùng một lúc. Ví dụ: bạn có thể đang điều hành một phòng gym cung cấp các gói đăng ký bơi và các gói đăng ký tập thể dục và mỗi gói đăng ký lại có thể có các mức giá khác nhau. Tất nhiên, khách hàng có thể đăng ký một hoặc cả hai gói.

Khi ứng dụng của bạn tạo các đăng ký, bạn có thể cung cấp type của đăng ký cho phương thức `subscribe` như là tham số thứ hai. Tên có thể là bất kỳ chuỗi nào mà đại diện cho loại đăng ký mà người dùng đang muốn sử dụng:

    use Illuminate\Http\Request;

    Route::post('/swimming/subscribe', function (Request $request) {
        $checkout = $request->user()->subscribe($swimmingMonthly = 'pri_123', 'swimming');

        return view('billing', ['checkout' => $checkout]);
    });

Trong ví dụ trên, chúng ta đã đăng ký bơi hàng tháng cho khách hàng. Nhưng, sau này có thể họ muốn chuyển sang đăng ký theo dạng hàng năm. Khi điều chỉnh đăng ký của khách hàng, chúng ta có thể chỉ cần hoán đổi giá của đăng ký `swimming`:

    $user->subscription('swimming')->swap($swimmingYearly = 'pri_456');

Tất nhiên, bạn cũng có thể hủy đăng ký:

    $user->subscription('swimming')->cancel();

<a name="pausing-subscriptions"></a>
### Tạm dừng Subscriptions

Để tạm dừng một subscription, hãy gọi phương thức `pause` trên subscription của người dùng:

    $user->subscription()->pause();

Khi một subscription bị tạm dừng, Cashier sẽ tự động set cột `paused_at` trong cơ sở dữ liệu của bạn. Cột này được sử dụng để xác định khi nào thì phương thức `paused` sẽ bắt đầu trả về `true`. Ví dụ: nếu một khách hàng tạm dừng subscription vào ngày 1 tháng 3, nhưng subscription đó có thời hạn đến ngày 5 tháng 3, thì phương thức `paused` sẽ tiếp tục trả về giá trị `false` cho đến hết ngày 5 tháng 3. Điều này được thực hiện là vì người dùng thường được phép tiếp tục sử dụng ứng dụng cho đến khi kết thúc chu kỳ thanh toán của họ.

Mặc định, việc tạm dừng này sẽ diễn ra ở thời điểm thanh toán tiếp theo của khách hàng để khách hàng có thể sử dụng phần còn lại của kỳ hạn của họ mà họ đã thanh toán. Nếu bạn muốn tạm dừng subscription ngay lập tức, bạn có thể sử dụng phương thức `pauseNow`:

    $user->subscription()->pauseNow();

Sử dụng phương thức `pauseUntil`, bạn có thể tạm dừng đăng ký cho đến một thời điểm cụ thể:

    $user->subscription()->pauseUntil(now()->addMonth());

Hoặc, bạn có thể sử dụng phương thức `pauseNowUntil` để tạm dừng đăng ký ngay lập tức cho đến một thời điểm nhất định:

    $user->subscription()->pauseNowUntil(now()->addMonth());

Bạn có thể xác định xem người dùng đã tạm dừng subscription của họ nhưng vẫn đang trong "thời gian có hạn subscription" bằng cách sử dụng phương thức `onPausedGracePeriod`:

    if ($user->subscription()->onPausedGracePeriod()) {
        // ...
    }

Để resume lại một subscription đã tạm dừng, bạn có thể gọi phương thức `resume` trên subscription của người dùng:

    $user->subscription()->resume();

> [!WARNING]
> Không thể sửa subscription khi nó đang bị tạm dừng. Nếu bạn muốn chuyển sang một gói khác hoặc cập nhật số lượng subscription, trước tiên bạn phải resume lại subscription.

<a name="canceling-subscriptions"></a>
### Huỷ Subscriptions

Để hủy một subscription, hãy gọi phương thức `cancel` trên subscription của người dùng:

    $user->subscription()->cancel();

Khi một subscription bị hủy, Cashier sẽ tự động set cột `ends_at` trong cơ sở dữ liệu của bạn. Cột này được sử dụng để xác định xem khi nào phương thức `subscribed` sẽ bắt đầu trả về `false`. Ví dụ: nếu khách hàng hủy subscription vào ngày 1 tháng 3, nhưng subscription không thể kết thúc cho đến khi hết ngày 5 tháng 3, thì phương thức `subscribed` vẫn sẽ tiếp tục trả về `true` cho đến ngày 5 tháng 3. Điều này được thực hiện là vì người dùng thường được phép tiếp tục sử dụng ứng dụng cho đến khi kết thúc chu kỳ thanh toán của họ.

Bạn có thể xác định những người dùng đã hủy subscription của họ nhưng vẫn đang trong "thời gian subscription có hiệu lực" bằng cách sử dụng phương thức `onGracePeriod`:

    if ($user->subscription()->onGracePeriod()) {
        // ...
    }

Nếu bạn muốn hủy subscription ngay lập tức, hãy gọi phương thức `cancelNow` trên subscription:

    $user->subscription()->cancelNow();

Để dừng việc hủy subscription trong thời gian gia hạn, bạn có thể sử dụng phương thức `stopCancelation`:

    $user->subscription()->stopCancelation();

> [!WARNING]
> Subscription của Paddle sẽ không thể resume sau khi nó bị hủy. Nếu khách hàng của bạn muốn resume lại subscription của họ, họ sẽ phải tạo một subscription mới.

<a name="subscription-trials"></a>
## Subscription dành cho dùng thử

<a name="with-payment-method-up-front"></a>
### Khai báo trước phương thức thanh toán

Nếu bạn muốn cung cấp thời gian dùng thử cho khách hàng của bạn trong khi vẫn muốn thu thập thông tin thanh toán của khách hàng, bạn nên set thời gian dùng thử trong bảng điều khiển Paddle theo mức giá mà khách hàng của bạn đang đăng ký. Sau đó, bắt đầu thanh toán như bình thường:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $checkout = $request->user()->subscribe('pri_monthly')
                    ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });

Khi ứng dụng của bạn nhận được event `subscription_created`, Cashier sẽ set ngày kết thúc của thời gian dùng thử vào trong bản ghi subscription trong cơ sở dữ liệu của application của bạn và sẽ bảo với Paddle là sẽ không tính phí khách hàng cho đến khi hết ngày dùng thử.

> [!WARNING]
> Nếu subscription của khách hàng không bị hủy trước ngày kết thúc dùng thử, họ sẽ bị tính phí ngay khi hết hạn dùng thử, vì vậy bạn nên chắc chắn là đã thông báo cho khách hàng biết về ngày kết thúc dùng thử của họ.

Bạn có thể xác định xem người dùng hiện tại có đang trong thời gian dùng thử hay không bằng cách sử dụng phương thức `onTrial` trên instance người dùng hoặc phương thức `onTrial` trên instance subscription. Hai ví dụ dưới đây là giống nhau:

    if ($user->onTrial()) {
        // ...
    }

    if ($user->subscription()->onTrial()) {
        // ...
    }
Để xác định xem bản dùng thử hiện tại đã hết hạn hay chưa, bạn có thể sử dụng phương thức `hasExpiredTrial`:

    if ($user->hasExpiredTrial()) {
        // ...
    }

    if ($user->subscription()->hasExpiredTrial()) {
        // ...
    }

Để xác định xem một người dùng có đang dùng thử một loại đăng ký nào không, bạn có thể cung cấp loại đăng ký đó cho phương thức `onTrial` hoặc `hasExpiredTrial`:

    if ($user->onTrial('default')) {
        // ...
    }

    if ($user->hasExpiredTrial('default')) {
        // ...
    }

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

Khi bạn đã sẵn sàng tạo một subscription thực sự cho người dùng, bạn có thể sử dụng phương thức `subscribe` như bình thường:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $checkout = $user->subscribe('pri_monthly')
            ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });

Để lấy ra ngày kết thúc dùng thử của một người dùng, bạn có thể sử dụng phương thức `trialEndsAt`. Phương thức này sẽ trả về một instance date Carbon nếu người dùng đang dùng thử hoặc `null` nếu họ không đang dùng thử. Bạn cũng có thể truyền một tham số loại subscription tùy chọn nếu bạn muốn biết ngày kết thúc dùng thử cho một subscription cụ thể khác với subscription mặc định:

    if ($user->onTrial('default')) {
        $trialEndsAt = $user->trialEndsAt();
    }

Bạn có thể sử dụng phương thức `onGenericTrial` nếu bạn muốn biết cụ thể rằng người dùng đang trong thời gian dùng thử "generic" và chưa tạo subscription thực tế:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

<a name="extend-or-activate-a-trial"></a>
### Gia hạn và kích hoạt dùng thử

Bạn có thể gia hạn thời gian dùng thử hiện tại trên một subscription bằng cách sử dụng phương thức `extendTrial` và chỉ định thời điểm kết thúc của gói dùng thử:

    $user->subsription()->extendTrial(now()->addDays(5));

Hoặc, bạn có thể kích hoạt ngay đăng ký bằng cách kết thúc thời gian dùng thử thông qua phương thức `activate` trên subscription:

    $user->subscription()->activate();

<a name="handling-paddle-webhooks"></a>
## Xử lý Paddle Webhooks

Paddle có thể thông báo cho ứng dụng của bạn về nhiều event thông qua webhook. Mặc định, một route sẽ trỏ đến một controller webhook của Cashier được đăng ký bởi service provider của Cashier. Controller này sẽ xử lý tất cả các request webhook gửi đến.

Mặc định, controller này sẽ tự động xử lý việc hủy đăng ký khi có quá nhiều khoản phí không thành công, cập nhật đăng ký và thay đổi phương thức thanh toán ; tuy nhiên, như bạn sẽ sớm khám phá ra rằng bạn có thể mở rộng controller này để xử lý bất kỳ sự kiện webhook nào của Paddle mà bạn muốn.

Để đảm bảo ứng dụng của bạn có thể xử lý Paddle webhook, hãy nhớ [cấu hình URL webhook trong bảng điều khiển Paddle](https://vendors.paddle.com/alerts-webhooks). Mặc định, webhook controller của Cashier sẽ response trên đường dẫn URL là `/paddle/webhook`. Danh sách đầy đủ của tất cả các webhook mà bạn nên bật trong bảng điều khiển Paddle sẽ là:

- Customer Updated
- Transaction Completed
- Transaction Updated
- Subscription Created
- Subscription Updated
- Subscription Paused
- Subscription Canceled

> [!WARNING]
> Hãy đảm bảo là bạn đã bảo vệ các request bằng các middleware [kiểm tra định dạng webhook](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures) của Cashier.

<a name="webhooks-csrf-protection"></a>
#### Webhooks và CSRF Protection

Vì các webhook của Paddle cần bỏ qua bước [bảo vệ CSRF](/docs/{{version}}/csrf) của Laravel, nên bạn hãy chắc chắn là đã khai báo URI của Paddle là một ngoại lệ trong middleware `App\Http\Middleware\VerifyCsrfToken` của bạn hoặc bạn có thể khai báo route này ra khỏi group middleware `web`:

    protected $except = [
        'paddle/*',
    ];

<a name="webhooks-local-development"></a>
#### Webhooks và Local Development

Để Paddle có thể gửi webhook cho ứng dụng của bạn trong quá trình phát triển ở local, bạn cần chia sẻ ứng dụng của bạn lên các dịch vụ chia sẻ trang web, chẳng hạn như [Ngrok](https://ngrok.com/) hoặc [Expose](https://expose.dev/docs/introduction). Nếu bạn đang phát triển ứng dụng của bạn bằng cách sử dụng [Laravel Sail](/docs/{{version}}/sail), thì bạn có thể sử dụng [lệnh chia sẻ trang web](/docs/{{version}}/sail#sharing-your-site) của Sail.

<a name="defining-webhook-event-handlers"></a>
### Định nghĩa xử lý Webhook Event

Cashier sẽ tự động xử lý hủy subscription nếu như các lần trả phí không thành công và các webhook Paddle phổ biến khác. Tuy nhiên, nếu bạn  muốn xử lý thêm các sự kiện webhook khác, thì bạn có thể làm như vậy bằng cách listening các event sau do Cashier gửi đi:

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

Cả hai event đều chứa toàn bộ payload của webhook Paddle. Ví dụ: nếu bạn muốn xử lý webhook `transaction_billed`, thì bạn có thể đăng ký [listener](/docs/{{version}}/events#defining-listeners) sẽ xử lý event đó:

    <?php

    namespace App\Listeners;

    use Laravel\Paddle\Events\WebhookReceived;

    class PaddleEventListener
    {
        /**
         * Handle received Paddle webhooks.
         */
        public function handle(WebhookReceived $event): void
        {
            if ($event->payload['alert_name'] === 'transaction_billed') {
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

- `Laravel\Paddle\Events\CustomerUpdated`
- `Laravel\Paddle\Events\TransactionCompleted`
- `Laravel\Paddle\Events\TransactionUpdated`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionPaused`
- `Laravel\Paddle\Events\SubscriptionCanceled`

</div>

Bạn cũng có thể ghi đè route webhook mặc định bằng cách định nghĩa biến môi trường `CASHIER_WEBHOOK` trong file `.env` của application của bạn. Giá trị này phải là một URL đầy đủ cho route webhook của bạn và cần giống với URL mà được set trong bảng điều khiển Paddle của bạn:

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### Kiểm tra định dạng Webhook

Để bảo mật webhook của bạn, bạn có thể sử dụng [định dạng webhook của Paddle](https://developer.paddle.com/webhook-reference/verifying-webhooks). Để thuận tiện, Cashier đã tự động thêm một middleware để kiểm tra các request webhook Paddle đến application là hợp lệ.

Để bật kiểm tra webhook, hãy đảm bảo rằng biến môi trường `PADDLE_WEBHOOK_SECRET` được định nghĩa trong file `.env` của application của bạn. Webhook secret này có thể được lấy ra từ trang tổng quan trong tài khoản Paddle của bạn.

<a name="single-charges"></a>
## Tính phí một lần

<a name="charging-for-products"></a>
### Tính phí sản phẩm

Nếu bạn muốn khởi tạo một sản phẩm mua cho một khách hàng, bạn có thể sử dụng phương thức `checkout` trên instance billable model để tạo một checkout session cho việc mua hàng. Phương thức `checkout` sẽ chấp nhận một hoặc nhiều ID giá. Nếu cần, bạn có thể cung cấp một mảng về số lượng sản phẩm đang được mua:

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $request->user()->checkout(['pri_tshirt', 'pri_socks' => 5]);

        return view('buy', ['checkout' => $checkout]);
    });

Sau khi đã tạo checkout session, bạn có thể sử dụng [Blade component](#overlay-checkout) `paddle-button` do Cashier cung cấp để cho phép người dùng xem checkout Paddle và hoàn thành purchase đó:

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy
</x-paddle-button>
```

Một checkout session sẽ có một phương thức `customData`, cho phép bạn truyền bất kỳ các custom data nào mà bạn muốn để tạo transaction. Vui lòng tham khảo [tài liệu của Paddle](https://developer.paddle.com/build/transactions/custom-data) để tìm hiểu thêm về các tùy chọn có sẵn cho bạn khi truyền custom data:

    $checkout = $user->checkout('pri_tshirt')
        ->customData([
            'custom_option' => $value,
        ]);

<a name="refunding-transactions"></a>
### Hoàn tiền giao dịch

Transaction hoàn tiền sẽ hoàn trả lại số tiền đã thanh toán vào phương thức thanh toán của khách hàng đã sử dụng tại thời điểm mua hàng. Nếu bạn cần hoàn một Paddle purchase, bạn có thể sử dụng phương thức `refund` trên model `Cashier\Paddle\Transaction`. Phương thức này chấp nhận lý do làm tham số đầu tiên, một hoặc nhiều ID giá để hoàn lại với số tiền tùy chọn dưới dạng mảng. Bạn có thể lấy ra các giao dịch của một billable model nhất định bằng phương thức `transactions`.

Ví dụ, hãy tưởng tượng chúng ta muốn hoàn lại một giao dịch cho giá `pri_123` và `pri_456`. Chúng ta muốn hoàn lại toàn bộ `pri_123`, nhưng chỉ hoàn lại hai đô la cho `pri_456`:

    use App\Models\User;

    $user = User::find(1);

    $transaction = $user->transactions()->first();

    $response = $transaction->refund('Accidental charge', [
        'pri_123', // Fully refund this price...
        'pri_456' => 200, // Only partially refund this price...
    ]);

Ví dụ trên sẽ hoàn lại các item có trong một giao dịch. Nếu bạn muốn hoàn lại toàn bộ giao dịch, chỉ cần cung cấp lý do:

    $response = $transaction->refund('Accidental charge');

Để biết thêm thông tin về việc hoàn tiền, vui lòng tham khảo [tài liệu hoàn tiền của Paddle](https://developer.paddle.com/build/transactions/create-transaction-adjustments).

> [!WARNING]
> Việc hoàn tiền phải luôn được Paddle chấp thuận trước khi xử lý.

<a name="crediting-transactions"></a>
### Credit transaction

Cũng giống như hoàn tiền, bạn cũng có thể tạo credit transaction. Credit transaction sẽ thêm tiền vào số dư của khách hàng để có thể sử dụng cho các giao dịch mua trong tương lai. Credit transaction chỉ có thể được thực hiện đối với các giao dịch thủ công chứ không cho các giao dịch tự động (như subscription) vì Paddle tự động xử lý các subscription credit:

    $transaction = $user->transactions()->first();

    // Credit a specific line item fully...
    $response = $transaction->credit('Compensation', 'pri_123');

Để biết thêm thông tin, hãy [xem tài liệu về credit của Paddle](https://developer.paddle.com/build/transactions/create-transaction-adjustments).

> [!WARNING]
> Chỉ có thể áp dụng credit cho các giao dịch thủ công. Các giao dịch tự động sẽ được credit có bởi chính Paddle.

<a name="transactions"></a>
## Giao dịch

Bạn có thể dễ dàng lấy ra một mảng các transaction của một model billable bằng cách sử dụng thuộc tính `transactions`:

    use App\Models\User;

    $user = User::find(1);

    $transactions = $user->transactions;

Transaction sẽ đại diện cho các khoản thanh toán cho một sản phẩm nào đó hoặc giao dịch mua của bạn và được kèm theo hóa đơn. Chỉ những giao dịch đã hoàn tất mới được lưu vào trong cơ sở dữ liệu của ứng dụng.

Khi liệt kê các transaction cho một khách hàng, bạn có thể sử dụng các phương thức của instance transaction để hiển thị các thông tin liên quan về payment. Ví dụ: bạn có thể muốn liệt kê mọi transaction vào trong một bảng, và cho phép người dùng dễ dàng tải xuống bất kỳ invoice nào:

```html
<table>
    @foreach ($transactions as $transaction)
        <tr>
            <td>{{ $transaction->billed_at->toFormattedDateString() }}</td>
            <td>{{ $transaction->total() }}</td>
            <td>{{ $transaction->tax() }}</td>
            <td><a href="{{ route('download-invoice', $transaction->id) }}" target="_blank">Download</a></td>
        </tr>
    @endforeach
</table>
```

Route `download-invoice` có thể trông như sau:

    use Illuminate\Http\Request;
    use Laravel\Cashier\Transaction;

    Route::get('/download-invoice/{transaction}', function (Request $request, Transaction $transaction) {
        return $transaction->redirectToInvoicePdf();
    })->name('download-invoice');

<a name="past-and-upcoming-payments"></a>
### Các khoản thanh toán quá khứ và tương lai

Bạn có thể sử dụng phương thức `lastPayment` và `nextPayment` để nhận về và hiển thị các khoản thanh toán quá khứ hoặc tương lai của khách hàng cho các subscription định kỳ:

    use App\Models\User;

    $user = User::find(1);

    $subscription = $user->subscription();

    $lastPayment = $subscription->lastPayment();
    $nextPayment = $subscription->nextPayment();

Cả hai phương thức này sẽ trả về một instance của `Laravel\Paddle\Payment`; tuy nhiên, `lastPayment` sẽ trả về `null` khi các giao dịch chưa được đồng bộ bởi webhook, trong khi `nextPayment` sẽ trả về `null` nếu chu kỳ thanh toán đã kết thúc (chẳng hạn như khi subscription bị hủy):

```blade
Next payment: {{ $nextPayment->amount() }} due on {{ $nextPayment->date()->format('d/m/Y') }}
```

<a name="testing"></a>
## Testing

Trong khi testing, bạn nên test luồng thanh toán của bạn bằng cách thủ công để đảm bảo các tích hợp của bạn hoạt động như mong đợi.

Đối với các automated test, chứa cả những bài test được thực hiện trong môi trường CI, bạn có thể sử dụng [HTTP Client của Laravel](/docs/{{version}}/http-client#testing) để fake các request HTTP được thực hiện tới Paddle. Mặc dù điều này không kiểm tra các phản hồi thực tế từ Paddle, nhưng nó cũng cung cấp một cách để kiểm tra ứng dụng của bạn mà không thực sự gọi đến API của Paddle.
