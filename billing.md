# Laravel Cashier

- [Giới thiệu](#introduction)
- [Cập nhật Cashier](#upgrading-cashier)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
    - [Billable Model](#billable-model)
    - [API Keys](#api-keys)
    - [Cấu hình loại tiền](#currency-configuration)
    - [Logging](#logging)
- [Customers](#customers)
    - [Lấy Customers](#retrieving-customers)
    - [Tạo Customers](#creating-customers)
    - [Cập nhật Customers](#updating-customers)
    - [Tuỳ biến địa chỉ mail](#custom-email-addresses)
- [Phương thức thanh toán](#payment-methods)
    - [Lưu phương thức thanh toán](#storing-payment-methods)
    - [Lấy phương thức thanh toán](#retrieving-payment-methods)
    - [Xác định người dùng có phương thức thanh toán hay không](#check-for-a-payment-method)
    - [Cập nhật phương thức Thanh toán mặc định](#updating-the-default-payment-method)
    - [Thêm phương thức thanh toán](#adding-payment-methods)
    - [Xoá phương thức thanh toán](#deleting-payment-methods)
- [Subscription](#subscriptions)
    - [Tạo Subscription](#creating-subscriptions)
    - [Kiểm tra trạng thái Subscription](#checking-subscription-status)
    - [Thay đổi gói](#changing-plans)
    - [Subscription số lượng lớn](#subscription-quantity)
    - [Thuế của Subscription](#subscription-taxes)
    - [Subscription cố định ngày](#subscription-anchor-date)
    - [Huỷ Subscription](#cancelling-subscriptions)
    - [Resume Subscription](#resuming-subscriptions)
- [Subscription dành cho dùng thử](#subscription-trials)
    - [Khai báo trước phương thức thanh toán](#with-payment-method-up-front)
    - [Khai báo phương thức thanh toán sau](#without-payment-method-up-front)
    - [Mở rộng thời gian dùng thử](#extending-trials)
- [Xử lý Stripe Webhooks](#handling-stripe-webhooks)
    - [Định nghĩa xử lý Webhook Event](#defining-webhook-event-handlers)
    - [Subscription bị thất bại](#handling-failed-subscriptions)
    - [Kiểm tra Webhook Signatures](#verifying-webhook-signatures)
- [Phí](#single-charges)
    - [Tính phí một lần](#simple-charge)
    - [Tính phí với hoá đơn](#charge-with-invoice)
    - [Hoàn trả](#refunding-charges)
- [Hoá đơn](#invoices)
    - [Tạo hoá đơn PDF](#generating-invoice-pdfs)
- [Strong Customer Authentication (SCA)](#strong-customer-authentication)
    - [Thanh toán yêu cầu xác nhận bổ sung](#payments-requiring-additional-confirmation)
    - [Thông báo thanh toán Off-session](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)

<a name="introduction"></a>
## Giới thiệu

Laravel Cashier cung cấp một interface dễ hiểu, rõ ràng cho các dịch vụ thanh toán subscription trực tuyến như [Stripe's](https://stripe.com). Nó gần như đã xử lý tất cả các đoạn code mà bạn đang sợ viết mà có liên quan đến các phần thanh toán subscription. Ngoài quản lý subscription cơ bản, Cashier cũng có thể xử lý cả các phiếu giảm giá, chuyển đổi subscription, đăng ký "nhiều" subscription, thời hạn hủy bỏ và thậm chí là tạo các file hóa đơn PDF.

<a name="upgrading-cashier"></a>
## Cập nhật Cashier

Khi nâng cấp lên phiên bản mới của Cashier, điều quan trọng là bạn phải xem kỹ [hướng dẫn nâng cấp](https://github.com/laravel/cashier/blob/master/UPGRADE.md).

> {note} Để tránh các thay đổi nghiêm trọng, Cashier sẽ sử dụng một phiên bản API Stripe cố định. Cashier 10.1 sẽ sử dụng phiên bản API Stripe `2019-08-14`. Phiên bản API Stripe này sẽ được cập nhật thành các bản phát hành nhỏ để sử dụng các tính năng và cải tiến mới của Stripe.

<a name="installation"></a>
## Cài đặt

Đầu tiên, thêm package Cashier cho Stripe với Composer:

    composer require laravel/cashier

> {note} Để đảm bảo Cashier xử lý đúng tất cả các event của Stripe, hãy nhớ [thiết lập xử lý webhook của Cashier](#handling-stripe-webhooks).

#### Database Migrations

Service provider của Cashier sẽ đăng ký thư mục migration database của chính nó, vì vậy hãy nhớ migration cơ sở dữ liệu của bạn sau khi cài đặt package. Việc migration Cashier sẽ thêm một số cột vào bảng `users` của bạn cũng như tạo một bảng `subscriptions` mới để chứa tất cả các đăng ký của khách hàng của bạn:

    php artisan migrate

Nếu bạn cần ghi đè các migration đi kèm với package Cashier, bạn có thể export chúng bằng lệnh Artisan `vendor:publish`:

    php artisan vendor:publish --tag="cashier-migrations"

Nếu bạn muốn ngăn việc migration của Cashier chạy, bạn có thể sử dụng phương thức `ignoreMigrations` được Cashier cung cấp. Thông thường, phương thức này nên được gọi trong phương thức `register` trong `AppServiceProvider` của bạn:

    use Laravel\Cashier\Cashier;

    Cashier::ignoreMigrations();

> {note} Stripe khuyến cáo rằng bất kỳ cột nào được sử dụng để lưu trữ Stripe identifier phải phân biệt giữa chữ hoa và chữ thường. Do đó, bạn nên đảm bảo collation cho cột `stripe_id` được set thành, ví dụ: `utf8_bin` trong MySQL. Có thể hiểu thêm thông tin [trong tài liệu Stripe](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible).

<a name="configuration"></a>
## Cấu hình

<a name="billable-model"></a>
### Billable Model

Trước khi sử dụng Cashier, hãy thêm trait `Billable` vào định nghĩa model của bạn. Trait này sẽ cung cấp các phương thức khác nhau cho phép bạn thực hiện các tác vụ thanh toán phổ biến, chẳng hạn như tạo subscription, áp dụng phiếu giảm giá hoặc cập nhật thông tin phương thức thanh toán:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Cashier sẽ giả định rằng billable model của bạn sẽ là class `App\User` đi kèm với Laravel. Nếu bạn muốn thay đổi điều này, bạn có thể chỉ định một model khác trong file `.env` của bạn:

    CASHIER_MODEL=App\User

> {note} Nếu bạn đang sử dụng model khác, khác với model `App\User` do Laravel cung cấp, bạn sẽ cần export và thay đổi [migration](#installation) được cung cấp để khớp với tên bảng của model thay thế mà bạn muốn.

<a name="api-keys"></a>
### API Keys

Tiếp theo, bạn nên cấu hình key của Stripe trong file `.env` của bạn. Bạn có thể lấy khóa API Stripe của bạn từ bảng điều khiển của Stripe.

    STRIPE_KEY=your-stripe-key
    STRIPE_SECRET=your-stripe-secret

<a name="currency-configuration"></a>
### Cấu hình loại tiền

Đơn vị tiền mặc định của Cashier là Đô la Mỹ (USD). Bạn có thể thay đổi loại tiền mặc định này bằng cách set biến môi trường `CASHIER_CURRENCY`:

    CASHIER_CURRENCY=eur

Ngoài việc cấu hình đơn vị tiền tệ của Cashier, bạn cũng có thể chỉ định ngôn ngữ được sử dụng khi định dạng tiền tệ để hiển thị trong hóa đơn. Cashier sử dụng [class `NumberFormatter` của PHP](https://www.php.net/manual/en/class.numberformatter.php) để set ngôn ngữ tiền tệ:

    CASHIER_CURRENCY_LOCALE=nl_BE

> {note} Để sử dụng các ngôn ngữ khác, khác với ngôn ngữ `en`, hãy đảm bảo là extension của PHP `ext-intl` đã được cài đặt và cấu hình trên server của bạn.

<a name="logging"></a>
#### Logging

Cashier cho phép bạn chỉ định channel log sẽ được sử dụng khi ghi log tất cả các exception liên quan đến Stripe. Bạn có thể chỉ định channel log này bằng cách sử dụng biến môi trường `CASHIER_LOGGER`:

    CASHIER_LOGGER=stack

<a name="customers"></a>
## Customers

<a name="retrieving-customers"></a>
### Lấy Customers

Bạn có thể lấy ra một khách hàng bằng ID Stripe của họ thông qua phương thức `Cashier::findBillable`. Điều này sẽ trả về một instance của Billable model:

    use Laravel\Cashier\Cashier;

    $user = Cashier::findBillable($stripeId);

<a name="creating-customers"></a>
### Tạo Customers

Đôi khi, bạn có thể muốn tạo một Stripe customer mà không cần phải đăng ký. Bạn có thể thực hiện việc này bằng phương thức `createAsStripeCustomer`:

    $stripeCustomer = $user->createAsStripeCustomer();

Khi customer đã được tạo trong Stripe, bạn có thể bắt đầu subscription. Bạn cũng có thể sử dụng mảng tùy chọn `$options` để truyền vào bất kỳ tham số bổ sung nào mà được hỗ trợ bởi API Stripe:

    $stripeCustomer = $user->createAsStripeCustomer($options);

Bạn cũng có thể sử dụng phương thức `createOrGetStripeCustomer` nếu bạn muốn trả về một đối tượng customer nếu đối tượng đó đã là một customer trong Stripe.

    $stripeCustomer = $user->createOrGetStripeCustomer();

<a name="updating-customers"></a>
### Cập nhật Customers

Đôi khi, bạn có thể muốn cập nhật trực tiếp thông tin cho customer của Stripe. Bạn có thể thực hiện việc này bằng phương pháp `updateStripeCustomer`:

    $stripeCustomer = $user->updateStripeCustomer($options);

<a name="custom-email-addresses"></a>
### Tuỳ biến địa chỉ mail

Mặc định, Cashier sẽ sử dụng thuộc tính `email` trên Billable model của bạn để tạo customer trong Stripe. Bạn có thể ghi đè điều này bằng cách sử dụng phương thức `stripeEmail`:

    /**
     * Get the email address used to create the customer in Stripe.
     *
     * @return string|null
     */
    public function stripeEmail()
    {
        return $this->email;
    }

Bạn cũng có thể chọn trả về giá trị `null` nếu địa chỉ email là không cần thiết để tạo một customer trong Stripe. Nếu bạn không cung cấp địa chỉ email, các tính năng trong Stripe như dunning email, email thanh toán không thành công và các tính năng liên quan đến email khác sẽ không khả dụng.

<a name="payment-methods"></a>
## Phương thức thanh toán

<a name="storing-payment-methods"></a>
### Lưu phương thức thanh toán

Để tạo một subscription hoặc thực hiện tính phí "một lần" với Stripe, bạn sẽ cần lưu phương thức thanh toán và lấy identifier của phương thức đó từ Stripe. Cách tiếp cận được sử dụng theo mục đích khác nhau dựa trên việc bạn sẽ sử dụng phương thức thanh toán này cho việc thanh toán các subscription hay các khoản phí một lần, vì vậy chúng ta sẽ xem xét cả hai ví dụ bên dưới.

#### Phương thức thanh toán cho Subscription

Khi lưu trữ thẻ tín dụng cho khách hàng để sử dụng trong tương lai, API Setup Intent của Stripe sẽ phải được sử dụng để thu thập thông tin chi tiết về phương thức thanh toán của khách hàng. "Setup Intent" cho biết ý định tính phí phương thức thanh toán của khách hàng. Trait `Billable` của Cashier có chứa một `createSetupIntent` để dễ dàng tạo một Setup Intent mới. Bạn nên gọi phương thức này từ route hoặc controller sẽ hiển thị form thu thập chi tiết phương thức thanh toán cho khách hàng của bạn:

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

Sau khi bạn đã tạo xong Setup Intent và truyền nó đến view, bạn nên gắn secret của intent đó vào trong element sẽ thu thập phương thức thanh toán. Ví dụ: hãy xem xét form "cập nhật phương thức thanh toán" này:

    <input id="card-holder-name" type="text">

    <!-- Stripe Elements Placeholder -->
    <div id="card-element"></div>

    <button id="card-button" data-secret="{{ $intent->client_secret }}">
        Update Payment Method
    </button>

Tiếp theo, thư viện Stripe.js có thể sử dụng element đó để gắn các element Stripe cần thiết vào form và thu thập chi tiết thanh toán của khách hàng một cách an toàn:

    <script src="https://js.stripe.com/v3/"></script>

    <script>
        const stripe = Stripe('stripe-public-key');

        const elements = stripe.elements();
        const cardElement = elements.create('card');

        cardElement.mount('#card-element');
    </script>

Tiếp theo, thẻ có thể được xác minh và một "identifier cho phương thức thanh toán" an toàn có thể được lấy ra từ Stripe bằng cách sử dụng [phương thức `confirmCardSetup` của Stripe](https://stripe.com/docs/js/setup_intents/confirm_card_setup):

    const cardHolderName = document.getElementById('card-holder-name');
    const cardButton = document.getElementById('card-button');
    const clientSecret = cardButton.dataset.secret;

    cardButton.addEventListener('click', async (e) => {
        const { setupIntent, error } = await stripe.confirmCardSetup(
            clientSecret, {
                payment_method: {
                    card: cardElement,
                    billing_details: { name: cardHolderName.value }
                }
            }
        );

        if (error) {
            // Display "error.message" to the user...
        } else {
            // The card has been verified successfully...
        }
    });

Sau khi thẻ đã được Stripe xác minh, bạn có thể truyền kết quả identifier `setupIntent.payment_method` vào ứng dụng Laravel của bạn, nơi nó có thể được lưu với thông tin khách hàng. Phương thức thanh toán này có thể được [thêm vào như là một phương thức thanh toán mới](#adding-payment-methods) hoặc [được sử dụng để cập nhật một phương thức thanh toán mặc định](#updating-the-default-payment-method). Bạn cũng có thể sử dụng ngay identifier phương thức thanh toán này để [tạo ra một subscription mới](#creating-subscriptions).

> {tip} Nếu bạn muốn biết thêm thông tin về Setup Intent và cách thu thập chi tiết thanh toán của khách hàng, vui lòng [xem lại tài liệu tổng quan do Stripe cung cấp](https://stripe.com/docs/payments/save-and-reuse#php).

#### Phương thức thanh toán cho phí

Tất nhiên, khi thực hiện một khoản tính phí một lần đối với một phương thức thanh toán của khách hàng, chúng ta sẽ chỉ cần sử dụng một identifier phương thức thanh toán trong một lần duy nhất. Do các giới hạn của Stripe, bạn không thể sử dụng phương thức thanh toán mặc định của khách hàng cho các khoản tính phí một lần. Bạn phải cho phép khách hàng nhập chi tiết phương thức thanh toán của họ bằng thư viện Stripe.js. Ví dụ, hãy xem xét form sau:

    <input id="card-holder-name" type="text">

    <!-- Stripe Elements Placeholder -->
    <div id="card-element"></div>

    <button id="card-button">
        Process Payment
    </button>

Tiếp theo, thư viện Stripe.js có thể sử dụng element đó để gắn các element Stripe cần thiết vào form và thu thập chi tiết thanh toán của khách hàng một cách an toàn:

    <script src="https://js.stripe.com/v3/"></script>

    <script>
        const stripe = Stripe('stripe-public-key');

        const elements = stripe.elements();
        const cardElement = elements.create('card');

        cardElement.mount('#card-element');
    </script>

Tiếp theo, thẻ có thể được xác minh và một "identifier cho phương thức thanh toán" an toàn có thể được lấy ra từ Stripe bằng cách sử dụng [phương thức `createPaymentMethod` của Stripe](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method):

    const cardHolderName = document.getElementById('card-holder-name');
    const cardButton = document.getElementById('card-button');

    cardButton.addEventListener('click', async (e) => {
        const { paymentMethod, error } = await stripe.createPaymentMethod(
            'card', cardElement, {
                billing_details: { name: cardHolderName.value }
            }
        );

        if (error) {
            // Display "error.message" to the user...
        } else {
            // The card has been verified successfully...
        }
    });

Nếu thẻ được xác minh thành công, bạn có thể truyền `paymentMethod.id` vào ứng dụng Laravel của bạn và xử lý tiếp [các khoản tính phí một lần](#simple-charge).

<a name="retrieving-payment-methods"></a>
### Lấy phương thức thanh toán

Phương thức `PaymentMethods` trên instance Billable model sẽ trả về một collection các instance `Laravel\Cashier\PaymentMethod`:

    $paymentMethods = $user->paymentMethods();

Để lấy phương thức thanh toán mặc định, phương thức `defaultPaymentMethod` có thể được sử dụng:

    $paymentMethod = $user->defaultPaymentMethod();

Bạn cũng có thể lấy ra một phương thức thanh toán cụ thể thuộc sở hữu của một Billable model bằng cách sử dụng phương thức `findPaymentMethod`:

    $paymentMethod = $user->findPaymentMethod($paymentMethodId);

<a name="check-for-a-payment-method"></a>
### Xác định người dùng có phương thức thanh toán hay không

Để xác định xem một Billable model có phương thức thanh toán nào được gắn với tài khoản của họ hay không, hãy sử dụng phương thức `hasPaymentMethod`:

    if ($user->hasPaymentMethod()) {
        //
    }

<a name="updating-the-default-payment-method"></a>
### Cập nhật phương thức thanh toán mặc định

Phương thức `updateDefaultPaymentMethod` có thể được sử dụng để cập nhật thông tin về phương thức thanh toán mặc định của khách hàng. Phương thức này chấp nhận một identifier phương thức thanh toán của Stripe và sẽ gắn phương thức thanh toán mới làm phương thức thanh toán hóa đơn mặc định:

    $user->updateDefaultPaymentMethod($paymentMethod);

Để đồng bộ thông tin phương thức thanh toán mặc định của bạn với thông tin phương thức thanh toán mặc định của khách hàng trong Stripe, bạn có thể sử dụng phương thức `updateDefaultPaymentMethodFromStripe`:

    $user->updateDefaultPaymentMethodFromStripe();

> {note} Phương thức thanh toán mặc định của khách hàng chỉ có thể được sử dụng để lập hóa đơn và tạo một subscription mới. Do những hạn chế từ Stripe, nó không thể được sử dụng cho các khoản tính phí một lần.

<a name="adding-payment-methods"></a>
### Thêm phương thức thanh toán

Để thêm một phương thức thanh toán mới, bạn có thể gọi phương thức `addPaymentMethod` trên một model billable user, và truyền identifier phương thức thanh toán:

    $user->addPaymentMethod($paymentMethod);

> {tip} Để tìm hiểu cách lấy identifier phương thức thanh toán, vui lòng xem lại [tài liệu lưu trữ phương thức thanh toán](#storing-payment-methods).

<a name="deleting-payment-methods"></a>
### Xoá phương thức thanh toán

Để xóa một phương thức thanh toán, bạn có thể gọi phương thức `delete` trên instance `Laravel\Cashier\PaymentMethod` mà bạn muốn xóa:

    $paymentMethod->delete();

Phương thức `deletePaymentMethods` sẽ xóa tất cả thông tin về phương thức thanh toán cho một Billable model:

    $user->deletePaymentMethods();

> {note} Nếu người dùng có một subscription đang hoạt động, bạn nên ngăn họ xóa phương thức thanh toán mặc định của họ.

<a name="subscriptions"></a>
## Subscriptions

<a name="creating-subscriptions"></a>
### Tạo Subscription

Để tạo một subscription, trước tiên hãy lấy ra một instance Billable model của bạn, thường là một instance của `App\User`. Khi bạn đã lấy được instance của model, bạn có thể sử dụng phương thức `newSubscription` để tạo ra một subscription cho model:

    $user = User::find(1);

    $user->newSubscription('default', 'premium')->create($paymentMethod);

Tham số đầu tiên được truyền cho phương thức `newSubscription` phải là tên của subscription. Nếu ứng dụng của bạn chỉ cung cấp một loại subscription duy nhất, bạn có thể gọi nó là `default` hoặc `primary`. Tham số thứ hai là gói cụ thể mà người dùng đang subscription. Giá trị này phải tương ứng với identifier của gói trong Stripe.

Phương thức `create`, chấp nhận [một identifier phương thức thanh toán Stripe](#storing-payment-methods) hoặc đối tượng Stripe `PaymentMethod`, sẽ bắt đầu đăng ký subscription cũng như cập nhật cơ sở dữ liệu của bạn với ID của khách hàng và thông tin thanh toán có liên quan khác.

> {note} Việc truyền trực tiếp identifier phương thức thanh toán vào phương thức subscription `create()` cũng sẽ tự động thêm identifier đó vào danh sách các phương thức thanh toán được lưu của người dùng đó.

#### Additional User Details

Nếu bạn muốn thêm chi tiết khách hàng, bạn có thể làm bằng cách truyền chúng làm tham số thứ hai cho phương thức `create`:

    $user->newSubscription('default', 'monthly')->create($paymentMethod, [
        'email' => $email,
    ]);

Để tìm hiểu thêm về các field được hỗ trợ bởi Stripe, hãy xem [tài liệu về tạo khách hàng](https://stripe.com/docs/api#create_customer) của Stripe.

#### Coupons

Nếu bạn muốn áp dụng phiếu giảm giá khi tạo subscription, bạn có thể sử dụng phương thức `withCoupon`:

    $user->newSubscription('default', 'monthly')
         ->withCoupon('code')
         ->create($paymentMethod);

<a name="checking-subscription-status"></a>
### Kiểm tra trạng thái Subscription

Khi người dùng đã subscription vào application của bạn, bạn có thể dễ dàng kiểm tra trạng thái subscription của họ bằng nhiều phương thức thuận tiện khác nhau. Đầu tiên, phương thức `subscribed` sẽ trả về `true` nếu người dùng đó có active subscription, ngay cả khi subscription hiện tại đang trong thời gian dùng thử:

    if ($user->subscribed('default')) {
        //
    }

Phương thức `subscribed` cũng là một cách tuyệt vời cho một [route middleware](/docs/{{version}}/middleware), cho phép bạn lọc quyền truy cập vào các route hoặc các controller dựa trên trạng thái subscription của người dùng:

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('default')) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }

Nếu bạn muốn xác định xem người dùng đó có còn trong thời gian dùng thử hay không, bạn có thể sử dụng phương thức `onTrial`. Phương thức này có thể hữu ích để hiển thị cảnh báo cho người dùng biết rằng họ vẫn đang trong thời gian dùng thử:

    if ($user->subscription('default')->onTrial()) {
        //
    }

Phương thức `subscribedToPlan` có thể được sử dụng để xác định xem người dùng có đăng ký gói dịch vụ đã cho hay không dựa vào ID của gói đó trong Stripe. Trong ví dụ này, chúng ta sẽ xác định xem subscription `default` của người dùng có đăng ký gói `monthly` hay không:

    if ($user->subscribedToPlan('monthly', 'default')) {
        //
    }

Bằng cách truyền một mảng cho phương thức `subscribedToPlan`, bạn có thể xác định xem subscription `default` của người dùng có được đăng ký với gói `monthly` hay `yearly` hay không:

    if ($user->subscribedToPlan(['monthly', 'yearly'], 'default')) {
        //
    }

Phương thức `recurring` có thể được sử dụng để xác định xem người dùng hiện tại có đang đăng ký và không còn trong thời gian dùng thử hay không:

    if ($user->subscription('default')->recurring()) {
        //
    }

#### Cancelled Subscription Status

Để xác định xem người dùng đã từng subscription, nhưng sau đó đã hủy, bạn có thể sử dụng phương thức `cancelled`:

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

<a name="incomplete-and-past-due-status"></a>
#### Incomplete and Past Due Status

Nếu subscription yêu cầu một hành động thanh toán phụ sau khi được tạo, thì subscription sẽ được đánh dấu là `incomplete`. Trạng thái của subscription được lưu trong cột `stripe_status` trong bảng `subscription` của Cashier.

Tương tự, nếu hành động thanh toán phụ được yêu cầu khi hoán đổi gói, subscription sẽ được đánh dấu là `past_due`. Khi subscription của bạn ở một trong hai trạng thái này, subscription sẽ không được active cho đến khi khách hàng xác nhận thanh toán. Để kiểm tra xem subscription có được thanh toán hay chưa, bạn có thể thực hiện bằng cách sử dụng phương thức `hasIncompletePayment` trên Billable model hoặc một instance subscription:

    if ($user->hasIncompletePayment('default')) {
        //
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        //
    }

Khi một subscription có một khoản thanh toán chưa hoàn thành, bạn nên hướng người dùng đến trang xác nhận thanh toán của Cashier, và truyền identifier của `latestPayment`. Bạn có thể sử dụng phương thức `latestPayment` có sẵn trên instance subscription để lấy identifier này:

    <a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
        Please confirm your payment.
    </a>

Nếu bạn muốn một subscription vẫn được coi là hoạt động khi nó ở trạng thái `past_due`, bạn có thể sử dụng phương thức `keepPastDueSubscriptionsActive` do Cashier cung cấp. Thông thường, phương thức này nên được gọi trong phương thức `register` trong `AppServiceProvider` của bạn:

    use Laravel\Cashier\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> {note} Khi một subscription ở trạng thái `incomplete`, bạn sẽ không thể thay đổi subscription cho đến khi xác nhận thanh toán. Do đó, các phương thức `swap` và `updateQuantity` sẽ đưa ra một ngoại lệ khi subscription ở trạng thái `incomplete`.

<a name="changing-plans"></a>
### Thay đổi gói

Sau khi một người dùng đã subscription vào application của bạn, đôi khi họ có thể muốn thay đổi sang gói subscription mới. Để chuyển người dùng sang subscription mới, hãy truyền vào một định danh của gói mới vào phương thức `swap`:

    $user = App\User::find(1);

    $user->subscription('default')->swap('provider-plan-id');

Nếu người dùng đang trong thời gian dùng thử, thì thời gian dùng thử sẽ được duy trì. Ngoài ra, nếu có "nhiều" subscription tồn tại, thì những subscription đó cũng sẽ vẫn được duy trì.

Nếu bạn muốn thay đổi gói và hủy tất cả các gói dùng thử mà người dùng hiện đang sử dụng, bạn có thể sử dụng phương thức `skipTrial`:

    $user->subscription('default')
            ->skipTrial()
            ->swap('provider-plan-id');

Nếu bạn muốn thay đổi gói và lập hóa đơn ngay cho người dùng thay vì đợi đến chu kỳ thanh toán tiếp theo của họ, bạn có thể sử dụng phương pháp `swapAndInvoice`:

    $user = App\User::find(1);

    $user->subscription('default')->swapAndInvoice('provider-plan-id');

#### Prorations

Mặc định, Stripe sẽ tính phí khi hoán đổi giữa các gói. Phương thức `noProrate` có thể được sử dụng để cập nhật subscription mà không bị tính phí:

    $user->subscription('default')->noProrate()->swap('provider-plan-id');

Để biết thêm thông tin về tính phí subscription, hãy tham khảo [tài liệu Stripe](https://stripe.com/docs/billing/subscriptions/prorations).

<a name="subscription-quantity"></a>
### Subscription số lượng lớn

Thỉnh thoảng subscription có thể bị ảnh hưởng bởi "số lượng". Ví dụ: application của bạn có thể tính phí $10 mỗi tháng **cho mỗi người dùng** trên mỗi tài khoản. Để dễ dàng tăng hoặc giảm số lượng subscription của bạn, hãy sử dụng các phương thức `incrementQuantity` hoặc `decrementQuantity`:

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription('default')->incrementQuantity(5);

    $user->subscription('default')->decrementQuantity();

    // Subtract five to the subscription's current quantity...
    $user->subscription('default')->decrementQuantity(5);

Ngoài ra, bạn cũng có thể set một số lượng cụ thể bằng phương thức `updateQuantity`:

    $user->subscription('default')->updateQuantity(10);

Phương thức `noProrate` có thể được sử dụng để cập nhật số lượng của subscription mà không cần chia tỷ lệ phí:

    $user->subscription('default')->noProrate()->updateQuantity(10);

Để biết thêm thông tin về số lượng đăng ký, hãy tham khảo [tài liệu của Stripe](https://stripe.com/docs/subscriptions/quantities).

<a name="subscription-taxes"></a>
### Thuế của Subscription

Để khai báo tỷ lệ phần trăm thuế mà người dùng sẽ phải trả cho một subscription, hãy implement phương thức `taxPercentage` trên model billable của bạn và trả về một giá trị số từ 0 đến 100, không quá 2 chữ số thập phân.

    public function taxPercentage()
    {
        return 20;
    }

Phương thức `taxPercentage` cho phép bạn áp dụng thuế suất cho từng model, có thể có hữu ích cho người dùng trải rộng trên nhiều quốc gia và có các loại thuế suất khác nhau.

> {note} Phương thức `taxPercentage` chỉ áp dụng cho các loại subscription. Nếu bạn sử dụng Cashier để thực hiện các khoản tính phí "một lần", bạn sẽ cần khai báo thuế suất thủ công tại thời điểm đó.

#### Syncing Tax Percentages

Khi thay đổi giá trị được trả về từ phương pháp `taxPercentage`, thì các cài đặt thuế có trên các subscription hiện có của người dùng sẽ vẫn giữ nguyên. Nếu bạn muốn cập nhật giá trị thuế cho các subscription hiện có với giá trị `taxPercentage` được trả về, bạn cần gọi phương thức `syncTaxPercentage` trên instance subscription của user đó:

    $user->subscription('default')->syncTaxPercentage();

<a name="subscription-anchor-date"></a>
### Subscription cố định ngày

Mặc định, ngày cố định thanh toán là ngày đã tạo ra subscription hoặc nếu có thời gian dùng thử, thì ngày dùng thử kết thúc sẽ là ngày thanh toán. Nếu bạn muốn sửa ngày cố định thanh toán, bạn có thể sử dụng phương thức `anchorBillingCycleOn`:

    use App\User;
    use Carbon\Carbon;

    $user = User::find(1);

    $anchor = Carbon::parse('first day of next month');

    $user->newSubscription('default', 'premium')
                ->anchorBillingCycleOn($anchor->startOfDay())
                ->create($paymentMethod);

Để biết thêm thông tin về cách quản lý chu kỳ thanh toán của subscription, hãy tham khảo [tài liệu về chu kỳ thanh toán của Stripe](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### Huỷ Subscription

Để hủy một subscription, hãy gọi phương thức `cancel` trên subscription của người dùng:

    $user->subscription('default')->cancel();

Khi một subscription bị hủy, Cashier sẽ tự động set cột `ends_at` trong cơ sở dữ liệu của bạn. Cột này được sử dụng để biết xem khi nào phương thức `subscribed` sẽ bắt đầu trả về `false`. Ví dụ: nếu khách hàng hủy subscription vào ngày 1 tháng 3, nhưng subscription không thể kết thúc cho đến khi hết ngày 5 tháng 3, thì phương thức `subscribed` vẫn sẽ tiếp tục trả về `true` cho đến ngày 5 tháng 3.

Bạn có thể biết những người dùng đã hủy subscription của họ nhưng vẫn đang trong "thời gian subscription có hiệu lực" bằng cách sử dụng phương thức `onGracePeriod`:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Nếu bạn muốn hủy subscription ngay lập tức, hãy gọi phương thức `cancelNow` trên subscription của người dùng:

    $user->subscription('default')->cancelNow();

<a name="resuming-subscriptions"></a>
### Resume Subscription

Nếu một người dùng đã hủy subscription của họ và bạn muốn resume tiếp subscription đó, hãy sử dụng phương thức `resume`. Để resume một subscription, người dùng vẫn **phải** đang trong thời gian subscription có hiệu lực:

    $user->subscription('default')->resume();

Nếu người dùng đã hủy subscription nhưng sau đó lại muốn resume tiếp subscription đó trước khi subscription hết hạn, họ sẽ không bị tính tiền ngay lập tức. Thay vào đó, subscription của họ sẽ được kích hoạt lại và họ sẽ thanh toán theo đúng chu kỳ thanh toán ban đầu của họ.

<a name="subscription-trials"></a>
## Subscription dành cho dùng thử

> {note} Cashier quản lý ngày dùng thử cho các subscription và không lấy chúng từ Stripe plan. Do đó, bạn nên cấu hình plan của bạn trong Stripe có thời gian dùng thử là 0 ngày để Cashier có thể quản lý thời gian dùng thử.

<a name="with-payment-method-up-front"></a>
### Khai báo trước phương thức thanh toán

Nếu bạn muốn cung cấp thời gian dùng thử cho khách hàng của bạn trong khi vẫn muốn thu thập thông tin thanh toán của khách hàng, bạn nên sử dụng phương thức `trialDays` khi tạo subscription của bạn:

    $user = User::find(1);

    $user->newSubscription('default', 'monthly')
                ->trialDays(10)
                ->create($paymentMethod);

Phương thức này sẽ set ngày kết thúc của thời gian dùng thử vào trong bản ghi subscription trong cơ sở dữ liệu, và sẽ bảo với Stripee là sẽ không tính phí khách hàng cho đến khi hết ngày dùng thử. Khi sử dụng phương thức `trialDays`, Cashier sẽ ghi đè lên bất kỳ khoảng thời gian dùng thử nào được cấu hình cho gói trong Stripe.

> {note} Nếu subscription của khách hàng không bị hủy trước ngày kết thúc dùng thử, họ sẽ bị tính phí ngay khi hết hạn dùng thử, vì vậy bạn nên chắc chắn là đã thông báo cho khách hàng biết về ngày kết thúc dùng thử của họ.

Phương thức `trialUntil` cho phép bạn cung cấp một instance `DateTime` để chỉ định khi nào thời gian dùng thử kết thúc:

    use Carbon\Carbon;

    $user->newSubscription('default', 'monthly')
                ->trialUntil(Carbon::now()->addDays(10))
                ->create($paymentMethod);

Bạn có thể xác định xem người dùng hiện tại có đang trong thời gian dùng thử hay không bằng cách sử dụng phương thức `onTrial` trên instance người dùng hoặc phương thức `onTrial` trên instance subscription. Hai ví dụ dưới đây có kết quả giống hệt nhau:

    if ($user->onTrial('default')) {
        //
    }

    if ($user->subscription('default')->onTrial()) {
        //
    }

<a name="without-payment-method-up-front"></a>
### Khai báo phương thức thanh toán sau

Nếu bạn muốn cung cấp thời gian dùng thử mà không muốn thu thập thông tin thanh toán của người dùng, bạn có thể set cột `trial_ends_at` trong bản ghi của người dùng thành ngày kết thúc dùng thử mà bạn mong muốn. Điều này thường được thực hiện trong quá trình đăng ký người dùng:

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => now()->addDays(10),
    ]);

> {note} Hãy chắc chắn là bạn đã thêm [date mutator](/docs/{{version}}/eloquent-mutators#date-mutators) cho cột `trial_ends_at` trong định nghĩa model của bạn.

Cashier sẽ coi các loại dùng thử như thế này là "dùng thử đại trà", vì nó sẽ không được gắn với bất kỳ thông tin subscription nào. Phương thức `onTrial` trên instance `User` sẽ trả về `true` nếu ngày hiện tại không vượt quá giá trị của ngày `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

Bạn có thể sử dụng phương thức `onGenericTrial` nếu bạn muốn biết người dùng hiện tại có đang trong thời gian dùng thử "đại trà" và chưa tạo bất kỳ thông tin subscription nào hay không:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

Khi bạn đã sẵn sàng tạo một subscription thực sự cho người dùng, bạn có thể sử dụng phương thức `newSubscription` như bình thường:

    $user = User::find(1);

    $user->newSubscription('default', 'monthly')->create($paymentMethod);

<a name="extending-trials"></a>
### Mở rộng thời gian dùng thử

Phương thức `extendTrial` cho phép bạn kéo dài thời gian dùng thử của một subscription sau khi nó được tạo:

    // End the trial 7 days from now...
    $subscription->extendTrial(
        now()->addDays(7)
    );

    // Add an additional 5 days to the trial...
    $subscription->extendTrial(
        $subscription->trial_ends_at->addDays(5)
    );

Nếu bản dùng thử đã hết hạn và khách hàng đã được lập hóa đơn cho subscription, bạn vẫn có thể cung cấp cho họ thêm thời gian dùng thử. Thời gian sử dụng trong thời gian dùng thử sẽ được trừ vào hóa đơn tiếp theo của khách hàng.

<a name="handling-stripe-webhooks"></a>
## Xử lý Stripe Webhooks

> {tip} Bạn có thể sử dụng [the Stripe CLI](https://stripe.com/docs/stripe-cli) để giúp kiểm tra webhook trong quá trình phát triển local.

Stripe có thể thông báo cho ứng dụng của bạn về nhiều loại event khác nhau thông qua webhooks. Mặc định, sẽ có một route sẽ trỏ đến controller webhook của Cashier được cấu hình thông qua service provider của Cashier. Controller này sẽ xử lý tất cả các request webhook đến.

Mặc định, controller này sẽ tự động xử lý việc hủy subscription khi có quá nhiều lần tính phí không thành công (được định nghĩa trong cài đặt Stripe của bạn), cập nhật khách hàng, xóa khách hàng, cập nhật subscription và thay đổi phương thức thanh toán; tuy nhiên, bạn sẽ sớm khám phá ra là bạn có thể extend controller này để xử lý bất kỳ event webhook nào bạn thích.

Để đảm bảo ứng dụng của bạn có thể xử lý Stripe webhook, hãy nhớ cấu hình URL webhook trong bảng điều khiển Stripe. Danh sách đầy đủ của tất cả các webhook mà bạn nên cấu hình trong bảng điều khiển Stripe sẽ là:

- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `invoice.payment_action_required`

> {note} Hãy đảm bảo là bạn đã bảo vệ các request bằng các middleware [kiểm tra webhook signature](/docs/{{version}}/billing#verifying-webhook-signatures) của Cashier.

#### Webhooks & CSRF Protection

Vì các webhook của Stripe cần bỏ qua bước [bảo vệ CSRF](/docs/{{version}}/csrf) của Laravel, nên bạn hãy chắc chắn là đã khai báo URI của Stripe là một ngoại lệ trong middleware `VerifyCsrfToken` của bạn hoặc bạn có thể khai báo route này ra khỏi group middleware `web`:

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### Định nghĩa xử lý event Webhook

Cashier sẽ tự động xử lý hủy subscription nếu như các lần trả phí không thành công, nhưng nếu bạn muốn thêm các event webhook mà bạn muốn tự xử lý, thì hãy extend controller Webhook. Tên phương thức của bạn phải tương ứng với quy ước của Cashier, cụ thể là, các phương thức sẽ cần được thêm tiền tố là `handle` vào tên của webhook mà bạn muốn xử lý, theo kiểu "camel case". Ví dụ: nếu bạn muốn xử lý webhook `invoice.payment_succeeded`, thì bạn cần thêm một phương thức `handleInvoicePaymentSucceeded` vào controller:

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle invoice payment succeeded.
         *
         * @param  array  $payload
         * @return \Symfony\Component\HttpFoundation\Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }

Tiếp theo, định nghĩa một route đến Cashier controller của bạn trong file `routes/web.php`. Điều này sẽ ghi đè lên route mặc định của Cashier:

    Route::post(
        'stripe/webhook',
        '\App\Http\Controllers\WebhookController@handleWebhook'
    );

Cashier sẽ phát ra một event `Laravel\Cashier\Events\WebhookReceived` khi nhận được một webhook và một event `Laravel\Cashier\Events\WebhookHandled` khi một webhook đã được xử lý bởi Cashier. Cả hai event đều chứa toàn bộ payload của webhook Stripe.

<a name="handling-failed-subscriptions"></a>
### Subscription thất bại

Điều gì sẽ xảy ra nếu thẻ tín dụng của khách hàng hết hạn? Đừng lo lắng - Controller Webhook của Cashier sẽ hủy subscription của khách hàng cho bạn. Các khoản thanh toán không thành công sẽ được tự động kiểm soát và xử lý bởi controller. Controller này sẽ hủy subscription của khách hàng khi Stripe xác định rằng subscription không thành công (thông thường là sau ba lần thanh toán không thành công).

<a name="verifying-webhook-signatures"></a>
### Kiểm tra Webhook Signatures

Để bảo mật webhook của bạn, bạn có thể sử dụng [webhook signature của Stripe](https://stripe.com/docs/webhooks/signatures). Để thuận tiện, Cashier đã tự động thêm một middleware để kiểm tra các request webhook Stripe đến application là hợp lệ.

Để bật kiểm tra webhook, hãy đảm bảo rằng biến môi trường `STRIPE_WEBHOOK_SECRET` được set trong file `.env` của bạn. Webhook `secret` có thể được lấy ra từ trang tổng quan trong tài khoản Stripe của bạn.

<a name="single-charges"></a>
## Phí

<a name="simple-charge"></a>
### Tính phí một lần

> {note} phương thức `charge` chấp nhận số tiền mà bạn muốn tính phí theo **loại tiền được set trong application của bạn**.

Nếu bạn muốn thực hiện một khoản tính phí "một lần" đối với phương thức thanh toán của khách hàng đã subscription, bạn có thể sử dụng phương thức `charge` trên một instance billable model. Bạn sẽ cần [cung cấp identifier phương thức thanh toán](#storing-payment-methods) làm tham số thứ hai cho phương thức:

    // Stripe Accepts Charges In Cents...
    $stripeCharge = $user->charge(100, $paymentMethod);

Phương thức `charge` chấp nhận một mảng làm tham số thứ ba của nó, cho phép bạn truyền vào bất kỳ tùy chọn nào mà bạn muốn cho việc tạo phí của Stripe. Tham khảo tài liệu của Stripe về các tùy chọn có sẵn cho bạn khi tạo phí:

    $user->charge(100, $paymentMethod, [
        'custom_option' => $value,
    ]);

Phương thức `charge` sẽ đưa ra một ngoại lệ nếu việc tính phí không thành công. Nếu tính phí thành công, thì một instance của `Laravel\Cashier\Payment` sẽ được trả về từ phương thức:

    try {
        $payment = $user->charge(100, $paymentMethod);
    } catch (Exception $e) {
        //
    }

<a name="charge-with-invoice"></a>
### Tính phí với hoá đơn

Thỉnh thoảng bạn có thể cần phải tạo tính phí một lần nhưng cũng cần tạo cả một hóa đơn cho khoản phí đó để bạn có thể cung cấp hóa đơn PDF đó cho khách hàng của bạn. Phương thức `invoiceFor` cho phép bạn làm điều đó. Ví dụ: hãy gửi hóa đơn "phí một lần" cho khách hàng của bạn là $5.00:

    // Stripe Accepts Charges In Cents...
    $user->invoiceFor('One Time Fee', 500);

Hóa đơn sẽ được tính ngay lập tức cho phương thức thanh toán mặc định của người dùng. Phương thức `invoiceFor` cũng chấp nhận một mảng làm tham số thứ ba của nó. Mảng này sẽ chứa các tùy chọn thanh toán cho các hàng được thanh toán. Tham số thứ tư được phương thức chấp nhận cũng là một mảng. Tham số cuối cùng này chấp nhận các tùy chọn thanh toán cho chính hóa đơn đó:

    $user->invoiceFor('Stickers', 500, [
        'quantity' => 50,
    ], [
        'tax_percent' => 21,
    ]);

> {note} Phương thức `invoiceFor` sẽ tạo ra một hóa đơn Stripe sẽ thử lại sau các lần thanh toán không thành công. Nếu bạn không muốn hóa đơn thử lại sau các lần trả phí không thành công, bạn sẽ cần phải close chúng bằng API Stripe sau lần tính phí không thành công đầu tiên.

<a name="refunding-charges"></a>
### Hoàn trả

Nếu bạn cần hoàn trả một phí đã được thanh toán trong Stripe, bạn có thể sử dụng phương thức `refund`. Phương thức này chấp nhận một ID của Payment Intent Stripe làm tham số đầu tiên của nó:

    $payment = $user->charge(100, $paymentMethod);

    $user->refund($payment->id);

<a name="invoices"></a>
## Hoá đơn

Bạn có thể dễ dàng lấy ra một mảng các hóa đơn của một model billable bằng cách sử dụng phương thức `invoices`:

    $invoices = $user->invoices();

    // Include pending invoices in the results...
    $invoices = $user->invoicesIncludingPending();

Khi liệt kê hóa đơn cho khách hàng, bạn có thể sử dụng các phương thức helper của hóa đơn để hiển thị thông tin hóa đơn. Ví dụ: bạn có thể muốn liệt kê tất cả các hóa đơn có trong một bảng, cho phép người dùng dễ dàng tải xuống bất kỳ cái nào trong số chúng:

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="generating-invoice-pdfs"></a>
### Tạo hoá đơn PDF

Từ trong một route hoặc một controller, sử dụng phương thức `downloadInvoice` để tạo một bản PDF cho hóa đơn để khách hàng có thể tải xuống. Phương thức này sẽ tự động tạo ra một response HTTP để gửi file download tới trình duyệt:

    use Illuminate\Http\Request;

    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor' => 'Your Company',
            'product' => 'Your Product',
        ]);
    });

<a name="strong-customer-authentication"></a>
## Strong Customer Authentication

Nếu doanh nghiệp của bạn có trụ sở tại Châu Âu, bạn sẽ cần tuân thủ các quy định về Strong Customer Authentication (SCA). Các quy định này đã được Liên minh Châu Âu áp dụng vào tháng 9 năm 2019 để ngăn chặn gian lận trong thanh toán. May mắn thay, Stripe và Cashier đã chuẩn bị sẵn sàng để xây dựng các ứng dụng tuân thủ theo quy định SCA này.

> {note} Trước khi bắt đầu, hãy xem [hướng dẫn của Stripe về PSD2 và SCA](https://stripe.com/en-be/guides/strong-customer-authentication) cũng như [tài liệu mới của họ về API cho SCA](https://stripe.com/docs/strong-customer-authentication).

<a name="payments-requiring-additional-confirmation"></a>
### Thanh toán yêu cầu xác nhận bổ sung

Các quy định của SCA thường yêu cầu xác minh thêm để xác nhận và xử lý một khoản thanh toán. Khi điều này xảy ra, Cashier sẽ đưa ra một ngoại lệ `IncompletePayment` để thông báo cho bạn biết là cần phải xác minh thêm. Sau khi lấy được ngoại lệ này, bạn có hai tùy chọn về cách tiếp tục.

Một là, bạn có thể chuyển hướng khách hàng của bạn đến trang xác nhận thanh toán chuyên dụng đi kèm với Cashier. Trang này đã có một route liên kết được đăng ký thông qua service provider của Cashier. Vì vậy, bạn có thể catch ngoại lệ `IncompletePayment` và chuyển hướng người dùng đến trang xác nhận thanh toán:

    use Laravel\Cashier\Exceptions\IncompletePayment;

    try {
        $subscription = $user->newSubscription('default', $planId)
                                ->create($paymentMethod);
    } catch (IncompletePayment $exception) {
        return redirect()->route(
            'cashier.payment',
            [$exception->payment->id, 'redirect' => route('home')]
        );
    }

Trong trang xác nhận thanh toán, khách hàng sẽ được yêu cầu nhập lại các thông tin thẻ tín dụng của họ và thực hiện thêm bất kỳ hành động nào theo yêu cầu của Stripe, chẳng hạn như xác nhận "3D Secure". Sau khi xác nhận thanh toán của họ, người dùng sẽ được chuyển hướng đến URL được cung cấp bởi tham số `redirect` được chỉ định ở trên.

Ngoài ra, bạn có thể cho phép Stripe xử lý xác nhận thanh toán cho bạn. Trong trường hợp này, thay vì chuyển hướng người dùng đến trang xác nhận thanh toán, bạn có thể [thiết lập email thanh toán tự động của Stripe](https://dashboard.stripe.com/account/billing/automatic) trong trang tổng quan của Stripe. Tuy nhiên, nếu gặp trường hợp ngoại lệ `IncompletePayment` này, bạn vẫn nên thông báo cho người dùng biết rằng họ sẽ nhận được email kèm theo hướng dẫn xác nhận thanh toán thêm.

Các ngoại lệ incomplete payment có thể được đưa ra trong các phương thức sau: `charge`, `invoiceFor`, và `invoice` trên một `Billable` user. Khi xử lý các subscription, phương thức `create` trên `SubscriptionBuilder`, và các phương thức `incrementAndInvoice` và `swapAndInvoice` trên model `Subscription` cũng có thể đưa ra các ngoại lệ.

#### Incomplete and Past Due State

Khi một khoản thanh toán yêu cầu xác nhận bổ sung, subscription sẽ vẫn ở trạng thái `incomplete` hoặc `past_due`, là giá trị cột `stripe_status`. Cashier sẽ tự động kích hoạt subscription của khách hàng qua webhook ngay sau khi xác nhận thanh toán hoàn tất.

Để biết thêm thông tin về trạng thái `incomplete` và `past_due`, vui lòng tham khảo [thêm tài liệu của chúng tôi](#incomplete-and-past-due-status).

<a name="off-session-payment-notifications"></a>
### Thông báo thanh toán Off-Session

Vì các quy định của SCA yêu cầu khách hàng thỉnh thoảng cần xác minh chi tiết thanh toán của họ ngay cả khi subscription của họ đang hoạt động, nên Cashier có thể gửi thông báo thanh toán cho khách hàng khi yêu cầu xác nhận thanh toán off-session. Ví dụ: điều này có thể xảy ra khi subscription đang được gia hạn. Thông báo thanh toán của Cashier có thể được bật bằng cách set biến môi trường `CASHIER_PAYMENT_NOTIFICATION` thành một class thông báo. Mặc định, thông báo này sẽ bị tắt. Tất nhiên, Cashier có chứa một class thông báo mà bạn có thể sử dụng cho mục đích này, nhưng bạn có thể tự do thêm class thông báo của bạn nếu muốn:

    CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment

Để đảm bảo rằng thông báo xác nhận thanh toán off-session sẽ được gửi, hãy chắc chắn rằng [Stripe webhooks đã được cấu hình](#handling-stripe-webhooks) cho ứng dụng của bạn và webhook `invoice.payment_action_required` được bật trong trang tổng quan Stripe của bạn. Ngoài ra, model `Billable` của bạn cũng nên sử dụng trait `Illuminate\Notifications\Notifiable` của Laravel.

> {note} Thông báo sẽ được gửi ngay cả khi khách hàng đang tự thực hiện thanh toán và nhận yêu cầu xác nhận bổ sung. Thật không may, không có cách nào để Stripe biết rằng một khoản thanh toán là được một khách hàng tự thực hiện hay là thông qua "off-session". Tuy nhiên, khách hàng sẽ chỉ nhận được thông báo "Thanh toán thành công" nếu họ truy cập vào trang thanh toán sau khi đã xác nhận thanh toán của mình. Khách hàng sẽ không được phép xác nhận cùng một khoản thanh toán tới hai lần và chịu khoản phí đến lần thứ hai.

<a name="stripe-sdk"></a>
## Stripe SDK

Nhiều đối tượng của Cashier là các wrapper của các đối tượng Stripe SDK. Nếu bạn muốn tương tác trực tiếp với các đối tượng Stripe, bạn có thể lấy ra chúng bằng phương thức `asStripe`:

    $stripeSubscription = $subscription->asStripeSubscription();

    $stripeSubscription->update($subscription->stripe_id, ['application_fee_percent' => 5]);
